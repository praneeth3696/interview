# Technical Skills 4 — Libraries and Frameworks (React.js, Node.js, Express.js, Judge0, Ollama, Chart.js)

This covers the "Libraries & Frameworks" line on my resume. Every one of these appears in a real project, so expect the interviewer to move straight from "what is it" to "how did you use it".

---

# PART A — Node.js

## A1. What is Node.js?
A **runtime**, not a framework or a language. It embeds Chrome's V8 JavaScript engine outside the browser and adds a standard library for file system, network, process, and cryptography access, so JavaScript can be used to write servers and command-line tools.

## A2. The architecture — this is the question that gets asked
Node is **single-threaded, event-driven, and non-blocking**. There is one main thread running your JavaScript, plus:
- **libuv** — the C library providing the event loop and a **thread pool** (4 threads by default) for operations that cannot be done asynchronously by the OS, such as file system work, DNS lookups, and crypto.
- **The event loop** — repeatedly checks queues and runs callbacks whose work has completed.

When you call `fs.readFile`, Node hands the work off and immediately continues executing; when the read finishes, the callback is queued and runs when the stack is clear. This is why Node handles thousands of concurrent connections with one thread while a thread-per-request server would need thousands of threads.

**Event loop phases** (worth naming): timers (`setTimeout`, `setInterval`) → pending callbacks → poll (I/O) → check (`setImmediate`) → close callbacks. Between every phase, the microtask queue drains: `process.nextTick` first, then promise callbacks.

## A3. What Node is good and bad at
**Good**: I/O-bound workloads — APIs, real-time applications, proxies, streaming, anything waiting on the network or disk. One language across front end and back end. A huge package ecosystem.
**Bad**: CPU-bound work — a long computation blocks the single thread and every other request stalls. The fix is `worker_threads`, `child_process`, or offloading to another service. In ClassRoom Code, running student code is exactly this kind of blocking work, which is one more reason it goes to Judge0 rather than being executed inline.

## A4. Node concepts to have ready
- **npm** is the package manager; `package.json` declares dependencies and scripts; `package-lock.json` pins the exact resolved versions so installs are reproducible. `dependencies` ship to production, `devDependencies` do not.
- **Semantic versioning**: `^1.2.3` allows minor and patch updates; `~1.2.3` allows only patch; `1.2.3` pins exactly.
- **CommonJS vs ES Modules**: `require`/`module.exports` versus `import`/`export`. My ClassRoom Code server sets `"type": "module"` and uses ES modules throughout.
- **Streams** process data in chunks instead of loading it all into memory — readable, writable, duplex, transform. Essential for large files.
- **Buffer** holds raw binary data outside the V8 heap. My Judge0 client base64-encodes source and input using `Buffer.from(s).toString('base64')`.
- **`process.env`** reads environment variables — the correct place for configuration and secrets.
- **Error handling**: an unhandled rejection or an uncaught exception can crash the process, so async handlers need `try/catch` and Express needs an error-handling middleware.

---

# PART B — Express.js

## B1. What is Express?
A minimal, unopinionated web framework for Node that provides routing, middleware, and helpers for requests and responses. It does not impose a project structure — which is why you have to impose one yourself.

## B2. Middleware — the core concept
An Express application is a **pipeline of middleware functions**, each with the signature `(req, res, next)`. Each function can inspect or modify the request and response, end the cycle by sending a response, or call `next()` to pass control on. Order matters: middleware runs in the order it is registered.

```js
app.use(express.json());                    // parse JSON bodies
app.use(cookieParser());                    // parse cookies into req.cookies
app.use(cors({ origin, credentials: true }));
app.use('/api/courses', requireAuth, coursesRouter);   // route-level middleware
app.use((err, req, res, next) => {          // error handler: four arguments
  res.status(err.status || 500).json({ error: err.message });
});
```
Types: application-level, router-level, route-specific, built-in (`express.json`, `express.static`), third-party (`cors`, `cookie-parser`, `multer`), and error-handling (identified by having four parameters).

## B3. How I structured the ClassRoom Code server
```
src/
  index.js        entry point, starts the HTTP server
  app.js          builds the Express app and registers middleware
  config.js       reads and validates environment variables
  db/             connection, migrations, seeding
  middleware/     auth (session verification), errors
  routes/         one router per resource: auth, courses, worksheets, submissions, review, imports
  services/       the actual business logic, no HTTP knowledge
  lib/            shared helpers: http errors, session JWT, language definitions
```
The point to make in an interview: **routes handle HTTP, services handle logic, the db layer handles SQL.** A route never writes SQL and a service never touches `req` or `res`. That separation is what makes the services testable without spinning up a server, and it is why the same submission-grading logic works identically whether the code ran on Judge0 or in a local subprocess.

