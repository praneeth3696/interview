# Technical Skills 1 — Programming Languages (C, C++, Python, Java, JavaScript, SQL, Bash)

This covers the "Programming Languages" line on my resume. For each language: what it is, why I used it, and the questions that actually get asked.

---

# PART A — C

## A1. What is C and why does it still matter?
A procedural, statically typed, compiled systems programming language from 1972. It sits one level above assembly: it gives direct memory access through pointers, has no runtime and no garbage collector, and compiles to small, fast native code. Operating system kernels, embedded firmware, database engines, and the runtimes of Python and Java are all written in C. Learning C is what makes memory, pointers, and the stack/heap distinction concrete.

## A2. Compilation stages
`gcc program.c -o program` runs four phases:
1. **Preprocessing** — handles `#include`, `#define`, and conditional compilation; produces expanded source.
2. **Compilation** — translates C to assembly, doing syntax and type checking.
3. **Assembly** — assembles into machine code, producing an object file (`.o`).
4. **Linking** — resolves references between object files and libraries, producing the executable.
Useful flags: `-Wall -Wextra` (warnings), `-g` (debug symbols for gdb), `-O2` (optimise), `-c` (compile only, no link).

## A3. Memory layout of a C program
| Segment | Holds | Lifetime |
|---|---|---|
| Text/Code | The compiled instructions, read-only | Program |
| Data | Initialised globals and statics | Program |
| BSS | Uninitialised globals and statics (zeroed) | Program |
| Heap | `malloc`/`calloc` allocations, grows upward | Until `free` |
| Stack | Local variables, parameters, return addresses, grows downward | Function call |

**Stack vs heap**: the stack is fast, automatically managed, and limited in size (stack overflow from deep recursion); the heap is larger, manually managed, slower, and leaks if you forget to `free`.

## A4. Pointers
```c
int x = 10;
int *p = &x;     // p holds the address of x
printf("%d", *p); // dereference: prints 10
```
- **Null pointer** — points to nothing; dereferencing it crashes. Always check.
- **Dangling pointer** — points to memory that has been freed. Set the pointer to `NULL` after `free`.
- **Wild pointer** — uninitialised, points somewhere random.
- **Void pointer** — `void *`, a generic pointer that must be cast before dereferencing; this is how `malloc` returns memory.
- **Pointer arithmetic** — `p + 1` moves by `sizeof(*p)` bytes, not one byte.
- **Double pointer** — `int **pp`, used to modify a pointer inside a function, and for arrays of strings (`char *argv[]`).
- **Function pointer** — `int (*fp)(int, int) = &add;` — the C mechanism behind callbacks and dispatch tables.

## A5. malloc / calloc / realloc / free
| Function | Behaviour |
|---|---|
| `malloc(n)` | Allocates n bytes, contents uninitialised (garbage) |
| `calloc(count, size)` | Allocates and zero-initialises |
| `realloc(p, n)` | Resizes a block, possibly moving it; returns the new pointer |
| `free(p)` | Releases the block; freeing twice or freeing a non-heap pointer is undefined behaviour |
Always check the return value for `NULL`, and never use a pointer after freeing it.

## A6. Arrays, strings, and structures
- An array name decays to a pointer to its first element when passed to a function, which is why `sizeof` inside the function gives the pointer size, not the array size.
- A C string is a `char` array terminated by `'\0'`. `strlen` does not count the terminator; the array must be one byte larger.
- `strcpy` and `strcat` are unsafe (no bounds check) and are a classic buffer-overflow source; prefer `strncpy`, `snprintf`.
- `struct` groups fields; **structure padding** aligns members to word boundaries, so `sizeof(struct)` can exceed the sum of its members. `union` overlays all members in the same memory, so only one is valid at a time.
- `typedef` creates an alias for a type.

## A7. Storage classes
`auto` (default, local), `register` (hint to keep in a CPU register), `static` (retains value between calls; at file scope limits visibility to that file), `extern` (declares a variable defined elsewhere).

