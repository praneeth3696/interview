# Miscellaneous 3 — Brute Force to Optimal: The Problem-Solving Method

**Read this if the round works like:** they state a problem → you give the brute force → then you find the right data structure and algorithm.

That format is not testing whether you have memorised solutions. It is testing one specific skill: **can you look at a slow solution, name exactly what is wasteful about it, and pick the structure that removes that waste.** This file is about that transition. The reference material — complexity tables, data structures, algorithms — is in `Misc_01_DSA_and_Coding_Basics.md`.

The whole method is three sentences:
> The brute force does *this* repeatedly. Repeating *this* is what costs me. The structure that makes *this* cheap is *that*.

---

# PART 1 — The Protocol

Follow this order every time. It works even when you do not know the answer.

**Step 1 — Restate and clarify.** Thirty seconds, non-negotiable.
> "So: given an array of integers and a target, return the two indices that sum to it. Can I assume exactly one answer exists? Can the array contain duplicates or negatives? Is it sorted? Roughly how large is n?"

That last question is not small talk — see Part 2.

**Step 2 — Give the brute force immediately, with its complexity.** Do not try to be clever first. You want a correct solution on the table inside a minute.
> "The obvious approach is to check every pair with a nested loop. That is O(n²) time and O(1) space. It is correct, so let me use it as the baseline and see what is wasteful about it."

Saying *"it is correct, so let me use it as the baseline"* is the sentence that buys you thinking time and makes you sound deliberate rather than stuck.

**Step 3 — Name the waste out loud.** This is the actual skill. Do not jump to an answer; describe the repeated work.
> "For every element I am scanning the rest of the array looking for `target - x`. That is a *search*, and I am redoing it n times from scratch. The search is the expensive part."

**Step 4 — Map the waste to a structure.** Now you are just doing a lookup — Part 3 is that lookup table.
> "Searching is what a hash map makes O(1). If I store the values I have already seen as I go, each lookup is constant time instead of linear."

**Step 5 — State the new complexity before writing anything.**
> "That gives me one pass, O(n) time and O(n) space — I have traded memory for time. Shall I code that?"

**Step 6 — Code it.** Meaningful names. Talk while you type, but not constantly.

**Step 7 — Trace a small example by hand, out loud.** Catches most bugs before they do.

**Step 8 — Edge cases, then final complexity, unprompted.** Empty input, one element, all duplicates, target at the boundaries, integer overflow.

**Step 9 — Say what you would improve.** "If the array were sorted I could do it with two pointers in O(1) extra space."

---

# PART 2 — Read the Constraints; They Tell You the Answer

This is the single most underused trick, and it is pure competitive-programming instinct. The intended complexity is usually inferable from the input size, because judges and interviewers pick limits deliberately. Roughly 10⁸ simple operations per second is the working assumption.

| n up to | Intended complexity | Technique that fits |
|---|---|---|
| 10 – 12 | O(n!) | Permutations, brute-force backtracking |
| 20 – 25 | O(2ⁿ) | Subsets, bitmask DP, meet-in-the-middle |
| 100 | O(n³) | Floyd–Warshall, 3-nested DP |
| 1,000 – 2,000 | O(n²) | 2-D DP, all pairs |
| 10⁵ | O(n log n) | Sorting, heap, binary search, divide and conquer |
| 10⁶ | O(n) | Single pass, two pointers, sliding window, counting |
| 10⁹+ | O(log n) or O(√n) | Binary search on the answer, maths, fast exponentiation |

**How to use it out loud**, which sounds extremely competent:
> "n is up to 10⁵, so O(n²) is about 10¹⁰ operations — far too slow. That rules out checking all pairs and points me at O(n log n), which usually means sorting, a heap, or binary search. Let me look for which one applies."

You have now narrowed the search space *before* having an idea. That is exactly what they want to see.

**The reverse also works:** if n ≤ 20, stop trying to be clever — exponential is intended, and a clean backtracking solution is the correct answer.

