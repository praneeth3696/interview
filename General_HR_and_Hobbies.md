# General, HR, and Hobbies — Interview Preparation

These questions decide the *tone* of the interview. They are asked first, they set the interviewer's expectation, and most candidates waste them by reciting their resume. Prepare these as carefully as the technical files.

**The rule for all of them**: a claim plus a specific piece of evidence. Never a claim alone.

---

# PART 1 — "Tell Me About Yourself"

This is not a request for your biography. It is asking: *why are you sitting here and why should I keep listening?* Structure: present → what you have built → what you are aiming at. Aim for 60–90 seconds.

## The prepared answer

> "I am Praneeth Reddy, a third-year student in the five-year Integrated M.Sc. Software Systems programme at PSG College of Technology, in the Department of Applied Mathematics and Computational Sciences.
>
> Most of what I have learned has come from building things end to end rather than from coursework alone. My largest project is ClassRoom Code — a coding lab platform for a college department, built with React and Node, where teachers publish worksheets and students solve them in an in-browser editor with auto-grading across C, C++, Java, Python, and also SQL and MongoDB lab questions. It runs student code inside Judge0 for sandboxing, uses Google OAuth restricted to the college domain, and models the department's real structure of programmes, batches, and co-taught subjects.
>
> Alongside that I have built two security tools. NetSpecter is a passive network auditor in Python that captures live traffic and detects credentials being sent in plaintext over HTTP, FTP, and mail protocols, with TCP stream reassembly so credentials split across packets are not missed. ModelAuth detects when an LLM API provider silently swaps the model you are paying for, using statistical change-point detection on the model's output distribution — with no access to weights or log-probabilities.
>
> The thread through all of it is that I keep choosing security problems. I want an internship where I can work on that seriously and learn from people who do it professionally."

## Variations
- **If they ask for something shorter**: name, programme, one flagship project in one sentence, the security interest, what you want. Twenty seconds.
- **If they ask "something not on your resume"**: photography, and the AXIOS 2025 coverage — see Part 4.
- **If they interrupt with a technical question mid-answer**: good sign. Follow them.

---

# PART 2 — The Standard HR Questions

## 2.1 "Why do you want this internship?"
Answer with what you want to *learn*, not what you want to *get*.
> "Everything I have built, I have built alone and to my own standards, which means I have never had my design decisions challenged by someone more experienced. I want to work on a codebase that other people depend on, go through real code review, and see how decisions get made when there are constraints I do not control — deadlines, existing systems, other people's priorities. Specifically, I want to see how security is done as a practice rather than as a set of things I read about."

## 2.2 "What are your strengths?"
Pick two or three and evidence each.
- **I finish things and I document them.** All four of my projects have substantial READMEs that explain not just how to run them but *why* decisions were made, and ClassRoom Code has 191 tests plus 82 end-to-end verification checks. A project nobody else can run or evaluate is not finished.
- **I think about failure modes.** In ClassRoom Code, if the code executor is unreachable in production the API returns 503 rather than falling back to an unsandboxed runner, and an imported question whose reference solution does not run gets no test cases rather than an invented expected output. I would rather fail loudly than be quietly wrong.
- **I go deep when something interests me.** ModelAuth was not an assignment. It came from a question I could not answer, and it turned into 60,000 probes across three difficulty tiers with four detectors and a proper benchmark, including an experiment specifically designed to find where my own method breaks.

## 2.3 "What is your weakness?"
Say a real one, and say what you are doing about it. Never "I work too hard".
- **Option A (scope):** "I over-engineer solo projects. ClassRoom Code supports four database engines and two code executors, and while each had a reason, it took much longer than a version that supported one of each. I have got better at asking what the minimum useful version is first — but it is still the direction I drift in."
- **Option B (collaboration):** "Almost everything I have built has been alone, so I have not had much practice with the things that only come up in a team — reading someone else's design and adapting to it, code review both ways, splitting work so it merges cleanly. It is a real gap and it is one of the specific reasons I want an internship rather than another solo project."
- **Option C (breadth):** "I have gone deep on the things that interest me and left gaps elsewhere. I have never deployed to a cloud environment properly or used a SIEM, and my machine learning knowledge is evaluation and statistics rather than model building. I would rather name those than have them found."

