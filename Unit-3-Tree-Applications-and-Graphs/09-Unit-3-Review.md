# Unit 3 — Topic 9: Comprehensive Review and Exam Preparation Guide

---

## Table of Contents

1. [Important Definitions Across Unit 3](#1-important-definitions-across-unit-3)
   - 1.1 [Binary Search Trees (BSTs) and Expression Trees](#11-binary-search-trees-bsts-and-expression-trees)
   - 1.2 [Threaded Binary Trees and Binary Heaps](#12-threaded-binary-trees-and-binary-heaps)
   - 1.3 [Self-Balancing BSTs: AVL Trees and Splay Trees](#13-self-balancing-bsts-avl-trees-and-splay-trees)
   - 1.4 [Graph Fundamentals, Representations, BFS, and DFS](#14-graph-fundamentals-representations-bfs-and-dfs)
2. [Essential Diagrams and Mathematical Formulas](#2-essential-diagrams-and-mathematical-formulas)
   - 2.1 [Implicit Array Indexing Formulas (Complete Binary Trees & Heaps)](#21-implicit-array-indexing-formulas-complete-binary-trees--heaps)
   - 2.2 [The $n + 1$ Null Pointers Theorem and Threaded Binary Trees](#22-the-n--1-null-pointers-theorem-and-threaded-binary-trees)
   - 2.3 [AVL Balance Factor and the 4 Rotation Cases (LL, RR, LR, RL)](#23-avl-balance-factor-and-the-4-rotation-cases-ll-rr-lr-rl)
   - 2.4 [Splay Tree Operations: Zig, Zig-Zig, and Zig-Zag](#24-splay-tree-operations-zig-zig-zig-and-zig-zag)
   - 2.5 [Graph Theory Formulas and the Handshaking Lemma](#25-graph-theory-formulas-and-the-handshaking-lemma)
3. [Commonly Confused Concepts (Exam Pitfall Disambiguation)](#3-commonly-confused-concepts-exam-pitfall-disambiguation)
   - 3.1 [Binary Search Tree (BST) vs. Binary Heap](#31-binary-search-tree-bst-vs-binary-heap)
   - 3.2 [AVL Double Rotations vs. Splay Tree Zig-Zig & Zig-Zag](#32-avl-double-rotations-vs-splay-tree-zig-zig--zig-zag)
   - 3.3 [Adjacency Matrix $\Theta(V^2)$ vs. Adjacency List $\Theta(V + E)$ Traversal](#33-adjacency-matrix-thetav2-vs-adjacency-list-thetav--e-traversal)
   - 3.4 [Marking `visited` on Enqueue vs. Dequeue in Breadth-First Search](#34-marking-visited-on-enqueue-vs-dequeue-in-breadth-first-search)
4. [Consolidated Operation-Complexity Table for Unit 3](#4-consolidated-operation-complexity-table-for-unit-3)
5. [Short-Answer Questions (10 Exam Questions)](#5-short-answer-questions-10-exam-questions)
6. [Conceptual Questions (8 Deep-Thinking Questions)](#6-conceptual-questions-8-deep-thinking-questions)
7. [Algorithm-Writing Questions (5 Pseudocode Problems)](#7-algorithm-writing-questions-5-pseudocode-problems)
8. [C Programming Questions (5 Coding Problems)](#8-c-programming-questions-5-coding-problems)
9. [Worked Problem-Solving Questions (6 Full Step-by-Step Traces)](#9-worked-problem-solving-questions-6-full-step-by-step-traces)
   - 9.1 [Problem 1: BST Construction and Two-Child Deletion Trace](#91-problem-1-bst-construction-and-two-child-deletion-trace)
   - 9.2 [Problem 2: Expression Tree Construction from Postfix and Recursive Evaluation](#92-problem-2-expression-tree-construction-from-postfix-and-recursive-evaluation)
   - 9.3 [Problem 3: Linear-Time $O(n)$ Bottom-Up Max-Heap Construction and `extractMax`](#93-problem-3-linear-time-on-bottom-up-max-heap-construction-and-extractmax)
   - 9.4 [Problem 4: Complete AVL Tree Insertion Trace Demonstrating LL, RR, LR, and RL Rotations](#94-problem-4-complete-avl-tree-insertion-trace-demonstrating-ll-rr-lr-and-rl-rotations)
   - 9.5 [Problem 5: Splay Tree Bottom-Up Splaying Trace (`Zig-Zig` and `Zig-Zag`)](#95-problem-5-splay-tree-bottom-up-splaying-trace-zig-zig-and-zig-zag)
   - 9.6 [Problem 6: Step-by-Step BFS and DFS Trace on a Graph](#96-problem-6-step-by-step-bfs-and-dfs-trace-on-a-graph)
10. [Complete Answer Keys with Detailed Explanations](#10-complete-answer-keys-with-detailed-explanations)
    - 10.1 [Answers to Short-Answer Questions (Q5.1 – Q5.10)](#101-answers-to-short-answer-questions-q51--q510)
    - 10.2 [Answers to Conceptual Questions (Q6.1 – Q6.8)](#102-answers-to-conceptual-questions-q61--q68)
    - 10.3 [Solutions to Algorithm-Writing Questions (Q7.1 – Q7.5)](#103-solutions-to-algorithm-writing-questions-q71--q75)
    - 10.4 [Complete C11 Solutions to Programming Questions (Q8.1 – Q8.5)](#104-complete-c11-solutions-to-programming-questions-q81--q85)
    - 10.5 [Navigation Links](#105-navigation-links)

---

## 1. Important Definitions Across Unit 3

### 1.1 Binary Search Trees (BSTs) and Expression Trees

- **Binary Search Tree (BST):** A binary tree in which every node $u$ stores a comparable key `u->key` satisfying the **Global BST Ordering Invariant**:
  1. Every key in $u$'s **left subtree** is strictly less than `u->key` ($\forall x \in T_L(u),\ x.\text{key} < u.\text{key}$).
  2. Every key in $u$'s **right subtree** is strictly greater than `u->key` ($\forall y \in T_R(u),\ y.\text{key} > u.\text{key}$).
  3. Both $T_L(u)$ and $T_R(u)$ are themselves valid Binary Search Trees.
  - *Fundamental Consequence:* An **in-order traversal** (`Left -> Root -> Right`) of any BST visits keys in strictly **ascending sorted order**.
- **In-Order Predecessor and In-Order Successor:**
  - The **In-Order Predecessor** of node $u$ is the node with the largest key strictly smaller than `u->key` (if $u$ has a left child, it is the **rightmost node in $u$'s left subtree**).
  - The **In-Order Successor** of node $u$ is the node with the smallest key strictly greater than `u->key` (if $u$ has a right child, it is the **leftmost node in $u$'s right subtree**).
- **Expression Tree:** A binary tree representing an algebraic or boolean expression in which:
  - Every **leaf node** is an **operand** (constant number or variable identifier).
  - Every **internal node** is an **operator** (`+`, `-`, `*`, `/`, `^`), whose subtrees represent its operands.
  - Preorder yields **Prefix (Polish)** notation, Postorder yields **Postfix (Reverse Polish)** notation, and Inorder (with parentheses at each operator) yields fully parenthesized **Infix** notation.

### 1.2 Threaded Binary Trees and Binary Heaps

- **Threaded Binary Tree:** A binary tree representation that repurposes the $n + 1$ wasted `NULL` child pointers in an $n$-node binary tree into **threads** that point directly to a node's **in-order predecessor** (via a null `left` pointer) and/or **in-order successor** (via a null `right` pointer), accompanied by boolean tag fields (`lthread`, `rthread`) to distinguish threads from real child links. Enables **stackless, recursion-free $O(n)$ in-order traversal in $O(1)$ auxiliary space**.
  - **One-Way (Right-Threaded):** Only `NULL` right pointers are replaced by threads to the in-order successor.
  - **Two-Way (Fully Threaded):** Both `NULL` left pointers (to in-order predecessor) and `NULL` right pointers (to in-order successor) are threaded.
- **Binary Heap:** A binary tree stored implicitly in a contiguous array (with zero pointers!) that simultaneously satisfies two invariants:
  1. **Structural Property (Complete Binary Tree):** Every level except possibly the last is completely full, and all nodes on the last level are packed as far **left** as possible.
  2. **Heap-Order Property:**
     - **Max-Heap:** Every parent's key is greater than or equal to its children's keys ($\text{arr}[\text{parent}(i)] \ge \text{arr}[i]$). The global maximum is always at the root (`arr[0]`).
     - **Min-Heap:** Every parent's key is less than or equal to its children's keys ($\text{arr}[\text{parent}(i)] \le \text{arr}[i]$). The global minimum is always at the root (`arr[0]`).

### 1.3 Self-Balancing BSTs: AVL Trees and Splay Trees

- **AVL Tree (Adelson-Velsky and Landis, 1962):** A strictly height-balanced Binary Search Tree in which every node $u$ satisfies the **AVL Balance Invariant**:
  $$\text{BF}(u) = \text{height}(u\to\text{left}) - \text{height}(u\to\text{right}) \in \{-1, 0, +1\}$$
  Whenever an insertion or deletion causes $|\text{BF}(u)| = 2$ at the lowest unbalanced ancestor, local tree rotations (**LL, RR, LR, or RL**) restore balance and guarantee worst-case height $h < 1.44 \log_2(n + 2)$ (i.e., $O(\log n)$ worst-case search, insert, and delete).
- **Splay Tree (Sleator and Tarjan, 1985):** A self-adjusting Binary Search Tree that stores **no explicit height or balance factor metadata**. Instead, after every access (`search`, `insert`, or `delete`) on node $x$, a restructuring operation called **splaying** (`splay(x)`) moves $x$ all the way to the **root** using a sequence of **Zig**, **Zig-Zig**, and **Zig-Zag** two-level rotations.
  - While a single operation can take $O(n)$ time in the worst case, any sequence of $m$ operations on an $n$-node Splay Tree runs in **$O(m \log n)$ amortized time** ($O(\log n)$ amortized per operation) and exploits **temporal locality** (the $90\text{–}10$ rule).

### 1.4 Graph Fundamentals, Representations, BFS, and DFS

- **Graph $G = (V, E)$:** An ordered pair of a finite vertex set $V$ and an edge set $E$ of unordered pairs $\{u, v\}$ (**undirected graph**) or ordered pairs $(u, v)$ (**directed graph / digraph**).
- **DAG (Directed Acyclic Graph):** A directed graph containing no directed cycles.
- **Adjacency Matrix vs. Adjacency List:**
  - **Adjacency Matrix:** $|V| \times |V|$ 2D array `adj[u][v]` ($\Theta(V^2)$ space, $O(1)$ edge lookup).
  - **Adjacency List:** Array of $|V|$ linked-list head pointers `head[u]` storing adjacent vertices ($\Theta(V + E)$ space, $\Theta(\deg(u))$ neighbor iteration).
- **Breadth-First Search (BFS):** A level-by-level graph traversal governed by a **FIFO Queue** that visits vertices in non-decreasing order of unweighted edge distance from source $s$.
- **Depth-First Search (DFS):** A backtracking graph traversal governed by a **LIFO Stack** (recursion or explicit stack) that plunges as deeply as possible along each path before backtracking.

---

## 2. Essential Diagrams and Mathematical Formulas

### 2.1 Implicit Array Indexing Formulas (Complete Binary Trees & Heaps)

| Indexing Convention | Root Index | Parent of `i` | Left Child of `i` | Right Child of `i` | Last Internal (Non-Leaf) Node | Leaf Index Range |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **0-Based C Array (`0 .. n-1`)** | `0` | $\lfloor (i - 1) / 2 \rfloor$ | $2i + 1$ | $2i + 2$ | $\lfloor n / 2 \rfloor - 1$ | $\lfloor n / 2 \rfloor \dots n - 1$ |
| **1-Based Array (`1 .. n`)** | `1` | $\lfloor i / 2 \rfloor$ | $2i$ | $2i + 1$ | $\lfloor n / 2 \rfloor$ | $\lfloor n / 2 \rfloor + 1 \dots n$ |

- **Height of an $n$-Element Binary Heap:** $h = \lfloor \log_2 n \rfloor$.
- **Maximum Nodes at Height $h$ in an $n$-Element Heap:** $\lceil n / 2^{h+1} \rceil$.
- **Bottom-Up `buildHeap` Complexity Sum:**
  $$\sum_{h=0}^{\lfloor \log_2 n \rfloor} \left\lceil \frac{n}{2^{h+1}} \right\rceil O(h) = O\!\left(n \sum_{h=0}^{\infty} \frac{h}{2^h}\right) = O(2n) = \Theta(n)$$

### 2.2 The $n + 1$ Null Pointers Theorem and Threaded Binary Trees

In any binary tree with $n$ nodes:
- Total child pointer fields $= 2n$.
- Total non-null child pointers (edges entering every node except the root) $= n - 1$.
- **Total `NULL` pointers available for threading:**
  $$\text{Null Pointers} = 2n - (n - 1) = n + 1$$

### 2.3 AVL Balance Factor and the 4 Rotation Cases (LL, RR, LR, RL)

$$\text{height}(\text{NULL}) = -1, \qquad \text{height}(u) = 1 + \max(\text{height}(u\to\text{left}),\ \text{height}(u\to\text{right}))$$

$$\text{BF}(u) = \text{height}(u\to\text{left}) - \text{height}(u\to\text{right}) \in \{-1, 0, +1\}$$

Let $z$ be the **lowest unbalanced ancestor** ($|\text{BF}(z)| = 2$), $y$ be the taller child of $z$, and $x$ be the taller grandchild on the insertion path:

```
  1. LL Case: BF(z)=+2, BF(y)>=0          2. RR Case: BF(z)=-2, BF(y)<=0
     Single Right Rotation: rotateRight(z)   Single Left Rotation: rotateLeft(z)

          (z) bf=+2                               (z) bf=-2
          /                                         \
        (y) bf=+1     ===>      (y)                 (y) bf=-1     ===>      (y)
        /                       / \                   \                     / \
      (x)                     (x) (z)                 (x)                 (z) (x)

  3. LR Case: BF(z)=+2, BF(y)=-1          4. RL Case: BF(z)=-2, BF(y)=+1
     rotateLeft(y), then rotateRight(z)      rotateRight(y), then rotateLeft(z)

          (z) bf=+2           (z)                     (z) bf=-2           (z)
          /                   /                       \                   \
        (y) bf=-1   ===>    (x)    ===>   (x)         (y) bf=+1   ===>    (x)    ===>   (x)
          \                 /             / \         /                     \           / \
          (x)             (y)           (y) (z)     (x)                     (y)       (z) (y)
```

### 2.4 Splay Tree Operations: Zig, Zig-Zig, and Zig-Zag

Let $x$ be the node being splayed, $p$ be its parent, and $g$ be its grandparent:

1. **Case 1 — `Zig` (Terminal Single Rotation when $p$ is the Root):**
   Rotate edge $(p, x)$ once so $x$ becomes the new root.
2. **Case 2 — `Zig-Zig` (Homogeneous / Collinear Grandchild: LL or RR):**
   **CRITICAL RULE:** Rotate **grandparent $g$ FIRST**, then rotate **parent $p$ SECOND**!
   *(Rotating $g$ first halves the depth of the path; doing `rotate(p)` first would merely perform two independent single rotations without halving path depth!)*
3. **Case 3 — `Zig-Zag` (Heterogeneous / Bent Grandchild: LR or RL):**
   Identical to an AVL double rotation: rotate **parent $p$ FIRST**, then rotate **grandparent $g$ SECOND**.

```
  SPLAY ZIG-ZIG (LL Collinear: x -> p -> g) — Rotate g FIRST, then p!
  ===================================================================
          (g)                      (p)                       (x)
          / \     rotateRight(g)   / \      rotateRight(p)   / \
        (p)  T4   =============> (x) (g)    =============>  T1 (p)
        / \                      / \ / \                       / \
      (x)  T3                   T1 T2 T3 T4                   T2 (g)
      / \                                                        / \
     T1  T2                                                     T3  T4
```

### 2.5 Graph Theory Formulas and the Handshaking Lemma

- **Undirected Handshaking Lemma:** $\sum_{v \in V} \deg(v) = 2|E|$.
- **Directed Handshaking Lemma:** $\sum_{v \in V} \deg^-(v) = \sum_{v \in V} \deg^+(v) = |E|$.
- **Maximum Edges in Simple Graph:**
  $$|E|_{\max}^{\text{undirected}} = \frac{|V|(|V| - 1)}{2}, \qquad |E|_{\max}^{\text{directed}} = |V|(|V| - 1)$$
- **Connected Undirected Tree Characterization:** Connected + Acyclic $\iff$ Connected + $|E| = |V| - 1$.

---

## 3. Commonly Confused Concepts (Exam Pitfall Disambiguation)

### 3.1 Binary Search Tree (BST) vs. Binary Heap

| Feature | Binary Search Tree (BST / AVL) | Binary Heap (Max-Heap / Min-Heap) |
| :--- | :--- | :--- |
| **Ordering Invariant** | **Total Horizontal Ordering:** $\text{Left} < \text{Parent} < \text{Right}$ | **Partial Vertical Ordering:** $\text{Parent} \ge \text{Both Children}$ (Max-Heap); no ordering between left and right siblings! |
| **Structural Shape** | Arbitrary binary tree ($h \in [\lfloor \log_2 n \rfloor, n-1]$) | **Strictly Complete Binary Tree** ($h = \lfloor \log_2 n \rfloor$ always) |
| **Memory Representation** | Dynamic pointer-based nodes (`left`, `right`) | **Contiguous 1D Array** (`2*i+1`, `2*i+2` index math, zero pointers) |
| **Search for Arbitrary Key $k$** | $O(\log n)$ average / AVL; $O(n)$ skewed | **$O(n)$ linear scan** (because sibling subtrees are mutually unordered!) |
| **Find Global Max (or Min)** | $O(h)$ (walk to rightmost/leftmost node) | **$O(1)$** (always sitting at `arr[0]`) |
| **Build from $n$ Unsorted Keys** | $\Theta(n \log n)$ | **$\Theta(n)$** via Floyd's bottom-up `buildHeap` |

### 3.2 AVL Double Rotations vs. Splay Tree Zig-Zig & Zig-Zag

- In an **AVL Tree**, a collinear LL or RR imbalance at node $z$ is fixed by **1 single rotation** (`rotateRight(z)` or `rotateLeft(z)`), leaving the middle node $y$ as the subtree root; the newly inserted grandchild $x$ remains a child of $y$.
- In a **Splay Tree**, the goal is to bring the accessed grandchild $x$ **all the way to the root**. Therefore, a collinear LL or RR configuration (`Zig-Zig`) performs **2 rotations in the same direction**, rotating the **grandparent $g$ first** and the **parent $p$ second**. By contrast, the bent LR or RL configuration (`Zig-Zag`) is identical in both AVL and Splay trees (rotate $p$ first, then $g$).

### 3.3 Adjacency Matrix $\Theta(V^2)$ vs. Adjacency List $\Theta(V + E)$ Traversal

Why does BFS or DFS take $\Theta(V^2)$ on an Adjacency Matrix even when the graph has $0$ edges?
- Because for each visited vertex $u$, discovering which vertices are adjacent to $u$ in an Adjacency Matrix requires scanning all $V$ columns of row `adj[u][0..V-1]`. Across $V$ vertices, $V \times V = \Theta(V^2)$ cells are inspected regardless of $E$.
- In an Adjacency List, `head[u]` contains *only* the $\deg(u)$ actual neighbors of $u$, so the total work across all $V$ vertices is $\sum_{u \in V} (1 + \deg(u)) = \Theta(V + E)$.

### 3.4 Marking `visited` on Enqueue vs. Dequeue in Breadth-First Search

- **Correct:** Set `visited[v] = true` **immediately when `v` is enqueued**. This ensures every vertex is enqueued at most once ($\le |V|$ queue entries).
- **Buggy:** Setting `visited[v] = true` only when `v` is dequeued allows every neighbor of `v` in the preceding wave to enqueue redundant copies of `v` while `v` is waiting in the queue, wasting up to $O(|E|)$ queue space and corrupting `parent[v]` and `dist[v]`.

---

## 4. Consolidated Operation-Complexity Table for Unit 3

| Data Structure | Search ($k$) | Insert | Delete | Find Min / Max | Build ($n$ items) | Space Complexity |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Unbalanced BST (Average)** | $\Theta(\log n)$ | $\Theta(\log n)$ | $\Theta(\log n)$ | $\Theta(\log n)$ | $\Theta(n \log n)$ | $\Theta(n)$ |
| **Unbalanced BST (Worst / Skewed)** | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n^2)$ | $\Theta(n)$ |
| **Threaded Binary Tree** | $\Theta(h)$ | $\Theta(h)$ | $\Theta(h)$ | $\Theta(h)$ | $\Theta(n \log n)$ | $\Theta(n)$ ($O(1)$ aux inorder!) |
| **Binary Max-Heap (Array)** | $\Theta(n)$ | $O(\log n)$ | $O(\log n)$ (`extractMax`) | $\Theta(1)$ (Max) | **$\Theta(n)$** (bottom-up) | $\Theta(n)$ |
| **AVL Tree (Worst Case)** | $\Theta(\log n)$ | $\Theta(\log n)$ ($\le 2$ rot.) | $\Theta(\log n)$ ($O(\log n)$ rot.) | $\Theta(\log n)$ | $\Theta(n \log n)$ | $\Theta(n)$ |
| **Splay Tree (Amortized)** | $O(\log n)$ amort. | $O(\log n)$ amort. | $O(\log n)$ amort. | $O(\log n)$ amort. | $O(n \log n)$ | $\Theta(n)$ |
| **Splay Tree (Worst Single Op)** | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n^2)$ | $\Theta(n)$ |
| **Graph — Adjacency Matrix** | $O(1)$ (`hasEdge`) | $O(1)$ (`addEdge`) | $O(1)$ (`remEdge`) | $\Theta(V^2)$ (BFS/DFS) | $\Theta(V^2)$ (init) | $\Theta(V^2)$ |
| **Graph — Adjacency List** | $O(\deg(u))$ | $O(1)$ (prepend) | $O(\deg(u))$ | $\Theta(V + E)$ (BFS/DFS) | $\Theta(V + E)$ | $\Theta(V + E)$ |

---

## 5. Short-Answer Questions (10 Exam Questions)

- **Q5.1:** State the two input sequences that cause an ordinary Binary Search Tree (BST) to degenerate into a skewed chain of height $n - 1$, and state its resulting worst-case search complexity.
- **Q5.2:** In a 0-based array representation of a Complete Binary Tree with $n = 15$ elements, what are the indices of the parent, left child, and right child of the node at index $i = 5$? What is the index of the last internal (non-leaf) node?
- **Q5.3:** Why can an Expression Tree's Postorder traversal be evaluated without any parentheses, whereas its Inorder traversal requires parentheses to avoid ambiguity?
- **Q5.4:** How many `NULL` child pointers exist in a binary tree of $n = 25$ nodes, and how does a Two-Way Threaded Binary Tree utilize them?
- **Q5.5:** What is the exact range of valid Balance Factors $\text{BF}(u)$ in an AVL tree, and what is the maximum number of rotations required to restore balance after a single **insertion** vs. a single **deletion**?
- **Q5.6:** What is the difference between a **Zig-Zig** step and a **Zig-Zag** step in a Splay Tree? Which node is rotated first in a `Zig-Zig` step?
- **Q5.7:** An undirected simple graph has $|V| = 8$ vertices. What is the minimum number of edges required for it to be connected, and what is the maximum number of edges it can have without parallel edges or self-loops?
- **Q5.8:** Can a simple undirected graph have $7$ vertices each of degree $3$? Justify your answer mathematically in one sentence.
- **Q5.9:** Which data structure governs Breadth-First Search (BFS) and which governs Depth-First Search (DFS)? Which of the two guarantees finding the shortest path in an unweighted graph?
- **Q5.10:** In a DFS of a directed graph, what color (`WHITE`, `GRAY`, or `BLACK`) is vertex $v$ when a **Back Edge** $(u, v)$ is inspected, and what structural property does a Back Edge prove?

---

## 6. Conceptual Questions (8 Deep-Thinking Questions)

- **Q6.1 (BST Local vs. Global Property Trap):** A student writes a function to check whether a binary tree is a valid BST by checking at every node $u$ whether `u->left->key < u->key` and `u->right->key > u->key`. Construct a concrete 4-node counterexample tree that passes the student's check at every node yet is **not** a valid BST, and explain how to fix the algorithm in $O(n)$ time.
- **Q6.2 (BST Deletion of a Degree-2 Node):** Explain why replacing a deleted 2-child BST node $u$ with its **In-Order Successor** $s$ preserves the global BST invariant, and explain why node $s$ is guaranteed to have **at most one child** (specifically, no left child).
- **Q6.3 (Bottom-Up $O(n)$ Heapify vs. Top-Down $O(n \log n)$ Insertion):** Why does Floyd's bottom-up `buildMaxHeap` algorithm run in $\Theta(n)$ worst-case time, whereas inserting $n$ elements one by one into an initially empty heap using `siftUp` takes $O(n \log n)$ time?
- **Q6.4 (AVL Trees vs. Splay Trees Architectural Trade-off):** Compare AVL Trees and Splay Trees in terms of (a) per-node memory overhead, (b) worst-case single-operation latency, and (c) performance on a skewed workload where $90\%$ of searches target $10\%$ of the keys. When would a real-time embedded system strictly prefer an AVL tree over a Splay tree?
- **Q6.5 (Why `Zig-Zig` Rotates the Grandparent First):** Suppose we modified Splay Trees so that whenever $x$, $p$, and $g$ form a collinear `Zig-Zig` chain, we rotated $p$ first and $g$ second (two single `Zig` rotations on $x$). What happens to the height of a skewed $n$-node chain when we splay the deepest leaf under both rules?
- **Q6.6 (In-Degree Computation in Adjacency Lists):** On a directed graph $G = (V, E)$ stored as an Adjacency List, computing the out-degree $\deg^+(u)$ of a single vertex $u$ takes $\Theta(\deg^+(u))$ time, whereas computing the in-degree $\deg^-(v)$ of a single vertex $v$ takes $\Theta(V + E)$ time. Explain why this asymmetry exists and propose two ways to support $O(1)$ or $O(\deg^-(v))$ in-degree queries.
- **Q6.7 (Why Recursive and Iterative DFS Can Produce Different Valid Orderings):** Given vertex $0$ with sorted adjacency list `head[0] -> 1 -> 2 -> 3`, explain step by step why a Recursive DFS visits `1` before `3`, whereas a naive Iterative Stack DFS that pushes neighbors in forward list order visits `3` before `1`. How can Iterative DFS be modified to match Recursive DFS exactly?
- **Q6.8 (Undirected vs. Directed Cycle Detection in DFS):** In a directed graph, seeing an already-visited vertex $v$ during DFS does **not** necessarily mean a cycle exists (for example, in the DAG $0 \to 1, 0 \to 2, 1 \to 2$, vertex $2$ is visited via $1$ and then seen again via $0 \to 2$). Why doesn't a boolean `visited[]` array suffice for directed cycle detection, and how does the 3-color (`WHITE`/`GRAY`/`BLACK`) or `on_stack[]` model solve this?

---

## 7. Algorithm-Writing Questions (5 Pseudocode Problems)

- **Q7.1:** Write clear pseudocode for `INORDER-SUCCESSOR(root, target)` in a Binary Search Tree *without* parent pointers, running in $O(h)$ time and $O(1)$ auxiliary space.
- **Q7.2:** Write pseudocode for `BUILD-EXPRESSION-TREE(postfix_tokens)` that constructs a binary expression tree from a postfix token array using an explicit stack of tree node pointers in $O(n)$ time.
- **Q7.3:** Write pseudocode for `SIFT-DOWN-MIN(arr, n, i)` and `BUILD-MIN-HEAP(arr, n)` for a 0-based array Min-Heap.
- **Q7.4:** Write pseudocode for `AVL-BALANCE(z)` that computes the balance factor of node $z$ and invokes the appropriate single or double rotation (`LL`, `RR`, `LR`, or `RL`) to restore the AVL invariant.
- **Q7.5:** Write pseudocode for `BFS-BIPARTITE-CHECK(G)` using Breadth-First Search and a 2-coloring array `color[0..V-1]` that returns `true` if an undirected (possibly disconnected) graph $G = (V, E)$ is bipartite, and `false` if it contains an odd-length cycle.

---

## 8. C Programming Questions (5 Coding Problems)

- **Q8.1 (BST Validation & Deletion in C11):** Write a C11 function `bool isValidBST(const BSTNode *root)` using min/max range propagation (`LLONG_MIN`, `LLONG_MAX`) in $O(n)$ time, along with `BSTNode *deleteBSTNode(BSTNode *root, int key)` handling all three deletion cases ($0$, $1$, and $2$ children).
- **Q8.2 (Recursive Expression Tree Evaluator in C11):** Define the `ExprNode` struct and write a C11 function `double evaluateExprTree(const ExprNode *root)` that recursively evaluates an algebraic expression tree supporting `+`, `-`, `*`, and `/` (with division-by-zero protection).
- **Q8.3 (Stackless $O(1)$-Space In-Order Traversal of a Right-Threaded Binary Tree):** Define the `TNode` struct for a One-Way Right-Threaded Binary Tree and write `void threadedInorder(TNode *root)` that prints all keys in ascending order without recursion or an explicit stack.
- **Q8.4 (In-Place $O(n)$ Max-Heap Construction & `extractMax` in C11):** Implement `void siftDownMax(int arr[], int n, int i)`, `void buildMaxHeap(int arr[], int n)`, and `int extractMax(int arr[], int *n)` for a 0-based C array.
- **Q8.5 (AVL Left-Right Double Rotation in C11):** Define the `AVLNode` struct and implement `AVLNode *rotateRight(AVLNode *z)`, `AVLNode *rotateLeft(AVLNode *y)`, and `AVLNode *rotateLeftRight(AVLNode *z)` with accurate height updates.

---

## 9. Worked Problem-Solving Questions (6 Full Step-by-Step Traces)

### 9.1 Problem 1: BST Construction and Two-Child Deletion Trace

**Problem Statement:**
1. Insert the keys `50, 30, 70, 20, 40, 60, 80, 35, 65` in that order into an initially empty Binary Search Tree (BST).
2. Delete the key `30` (a node with 2 children), then delete the root key `50` (a node with 2 children), using the **In-Order Successor** replacement convention. Draw the BST after each step.

#### Step-by-Step Solution:

**Part 1 — BST After Inserting `50, 30, 70, 20, 40, 60, 80, 35, 65`:**

```
                    (50)
                   /    \
                (30)    (70)
                /  \    /  \
             (20) (40)(60) (80)
                  /      \
                (35)     (65)
```

**Part 2a — Delete `30` (Two Children: `20` and `40`):**
- Find the **in-order successor** of `30`: go to `30->right` (`40`), then walk left as far as possible $\implies$ `35`.
- Copy key `35` into node `30`, and recursively delete leaf `35` from `40->left`:

```
                    (50)
                   /    \
                (35)    (70)
                /  \    /  \
             (20) (40)(60) (80)
                         \
                         (65)
```

**Part 2b — Delete `50` (Two Children: `35` and `70`):**
- Find the **in-order successor** of `50`: go to `50->right` (`70`), then walk left as far as possible $\implies$ `60`.
- Notice that `60` has **no left child** and **one right child** (`65`).
- Copy key `60` into the root node, and delete node `60` from `70->left` by promoting its single right child `65`:

```
                    (60)
                   /    \
                (35)    (70)
                /  \    /  \
             (20) (40)(65) (80)
```

- **Final In-Order Verification:** `20, 35, 40, 60, 65, 70, 80` (strictly sorted!).

---

### 9.2 Problem 2: Expression Tree Construction from Postfix and Recursive Evaluation

**Problem Statement:**
Given the postfix expression `6 2 + 5 3 - * 4 /` (corresponding to infix $((6 + 2) \times (5 - 3)) / 4$):
1. Trace the construction of the Expression Tree using a stack.
2. Evaluate the resulting Expression Tree bottom-up and write its Prefix (Preorder) form.

#### Step-by-Step Solution:

1. **Stack Construction Trace:**
   - Read `6`, `2`: push leaf(6), leaf(2). Stack: `[6, 2]`.
   - Read `+`: pop `2` (right), pop `6` (left), push tree `(6 + 2)`. Stack: `[+]`.
   - Read `5`, `3`: push leaf(5), leaf(3). Stack: `[+, 5, 3]`.
   - Read `-`: pop `3` (right), pop `5` (left), push tree `(5 - 3)`. Stack: `[+, -]`.
   - Read `*`: pop `(-)` (right), pop `(+)` (left), push tree `(+) * (-)`. Stack: `[*]`.
   - Read `4`: push leaf(4). Stack: `[*, 4]`.
   - Read `/`: pop `4` (right), pop `(*)` (left), push root `(/)`. Stack: `[/]`.

```
                  ( / )   <-- Evaluates: 16 / 4 = 4
                 /     \
              ( * )    [4]
             /     \    ^-- Evaluates: 8 * 2 = 16
          ( + )   ( - )
          /   \   /   \
        [6]  [2] [5]  [3]
        \_____/   \_____/
          = 8       = 2
```

2. **Bottom-Up Postorder Evaluation:**
   - Left-left subtree: $6 + 2 = 8$.
   - Left-right subtree: $5 - 3 = 2$.
   - Left child (`*`): $8 \times 2 = 16$.
   - Root (`/`): $16 / 4 = \mathbf{4}$.
3. **Prefix (Preorder) Form:** `/ * + 6 2 - 5 3 4`.

---

### 9.3 Problem 3: Linear-Time $O(n)$ Bottom-Up Max-Heap Construction and `extractMax`

**Problem Statement:**
Given the unsorted 0-based array `arr = [12, 35, 8, 42, 19, 50, 27]` ($n = 7$):
1. Trace Floyd's bottom-up `buildMaxHeap` starting from index $\lfloor n/2 \rfloor - 1 = 2$ down to $0$.
2. Trace one call to `extractMax` on the resulting Max-Heap.

#### Step-by-Step Solution:

1. **Initial Complete Binary Tree (`[12, 35, 8, 42, 19, 50, 27]`):**
   - Internal node indices: $i = 2$ (`8`), $i = 1$ (`35`), $i = 0$ (`12`).
   - **Step A (`i = 2`, key `8`):** Children are `2(2)+1 = 5` (`50`) and `2(2)+2 = 6` (`27`). Largest child is `50` at index 5. Swap `arr[2]` and `arr[5]`:
     `arr = [12, 35, 50, 42, 19, 8, 27]`
   - **Step B (`i = 1`, key `35`):** Children are `2(1)+1 = 3` (`42`) and `2(1)+2 = 4` (`19`). Largest child is `42` at index 3. Swap `arr[1]` and `arr[3]`:
     `arr = [12, 42, 50, 35, 19, 8, 27]`
   - **Step C (`i = 0`, key `12`):**
     - Children of `0` are `1` (`42`) and `2` (`50`). Largest child is `50` at index 2. Swap `arr[0]` and `arr[2]`:
       `arr = [50, 42, 12, 35, 19, 8, 27]`
     - Continue sifting `12` at index `2`: children are `5` (`8`) and `6` (`27`). Largest child is `27` at index 6. Swap `arr[2]` and `arr[6]`:
       `arr = [50, 42, 27, 35, 19, 8, 12]`

```
        MAX-HEAP AFTER buildMaxHeap                  MAX-HEAP AFTER extractMax() -> 50
        ===========================                  =================================
                   (50)                                             (42)
                  /    \                                           /    \
               (42)    (27)                                     (35)    (27)
               /  \    /  \                                     /  \    /
            (35) (19)(8)  (12)                               (12) (19)(8)
   Array: [50, 42, 27, 35, 19, 8, 12]               Array: [42, 35, 27, 12, 19, 8]
```

2. **Trace `extractMax()`:**
   - Save root `max_val = 50`.
   - Move last element `arr[6] = 12` to `arr[0]` and decrement $n$ to $6$: `[12, 42, 27, 35, 19, 8]`.
   - Sift `12` down from index `0`: swap with larger child `42` (index 1) $\implies$ `[42, 12, 27, 35, 19, 8]`; then compare `12` at index 1 with children `35` (index 3) and `19` (index 4), swapping with `35` $\implies$ **`[42, 35, 27, 12, 19, 8]`**.

---

### 9.4 Problem 4: Complete AVL Tree Insertion Trace Demonstrating LL, RR, LR, and RL Rotations

**Problem Statement:**
Insert the sequence of keys **`30, 20, 10, 40, 50, 25, 45`** into an initially empty AVL Tree. At each imbalance, identify the lowest unbalanced ancestor $z$, the imbalance case (**LL, RR, LR, or RL**), and draw the tree after rotation.

#### Step-by-Step Solution:

1. **Insert `30, 20, 10` $\implies$ Imbalance at $z = 30$ ($\text{BF}(30) = +2, \text{BF}(20) = +1 \implies$ LL Case):**
   - Perform **Single Right Rotation** `rotateRight(30)`:
   ```
        (30) bf=+2                  (20) bf=0
        /          rotateRight(30)  /  \
      (20) bf=+1   ==============> (10) (30)
      /
    (10)
   ```

2. **Insert `40, 50` $\implies$ Imbalance at $z = 30$ ($\text{BF}(30) = -2, \text{BF}(40) = -1 \implies$ RR Case):**
   - Perform **Single Left Rotation** `rotateLeft(30)`:
   ```
         (20)                           (20) bf=-1
         /  \                           /  \
       (10) (30) bf=-2  rotateLeft(30)(10) (40) bf=0
               \        =============>     /  \
               (40) bf=-1                (30) (50)
                  \
                  (50)
   ```

3. **Insert `25` $\implies$ Imbalance at Root $z = 20$ ($\text{BF}(20) = -2$, right child $y = 40$ has $\text{BF}(40) = +1 \implies$ RL Case):**
   - Path from $z=20 \to y=40 \to x=30$ is **Right-Left (RL)**.
   - Perform `rotateRight(40)` followed by `rotateLeft(20)` (promoting $x = 30$ to root):
   ```
         (20) bf=-2                     (20) bf=-2                      (30) bf=0
         /  \                           /  \                            /    \
       (10) (40) bf=+1  rotateRight(40)(10)(30)         rotateLeft(20)(20)   (40)
            /  \        =============>     /  \         ============> /  \     \
          (30) (50)                      (25) (40)                  (10)(25)  (50)
          /                                      \
        (25)                                     (50)
   ```

4. **Insert `45` $\implies$ Imbalance at $z = 40$ ($\text{BF}(40) = -2$, right child $y = 50$ has $\text{BF}(50) = +1 \implies$ RL Case; or if `22` were inserted under `25`, LR Case):**
   - Here $z = 40$ has $\text{BF}(40) = -2$ and $y = 50$ has $\text{BF}(50) = +1$ (RL Case at node `40`): `rotateRight(50)` then `rotateLeft(40)` makes `45` the parent of `40` and `50`:
   ```
                  (30) bf=0
                 /    \
              (20)    (45) bf=0
              /  \    /  \
           (10) (25)(40) (50)
   ```
   *(And note: an **LR Case** is simply the mirror image of RL—for instance, inserting `30, 10, 20` gives $\text{BF}(30) = +2, \text{BF}(10) = -1$, resolved by `rotateLeft(10)` then `rotateRight(30)` to make `20` the root with children `10` and `30`).*

---

### 9.5 Problem 5: Splay Tree Bottom-Up Splaying Trace (`Zig-Zig` and `Zig-Zag`)

**Problem Statement:**
Consider the BST below. Trace the bottom-up splaying operation **`splay(15)`** step by step until `15` reaches the root.

```
                (40)
                /
              (30)
              /
            (10)
              \
              (20)
              /
            (15)
```

#### Step-by-Step Solution:

1. **Step 1 — Bottom 3 nodes `10 -> 20 -> 15` form a `Zig-Zag` (RL Case: $g = 10, p = 20, x = 15$):**
   - Rotate parent $p = 20$ right (`rotateRight(20)`), then rotate grandparent $g = 10$ left (`rotateLeft(10)`):
   ```
          (40)                        (40)
          /                           /
        (30)                        (30)
        /       Zig-Zag at 15       /
      (10)      =============>    (15)
        \                         /  \
        (20)                    (10) (20)
        /
      (15)
   ```

2. **Step 2 — Top 3 nodes `40 -> 30 -> 15` form a `Zig-Zig` (LL Collinear Case: $g = 40, p = 30, x = 15$):**
   - **Rotate grandparent $g = 40$ right FIRST** (`rotateRight(40)`), then **rotate parent $p = 30$ right SECOND** (`rotateRight(30)`):
   ```
          (40)                        (30)                        (15)
          /                           /  \                        /  \
        (30)     rotateRight(40)    (15) (40)   rotateRight(30) (10) (30)
        /        ==============>    /  \        ==============>      /  \
      (15)                        (10) (20)                        (20) (40)
      /  \
    (10) (20)
   ```
   - Node `15` is now at the root, and the tree height has been compressed from $4$ down to $2$!

---

### 9.6 Problem 6: Step-by-Step BFS and DFS Trace on a Graph

**Problem Statement:**
Consider the undirected graph $G = (V, E)$ with $V = \{0, 1, 2, 3, 4\}$ and edges $\{0,1\}, \{0,2\}, \{0,3\}, \{1,3\}, \{2,4\}, \{3,4\}$ (neighbors visited in ascending vertex order):
1. Trace **BFS** starting at vertex `0` (give traversal order, `dist[]`, and `parent[]`).
2. Trace **Recursive DFS** starting at vertex `0` (give traversal order and discovery/finish times $d[v]/f[v]$).

#### Step-by-Step Solution:

- **Sorted Adjacency Lists:**
  `head[0]: 1, 2, 3` | `head[1]: 0, 3` | `head[2]: 0, 4` | `head[3]: 0, 1, 4` | `head[4]: 2, 3`

1. **BFS Trace from `0`:**
   - Init: Enqueue `0` (`dist[0]=0, parent[0]=-1`). Queue: `[0]`, `visited = {0}`.
   - Dequeue `0`: Enqueue `1, 2, 3` (`dist=1, parent=0`). Queue: `[1, 2, 3]`, `visited = {0,1,2,3}`.
   - Dequeue `1`: Neighbors `0, 3` already visited. Queue: `[2, 3]`.
   - Dequeue `2`: Neighbor `4` unvisited $\implies$ Enqueue `4` (`dist[4]=2, parent[4]=2`). Queue: `[3, 4]`, `visited = {0,1,2,3,4}`.
   - Dequeue `3`, then `4`: all neighbors visited.
   - **BFS Traversal Order:** **`0, 1, 2, 3, 4`**
   - **Distances `dist[0..4]`:** `[0, 1, 1, 1, 2]` | **Parents `parent[0..4]`:** `[-1, 0, 0, 0, 2]`.

2. **Recursive DFS Trace from `0`:**
   - `t=1`: Discover `0` ($d[0]=1$) $\to$ call `dfs(1)`.
   - `t=2`: Discover `1` ($d[1]=2$) $\to$ neighbor `0` visited, call `dfs(3)`.
   - `t=3`: Discover `3` ($d[3]=3$) $\to$ `0, 1` visited (edge `3->0` is a Back Edge!), call `dfs(4)`.
   - `t=4`: Discover `4` ($d[4]=4$) $\to$ call `dfs(2)`.
   - `t=5`: Discover `2` ($d[2]=5$) $\to$ `0, 4` visited (edge `2->0` is a Back Edge!).
   - Finish times as calls return: $f[2]=6,\ f[4]=7,\ f[3]=8,\ f[1]=9,\ f[0]=10$.
   - **Recursive DFS Traversal Order:** **`0, 1, 3, 4, 2`**.

---

## 10. Complete Answer Keys with Detailed Explanations

### 10.1 Answers to Short-Answer Questions (Q5.1 – Q5.10)

- **A5.1:** Inserting keys in **strictly ascending order** (e.g., `10, 20, 30, 40`) or **strictly descending order** (e.g., `40, 30, 20, 10`) causes an ordinary BST to degenerate into a linked list of height $h = n - 1$, degrading search, insert, and delete to **$\Theta(n)$** worst-case time.
- **A5.2:** For $i = 5$ in a 0-based array:
  - $\text{parent}(5) = \lfloor (5 - 1)/2 \rfloor = \mathbf{2}$.
  - $\text{left}(5) = 2(5) + 1 = \mathbf{11}$.
  - $\text{right}(5) = 2(5) + 2 = \mathbf{12}$.
  - Last internal node for $n = 15$ is $\lfloor 15 / 2 \rfloor - 1 = \mathbf{6}$ (indices `7..14` are leaves).
- **A5.3:** In Postfix (Reverse Polish) notation, the arity of each operator is fixed and every operator immediately follows its two operand sub-expressions, uniquely encoding tree structure without parentheses. In Infix notation, operator precedence and associativity are lost when flattening the tree unless parentheses are emitted around each sub-expression.
- **A5.4:** By the $n + 1$ Null Pointers Theorem, a 25-node binary tree has exactly $25 + 1 = \mathbf{26}$ `NULL` child pointers. A Two-Way Threaded Binary Tree replaces null `left` pointers with threads to the **in-order predecessor** and null `right` pointers with threads to the **in-order successor**.
- **A5.5:** Valid AVL balance factors are $\text{BF}(u) \in \{-1, 0, +1\}$.
  - **Insertion** requires at most **1 single or double rotation ($\le 2$ pointer rotations)** at the lowest unbalanced ancestor, because a rotation restores the subtree's original pre-insertion height.
  - **Deletion** can shorten a subtree and propagate height decreases up to the root, requiring up to **$O(\log n)$ rotations**.
- **A5.6:** `Zig-Zig` occurs when $x$ and $p$ are both left children (LL) or both right children (RR); `Zig-Zag` occurs when one is a left child and the other is a right child (LR or RL). In a **`Zig-Zig`** step, the **grandparent $g$ is rotated first**, followed by the parent $p$.
- **A5.7:** For $|V| = 8$:
  - Minimum edges to be connected $= |V| - 1 = \mathbf{7}$ (a spanning tree).
  - Maximum edges in a simple undirected graph $= \frac{8 \times 7}{2} = \mathbf{28}$ ($K_8$).
- **A5.8:** **No.** By the Handshaking Lemma ($\sum \deg(v) = 2|E|$), the sum of degrees would be $7 \times 3 = 21$, which is odd and cannot equal $2|E|$ (equivalently, the number of odd-degree vertices must always be even).
- **A5.9:** **BFS** uses a **FIFO Queue**; **DFS** uses a **LIFO Stack** (call stack or explicit stack). **BFS** guarantees finding the shortest path (minimum edge count) in any unweighted graph.
- **A5.10:** In a directed DFS, a Back Edge $(u, v)$ points to a vertex $v$ whose color is **`GRAY`** (discovered and currently active on the recursion call stack). The existence of a Back Edge proves that the graph contains a **directed cycle**.

---

### 10.2 Answers to Conceptual Questions (Q6.1 – Q6.8)

- **A6.1 (BST Local vs. Global Trap):**
  Consider the tree with root `20`, left child `10`, and `10`'s right child `25` (`20->left = 10`, `10->right = 25`, `20->right = 30`). Locally at node `10`, `25 > 10` holds, and at node `20`, `10 < 20 < 30` holds. Yet `25` sits in the **left subtree of `20`** while $25 > 20$, violating the global BST invariant!
  - *Fix:* Pass a valid open interval `(min_val, max_val)` down the recursion: `isValidBST(u->left, min_val, u->key)` and `isValidBST(u->right, u->key, max_val)` in $O(n)$ time.
- **A6.2 (BST 2-Child Deletion):**
  Because the In-Order Successor $s$ is the *smallest* key in $u$'s right subtree $T_R(u)$, $s$ is strictly greater than every key in $T_L(u)$ and strictly less than every other key in $T_R(u)$, so placing $s$ at $u$ preserves the BST invariant. Furthermore, $s$ cannot have a left child because if $s\to\text{left}$ existed, that left child would be in $T_R(u)$ and strictly smaller than $s$, contradicting that $s$ is the minimum of $T_R(u)$!
- **A6.3 (Bottom-Up $O(n)$ vs. Top-Down $O(n \log n)$ Heap Build):**
  In top-down insertion, the $\lceil n/2 \rceil$ leaves at the bottom of the tree can each `siftUp` all the way to the root ($O(\log n)$ work for half the elements $\implies \Theta(n \log n)$). In bottom-up `buildMaxHeap`, the $\lceil n/2 \rceil$ leaves do **$0$ work**, $\lceil n/4 \rceil$ nodes at height $1$ do at most $1$ swap, and only $1$ node (the root) can travel $\log_2 n$ levels. Summing $\sum_{h=0}^{\log n} \frac{n}{2^{h+1}} h = O(n)$ yields linear time.
- **A6.4 (AVL vs. Splay Trade-offs):**
  - (a) **Memory:** Splay trees require **zero balance bits/height fields** per node; AVL trees require an `int height` or 2-bit balance factor per node.
  - (b) **Worst-Case Single Operation:** AVL guarantees strict $O(\log n)$ worst-case time on *every* operation; a Splay tree can take $\Theta(n)$ on a single operation if the tree is temporarily skewed.
  - (c) **Skewed $90\text{–}10$ Workload:** Splay trees keep frequently accessed "hot" keys near the root, achieving faster access on skewed workloads. However, a **hard real-time embedded system** (e.g., avionics flight control) must choose an **AVL tree** because it cannot tolerate an unpredictable $\Theta(n)$ latency spike on a single lookup.
- **A6.5 (Why `Zig-Zig` Rotates Grandparent First):**
  When splaying the deepest leaf of a skewed chain of height $n$ using true `Zig-Zig` (rotate $g$ first, then $p$), every step pairs two nodes along the chain, **halving the height of the tree to $\approx n/2$**. If instead we rotated $p$ first (two ordinary `Zig` single rotations), the leaf moves to the root while pushing the rest of the chain down by $1$, leaving the tree height at **$n - 1$** (no height reduction, causing $\Theta(n)$ repeated cost!).
- **A6.6 (In-Degree in Adjacency Lists):**
  `head[u]` only stores edges *leaving* $u$, so incoming edges to $v$ are scattered across the adjacency lists of other vertices. To achieve fast in-degree queries:
  1. Maintain an integer array `in_degree[0..V-1]` updated in $O(1)$ whenever `addEdge(u, v)` (`in_degree[v]++`) or `removeEdge(u, v)` (`in_degree[v]--`) is called.
  2. Maintain a secondary **Reverse (Transpose) Adjacency List** `in_head[v]` storing incoming neighbors if you also need to iterate over incoming edges in $\Theta(\deg^-(v))$ time.
- **A6.7 (Recursive vs. Iterative DFS Ordering):**
  Recursive DFS iterates over `head[0] = [1, 2, 3]` and immediately pauses the loop at `1` to execute `dfs(1)`. Naive Iterative DFS pushes `1`, then `2`, then `3` onto a LIFO stack; because `3` is pushed last, `3` is at the top of the stack and popped *first*. Pushing `head[u]`'s neighbors in **reverse order** (`3`, then `2`, then `1`) puts `1` on top of the stack, matching Recursive DFS.
- **A6.8 (Directed Cycle Detection):**
  In the DAG $0 \to 1, 0 \to 2, 1 \to 2$, DFS from `0` visits `1` then `2`, and finishes both `2` and `1` (turning them `BLACK`). When DFS backtracks to `0` and inspects edge $0 \to 2$, `visited[2]` is `true`, but `2` is already **finished (`BLACK`)**, so $0 \to 2$ is a harmless **Forward/Cross Edge**, not a cycle! A directed cycle exists if and only if DFS sees a neighbor $v$ that is currently **`GRAY`** (`on_recursion_stack[v] == true`), which proves $v$ is an ancestor of $u$ with an active path $v \leadsto u$ that edge $(u, v)$ closes into a cycle.

---

### 10.3 Solutions to Algorithm-Writing Questions (Q7.1 – Q7.5)

#### A7.1: `INORDER-SUCCESSOR(root, target)` in a BST ($O(h)$ Time, $O(1)$ Space)

```text
ALGORITHM: INORDER-SUCCESSOR(root, target)
1. if target->right != NULL then
2.     curr <- target->right
3.     while curr->left != NULL do
4.         curr <- curr->left
5.     end while
6.     return curr
7. end if
8.
9. succ <- NULL
10. ancestor <- root
11. while ancestor != target and ancestor != NULL do
12.     if target->key < ancestor->key then
13.         succ <- ancestor            // ancestor is a candidate successor
14.         ancestor <- ancestor->left
15.     else
16.         ancestor <- ancestor->right
17.     end if
18. end while
19. return succ
```

#### A7.2: `BUILD-EXPRESSION-TREE(postfix_tokens)`

```text
ALGORITHM: BUILD-EXPRESSION-TREE(postfix_tokens[0..m-1])
1. S <- empty Stack of ExprNode pointers
2. for i <- 0 to m - 1 do
3.     tok <- postfix_tokens[i]
4.     node <- CREATE-EXPR-NODE(tok)
5.     if IS-OPERAND(tok) then
6.         PUSH(S, node)
7.     else                             // Binary operator (+, -, *, /, ^)
8.         node->right <- POP(S)        // First popped is RIGHT operand!
9.         node->left  <- POP(S)        // Second popped is LEFT operand!
10.        PUSH(S, node)
11.    end if
12. end for
13. return POP(S)                       // Root of Expression Tree
```

#### A7.3: `SIFT-DOWN-MIN(arr, n, i)` and `BUILD-MIN-HEAP(arr, n)`

```text
ALGORITHM: SIFT-DOWN-MIN(arr, n, i)
1. while true do
2.     smallest <- i
3.     left     <- 2 * i + 1
4.     right    <- 2 * i + 2
5.     if left < n and arr[left] < arr[smallest] then
6.         smallest <- left
7.     end if
8.     if right < n and arr[right] < arr[smallest] then
9.         smallest <- right
10.    end if
11.    if smallest == i then
12.        break
13.    end if
14.    SWAP(arr[i], arr[smallest])
15.    i <- smallest
16. end while

ALGORITHM: BUILD-MIN-HEAP(arr, n)
1. for i <- FLOOR(n / 2) - 1 down to 0 do
2.     SIFT-DOWN-MIN(arr, n, i)
3. end for
```

#### A7.4: `AVL-BALANCE(z)` Dispatch Logic

```text
ALGORITHM: AVL-BALANCE(z)
1. UPDATE-HEIGHT(z)
2. bf <- HEIGHT(z->left) - HEIGHT(z->right)
3. if bf > 1 then                                             // Left-heavy (+2)
4.     if HEIGHT(z->left->left) >= HEIGHT(z->left->right) then
5.         return ROTATE-RIGHT(z)                             // LL Case
6.     else
7.         z->left <- ROTATE-LEFT(z->left)                    // LR Case
8.         return ROTATE-RIGHT(z)
9.     end if
10. else if bf < -1 then                                      // Right-heavy (-2)
11.    if HEIGHT(z->right->right) >= HEIGHT(z->right->left) then
12.        return ROTATE-LEFT(z)                              // RR Case
13.    else
14.        z->right <- ROTATE-RIGHT(z->right)                 // RL Case
15.        return ROTATE-LEFT(z)
16.    end if
17. end if
18. return z
```

#### A7.5: `BFS-BIPARTITE-CHECK(G)` (2-Coloring via BFS)

```text
ALGORITHM: BFS-BIPARTITE-CHECK(G)
1. for u <- 0 to |V| - 1 do
2.     color[u] <- -1                                         // -1 = uncolored
3. end for
4. for start <- 0 to |V| - 1 do
5.     if color[start] == -1 then
6.         color[start] <- 0
7.         Q <- empty Queue
8.         ENQUEUE(Q, start)
9.         while not IS-EMPTY(Q) do
10.            u <- DEQUEUE(Q)
11.            for each neighbor v of u do
12.                if color[v] == -1 then
13.                    color[v] <- 1 - color[u]               // Assign opposite color (0 or 1)
14.                    ENQUEUE(Q, v)
15.                else if color[v] == color[u] then
16.                    return false                           // Odd cycle detected! Not bipartite
17.                end if
18.            end for
19.        end while
20.    end if
21. end for
22. return true
```

---

### 10.4 Complete C11 Solutions to Programming Questions (Q8.1 – Q8.5)

All five C11 solutions below are combined into a single compilable, warning-free reference file (`gcc -std=c11 -Wall -Wextra`):

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <limits.h>

/* =========================================================================
 * SOLUTION Q8.1: Validate BST (Min/Max Range) & Delete BST Node
 * ========================================================================= */
typedef struct BSTNode {
    int key;
    struct BSTNode *left;
    struct BSTNode *right;
} BSTNode;

static bool isValidBSTHelper(const BSTNode *root, long long min_v, long long max_v) {
    if (root == NULL) {
        return true;
    }
    if (root->key <= min_v || root->key >= max_v) {
        return false;
    }
    return isValidBSTHelper(root->left, min_v, root->key) &&
           isValidBSTHelper(root->right, root->key, max_v);
}

bool isValidBST(const BSTNode *root) {
    return isValidBSTHelper(root, LLONG_MIN, LLONG_MAX);
}

BSTNode *deleteBSTNode(BSTNode *root, int key) {
    if (root == NULL) {
        return NULL;
    }
    if (key < root->key) {
        root->left = deleteBSTNode(root->left, key);
    } else if (key > root->key) {
        root->right = deleteBSTNode(root->right, key);
    } else {
        /* Cases 1 & 2: 0 or 1 child */
        if (root->left == NULL) {
            BSTNode *temp = root->right;
            free(root);
            return temp;
        }
        if (root->right == NULL) {
            BSTNode *temp = root->left;
            free(root);
            return temp;
        }
        /* Case 3: 2 children -> replace with In-Order Successor */
        BSTNode *succ = root->right;
        while (succ->left != NULL) {
            succ = succ->left;
        }
        root->key = succ->key;
        root->right = deleteBSTNode(root->right, succ->key);
    }
    return root;
}

/* =========================================================================
 * SOLUTION Q8.2: Recursive Algebraic Expression Tree Evaluator
 * ========================================================================= */
typedef struct ExprNode {
    bool is_operator;
    char op;
    double value;
    struct ExprNode *left;
    struct ExprNode *right;
} ExprNode;

double evaluateExprTree(const ExprNode *root) {
    if (root == NULL) {
        return 0.0;
    }
    if (!root->is_operator) {
        return root->value;
    }
    double l_val = evaluateExprTree(root->left);
    double r_val = evaluateExprTree(root->right);
    switch (root->op) {
        case '+': return l_val + r_val;
        case '-': return l_val - r_val;
        case '*': return l_val * r_val;
        case '/': return (r_val != 0.0) ? (l_val / r_val) : 0.0;
        default:  return 0.0;
    }
}

/* =========================================================================
 * SOLUTION Q8.3: Stackless O(1)-Space In-Order of Right-Threaded Tree
 * ========================================================================= */
typedef struct TNode {
    int data;
    struct TNode *left;
    struct TNode *right;
    bool right_thread;
} TNode;

static TNode *leftmostNode(TNode *n) {
    while (n != NULL && n->left != NULL) {
        n = n->left;
    }
    return n;
}

void threadedInorder(TNode *root) {
    TNode *curr = leftmostNode(root);
    while (curr != NULL) {
        printf("%d ", curr->data);
        if (curr->right_thread) {
            curr = curr->right;             /* Follow right thread to successor */
        } else {
            curr = leftmostNode(curr->right); /* Enter right subtree */
        }
    }
}

/* =========================================================================
 * SOLUTION Q8.4: O(n) Bottom-Up Max-Heapify and extractMax
 * ========================================================================= */
void siftDownMax(int arr[], int n, int i) {
    while (true) {
        int largest = i;
        int left  = 2 * i + 1;
        int right = 2 * i + 2;

        if (left < n && arr[left] > arr[largest]) {
            largest = left;
        }
        if (right < n && arr[right] > arr[largest]) {
            largest = right;
        }
        if (largest == i) {
            break;
        }
        int tmp = arr[i];
        arr[i] = arr[largest];
        arr[largest] = tmp;
        i = largest;
    }
}

void buildMaxHeap(int arr[], int n) {
    for (int i = (n / 2) - 1; i >= 0; i--) {
        siftDownMax(arr, n, i);
    }
}

int extractMax(int arr[], int *n) {
    if (*n <= 0) {
        return INT_MIN;
    }
    int max_val = arr[0];
    arr[0] = arr[*n - 1];
    (*n)--;
    siftDownMax(arr, *n, 0);
    return max_val;
}

/* =========================================================================
 * SOLUTION Q8.5: AVL Left-Right (LR) Double Rotation
 * ========================================================================= */
typedef struct AVLNode {
    int key;
    int height;
    struct AVLNode *left;
    struct AVLNode *right;
} AVLNode;

static int nodeHeight(const AVLNode *n) {
    return (n == NULL) ? -1 : n->height;
}

static int maxInt(int a, int b) {
    return (a > b) ? a : b;
}

AVLNode *rotateRight(AVLNode *z) {
    AVLNode *y = z->left;
    AVLNode *T3 = y->right;
    y->right = z;
    z->left = T3;
    z->height = 1 + maxInt(nodeHeight(z->left), nodeHeight(z->right));
    y->height = 1 + maxInt(nodeHeight(y->left), nodeHeight(y->right));
    return y;
}

AVLNode *rotateLeft(AVLNode *y) {
    AVLNode *x = y->right;
    AVLNode *T2 = x->left;
    x->left = y;
    y->right = T2;
    y->height = 1 + maxInt(nodeHeight(y->left), nodeHeight(y->right));
    x->height = 1 + maxInt(nodeHeight(x->left), nodeHeight(x->right));
    return x;
}

AVLNode *rotateLeftRight(AVLNode *z) {
    z->left = rotateLeft(z->left);
    return rotateRight(z);
}
```

---

### 10.5 Navigation Links

- **Unit 3 Topic 1:** [01-BST-Array-and-Dynamic-Implementations.md](./01-BST-Array-and-Dynamic-Implementations.md)
- **Unit 3 Topic 2:** [02-Expression-Trees.md](./02-Expression-Trees.md)
- **Unit 3 Topic 3:** [03-Threaded-Binary-Search-Trees.md](./03-Threaded-Binary-Search-Trees.md)
- **Unit 3 Topic 4:** [04-Heaps-and-Heap-Priority-Queues.md](./04-Heaps-and-Heap-Priority-Queues.md)
- **Unit 3 Topic 5:** [05-AVL-Trees.md](./05-AVL-Trees.md)
- **Unit 3 Topic 6:** [06-Splay-Trees.md](./06-Splay-Trees.md)
- **Unit 3 Topic 7:** [07-Graph-Fundamentals-and-Representations.md](./07-Graph-Fundamentals-and-Representations.md)
- **Unit 3 Topic 8:** [08-DFS-and-BFS.md](./08-DFS-and-BFS.md)
- **Current File:** `09-Unit-3-Review.md` (Unit 3 — Comprehensive Review & Exam Preparation)