---

# PART 3 — The Bottleneck Catalogue

This is the core of the file. Find what your brute force repeats, read across.

| Your brute force repeatedly… | Use | Typical improvement |
|---|---|---|
| searches for a value | **hash set / hash map** | O(n) → O(1) |
| searches *sorted* data | **binary search** | O(n) → O(log n) |
| recomputes a sum over a range | **prefix sums** | O(n) → O(1) per query |
| recomputes an overlapping subproblem | **memoisation / DP** | exponential → polynomial |
| finds the min or max of a changing set | **heap** | O(n) → O(log n) |
| compares every pair | **sort, then two pointers** | O(n²) → O(n log n) |
| recomputes an overlapping window | **sliding window** | O(n·k) → O(n) |
| looks left/right for the next greater or smaller | **monotonic stack** | O(n²) → O(n) |
| matches string prefixes | **trie** | O(n·m) → O(m) |
| merges groups or asks "same group?" | **union–find** | O(n²) → near O(n) |
| tries every ordering | **greedy** (only if an exchange argument holds) | O(n!) → O(n log n) |
| does range query *and* update | **segment tree / BIT** | O(n) → O(log n) |
| counts pairs or inversions | **merge sort / BIT** | O(n²) → O(n log n) |
| explores paths, unweighted | **BFS** | — |
| explores paths, weighted non-negative | **Dijkstra + heap** | — |
| tests each number for primality | **sieve** | O(n√n) → O(n log log n) |
| tries every possible answer value | **binary search on the answer** | O(range) → O(log range) |
| needs both the median and O(log n) inserts | **two heaps** | — |
| needs O(1) lookup *and* O(1) ordering | **hash map + doubly linked list** | — |

**The four moves that cover most interview problems** — if you remember nothing else:
1. **Searching → hash map.** By far the most common.
2. **Recomputing → store it** (prefix sums, memoisation, DP).
3. **All pairs → sort first**, then two pointers or binary search.
4. **Overlapping windows → slide, do not rebuild.**

---

# PART 4 — Signals in the Problem Statement

Certain phrasings almost always point at one technique. Read the statement for these before you think.

| Phrase | Reach for |
|---|---|
| "kth largest / smallest", "top k" | Heap of size k, or quickselect |
| "contiguous subarray" | Sliding window, prefix sums, or Kadane |
| "subsequence" (not contiguous) | Dynamic programming |
| "all subsets / permutations / combinations" | Backtracking |
| "shortest path", unweighted | BFS |
| "shortest path", weighted | Dijkstra |
| "prerequisites", "build order", "ordering" | Topological sort |
| "cycle", "connected", "groups merging" | DFS or union–find |
| "maximum X such that Y is feasible" | Binary search on the answer |
| "next greater / previous smaller" | Monotonic stack |
| "the array is sorted" | Two pointers or binary search — never scan |
| "intervals", "meetings", "overlap" | Sort by start, then sweep |
| "median of a stream" | Two heaps (max-heap low half, min-heap high half) |
| "prefix", "autocomplete", "dictionary" | Trie |
| "palindrome" | Two pointers, expand-from-centre, or DP |
| "in-place", "O(1) extra space" | Two pointers, or index-as-hash trickery |
| "count the ways" | DP |
| "minimum number of steps" | BFS on the state graph |
| "cache", "least recently used" | Hash map + doubly linked list |

---

# PART 5 — Worked Transitions

Six problems, narrated the way you should narrate them. Notice the shape is always the same.

### 5.1 Two Sum — search elimination
**Brute force.** Check every pair. O(n²) time, O(1) space.
**The waste.** "For each `x` I am scanning the array for `target - x`. That is a repeated search."
**The move.** Searching → hash map. Store each value's index as I pass it; for each `x`, check whether `target - x` is already in the map.
**Result.** O(n) time, O(n) space. One pass.
```python
seen = {}
for i, x in enumerate(nums):
    if target - x in seen:
        return [seen[target - x], i]
    seen[x] = i
```

