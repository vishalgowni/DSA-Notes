# Appendix 01: Master Big-O Time & Space Complexity Cheat Sheet

> ** Navigation**
> - **Parent Directory**: [DSA Notes Root](../README.md)
> - **Next Appendix**: [Appendix 02: C Programming Patterns & Idioms](./02-C-Programming-Patterns.md)
> - **Related Appendices**: [Appendix 03: Common Errors & Edge Cases](./03-Common-Errors-and-Edge-Cases.md) | [Appendix 04: Practice Questions & Answers](./04-Practice-Questions-and-Answers.md)

---

## 1. Variable Notation Legend

Throughout all complexity tables in this reference, asymptotic bounds are expressed in terms of the specific structural parameters below. Always specify which parameter a bound depends on during exams!

| Symbol | Formal Definition | Typical Contexts |
| :--- | :--- | :--- |
| $n$ | Number of elements, keys, or nodes currently stored in the data structure | Arrays, Linked Lists, Stacks, Queues, Trees, Heaps, Hash Tables |
| $V$ (or $|V|$) | Number of vertices (nodes) in a graph $G = (V, E)$ | Graph Representations, BFS, DFS, Connected Components |
| $E$ (or $|E|$) | Number of edges (links) in a graph ($0 \le E \le V(V-1)$ directed, $\le \binom{V}{2}$ undirected) | Adjacency Lists, Graph Traversals, Cycle Detection |
| $h$ | Height of a tree (longest root-to-leaf path in edges; $\lfloor \log_2 n \rfloor \le h \le n-1$ for binary trees) | BSTs, AVL Trees, Heaps, Recursive Tree Traversals |
| $m$ | (1) Capacity (number of buckets/slots) of a Hash Table, OR<br>(2) Length of a text/pattern string $|S|$ | (1) Hashing & Open Addressing<br>(2) Tries, Suffix Tries, Suffix Trees |
| $k$ | (1) Step size / skip count (e.g., Josephus problem), OR<br>(2) Branching factor / degree of a vertex $\text{deg}(u)$, OR<br>(3) Number of output matches / components | (1) Josephus Queue Simulation<br>(2) N-ary Trees, Graph Neighbor Iteration<br>(3) Range / Pattern Queries |
| $|\Sigma|$ | Size of the character alphabet (e.g., $26$ for lowercase English, $128$ for ASCII, $256$ for extended byte) | Standard Tries, Compressed Tries, Suffix Trees |
| $\alpha$ | Load factor of a hash table, defined as $\alpha = \frac{n}{m}$ | Separate Chaining ($\alpha$ can exceed $1$), Open Addressing ($0 \le \alpha < 1$) |
| $R, C$ | Number of rows ($R$) and columns ($C$) in a matrix | Dense 2D Matrices, Sparse Matrix Triplets & Multilists |
| $nnz$ (or $t$) | Number of non-zero elements in a sparse matrix or non-zero terms in a polynomial | Sparse Matrices ($nnz \ll R \cdot C$), Polynomial Linked Lists |
| $d$ | Maximum degree (highest exponent) of a polynomial | Dense Array Polynomial Representation |

---

## 2. Asymptotic Growth Hierarchy & Master Mathematical Sums

### 2.1 Strict Asymptotic Growth Ordering

For sufficiently large $n$ ($n \to \infty$), standard complexity classes obey the strict ordering:

$$O(1) \subset O(\log \log n) \subset O(\log n) \subset O(\sqrt{n}) \subset O(n) \subset O(n \log n) \subset O(n^2) \subset O(n^3) \subset O(2^n) \subset O(n!) \subset O(n^n)$$

```
Operations
   ^
   |      O(n!)   O(2^n)     O(n^2)        O(n log n)
   |        |       /          /             /
   |        |      /          /             /       O(n)
   |        |     /          /             /       /
   |        |    /          /             /       /       O(sqrt(n))
   |        |   /          /             /       /       /
   |        |  /          /             /       /   .--'
   |        | /          /          .--'    .--' .-'     O(log n)
   |        |/       .--'       .--'    .--' .--'    .--'
   |       /|    .--'       .--'    .--' .--'    .--'
   |      / |.--'       .--'    .--' .--'    .--'        O(1)
   +-----/--|--------.--'----.--'-.--'----.--'------------------> Input Size (n)
```

### 2.2 Concrete Operation Counts vs. Input Size $n$

Assuming a modern processor executes roughly $10^8$ basic operations per second:

| Complexity Class | $n = 10$ | $n = 10^2$ | $n = 10^3$ | $n = 10^6$ | $n = 10^9$ | Typical DSA Examples |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| $O(1)$ | $1$ | $1$ | $1$ | $1$ | $1$ | Array index access, Stack `push`/`pop`, Hash lookup (avg) |
| $O(\log_2 n)$ | $\approx 3$ | $\approx 7$ | $\approx 10$ | $\approx 20$ | $\approx 30$ | Binary search, AVL search/insert/delete, Heap `insert` |
| $O(\sqrt{n})$ | $\approx 3$ | $10$ | $\approx 32$ | $10^3$ | $\approx 3.16 \times 10^4$ | Jump search on sorted array, Primality trial division |
| $O(n)$ | $10$ | $10^2$ | $10^3$ | $10^6$ ($\approx 10\text{ ms}$) | $10^9$ ($\approx 10\text{ s}$) | Unsorted list search, Tree traversal, Floyd's `build_heap`, Ukkonen's Suffix Tree |
| $O(n \log_2 n)$ | $\approx 33$ | $\approx 664$ | $\approx 10^4$ | $\approx 2 \times 10^7$ ($\approx 0.2\text{ s}$) | $\approx 3 \times 10^{10}$ ($\approx 5\text{ min}$) | Heap Sort, Merge Sort, Building AVL tree of $n$ keys |
| $O(n^2)$ | $10^2$ | $10^4$ | $10^6$ ($\approx 10\text{ ms}$) | $10^{12}$ ($\approx 2.8\text{ hr}$) | $10^{18}$ ($\approx 317\text{ yr}$) | Skewed BST construction, Adjacency Matrix BFS/DFS, Naive Suffix Trie build |
| $O(2^n)$ | $1,024$ | $1.26 \times 10^{30}$ | $\infty$ | $\infty$ | $\infty$ | Subset generation, Unmemoized recursive Fibonacci / Tower of Hanoi |
| $O(n!)$ | $3.63 \times 10^6$ | $9.33 \times 10^{157}$ | $\infty$ | $\infty$ | $\infty$ | Brute-force permutation enumeration (TSP) |

### 2.3 Master Mathematical Sums Used in DSA Complexity Proofs

Every major complexity proof in Data Structures reduces to one of these seven closed-form identities:

