# Project 1 — ClassRoom Code

**Repository:** https://github.com/praneeth3696/ClassRoom-Code
**Stack:** React 18 + Vite + Monaco Editor / Node.js + Express / PostgreSQL / Judge0 / Google OAuth 2.0 / Anthropic API
**Size:** ~95 files, monorepo with `server/` and `web/` packages, 191 unit and integration tests, 53 end-to-end verification checks, 29 department-structure checks.

This is my largest project and the one most likely to be drilled into. Read this file most carefully.

---

## 1. The one-minute pitch

"ClassRoom Code is a coding lab platform for a college department. Teachers publish worksheets of programming questions; students solve them in an in-browser editor and get auto-graded against test cases; teachers review submissions and leave feedback. It supports C, C++, Java, and Python for programming questions, and SQLite, PostgreSQL, Oracle, and MongoDB for database lab questions — which are judged on the rows they return rather than printed output. Authentication is Google OAuth restricted to the college domain, with per-department class hierarchies mapping programmes, batches, subjects, and staff. There is also an AI-assisted importer that takes the `.docx` or PDF question sheet a teacher already hands out and drafts the worksheet from it — but it never invents an expected output; it proposes a reference solution, the platform *runs* that solution against the real engine, and whatever it actually produced becomes the expected output."

## 2. The problem it solves

Coding labs in a college department run on a manual loop: the teacher hands out a printed or Word question sheet, students write code locally in whatever environment they have, and the teacher walks around the lab checking each screen. Problems with that:
- **Evaluation does not scale.** Sixty students times ten questions is six hundred manual checks per lab.
- **Feedback is verbal and lost.** Nothing is recorded, so neither the student nor the teacher can look back at what went wrong.
- **Environments differ.** A program that compiles on one student's machine fails on another's, and time goes into environment problems rather than the subject.
- **Database labs are worse**, because they need a seeded database per student, and one student's `DROP TABLE` breaks everyone sharing a login.
- **Existing platforms do not fit.** HackerRank and similar are built around competitive programming and individual accounts, not around a department's actual structure of programmes, batches, subjects, and co-taught classes — and they do not run Oracle object-relational features or real `mongosh` scripts, which are on the syllabus.

So the goal was: keep the teacher's existing workflow (they still write the same questions), automate the mechanical part (running and checking), record the feedback, and model the college's real academic structure rather than a generic "course" abstraction.

## 3. Architecture

```
Browser (React SPA)
  |  fetch with credentials, session cookie
  v
Express API (Node.js)
  |-- routes/      HTTP layer: parse, validate with Zod, authorise, respond
  |-- services/    business logic, no knowledge of req/res
  |-- db/          SQL, migrations, seeding
  |-- lib/         session JWT, http errors, language table
  |
  |--> PostgreSQL (or PGlite in development)
  |--> Judge0 CE (self-hosted, Docker) for C/C++/Java/Python
  |--> node:sqlite / PGlite / oracledb / mongosh for database questions
  |--> Anthropic API for the worksheet importer
  |--> Google OAuth 2.0 for sign-in
```

**Why this layering matters, if asked:** a route never writes SQL, and a service never touches `req` or `res`. That means the services are unit-testable without an HTTP server, and the same grading logic runs identically whether the code executed on Judge0 or in the local development runner.

**Deployment**: in development the API and the Vite dev server run as two processes, with Vite proxying `/api` to the server so the browser stays on one origin and the session cookie is first-party exactly as it is in production. For the college server, the frontend is built and the API serves the static files plus the SPA shell for any non-`/api` route, so it is a single process on one port and deep links and refreshes work.

## 4. Database schema — be ready to draw this

```
departments ──< programmes ──< batches ──< users (students)
                    │              │
                    └──< subjects  │
                          │        │
                    courses (subject_id, batch_id)
                       ├──< course_teachers  (course_id, user_id)  M:N
                       ├──< course_enrollments (course_id, user_id) M:N
                       └──< worksheets
                              └──< questions
                                     ├──< test_cases
                                     └──< submissions ──1:1── feedback
```