## A8. Common C interview questions
- **Difference between `#define` and `const`?** `#define` is a preprocessor text substitution with no type and no scope; `const` is a typed, scoped variable the compiler knows about, so it gives better errors and debugging.
- **What is a memory leak?** Allocated heap memory that is no longer reachable and never freed. Detected with `valgrind`.
- **What is a segmentation fault?** The program accessed memory it does not own — dereferencing NULL or a dangling pointer, writing past an array, or stack overflow. The MMU raises it and the kernel sends SIGSEGV.
- **Call by value vs call by reference?** C is always call by value; passing a pointer simulates call by reference by copying the *address*.
- **What is undefined behaviour?** Code the standard does not define, so the compiler may do anything — signed overflow, out-of-bounds access, use after free. It is why C is fast and dangerous.
- **`i++` vs `++i`?** Post-increment returns the old value then increments; pre-increment increments then returns the new value.
- **Can you have recursion without a base case?** It will recurse until the stack overflows.

---

# PART B — C++

## B1. What C++ adds over C
C++ is a multi-paradigm superset-in-spirit of C, adding classes and objects, inheritance and polymorphism, function and operator overloading, references, templates and generic programming, exceptions, namespaces, the Standard Template Library, and RAII with constructors and destructors. It keeps C's performance and manual memory control while adding abstraction with, in principle, zero runtime cost.

## B2. References vs pointers
A reference is an alias for an existing object: it must be initialised at declaration, cannot be reseated, cannot be null, and needs no dereference syntax. A pointer can be null, reassigned, and arithmetic can be done on it. Prefer references for function parameters where the argument must exist; use pointers when the value is optional or must be reassigned.

## B3. The STL
- **Containers**: `vector` (dynamic array, contiguous, amortised O(1) push_back), `list` (doubly linked, O(1) insert anywhere given an iterator), `deque`, `set`/`map` (balanced BST, ordered, O(log n)), `unordered_set`/`unordered_map` (hash table, average O(1)), `stack`, `queue`, `priority_queue` (heap).
- **Iterators** generalise pointers and let algorithms work over any container.
- **Algorithms**: `sort`, `find`, `binary_search`, `lower_bound`, `accumulate`, `count`, `reverse`, `max_element`.
- `vector` versus `array`: `vector` grows dynamically by reallocating (usually doubling), `std::array` is a fixed-size stack-allocated wrapper.

## B4. Modern C++ points worth knowing
- **Smart pointers**: `unique_ptr` (exclusive ownership, move-only), `shared_ptr` (reference counted), `weak_ptr` (non-owning observer, breaks `shared_ptr` cycles). They implement RAII, so memory is released when the owner goes out of scope even if an exception is thrown.
- **Rule of Three / Five** — if you write a destructor, copy constructor, or copy assignment operator, you probably need all three; in modern C++ add the move constructor and move assignment.
- **Move semantics** — `std::move` casts to an rvalue reference so a resource can be transferred rather than copied, which is why returning a large `vector` by value is cheap.
- **`auto`** deduces the type; **range-based for** (`for (auto& x : v)`) iterates cleanly.
- **Lambdas** — `[capture](params) { body }` — anonymous functions with captured state.
- **`nullptr`** replaces `NULL` and is type-safe.
- **`const` correctness** — mark anything that does not modify as `const`, including member functions.

## B5. C vs C++ summary
| C | C++ |
|---|---|
| Procedural | Multi-paradigm (procedural + OOP + generic) |
| No classes | Classes, inheritance, polymorphism |
| `malloc`/`free` | `new`/`delete`, smart pointers, RAII |
| No overloading | Function and operator overloading |
| No exceptions | try/catch/throw |
| `printf`/`scanf` | `cout`/`cin` (type-safe streams) |
| No namespaces | Namespaces avoid name collisions |

---

# PART C — Python

## C1. What is Python and what characterises it?
A high-level, interpreted, dynamically typed, garbage-collected, multi-paradigm language. It emphasises readability (significant indentation instead of braces), has a very large standard library, and an enormous ecosystem — which is why it dominates scripting, automation, data science, machine learning, and rapid prototyping. It is slower than C/C++/Java because it is interpreted and dynamically typed, which is usually irrelevant because the heavy work happens in C libraries like NumPy.

