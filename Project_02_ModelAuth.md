# Project 2 — ModelAuth

**Repository:** https://github.com/praneeth3696/modelauth
**Branches:** `main` (unified three-tier benchmark), `easy` (cross-architecture + cold-start), `medium+hard` (scale and quantisation shifts)
**Stack:** Python 3.10+, NumPy, SciPy, Matplotlib, Chart.js, Ollama (local LLM server), OpenAI-compatible client
**Scale:** 3 difficulty tiers × 30 runs per tier (15 null + 15 substitution) × 400 probes each, plus 75 cold-start contamination runs — roughly 60,000 probes total.

This is the most technically distinctive project on my resume. It is a statistics project applied to LLMs, and the interviewer will most likely ask *why* the idea works before asking how it is built.

---

## 1. The one-minute pitch

"When you pay for an LLM API, you send text and get text back. You never see the weights, the log-probabilities, or the internal states — so you have no way to verify which model actually produced the response. TLS and API keys authenticate the *host*, not the *weights*. Providers have a direct financial incentive to silently route traffic to a smaller or more heavily quantised model to cut GPU costs, and you would not be able to tell.

ModelAuth detects that. It interleaves cheap single-token probes — 'pick a random number between 1 and 100' — alongside normal traffic, and treats the resulting stream of numbers as a statistical process. Different models have measurably different output distributions on open-ended stochastic queries, so a substitution shows up as a change point in that stream. I implemented four sequential change-point detectors and benchmarked them across three difficulty tiers, from swapping Llama for Qwen down to swapping 4-bit for 8-bit quantisation of the same model. The best detectors flag a cross-architecture swap within about 11 to 15 probes at a false alarm rate below half a percent."

## 2. The problem, stated properly

**Black-box API asymmetry.** A commercial LLM endpoint exposes only an HTTP `generate` or `chat` call returning text. Weights, token log-probabilities, and hidden states are not observable. Some providers expose log-probabilities, but many do not, and a provider intent on substituting could simply stop.

**Traditional security does not help.** TLS proves you are talking to the right *server*. mTLS and API tokens prove *identity of the host*. None of them says anything about which model weights generated the tokens. There is no attestation mechanism in the current API ecosystem.

**The economic incentive is real.** Serving a 3-billion-parameter model costs meaningfully more GPU memory bandwidth than a 1-billion-parameter one, and an 8-bit quantisation costs more than a 4-bit one. Under load, silently routing a fraction of traffic to the cheaper variant is invisible to the customer and directly improves margin. This is a **supply-chain integrity** problem — the same category as a dependency being swapped for a malicious version, except the artefact is a model and there is no hash to check.

**Constraints I set for the solution**, which are what make it interesting:
- No access to weights, log-probabilities, or activations.
- No reference dataset from the provider.
- No cooperation from the provider.
- Must work online, sequentially, on a live stream — not as a post-hoc batch analysis.
- Must be cheap: a probe is 5 tokens.

## 3. The core insight

Every autoregressive language model has an intrinsic probabilistic fingerprint when asked an **open-ended stochastic** question. Ask a model "pick a random number between 1 and 100" a few hundred times at temperature 1.0 and you do not get a uniform distribution — you get that model's characteristic bias. Different architectures, different parameter counts, and even different quantisation levels of the same weights produce measurably different empirical distributions P(X).

So the problem reduces to: **given a sequence of samples, detect the point at which the underlying distribution changed.** That is a classical sequential change-point detection problem, and there is a well-developed statistical literature for it.

**Why temperature 1.0 matters**: the whole method depends on the output *distribution* being visible. At temperature 0 the model is near-deterministic and there is no distribution to compare. This is a design decision I can defend if challenged.

**Why `max_tokens = 5`**: a probe only needs a number, so each probe is trivially cheap. Three probe templates are rotated so the detector is not fingerprinting a single exact prompt string.

## 4. The four detectors

All four take a list of numeric answers and return, for each index, whether an alarm fired — one uniform contract, which is what made a fair benchmark possible.