## B4. REST API design points
- Resources are nouns, and the HTTP method is the verb: `GET /api/courses`, `POST /api/courses`, `PATCH /api/courses/:id`, `DELETE /api/courses/:id`.
- Return the right status code: 200 OK, 201 Created, 204 No Content, 400 Bad Request, 401 Unauthorized (not authenticated), 403 Forbidden (authenticated but not allowed), 404 Not Found, 409 Conflict, 422 Unprocessable, 429 Too Many Requests, 500 Server Error, 503 Unavailable.
- **Validate every input.** I use **Zod** schemas, with unknown keys treated as errors rather than ignored, so a misspelled `testcases` is reported instead of silently producing a question with no test cases.
- Two authorisation decisions from my project that are worth describing because they show security thinking:
  1. **Holding the `teacher` role is not enough to touch a course** — the teacher must be *assigned* to that course, because courses are co-taught. Assignment, not role, is the unit of permission.
  2. **Requests for a resource you have no relationship with return 404, not 403.** A 403 confirms the resource exists, which leaks information to someone outside it. An unpublished worksheet is invisible to a student rather than forbidden.
- Destructive operations are not silent: deleting a worksheet that has student submissions returns **409 Conflict** naming how many submissions would be lost, and only proceeds with `?force=true`.

## B5. Authentication in my project
Sessions are a **signed JWT in an `httpOnly`, `SameSite=Lax`, `Secure`-in-production cookie**.
- `httpOnly` means JavaScript cannot read it, so an XSS bug cannot steal the session.
- `SameSite=Lax` means the browser will not send it on cross-site POSTs, which mitigates CSRF.
- The **role is re-read from the database on every request** rather than trusted from the token — so revoking someone's teacher role takes effect immediately instead of when their token expires. This is the classic JWT weakness (tokens cannot be revoked), and that is how I worked around it.
- The **OAuth `state` and `nonce`** are carried across the Google redirect in a separate short-lived signed cookie, so the callback can prove it is answering a request this server actually started (CSRF protection) and that the id token is not being replayed.
- Domain restriction matches on the full domain, so `college.edu.attacker.com` is rejected — a subtle but real check.

---

# PART C — React.js

## C1. What is React?
A JavaScript library for building user interfaces out of **components**. Its central ideas are: describe the UI **declaratively** as a function of state, compose small components into large ones, and let React work out the minimal DOM changes needed when the state changes.

## C2. Core concepts
- **Component** — a function returning JSX. Named in PascalCase.
- **JSX** — HTML-like syntax that compiles to `React.createElement` calls. `className` instead of `class`, `{}` to embed expressions.
- **Props** — read-only inputs passed from parent to child. Data flows one way, downward.
- **State** — data owned by a component that can change over time; changing it triggers a re-render.
- **Virtual DOM** — React keeps an in-memory tree, diffs the new tree against the old one (reconciliation), and applies only the necessary real DOM updates. Direct DOM manipulation is slow; batching and minimising it is why this is fast.
- **Keys** — a stable, unique `key` on list items lets the diffing algorithm match elements across renders. Using the array index as a key causes bugs when the list is reordered or filtered.

## C3. Hooks
| Hook | Purpose |
|---|---|
| `useState` | Local state in a function component |
| `useEffect` | Side effects after render — fetching, subscriptions, timers. The dependency array controls when it re-runs; return a cleanup function to unsubscribe |
| `useContext` | Read a value from a Context provider without prop drilling |
| `useRef` | A mutable value that persists across renders without causing one; also used to hold a DOM node |
| `useMemo` | Cache an expensive computed value |
| `useCallback` | Cache a function identity so a memoised child does not re-render |
| `useReducer` | State transitions via a reducer, for complex or interdependent state |

**Rules of hooks**: only call them at the top level (never inside conditionals or loops), and only from React functions. React tracks hooks by call order, which is why the order must be stable.

**`useEffect` gotchas**: an empty dependency array runs once on mount; omitting the array runs on every render; a missing dependency captures a stale closure value. The cleanup function runs before the next effect and on unmount.

## C4. State management and data flow
- **Lifting state up** — when two siblings need the same data, move it into their common parent.
- **Prop drilling** — passing props through many intermediate layers; solved by Context or a state library.
- **Context** — `createContext` plus a provider makes a value available anywhere below it. My ClassRoom Code frontend uses an `AuthContext` holding the signed-in user, so any page can render a teacher or a student variant from one source of truth.
- **Controlled component** — the form input's value comes from state and every keystroke updates it, so React owns the value. **Uncontrolled** — the DOM keeps the value and you read it with a ref.

