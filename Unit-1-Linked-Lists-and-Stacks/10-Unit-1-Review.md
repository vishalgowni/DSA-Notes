# 10 — Unit 1 Comprehensive Review & Exam Preparation

---

## Table of Contents

1. [Important Definitions (Unit 1 Glossary)](#1-important-definitions-unit-1-glossary)
2. [Essential Diagrams and Formulas](#2-essential-diagrams-and-formulas)
   - 2.1 [Core Pointer Rewirings (SLL, DLL, CLL)](#21-core-pointer-rewirings-sll-dll-cll)
   - 2.2 [Sparse Matrix Triplet Representation & Memory Breakeven Formula](#22-sparse-matrix-triplet-representation--memory-breakeven-formula)
   - 2.3 [Skip List Probability, Expected Height, and Space Formulas](#23-skip-list-probability-expected-height-and-space-formulas)
   - 2.4 [Stack ADT Invariants and Operator Precedence/Associativity Reference](#24-stack-adt-invariants-and-operator-precedenceassociativity-reference)
3. [Commonly Confused Concepts](#3-commonly-confused-concepts)
   - 3.1 [`malloc` vs. `calloc` vs. `realloc`](#31-malloc-vs-calloc-vs-realloc)
   - 3.2 [Dangling Pointer vs. Memory Leak](#32-dangling-pointer-vs-memory-leak)
   - 3.3 [Singly (SLL) vs. Doubly (DLL) vs. Circular Linked List (CLL) Tradeoffs](#33-singly-sll-vs-doubly-dll-vs-circular-linked-list-cll-tradeoffs)
   - 3.4 [Infix-to-Postfix vs. Infix-to-Prefix Equal-Precedence Tie-Breaking](#34-infix-to-postfix-vs-infix-to-prefix-equal-precedence-tie-breaking)
   - 3.5 [Operand Pop Order (`op1 op op2`) in Postfix vs. Prefix Evaluation](#35-operand-pop-order-op1-op-op2-in-postfix-vs-prefix-evaluation)
   - 3.6 [Tail Recursion vs. Non-Tail Recursion](#36-tail-recursion-vs-non-tail-recursion)
4. [Consolidated Operation-Complexity Table for Unit 1](#4-consolidated-operation-complexity-table-for-unit-1)
5. [Short-Answer Questions (10 Questions)](#5-short-answer-questions-10-questions)
6. [Conceptual Questions (8 Deep-Thinking Questions)](#6-conceptual-questions-8-deep-thinking-questions)
7. [Algorithm-Writing Questions (5 Pseudocode Design Problems)](#7-algorithm-writing-questions-5-pseudocode-design-problems)
8. [C Programming Questions (5 Coding Problems)](#8-c-programming-questions-5-coding-problems)
9. [Worked Problem-Solving Questions (4 Step-by-Step Exam Problems)](#9-worked-problem-solving-questions-4-step-by-step-exam-problems)
   - 9.1 [Worked Problem 1: Pointer Trace & In-Place SLL Reversal](#91-worked-problem-1-pointer-trace--in-place-sll-reversal)
   - 9.2 [Worked Problem 2: Sparse Matrix Triplet Representation & Fast Transpose](#92-worked-problem-2-sparse-matrix-triplet-representation--fast-transpose)
   - 9.3 [Worked Problem 3: Polynomial Addition Trace Using Linked Lists](#93-worked-problem-3-polynomial-addition-trace-using-linked-lists)
   - 9.4 [Worked Problem 4: Infix-to-Postfix, Infix-to-Prefix, and Stack Evaluation Trace](#94-worked-problem-4-infix-to-postfix-infix-to-prefix-and-stack-evaluation-trace)
10. [Complete Answer Keys with Detailed Explanations](#10-complete-answer-keys-with-detailed-explanations)
    - 10.1 [Answers to Short-Answer Questions (Q1–Q10)](#101-answers-to-short-answer-questions-q1q10)
    - 10.2 [Answers to Conceptual Questions (C1–C8)](#102-answers-to-conceptual-questions-c1c8)
    - 10.3 [Solutions to Algorithm-Writing Questions (A1–A5)](#103-solutions-to-algorithm-writing-questions-a1a5)
    - 10.4 [Complete C11 Solutions to Programming Questions (P1–P5)](#104-complete-c11-solutions-to-programming-questions-p1p5)

---

## 1. Important Definitions (Unit 1 Glossary)

| Term | Concise Formal Definition |
| :--- | :--- |
| **Self-Referential Structure** | A C `struct` containing one or more pointer members that point to a `struct` of the same tag type (e.g., `struct Node *next;`). |
| **Dangling Pointer** | A pointer that still holds the memory address of a block that has already been deallocated (`free`d) or whose stack frame has gone out of scope. |
| **Memory Leak** | Heap-allocated memory (`malloc`/`calloc`/`realloc`) that is no longer reachable through any live pointer variable and was never released via `free()`. |
| **Singly Linked List (SLL)** | A linear collection of heap-allocated nodes where each node stores a data payload and a single forward pointer `next` to its successor (`NULL` at the tail). |
| **Doubly Linked List (DLL)** | A linked list where each node maintains two pointers—`prev` (to its predecessor) and `next` (to its successor)—enabling $O(1)$ bidirectional traversal and $O(1)$ deletion given a node pointer. |
| **Circular Linked List (CLL)** | A linked list in which the last (tail) node's `next` pointer points back to the `head` node instead of `NULL`, forming a closed ring. |
| **Sentinel (Dummy Header) Node** | A permanent, non-data header/trailer node kept in a linked list to eliminate special-case `NULL` checks when inserting into or deleting from the head/tail. |
| **Multilist** | A linked structure in which a single node belongs simultaneously to multiple independent lists via multiple link pointers (e.g., student-course enrollment grids). |
| **Sparse Matrix** | An $m \times n$ matrix in which the number of non-zero elements $k$ is much smaller than the total elements ($k \ll m \cdot n$). |
| **3-Tuple (Triplet) Representation** | A compact $(k+1) \times 3$ array storing a sparse matrix: row `0` stores `[rows, cols, nnz]`, and rows `1..k` store `[row_idx, col_idx, value]` in row-major order. |
| **Orthogonal Cross-List** | A linked sparse matrix representation where every non-zero element is a node linked into both its row circular list (`right`) and its column circular list (`down`). |
| **Skip List** | A probabilistic, multi-layered linked list where nodes are promoted to higher express levels with probability $p$, achieving $O(\log n)$ expected search, insertion, and deletion time. |
| **Polynomial Linked List** | A singly linked list ordered in strictly decreasing powers of $x$, where each node stores a non-zero `(coeff, exp)` term. |
| **Stack ADT** | A linear Abstract Data Type restricting all insertions (`push`), removals (`pop`), and inspections (`peek`) to one end (`top`) under the **LIFO (Last-In, First-Out)** principle. |
| **Stack Overflow / Underflow** | **Overflow**: attempting to `push` onto a full fixed-capacity stack (or exhausted heap). **Underflow**: attempting to `pop` or `peek` from an empty stack. |
| **Activation Record (Stack Frame)** | The runtime block pushed onto the system call stack for each active function call, storing arguments, return address, saved frame pointer, and local variables. |
| **Tail Recursion** | A linear recursive call that is the absolute final operation of a function before returning, allowing the compiler to reuse the current stack frame in $O(1)$ space. |
| **Infix / Postfix (RPN) / Prefix** | **Infix**: `A + B` (operator between operands). **Postfix**: `A B +` (operator after operands). **Prefix**: `+ A B` (operator before operands). |
| **Shunting-Yard Algorithm** | Edsger Dijkstra's $O(n)$ operator-stack algorithm for converting parenthesized infix expressions into parenthesis-free Postfix (or Prefix) notation. |

---

## 2. Essential Diagrams and Formulas

### 2.1 Core Pointer Rewirings (SLL, DLL, CLL)

#### A. Singly Linked List In-Place Iterative Reversal (`prev`, `curr`, `next`)
```c
/* Invariant: At the start of each iteration, 'prev' points to the reversed prefix,
 * and 'curr' points to the unreversed suffix. */
Node *prev = NULL, *curr = head, *next = NULL;
while (curr != NULL) {
    next = curr->next; /* 1. Save remainder of list */
    curr->next = prev; /* 2. Reverse current link   */
    prev = curr;       /* 3. Advance prev           */
    curr = next;       /* 4. Advance curr           */
}
head = prev;
```

#### B. Doubly Linked List Insertion After Node `p` (Safe 4-Pointer Order)
```
  BEFORE:   [ p ] <=========================> [ q (p->next) ]

  SAFE ORDER (Wire new_node first before overwriting p->next!):
    1. new_node->next = p->next;
    2. new_node->prev = p;
    3. if (p->next != NULL) p->next->prev = new_node;
    4. p->next = new_node;

  AFTER:    [ p ] <===> [ new_node ] <===> [ q ]
```

#### C. Circular Linked List Maintained by `tail` Pointer Only
Keeping a pointer to `tail` (where `tail->next` is `head`) gives **$O(1)$ access to both `head` and `tail`**:
- **Insert at Head ($O(1)$)**: `new_node->next = tail->next; tail->next = new_node;`
- **Insert at Tail ($O(1)$)**: `new_node->next = tail->next; tail->next = new_node; tail = new_node;`

---

### 2.2 Sparse Matrix Triplet Representation & Memory Breakeven Formula

For an $m \times n$ matrix with $k$ non-zero elements (assuming each integer/entry takes $w$ bytes):
- **Dense 2D Array Memory**: $M_{\text{dense}} = m \cdot n \cdot w$ bytes.
- **3-Tuple (Triplet) Array Memory**: $(k + 1)$ rows $\times 3$ columns (`row`, `col`, `val`) $\implies M_{\text{triplet}} = 3(k + 1) \cdot w$ bytes.
- **Breakeven Inequality (When Triplet Representation Saves Memory)**:
  $$3(k + 1) < m \cdot n \iff k < \frac{m \cdot n}{3} - 1$$
  Thus, the 3-tuple array saves memory whenever the matrix density $\rho = \frac{k}{m \cdot n} < \frac{1}{3}$ ($\approx 33.3\%$).

---

### 2.3 Skip List Probability, Expected Height, and Space Formulas

Let $n$ be the number of elements in a Skip List and $p \in (0, 1)$ be the level-promotion probability (typically $p = \frac{1}{2}$):
- **Probability a Node Reaches Level $L$ (0-indexed)**: $P(\text{level} \ge L) = p^L$.
- **Expected Number of Nodes at Level $L$**: $E[N_L] = n \cdot p^L$.
- **Maximum Level Cap ($L_{\max}$)**: Chosen where $n \cdot p^{L_{\max}} \approx 1 \implies L_{\max} = \lceil \log_{1/p} n \rceil$ (for $p = 1/2$, $L_{\max} = \lceil \log_2 n \rceil$).
- **Expected Number of Forward Pointers per Node**:
  $$E[\text{pointers}] = \sum_{k=1}^{\infty} k(1-p)p^{k-1} = \frac{1}{1 - p} \quad \left(\text{For } p = \frac{1}{2}, \text{ average pointers per node} = \frac{1}{1 - 1/2} = 2\right)$$
- **Total Expected Space**: $\Theta\left(\frac{n}{1-p}\right) = \Theta(n)$ pointers.
- **Expected Search / Insert / Delete Time**: $\Theta(\log n)$ expected, $O(n)$ worst-case.

---

### 2.4 Stack ADT Invariants and Operator Precedence/Associativity Reference

- **Array Stack Invariants (`0`-indexed, `capacity` $= C$)**:
  - $\text{Empty} \iff \text{top} == -1$
  - $\text{Full} \iff \text{top} == C - 1$
  - $\text{Current Size} = \text{top} + 1$
  - `push(x)`: `s->data[++(s->top)] = x;`
  - `pop()`: `return s->data[(s->top)--];`
- **Operator Precedence & Associativity Summary**:
  1. `()` — Highest (Grouping Override)
  2. `~` (Unary `-`) — Precedence `4`, **Right-to-Left**
  3. `^` (Exponentiation) — Precedence `3`, **Right-to-Left**
  4. `*`, `/`, `%` — Precedence `2`, **Left-to-Right**
  5. `+`, `-` — Precedence `1`, **Left-to-Right**

---

## 3. Commonly Confused Concepts

### 3.1 `malloc` vs. `calloc` vs. `realloc`

| Function | Signature | Initialization of Allocated Bytes | Behavior / Common Pitfall |
| :--- | :--- | :--- | :--- |
| **`malloc`** | `void *malloc(size_t size)` | **Uninitialized** (contains indeterminate garbage bytes) | Fastest; reading before writing is Undefined Behavior. |
| **`calloc`** | `void *calloc(size_t n, size_t sz)` | **Zero-initialized** (all bits set to `0`) | Checks for multiplication overflow $n \times \text{sz}$; ideal for arrays/buckets. |
| **`realloc`** | `void *realloc(void *ptr, size_t new_sz)` | Preserves existing prefix; newly added tail bytes are **uninitialized** | If `realloc` fails, it returns `NULL` **and leaves the original block `ptr` allocated**! Never write `p = realloc(p, sz)` directly. |

### 3.2 Dangling Pointer vs. Memory Leak

| Aspect | Dangling Pointer | Memory Leak |
| :--- | :--- | :--- |
| **Root Cause** | Memory was **freed** (or stack frame exited), but a pointer still points to that freed address. | Memory is **still allocated** on the heap, but all pointers to it were overwritten or lost without calling `free()`. |
| **Symptom / Hazard** | Dereferencing or double-freeing causes **Undefined Behavior**, silent data corruption, or `SIGSEGV`. | Process heap consumption grows steadily until heap exhaustion (`malloc == NULL` or OOM crash). |
| **Prevention** | Set pointer to `NULL` immediately after `free(ptr); ptr = NULL;`; never return the address of a local stack variable. | Free every dynamically allocated node before overwriting its last reference or exiting scope. |

### 3.3 Singly (SLL) vs. Doubly (DLL) vs. Circular Linked List (CLL) Tradeoffs

| Operation / Property | Singly Linked List (`head` only) | Singly Linked List (`head` + `tail`) | Circular Linked List (`tail` only) | Doubly Linked List (`head` + `tail`) |
| :--- | :---: | :---: | :---: | :---: |
| **Pointers per Node** | 1 (`next`) | 1 (`next`) | 1 (`next`) | 2 (`prev`, `next`) |
| **Insert at Head** | $O(1)$ | $O(1)$ | $O(1)$ (`tail->next`) | $O(1)$ |
| **Insert at Tail** | $O(n)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **Delete at Head** | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **Delete at Tail** | $O(n)$ | $O(n)$ *(must find pred of `tail`!)* | $O(n)$ *(must find pred of `tail`!)* | **$O(1)$** *(via `tail->prev`)* |
| **Delete Given Node `p`** | $O(n)$ *(need predecessor)* | $O(n)$ *(need predecessor)* | $O(n)$ *(need predecessor)* | **$O(1)$** *(rewire `p->prev` & `p->next`)* |
| **Backward Traversal** | Impossible ($O(n)$ to reverse) | Impossible | Impossible | **$O(1)$ per step** |

### 3.4 Infix-to-Postfix vs. Infix-to-Prefix Equal-Precedence Tie-Breaking

When an incoming operator `op` has the **exact same precedence** as the operator currently at `top` of the operator stack (`prec(top) == prec(op)`):
- **Infix-to-Postfix (Left-to-Right Scan)**:
  - **Pop `top`** if `op` is **Left-Associative** (`+`, `-`, `*`, `/`, `%`). Example: `A - B - C` $\to$ `A B - C -`.
  - **Push `op` (do NOT pop)** if `op` is **Right-Associative** (`^`). Example: `A ^ B ^ C` $\to$ `A B C ^ ^`.
- **Infix-to-Prefix (Right-to-Left Reversed Scan)**:
  - Because the scan direction is reversed, the tie-breaking rule **flips**:
  - **Do NOT pop `top`** if `op` is **Left-Associative** (`+`, `-`, `*`, `/`, `%`).
  - **Pop `top`** if `op` is **Right-Associative** (`^`).

### 3.5 Operand Pop Order (`op1 op op2`) in Postfix vs. Prefix Evaluation

- **Postfix Evaluation (Scanned Left-to-Right)**:
  - In `A B -`, `A` (`op1`) is pushed first and `B` (`op2`) is pushed second. Thus `B` sits on top of the stack!
  - **1st Pop** = `op2` (right operand), **2nd Pop** = `op1` (left operand) $\implies$ compute `op1 - op2`.
- **Prefix Evaluation (Scanned Right-to-Left)**:
  - In `- A B` scanned right-to-left, `B` (`op2`) is pushed first and `A` (`op1`) is pushed second. Thus `A` sits on top of the stack!
  - **1st Pop** = `op1` (left operand), **2nd Pop** = `op2` (right operand) $\implies$ compute `op1 - op2`.

### 3.6 Tail Recursion vs. Non-Tail Recursion

- **Non-Tail Recursive**: `return n * fact(n - 1);` — Even though `fact(n - 1)` is written at the end of the `return` statement, the multiplication `n * (...)` must wait until `fact(n - 1)` returns. Requires $\Theta(n)$ stack frames.
- **Tail Recursive**: `return fact_tail(n - 1, n * acc);` — Zero operations remain in the caller after `fact_tail` returns; its return value is passed straight through. Optimizing compilers reuse the same stack frame in $\Theta(1)$ space.

---

## 4. Consolidated Operation-Complexity Table for Unit 1

Let $n$ be the number of stored elements (or non-zero terms), $R \times C$ be matrix dimensions, and $k$ be the number of non-zero matrix entries.

| Data Structure | Access / Search | Insert at Front | Insert at Back | Insert After Given Node `p` | Delete at Front | Delete at Back | Delete Given Node `p` | Space Complexity |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Static / Dynamic Array** | $O(1)$ index / $O(n)$ search | $O(n)$ | $O(1)$ amort. | $O(n)$ | $O(n)$ | $O(1)$ | $O(n)$ | $\Theta(n)$ |
| **Singly Linked List (`head` only)** | $O(n)$ | $O(1)$ | $O(n)$ | $O(1)$ | $O(1)$ | $O(n)$ | $O(n)$ | $\Theta(n)$ |
| **Singly Linked List (`head` + `tail`)** | $O(n)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(n)$ | $O(n)$ | $\Theta(n)$ |
| **Doubly Linked List (`head` + `tail`)** | $O(n)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | **$O(1)$** | $\Theta(n)$ |
| **Circular SLL (`tail` pointer)** | $O(n)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(n)$ | $O(n)$ | $\Theta(n)$ |
| **Circular DLL (Sentinel Header)** | $O(n)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | **$O(1)$** | $\Theta(n)$ |
| **Skip List (Probabilistic)** | $O(\log n)$ exp. / $O(n)$ worst | $O(\log n)$ exp. | $O(\log n)$ exp. | — | $O(\log n)$ exp. | $O(\log n)$ exp. | $O(\log n)$ exp. | $\Theta(n)$ exp. |
| **Array Stack (Static / Dynamic)** | $O(1)$ `peek` | $O(1)$ `push` (amort.) | — | — | $O(1)$ `pop` | — | — | $\Theta(n)$ |
| **Linked-List Stack (`top == head`)** | $O(1)$ `peek` | $O(1)$ `push` (worst) | — | — | $O(1)$ `pop` (worst) | — | — | $\Theta(n)$ |
| **Sparse Matrix (3-Tuple Array)** | $O(\log k)$ binary / $O(k)$ linear | — | **Naive Transpose**: $O(C \cdot k)$ | **Fast Transpose**: $O(C + k)$ | **Addition**: $O(k_1 + k_2)$ | — | — | $\Theta(k)$ |
| **Polynomial SLL ($m, n$ terms)** | $O(n)$ | — | **Addition**: $O(m + n)$ | **Multiplication**: $O(m \cdot n^2)$ or $O(mn \log(mn))$ | **Evaluation (Horner)**: $O(n)$ | — | — | $\Theta(n)$ |

---

## 5. Short-Answer Questions (10 Questions)

1. **Q1**: Why does a C structure definition `struct Node { int data; struct Node next; };` fail to compile, whereas `struct Node { int data; struct Node *next; };` succeeds?
2. **Q2**: In a Singly Linked List that maintains both `head` and `tail` pointers, why does deleting the `tail` node still require $\Theta(n)$ time?
3. **Q3**: Why is it advantageous to maintain a single `tail` pointer rather than a `head` pointer in a Circular Singly Linked List?
4. **Q4**: Under what exact condition on the number of non-zero elements $k$ does a 3-Tuple (Triplet) sparse matrix representation use strictly less memory than a standard 2D array for an $m \times n$ matrix?
5. **Q5**: In a Skip List with promotion probability $p = 1/2$, what is the expected number of forward pointers per node, and why does a Skip List not require complex rotations like balanced trees?
6. **Q6**: When implementing a Stack using a Singly Linked List, why must the `top` of the stack be mapped to the `head` of the list rather than the `tail`?
7. **Q7**: Explain why shrinking a dynamically resizable array stack by half when `size == capacity / 2` leads to $O(n)$ worst-case amortized thrashing, and state the correct shrinking threshold.
8. **Q8**: State the three distinct failure modes that a Balanced Delimiters (`()`, `[]`, `{}`) validator must check.
9. **Q9**: Convert the infix expression `A ^ B ^ C - D + E / F / G` into both **Postfix** and **Prefix** notations.
10. **Q10**: Why does naive recursive Fibonacci `fib(n)` require $\Theta(2^n)$ time but only $\Theta(n)$ system call-stack space?

---

## 6. Conceptual Questions (8 Deep-Thinking Questions)

1. **C1 (Pointer-to-Pointer Elegance vs. Sentinel Nodes)**: Explain how passing a double pointer `Node **head_ref` (or using a dummy sentinel node) eliminates the special-case branch for inserting or deleting at the head of a Singly Linked List. Trace how a `Node **curr = &head` loop deletes a target node without needing a separate `prev` pointer.
2. **C2 (The `O(1)` SLL Node Deletion Trick and Its Fatal Limitation)**: Given *only* a pointer `Node *p` to a node inside a Singly Linked List (and no `head` pointer), you can delete `p` in $O(1)$ time by copying `p->next->data` into `p->data` and freeing `p->next`. Why does this trick fail if `p` is the tail node, and why can it break external pointers in production systems?
3. **C3 (Fast Transpose vs. Naive Transpose of a Sparse Matrix)**: Naive transposition of a 3-Tuple sparse matrix with $C$ columns and $k$ non-zero elements scans the $k$ rows $C$ times, taking $O(C \cdot k)$ time (which degrades to $O(n^3)$ when $k = \Theta(n^2)$). Explain how the **Fast Transpose** algorithm achieves $O(C + k)$ time using `col_counts[]` and `starting_pos[]` arrays.
4. **C4 (Skip List Express Lanes vs. Array Binary Search)**: Binary search achieves $O(\log n)$ search on a sorted array, yet fails to support $O(\log n)$ insertions. How does a Skip List mimic binary search over a linked structure while supporting $O(\log n)$ insertions and deletions without global rebalancing?
5. **C5 (Polynomial Multiplication Complexity Bottleneck)**: When multiplying two linked-list polynomials of $m$ and $n$ terms, generating all $m \cdot n$ product terms takes $O(m \cdot n)$ operations, yet inserting each product term into a sorted result list can degrade total time to $O(m^2 n^2)$ or $O(m \cdot n^2)$. Explain why, and describe how sorting or merging reduces this complexity.
6. **C6 (Hardware Call Stack vs. Explicit Heap Stack)**: Suppose a recursive graph/tree traversal crashes with a `Segmentation Fault` on a skewed input of depth $n = 200,000$, even though the machine has 16 GB of free RAM. Explain the OS memory layout reason for this crash and how converting the recursion to an explicit heap-allocated stack resolves it.
7. **C7 (Why Reversing an Infix String Flips Associativity Rules)**: Prove why converting Infix to Prefix via the "Reverse String $\to$ Shunting-Yard $\to$ Reverse Output" method requires popping equal-precedence operators (`prec(top) == prec(op)`) **only when `op` is right-associative** (the exact opposite of Infix-to-Postfix).
8. **C8 (Amortized Analysis of Dynamic Stack Doubling)**: Using the aggregate method or accounting (banker's) method, prove that starting with capacity $C_0 = 1$ and doubling capacity whenever the stack is full results in at most $2n$ total element copies across $n$ consecutive `push` operations ($O(1)$ amortized time per `push`).

---

## 7. Algorithm-Writing Questions (5 Pseudocode Design Problems)

1. **A1 (Pairwise Swap of Nodes in a Singly Linked List)**: Write an $O(n)$-time, $O(1)$-space algorithm `PairwiseSwapLinks(head)` that swaps adjacent nodes of a Singly Linked List **by rewiring pointers** (not by swapping `data` fields). For example, `1 -> 2 -> 3 -> 4 -> 5` becomes `2 -> 1 -> 4 -> 3 -> 5`.
2. **A2 (Delete All Occurrences of Key `K` in a Doubly Linked List)**: Write an $O(n)$-time algorithm `DeleteAllKeyDLL(head, tail, K)` that removes and frees every node whose `data == K` in a Doubly Linked List, properly updating `head`, `tail`, `prev`, and `next` pointers across all boundary cases (empty list, all nodes equal `K`, head/tail matches).
3. **A3 (Split a Circular Linked List into Two Halves)**: Given the `head` pointer of a Circular Singly Linked List of $n \ge 2$ nodes, write an $O(n)$-time algorithm `SplitCircularList(head, &head1, &head2)` using the slow/fast pointer technique that splits the ring into two separate Circular Linked Lists (`head1` with $\lceil n/2 \rceil$ nodes and `head2` with $\lfloor n/2 \rfloor$ nodes).
4. **A4 (Sort a Stack Using Only One Temporary Stack)**: Given a stack `S` of integers, write an $O(n^2)$-time algorithm `SortStack(S)` that sorts `S` in ascending order (largest element on top) using at most **one** additional temporary stack `T` and $O(1)$ scalar variables (no arrays allowed).
5. **A5 (Single-Pass Direct Infix Evaluation Using Two Stacks)**: Write an $O(n)$-time algorithm `EvaluateInfixDirect(expr)` that evaluates a fully or partially parenthesized infix arithmetic expression directly in a single left-to-right pass using an `OperatorStack` and an `OperandStack`.

---

## 8. C Programming Questions (5 Coding Problems)

1. **P1 (In-Place Iterative & Recursive SLL Reversal)**: Write a complete C11 function `Node *reverse_sll_iterative(Node *head)` and `Node *reverse_sll_recursive(Node *head)` that reverse a Singly Linked List in-place in $\Theta(n)$ time without allocating any new nodes.
2. **P2 (Floyd's Cycle Detection & Cycle-Start Node in SLL)**: Write a C11 function `Node *detect_and_find_cycle_start(Node *head)` that uses Floyd's Tortoise-and-Hare algorithm to determine in $O(n)$ time and $O(1)$ space whether a Singly Linked List contains a cycle, and if so, returns a pointer to the exact node where the cycle begins (or `NULL` if acyclic).
3. **P3 (Polynomial Addition Using Singly Linked Lists)**: Write a complete C11 function `PolyNode *poly_add(const PolyNode *P, const PolyNode *Q)` that computes the sum $R(x) = P(x) + Q(x)$ of two polynomials stored in decreasing exponent order in $O(m + n)$ time, omitting any terms whose coefficients cancel to zero.
4. **P4 (Fast Transpose of a 3-Tuple Sparse Matrix)**: Write a C11 function `void sparse_fast_transpose(const Triplet A[], Triplet B[])` that computes the transpose of a 3-Tuple sparse matrix `A` into `B` in $O(\text{cols} + \text{nnz})$ time.
5. **P5 ($O(1)$ Worst-Case `MinStack` Implementation)**: Design and implement a C11 stack data structure `MinStack` that supports `push(x)`, `pop()`, `peek()`, and `get_min()` (returning the minimum element currently in the stack) such that **all four operations run in worst-case $O(1)$ time**.

*(Complete, compilable C11 solutions and verified outputs for P1–P5 are provided in [Section 10.4](#104-complete-c11-solutions-to-programming-questions-p1p5).)*

---

## 9. Worked Problem-Solving Questions (4 Step-by-Step Exam Problems)

### 9.1 Worked Problem 1: Pointer Trace & In-Place SLL Reversal

**Problem Statement**:
Given a Singly Linked List `head -> [10] -> [20] -> [30] -> [40] -> NULL`:
1. Trace the exact state of pointers `prev`, `curr`, `next`, and the direction of all links after **each iteration** of the standard iterative reversal loop.
2. What happens if a student mistakenly writes `curr->next = prev; next = curr->next;` (reversing those two lines)?

#### Step-by-Step Solution:

**Part 1: Iteration-by-Iteration State Trace**

Initial state before loop:
- `prev = NULL`
- `curr = [10]`
- `next = NULL`
- List structure: `[10] -> [20] -> [30] -> [40] -> NULL`

| Iteration | Step 1: `next = curr->next` | Step 2: `curr->next = prev` | Step 3: `prev = curr` | Step 4: `curr = next` | Resulting Link State after Iteration |
| :---: | :---: | :--- | :---: | :---: | :--- |
| **Init** | `NULL` | — | `NULL` | `[10]` | `NULL` $\quad$ `[10] -> [20] -> [30] -> [40] -> NULL` |
| **Iter 1** | `next = [20]` | `[10]->next = NULL` | `prev = [10]` | `curr = [20]` | `NULL <- [10]` $\quad$ `[20] -> [30] -> [40] -> NULL` |
| **Iter 2** | `next = [30]` | `[20]->next = [10]` | `prev = [20]` | `curr = [30]` | `NULL <- [10] <- [20]` $\quad$ `[30] -> [40] -> NULL` |
| **Iter 3** | `next = [40]` | `[30]->next = [20]` | `prev = [30]` | `curr = [40]` | `NULL <- [10] <- [20] <- [30]` $\quad$ `[40] -> NULL` |
| **Iter 4** | `next = NULL` | `[40]->next = [30]` | `prev = [40]` | `curr = NULL` | `NULL <- [10] <- [20] <- [30] <- [40]` |

After loop terminates (`curr == NULL`):
- Set `head = prev` (`[40]`).
- Final reversed list: `head -> [40] -> [30] -> [20] -> [10] -> NULL`.

**Part 2: Consequence of Swapping `next = curr->next` and `curr->next = prev`**
If `curr->next = prev` executes **first** in Iteration 1:
- `[10]->next` is overwritten with `prev` (`NULL`).
- Then `next = curr->next` sets `next = NULL`!
- Then `prev = [10]` and `curr = next` (`NULL`), terminating the loop after just 1 node!
- Nodes `[20] -> [30] -> [40]` are permanently **leaked** (unreachable), and the list is truncated to a single node `[10] -> NULL`.

---

### 9.2 Worked Problem 2: Sparse Matrix Triplet Representation & Fast Transpose

**Problem Statement**:
Consider the following $4 \times 5$ Sparse Matrix $M$ with $k = 6$ non-zero elements:

$$M = \begin{bmatrix} 0 & 0 & 15 & 0 & 22 \\ 0 & 11 & 0 & 0 & 0 \\ 0 & 0 & 0 & -6 & 0 \\ 91 & 0 & 0 & 0 & 39 \end{bmatrix}$$

1. Write the **3-Tuple (Triplet) Array `A[0..6]`** for $M$.
2. Compute the auxiliary arrays `col_counts[0..4]` and `starting_pos[0..4]` used by the **Fast Transpose** algorithm.
3. Trace the placement of each non-zero element from `A[1..6]` into the transposed 3-Tuple array `B[0..6]`.

#### Step-by-Step Solution:

**Step 1: Original 3-Tuple Array `A[0..6]` (Row-Major Order)**

| Index `i` | `A[i].row` | `A[i].col` | `A[i].val` | Meaning |
| :---: | :---: | :---: | :---: | :--- |
| **0 (Header)** | **4** | **5** | **6** | $4$ rows, $5$ columns, $6$ non-zeros |
| **1** | 0 | 2 | 15 | $M[0][2] = 15$ |
| **2** | 0 | 4 | 22 | $M[0][4] = 22$ |
| **3** | 1 | 1 | 11 | $M[1][1] = 11$ |
| **4** | 2 | 3 | -6 | $M[2][3] = -6$ |
| **5** | 3 | 0 | 91 | $M[3][0] = 91$ |
| **6** | 3 | 4 | 39 | $M[3][4] = 39$ |

**Step 2: Compute `col_counts[]` and `starting_pos[]` for $C = 5$ Columns**
- Count how many times each column index $c \in \{0, 1, 2, 3, 4\}$ appears in `A[1..6].col`:
  - `col 0`: appears once (`A[5]`) $\implies \text{col\_counts}[0] = 1$
  - `col 1`: appears once (`A[3]`) $\implies \text{col\_counts}[1] = 1$
  - `col 2`: appears once (`A[1]`) $\implies \text{col\_counts}[2] = 1$
  - `col 3`: appears once (`A[4]`) $\implies \text{col\_counts}[3] = 1$
  - `col 4`: appears twice (`A[2]`, `A[6]`) $\implies \text{col\_counts}[4] = 2$
- Compute prefix sums `starting_pos[c]` (where `starting_pos[0] = 1` and `starting_pos[c] = starting_pos[c-1] + col_counts[c-1]`):

| Column $c$ | `col 0` | `col 1` | `col 2` | `col 3` | `col 4` |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **`col_counts[c]`** | 1 | 1 | 1 | 1 | 2 |
| **Initial `starting_pos[c]`** | **1** | $1+1 = \mathbf{2}$ | $2+1 = \mathbf{3}$ | $3+1 = \mathbf{4}$ | $4+1 = \mathbf{5}$ |

**Step 3: Single Pass Over `A[1..6]` to Populate Transposed Array `B[0..6]`**
- Header `B[0]` gets `{ row = 5, col = 4, val = 6 }`.
- For `i = 1..6`, let `c = A[i].col` and `pos = starting_pos[c]++`; set `B[pos] = { A[i].col, A[i].row, A[i].val }`:
  1. `i = 1`: `A[1] = (0, 2, 15)` $\to c = 2 \to \text{pos} = \text{starting\_pos}[2]++ = 3 \implies B[3] = (2, 0, 15)$.
  2. `i = 2`: `A[2] = (0, 4, 22)` $\to c = 4 \to \text{pos} = \text{starting\_pos}[4]++ = 5 \implies B[5] = (4, 0, 22)$ (`starting_pos[4]` becomes `6`).
  3. `i = 3`: `A[3] = (1, 1, 11)` $\to c = 1 \to \text{pos} = \text{starting\_pos}[1]++ = 2 \implies B[2] = (1, 1, 11)$.
  4. `i = 4`: `A[4] = (2, 3, -6)` $\to c = 3 \to \text{pos} = \text{starting\_pos}[3]++ = 4 \implies B[4] = (3, 2, -6)$.
  5. `i = 5`: `A[5] = (3, 0, 91)` $\to c = 0 \to \text{pos} = \text{starting\_pos}[0]++ = 1 \implies B[1] = (0, 3, 91)$.
  6. `i = 6`: `A[6] = (3, 4, 39)` $\to c = 4 \to \text{pos} = \text{starting\_pos}[4]++ = 6 \implies B[6] = (4, 3, 39)$.

**Final Transposed 3-Tuple Array `B[0..6]`**:

| Index `j` | `B[j].row` | `B[j].col` | `B[j].val` |
| :---: | :---: | :---: | :---: |
| **0 (Header)** | **5** | **4** | **6** |
| **1** | 0 | 3 | 91 |
| **2** | 1 | 1 | 11 |
| **3** | 2 | 0 | 15 |
| **4** | 3 | 2 | -6 |
| **5** | 4 | 0 | 22 |
| **6** | 4 | 3 | 39 |

---

### 9.3 Worked Problem 3: Polynomial Addition Trace Using Linked Lists

**Problem Statement**:
Given two polynomials represented as sorted Singly Linked Lists of `(coeff, exp)` nodes:
- $P(x) = 5x^4 - 3x^2 + 7x^1 + 9$ $\implies$ `[5|4] -> [-3|2] -> [7|1] -> [9|0] -> NULL`
- $Q(x) = 2x^4 + 3x^2 - 4x^1 + 6x^0$ wait—let's include a mismatched exponent too:
  Let $Q(x) = 2x^5 - 5x^4 + 3x^2 + 4x^1 - 2$ $\implies$ `[2|5] -> [-5|4] -> [3|2] -> [4|1] -> [-2|0] -> NULL`

Trace the two-pointer merge algorithm (`p` scanning $P$, `q` scanning $Q$) step-by-step to construct $R(x) = P(x) + Q(x)$.

#### Step-by-Step Merge Trace:

| Step | Node `p` (`coeff, exp`) | Node `q` (`coeff, exp`) | Exponent Comparison | Action Taken | Result List $R(x)$ Appended Node |
| :---: | :---: | :---: | :---: | :--- | :--- |
| **1** | `(5, 4)` | `(2, 5)` | `p->exp (4) < q->exp (5)` | Copy `q` term; advance `q` | `[2 | 5]` |
| **2** | `(5, 4)` | `(-5, 4)` | `p->exp (4) == q->exp (4)` | Sum $= 5 + (-5) = 0 \implies$ **Cancel! Do NOT append node**; advance `p` and `q` | *(None — zero term omitted)* |
| **3** | `(-3, 2)` | `(3, 2)` | `p->exp (2) == q->exp (2)` | Sum $= -3 + 3 = 0 \implies$ **Cancel! Do NOT append node**; advance `p` and `q` | *(None — zero term omitted)* |
| **4** | `(7, 1)` | `(4, 1)` | `p->exp (1) == q->exp (1)` | Sum $= 7 + 4 = 11 \ne 0 \implies$ Append `(11, 1)`; advance `p` and `q` | `[2 | 5] -> [11 | 1]` |
| **5** | `(9, 0)` | `(-2, 0)` | `p->exp (0) == q->exp (0)` | Sum $= 9 + (-2) = 7 \ne 0 \implies$ Append `(7, 0)`; advance `p` and `q` | `[2 | 5] -> [11 | 1] -> [7 | 0]` |
| **End** | `NULL` | `NULL` | Both lists exhausted | Terminate merge | $R(x) = 2x^5 + 11x + 7$ |

---

### 9.4 Worked Problem 4: Infix-to-Postfix, Infix-to-Prefix, and Stack Evaluation Trace

**Problem Statement**:
Given the arithmetic expression:
$$\text{Infix} = \texttt{12 + 60 / (3 \text{ \^{} } 2 - 3) * 2}$$
1. Convert the expression to **Postfix** using a step-by-step Shunting-Yard stack trace.
2. Write the equivalent **Prefix** expression.
3. Evaluate the resulting **Postfix** expression step-by-step using an operand stack.

#### Step-by-Step Solution:

**Part 1: Infix-to-Postfix Shunting-Yard Trace**

| Step | Token | Action | Operator Stack (Bottom $\to$ Top) | Postfix Output |
| :---: | :---: | :--- | :--- | :--- |
| 1 | `12` | Operand $\to$ output | `[ ]` | `12` |
| 2 | `+` | Push `+` | `+` | `12` |
| 3 | `60` | Operand $\to$ output | `+` | `12 60` |
| 4 | `/` | `prec(/) = 2 > prec(+) = 1` $\to$ push `/` | `+ /` | `12 60` |
| 5 | `(` | Push `(` | `+ / (` | `12 60` |
| 6 | `3` | Operand $\to$ output | `+ / (` | `12 60 3` |
| 7 | `^` | Top is `(` $\to$ push `^` | `+ / ( ^` | `12 60 3` |
| 8 | `2` | Operand $\to$ output | `+ / ( ^` | `12 60 3 2` |
| 9 | `-` | `prec(^) = 3 > prec(-) = 1` $\to$ pop `^`, push `-` | `+ / ( -` | `12 60 3 2 ^` |
| 10 | `3` | Operand $\to$ output | `+ / ( -` | `12 60 3 2 ^ 3` |
| 11 | `)` | Pop until `(`: pop `-`, discard `(` | `+ /` | `12 60 3 2 ^ 3 -` |
| 12 | `*` | `prec(/) = 2 == prec(*) = 2` (Left-Assoc!) $\to$ **pop `/`**, push `*` | `+ *` | `12 60 3 2 ^ 3 - /` |
| 13 | `2` | Operand $\to$ output | `+ *` | `12 60 3 2 ^ 3 - / 2` |
| 14 | `EOF` | Pop remaining `*`, then `+` | `[ ]` | `12 60 3 2 ^ 3 - / 2 * +` |

**Part 2: Equivalent Prefix Expression**
- Grouping: `12 + ((60 / ((3 ^ 2) - 3)) * 2)`
- Prefix: `+ 12 * / 60 - ^ 3 2 3 2`

**Part 3: Postfix Evaluation Trace of `"12 60 3 2 ^ 3 - / 2 * +"`**

| Step | Token | Action (`op2 = pop()`, `op1 = pop()`, push `op1 op op2`) | Operand Stack (Bottom $\to$ Top) |
| :---: | :---: | :--- | :--- |
| 1 | `12` | `push(12)` | `[ 12 ]` |
| 2 | `60` | `push(60)` | `[ 12, 60 ]` |
| 3 | `3` | `push(3)` | `[ 12, 60, 3 ]` |
| 4 | `2` | `push(2)` | `[ 12, 60, 3, 2 ]` |
| 5 | `^` | `op2 = 2, op1 = 3` $\to$ `push(3 ^ 2 = 9)` | `[ 12, 60, 9 ]` |
| 6 | `3` | `push(3)` | `[ 12, 60, 9, 3 ]` |
| 7 | `-` | `op2 = 3, op1 = 9` $\to$ `push(9 - 3 = 6)` | `[ 12, 60, 6 ]` |
| 8 | `/` | `op2 = 6, op1 = 60` $\to$ `push(60 / 6 = 10)` | `[ 12, 10 ]` |
| 9 | `2` | `push(2)` | `[ 12, 10, 2 ]` |
| 10 | `*` | `op2 = 2, op1 = 10` $\to$ `push(10 * 2 = 20)` | `[ 12, 20 ]` |
| 11 | `+` | `op2 = 20, op1 = 12` $\to$ `push(12 + 20 = 32)` | `[ 32 ]` |

**Final Evaluated Value**: **`32`**

---

## 10. Complete Answer Keys with Detailed Explanations

### 10.1 Answers to Short-Answer Questions (Q1–Q10)

- **Answer Q1**: In C, the compiler must know the exact byte size (`sizeof`) of a `struct` at compile time. Including a value member `struct Node next;` inside `struct Node` creates an infinite recursive size recurrence: $\text{sizeof(Node)} = \text{sizeof(int)} + \text{sizeof(Node)} = \infty$ (incomplete type error). In contrast, a pointer `struct Node *next;` has a fixed, known size (8 bytes on 64-bit architectures) regardless of the size of `struct Node`.
- **Answer Q2**: Deleting the `tail` node requires updating the `tail` pointer to point to the **second-to-last node** (the predecessor of `tail`) and setting that predecessor's `next` pointer to `NULL`. Because a Singly Linked List only has forward `next` pointers, finding the predecessor of `tail` requires traversing from `head` through $n - 1$ nodes, taking $\Theta(n)$ time.
- **Answer Q3**: In a Circular Singly Linked List, the `tail` node's `next` pointer points directly to the `head` node (`head == tail->next`). Therefore, maintaining a single `tail` pointer provides $O(1)$ access to **both** the tail (`tail`) and the head (`tail->next`), enabling both `insert_front` and `insert_back` in $O(1)$ time without needing two separate struct pointers.
- **Answer Q4**: An $m \times n$ dense array stores $m \cdot n$ integers. A 3-Tuple representation with $k$ non-zero elements stores $k + 1$ rows (1 header row + $k$ element rows) of $3$ integers each, totaling $3(k + 1)$ integers. Thus, the 3-Tuple representation uses strictly less memory when $3(k + 1) < m \cdot n$, or $k < \frac{m \cdot n}{3} - 1$.
- **Answer Q5**: Each node has at least 1 level-0 pointer, and is promoted to each successive level independently with probability $p = 1/2$. By the geometric series, the expected number of pointers per node is $\sum_{i=0}^{\infty} (1/2)^i = \frac{1}{1 - 1/2} = 2$. Because level heights are chosen randomly upon node insertion using a local coin flip without depending on global subtree balance factors, insertions and deletions only rewire local predecessor pointers at each level—no tree rotations are ever required.
- **Answer Q6**: A Stack requires both `push` and `pop` to execute in $O(1)$ time at the same end (`top`). At the `head` of a Singly Linked List, both prepending a new node (`push`) and removing the first node (`pop`) take $O(1)$ worst-case time. If `top` were placed at the `tail`, `pop` would require finding the predecessor of `tail`, degrading `pop` to $\Theta(n)$ time.
- **Answer Q7**: If capacity doubles at $n = C$ and halves immediately at $n = C / 2$, then a sequence of alternating `push` and `pop` operations right at the boundary ($n = C/2 \to C/2 + 1 \to C/2 \to \dots$) triggers an $O(n)$ `realloc` copy on **every single operation**. Shrinking only when `size <= capacity / 4` (25% full) guarantees that immediately after halving, the array is 50% full, requiring at least $C'/2$ insertions or $C'/4$ deletions before another resize can occur ($O(1)$ amortized cost).
- **Answer Q8**:
  1. **Closer on Empty Stack** (e.g., `"())"`): A closing bracket is encountered when the stack is currently empty (`isEmpty == true`).
  2. **Mismatched Pair** (e.g., `"[(])"`): A closing bracket is encountered, and the opening bracket popped from the stack belongs to a different bracket type (`'('` vs. `']'`).
  3. **Unclosed Opener at End** (e.g., `"((("`): The end of the input string is reached, and the stack is still non-empty (`isEmpty == false`).
- **Answer Q9**:
  - Given Infix: `A ^ B ^ C - D + E / F / G`
  - Step 1 (Parenthesize by precedence & associativity): `^` is right-associative $\to$ `(A ^ (B ^ C))`; `/` is left-associative $\to$ `((E / F) / G)`; `-` and `+` are left-associative $\to$ `(((A ^ (B ^ C)) - D) + ((E / F) / G))`.
  - **Postfix**: `A B C ^ ^ D - E F / G / +`
  - **Prefix**: `+ - ^ A ^ B C D / / E F G`
- **Answer Q10**: Naive `fib(n)` makes two recursive calls (`fib(n-1)` and `fib(n-2)`), generating a binary recursion tree with $\Theta(\phi^n) \approx O(2^n)$ total nodes (time). However, C evaluates `fib(n-1)` **completely** and pops all of its stack frames before calling `fib(n-2)`. Therefore, at any single instant in time, only a single root-to-leaf path of length at most $n$ is active on the call stack, requiring $\Theta(n)$ space.

---

### 10.2 Answers to Conceptual Questions (C1–C8)

- **Answer C1 (`Node **curr` Double-Pointer Technique)**:
  In a standard Singly Linked List, `head` is a `Node *` variable outside the list, whereas every subsequent link is a `Node *` field (`prev->next`) inside a node. Treating them separately requires an `if (curr == head)` branch. By maintaining a pointer `Node **indir = &head`, `*indir` aliases the exact pointer variable that points to the current node (initially `head` itself, and subsequently `prev->next` via `indir = &((*indir)->next)`). To delete the current node `*indir`, we simply write:
  ```c
  Node *victim = *indir;
  *indir = victim->next; /* Updates either 'head' or 'prev->next' uniformly! */
  free(victim);
  ```
- **Answer C2 ($O(1)$ SLL Node Deletion Trick Limitations)**:
  1. **Fails at the Tail Node**: If `p` is the last node (`p->next == NULL`), there is no successor node from which to copy data, and we cannot set the predecessor's `next` pointer to `NULL` without traversing from `head`.
  2. **Invalidates External Pointers**: This trick does not actually delete node `p`—it mutates `p` and frees `p->next`! If any other part of the program holds a pointer to `p->next`, that pointer silently becomes a **dangling pointer**, and any pointer expecting `p` to be gone still points to `p` (now holding `p->next`'s old value).
- **Answer C3 (Fast Transpose $O(C + k)$ Mechanics)**:
  In a 3-Tuple matrix `A[1..k]`, entries are sorted by `(row, col)`. In the transpose `B[1..k]`, entries must be sorted by `(col, row)` of `A`. Because `A` is already ordered by row, if we bucket `A[1..k]` stably by column `c = A[i].col`, elements within each column bucket automatically appear in increasing row order!
  - Pass 1 ($O(C)$): Zero-initialize `col_counts[0..C-1]`.
  - Pass 2 ($O(k)$): Count occurrences of each column `A[i].col`.
  - Pass 3 ($O(C)$): Compute cumulative prefix sums `starting_pos[c]`, giving the exact starting index in `B` where column `c`'s elements belong.
  - Pass 4 ($O(k)$): Place each `A[i]` directly at `B[starting_pos[A[i].col]++]`.
  - Total Time: $O(C + k)$, avoiding the $O(C \cdot k)$ nested loop.
- **Answer C4 (Skip List Express Lanes)**:
  In a sorted array, binary search jumps to the midpoint in $O(1)$ time via index arithmetic, but inserting an element requires shifting $O(n)$ contiguous elements. A Skip List builds a hierarchy of sparse linked lists ($L_0, L_1, \dots, L_{\max}$) where level $L_k$ skips roughly $2^k$ nodes. Searching starts at the top level $L_{\max}$ and drops down a level whenever the next key exceeds the target, halving the search interval like binary search in $O(\log n)$ expected steps. Because each level is just a singly linked list and node heights are chosen via local random coin flips ($p=1/2$), inserting or deleting a node only updates the `forward[lvl]` pointers of the immediate predecessors along the search path (`update[0..lvl]`), requiring $O(\log n)$ time with zero global array shifts or tree rotations.
- **Answer C5 (Polynomial Multiplication Complexity)**:
  When multiplying $P(x)$ ($m$ terms) and $Q(x)$ ($n$ terms), there are $m \cdot n$ pair products `(p->coeff * q->coeff, p->exp + q->exp)`. If we insert each of the $m \cdot n$ product terms one-by-one into a result linked list of length up to $O(m \cdot n)$ via linear scan, the $i$-th insertion scans up to $i$ nodes, taking $\sum_{i=1}^{mn} O(i) = O(m^2 n^2)$ time in the worst case!
  - **Optimization 1 (Row-wise Merge)**: For each of the $m$ terms of $P$, multiply it by all $n$ terms of $Q$ (producing a sorted list of length $n$ in $O(n)$ time) and merge it into the accumulator list using `poly_add`, reducing time to $O(m^2 n)$ (or $O(mn \log m)$ with divide-and-conquer pairwise merging).
  - **Optimization 2 (Array/Sort)**: Collect all $m \cdot n$ terms in an array, sort by exponent in $O(mn \log(mn))$ time, and combine adjacent equal exponents in a single $O(mn)$ pass.
- **Answer C6 (OS Call Stack Overflow vs. Heap Stack)**:
  The Operating System allocates a fixed-size contiguous virtual memory region for each thread's hardware call stack (typically 1 MB on Windows, 8 MB on Linux). At 32–64 bytes per stack frame, $200,000$ nested recursive calls consume $6.4\text{–}12.8$ MB, crossing into the unmapped **guard page** below the stack and triggering a fatal `SIGSEGV` / `STATUS_STACK_OVERFLOW`—even when gigabytes of **Heap** memory remain unused. Replacing recursion with an explicit `DynamicStack` allocated via `malloc`/`realloc` places the activation frames in the **Heap segment**, which can grow up to available system RAM (gigabytes) and shrinks cleanly when done.
- **Answer C7 (Why Reversing an Infix String Flips Associativity Tie-Breaking)**:
  Consider left-associative subtraction `A - B - C`, which means `(A - B) - C` and has Prefix form `- - A B C`.
  When we reverse `A - B - C`, we scan `C - B - A` from left to right (or equivalently scan `A - B - C` from right to left):
  - The first `-` encountered (between `B` and `C`) is the **outer** subtraction that must be evaluated **last**, so in the reversed output it must appear **last** (so that after the final reversal, it appears **first** at the front of `- - A B C`).
  - Therefore, when the second `-` (between `A` and `B`) is encountered during the reversed scan, it must **NOT** pop the first `-` from the stack; instead, it must be pushed on top of the first `-` so that the `A - B` operator is output *before* the `(A - B) - C` operator (`C B A - -` $\xrightarrow{\text{reverse}}$ `- - A B C`).
  - Conversely, for right-associative `A ^ B ^ C` (`A ^ (B ^ C)`, Prefix `^ A ^ B C`), during the right-to-left scan the first `^` encountered (between `B` and `C`) is the **inner** operator and **must** be popped when the second `^` arrives (`C B ^ A ^` $\xrightarrow{\text{reverse}}$ `^ A ^ B C`).
- **Answer C8 (Amortized Analysis of Dynamic Stack Doubling)**:
  - **Aggregate Method**: Suppose $C_0 = 1$ and $n = 2^k + 1$ pushes are performed. Resizes occur when the stack size reaches $1, 2, 4, 8, \dots, 2^k$. The $i$-th resize copies $2^{i-1}$ existing elements. Across $n$ pushes, the total work is $n$ direct array writes plus the sum of resize copies:
    $$\text{Total Cost} = n + \sum_{i=0}^{k} 2^i = n + (2^{k+1} - 1) = n + 2(n - 1) - 1 = 3n - 3 = O(n)$$
    Dividing by $n$ operations gives an **amortized cost of $\frac{3n - 3}{n} < 3 = O(1)$ per `push`**.
  - **Accounting (Banker's) Method**: Charge $\$3$ per `push`: $\$1$ pays for writing the new element itself, $\$1$ is stored as credit on the newly inserted element, and $\$1$ is stored as credit on one of the older elements in the first half of the array. Whenever capacity $C$ fills up after $C/2$ new insertions since the last doubling, all $C$ elements hold $\$1$ of credit, paying for all $C$ element copies during `realloc` with zero debt!

---

### 10.3 Solutions to Algorithm-Writing Questions (A1–A5)

#### Solution A1: Pairwise Swap of Nodes in a Singly Linked List by Rewiring Links
```text
Algorithm PairwiseSwapLinks(head):
    if head == NULL or head.next == NULL then
        return head

    dummy <- AllocateNode(0)
    dummy.next <- head
    prev <- dummy

    while prev.next != NULL and prev.next.next != NULL do
        first  <- prev.next
        second <- first.next

        // Rewire links to swap 'first' and 'second': prev -> second -> first -> (second.next)
        first.next  <- second.next
        second.next <- first
        prev.next   <- second

        // Advance prev by two nodes for the next pair
        prev <- first

    new_head <- dummy.next
    FreeNode(dummy)
    return new_head
```

#### Solution A2: Delete All Occurrences of Key `K` in a Doubly Linked List
```text
Algorithm DeleteAllKeyDLL(head_ref, tail_ref, K):
    curr <- *head_ref
    while curr != NULL do
        next_node <- curr.next
        if curr.data == K then
            // Rewire predecessor link
            if curr.prev != NULL then
                curr.prev.next <- curr.next
            else
                *head_ref <- curr.next    // Deleting current head

            // Rewire successor link
            if curr.next != NULL then
                curr.next.prev <- curr.prev
            else
                *tail_ref <- curr.prev    // Deleting current tail

            FreeNode(curr)
        curr <- next_node
```

#### Solution A3: Split a Circular Linked List into Two Halves (`Tortoise & Hare`)
```text
Algorithm SplitCircularList(head, head1_ref, head2_ref):
    if head == NULL or head.next == head then
        *head1_ref <- head
        *head2_ref <- NULL
        return

    slow <- head
    fast <- head

    // Advance fast by 2 and slow by 1 until fast reaches tail or second-to-last node
    while fast.next != head and fast.next.next != head do
        fast <- fast.next.next
        slow <- slow.next

    // If even number of nodes, move fast one step to reach the actual tail node
    if fast.next.next == head then
        fast <- fast.next

    // Set heads of both circular halves
    *head1_ref <- head
    *head2_ref <- slow.next

    // Close both rings into valid Circular Linked Lists
    fast.next <- *head2_ref    // Close second half ring
    slow.next <- *head1_ref    // Close first half ring
```

#### Solution A4: Sort a Stack Using Only One Temporary Stack
```text
Algorithm SortStack(S):
    T <- CreateEmptyStack()    // Holds sorted elements (largest on top)
    while not IsEmpty(S) do
        curr <- Pop(S)
        // Move elements from T back to S that are strictly greater than 'curr'
        while not IsEmpty(T) and Peek(T) > curr do
            Push(S, Pop(T))
        Push(T, curr)

    // Optional: If sorted order with smallest on top in S is desired, or copy T back:
    // Here T has largest on top; to put largest on top of S, transfer via S or return T.
    return T
```

#### Solution A5: Single-Pass Direct Infix Evaluation Using Two Stacks
```text
Algorithm ApplyTopOperator(ValStack, OpStack):
    op  <- Pop(OpStack)
    op2 <- Pop(ValStack)
    op1 <- Pop(ValStack)
    Push(ValStack, Compute(op1, op, op2))

Algorithm EvaluateInfixDirect(expr):
    ValStack <- CreateEmptyStack()
    OpStack  <- CreateEmptyStack()

    for each token T in expr (left to right) do
        if T is Number then
            Push(ValStack, ToNumber(T))
        else if T == '(' then
            Push(OpStack, '(')
        else if T == ')' then
            while Peek(OpStack) != '(' do
                ApplyTopOperator(ValStack, OpStack)
            Pop(OpStack)   // Discard '('
        else if T is BinaryOperator then
            while not IsEmpty(OpStack) and Peek(OpStack) != '(' and
                  (Prec(Peek(OpStack)) > Prec(T) or
                   (Prec(Peek(OpStack)) == Prec(T) and IsLeftAssociative(T))) do
                ApplyTopOperator(ValStack, OpStack)
            Push(OpStack, T)

    while not IsEmpty(OpStack) do
        ApplyTopOperator(ValStack, OpStack)

    return Pop(ValStack)
```

---

### 10.4 Complete C11 Solutions to Programming Questions (P1–P5)

The following single, self-contained C11 program implements and verifies all five programming questions (**P1–P5**):
1. **P1**: Iterative & Recursive In-Place SLL Reversal.
2. **P2**: Floyd's Cycle Detection & Cycle-Entry Node Identification.
3. **P3**: Polynomial Addition on Sorted Linked Lists (with zero-cancellation).
4. **P4**: $O(\text{cols} + \text{nnz})$ Fast Transpose of a 3-Tuple Sparse Matrix.
5. **P5**: Worst-Case $O(1)$ `MinStack` supporting `push`, `pop`, `peek`, and `get_min`.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

/* =========================================================================
 * P1 & P2: SINGLY LINKED LIST REVERSAL AND FLOYD'S CYCLE DETECTION
 * ========================================================================= */
typedef struct Node {
    int data;
    struct Node *next;
} Node;

Node *create_node(int data) {
    Node *n = (Node *)malloc(sizeof(Node));
    if (n != NULL) {
        n->data = data;
        n->next = NULL;
    }
    return n;
}

/* P1(a): Iterative In-Place SLL Reversal — O(n) time, O(1) space */
Node *reverse_sll_iterative(Node *head) {
    Node *prev = NULL;
    Node *curr = head;
    while (curr != NULL) {
        Node *next_node = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next_node;
    }
    return prev;
}

/* P1(b): Recursive In-Place SLL Reversal — O(n) time, O(n) stack space */
Node *reverse_sll_recursive(Node *head) {
    if (head == NULL || head->next == NULL) {
        return head;
    }
    Node *new_head = reverse_sll_recursive(head->next);
    head->next->next = head;
    head->next = NULL;
    return new_head;
}

void print_sll(const Node *head) {
    for (const Node *c = head; c != NULL; c = c->next) {
        printf("[%d] -> ", c->data);
    }
    printf("NULL\n");
}

void free_sll(Node *head) {
    while (head != NULL) {
        Node *tmp = head;
        head = head->next;
        free(tmp);
    }
}

/* P2: Floyd's Cycle-Finding Algorithm (returns cycle-entry node or NULL) */
Node *detect_and_find_cycle_start(Node *head) {
    Node *slow = head;
    Node *fast = head;

    while (fast != NULL && fast->next != NULL) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) {
            /* Cycle detected! Reset slow to head and advance both by 1 step */
            slow = head;
            while (slow != fast) {
                slow = slow->next;
                fast = fast->next;
            }
            return slow; /* Exact cycle entry node */
        }
    }
    return NULL;
}

/* =========================================================================
 * P3: POLYNOMIAL ADDITION USING SINGLY LINKED LISTS — O(m + n) TIME
 * ========================================================================= */
typedef struct PolyNode {
    int coeff;
    int exp;
    struct PolyNode *next;
} PolyNode;

void poly_append_term(PolyNode **head_ref, PolyNode **tail_ref, int coeff, int exp) {
    if (coeff == 0) return;
    PolyNode *node = (PolyNode *)malloc(sizeof(PolyNode));
    if (node == NULL) return;
    node->coeff = coeff;
    node->exp   = exp;
    node->next  = NULL;
    if (*head_ref == NULL) {
        *head_ref = *tail_ref = node;
    } else {
        (*tail_ref)->next = node;
        *tail_ref = node;
    }
}

PolyNode *poly_add(const PolyNode *P, const PolyNode *Q) {
    PolyNode *res_head = NULL, *res_tail = NULL;

    while (P != NULL && Q != NULL) {
        if (P->exp > Q->exp) {
            poly_append_term(&res_head, &res_tail, P->coeff, P->exp);
            P = P->next;
        } else if (Q->exp > P->exp) {
            poly_append_term(&res_head, &res_tail, Q->coeff, Q->exp);
            Q = Q->next;
        } else {
            int sum_coeff = P->coeff + Q->coeff;
            if (sum_coeff != 0) {
                poly_append_term(&res_head, &res_tail, sum_coeff, P->exp);
            }
            P = P->next;
            Q = Q->next;
        }
    }
    while (P != NULL) {
        poly_append_term(&res_head, &res_tail, P->coeff, P->exp);
        P = P->next;
    }
    while (Q != NULL) {
        poly_append_term(&res_head, &res_tail, Q->coeff, Q->exp);
        Q = Q->next;
    }
    return res_head;
}

void poly_print(const PolyNode *P) {
    if (P == NULL) {
        printf("0\n");
        return;
    }
    for (const PolyNode *c = P; c != NULL; c = c->next) {
        if (c != P && c->coeff > 0) printf(" + ");
        else if (c != P && c->coeff < 0) printf(" - ");
        else if (c == P && c->coeff < 0) printf("-");
        int abs_c = (c->coeff < 0) ? -c->coeff : c->coeff;
        printf("%dx^%d", abs_c, c->exp);
    }
    printf("\n");
}

void poly_free(PolyNode *P) {
    while (P != NULL) {
        PolyNode *tmp = P;
        P = P->next;
        free(tmp);
    }
}

/* =========================================================================
 * P4: FAST TRANSPOSE OF A 3-TUPLE SPARSE MATRIX — O(cols + nnz) TIME
 * ========================================================================= */
#define MAX_COLS 32

typedef struct {
    int row;
    int col;
    int val;
} Triplet;

void sparse_fast_transpose(const Triplet A[], Triplet B[]) {
    int num_rows = A[0].row;
    int num_cols = A[0].col;
    int num_nnz  = A[0].val;

    B[0].row = num_cols;
    B[0].col = num_rows;
    B[0].val = num_nnz;

    if (num_nnz <= 0) return;

    int col_counts[MAX_COLS]   = {0};
    int starting_pos[MAX_COLS] = {0};

    /* Step 1: Count non-zeros in each column of A */
    for (int i = 1; i <= num_nnz; i++) {
        col_counts[A[i].col]++;
    }

    /* Step 2: Compute starting output index for each column of A (row of B) */
    starting_pos[0] = 1;
    for (int c = 1; c < num_cols; c++) {
        starting_pos[c] = starting_pos[c - 1] + col_counts[c - 1];
    }

    /* Step 3: Place each element directly into its transposed position */
    for (int i = 1; i <= num_nnz; i++) {
        int dest_idx = starting_pos[A[i].col]++;
        B[dest_idx].row = A[i].col;
        B[dest_idx].col = A[i].row;
        B[dest_idx].val = A[i].val;
    }
}

/* =========================================================================
 * P5: O(1) WORST-CASE MIN-STACK IMPLEMENTATION
 * ========================================================================= */
#define MAX_STACK 64

typedef struct {
    int val;
    int current_min; /* Minimum of all elements at or below this slot */
} MinStackEntry;

typedef struct {
    MinStackEntry data[MAX_STACK];
    int top;
} MinStack;

void min_stack_init(MinStack *ms) {
    ms->top = -1;
}

bool min_stack_push(MinStack *ms, int val) {
    if (ms->top >= MAX_STACK - 1) return false;
    int new_min = (ms->top == -1 || val < ms->data[ms->top].current_min)
                  ? val
                  : ms->data[ms->top].current_min;
    ms->top++;
    ms->data[ms->top].val         = val;
    ms->data[ms->top].current_min = new_min;
    return true;
}

bool min_stack_pop(MinStack *ms, int *out_val) {
    if (ms->top == -1) return false;
    *out_val = ms->data[(ms->top)--].val;
    return true;
}

bool min_stack_get_min(const MinStack *ms, int *out_min) {
    if (ms->top == -1) return false;
    *out_min = ms->data[ms->top].current_min;
    return true;
}

int main(void) {
    /* --- Test P1: SLL Reversal --- */
    printf("=== P1: Iterative & Recursive SLL Reversal ===\n");
    Node *head = create_node(10);
    head->next = create_node(20);
    head->next->next = create_node(30);
    head->next->next->next = create_node(40);
    printf("  Original : "); print_sll(head);
    head = reverse_sll_iterative(head);
    printf("  Iter Rev : "); print_sll(head);
    head = reverse_sll_recursive(head);
    printf("  Rec  Rev : "); print_sll(head);

    /* --- Test P2: Floyd's Cycle Detection --- */
    printf("\n=== P2: Floyd's Cycle Detection ===\n");
    head->next->next->next->next = head->next; /* Create cycle: 40 -> 20 */
    Node *entry = detect_and_find_cycle_start(head);
    printf("  Cycle detected! Entry node value = %d\n", entry ? entry->data : -1);
    head->next->next->next->next = NULL; /* Break cycle before freeing */
    free_sll(head);

    /* --- Test P3: Polynomial Addition --- */
    printf("\n=== P3: Polynomial Addition ===\n");
    PolyNode *P = NULL, *P_tail = NULL;
    PolyNode *Q = NULL, *Q_tail = NULL;
    poly_append_term(&P, &P_tail, 5, 4);
    poly_append_term(&P, &P_tail, -3, 2);
    poly_append_term(&P, &P_tail, 7, 1);
    poly_append_term(&P, &P_tail, 9, 0);

    poly_append_term(&Q, &Q_tail, 2, 5);
    poly_append_term(&Q, &Q_tail, -5, 4);
    poly_append_term(&Q, &Q_tail, 3, 2);
    poly_append_term(&Q, &Q_tail, 4, 1);
    poly_append_term(&Q, &Q_tail, -2, 0);

    PolyNode *R = poly_add(P, Q);
    printf("  P(x) = "); poly_print(P);
    printf("  Q(x) = "); poly_print(Q);
    printf("  R(x) = "); poly_print(R);
    poly_free(P); poly_free(Q); poly_free(R);

    /* --- Test P4: Fast Transpose --- */
    printf("\n=== P4: Sparse Matrix Fast Transpose ===\n");
    Triplet A[7] = {
        {4, 5, 6},
        {0, 2, 15}, {0, 4, 22}, {1, 1, 11},
        {2, 3, -6}, {3, 0, 91}, {3, 4, 39}
    };
    Triplet B[7];
    sparse_fast_transpose(A, B);
    printf("  Transposed Header: rows=%d, cols=%d, nnz=%d\n", B[0].row, B[0].col, B[0].val);
    for (int i = 1; i <= B[0].val; i++) {
        printf("  B[%d] = (%d, %d, %d)\n", i, B[i].row, B[i].col, B[i].val);
    }

    /* --- Test P5: O(1) MinStack --- */
    printf("\n=== P5: O(1) Worst-Case MinStack ===\n");
    MinStack ms;
    min_stack_init(&ms);
    int vals[] = {30, 10, 20, 5, 15};
    for (int i = 0; i < 5; i++) {
        int cur_min = 0;
        min_stack_push(&ms, vals[i]);
        min_stack_get_min(&ms, &cur_min);
        printf("  Pushed %2d -> current O(1) min = %d\n", vals[i], cur_min);
    }
    int popped = 0, cur_min = 0;
    min_stack_pop(&ms, &popped);
    min_stack_pop(&ms, &popped);
    min_stack_get_min(&ms, &cur_min);
    printf("  After popping 15 and 5 -> current O(1) min = %d\n", cur_min);

    return EXIT_SUCCESS;
}
```

#### Program Output:

```text
=== P1: Iterative & Recursive SLL Reversal ===
  Original : [10] -> [20] -> [30] -> [40] -> NULL
  Iter Rev : [40] -> [30] -> [20] -> [10] -> NULL
  Rec  Rev : [10] -> [20] -> [30] -> [40] -> NULL

=== P2: Floyd's Cycle Detection ===
  Cycle detected! Entry node value = 20

=== P3: Polynomial Addition ===
  P(x) = 5x^4 - 3x^2 + 7x^1 + 9x^0
  Q(x) = 2x^5 - 5x^4 + 3x^2 + 4x^1 - 2x^0
  R(x) = 2x^5 + 11x^1 + 7x^0

=== P4: Sparse Matrix Fast Transpose ===
  Transposed Header: rows=5, cols=4, nnz=6
  B[1] = (0, 3, 91)
  B[2] = (1, 1, 11)
  B[3] = (2, 0, 15)
  B[4] = (3, 2, -6)
  B[5] = (4, 0, 22)
  B[6] = (4, 3, 39)

=== P5: O(1) Worst-Case MinStack ===
  Pushed 30 -> current O(1) min = 30
  Pushed 10 -> current O(1) min = 10
  Pushed 20 -> current O(1) min = 10
  Pushed  5 -> current O(1) min = 5
  Pushed 15 -> current O(1) min = 5
  After popping 15 and 5 -> current O(1) min = 10
```

---

## Unit 1 Topic Navigation

- [01 — C Structures, Pointers, and Dynamic Memory](01-C-Structures-Pointers-and-Memory.md)
- [02 — Singly Linked Lists](02-Singly-Linked-Lists.md)
- [03 — Doubly Linked Lists](03-Doubly-Linked-Lists.md)
- [04 — Circular Linked Lists](04-Circular-Linked-Lists.md)
- [05 — Multilists, Sparse Matrices, and Skip Lists](05-Multilists-Sparse-Matrices-and-Skip-Lists.md)
- [06 — Polynomial Representation and Operations](06-Polynomial-Representation-and-Operations.md)
- [07 — Stacks: Array and Linked-List Implementations](07-Stacks-Array-and-Linked-List.md)
- [08 — Stack Applications and Recursion](08-Stack-Applications-and-Recursion.md)
- [09 — Expression Conversion and Evaluation](09-Expression-Conversion-and-Evaluation.md)