Pick **one** — usually B, because it is honest, verifiable from the resume, and directly answered by the internship.

## 2.4 "Where do you see yourself in five years?"
> "In five years I will have finished this programme. I want to be working in security engineering — ideally somewhere between building systems and breaking them, because I do not think you can secure something you have never built. Concretely: I want to have worked on a production system where security was a real requirement, not a checklist, and to have gone deep in one area — right now that looks like application and network security, with an interest in the forensics side."

## 2.5 "Why should we select you?"
> "Because I build things nobody asked me to build, and I finish them. Three of my four projects were entirely self-directed, and each one exists because I ran into a question I could not answer by reading. I also do the parts most students skip — writing tests, writing documentation that explains the reasoning, and being explicit about what my own work does *not* do. NetSpecter's README lists its limitations, ModelAuth includes an experiment designed to find where my method fails, and ClassRoom Code's README says outright that its local code runner is not a sandbox. I would rather be accurate about my own work than impressive about it."

## 2.6 "Tell me about a challenge you faced and how you handled it."
Use the **STAR** structure: Situation, Task, Action, Result.
> **Situation** — In ClassRoom Code I added support for database lab questions, where a student writes SQL or a MongoDB script and it is graded automatically.
> **Task** — I needed to decide what "the same answer" means for a query result, which turns out to be far harder than for printed output.
> **Action** — I found three separate problems. Oracle upper-cases unquoted column names while PostgreSQL lower-cases them. MongoDB does not preserve field order from a `$project`, so a correct pipeline can return the columns in a different order than the teacher wrote. And any query without `ORDER BY` has no defined row order at all, so comparing row sequences would fail correct answers randomly. For each I had to decide whether it was part of the answer or an artefact of the engine. I made column name case and column order always ignored, and made row order a per-question setting — because for a question specifically about `ORDER BY`, the order *is* the answer.
> **Result** — Database questions grade reliably across four engines, and each student gets a freshly seeded database per run so one student's `DROP TABLE` cannot affect anyone else. What I took from it is that automated grading is mostly a problem of deciding what you are actually testing, not of writing a comparison function.

## 2.7 "Tell me about a failure or a mistake."
> "In ModelAuth I initially computed detection delay as the first alarm minus the true switch point, without requiring the alarm to come *after* the switch. Pre-switch false alarms were producing negative delays, which were being averaged in and making my detectors look faster than they were. I only caught it when a mean delay came out negative, which is impossible. The fix was one condition — only count the first flag at or after the switch point — but I had to regenerate every result and rewrite parts of the report. The real lesson was that I had no sanity check that would have caught it automatically, so I added a `sanity_checks.py` module that audits data completeness and separability before any results are computed."

## 2.8 "How do you handle pressure / deadlines?"
> "I break the thing down until each piece is small enough that I know whether it is done, and I do the risky part first. ClassRoom Code was built in phases with an explicit record of what happened in each and why — schema first, then auth, then the teacher API, then execution, then the frontends. That ordering was deliberate: code execution was the part most likely to go wrong, so I wanted to hit it before I had built a UI around assumptions that turned out to be false."

## 2.9 "Are you a team player? / How do you handle disagreement?"
Be honest that most of your work is solo, and give what evidence you have — the AXIOS 2025 event coverage is genuine team work under deadline.
> "Most of my building has been solo, so I will not overclaim. Where I have worked in a team is event coverage — I shot photography for every event of AXIOS 2025, which meant coordinating with organisers, working to other people's schedules, and delivering on someone else's deadline rather than my own. On disagreement: I try to convert it into a question that has an answer. In technical arguments there is usually a way to test the claim, and if there is, arguing is a waste of time."

## 2.10 "Do you have any questions for us?"
**Always have three.** Saying no signals you do not care. Good ones:
- "What does the security work here actually look like day to day — is it review and hardening of existing systems, or building new tooling?"
- "What separates an intern who does well here from one who struggles?"
- "What would you want me to have learned by the end of the internship?"
- "What is the code review culture like — how much of what I write would be reviewed?"
- "Is there something you would recommend I learn or read before starting?"

## 2.11 Awkward questions, answered honestly