### 5.2 Maximum Subarray — three stages, and the DP insight
**Brute force.** Every start, every end, sum each. O(n³).
**First cut.** Carry a running sum as the end moves — the sum is being recomputed. O(n²).
**The real waste.** "I am still restarting at every index. But at position i I only need one thing: the best subarray *ending here*. And that is either the previous best extended by `x`, or `x` alone."
**The move.** That recurrence is Kadane's algorithm.
**Result.** O(n) time, O(1) space.
```python
best = cur = nums[0]
for x in nums[1:]:
    cur = max(x, cur + x)      # extend, or start fresh
    best = max(best, cur)
```
Say the recurrence out loud — it shows you derived it rather than recalled it.

### 5.3 Longest Substring Without Repeating Characters — window rebuilding
**Brute force.** Every substring, check each for duplicates. O(n³).
**The waste.** "Each window is rebuilt from scratch, and I recheck characters I already validated."
**The move.** Overlapping windows → slide. Keep a left pointer and a set; when a duplicate appears, advance left past its previous position instead of restarting.
**Result.** O(n) time, O(min(n, alphabet)) space — each character enters and leaves the window once.

### 5.4 Kth Largest Element — three valid answers, know the trade-offs
**Brute force.** Sort descending, take index k−1. O(n log n).
**Better when k is small.** A min-heap of size k: push, and pop when it exceeds k. The root is the answer. O(n log k) time, O(k) space — and it works on a **stream**, where sorting cannot.
**Better on average.** Quickselect — partition like quicksort but recurse into one side only. O(n) average, O(n²) worst.
**What to say:** "Sorting is the simplest. If k is much smaller than n, or the data arrives as a stream, a size-k heap is better. If I need average-case linear and can accept a bad worst case, quickselect." Presenting the trade-off is worth more than picking one.

### 5.5 Trapping Rain Water — the three-stage narration they love
**Brute force.** For each position, scan left and right for the tallest bar; water held is `min(maxLeft, maxRight) - height[i]`. O(n²) time, O(1) space.
**Stage two.** "Those two scans are recomputed at every index." Precompute `maxLeft[]` and `maxRight[]` in two passes. O(n) time, O(n) space.
**Stage three.** "I do not actually need both arrays stored — I only need whichever side is currently smaller, because that side determines the water level." Two pointers from both ends, moving the smaller one inward. O(n) time, **O(1) space**.

This problem is worth rehearsing precisely because it shows two separate optimisations: first remove recomputation, then remove the storage.

### 5.6 Next Greater Element — the monotonic stack
**Brute force.** For each element, scan right until something bigger. O(n²).
**The waste.** "Elements I have already passed and rejected get rescanned repeatedly."
**The move.** Keep a stack of indices whose answer is still unknown, in decreasing height order. When a taller element arrives, it is the answer for everything shorter on the stack — pop them all.
**Result.** O(n), because each index is pushed and popped exactly once. Say that amortised argument out loud; it is the whole justification.

---

# PART 6 — Templates Worth Having in Muscle Memory