## C2. Interpreted vs compiled — the accurate answer
Python source is compiled to **bytecode** (`.pyc`) and then executed by the CPython **virtual machine**. So it is compiled-then-interpreted, not purely interpreted. This matters because it explains `__pycache__` directories and why start-up is fast on a second run.

## C3. Mutable vs immutable
- **Immutable**: `int`, `float`, `str`, `tuple`, `bool`, `frozenset`. Rebinding creates a new object.
- **Mutable**: `list`, `dict`, `set`, and most custom class instances.
- **Why it matters**: a mutable default argument is the classic Python bug —
```python
def add(item, target=[]):   # WRONG: the list is created once, at definition
    target.append(item)
    return target
def add(item, target=None):  # correct
    target = [] if target is None else target
```
- Only immutable (hashable) objects can be dictionary keys or set members.

## C4. Core data structures
| Structure | Syntax | Properties |
|---|---|---|
| list | `[1, 2, 3]` | Ordered, mutable, allows duplicates, O(1) index, O(n) insert at front |
| tuple | `(1, 2, 3)` | Ordered, immutable, hashable, slightly faster |
| set | `{1, 2, 3}` | Unordered, unique, O(1) membership, supports union/intersection |
| dict | `{"a": 1}` | Key-value hash map, O(1) average lookup, insertion-ordered since 3.7 |

**List comprehension**: `[x*x for x in range(10) if x % 2 == 0]` — more readable and faster than an explicit loop. Dict and set comprehensions exist too.

## C5. Functions and functional features
- `*args` collects positional arguments into a tuple; `**kwargs` collects keyword arguments into a dict.
- **Lambda** — a one-expression anonymous function: `sorted(data, key=lambda x: x[1])`.
- `map`, `filter`, `reduce`, `zip`, `enumerate`, `any`, `all`, `sorted` with a `key`.
- **Decorator** — a function that wraps another to add behaviour without modifying it:
```python
def timed(fn):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = fn(*args, **kwargs)
        print(f"{fn.__name__} took {time.time()-start:.3f}s")
        return result
    return wrapper

@timed
def slow(): ...
```
Used for logging, caching (`@functools.lru_cache`), authentication, and timing.
- **Generator** — a function with `yield` that produces values lazily one at a time, holding only one in memory. Essential for streaming large files. `(x*x for x in range(10**9))` is a generator expression and uses constant memory.
- **Closure** — an inner function that captures variables from the enclosing scope.

## C6. OOP in Python
- `__init__` is the initialiser (the constructor proper is `__new__`); `self` is explicit.
- **Dunder methods** customise behaviour: `__str__` (human readable), `__repr__` (unambiguous, for developers), `__len__`, `__eq__`, `__hash__`, `__iter__`, `__enter__`/`__exit__` (context managers).
- `@staticmethod` (no implicit first argument), `@classmethod` (receives the class as `cls`, used for alternative constructors), `@property` (getter that looks like an attribute).
- **`@dataclass`** auto-generates `__init__`, `__repr__`, and `__eq__` from annotated fields. My ModeOS `ModeConfig` is a dataclass.
- **`abc.ABC` and `@abstractmethod`** define abstract base classes — used throughout ModeOS's backend interfaces.
- **MRO** resolves multiple inheritance using C3 linearisation.

## C7. The GIL (Global Interpreter Lock)
CPython allows only one thread to execute Python bytecode at a time, so threads do **not** give CPU parallelism. Consequences:
- **I/O-bound work** (network calls, file reads) benefits from threads, because the GIL is released during I/O waits. This is why ModelAuth uses `ThreadPoolExecutor` to issue LLM probes concurrently.
- **CPU-bound work** needs `multiprocessing` (separate processes, separate interpreters) or a C extension that releases the GIL (NumPy does).
- `asyncio` gives cooperative concurrency in a single thread with `async`/`await` — good for thousands of concurrent I/O operations.