Design decisions I can defend:
- **UUID primary keys** (`gen_random_uuid()`) instead of sequential integers, so an id in a URL is not guessable and does not leak how many rows exist.
- **`CREATE UNIQUE INDEX users_email_lower_idx ON users (lower(email))`** — a functional index, so `Praneeth@x.edu` and `praneeth@x.edu` cannot both exist.
- **Composite primary keys on the junction tables** `(course_id, user_id)`, because a course is co-taught and a student takes many courses — both genuinely many-to-many — and the composite key prevents duplicate enrolment rows without needing an extra unique constraint.
- **`UNIQUE (question_id, student_id)` on submissions**, because the design keeps only the latest revision per student per question. Versioning was deliberately deferred, and I can say that honestly.
- **`ON DELETE CASCADE` where the child is meaningless without the parent** (a test case without its question), **`ON DELETE SET NULL` where it is not** (a course whose creator's account was removed should not vanish).
- **`CHECK` constraints for enum-like columns** (`role IN ('student','teacher','admin')`, `status IN ('draft','published')`) so bad data cannot enter even if the application has a bug. The database is the last line of defence, not the first.
- **JSONB columns** for genuinely schemaless data — `starter_code` per language and `last_run_result` — rather than adding a second database.
- **`allowed_languages text[]`** uses a PostgreSQL array, avoiding a junction table for what is a small fixed set.
- Migrations are numbered SQL files applied in order and recorded, so any environment reaches the same schema deterministically.

## 5. Authentication and authorisation

**Sign-in flow (Google OAuth 2.0 Authorization Code flow):**
1. Browser hits `GET /api/auth/google/start?next=/path`.
2. Server generates `state` and `nonce`, stores them in a **short-lived signed cookie** (10 minutes), and redirects to Google.
3. User authenticates with Google.
4. Google redirects to `GET /api/auth/google/callback?code=...&state=...`.
5. Server checks the returned `state` against the cookie — this proves the response answers a request *this server* started, which is the CSRF protection for OAuth. It also checks the `nonce` inside the id token to prevent replay.
6. Server exchanges the code for tokens, reads the user's email and `hd` claim, checks the domain restriction, finds or creates the user, and sets the session cookie.

**Session**: a signed JWT in an `httpOnly`, `SameSite=Lax`, `Secure`-in-production cookie.
- `httpOnly` — JavaScript cannot read it, so an XSS bug cannot exfiltrate the session.
- `SameSite=Lax` — the browser will not attach it to cross-site POST requests, mitigating CSRF.
- **The role is re-read from the database on every request rather than trusted from the token.** This is the point worth making: JWTs are self-contained and cannot be revoked before expiry, so if someone's teacher role is removed, a token-trusting design would leave them a teacher until it expired. Re-reading the role costs one indexed lookup and makes revocation immediate.
- **Domain restriction handles both Google account styles**: Workspace accounts are matched on the `hd` claim *or* the email suffix, plain accounts on the suffix alone. Matching is on the full domain, so `college.edu.attacker.com` is rejected — a suffix check without that anchoring is a real vulnerability.

**Authorisation model — two decisions worth describing:**
1. **Holding the `teacher` role is not enough to touch a given course.** A teacher must be *assigned* to it, because courses are co-taught. Assignment is the unit of permission, not the role. This is the difference between role-based access control and resource-level authorisation, and getting it wrong is one of the most common real-world API bugs (OWASP calls it Broken Access Control, the number one risk).
2. **Requests for a resource you have no relationship with return 404, not 403.** A 403 confirms the resource exists to someone outside it. The same applies to a draft worksheet viewed by a student: unpublished means invisible, not forbidden.

**Destructive operations are not silent**: deleting a worksheet or question that has student submissions returns **409 Conflict** naming how many submissions would be lost, and only goes through with `?force=true`, reporting the count.

**Development sign-in**: without Google credentials, `POST /api/auth/dev-login` starts a session as any *already-seeded* user — it never creates accounts — and is guarded by a `DEV_LOGIN` flag that the production config check refuses to let you enable. The server refuses to boot with an insecure production configuration rather than starting up quietly.

## 6. Code execution — the security heart of the project

**Why this is hard**: running arbitrary user-submitted code on your server is one of the most dangerous things a web application can do. Student code can read files, open network connections, fork endlessly, allocate all available memory, or loop forever.

**Production path — Judge0 CE, self-hosted with Docker.** Judge0 sandboxes execution using `isolate`, which is built on Linux **cgroups** (capping CPU, memory, process count, and disk) and **namespaces** (isolating the process, filesystem, and network view), plus wall-clock and CPU limits and output size caps. Self-hosting means no student code leaves college infrastructure and there is no per-request cost or rate limit.

**Implementation details worth naming:**
- I use the **batch submission API with polling**, not `wait=true`, because synchronous waiting is disabled by default on self-hosted instances.
- Source and stdin are **base64-encoded** in the request and decoded from the response.
- **Language ids are resolved from the instance's own `/languages` endpoint and cached for an hour**, preferring the newest compiler for each language, because a self-hosted instance may not carry the same numeric ids as the public one. The hard-coded ids are only a fallback.
- **Comparison happens in my application, not in Judge0.** Judge0 *can* compare against an `expected_output` itself, but I deliberately do not ask it to, so a submission is judged identically whichever executor ran it. Judge0 is trusted only for what it alone knows: compile errors, timeouts, signals, and resource usage.

**Development fallback, and why it is honest about itself**: with no Judge0 configured, code compiles and runs in a local subprocess (`cc`, `c++`, `javac`/`java`, `python3`) with a wall-clock timeout, a 64 KB output cap, and a **process-group kill** on timeout so a forked child cannot survive. But the README states plainly: **it is not a sandbox.** Student code runs as your user with full access to the machine. That is exactly the problem Judge0 exists to solve, which is why `ALLOW_LOCAL_EXECUTION` is refused when `NODE_ENV=production`. If Judge0 is configured but unreachable, development falls back locally and flags the result `degraded: true`; production has no fallback and returns 503. **Failing loudly beats grading wrongly.**

**Grading rules, and the reasoning behind each:**
- Trailing whitespace per line, trailing blank lines, and CRLF versus LF are ignored — a correct answer should not fail over a missing final newline, and a Windows student should not be penalised for line endings. Whitespace *inside* a line is still significant.
- A question passes automatically only if **every** test case passes.
- A question with **no test cases** is executed once with empty input so the student sees their output, and records `autoPassed: null` — teacher-graded only, not auto-failed.
- **Submit re-runs the code being submitted**, so the recorded pass/fail always describes the submitted answer rather than whatever was last "Run".
- Every test case is visible to the student — there is no hidden-test flag. This is a pedagogical choice: the platform is for learning, not for competition.

## 7. Database lab questions — the part nobody expects

Database questions are judged on **the rows returned**, not printed output. Each answer runs against a **freshly seeded database per student per run**, so a student can `CREATE TABLE` freely without colliding with the rest of the class. This is verified by a test.

Engines: SQLite via Node's built-in `node:sqlite`; PostgreSQL via PGlite in-memory per run; Oracle via a real Oracle server (`oracledb`); MongoDB via the real `mongosh`.

**Three things are deliberately ignored in the comparison, and each has a reason:**
1. **Column name case** — Oracle upper-cases unquoted identifiers, PostgreSQL lower-cases them. Penalising a student for the engine's own behaviour teaches nothing.
2. **Column order** — MongoDB does not preserve the field order written in a `$project`, so a correct pipeline can return `count, language` where the teacher wrote `language, count`.
3. **Row order**, unless the question explicitly ticks *row order matters* — because a query without `ORDER BY` has no defined row order. That is a direct application of relational theory.

Which columns exist, and every value in them, still count.

**Verification queries**: leaving the verification query empty judges whatever the student's own script returns, which covers ordinary "write a query" questions. Filling it in covers questions that ask the student to *create* something — it runs after their script, so `SELECT count(*) AS n FROM customer_2;` checks they really made the table and inserted the rows.

**Oracle war story** (a good answer to "tell me about something that surprised you"): I roll each run back afterwards, but **DDL commits implicitly in Oracle** — a `CREATE TABLE` ends the transaction and cannot be rolled back. So a shared login would accumulate every student's tables. The correct deployment is a separate schema per student. Also, Oracle's object types, `VARRAY`s, nested tables, `REF` types, and type inheritance genuinely cannot be emulated, which is why that engine talks to a real server and says "Oracle is not connected" rather than pretending, with PostgreSQL covering composite types and array collections in the meantime.

## 8. The AI worksheet importer — the most interesting design decision

**The feature**: upload the `.docx` or PDF question sheet you already hand out, and the platform drafts the worksheet from it.

**The design constraint that shaped everything**: an LLM will happily tell you what a question's expected output is, and it will sometimes be wrong. A wrong expected output **marks correct students wrong, and nobody notices** — the student assumes they are wrong. That is a silent, compounding failure, which is much worse than a visible one.

**So the model is never asked what the expected output is.** It reads the sheet and drafts the questions — titles, descriptions, references, languages, points, and a dataset when the sheet describes entities without giving data — and it supplies a **reference solution**. The platform then **executes that reference solution against the real engine**, and whatever it actually produced becomes the expected output. The trust boundary is drawn at "the model proposes, the runtime disposes."

**The consequence, stated plainly**: a question whose reference solution does not run gets **no test cases at all** and is imported as teacher-graded. Nothing is invented.

**The review screen** labels every question **Checked**, **Partly checked**, **You grade this**, or **Not checked**, and shows the computed expected output next to the solution that produced it. The teacher edits before anything is created, and the worksheet is always created as a **draft** to be published deliberately.

**A concrete engineering detail**: Word files convert through **HTML rather than markdown**, because the markdown converter escapes punctuation — which corrupts SQL — and drops tables, and the command references in these labs are mostly tables. PDFs are passed to the model as-is so their layout survives. That kind of detail is exactly what an interviewer means by "did you actually build it".

## 9. Frontend

One React application with **role-based views** — not two apps. The same route renders a teacher or student variant from the signed-in role.

| Route | Student sees | Teacher sees |
|---|---|---|
| `/` | Enrolled courses | Courses they teach, plus a create action |
| `/courses/:id` | Published worksheets | All worksheets with draft badges, plus the roster |
| `/worksheets/:id` | Questions with their own status | Questions with class-wide progress and review links |
| `/questions/:id` | Editor, Run, Submit, feedback | — |
| `/questions/:id/submissions` | — | Every submission, its result, and a feedback form |
| `/worksheets/:id/edit` | — | Worksheet and question authoring |

**The Monaco decision** — the best frontend answer I have: `@monaco-editor/react` fetches Monaco from jsDelivr by default. That would break the editor — the core of the student experience — on a college network that blocks external CDNs, or on an offline lab machine. So Monaco is **bundled locally**, trimmed to the four taught languages, and **code-split** so it downloads only when an editor is actually shown. The main bundle is ~66 KB gzipped with Monaco as a separate ~840 KB chunk. The lesson: a default that is fine on the open internet can be a total failure in the actual deployment environment, so you have to know where your software will run.

An `AuthContext` holds the signed-in user so any page can render the right variant from one source of truth, and a thin `api/client.js` wraps `fetch` with `credentials: 'include'` and centralises error handling.

## 10. Academic structure

```
Department   AMCS
  Programme    M.Sc Software Systems (also Theoretical CS, Data Science, Cyber Security)
    Batch        2023 intake, Section A — students with roll numbers
      Class        5SSL01 Big Data and Modern Databases Lab (Dr Anita Rao + Prof Vikram Shah)
      Class        5SSL03 Software Engineering Lab (Dr Meena Sundaram)
```
Each programme carries lab and theory subjects, and a subject may be taught by a different teacher for each batch. Creating a class and choosing a batch **enrols every student in it immediately**, so there is usually no roster to type. A six-character **join code** covers anyone not on the roll — a repeating student, a late admission — and it can be rotated or switched off; students never see the code for a class they are already in.

Students enrolled who have never signed in are created as **placeholder accounts**, and their Google account claims the row on first sign-in, keeping the enrolment and any work attached to it. That is a small but genuinely thoughtful piece of identity design.

## 11. Testing

- `npm test` — 191 unit and integration tests using Node's built-in test runner. **Each test process gets its own throwaway PGlite database**, so files run in parallel without touching each other or development data.
- `npm run verify` — 53 end-to-end checks walking the whole product against a running server: the teacher flow, the student flow, all four languages, the grading and feedback loop, the authorisation boundaries, and the deadline rules. Exits non-zero on any failure.
- `npm run verify:college` — 29 checks of the hierarchy and database labs.
- `npm run verify:solutions` — re-runs every reference solution.
The `verify` scripts need `DEV_LOGIN`, so they are staging and development checks rather than production ones — which is stated in the README rather than glossed over.

## 12. Questions they will ask, with answers

**"Why not just use HackerRank / Google Classroom?"**
Google Classroom has no execution or grading. HackerRank is built around competitive programming with individual accounts, and it will not run Oracle object-relational features or real `mongosh` scripts, both of which are on the department syllabus. Neither models the actual academic structure — programmes, batches, co-taught subjects — so the teacher would be maintaining rosters by hand. And student data would leave college infrastructure.

**"What was the hardest part?"**
Judging database questions fairly. Program output is a string you compare. A query result is a set of rows, and "the same answer" turns out to be ambiguous: Oracle upper-cases column names and PostgreSQL lower-cases them, MongoDB does not preserve `$project` field order, and any query without `ORDER BY` has no defined row order at all. I had to decide, for each of those, whether it was part of the answer or an artefact of the engine — and then make the row-order rule a per-question setting, because for a question specifically about `ORDER BY` it *is* the answer.

**"What would you do differently / what is missing?"**
Submission versioning — right now only the latest revision per student is kept, which loses the history of how a student arrived at an answer. Admin and HOD oversight views. Deployment onto actual college infrastructure with real Oracle credentials. And I would add rate limiting on the execution endpoints, because right now a student could hammer Run.

**"How do you prevent a student from cheating?"**
Honestly: this platform is not built to. Every test case is visible by design, because it is a learning tool rather than an exam. What it does provide is a record — the submitted code and the timestamp — so a teacher reviewing submissions can see identical answers. Real exam integrity would need hidden tests, a lockdown mode, and plagiarism detection like MOSS, and I would say that rather than overclaiming.

**"How would you scale it to the whole college?"**
The API is stateless apart from the database, so it scales horizontally behind a load balancer — the session is a signed cookie, not server memory. Judge0 is the bottleneck, and it scales by adding workers behind its queue. The database would need read replicas for the dashboard queries and connection pooling. I would add caching for the worksheet listing, which is read constantly and written rarely. And I would move execution to a proper queue so a lab of sixty students pressing Run at once degrades into a wait rather than a timeout.

**"What did you learn?"**
That the interesting decisions were almost never about which framework to use. They were about trust boundaries — what do I let a model decide, what do I let a student's code touch, what do I let a token assert about a user's role — and about failure modes, specifically preferring a loud failure over a quiet wrong answer. Returning 503 instead of grading with a degraded executor, and importing a question with no test cases instead of an invented expected output, are the same decision made twice.
