# Miscellaneous 2 — The Four-Tier Question Bank

Four tiers, by how the question *feels* when it lands:

1. **Obvious** — you know it is coming. There is no excuse for fumbling these.
2. **Linking** — two topics joined together. Most candidates prepare topics in isolation and fall apart here. This is where you can be visibly better than average.
3. **Oh no, I forgot about that** — things that are true about *you* and your own work that you will blank on because you have not thought about them in months.
4. **What on earth is this** — curveballs, puzzles, pressure tests, and ethics. Nobody can prepare the exact question. You prepare the *method*.

Answers here are compressed. The full versions live in the other files; pointers are given.

---

# SECTION 1 — Obvious Questions

These are near-certain. Answer each in under 60 seconds, cleanly, then stop.

## 1.1 About you
| Question | Skeleton | Full answer |
|---|---|---|
| Tell me about yourself | Present → what you built → what you want. 90 seconds. | HR file, Part 1 |
| Why cybersecurity? | Anchor in NetSpecter and ModelAuth: "how do you verify something when the other side has no obligation to be honest?" | Cybersecurity file, Part 0 |
| Strengths / weaknesses | Two strengths with evidence; one honest weakness (solo work, no team experience) with what you are doing about it | HR file, 2.2–2.3 |
| Why should we take you? | You build things nobody asked for, and you finish them. Three of four projects were self-directed. | HR file, 2.5 |
| Your CGPA is 6.84 | Acknowledge, do not excuse, redirect to inspectable work, say you are aware you need to raise it | HR file, 2.11 |
| Where in five years? | Security engineering, between building and breaking | HR file, 2.4 |
| Questions for us? | Always have three | HR file, 2.10 |

## 1.2 About your projects
Every one of these will be asked about your flagship at minimum:
- Walk me through your best project. *(One-minute pitch — the opening of each project file.)*
- What problem does it solve? Why did existing tools not work?
- What was the hardest part?
- What would you do differently?
- How would you scale it?
- What is the architecture? *(Be able to draw it.)*
- Which part are you least happy with?

## 1.3 Core subject certainties
**Networks** — OSI vs TCP/IP · TCP vs UDP · three-way handshake · what happens when you type a URL · DNS resolution · HTTP vs HTTPS · subnetting · switch vs router.
**OS** — process vs thread · scheduling algorithms and their trade-offs · deadlock and the four conditions · virtual memory and paging · page replacement · semaphore vs mutex · context switching.
**DBMS** — normalization to 3NF · ACID · joins · indexing and when an index is not used · DELETE vs TRUNCATE vs DROP · WHERE vs HAVING · transactions and isolation levels.
**OOP** — four pillars with real examples · abstract class vs interface · overloading vs overriding · SOLID · composition over inheritance.
**DSA** — Big-O of common operations · array vs linked list · why a hash map is O(1) · sorting comparison · recursion vs iteration.

## 1.4 Language certainties
- Which language are you strongest in and why? *(Python — packet parsing, statistics, and system control across three different projects.)*
- Difference between Python lists and tuples.
- What is the GIL and when does it matter?
- Java: why not 100% object-oriented? `final` vs `finally` vs `finalize`?
- C: stack vs heap, what is a segmentation fault, what is a dangling pointer.
- JavaScript: `var` vs `let` vs `const`, what is a closure, explain the event loop.
- Git: merge vs rebase, `reset` vs `revert`, what is the staging area for.

---

# SECTION 2 — Linking Questions

This is the section that wins interviews. The pattern is always: *"You mentioned X. How does that relate to Y?"* Prepare the bridges, not just the islands.

## 2.1 The same idea appearing in four subjects

**Deadlock — OS and DBMS.**
Same four Coffman conditions, different handling. An OS mostly *ignores* deadlock (the ostrich algorithm) because it is rare and prevention is expensive. A database *detects* it — it builds a wait-for graph, finds the cycle, picks the cheaper transaction as a victim, aborts it, and expects the application to retry. The difference is that a database has a natural rollback mechanism (the transaction log) and an OS does not: you cannot cleanly roll back a process that has already written to a file or a socket.

