# Quick Revision Guide: Data Structures and Algorithms in C

---

## Table of Contents
1. [How to Use This Guide](#1-how-to-use-this-guide)
2. [Unit 1: Linked Lists and Stacks — Rapid Revision](#2-unit-1-linked-lists-and-stacks--rapid-revision)
   - 2.1 [C Memory & Pointers Essentials](#21-c-memory--pointers-essentials)
   - 2.2 [Linked List Variants & Pointer Invariants](#22-linked-list-variants--pointer-invariants)
   - 2.3 [Multilists, Sparse Matrices, Skip Lists & Polynomials](#23-multilists-sparse-matrices-skip-lists--polynomials)
   - 2.4 [Stacks, Recursion & Expression Evaluation](#24-stacks-recursion--expression-evaluation)
3. [Unit 2: Queues and Trees — Rapid Revision](#3-unit-2-queues-and-trees--rapid-revision)
   - 3.1 [Queue Variants & Index Arithmetic](#31-queue-variants--index-arithmetic)
   - 3.2 [Queue Applications: Josephus & CPU Scheduling Formulas](#32-queue-applications-josephus--cpu-scheduling-formulas)
   - 3.3 [Trees, Binary Trees, Forests & LCRS Conversion](#33-trees-binary-trees-forests--lcrs-conversion)
   - 3.4 [Tree Traversal Orders & Properties](#34-tree-traversal-orders--properties)
4. [Unit 3: Tree Applications and Graphs — Rapid Revision](#4-unit-3-tree-applications-and-graphs--rapid-revision)
   - 4.1 [Binary Search Trees (BST), Expression Trees & Threaded BSTs](#41-binary-search-trees-bst-expression-trees--threaded-bsts)
   - 4.2 [Binary Heaps & Priority Queues](#42-binary-heaps--priority-queues)
   - 4.3 [Balanced Trees: AVL Trees vs. Splay Trees](#43-balanced-trees-avl-trees-vs-splay-trees)
   - 4.4 [Graphs, Representations, BFS & DFS](#44-graphs-representations-bfs--dfs)
5. [Unit 4: Graph Applications, Tries, and Hashing — Rapid Revision](#5-unit-4-graph-applications-tries-and-hashing--rapid-revision)
   - 5.1 [Graph Connectivity, Components & Shortest Unweighted Paths](#51-graph-connectivity-components--shortest-unweighted-paths)
   - 5.2 [Tries, Suffix Tries & Suffix Trees](#52-tries-suffix-tries--suffix-trees)
   - 5.3 [Hash Functions, Load Factor & Collision Resolution](#53-hash-functions-load-factor--collision-resolution)
6. [Master Comparison Distiller ("X vs. Y" Exam Distinctions)](#6-master-comparison-distiller-x-vs-y-exam-distinctions)
7. [Consolidated Four-Unit Operation Complexity Table](#7-consolidated-four-unit-operation-complexity-table)
8. [Top 15 Exam Traps & Edge-Case Reminders](#8-top-15-exam-traps--edge-case-reminders)

---

## 1. How to Use This Guide

This document is a high-density revision companion covering all four units of the Data Structures and Algorithms in C syllabus. Use it for:
- **Pre-examination formula & invariant lookup**
- **Comparing related data structures and algorithms side-by-side**
- **Verifying Big-O time and auxiliary-space complexities**
- **Checking boundary conditions and pointer-wiring rules**

For full step-by-step derivations, ASCII diagrams, and complete C11 implementations, follow the links in [README.md](./README.md) and [SYLLABUS-COVERAGE.md](./SYLLABUS-COVERAGE.md).

---

## 2. Unit 1: Linked Lists and Stacks — Rapid Revision

### 2.1 C Memory & Pointers Essentials

| Function / Operator | Prototype / Syntax | Behavior & Initialization | Key Pitfall |
| :--- | :--- | :--- | :--- |
| `ptr->member` | `(*ptr).member` | Dereferences structure pointer `ptr` and accesses field `member`. | Undefined behavior if `ptr == NULL` or uninitialized. |
| `malloc(size)` | `void* malloc(size_t size)` | Allocates `size` bytes on heap; contents are **uninitialized (garbage)**. | Must check `if (!ptr)` before dereferencing. |
| `calloc(n, sz)` | `void* calloc(size_t n, size_t sz)` | Allocates `n * sz` bytes on heap and **zero-initializes** all bits. | Zero-fill has minor runtime overhead over `malloc`. |
| `realloc(p, sz)` | `void* realloc(void* p, size_t sz)` | Resizes existing heap block `p` to `sz` bytes, copying existing data. | **Never** write `p = realloc(p, sz)`; if it returns `NULL`, the original pointer `p` is leaked! Always use `tmp = realloc(p, sz)`. |
| `free(p)` | `void free(void* p)` | Returns dynamically allocated block `p` to the heap. | Does **not** set `p = NULL`. Set `p = NULL` manually to prevent dangling-pointer/double-free bugs. |

- **Memory Segments**:
  - **Text/Code**: Compiled machine instructions (read-only).
  - **Data / BSS**: Global and `static` variables (initialized vs. zero-initialized; lifetime = entire program).
  - **Stack**: Automatic local variables and function call activation records (LIFO; deallocated automatically on return).
  - **Heap**: Dynamically allocated blocks via `malloc`/`calloc`/`realloc` (managed explicitly via `free`).

---

### 2.2 Linked List Variants & Pointer Invariants

| List Variant | Node Structure | Empty Invariant | Single-Node Invariant | Boundary Invariant |
| :--- | :--- | :--- | :--- | :--- |
| **Singly Linked List (SLL)** | `data`, `next` | `head == NULL` | `head->next == NULL` | Last node's `next == NULL` |
| **Doubly Linked List (DLL)** | `prev`, `data`, `next` | `head == NULL && tail == NULL` | `head == tail`, `prev == NULL`, `next == NULL` | `head->prev == NULL`, `tail->next == NULL` |
| **Circular Singly Linked List (CLL)** | `data`, `next` | `tail == NULL` (or `head == NULL`) | `node->next == node` | `tail->next == head` (no `NULL` links) |

#### Crucial Pointer-Rewiring Rules (Order Matters!)
1. **SLL Insert After `prev_node`**:
   ```c
   new_node->next = prev_node->next; /* 1. Connect new_node to rest of list FIRST */
   prev_node->next = new_node;       /* 2. Update prev_node->next SECOND */
   ```
2. **DLL Insert Between `A` and `B` (`A <-> B`)**:
   ```c
   new_node->prev = A;
   new_node->next = B;
   A->next = new_node;
   if (B != NULL) B->prev = new_node;
   ```
3. **Iterative SLL Reversal (3-Pointer Invariant)**:
   ```c
   Node *prev = NULL, *curr = head, *next = NULL;
   while (curr != NULL) {
       next = curr->next; /* Save next */
       curr->next = prev; /* Reverse link */
       prev = curr;       /* Advance prev */
       curr = next;       /* Advance curr */
   }
   head = prev;
   ```
4. **Why CLL Preferentially Tracks `tail` (`last`) Instead of `head`**:
   - With only `head`: Insert at head or tail requires traversing the entire circle to update `tail->next`, costing $O(n)$.
   - With `tail`: `head` is immediately accessible as `tail->next`! Both **insert at beginning** and **insert at end** run in **$O(1)$ time**.

---

### 2.3 Multilists, Sparse Matrices, Skip Lists & Polynomials

- **Sparse Matrix Condition**: A matrix of order $R \times C$ with $nnz$ non-zero elements is sparse when $nnz \ll R \times C$.
- **Triplet (3-Tuple / COO) Array Representation**:
  - Dimensions: `(nnz + 1) x 3` integer array.
  - Row `0` (Header): `[Total_Rows, Total_Cols, nnz]`.
  - Rows `1 .. nnz`: `[row_index, col_index, non_zero_value]` ordered in row-major order.
  - Space saved when $3(nnz + 1) < R \times C$.
- **Orthogonal Cross-Linked Multilist Sparse Matrix**:
  - Every non-zero entry is a shared node `{row, col, val, right, down}` linked simultaneously into its circular/singly **row list** (`right`) and **column list** (`down`).
- **Skip List**:
  - Probabilistic multi-level linked list providing express lanes over a sorted bottom list ($L_0$).
  - A node at level $k$ is promoted to level $k+1$ with probability $p$ (typically $p = 1/2$).
  - Expected max level: $L_{\max} = \lceil \log_{1/p} n \rceil = \lceil \log_2 n \rceil$.
  - Expected number of pointers per node: $\frac{1}{1 - p} = 2$ (for $p = 1/2$).
  - **Expected Time**: $O(\log n)$ for search, insert, and delete. **Worst-Case Time**: $O(n)$ (if coin flips degenerate).
- **Polynomial Linked List Invariants**:
  - Nodes `{coeff, exp, next}` stored in **strictly decreasing order of exponents** ($e_1 > e_2 > \dots > e_k \ge 0$).
  - Terms with equal exponents are combined (`c1 + c2`); if `c1 + c2 == 0`, the node is **deleted/omitted** (never store zero-coefficient nodes).
  - Addition/Subtraction of $m$-term and $n$-term polynomials: **$O(m + n)$** via two-pointer merge.
  - Multiplication: **$O(m \cdot n)$** term products ($O(m \cdot n^2)$ or $O(m n \log(mn))$ depending on merging strategy).

---

### 2.4 Stacks, Recursion & Expression Evaluation

- **Stack ADT (LIFO — Last-In, First-Out)**:
  - **Array Stack**: `top` initialized to `-1`.
    - Empty: `top == -1` | Full: `top == CAPACITY - 1`
    - Push: `data[++top] = x` | Pop: `return data[top--]`
  - **Linked-List Stack**: `top` MUST be the `head` of the singly linked list so both `push` (insert at head) and `pop` (delete at head) are worst-case **$O(1)$**.

#### Operator Precedence & Associativity Table
| Operator(s) | Description | Precedence Rank | Associativity |
| :---: | :--- | :---: | :---: |
| `(` `)` | Parentheses (Grouping) | Highest (Special) | N/A |
| `^` | Exponentiation | 3 (High) | **Right-to-Left** (`a ^ b ^ c = a ^ (b ^ c)`) |
| `*`, `/`, `%` | Multiplication, Division, Modulo | 2 (Medium) | **Left-to-Right** |
| `+`, `-` | Binary Addition, Subtraction | 1 (Low) | **Left-to-Right** |

#### Expression Conversion & Evaluation Rules
1. **Infix $\to$ Postfix (Shunting-Yard)**:
   - **Operand**: Append directly to output.
   - **`(`**: Push onto operator stack.
   - **`)`**: Pop and output operators until `(` is popped (discard both parentheses).
   - **Operator `op`**: Pop and output operators from stack while `top` is not `(` AND:
     - `prec(top) > prec(op)`, OR
     - `prec(top) == prec(op)` AND `op` is **Left-Associative**.
     - *(If `op` is Right-Associative like `^`, do NOT pop an equal-precedence `^`!)*
     - Then push `op`.
2. **Infix $\to$ Prefix**:
   - Reverse infix string, swap `(` $\leftrightarrow$ `)`, run Shunting-Yard with **inverted tie-breaking** (pop on strictly greater precedence `prec(top) > prec(op)` for left-associative operators, and `prec(top) >= prec(op)` for right-associative operators), then reverse the output string.
3. **Postfix Evaluation (Scan Left-to-Right)**:
   - Push operands. On operator `op`: **`op2 = pop()`, `op1 = pop()`**, push `op1 op op2`. *(Crucial for `-`, `/`, `^`!)*
4. **Prefix Evaluation (Scan Right-to-Left)**:
   - Push operands. On operator `op`: **`op1 = pop()`, `op2 = pop()`**, push `op1 op op2`.

---

## 3. Unit 2: Queues and Trees — Rapid Revision

### 3.1 Queue Variants & Index Arithmetic

- **Queue ADT (FIFO — First-In, First-Out)**: Insertion (`enqueue`) at `rear`; deletion (`dequeue`) at `front`.
- **Why Simple Linear Array Queue Fails**: As `front` and `rear` advance rightward, slots `0 .. front-1` become permanently unusable ("**False Overflow**" when `rear == CAPACITY - 1` even though `count < CAPACITY`).

#### Circular Queue Modular Arithmetic (`CAPACITY = C`)
| Operation / Check | Using Explicit `count` Field (Recommended) | Using 1 Sacrificed Slot (No `count`) |
| :--- | :--- | :--- |
| **Initialization** | `front = 0, rear = -1, count = 0` | `front = 0, rear = 0` (Max elements = $C - 1$) |
| **Empty Condition** | `count == 0` | `front == rear` |
| **Full Condition** | `count == C` | `(rear + 1) % C == front` |
| **Advance Rear (Enqueue)** | `rear = (rear + 1) % C; count++;` | `rear = (rear + 1) % C;` |
| **Advance Front (Dequeue)** | `front = (front + 1) % C; count--;` | `front = (front + 1) % C;` |
| **Current Size** | `count` | `(rear - front + C) % C` |

- **Deque (Double-Ended Queue)**:
  - **Insert Front (Decrement Wraparound)**: `front = (front - 1 + C) % C`
  - **Delete Rear (Decrement Wraparound)**: `rear = (rear - 1 + C) % C`
  - **Input-Restricted Deque**: Insert at ONE end only; delete from BOTH ends.
  - **Output-Restricted Deque**: Insert at BOTH ends; delete from ONE end only.

---

### 3.2 Queue Applications: Josephus & CPU Scheduling Formulas

1. **Josephus Problem ($n$ people, step $k$)**:
   - **Queue Simulation**: Enqueue $1 \dots n$. Repeat $n - 1$ times: perform $k - 1$ rotations (`enqueue(dequeue())`), then permanently eliminate the $k$-th person (`dequeue()`). Complexity: **$O(n \cdot k)$** time, $O(n)$ space.
   - **Mathematical Recurrence ($O(n)$ time, $O(1)$ space)**:
     - 0-indexed ($0 \dots n-1$): $J_0(1, k) = 0$, $\quad J_0(n, k) = (J_0(n-1, k) + k) \bmod n$.
     - 1-indexed ($1 \dots n$): $J_1(1, k) = 1$, $\quad J_1(n, k) = ((J_1(n-1, k) + k - 1) \bmod n) + 1$.
2. **CPU Scheduling Metrics**:
   - **Turnaround Time ($\text{TAT}$)**: $\text{TAT}_i = \text{Completion Time (CT)}_i - \text{Arrival Time (AT)}_i$
   - **Waiting Time ($\text{WT}$)**: $\text{WT}_i = \text{TAT}_i - \text{Burst Time (BT)}_i$
   - **Response Time ($\text{RT}$)**: $\text{RT}_i = \text{First CPU Start Time}_i - \text{AT}_i$
   - **Round-Robin (RR) Scheduling Rule**: Maintain a FIFO Ready Queue. When a running process's time quantum $q$ expires at time $t$, **enqueue any newly arrived processes at time $\le t$ FIRST**, and then re-enqueue the preempted process at the rear of the queue!

---

### 3.3 Trees, Binary Trees, Forests & LCRS Conversion

#### Essential Mathematical Formulas for Trees
| Property | Binary Tree Formula | General $N$-ary Tree Formula |
| :--- | :--- | :--- |
| **Edges for $n$ vertices** | $\|E\| = n - 1$ | $\|E\| = n - 1$ |
| **Edges in Forest of $k$ trees** | $\|E\| = \|V\| - k$ | $\|E\| = \|V\| - k$ |
| **Max nodes at depth $d$ (root $d=0$)** | $2^d$ | $N^d$ |
| **Max nodes for height $h$ (edges)** | $2^{h+1} - 1$ | $\frac{N^{h+1} - 1}{N - 1} \quad (N > 1)$ |
| **Min height for $n$ nodes (edges)** | $\lfloor \log_2 n \rfloor$ | $\lceil \log_N(n(N-1) + 1) \rceil - 1$ |
| **Leaf vs. Internal Node Theorem** | $L = I_2 + 1$ | $L = 1 + \sum_{i=2}^N (i - 1)I_i$ |
| **Null Child Links in $n$-node Tree** | $n + 1$ wasted `NULL` pointers | $(N - 1)n + 1$ wasted `NULL` pointers |

#### First-Child / Next-Sibling (LCRS / Knuth Transform)
- `b_node->left` = **First (leftmost) child** in the original $N$-ary tree.
- `b_node->right` = **Immediate next sibling** to the right in the original $N$-ary tree (or the **root of the next tree $T_{i+1}$** when converting a forest!).
- **Single $N$-ary Tree vs. Forest Invariant**:
  - Converted single $N$-ary tree: `binary_root->right == NULL` always.
  - Converted forest of $k \ge 2$ trees: `binary_root->right` points to $\text{root}(T_2)$ (`!= NULL`), chaining $T_1, T_2, \dots, T_k$ along the right spine.

---

### 3.4 Tree Traversal Orders & Properties

| Traversal | Order | BST / Expression Property | N-ary / LCRS Correspondence |
| :--- | :--- | :--- | :--- |
| **Preorder** | $\text{Root} \to \text{Left} \to \text{Right}$ | Prefix expression; top-down tree cloning | Matches **Preorder** of LCRS Binary Tree |
| **Inorder** | $\text{Left} \to \text{Root} \to \text{Right}$ | **Sorted ascending order** on BST; Infix expression | Matches **Postorder** of original N-ary Tree! |
| **Postorder** | $\text{Left} \to \text{Right} \to \text{Root}$ | Postfix expression; **Mandatory for `free()`** | N/A (requires visiting right siblings before parent) |
| **Level-Order** | Level $0, 1, \dots, h$ (Left-to-Right) | Uses FIFO Queue; finds shallowest leaf | Breadth-First traversal |

- **Unique Binary Tree Reconstruction**:
  - **Inorder + Preorder** $\implies$ Unique binary tree.
  - **Inorder + Postorder** $\implies$ Unique binary tree.
  - **Preorder + Postorder** $\implies$ **NOT unique** in general (cannot tell if a single child is left or right), **unless** the tree is known to be a **Full (Strict) Binary Tree**!

---

## 4. Unit 3: Tree Applications and Graphs — Rapid Revision

### 4.1 Binary Search Trees (BST), Expression Trees & Threaded BSTs

- **BST Invariant**: Every key in `root->left` $< \text{root->key} <$ every key in `root->right`. *(Warning: Checking only immediate children is insufficient; must track `(min_allowed, max_allowed)` range recursively!)*
- **BST Deletion (3 Cases)**:
  1. **Leaf (0 children)**: Free node, return `NULL`.
  2. **1 Child**: Bypass node, return the single child pointer, free node.
  3. **2 Children**: Replace `node->key` with its **Inorder Successor** (`find_min(node->right)`) or **Inorder Predecessor** (`find_max(node->left)`), then recursively delete that successor/predecessor from the respective subtree.
- **Expression Tree**:
  - Leaves = Operands; Internal nodes = Operators.
  - Built from Postfix using a stack of `ExprNode*`: operands push new leaf; operator pops `right`, pops `left`, attaches both, and pushes new subtree root.
- **Threaded BST**:
  - Replaces the $n + 1$ `NULL` pointers in an $n$-node BST with threads pointing to the **Inorder Predecessor** (`lthread = true`) and **Inorder Successor** (`rthread = true`).
  - Enables **stackless, recursion-free Inorder traversal in $O(n)$ time and $O(1)$ auxiliary space**.

---

### 4.2 Binary Heaps & Priority Queues

- **Two Mandatory Heap Invariants**:
  1. **Structural (Shape) Property**: Complete Binary Tree (stored compactly in an array with zero internal gaps; height $h = \lfloor \log_2 n \rfloor$).
  2. **Heap-Order Property**:
     - **Max-Heap**: `A[parent(i)] >= A[i]` (Max at root `A[0]`).
     - **Min-Heap**: `A[parent(i)] <= A[i]` (Min at root `A[0]`).
- **Array Index Formulas**:
  - **0-Based Indexing**: `parent(i) = (i - 1) / 2`, `left(i) = 2*i + 1`, `right(i) = 2*i + 2`. Leaves are at indices $\lfloor n/2 \rfloor \dots n-1$.
  - **1-Based Indexing**: `parent(i) = i / 2`, `left(i) = 2*i`, `right(i) = 2*i + 1`. Leaves are at indices $\lfloor n/2 \rfloor + 1 \dots n$.
- **Core Complexities**:
  - `peek`: $O(1)$ | `insert` (sift-up): $O(\log n)$ | `extract_root` (sift-down / `heapify`): $O(\log n)$.
  - **Bottom-Up `build_heap` (Floyd's Algorithm)**: Run `sift_down` from $i = \lfloor n/2 \rfloor - 1$ down to $0$ $\implies$ **$O(n)$ linear time**!

---

### 4.3 Balanced Trees: AVL Trees vs. Splay Trees

#### AVL Tree Balance Factor & Rotations
$$\text{Balance Factor } BF(v) = \text{height}(v\to\text{left}) - \text{height}(v\to\text{right}) \in \{-1, 0, +1\}$$

| Imbalance Case | Condition at Unbalanced Node $z$ and Heavy Child $y$ | Fix (Rotation Required) |
| :--- | :--- | :--- |
| **LL (Left-Left)** | $BF(z) = +2$ and $BF(y) \ge 0$ | Single **Right Rotation** at $z$ |
| **RR (Right-Right)** | $BF(z) = -2$ and $BF(y) \le 0$ | Single **Left Rotation** at $z$ |
| **LR (Left-Right)** | $BF(z) = +2$ and $BF(y) = -1$ | **Left Rotation at $y$**, then **Right Rotation at $z$** |
| **RL (Right-Left)** | $BF(z) = -2$ and $BF(y) = +1$ | **Right Rotation at $y$**, then **Left Rotation at $z$** |

#### Splay Tree Restructuring Steps (Moving Target Node $x$ to Root)
| Splay Step | Structural Configuration (Parent $p$, Grandparent $g$) | Rotation Sequence |
| :--- | :--- | :--- |
| **Zig (or Zag)** | $p$ is the tree root ($g == \text{NULL}$) | Single rotation on $p$ to bring $x$ to root |
| **Zig-Zig (or Zag-Zag)** | $x$ and $p$ are **both left** (or **both right**) children | Rotate **grandparent $g$ FIRST**, then rotate **parent $p$**! |
| **Zig-Zag (or Zag-Zig)** | $x$ is right child of left child $p$ (or left child of right child $p$) | Rotate **parent $p$ FIRST**, then rotate **grandparent $g$** |

> **Critical Exam Distinction**:
> - **AVL Tree**: Strict height balance ($h < 1.44 \log_2 n$), guaranteeing **$O(\log n)$ worst-case time for every individual operation**, at the cost of storing height/BF metadata.
> - **Splay Tree**: Self-adjusting cache-friendly tree storing **zero balance metadata**. A single operation can take **$O(n)$ worst-case time**, but any sequence of $m$ operations takes **$O(m \log n)$ amortized time** ($O(\log n)$ amortized per operation).

---

### 4.4 Graphs, Representations, BFS & DFS

- **Handshaking Lemma**:
  - Undirected graph: $\sum_{v \in V} \deg(v) = 2|E|$.
  - Directed graph: $\sum_{v \in V} \deg^-(v) = \sum_{v \in V} \deg^+(v) = |E|$.

| Feature / Operation | Adjacency Matrix ($V \times V$) | Adjacency List ($V$ lists) |
| :--- | :--- | :--- |
| **Space Complexity** | $\Theta(V^2)$ (Best for **Dense** graphs) | $\Theta(V + E)$ (Best for **Sparse** graphs) |
| **Check if edge $(u, v)$ exists** | **$O(1)$** | $O(\deg(u))$ |
| **Iterate over all neighbors of $u$** | $O(V)$ | **$O(\deg(u))$** |
| **Insert Edge $(u, v)$** | $O(1)$ | $O(1)$ (head insert) |
| **Remove Edge $(u, v)$** | $O(1)$ | $O(\deg(u))$ |
| **Full BFS or DFS Traversal** | **$O(V^2)$** | **$O(V + E)$** |

---

## 5. Unit 4: Graph Applications, Tries, and Hashing — Rapid Revision

### 5.1 Graph Connectivity, Components & Shortest Unweighted Paths

- **Undirected Connectivity**: Connected iff a single BFS/DFS from vertex $0$ visits all $|V|$ vertices (`visited_count == V`).
- **Counting Connected Components**: Loop `v = 0 .. V-1`; each time `!visited[v]`, increment `components++` and launch BFS/DFS from `v`. Time: $O(V + E)$ (Adjacency List) or $O(V^2)$ (Adjacency Matrix).
- **BFS vs. DFS for Paths from $s$ to $t$**:
  - **BFS** explores vertices in concentric layers of hop distance $d = 0, 1, 2, \dots$. By induction, the first time $t$ is reached, the path reconstructed via `parent[curr]` is guaranteed to be a **Shortest Path (minimum number of edges)** in an unweighted graph.
  - **DFS** finds *a* valid path if one exists (reachability), but **does NOT guarantee the shortest path**.

---

### 5.2 Tries, Suffix Tries & Suffix Trees

| Structure | What It Stores | Time (Search Pattern $P$ of len $q$) | Space / Nodes (String $S$ of len $m$, Dictionary of $N$ words) |
| :--- | :--- | :--- | :--- |
| **Standard Trie** | Set of $N$ dictionary strings (shared prefixes) | $O(q)$ exact/prefix search & insert | $O(\|\Sigma\| \cdot \text{total chars})$ space |
| **Suffix Trie** | All $m$ suffixes of a single string $S\$$ as an uncompressed trie | $O(q)$ substring search | **$O(m^2)$** nodes and $O(m^2)$ construction time in worst case |
| **Suffix Tree** | Path-compressed Suffix Trie of $S\$$ (single-child chains merged into `[start, end]` edge labels) | $O(q)$ substring search | **$\le 2m - 1$ nodes** ($m$ leaves, $\le m-1$ internal nodes); $O(m)$ space with index pairs; $O(m^2)$ naive build vs. **$O(m)$ Ukkonen's algorithm** |

- **Why Append Terminal Symbol `$` ($\notin \Sigma$)**: Ensures no suffix of $S\$$ is a proper prefix of another suffix, guaranteeing every suffix terminates at its own distinct **leaf node**.
- **Trie Deletion (4 Cases)**:
  1. Key not found $\implies$ do nothing.
  2. Key is a prefix of a longer key (`has_children(curr) == true`) $\implies$ set `is_end_of_word = false`, free **no** nodes.
  3. Key shares a proper prefix with another key $\implies$ free nodes bottom-up only until reaching the lowest ancestor that has another child or `is_end_of_word == true`.
  4. Key is completely unique $\implies$ free all nodes along the key path up to the root's child.

---

### 5.3 Hash Functions, Load Factor & Collision Resolution

- **Load Factor**: $\alpha = \frac{n}{m}$ ($n$ = stored elements, $m$ = table size / buckets).
- **Collision Resolution Formulas ($i = 0, 1, \dots, m-1$)**:

| Method | Probe Formula $h(k, i)$ | Clustering Behavior | Load Factor & Capacity Rules |
| :--- | :--- | :--- | :--- |
| **Separate Chaining** | Bucket $h(k) = k \bmod m$ (linked list) | None (chains grow linearly with $\alpha$) | $\alpha$ can exceed $1.0$; typically rehash when $\alpha > 1.0$. |
| **Linear Probing** | $(h'(k) + i) \bmod m$ | **Primary Clustering** (contiguous runs merge and grow rapidly) | $0 \le \alpha < 1$; performance degrades sharply for $\alpha > 0.7$. |
| **Quadratic Probing** | $(h'(k) + c_1 i + c_2 i^2) \bmod m$ (usually $(h'(k) + i^2) \bmod m$) | **Secondary Clustering** (only keys with identical $h'(k)$ collide along path) | Guaranteed to find an empty slot if **$m$ is prime and $\alpha \le 0.5$**. |
| **Double Hashing** | $(h_1(k) + i \cdot h_2(k)) \bmod m$ | **No Clustering** (near-ideal uniform probing) | $h_2(k)$ must **never be $0$** and $\gcd(h_2(k), m) = 1$ (e.g., $m$ prime, $h_2(k) = R - (k \bmod R)$ with prime $R < m$). |

- **Why Open Addressing Requires Tombstones (`DELETED` State)**:
  - During `search(k)`, probing stops only at an `EMPTY` slot. If a deleted slot were reset to `EMPTY`, any key inserted later in that same probe chain would become **unreachable**!
  - Marking deleted slots as `DELETED` (Tombstone) allows `search` to keep probing past them while allowing `insert` to reuse the first `DELETED` slot encountered.
- **Rehashing**:
  - Triggered when $\alpha$ exceeds threshold (e.g., $\alpha > 0.7$ in open addressing).
  - Allocate a new table of prime size $m' \approx 2m$, iterate through old table, and **recompute $h_{m'}(k) = k \bmod m'$ to re-insert every active `OCCUPIED` key** (purging all `DELETED` tombstones). Single rehash costs $O(n + m')$; amortized insertion remains **$O(1)$**.

---

## 6. Master Comparison Distiller ("X vs. Y" Exam Distinctions)

| Comparison Pair | Structure / Algorithm X | Structure / Algorithm Y | Decisive Exam Distinction |
| :--- | :--- | :--- | :--- |
| **SLL vs. DLL** | 1 pointer (`next`) per node; forward-only traversal. | 2 pointers (`prev`, `next`); bidirectional traversal. | Deleting a given node pointer `p` is **$O(1)$ in DLL**, but **$O(n)$ in SLL** (must find predecessor). |
| **Array Stack vs. Linked Stack** | Contiguous array; $O(1)$ cache-friendly access; fixed cap or $O(n)$ `realloc`. | Dynamic nodes; $O(1)$ worst-case push/pop without resize spikes. | Linked stack uses extra pointer memory per node and has poor cache locality, but never suffers false capacity limits. |
| **Linear Queue vs. Circular Queue** | `rear++`, `front++` without modulo wraparound. | `rear = (rear+1)%C`, `front = (front+1)%C`. | Linear queue suffers **false overflow** when `rear == C-1` even if `front > 0`; Circular queue reuses vacated front slots in $O(1)$. |
| **Full vs. Complete vs. Perfect Binary Tree** | **Full**: Every node has $0$ or $2$ children. | **Complete**: All levels full except possibly last, which is packed left-to-right. | **Perfect** has $2^{h+1}-1$ nodes (all leaves at depth $h$). Every Perfect tree is Full and Complete, but not vice versa! |
| **Binary Tree vs. BST** | No ordering constraint on keys; search is $O(n)$. | $\text{Left} < \text{Root} < \text{Right}$ at every subtree. | Inorder traversal of a BST is strictly sorted; search/insert/delete is $O(\log n)$ average, $O(n)$ skewed. |
| **BST vs. Binary Heap** | Total left-to-right ordering; pointer-based; height $O(\log n)$ to $O(n)$. | Parent-child dominance only; array-based Complete Binary Tree; height always $\lfloor \log_2 n \rfloor$. | Heap finds min/max in $O(1)$ and builds in $O(n)$, but searching an arbitrary key in a Heap takes **$O(n)$**! |
| **AVL Tree vs. Splay Tree** | Stores height/BF; guarantees $|BF| \le 1$. | Stores no balance metadata; splays accessed key to root. | AVL guarantees **worst-case $O(\log n)$** per op; Splay guarantees **amortized $O(\log n)$** with $O(n)$ single-op worst case. |
| **BFS vs. DFS** | Uses FIFO Queue; explores level-by-level. | Uses LIFO Stack / Recursion; explores deep paths first. | **BFS guarantees shortest path in unweighted graphs**; DFS does not, but excels at cycle/topological/backtracking analysis. |
| **Trie vs. Hash Table** | Tree indexed by character sequence ($O(m)$ worst-case time). | Array indexed by $h(k)$ ($O(1)$ expected, $O(n)$ worst-case). | Tries support **prefix search, autocomplete, and alphabetical iteration** without collisions; Hash tables do not. |
| **Primary vs. Secondary Clustering** | **Linear Probing**: Any collision in a cluster expands that cluster. | **Quadratic Probing**: Only keys sharing the exact same initial hash $h'(k)$ follow the same probe path. | **Double Hashing** eliminates both by making the step size $h_2(k)$ dependent on the key itself. |

---

## 7. Consolidated Four-Unit Operation Complexity Table

Variables: $n$ = elements/nodes, $h$ = tree height, $V$ = graph vertices, $E$ = graph edges, $m$ = string length (or hash table capacity), $|\Sigma|$ = alphabet size, $\alpha = n/m$ = load factor.

| Data Structure | Access / Search | Insertion | Deletion | Auxiliary Space | Notes / Conditions |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **Dynamic Array** | $O(1)$ by index / $O(n)$ search | $O(1)$ amort. (end) / $O(n)$ (mid) | $O(1)$ (end) / $O(n)$ (mid) | $O(1)$ | Resize copies $n$ items in $O(n)$ |
| **Singly Linked List (SLL)** | $O(n)$ | $O(1)$ at head / $O(n)$ at tail* | $O(1)$ at head / $O(n)$ at tail | $O(1)$ | *$O(1)$ insert at tail if `tail` pointer kept |
| **Doubly Linked List (DLL)** | $O(n)$ | $O(1)$ at head/tail/known node | $O(1)$ at head/tail/known node | $O(1)$ | 2 pointers (`prev`, `next`) per node |
| **Circular Linked List (`tail`)** | $O(n)$ | $O(1)$ at both head and tail | $O(1)$ at head / $O(n)$ at tail | $O(1)$ | `tail->next` gives $O(1)$ access to `head` |
| **Skip List** | $O(\log n)$ avg / $O(n)$ worst | $O(\log n)$ avg / $O(n)$ worst | $O(\log n)$ avg / $O(n)$ worst | $O(\log n)$ | Probabilistic multi-level links |
| **Stack (Array / Linked)** | $O(1)$ (`peek`) | $O(1)$ (`push`) | $O(1)$ (`pop`) | $O(1)$ | LIFO discipline |
| **Queue / Circular Queue / Deque** | $O(1)$ (`front`/`rear`) | $O(1)$ (`enqueue`) | $O(1)$ (`dequeue`) | $O(1)$ | FIFO / double-ended discipline |
| **Binary Tree (General)** | $O(n)$ | $O(1)$ at known link | $O(n)$ | $O(h)$ | $h \in [\lfloor \log_2 n \rfloor, n-1]$ |
| **Binary Search Tree (Balanced)** | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | When $h = \Theta(\log n)$ |
| **Binary Search Tree (Skewed)** | $O(n)$ | $O(n)$ | $O(n)$ | $O(n)$ | Sorted insertions without rebalancing |
| **Threaded BST** | $O(h)$ | $O(h)$ | $O(h)$ | **$O(1)$** (Inorder) | Stackless Inorder traversal in $O(n)$ time, $O(1)$ space |
| **Binary Heap (Min / Max)** | $O(1)$ (top) / $O(n)$ (arbitrary) | $O(\log n)$ | $O(\log n)$ (extract root) | $O(1)$ | Bottom-up `build_heap` is **$O(n)$** |
| **AVL Tree** | **$O(\log n)$ worst** | **$O(\log n)$ worst** | **$O(\log n)$ worst** | $O(\log n)$ | At most 1 rotation (single/double) on insert |
| **Splay Tree** | **$O(\log n)$ amort.** / $O(n)$ worst | **$O(\log n)$ amort.** / $O(n)$ worst | **$O(\log n)$ amort.** / $O(n)$ worst | $O(h)$ | Self-adjusting; no balance factor stored |
| **Graph (Adjacency Matrix)** | $O(1)$ edge check | $O(1)$ edge insert | $O(1)$ edge delete | $\Theta(V^2)$ total | BFS / DFS takes **$O(V^2)$** time |
| **Graph (Adjacency List)** | $O(\deg(u))$ edge check | $O(1)$ edge insert | $O(\deg(u))$ edge delete | $\Theta(V + E)$ total | BFS / DFS takes **$O(V + E)$** time |
| **Standard Trie** | $O(m)$ | $O(m)$ | $O(m)$ | $O(m)$ | $m$ = key length; space $O(\|\Sigma\| \cdot \text{nodes})$ |
| **Suffix Tree (String len $m$)** | $O(q)$ (pattern len $q$) | $O(m^2)$ naive / $O(m)$ Ukkonen | N/A (static index) | $\le 2m - 1$ nodes | Substring check in $O(\|P\|)$ time |
| **Hash Table (Chaining / Open)** | $O(1)$ expected / $O(n)$ worst | $O(1)$ amort. / $O(n)$ worst | $O(1)$ expected / $O(n)$ worst | $O(1)$ | Requires good hash function & controlled $\alpha$ |

---

## 8. Top 15 Exam Traps & Edge-Case Reminders

1. **Order of Pointer Assignment in Linked List Insertion**: Always point `new_node->next` to the successor **before** overwriting `prev_node->next = new_node`.
2. **Last Node Dequeue in Linked Queue**: When `front` becomes `NULL` after `dequeue()`, you **must** also set `rear = NULL`, or `rear` becomes a dangling pointer!
3. **Operand Order in Postfix Evaluation**: On encountering `-`, `/`, or `^`, the **first** popped value is the **right** operand (`op2 = pop()`) and the **second** popped value is the **left** operand (`op1 = pop()`).
4. **Right-Associativity of `^` in Shunting-Yard**: When scanning `^` and the top of the operator stack is also `^`, **push** the new `^` (do not pop!) because $2^{3^2} = 2^{(3^2)} = 512$, not $(2^3)^2 = 64$.
5. **Postorder Deallocation of Trees**: Always `free(root->left)` and `free(root->right)` **before** calling `free(root)`. Preorder freeing causes immediate use-after-free undefined behavior.
6. **BST Validation Trap**: Checking only `node->left->key < node->key && node->right->key > node->key` is **wrong** (a node in the left subtree's right branch could exceed the root!). Pass `(min_val, max_val)` bounds recursively.
7. **LCRS Forest vs. Single Tree Root**: In LCRS representation, a single N-ary tree always has `root->right == NULL`, whereas a forest of $k \ge 2$ trees chains the roots $T_2, \dots, T_k$ along `root->right`.
8. **AVL Rotation Height Update Order**: During `rotate_left(x)` or `rotate_right(y)`, always recompute the height of the **demoted (lower) node first**, and the **promoted (upper) node second**.
9. **Splay Tree Zig-Zig vs. AVL Double Rotation**: In Splay Tree **Zig-Zig** (homogeneous left-left or right-right), you MUST rotate the **grandparent $g$ first**, then the parent $p$. Rotating $p$ first fails to halve tree depth!
10. **0-Based vs. 1-Based Heap Indices**: In 0-based arrays, `left(0) = 1, right(0) = 2, parent(i) = (i-1)/2`. In 1-based arrays, `left(1) = 2, right(1) = 3, parent(i) = i/2`. Never mix the two!
11. **Marking `visited` in BFS**: Always set `visited[v] = true` **at the moment `v` is enqueued**, NOT when it is dequeued. Otherwise, the same vertex can be enqueued multiple times!
12. **BFS vs. DFS Shortest Path**: BFS guarantees shortest paths in **unweighted** graphs ($O(V+E)$). Ordinary DFS does **not** guarantee shortest paths.
13. **Trie Prefix vs. Full Word**: Reaching the end of a string in a Trie only proves it is a **prefix** unless `curr->is_end_of_word == true`.
14. **Open-Addressing Deletion (Tombstones)**: Never set a deleted slot in Linear/Quadratic Probing or Double Hashing to `EMPTY`; set it to `DELETED` so searches for keys probed past that slot do not terminate prematurely.
15. **Double Hashing Zero Step**: The secondary hash function $h_2(k)$ must **never return $0$**, and must be coprime to table capacity $m$.
