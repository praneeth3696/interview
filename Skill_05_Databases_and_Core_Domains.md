# Technical Skills 5 — Database Management (Oracle, MySQL, MongoDB) and Core Domains (AI, ML, Information Security)

This covers the last two lines of my resume's technical skills. The DBMS *theory* is in **DBMS_Interview_Prep.md**; Part A here is about the three specific products and how I used them. Part B covers the three core domains. Information Security has its own dedicated file (**Cybersecurity_Domain.md**) because that is the domain I am choosing, so Part B keeps it brief and points there.

---

# PART A — Database Management

## A1. The three products at a glance
| | Oracle | MySQL | MongoDB |
|---|---|---|---|
| Model | Object-relational | Relational | Document (NoSQL) |
| Query language | SQL + PL/SQL | SQL | MQL / aggregation pipeline |
| Schema | Fixed, strict | Fixed, strict | Flexible, optional validation |
| Licence | Commercial (free XE edition) | Open source (Oracle-owned) | Source-available (SSPL) |
| Scaling | Vertical, RAC for clustering | Vertical, read replicas | Horizontal, native sharding |
| Transactions | Full ACID, mature | Full ACID with InnoDB | ACID per document; multi-document since 4.0 |
| Typical use | Large enterprise, banking, ERP | Web applications, LAMP stack | Content, catalogues, evolving schemas, high write volume |
| Default port | 1521 | 3306 | 27017 |