**"Your CGPA is 6.84. Why?"**
Do not make excuses and do not be defensive. Acknowledge, explain briefly, redirect to evidence.
> "It is lower than it should be, and the honest reason is that I put most of my time into building things rather than into exam preparation. I am not going to pretend that was an optimal trade — it cost me marks. What I would point to is that the work I did instead is real and inspectable: four projects, all on GitHub, with documentation and tests. If the concern is whether I can learn hard material and finish things, I would rather be judged on ClassRoom Code and ModelAuth than on a semester average. And I am aware I need to bring the number up."

**"You have not done an internship before."**
> "No, this would be my first, which is exactly why I want it. What I do have is the experience of building complete systems rather than assignments — deciding a schema, handling authentication properly, thinking about what happens when a dependency is unreachable, and writing tests I would trust. What I have never had is my decisions reviewed by someone more experienced, and that is the gap I want to close."

**"Which of your projects is the weakest?"**
> "ModeOS, in ambition rather than execution. It is well engineered — pluggable backends, a guaranteed dry run, exact state restoration — but it solves a personal convenience problem rather than a hard one. It is the project I would defend on craft and not on significance."

**"Did you build these alone or with a team? Did you use AI?"**
Be straightforward. AI-assisted development is normal now, and pretending otherwise is the risky answer.
> "I built them, and I used AI tooling for parts of them the way I would use documentation or Stack Overflow. What I can tell you is that I understand every design decision in them and why the alternative was rejected — which is what actually matters. Ask me about any part of the code and I will explain what it does and why it is that way."
Then be ready to actually do that. This is why the project files exist.

---

# PART 3 — Behavioural Questions Worth Preparing

Use STAR for all of these and reuse the same three or four stories rather than inventing new ones.

**Your stories:**
1. **Database grading across four engines** (ClassRoom Code) — problem solving, attention to correctness, understanding your domain. Use for: hardest technical challenge, attention to detail, ambiguity.
2. **The negative-delay bug** (ModelAuth) — catching your own mistake, rigour, building a safeguard afterwards. Use for: failure, learning from a mistake, quality.
3. **The Monaco CDN decision** (ClassRoom Code) — understanding the deployment environment, not accepting a default. Use for: a decision you are proud of, thinking about users, engineering judgment.
4. **The two-stage SIGTERM/SIGKILL termination and protection whitelist** (ModeOS) — care, safety, thinking about consequences. Use for: attention to detail, thinking about edge cases.
5. **AXIOS 2025 photography** — teamwork, working to someone else's deadline, delivering under pressure. Use for: team, pressure, non-technical.

**Questions these cover**: describe a difficult technical problem; a time you failed; a decision you are proud of; a time you had to learn something quickly; a time you disagreed with someone; how you prioritise; a time you went beyond what was asked; a time you had incomplete information.

---

# PART 4 — Hobbies and Activities

Both of my listed activities are genuine and both connect to my technical interests. That is the thing to bring out — an interviewer remembers a candidate whose hobbies explain something about how they think.

## 4.1 Photography — AXIOS 2025

**The resume line**: covered photography for all of the events of AXIOS 2025.

**How to talk about it:**
> "I shot photography for every event of AXIOS 2025, our college symposium. It was not one event — it was the whole run, which meant being in the right place across a full schedule I did not control, working in whatever light the venue happened to have, and turning images around fast enough to be useful to the organisers rather than a week later. The part I found hardest was that you get one attempt at each moment. A build can be re-run; a prize being handed over happens once."

**If they push further — what photography actually teaches:**
- **Composition is subtraction.** A good frame is mostly about what you leave out. That is the same instinct as writing a function that does one thing.
- **Working within constraints.** You cannot change the light, the venue, or the schedule; you change your position, your settings, and your timing. Most engineering is the same shape.
- **The technical side is real**: the exposure triangle — aperture (also depth of field), shutter speed (also motion blur), and ISO (also noise) — and every setting trades against another. It is a genuine multi-objective optimisation problem you solve in a second.
- **Post-processing is a pipeline** — a repeatable set of transformations applied consistently across hundreds of images, which is where photography and scripting meet. Batch renaming, culling, and consistent processing are exactly the kind of task I would automate.

**"Do you do anything technical with it?"** — Have an honest answer. If you have written scripts for renaming, culling, EXIF extraction, or batch conversion, say so; that is a genuine link. If not, say it is deliberately the part of your life that is not code, which is also a good answer — people who do only one thing burn out.