**Caching — everywhere.**
CPU cache (temporal and spatial locality) → TLB (caching page-table entries) → OS page cache → database buffer pool → DNS resolver cache → browser cache → CDN edge cache → my Judge0 client caching language ids for an hour. Every one is the same bet: recently or nearby-used things will be used again. Every one faces the same three problems — what to evict (LRU, LFU, FIFO, clock), when to invalidate, and what to do on a miss. Phil Karlton's line is worth knowing: the two hard problems in computer science are cache invalidation and naming things.

**Hashing — DSA, DBMS, security, Git.**
A hash table maps a key to a bucket for O(1) lookup, and collisions are handled by chaining or probing. A database hash index does the same thing on disk, which is why it serves equality but not range queries. Password hashing deliberately inverts the goal: you want it *slow* (bcrypt, Argon2) because speed helps the attacker, and you add a per-user salt so identical passwords produce different digests. Git names every object by the hash of its content, which is what makes history tamper-evident — change an old commit and every descendant hash changes. Same primitive, four completely different requirements.

**Trees — DSA, DBMS, OS, web, Git.**
BST for ordered lookup → B+ tree for database indexes (high branching factor means few levels, so few disk reads, and linked leaves make range scans cheap) → the file system directory tree → the DOM tree in a browser → Git's tree objects representing directories → a heap backing a priority queue. The B+ tree choice is the interesting one: it exists specifically because the bottleneck is disk seeks, not comparisons.

**Indexing and paging — DBMS and OS.**
A database index and an OS page table are both indirection layers that trade memory and write cost for read speed. A page table maps a virtual page to a physical frame; a B+ tree index maps a key to a row location. Both are multi-level because a flat structure would be too large. Both have a fast cache in front (TLB for one, buffer pool for the other). And both make writes more expensive — every index slows down INSERT, just as every page-table update costs a TLB flush.

**Concurrency — OS and DBMS.**
An OS race condition and a database lost update are the same bug at different altitudes. `counter++` interleaving badly is exactly two transactions reading a balance, both computing a new one, and the second overwriting the first. Mutual exclusion is the OS answer; two-phase locking is the database answer. Serializability in a database is the formal version of "the result must be as if they ran one at a time," which is what mutual exclusion buys you informally.

**Layering and abstraction — OSI, OS, and OOP.**
The OSI stack, the OS system-call boundary, and an abstract base class are the same idea: define a contract, hide the implementation, and let either side change independently. My ModeOS `AudioBackend` interface is a two-method OSI layer — the caller says `set_volume(30)` and never learns whether WirePlumber, PulseAudio, or ALSA did it, exactly as the transport layer never learns whether the data crossed fibre or Wi-Fi.

**Reliability — TCP and databases.**
TCP guarantees delivery with sequence numbers, acknowledgements, and retransmission. A database guarantees durability with write-ahead logging — write the log record and flush it *before* the data pages, so a crash can be recovered by replaying. Both solve "the medium is unreliable" by writing down what you intended before you rely on it having happened. A filesystem journal is the same trick a third time.

**Virtualisation — OS, containers, and my projects.**
Virtual memory gives each process the illusion of a private contiguous address space. Containers give each process group the illusion of a private machine, using namespaces (isolating the process, filesystem, network, and user views) and cgroups (capping CPU, memory, and process count). Judge0's `isolate` sandbox is built on exactly those two kernel features — so "how does Judge0 keep student code from wrecking the server?" is an operating systems question, not a web question.

## 2.2 Linking your projects to subjects
Have one sentence ready for every cell you might be asked about.

