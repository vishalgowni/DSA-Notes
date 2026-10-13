# Binary Search Trees: Array and Dynamic Implementations

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [What Is a Binary Search Tree?](#11-what-is-a-binary-search-tree)
   - 1.2 [Why Do We Need Binary Search Trees?](#12-why-do-we-need-binary-search-trees)
   - 1.3 [Relationship to Prior Concepts](#13-relationship-to-prior-concepts)
   - 1.4 [Learning Objectives](#14-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Formal BST Ordering Invariant](#21-formal-bst-ordering-invariant)
   - 2.2 [Duplicate Key Handling Policies](#22-duplicate-key-handling-policies)
   - 2.3 [Terminology Table](#23-terminology-table)
   - 2.4 [Mathematical Properties and Structural Bounds](#24-mathematical-properties-and-structural-bounds)
   - 2.5 [Balanced vs. Degenerate (Skewed) BST Topologies](#25-balanced-vs-degenerate-skewed-bst-topologies)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Implicit Array-Based Representation (0-Based and 1-Based Indexing)](#31-implicit-array-based-representation-0-based-and-1-based-indexing)
   - 3.2 [Dynamically Allocated Pointer-Based Representation](#32-dynamically-allocated-pointer-based-representation)
   - 3.3 [Memory Overhead and Pathological Sparsity in Array BSTs](#33-memory-overhead-and-pathological-sparsity-in-array-bsts)
4. [Operations and Algorithms](#4-operations-and-algorithms)
   - 4.1 [Implicit Array-Based BST Operations](#41-implicit-array-based-bst-operations)
   - 4.2 [Dynamic BST Searching (Recursive and Iterative)](#42-dynamic-bst-searching-recursive-and-iterative)
   - 4.3 [Dynamic BST Insertion (Recursive and Iterative)](#43-dynamic-bst-insertion-recursive-and-iterative)
   - 4.4 [Minimum, Maximum, Inorder Predecessor, and Inorder Successor](#44-minimum-maximum-inorder-predecessor-and-inorder-successor)
   - 4.5 [Dynamic BST Deletion: All Three Structural Cases](#45-dynamic-bst-deletion-all-three-structural-cases)
   - 4.6 [Traversals, Structural Metrics, Range Queries, and Global BST Validation](#46-traversals-structural-metrics-range-queries-and-global-bst-validation)
   - 4.7 [Complete, Compilable C11 Implementation (Array + Dynamic BST)](#47-complete-compilable-c11-implementation-array--dynamic-bst)
5. [Comparison Tables and Complexity Summary](#5-comparison-tables-and-complexity-summary)
   - 5.1 [Implicit Array-Based BST vs. Dynamic Pointer-Based BST](#51-implicit-array-based-bst-vs-dynamic-pointer-based-bst)
   - 5.2 [Time and Auxiliary Space Complexity Summary](#52-time-and-auxiliary-space-complexity-summary)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Concrete Bug vs. Fix Examples in C](#61-concrete-bug-vs-fix-examples-in-c)
   - 6.2 [Boundary Conditions Checklist](#62-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)
   - 7.1 [Key Takeaways and Core Invariants](#71-key-takeaways-and-core-invariants)
   - 7.2 [Self-Test Revision Checklist](#72-self-test-revision-checklist)
   - 7.3 [Navigation and Related Topics](#73-navigation-and-related-topics)

---

## 1. Introduction and Learning Objectives

### 1.1 What Is a Binary Search Tree?

A **Binary Search Tree (BST)** is a node-based (or implicitly indexed) binary tree data structure in which every node stores a comparable key (and optionally an associated satellite value) subject to a global **symmetric ordering invariant**: for every node $x$, all keys stored in the left subtree of $x$ are strictly less than $\text{key}(x)$, and all keys stored in the right subtree of $x$ are strictly greater than $\text{key}(x)$.

### 1.2 Why Do We Need Binary Search Trees?

In earlier units, we observed a fundamental tension between static arrays and dynamic linked lists when implementing a **Dynamic Ordered Dictionary** (supporting `Search`, `Insert`, `Delete`, `Min`, `Max`, `Predecessor`, and `Successor`):

1. **Sorted Contiguous Array**:
   - Supports $O(\log n)$ binary search via random access (`arr[mid]`).
   - Suffers from $O(n)$ worst-case `Insert` and `Delete` because maintaining sorted order requires shifting up to $n$ elements in contiguous memory.
2. **Doubly Linked List**:
   - Supports $O(1)$ pointer splicing for `Insert` and `Delete` once the target position is known.
   - Suffers from $O(n)$ linear search because a linked list lacks random access to the middle element, preventing binary search.

A **Binary Search Tree** resolves this dilemma by lifting the decision tree of binary search directly into a hierarchical tree topology:
- Every comparison at a node eliminates either its entire left subtree or its entire right subtree.
- In a reasonably balanced BST of height $h = \Theta(\log n)$, **all** dictionary operations—`Search`, `Insert`, `Delete`, `Min`, `Max`, `Predecessor`, and `Successor`—execute in $O(h) = O(\log n)$ time without shifting contiguous memory blocks.

### 1.3 Relationship to Prior Concepts

- **Binary Search on Sorted Arrays**: The root of a balanced BST corresponds to `arr[mid]`, the root of the left subtree corresponds to the midpoint of the left half `arr[low .. mid-1]`, and the root of the right subtree corresponds to the midpoint of the right half `arr[mid+1 .. high]`.
- **General Binary Trees (Unit 2)**: A BST is a structural binary tree (every node has degree $\le 2$ and ordered left/right children) augmented with a key-ordering constraint. Consequently, an **Inorder Traversal** (`Left -> Root -> Right`) of any valid BST visits keys in strictly ascending sorted order.

### 1.4 Learning Objectives

After studying this module, you will be able to:

1. State and prove the **Global BST Ordering Invariant** and explain why a local parent-child check is insufficient to validate a BST.
2. Implement an **Implicit Array-Based BST** using 0-based and 1-based parent/child index arithmetic, and analyze why skewed trees cause $O(2^n)$ exponential memory waste in arrays.
3. Implement a **Dynamically Allocated Pointer-Based BST** in C11 supporting both recursive and iterative `Search` and `Insert`.
4. Derive and trace **Deletion in a BST** across all three structural cases (leaf node, single-child node, and two-child node replaced by its Inorder Successor or Predecessor).
5. Implement **Inorder Predecessor**, **Inorder Successor**, **Pruned Range Queries** (`[k1, k2]`), and **$O(n)$ Global Range-Bound BST Validation**.

---

## 2. Definitions and Terminology

### 2.1 Formal BST Ordering Invariant

Let $T$ be a binary tree in which each node $x \in T$ stores a key denoted $\text{key}(x)$ drawn from a totally ordered universe $(\mathcal{U}, <)$. Let $T_L(x)$ denote the set of all nodes in the left subtree rooted at $x.\text{left}$, and let $T_R(x)$ denote the set of all nodes in the right subtree rooted at $x.\text{right}$.

> **Definition (Strict Binary Search Tree Property):**
> A binary tree $T$ is a **Binary Search Tree** if and only if for **every** node $x \in T$:
> $$\forall y \in T_L(x), \quad \text{key}(y) < \text{key}(x)$$
> $$\forall z \in T_R(x), \quad \text{key}(z) > \text{key}(x)$$

Notice the universal quantifiers $\forall y \in T_L(x)$ and $\forall z \in T_R(x)$. The constraint applies not merely to the immediate left child and immediate right child of $x$, but to **every descendant** in the left and right subtrees of $x$.

```
         VALID BST                                  INVALID BST (Local Check Trap!)
           [50]                                                 [50]
          /    \                                               /    \
       [30]    [70]                                         [30]    [70]
       /  \    /  \                                         /  \    /  \
     [20][40][60][80]                                     [20][65][60][80]
                                                               ^^
   Every node in left subtree of 50                 Node 65 > 30 (locally valid under 30),
   is < 50: {20, 30, 40} < 50.                      BUT 65 is in the LEFT subtree of 50
   Every node in right subtree of 50                and 65 > 50! Violates global invariant!
   is > 50: {60, 70, 80} > 50.
```

### 2.2 Duplicate Key Handling Policies

In a strict mathematical set, keys are unique. When an application requires storing duplicate keys (a *multiset* or *bag*), three standard engineering policies exist:

| Policy Name | Structural Rule | Advantages | Disadvantages |
| :--- | :--- | :--- | :--- |
| **1. Strict Rejection (Set Semantics)** | If $\text{key} == \text{key}(x)$, ignore the insertion or update satellite value. | Preserves strict inequality; simplest deletion and validation logic. | Cannot store multiple occurrences without auxiliary fields. |
| **2. Frequency Counter Augmentation (Recommended for Multisets)** | Add `int count;` to each `BSTNode`. Increment `count` on duplicate insert; decrement on delete (free node only when `count == 0`). | Keeps tree height minimal; $N$ identical keys occupy **1 node** ($h = 0$) instead of a chain of length $N$. | Requires extra `int` field (4 bytes) per node. |
| **3. Non-Strict Inequality Convention** | Enforce $\text{key}(y) \le \text{key}(x) < \text{key}(z)$ (duplicates go left) or $\text{key}(y) < \text{key}(x) \le \text{key}(z)$ (duplicates go right). | No extra struct fields needed. | Inserting $n$ identical keys degrades the tree into a skewed chain of height $h = n - 1$, causing $O(n)$ search time! |

### 2.3 Terminology Table

| Term | Formal Definition | Meaning in a BST |
| :--- | :--- | :--- |
| **Key ($\text{key}(x)$)** | Comparable identifier stored in node $x$. | Determines the unique path from the root to $x$. |
| **Depth of Node $x$** | Number of edges on the unique simple path from `root` to $x$. | Number of key comparisons required to reach $x$ after checking the root ($\text{comparisons} = \text{depth}(x) + 1$). |
| **Height of Tree ($h$)** | Maximum depth among all nodes in the tree ($\text{height}(\text{empty}) = -1$, $\text{height}(\text{single node}) = 0$). | Worst-case number of edge traversals during `Search`, `Insert`, or `Delete`. |
| **Inorder Predecessor** | The node $p$ with the largest key strictly smaller than $\text{key}(x)$: $\max \{y \in T : \text{key}(y) < \text{key}(x)\}$. | Immediately precedes $x$ in an ascending Inorder Traversal. |
| **Inorder Successor** | The node $s$ with the smallest key strictly larger than $\text{key}(x)$: $\min \{z \in T : \text{key}(z) > \text{key}(x)\}$. | Immediately follows $x$ in an ascending Inorder Traversal. |
| **Internal Path Length ($I(T)$)** | Sum of depths of all $n$ nodes: $I(T) = \sum_{x \in T} \text{depth}(x)$. | Determines the average number of comparisons for a successful search: $C_{\text{avg}} = \frac{I(T)}{n} + 1$. |

### 2.4 Mathematical Properties and Structural Bounds

1. **Inorder Monotonicity Theorem**:
   An inorder traversal (`Inorder(left)`, visit `root`, `Inorder(right)`) of an $n$-node BST outputs keys $k_1 < k_2 < \dots < k_n$ in strictly ascending order in $\Theta(n)$ time.
   *Proof Sketch (by structural induction)*: For the base case $n \le 1$, the output is trivially sorted. For the inductive step at root $x$, all elements in $T_L(x)$ are printed before $x$ and are $< \text{key}(x)$ by the BST invariant; all elements in $T_R(x)$ are printed after $x$ and are $> \text{key}(x)$. By the inductive hypothesis, both subtrees are printed in sorted order, so the concatenation $\text{Inorder}(T_L(x)) \mathbin{\Vert} [\text{key}(x)] \mathbin{\Vert} \text{Inorder}(T_R(x))$ is strictly sorted. $\blacksquare$

2. **Height Bounds for an $n$-Node BST ($n \ge 1$)**:
   $$\lfloor \log_2 n \rfloor \le h \le n - 1$$
   - **Best Case ($h = \lfloor \log_2 n \rfloor$)**: Occurs when the BST is a complete or perfectly balanced binary tree.
   - **Worst Case ($h = n - 1$)**: Occurs when keys are inserted in strictly ascending or strictly descending order, forming a degenerate linear chain.
   - **Average Case under Random Permutations**: If $n$ distinct keys are inserted in a uniformly random permutation (all $n!$ orders equally likely), the expected height is $\mathbb{E}[h] = \Theta(\log n)$ and the expected search depth is $2 \ln n \approx 1.386 \log_2 n$.

3. **Number of Structurally Distinct BSTs on $n$ Distinct Keys**:
   Given $n$ distinct keys $k_1 < k_2 < \dots < k_n$, once a root $k_i$ is chosen, the subset $\{k_1, \dots, k_{i-1}\}$ must form the left subtree and $\{k_{i+1}, \dots, k_n\}$ must form the right subtree. Thus, the number of distinct BSTs is given by the $n$-th **Catalan Number** $C_n$:
   $$C_n = \frac{1}{n + 1}\binom{2n}{n} = \frac{(2n)!}{(n + 1)!\,n!}$$
   For $n = 0, 1, 2, 3, 4, 5$, $C_n = 1, 1, 2, 5, 14, 42$.

### 2.5 Balanced vs. Degenerate (Skewed) BST Topologies

Inserting the exact same set of 7 keys $\{10, 20, 30, 40, 50, 60, 70\}$ in two different orders yields drastically different heights:

```
   Insertion Order A: 40, 20, 60, 10, 30, 50, 70        Insertion Order B: 10, 20, 30, 40, 50, 60, 70
          (Perfectly Balanced: h = 2)                         (Right-Skewed Chain: h = 6)

                     [40]                                       [10]
                    /    \                                         \
                 [20]    [60]                                      [20]
                 /  \    /  \                                         \
              [10] [30][50] [70]                                      [30]
                                                                         \
   Max comparisons = h + 1 = 3                                           [40]
   Internal Path Length I(T) = 0 + 2(1) + 4(2) = 10                         \
   Avg comparisons = 10/7 + 1 = 2.43                                        [50]
                                                                               \
                                                                               [60]
                                                                                  \
                                                                                  [70]
                                                        Max comparisons = h + 1 = 7
                                                        Internal Path Length I(T) = 21
                                                        Avg comparisons = 21/7 + 1 = 4.00
```

---

## 3. Representation and Memory Layout in C

A Binary Search Tree can be represented in C using either:
1. **An Implicit Array-Based Representation** (storing nodes at computed level-order indices inside a fixed or dynamically resized contiguous array), or
2. **A Dynamically Allocated Pointer-Based Representation** (allocating individual `struct BSTNode` blocks on the heap connected by explicit `left` and `right` pointers).

### 3.1 Implicit Array-Based Representation (0-Based and 1-Based Indexing)

In an implicit array representation, no pointers are stored. Instead, the position of a node in the level-order numbering of a complete binary tree determines its array index.

#### Index Arithmetic Formulas

| Relationship | **0-Based Indexing** (`root` at index `0`) | **1-Based Indexing** (`root` at index `1`, index `0` unused) |
| :--- | :--- | :--- |
| **Root Index** | $0$ | $1$ |
| **Left Child of $i$** | $\text{left}(i) = 2i + 1$ | $\text{left}(i) = 2i$ |
| **Right Child of $i$** | $\text{right}(i) = 2i + 2$ | $\text{right}(i) = 2i + 1$ |
| **Parent of $i$ ($i > i_{\text{root}}$)** | $\text{parent}(i) = \lfloor (i - 1) / 2 \rfloor$ | $\text{parent}(i) = \lfloor i / 2 \rfloor$ |

#### Empty Slot Tracking (`is_occupied[]` vs. Sentinel `INT_MIN`)

Unlike a binary heap (which is always a complete binary tree where all indices $0 \dots n-1$ are contiguously occupied), an arbitrary BST can have **interior gaps** (unoccupied array slots between occupied slots). Two techniques mark empty slots:
1. **Parallel Boolean Occupancy Array (`bool is_occupied[MAX_SIZE]`)**: Cleanest and safest approach; allows any `int` key (including `INT_MIN`) to be stored validly.
2. **Sentinel Value (`keys[i] == INT_MIN`)**: Saves 1 byte per slot by reserving `INT_MIN` from `<limits.h>` to mean "empty slot", at the cost of forbidding `INT_MIN` as a user key.

```c
#include <stdbool.h>

#define MAX_ARRAY_BST_SIZE 63  /* Supports trees up to height h = 5 (2^6 - 1 = 63 slots) */

typedef struct {
    int  keys[MAX_ARRAY_BST_SIZE];
    bool is_occupied[MAX_ARRAY_BST_SIZE];
    int  node_count;
} ArrayBST;
```

#### ASCII Memory Diagram of an Array-Based BST (0-Based Indexing)

Consider inserting keys `50, 30, 70, 20, 40, 60, 80, 35, 65`:

```
Tree Topology (with 0-based level-order array indices shown in brackets):

                       50 [idx:0]
                     /            \
              30 [idx:1]          70 [idx:2]
               /      \            /      \
         20 [idx:3]  40 [idx:4] 60 [idx:5] 80 [idx:6]
                      /            \
                35 [idx:9]        65 [idx:12]

Contiguous Array Memory Layout (Indices 0..14):
+-------+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+
| Index |  0 |  1 |  2 |  3 |  4 |  5 |  6 |  7 |  8 |  9 | 10 | 11 | 12 | 13 | 14 |
+-------+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+
| keys  | 50 | 30 | 70 | 20 | 40 | 60 | 80 | -- | -- | 35 | -- | -- | 65 | -- | -- |
+-------+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+
| occup |  1 |  1 |  1 |  1 |  1 |  1 |  1 |  0 |  0 |  1 |  0 |  0 |  1 |  0 |  0 |
+-------+----+----+----+----+----+----+----+----+----+----+----+----+----+----+----+
          ^    ^         ^    ^    ^              ^         ^
          |    |_________|    |____|______________|         |
        root   left(1)=3      left(4)=9                 right(5)=12
               right(1)=4
```

### 3.2 Dynamically Allocated Pointer-Based Representation

In a pointer-based BST, each node is allocated independently on the heap via `malloc(sizeof(BSTNode))` and linked through two explicit self-referential pointers:

```c
typedef struct BSTNode {
    int key;                 /* 4 bytes (on LP64 / LLP64 64-bit platforms) */
    /* 4 bytes of compiler struct padding for 8-byte pointer alignment */
    struct BSTNode *left;    /* 8 bytes: pointer to left subtree root (keys < key) */
    struct BSTNode *right;   /* 8 bytes: pointer to right subtree root (keys > key) */
} BSTNode;                   /* sizeof(BSTNode) == 24 bytes on 64-bit systems */
```

#### Heap Box-and-Pointer Memory Layout

```
root (0x5000)
  |
  v
+--------------------------+
|        0x5000            |
|  key   : 50              |
|  left  : 0x5040 ---------+-------------------+
|  right : 0x5080 ---------+-------------------+--------------------+
+--------------------------+                   |                    |
                                               v                    v
                             +--------------------------+  +--------------------------+
                             |        0x5040            |  |        0x5080            |
                             |  key   : 30              |  |  key   : 70              |
                             |  left  : 0x50C0          |  |  left  : 0x5140          |
                             |  right : 0x5100          |  |  right : 0x5180          |
                             +--------------------------+  +--------------------------+
                                /                    \        /                    \
                               v                      v      v                      v
                     [0x50C0: 20|NULL|NULL] [0x5100: 40]   [0x5140: 60]   [0x5180: 80|NULL|NULL]
                                                /                   \
                                               v                     v
                                     [0x51C0: 35|NULL|NULL] [0x5200: 65|NULL|NULL]
```

### 3.3 Memory Overhead and Pathological Sparsity in Array BSTs

Why do production libraries almost universally prefer **Pointer-Based Dynamic BSTs** over **Implicit Array-Based BSTs**?

1. **Exponential Memory Waste on Skewed Trees**:
   Suppose $n$ keys are inserted in ascending order ($10, 20, 30, \dots, 10n$) into a 0-based Array BST. Every node is the right child of its parent:
   $$i_0 = 0, \quad i_1 = 2(0)+2 = 2, \quad i_2 = 2(2)+2 = 6, \quad \dots, \quad i_{k} = 2^{k+1} - 2$$
   A tree with just **$n$ nodes** of height $h = n - 1$ requires an array capacity of:
   $$\text{Capacity} = 2^{h+1} - 1 = 2^n - 1 \text{ slots}$$
   For a mere $n = 30$ sorted keys, an Array BST requires $2^{30} - 1 \approx 1.07 \times 10^9$ slots ($\approx 5.3\text{ GB}$ of RAM) to store **30 integers**, leaving $99.999997\%$ of the array empty! By contrast, a pointer-based BST uses exactly $30 \times 24 = 720\text{ bytes}$.

2. **Expensive Subtree Shifting on Deletion**:
   In a pointer-based BST, promoting a child to replace a deleted single-child parent requires changing **one pointer** ($O(1)$ link update). In an implicit array BST, replacing node $i$ with its child at $2i+1$ requires recursively shifting **every node** in that child's subtree upward to new array indices, costing $O(k)$ movements where $k$ is the subtree size!

---

## 4. Operations and Algorithms

### 4.1 Implicit Array-Based BST Operations

- **Purpose**: Perform `Insert`, `Search`, `Inorder`, and `Delete` using 0-based index arithmetic (`2*i + 1`, `2*i + 2`) inside a fixed-capacity `ArrayBST`.
- **Preconditions**: `tree != NULL` and `array_bst_init(tree)` has initialized all `is_occupied[i] = false`.
- **Step-by-Step Deletion Logic in an Array BST**:
  - **Case 1 (Leaf at `idx`)**: Neither `left_idx(idx)` nor `right_idx(idx)` is occupied. Simply set `tree->is_occupied[idx] = false`.
  - **Case 2 (One Child at `idx`)**: Suppose only `left_idx(idx)` is occupied. Because `left_idx(idx)` and all of its descendants have indices computed relative to `left_idx(idx)`, promoting that child to `idx` requires recursively copying every node in the subtree rooted at `src = 2*idx + 1` to its corresponding position under `dst = idx` via `array_bst_shift_up(tree, src, dst)`.
  - **Case 3 (Two Children at `idx`)**: Find the inorder successor `succ` by starting at `right_idx(idx)` and stepping to `left_idx(succ)` while occupied. Copy `tree->keys[succ]` into `tree->keys[idx]`, then recursively delete `succ` (which is guaranteed to fall into Case 1 or Case 2b).

```
Before Deleting 60 at idx 5 (Case 2b: One Right Child 65 at idx 12):
  idx 5:  60
           \
  idx 12:  65 (right child of 5: 2*5 + 2 = 12)

After array_bst_shift_up(tree, src=12, dst=5):
  idx 5:  65 (shifted up to idx 5; idx 12 marked unoccupied!)
```

### 4.2 Dynamic BST Searching (Recursive and Iterative)

- **Purpose**: Locate a node containing `target_key` in a pointer-based BST, returning its `BSTNode*` address or `NULL` if absent.
- **Preconditions**: `root` points to a valid (possibly empty, `NULL`) BST.
- **Step-by-Step Algorithm**:
  1. Start at `curr = root`.
  2. If `curr == NULL` (search miss) or `curr->key == target_key` (search hit), return `curr`.
  3. If `target_key < curr->key`, by the BST invariant `target_key` cannot exist in `curr->right`; descend into `curr->left`.
  4. Otherwise (`target_key > curr->key`), descend into `curr->right`.

#### Pseudocode

```
ALGORITHM BST-Search-Iterative(root, target_key)
Input:  Pointer root to BST root node, integer target_key
Output: Pointer to matching BSTNode, or NULL if not found

1. curr <- root
2. WHILE curr != NULL AND curr.key != target_key DO
3.     IF target_key < curr.key THEN
4.         curr <- curr.left
5.     ELSE
6.         curr <- curr.right
7. RETURN curr
```

- **Complexity**:
  - **Time**: $O(h)$ worst-case, where $h$ is the tree height ($O(\log n)$ for balanced trees, $O(n)$ for skewed chains).
  - **Space**: Iterative search uses **$O(1)$** auxiliary space; recursive search uses **$O(h)$** call-stack frames (unless tail-call optimized by the compiler).

### 4.3 Dynamic BST Insertion (Recursive and Iterative)

- **Purpose**: Insert a new key `k` into a BST while preserving the global BST ordering invariant.
- **Preconditions**: `root` is a valid BST.
- **Key Insight**: A new key is **always inserted as a new leaf node** at the exact `NULL` child link where an unsuccessful search for `k` terminates!

#### Step-by-Step Algorithm (Iterative with Trailing `parent` Pointer)
1. Allocate `new_node` with `key = k`, `left = NULL`, `right = NULL`.
2. If `root == NULL`, return `new_node` as the new root of the tree.
3. Initialize `curr = root` and `parent = NULL`.
4. While `curr != NULL`:
   - Set `parent = curr`.
   - If `k < curr->key`, advance `curr = curr->left`.
   - Else if `k > curr->key`, advance `curr = curr->right`.
   - Else (`k == curr->key`), free `new_node` and return `root` unchanged (strict duplicate rejection).
5. Attach `new_node` to `parent`: if `k < parent->key`, set `parent->left = new_node`; else set `parent->right = new_node`.
6. Return `root`.

#### Before-and-After ASCII Diagram: Inserting `35` into a BST

```
BEFORE INSERTING 35:                            AFTER INSERTING 35:
Search path: 50 (L) -> 30 (R) -> 40 (L) -> NULL  35 attaches as left child of 40

          [50]                                            [50]
         /    \                                          /    \
      [30]    [70]                                    [30]    [70]
      /  \    /  \                                    /  \    /  \
   [20] [40][60] [80]                              [20] [40][60] [80]
        /                                               /
      NULL  <-- insertion point                      [35]  <-- new leaf node
```

### 4.4 Minimum, Maximum, Inorder Predecessor, and Inorder Successor

#### 1. Minimum and Maximum
By the BST ordering invariant:
- The **Minimum** key is located at the **leftmost node** (follow `curr = curr->left` until `curr->left == NULL`).
- The **Maximum** key is located at the **rightmost node** (follow `curr = curr->right` until `curr->right == NULL`).

#### 2. Inorder Successor of a Node `target` (Without Parent Pointers)
The **Inorder Successor** of `target` is the node with the smallest key strictly greater than `target->key`. There are two mutually exclusive structural cases:

- **Case A (`target->right != NULL`)**:
  `target` has a non-empty right subtree. Every key in `target->right` is greater than `target->key`, so the smallest among them is simply the **minimum (leftmost) node in `target->right`**:
  $$\text{Successor}(target) = \text{BST-Find-Min}(target.\text{right})$$
- **Case B (`target->right == NULL`)**:
  `target` has no right subtree. We must look upward among `target`'s ancestors for the **lowest ancestor `succ` whose left subtree contains `target`**. Without parent pointers, we find this in $O(h)$ time by walking down from `root` to `target`: every time we turn **left** (`target->key < ancestor->key`), that `ancestor` is larger than `target->key` and becomes our newest candidate `succ`.

```
Finding Inorder Successor in Both Cases:

              [50]  <-- Successor of 40 (Case B: lowest ancestor where we turned LEFT)
             /    \
          [30]    [70]
          /  \    /  \
       [20] [40][60] [80]
            /
         [35]  <-- Successor of 30 (Case A: min of 30's right subtree [40] is [35])

  * Successor(30): 30->right is [40] (non-NULL) -> leftmost of [40] is [35].
  * Successor(40): 40->right is NULL -> walk from root [50]:
      - At 50: 40 < 50 (turn LEFT)  => candidate succ = 50
      - At 30: 40 > 30 (turn RIGHT) => succ remains 50
      - At 40: reached target       => return succ = 50.
```

Symmetrically, the **Inorder Predecessor** of `target` is:
- **Case A (`target->left != NULL`)**: $\text{BST-Find-Max}(target.\text{left})$ (the rightmost node in `target`'s left subtree).
- **Case B (`target->left == NULL`)**: The lowest ancestor `pred` from which the search path to `target` turned **right** (`target->key > ancestor->key`).

### 4.5 Dynamic BST Deletion: All Three Structural Cases

Deleting a node `z` with key `k` from a BST is the most intricate operation because removing an interior node must not violate the BST ordering invariant for any remaining nodes. Once `z` is located, we classify it into one of **three structural cases** based on how many children `z` has:

#### Case 1: `z` Is a Leaf Node (`z->left == NULL && z->right == NULL`)
- **Action**: `z` has no subtrees. We simply `free(z)` and return `NULL` to `z`'s parent (updating the parent's `left` or `right` pointer to `NULL`).

```
CASE 1: DELETE LEAF NODE [20]

      BEFORE DELETION                           AFTER DELETION
            [50]                                      [50]
           /    \                                    /    \
        [30]    [70]                              [30]    [70]
        /  \    /  \                              /  \    /  \
     [20] [40][60] [80]                        NULL [40][60] [80]
     ^^^^
   (Leaf: free and set 30->left = NULL)
```

#### Case 2: `z` Has Exactly One Child (`z->left == NULL` XOR `z->right == NULL`)
- **Action**: Let `child` be `z`'s sole non-NULL child (`z->left` or `z->right`). Because every node in the subtree rooted at `child` already satisfies the BST ordering invariant relative to `z`'s parent, we **splice `z` out** by linking `z`'s parent directly to `child`, and then `free(z)`.

```
CASE 2: DELETE SINGLE-CHILD NODE [60] (has only right child [65])

      BEFORE DELETION                           AFTER DELETION
            [50]                                      [50]
           /    \                                    /    \
        [30]    [70]                              [30]    [70]
          \     /  \                                \     /  \
         [40] [60] [80]                            [40] [65] [80]
                 \                                      ^^^^
                [65]                            (65 spliced directly into 70->left;
                                                 node 60 freed)
```

#### Case 3: `z` Has Two Non-NULL Children (`z->left != NULL && z->right != NULL`)
- **Why We Cannot Simply Splice**: `z`'s parent has only one pointer pointing to `z`, so it cannot simultaneously adopt both `z->left` and `z->right`.
- **Which Node Can Safely Replace `z`?**
  If we replace `z`'s key with another key $y$ from $z$'s subtrees, $y$ must satisfy:
  $$\forall a \in T_L(z),\; \text{key}(a) < y \quad \text{AND} \quad \forall b \in T_R(z) \setminus \{y\},\; y < \text{key}(b)$$
  Exactly **two nodes** in $z$'s subtrees satisfy this condition:
  1. **The Inorder Successor $s = \text{BST-Find-Min}(z.\text{right})$**: the smallest key in $z$'s right subtree. Crucially, because $s$ is the leftmost node in $z.\text{right}$, **$s$ is guaranteed to have no left child ($s.\text{left} == \text{NULL}$)**!
  2. **The Inorder Predecessor $p = \text{BST-Find-Max}(z.\text{left})$**: the largest key in $z$'s left subtree, which is guaranteed to have no right child ($p.\text{right} == \text{NULL}$).
- **Action (Using Inorder Successor $s$)**:
  1. Find $s = \text{BST-Find-Min}(z\text{->right})$.
  2. Copy $s\text{->key}$ (and any satellite data) into $z\text{->key}$.
  3. Recursively delete key $s\text{->key}$ from $z\text{->right}$: `z->right = bst_delete(z->right, s->key)`. Because $s\text{->left} == \text{NULL}$, this recursive deletion of $s$ is **guaranteed to terminate in Case 1 (leaf) or Case 2 (single right child)**!

```
CASE 3: DELETE TWO-CHILD NODE [50] (Root) USING INORDER SUCCESSOR

      STEP 1: Locate Successor of 50            STEP 2: Copy 65 into root & delete old 65
      (Leftmost node in right subtree [70])     (Old 65 is a leaf -> deleted via Case 1)

            [50] <-- target z                         [65] <-- z->key overwritten with 65
           /    \                                    /    \
        [30]    [70]                              [30]    [70]
          \     /  \                                \     /  \
         [40] [65] [80]                            [40] NULL [80]
          /   ^^^^                                  /
       [35]  Inorder Successor s                 [35]
             (s->left == NULL!)
```

#### Pseudocode for Recursive BST Deletion

```
ALGORITHM BST-Delete(root, key)
Input:  Pointer root to BST root node, integer key to delete
Output: Pointer to the new root of the modified subtree

1. IF root == NULL THEN
2.     RETURN NULL
3. IF key < root.key THEN
4.     root.left <- BST-Delete(root.left, key)
5. ELSE IF key > root.key THEN
6.     root.right <- BST-Delete(root.right, key)
7. ELSE
8.     // Node to delete found at root
9.     IF root.left == NULL THEN            // Covers Case 1 (leaf) and Case 2b (right child only)
10.        temp <- root.right
11.        FREE(root)
12.        RETURN temp
13.    ELSE IF root.right == NULL THEN      // Covers Case 2a (left child only)
14.        temp <- root.left
15.        FREE(root)
16.        RETURN temp
17.    ELSE                                 // Case 3: Two children
18.        succ <- BST-Find-Min(root.right)
19.        root.key <- succ.key
20.        root.right <- BST-Delete(root.right, succ.key)
21. RETURN root
```

### 4.6 Traversals, Structural Metrics, Range Queries, and Global BST Validation

1. **Pruned Range Query (`[k1, k2]`)**:
   Instead of visiting all $n$ nodes via a full inorder traversal, we exploit the BST ordering invariant to prune entire subtrees:
   - Recurse into `root->left` **only if** `root->key > k1` (if `root->key <= k1`, every node in `root->left` is $< k_1$ and outside the range!).
   - Print `root->key` if $k_1 \le \text{root->key} \le k_2$.
   - Recurse into `root->right` **only if** `root->key < k2` (if `root->key >= k2`, every node in `root->right` is $> k_2$ and outside the range!).
   - **Time Complexity**: $O(h + m)$, where $m$ is the number of keys reported in $[k_1, k_2]$.

2. **Global BST Validation (`bst_is_valid(root)`)**:
   - **Why Local Checking Fails**: Checking only `root->left->key < root->key && root->right->key > root->key` fails when a grandchild violates an ancestor's bound (as shown in Section 2.1 where `65` is the right child of `30` inside `50`'s left subtree).
   - **Correct $O(n)$ Range-Window Algorithm**: Pass an allowable open interval $(\text{min\_val}, \text{max\_val})$ down the recursion, starting with $(-\infty, +\infty)$ (`LLONG_MIN`, `LLONG_MAX` in `long long` so that even `INT_MIN` and `INT_MAX` keys are handled safely without integer overflow):
     - At node `x`, verify $\text{min\_val} < x\text{->key} < \text{max\_val}$.
     - Recurse left with narrowed upper bound $(\text{min\_val}, x\text{->key})$.
     - Recurse right with narrowed lower bound $(x\text{->key}, \text{max\_val})$.

### 4.7 Complete, Compilable C11 Implementation (Array + Dynamic BST)

The following self-contained C11 program implements **both** the Implicit Array-Based BST and the Dynamically Allocated Pointer-Based BST, demonstrating every operation covered above.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <limits.h>

/* =========================================================================
 * PART 1: IMPLICIT ARRAY-BASED BINARY SEARCH TREE (0-BASED INDEXING)
 * ========================================================================= */
#define MAX_ARRAY_BST_SIZE 63  /* Supports up to height 5 (2^6 - 1 = 63 slots) */

typedef struct {
    int keys[MAX_ARRAY_BST_SIZE];
    bool is_occupied[MAX_ARRAY_BST_SIZE];
    int node_count;
} ArrayBST;

void array_bst_init(ArrayBST *tree) {
    for (int i = 0; i < MAX_ARRAY_BST_SIZE; i++) {
        tree->keys[i] = 0;
        tree->is_occupied[i] = false;
    }
    tree->node_count = 0;
}

static inline int left_idx(int i)   { return 2 * i + 1; }
static inline int right_idx(int i)  { return 2 * i + 2; }
static inline int parent_idx(int i) { return (i - 1) / 2; }

bool array_bst_insert(ArrayBST *tree, int key) {
    int idx = 0;
    while (idx < MAX_ARRAY_BST_SIZE && tree->is_occupied[idx]) {
        if (key == tree->keys[idx]) {
            return false; /* Duplicate key rejected */
        } else if (key < tree->keys[idx]) {
            idx = left_idx(idx);
        } else {
            idx = right_idx(idx);
        }
    }
    if (idx >= MAX_ARRAY_BST_SIZE) {
        fprintf(stderr, "[ArrayBST Error] Index %d exceeds capacity %d for key %d!\n",
                idx, MAX_ARRAY_BST_SIZE, key);
        return false;
    }
    tree->keys[idx] = key;
    tree->is_occupied[idx] = true;
    tree->node_count++;
    return true;
}

int array_bst_search(const ArrayBST *tree, int key) {
    int idx = 0;
    while (idx < MAX_ARRAY_BST_SIZE && tree->is_occupied[idx]) {
        if (key == tree->keys[idx]) {
            return idx;
        } else if (key < tree->keys[idx]) {
            idx = left_idx(idx);
        } else {
            idx = right_idx(idx);
        }
    }
    return -1;
}

void array_bst_inorder(const ArrayBST *tree, int idx) {
    if (idx >= MAX_ARRAY_BST_SIZE || !tree->is_occupied[idx]) {
        return;
    }
    array_bst_inorder(tree, left_idx(idx));
    printf("%d(idx:%d) ", tree->keys[idx], idx);
    array_bst_inorder(tree, right_idx(idx));
}

/* Helper to shift a subtree rooted at src_idx up to dst_idx during deletion */
void array_bst_shift_up(ArrayBST *tree, int src_idx, int dst_idx) {
    if (src_idx >= MAX_ARRAY_BST_SIZE || !tree->is_occupied[src_idx]) {
        if (dst_idx < MAX_ARRAY_BST_SIZE) {
            tree->is_occupied[dst_idx] = false;
        }
        return;
    }
    tree->keys[dst_idx] = tree->keys[src_idx];
    tree->is_occupied[dst_idx] = true;
    tree->is_occupied[src_idx] = false;

    array_bst_shift_up(tree, left_idx(src_idx), left_idx(dst_idx));
    array_bst_shift_up(tree, right_idx(src_idx), right_idx(dst_idx));
}

void array_bst_delete_at(ArrayBST *tree, int idx) {
    if (idx < 0 || idx >= MAX_ARRAY_BST_SIZE || !tree->is_occupied[idx]) {
        return;
    }
    int l = left_idx(idx);
    int r = right_idx(idx);
    bool has_left  = (l < MAX_ARRAY_BST_SIZE && tree->is_occupied[l]);
    bool has_right = (r < MAX_ARRAY_BST_SIZE && tree->is_occupied[r]);

    if (!has_left && !has_right) {
        /* Case 1: Leaf node */
        tree->is_occupied[idx] = false;
        tree->node_count--;
    } else if (has_left && !has_right) {
        /* Case 2a: Only left child - shift entire left subtree up */
        array_bst_shift_up(tree, l, idx);
        tree->node_count--;
    } else if (!has_left && has_right) {
        /* Case 2b: Only right child - shift entire right subtree up */
        array_bst_shift_up(tree, r, idx);
        tree->node_count--;
    } else {
        /* Case 3: Two children - replace with inorder successor (min of right) */
        int succ = r;
        while (left_idx(succ) < MAX_ARRAY_BST_SIZE && tree->is_occupied[left_idx(succ)]) {
            succ = left_idx(succ);
        }
        tree->keys[idx] = tree->keys[succ];
        array_bst_delete_at(tree, succ);
    }
}

/* =========================================================================
 * PART 2: DYNAMICALLY ALLOCATED POINTER-BASED BINARY SEARCH TREE
 * ========================================================================= */
typedef struct BSTNode {
    int key;
    struct BSTNode *left;
    struct BSTNode *right;
} BSTNode;

BSTNode *bst_create_node(int key) {
    BSTNode *node = (BSTNode *)malloc(sizeof(BSTNode));
    if (!node) {
        perror("Failed to allocate BSTNode");
        exit(EXIT_FAILURE);
    }
    node->key = key;
    node->left = NULL;
    node->right = NULL;
    return node;
}

/* Recursive Insertion */
BSTNode *bst_insert_recursive(BSTNode *root, int key) {
    if (root == NULL) {
        return bst_create_node(key);
    }
    if (key < root->key) {
        root->left = bst_insert_recursive(root->left, key);
    } else if (key > root->key) {
        root->right = bst_insert_recursive(root->right, key);
    }
    /* Duplicate keys are ignored under strict BST policy */
    return root;
}

/* Iterative Insertion */
BSTNode *bst_insert_iterative(BSTNode *root, int key) {
    BSTNode *new_node = bst_create_node(key);
    if (root == NULL) {
        return new_node;
    }
    BSTNode *curr = root;
    BSTNode *parent = NULL;
    while (curr != NULL) {
        parent = curr;
        if (key < curr->key) {
            curr = curr->left;
        } else if (key > curr->key) {
            curr = curr->right;
        } else {
            free(new_node); /* Duplicate key */
            return root;
        }
    }
    if (key < parent->key) {
        parent->left = new_node;
    } else {
        parent->right = new_node;
    }
    return root;
}

/* Recursive Search */
BSTNode *bst_search_recursive(BSTNode *root, int key) {
    if (root == NULL || root->key == key) {
        return root;
    }
    if (key < root->key) {
        return bst_search_recursive(root->left, key);
    }
    return bst_search_recursive(root->right, key);
}

/* Iterative Search */
BSTNode *bst_search_iterative(BSTNode *root, int key) {
    BSTNode *curr = root;
    while (curr != NULL && curr->key != key) {
        curr = (key < curr->key) ? curr->left : curr->right;
    }
    return curr;
}

/* Minimum and Maximum */
BSTNode *bst_find_min(BSTNode *root) {
    if (root == NULL) return NULL;
    while (root->left != NULL) {
        root = root->left;
    }
    return root;
}

BSTNode *bst_find_max(BSTNode *root) {
    if (root == NULL) return NULL;
    while (root->right != NULL) {
        root = root->right;
    }
    return root;
}

/* Inorder Successor without parent pointers (O(h) top-down search) */
BSTNode *bst_inorder_successor(BSTNode *root, int target_key) {
    BSTNode *target = bst_search_iterative(root, target_key);
    if (target == NULL) return NULL;

    /* Case A: Right subtree is non-empty -> leftmost node in right subtree */
    if (target->right != NULL) {
        return bst_find_min(target->right);
    }

    /* Case B: No right subtree -> lowest ancestor for which target is in left subtree */
    BSTNode *succ = NULL;
    BSTNode *ancestor = root;
    while (ancestor != target) {
        if (target->key < ancestor->key) {
            succ = ancestor;
            ancestor = ancestor->left;
        } else {
            ancestor = ancestor->right;
        }
    }
    return succ;
}

/* Inorder Predecessor without parent pointers (O(h) top-down search) */
BSTNode *bst_inorder_predecessor(BSTNode *root, int target_key) {
    BSTNode *target = bst_search_iterative(root, target_key);
    if (target == NULL) return NULL;

    /* Case A: Left subtree is non-empty -> rightmost node in left subtree */
    if (target->left != NULL) {
        return bst_find_max(target->left);
    }

    /* Case B: No left subtree -> lowest ancestor for which target is in right subtree */
    BSTNode *pred = NULL;
    BSTNode *ancestor = root;
    while (ancestor != target) {
        if (target->key > ancestor->key) {
            pred = ancestor;
            ancestor = ancestor->right;
        } else {
            ancestor = ancestor->left;
        }
    }
    return pred;
}

/* Deletion handling all 3 structural cases */
BSTNode *bst_delete(BSTNode *root, int key) {
    if (root == NULL) {
        return NULL;
    }
    if (key < root->key) {
        root->left = bst_delete(root->left, key);
    } else if (key > root->key) {
        root->right = bst_delete(root->right, key);
    } else {
        /* Node to delete found */
        /* Case 1 (Leaf) & Case 2 (Single Child) */
        if (root->left == NULL) {
            BSTNode *temp = root->right;
            free(root);
            return temp;
        } else if (root->right == NULL) {
            BSTNode *temp = root->left;
            free(root);
            return temp;
        }
        /* Case 3: Two Children -> replace with Inorder Successor */
        BSTNode *succ = bst_find_min(root->right);
        root->key = succ->key;
        root->right = bst_delete(root->right, succ->key);
    }
    return root;
}

/* Traversals */
void bst_inorder(const BSTNode *root) {
    if (root == NULL) return;
    bst_inorder(root->left);
    printf("%d ", root->key);
    bst_inorder(root->right);
}

void bst_preorder(const BSTNode *root) {
    if (root == NULL) return;
    printf("%d ", root->key);
    bst_preorder(root->left);
    bst_preorder(root->right);
}

void bst_postorder(const BSTNode *root) {
    if (root == NULL) return;
    bst_postorder(root->left);
    bst_postorder(root->right);
    printf("%d ", root->key);
}

/* Structural Metrics: Height (in edges, empty tree = -1) and Node Count */
int bst_height(const BSTNode *root) {
    if (root == NULL) return -1;
    int lh = bst_height(root->left);
    int rh = bst_height(root->right);
    return 1 + (lh > rh ? lh : rh);
}

int bst_count_nodes(const BSTNode *root) {
    if (root == NULL) return 0;
    return 1 + bst_count_nodes(root->left) + bst_count_nodes(root->right);
}

/* Pruned Range Query [k1, k2] */
void bst_range_query(const BSTNode *root, int k1, int k2) {
    if (root == NULL) return;
    if (root->key > k1) {
        bst_range_query(root->left, k1, k2);
    }
    if (root->key >= k1 && root->key <= k2) {
        printf("%d ", root->key);
    }
    if (root->key < k2) {
        bst_range_query(root->right, k1, k2);
    }
}

/* Global Range-Bound BST Validation */
bool bst_is_valid_helper(const BSTNode *root, long long min_val, long long max_val) {
    if (root == NULL) return true;
    if (root->key <= min_val || root->key >= max_val) {
        return false;
    }
    return bst_is_valid_helper(root->left, min_val, root->key) &&
           bst_is_valid_helper(root->right, root->key, max_val);
}

bool bst_is_valid(const BSTNode *root) {
    return bst_is_valid_helper(root, LLONG_MIN, LLONG_MAX);
}

void bst_free(BSTNode *root) {
    if (root == NULL) return;
    bst_free(root->left);
    bst_free(root->right);
    free(root);
}

int main(void) {
    printf("=== PART 1: IMPLICIT ARRAY-BASED BST DEMO ===\n");
    ArrayBST arr_tree;
    array_bst_init(&arr_tree);

    int keys[] = {50, 30, 70, 20, 40, 60, 80, 35, 65};
    int n = (int)(sizeof(keys) / sizeof(keys[0]));
    for (int i = 0; i < n; i++) {
        array_bst_insert(&arr_tree, keys[i]);
    }

    printf("Array BST Inorder (key and array index):\n  ");
    array_bst_inorder(&arr_tree, 0);
    printf("\n");

    int target = 65;
    int found_idx = array_bst_search(&arr_tree, target);
    printf("Search %d in Array BST -> Found at index %d (parent index %d = key %d)\n",
           target, found_idx, parent_idx(found_idx), arr_tree.keys[parent_idx(found_idx)]);

    printf("Deleting 30 (two children) and 60 (one child: 65) from Array BST...\n");
    array_bst_delete_at(&arr_tree, array_bst_search(&arr_tree, 30));
    array_bst_delete_at(&arr_tree, array_bst_search(&arr_tree, 60));
    printf("Array BST Inorder after deletions:\n  ");
    array_bst_inorder(&arr_tree, 0);
    printf("\n\n");

    printf("=== PART 2: DYNAMIC POINTER-BASED BST DEMO ===\n");
    BSTNode *root = NULL;
    for (int i = 0; i < n; i++) {
        if (i % 2 == 0)
            root = bst_insert_recursive(root, keys[i]);
        else
            root = bst_insert_iterative(root, keys[i]);
    }

    printf("Inorder   : "); bst_inorder(root);   printf("\n");
    printf("Preorder  : "); bst_preorder(root);  printf("\n");
    printf("Postorder : "); bst_postorder(root); printf("\n");
    printf("Nodes: %d | Height (edges): %d | Valid BST: %s\n",
           bst_count_nodes(root), bst_height(root), bst_is_valid(root) ? "YES" : "NO");
    printf("Min Key: %d | Max Key: %d\n",
           bst_find_min(root)->key, bst_find_max(root)->key);

    int q = 40;
    BSTNode *pred = bst_inorder_predecessor(root, q);
    BSTNode *succ = bst_inorder_successor(root, q);
    printf("Key %d -> Predecessor: %d, Successor: %d\n",
           q, pred ? pred->key : -1, succ ? succ->key : -1);

    printf("Range Query [32, 68]: ");
    bst_range_query(root, 32, 68);
    printf("\n\n");

    printf("--- Testing All 3 Deletion Cases ---\n");
    printf("1. Delete Leaf Node (20):\n   Inorder: ");
    root = bst_delete(root, 20);
    bst_inorder(root);
    printf("\n");

    printf("2. Delete 1-Child Node (60, which has right child 65):\n   Inorder: ");
    root = bst_delete(root, 60);
    bst_inorder(root);
    printf("\n");

    printf("3. Delete 2-Child Root Node (50, replaced by successor 65):\n   Inorder: ");
    root = bst_delete(root, 50);
    bst_inorder(root);
    printf("\n   Preorder after root deletion: ");
    bst_preorder(root);
    printf("\n   Valid BST after deletions: %s\n", bst_is_valid(root) ? "YES" : "NO");

    bst_free(root);
    return 0;
}
```

#### Program Output:

```text
=== PART 1: IMPLICIT ARRAY-BASED BST DEMO ===
Array BST Inorder (key and array index):
  20(idx:3) 30(idx:1) 35(idx:9) 40(idx:4) 50(idx:0) 60(idx:5) 65(idx:12) 70(idx:2) 80(idx:6) 
Search 65 in Array BST -> Found at index 12 (parent index 5 = key 60)
Deleting 30 (two children) and 60 (one child: 65) from Array BST...
Array BST Inorder after deletions:
  20(idx:3) 35(idx:1) 40(idx:4) 50(idx:0) 65(idx:5) 70(idx:2) 80(idx:6) 

=== PART 2: DYNAMIC POINTER-BASED BST DEMO ===
Inorder   : 20 30 35 40 50 60 65 70 80 
Preorder  : 50 30 20 40 35 70 60 65 80 
Postorder : 20 35 40 30 65 60 80 70 50 
Nodes: 9 | Height (edges): 3 | Valid BST: YES
Min Key: 20 | Max Key: 80
Key 40 -> Predecessor: 35, Successor: 50
Range Query [32, 68]: 35 40 50 60 65 

--- Testing All 3 Deletion Cases ---
1. Delete Leaf Node (20):
   Inorder: 30 35 40 50 60 65 70 80 
2. Delete 1-Child Node (60, which has right child 65):
   Inorder: 30 35 40 50 65 70 80 
3. Delete 2-Child Root Node (50, replaced by successor 65):
   Inorder: 30 35 40 65 70 80 
   Preorder after root deletion: 65 30 40 35 70 80 
   Valid BST after deletions: YES
```

---

## 5. Comparison Tables and Complexity Summary

### 5.1 Implicit Array-Based BST vs. Dynamic Pointer-Based BST

| Dimension | Implicit Array-Based BST (`ArrayBST`) | Dynamic Pointer-Based BST (`BSTNode*`) |
| :--- | :--- | :--- |
| **Child/Parent Navigation** | Index arithmetic: $2i+1$, $2i+2$, $\lfloor(i-1)/2\rfloor$ | Explicit pointer dereferencing: `node->left`, `node->right` |
| **Per-Node Pointer Overhead** | $0\text{ bytes}$ for pointers ($+1\text{ byte}$ for `is_occupied[i]`) | $16\text{ bytes}$ for two 64-bit pointers (+ 4 bytes struct alignment padding) |
| **Total Space for $n$ Nodes of Height $h$** | $\Theta(2^{h+1})$ array slots ($\Theta(2^n)$ worst-case on skewed input!) | **Strictly $\Theta(n)$** heap blocks regardless of tree shape |
| **Cache Locality** | High for shallow levels (contiguous array in L1/L2 cache) | Lower (heap allocations may be scattered across pages) |
| **Single-Child Deletion Cost** | $O(k)$ subtree shifting to relocate all $k$ descendants in the array | **$O(1)$** pointer relinking once the target node is reached |
| **Capacity Flexibility** | Fixed upper bound on height ($h_{\max} = \lfloor\log_2(\text{MAX\_SIZE}+1)\rfloor - 1$) | Grows dynamically up to available system heap memory |
| **Primary Ideal Use Case** | Complete or nearly complete static trees (heaps, segment trees) | General-purpose dynamic ordered dictionaries and symbol tables |

### 5.2 Time and Auxiliary Space Complexity Summary

Let $n$ be the number of nodes currently stored in the Dynamic BST, $h$ be the height of the tree ($\lfloor \log_2 n \rfloor \le h \le n - 1$), and $m$ be the number of matching keys in a range query.

| Operation | Average / Balanced Time ($h = \Theta(\log n)$) | Worst-Case Skewed Time ($h = n - 1$) | Auxiliary Space (Iterative) | Auxiliary Space (Recursive) |
| :--- | :---: | :---: | :---: | :---: |
| **`Search(key)`** | $O(\log n)$ | $O(n)$ | $O(1)$ | $O(h)$ |
| **`Insert(key)`** | $O(\log n)$ | $O(n)$ | $O(1)$ | $O(h)$ |
| **`Delete(key)`** | $O(\log n)$ | $O(n)$ | $O(1)$ | $O(h)$ |
| **`FindMin` / `FindMax`** | $O(\log n)$ | $O(n)$ | $O(1)$ | $O(h)$ |
| **`Predecessor` / `Successor`** | $O(\log n)$ | $O(n)$ | $O(1)$ | $O(h)$ |
| **`Inorder` / `Preorder` / `Postorder`** | $\Theta(n)$ | $\Theta(n)$ | $O(h)$ (explicit stack) | $O(h)$ (call stack) |
| **`RangeQuery(k1, k2)`** | $O(\log n + m)$ | $O(n)$ | $O(h)$ | $O(h)$ |
| **`IsValidBST(root)`** | $\Theta(n)$ | $\Theta(n)$ | $O(h)$ | $O(h)$ |

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Concrete Bug vs. Fix Examples in C

#### Pitfall 1: The Local-Only BST Validation Bug
Checking only whether `root->left->key < root->key` and `root->right->key > root->key` fails to detect a node in the left subtree that is larger than an ancestor above its immediate parent.

```c
/* BUG: Only checks immediate children; returns true on invalid tree [50 -> L:30 -> R:65]! */
bool is_bst_buggy(const BSTNode *root) {
    if (root == NULL) return true;
    if (root->left != NULL && root->left->key >= root->key) return false;
    if (root->right != NULL && root->right->key <= root->key) return false;
    return is_bst_buggy(root->left) && is_bst_buggy(root->right);
}

/* FIX: Propagate the strict allowable open range (min_val, max_val) down to every descendant */
bool is_bst_fixed(const BSTNode *root, long long min_val, long long max_val) {
    if (root == NULL) return true;
    if (root->key <= min_val || root->key >= max_val) return false;
    return is_bst_fixed(root->left, min_val, root->key) &&
           is_bst_fixed(root->right, root->key, max_val);
}
```

#### Pitfall 2: Discarding the Return Value of `bst_insert` or `bst_delete`
When inserting into an empty tree (`root == NULL`) or deleting the root node itself, the address of the tree's root changes. Failing to assign `root = bst_insert(root, k)` or `root = bst_delete(root, k)` leaves `root` either `NULL` (leaking the newly allocated root) or dangling (pointing to freed memory!).

```c
/* BUG: Ignoring return value when root is NULL or when root itself is deleted */
BSTNode *root = NULL;
bst_insert_recursive(root, 50); /* Memory leak! root in main() is still NULL! */
bst_delete(root, 50);           /* If root was non-NULL, root is now a dangling pointer! */

/* FIX: Always capture the updated subtree root pointer */
BSTNode *root = NULL;
root = bst_insert_recursive(root, 50);
root = bst_delete(root, 50);
```

#### Pitfall 3: Freeing the Parent Before Children During Tree Deallocation
Freeing `root` in Preorder (`free(root)` before `bst_free(root->left)`) triggers undefined behavior (Use-After-Free) when reading `root->left` and `root->right` from a deallocated heap block.

```c
/* BUG: Preorder free reads root->left and root->right AFTER freeing root! */
void bst_free_buggy(BSTNode *root) {
    if (root == NULL) return;
    free(root);                 /* Undefined Behavior on the next two lines! */
    bst_free_buggy(root->left);
    bst_free_buggy(root->right);
}

/* FIX: Always use Postorder Traversal (Left, Right, Root) when freeing a tree */
void bst_free_fixed(BSTNode *root) {
    if (root == NULL) return;
    bst_free_fixed(root->left);
    bst_free_fixed(root->right);
    free(root);
}
```

#### Pitfall 4: Signed Integer Overflow in Array BST Child Index Calculation
In an `ArrayBST`, if `idx` can become large or if bounds are not checked before dereferencing `tree->is_occupied[idx]`, stepping to `2 * idx + 2` will read/write out of array bounds.

```c
/* BUG: Dereferencing tree->is_occupied[idx] BEFORE checking idx < MAX_ARRAY_BST_SIZE */
while (tree->is_occupied[idx]) {
    idx = (key < tree->keys[idx]) ? (2 * idx + 1) : (2 * idx + 2);
}

/* FIX: Rely on C's short-circuit evaluation (&&) with bounds check FIRST */
while (idx < MAX_ARRAY_BST_SIZE && tree->is_occupied[idx]) {
    idx = (key < tree->keys[idx]) ? (2 * idx + 1) : (2 * idx + 2);
}
```

### 6.2 Boundary Conditions Checklist

- [ ] **Empty Tree (`root == NULL`)**: Do `bst_search`, `bst_find_min`, `bst_find_max`, `bst_delete`, `bst_height`, and `bst_free` handle `NULL` gracefully without dereferencing it?
- [ ] **Single-Node Tree (`root->left == NULL && root->right == NULL`)**: Does deleting the sole root node return `NULL` and properly `free` the node?
- [ ] **Deleting the Root Node with Two Children**: Does `bst_delete(root, root->key)` properly replace the root's key with its inorder successor and update `root->right`?
- [ ] **Extremum Predecessor / Successor Queries**: Does `bst_inorder_predecessor(root, min_key)` return `NULL`? Does `bst_inorder_successor(root, max_key)` return `NULL`?
- [ ] **Extreme Integer Keys (`INT_MIN` and `INT_MAX`)**: Does `bst_is_valid` use `long long` bounds (`LLONG_MIN`, `LLONG_MAX`) so that legitimate keys equal to `INT_MIN` or `INT_MAX` are not falsely rejected?

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways and Core Invariants

1. **Global BST Invariant**: At every node $x$, **all** keys in $T_L(x)$ are $< \text{key}(x)$ and **all** keys in $T_R(x)$ are $> \text{key}(x)$.
2. **Sorted Inorder Property**: An Inorder Traversal (`Left -> Root -> Right`) of a BST visits all $n$ keys in strictly ascending order in $\Theta(n)$ time.
3. **Array vs. Pointer Trade-Off**: Implicit Array BSTs eliminate pointers using $2i+1$ and $2i+2$ indexing, but require $\Theta(2^{h+1})$ space ($\Theta(2^n)$ for skewed trees) and $O(k)$ subtree shifting on deletion. Dynamic Pointer BSTs guarantee $\Theta(n)$ space and $O(1)$ pointer splicing.
4. **Three Deletion Cases**:
   - **Leaf**: `free` node and return `NULL`.
   - **One Child**: `free` node and return its single child pointer to bypass the deleted node.
   - **Two Children**: Copy the **Inorder Successor** ($\min(x.\text{right})$) or **Inorder Predecessor** ($\max(x.\text{left})$) into $x$, and recursively delete that successor/predecessor from its subtree (which reduces to Case 1 or Case 2).
5. **Height Dictates Performance**: All dictionary operations run in $O(h)$ time: $O(\log n)$ when balanced, degrading to $O(n)$ when sorted insertions create a skewed chain.

### 7.2 Self-Test Revision Checklist

- [ ] Can I write the 0-based and 1-based formulas for `left(i)`, `right(i)`, and `parent(i)` from memory?
- [ ] Can I prove why a skewed 20-node BST requires $2^{20} - 1 = 1{,}048{,}575$ slots in an implicit array?
- [ ] Can I trace both cases of `bst_inorder_successor` (when `target->right != NULL` vs. when `target->right == NULL`) without using parent pointers?
- [ ] Can I draw the before-and-after pointer diagrams for deleting a 2-child node using its Inorder Successor, including why the successor never has a left child?
- [ ] Can I write the 8-line `bst_is_valid_helper(root, LLONG_MIN, LLONG_MAX)` function and construct a 4-node counterexample tree that fools a local parent-child check?

### 7.3 Navigation and Related Topics

- **Previous Unit**: [Unit 2 — Trees and Binary Trees](../Unit-2-Queues-and-Trees/04-N-Ary-Binary-Trees-and-Forests.md)
- **Next Topic in Unit 3**: [02 — Expression Trees](./02-Expression-Trees.md)
- **Related Topic in Unit 3**: [03 — Threaded Binary Search Trees](./03-Threaded-Binary-Search-Trees.md)
