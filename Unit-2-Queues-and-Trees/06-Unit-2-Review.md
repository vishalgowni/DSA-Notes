# Unit 2: Comprehensive Review and Exam Preparation Guide

---

## Table of Contents

1. [Important Definitions Glossary (Complete Unit 2)](#1-important-definitions-glossary-complete-unit-2)
   - 1.1 [Queues, Deques, and Priority Queues (Unit 2A)](#11-queues-deques-and-priority-queues-unit-2a)
   - 1.2 [Queue Applications: CPU Scheduling and Josephus Problem (Unit 2B)](#12-queue-applications-cpu-scheduling-and-josephus-problem-unit-2b)
   - 1.3 [N-Ary Trees, Binary Trees, Forests, and Traversals (Unit 2C)](#13-n-ary-trees-binary-trees-forests-and-traversals-unit-2c)
2. [Essential Diagrams and Mathematical Formulas](#2-essential-diagrams-and-mathematical-formulas)
   - 2.1 [Circular Queue and Circular Deque Modular Arithmetic](#21-circular-queue-and-circular-deque-modular-arithmetic)
   - 2.2 [CPU Scheduling Metrics and Josephus Recurrence](#22-cpu-scheduling-metrics-and-josephus-recurrence)
   - 2.3 [Tree and Forest Mathematical Formulas and LCRS Mapping Rules](#23-tree-and-forest-mathematical-formulas-and-lcrs-mapping-rules)
3. [Commonly Confused Concepts (Side-by-Side Disambiguation)](#3-commonly-confused-concepts-side-by-side-disambiguation)
4. [Consolidated Operation-Complexity Table for Unit 2](#4-consolidated-operation-complexity-table-for-unit-2)
5. [Short-Answer Questions (10 Questions)](#5-short-answer-questions-10-questions)
6. [Conceptual and Deep-Thinking Questions (8 Questions)](#6-conceptual-and-deep-thinking-questions-8-questions)
7. [Algorithm-Writing Questions (5 Exam-Style Pseudocode Problems)](#7-algorithm-writing-questions-5-exam-style-pseudocode-problems)
8. [C Programming Questions (5 Coding Problems)](#8-c-programming-questions-5-coding-problems)
9. [Worked Problem-Solving Questions (5 Step-by-Step Traces)](#9-worked-problem-solving-questions-5-step-by-step-traces)
10. [Complete Answer Keys with Detailed Explanations](#10-complete-answer-keys-with-detailed-explanations)
    - 10.1 [Answers to Short-Answer Questions](#101-answers-to-short-answer-questions)
    - 10.2 [Answers to Conceptual Questions](#102-answers-to-conceptual-questions)
    - 10.3 [Solutions to Algorithm-Writing Questions](#103-solutions-to-algorithm-writing-questions)
    - 10.4 [Complete C11 Solutions to Programming Questions](#104-complete-c11-solutions-to-programming-questions)
    - 10.5 [Complete Step-by-Step Solutions to Worked Problems](#105-complete-step-by-step-solutions-to-worked-problems)

---

## 1. Important Definitions Glossary (Complete Unit 2)

### 1.1 Queues, Deques, and Priority Queues (Unit 2A)

- **Queue (FIFO — First-In, First-Out):** An ordered linear data structure in which insertions (**enqueue**) occur exclusively at one end called the **rear (tail)** and deletions (**dequeue**) occur exclusively at the opposite end called the **front (head)**.
- **False Overflow (in Linear Array Queue):** The pathological state in a non-circular array queue where `rear == CAPACITY - 1` (preventing further insertions) even though `front > 0` leaves unoccupied slots at the beginning of the array.
- **Circular Queue (Ring Buffer):** An array-backed queue that logically connects index `CAPACITY - 1` back to index `0` using modular arithmetic `(index + 1) % CAPACITY`, eliminating false overflow and achieving $O(1)$ amortized/worst-case enqueue and dequeue without element shifting.
- **Double-Ended Queue (Deque):** A generalized linear structure that supports $O(1)$ insertion and deletion at **both** the `front` and `rear` ends (`insert_front`, `insert_rear`, `delete_front`, `delete_rear`).
  - **Input-Restricted Deque:** Allows insertion at **only one end** (`rear`), while permitting deletion from **both ends** (`front` and `rear`).
  - **Output-Restricted Deque:** Allows deletion from **only one end** (`front`), while permitting insertion at **both ends** (`front` and `rear`).
- **Priority Queue (PQ):** An abstract data type in which each element is associated with a numeric **priority key**, and `extract_min` (or `extract_max`) always removes the element with the highest priority (using **FIFO arrival order** to break ties among equal priorities).

### 1.2 Queue Applications: CPU Scheduling and Josephus Problem (Unit 2B)

- **First-Come, First-Served (FCFS):** Non-preemptive CPU scheduling algorithm backed by a standard FIFO queue; suffers from the **Convoy Effect** when short jobs wait behind a long CPU-bound job.
- **Round-Robin (RR) Scheduling:** Preemptive time-sliced scheduling algorithm backed by a circular FIFO ready queue with a fixed **Time Quantum ($q$)**; guarantees fair response times and prevents starvation.
- **Priority / Shortest-Job-First (SJF / SRTF) Scheduling:** Schedules processes using a Priority Queue keyed by priority number or remaining burst time; suffers from **Starvation (Indefinite Blocking)** of low-priority processes, solved via **Aging** (gradually boosting priority as wait time increases).
- **Josephus Problem ($J(n, k)$):** A mathematical elimination game in which $n$ people stand in a circle ($1 \dots n$) and every $k^{\text{th}}$ remaining person is eliminated until a single survivor remains. Naturally simulated in $O(n \cdot k)$ time using a Circular Queue.

### 1.3 N-Ary Trees, Binary Trees, Forests, and Traversals (Unit 2C)

- **Rooted Tree $T = (V, E, r)$:** A connected, acyclic hierarchical structure of $n = |V|$ vertices and $|E| = n - 1$ edges with a unique root $r$ (in-degree $0$) and where every other vertex has in-degree $1$ (a unique parent).
- **Degree of a Node ($\deg(u)$) vs. Degree of a Tree ($\Delta(T)$):** $\deg(u)$ is the number of children of node $u$; $\Delta(T) = \max_{u \in V} \deg(u)$. Leaves have $\deg(u) = 0$; internal nodes have $\deg(u) \ge 1$.
- **Depth ($d(u)$) vs. Height ($h(u)$):**
  - **Depth $d(u)$:** Number of edges on the unique path from the **root** down to $u$ ($d(\text{root}) = 0$).
  - **Height $h(u)$:** Number of edges on the **longest downward path** from $u$ to a leaf in $T_u$ ($h(\text{leaf}) = 0$, $h(\emptyset) = -1$ in 0-based edge convention; $H = h + 1$ in 1-based node convention).
- **Binary Tree:** An ordered tree where each node has at most two children, explicitly distinguished as **left child** and **right child** even when only one child is present.
  - **Full (Strict / Proper) Binary Tree:** Every node has either $0$ or $2$ children ($I_1 = 0$).
  - **Complete Binary Tree:** Levels $0 \dots h-1$ are completely full, and nodes at level $h$ are packed as far left as possible.
  - **Perfect Binary Tree:** Every internal node has $2$ children AND all leaves are at the same depth $h$ ($n = 2^{h+1} - 1$).
  - **Balanced Binary Tree:** For every node $u$, $|h(u.\text{left}) - h(u.\text{right})| \le 1$.
  - **Degenerate (Skewed) Binary Tree:** Every internal node has $1$ child ($h = n - 1$).
- **Binary Search Tree (BST):** A binary tree satisfying the global ordering invariant: for every node $u$, all keys in $T_{\text{left}}(u)$ are strictly $< \text{key}(u)$ and all keys in $T_{\text{right}}(u)$ are strictly $> \text{key}(u)$.
- **Forest ($\mathcal{F} = \{T_1, \dots, T_k\}$):** An ordered collection of $k \ge 0$ disjoint rooted trees with $|E| = |V| - k$.
- **Left-Child Right-Sibling (LCRS / Knuth) Transform:** A bijection mapping any $N$-ary tree or forest to a binary tree where `bin->left` points to the **first child** and `bin->right` points to the **next sibling** (or the next tree root $T_{i+1}$ in a forest).
- **Tree Traversals:**
  - **Preorder ($\text{NLR}$):** $\text{Root} \to \text{Left} \to \text{Right}$.
  - **Inorder ($\text{LNR}$):** $\text{Left} \to \text{Root} \to \text{Right}$ (produces sorted output on a BST; equals N-ary Postorder on an LCRS binary tree).
  - **Postorder ($\text{LRN}$):** $\text{Left} \to \text{Right} \to \text{Root}$ (mandatory for `free()` and bottom-up aggregations).
  - **Level-Order (BFS):** Visits depth $d = 0, 1, \dots, h$ from left to right using a FIFO Queue.

---

## 2. Essential Diagrams and Mathematical Formulas

### 2.1 Circular Queue and Circular Deque Modular Arithmetic

Let $M = \text{CAPACITY}$ be the size of the backing array:

```text
          Circular Backward Wrap                          Circular Forward Wrap
   front = (front - 1 + M) % M                     rear = (rear + 1) % M
          +-----------------------------------------------+
          |                                               |
          v                                               v
Index:  [ 0 ]     [ 1 ]     [ 2 ]     ...     [ M - 2 ] [ M - 1 ]
```

| Structure & Strategy | Empty Condition | Full Condition | Advance Forward (`+1`) | Move Backward (`-1`) | Current Element Count |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Circular Queue (Explicit `size`)** | `size == 0` | `size == M` | `(idx + 1) % M` | — | `size` |
| **Circular Queue (Sacrificed Slot, $M-1$ max)** | `front == rear` | `(rear + 1) % M == front` | `(idx + 1) % M` | — | `(rear - front + M) % M` |
| **Circular Deque (Explicit `size`)** | `size == 0` | `size == M` | `(rear + 1) % M` | `(front - 1 + M) % M` | `size` |

---

### 2.2 CPU Scheduling Metrics and Josephus Recurrence

#### CPU Scheduling Formulas (for Process $P_i$)
$$\text{Turnaround Time } (\text{TAT}_i) = \text{Completion Time } (\text{CT}_i) - \text{Arrival Time } (\text{AT}_i)$$
$$\text{Waiting Time } (\text{WT}_i) = \text{Turnaround Time } (\text{TAT}_i) - \text{Burst Time } (\text{BT}_i)$$
$$\text{Response Time } (\text{RT}_i) = \text{First CPU Start Time}_i - \text{Arrival Time } (\text{AT}_i)$$
*(Note: In non-preemptive scheduling, $\text{RT}_i = \text{WT}_i$. In preemptive scheduling such as Round-Robin or SRTF, $\text{RT}_i \le \text{WT}_i$.)*

#### Josephus Problem Recurrence ($n$ people, step size $k$)
- **0-Based Indexing ($0, 1, \dots, n - 1$):**
  $$J_0(1, k) = 0, \qquad J_0(n, k) = \big(J_0(n - 1, k) + k\big) \bmod n \quad (n \ge 2)$$
- **1-Based Indexing ($1, 2, \dots, n$):**
  $$J_1(1, k) = 1, \qquad J_1(n, k) = \big((J_1(n - 1, k) + k - 1) \bmod n\big) + 1 \quad (n \ge 2)$$
- **Closed-Form Special Case for $k = 2$:** Writing $n = 2^m + L$ where $0 \le L < 2^m$, the 1-based survivor is:
  $$J_1(2^m + L, 2) = 2L + 1$$

---

### 2.3 Tree and Forest Mathematical Formulas and LCRS Mapping Rules

| Property / Theorem | General $N$-Ary Tree / Forest Formula | Binary Tree ($N = 2$) Formula |
| :--- | :---: | :---: |
| **Edges in a Single Tree ($n$ nodes)** | $\|E\| = n - 1$ | $\|E\| = n - 1$ |
| **Edges in a Forest of $k$ Trees** | $\|E\| = n - k$ | $\|E\| = n - k$ |
| **Max Nodes at Depth $d$ ($d \ge 0$)** | $N^d$ | $2^d$ |
| **Max Total Nodes of Height $h$ (0-based edges)** | $\dfrac{N^{h+1} - 1}{N - 1}$ | $2^{h+1} - 1$ *(or $2^H - 1$ for 1-based $H$)* |
| **Minimum 0-Based Height for $n$ Nodes** | $\lceil \log_N(n(N-1) + 1) \rceil - 1$ | $\lfloor \log_2 n \rfloor = \lceil \log_2(n + 1) \rceil - 1$ |
| **Maximum 0-Based Height for $n$ Nodes** | $n - 1$ | $n - 1$ |
| **Degree-Leaf Theorem** | $L = 1 + \sum_{i=2}^{N} (i - 1)I_i$ | $L = I_2 + 1$ |
| **Leaves in a Full (Strict) Tree ($I$ internal nodes)** | $L = (N - 1)I + 1,\quad n = NI + 1$ | $L = I + 1,\quad n = 2I + 1$ |
| **Total `NULL` Child Pointers in $n$-Node Tree** | $n(N - 1) + 1$ | $n + 1$ |
| **Array Representation Index Math (0-based)** | `child(i, c) = N*i + c + 1` ($0 \le c < N$) | `left = 2i + 1`, `right = 2i + 2`, `parent = (i - 1)/2` |
| **Array Representation Index Math (1-based)** | — | `left = 2i`, `right = 2i + 1`, `parent = i / 2` |

---

## 3. Commonly Confused Concepts (Side-by-Side Disambiguation)

| Confused Pair | Concept A | Concept B | How to Never Confuse Them Again |
| :--- | :--- | :--- | :--- |
| **1. Full vs. Complete vs. Perfect Binary Tree** | **Full (Strict):** Every node has $0$ or $2$ children ($I_1 = 0$). Leaves can be at different depths! | **Complete:** All levels full except possibly the last, which is packed **left-to-right**. Can have one degree-1 node! **Perfect:** Full + all leaves at depth $h$ ($n = 2^{h+1}-1$). | Ask: *"Are degree-1 nodes forbidden?"* $\to$ **Full**. *"Does it pack into an array with zero gaps?"* $\to$ **Complete**. *"Is it a completely filled triangle of $2^{h+1}-1$ nodes?"* $\to$ **Perfect**. |
| **2. Depth vs. Height of a Node $u$** | **Depth $d(u)$:** Distance (edges) from the **root down to $u$**. $d(\text{root}) = 0$. | **Height $h(u)$:** Longest distance (edges) from **$u$ down to a leaf** in $T_u$. $h(\text{leaf}) = 0$. | **Depth** = how deep underwater $u$ is below the surface (root). **Height** = how tall $u$ stands above the ground (leaves). |
| **3. Linear vs. Circular Queue Full Condition** | **Linear Queue:** `rear == M - 1` triggers overflow even when `front > 0` (**False Overflow**). | **Circular Queue:** Truly full only when `size == M` (or `(rear + 1) % M == front` in sacrificed-slot design). | Linear queues never reuse vacated front indices (`0 .. front-1`) without $O(n)$ shifting; circular queues wrap around via `% M`. |
| **4. N-Ary Postorder vs. LCRS Binary Traversals** | **$\text{Postorder}(T_{\text{N-ary}})$** visits all children $c_1 \dots c_d$ of $u$ before visiting $u$, then moves to $u$'s next sibling. | **$\text{Inorder}(B_{\text{LCRS}})$** visits `u->left` (all children of $u$), then visits $u$, then visits `u->right` ($u$'s next sibling). | In an LCRS binary tree, right children are **siblings**, not children! Thus $\text{Postorder}(T_{\text{N-ary}}) \equiv \mathbf{\text{Inorder}}(B_{\text{LCRS}})$, NOT $\text{Postorder}(B_{\text{LCRS}})$! |
| **5. Single Tree vs. Forest in LCRS Binary Form** | **Single Converted Tree $B(T)$:** Root has no siblings, so `bin_root->right == NULL`. | **Converted Forest $B(T_1, \dots, T_k)$ ($k \ge 2$):** Roots of $T_2 \dots T_k$ are chained on the right spine, so `bin_root->right != NULL`. | Count the nodes along the top-level right spine starting at `bin_root`: that count is $k$, the exact number of trees in the forest! |
| **6. Input-Restricted vs. Output-Restricted Deque** | **Input-Restricted:** Insert at **1 end** only; Delete from **both ends**. | **Output-Restricted:** Delete from **1 end** only; Insert at **both ends**. | The word before "-Restricted" tells you which operation (`Input` = insert, `Output` = delete) is limited to a single end. |

---

## 4. Consolidated Operation-Complexity Table for Unit 2

**Variable Definitions:** $n =$ number of elements/nodes; $M =$ array capacity; $h =$ tree height ($\Theta(\log n)$ balanced, $\Theta(n)$ skewed); $w =$ max tree width; $k =$ Josephus step size or number of forest components.

| Data Structure / Algorithm | Insert / Enqueue | Delete / Dequeue | Peek / Search | Auxiliary Space |
| :--- | :---: | :---: | :---: | :---: |
| **Linear Array Queue (no shift)** | $O(1)$ | $O(1)$ | $O(1)$ (front) | $O(M)$ (wastes vacated slots) |
| **Linear Array Queue (shift on dequeue)** | $O(1)$ | $O(n)$ | $O(1)$ (front) | $O(M)$ |
| **Circular Array Queue** | $O(1)$ | $O(1)$ | $O(1)$ (front) | $O(M)$ |
| **Singly Linked List Queue (`head` + `tail`)** | $O(1)$ | $O(1)$ | $O(1)$ (front) | $O(n)$ dynamic nodes |
| **Circular Array Deque / Doubly Linked Deque** | $O(1)$ (both ends) | $O(1)$ (both ends) | $O(1)$ (both ends) | $O(M)$ or $O(n)$ |
| **Priority Queue (Unordered Array / List)** | $O(1)$ | $O(n)$ | $O(n)$ | $O(n)$ |
| **Priority Queue (Sorted Array)** | $O(n)$ (shifting) | $O(1)$ (if min at end) | $O(1)$ | $O(M)$ |
| **Priority Queue (Sorted Singly Linked List)** | $O(n)$ (traversal) | $O(1)$ (at `head`) | $O(1)$ (at `head`) | $O(n)$ |
| **Priority Queue (Binary Min/Max Heap)** | $O(\log n)$ | $O(\log n)$ | $O(1)$ | $O(n)$ |
| **Josephus Problem (Circular Queue Simulation)** | — | $O(n \cdot k)$ total | $O(1)$ per step | $O(n)$ |
| **Josephus Problem (Mathematical Recurrence)** | — | $O(n)$ total | — | $O(1)$ iterative |
| **General Binary / N-Ary Tree Search & Height** | — | — | $O(n)$ | $O(h)$ stack |
| **Binary Search Tree (BST) Operations** | $O(h)$ ($O(\log n)$ avg, $O(n)$ worst) | $O(h)$ | $O(h)$ | $O(h)$ (or $O(1)$ iterative) |
| **N-Ary / Forest $\leftrightarrow$ LCRS Binary Conversion** | $O(n)$ total | — | — | $O(h)$ stack |
| **DFS Traversals (Pre / In / Post — 1 Stack)** | — | — | $\Theta(n)$ | $\Theta(h)$ |
| **Iterative Postorder Traversal (2 Stacks)** | — | — | $\Theta(n)$ | $\Theta(n)$ |
| **Level-Order Traversal (BFS Queue)** | — | — | $\Theta(n)$ | $\Theta(w) = O(n)$ |

---

## 5. Short-Answer Questions (10 Questions)

1. **Q1:** In a circular queue of capacity $M = 8$ using the sacrificed-slot convention (without a `size` counter), if `front = 5` and `rear = 2`, how many elements are currently in the queue, and what is the maximum number of elements this queue can hold?
2. **Q2:** Why is a Singly Linked List (even with both `head` and `tail` pointers) unsuitable for implementing a general Double-Ended Queue (Deque) where all four operations must run in $O(1)$ time?
3. **Q3:** Compare the time complexities of `enqueue(x, p)` and `extract_min()` for an **Unordered Linked List Priority Queue** versus a **Sorted Linked List Priority Queue**.
4. **Q4:** What is the **Convoy Effect** in CPU scheduling, and which scheduling algorithm is most vulnerable to it?
5. **Q5:** Using the 1-based Josephus closed-form formula for $k = 2$, find the survivor $J_1(100, 2)$ in two lines of arithmetic.
6. **Q6:** A full (strict) $4$-ary tree has $I = 15$ internal nodes. How many leaf nodes ($L$) and how many total nodes ($n$) does it contain?
7. **Q7:** In any non-empty binary tree with $n$ nodes implemented using standard `left` and `right` pointers, if there are $12$ nodes with degree $2$ ($I_2 = 12$) and $5$ nodes with degree $1$ ($I_1 = 5$), how many leaf nodes ($L$) and how many `NULL` child pointers exist in the tree?
8. **Q8:** A forest $\mathcal{F}$ contains $n = 24$ vertices and $|E| = 19$ edges. How many disjoint trees ($k$) are in $\mathcal{F}$, and if $\mathcal{F}$ is converted into an LCRS Binary Tree $B(\mathcal{F})$, how many nodes lie on the topmost right spine starting at `bin_root`?
9. **Q9:** Why must `curr->right` be pushed onto the explicit stack **before** `curr->left` in an Iterative Preorder traversal of a binary tree?
10. **Q10:** If an $N$-ary tree $T$ has Postorder traversal `E F B C G D A`, what is the **Inorder** traversal of its LCRS Binary Tree representation $B(T)$?

---

## 6. Conceptual and Deep-Thinking Questions (8 Questions)

1. **CQ1 (Ambiguity of `front == rear` in Circular Queues):** Explain why the condition `front == rear` (or `(rear + 1) % M == front`) cannot distinguish between an **empty** circular queue and a **full** circular queue of $M$ elements without additional state. Compare three distinct architectural ways to resolve this ambiguity in C.
2. **CQ2 (Round-Robin Time Quantum Trade-off):** Analyze the behavior of a Round-Robin CPU scheduler as the time quantum $q \to \infty$ versus as $q \to 0^+$. Why is the Turnaround Time (TAT) non-monotonic with respect to $q$, and what is the practical "80% rule of thumb" used by operating system designers?
3. **CQ3 (Starvation vs. Deadlock & Aging):** Distinguish between **Starvation** in Priority Scheduling and **Deadlock**. If a system assigns priorities $0$ (highest) to $127$ (lowest) and a new priority-$10$ process arrives every millisecond, how long would a priority-$100$ process wait without Aging? Formulate a mathematical Aging rule that guarantees bounded waiting time.
4. **CQ4 (Binary Tree vs. 2-Ary Ordered Tree):** Prove with a concrete small example why the set of all distinct Binary Trees of $n = 3$ nodes is strictly larger than the set of all distinct Ordered $2$-Ary Trees of $n = 3$ nodes.
5. **CQ5 (Validating the BST Invariant):** Draw a 4-node binary tree where every node $u$ satisfies `u->left->key < u->key` and `u->right->key > u->key`, yet the tree is **not** a valid Binary Search Tree. Explain how the recursive `(min_val, max_val)` interval algorithm catches this violation.
6. **CQ6 (When Preorder + Postorder CAN Reconstruct a Binary Tree):** We proved in Chapter 05 that `(Preorder + Postorder)` cannot uniquely reconstruct a general binary tree. Prove that if the binary tree is guaranteed to be a **Full (Strict) Binary Tree** (every node has $0$ or $2$ children), then `(Preorder + Postorder)` **does** uniquely determine the tree!
7. **CQ7 (Why Single-Stack Iterative Postorder Needs `last_visited`):** Trace what goes wrong in a single-stack iterative Postorder traversal if you only inspect `peek_node = Top(S)` without maintaining a `last_visited` pointer (or state flag).
8. **CQ8 (Space Complexity Crossover Between DFS and BFS):** For a balanced Perfect Binary Tree of $n = 2^{h+1} - 1$ nodes versus a Degenerate Skewed Binary Tree of $n$ nodes, compare the peak memory consumption of **Iterative Inorder (Stack)** against **Level-Order (Queue)**. For which tree topology does BFS use *less* memory than DFS?

---

## 7. Algorithm-Writing Questions (5 Exam-Style Pseudocode Problems)

1. **AQ1:** Write clear, structured pseudocode for `DequeInsertFront(dq, x)` and `DequeDeleteRear(dq)` on a Circular Array Deque of capacity $M$ that maintains `front`, `rear`, and `size`.
2. **AQ2:** Write an $O(n)$ recursive or iterative algorithm `CheckCompleteBinaryTree(root)` using a Level-Order FIFO Queue that returns `true` if a linked binary tree is a **Complete Binary Tree** and `false` otherwise. *(Hint: Once a missing child / `NULL` slot is seen in level-order, no subsequent node may be non-`NULL`!)*
3. **AQ3:** Write pseudocode for `ForestToBinaryLCRS(T[0..k-1], k)` and its inverse `BinaryLCRSToForest(bin_root)` that convert between an array of $k$ $N$-ary trees and a single LCRS Binary Tree.
4. **AQ4:** Write pseudocode for `IterativePostorderSingleStack(root)` using a single stack `S` and a `last_visited` pointer.
5. **AQ5:** Write pseudocode for `ReconstructFromInorderPostorder(In[0..n-1], Post[0..n-1])` that rebuilds a unique binary tree from its Inorder and Postorder traversal arrays.

---

## 8. C Programming Questions (5 Coding Problems)

1. **PQ1:** Implement a complete C11 `CircularDeque` module (`deque_init`, `deque_insert_front`, `deque_insert_rear`, `deque_delete_front`, `deque_delete_rear`) backed by a fixed array of capacity `DEQUE_CAP`, handling empty/full boundary conditions and modular wrap-around.
2. **PQ2:** Implement a **Sorted Singly Linked List Priority Queue** in C11 (`pq_enqueue` in $O(n)$ time and `pq_dequeue` in $O(1)$ time) where lower integer values denote higher priority and ties are broken in strict **FIFO arrival order**.
3. **PQ3:** Write a C11 function `int josephus_queue(int n, int k)` that simulates the Josephus problem for $n$ people ($1 \dots n$) and step $k$ using a circular queue, returning the 1-based survivor number.
4. **PQ4:** Write C11 functions `int bt_count_leaves(const BTNode *root)`, `int bt_count_deg2(const BTNode *root)`, and `bool bt_is_valid_bst(const BTNode *root, long min_v, long max_v)` for a linked binary tree.
5. **PQ5:** Write a C11 function `void inorder_iterative(const BTNode *root)` that performs an Inorder traversal of a binary tree without recursion using an explicit `NodeStack`.

---

## 9. Worked Problem-Solving Questions (5 Step-by-Step Traces)

### Problem 1: Circular Deque Index Trace
A Circular Array Deque `dq` of capacity $M = 5$ is initialized with `front = 0, rear = -1, size = 0`. Trace the exact values of `front`, `rear`, `size`, and the array contents `data[0..4]` after each of the following 8 operations:
1. `insert_rear(10)`
2. `insert_rear(20)`
3. `insert_front(30)`
4. `insert_front(40)`
5. `delete_rear()`
6. `insert_front(50)`
7. `delete_front()`
8. `insert_rear(60)`

---

### Problem 2: Josephus Elimination Trace and Recurrence Verification
For $n = 6$ people numbered $1, 2, 3, 4, 5, 6$ and step size $k = 3$:
1. Trace the circular queue state and eliminated person in each of the $5$ elimination rounds until the survivor is found.
2. Compute $J_0(k, 3)$ and $J_1(k, 3)$ for $k = 1, 2, 3, 4, 5, 6$ using the mathematical recurrence and confirm it matches your queue simulation.

---

### Problem 3: Round-Robin CPU Scheduling Gantt Chart and Metrics
Consider the following 4 processes arriving at a CPU scheduler using **Round-Robin (RR)** with Time Quantum **$q = 3\text{ ms}$** (newly arriving processes at time $t$ are placed in the ready queue *before* a process preempted at the exact same instant $t$):

| Process | Arrival Time ($\text{AT}$) | Burst Time ($\text{BT}$) |
| :---: | :---: | :---: |
| $P_1$ | $0\text{ ms}$ | $7\text{ ms}$ |
| $P_2$ | $2\text{ ms}$ | $4\text{ ms}$ |
| $P_3$ | $4\text{ ms}$ | $1\text{ ms}$ |
| $P_4$ | $5\text{ ms}$ | $4\text{ ms}$ |

1. Draw the complete **Gantt Chart** and show the Ready Queue contents at every context switch.
2. Compute **CT, TAT, WT, and RT** for each process and the **Average TAT** and **Average WT**.

---

### Problem 4: Forest to LCRS Binary Tree Conversion and Traversal Verification
Given a forest $\mathcal{F} = (T_1, T_2)$ of two $N$-ary trees:
- **Tree $T_1$:** Root `A` has children `B, C, D`. Node `B` has children `E, F`. Node `D` has child `G`.
- **Tree $T_2$:** Root `H` has children `I, J`. Node `I` has child `K`.
1. Convert $\mathcal{F} = (T_1, T_2)$ into a single LCRS Binary Tree $B(\mathcal{F})$ and draw its ASCII diagram.
2. Count $n, L, I_1, I_2$ in $B(\mathcal{F})$ and verify $L = I_2 + 1$.
3. Write out $\text{Preorder}(\mathcal{F})$ and $\text{Postorder}(\mathcal{F})$, and verify that they equal $\text{Preorder}(B(\mathcal{F}))$ and $\text{Inorder}(B(\mathcal{F}))$, respectively.

---

### Problem 5: Unique Binary Tree Reconstruction from Inorder and Preorder
Reconstruct the unique binary tree $B$ given the following traversal sequences:
- **`Inorder   = [H, D, I, B, E, A, F, C, G]`**
- **`Preorder  = [A, B, D, H, I, E, C, F, G]`**
Show each recursive partitioning step, draw the final binary tree, and determine its **Postorder** and **Level-Order** traversals.

---

## 10. Complete Answer Keys with Detailed Explanations

---

### 10.1 Answers to Short-Answer Questions

1. **A1:**
   - Current element count $= (\text{rear} - \text{front} + M) \bmod M = (2 - 5 + 8) \bmod 8 = 5$ elements (at indices `5, 6, 7, 0, 1`).
   - Because the sacrificed-slot convention leaves 1 slot permanently unused to distinguish full (`(rear + 1) % M == front`) from empty (`front == rear`), the maximum capacity is $M - 1 = 7$ elements.

2. **A2:**
   - In a Singly Linked List, `insert_front`, `insert_rear`, and `delete_front` all run in $O(1)$ time. However, **`delete_rear`** requires updating the `tail` pointer to the *second-to-last* node (`prev`). Because singly linked nodes have no `prev` pointer, finding the predecessor of `tail` requires an $O(n)$ traversal from `head`. A Doubly Linked List (or Circular Array) is required for $O(1)$ `delete_rear`.

3. **A3:**
   - **Unordered Linked List PQ:** `enqueue` is $O(1)$ (insert at head), but `extract_min` is $O(n)$ (must scan all $n$ nodes to find the minimum key).
   - **Sorted Linked List PQ:** `enqueue` is $O(n)$ (must walk the list to insert in sorted priority order), while `extract_min` is $O(1)$ (remove the `head` node).

4. **A4:**
   - The **Convoy Effect** occurs when one CPU-bound process with a very long burst time holds the CPU while many short I/O-bound processes pile up behind it in the ready queue, dramatically inflating average waiting and turnaround times. **First-Come, First-Served (FCFS)** is most vulnerable to it because it is non-preemptive.

5. **A5:**
   - Express $n = 100$ as $2^m + L$ where $2^m \le 100$ is the largest power of $2$: $100 = 64 + 36 \implies 2^m = 64,\ L = 36$.
   - By the closed-form formula $J_1(2^m + L, 2) = 2L + 1$: $J_1(100, 2) = 2(36) + 1 = \mathbf{73}$.

6. **A6:**
   - For a full $N$-ary tree with $N = 4$ and $I = 15$ internal nodes:
     $$L = (N - 1)I + 1 = (4 - 1)(15) + 1 = 45 + 1 = \mathbf{46\text{ leaves}}$$
     $$n = L + I = 46 + 15 = \mathbf{61\text{ total nodes}} \quad (\text{or } n = NI + 1 = 4(15) + 1 = 61)$$

7. **A7:**
   - By the Binary Tree Degree-2 Leaf Theorem ($L = I_2 + 1$): $L = 12 + 1 = \mathbf{13\text{ leaf nodes}}$ (notice $I_1 = 5$ does not affect $L$!).
   - Total nodes $n = L + I_1 + I_2 = 13 + 5 + 12 = 30$.
   - Total `NULL` child pointers in a $30$-node binary tree $= n + 1 = 30 + 1 = \mathbf{31\text{ NULL pointers}}$.

8. **A8:**
   - By Theorem 6 ($|E| = |V| - k \implies k = |V| - |E|$): $k = 24 - 19 = \mathbf{5\text{ disjoint trees}}$ in the forest $\mathcal{F}$.
   - When converted to an LCRS Binary Tree $B(\mathcal{F})$, the roots of the $k = 5$ trees are chained along the topmost right spine starting at `bin_root`, so there are **exactly $5$ nodes** on that right spine.

9. **A9:**
   - Because an explicit stack is **LIFO (Last-In, First-Out)**, the item pushed *last* is popped *first*. Preorder ($\text{NLR}$) requires visiting the **left** child before the **right** child. Therefore, we must push `curr->right` first (placing it deeper in the stack) and `curr->left` second (placing it on top of the stack to be popped immediately on the next iteration).

10. **A10:**
    - By the **LCRS Traversal Correspondence Theorem** ($\text{Postorder}(T) \equiv \text{Inorder}(B(T))$), the Inorder traversal of the LCRS Binary Tree $B(T)$ is identical to the N-ary Postorder of $T$: **`E F B C G D A`**.

---

### 10.2 Answers to Conceptual Questions

1. **Answer to CQ1:**
   - In a circular array of size $M$, there are $M$ possible positions for `front` and $M$ possible positions for `rear`. For any fixed `front`, the queue can hold $0, 1, 2, \dots, M$ elements—which is **$M + 1$ distinct occupancy states**! By the Pigeonhole Principle, $M$ possible values of `rear` cannot uniquely encode $M + 1$ occupancy states; both $0$ elements (empty) and $M$ elements (full) wrap `rear` to the exact same relative offset from `front`.
   - **Three C Resolutions:**
     1. **Explicit `int size` Counter (Best Practice):** Track `size` ($0 \dots M$); empty is `size == 0`, full is `size == M`, using all $M$ slots.
     2. **Sacrificed One Slot:** Never fill the last slot; empty is `front == rear`, full is `(rear + 1) % M == front` (max capacity $M - 1$).
     3. **Boolean `is_full` Flag / Unbounded Monotonic Counters:** Maintain `bool full` (or increment unsigned `read_cnt` and `write_cnt` before taking `% M`, so `write_cnt - read_cnt == M` means full and `== 0` means empty).

2. **Answer to CQ2:**
   - **As $q \to \infty$:** Every process finishes its entire CPU burst in a single time slice without preemption; Round-Robin **degenerates into FCFS**.
   - **As $q \to 0^+$:** Context switches occur after every infinitesimal instruction. If context switch overhead is $c > 0$, useful CPU utilization $\frac{q}{q + c} \to 0$, and processes spend nearly $100\%$ of time in OS context-switch overhead.
   - **80% Rule of Thumb:** Choose $q$ (~$10\text{–}100\text{ ms}$) so that approximately **$80\%$ of CPU bursts** are shorter than $q$ (completing in a single quantum without extra context-switch overhead), while longer CPU-bound jobs still time-slice fairly.

3. **Answer to CQ3:**
   - **Starvation vs. Deadlock:** In **Starvation**, low-priority processes are ready to run and making system-wide progress is possible, but high-priority arrivals continuously preempt/bypass them. In **Deadlock**, a set of processes are blocked waiting on resources held by each other, so **none** of the deadlocked processes can ever proceed even if the CPU is completely idle.
   - **Guaranteed Aging Formula:** If priority $P \in [0, 127]$ ($0 = \text{highest}$) is decremented by $1$ every $\Delta t$ time units spent waiting in the ready queue:
     $$P_{\text{effective}}(t) = \max\!\left(0,\; P_{\text{initial}} - \left\lfloor \frac{t - t_{\text{arrival}}}{\Delta t} \right\rfloor\right)$$
     Then a process with $P_{\text{initial}} = 100$ is guaranteed to reach priority $0$ (the highest possible priority in the system) in at most $100 \cdot \Delta t$ time units, bounding its maximum wait time!

4. **Answer to CQ4:**
   - For $n = 3$ unlabeled nodes, the number of structurally distinct Binary Trees is given by the $3^{\text{rd}}$ Catalan number $C_3 = \frac{1}{3+1}\binom{6}{3} = \mathbf{5}$ (1 balanced tree + 4 skewed chains: Left-Left, Left-Right, Right-Left, Right-Right).
   - By contrast, in an Ordered $2$-Ary Tree, a single child is not distinguished as "left" or "right"—all 4 skewed chains collapse into a **single** chain of height $2$ ($r \to c_1 \to c_2$), leaving only **2** distinct ordered $2$-ary trees (the chain of height $2$ and the root with $2$ children).

5. **Answer to CQ5:**
   - **Counterexample Tree:** Root `20`, Left child `10`, and `10`'s Right child is `25` (with `20`'s Right child `30`).
   - Locally at `20`: `10 < 20 < 30` (passes). Locally at `10`: `10->right` is `25 > 10` (passes). Yet `25` is in the **left subtree** of `20` and $25 \not< 20$, violating the global BST invariant!
   - The recursive range algorithm calls `is_bst(root->left, -inf, 20)` at node `10`, and then calls `is_bst(10->right, 10, 20)` at node `25`. Because $25 \notin (10, 20)$, it immediately returns `false`.

6. **Answer to CQ6:**
   - Suppose $T$ is a **Full (Strict) Binary Tree** ($I_1 = 0$), and we are given `Preorder` and `Postorder`.
   - The first element of `Preorder` (and last of `Postorder`) is the root $r$. If $n = 1$, $r$ is a leaf and we are done.
   - If $n > 1$, because $T$ is a **Full Binary Tree**, $r$ is guaranteed to have **both** a non-empty left subtree $T_L$ and a non-empty right subtree $T_R$!
   - Therefore, the second element of `Preorder`, say $x = \text{Preorder}[1]$, is guaranteed to be the **root of the left subtree $T_L$**.
   - In `Postorder`, the root $x$ of $T_L$ appears at the **very end** of $T_L$'s postorder sequence! Finding $x$ at index $k$ in `Postorder` uniquely identifies $\text{Postorder}[0 \dots k]$ as all nodes of $T_L$ (size $k + 1$) and $\text{Postorder}[k + 1 \dots n - 2]$ as all nodes of $T_R$. By induction, $T$ is uniquely determined!

7. **Answer to CQ7:**
   - In single-stack iterative Postorder, a parent node $u$ remains on the stack while we visit `u->left` AND while we visit `u->right`. When `curr` becomes `NULL` and we inspect `peek_node = Top(S) = u`:
     - The first time we return to $u$ (from `u->left`), we must NOT pop $u$; we must set `curr = u->right`.
     - The second time we return to $u$ (after finishing `u->right`), we MUST pop and visit $u$.
   - Without `last_visited` to record that `last_visited == u->right`, the algorithm cannot tell whether it just returned from `u->left` or `u->right`, so it will infinitely re-enter `curr = u->right`!

8. **Answer to CQ8:**
   - **Perfect Balanced Binary Tree ($n = 2^{h+1} - 1$):**
     - **DFS Stack Space:** $\Theta(h) = \Theta(\log_2 n)$ pointers (for $n = 1023$, $h = 9 \implies \sim 10$ pointers).
     - **BFS Queue Space:** $\Theta(w) = 2^h = \frac{n + 1}{2} = \Theta(n)$ pointers (for $n = 1023$, the bottom level alone holds $512$ nodes in the queue!). Here **DFS uses exponentially less memory than BFS**.
   - **Degenerate Skewed Binary Tree ($h = n - 1$, width $w = 1$):**
     - **DFS Stack Space:** $\Theta(h) = \Theta(n)$ pointers (a left-skewed tree pushes all $n$ nodes onto the Inorder/Postorder stack).
     - **BFS Queue Space:** $\Theta(w) = \Theta(1)$ (at most $1$ node is in the queue at each level!). Here **BFS uses $O(1)$ space while DFS uses $O(n)$ space**.

---

### 10.3 Solutions to Algorithm-Writing Questions

#### Solution to AQ1: Circular Array Deque (`DequeInsertFront` & `DequeDeleteRear`)
```text
ALGORITHM DequeInsertFront(dq, x)
Input:  CircularDeque dq (fields: data[0..M-1], front, rear, size), element x
1.  if dq.size == M then
2.      error "Deque Overflow"
3.  if dq.size == 0 then
4.      dq.front <- 0
5.      dq.rear  <- 0
6.  else
7.      dq.front <- (dq.front - 1 + M) mod M
8.  dq.data[dq.front] <- x
9.  dq.size <- dq.size + 1

ALGORITHM DequeDeleteRear(dq)
Input:  CircularDeque dq
Output: Element removed from rear
1.  if dq.size == 0 then
2.      error "Deque Underflow"
3.  val <- dq.data[dq.rear]
4.  dq.rear <- (dq.rear - 1 + M) mod M
5.  dq.size <- dq.size - 1
6.  return val
```

#### Solution to AQ2: Check Complete Binary Tree Using a Level-Order Queue
```text
ALGORITHM CheckCompleteBinaryTree(root)
Input:  Root pointer root of a binary tree
Output: true if the binary tree is complete, false otherwise
1.  if root == NULL then return true
2.  Q <- EmptyQueue()
3.  Enqueue(Q, root)
4.  seen_null_child <- false
5.  while not IsEmpty(Q) do
6.      curr <- Dequeue(Q)
7.      if curr == NULL then
8.          seen_null_child <- true
9.      else
10.         if seen_null_child == true then
11.             return false   // Found a non-NULL node after a gap!
12.         Enqueue(Q, curr.left)
13.         Enqueue(Q, curr.right)
14. return true
```

#### Solution to AQ3: Forest $\leftrightarrow$ LCRS Binary Tree Conversion
```text
ALGORITHM NaryToLCRS(u)
1.  if u == NULL then return NULL
2.  b <- CreateBTNode(u.label)
3.  if u.child_count > 0 then
4.      b.left <- NaryToLCRS(u.children[0])
5.      curr <- b.left
6.      for i <- 1 to u.child_count - 1 do
7.          curr.right <- NaryToLCRS(u.children[i])
8.          curr <- curr.right
9.  return b

ALGORITHM ForestToBinaryLCRS(T[0..k-1], k)
1.  if k <= 0 then return NULL
2.  bin_root <- NaryToLCRS(T[0])
3.  curr <- bin_root
4.  for i <- 1 to k - 1 do
5.      curr.right <- NaryToLCRS(T[i])
6.      curr <- curr.right
7.  return bin_root

ALGORITHM BinarySubtreeToNary(b)
1.  if b == NULL then return NULL
2.  u <- CreateNAryNode(b.label)
3.  c <- b.left
4.  while c != NULL do
5.      AddChild(u, BinarySubtreeToNary(c))
6.      c <- c.right
7.  return u

ALGORITHM BinaryLCRSToForest(bin_root)
1.  F <- EmptyList()
2.  curr <- bin_root
3.  while curr != NULL do
4.      Append BinarySubtreeToNary(curr) to F
5.      curr <- curr.right
6.  return F
```

#### Solution to AQ4: Single-Stack Iterative Postorder Traversal
```text
ALGORITHM IterativePostorderSingleStack(root)
1.  S <- EmptyStack()
2.  curr <- root
3.  last_visited <- NULL
4.  while curr != NULL or not IsEmpty(S) do
5.      if curr != NULL then
6.          Push(S, curr)
7.          curr <- curr.left
8.      else
9.          peek_node <- Peek(S)
10.         if peek_node.right != NULL and last_visited != peek_node.right then
11.             curr <- peek_node.right
12.         else
13.             Visit(peek_node)
14.             last_visited <- Pop(S)
```

#### Solution to AQ5: Reconstruct Binary Tree from `Inorder` and `Postorder`
```text
ALGORITHM BuildFromInPost(In, Post, in_start, in_end, post_idx_ref)
1.  if in_start > in_end then return NULL
2.  root_val <- Post[post_idx_ref]
3.  post_idx_ref <- post_idx_ref - 1
4.  root <- CreateBTNode(root_val)
5.  if in_start == in_end then return root
6.  mid <- IndexOf(In, in_start, in_end, root_val)
7.  // CRITICAL: Build RIGHT subtree before LEFT subtree when reading Postorder backwards!
8.  root.right <- BuildFromInPost(In, Post, mid + 1, in_end, post_idx_ref)
9.  root.left  <- BuildFromInPost(In, Post, in_start, mid - 1, post_idx_ref)
10. return root
```

---

### 10.4 Complete C11 Solutions to Programming Questions

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <limits.h>

/* =========================================================================
 * SOLUTION TO PQ1: CIRCULAR ARRAY DEQUE
 * ========================================================================= */
#define DEQUE_CAP 5

typedef struct {
    int data[DEQUE_CAP];
    int front;
    int rear;
    int size;
} CircularDeque;

void deque_init(CircularDeque *dq) {
    dq->front = 0;
    dq->rear = -1;
    dq->size = 0;
}

bool deque_is_full(const CircularDeque *dq)  { return dq->size == DEQUE_CAP; }
bool deque_is_empty(const CircularDeque *dq) { return dq->size == 0; }

bool deque_insert_front(CircularDeque *dq, int val) {
    if (deque_is_full(dq)) return false;
    if (deque_is_empty(dq)) {
        dq->front = 0;
        dq->rear = 0;
    } else {
        dq->front = (dq->front - 1 + DEQUE_CAP) % DEQUE_CAP;
    }
    dq->data[dq->front] = val;
    dq->size++;
    return true;
}

bool deque_insert_rear(CircularDeque *dq, int val) {
    if (deque_is_full(dq)) return false;
    if (deque_is_empty(dq)) {
        dq->front = 0;
        dq->rear = 0;
    } else {
        dq->rear = (dq->rear + 1) % DEQUE_CAP;
    }
    dq->data[dq->rear] = val;
    dq->size++;
    return true;
}

bool deque_delete_front(CircularDeque *dq, int *out) {
    if (deque_is_empty(dq)) return false;
    *out = dq->data[dq->front];
    dq->front = (dq->front + 1) % DEQUE_CAP;
    dq->size--;
    return true;
}

bool deque_delete_rear(CircularDeque *dq, int *out) {
    if (deque_is_empty(dq)) return false;
    *out = dq->data[dq->rear];
    dq->rear = (dq->rear - 1 + DEQUE_CAP) % DEQUE_CAP;
    dq->size--;
    return true;
}

/* =========================================================================
 * SOLUTION TO PQ2: SORTED SINGLY LINKED LIST PRIORITY QUEUE (FIFO TIE-BREAK)
 * ========================================================================= */
typedef struct PQNode {
    int data;
    int priority; /* Lower integer = Higher priority */
    struct PQNode *next;
} PQNode;

void pq_enqueue(PQNode **head, int data, int priority) {
    PQNode *new_node = (PQNode *)malloc(sizeof(PQNode));
    if (new_node == NULL) exit(EXIT_FAILURE);
    new_node->data = data;
    new_node->priority = priority;
    new_node->next = NULL;

    /* Insert before head only if strictly higher priority (<, not <=) */
    if (*head == NULL || priority < (*head)->priority) {
        new_node->next = *head;
        *head = new_node;
        return;
    }
    /* Walk past all nodes with priority <= new priority to preserve FIFO order */
    PQNode *curr = *head;
    while (curr->next != NULL && curr->next->priority <= priority) {
        curr = curr->next;
    }
    new_node->next = curr->next;
    curr->next = new_node;
}

bool pq_dequeue(PQNode **head, int *out_data, int *out_prio) {
    if (*head == NULL) return false;
    PQNode *temp = *head;
    *out_data = temp->data;
    *out_prio = temp->priority;
    *head = temp->next;
    free(temp);
    return true;
}

/* =========================================================================
 * SOLUTION TO PQ3: JOSEPHUS SURVIVOR USING A CIRCULAR QUEUE
 * ========================================================================= */
int josephus_queue(int n, int k) {
    if (n <= 0 || k <= 0) return -1;
    int *q = (int *)malloc((size_t)n * sizeof(int));
    if (q == NULL) exit(EXIT_FAILURE);

    int front = 0, rear = n - 1, size = n;
    for (int i = 0; i < n; i++) {
        q[i] = i + 1; /* 1-based numbering */
    }

    while (size > 1) {
        /* Dequeue and re-enqueue k - 1 people */
        for (int step = 1; step < k; step++) {
            int person = q[front];
            front = (front + 1) % n;
            rear = (rear + 1) % n;
            q[rear] = person;
        }
        /* Permanently eliminate the k-th person at front */
        front = (front + 1) % n;
        size--;
    }

    int survivor = q[front];
    free(q);
    return survivor;
}

/* =========================================================================
 * SOLUTION TO PQ4 & PQ5: BINARY TREE DEGREE COUNTS, BST CHECK & INORDER
 * ========================================================================= */
typedef struct BTNode {
    int key;
    struct BTNode *left;
    struct BTNode *right;
} BTNode;

int bt_count_leaves(const BTNode *root) {
    if (root == NULL) return 0;
    if (root->left == NULL && root->right == NULL) return 1;
    return bt_count_leaves(root->left) + bt_count_leaves(root->right);
}

int bt_count_deg2(const BTNode *root) {
    if (root == NULL) return 0;
    int is_deg2 = (root->left != NULL && root->right != NULL) ? 1 : 0;
    return is_deg2 + bt_count_deg2(root->left) + bt_count_deg2(root->right);
}

bool bt_is_valid_bst(const BTNode *root, long min_v, long max_v) {
    if (root == NULL) return true;
    if (root->key <= min_v || root->key >= max_v) return false;
    return bt_is_valid_bst(root->left, min_v, root->key) &&
           bt_is_valid_bst(root->right, root->key, max_v);
}

void inorder_iterative(const BTNode *root) {
    const BTNode *stack[64];
    int top = -1;
    const BTNode *curr = root;

    while (curr != NULL || top != -1) {
        while (curr != NULL) {
            stack[++top] = curr;
            curr = curr->left;
        }
        curr = stack[top--];
        printf("%d ", curr->key);
        curr = curr->right;
    }
}
```

---

### 10.5 Complete Step-by-Step Solutions to Worked Problems

---

#### Solution to Problem 1: Circular Deque Index Trace ($M = 5$)

| Step | Operation | Formula Executed | `front` | `rear` | `size` | Array `data[0 .. 4]` | Logical Deque (Front $\to$ Rear) |
| :---: | :--- | :--- | :---: | :---: | :---: | :--- | :--- |
| **0** | `init` | — | `0` | `-1` | `0` | `[-, -, -, -, -]` | `[]` |
| **1** | `insert_rear(10)` | Empty $\to$ `front=0, rear=0` | `0` | `0` | `1` | `[10, -, -, -, -]` | `[10]` |
| **2** | `insert_rear(20)` | `rear = (0 + 1) % 5 = 1` | `0` | `1` | `2` | `[10, 20, -, -, -]` | `[10, 20]` |
| **3** | `insert_front(30)` | `front = (0 - 1 + 5) % 5 = 4` | `4` | `1` | `3` | `[10, 20, -, -, 30]` | `[30, 10, 20]` |
| **4** | `insert_front(40)` | `front = (4 - 1 + 5) % 5 = 3` | `3` | `1` | `4` | `[10, 20, -, 40, 30]` | `[40, 30, 10, 20]` |
| **5** | `delete_rear()` | Returns `20`; `rear = (1 - 1 + 5) % 5 = 0` | `3` | `0` | `3` | `[10, (20), -, 40, 30]` | `[40, 30, 10]` |
| **6** | `insert_front(50)` | `front = (3 - 1 + 5) % 5 = 2` | `2` | `0` | `4` | `[10, -, 50, 40, 30]` | `[50, 40, 30, 10]` |
| **7** | `delete_front()` | Returns `50`; `front = (2 + 1) % 5 = 3` | `3` | `0` | `3` | `[10, -, (50), 40, 30]` | `[40, 30, 10]` |
| **8** | `insert_rear(60)` | `rear = (0 + 1) % 5 = 1` | `3` | `1` | `4` | `[10, 60, -, 40, 30]` | **`[40, 30, 10, 60]`** |

---

#### Solution to Problem 2: Josephus Elimination Trace ($n = 6,\ k = 3$)

1. **Circular Queue Elimination Trace:**
   - **Initial Queue:** `[1, 2, 3, 4, 5, 6]`
   - **Round 1:** Rotate `1`, `2` to back $\to$ `[3, 4, 5, 6, 1, 2]`. Eliminate $3^{\text{rd}}$ person: **`3`**. Remaining: `[4, 5, 6, 1, 2]`.
   - **Round 2:** Rotate `4`, `5` to back $\to$ `[6, 1, 2, 4, 5]`. Eliminate $3^{\text{rd}}$ person: **`6`**. Remaining: `[1, 2, 4, 5]`.
   - **Round 3:** Rotate `1`, `2` to back $\to$ `[4, 5, 1, 2]`. Eliminate $3^{\text{rd}}$ person: **`4`**. Remaining: `[5, 1, 2]`.
   - **Round 4:** Rotate `5`, `1` to back $\to$ `[2, 5, 1]`. Eliminate $3^{\text{rd}}$ person: **`2`**. Remaining: `[5, 1]`.
   - **Round 5:** Rotate `5`, `1` to back:
     - Step 1: Rotate `5` $\to$ `[1, 5]`.
     - Step 2: Rotate `1` $\to$ `[5, 1]`.
     - Step 3: Eliminate front person **`5`**.
   - **Final Survivor:** **`1`**.

2. **Recurrence Verification ($J_0(m, 3) = (J_0(m - 1, 3) + 3) \bmod m$, and $J_1 = J_0 + 1$):**
   - $m = 1:\ J_0(1, 3) = 0 \implies J_1(1, 3) = 1$
   - $m = 2:\ J_0(2, 3) = (0 + 3) \bmod 2 = 1 \implies J_1(2, 3) = 2$
   - $m = 3:\ J_0(3, 3) = (1 + 3) \bmod 3 = 1 \implies J_1(3, 3) = 2$
   - $m = 4:\ J_0(4, 3) = (1 + 3) \bmod 4 = 0 \implies J_1(4, 3) = 1$
   - $m = 5:\ J_0(5, 3) = (0 + 3) \bmod 5 = 3 \implies J_1(5, 3) = 4$
   - $m = 6:\ J_0(6, 3) = (3 + 3) \bmod 6 = 0 \implies J_1(6, 3) = 0 + 1 = \mathbf{1}$ *(Matches queue simulation!)*

---

#### Solution to Problem 3: Round-Robin CPU Scheduling ($q = 3\text{ ms}$)

1. **Step-by-Step Ready Queue & Execution Trace:**
   - **$t = 0$:** $P_1$ arrives ($\text{rem} = 7$). Ready Queue: `[P1]`. $P_1$ runs for $q = 3\text{ ms}$ ($t = 0 \to 3$, $\text{rem} = 4$).
     - During $t = 0 \to 3$, $P_2$ arrives at $t = 2$ ($\text{rem} = 4$). Ready Queue at $t = 3$ before $P_1$ re-joins: `[P2]`. After $P_1$ is preempted: `[P2, P1]`.
   - **$t = 3 \to 6$:** $P_2$ runs for $q = 3\text{ ms}$ ($\text{rem} = 1$).
     - During $t = 3 \to 6$, $P_3$ arrives at $t = 4$ ($\text{rem} = 1$) and $P_4$ arrives at $t = 5$ ($\text{rem} = 4$). Ready Queue at $t = 6$: `[P1, P3, P4, P2]`.
   - **$t = 6 \to 9$:** $P_1$ runs for $q = 3\text{ ms}$ ($\text{rem} = 1$). Ready Queue at $t = 9$: `[P3, P4, P2, P1]`.
   - **$t = 9 \to 10$:** $P_3$ needs only $1\text{ ms} < q$ and **completes at $t = 10$**! Ready Queue at $t = 10$: `[P4, P2, P1]`.
   - **$t = 10 \to 13$:** $P_4$ runs for $q = 3\text{ ms}$ ($\text{rem} = 1$). Ready Queue at $t = 13$: `[P2, P1, P4]`.
   - **$t = 13 \to 14$:** $P_2$ runs for $1\text{ ms}$ and **completes at $t = 14$**! Ready Queue: `[P1, P4]`.
   - **$t = 14 \to 15$:** $P_1$ runs for $1\text{ ms}$ and **completes at $t = 15$**! Ready Queue: `[P4]`.
   - **$t = 15 \to 16$:** $P_4$ runs for $1\text{ ms}$ and **completes at $t = 16$**! Ready Queue: `[]`.

```text
Gantt Chart (Round-Robin, q = 3 ms):
+--------+--------+--------+----+--------+----+----+----+
|   P1   |   P2   |   P1   | P3 |   P4   | P2 | P1 | P4 |
+--------+--------+--------+----+--------+----+----+----+
0        3        6        9   10       13   14   15   16
```

2. **Scheduling Metrics Table:**

| Process | $\text{AT}$ | $\text{BT}$ | First Start | $\text{CT}$ | $\text{TAT} = \text{CT} - \text{AT}$ | $\text{WT} = \text{TAT} - \text{BT}$ | $\text{RT} = \text{Start} - \text{AT}$ |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **$P_1$** | $0$ | $7$ | $0$ | $15$ | $15 - 0 = \mathbf{15}$ | $15 - 7 = \mathbf{8}$ | $0 - 0 = \mathbf{0}$ |
| **$P_2$** | $2$ | $4$ | $3$ | $14$ | $14 - 2 = \mathbf{12}$ | $12 - 4 = \mathbf{8}$ | $3 - 2 = \mathbf{1}$ |
| **$P_3$** | $4$ | $1$ | $9$ | $10$ | $10 - 4 = \mathbf{6}$ | $6 - 1 = \mathbf{5}$ | $9 - 4 = \mathbf{5}$ |
| **$P_4$** | $5$ | $4$ | $10$ | $16$ | $16 - 5 = \mathbf{11}$ | $11 - 4 = \mathbf{7}$ | $10 - 5 = \mathbf{5}$ |
| **Average** | — | — | — | — | **$44 / 4 = 11.00\text{ ms}$** | **$28 / 4 = 7.00\text{ ms}$** | **$11 / 4 = 2.75\text{ ms}$** |

---

#### Solution to Problem 4: Forest $\mathcal{F} = (T_1, T_2)$ to LCRS Binary Tree $B(\mathcal{F})$

1. **ASCII Diagram of Forest $\mathcal{F}$ and Unified LCRS Binary Tree $B(\mathcal{F})$:**
```text
Original Forest F = { T1, T2 }:                  Unified LCRS Binary Tree B(F):
       Tree T1              Tree T2                         [ A ]
        [ A ]                [ H ]                         /     \
      /   |   \              /   \                      [ B ]    [ H ]  <-- Root of T2 on right spine!
    [B]  [C]  [D]          [I]   [J]                    /   \    /
    / \        |            |                         [E]  [C] [I]
  [E] [F]     [G]          [K]                          \    \  / \
                                                        [F] [D][K] [J]
                                                            /
                                                          [G]
```

2. **Degree Counts & Verification of $L = I_2 + 1$ in $B(\mathcal{F})$:**
   - Total nodes $n = 11$ (`A, B, C, D, E, F, G, H, I, J, K`).
   - **Degree-2 nodes in $B(\mathcal{F})$ ($I_2$):** `A` (`L=B, R=H`), `B` (`L=E, R=C`), `I` (`L=K, R=J`) $\implies \mathbf{I_2 = 3}$.
   - **Degree-1 nodes in $B(\mathcal{F})$ ($I_1$):** `E` (`R=F`), `C` (`R=D`), `D` (`L=G`), `H` (`L=I`) $\implies \mathbf{I_1 = 4}$.
   - **Leaf nodes in $B(\mathcal{F})$ ($L = I_0$):** `F, G, K, J` $\implies \mathbf{L = 4}$.
   - **Theorem Check:** $L = I_2 + 1 \implies 4 = 3 + 1$ (**Verified!**).

3. **Traversal Isomorphism Check:**
   - $\text{Preorder}(\mathcal{F}) = \text{Preorder}(T_1) + \text{Preorder}(T_2) =$ **`A B E F C D G H I K J`**
   - $\text{Preorder}(B(\mathcal{F})) =$ **`A B E F C D G H I K J`** *(Exact match!)*
   - $\text{Postorder}(\mathcal{F}) = \text{Postorder}(T_1) + \text{Postorder}(T_2) =$ **`E F B C G D A K I J H`**
   - $\text{Inorder}(B(\mathcal{F})) =$ **`E F B C G D A K I J H`** *(Exact match!)*

---

#### Solution to Problem 5: Unique Binary Tree Reconstruction from `Inorder` and `Preorder`

Given:
- **`Inorder  = [H, D, I, B, E, A, F, C, G]`**
- **`Preorder = [A, B, D, H, I, E, C, F, G]`**

1. **Step-by-Step Recursive Partitioning:**
   - **Root 1 (`Pre[0] = A`):** Splits `Inorder` around `A` into:
     - Left Subtree of `A`: `[H, D, I, B, E]` (5 nodes, `Pre[1..5] = [B, D, H, I, E]`)
     - Right Subtree of `A`: `[F, C, G]` (3 nodes, `Pre[6..8] = [C, F, G]`)
   - **Left of `A` (`Pre[1] = B`):** Splits `[H, D, I, B, E]` around `B` into:
     - Left of `B`: `[H, D, I]` (`Pre[2..4] = [D, H, I]`) $\implies$ Root is **`D`**, with Left **`H`** and Right **`I`**.
     - Right of `B`: `[E]` (`Pre[5] = [E]`) $\implies$ Leaf **`E`**.
   - **Right of `A` (`Pre[6] = C`):** Splits `[F, C, G]` around `C` into:
     - Left of `C`: `[F]` (`Pre[7] = [F]`) $\implies$ Leaf **`F`**.
     - Right of `C`: `[G]` (`Pre[8] = [G]`) $\implies$ Leaf **`G`**.

2. **Reconstructed Binary Tree Diagram:**
```text
                    [ A ]
                   /     \
                [ B ]   [ C ]
                /   \   /   \
              [D]   [E][F]  [G]
              / \
            [H] [I]
```

3. **Resulting Postorder and Level-Order Traversals:**
   - **Postorder ($\text{LRN}$):** **`H I D E B F G C A`**
   - **Level-Order (BFS):** **`A B C D E F G H I`**

---

### Navigation Links
- **Unit 2 Topic 01:** [01 - Queue Fundamentals](./01-Queue-Fundamentals.md)
- **Unit 2 Topic 02:** [02 - Circular Queue, Priority Queue, and Deque](./02-Circular-Priority-and-Deque.md)
- **Unit 2 Topic 03:** [03 - Josephus Problem and CPU Scheduling](./03-Josephus-Problem-and-CPU-Scheduling.md)
- **Unit 2 Topic 04:** [04 - N-Ary Trees, Binary Trees, and Forests](./04-N-Ary-Binary-Trees-and-Forests.md)
- **Unit 2 Topic 05:** [05 - Tree Traversals (Preorder, Inorder, Postorder, Level-Order & N-Ary)](./05-Tree-Traversals.md)
