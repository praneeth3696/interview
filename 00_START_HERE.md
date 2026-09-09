# Interview Preparation — Start Here

Everything in this folder is written to be read straight through. There is no filler. If you read all of it properly, you can hold a conversation at basic-to-medium depth on anything on your resume.

---

## The files

| # | File | What it covers | Priority |
|---|---|---|---|
| 1 | `Project_01_ClassRoom_Code.md` | Full-stack platform: React, Node, Express, PostgreSQL, Judge0, OAuth, AI importer | **Highest** |
| 2 | `Project_02_ModelAuth.md` | LLM substitution detection: KS test, CUSUM, DAS-CUSUM, Ollama, benchmark results | **Highest** |
| 3 | `Project_03_NetSpecter.md` | Passive credential-leak auditor: Scapy, TCP reassembly, multi-protocol detectors | **Highest** |
| 4 | `Project_04_ModeOS.md` | Linux mode manager: pluggable backends, process control, state restoration | **Highest** |
| 5 | `Cybersecurity_Domain.md` | The domain you are choosing — foundations, crypto, network, appsec, forensics, offensive | **Highest** |
| 6 | `General_HR_and_Hobbies.md` | Introduce yourself, strengths, weaknesses, CGPA question, photography, forensics interest | **Highest** |
| 7 | `Misc_02_Question_Bank.md` | Four tiers: obvious questions, linking questions across topics, blind spots in your own work, and curveballs/ethics | **Highest** |
| 8 | `Misc_03_DSA_Bruteforce_to_Optimal.md` | The brute-force → optimal method: constraint-to-complexity, bottleneck catalogue, worked transitions, templates | **Highest** |
| 9 | `Computer_Networks_Interview_Prep.md` | OSI/TCP-IP, TCP/UDP, DNS, HTTP, TLS, subnetting, 36 Q&A | High |
| 10 | `OS_Interview_Prep.md` | Processes, scheduling, deadlock, memory, paging, 42 Q&A | High |
| 11 | `DBMS_Interview_Prep.md` | Normalization, ACID, indexing, transactions, SQL patterns, 40 Q&A | High |
| 12 | `OOPS_Interview_Prep.md` | Four pillars, SOLID, patterns, 35 Q&A | High |
| 13 | `Skill_01_Programming_Languages.md` | C, C++, Python, Java, JavaScript, SQL, Bash | High |
| 14 | `Misc_01_DSA_and_Coding_Basics.md` | Complexity, data structures, algorithms, 38 practice problems, how to handle a coding question | High |
| 15 | `Skill_04_Libraries_Frameworks.md` | React, Node, Express, Judge0, Ollama, Chart.js | Medium |
| 16 | `Skill_05_Databases_and_Core_Domains.md` | Oracle, MySQL, MongoDB, AI, ML, Information Security | Medium |
| 17 | `Skill_03_Developer_Tooling.md` | Git and GitHub — model, commands, merge vs rebase, undoing | Medium |
| 18 | `Skill_02_Operating_Environments.md` | Linux, macOS, Windows as environments you work in | Medium |

Two files exist that you did not explicitly ask for, and here is why:
- **`Misc_01_DSA_and_Coding_Basics.md`** — nothing else in the folder covers complexity, data structures, or "how would you solve this", and that is very likely to come up in a technical round with seniors. It was the biggest actual gap.
- **`00_START_HERE.md`** (this file) — an index plus a study plan, so you know what to read first if time is short.

---

## Study order

**If you have several days:** read in the priority order above — the four project files first, then cybersecurity, then HR, then core subjects, then skills.

**If you have one day:**
1. All four project files (~2 hours). Everything else is recoverable in the room; forgetting your own project is not.
2. `Cybersecurity_Domain.md` — Parts 0, 1, 2, 4 (~1 hour).
3. `General_HR_and_Hobbies.md` — rehearse "tell me about yourself" and the CGPA answer out loud (~30 min).
4. The Q&A sections of the four core subject files (~1.5 hours).
5. `Misc_01_DSA_and_Coding_Basics.md` Parts 1, 5, 6 (~30 min).
6. `Misc_02_Question_Bank.md` — Sections 2 and 3 especially (~45 min).
7. `Misc_03_DSA_Bruteforce_to_Optimal.md` — Parts 2, 3 and 4 are the load-bearing ones (~30 min).

**If you have two hours:** the four project files, plus Part 0 of the cybersecurity file, plus Part 1 of the HR file.

**The morning of:** re-read only the four project files and Part 0 of the cybersecurity file. Do not try to learn anything new.

---

## The five things you must be able to say without hesitation

1. **"Tell me about yourself."** Ninety seconds. Rehearse it out loud until it is not memorised-sounding.
2. **A one-minute pitch for each of the four projects.** Each project file opens with one — those are written to be said aloud.
3. **"Which domain do you want and why?"** Cybersecurity, anchored in NetSpecter and ModelAuth. Part 0 of the cybersecurity file.
4. **"Why is your CGPA 6.84?"** Acknowledge, do not excuse, redirect to inspectable work. In the HR file, 2.11.
5. **Three questions to ask them.** HR file, 2.10.

---

## Facts and numbers worth memorising

These make answers concrete, and concrete answers are remembered.

**ClassRoom Code** — 191 unit and integration tests; 53 end-to-end verification checks plus 29 for the academic hierarchy; 4 program languages (C, C++, Java, Python) and 4 database engines (SQLite, PostgreSQL, Oracle, MongoDB); ~66 KB gzipped main bundle with Monaco as a separate ~840 KB chunk; 3 SQL migrations; returns 404 not 403 for resources you have no relationship with; 409 with a count before destroying submissions.

**ModelAuth** — 3 difficulty tiers, 4 detectors, 400 probes per run with the switch at probe 200, 15 null and 15 substitution runs per tier, 75 cold-start runs. Headline result: on the medium tier the sliding-window KS test gets **14.29%** detection power and adaptive CUSUM gets **92.86%** on the same data. Easy tier: detection in **+11 to +15 probes** at under 0.5% false alarms. Hard tier (quantisation only): **71.43%**. Cold start: above 85% recovery up to **25%** contamination, breaks down beyond 50%. Easy-tier separability KS = 0.659.

**NetSpecter** — 7 HTTP leak formats; v2 adds FTP, SMTP, POP3, IMAP, Redis; detects AWS, GitHub, Slack, Stripe keys and decodes JWT claims; TCP reassembly with a 64 KB per-flow buffer cap, 1000-flow table, 45-second idle timeout; BPF filter `tcp port 80 or tcp port 23`; requires root for raw sockets.

**ModeOS** — 11 shipped mode profiles; 4 audio backends, 4 display backends, 5 night-light backends; SIGTERM then a 2-second grace period then SIGKILL; nice range −20 to +19; runs entirely rootless; XDG-compliant paths.

---

## The three habits that decide the interview

1. **Say the limitation before you are asked.** "NetSpecter v1 misses split-packet payloads because it has no TCP reassembly — that is what v2 fixes." Volunteering the boundary of your own work is the single most credible thing you can do, and it makes everything else you say trustworthy.

2. **Never bluff.** "I have not used that directly, but based on X I would expect Y — is that right?" is a strong answer. A confident wrong answer is the only genuinely bad one, because the follow-up always exposes it.

3. **Use numbers.** 191 tests. 92.86% versus 14.29%. 60,000 probes. A 2-second grace period. Numbers prove you did the thing rather than read about it.

---

## What to have open in the room

- The four GitHub repositories, so you can navigate to the file behind any claim.
- Your resume, so you are never surprised by your own bullet point.
- This folder.

Good luck.