| | Networks | OS | DBMS | OOP | Security |
|---|---|---|---|---|---|
| **ClassRoom Code** | REST over HTTP, OAuth redirects, cookies with `SameSite`, CORS and the Vite proxy | Untrusted code in isolated processes, wall-clock timeouts, process-group kill, Judge0 = cgroups + namespaces | The whole schema: UUIDs, composite keys on junction tables, CASCADE vs SET NULL, functional index on `lower(email)`, JSONB, four engines behind one interface | Routes/services/db layering; DB engines and executors interchangeable behind one interface | 404-not-403, assignment-not-role authorisation, trust boundary around the AI importer, refusing to boot insecure |
| **ModelAuth** | HTTP REST client to Ollama on port 11434, retries, concurrent requests | `ThreadPoolExecutor` — threads because the work is I/O bound and the GIL releases during waits | JSONL instead of a database, and why that was right | Four detectors with one uniform contract, so one benchmark function fits all | Supply-chain integrity: OWASP A08. Verifying the artefact you were sold |
| **NetSpecter** | The entire file: layers, raw sockets, BPF, ports, TCP reassembly, what TLS hides | Root and raw sockets, bounded buffers, flow eviction on timeout | — | Detector engine dispatching to a family of protocol detector classes | Passive monitoring, plaintext credential exposure, base64 is not encryption |
| **ModeOS** | — | The whole file: `/proc`, signals, nice values, sysfs, XDG, containers | State persisted as JSON | Abstract base classes, four backend families, Strategy pattern, dependency inversion | Rootless by design, protection whitelist, guaranteed dry run |

## 2.3 Linking questions they may actually ask
- "You used PostgreSQL and MongoDB. When would you pick each, and did you consider using only one?"
- "Your ClassRoom Code runs untrusted code. Explain the OS mechanisms that make that safe."
- "NetSpecter buffers TCP flows. What OS problem is that, and how did you bound it?"
- "ModelAuth uses threads. Why not processes? Why not async?"
- "ModeOS changes nice values. Explain what the scheduler actually does with that."
- "You said your API returns 404 instead of 403. Is that not lying to the client? What does the HTTP spec say?"
- "You store sessions in a JWT. What is the downside, and how did you work around it?"
- "Your grader ignores row order. Justify that using relational theory."
- "You bundled Monaco instead of using the CDN. What is the trade-off you accepted?"
- "Both NetSpecter and ModelAuth are detectors. Compare them as detection systems."
  *(Good answer: NetSpecter is signature-based — fast, precise, zero false positives on a known pattern, blind to anything novel. ModelAuth is anomaly-based — it models normal and flags deviation, so it catches things nobody wrote a rule for, at the cost of false alarms and a detection delay. I have built one of each, which is the classic IDS trade-off.)*

---

# SECTION 3 — "Oh No, I Forgot About That"

These are things that are **true about you** and that you will blank on. Read this section twice.

## 3.1 Resume items you barely think about
- **Chart.js.** It is on your resume. You used it in exactly one place: `interactive_dashboard.py` generates a standalone HTML dashboard with four charts (power by tier, delay by tier, ROC, cold-start curve). Know canvas vs SVG. Do not overclaim.
- **Java.** It is on your resume, but **you did not write Java in any project** — ClassRoom Code *supports* Java as a student language via Judge0. Say that honestly if pushed: "I know the language and I have used it in coursework; in my projects it appears as a target language rather than an implementation language."
- **Bash.** On your resume, easy to forget. Know redirection, pipes, `$?`, quoting variables, `chmod` numbers, and two or three real one-liners.
- **macOS and Windows.** Listed as operating environments. Know BSD vs GNU `sed -i`, APFS case-insensitivity, CRLF vs LF, and WSL.
- **Oracle.** Listed under databases. Know PL/SQL blocks, `DUAL`, sequences, and above all **DDL commits implicitly** — that is your best Oracle war story.
- **MySQL.** Listed, but you used PostgreSQL. Know InnoDB vs MyISAM and that MySQL defaults to REPEATABLE READ.
- **"Artificial Intelligence, Machine Learning."** Listed as core domains. Be ready for "have you trained a model?" — the honest answer is no, ModelAuth is classical sequential statistics, not learned weights. Say so; it is a stronger answer than bluffing.