## 4.2 Forensic Science and Cybersecurity

**The resume line**: investigating the synergy between classical forensics and digital investigative methodologies to detect security weaknesses and interpret complex system deviations.

That line is dense, so be ready to say it in plain language.

**The plain-language version:**
> "I am interested in how the reasoning used in classical forensic science transfers to digital investigation. Both are about reconstructing what happened from the traces left behind, when whoever did it was not trying to help you. The specific principle that connects them is **Locard's exchange principle** — every contact leaves a trace. In physical forensics that is fibres, prints, and residue. In digital forensics it is log entries, file timestamps, memory artefacts, network flows, and shell history. In both cases the skill is knowing where traces persist, in what order they decay, and how to establish that what you found is genuine."

**Concrete parallels to draw:**
| Classical forensics | Digital forensics |
|---|---|
| Locard's exchange principle | Every action leaves logs, timestamps, memory artefacts |
| Chain of custody | Documented handling, hashing the evidence image |
| Never contaminate the scene | Write blockers, work on a bit-for-bit copy, never the original |
| Preserve the most fragile evidence first | Order of volatility: registers, RAM, network state, then disk |
| Establishing a timeline | Correlating MACB timestamps, logs, and network captures |
| Anti-forensic countermeasures | Timestomping, log wiping, encryption, living off the land |
| Expert testimony must be defensible | Findings must be reproducible and the method explainable |

**Where this shows up in my work:**
> "NetSpecter v2 has an offline PCAP replay mode — you point it at a captured file and it reconstructs the flows and reports what leaked. That is a forensic tool, not a monitoring tool: post-incident analysis of evidence you already have. And ModelAuth is the same reasoning at a different level — you cannot inspect the model, so you infer what changed from the statistical traces its outputs leave behind."

**If they ask what you have read or done in forensics** — be honest about the level. "It is an interest rather than formal training. I have read about the discipline, I know the frameworks — the NIST incident response lifecycle, order of volatility, chain of custody — and I know the tool names, Autopsy and Sleuth Kit for disk, Volatility for memory. What I have actually done hands-on is the network side, through NetSpecter. Malware reverse engineering is what I most want to learn next, because it is the closest thing to the forensic reasoning I find interesting."

## 4.3 If asked about hobbies more generally
Do not invent things. If asked what you do outside work and study, photography is the real answer, and CTF-style security practice is a legitimate second one if you actually do it. It is fine to say your projects *are* your hobby — three of the four were entirely self-directed, which is itself the evidence.

---

# PART 5 — Delivery: How to Actually Perform

**Before:**
- Re-read the four project files and the domain file. Everything else is support.
- Have the repositories open in tabs, and be able to navigate to the file behind any claim you make.
- Know your own resume cold. Anything on it is fair game, including "Chart.js" and "Bash".

**During:**
- **If you do not know something, say so, then reason out loud toward it.** "I have not worked with that directly, but based on X I would expect Y — is that right?" That is a strong answer. Bluffing is the only truly bad one, because the follow-up question always exposes it.
- **Ask a clarifying question before answering a broad one.** "When you say secure the application, do you mean the design or the implementation?" It buys thinking time and shows you do not answer questions you have not understood.
- **Use concrete numbers.** 191 tests. 60,000 probes. 92.86% detection power versus 14.29%. A 66 KB main bundle. Numbers are memorable and they prove you actually did the thing.
- **Structure long answers out loud.** "Three things: first..., second..., third..." Interviewers can follow it and it stops you rambling.
- **When you finish an answer, stop.** Filling silence is how good answers get undone.
- **Bring it back to your projects whenever it is genuinely relevant** — but not when it is not, because forcing it is obvious.

**Tone:**
Be accurate rather than impressive. The single most credible thing you can do is state a limitation of your own work before you are asked — "NetSpecter v1 does not do TCP reassembly, so split-packet payloads are missed, which is what v2 fixes." Candidates who volunteer the boundaries of their own knowledge are trusted on everything else they say.

**At the end:**
Ask your three questions. Thank them. If a question stumped you, it is entirely reasonable to say "I did not have a good answer for the question about X — I would like to go and understand it properly." That is a strong closing note.