### 4.1 `v1 naive` — sliding-window two-sample Kolmogorov-Smirnov
```python
def sliding_window_detector(numeric_answers, window_size=20, alpha=0.01):
    for t in range(2 * window_size, len(numeric_answers) + 1):
        baseline_window = numeric_answers[t - 2*window_size : t - window_size]
        recent_window   = numeric_answers[t - window_size : t]
        stat, p_value = ks_2samp(baseline_window, recent_window)
        flagged = p_value < alpha
```
Compares two adjacent rolling windows of 20 samples each. The **KS statistic** is D = sup|F₁(x) − F₂(x)| — the maximum vertical distance between the two empirical cumulative distribution functions. It is **non-parametric**, meaning it assumes no particular distribution shape, which matters because I have no theory of what an LLM's number distribution should look like.
**Strength**: fast and clean for large distributional changes, with zero false alarms in my easy tier.
**Weakness**: it only ever compares the last 40 samples, so it has no memory. A small persistent shift never produces a big enough within-window difference — which is exactly what happened, and its power collapsed to 14.29% on the medium tier.

### 4.2 `adaptive CUSUM` — cumulative sum with a self-estimated baseline
```python
z = (answers[t] - mu) / sigma            # standardise against a rolling baseline
pos_cusum = max(0, pos_cusum + z - k)    # accumulate positive drift, allowance k
neg_cusum = min(0, neg_cusum + z + k)    # and negative drift
flagged = (pos_cusum > h) or (abs(neg_cusum) > h)
```
Warmup of 40 observations seeds the baseline mean and standard deviation, which are then re-estimated from a rolling window that is capped so it does not grow unbounded. `k = 0.5` is the allowance — the drift it tolerates as noise. `h = 5.0` is the decision threshold.
**Why it beats the KS test on subtle shifts**: CUSUM **accumulates** evidence rather than forgetting it. A shift too small to show up in any 20-sample window still pushes z consistently in one direction, and the sum grows until it crosses h. That is why it reached **92.86% power on the medium tier** where the sliding window managed 14.29%.
On flagging it resets the statistic and reseeds the baseline from post-flag data, so it can detect a second change.

### 4.3 `DAS-CUSUM` — dispersion-aware CUSUM
```python
symmetric_stat = 0.5 * (z**2 - 1)   # expectation ~0 under the null
pos_cusum = max(0, pos_cusum + symmetric_stat - k)
```
Uses the **second moment** rather than the first. Because E[z²] = 1 under the null hypothesis, the statistic 0.5(z² − 1) has expectation zero when nothing has changed, and grows for **either a mean shift or a variance shift**. A plain CUSUM on the mean is blind to a change that widens the distribution without moving its centre; this is not.
**Result**: slower (higher detection delay) but achieved a **0.00% false alarm rate** on the medium tier — the most conservative detector.

### 4.4 `fixed-reference` — held-out baseline
Compares incoming batches against a pristine, archived reference stream (`null_rep14.jsonl`) using the same KS test.
**Result**: **100% detection power** on both the easy and medium tiers — the highest possible.
**The catch, and it is the important part**: it requires that you captured a trustworthy baseline *before* any substitution occurred. That assumption is exactly what the self-baselining detectors avoid needing, and it is exactly what the cold-start experiment tests.

## 5. The three difficulty tiers

| Tier | Substitution | Nature | Separability |
|---|---|---|---|
| **Easy** | `llama3.2:3b` → `qwen2.5:3b` | Different architecture entirely | KS = 0.659, p < 10⁻²⁷⁰ |
| **Medium** | `llama3.2:1b` → `llama3.2:3b` | Same family, different capacity | KS = 0.402 |
| **Hard** | `llama3.2:3b-instruct-q4_K_M` → `q8_0` | Identical architecture and parameter count, different quantisation precision | Smallest |

Designing the tiers this way was deliberate: it turns "does this work?" into "where does it stop working?", which is a far more useful and far more defensible result.

## 6. Results — know these numbers

Ground-truth switch at probe T = 200 in a 400-probe stream, with strict post-switch delay accounting (only alarms at index ≥ 200 count as detections, which fixed an earlier bug where pre-switch false alarms were being counted as negative delays).