## C8. Error handling and context managers
```python
try:
    ...
except ValueError as e:
    ...
except (TypeError, KeyError):
    ...
else:            # runs if no exception
    ...
finally:         # always runs
    ...

with open("f.txt") as f:   # context manager: closes automatically, even on exception
    data = f.read()
```
Never write a bare `except:` — it swallows `KeyboardInterrupt` and `SystemExit` too.

## C9. Common Python interview questions
- **`is` vs `==`?** `is` compares identity (same object in memory), `==` compares value. Use `is` only for `None`, `True`, `False`.
- **Shallow vs deep copy?** `copy.copy()` copies the outer object but shares nested objects; `copy.deepcopy()` recursively copies everything.
- **List vs tuple?** Mutability, hashability, and a small speed/memory advantage for tuples.
- **How does Python manage memory?** Reference counting plus a generational cycle collector, and a private heap managed by the interpreter's allocator.
- **What is PEP 8?** The style guide — 4-space indents, snake_case for functions and variables, PascalCase for classes, 79-character lines.
- **What is a virtual environment?** An isolated per-project directory of dependencies (`python -m venv .venv`), so projects do not fight over package versions. I use one in every project.
- **`__name__ == "__main__"`?** True only when the file is run directly rather than imported, so you can put a script entry point in an importable module. Every one of my Python projects uses it.

---

# PART D — Java

## D1. What is Java and what characterises it?
A statically typed, class-based, object-oriented, compiled-to-bytecode language designed for portability. `javac` compiles source to platform-independent `.class` bytecode, which the **JVM** executes — "write once, run anywhere". It has automatic garbage collection, no pointer arithmetic, strong typing, built-in multithreading, and a huge standard library. It dominates enterprise backends and Android.

## D2. JDK vs JRE vs JVM
- **JVM** — the abstract machine that loads and executes bytecode, does JIT compilation, and manages memory. Platform-specific.
- **JRE** — JVM plus the standard class libraries; enough to *run* Java.
- **JDK** — JRE plus development tools (`javac`, `javadoc`, `jdb`); needed to *write* Java.

## D3. Memory model
- **Heap** — all objects; split into Young Generation (Eden + 2 Survivor spaces) and Old Generation. Shared by all threads.
- **Stack** — one per thread; holds frames with local variables and partial results.
- **Metaspace** — class metadata (replaced PermGen in Java 8).
- **Garbage collection** — minor GC clears the young generation cheaply (most objects die young); major/full GC handles the old generation. Collectors: Serial, Parallel, CMS, G1 (default since 9), ZGC.

## D4. Primitives, wrappers, and Strings
- 8 primitives: `byte`, `short`, `int`, `long`, `float`, `double`, `char`, `boolean` — stored by value, not objects.
- Wrapper classes (`Integer`, `Double`) allow primitives in collections; **autoboxing** converts automatically. Watch out: `Integer` caches −128 to 127, so `==` on two `Integer`s outside that range is false even when the values match. Always use `.equals()`.
- **String is immutable** — every modification creates a new object. This makes strings safe to share and cache, allows the **string pool** (identical literals share one object), and makes them safe as HashMap keys. Use **StringBuilder** (not thread-safe, faster) or **StringBuffer** (synchronised) for repeated concatenation in a loop.

## D5. Collections framework
- `List` — `ArrayList` (dynamic array, O(1) random access, O(n) insert in the middle) vs `LinkedList` (O(1) insert/delete given a node, O(n) access).
- `Set` — `HashSet` (unordered, O(1)), `LinkedHashSet` (insertion order), `TreeSet` (sorted, O(log n)).
- `Map` — `HashMap` (unordered, allows one null key, not thread-safe), `LinkedHashMap`, `TreeMap` (sorted by key), `Hashtable` (legacy, synchronised), `ConcurrentHashMap` (thread-safe with segment/bucket-level locking, the modern choice).
- `Queue`/`Deque` — `ArrayDeque`, `PriorityQueue`.
- **HashMap internals**: an array of buckets; the key's `hashCode()` picks the bucket, `equals()` distinguishes keys within it. Collisions form a linked list, converted to a balanced tree above 8 entries in a bucket (Java 8+), giving O(log n) worst case instead of O(n).