## 3.2 Facts about yourself
- What "Integrated M.Sc. Software Systems" actually means — a five-year programme combining bachelor's and master's, no separate entrance after three years.
- Your department: **Applied Mathematics and Computational Sciences**. Be ready for "why is a software programme under applied maths?"
- Your batch years: **2024–2029**. You are in **year 3, semester 5**.
- Your 12th percentage (96.30%) and 10th (89.80%) — and the obvious follow-up: "your school marks were excellent and your CGPA is 6.84. What changed?" Have a real answer, not a deflection.
- **AXIOS 2025** — what it is (your college symposium), what you did (photography for every event), how long it ran.
- Which board you studied under (Andhra Pradesh State Board) and that you moved from Kadapa/Guntur to Coimbatore.

## 3.3 Your own code, six months later
You wrote these and you will not remember them under pressure. Skim before you go in:
- The **exact numbers**: 191 tests. 53 + 29 verification checks. 400 probes with the switch at 200. 92.86% vs 14.29% on the medium tier. 64 KB flow buffer, 1000 flows, 45-second timeout. 2-second SIGTERM grace period. ~66 KB main bundle.
- **Which branch holds what.** NetSpecter: `main` is v1, `cli` is v2, `frontend-backend` is v2 plus a web dashboard. ModelAuth: `main` is all three tiers, `easy` and `medium+hard` are the separate experiment tiers.
- **Why PGlite exists in ClassRoom Code** — PostgreSQL compiled to WebAssembly so development needs no Postgres install and no Docker; `DATABASE_URL` switches to a real server.
- **The three things your database grader ignores** and why: column name case (Oracle upper-cases, Postgres lower-cases), column order (MongoDB `$project` does not preserve it), row order (no `ORDER BY` means no defined order).
- **What `k` and `h` are in your CUSUM** — allowance and decision threshold. You will be asked and it is embarrassing to blank.
- **Why temperature is 1.0 in ModelAuth** — the method needs a visible output *distribution*; at temperature 0 there is nothing to compare.
- **Why ModeOS does not need root** — raising a nice value needs no privilege; only lowering below zero does.

## 3.4 Basics that vanish under pressure
- **Write a SQL query on paper.** Second-highest salary. Duplicates with `GROUP BY ... HAVING`. A join with a `WHERE`. Practise once by hand.
- **Reverse a string / linked list / array** without an IDE.
- **The logical order of SQL evaluation** — FROM, JOIN, WHERE, GROUP BY, HAVING, SELECT, DISTINCT, ORDER BY, LIMIT. This is why an alias works in ORDER BY but not in WHERE.
- **Big-O of things you use daily** — dict lookup O(1), list `insert(0, x)` O(n), `in` on a list O(n) but on a set O(1), sorting O(n log n).
- **Git under pressure** — how do you undo the last commit but keep the changes? (`git reset --soft HEAD~1`.) How do you undo a pushed commit? (`git revert`, never `reset`.)
- **`chmod 755`** — owner rwx, group r-x, others r-x. Read 4, write 2, execute 1.
- **Port numbers** — 22, 80, 443, 3306, 5432, 27017, 1521.
- **What HTTP status code** for "authenticated but not allowed"? 403. "Not authenticated"? 401.

## 3.5 Things about your projects you may not have framed as achievements
You did these; you may not think to mention them.
- You wrote **273 automated checks** across ClassRoom Code and never said so out loud.
- You ran roughly **60,000 LLM probes** to build a labelled dataset.
- You designed an experiment (cold-start contamination) **specifically to find where your own method fails** — that is unusual and worth stating.
- You **found and fixed a bug in your own metric** (negative detection delays) and then built a sanity-check module to catch that class of error.
- Every repository has a **README that documents limitations**, not just features.
- You made a product decision to **fail loudly rather than grade wrongly**, twice, in two different subsystems.

---

# SECTION 4 — "What On Earth Is This"

You cannot prepare the exact question. You prepare the **method**: think out loud, state assumptions, decompose, and never freeze.

## 4.1 The universal method
1. **Say the question back.** Buys five seconds and confirms you understood.
2. **Ask one clarifying question.** Almost every curveball is deliberately underspecified.
3. **State your assumptions out loud.** "I will assume a mid-size city and that everyone owns one phone."
4. **Decompose.** Break it into parts you can each estimate or reason about.
5. **Commit to an answer.** A wrong number with clear reasoning beats no number.
6. **Sanity check it.** "That gives 40,000, which feels high — let me check the population assumption."