| Tier | Detector | Mean delay | Power | False alarm rate |
|---|---|---|---|---|
| Easy | v1 naive (KS) | +15.33 | 85.71% | 0.00% |
| Easy | adaptive CUSUM | **+11.00** | 78.57% | 0.42% |
| Easy | DAS-CUSUM | +53.00 | 57.14% | 0.38% |
| Easy | fixed-reference | +20.00 | **100.00%** | 0.36% |
| Medium | v1 naive (KS) | +14.50 | **14.29%** | 0.16% |
| Medium | adaptive CUSUM | +41.15 | **92.86%** | 0.08% |
| Medium | DAS-CUSUM | +83.55 | 78.57% | **0.00%** |
| Medium | fixed-reference | +22.86 | **100.00%** | 0.75% |
| Hard | v1 naive (KS) | +126.00 | 28.57% | 0.00% |
| Hard | adaptive CUSUM | +71.20 | **71.43%** | 0.58% |
| Hard | DAS-CUSUM | +88.75 | 57.14% | 0.54% |
| Hard | fixed-reference | +90.00 | 14.29% | 0.36% |

**The three findings to state:**
1. **Cross-architecture substitutions are easy to catch** — distinct distribution modes give detection in 11–15 probes with essentially no false alarms.
2. **Windowed tests fail on subtle shifts; accumulating tests do not.** The medium tier is the headline: 14.29% versus 92.86% on the same data. That is the strongest single result in the project because it is a clean, explainable difference in method rather than a tuning artefact.
3. **Even a pure quantisation change is detectable** — 71.43% power on 4-bit versus 8-bit weights, which means output distribution alone carries information about weight precision. That surprised me.

## 7. The cold-start contamination experiment (research question 2)

**The question**: self-baselining detectors estimate "normal" from the stream's own early history. What if you start monitoring an endpoint that is *already* partially compromised?

**The experiment**: 75 independent runs with pre-switch contamination fractions f ∈ {0.0, 0.25, 0.50, 0.75, 1.00} — that fraction of the warmup period already served by the substitute model.

**The finding**: detectors keep **above 85% post-warmup recovery power up to 25% initial contamination**. Beyond 50%, the self-baselining reference statistics assimilate the substitute's distribution and the detector concludes that the substitute *is* normal — so it can no longer see it.

**Why this matters**: it establishes the boundary of the method's core assumption, and it is the argument for periodically re-verifying against a trusted held-out baseline rather than relying on self-baselining forever. A result that says "here is where my approach breaks" is more credible than one that only reports successes.

## 8. Architecture

```
substitution-sim/
  config.py                     model pairs, probe templates, temperature, switch point
  probe_client.py               Ollama REST client (OpenAI-compatible), retries with backoff
  simulator.py                  stream generator; ThreadPoolExecutor for concurrency
  run_experiments.py            resumable suite runner
  run_cold_start_experiment.py  contamination stream generator
  data_loader.py                regex numeric parser, JSONL loader
  detector_v1.py                sliding-window KS
  detector_cusum.py             adaptive CUSUM
  detector_das_cusum.py         DAS-CUSUM
  detector_fixed_reference.py   held-out baseline
  evaluate.py                   multi-tier benchmark, computes power/delay/false alarms
  data/                         JSONL streams, 30 reps per tier + cold start

final-analysis/
  sanity_checks.py              completeness and separability audit
  visualizations.py             Matplotlib traces, ROC curves, contamination plots
  interactive_dashboard.py      generates a standalone Chart.js HTML dashboard
  run_final_steps.py            master runner
  figures/                      PNGs, CSV summary tables, dashboard.html
```

**Engineering points worth mentioning:**
- **JSONL** for the streams: append-only, one record per line, so a run can be resumed and streamed without loading everything into memory. A reasonable answer to "why not a database?" — write-once, read-sequentially scientific data with no concurrent writers and no relational queries.
- **`ThreadPoolExecutor` with 8 workers** for probing. This is the right choice because the work is **I/O bound** (waiting on HTTP), so Python's GIL is released during the wait and threads genuinely help. CPU-bound work would need `multiprocessing`.
- **Retries with backoff** in the probe client, because a local model server does occasionally time out, and a failed probe is recorded as `failed: True` rather than silently dropped.
- **A resumable runner**, because generating 60,000 probes takes hours.
- **`data_loader.py` parses the numeric answer with a regex**, because a model asked for "just the number" will still sometimes return "The number is 42." Records that yield no number are handled explicitly.