## D6. Multithreading
- Create a thread by extending `Thread` or, better, implementing `Runnable`/`Callable` and submitting to an `ExecutorService`.
- Lifecycle: New → Runnable → Running → Blocked/Waiting/Timed Waiting → Terminated.
- `synchronized` on a method or block gives mutual exclusion using the object's intrinsic lock. `volatile` guarantees visibility of a variable across threads (but not atomicity). `java.util.concurrent` provides `AtomicInteger`, `ReentrantLock`, `CountDownLatch`, `Semaphore`, and the executor framework.
- `wait()`/`notify()`/`notifyAll()` for coordination, always inside a `synchronized` block and always in a `while` loop guarding the condition.

## D7. Common Java interview questions
- **Why is Java not 100% object-oriented?** Because of the eight primitive types.
- **`final`, `finally`, `finalize`?** `final` is a modifier (constant / no override / no subclass); `finally` is the always-executing block; `finalize()` was a deprecated GC hook.
- **Can you override a `static` method?** No — you hide it. Static binding uses the reference type.
- **Checked vs unchecked exceptions?** Checked must be declared or caught (`IOException`); unchecked extend `RuntimeException` and represent bugs (`NullPointerException`).
- **Interface vs abstract class?** See the OOPs file, section 7 and question 8.
- **What is the `Comparable` vs `Comparator` difference?** `Comparable` defines a class's natural ordering via `compareTo` inside the class; `Comparator` is an external, swappable ordering, which lets you sort the same type multiple ways.
- **Pass by value or reference?** Java is always pass by value — but for objects, the *reference* is what is passed by value, so the method can mutate the object but cannot make the caller's variable point elsewhere.

---

# PART E — JavaScript

## E1. What is JavaScript?
A dynamically typed, interpreted (JIT-compiled), single-threaded, prototype-based language originally for browsers and now also for servers through Node.js. It is the only language that runs natively in every browser. It is multi-paradigm — functional features and object-oriented `class` syntax both work.

## E2. var vs let vs const
| | Scope | Hoisting | Reassignable |
|---|---|---|---|
| `var` | Function | Hoisted and initialised to `undefined` | Yes |
| `let` | Block | Hoisted but in the temporal dead zone | Yes |
| `const` | Block | Same as let | No (but object contents can still change) |
Use `const` by default, `let` when you must reassign, and never `var` in new code.

## E3. The event loop — the most asked JS question
JavaScript is single-threaded, but non-blocking. The runtime has a **call stack**, a **task (macrotask) queue**, and a **microtask queue**. Asynchronous work (timers, network, file I/O) is handed to the environment (the browser's Web APIs or Node's libuv thread pool). When it completes, its callback is queued. The **event loop** takes work from a queue only when the call stack is empty, and it drains the **entire microtask queue** (promise callbacks, `queueMicrotask`) before taking the next macrotask (`setTimeout`, I/O callbacks). This is why:
```js
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
console.log('4');
// prints 1, 4, 3, 2
```

## E4. Callbacks, promises, async/await
- **Callback** — a function passed to be called later; nesting them deeply produces "callback hell".
- **Promise** — an object representing a future value, in one of three states: pending, fulfilled, rejected. Chained with `.then()`/`.catch()`/`.finally()`. Combinators: `Promise.all` (all must succeed), `allSettled` (wait for all outcomes), `race` (first to settle), `any` (first to succeed).
- **async/await** — syntactic sugar over promises that makes asynchronous code read synchronously. An `async` function always returns a promise; `await` pauses it until the promise settles. Wrap in `try/catch` for errors.

## E5. Closures
A closure is a function that retains access to variables from the scope where it was defined, even after that scope has returned.
```js
function counter() {
  let count = 0;                  // private state
  return () => ++count;
}
const next = counter();
next(); // 1
next(); // 2
```
Used for data privacy, factory functions, memoisation, and every callback that references outer variables. React hooks depend on closures.