They are watching the process. The number is nearly irrelevant.

## 4.2 Estimation / Fermi questions
*"How many piano tuners are in Coimbatore?" "How much data does WhatsApp process daily?" "How many golf balls fit in a bus?"*

**Method**: anchor on a population, apply a rate, apply a frequency, divide by a capacity. Say every number as you use it.
> "Coimbatore is roughly 2 million people, say 400,000 households. Maybe 1 in 200 has a piano — 2,000 pianos. Each is tuned once a year. A tuner does 3 a day, 250 days a year, so 750 a year. 2,000 ÷ 750 ≈ 3 tuners. Given institutions and schools have more, I would say 3 to 10."

## 4.3 System design (they will scale it down for a student)
*"Design a URL shortener." "Design a rate limiter." "How would you build Instagram's feed?"*

**Method**: clarify requirements and scale → define the API → design the data model → sketch the components → identify the bottleneck → discuss the trade-off.
> URL shortener: `POST /shorten` returns a short code; `GET /:code` redirects with 301 or 302. Generate the code by base62-encoding an auto-increment id, or hash the URL and take 7 characters, handling collisions. Store `code → url` in a key-value store; the read-to-write ratio is enormous, so cache aggressively and use 302 if you want click analytics, 301 if you want browsers to stop asking. The bottleneck is read throughput, so cache and read replicas. Scaling writes means distributed id generation, which is where you mention Snowflake ids or pre-allocated ranges.

Everything you need is already in your own projects: you have built authentication, authorisation, a schema, an execution queue, and a caching layer.

## 4.4 Brainteasers and puzzles
*Bridge crossing, weighing balls, egg drop, two ropes burning, 100 prisoners.*

Nobody expects instant recall. They expect structure: state the constraint, try a small case, look for the invariant, generalise. If you know the puzzle, **say so** — pretending to derive a memorised answer is transparent and looks worse than honesty.

**Two ropes:** each rope burns in 60 minutes but not uniformly. Measure 45 minutes: light rope A at both ends and rope B at one end. A is gone at 30. At that moment light B's other end; B's remaining 30 minutes of rope burns from both ends and finishes in 15. Total 45.

## 4.5 Deep-trivia "why" questions
- **Why does `0.1 + 0.2 != 0.3`?** IEEE 754 binary floating point cannot represent 0.1 or 0.2 exactly, in the same way decimal cannot represent 1/3. The tiny errors do not cancel. Never compare floats with `==`; compare within an epsilon, and use a decimal type for money.
- **Why is `NULL` not equal to `NULL` in SQL?** NULL means *unknown*. Two unknowns cannot be proven equal, so the result is NULL, not true. Hence `IS NULL`.
- **Why is a Python `int` unbounded when a C `int` is 32 bits?** Python integers are heap objects with arbitrary precision; C ints are fixed-width machine words. Trade-off: correctness versus speed and memory.
- **Why does `git` store snapshots rather than diffs?** Because a snapshot with content-addressed objects makes branching, merging, and history rewriting cheap — unchanged files simply point at the same existing blob. Diffs are computed on demand.
- **Why is deleting from the middle of an array O(n) if memory is random-access?** Random access finds the element in O(1); the cost is shifting the remaining n−k elements to close the gap.
- **What actually happens when you `rm` a file?** The directory entry is removed and the inode's link count is decremented. The data blocks are freed only when the count reaches zero *and* no process still holds the file open — which is why deleting a log file being written to does not free the disk space until the process closes it.
- **Why does `sudo rm -rf /` not just work anymore?** Modern `rm` refuses to act on `/` without `--no-preserve-root`. Do not demonstrate.

## 4.6 Pressure and stress questions
*"I do not think your project is impressive." "Your CGPA says you cannot handle pressure." "You would not survive here."*