## 9. Why Ollama and not a commercial API

This is the question that tests whether I understand my own experiment. **I need ground truth.** To measure detection delay I must know the exact probe index at which the model changed. No commercial provider will substitute a model on demand, and I could not verify it if they did. Hosting both models locally with Ollama lets me switch at a known index, producing labelled data.

It also gave me **controlled difficulty**: I could pick model pairs that differ by architecture, by capacity, or by quantisation alone. And practically — 60,000 probes with no per-token cost, no rate limits, and no data leaving the machine.

Ollama exposes an **OpenAI-compatible endpoint on port 11434**, so the client code is `OpenAI(base_url="http://localhost:11434/v1", api_key="ollama")` — meaning the same code would work unchanged against a real provider. That is deliberate: the simulation is a stand-in for a deployment, not a different system.

## 10. Questions they will ask, with answers

**"Isn't this just anomaly detection?"**
It is a specific and harder case. Ordinary anomaly detection asks "is this single point unusual?" This asks "has the *distribution generating the points* changed, and when?" — a sequential change-point problem, where the three axes are detection power, detection delay, and false alarm rate, and they trade off against each other. Reporting only accuracy would hide that a detector which alarms constantly has perfect power and is useless.

**"How do you know the model changed and it is not just randomness?"**
That is precisely what the null streams control for. For every tier I generated 15 **null** runs where no substitution occurs and 15 **substitution** runs where it does. The false alarm rate is measured on the null streams. A detector is only credible if it has high power on the substitution streams *and* a low false alarm rate on the null streams — which is why the results table reports both, and why the 0.00% false alarm rates matter as much as the 92.86% power.

**"Why the KS test rather than a chi-squared test or a t-test?"**
A t-test only detects a mean shift and assumes normality. A chi-squared goodness-of-fit test requires binning, and the choice of bins affects the result. The KS test is non-parametric, works on the full empirical CDF, needs no binning, and detects any distributional difference, not just a shift in location. Given that I have no model of what an LLM's answer distribution should look like, assuming as little as possible is the right call.

**"What are the limitations?"**
Several, and I would rather name them than be caught by them.
1. **The probe is detectable.** A sophisticated provider could recognise "pick a random number" prompts and route them to the expensive model. Defences would be probe diversification and embedding probes in traffic-shaped requests, which I have not built.
2. **Cold-start contamination above 50% defeats self-baselining**, as the experiment shows.
3. **A single probe type is a single fingerprint.** More probe families would be more robust.
4. **Detection delay is not zero.** A provider substituting for only a few requests would slip through.
5. **The thresholds (k = 0.5, h = 5.0, window 20, α = 0.01) were tuned empirically**, not derived from a target false-alarm rate. A more principled approach would set h from a desired average run length under the null.

**"What would you build next?"**
Multiple probe families with an ensemble vote; principled threshold derivation from a target average-run-length; testing against real commercial APIs to establish a baseline false-alarm rate in the wild (where legitimate model *updates* would also fire the detector, which is a genuinely interesting confound); and packaging it as a middleware library that sits in front of an existing API client.

**"What did you learn?"**
That the choice of statistical method was the entire project. Four detectors on identical data gave 14% and 92% power on the same tier, and the difference was purely whether the method accumulated evidence or forgot it. It also taught me to design experiments that find the failure boundary — the cold-start contamination result and the hard tier are more useful than the easy tier, because they say where the method stops working.

**"You have odd branch names and a file called FINNNNNNAAAAALreport.md."**
I would be honest and slightly self-deprecating about that: the branches are experiment tiers, which keeps each tier's data and results reproducible in isolation, though a directory or a tag would be the more conventional choice for something that is not divergent code. The report filename is the natural consequence of a long project and I should rename it.