## E6. `this`, prototypes, and classes
- `this` depends on **how a function is called**: in a method call it is the object; in a plain function call it is `undefined` in strict mode (or the global object otherwise); with `new` it is the new instance; with `call`/`apply`/`bind` it is what you specify.
- **Arrow functions do not have their own `this`** — they inherit it lexically, which is why they are the right choice for callbacks inside methods.
- JavaScript inherits through the **prototype chain**: every object has a hidden link to a prototype object, and property lookup walks that chain. `class` syntax is sugar over this.

## E7. Other frequently asked points
- **`==` vs `===`** — `==` performs type coercion (`'1' == 1` is true), `===` does not. Always use `===`.
- **Falsy values** — `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, `NaN`. Everything else is truthy, including `[]` and `{}`.
- **`null` vs `undefined`** — `undefined` means a variable was declared but never assigned (or a missing property); `null` is an explicit "no value" that you assigned.
- **Hoisting** — declarations are moved to the top of their scope at compile time; `function` declarations are fully hoisted, `var` is hoisted as `undefined`, `let`/`const` are hoisted but unusable until declared.
- **Spread/rest** — `...` spreads an iterable into elements or collects arguments into an array.
- **Destructuring** — `const { a, b } = obj;` and `const [x, y] = arr;`.
- **Array methods** — `map`, `filter`, `reduce`, `find`, `some`, `every`, `forEach`, `flat`, `includes`.
- **Shallow copy** — `{...obj}` and `Object.assign` copy one level; nested objects are still shared. Use `structuredClone()` for a deep copy.
- **Event bubbling and capturing** — an event travels down (capture) then up (bubble) the DOM; **event delegation** puts one listener on a parent instead of many on children.
- **Modules** — CommonJS (`require`/`module.exports`, Node's original) versus ES Modules (`import`/`export`, the standard). My ClassRoom Code server uses ES modules (`"type": "module"`).

---

# PART F — SQL

SQL is covered fully in **DBMS_Interview_Prep.md**, including the query patterns you must be able to write, joins, the logical order of evaluation, NULL semantics, indexing, and optimisation. The one-line summary for a rapid-fire round:

SQL is a **declarative** language — you describe the result you want and the query optimiser decides how to get it. That is the fundamental difference from C, Python, or Java, which are **imperative**: you describe the steps.

Key sublanguages: DDL (structure), DML (data), DCL (permissions), TCL (transactions).

---

# PART G — Bash / Shell Scripting

## G1. What is Bash?
Bash (Bourne Again Shell) is both an interactive command interpreter and a scripting language for Unix-like systems. It glues programs together: its power is that every command is a filter reading standard input and writing standard output, so complex work is composed by piping small tools.

## G2. Script basics
```bash
#!/usr/bin/env bash        # shebang: which interpreter runs this
set -euo pipefail          # exit on error, error on unset variable, catch failures in pipes

NAME="world"               # no spaces around =
echo "Hello, ${NAME}"      # quote your variables, always

if [[ -f "$file" ]]; then  # -f file exists, -d directory, -z empty string, -n non-empty
  echo "found"
elif [[ "$a" -gt 5 ]]; then  # -eq -ne -lt -le -gt -ge for numbers; == != for strings
  echo "big"
else
  echo "no"
fi

for f in *.log; do echo "$f"; done
while read -r line; do echo "$line"; done < input.txt

