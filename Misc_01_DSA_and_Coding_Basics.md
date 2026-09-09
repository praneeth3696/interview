# Miscellaneous 1 — Data Structures, Algorithms, and Coding Basics

**Why this file exists:** none of your other files cover DSA, and a technical interview with seniors almost always includes at least a few questions on complexity, arrays, strings, hashing, or "how would you solve this". It is the most likely gap between the material you have and the questions you will get. This is the basic-to-medium level you need, not competitive programming.

---

# PART 1 — Complexity Analysis

## 1.1 Big-O notation
Describes how running time or memory grows as input size n grows, ignoring constants and lower-order terms. It is about **growth rate**, not absolute speed.

- **O(1)** constant — array index, hash map lookup (average), stack push/pop.
- **O(log n)** logarithmic — binary search, balanced BST operations, heap insert. The input halves each step.
- **O(n)** linear — one pass over the data.
- **O(n log n)** — efficient comparison sorting (merge sort, heap sort, average quicksort). This is the proven lower bound for comparison-based sorting.
- **O(n²)** quadratic — nested loop over the same data, bubble/selection/insertion sort.
- **O(2ⁿ)** exponential — naive recursive subset generation, naive Fibonacci.
- **O(n!)** factorial — brute-force permutations, the travelling salesman by enumeration.

**Big-O (upper bound), Big-Ω (lower bound), Big-Θ (tight bound).** In practice people say Big-O when they mean Θ.

**Best / average / worst case** are different from Big-O/Ω/Θ — they describe *which input*, not which bound. Quicksort is O(n log n) average and O(n²) worst case (already-sorted input with a bad pivot).

**Amortised complexity** — the average cost per operation over a sequence. A dynamic array's `push_back` is O(n) when it resizes but O(1) amortised, because doubling means resizes become geometrically rarer.

**Space complexity** — extra memory used beyond the input. Recursion counts: the call stack is O(depth).

**How to analyse a loop**: a single loop over n is O(n); nested loops over n are O(n²); a loop that halves the range each step is O(log n); a loop over n containing a halving loop is O(n log n). Two sequential loops are O(n + n) = O(n), not O(n²).

## 1.2 Common data structure complexities
| Structure | Access | Search | Insert | Delete | Space |
|---|---|---|---|---|---|
| Array | O(1) | O(n) | O(n) | O(n) | O(n) |
| Sorted array | O(1) | O(log n) | O(n) | O(n) | O(n) |
| Dynamic array | O(1) | O(n) | O(1) amortised at end | O(n) | O(n) |
| Singly linked list | O(n) | O(n) | O(1) at head | O(1) given the node | O(n) |
| Stack / Queue | O(n) | O(n) | O(1) | O(1) | O(n) |
| Hash table | — | O(1) avg, O(n) worst | O(1) avg | O(1) avg | O(n) |
| Balanced BST | O(log n) | O(log n) | O(log n) | O(log n) | O(n) |
| Binary heap | O(1) for min/max | O(n) | O(log n) | O(log n) | O(n) |
| Trie | O(m) | O(m) | O(m) | O(m) | O(alphabet × nodes) |
(m = length of the key)

---

# PART 2 — Data Structures

## 2.1 Array
Contiguous memory, fixed or dynamic size, O(1) access by index because the address is `base + i × sizeof(element)`.
**Strengths**: cache-friendly because elements are adjacent, so traversal is fast in practice even beyond what Big-O suggests.
**Weaknesses**: insertion and deletion in the middle require shifting; fixed-size arrays need resizing.
**Dynamic array** (`vector`, `ArrayList`, Python `list`) grows by allocating a larger block (usually double) and copying — O(n) on resize, O(1) amortised.