## C5. How I used React in ClassRoom Code
- **One application with role-based views**, not two apps. The same route renders a teacher or student variant depending on the signed-in role — `/courses/:id` shows published worksheets to a student and all worksheets with draft badges plus the roster to a teacher.
- **React Router** for client-side routing, with the Express server serving the SPA shell for any non-`/api` route so deep links and refreshes work in the single-process deployment.
- **Monaco Editor** (the editor that powers VS Code) is the code editor students type in.
- **A real engineering decision worth describing**: `@monaco-editor/react` loads Monaco from a jsDelivr CDN by default. That would break the editor — the core of the student experience — on a college network that blocks external CDNs or on an offline lab machine. So I **bundled Monaco locally**, trimmed it to the four taught languages, and **code-split** it so it downloads only when an editor is actually shown. The result is a ~66 KB gzipped main bundle with Monaco as a separate ~840 KB chunk. That is a good answer to "tell me about a performance or reliability decision you made."
- A thin `api/client.js` wraps `fetch` with `credentials: 'include'` so the session cookie is sent, and centralises error handling.

## C6. React questions to expect
- **Why React over plain DOM manipulation?** Declarative code is easier to reason about, components are reusable and independently testable, and the reconciler batches updates so you do not hand-optimise DOM writes.
- **Class components vs function components?** Function components with hooks are the modern standard — less boilerplate, no `this` confusion, and logic is reusable through custom hooks. Class components use lifecycle methods (`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`) which `useEffect` subsumes.
- **What causes a re-render?** A state change, a prop change, a context value change, or a parent re-rendering.
- **How do you optimise a slow React app?** `React.memo` for pure components, `useMemo`/`useCallback` to stabilise expensive values and function identities, correct `key` usage, code splitting with `React.lazy` and `Suspense`, virtualising long lists, and measuring with the React Profiler first.
- **What is the Virtual DOM diffing algorithm?** React compares element types first — a different type means tear down and rebuild the subtree — then props, then children matched by key. It is O(n) rather than the O(n³) of a general tree diff, by making those two assumptions.

---

# PART D — Judge0

## D1. What is Judge0?
An **open-source online code execution system**. You POST source code, an optional stdin, and a language id to its REST API; it compiles and runs the code inside a sandbox and returns stdout, stderr, compile output, exit status, execution time, and memory usage. It supports 60+ languages and is what powers many online judges and coding-interview platforms.

## D2. Why it exists — the security answer
Running arbitrary user-submitted code on your own server is one of the most dangerous things a web application can do. The code can read files, open network connections, fork bombs, consume all memory, or run indefinitely. Judge0 solves this using **isolate**, a sandbox built on Linux **cgroups** (to cap CPU, memory, processes, and disk) and **namespaces** (to isolate the process, filesystem, and network view), plus wall-clock and CPU time limits and output size caps.

This is a strong point to make in an interview: my project's local fallback runner compiles and runs code in a subprocess with a wall-clock timeout, a 64 KB output cap, and a process-group kill on timeout — **and my README states plainly that it is not a sandbox.** Student code runs as my user with full access to the machine. That is precisely the problem Judge0 exists to solve, which is why `ALLOW_LOCAL_EXECUTION` is refused when `NODE_ENV=production`, and why production returns 503 rather than falling back.

## D3. How I integrated it
- Judge0 CE is **self-hosted with Docker** so no student code leaves college infrastructure and there is no per-request cost or rate limit.
- I use the **batch submission API with polling** rather than `wait=true`, because synchronous waiting is disabled by default on self-hosted instances.
- Source and stdin are **base64-encoded** in the request and decoded from the response.
- **Language ids are resolved from the instance's own `/languages` endpoint and cached for an hour**, preferring the newest compiler for each language — because a self-hosted instance may not carry the same numeric ids as the public one. The hard-coded ids in `lib/languages.js` are only a fallback for when that call fails.
- **The comparison happens in my application, not in Judge0.** Judge0 can compare against an `expected_output` itself, but I deliberately do not ask it to, so a submission is judged identically whichever executor ran it. Judge0 is trusted only for what it alone knows: compile errors, timeouts, signals, and resource usage.
- Grading rules: trailing whitespace per line, trailing blank lines, and CRLF versus LF are ignored, but whitespace *inside* a line is significant. A question passes only if every test case passes. A question with no test cases is executed once with empty input and recorded as teacher-graded rather than auto-passed.
- Judge0 status codes I handle: 3 Accepted, 4 Wrong Answer, 5 Time Limit Exceeded, 6 Compilation Error, 7–12 various runtime errors (SIGSEGV, SIGFPE, SIGABRT, non-zero exit code), 13 Internal Error, 14 Exec Format Error.
- If Judge0 is configured but unreachable, development falls back locally and flags the result `degraded: true`; production has no fallback and returns 503. Failing loudly beats grading wrongly.

---

# PART E — Ollama