greet() { echo "hi $1"; }   # $1..$9 positional, $@ all args, $# count, $? last exit code, $$ pid
greet praneeth
```

## G3. Redirection and pipes
| Syntax | Meaning |
|---|---|
| `>` | Redirect stdout, overwriting |
| `>>` | Redirect stdout, appending |
| `2>` | Redirect stderr |
| `&>` or `> file 2>&1` | Redirect both |
| `<` | Read stdin from a file |
| `\|` | Pipe stdout of one command into stdin of the next |
| `\|\|` and `&&` | Run the next command on failure / on success |
| `;` | Run sequentially regardless |
| `&` | Run in the background |

## G4. Commands that come up constantly
- **Files**: `ls`, `cd`, `pwd`, `cp`, `mv`, `rm`, `mkdir`, `touch`, `find . -name "*.py"`, `ln -s`.
- **Viewing**: `cat`, `less`, `head -n 20`, `tail -f` (follow a log live), `wc -l`.
- **Text processing**: `grep -rn "pattern" .` (search recursively with line numbers), `sed 's/old/new/g'` (stream edit), `awk '{print $2}'` (column extraction), `cut -d',' -f1`, `sort`, `uniq -c`, `tr`.
- **Permissions**: `chmod 755 file` (owner rwx, group rx, others rx — read 4, write 2, execute 1), `chown user:group file`, `sudo`.
- **Processes**: `ps aux`, `top`/`htop`, `kill -15 <pid>` (SIGTERM), `kill -9 <pid>` (SIGKILL), `jobs`, `bg`, `fg`, `nohup`.
- **Network**: `curl`, `wget`, `ping`, `ss -tulpn`, `dig`, `ssh`, `scp`, `rsync`.
- **Disk**: `df -h`, `du -sh *`, `mount`.
- **Archives**: `tar -czf a.tar.gz dir/`, `tar -xzf a.tar.gz`, `zip`/`unzip`.

## G5. Classic one-liners to have ready
```bash
# 10 largest files under the current directory
du -ah . | sort -rh | head -10

# Count occurrences of each unique line
sort file.txt | uniq -c | sort -rn

# Find and delete all .pyc files
find . -name "*.pyc" -delete

# Most frequent IPs in an access log
awk '{print $1}' access.log | sort | uniq -c | sort -rn | head

# Which process is using port 8080
ss -tulpn | grep :8080

# Replace text across many files
grep -rl "oldname" . | xargs sed -i 's/oldname/newname/g'
```

## G6. Common Bash interview questions
- **Shell vs terminal vs kernel?** The kernel manages hardware; the shell is the program that interprets your commands; the terminal (emulator) is the window that displays them.
- **What does the shebang do?** Tells the kernel which interpreter to run the script with.
- **Difference between `$*` and `$@`?** Quoted, `"$@"` expands to each argument as a separate word (what you almost always want); `"$*"` joins them into one string.
- **What is an exit code?** 0 means success, non-zero means failure; available in `$?` and used by `&&`, `||`, and `set -e`.
- **Difference between `sh` and `bash`?** `sh` is the POSIX shell specification; `bash` is a superset with arrays, `[[ ]]`, brace expansion, and more. Scripts written for `bash` may break under `sh`.
- **Why quote variables?** Unquoted expansion splits on whitespace and does glob expansion, so a filename with a space becomes two arguments.
- **What is a cron job?** A scheduled task; `crontab -e` edits the schedule, with five fields: minute, hour, day of month, month, day of week.

---

# Cross-Language Comparison Table (a favourite closing question)

| | C | C++ | Java | Python | JavaScript |
|---|---|---|---|---|---|
| Typing | Static, weak | Static, stronger | Static, strong | Dynamic, strong | Dynamic, weak |
| Execution | Compiled to native | Compiled to native | Compiled to bytecode, JIT on JVM | Compiled to bytecode, interpreted | JIT compiled |
| Memory | Manual | Manual + RAII/smart pointers | Garbage collected | Reference counting + GC | Garbage collected |
| OOP | No | Yes | Yes (core) | Yes | Prototype-based + class sugar |
| Speed | Fastest | Near C | Fast after JIT warm-up | Slow | Fast for a dynamic language |
| Typical use | OS, embedded, drivers | Games, systems, high performance | Enterprise backends, Android | Scripting, ML, automation | Web front end, Node backends |

**"Which is your strongest language and why?"** — Answer: Python, because all four of my projects use it or were prototyped in it, and I have used it across three very different domains: packet parsing and byte-level protocol work in NetSpecter, statistical analysis with NumPy and SciPy in ModelAuth, and system-level process and hardware control with `psutil` and subprocess management in ModeOS. JavaScript is my second, from building the full ClassRoom Code stack in Node.js, Express, and React.