These test composure, not correctness. **Do not get defensive and do not collapse and agree.** Acknowledge the point, give evidence, invite specifics.
> "That is fair to push on. What would make it more impressive to you — the scale, the novelty, or the engineering? ModelAuth was 60,000 probes across three difficulty tiers with four detectors and a benchmark that includes an experiment designed to find where my own method breaks. If that still is not the bar, I would genuinely like to know what is, so I can go and build toward it."

*"Teach me something in two minutes."* Have one ready. The best options for you: why a CUSUM detector beats a sliding window on subtle shifts; why base64 is not encryption; why `SIGTERM` before `SIGKILL` matters.

*"What question should I have asked you?"* A gift. Point at the thing you are proudest of that has not come up.

## 4.7 Ethics curveballs — likely, because you chose security
These matter more for you than for other candidates. Answer with a clear principle, not a hedge.

**"You find a vulnerability in a company's website that has not authorised you to test. What do you do?"**
> "First, I stop. Finding it by accident is not a crime; probing further to confirm it is unauthorised access under the IT Act, regardless of intent. I document only what I already saw, do not access any data, and look for a responsible disclosure or security.txt contact. If there is none, I contact them through an official channel, describe the class of issue without a working exploit, and give them time before saying anything publicly. What I do not do is test further to 'prove' it, and I do not post it."

**"Your friend asks you to check if their partner's password is weak."**
> "No. Consent from the account owner is the entire line, and it is not my friend's account to consent for."

**"You are told to ship something you believe is insecure because of a deadline."**
> "I would write down the specific risk, the realistic impact, and the cheapest mitigation, and give that to whoever owns the decision — because it is their call, not mine. Often there is a partial fix that costs a day. If it ships anyway, the decision is recorded and I would push to get it on the backlog with a date. What I would not do is quietly refuse, or quietly ship it and say nothing."

**"Would you use a leaked password database to test your own users?"**
> "Not by downloading it. The right way is a k-anonymity service like Have I Been Pwned's range API, where you send the first five characters of the hash and never the password or the full hash. You get the same protective outcome without handling stolen data."

**"Have you ever hacked anything?"**
> "Not anything I did not own. NetSpecter runs on my own machines against traffic I generate myself — the demo is a `curl` to a deliberately vulnerable test site from another terminal on the same laptop. The distinction I care about is authorisation, and it is the reason the README carries a warning and the tool masks credentials by default."

## 4.8 Absurd or personality questions
*"If you were a data structure, which would you be?" "Sell me this pen." "How would you explain recursion to a five-year-old?"*

They are testing whether you can be human and think on your feet. Be brief, be playful, do not agonise.
> **Recursion to a five-year-old:** "Stand in a line of people and you want to know your position. Ask the person in front what number they are. They do not know either, so they ask the person in front of them. Eventually someone at the very front says 'I am first.' Then each person adds one and tells the person behind. That is recursion: the base case is the person at the front, and everyone else waits for the answer behind them."

> **A data structure:** "A hash table — I want the direct route to the answer rather than scanning, and I am willing to pay some memory up front for it." Any answer with a reason is a good answer.

## 4.9 The two answers that always work
**When you genuinely do not know:**
> "I have not worked with that. Based on [related thing I do know], I would expect [reasoned guess] — is that close?"
This converts a dead end into a demonstration of reasoning. It is a strong answer. **Bluffing is the only genuinely bad one**, because the follow-up always exposes it.

**When you are stuck mid-problem:**
> "Let me say where I am. I know I want [goal], and I am stuck on [specific blocker]."
Interviewers give hints to people who show them where they are. They cannot help silence.

---

# Final Checklist

- [ ] Can I do all four one-minute project pitches without notes?
- [ ] Can I answer the CGPA question calmly, in three sentences?
- [ ] Can I name three linking answers (deadlock, caching, hashing) cold?
- [ ] Do I know my own numbers — 191, 92.86 vs 14.29, 60,000, 2 seconds?
- [ ] Do I know which NetSpecter and ModelAuth branch holds what?
- [ ] Have I got one ethics answer and one "teach me something" answer ready?
- [ ] Have I got three questions to ask them?
- [ ] Have I said one limitation of my own work out loud, unprompted?