## E1. What is Ollama?
A tool for running **large language models locally** on your own machine. It handles downloading and managing model weights, quantised for consumer hardware, and exposes a local **REST API on port 11434** — including an **OpenAI-compatible endpoint**, so any OpenAI client library works by pointing its `base_url` at `http://localhost:11434/v1`.

`ollama pull llama3.2:3b` fetches a model; `ollama run` starts an interactive session; the API serves `/api/generate` and `/v1/chat/completions`.

## E2. Why I used it in ModelAuth
ModelAuth needs to run **controlled experiments where I know the ground truth** — the exact probe at which one model was swapped for another. That is impossible with a commercial API, because you cannot make a provider substitute a model on demand and you cannot verify what they actually served. Ollama lets me host both models locally and switch between them at a known index, so I have labelled data to measure detection delay and false alarm rates against.

It also let me construct three difficulty tiers by choosing model pairs:
- **Easy** — `llama3.2:3b` → `qwen2.5:3b` (different architectures entirely).
- **Medium** — `llama3.2:1b` → `llama3.2:3b` (same family, different capacity).
- **Hard** — `llama3.2:3b-instruct-q4_K_M` → `q8_0` (identical architecture and parameter count, differing only in quantisation precision).

## E3. Concepts worth knowing when Ollama comes up
- **Quantisation** — storing weights at lower precision (8-bit, 4-bit) to cut memory and increase speed, at some cost in output quality. `q4_K_M` and `q8_0` are quantisation formats. My hard tier exists to test whether that quality cost is statistically detectable from outputs alone — and it partly is: adaptive CUSUM reached 71.43% detection power on it.
- **Parameters** — `3b` means three billion parameters. More parameters generally means more capability and more memory.
- **Temperature** — controls sampling randomness. I deliberately use **temperature 1.0**, because the whole method depends on the model's output *distribution* being visible; at temperature 0 every model would give a nearly deterministic answer and there would be no distribution to compare.
- **`max_tokens`** — I cap it at 5, because each probe asks only for a single number.
- **Why local over an API?** Ground truth, reproducibility, no per-token cost across ~19,000 probes, no rate limits, and no data leaving the machine.

---

# PART F — Chart.js

## F1. What is Chart.js?
A small, open-source JavaScript charting library that renders to an HTML5 `<canvas>` element. It supports line, bar, pie, doughnut, radar, scatter, bubble, and polar charts through a simple declarative config object of `{ type, data, options }`.

## F2. Canvas vs SVG — the question behind the question
Chart.js draws on **canvas**, which is a single raster element — fast for many data points because there is one DOM node no matter how much you draw, but the individual marks are not DOM elements, so they cannot be styled with CSS or selected by a screen reader, and it does not scale losslessly. D3 typically draws **SVG**, where every mark is a DOM node — inspectable, stylable, and accessible, but slow past a few thousand elements. Choose canvas for volume, SVG for interactivity and accessibility.

## F3. How I used it
ModelAuth's `final-analysis/interactive_dashboard.py` **generates a standalone HTML dashboard** with Chart.js loaded from a CDN, containing four charts: detection power across the three tiers, detection delay across tiers, an ROC comparison, and the cold-start contamination recovery curve. The Python script writes the HTML and injects the computed results as JSON — so the analysis pipeline produces a shareable artefact that opens in any browser with no server and no build step.

The static figures (traces, ROC curves, distribution separability, the contamination boundary) are generated separately with **Matplotlib** and saved as PNGs, alongside CSV summary tables. Chart.js is for the interactive view; Matplotlib is for the report figures.

## F4. Charting points that generalise
- **Choose the mark for the question**: line for a value over an ordered axis (my probe traces over time), bar for comparison across categories (detection power per detector per tier), scatter for the relationship between two continuous variables (ROC: detection delay against false alarm rate).
- Always label axes with units, and never truncate a bar chart's y-axis at a non-zero baseline — it exaggerates differences.
- Keep the number of colours small and consistent across every chart in a report, so a reader learns the mapping once.

---

# Cross-Cutting Question: "How do these fit together?"

Have this ready as a single sentence per project:

- **ClassRoom Code** — React (Vite, React Router, Monaco) in the browser talks over a REST API to an Express server on Node.js, which persists to PostgreSQL and delegates untrusted code execution to Judge0, with Anthropic's SDK used server-side for the worksheet importer.
- **ModelAuth** — Python with NumPy and SciPy for the detectors, Ollama serving the local models over an OpenAI-compatible REST API, Matplotlib for the report figures, and Chart.js for the interactive dashboard.
- **NetSpecter** — Python with Scapy for capture and Rich for the terminal UI, with a v2 branch adding a small web dashboard.
- **ModeOS** — Python with psutil for process control and PyYAML for the mode profiles, wrapping Linux system tools behind pluggable backends.