## 2.2 Linked list
Nodes holding data plus a pointer to the next node.
- **Singly** — one pointer forward. **Doubly** — forward and backward, so deletion given a node is O(1) without a previous pointer. **Circular** — the last points to the first.
- **Versus array**: no contiguous memory needed, insertion/deletion at a known position is O(1), but access is O(n), there is per-node pointer overhead, and traversal is cache-hostile.
- **Classic problems**: reverse a list (iteratively, tracking prev/curr/next), detect a cycle (**Floyd's tortoise and hare** — a slow pointer moving one step and a fast one moving two; if they meet there is a cycle), find the middle (same two-pointer trick — when fast reaches the end, slow is at the middle), merge two sorted lists, remove the nth node from the end (two pointers n apart).

## 2.3 Stack — LIFO
Operations: `push`, `pop`, `peek`, `isEmpty`, all O(1).
**Uses**: function call stack, undo, expression evaluation and bracket matching, backtracking, iterative DFS, converting infix to postfix.
**The classic question**: check whether brackets are balanced — push opening brackets, and on a closing bracket pop and check it matches. Empty at the end means balanced.

## 2.4 Queue — FIFO
Operations: `enqueue`, `dequeue`, `front`, all O(1).
**Variants**: circular queue (reuses freed space in a fixed array), deque (insert and remove at both ends), priority queue (highest priority first, usually a heap).
**Uses**: BFS, scheduling, buffering, producer-consumer.

## 2.5 Hash table
Maps keys to values using a hash function that converts the key into a bucket index. O(1) average lookup, insert, and delete.
- **Collision** — two keys hash to the same bucket. Resolved by **chaining** (a linked list or tree per bucket; Java converts to a tree above 8 entries so the worst case is O(log n) not O(n)) or **open addressing** (probe for the next free slot — linear, quadratic, or double hashing).
- **Load factor** = entries / buckets. Above a threshold (0.75 typically) the table **rehashes** into a larger array, which is O(n) but amortises away.
- **A good hash function** distributes uniformly, is fast, and is deterministic.
- **Why keys must be immutable**: mutating a key after insertion changes its hash, so it lands in the wrong bucket and becomes unfindable. This is why Python only allows hashable (immutable) keys, and why Java's `equals`/`hashCode` contract exists.
- **Uses**: dictionaries, caches, deduplication, counting frequencies, two-sum-style problems, database indexes.

## 2.6 Tree
A hierarchical structure of nodes with one root and no cycles.
- **Terms**: root, leaf, parent, child, sibling, depth (distance from root), height (longest path to a leaf), subtree, degree.
- **Binary tree** — at most two children. **Full** (every node has 0 or 2 children), **complete** (every level filled except possibly the last, filled left to right — this is how a heap is stored in an array), **perfect** (all leaves at the same depth), **balanced** (height O(log n)).
- **Binary Search Tree (BST)** — left subtree < node < right subtree. Search, insert, and delete are O(h) where h is height: O(log n) if balanced, O(n) if degenerate (inserting sorted data makes it a linked list).
- **Self-balancing trees**: AVL (strictly balanced, faster lookup), Red-Black (looser, faster insertion — used in most standard libraries), B/B+ trees (high branching factor, few levels, so few disk reads — this is why databases use them for indexes).
- **Traversals**:
  - **Inorder** (left, node, right) — on a BST this yields sorted order.
  - **Preorder** (node, left, right) — used to copy a tree or produce a prefix expression.
  - **Postorder** (left, right, node) — used to delete a tree or produce a postfix expression.
  - **Level order** (BFS) — uses a queue, visits level by level.
- **Heap** — a complete binary tree where every parent is ≤ (min-heap) or ≥ (max-heap) its children. Stored in an array: for index i, children are at 2i+1 and 2i+2, parent at (i−1)/2. Insert and extract are O(log n), peek is O(1). Uses: priority queues, heap sort, "top k" problems, Dijkstra's algorithm.
- **Trie (prefix tree)** — each edge is a character, so lookup is O(length of key) regardless of how many keys are stored. Uses: autocomplete, spell check, IP routing tables, dictionary storage.

## 2.7 Graph
A set of vertices and edges. **Directed or undirected**, **weighted or unweighted**, **cyclic or acyclic**, **connected or disconnected**.
- **Representations**: **adjacency matrix** (V×V, O(1) edge lookup, O(V²) space — good for dense graphs) versus **adjacency list** (a list of neighbours per vertex, O(V+E) space, O(degree) edge lookup — good for sparse graphs, which is most real graphs).
- **BFS** — queue, explores level by level, finds the **shortest path in an unweighted graph**, O(V+E).
- **DFS** — stack or recursion, goes deep first, used for cycle detection, topological sort, connected components, and path finding, O(V+E).
- **Dijkstra's algorithm** — shortest path with non-negative weights, greedy with a priority queue, O((V+E) log V). Fails with negative edges.
- **Bellman-Ford** — handles negative weights and detects negative cycles, O(VE). This is the basis of distance-vector routing.
- **Topological sort** — a linear ordering of a DAG such that every edge points forward. Used for build dependencies, course prerequisites, and task scheduling.
- **Minimum spanning tree** — Kruskal (sort edges, union-find) and Prim (grow from a vertex with a priority queue).
- **Real uses**: social networks, routing (OSPF uses Dijkstra), dependency resolution, web crawling.

---

# PART 3 — Algorithms

## 3.1 Searching
- **Linear search** — O(n), works on unsorted data.
- **Binary search** — O(log n), requires **sorted** data. Repeatedly halve the range.
```python
def binary_search(arr, target):
    lo, hi = 0, len(arr) - 1
    while lo <= hi:
        mid = lo + (hi - lo) // 2      # avoids overflow in languages with fixed ints
        if arr[mid] == target: return mid
        if arr[mid] < target: lo = mid + 1
        else: hi = mid - 1
    return -1
```
The classic bugs: `mid = (lo + hi) / 2` overflows, `while lo < hi` misses the last element, and forgetting `mid + 1`/`mid - 1` causes an infinite loop.

## 3.2 Sorting
| Algorithm | Best | Average | Worst | Space | Stable? |
|---|---|---|---|---|---|
| Bubble | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Selection | O(n²) | O(n²) | O(n²) | O(1) | No |
| Insertion | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Merge | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Quick | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Heap | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Counting | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes |

- **Stable** means equal elements keep their relative order — which matters when sorting by one key after another.
- **Merge sort** — divide the array in half, sort each recursively, merge. Guaranteed O(n log n), stable, but needs O(n) extra space. The choice when worst-case guarantees matter or when sorting linked lists.
- **Quicksort** — pick a pivot, partition into less-than and greater-than, recurse. Fastest in practice because of good cache behaviour and low constants, in-place, but O(n²) if the pivot is consistently bad (already-sorted input with a first-element pivot). Fixed with a random or median-of-three pivot.
- **Why is O(n log n) the lower bound for comparison sorting?** There are n! possible orderings, and each comparison gives one bit of information, so you need at least log₂(n!) ≈ n log n comparisons. Counting and radix sort beat it only because they do not compare — they use the values as indices.

## 3.3 Recursion
A function calling itself on a smaller input. Needs a **base case** (or it overflows the stack) and a **recursive case that makes progress toward it**.
- **Call stack** — each call gets a frame with its own locals; depth d costs O(d) space.
- **Tail recursion** — the recursive call is the last operation, so some compilers optimise it into a loop. Python does not.
- **When to use it**: tree and graph traversal, divide and conquer, backtracking — anywhere the problem is naturally self-similar. Iteration is usually better when the recursion is linear (like a loop).

## 3.4 The four algorithmic paradigms
- **Brute force** — try everything. Always correct, usually too slow, but a valid starting point in an interview.
- **Divide and conquer** — split into subproblems, solve independently, combine. Merge sort, quicksort, binary search.
- **Greedy** — take the locally best choice at each step. Fast, but only correct when the problem has the greedy-choice property and optimal substructure. Works: Dijkstra, Huffman coding, activity selection, fractional knapsack. Fails: 0/1 knapsack, coin change with arbitrary denominations.
- **Dynamic programming** — break into overlapping subproblems, solve each once, and store the result. Requires **optimal substructure** and **overlapping subproblems**. Two forms: **memoisation** (top-down recursion with a cache) and **tabulation** (bottom-up table filling). Classic problems: Fibonacci, 0/1 knapsack, longest common subsequence, edit distance, coin change, longest increasing subsequence.
- **Backtracking** — build a solution incrementally and abandon a partial solution as soon as it cannot be completed. N-queens, Sudoku, permutations, subsets.

## 3.5 Patterns worth recognising
- **Two pointers** — one from each end (pair sum in a sorted array), or both forward at different speeds (remove duplicates in place, cycle detection).
- **Sliding window** — a moving range over an array or string, for "longest/shortest subarray with property X". Turns an O(n²) nested loop into O(n).
- **Hash map for counting or lookup** — the single most useful trick. Two-sum, anagram check, first non-repeating character, frequency problems.
- **Prefix sums** — precompute cumulative sums so any range sum is O(1).
- **Fast and slow pointers** — cycle detection, finding the middle.
- **Sort first** — many problems become easy once sorted, and O(n log n) is often acceptable.

---

# PART 4 — Problems You Should Be Able to Solve or Explain

These are the level actually asked in a first internship interview. Be able to explain the approach and the complexity even if you do not write perfect code.

**Arrays and strings**
1. Reverse an array or string in place (two pointers).
2. Find the maximum and minimum in one pass.
3. Find the second largest element.
4. Check if a string is a palindrome (two pointers, skipping non-alphanumerics).
5. Check if two strings are anagrams (sort both, or count characters in a hash map).
6. Find the first non-repeating character (count frequencies, then scan in order).
7. Remove duplicates from a sorted array in place (two pointers).
8. Two Sum — find two numbers adding to a target. Brute force is O(n²); with a hash map storing "value → index" as you scan, it is O(n).
9. Find the missing number in 1..n (sum formula n(n+1)/2 minus the actual sum, or XOR).
10. Rotate an array by k (reverse the whole array, then reverse the two parts).
11. Maximum subarray sum (**Kadane's algorithm** — at each position, either extend the previous subarray or start fresh; O(n)).
12. Move all zeroes to the end in place.
13. Merge two sorted arrays.
14. Find the majority element (Boyer-Moore voting, O(n) time O(1) space).

**Linked lists**
15. Reverse a linked list.
16. Detect a cycle (Floyd's).
17. Find the middle node.
18. Merge two sorted lists.
19. Remove the nth node from the end.

**Stacks and queues**
20. Balanced brackets.
21. Implement a queue using two stacks.
22. Next greater element (monotonic stack).
23. Min stack — a stack that also returns its minimum in O(1) (push the running minimum alongside each element).

**Trees**
24. All four traversals, recursive and iterative.
25. Height of a binary tree.
26. Check if a tree is a valid BST (inorder must be sorted, or pass down min/max bounds).
27. Lowest common ancestor.
28. Mirror or invert a binary tree.
29. Level order traversal.

**Recursion and maths**
30. Factorial and Fibonacci, recursively and iteratively, and why the naive recursive Fibonacci is O(2ⁿ) while memoised it is O(n).
31. Check if a number is prime (trial division up to √n).
32. GCD (Euclid's algorithm).
33. Power of a number in O(log n) (fast exponentiation by squaring).
34. Generate all subsets or permutations (backtracking).
35. Tower of Hanoi.

**Sorting and searching**
36. Implement binary search.
37. Implement bubble, selection, and insertion sort.
38. Explain merge sort and quicksort with their complexities and when you would pick each.

---

# PART 5 — How to Handle a Coding Question in an Interview

The process matters as much as the answer. **If the round is specifically "give me the brute force, then optimise it", read `Misc_03_DSA_Bruteforce_to_Optimal.md` — it expands step 4 below, which is where that round is actually won.**

1. **Restate the problem** and confirm you understood it. "So given an unsorted array of integers, return the indices of two that sum to the target — and can I assume exactly one solution exists?"
2. **Ask about constraints and edge cases before coding.** How large is n? Can the array be empty? Negative numbers? Duplicates? Is it sorted? Can I modify the input? These questions are themselves being assessed.
3. **State a brute-force solution first, with its complexity.** "The obvious approach is a nested loop, O(n²). Let me see if I can do better." This guarantees you have *something* and shows you understand the baseline.
4. **Think out loud toward an improvement.** Silence reads as being stuck. "The repeated work here is searching for the complement, and searching is what a hash map makes O(1)..."
5. **State the approach before writing code.** Get agreement first — it is much cheaper to fix a plan than an implementation.
6. **Write the code**, naming variables meaningfully.
7. **Trace through a small example by hand**, out loud. This catches most bugs.
8. **Check the edge cases** you identified: empty input, one element, all identical, the target at the boundaries.
9. **State the final time and space complexity** without being asked.
10. **Mention what you would improve** with more time.

**If you get stuck**: say so and say what you are stuck on — "I know I want O(n) but I cannot see what to store." Interviewers give hints to people who show them where they are. They cannot help silence.

---

# PART 6 — Rapid-Fire

- **Array vs linked list?** Contiguous with O(1) access versus scattered with O(1) insertion. Arrays win on cache behaviour; lists win when you insert and delete a lot at known positions.
- **Stack vs queue?** LIFO versus FIFO.
- **Why is a hash map O(1)?** The hash function computes the location directly rather than searching. It degrades to O(n) if every key collides.
- **When is O(n²) acceptable?** When n is small and bounded, or when the constant factor of the O(n log n) alternative is worse — insertion sort beats quicksort for very small arrays, which is why real implementations switch to it below a threshold.
- **What is a stable sort and why care?** Equal elements keep their order, which matters when you sort by a second key after a first.
- **Recursion vs iteration?** Recursion is clearer for self-similar problems but costs stack space and function-call overhead. Any recursion can be rewritten iteratively with an explicit stack.
- **What is memoisation?** Caching a function's result by its arguments so repeated calls are free. It is the top-down half of dynamic programming.
- **What is the difference between DFS and BFS?** DFS goes deep using a stack or recursion and uses less memory on wide graphs; BFS explores level by level using a queue and finds the shortest unweighted path.
- **What data structure would you use for a cache with eviction?** A hash map for O(1) lookup plus a doubly linked list for O(1) reordering — that is exactly how an LRU cache is built.
- **What data structure is a database index?** A B+ tree — high branching factor means few levels, so few disk reads, and linked leaves make range scans efficient.
- **What data structure powers autocomplete?** A trie.
- **What data structure powers a priority queue?** A binary heap.
- **What data structure powers undo?** A stack.