```python
# Sliding window (variable size)
left = 0
for right in range(len(s)):
    add(s[right])
    while invalid():
        remove(s[left]); left += 1
    best = max(best, right - left + 1)

# Two pointers on a sorted array
lo, hi = 0, len(a) - 1
while lo < hi:
    total = a[lo] + a[hi]
    if total == target: return (lo, hi)
    if total < target:  lo += 1
    else:               hi -= 1

# Binary search on the ANSWER (not the array)
lo, hi = min_possible, max_possible
while lo < hi:
    mid = (lo + hi) // 2
    if feasible(mid): hi = mid        # mid works, try smaller
    else:             lo = mid + 1
return lo

# BFS shortest path on a grid / state graph
from collections import deque
q, seen = deque([(start, 0)]), {start}
while q:
    node, d = q.popleft()
    if node == goal: return d
    for nxt in neighbours(node):
        if nxt not in seen:
            seen.add(nxt); q.append((nxt, d + 1))

# Monotonic stack (next greater)
stack, res = [], [-1] * len(a)
for i, x in enumerate(a):
    while stack and a[stack[-1]] < x:
        res[stack.pop()] = x
    stack.append(i)

# Backtracking skeleton
def solve(path, choices):
    if done(path): record(path); return
    for c in choices:
        if not valid(c): continue
        path.append(c)
        solve(path, remaining(choices, c))
        path.pop()                     # undo — the whole point

# Memoisation
from functools import lru_cache
@lru_cache(maxsize=None)
def f(i, j): ...

# Union-Find
parent = list(range(n))
def find(x):
    while parent[x] != x:
        parent[x] = parent[parent[x]]  # path compression
        x = parent[x]
    return x
def union(a, b): parent[find(a)] = find(b)
```

---

# PART 7 — When You Cannot Find the Optimal

This happens, and how you handle it matters more than the outcome.

**Say where you are, precisely.** Vague stuckness gets no help; specific stuckness gets a hint.
> "My brute force is O(n²) because of the repeated search. n is 10⁵, so I need around O(n log n). I have ruled out hashing because the answer depends on order, and I think sorting destroys information I need. I suspect the answer involves a stack but I cannot see the invariant yet."

That paragraph demonstrates constraint analysis, elimination, and self-awareness. Many interviewers will hand you the key at that point, and using a hint well is a positive signal, not a negative one.

**Other moves that keep you productive:**
- **Solve a smaller version.** "Let me do it for a sorted array first, then generalise."
- **Solve a special case.** "If all values were distinct and positive, I would do X."
- **Work an example by hand and watch what you do.** Your own manual shortcut is usually the algorithm.
- **Say the trade-off you would accept.** "I can get O(n) time if I am allowed O(n) space — is that acceptable?"
- **Ship the brute force.** A working O(n²) with an honest "here is what I would optimise and how" beats an elegant thing that does not run.

**Never** go silent, and never bluff a complexity you have not reasoned through. "I think this is O(n log n) — let me check: the sort is n log n, the loop is n, so it is dominated by the sort" is far better than asserting a number.

---

# PART 8 — Competitive-Programming Hygiene

Small things that cost real marks.

- **Integer overflow.** In C++/Java, `int` is 32-bit — use `long long` / `long` for sums of large arrays or products. Python is arbitrary precision, so this bites only in the other languages.
- **`mid = lo + (hi - lo) // 2`**, not `(lo + hi) // 2`, in fixed-width languages.
- **Off-by-one.** Decide once whether your range is inclusive or exclusive on both ends, and stay consistent.
- **Modular arithmetic.** When asked for an answer mod 10⁹+7, take the modulus at every step, not at the end.
- **Fast I/O.** Python: `input = sys.stdin.readline`. C++: `ios_base::sync_with_stdio(false); cin.tie(NULL);`
- **Recursion depth.** Python defaults to about 1000 frames; `sys.setrecursionlimit(10**6)` or convert to an explicit stack.
- **Sorting cost is not free.** If you sort inside a loop you have added a log factor you may not have noticed.
- **Python-specific costs.** `list.pop(0)` is O(n) — use `collections.deque`. `x in list` is O(n) — use a set. String concatenation in a loop is O(n²) — build a list and `"".join()` it.
- **Watch space as well as time.** An O(n) solution that allocates an n×n table is not O(n).

---

# Two Sentences to Memorise

Opening, after you have the brute force:
> "That is correct at O(n²), so let me use it as the baseline. The expensive part is that I keep [repeating X] — and [structure Y] makes [X] cheap."

Closing, before you code:
> "So that is O(n) time and O(n) space, trading memory for time. Shall I write it?"
