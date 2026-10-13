# Unit 2: N-Ary Trees, Binary Trees, and Forests

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [Why Non-Linear Data Structures?](#11-why-non-linear-data-structures)
   - 1.2 [Relationship to Prior Concepts](#12-relationship-to-prior-concepts)
   - 1.3 [Learning Objectives](#13-learning-objectives)
2. [Tree Definitions, Terminology, and Mathematical Properties](#2-tree-definitions-terminology-and-mathematical-properties)
   - 2.1 [Formal Graph-Theoretic and Recursive Definitions](#21-formal-graph-theoretic-and-recursive-definitions)
   - 2.2 [Master Terminology Reference and Annotated Diagram](#22-master-terminology-reference-and-annotated-diagram)
   - 2.3 [Depth vs. Height: The Two Standard Conventions](#23-depth-vs-height-the-two-standard-conventions)
   - 2.4 [Fundamental Mathematical Theorems and Proofs](#24-fundamental-mathematical-theorems-and-proofs)
3. [N-Ary Trees: Concepts and Memory Representations in C](#3-n-ary-trees-concepts-and-memory-representations-in-c)
   - 3.1 [Fixed-Size Pointer Array Representation](#31-fixed-size-pointer-array-representation)
   - 3.2 [Dynamic Resizable Array (`realloc`) Representation](#32-dynamic-resizable-array-realloc-representation)
   - 3.3 [First-Child / Next-Sibling (LCRS) Representation](#33-first-child--next-sibling-lcrs-representation)
4. [Binary Trees and Common Structural Types](#4-binary-trees-and-common-structural-types)
   - 4.1 [Formal Definition of a Binary Tree](#41-formal-definition-of-a-binary-tree)
   - 4.2 [Five Canonical Types of Binary Trees](#42-five-canonical-types-of-binary-trees)
   - 4.3 [Array (Sequential) vs. Pointer (Linked) Representation in C](#43-array-sequential-vs-pointer-linked-representation-in-c)
   - 4.4 [Binary Search Tree (BST) Fundamentals and Ordering Invariant](#44-binary-search-tree-bst-fundamentals-and-ordering-invariant)
5. [Forests](#5-forests)
   - 5.1 [Formal Definition and Edge-Vertex-Component Theorem](#51-formal-definition-and-edge-vertex-component-theorem)
   - 5.2 [Structural Duality Between Trees and Forests](#52-structural-duality-between-trees-and-forests)
6. [Operations and Conversion Algorithms (Knuth / LCRS Transform)](#6-operations-and-conversion-algorithms-knuth--lcrs-transform)
   - 6.1 [Converting an N-Ary Tree to a Binary Tree](#61-converting-an-n-ary-tree-to-a-binary-tree)
   - 6.2 [Converting a Forest to a Binary Tree](#62-converting-a-forest-to-a-binary-tree)
   - 6.3 [Reverse Reconstruction: Binary Tree Back to Forest / N-Ary Tree](#63-reverse-reconstruction-binary-tree-back-to-forest--n-ary-tree)
   - 6.4 [Complete C11 Implementation and Execution Trace](#64-complete-c11-implementation-and-execution-trace)
7. [Comparison Tables and Complexity Summary](#7-comparison-tables-and-complexity-summary)
   - 7.1 [N-Ary Tree vs. Binary Tree](#71-n-ary-tree-vs-binary-tree)
   - 7.2 [General Binary Tree vs. Binary Search Tree (BST)](#72-general-binary-tree-vs-binary-search-tree-bst)
   - 7.3 [Single Tree vs. Forest](#73-single-tree-vs-forest)
   - 7.4 [Operation Time and Space Complexity Summary](#74-operation-time-and-space-complexity-summary)
8. [Common Errors, Pitfalls, and Edge Cases](#8-common-errors-pitfalls-and-edge-cases)
   - 8.1 [Concrete Bug vs. Fix Examples in C](#81-concrete-bug-vs-fix-examples-in-c)
   - 8.2 [Boundary Conditions Checklist](#82-boundary-conditions-checklist)
9. [Summary and Revision Checklist](#9-summary-and-revision-checklist)

---

## 1. Introduction and Learning Objectives

### 1.1 Why Non-Linear Data Structures?

In Unit 1 and the first half of Unit 2, every data structure we studied—**arrays, linked lists, stacks, and queues**—organized elements in a strictly **linear** sequence. In a linear structure, each element has at most one immediate predecessor and at most one immediate successor.

Linear structures suffer from two fundamental limitations:
1. **Inability to Model Natural Hierarchies**: Real-world systems such as operating system file directories (`/usr/local/bin`), HTML/XML Document Object Models (DOM), compiler Abstract Syntax Trees (ASTs), and corporate organizational charts are inherently **hierarchical** (one-to-many relationships).
2. **Search vs. Update Trade-Off**:
   - In an **unsorted array** or **linked list**, searching takes $O(n)$ time.
   - In a **sorted array**, binary search achieves $O(\log n)$ time, but inserting or deleting an element requires shifting $O(n)$ elements.
   - **Trees** bridge this gap: a balanced search tree organizes keys hierarchically so that search, insertion, and deletion can **all** be performed in $O(\log n)$ time.

### 1.2 Relationship to Prior Concepts

- **Generalization of Linked Lists**: A singly linked list node has a single `next` pointer (`degree <= 1`). Extending a node to hold two pointers (`left`, `right`) yields a **Binary Tree**; extending it to hold $N$ child pointers yields an **N-Ary Tree**.
- **Stacks and Recursion (Unit 1)**: Because trees are recursively defined (a tree consists of a root and subtrees), tree algorithms naturally use recursion (implicitly backed by the call stack) or an explicit `Stack` for depth-first traversal.
- **Queues (Unit 2A)**: Visiting a tree level-by-level from top to bottom (Breadth-First / Level-Order traversal) requires a `Queue` to hold discovered children in FIFO order.

### 1.3 Learning Objectives

After completing this chapter, you will be able to:
1. Define all standard tree terminology and prove the fundamental mathematical properties of $N$-ary and binary trees ($|E| = |V| - 1$, height-node bounds, and the Degree-2 Leaf Theorem $L = I_2 + 1$).
2. Clearly distinguish between **0-based edge height** and **1-based node height** conventions.
3. Implement **N-ary trees** in C using fixed pointer arrays, dynamic `realloc` arrays, and the **Left-Child Right-Sibling (LCRS)** representation.
4. Classify and analyze **Full, Complete, Perfect, Balanced, and Degenerate Binary Trees**, along with array and pointer storage schemes and the **Binary Search Tree (BST)** ordering invariant.
5. Define a **Forest** ($\mathcal{F} = \{T_1, \dots, T_k\}$), prove $|E| = |V| - k$, and execute the **Knuth / LCRS isomorphism** to convert both single $N$-ary trees and forests into binary trees (and reconstruct them in reverse).

---

## 2. Tree Definitions, Terminology, and Mathematical Properties

### 2.1 Formal Graph-Theoretic and Recursive Definitions

> **Definition 1 (Free Tree — Graph Theory):**
> An undirected graph $G = (V, E)$ with vertex set $V$ and edge set $E$ is a **free tree** if and only if it is **connected** and **acyclic** (contains no cycles).

> **Definition 2 (Rooted Tree — Recursive Definition):**
> A **rooted tree** $T$ is a finite set of $n \ge 0$ nodes such that:
> 1. If $n = 0$, $T = \emptyset$ is the **empty (null) tree**.
> 2. If $n \ge 1$, there is one specially designated node $r \in T$ called the **root**, and the remaining $n - 1$ nodes are partitioned into $d \ge 0$ disjoint sets $T_1, T_2, \dots, T_d$, each of which is itself a rooted tree called a **subtree** of $r$.

> **Definition 3 (Ordered Tree):**
> A rooted tree $T$ is an **ordered tree** if the subtrees $T_1, T_2, \dots, T_d$ of every node are arranged in a fixed, meaningful left-to-right sequence ($1^{\text{st}}$ child, $2^{\text{nd}}$ child, $\dots$, $d^{\text{th}}$ child). Unless stated otherwise, trees in data structures are **ordered rooted trees**.

---

### 2.2 Master Terminology Reference and Annotated Diagram

Consider the following rooted tree $T$ with $n = 11$ nodes:

```text
Level / Depth                                                  Node Height
  (0-based)                                                     (0-based)
  ---------                                                     ---------
     d = 0                        [ A ]                          h(A) = 3
                                /   |   \
                              /     |     \
     d = 1                [ B ]   [ C ]   [ D ]                  h(B)=2, h(C)=0, h(D)=1
                          /   \           / | \
     d = 2            [ E ]   [ F ]    [G] [H] [I]               h(E)=1, h(F)=h(G)=h(H)=h(I)=0
                      /   \
     d = 3        [ J ]   [ K ]                                  h(J) = 0, h(K) = 0
```

| Term | Formal Definition | Example in Diagram Above |
| :--- | :--- | :--- |
| **Node (Vertex)** | Basic unit of a tree containing a data payload and links to other nodes. | `A, B, C, D, E, F, G, H, I, J, K` ($n = 11$ nodes) |
| **Edge (Branch / Link)** | Directed connection $(u, v)$ from parent node $u$ to child node $v$. | `(A, B)`, `(B, E)`, `(E, J)` ($|E| = 10$ edges) |
| **Root** | The unique node with no parent (in-degree $= 0$). | `A` |
| **Parent** | For directed edge $(u, v)$, $u$ is the unique parent of $v$. | `B` is the parent of `E` and `F` |
| **Child** | For directed edge $(u, v)$, $v$ is a child of $u$. | `G`, `H`, `I` are children of `D` |
| **Siblings** | Nodes that share the exact same parent. | `{B, C, D}` are siblings; `{G, H, I}` are siblings |
| **Degree of a Node ($\deg(u)$)** | The number of **children** (out-degree) of node $u$. *(Note: Unlike graph theory, the parent edge is NOT counted in tree node degree.)* | $\deg(A)=3,\ \deg(B)=2,\ \deg(C)=0,\ \deg(D)=3$ |
| **Degree of a Tree ($\Delta(T)$)** | The maximum node degree in the tree: $\Delta(T) = \max_{u \in V} \deg(u)$. | $\Delta(T) = \max(3, 2, 0, 3, \dots) = 3$ (a $3$-ary tree) |
| **Leaf / External / Terminal Node** | A node with no children ($\deg(u) = 0$). | `C, F, G, H, I, J, K` ($L = 7$ leaves) |
| **Internal / Non-Terminal Node** | A node with at least one child ($\deg(u) \ge 1$). Includes the root (unless $n=1$). | `A, B, D, E` ($I = 4$ internal nodes) |
| **Path & Path Length** | A sequence of nodes $v_0, v_1, \dots, v_k$ where each $v_i$ is the parent of $v_{i+1}$. Its **length** is the number of edges $k$. | Path $A \to B \to E \to K$ has length $3$ edges ($4$ nodes) |
| **Ancestor / Descendant** | If a path exists from $u$ to $v$, $u$ is an **ancestor** of $v$ and $v$ is a **descendant** of $u$. (*Proper* if $u \ne v$.) | Proper ancestors of `J`: `{E, B, A}`. Proper descendants of `B`: `{E, F, J, K}` |
| **Depth of a Node ($d(u)$)** | Length (number of edges) of the unique path from the **root** to node $u$. | $d(A)=0,\ d(B)=1,\ d(E)=2,\ d(J)=3$ |
| **Height of a Node ($h(u)$)** | Length (number of edges) of the **longest downward path** from node $u$ to a leaf in $u$'s subtree. | $h(J)=0,\ h(E)=1,\ h(B)=2,\ h(A)=3$ |
| **Height of a Tree ($h(T)$)** | The height of the root node, equal to the maximum depth of any node in $T$: $h(T) = \max_{u \in V} d(u)$. | $h(T) = h(A) = 3$ (0-based edge convention) |
| **Level of a Node** | The set of all nodes at the same depth $d$. *(0-based level $= d$; 1-based level $= d + 1$.)* | Level $2$ (0-based): `{E, F, G, H, I}` |
| **Subtree ($T_u$)** | The tree rooted at node $u$ comprising $u$ and all of $u$'s descendants. | $T_B$ contains `{B, E, F, J, K}` |

---

### 2.3 Depth vs. Height: The Two Standard Conventions

One of the most frequent sources of exam mistakes is confusing **Depth** with **Height**, or mixing the **0-based (edge-count)** and **1-based (node-count)** height conventions.

1. **Direction of Measurement**:
   - **Depth ($d(u)$)** measures **top-down** from the root to $u$:
     $$d(\text{root}) = 0, \qquad d(u) = d(\text{parent}(u)) + 1$$
   - **Height ($h(u)$)** measures **bottom-up** from the furthest leaf in $T_u$ up to $u$:
     $$h(\text{leaf}) = 0, \qquad h(u) = 1 + \max_{v \in \text{children}(u)} h(v)$$

2. **Convention A (0-Based / Edge-Count Convention — Standard in CLRS, Knuth, Weiss)**:
   - Height is the **number of edges** on the longest root-to-leaf path.
   - Empty tree: $h(\emptyset) = -1$.
   - Single-node tree (root only): $h(\{r\}) = 0$.
   - Max nodes in a binary tree of height $h$: $n_{\max} = 2^{h+1} - 1$.

3. **Convention B (1-Based / Node-Count Convention — Used in some introductory texts)**:
   - Height $H$ is the **number of nodes (levels)** on the longest root-to-leaf path ($H = h + 1$).
   - Empty tree: $H(\emptyset) = 0$.
   - Single-node tree (root only): $H(\{r\}) = 1$.
   - Max nodes in a binary tree of height $H$: $n_{\max} = 2^{H} - 1$.

> **Note:** Throughout these notes, we default to the **0-based edge-count convention ($h$)** and explicitly display the **1-based node-count equivalent ($H = h + 1$)** alongside it so you can effortlessly answer questions under either convention.

---

### 2.4 Fundamental Mathematical Theorems and Proofs

#### Theorem 1: Edge-Vertex Invariant of a Tree
> **Statement:** Every rooted (or free) tree $T = (V, E)$ with $n = |V| \ge 1$ vertices has exactly:
> $$|E| = n - 1$$

**Proof (Bijection Argument):**
In a rooted tree, every node $v \in V \setminus \{r\}$ has **exactly one parent**, and therefore has **exactly one incoming edge** $(\text{parent}(v), v)$ directed into it. The root $r$ has **zero incoming edges**. Since every edge $e = (u, v) \in E$ enters a unique non-root child vertex $v$, there is a one-to-one correspondence (bijection) between the set of edges $E$ and the set of non-root vertices $V \setminus \{r\}$. Because $|V \setminus \{r\}| = n - 1$, it follows immediately that $|E| = n - 1$. $\blacksquare$

---

#### Theorem 2: Maximum Nodes at Depth $d$ in an $N$-Ary Tree
> **Statement:** In an $N$-ary tree (where every node has at most $N$ children), the maximum number of nodes at depth $d \ge 0$ is:
> $$n_d \le N^d$$
> *Special Case for Binary Trees ($N = 2$):* At depth $d$, there are at most $2^d$ nodes.

**Proof (By Mathematical Induction on $d$):**
- **Base Case ($d = 0$):** Depth $0$ contains only the root node. Since $1 \le N^0 = 1$, the base case holds.
- **Inductive Hypothesis:** Assume that at depth $d = k$, the number of nodes satisfies $n_k \le N^k$.
- **Inductive Step ($d = k + 1$):** Every node at depth $k + 1$ must be a child of some node at depth $k$. Since each of the $n_k$ nodes at depth $k$ can have at most $N$ children, the number of nodes at depth $k + 1$ satisfies:
  $$n_{k+1} \le N \cdot n_k \le N \cdot N^k = N^{k+1}$$
  By induction, $n_d \le N^d$ for all $d \ge 0$. $\blacksquare$

---

#### Theorem 3: Maximum Total Nodes in an $N$-Ary Tree of Height $h$
> **Statement:** An $N$-ary tree ($N \ge 2$) of 0-based edge height $h \ge 0$ (or 1-based node height $H = h + 1$) contains at most:
> $$n_{\max} = \frac{N^{h+1} - 1}{N - 1} = \frac{N^H - 1}{N - 1}$$
> *Special Case for Binary Trees ($N = 2$):*
> $$n_{\max} = 2^{h+1} - 1 = 2^H - 1$$

**Proof:**
A tree of height $h$ has levels $d = 0, 1, 2, \dots, h$. Summing the maximum possible nodes at each level from Theorem 2 gives a finite geometric series:
$$n \le \sum_{d=0}^{h} N^d = 1 + N + N^2 + \dots + N^h = \frac{N^{h+1} - 1}{N - 1}$$
Substituting $N = 2$ yields $\frac{2^{h+1} - 1}{2 - 1} = 2^{h+1} - 1$. $\blacksquare$

---

#### Theorem 4: Minimum and Maximum Height of an $N$-Ary Tree with $n$ Nodes
> **Statement:** For any $N$-ary tree ($N \ge 2$) with $n \ge 1$ nodes, its 0-based height $h$ satisfies:
> $$\left\lceil \log_N\!\big(n(N - 1) + 1\big) \right\rceil - 1 \;\le\; h \;\le\; n - 1$$
> *Special Case for Binary Trees ($N = 2$):*
> $$\lfloor \log_2 n \rfloor = \lceil \log_2(n + 1) \rceil - 1 \;\le\; h \;\le\; n - 1$$
> *(In 1-based node height $H$: $\lfloor \log_2 n \rfloor + 1 \le H \le n$.)*

**Proof:**
1. **Upper Bound ($h \le n - 1$):** Every level $d = 0, 1, \dots, h$ must contain at least $1$ node. Thus $n \ge h + 1 \implies h \le n - 1$ (achieved when the tree is a single degenerate chain).
2. **Lower Bound:** From Theorem 3, a tree of height $h$ can hold at most $\frac{N^{h+1} - 1}{N - 1}$ nodes, and because its height is $h$ (not $h - 1$), $n$ exceeds the capacity of height $h - 1$:
   $$\frac{N^h - 1}{N - 1} < n \le \frac{N^{h+1} - 1}{N - 1}$$
   Multiplying by $(N - 1)$ and adding $1$:
   $$N^h < n(N - 1) + 1 \le N^{h+1}$$
   Taking $\log_N$ and noting that $h + 1$ is an integer:
   $$h + 1 = \left\lceil \log_N\!\big(n(N - 1) + 1\big) \right\rceil \implies h_{\min} = \left\lceil \log_N\!\big(n(N - 1) + 1\big) \right\rceil - 1 \quad \blacksquare$$

---

#### Theorem 5: The Generalized Degree-Leaf Theorem & Binary Tree Leaf Theorem ($L = I_2 + 1$)
> **Statement (General $N$-Ary Tree):** Let $I_i$ denote the number of nodes in an $N$-ary tree that have exactly $i$ children ($0 \le i \le N$), where $L = I_0$ is the number of leaf nodes. Then:
> $$L = 1 + \sum_{i=2}^{N} (i - 1) I_i = 1 + I_2 + 2I_3 + 3I_4 + \dots + (N - 1)I_N$$
> **Corollary 5.1 (Binary Tree Degree-2 Leaf Theorem):** In any non-empty binary tree ($N = 2$), the number of leaf nodes ($L = I_0$) is **always one more** than the number of nodes with two children ($I_2$):
> $$L = I_2 + 1$$
> **Corollary 5.2 (Full / Strict $N$-Ary Tree Leaf Theorem):** In a full $N$-ary tree where every internal node has **exactly $N$ children** ($I_1 = I_2 = \dots = I_{N-1} = 0$, and $I = I_N$ internal nodes), the number of leaves is:
> $$L = (N - 1)I + 1 \qquad \text{and total nodes } n = L + I = N \cdot I + 1$$

**Proof:**
Let $n$ be the total number of nodes in the tree. Partitioning the nodes by their out-degree $i \in \{0, 1, \dots, N\}$ gives:
$$n = I_0 + I_1 + I_2 + \dots + I_N = L + \sum_{i=1}^{N} I_i \qquad \text{(Equation 1)}$$
Next, count the total number of edges $|E|$ by summing the out-degrees (number of children) over all nodes:
$$|E| = 0 \cdot I_0 + 1 \cdot I_1 + 2 \cdot I_2 + \dots + N \cdot I_N = \sum_{i=1}^{N} i \cdot I_i \qquad \text{(Equation 2)}$$
By **Theorem 1**, $|E| = n - 1$, or equivalently $n = |E| + 1$. Substituting Equation 1 and Equation 2 into $n = |E| + 1$:
$$L + \sum_{i=1}^{N} I_i = 1 + \sum_{i=1}^{N} i \cdot I_i$$
Subtracting $\sum_{i=1}^{N} I_i$ from both sides:
$$L = 1 + \sum_{i=1}^{N} (i - 1) I_i = 1 + 0 \cdot I_1 + 1 \cdot I_2 + 2 \cdot I_3 + \dots + (N - 1)I_N = 1 + \sum_{i=2}^{N} (i - 1) I_i$$
Notice that degree-1 nodes ($I_1$) completely cancel out ($(1 - 1)I_1 = 0$)! Setting $N = 2$ immediately yields $L = I_2 + 1$. $\blacksquare$

---

## 3. N-Ary Trees: Concepts and Memory Representations in C

An **N-Ary Tree** (also called a **$k$-ary tree** or **general tree**) is a rooted tree in which each node has at most $N$ children (or an arbitrary number of children in an unbounded general tree). Because different nodes in a general tree can have wildly varying degrees, choosing the right C memory layout is a critical design decision.

### 3.1 Fixed-Size Pointer Array Representation

If the maximum branching factor $N$ is known in advance and small (e.g., $N = 3$ for a ternary tree or $N = 26$ for a lowercase alphabet Trie), each node can store a fixed array of $N$ child pointers:

```c
#define MAX_CHILDREN 3

typedef struct NAryNodeFixed {
    int data;
    struct NAryNodeFixed *children[MAX_CHILDREN];
} NAryNodeFixed;
```

```text
+------+--------------+--------------+--------------+
| data | children[0]  | children[1]  | children[2]  |
+------+------+-------+------+-------+------+-------+
              |              |              |
              v              v              v
           Child 0        Child 1         NULL
```

- **Null Pointer Waste Theorem:** An $N$-ary tree of $n$ nodes contains $n \cdot N$ total child pointer slots across all nodes. Since only $|E| = n - 1$ pointers actually point to child nodes, the number of wasted `NULL` pointers is:
  $$\text{NULL Pointers} = nN - (n - 1) = n(N - 1) + 1$$
  For $N = 10$ and $n = 1000$, $9001$ out of $10000$ pointers ($90\%$) are wasted `NULL`s!

---

### 3.2 Dynamic Resizable Array (`realloc`) Representation

Instead of allocating $N$ pointers for every node, each node maintains a heap-allocated array `children` sized dynamically to its actual degree (using a geometric doubling capacity strategy):

```c
typedef struct NAryNode {
    char label;
    struct NAryNode **children; /* Heap-allocated array of child pointers */
    int child_count;            /* Current number of children (degree) */
    int capacity;               /* Allocated slots in children[] */
} NAryNode;
```

```text
NAryNode 'A' (child_count = 3, capacity = 4)
+-------+----------+-------------+----------+
|  'A'  | children | child_count | capacity |
+-------+----+-----+-------------+----------+
             |
             v  (Heap Block of NAryNode*)
           +---------+---------+---------+---------+
           |  &B     |  &C     |  &D     |  unused |
           +---------+---------+---------+---------+
```

- **Pros:** $O(1)$ random access to the $i^{\text{th}}$ child (`node->children[i]`); leaves allocate `NULL` (`0` extra pointer array bytes).
- **Cons:** Requires two heap allocations per internal node (one for the struct, one for `children[]`) and metadata overhead (`child_count`, `capacity`).

---

### 3.3 First-Child / Next-Sibling (LCRS) Representation

Also known as the **Left-Child Right-Sibling (LCRS)** or **Knuth Representation**, this scheme represents *any* general or $N$-ary tree using **only two pointers per node**, regardless of how many children a node has!

```c
typedef struct LCRSNode {
    char label;
    struct LCRSNode *first_child;  /* Points to the leftmost child */
    struct LCRSNode *next_sibling; /* Points to the immediate right sibling */
} LCRSNode;
```

```text
Logical N-Ary View:              Physical LCRS Memory Layout:
         [ A ]                              [ A ]
       /   |   \                            /   \
     [B]  [C]  [D]                   first_child  NULL
     / \        |                        /
   [E] [F]     [G]                     [B] ---> [C] ---> [D] ---> NULL
                                       /        /        /
                                     [E]->[F] NULL     [G]
```

- **Pros:** Uniform node size (exactly $2$ pointers per node, matching a binary tree!); zero `realloc` overhead; establishes a direct 1-to-1 structural isomorphism with binary trees.
- **Cons:** Accessing the $i^{\text{th}}$ child of a node requires $O(i)$ sequential traversal along the `next_sibling` chain instead of $O(1)$ array indexing.

| Representation | Memory per Node | Access $i^{\text{th}}$ Child | Wasted Pointers (Total) | Best Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **Fixed Array (`children[N]`)** | $\text{Data} + N \text{ ptrs}$ | $O(1)$ | $n(N - 1) + 1$ | Small, fixed $N$ (e.g., Quadtrees $N=4$, Octrees $N=8$) |
| **Dynamic Array (`**children`)** | $\text{Data} + 1 \text{ ptr} + 2 \text{ ints}$ | $O(1)$ | $0$ to $\le n$ (amortized) | General ASTs, DOM trees, scene graphs with fast child indexing |
| **First-Child / Next-Sibling (LCRS)** | $\text{Data} + 2 \text{ ptrs}$ | $O(i)$ | $n + 1$ | File systems, general trees with unbounded/variable degree |

---

## 4. Binary Trees and Common Structural Types

### 4.1 Formal Definition of a Binary Tree

> **Definition 4 (Binary Tree):**
> A **binary tree** $B$ is a finite set of nodes that is either:
> 1. Empty ($B = \emptyset$), or
> 2. Consists of a **root** node $r$ together with two disjoint binary trees $B_L$ and $B_R$, called the **left subtree** and **right subtree** of $r$, respectively.

> **Critical Distinction (Binary Tree vs. Ordered Tree of Degree $\le 2$):**
> A binary tree is **NOT** merely an $N$-ary tree with $N = 2$. In a binary tree, even if a node has **only one child**, that child is explicitly distinguished as either a **left child** or a **right child**. The two 2-node binary trees below are **distinct** binary trees, whereas as general ordered trees of degree $\le 2$ they would be identical (a root with a single first child):
> ```text
>     Tree 1 (Left Child)          Tree 2 (Right Child)
>            [ A ]                        [ A ]
>            /                              \
>          [ B ]                            [ B ]
> ```

---

### 4.2 Five Canonical Types of Binary Trees

#### 1. Full (Strict / Proper / 2-Tree) Binary Tree
- **Definition:** Every node in the tree has **either $0$ children (leaf) or $2$ children**; no node has degree $1$ ($I_1 = 0$).
- **Mathematical Properties:**
  - Since $I_1 = 0$ and $L = I_2 + 1$, total internal nodes $I = I_2$, so:
    $$L = I + 1, \qquad n = L + I = 2I + 1 = 2L - 1$$
  - Consequently, the total number of nodes $n$ in a Full Binary Tree is **always odd**.
  - For a given 0-based height $h$: $2h + 1 \le n \le 2^{h+1} - 1$.

#### 2. Complete Binary Tree
- **Definition:** A binary tree of height $h$ in which:
  1. Every level $d \in \{0, 1, \dots, h - 1\}$ is **completely filled** (contains $2^d$ nodes), and
  2. At the last level $h$, all nodes are packed **as far to the left as possible** (no gaps in level-order numbering).
- **Mathematical Properties:**
  - Total nodes $n$ satisfies: $2^h \le n \le 2^{h+1} - 1$.
  - Height is strictly $h = \lfloor \log_2 n \rfloor$.
  - At most **one** node in the entire tree can have degree $1$ ($I_1 \in \{0, 1\}$): specifically, $I_1 = 1$ when $n$ is even, and $I_1 = 0$ (making it also a Full Binary Tree) when $n$ is odd.
  - **Key Application:** Array-based **Binary Heaps (Priority Queues)** rely on Complete Binary Trees because elements map contiguously to indices $0 \dots n - 1$ with **zero empty gaps**.

#### 3. Perfect Binary Tree
- **Definition:** A binary tree in which **every internal node has exactly $2$ children** AND **all leaf nodes are at the exact same depth $h$**.
- **Mathematical Properties:**
  - Every level $d \in \{0, 1, \dots, h\}$ has the maximum possible $2^d$ nodes:
    $$n = 2^{h+1} - 1, \qquad L = 2^h, \qquad I = 2^h - 1, \qquad h = \log_2(n + 1) - 1$$
  - **Subset Relationship:** Every **Perfect** binary tree is both **Full** and **Complete**, but a tree can be Full without being Complete, and Complete without being Full!

#### 4. Balanced (Height-Balanced) Binary Tree
- **Definition (AVL Criterion):** A binary tree in which, for **every** node $u \in T$, the heights of its left and right subtrees differ by at most $1$:
  $$\forall u \in T, \quad \big|\text{height}(u.\text{left}) - \text{height}(u.\text{right})\big| \le 1$$
  *(More generally, a class of binary trees is balanced if $h = O(\log n)$ as $n \to \infty$, e.g., AVL trees and Red-Black trees).*

#### 5. Degenerate (Pathological / Skewed) Binary Tree
- **Definition:** A binary tree in which every internal node has **only $1$ child** ($I_2 = 0,\ I_1 = n - 1$).
- **Subtypes:** **Left-skewed** (only left children), **Right-skewed** (only right children), or **Zig-Zag** degenerate.
- **Mathematical Properties:**
  - $L = 1$ (only a single leaf node at the bottom!), $h = n - 1$.
  - Behaves identically to a singly linked list; search/insert/delete degrade from $O(\log n)$ to worst-case $O(n)$.

```text
1. FULL (Strict)        2. COMPLETE             3. PERFECT            5. DEGENERATE (Skewed)
(0 or 2 children;       (All levels full        (All internal deg 2   (Every internal node
 not complete!)          except last, left-      AND all leaves at     has 1 child; h = n-1)
                         packed; not full!)      same depth h=2)
      [ A ]                   [ A ]                   [ A ]             [A]          [A]
      /   \                   /   \                   /   \             /              \
    [B]   [C]               [B]   [C]               [B]   [C]         [B]              [C]
          / \               / \   /                 / \   / \         /                /
        [D] [E]           [D][E] [F]              [D][E] [F][G]     [D]              [E]
   n=5, L=3, I2=2          n=6, L=3, I1=1          n=7, L=4, I2=3    Left-Skewed    Zig-Zag
```

---

### 4.3 Array (Sequential) vs. Pointer (Linked) Representation in C

#### A. Array (Sequential) Representation
By numbering nodes in **level-order** (top-to-bottom, left-to-right), a binary tree can be stored in a single flat array `tree[]` without any pointers:

| Relationship | 0-Based Indexing (`root` at `i = 0`) | 1-Based Indexing (`root` at `i = 1`) |
| :--- | :--- | :--- |
| **Root Index** | `0` | `1` |
| **Left Child of `i`** | `2 * i + 1` | `2 * i` |
| **Right Child of `i`** | `2 * i + 2` | `2 * i + 1` |
| **Parent of `i` (`i > root`)** | `(i - 1) / 2` (integer division) | `i / 2` (integer division) |

```text
Binary Tree (Missing left child of C):        0-Based Array Layout:
              [ A ] (0)                       Index:   0     1     2     3     4     5     6
              /   \                           Value: [ A ] [ B ] [ C ] [ D ] [ E ] [ - ] [ G ]
        (1) [B]   [C] (2)                                                      ^
            / \     \                                                   Empty gap at 2*2+1 = 5!
      (3) [D] [E]   [G] (6)
          (4)
```

- **When Array Representation Excels:** For **Complete** and **Perfect** binary trees (like Binary Heaps), occupancy is $100\%$ with zero gaps and superb CPU cache locality.
- **When Array Representation Fails:** For a right-skewed degenerate tree of $n$ nodes, the $k^{\text{th}}$ node (0-indexed) lands at index $2^{k+1} - 2$. Storing just $n = 20$ right-skewed nodes requires an array of length $2^{20} - 1 = 1{,}048{,}575$ slots ($99.998\%$ wasted memory!).

#### B. Pointer (Linked) Representation
Each node is dynamically allocated on the heap with explicit `left` and `right` pointers:

```c
typedef struct BTNode {
    int data;
    struct BTNode *left;
    struct BTNode *right;
} BTNode;
```

- **Null-Pointer Count in Any Linked Binary Tree:** Across $n$ nodes, there are $2n$ pointer fields (`left` and `right`). Exactly $n - 1$ pointers point to non-root nodes, so **every** linked binary tree of $n$ nodes contains **exactly $n + 1$ `NULL` pointers** (regardless of whether the tree is balanced or skewed!).
  *(Proof Check: Each degree-0 leaf has $2$ `NULL`s, and each degree-1 node has $1$ `NULL`: $2L + I_1 = 2(I_2 + 1) + I_1 = (I_2 + 1) + I_1 + I_2 + 1 = L + I_1 + I_2 + 1 = n + 1$.)*

---

### 4.4 Binary Search Tree (BST) Fundamentals and Ordering Invariant

A general Binary Tree imposes **no ordering constraint** on the data values stored in its nodes—searching for a value $x$ requires inspecting both subtrees in $O(n)$ time. A **Binary Search Tree (BST)** augments the binary tree structure with a global **Search Ordering Invariant**:

> **Definition 5 (Binary Search Tree Invariant):**
> A binary tree $T$ is a **Binary Search Tree (BST)** (with distinct keys) if and only if for **every** node $u \in T$:
> 1. Every key in the **entire left subtree** $T_{\text{left}}(u)$ is strictly **less** than $u$'s key:
>    $$\forall x \in T_{\text{left}}(u), \quad \text{key}(x) < \text{key}(u)$$
> 2. Every key in the **entire right subtree** $T_{\text{right}}(u)$ is strictly **greater** than $u$'s key:
>    $$\forall y \in T_{\text{right}}(u), \quad \text{key}(y) > \text{key}(u)$$
> 3. Both $T_{\text{left}}(u)$ and $T_{\text{right}}(u)$ are themselves valid Binary Search Trees.

```text
        VALID BST                              INVALID BST (Classic Trap!)
          [ 20 ]                                         [ 20 ]
          /    \                                         /    \
       [10]    [30]                                   [10]    [30]
       /  \    /  \                                   /  \
     [5] [15][25] [40]                              [5] [25]  <-- 25 > 10 locally,
   All left < 20 < All right                            but 25 > 20 violates root's
                                                        left-subtree bound (-inf, 20)!
```

- **Why the Global Subtree Constraint Matters:** Checking only `u->left->key < u->key && u->right->key > u->key` is **not sufficient** to verify a BST! In the right diagram above, `25 > 10` satisfies its immediate parent `10`, but `25` sits inside the left subtree of `20`, violating $25 < 20$. Validating a BST requires passing down an allowable key interval `(min_val, max_val)` recursively (implemented in Section 6.4).
- **Forward Link:** In **Unit 3**, we will build upon this invariant to study full BST search, insertion, three-case Hibbard deletion, and self-balancing trees (AVL Trees).

---

## 5. Forests

### 5.1 Formal Definition and Edge-Vertex-Component Theorem

> **Definition 6 (Forest):**
> - **Graph-Theoretic Definition:** An undirected graph $\mathcal{F} = (V, E)$ is a **forest** if it is **acyclic** (contains no cycles). Equivalently, a forest is a disjoint union of $k \ge 0$ connected components, where each component is a tree.
> - **Ordered Rooted Forest Definition:** An **ordered forest** $\mathcal{F} = (T_1, T_2, \dots, T_k)$ is an ordered sequence of $k \ge 0$ disjoint ordered rooted trees.

#### Theorem 6: Edge-Vertex-Component Formula for a Forest
> **Statement:** Let $\mathcal{F} = (T_1, T_2, \dots, T_k)$ be a forest consisting of $k \ge 0$ disjoint trees with a total of $n = |V|$ vertices and $|E|$ edges. Then:
> $$|E| = |V| - k = n - k$$

**Proof:**
Let tree $T_i$ ($1 \le i \le k$) have $n_i = |V_i|$ vertices and $|E_i|$ edges. Because the trees are disjoint, $\sum_{i=1}^{k} n_i = n$ and $\sum_{i=1}^{k} |E_i| = |E|$. By **Theorem 1**, each individual tree satisfies $|E_i| = n_i - 1$. Summing across all $k$ trees:
$$|E| = \sum_{i=1}^{k} |E_i| = \sum_{i=1}^{k} (n_i - 1) = \left(\sum_{i=1}^{k} n_i\right) - \sum_{i=1}^{k} 1 = n - k \quad \blacksquare$$

---

### 5.2 Structural Duality Between Trees and Forests

Trees and forests are mutually recursive structures:
1. **Tree $\to$ Forest (Root Removal):** Removing the root node $r$ (of degree $d$) from any rooted tree $T$ immediately leaves behind an ordered forest of $d$ disjoint subtrees $\mathcal{F} = (T_1, T_2, \dots, T_d)$.
2. **Forest $\to$ Tree (Virtual Root Addition):** Given any forest $\mathcal{F} = (T_1, T_2, \dots, T_k)$ of $k$ trees, creating a new virtual root node $r^*$ and adding $k$ edges from $r^*$ to the roots of $T_1, \dots, T_k$ merges the entire forest into a single rooted tree with $|V| + 1$ vertices and $(|V| - k) + k = |V|$ edges.

---

## 6. Operations and Conversion Algorithms (Knuth / LCRS Transform)

Because binary trees have fixed 2-pointer nodes (`left`, `right`) and a rich library of algorithms, Donald Knuth formalized a fundamental **one-to-one correspondence (bijection)** between general $N$-ary trees / ordered forests and binary trees using the **Left-Child Right-Sibling (LCRS)** transformation.

---

### 6.1 Converting an N-Ary Tree to a Binary Tree

#### Purpose & Preconditions
- **Purpose:** Transform any ordered $N$-ary tree $T$ of $n$ nodes into an equivalent binary tree $B(T)$ of $n$ nodes with zero information loss.
- **Preconditions:** $T$ is a valid ordered rooted tree (or `NULL`).

#### Step-by-Step Algorithm (The LCRS / $45^\circ$ Clockwise Rotation Rule)
For every node $u$ in the $N$-ary tree with ordered children $c_1, c_2, \dots, c_d$:
1. **Left Pointer = First Child:** Set `u->left` in the binary tree to point to $u$'s **first (leftmost) child** $c_1$ (or `NULL` if $u$ is a leaf).
2. **Right Pointer = Next Sibling:** Set `u->right` in the binary tree to point to $u$'s **immediate right sibling** (the next child of $u$'s parent), or `NULL` if $u$ is the last child of its parent.
3. **Visual Intuition:**
   - **Step A:** Draw a horizontal dashed line connecting each sibling to its immediate right sibling ($c_1 \to c_2 \to \dots \to c_d$).
   - **Step B:** Erase all vertical parent-to-child links **except** the link from each parent to its **first (leftmost) child** $c_1$.
   - **Step C:** Rotate the horizontal sibling links $45^\circ$ clockwise so that first-child links point down-left (`left`) and next-sibling links point down-right (`right`).

#### Crucial Structural Invariant of a Single Converted N-Ary Tree
> **Invariant:** Because the root $r$ of a single $N$-ary tree has **no siblings**, the root of the resulting LCRS Binary Tree $B(T)$ **ALWAYS has an empty right subtree**:
> $$\text{bin\_root}\to\text{right} == \text{NULL}$$

#### Before-and-After ASCII Diagram (Single N-Ary Tree $\to$ Binary Tree)

```text
STEP 1: Original N-Ary Tree T1          STEP 2: Link Siblings & Keep Only First-Child Edge
             [ A ]                                         [ A ]
           /   |   \                                       /
         [B]  [C]  [D]                                   [B] ===> [C] ===> [D]
         / \        |                                    /                 /
       [E] [F]     [G]                                 [E] ===> [F]      [G]

STEP 3: Rotate 45 Degrees Clockwise -> Final LCRS Binary Tree B(T1)
                     [ A ]
                     /   \
          (left)   [B]   NULL   <-- INVARIANT: Root of single converted tree has right == NULL!
                   / \
         (left)  [E] [C] (right = B's next sibling)
                   \   \
                   [F] [D] (right = C's next sibling)
                       /
                     [G] (left = D's first child)
```

---

### 6.2 Converting a Forest to a Binary Tree

#### Purpose & Preconditions
- **Purpose:** Convert an ordered forest $\mathcal{F} = (T_1, T_2, \dots, T_k)$ of $k \ge 1$ disjoint $N$-ary trees into a **single** binary tree $B(\mathcal{F})$.

#### Step-by-Step Algorithm
1. Convert each individual $N$-ary tree $T_i$ ($1 \le i \le k$) into its LCRS binary tree $B(T_i)$ using the algorithm in Section 6.1.
2. Notice that every individual $B(T_i)$ has an unused `right == NULL` pointer at its root $r_i$!
3. Treat the roots $r_1, r_2, \dots, r_k$ of the trees in the forest as **siblings of each other**:
   - Set `B(T_1)->right = B(T_2)`
   - Set `B(T_2)->right = B(T_3)`
   - $\dots$
   - Set `B(T_{k-1})->right = B(T_k)`
   - Set `B(T_k)->right = NULL`

#### Recursive Mathematical Definition of the Forest-to-Binary Isomorphism
Let $\mathcal{F} = (T_1, T_2, \dots, T_k)$ be a forest where tree $T_1$ has root $r_1$ and subtrees $T_{1,1}, T_{1,2}, \dots, T_{1,d}$. Then the binary tree $B(\mathcal{F})$ is defined recursively as:
- If $k = 0$ ($\mathcal{F} = \emptyset$), then $B(\emptyset) = \emptyset$.
- If $k \ge 1$, then $B(T_1, T_2, \dots, T_k)$ is a binary tree whose:
  - **Root** is $r_1$,
  - **Left Subtree** is $B(T_{1,1}, T_{1,2}, \dots, T_{1,d})$ (the forest of subtrees of $r_1$), and
  - **Right Subtree** is $B(T_2, T_3, \dots, T_k)$ (the remaining trees in the forest $\mathcal{F}$).

#### Before-and-After ASCII Diagram (Forest $\mathcal{F} = \{T_1, T_2, T_3\} \to$ Single Binary Tree)

```text
BEFORE: Ordered Forest F = { T1, T2, T3 } (k = 3 trees, |V| = 11 nodes, |E| = 11 - 3 = 8 edges)

       Tree T1                  Tree T2             Tree T3
        [ A ]                    [ H ]               [ K ]
      /   |   \                  /   \
    [B]  [C]  [D]              [I]   [J]
    / \        |
  [E] [F]     [G]

INTERMEDIATE: Individual Binary Trees B(T1), B(T2), B(T3) Chained Along Root Right Spine
      [ A ] ===================> [ H ] =============> [ K ] ---> NULL
      /                          /                    /
    [B]                        [I]                  NULL
    / \                          \
  [E] [C]                        [J]
    \   \
    [F] [D]
        /
      [G]

AFTER: Final Unified Binary Tree B(F) (|V| = 11 nodes, |E_bin| = 10 edges)
                            [ A ]   <-- Root of T1 (and root of B(F))
                           /     \
                        [ B ]    [ H ]  <-- Root of T2 (A->right != NULL proves k >= 2!)
                        /   \    /   \
                      [E]  [C] [I]   [ K ]  <-- Root of T3
                        \    \   \
                        [F]  [D] [J]
                             /
                           [G]
```

#### Structural Distinction: Single Converted Tree vs. Converted Forest
| Binary Tree Root Property | Meaning in LCRS Domain | Number of Trees $k$ in Original Forest |
| :--- | :--- | :--- |
| `bin_root == NULL` | Empty forest | $k = 0$ |
| `bin_root->right == NULL` | Single $N$-ary tree $T_1$ | $k = 1$ |
| `bin_root->right != NULL` | Forest $\mathcal{F} = \{T_1, \dots, T_k\}$ of multiple trees | $k = \text{length of right spine from } \text{bin\_root} \ge 2$ |

---

### 6.3 Reverse Reconstruction: Binary Tree Back to Forest / N-Ary Tree

Because the LCRS mapping is a bijection, **any** binary tree $B$ can be uniquely decoded back into an ordered forest $\mathcal{F} = (T_1, \dots, T_k)$:
1. **Recover the Roots of the Forest:** Start at `curr = bin_root` and walk down the **right spine** (`curr = curr->right`). Each node visited on the top-level right spine is the root of a separate tree $T_1, T_2, \dots, T_k$ in the forest.
2. **Recover the Children of Any Node $u$:** For any node $u$, its children in the $N$-ary tree are obtained by starting at `child = u->left` and walking down `child`'s **right spine** (`child = child->right`). Recursively reconstruct each child's subtree.

#### Pseudocode (Both Directions)

```text
ALGORITHM NaryToBinary(u)
Input:  Root node u of an N-ary tree (with children u.children[0 .. d-1])
Output: Root node b of the equivalent LCRS Binary Tree
1.  if u == NULL then return NULL
2.  b <- CreateBinaryNode(u.label)
3.  if u.child_count > 0 then
4.      b.left <- NaryToBinary(u.children[0])
5.      curr <- b.left
6.      for i <- 1 to u.child_count - 1 do
7.          curr.right <- NaryToBinary(u.children[i])
8.          curr <- curr.right
9.  return b

ALGORITHM ForestToBinary(T[0 .. k-1], k)
Input:  Array of k N-ary tree roots T[0 .. k-1]
Output: Root node of the unified Binary Tree B(F)
1.  if k <= 0 then return NULL
2.  root <- NaryToBinary(T[0])
3.  curr <- root
4.  for i <- 1 to k - 1 do
5.      curr.right <- NaryToBinary(T[i])
6.      curr <- curr.right
7.  return root

ALGORITHM BinaryToForest(bin_root)
Input:  Root node bin_root of an LCRS Binary Tree
Output: List of reconstructed N-ary trees F = [T_1, ..., T_k]
1.  F <- EmptyList()
2.  curr_tree_root <- bin_root
3.  while curr_tree_root != NULL do
4.      Append BinarySubtreeToNary(curr_tree_root) to F
5.      curr_tree_root <- curr_tree_root.right
6.  return F
```

---

### 6.4 Complete C11 Implementation and Execution Trace

The following self-contained C11 program implements dynamic $N$-ary trees, verifies the mathematical node/height/leaf theorems, converts both a single $N$-ary tree and a 3-tree forest into binary trees, and reconstructs the original forest from the binary tree.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <limits.h>

/* =========================================================================
 * 1. DYNAMIC N-ARY TREE DATA STRUCTURE
 * ========================================================================= */
typedef struct NAryNode {
    char label;
    struct NAryNode **children; /* Dynamic array of child pointers */
    int child_count;            /* Current number of children (degree) */
    int capacity;               /* Allocated capacity of children array */
} NAryNode;

/* Create a new N-ary tree node */
NAryNode *nary_create_node(char label) {
    NAryNode *node = (NAryNode *)malloc(sizeof(NAryNode));
    if (node == NULL) {
        fprintf(stderr, "Error: malloc failed for NAryNode '%c'\n", label);
        exit(EXIT_FAILURE);
    }
    node->label = label;
    node->children = NULL;
    node->child_count = 0;
    node->capacity = 0;
    return node;
}

/* Add a child node to a parent in an N-ary tree */
void nary_add_child(NAryNode *parent, NAryNode *child) {
    if (parent == NULL || child == NULL) {
        return;
    }
    if (parent->child_count == parent->capacity) {
        int new_cap = (parent->capacity == 0) ? 2 : parent->capacity * 2;
        NAryNode **new_arr = (NAryNode **)realloc(
            parent->children, (size_t)new_cap * sizeof(NAryNode *)
        );
        if (new_arr == NULL) {
            fprintf(stderr, "Error: realloc failed in nary_add_child\n");
            exit(EXIT_FAILURE);
        }
        parent->children = new_arr;
        parent->capacity = new_cap;
    }
    parent->children[parent->child_count++] = child;
}

/* Count total nodes in an N-ary tree */
int nary_count_nodes(const NAryNode *root) {
    if (root == NULL) {
        return 0;
    }
    int total = 1;
    for (int i = 0; i < root->child_count; i++) {
        total += nary_count_nodes(root->children[i]);
    }
    return total;
}

/* Count leaf nodes (degree 0) in an N-ary tree */
int nary_count_leaves(const NAryNode *root) {
    if (root == NULL) {
        return 0;
    }
    if (root->child_count == 0) {
        return 1;
    }
    int leaves = 0;
    for (int i = 0; i < root->child_count; i++) {
        leaves += nary_count_leaves(root->children[i]);
    }
    return leaves;
}

/* Compute 0-based edge height of an N-ary tree (empty = -1, single node = 0) */
int nary_height_edges(const NAryNode *root) {
    if (root == NULL) {
        return -1;
    }
    int max_child_h = -1;
    for (int i = 0; i < root->child_count; i++) {
        int h = nary_height_edges(root->children[i]);
        if (h > max_child_h) {
            max_child_h = h;
        }
    }
    return max_child_h + 1;
}

/* Find the degree of an N-ary tree (maximum degree among all nodes) */
int nary_tree_degree(const NAryNode *root) {
    if (root == NULL) {
        return 0;
    }
    int max_deg = root->child_count;
    for (int i = 0; i < root->child_count; i++) {
        int child_max = nary_tree_degree(root->children[i]);
        if (child_max > max_deg) {
            max_deg = child_max;
        }
    }
    return max_deg;
}

/* Pretty-print an N-ary tree with indentation */
void nary_print(const NAryNode *root, int depth) {
    if (root == NULL) {
        return;
    }
    for (int i = 0; i < depth; i++) {
        printf("  ");
    }
    printf("|- %c (deg=%d)\n", root->label, root->child_count);
    for (int i = 0; i < root->child_count; i++) {
        nary_print(root->children[i], depth + 1);
    }
}

/* Postorder deallocation of an N-ary tree */
void nary_free(NAryNode *root) {
    if (root == NULL) {
        return;
    }
    for (int i = 0; i < root->child_count; i++) {
        nary_free(root->children[i]);
    }
    free(root->children);
    free(root);
}

/* =========================================================================
 * 2. BINARY TREE DATA STRUCTURE (USED FOR LCRS TRANSFORM & BST CHECK)
 * ========================================================================= */
typedef struct BTNode {
    char label;
    int key;              /* Used when demonstrating BST invariant */
    struct BTNode *left;  /* In LCRS: points to FIRST CHILD */
    struct BTNode *right; /* In LCRS: points to NEXT SIBLING */
} BTNode;

BTNode *bt_create_node(char label, int key) {
    BTNode *node = (BTNode *)malloc(sizeof(BTNode));
    if (node == NULL) {
        fprintf(stderr, "Error: malloc failed for BTNode '%c'\n", label);
        exit(EXIT_FAILURE);
    }
    node->label = label;
    node->key = key;
    node->left = NULL;
    node->right = NULL;
    return node;
}

/* =========================================================================
 * 3. CONVERSION: N-ARY TREE & FOREST <-> BINARY TREE (LCRS / KNUTH)
 * ========================================================================= */

/* Convert a single N-ary Tree to its LCRS Binary Tree representation */
BTNode *nary_to_binary(const NAryNode *root) {
    if (root == NULL) {
        return NULL;
    }
    BTNode *bin_node = bt_create_node(root->label, (int)root->label);

    if (root->child_count > 0) {
        /* Left pointer of binary node is the first child */
        bin_node->left = nary_to_binary(root->children[0]);

        /* Chain all remaining children along the right spine of first child */
        BTNode *curr_sibling = bin_node->left;
        for (int i = 1; i < root->child_count; i++) {
            curr_sibling->right = nary_to_binary(root->children[i]);
            curr_sibling = curr_sibling->right;
        }
    }
    return bin_node;
}

/* Convert a Forest of k N-ary Trees into a single Binary Tree */
BTNode *forest_to_binary(NAryNode *const *trees, int k) {
    if (trees == NULL || k <= 0) {
        return NULL;
    }
    /* Convert first tree T_1 */
    BTNode *forest_bin_root = nary_to_binary(trees[0]);
    BTNode *curr_root = forest_bin_root;

    /* Chain roots of T_2, ..., T_k along the right spine of T_1's root */
    for (int i = 1; i < k; i++) {
        curr_root->right = nary_to_binary(trees[i]);
        curr_root = curr_root->right;
    }
    return forest_bin_root;
}

/* Helper: Reconstruct a single N-ary Node (and its subtree) from an LCRS BTNode
 * Note: This ignores bin_node->right because bin_node->right belongs to its parent's
 * child list (or the forest's root list). */
static NAryNode *binary_node_to_nary_subtree(const BTNode *bin_node) {
    if (bin_node == NULL) {
        return NULL;
    }
    NAryNode *nary_node = nary_create_node(bin_node->label);

    /* All children of nary_node live along the right spine starting at bin_node->left */
    const BTNode *child_cursor = bin_node->left;
    while (child_cursor != NULL) {
        NAryNode *child_nary = binary_node_to_nary_subtree(child_cursor);
        nary_add_child(nary_node, child_nary);
        child_cursor = child_cursor->right;
    }
    return nary_node;
}

/* Reconstruct a Forest of N-ary Trees from an LCRS Binary Tree.
 * Sets *out_k to the number of trees recovered in the forest. */
NAryNode **binary_to_forest(const BTNode *bin_root, int *out_k) {
    *out_k = 0;
    if (bin_root == NULL) {
        return NULL;
    }
    /* Count how many trees are in the forest (length of root's right spine) */
    int count = 0;
    for (const BTNode *curr = bin_root; curr != NULL; curr = curr->right) {
        count++;
    }

    NAryNode **forest = (NAryNode **)malloc((size_t)count * sizeof(NAryNode *));
    if (forest == NULL) {
        fprintf(stderr, "Error: malloc failed in binary_to_forest\n");
        exit(EXIT_FAILURE);
    }

    int idx = 0;
    for (const BTNode *curr = bin_root; curr != NULL; curr = curr->right) {
        forest[idx++] = binary_node_to_nary_subtree(curr);
    }
    *out_k = count;
    return forest;
}

/* Pretty-print a Binary Tree showing Left (L) and Right (R) links */
void bt_print(const BTNode *root, int depth, char branch) {
    if (root == NULL) {
        return;
    }
    for (int i = 0; i < depth; i++) {
        printf("  ");
    }
    printf("[%c] %c\n", branch, root->label);
    bt_print(root->left, depth + 1, 'L');
    bt_print(root->right, depth + 1, 'R');
}

/* Count degree-0 (leaf), degree-1, and degree-2 nodes in a Binary Tree */
void bt_count_degrees(const BTNode *root, int *leaves, int *deg1, int *deg2) {
    if (root == NULL) {
        return;
    }
    int children = (root->left != NULL ? 1 : 0) + (root->right != NULL ? 1 : 0);
    if (children == 0) (*leaves)++;
    else if (children == 1) (*deg1)++;
    else (*deg2)++;

    bt_count_degrees(root->left, leaves, deg1, deg2);
    bt_count_degrees(root->right, leaves, deg1, deg2);
}

/* Validate Binary Search Tree ordering invariant using (min_key, max_key) range */
bool bt_is_valid_bst(const BTNode *root, long min_val, long max_val) {
    if (root == NULL) {
        return true;
    }
    if (root->key <= min_val || root->key >= max_val) {
        return false;
    }
    return bt_is_valid_bst(root->left, min_val, root->key) &&
           bt_is_valid_bst(root->right, root->key, max_val);
}

void bt_free(BTNode *root) {
    if (root == NULL) {
        return;
    }
    bt_free(root->left);
    bt_free(root->right);
    free(root);
}

int main(void) {
    printf("=== 1. N-Ary Tree Construction & Properties ===\n");
    /* Construct Tree T1:
     *         A
     *      /  |  \
     *     B   C   D
     *    / \      |
     *   E   F     G
     */
    NAryNode *A = nary_create_node('A');
    NAryNode *B = nary_create_node('B');
    NAryNode *C = nary_create_node('C');
    NAryNode *D = nary_create_node('D');
    NAryNode *E = nary_create_node('E');
    NAryNode *F = nary_create_node('F');
    NAryNode *G = nary_create_node('G');

    nary_add_child(A, B);
    nary_add_child(A, C);
    nary_add_child(A, D);
    nary_add_child(B, E);
    nary_add_child(B, F);
    nary_add_child(D, G);

    nary_print(A, 0);
    printf("Total Nodes (n)        : %d\n", nary_count_nodes(A));
    printf("Leaf Nodes (L)         : %d\n", nary_count_leaves(A));
    printf("Tree Degree (Delta)    : %d\n", nary_tree_degree(A));
    printf("Height (0-based edges) : %d\n", nary_height_edges(A));
    printf("Height (1-based nodes) : %d\n\n", nary_height_edges(A) + 1);

    printf("=== 2. Single N-Ary Tree T1 -> LCRS Binary Tree ===\n");
    BTNode *bin_T1 = nary_to_binary(A);
    bt_print(bin_T1, 0, '*');
    printf("Invariant check: bin_T1->right == NULL? %s\n",
           (bin_T1->right == NULL) ? "YES (Single Tree)" : "NO (Forest)");

    int L = 0, I1 = 0, I2 = 0;
    bt_count_degrees(bin_T1, &L, &I1, &I2);
    printf("Binary Tree Degree Theorem: L=%d, I1=%d, I2=%d -> L == I2 + 1 (%d == %d)\n\n",
           L, I1, I2, L, I2 + 1);

    printf("=== 3. Forest F = {T1, T2, T3} -> LCRS Binary Tree ===\n");
    /* Tree T2: H -> (I, J) */
    NAryNode *H = nary_create_node('H');
    nary_add_child(H, nary_create_node('I'));
    nary_add_child(H, nary_create_node('J'));

    /* Tree T3: K (single root node) */
    NAryNode *K = nary_create_node('K');

    NAryNode *forest_in[3] = {A, H, K};
    BTNode *bin_forest = forest_to_binary(forest_in, 3);
    bt_print(bin_forest, 0, '*');
    printf("Invariant check: bin_forest->right != NULL? %s\n\n",
           (bin_forest->right != NULL) ? "YES (Forest with k >= 2 trees)" : "NO");

    printf("=== 4. Reverse Reconstruction: Binary Tree -> Forest ===\n");
    int recovered_k = 0;
    NAryNode **recovered_forest = binary_to_forest(bin_forest, &recovered_k);
    printf("Recovered %d trees from Binary Tree:\n", recovered_k);
    for (int i = 0; i < recovered_k; i++) {
        printf("Tree %d:\n", i + 1);
        nary_print(recovered_forest[i], 1);
        nary_free(recovered_forest[i]);
    }
    free(recovered_forest);

    bt_free(bin_T1);
    bt_free(bin_forest);
    nary_free(A);
    nary_free(H);
    nary_free(K);
    return 0;
}
```

#### Program Output:

```text
=== 1. N-Ary Tree Construction & Properties ===
|- A (deg=3)
  |- B (deg=2)
    |- E (deg=0)
    |- F (deg=0)
  |- C (deg=0)
  |- D (deg=1)
    |- G (deg=0)
Total Nodes (n)        : 7
Leaf Nodes (L)         : 4
Tree Degree (Delta)    : 3
Height (0-based edges) : 2
Height (1-based nodes) : 3

=== 2. Single N-Ary Tree T1 -> LCRS Binary Tree ===
[*] A
  [L] B
    [L] E
      [R] F
    [R] C
      [R] D
        [L] G
Invariant check: bin_T1->right == NULL? YES (Single Tree)
Binary Tree Degree Theorem: L=2, I1=4, I2=1 -> L == I2 + 1 (2 == 2)

=== 3. Forest F = {T1, T2, T3} -> LCRS Binary Tree ===
[*] A
  [L] B
    [L] E
      [R] F
    [R] C
      [R] D
        [L] G
  [R] H
    [L] I
      [R] J
    [R] K
Invariant check: bin_forest->right != NULL? YES (Forest with k >= 2 trees)

=== 4. Reverse Reconstruction: Binary Tree -> Forest ===
Recovered 3 trees from Binary Tree:
Tree 1:
  |- A (deg=3)
    |- B (deg=2)
      |- E (deg=0)
      |- F (deg=0)
    |- C (deg=0)
    |- D (deg=1)
      |- G (deg=0)
Tree 2:
  |- H (deg=2)
    |- I (deg=0)
    |- J (deg=0)
Tree 3:
  |- K (deg=0)
```

---

## 7. Comparison Tables and Complexity Summary

### 7.1 N-Ary Tree vs. Binary Tree

| Feature | N-Ary (General) Tree | Binary Tree |
| :--- | :--- | :--- |
| **Max Children per Node** | Up to $N$ (or unbounded in general trees) | Strictly at most $2$ (`left` and `right`) |
| **Single-Child Distinction** | No left/right distinction for a single child (it is simply the $1^{\text{st}}$ child) | Explicit distinction: a single child is either a **left child** or a **right child** |
| **Max Nodes at Depth $d$** | $N^d$ | $2^d$ |
| **Max Nodes of Height $h$ (0-based)** | $\dfrac{N^{h+1} - 1}{N - 1}$ | $2^{h+1} - 1$ |
| **Leaf-to-Internal Node Formula** | $L = 1 + \sum_{i=2}^{N} (i - 1)I_i$ | $L = I_2 + 1$ |
| **Pointer Storage per Node** | Variable (`realloc` array) or $N$ pointers, or 2 pointers via LCRS | Fixed: $2$ pointers (`left`, `right`) |

---

### 7.2 General Binary Tree vs. Binary Search Tree (BST)

| Feature | General Binary Tree | Binary Search Tree (BST) |
| :--- | :--- | :--- |
| **Key Ordering Constraint** | **None** — nodes may store any value at any position | **Strict Global Invariant:** $\text{keys}(T_L) < \text{key}(u) < \text{keys}(T_R)$ for all $u$ |
| **Search Time (Average / Balanced)** | $O(n)$ (must inspect both subtrees) | $O(\log n)$ (eliminates half the tree at each step) |
| **Search Time (Worst-Case Skewed)** | $O(n)$ | $O(n)$ (degenerates into a sorted linked list) |
| **Inorder Traversal Output** | Arbitrary sequence reflecting structural placement | **Strictly sorted** in ascending order |
| **Primary Applications** | Expression trees, Huffman coding trees, Heaps, LCRS representations | Dynamic dictionaries, ordered sets/maps, database indexing |

---

### 7.3 Single Tree vs. Forest

| Property | Single Rooted Tree $T$ | Ordered Forest $\mathcal{F} = (T_1, \dots, T_k)$ |
| :--- | :--- | :--- |
| **Connected Components** | $1$ connected acyclic component (for $n \ge 1$) | $k \ge 0$ disjoint connected acyclic components |
| **Edge-Vertex Formula** | $\|E\| = \|V\| - 1 = n - 1$ | $\|E\| = \|V\| - k = n - k$ |
| **LCRS Binary Tree Root Property** | `bin_root->right == NULL` | `bin_root->right != NULL` (for $k \ge 2$); right spine has $k$ nodes |
| **Effect of Deleting Root(s)** | Deleting root $r$ (of degree $d$) yields a forest of $d$ trees | Deleting root of $T_1$ yields a forest of $d_1 + k - 1$ trees |

---

### 7.4 Operation Time and Space Complexity Summary

**Variable Definitions:**
- $n = |V|$: Total number of nodes in the tree or forest.
- $N$: Maximum branching factor (degree) of an $N$-ary tree.
- $h$: Height of the tree ($O(\log_N n)$ for balanced trees; $O(n)$ for skewed trees).
- $h_B$: Height of the LCRS Binary Tree ($h_B \le n - 1$).
- $k$: Number of disjoint trees in a forest $\mathcal{F}$.

| Operation | Time Complexity | Auxiliary Space Complexity | Notes |
| :--- | :---: | :---: | :--- |
| **Append Child to $N$-Ary Node (`realloc`)** | $O(1)$ amortized | $O(1)$ | Worst-case $O(\deg(u))$ when capacity doubles |
| **Access $i^{\text{th}}$ Child (Dynamic Array)** | $O(1)$ | $O(1)$ | Direct indexing `u->children[i]` |
| **Access $i^{\text{th}}$ Child (LCRS Representation)** | $O(i) = O(N)$ | $O(1)$ | Walks `u->first_child` then $i$ `next_sibling` links |
| **Count Total Nodes / Leaf Nodes** | $O(n)$ | $O(h)$ | Visits every node once; call stack depth $O(h)$ |
| **Compute Tree Height** | $O(n)$ | $O(h)$ | Postorder aggregation over all $n$ nodes |
| **Convert $N$-Ary Tree $\to$ Binary Tree** | $O(n)$ | $O(h)$ | Allocates $n$ binary nodes; links siblings in $O(1)$ per edge |
| **Convert Forest $\mathcal{F} \to$ Binary Tree** | $O(n)$ | $O(h)$ | Chains $k$ converted roots in $O(k) \le O(n)$ |
| **Reconstruct Forest $\leftarrow$ Binary Tree** | $O(n)$ | $O(h_B)$ | Visits each binary node once along left/right spines |
| **Validate BST Invariant (`min`, `max`)** | $O(n)$ | $O(h)$ | Range check at each node |
| **Deallocate Tree (`nary_free` / `bt_free`)** | $O(n)$ | $O(h)$ | Must use **postorder** (children before parent) |

---

## 8. Common Errors, Pitfalls, and Edge Cases

### 8.1 Concrete Bug vs. Fix Examples in C

#### Pitfall 1: Preorder `free()` Causing Use-After-Free or Memory Leaks
Freeing a parent node *before* freeing its children (or forgetting to free the dynamic `children` pointer array in an $N$-ary node) causes undefined behavior or leaks heap memory.

```c
/* BUG: Frees parent struct BEFORE reading its children pointers (Use-After-Free!)
 *      and forgets to free(root->children) array! */
void nary_free_buggy(NAryNode *root) {
    if (root == NULL) return;
    free(root); /* Dangling pointer access on the next line! */
    for (int i = 0; i < root->child_count; i++) {
        nary_free_buggy(root->children[i]);
    }
}

/* FIX: Strictly Postorder — recursively free all children first,
 *      then free the children pointer array, then free the node itself. */
void nary_free_fixed(NAryNode *root) {
    if (root == NULL) return;
    for (int i = 0; i < root->child_count; i++) {
        nary_free_fixed(root->children[i]);
    }
    free(root->children);
    free(root);
}
```

#### Pitfall 2: Returning `0` for `NULL` When Computing 0-Based Edge Height
If `height(NULL)` returns `0`, then a leaf node gets height $\max(0, 0) + 1 = 1$, which silently computes **1-based node height** instead of **0-based edge height**!

```c
/* BUG: Claims to compute 0-based edge height, but returns 0 for NULL,
 *      causing a single root node to report height = 1 instead of 0! */
int bt_height_edges_buggy(const BTNode *root) {
    if (root == NULL) return 0; /* Wrong base case for edge count! */
    int lh = bt_height_edges_buggy(root->left);
    int rh = bt_height_edges_buggy(root->right);
    return (lh > rh ? lh : rh) + 1;
}

/* FIX: Return -1 for NULL so that a leaf node has max(-1, -1) + 1 = 0 edges. */
int bt_height_edges_fixed(const BTNode *root) {
    if (root == NULL) return -1;
    int lh = bt_height_edges_fixed(root->left);
    int rh = bt_height_edges_fixed(root->right);
    return (lh > rh ? lh : rh) + 1;
}
```

#### Pitfall 3: Local-Only Comparison When Validating a Binary Search Tree
Checking only whether `root->left->key < root->key` and `root->right->key > root->key` fails to detect nodes deeper in the left subtree that exceed `root->key`.

```c
/* BUG: Only checks immediate children! Returns true for invalid BSTs where
 *      root=20, root->left=10, root->left->right=25 (25 > 20 in left subtree!). */
bool is_bst_buggy(const BTNode *u) {
    if (u == NULL) return true;
    if (u->left != NULL && u->left->key >= u->key) return false;
    if (u->right != NULL && u->right->key <= u->key) return false;
    return is_bst_buggy(u->left) && is_bst_buggy(u->right);
}

/* FIX: Propagate strict open interval (min_val, max_val) down the tree. */
bool is_bst_fixed(const BTNode *u, long min_val, long max_val) {
    if (u == NULL) return true;
    if (u->key <= min_val || u->key >= max_val) return false;
    return is_bst_fixed(u->left, min_val, u->key) &&
           is_bst_fixed(u->right, u->key, max_val);
}
```

#### Pitfall 4: Mixing 0-Based and 1-Based Array Indexing for Binary Trees
Using `2 * i` for the left child in a 0-based array maps `i = 0` (the root) to `2 * 0 = 0` (itself!), creating an infinite loop.

```c
/* BUG: Uses 1-based formulas (2*i, 2*i+1) on a 0-based array (root at index 0)!
 *      For i = 0, left_child = 2 * 0 = 0 (points back to root!). */
int left_child_buggy(int i)  { return 2 * i; }
int right_child_buggy(int i) { return 2 * i + 1; }
int parent_buggy(int i)      { return i / 2; }

/* FIX: Use 0-based formulas when root is at index 0. */
int left_child_0based(int i)  { return 2 * i + 1; }
int right_child_0based(int i) { return 2 * i + 2; }
int parent_0based(int i)      { return (i - 1) / 2; } /* Valid for i > 0 */
```

#### Pitfall 5: Treating `bin_node->right` as a Child Instead of a Sibling During LCRS Reconstruction
When converting an LCRS Binary Tree node `u` back to an $N$-ary node, `u->right` is **$u$'s sibling**, NOT $u$'s child! Only the right spine starting at `u->left` belongs to $u$'s children.

```c
/* BUG: Mistakenly adds bin_node->right as a child of nary_node! */
NAryNode *reconstruct_buggy(const BTNode *bin_node) {
    if (bin_node == NULL) return NULL;
    NAryNode *u = nary_create_node(bin_node->label);
    if (bin_node->left)  nary_add_child(u, reconstruct_buggy(bin_node->left));
    if (bin_node->right) nary_add_child(u, reconstruct_buggy(bin_node->right)); /* WRONG! */
    return u;
}

/* FIX: Traverse the right spine of bin_node->left to collect all children of u. */
NAryNode *reconstruct_fixed(const BTNode *bin_node) {
    if (bin_node == NULL) return NULL;
    NAryNode *u = nary_create_node(bin_node->label);
    for (const BTNode *c = bin_node->left; c != NULL; c = c->right) {
        nary_add_child(u, reconstruct_fixed(c));
    }
    return u;
}
```

---

### 8.2 Boundary Conditions Checklist

- [ ] **Empty Tree / Empty Forest (`root == NULL`, $n = 0$, $k = 0$):**
  - `count_nodes(NULL)` $\to 0$, `count_leaves(NULL)` $\to 0$, `height_edges(NULL)` $\to -1$, `height_nodes(NULL)` $\to 0$.
- [ ] **Single-Node Tree ($n = 1$, Root is Also a Leaf):**
  - `count_nodes` $\to 1$, `count_leaves` $\to 1$ ($\deg(\text{root}) = 0$), `height_edges` $\to 0$, `height_nodes` $\to 1$.
  - Verify $L = I_2 + 1 \implies 1 = 0 + 1$ holds!
- [ ] **Skewed / Degenerate Tree ($h = n - 1$):**
  - Ensure recursive functions are aware of $O(n)$ stack depth and array representations are not used for sparse skewed trees.
- [ ] **Single Tree vs. Multi-Tree Forest in LCRS:**
  - Verify `bin_root->right == NULL` when $k = 1$, and `bin_root->right != NULL` when $k \ge 2$.
- [ ] **Integer Overflow in BST Range Check:**
  - Pass `LONG_MIN` and `LONG_MAX` (using `long` or `long long` bounds) rather than `INT_MIN` / `INT_MAX` if node keys can equal `INT_MIN` or `INT_MAX`.

---

## 9. Summary and Revision Checklist

### Key Takeaways & Mathematical Invariants
1. **Edge-Vertex Invariants:**
   - Single Tree: $|E| = |V| - 1 = n - 1$.
   - Forest of $k$ Trees: $|E| = |V| - k = n - k$.
2. **Height and Capacity Bounds (0-Based Edge Height $h$, 1-Based Node Height $H = h + 1$):**
   - Max nodes at depth $d$: $N^d$ ($N$-ary) and $2^d$ (binary).
   - Max nodes in tree of height $h$: $\dfrac{N^{h+1} - 1}{N - 1}$ ($N$-ary) and $2^{h+1} - 1 = 2^H - 1$ (binary).
   - Height range for $n$ nodes in a binary tree: $\lfloor \log_2 n \rfloor \le h \le n - 1$.
3. **Degree-Leaf Theorems:**
   - General $N$-ary Tree: $L = 1 + \sum_{i=2}^{N} (i - 1)I_i$.
   - Full $N$-ary Tree ($I$ internal nodes): $L = (N - 1)I + 1$ and $n = NI + 1$.
   - Any Binary Tree: $L = I_2 + 1$, and total `NULL` pointers $= n + 1$.
4. **Knuth / LCRS Transform Rules:**
   - `bin->left` = **First Child** in $N$-ary tree.
   - `bin->right` = **Next Sibling** in $N$-ary tree (or root of the next tree $T_{i+1}$ in a forest).

### Self-Test Revision Checklist
- [ ] Can I prove $|E| = n - 1$ for a tree and $|E| = n - k$ for a forest?
- [ ] Can I derive $L = I_2 + 1$ from $n = L + I_1 + I_2$ and $|E| = I_1 + 2I_2 = n - 1$?
- [ ] Can I draw examples showing why a **Full** binary tree need not be **Complete**, and why a **Complete** binary tree need not be **Full**?
- [ ] Can I convert any $N$-ary tree or forest $\mathcal{F} = \{T_1, T_2, T_3\}$ into its LCRS binary tree by hand and reconstruct the original forest from the binary tree?
- [ ] Can I write C functions for $N$-ary tree creation, height calculation, LCRS conversion, and postorder `free()` without memory leaks?

### Navigation Links
- **Previous Topic:** [03 - Josephus Problem and CPU Scheduling](./03-Josephus-Problem-and-CPU-Scheduling.md)
- **Next Topic:** [05 - Tree Traversals (Preorder, Inorder, Postorder, Level-Order & N-Ary)](./05-Tree-Traversals.md)
- **Unit Review:** [06 - Unit 2 Comprehensive Review & Exam Preparation](./06-Unit-2-Review.md)