## A2. Oracle
- **Object-relational**: beyond ordinary tables it supports `CREATE TYPE` object types, **VARRAYs**, **nested tables**, `REF` types, and type inheritance. These are genuinely Oracle-specific and cannot be faithfully emulated, which is why my ClassRoom Code platform connects to a **real Oracle server** for object-relational lab questions rather than pretending to run them. Without a connection it says Oracle is not connected instead of returning a wrong answer.
- **PL/SQL** is the procedural extension: `DECLARE / BEGIN / EXCEPTION / END` blocks, explicit cursors, procedures, functions, **packages** (a spec plus a body, grouping related routines), and triggers.
- **Sequences** (`seq.NEXTVAL`) generate surrogate keys; **synonyms** alias objects; `DUAL` is the one-row dummy table used to evaluate expressions (`SELECT SYSDATE FROM DUAL`).
- **DDL commits implicitly** — a `CREATE TABLE` ends the current transaction and cannot be rolled back. This directly affected my project: I roll back each run afterwards, but because DDL self-commits, real deployment needs a **separate schema per student** rather than a shared login.
- **Identifier case**: Oracle upper-cases unquoted identifiers; PostgreSQL lower-cases them. My grader compares result-set column names case-insensitively for exactly this reason — a student should not fail because the engine changed the case of their column alias.
- Other Oracle idioms: `ROWNUM` and `ROWID`, `NVL` (Oracle's `COALESCE` for two arguments), `DECODE`, hierarchical queries with `CONNECT BY`, and tablespaces for physical storage management.

## A3. MySQL
- **Storage engines** are the distinguishing feature. **InnoDB** (the default since 5.5) is transactional and ACID compliant, uses row-level locking and MVCC so readers do not block writers, supports foreign keys, and clusters the table on the primary key. **MyISAM** is the older engine: table-level locking, no transactions, no foreign keys, faster for read-only workloads, and now largely obsolete.
- Default isolation level is **REPEATABLE READ**, which is unusual — PostgreSQL and Oracle default to READ COMMITTED. MySQL additionally uses **gap locks** so phantoms are prevented at REPEATABLE READ.
- `AUTO_INCREMENT` for surrogate keys; `LIMIT`/`OFFSET` for pagination; backtick-quoted identifiers.
- **Replication**: a primary writes a binary log that replicas replay, giving read scaling and failover. Asynchronous by default, so a replica can lag.
- MySQL versus PostgreSQL is a fair question: PostgreSQL is stricter about standards, has richer types (arrays, JSONB, composite types, ranges), better support for complex queries and window functions, and true MVCC without gap locks; MySQL historically had simpler replication and was faster for simple read-heavy workloads. I used PostgreSQL for ClassRoom Code because I needed arrays, JSONB, and strict constraint enforcement.

## A4. MongoDB
- **Document model**: BSON documents in collections, no fixed schema. Mapping: database → database, table → **collection**, row → **document**, column → **field**, join → **`$lookup`**, primary key → **`_id`** (an ObjectId, which encodes a timestamp, machine id, process id, and counter).
- **CRUD**: `insertOne`/`insertMany`, `find({ age: { $gt: 21 } })`, `updateOne({...}, { $set: {...} })`, `deleteOne`. Query operators: `$eq $ne $gt $gte $lt $lte $in $nin $and $or $not $exists $regex`.
- **Aggregation pipeline** — a sequence of stages, each transforming the stream:
```js
db.orders.aggregate([
  { $match: { status: "shipped" } },              // filter first so it can use an index
  { $unwind: "$items" },                          // one document per array element
  { $group: { _id: "$customerId", total: { $sum: "$items.price" } } },
  { $lookup: { from: "customers", localField: "_id", foreignField: "_id", as: "customer" } },
  { $sort: { total: -1 } },
  { $limit: 10 }
])
```
  Put `$match` and `$limit` as early as possible — that is the single biggest aggregation optimisation.
- **Embedding versus referencing**: embed when the child is always read with the parent, is bounded in size, and does not change independently (an address inside a user). Reference when the data is large, shared between parents, or grows without bound (comments on a popular post). The 16 MB document size limit eventually forces referencing.
- **Indexes**: single-field, compound (with the same leftmost-prefix rule as SQL), multikey (automatic on array fields), text, geospatial, and TTL indexes that expire documents automatically.
- **Replica set** — one primary and several secondaries with automatic election on failure; this is how MongoDB gets high availability. **Sharding** partitions a collection across shards by a shard key for horizontal scale — a poorly chosen shard key creates a hot shard.
- **Consistency**: writes go to the primary; `writeConcern` controls how many nodes must acknowledge, and `readPreference` controls whether reads may go to secondaries (and therefore be stale). With defaults, MongoDB is **CP** in CAP terms.
- **Transactions** across documents exist since 4.0, but single-document operations have always been atomic — and the schema design advice is to embed so that a single-document update is enough.
- In ClassRoom Code, MongoDB lab questions run through the **real `mongosh`**, not an emulation, so aggregation pipelines, `$lookup`, `$unwind`, indexes, and validators behave exactly as they do in the lab. Each run gets a uniquely named database that is dropped afterwards, so one student's `CREATE`/`insert` can never affect another's.

## A5. "How did you decide which database to use?"
A strong, concrete answer from my own work:
- ClassRoom Code's **core data is highly relational** — users, courses, enrolments, worksheets, questions, test cases, submissions, feedback — with strict integrity requirements (a submission must belong to a real question and a real student) and multi-table transactions. That is a textbook case for a relational database, so PostgreSQL.
- I used **JSONB columns** inside PostgreSQL for the genuinely schemaless parts — starter code per language, and the stored run result — which is the pragmatic middle ground rather than adding a second database.
- MongoDB appears as a **lab engine**, not as the application store, because the department teaches it and students must run real MongoDB.
- In development the default driver is **PGlite** — PostgreSQL 16 compiled to WebAssembly, running embedded with no install and no Docker — and setting `DATABASE_URL` switches to a real PostgreSQL server. Same SQL, same migrations. That is a deployment-friction decision worth mentioning.

## A6. Migrations
Schema changes are versioned SQL files applied in order and recorded in a migrations table, so any environment can be brought to the same schema deterministically. My project has `001_init.sql`, `002_academic_structure.sql`, and `003_worksheet_imports.sql`. Seeding is **idempotent** — users match on email, courses on name plus code, worksheets on course plus title — so re-running updates in place rather than duplicating. Content files are **validated strictly before anything touches the database**, and every problem is reported at once with the exact field path that caused it.

---

# PART B — Core Domains

## B1. Artificial Intelligence

**Definition.** AI is the field of building systems that perform tasks normally requiring human intelligence — perception, reasoning, planning, language, and decision making. Machine learning is a subset of AI; deep learning is a subset of machine learning.

**Types by capability**: Narrow/Weak AI (does one task well — everything that exists today), General AI (human-level across domains — hypothetical), Super AI (beyond human — hypothetical).
**Types by function**: reactive machines, limited memory, theory of mind, self-aware.

**Classical AI topics likely to be asked**:
- **Search**: uninformed (BFS, DFS, uniform cost, iterative deepening) versus informed/heuristic (Greedy best-first, **A\*** which uses f(n) = g(n) + h(n) and is optimal when the heuristic is admissible — never overestimates — and consistent).
- **Adversarial search**: **Minimax** for two-player zero-sum games, with **alpha-beta pruning** cutting branches that cannot affect the result, reducing the effective branching factor.
- **Knowledge representation**: propositional and first-order logic, semantic networks, frames, ontologies. Inference by forward chaining (data-driven) and backward chaining (goal-driven).
- **Constraint satisfaction problems**: variables, domains, constraints; solved with backtracking plus forward checking and arc consistency (AC-3). Map colouring and Sudoku are the standard examples.
- **Expert systems**: a knowledge base of rules plus an inference engine.
- **Turing test** as the classic (and much-criticised) behavioural definition of machine intelligence.

**Modern AI / LLM topics** — I should be fluent here because ModelAuth is an LLM project:
- A **large language model** is a transformer trained to predict the next token over vast text. It is autoregressive: each token is sampled from a probability distribution conditioned on everything before it.
- **Transformer** architecture: self-attention lets every token attend to every other token, so context is captured without recurrence, and it parallelises across the sequence — which is why it scaled where RNNs did not.
- **Tokens** are sub-word units; models are priced and limited by token count. **Context window** is how many tokens the model can attend to at once.
- **Temperature** scales the logits before sampling: low temperature concentrates probability on the most likely token (near-deterministic), high temperature flattens the distribution (more varied). **Top-k** and **top-p (nucleus)** sampling truncate the distribution instead.
- **Hallucination** — fluent, confident output that is factually wrong, because the model optimises for plausible continuations, not truth.
- **Prompt engineering**, **few-shot prompting**, **chain of thought**, **RAG** (retrieval-augmented generation: fetch relevant documents and put them in the context so answers are grounded in real sources), **fine-tuning** versus **prompting**.
- **Quantisation** — reducing weight precision (8-bit, 4-bit) to fit models on smaller hardware, trading some quality for memory and speed.

**Why AI interests me — the honest, specific answer**: ModelAuth started from noticing an asymmetry. When you call a commercial LLM API, you get back text and nothing else — no weights, no log-probabilities, no attestation of which model produced it. TLS and API keys authenticate the *host*, not the *weights*. So a provider has both the ability and a financial incentive to silently route your traffic to a cheaper model, and you would have no way to tell. That is an AI problem and a security problem at the same time, which is exactly the intersection I want to work in.

## B2. Machine Learning

**Definition.** Systems that improve at a task from data rather than from explicitly written rules. Tom Mitchell's framing: a program learns from experience E with respect to task T and performance measure P if its performance at T, measured by P, improves with E.

**Three paradigms**:
- **Supervised** — labelled data. *Classification* (discrete output: spam or not, which digit) and *regression* (continuous output: price, temperature). Algorithms: linear and logistic regression, k-nearest neighbours, decision trees, random forests, gradient boosting (XGBoost), support vector machines, naive Bayes, neural networks.
- **Unsupervised** — no labels; find structure. Clustering (k-means, hierarchical, DBSCAN), dimensionality reduction (PCA, t-SNE), association rules, and **anomaly detection** — which is the family my ModelAuth work belongs to.
- **Reinforcement** — an agent takes actions in an environment and learns from rewards. Q-learning, policy gradients; used in games, robotics, and RLHF for language models.

**The core concepts that get asked**:
- **Overfitting** — the model memorises training data including its noise, so training error is low and test error is high. Signs: a large train/test gap. Fixes: more data, simpler model, regularisation (L1/Lasso which drives coefficients to zero and so selects features, L2/Ridge which shrinks them), dropout, early stopping, cross-validation.
- **Underfitting** — the model is too simple to capture the pattern; both training and test error are high. Fix: a more expressive model or better features.
- **Bias-variance tradeoff** — bias is error from wrong assumptions (underfitting), variance is error from sensitivity to the particular training sample (overfitting). Total error = bias² + variance + irreducible noise. Increasing model complexity lowers bias and raises variance.
- **Train / validation / test split** — train fits parameters, validation tunes hyperparameters and selects models, test is touched once at the end for an unbiased estimate. **k-fold cross-validation** rotates the validation fold to use all the data.
- **Feature engineering** — normalisation and standardisation, one-hot encoding for categoricals, handling missing values, and creating derived features. Usually worth more than model choice.
- **Curse of dimensionality** — as dimensions grow, data becomes sparse and distance metrics lose meaning, so you need exponentially more data.

**Evaluation metrics — know these cold, because they connect directly to my project**:
- **Confusion matrix**: true positives, false positives, true negatives, false negatives.
- **Accuracy** = (TP+TN)/total — misleading on imbalanced data. If 99% of traffic is benign, a detector that always says "benign" is 99% accurate and useless.
- **Precision** = TP/(TP+FP) — of the things I flagged, how many were real? Optimise this when false alarms are costly.
- **Recall / sensitivity / true positive rate** = TP/(TP+FN) — of the real things, how many did I catch? Optimise this when misses are costly.
- **F1** = harmonic mean of precision and recall.
- **ROC curve** plots TPR against FPR across thresholds; **AUC** summarises it.
- **Precision-recall curve** is more informative than ROC on heavily imbalanced data.
- For regression: MSE, RMSE, MAE, R².

**Statistics I actually used in ModelAuth** — be ready to explain these, because they are on my resume by implication:
- **Kolmogorov-Smirnov two-sample test** — a non-parametric test of whether two samples come from the same distribution. The statistic D is the maximum vertical distance between the two empirical cumulative distribution functions: D = sup|F₁(x) − F₂(x)|. Non-parametric means it assumes no particular distribution shape, which matters because I have no model of what an LLM's number distribution should look like. I flag when p < 0.01.
- **CUSUM (cumulative sum) change-point detection** — a sequential test that accumulates standardised deviations from a baseline: z = (x − μ̂)/σ̂, then S = max(0, S_prev + z − k), and flags when S exceeds a threshold h. The allowance k prevents drift from noise; h sets the sensitivity/false-alarm trade-off. Its strength is detecting **small, persistent** shifts that a windowed test misses, because evidence accumulates instead of being forgotten.
- **DAS-CUSUM (dispersion-aware)** — uses the statistic 0.5(z² − 1), which has expectation zero under the null and grows for either a mean shift *or* a variance shift, so it catches changes a mean-only CUSUM misses.
- **Type I versus Type II error** — a false alarm (flagging a swap that did not happen) versus a miss. My benchmark reports **detection power** (1 − Type II rate) and **false alarm rate** (Type I) separately, because a detector that flags constantly has perfect power and is worthless.
- **p-value** — the probability of observing data at least this extreme if the null hypothesis is true. Not the probability the hypothesis is true.
- **Detection delay** — how many probes after the true change point the alarm fires. This is the third axis alongside power and false alarms, and it is what makes this a *sequential* problem rather than a classification problem.

## B3. Information Security

Covered in depth in **Cybersecurity_Domain.md** — that is the domain I am choosing when asked, so read that file properly. The one-paragraph version for this file:

Information security is the practice of protecting information and systems against unauthorised access, modification, destruction, and disruption. Its foundation is the **CIA triad** — Confidentiality (only authorised parties can read it), Integrity (it has not been altered), Availability (it is there when needed) — extended by authentication, authorisation, non-repudiation, and accountability. It spans network security, application security, cryptography, identity and access management, incident response, and digital forensics.

My two security-adjacent projects sit at opposite ends of it: **NetSpecter** is defensive network monitoring — passively capturing traffic to prove that plaintext protocols leak credentials, tokens, and session cookies. **ModelAuth** is a supply-chain integrity problem — verifying that the AI model you are paying for is the one actually serving your requests, when the provider gives you nothing to verify with. Both come from the same instinct: assume the channel is untrustworthy and look for evidence rather than taking a claim at face value.

---

# Likely Cross-Domain Questions

**1. "You list AI, ML, and Information Security. Which is your strongest and why?"**
Information Security, and I would rather be honest about that than claim all three equally. It is the domain I have gone deepest in — NetSpecter is a working credential-leak auditor with TCP stream reassembly and multi-protocol detectors, and my hobby interest in forensic science is the same instinct applied outside computing. My AI and ML work is real but narrower: ModelAuth is a statistics-and-detection project applied to LLMs rather than model building, so my strength there is in evaluation and change-point detection, not in training architectures.

**2. "How do AI and security intersect?"**
Three ways. **Security for AI** — protecting models and pipelines: prompt injection, training-data poisoning, model extraction, adversarial examples, and the supply-chain question ModelAuth attacks, which is whether the model serving you is the model you contracted for. **AI for security** — using machine learning for intrusion detection, spam and malware classification, and user-behaviour anomaly detection, with the caveat that base rates are brutal, so precision matters far more than accuracy. **AI as an attack tool** — better phishing, automated vulnerability discovery, and deepfakes.

**3. "Have you deployed a machine learning model?"**
No, and I would say so directly. ModelAuth is statistical detection rather than trained models — the detectors are classical sequential-analysis algorithms with hyperparameters (window size, warmup, allowance k, threshold h) that I tuned empirically across 3 tiers × 30 runs, not learned weights. What I do have is the full evaluation discipline: ground-truth labelling, held-out references, ROC analysis, and reporting power, delay, and false alarms separately rather than a single accuracy number.