| Name of Series | Summation Formula | Exact Closed Form / Asymptotic Tight Bound | Where It Appears in DSA |
| :--- | :--- | :--- | :--- |
| **1. Arithmetic Series** | $\displaystyle\sum_{i=1}^{n} i = 1 + 2 + 3 + \dots + n$ | $\displaystyle\frac{n(n+1)}{2} = \Theta(n^2)$ | Inserting $n$ sorted keys into an unbalanced BST; Naive array shifting in linear queue; Selection/Insertion sort |
| **2. Sum of Squares** | $\displaystyle\sum_{i=1}^{n} i^2 = 1^2 + 2^2 + \dots + n^2$ | $\displaystyle\frac{n(n+1)(2n+1)}{6} = \Theta(n^3)$ | Nested triangular matrix loops; Dense matrix elimination |
| **3. Finite Geometric Series** | $\displaystyle\sum_{i=0}^{h} 2^i = 1 + 2 + 4 + \dots + 2^h$ | $2^{h+1} - 1 = \Theta(2^h)$ | Maximum number of nodes in a binary tree of height $h$; Dynamic array doubling total copy cost ($1 + 2 + 4 + \dots + n < 2n = O(n)$) |
| **4. General Geometric Series** | $\displaystyle\sum_{i=0}^{h} k^i = 1 + k + k^2 + \dots + k^h$ ($k > 1$) | $\displaystyle\frac{k^{h+1} - 1}{k - 1} = \Theta(k^h)$ | Maximum nodes in a complete $k$-ary tree of height $h$ |
| **5. Harmonic Series** | $\displaystyle H_n = \sum_{i=1}^{n} \frac{1}{i} = 1 + \frac{1}{2} + \frac{1}{3} + \dots + \frac{1}{n}$ | $\ln n + \gamma + O\left(\frac{1}{n}\right) = \Theta(\log n)$<br>($\gamma \approx 0.5772$) | Expected probes in Double Hashing successful search; Average height of randomly built BST |
| **6. Arithmetico-Geometric Series (Floyd's Heap Sum)** | $\displaystyle\sum_{i=0}^{h} \frac{i}{2^i} < \sum_{i=1}^{\infty} \frac{i}{2^i}$ | $\displaystyle\sum_{i=1}^{\infty} \frac{i}{2^i} = 2 \implies O(1)$ | Proving bottom-up `build_heap` runs in $\displaystyle\sum_{i=0}^{\lfloor\log_2 n\rfloor} \left\lceil \frac{n}{2^{i+1}} \right\rceil O(i) = O\left(n \sum \frac{i}{2^i}\right) = O(n)$ linear time! |
| **7. Log-Factorial Series (Stirling's Approximation)** | $\displaystyle\sum_{i=1}^{n} \log_2 i = \log_2(n!)$ | $n \log_2 n - n \log_2 e + \Theta(\log n) = \Theta(n \log n)$ | Top-down heap construction $\sum_{i=1}^n O(\log i) = O(n \log n)$; Comparison-based sorting lower bound |

---

## 3. Unit 1 Complexity Tables: Linear Data Structures, Matrices, Polynomials & Stacks

### 3.1 Arrays vs. Linked List Variants

Assume $n$ is the current number of elements. For operations given a direct pointer `p` to the target node, search time to locate `p` is excluded; if only a key or index is given, add $O(n)$ search time for linked lists.

| Operation | Static Array (`arr[CAP]`) | Dynamic Array (`realloc` $\times 2$) | Singly Linked List (Head Only) | Singly Linked List (Head + Tail) | Doubly Linked List (Head + Tail) | Circular SLL (Head Only) | Circular SLL (Tail Pointer Only)* |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Access $i$-th element (`get(i)`)** | $O(1)$ | $O(1)$ | $O(n)$ | $O(n)$ | $O(n)$ (or $O(\min(i, n-i))$) | $O(n)$ | $O(n)$ |
| **Search (Unsorted)** | $O(n)$ | $O(n)$ | $O(n)$ | $O(n)$ | $O(n)$ | $O(n)$ | $O(n)$ |
| **Search (Sorted)** | $O(\log n)$ (Binary Search) | $O(\log n)$ (Binary Search) | $O(n)$ (No random access!) | $O(n)$ | $O(n)$ | $O(n)$ | $O(n)$ |
| **Insert at Front (`push_front`)** | $O(n)$ (shift right) | $O(n)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(n)$ (must update tail$\to$next) | $O(1)$ (`tail->next` is head!) |
| **Insert at Back (`push_back`)** | $O(1)$ (if space) | $O(1)$ amortized, $O(n)$ worst | $O(n)$ | $O(1)$ | $O(1)$ | $O(n)$ | $O(1)$ (insert after `tail`, advance `tail`) |
| **Insert After Node `p`** | $O(n)$ | $O(n)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **Insert Before Node `p`** | $O(n)$ | $O(n)$ | $O(n)$ (or $O(1)$ data-swap trick) | $O(n)$ | $O(1)$ (uses `p->prev`) | $O(n)$ | $O(n)$ |
| **Delete at Front (`pop_front`)** | $O(n)$ (shift left) | $O(n)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(n)$ (must find tail) | $O(1)$ (`tail->next = head->next`) |
| **Delete at Back (`pop_back`)** | $O(1)$ | $O(1)$ | $O(n)$ | $O(n)$ (no `prev` to tail!) | $O(1)$ (uses `tail->prev`) | $O(n)$ | $O(n)$ (must find predecessor of `tail`) |
| **Delete Given Node `p`** | $O(n)$ | $O(n)$ | $O(n)$ (unless `p` $\ne$ tail & data-swap $O(1)$) | $O(n)$ | $O(1)$ | $O(n)$ | $O(n)$ |
| **Reverse Structure In-Place** | $O(n)$ time, $O(1)$ space | $O(n)$ time, $O(1)$ space | $O(n)$ time, $O(1)$ space | $O(n)$ time, $O(1)$ space | $O(n)$ time, $O(1)$ space | $O(n)$ time, $O(1)$ space | $O(n)$ time, $O(1)$ space |
| **Concatenate Two Lists ($n_1, n_2$)** | $O(n_2)$ or $O(n_1+n_2)$ | $O(n_2)$ amortized | $O(n_1)$ | $O(1)$ | $O(1)$ | $O(n_1 + n_2)$ | $O(1)$ (swap `tail1->next` & `tail2->next`) |
| **Pointer Overhead / Element** | $0$ bytes | $0$ bytes (plus unused capacity) | $1$ pointer ($8$ B on 64-bit) | $1$ pointer ($8$ B) | $2$ pointers ($16$ B) | $1$ pointer ($8$ B) | $1$ pointer ($8$ B) |

*\*Note on Circular SLL with a `tail` pointer (`last` pointer): Because `tail->next` points directly to `head`, both front insertion/deletion and back insertion run in $O(1)$ time with only 1 pointer per node! However, deleting the back node still requires $O(n)$ to find the node preceding `tail`.*

---

### 3.2 Skip List & Sparse Matrix Representations

#### Skip List (Probabilistic Multi-Level Linked List, $n$ elements, coin probability $p = 1/2$)

| Operation | Average / With High Probability (w.h.p.) | Worst-Case (Pathological Coin Flips) | Notes |
| :--- | :--- | :--- | :--- |
| **Search (`search(key)`)** | $O(\log n)$ | $O(n)$ | Expected $\le 2 \log_2 n$ comparisons for $p = 1/2$ |
| **Insert (`insert(key)`)** | $O(\log n)$ | $O(n)$ | Updates predecessors in `update[0..lvl]` array |
| **Delete (`delete(key)`)** | $O(\log n)$ | $O(n)$ | Unlinks node at all levels where it appears |
| **Find Min / Max** | $O(1)$ (Min at `header->forward[0]`) / $O(\log n)$ (Max) | $O(1)$ / $O(n)$ | Min is immediately after sentinel header at level 0 |
| **Auxiliary Space** | $O(n)$ expected ($\frac{1}{1-p} n = 2n$ pointers for $p=1/2$) | $O(n \log n)$ (if max level capped at $\lceil \log_2 n \rceil$) | Average number of levels per node is $\sum_{i=0}^\infty (1/2)^i = 2$ |

#### Sparse Matrix Representations ($R$ rows, $C$ columns, $nnz$ non-zero elements, $nnz \ll R \cdot C$)

| Operation | Dense 2D Array (`A[R][C]`) | Sorted Triplet Array (`3 x (nnz+1)` / COO) | Row-Major CSR / Array of Row Lists | Orthogonal Cross-Linked Multilist |
| :--- | :--- | :--- | :--- | :--- |
| **Total Space** | $\Theta(R \cdot C)$ | $\Theta(nnz)$ ($3(nnz+1)$ ints) | $\Theta(R + nnz)$ | $\Theta(R + C + nnz)$ (5 fields/node) |
| **Access Element $(r, c)$** | $O(1)$ | $O(\log nnz)$ (binary search) | $O(\text{nnz in row } r)$ | $O(\min(\text{nnz in row } r, \text{nnz in col } c))$ |
| **Traverse Row $r$ / Col $c$** | $O(C)$ / $O(R)$ | $O(\log nnz + k)$ row / $O(nnz)$ col | $O(\text{nnz in row } r)$ / $O(nnz)$ col | $O(\text{nnz in row } r)$ / $O(\text{nnz in col } c)$ |
| **Insert / Delete Non-Zero** | $O(1)$ | $O(nnz)$ (array shift) | $O(\text{nnz in row } r)$ | $O(\text{nnz in row } r + \text{nnz in col } c)$ |
| **Add Two Matrices ($nnz_1, nnz_2$)** | $O(R \cdot C)$ | $O(nnz_1 + nnz_2)$ (2-pointer merge) | $O(R + nnz_1 + nnz_2)$ | $O(R + C + nnz_1 + nnz_2)$ |
| **Matrix Transpose ($A^T$)** | $O(R \cdot C)$ | **Naive**: $O(C \cdot nnz)$<br>**Fast Transpose**: $O(C + nnz)$ | $O(R + C + nnz)$ | $O(R + C + nnz)$ |

---

### 3.3 Polynomial Representations & Operations

Let $d_1, d_2$ be the maximum degrees and $t_1, t_2$ be the number of non-zero terms ($t_i \le d_i + 1$).

| Operation | Dense Coefficient Array (`coeff[0..d]`) | Sparse Sorted Term Array (`Term[0..t-1]`) | Sorted Singly Linked List ($t$ nodes) |
| :--- | :--- | :--- | :--- |
| **Space Complexity** | $\Theta(d)$ (wasteful if sparse, e.g., $x^{1000}+1$) | $\Theta(t)$ | $\Theta(t)$ |
| **Addition / Subtraction ($P_1 \pm P_2$)** | $O(\max(d_1, d_2))$ | $O(t_1 + t_2)$ (Two-pointer merge) | $O(t_1 + t_2)$ (Two-pointer merge with dummy tail) |
| **Multiplication ($P_1 \times P_2$)** | $O(d_1 \cdot d_2)$ | $O(t_1 \cdot t_2 \log(t_1 t_2))$ or $O(t_1^2 t_2)$ | $O(t_1 \cdot t_2 \cdot \min(t_1, t_2))$ (or $O(t_1 t_2 \log(t_1 t_2))$ if merge-sorted) |
| **Evaluation at $x = x_0$** | **Naive**: $O(d^2)$<br>**Horner's Rule**: $O(d)$ ($d$ mults + $d$ adds) | $O(t \log d)$ (using fast exponentiation $x^e$) | $O(t \log d)$ (or $O(d)$ via gap exponentiation) |
| **Insert Single Term $c \cdot x^e$** | $O(1)$ (if $e \le d$) | $O(t)$ (shift terms) | $O(t)$ (find sorted position) |

---

### 3.4 Stack Implementations & Expression Applications

#### Stack Implementations ($n$ elements currently on stack)

| Operation | Fixed-Size Array Stack | Dynamic Doubling Array Stack | Singly Linked List Stack (Push/Pop at Head) |
| :--- | :--- | :--- | :--- |
| **`push(val)`** | $O(1)$ (fails if `top == CAP - 1`) | $O(1)$ amortized, $O(n)$ worst-case resize | $O(1)$ worst-case (malloc node, link before head) |
| **`pop()`** | $O(1)$ | $O(1)$ (or $O(1)$ amortized with quarter-halving) | $O(1)$ worst-case (save head, advance head, free) |
| **`peek()` / `top()`** | $O(1)$ | $O(1)$ | $O(1)$ |
| **`isEmpty()` / `isFull()`** | $O(1)$ / $O(1)$ | $O(1)$ / $O(1)$ | $O(1)$ / $O(1)$ (only full on heap exhaustion) |
| **Total Space** | $\Theta(\text{CAP})$ static | $\Theta(n)$ dynamic (up to $2n$ capacity) | $\Theta(n)$ dynamic ($1$ pointer overhead per node) |

#### Stack Applications (Expression length $L$ tokens/characters)

| Application / Algorithm | Time Complexity | Auxiliary Space Complexity | Key Invariant / Reason |
| :--- | :--- | :--- | :--- |
| **Balanced Parentheses / Brackets Check** | $O(L)$ | $O(L)$ worst-case (e.g., `((((((`) | Each character is pushed at most once and popped at most once |
| **Infix to Postfix (Dijkstra's Shunting-Yard)** | $O(L)$ | $O(L)$ (operator stack) | Every token enters and leaves the operator stack at most once ($2L$ stack ops max) |
| **Infix to Prefix (Reverse $\to$ Postfix $\to$ Reverse)** | $O(L)$ | $O(L)$ (stack + reversed buffer) | Three linear $O(L)$ passes: reverse string & swap `(` `)`, run Shunting-Yard, reverse output |
| **Postfix Expression Evaluation** | $O(L)$ | $O(L)$ (operand stack) | Single left-to-right scan; each number pushed once, each binary operator pops 2 and pushes 1 |
| **Prefix Expression Evaluation** | $O(L)$ | $O(L)$ (operand stack) | Single right-to-left scan (or recursion); pop order is `op1 = pop(); op2 = pop()` |

---

## 4. Unit 2 Complexity Tables: Queues, Priority Queues, Applications & Trees

### 4.1 Queue & Deque Implementations

| Operation | Simple Linear Array Queue (No Shift) | Simple Linear Array Queue (Shift on Dequeue) | Circular Array Queue (`% CAP`) | Singly Linked List Queue (`front` & `rear`) | Deque (Circular Array) | Deque (Doubly Linked List) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **`enqueue` / `insertRear`** | $O(1)$ (suffers **false overflow**!) | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **`dequeue` / `deleteFront`** | $O(1)$ | $O(n)$ (shifts $n-1$ items left!) | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **`insertFront`** | N/A | N/A | N/A | $O(1)$ | $O(1)$ (`(front - 1 + CAP) % CAP`) | $O(1)$ |
| **`deleteRear`** | N/A | N/A | N/A | $O(n)$ (no `prev` pointer!) | $O(1)$ (`(rear - 1 + CAP) % CAP`) | $O(1)$ (uses `rear->prev`) |
| **`peekFront` / `peekRear`** | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **Space Utilization** | Poor (dead slots before `front`) | Full $\Theta(\text{CAP})$ | Full $\Theta(\text{CAP})$ | Exact $\Theta(n)$ + pointers | Full $\Theta(\text{CAP})$ | Exact $\Theta(n)$ + 2 ptrs/node |

---

### 4.2 Priority Queue Implementations ($n$ elements)

Assuming a **Min-Priority Queue** (symmetric bounds hold for Max-Priority Queue):

| Operation | Unsorted Array | Unsorted Singly Linked List | Sorted Array (Descending)* | Sorted Singly Linked List (Ascending) | Binary Min-Heap (Array) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **`insert(key)`** | $O(1)$ (append at end) | $O(1)$ (insert at head) | $O(n)$ (binary search $O(\log n)$ + shift $O(n)$) | $O(n)$ (linear scan to insert in order) | $O(\log n)$ (sift-up), $O(1)$ avg |
| **`peek_min()`** | $O(n)$ (linear scan) | $O(n)$ (linear scan) | $O(1)$ (at index `n - 1`) | $O(1)$ (at `head`) | $O(1)$ (at root index `0`) |
| **`extract_min()`** | $O(n)$ (find min + swap last) | $O(n)$ (find min + unlink) | $O(1)$ (decrement `n`) | $O(1)$ (remove `head`) | $O(\log n)$ (swap last to root + sift-down) |
| **`decrease_key(idx, new_k)`** | $O(1)$ | $O(1)$ (given node ptr) | $O(n)$ (re-shift) | $O(n)$ (re-position) | $O(\log n)$ (sift-up from `idx`) |
| **`build_pq` (from $n$ items)** | $O(n)$ | $O(n)$ | $O(n \log n)$ (sort array) | $O(n^2)$ (or $O(n \log n)$ merge sort) | $O(n)$ (Floyd's bottom-up heapify) |
| **Merge Two PQs ($n_1 + n_2 = N$)** | $O(n_2)$ (array copy) | $O(1)$ (link tail to head) | $O(N)$ (two-pointer merge) | $O(N)$ (two-pointer merge) | $O(N)$ (concatenate arrays + Floyd's `build_heap`) |

*\*Note: Keeping a sorted array in descending order puts the minimum element at the end (`arr[n-1]`), allowing `extract_min()` in $O(1)$ by simply decrementing `size`, whereas ascending order requires an $O(n)$ shift on `extract_min()`.*

---

### 4.3 Queue Applications: Josephus Problem & CPU Scheduling

| Problem / Algorithm | Approach / Data Structure | Time Complexity | Space Complexity | Remarks |
| :--- | :--- | :--- | :--- | :--- |
| **Josephus Problem ($n$ people, step $k$)** | Circular Queue Simulation (`k-1` dequeues/enqueues per elimination) | $O(n \cdot k)$ | $O(n)$ | Simple to implement; slow when $k \gg n$ |
| **Josephus Problem ($n$ people, step $k$)** | Circular Singly Linked List Simulation | $O(n \cdot k)$ | $O(n)$ | Can optimize each step to $k \bmod \text{remaining}$ hops |
| **Josephus Problem ($n$ people, step $k$)** | Order-Statistic Balanced BST / Fenwick Tree | $O(n \log n)$ | $O(n)$ | Independent of $k$; produces full elimination sequence |
| **Josephus Survivor Only** | Mathematical Recurrence: $J(n,k) = (J(n-1,k) + k) \bmod n$ | $O(n)$ | $O(1)$ (iterative) / $O(n)$ (recursive) | Computes final survivor index in a single `for` loop ($i = 2 \dots n$) |
| **CPU Scheduling: FCFS (FIFO)** | Circular / Linked FIFO Queue ($n$ processes) | $O(n \log n)$ (if sorting arrival times) or $O(n)$ | $O(n)$ | Non-preemptive; susceptible to **Convoy Effect** |
| **CPU Scheduling: SJF / Priority (Non-Preemptive)** | Binary Min-Heap Priority Queue ($n$ processes) | $O(n \log n)$ | $O(n)$ | Optimal minimum average waiting time among non-preemptive policies |
| **CPU Scheduling: SRTF (Preemptive SJF)** | Binary Min-Heap keyed on Remaining Burst Time | $O((n + C_{sw}) \log n)$ ($C_{sw} \le n$ preemptions) | $O(n)$ | Preempts only when a newly arriving job has strictly shorter remaining time |
| **CPU Scheduling: Round-Robin (Quantum $q$)** | FIFO Ready Queue ($T_{\text{total}} = \sum BT_i$) | $O\left(n \log n + \frac{T_{\text{total}}}{q}\right)$ | $O(n)$ | Each time slice of duration $\le q$ takes $O(1)$ queue operations |

---

### 4.4 General Trees, Binary Trees, LCRS & Tree Traversals

Let $n$ be the number of nodes and $h$ be the height of the tree ($\lfloor \log_2 n \rfloor \le h \le n-1$ for binary trees).

| Operation / Traversal | Time Complexity | Auxiliary Space (Balanced Tree, $h = \Theta(\log n)$) | Auxiliary Space (Skewed Worst-Case, $h = \Theta(n)$) | Implementation Mechanism |
| :--- | :--- | :--- | :--- | :--- |
| **Recursive Preorder (`NLR`)** | $\Theta(n)$ | $\Theta(\log n)$ | $\Theta(n)$ | OS Call Stack of depth $h + 1$ |
| **Recursive Inorder (`LNR`)** | $\Theta(n)$ | $\Theta(\log n)$ | $\Theta(n)$ | OS Call Stack of depth $h + 1$ |
| **Recursive Postorder (`LRN`)** | $\Theta(n)$ | $\Theta(\log n)$ | $\Theta(n)$ | OS Call Stack of depth $h + 1$ |
| **Iterative Preorder / Inorder (1 Explicit Stack)** | $\Theta(n)$ | $\Theta(\log n)$ | $\Theta(n)$ | Explicit stack holds at most $h+1$ ancestors |
| **Iterative Postorder (Two-Stack Method)** | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | Second stack (`s2`) accumulates all $n$ nodes before printing! |
| **Iterative Postorder (One-Stack + `last_visited`)** | $\Theta(n)$ | $\Theta(\log n)$ | $\Theta(n)$ | Single stack tracks active root-to-node path of length $\le h+1$ |
| **Level-Order Traversal (BFS Queue)** | $\Theta(n)$ | $\Theta(n)$ (Max width $w \approx \lceil n/2 \rceil$ at bottom level!) | $\Theta(1)$ (Skewed tree has width $w = 1$!) | Queue size is bounded by maximum tree width $W \le \lceil n/2 \rceil$, **opposite** to DFS stack! |
| **Morris / Threaded Inorder Traversal** | $\Theta(n)$ (visits each edge $\le 3$ times) | $\Theta(1)$ | $\Theta(1)$ | Uses temporary right threads on leaf/predecessor nodes |
| **Compute Height / Count Nodes / Count Leaves** | $\Theta(n)$ | $\Theta(\log n)$ (DFS) / $\Theta(n)$ (BFS) | $\Theta(n)$ (DFS) / $\Theta(1)$ (BFS) | Visits every node exactly once |
| **N-ary Tree $\leftrightarrow$ Binary Tree (LCRS) Conversion** | $\Theta(n)$ | $\Theta(\log n)$ | $\Theta(n)$ | `left` = First Child, `right` = Next Sibling; bijection preserves $n$ nodes |
| **Forest of $k$ Trees $\leftrightarrow$ Binary Tree (LCRS)** | $\Theta(n)$ | $\Theta(\log n)$ | $\Theta(n)$ | Roots $T_1, T_2, \dots, T_k$ linked as right siblings: $T_1\text{->right} = T_2\text{->right} = \dots$ |
| **Reconstruct Binary Tree from Inorder + Preorder/Postorder** | **Naive**: $O(n^2)$ (skewed), $O(n \log n)$ (balanced)<br>**With Hash Map**: $\Theta(n)$ | $\Theta(\log n)$ (or $\Theta(n)$ with hash map) | $\Theta(n)$ | Naive linear scan for root in `inorder[]` takes $O(n)$ per node; Hash Map lookup takes $O(1)$ |

---

## 5. Unit 3 Complexity Tables: BSTs, Threaded Trees, Heaps, AVL, Splay & Graph Representations

### 5.1 Binary Search Trees (BST), Expression Trees & Threaded BSTs

| Structure & Representation | Search (`key`) | Insert (`key`) | Delete (`key`) | Inorder Successor / Predecessor | Full Inorder Traversal (Time / Aux Space) | Total Storage Space |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Pointer BST (Average / Random Keys)** | $\Theta(\log n)$ | $\Theta(\log n)$ | $\Theta(\log n)$ | $\Theta(\log n)$ (from root) or $O(h)$ | $\Theta(n)$ time / $\Theta(\log n)$ stack space | $\Theta(n)$ |
| **Pointer BST (Skewed Worst-Case, sorted input)** | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ time / $\Theta(n)$ stack space | $\Theta(n)$ |
| **Implicit Array BST (`2i+1`, `2i+2`)** | $O(h)$ | $O(h)$ | $O(2^h)$ (shifting subtree!) | $O(h)$ | $\Theta(2^h)$ time / $O(h)$ space | $\Theta(2^h)$ ($\Theta(2^n)$ worst-case if skewed!) |
| **Double-Threaded BST (`lthread`, `rthread`)** | $O(h)$ | $O(h)$ | $O(h)$ | $O(h)$ worst, **$O(1)$ amortized** | **$\Theta(n)$ time / $\Theta(1)$ aux space!** | $\Theta(n)$ (+2 boolean flags/node) |
| **Expression Tree ($n$ nodes)** | N/A | **Build from Postfix**: $\Theta(n)$ time, $\Theta(n)$ stack | N/A | **Evaluate**: $\Theta(n)$ time, $\Theta(h)$ stack | $\Theta(n)$ time / $\Theta(h)$ stack | $\Theta(n)$ |

> [!TIP]
> **Why Threaded BST Successor is $O(1)$ Amortized**: Although finding the inorder successor of a single node with `rthread == false` requires stepping to `right` and following `left` pointers down to the leftmost node (which can take $O(h)$ steps for a single call), across a complete inorder traversal of all $n$ nodes, every tree edge is traversed at most twice (once down, once up via a thread). Thus $n$ calls to `inorder_successor()` take $\Theta(n)$ total time, or **$O(1)$ amortized time per step** with **$O(1)$ auxiliary memory**!

---

### 5.2 Binary Heap, AVL Tree & Splay Tree Comparison

| Operation / Property | Binary Min/Max Heap (Array) | Unbalanced BST | AVL Tree (Height-Balanced BST) | Splay Tree (Self-Adjusting BST) |
| :--- | :--- | :--- | :--- | :--- |
| **Structural Invariant** | **Complete Binary Tree** ($h = \lfloor \log_2 n \rfloor$) + Heap Order | BST Order ($L < N < R$) | BST Order + $\|\text{height}(L) - \text{height}(R)\| \le 1$ | BST Order + Splay accessed node to root |
| **Height $h$ (Worst-Case)** | $\lfloor \log_2 n \rfloor \approx 1.00 \log_2 n$ | $n - 1$ (Linear chain) | $< 1.4404 \log_2(n + 2) - 0.328 = \Theta(\log n)$ | $n - 1$ (e.g., sequential access $1, 2, \dots, n$) |
| **Search (`search(key)`)** | $O(n)$ (Heap is NOT ordered for arbitrary search!) | $O(\log n)$ avg, **$O(n)$ worst** | **$O(\log n)$ worst-case** | **$O(\log n)$ amortized**, $O(n)$ single worst-case |
| **Insert (`insert(key)`)** | $O(\log n)$ worst, $O(1)$ avg | $O(\log n)$ avg, **$O(n)$ worst** | **$O(\log n)$ worst-case** ($\le 1$ single or double rotation!) | **$O(\log n)$ amortized**, $O(n)$ worst-case |
| **Delete (`delete(key)`)** | $O(\log n)$ (given index), $O(n)$ (by value) | $O(\log n)$ avg, **$O(n)$ worst** | **$O(\log n)$ worst-case** (up to $O(\log n)$ rotations) | **$O(\log n)$ amortized**, $O(n)$ worst-case |
| **Find Min / Find Max** | **Min-Heap**: $O(1)$ min, $O(n)$ max (in $\lceil n/2 \rceil$ leaves) | $O(\log n)$ avg, $O(n)$ worst | $O(\log n)$ worst-case (or $O(1)$ with cached ptr) | $O(\log n)$ amortized |
| **Extract Min/Max (Root)** | **$O(\log n)$ worst-case** | $O(\log n)$ avg, $O(n)$ worst | $O(\log n)$ worst-case | $O(\log n)$ amortized |
| **Build from $n$ Unsorted Keys** | **$O(n)$** (Floyd's bottom-up) / $O(n \log n)$ (top-down) | $O(n \log n)$ avg, $O(n^2)$ worst | $\Theta(n \log n)$ worst-case | $O(n \log n)$ |
| **Sort $n$ Keys** | **Heap Sort**: $\Theta(n \log n)$ time, $O(1)$ aux space | Tree Sort: $O(n \log n)$ avg, $O(n^2)$ worst | $\Theta(n \log n)$ time, $\Theta(n)$ space | $\Theta(n \log n)$ time, $\Theta(n)$ space |
| **Extra Metadata per Node** | $0$ bytes (implicit array indices!) | $2$ pointers (`left`, `right`) | $2$ pointers + `int height` (or 2-bit balance factor) | $2$ pointers (or $3$ with `parent`; **0 balance bits!**) |

---

### 5.3 Graph Representations ($G = (V, E)$)

| Operation / Metric | Adjacency Matrix (`adj[V][V]`) | Adjacency List (`head[V]` + Linked Nodes) | Incidence Matrix (`inc[V][E]`) | Edge List (`Edge[E]`) |
| :--- | :--- | :--- | :--- | :--- |
| **Storage Space** | $\Theta(V^2)$ | $\Theta(V + E)$ | $\Theta(V \cdot E)$ | $\Theta(E)$ |
| **Check if Edge $(u, v)$ Exists** | **$O(1)$** | $O(\text{deg}(u)) = O(V)$ worst | $O(E)$ | $O(E)$ |
| **Find Degree of Vertex $u$** | $\Theta(V)$ | $\Theta(\text{deg}(u))$ (or $O(1)$ if cached) | $\Theta(E)$ | $\Theta(E)$ |
| **Iterate Over All Neighbors of $u$** | $\Theta(V)$ (must scan entire row $u$) | **$\Theta(\text{deg}(u))$** (only visits actual neighbors) | $\Theta(E)$ | $\Theta(E)$ |
| **Insert Edge $(u, v)$** | **$O(1)$** | **$O(1)$** (at head of `adj[u]` if no dup check) | $O(V \cdot E)$ (resize) | $O(1)$ |
| **Remove Edge $(u, v)$** | **$O(1)$** (`adj[u][v] = 0`) | $O(\text{deg}(u))$ (find & unlink node) | $O(V \cdot E)$ | $O(E)$ |
| **Add New Vertex** | $O(V^2)$ (must reallocate 2D matrix) | $O(1)$ amortized (append to header array) | $O(V \cdot E)$ | $O(1)$ |
| **Remove Vertex $u$** | $O(V^2)$ | $O(V + E)$ (must scan all lists for incoming edges) | $O(V \cdot E)$ | $O(E)$ |
| **Full BFS Traversal** | $\Theta(V^2)$ | **$\Theta(V + E)$** | $\Theta(V \cdot E)$ | $\Theta(V \cdot E)$ |
| **Full DFS Traversal** | $\Theta(V^2)$ | **$\Theta(V + E)$** | $\Theta(V \cdot E)$ | $\Theta(V \cdot E)$ |
| **Best Suited For** | Dense graphs ($E \approx V^2$), frequent edge existence queries | Sparse graphs ($E \ll V^2$), most real-world networks | Hypergraphs, Network flow theory | Kruskal's MST algorithm |

---

## 6. Unit 4 Complexity Tables: Graph Algorithms, Tries, Suffix Trees & Hashing

### 6.1 Graph Traversal & Connectivity Algorithms

| Algorithm / Problem | Adjacency List Time | Adjacency Matrix Time | Auxiliary Space | Key Data Structures Used |
| :--- | :--- | :--- | :--- | :--- |
| **Breadth-First Search (BFS — Full Graph)** | $\Theta(V + E)$ | $\Theta(V^2)$ | $\Theta(V)$ | FIFO Queue `Q[V]` + `visited[V]` |
| **Depth-First Search (DFS — Full Graph)** | $\Theta(V + E)$ | $\Theta(V^2)$ | $\Theta(V)$ | Recursion Stack (or Explicit Stack) + `visited[V]` |
| **Path Existence ($s \leadsto t$ Reachability)** | $O(V + E)$ | $O(V^2)$ | $O(V)$ | Either BFS or DFS starting from $s$ |
| **Shortest Path in Unweighted Graph (Single Source)** | $\Theta(V + E)$ | $\Theta(V^2)$ | $\Theta(V)$ | **BFS ONLY** (`dist[v] = dist[u] + 1`, `parent[v] = u`) |
| **Connected Components (Undirected Graph)** | $\Theta(V + E)$ | $\Theta(V^2)$ | $\Theta(V)$ | Outer loop $u = 0 \dots V-1$ launching BFS/DFS on unvisited $u$ |
| **Cycle Detection (Undirected Graph)** | $O(V + E)$ | $O(V^2)$ | $O(V)$ | DFS/BFS checking visited neighbor $v \ne \text{parent}[u]$ (or $E > V - 1$ early stop) |
| **Cycle Detection (Directed Graph)** | $O(V + E)$ | $O(V^2)$ | $O(V)$ | 3-Color DFS (`WHITE=0`, `GRAY=1` active stack, `BLACK=2` done); back-edge to `GRAY` |
| **Bipartite Graph Check (2-Coloring)** | $O(V + E)$ | $O(V^2)$ | $O(V)$ | BFS/DFS assigning alternating colors; odd-length cycle $\iff$ same-color edge |

---

### 6.2 Tries, Suffix Tries & Suffix Trees

Let $W$ be a word/pattern of length $L = |W|$, $n$ be the number of words in a dictionary, $N = \sum |W_i|$ be the total characters across all dictionary words, and $m = |S|$ be the length of a single text string $S$ for suffix structures.

| Structure | Construction Time | Total Space Complexity | Exact Search (`|P| = L`) | Prefix Search (`|P| = L`) | Delete Word (`|W| = L`) | Find All $k$ Occurrences of $P$ in Text $S$ |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Standard Trie (Array `children[\|\Sigma\|]`)** | $O(N \cdot |\Sigma|)$ (or $O(N)$ if zero-init amortized) | $O(N \cdot |\Sigma|)$ worst-case ($O(\text{nodes} \cdot |\Sigma|)$) | **$O(L)$** (independent of $n$!) | **$O(L)$** | $O(L \cdot |\Sigma|)$ (or $O(L)$ with child counter) | N/A (Dictionary structure) |
| **Standard Trie (Linked List of Children)** | $O(N \cdot |\Sigma|)$ | **$O(N)$** (no null child pointers!) | $O(L \cdot |\Sigma|)$ | $O(L \cdot |\Sigma|)$ | $O(L \cdot |\Sigma|)$ | N/A |
| **Compressed Trie (Radix / Patricia Tree)** | $O(N)$ | **$O(n \cdot |\Sigma|)$** ($\le 2n$ nodes!) | **$O(L)$** | **$O(L)$** | $O(L)$ | N/A |
| **Uncompressed Suffix Trie (of string $S$, $|S|=m$)** | **$\Theta(m^2)$** | **$\Theta(m^2 \cdot |\Sigma|)$** (up to $\frac{m(m+1)}{2} + 1$ nodes) | **$O(L)$** (Substring check!) | **$O(L)$** | N/A (Static text index) | $O(L + m^2)$ (or $O(L + k)$ with leaf lists) |
| **Compact Suffix Tree (Naive Insertion of $m$ suffixes)** | $O(m^2)$ | **$\Theta(m)$** ($\le 2m$ nodes, edge labels as `[start, end]` indices!) | **$O(L)$** (or $O(L \log |\Sigma|)$) | **$O(L)$** | N/A | **$O(L + k)$** ($k$ leaves in subtree) |
| **Compact Suffix Tree (Ukkonen's Online Algorithm)** | **$\Theta(m)$** (for constant $|\Sigma|$) | **$\Theta(m)$** ($\le 2m$ nodes + suffix links) | **$O(L)$** | **$O(L)$** | N/A | **$O(L + k)$** |

#### Common String Problems Solved in Linear Time via Compact Suffix Tree ($|S| = m$, $|P| = L$)

| String Problem | Suffix Tree Complexity | Method |
| :--- | :--- | :--- |
| **Check if $P$ is a Substring of $S$** | $O(L)$ | Traverse characters of $P$ from root; succeeds iff $P$ is exhausted without mismatch |
| **Check if $P$ is a Suffix of $S$** | $O(L)$ | Traverse $P$'s characters + terminator `$`; or check if match ends at a leaf mark |
| **Count Occurrences of $P$ in $S$** | $O(L)$ (with precomputed leaf counts) | Precompute `leaf_count[u]` via $O(m)$ DFS; return `leaf_count` of node reached after matching $P$ |
| **Find Longest Repeated Substring (LRS) in $S$** | $\Theta(m)$ | Find the internal (branching) node with maximum **string-depth** (total edge characters from root) |
| **Longest Common Substring (LCS) of $S_1$ and $S_2$** | $\Theta(|S_1| + |S_2|)$ | Build Generalized Suffix Tree for $S_1\text{\#}S_2\text{\$}$; find deepest internal node having leaves from both strings |
| **Count Distinct Substrings of $S$** | $\Theta(m)$ | Sum lengths `(end - start + 1)` of all non-`$` edges across the suffix tree |

---

### 6.3 Hash Tables, Collision Resolution & Rehashing

Let $n$ be the number of stored keys, $m$ be the hash table capacity (number of buckets), and $\alpha = \frac{n}{m}$ be the load factor. Assume Simple Uniform Hashing (SUHA) for average-case bounds.

#### General Operations Complexity

| Method | Load Factor $\alpha$ Range | Search (Average) | Insert (Average) | Delete (Average) | Worst-Case (All $n$ keys collide) | Space Complexity | Cache Locality |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Separate Chaining (Unsorted SLL Chains)** | $0 \le \alpha < \infty$ (typically $\alpha \le 1.0$) | $\Theta(1 + \alpha)$ | $O(1)$ (no dup check) or $\Theta(1 + \alpha)$ | $\Theta(1 + \alpha)$ | $\Theta(n)$ | $\Theta(m + n)$ | Poor (pointer chasing) |
| **Separate Chaining (AVL Tree Buckets)** | $0 \le \alpha < \infty$ | $\Theta(1 + \log(1+\alpha))$ | $\Theta(1 + \log(1+\alpha))$ | $\Theta(1 + \log(1+\alpha))$ | **$\Theta(\log n)$** | $\Theta(m + n)$ | Poor |
| **Open Addressing: Linear Probing** | $0 \le \alpha < 1$ (keep $\alpha \le 0.7$) | $O(1)$ (for bounded $\alpha$) | $O(1)$ | $O(1)$ (Requires **Tombstone `DELETED`**!) | $\Theta(n)$ | $\Theta(m)$ (No pointers!) | **Excellent** (L1 cache line!) |
| **Open Addressing: Quadratic Probing** | $0 \le \alpha \le 0.5$ ($m$ prime!) | $O(1)$ | $O(1)$ | $O(1)$ (Tombstone) | $\Theta(n)$ | $\Theta(m)$ | Good |
| **Open Addressing: Double Hashing** | $0 \le \alpha < 1$ ($h_2(k) \perp m$) | $O(1)$ | $O(1)$ | $O(1)$ (Tombstone) | $\Theta(n)$ | $\Theta(m)$ | Moderate |
| **Rehashing (Resize $m \to m' \approx 2m$)** | Triggered when $\alpha > \alpha_{\max}$ | N/A | **$O(n + m')$ single resize**, **$O(1)$ amortized** | N/A | $\Theta(n + m')$ | $\Theta(m')$ | Re-eliminates all tombstones! |

#### Exact Expected Number of Probes as a Function of Load Factor $\alpha$ (Knuth's Formulas)

| Collision Resolution Scheme | Probe Sequence $h(k, i)$ ($i = 0, 1, \dots, m-1$) | Expected Probes: Unsuccessful Search / Insert ($U_\alpha$) | Expected Probes: Successful Search ($S_\alpha$) | Clustering / Pathological Vulnerability |
| :--- | :--- | :--- | :--- | :--- |
| **Separate Chaining** | Bucket $h(k)$ chain scan | $1 + \alpha$ (or $\alpha$ key comparisons) | $1 + \frac{\alpha}{2}$ | Graceful degradation even when $\alpha > 1$ |
| **Linear Probing** | $(h(k) + i) \bmod m$ | $\displaystyle\frac{1}{2}\left(1 + \frac{1}{(1 - \alpha)^2}\right)$ | $\displaystyle\frac{1}{2}\left(1 + \frac{1}{1 - \alpha}\right)$ | **Primary Clustering**: Any key landing in a cluster grows that cluster |
| **Quadratic Probing** | $(h(k) + c_1 i + c_2 i^2) \bmod m$ | $\displaystyle\frac{1}{1 - \alpha} - \alpha - \ln(1 - \alpha)$ | $\displaystyle 1 - \ln(1 - \alpha) - \frac{\alpha}{2}$ | **Secondary Clustering**: Keys with identical $h(k)$ trace the exact same probe sequence; guaranteed empty slot only if $\alpha < 0.5$ and $m$ is prime |
| **Double Hashing (Ideal Uniform Probing)** | $(h_1(k) + i \cdot h_2(k)) \bmod m$ | $\displaystyle\frac{1}{1 - \alpha}$ | $\displaystyle\frac{1}{\alpha} \ln\left(\frac{1}{1 - \alpha}\right)$ | **Eliminates both Primary & Secondary Clustering**; requires $h_2(k) \ne 0$ and $\gcd(h_2(k), m) = 1$ |

##### Numerical Comparison of Expected Probes ($U_\alpha$ / $S_\alpha$) Across Load Factors

| Load Factor $\alpha$ | Separate Chaining ($U_\alpha$ / $S_\alpha$) | Double Hashing ($U_\alpha$ / $S_\alpha$) | Linear Probing ($U_\alpha$ / $S_\alpha$) |
| :--- | :--- | :--- | :--- |
| **$\alpha = 0.50$ (50% full)** | $1.50$ / $1.25$ | $2.00$ / $1.39$ | $2.50$ / $1.50$ |
| **$\alpha = 0.75$ (75% full)** | $1.75$ / $1.38$ | $4.00$ / $1.85$ | $8.50$ / $2.50$ |
| **$\alpha = 0.90$ (90% full)** | $1.90$ / $1.45$ | $10.00$ / $2.56$ | **$50.50$** / $5.50$ |
| **$\alpha = 0.95$ (95% full)** | $1.95$ / $1.48$ | $20.00$ / $3.15$ | **$200.50$** / $10.50$ |

---

## 7. Quick Decision Matrix: "Which Data Structure Should I Choose When...?"

Use this architectural decision guide to justify data structure selection in design questions:

| If Your Application Requires... | Best Data Structure Choice | Why It Wins (Complexity & Structural Justification) | What to Avoid & Why |
| :--- | :--- | :--- | :--- |
| **$O(1)$ random access by index $i$ with rare mid-array insertions** | **Dynamic Array** | Contiguous memory provides $O(1)$ address arithmetic `base + i * sizeof(T)` and unbeatable CPU cache locality | Linked Lists ($O(n)$ sequential traversal to reach index $i$) |
| **Frequent $O(1)$ insertions & deletions at both ends (front and back)** | **Circular Array Deque** or **Doubly Linked List** (or **Circular SLL with Tail** if `deleteRear` not needed) | Circular Deque gives $O(1)$ at both ends with zero per-element pointer overhead; DLL gives $O(1)$ without capacity limits | Singly Linked List with only Head ($O(n)$ at back) or Head+Tail ($O(n)$ for `deleteRear`) |
| **Representing an $R \times C$ matrix where $< 5\%$ of entries are non-zero, needing fast transpose & addition** | **Sorted Triplet Array (COO)** or **CSR** (static) / **Cross-Linked Multilist** (dynamic updates) | Triplet Fast Transpose runs in $O(C + nnz)$ instead of $O(R \cdot C)$; Multilist supports row/column traversal in $O(nnz_{\text{row/col}})$ | Dense 2D Array ($\Theta(R \cdot C)$ wasted memory and traversal time) |
| **Undo/Redo history, function call frames, or nested bracket/syntax validation** | **Stack (LIFO)** | Strictly Last-In-First-Out access pattern; $O(1)$ `push` and `pop` | General List or Queue (wrong ordering discipline) |
| **Fair resource sharing, print spooling, Round-Robin CPU scheduling, or level-by-level graph/tree exploration** | **Circular Array Queue** or **Linked FIFO Queue** | Strictly First-In-First-Out; Circular modular indexing `(idx + 1) % CAP` prevents false overflow in $O(1)$ time | Simple Linear Array Queue (suffers false overflow or $O(n)$ shifting) |
| **Repeatedly extracting the highest-priority task / shortest remaining job without needing full sorted traversal** | **Binary Min/Max Heap** | $O(1)$ `peek`, $O(\log n)$ `insert`/`extract`, $O(n)$ `build_heap`, and **zero pointer overhead** in a flat array | Sorted Array ($O(n)$ insert) or Balanced BST (higher pointer & rotation overhead) |
| **Dynamic dictionary requiring strict worst-case $O(\log n)$ search, insert, delete, AND ordered range queries `[low, high]`** | **AVL Tree** | Strict height balance ($|\text{BF}| \le 1$, $h < 1.44 \log_2 n$) guarantees lookup speed and supports $O(\log n + k)$ range scans | Hash Table (destroys key ordering; range query takes $O(m)$!) or Unbalanced BST ($O(n)$ worst-case on sorted input) |
| **Ordered dictionary without deterministic rebalance code, or probabilistic $O(\log n)$ concurrent access** | **Skip List** | Expected $O(\log n)$ search/insert/delete using simple linked-list pointer splices instead of complex tree rotations | Unbalanced BST ($O(n)$ on sorted input) |
| **Self-optimizing cache where $80\%$ of queries hit $20\%$ of recently accessed keys (temporal locality)** | **Splay Tree** | Splaying moves accessed keys to the root; $O(\log n)$ amortized cost, zero balance-factor storage, working-set property | AVL Tree (keeps rarely used and hot keys at similar average depth) |
| **Fastest possible average $O(1)$ exact key-value lookup when ordering/range queries are NOT needed** | **Hash Table** (Linear Probing for small keys & cache speed; Separate Chaining for high/unpredictable $\alpha$) | Expected $O(1)$ search/insert/delete when $\alpha$ is bounded via rehashing | Any comparison-based tree ($\Omega(\log n)$ lower bound per lookup) |
| **Autocomplete, spell-checking, IP longest-prefix matching, or dictionary of strings sharing prefixes** | **Trie (Prefix Tree)** or **Compressed Radix Tree** | Lookup takes $O(|P|)$ time proportional only to query word length, completely independent of dictionary size $n$! | Hash Table (cannot do prefix search without hashing all prefixes; vulnerable to hash-flooding DoS) |
| **Repeated $O(|P|)$ substring queries, Longest Repeated Substring, or Longest Common Substring on a large static text $S$** | **Compact Suffix Tree** (built via Ukkonen's Algorithm) | $\Theta(|S|)$ preprocessing time and space; subsequently answers any substring query of length $|P|$ in $O(|P|)$ time! | Uncompressed Suffix Trie ($\Theta(|S|^2)$ memory explosion) |
| **Inorder traversal of a binary tree in a memory-constrained embedded system with NO recursion or stack allowed** | **Double-Threaded Binary Search Tree** | Repurposes the $n+1$ wasted `NULL` child pointers into inorder predecessor/successor threads for $\Theta(n)$ traversal in **$O(1)$ auxiliary RAM** | Standard BST (requires $O(h)$ call stack or explicit stack) |
| **Representing a real-world road network, social graph, or web graph ($E \ll V^2$)** | **Adjacency List** | Uses $\Theta(V + E)$ space and traverses in $\Theta(V + E)$ time | Adjacency Matrix ($\Theta(V^2)$ space is $10^{12}$ cells for $V = 10^6$ vertices!) |
| **Finding the shortest path (minimum number of hops) in an unweighted graph** | **Breadth-First Search (BFS)** | Explores vertices in monotonically increasing distance layers $d = 0, 1, 2, \dots$ in $\Theta(V + E)$ time | Depth-First Search (DFS finds *a* path, NOT the shortest path!) |

---

> **Next Step**: Continue to **[Appendix 02: C Programming Patterns & Idioms](./02-C-Programming-Patterns.md)** to master the production-grade C11 pointer and memory idioms used to implement these structures without bugs.
