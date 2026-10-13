# Threaded Binary Search Trees

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [Motivation: The Wasted `NULL` Pointer Problem](#11-motivation-the-wasted-null-pointer-problem)
   - 1.2 [What Is a Threaded Binary Search Tree?](#12-what-is-a-threaded-binary-search-tree)
   - 1.3 [Relationship to Prior Concepts](#13-relationship-to-prior-concepts)
   - 1.4 [Learning Objectives](#14-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Formal Proof of the $n + 1$ `NULL` Pointers Theorem](#21-formal-proof-of-the-n--1-null-pointers-theorem)
   - 2.2 [Taxonomy of Threaded Binary Trees](#22-taxonomy-of-threaded-binary-trees)
   - 2.3 [Boundary Thread Conventions (`NULL` Bounds vs. Dummy Header Node)](#23-boundary-thread-conventions-null-bounds-vs-dummy-header-node)
   - 2.4 [Terminology Table](#24-terminology-table)
   - 2.5 [Visual Comparison: Unthreaded vs. Right-Threaded vs. Fully Threaded BST](#25-visual-comparison-unthreaded-vs-right-threaded-vs-fully-threaded-bst)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [The `ThreadedNode` Struct and Thread Flag Invariants](#31-the-threadednode-struct-and-thread-flag-invariants)
   - 3.2 [Zero-Space-Overhead Struct Alignment on 64-Bit Architectures](#32-zero-space-overhead-struct-alignment-on-64-bit-architectures)
4. [Operations and Algorithms](#4-operations-and-algorithms)
   - 4.1 [Finding Inorder Successor and Inorder Predecessor Using Threads](#41-finding-inorder-successor-and-inorder-predecessor-using-threads)
   - 4.2 [Stackless Forward and Backward Inorder Traversal ($O(1)$ Space)](#42-stackless-forward-and-backward-inorder-traversal-o1-space)
   - 4.3 [Searching in a Threaded BST](#43-searching-in-a-threaded-bst)
   - 4.4 [Insertion into a Fully Threaded BST](#44-insertion-into-a-fully-threaded-bst)
   - 4.5 [Deletion in a Fully Threaded BST: All Three Structural Cases](#45-deletion-in-a-fully-threaded-bst-all-three-structural-cases)
   - 4.6 [Complete, Compilable C11 Implementation](#46-complete-compilable-c11-implementation)
5. [Comparison Tables and Complexity Summary](#5-comparison-tables-and-complexity-summary)
   - 5.1 [Standard BST vs. Single-Threaded BST vs. Fully Threaded BST](#51-standard-bst-vs-single-threaded-bst-vs-fully-threaded-bst)
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

### 1.1 Motivation: The Wasted `NULL` Pointer Problem

In a standard dynamically allocated Binary Search Tree with $n$ nodes, every node allocates storage for two child pointers (`left` and `right`), giving $2n$ total pointer fields across the tree. However, every non-root node has **exactly one incoming parent pointer**, meaning only $n - 1$ pointers actually link to child nodes. The remaining:
$$2n - (n - 1) = n + 1 \text{ pointers}$$
are set to `NULL`! More than **50% of all pointer fields in a standard binary tree are wasted `NULL` values**.

At the same time, consider what happens when we perform an **Inorder Traversal** (`Left -> Root -> Right`) of a standard BST:
- Because a standard `BSTNode` has no upward parent pointers, once we descend to a leaf node, we have no direct way to climb back up to the ancestor that should be visited next.
- Consequently, both recursive and iterative traversals must maintain an auxiliary call stack or explicit pointer stack of height $O(h)$—which degrades to **$O(n)$ auxiliary memory** on a skewed tree—solely to remember which ancestor to return to!

### 1.2 What Is a Threaded Binary Search Tree?

Introduced by Alan J. Perlis and Charles Thornton in 1960, a **Threaded Binary Search Tree (TBST)** eliminates both the wasted `NULL` pointers and the $O(h)$ traversal stack by repurposing `NULL` child pointers into **threads**:
- If a node's `left` child pointer would normally be `NULL`, it is replaced by a **Left Thread** pointing directly to the node's **Inorder Predecessor**.
- If a node's `right` child pointer would normally be `NULL`, it is replaced by a **Right Thread** pointing directly to the node's **Inorder Successor**.
- Two boolean tag fields (`lthread` and `rthread`) distinguish whether `left` and `right` point to **real downward child subtrees** (`false`) or **upward/lateral inorder threads** (`true`).

With threads in place, **Inorder Traversal becomes completely stackless and recursion-free**, running in $\Theta(n)$ time and strictly **$O(1)$ auxiliary space**, and finding a node's Inorder Successor or Predecessor from its pointer requires no top-down search from the root!

### 1.3 Relationship to Prior Concepts

- **Pointer-Based BSTs ([01 — BST Implementations](./01-BST-Array-and-Dynamic-Implementations.md))**: A Threaded BST obeys the exact same BST key-ordering invariant ($\forall y \in T_L(x), \text{key}(y) < \text{key}(x) < \text{key}(z), \forall z \in T_R(x)$) on all *real* child links.
- **Doubly Linked Lists (Unit 2)**: A Fully Threaded BST superimposes a virtual sorted doubly linked list (via predecessor and successor threads) directly onto the leaves and half-internal nodes of a hierarchical binary search tree.

### 1.4 Learning Objectives

After studying this module, you will be able to:

1. Prove mathematically why any $n$-node binary tree contains exactly $n + 1$ `NULL` child pointers.
2. Distinguish between **Single-Threaded (Right-Threaded / Left-Threaded)** and **Fully (Double / Two-Way) Threaded** BSTs, as well as the **`NULL` Boundary** vs. **Dummy Header Node** conventions.
3. Implement **$O(1)$-Auxiliary-Space Stackless Forward and Backward Inorder Traversals** and prove why a full traversal performs at most $2n - 1$ pointer steps ($\Theta(n)$ time).
4. Derive and trace **Threaded BST Insertion** (showing how a new leaf inherits its parent's old thread and sets its opposite thread to its parent).
5. Derive and implement **Threaded BST Deletion across all three structural cases**, including updating predecessor/successor threads that pointed to the deleted node.

---

## 2. Definitions and Terminology

### 2.1 Formal Proof of the $n + 1$ `NULL` Pointers Theorem

> **Theorem (Null Pointer Count in Binary Trees):**
> Any binary tree $T$ with $n \ge 1$ nodes contains exactly $n + 1$ `NULL` child pointers.

**Proof (by Counting Degrees / Edges):**
1. Each of the $n$ nodes contains exactly $2$ outgoing pointer fields (`left` and `right`), so the total number of pointer fields in $T$ is:
   $$P_{\text{total}} = 2n$$
2. Every node in $T$ **except the root** is pointed to by exactly one parent pointer field (`parent->left` or `parent->right`). The root has $0$ incoming pointers from within the tree. Therefore, the number of non-`NULL` pointer fields (tree edges) is:
   $$P_{\text{non-null}} = n - 1$$
3. Subtracting non-`NULL` pointers from total pointer fields yields the exact number of `NULL` pointers:
   $$P_{\text{null}} = P_{\text{total}} - P_{\text{non-null}} = 2n - (n - 1) = n + 1 \qquad \blacksquare$$

### 2.2 Taxonomy of Threaded Binary Trees

| Threading Variant | `NULL` Left Pointers Replaced By | `NULL` Right Pointers Replaced By | Supported $O(1)$-Space Traversals |
| :--- | :--- | :--- | :--- |
| **1. Right-Threaded (One-Way)** | Remain `NULL` | Thread to **Inorder Successor** (`rthread = true`) | Forward Inorder (`Min -> Max`) |
| **2. Left-Threaded (One-Way)** | Thread to **Inorder Predecessor** (`lthread = true`) | Remain `NULL` | Reverse Inorder (`Max -> Min`) |
| **3. Fully Threaded (Two-Way / Double)** | Thread to **Inorder Predecessor** (`lthread = true`) | Thread to **Inorder Successor** (`rthread = true`) | Both Forward and Reverse Inorder |

### 2.3 Boundary Thread Conventions (`NULL` Bounds vs. Dummy Header Node)

In a Fully Threaded BST of $n$ nodes, $n - 1$ of the $n + 1$ threads point to actual nodes in the tree. What about the **left thread of the global minimum node** (which has no inorder predecessor) and the **right thread of the global maximum node** (which has no inorder successor)? Two conventions are used in practice:

1. **`NULL` Boundary Thread Convention (Used in Our C Implementation)**:
   - The global minimum node has `left = NULL` and `lthread = true`.
   - The global maximum node has `right = NULL` and `rthread = true`.
   - *Advantage*: No extra sentinel node allocation; `root` is a direct `ThreadedNode*` (`NULL` when the tree is empty), matching standard BST function signatures.

2. **Dummy Header (Sentinel) Node Convention**:
   - A permanent dummy header node `header` is allocated before any keys are inserted.
   - `header->left` points to the real `root` of the tree (`header->lthread = false` when non-empty; `header->left = header` and `header->lthread = true` when empty).
   - `header->right` points to `header` itself (`header->rthread = false`).
   - The global minimum node's left thread points to `header`, and the global maximum node's right thread points to `header`.
   - *Advantage*: Eliminates `NULL` checks at the ends of the inorder sequence; traversal stops when `curr == header`.

```
DUMMY HEADER NODE CONVENTION (Circular Threading):

                  +-------------------------------+
                  |      DUMMY HEADER [H]         | <---+
                  | lthread=false | rthread=false |     | (right=H)
                  +-------+---------------+-------+     |
                          | (left)        |_____________|
                          v
                        [50]
                       /    \
         (lthread)  [30]    [70]  (rthread)
            |                  |
            +---> [H]          +---> [H]
      (Global min's left       (Global max's right
       thread points to H)      thread points to H)
```

### 2.4 Terminology Table

| Symbol / Field | Type | Meaning |
| :--- | :--- | :--- |
| **`node->left`** | `ThreadedNode*` | Points to left child if `lthread == false`; points to **Inorder Predecessor** if `lthread == true`. |
| **`node->right`** | `ThreadedNode*` | Points to right child if `rthread == false`; points to **Inorder Successor** if `rthread == true`. |
| **`node->lthread`** | `bool` | `true` $\implies$ `left` is a **thread** (no real left child); `false` $\implies$ `left` is a **real child link**. |
| **`node->rthread`** | `bool` | `true` $\implies$ `right` is a **thread** (no real right child); `false` $\implies$ `right` is a **real child link**. |

### 2.5 Visual Comparison: Unthreaded vs. Right-Threaded vs. Fully Threaded BST

Consider the 5-node BST with keys `{20, 30, 40, 50, 70}` (Inorder sequence: `20, 30, 40, 50, 70`). Solid lines (`/`, `\`) denote **real child links** (`thread = false`); dashed arrows (`..>`) denote **threads** (`thread = true`):

```
1. STANDARD UNTHREADED BST (6 wasted NULL pointers = n + 1):
                 [50]
                /    \
             [30]    [70]
             /  \    /  \
          [20] [40] NULL NULL
          /  \ /  \
        NULL NULL NULL NULL

2. RIGHT-THREADED (SINGLE-THREADED) BST (NULL right pointers become successor threads ..>):
                 [50]
                /    \
             [30]    [70]--rthread--> NULL
             /  \    ^
          [20] [40]  |
            \    \   | (40's rthread points to 50)
             \    +..+ (Wait: 40->right ..> 50, 20->right ..> 30!)
              +..> [30]

3. FULLY (DOUBLE) THREADED BST (Both predecessor <.. and successor ..> threads):

                           [50]
                          /    \
                       [30]    [70] ..rthread..> NULL
                       /  \     ^
                    [20]  [40]  | (70's lthread <.. [50])
                    /  \  /  \  |
     NULL <..lthread  r  l    r.+ (40's rthread ..> [50])
                      |  |
                      v  v
                    [30][30]
   Explicit Thread Table for this 5-node Fully Threaded BST:
   +------+-------------------+-------------------+
   | Node | Left Link / Flag  | Right Link / Flag |
   +------+-------------------+-------------------+
   |  20  | THREAD -> NULL    | THREAD -> 30      |
   |  30  | CHILD  -> 20      | CHILD  -> 40      |
   |  40  | THREAD -> 30      | THREAD -> 50      |
   |  50  | CHILD  -> 30      | CHILD  -> 70      |
   |  70  | THREAD -> 50      | THREAD -> NULL    |
   +------+-------------------+-------------------+
```

---

## 3. Representation and Memory Layout in C

### 3.1 The `ThreadedNode` Struct and Thread Flag Invariants

```c
#include <stdbool.h>

typedef struct ThreadedNode {
    int key;
    struct ThreadedNode *left;
    struct ThreadedNode *right;
    bool lthread; /* true: left is predecessor thread; false: left is real child */
    bool rthread; /* true: right is successor thread;  false: right is real child */
} ThreadedNode;
```

#### Why `lthread` and `rthread` Flags Are Strictly Mandatory
In an unthreaded BST, we test whether a node has a left child by checking `if (node->left != NULL)`. In a Threaded BST, **almost every node has a non-`NULL` `left` and `right` pointer**! If we checked `if (node->left != NULL)` to descend the tree, we would follow a leaf's predecessor thread back up to its ancestor and enter an **infinite loop**!

> **Golden Rule of Threaded BST Programming:**
> - Never test `node->left == NULL` to check for a missing left child; **always test `node->lthread == true`**.
> - Never test `node->right == NULL` to check for a missing right child; **always test `node->rthread == true`**.
> - The only places where `node->left == NULL` or `node->right == NULL` occur in a non-empty tree are the **left boundary thread of the global minimum node** and the **right boundary thread of the global maximum node**.

### 3.2 Zero-Space-Overhead Struct Alignment on 64-Bit Architectures

A common misconception is that adding `bool lthread` and `bool rthread` increases the memory footprint of each tree node. On 64-bit architectures (x86-64 / ARM64), pointers require 8-byte alignment. If we place `lthread` and `rthread` immediately after `int key`, they occupy 2 of the 4 bytes that the compiler would otherwise waste as alignment padding!

```
Standard BSTNode (24 bytes):
+-------------------+-------------------+------------------------+------------------------+
| int key (4 bytes) | PADDING (4 bytes) | BSTNode *left (8 B)    | BSTNode *right (8 B)   |
+-------------------+-------------------+------------------------+------------------------+
Byte offset: 0..3          4..7                   8..15                    16..23

Optimally Ordered ThreadedNode ( STILL 24 bytes! ):
+-------------------+---------+---------+---------+------------------------+------------------------+
| int key (4 bytes) | lthread | rthread | PAD(2B) | ThreadedNode *left(8B) | ThreadedNode *right(8B)|
+-------------------+---------+---------+---------+------------------------+------------------------+
Byte offset: 0..3       4         5        6..7             8..15                    16..23
```

---

## 4. Operations and Algorithms

### 4.1 Finding Inorder Successor and Inorder Predecessor Using Threads

- **Purpose**: Given a pointer `node` to any `ThreadedNode` in a Fully Threaded BST, return its Inorder Successor or Inorder Predecessor **without needing the tree's `root` pointer or a stack**.
- **Step-by-Step Logic for `tbst_inorder_successor(node)`**:
  1. **Case 1 (`node->rthread == true`)**: `node` has no real right child, so `node->right` **is already a thread pointing directly to `node`'s Inorder Successor**! Return `node->right` in **$O(1)$ time**.
  2. **Case 2 (`node->rthread == false`)**: `node` has a real right child (`node->right`). By the BST invariant, `node`'s Inorder Successor is the **leftmost node in `node->right`**. Start at `curr = node->right` and follow real left links (`while (!curr->lthread) curr = curr->left;`), then return `curr`.

```
Tracing tbst_inorder_successor(node):

        [50]  <-- 3. For node [40]: 40->rthread == true, so 40->right jumps DIRECTLY to [50] in O(1)!
       /    \
    [30]    [70]
    /  \    /  \
 [20] [40][60] [80]
      /      \
   [35]     [65]

  * Successor of [40]: 40->rthread is true  ==> return 40->right ([50]) in 1 step!
  * Successor of [30]: 30->rthread is false ==> step to 30->right ([40]),
                       then follow real left links (!lthread) to [35]. Return [35].
```

#### Pseudocode

```
ALGORITHM TBST-Inorder-Successor(node)
Input:  Pointer node in a Threaded BST
Output: Pointer to Inorder Successor of node (or NULL if node is global max)

1. IF node == NULL THEN RETURN NULL
2. IF node.rthread == TRUE THEN
3.     RETURN node.right               // O(1) direct thread jump!
4. curr <- node.right                  // Step into real right child
5. WHILE curr.lthread == FALSE DO      // Follow real left links to minimum
6.     curr <- curr.left
7. RETURN curr
```

Symmetrically, `TBST-Inorder-Predecessor(node)` returns `node->left` immediately if `node->lthread == true`, or else steps to `curr = node->left` and follows real right links (`while (!curr->rthread) curr = curr->right;`).

### 4.2 Stackless Forward and Backward Inorder Traversal ($O(1)$ Space)

- **Purpose**: Visit all $n$ nodes of a Threaded BST in ascending order (Forward Inorder) or descending order (Backward Inorder) using **$O(1)$ auxiliary memory** (zero recursion, zero stack).
- **Step-by-Step Algorithm (Forward Inorder)**:
  1. Start at `curr = tbst_leftmost(root)` (follow `curr = curr->left` while `!curr->lthread`), which is the global minimum node.
  2. While `curr != NULL`:
     - Visit (print) `curr->key`.
     - Advance `curr = tbst_inorder_successor(curr)`.

#### Proof that Stackless Inorder Traversal Runs in $\Theta(n)$ Time (Not $O(n h)$!)
At first glance, `tbst_inorder_successor(curr)` contains a `while (!curr->lthread)` loop that can take $O(h)$ steps, and it is called $n$ times—so a naive bound might guess $O(n h)$. Why is the true worst-case time strictly $\Theta(n)$?
- Every real tree edge (`lthread == false` or `rthread == false`) is traversed downward **at most once** during the entire traversal!
- Every upward jump uses a thread (`rthread == true`) in $O(1)$ time, and at most $n$ threads are followed across the entire traversal.
- Therefore, the total number of pointer dereferences across all $n$ calls to `tbst_inorder_successor` is bounded by:
  $$\text{Total Steps} = (\text{Real Edges Traversed}) + (\text{Threads Followed}) \le (n - 1) + n = 2n - 1 = \Theta(n) \qquad \blacksquare$$

### 4.3 Searching in a Threaded BST

- **Purpose**: Locate a node with a given `key` in $O(h)$ time and $O(1)$ space.
- **Algorithm**: Start at `curr = root`. At each node:
  - If `key == curr->key`, return `curr`.
  - If `key < curr->key`: if `curr->lthread == true`, there is no real left child (search miss $\implies$ return `NULL`); otherwise descend `curr = curr->left`.
  - If `key > curr->key`: if `curr->rthread == true`, there is no real right child (search miss $\implies$ return `NULL`); otherwise descend `curr = curr->right`.

### 4.4 Insertion into a Fully Threaded BST

- **Purpose**: Insert a new key `k` into a Fully Threaded BST while maintaining all real child links and predecessor/successor threads in $O(h)$ time and $O(1)$ space.
- **Step-by-Step Algorithm**:
  1. Search for `k` starting at `curr = root`, keeping a trailing pointer `parent = NULL`, stopping as soon as we hit a thread (`curr->lthread == true` when `k < curr->key`, or `curr->rthread == true` when `k > curr->key`).
  2. Allocate `new_node` with `key = k`, `lthread = true`, `rthread = true`, `left = NULL`, `right = NULL`.
  3. **Subcase 1 (`parent == NULL`)**: The tree was empty; return `new_node` as the new root.
  4. **Subcase 2 (`k < parent->key` — Insert as Left Child of `parent`)**:
     - Before insertion, `parent` had no left child (`parent->lthread == true`), so `parent->left` was holding a thread to `parent`'s inorder predecessor $P$.
     - Because `new_node` is placed as `parent`'s left leaf, in the sorted inorder sequence `new_node` sits **immediately between $P$ and `parent`**:
       $$P \quad \prec \quad \texttt{new\_node} \quad \prec \quad \texttt{parent}$$
     - Therefore:
       1. `new_node` **inherits `parent`'s old predecessor thread**: `new_node->left = parent->left; new_node->lthread = true;`
       2. `new_node`'s **successor thread points to `parent`**: `new_node->right = parent; new_node->rthread = true;`
       3. `parent->left` converts from a thread into a **real child link**: `parent->left = new_node; parent->lthread = false;`

```
INSERTING 35 AS LEFT CHILD OF 40 (Subcase 2):

   BEFORE INSERTION:                              AFTER INSERTION:
   40 has lthread=true pointing to [30],          35 steals 40's old left thread to [30],
   and rthread=true pointing to [50].             35's right thread points to [40],
                                                  and 40->left becomes a REAL child link!

          [50]                                           [50]
         /                                              /
      [30]                                           [30]
        \                                              \
        [40] ..rthread..> [50]                         [40] ..rthread..> [50]
        /                                              /
   lthread                                       lthread=false (REAL CHILD)
      |                                              /
      v                                           [35]
    [30]                                         /    \
                                       lthread..>      <..rthread
                                          |                |
                                          v                v
                                        [30]             [40]
```

  5. **Subcase 3 (`k > parent->key` — Insert as Right Child of `parent`)**:
     - Before insertion, `parent` had no right child (`parent->rthread == true`), so `parent->right` was holding a thread to `parent`'s inorder successor $S$.
     - Because `new_node` is placed as `parent`'s right leaf, in the sorted inorder sequence `new_node` sits **immediately between `parent` and $S$**:
       $$\texttt{parent} \quad \prec \quad \texttt{new\_node} \quad \prec \quad S$$
     - Therefore:
       1. `new_node`'s **predecessor thread points to `parent`**: `new_node->left = parent; new_node->lthread = true;`
       2. `new_node` **inherits `parent`'s old successor thread**: `new_node->right = parent->right; new_node->rthread = true;`
       3. `parent->right` converts from a thread into a **real child link**: `parent->right = new_node; parent->rthread = false;`

```
INSERTING 65 AS RIGHT CHILD OF 60 (Subcase 3):

   BEFORE INSERTION:                              AFTER INSERTION:
   60 has lthread=true pointing to [50],          65's left thread points to [60],
   and rthread=true pointing to [70].             65 steals 60's old right thread to [70],
                                                  and 60->right becomes a REAL child link!

          [70]                                           [70]
         /                                              /
      [60] ..rthread..> [70]                         [60]
      /                                              /  \
   lthread                                     lthread  rthread=false (REAL CHILD)
     |                                           |        \
     v                                           v        [65]
   [50]                                        [50]      /    \
                                               lthread..>      <..rthread
                                                  |                |
                                                  v                v
                                                [60]             [70]
```

### 4.5 Deletion in a Fully Threaded BST: All Three Structural Cases

Deleting a node `target` (with parent `parent`) in a Fully Threaded BST requires not only updating `parent`'s child link, but also **redirecting any thread inside `target`'s subtrees that pointed to `target`** so that no dangling pointers remain!

#### Case 1: Deleting a Leaf Node (`target->lthread == true && target->rthread == true`)
- `target` has no real children, so no node in the tree has a thread pointing to `target`!
  *(Why? Because a thread always points from a descendant upward to an ancestor, and a leaf has no descendants!)*
- **Action**:
  - If `parent == NULL`, the single-node tree becomes empty (`root = NULL`).
  - If `target == parent->left`, `parent` loses its left child and **reclaims `target`'s predecessor thread**:
    `parent->lthread = true; parent->left = target->left;`
  - If `target == parent->right`, `parent` loses its right child and **reclaims `target`'s successor thread**:
    `parent->rthread = true; parent->right = target->right;`
  - `free(target)`.

```
CASE 1: DELETE LEAF NODE [35] (Left Child of [40])

   BEFORE DELETION:                               AFTER DELETION:
   [35] has lthread -> [30], rthread -> [40].     [40] reclaims [35]'s left thread to [30]
   [40] has real child link (lthread=false).      and sets 40->lthread = true!

        [40]                                           [40]
        / (lthread=false)                              / (lthread=true: THREAD!)
     [35]                                             v
     /  \                                           [30]
   ..>  <..
  [30]  [40]
```

#### Case 2: Deleting a Node with One Real Child (`target->lthread != target->rthread`)
Let `child` be `target`'s single real child (`target->left` if `!target->lthread`, else `target->right`).
- **Crucial Thread-Fixup Insight**:
  - Before unlinking `target`, compute `pred = tbst_inorder_predecessor(target)` and `succ = tbst_inorder_successor(target)`.
  - If `target` has a **real left subtree** (`!target->lthread`), then `pred` is the rightmost node inside `target`'s left subtree, and `pred->right` is a thread pointing to `target`! Since `target` had no right subtree (`target->rthread == true`), `target->right` is `target`'s successor thread. After `target` is removed, `pred`'s new inorder successor is `target->right`, so we must update:
    $$\texttt{pred->right = target->right;}$$
  - Symmetrically, if `target` has a **real right subtree** (`!target->rthread`), then `succ` is the leftmost node inside `target`'s right subtree, and `succ->left` is a thread pointing to `target`! After `target` is removed, `succ`'s new inorder predecessor is `target->left`, so we must update:
    $$\texttt{succ->left = target->left;}$$
- Splice `child` into `parent` (`parent->left = child` or `parent->right = child`), and `free(target)`.

```
CASE 2: DELETE 1-CHILD NODE [60] (Left child of [70], has real right child [65])

   BEFORE DELETION:                               AFTER DELETION:
   * 60->left is a thread to [50].                * 70->left spliced to [65].
   * 65 is succ(60), and 65->left is a            * 65->left thread redirected from [60]
     thread pointing back to [60]!                  to 60->left ([50])!

            [70]                                           [70]
            / (lthread=false)                              / (lthread=false)
         [60]                                           [65]
         /  \ (rthread=false)                           /  \
   thread    [65]                                 thread    thread
     |       /  \                                   |         |
     v  thread  thread                              v         v
   [50]   |       |                               [50]      [70]
          v       v
        [60]    [70]
```

#### Case 3: Deleting a Node with Two Real Children (`!target->lthread && !target->rthread`)
- Locate `target`'s Inorder Successor `succ = tbst_leftmost(target->right)` along with its parent `parent_succ`.
- Copy `succ->key` into `target->key`.
- Because `target` itself remains at the same memory address, the predecessor thread from `tbst_rightmost(target->left)` that points to `target` **remains 100% valid without modification**!
- Now delete the physical node `succ` (under `parent_succ`). Because `succ` is the leftmost node in `target->right`, `succ->lthread` is guaranteed to be `true` (no left child), so deleting `succ` reduces directly to **Case 1** (if `succ->rthread == true`) or **Case 2** (if `succ->rthread == false`)!

### 4.6 Complete, Compilable C11 Implementation

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

/* =========================================================================
 * FULLY (DOUBLE / TWO-WAY) THREADED BINARY SEARCH TREE
 * Boundary threads at the global minimum (left) and global maximum (right)
 * are set to NULL with lthread = true / rthread = true.
 * ========================================================================= */
typedef struct ThreadedNode {
    int key;
    struct ThreadedNode *left;
    struct ThreadedNode *right;
    bool lthread; /* true -> left points to Inorder Predecessor (or NULL at min) */
    bool rthread; /* true -> right points to Inorder Successor  (or NULL at max) */
} ThreadedNode;

ThreadedNode *tbst_create_node(int key) {
    ThreadedNode *node = (ThreadedNode *)malloc(sizeof(ThreadedNode));
    if (!node) {
        perror("Failed to allocate ThreadedNode");
        exit(EXIT_FAILURE);
    }
    node->key = key;
    node->left = NULL;
    node->right = NULL;
    node->lthread = true;
    node->rthread = true;
    return node;
}

/* Find leftmost node in a subtree (follows real left links only) */
ThreadedNode *tbst_leftmost(ThreadedNode *node) {
    if (node == NULL) return NULL;
    while (!node->lthread) {
        node = node->left;
    }
    return node;
}

/* Find rightmost node in a subtree (follows real right links only) */
ThreadedNode *tbst_rightmost(ThreadedNode *node) {
    if (node == NULL) return NULL;
    while (!node->rthread) {
        node = node->right;
    }
    return node;
}

/* Inorder Successor using threads: O(1) when rthread is true, O(h) worst-case */
ThreadedNode *tbst_inorder_successor(ThreadedNode *node) {
    if (node == NULL) return NULL;
    if (node->rthread) {
        return node->right; /* Direct thread jump! */
    }
    /* Otherwise, leftmost node in the real right subtree */
    return tbst_leftmost(node->right);
}

/* Inorder Predecessor using threads: O(1) when lthread is true, O(h) worst-case */
ThreadedNode *tbst_inorder_predecessor(ThreadedNode *node) {
    if (node == NULL) return NULL;
    if (node->lthread) {
        return node->left; /* Direct thread jump! */
    }
    /* Otherwise, rightmost node in the real left subtree */
    return tbst_rightmost(node->left);
}

/* Stackless Forward Inorder Traversal: O(n) Time, O(1) Auxiliary Space */
void tbst_inorder_forward(ThreadedNode *root) {
    ThreadedNode *curr = tbst_leftmost(root);
    while (curr != NULL) {
        printf("%d ", curr->key);
        curr = tbst_inorder_successor(curr);
    }
}

/* Stackless Backward Reverse-Inorder Traversal: O(n) Time, O(1) Auxiliary Space */
void tbst_inorder_backward(ThreadedNode *root) {
    ThreadedNode *curr = tbst_rightmost(root);
    while (curr != NULL) {
        printf("%d ", curr->key);
        curr = tbst_inorder_predecessor(curr);
    }
}

/* Detailed Thread Inspection Dump */
void tbst_dump_threads(ThreadedNode *root) {
    ThreadedNode *curr = tbst_leftmost(root);
    while (curr != NULL) {
        printf("  Node %2d | L: %s(%2d) | R: %s(%2d)\n",
               curr->key,
               curr->lthread ? "THREAD" : "CHILD ",
               curr->left ? curr->left->key : -1,
               curr->rthread ? "THREAD" : "CHILD ",
               curr->right ? curr->right->key : -1);
        curr = tbst_inorder_successor(curr);
    }
}

/* Search in a Threaded BST */
ThreadedNode *tbst_search(ThreadedNode *root, int key) {
    ThreadedNode *curr = root;
    while (curr != NULL) {
        if (key == curr->key) {
            return curr;
        } else if (key < curr->key) {
            if (curr->lthread) return NULL; /* Hit thread without finding key */
            curr = curr->left;
        } else {
            if (curr->rthread) return NULL; /* Hit thread without finding key */
            curr = curr->right;
        }
    }
    return NULL;
}

/* Insertion into a Fully Threaded BST */
ThreadedNode *tbst_insert(ThreadedNode *root, int key) {
    ThreadedNode *parent = NULL;
    ThreadedNode *curr = root;

    while (curr != NULL) {
        if (key == curr->key) {
            return root; /* Duplicate key ignored */
        }
        parent = curr;
        if (key < curr->key) {
            if (!curr->lthread) {
                curr = curr->left;
            } else {
                break;
            }
        } else {
            if (!curr->rthread) {
                curr = curr->right;
            } else {
                break;
            }
        }
    }

    ThreadedNode *new_node = tbst_create_node(key);

    if (parent == NULL) {
        /* Empty tree: new_node becomes root with NULL boundary threads */
        return new_node;
    } else if (key < parent->key) {
        /* Insert as LEFT child of parent:
         * 1. new_node inherits parent's old predecessor thread (parent->left)
         * 2. new_node's successor thread points to parent
         * 3. parent->left becomes a real child link (lthread = false)
         */
        new_node->left    = parent->left;
        new_node->lthread = true;
        new_node->right   = parent;
        new_node->rthread = true;

        parent->left      = new_node;
        parent->lthread   = false;
    } else {
        /* Insert as RIGHT child of parent:
         * 1. new_node's predecessor thread points to parent
         * 2. new_node inherits parent's old successor thread (parent->right)
         * 3. parent->right becomes a real child link (rthread = false)
         */
        new_node->left    = parent;
        new_node->lthread = true;
        new_node->right   = parent->right;
        new_node->rthread = true;

        parent->right     = new_node;
        parent->rthread   = false;
    }
    return root;
}

/* Helper for Case 1: Delete a Leaf Node (lthread == true && rthread == true) */
static ThreadedNode *delete_case_leaf(ThreadedNode *root, ThreadedNode *parent, ThreadedNode *target) {
    if (parent == NULL) {
        root = NULL;
    } else if (target == parent->left) {
        /* Target was parent's left child -> parent inherits target's predecessor thread */
        parent->lthread = true;
        parent->left    = target->left;
    } else {
        /* Target was parent's right child -> parent inherits target's successor thread */
        parent->rthread = true;
        parent->right   = target->right;
    }
    free(target);
    return root;
}

/* Helper for Case 2: Delete a Node with Exactly One Real Child */
static ThreadedNode *delete_case_one_child(ThreadedNode *root, ThreadedNode *parent, ThreadedNode *target) {
    ThreadedNode *child = (!target->lthread) ? target->left : target->right;

    /* Find target's inorder predecessor and successor BEFORE unlinking */
    ThreadedNode *pred = tbst_inorder_predecessor(target);
    ThreadedNode *succ = tbst_inorder_successor(target);

    if (parent == NULL) {
        root = child;
    } else if (target == parent->left) {
        parent->left = child;
    } else {
        parent->right = child;
    }

    /* Update any threads inside child's subtree that pointed back to target! */
    if (!target->lthread) {
        /* Target had a LEFT subtree -> its rightmost node (pred) had rthread pointing to target */
        pred->right = target->right; /* Redirect to target's successor thread */
    } else {
        /* Target had a RIGHT subtree -> its leftmost node (succ) had lthread pointing to target */
        succ->left = target->left;   /* Redirect to target's predecessor thread */
    }

    free(target);
    return root;
}

/* Helper for Case 3: Delete a Node with Two Real Children */
static ThreadedNode *delete_case_two_children(ThreadedNode *root, ThreadedNode *target) {
    /* Find inorder successor (leftmost node in target's right subtree) and its parent */
    ThreadedNode *parent_succ = target;
    ThreadedNode *succ = target->right;
    while (!succ->lthread) {
        parent_succ = succ;
        succ = succ->left;
    }

    /* Copy successor's key into target */
    target->key = succ->key;

    /* Successor is guaranteed to have no left child (succ->lthread == true),
     * so deleting succ reduces to Case 1 (leaf) or Case 2 (one right child)! */
    if (succ->lthread && succ->rthread) {
        root = delete_case_leaf(root, parent_succ, succ);
    } else {
        root = delete_case_one_child(root, parent_succ, succ);
    }
    return root;
}

/* General Deletion in a Fully Threaded BST */
ThreadedNode *tbst_delete(ThreadedNode *root, int key) {
    ThreadedNode *parent = NULL;
    ThreadedNode *curr = root;
    bool found = false;

    while (curr != NULL) {
        if (key == curr->key) {
            found = true;
            break;
        }
        parent = curr;
        if (key < curr->key) {
            if (!curr->lthread) curr = curr->left;
            else break;
        } else {
            if (!curr->rthread) curr = curr->right;
            else break;
        }
    }

    if (!found) {
        printf("[Warning] Key %d not found in Threaded BST.\n", key);
        return root;
    }

    if (!curr->lthread && !curr->rthread) {
        root = delete_case_two_children(root, curr);
    } else if (curr->lthread && curr->rthread) {
        root = delete_case_leaf(root, parent, curr);
    } else {
        root = delete_case_one_child(root, parent, curr);
    }
    return root;
}

/* Stackless O(1)-Auxiliary-Space Deallocation using Inorder Successor Threads */
void tbst_free(ThreadedNode *root) {
    ThreadedNode *curr = tbst_leftmost(root);
    while (curr != NULL) {
        ThreadedNode *next = tbst_inorder_successor(curr);
        free(curr);
        curr = next;
    }
}

int main(void) {
    ThreadedNode *root = NULL;
    int keys[] = {50, 30, 70, 20, 40, 60, 80, 35, 65};
    int n = (int)(sizeof(keys) / sizeof(keys[0]));

    for (int i = 0; i < n; i++) {
        root = tbst_insert(root, keys[i]);
    }

    printf("=== FULLY THREADED BST INITIAL STATE ===\n");
    printf("Stackless Forward  Inorder : "); tbst_inorder_forward(root);  printf("\n");
    printf("Stackless Backward Inorder : "); tbst_inorder_backward(root); printf("\n\n");

    printf("Thread & Child Link Table (key = -1 indicates NULL boundary thread):\n");
    tbst_dump_threads(root);

    int target = 40;
    ThreadedNode *found = tbst_search(root, target);
    if (found) {
        ThreadedNode *pred = tbst_inorder_predecessor(found);
        ThreadedNode *succ = tbst_inorder_successor(found);
        printf("\nSearch %d -> Found! Predecessor = %d, Successor = %d\n",
               target, pred ? pred->key : -1, succ ? succ->key : -1);
    }

    printf("\n=== TESTING ALL 3 DELETION CASES IN THREADED BST ===\n");
    printf("1. Delete Leaf Node (35):\n");
    root = tbst_delete(root, 35);
    printf("   Forward  : "); tbst_inorder_forward(root);  printf("\n");
    printf("   Backward : "); tbst_inorder_backward(root); printf("\n");

    printf("2. Delete 1-Child Node (60, which has right child 65):\n");
    root = tbst_delete(root, 60);
    printf("   Forward  : "); tbst_inorder_forward(root);  printf("\n");
    printf("   Backward : "); tbst_inorder_backward(root); printf("\n");

    printf("3. Delete 2-Child Root Node (50, replaced by successor 65):\n");
    root = tbst_delete(root, 50);
    printf("   Forward  : "); tbst_inorder_forward(root);  printf("\n");
    printf("   Backward : "); tbst_inorder_backward(root); printf("\n\n");

    printf("Thread & Child Link Table After Deletions:\n");
    tbst_dump_threads(root);

    tbst_free(root);
    return 0;
}
```

#### Program Output:

```text
=== FULLY THREADED BST INITIAL STATE ===
Stackless Forward  Inorder : 20 30 35 40 50 60 65 70 80 
Stackless Backward Inorder : 80 70 65 60 50 40 35 30 20 

Thread & Child Link Table (key = -1 indicates NULL boundary thread):
  Node 20 | L: THREAD(-1) | R: THREAD(30)
  Node 30 | L: CHILD (20) | R: CHILD (40)
  Node 35 | L: THREAD(30) | R: THREAD(40)
  Node 40 | L: CHILD (35) | R: THREAD(50)
  Node 50 | L: CHILD (30) | R: CHILD (70)
  Node 60 | L: THREAD(50) | R: CHILD (65)
  Node 65 | L: THREAD(60) | R: THREAD(70)
  Node 70 | L: CHILD (60) | R: CHILD (80)
  Node 80 | L: THREAD(70) | R: THREAD(-1)

Search 40 -> Found! Predecessor = 35, Successor = 50

=== TESTING ALL 3 DELETION CASES IN THREADED BST ===
1. Delete Leaf Node (35):
   Forward  : 20 30 40 50 60 65 70 80 
   Backward : 80 70 65 60 50 40 30 20 
2. Delete 1-Child Node (60, which has right child 65):
   Forward  : 20 30 40 50 65 70 80 
   Backward : 80 70 65 50 40 30 20 
3. Delete 2-Child Root Node (50, replaced by successor 65):
   Forward  : 20 30 40 65 70 80 
   Backward : 80 70 65 40 30 20 

Thread & Child Link Table After Deletions:
  Node 20 | L: THREAD(-1) | R: THREAD(30)
  Node 30 | L: CHILD (20) | R: CHILD (40)
  Node 40 | L: THREAD(30) | R: THREAD(65)
  Node 65 | L: CHILD (30) | R: CHILD (70)
  Node 70 | L: THREAD(65) | R: CHILD (80)
  Node 80 | L: THREAD(70) | R: THREAD(-1)
```

---

## 5. Comparison Tables and Complexity Summary

### 5.1 Standard BST vs. Single-Threaded BST vs. Fully Threaded BST

| Feature / Metric | Standard Unthreaded BST | Right-Threaded (Single) BST | Fully Threaded (Double) BST |
| :--- | :--- | :--- | :--- |
| **Number of `NULL` Child Pointers ($n$ nodes)** | $n + 1$ | $\lceil (n+1)/2 \rceil$ to $n$ (left pointers + max right) | **$2$** (only at global min left & global max right; $0$ with Dummy Header) |
| **Extra Tag Flags per Node** | None | `bool rthread` (1 bit / 1 byte) | `bool lthread, rthread` (2 bits / 2 bytes) |
| **64-Bit C Struct Size (`sizeof`)** | $24\text{ bytes}$ | $24\text{ bytes}$ (fits in padding) | $24\text{ bytes}$ (fits in padding) |
| **Forward Inorder Auxiliary Space** | $O(h)$ (stack/recursion) | **$O(1)$** (stackless) | **$O(1)$** (stackless) |
| **Reverse Inorder Auxiliary Space** | $O(h)$ (stack/recursion) | $O(h)$ (no left threads) | **$O(1)$** (stackless) |
| **`InorderSuccessor(node)` from Node Pointer** | $O(h)$ top-down from `root` | $O(1)$ if `rthread`, $O(h)$ worst-case ($O(1)$ amortized) | $O(1)$ if `rthread`, $O(h)$ worst-case ($O(1)$ amortized) |
| **`InorderPredecessor(node)` from Node Pointer** | $O(h)$ top-down from `root` | $O(h)$ top-down from `root` | $O(1)$ if `lthread`, $O(h)$ worst-case ($O(1)$ amortized) |
| **Insertion / Deletion Pointer Complexity** | Simplest | Moderate | Highest (must maintain both threads) |

### 5.2 Time and Auxiliary Space Complexity Summary

Let $n$ be the number of nodes in a Fully Threaded BST and $h$ be its height ($\lfloor \log_2 n \rfloor \le h \le n - 1$).

| Operation | Time Complexity | Auxiliary Space Complexity | Comparison to Standard Recursive BST |
| :--- | :---: | :---: | :--- |
| **`tbst_search(root, key)`** | $O(h)$ | **$O(1)$** | Same time; saves $O(h)$ recursion stack |
| **`tbst_insert(root, key)`** | $O(h)$ | **$O(1)$** | Same time; thread updates take $O(1)$ extra assignments |
| **`tbst_delete(root, key)`** | $O(h)$ | **$O(1)$** | Same time; Case 2 requires $O(h)$ pred/succ lookup |
| **`tbst_inorder_successor(node)`** | $O(1)$ amortized ($O(h)$ worst-case) | **$O(1)$** | Does not require `root` pointer! |
| **`tbst_inorder_predecessor(node)`** | $O(1)$ amortized ($O(h)$ worst-case) | **$O(1)$** | Does not require `root` pointer! |
| **`tbst_inorder_forward(root)`** | $\Theta(n)$ ($\le 2n - 1$ steps) | **$O(1)$** | Eliminates $O(h)$ stack space completely |
| **`tbst_inorder_backward(root)`** | $\Theta(n)$ ($\le 2n - 1$ steps) | **$O(1)$** | Eliminates $O(h)$ stack space completely |
| **`tbst_free(root)`** | $\Theta(n)$ | **$O(1)$** | Frees entire tree in $O(1)$ space without recursion! |

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Concrete Bug vs. Fix Examples in C

#### Pitfall 1: Testing `curr->left != NULL` Instead of `!curr->lthread`
In a Threaded BST, `curr->left` is non-`NULL` for every node except the global minimum! Testing `while (curr->left != NULL)` follows a predecessor thread back up to an ancestor and loops forever.

```c
/* BUG: Infinite loop! Follows predecessor threads back up to ancestors! */
ThreadedNode *tbst_leftmost_buggy(ThreadedNode *node) {
    if (node == NULL) return NULL;
    while (node->left != NULL) { /* WRONG: node->left is non-NULL even on leaves! */
        node = node->left;
    }
    return node;
}

/* FIX: Always check !node->lthread to follow only REAL downward left child links */
ThreadedNode *tbst_leftmost_fixed(ThreadedNode *node) {
    if (node == NULL) return NULL;
    while (!node->lthread) {
        node = node->left;
    }
    return node;
}
```

#### Pitfall 2: Overwriting `parent->left` Before Copying Its Old Thread into `new_node->left`
When inserting `new_node` as the left child of `parent`, `parent->left` currently stores `parent`'s old inorder predecessor thread. If you assign `parent->left = new_node` first and then set `new_node->left = parent->left`, you create a **self-loop (`new_node->left = new_node`)**!

```c
/* BUG: Overwrites parent->left first, causing new_node->left to point to new_node itself! */
parent->left      = new_node;
parent->lthread   = false;
new_node->left    = parent->left; /* BUG: parent->left is already new_node! Self-loop! */
new_node->right   = parent;

/* FIX: Copy parent->left into new_node->left BEFORE modifying parent->left */
new_node->left    = parent->left; /* Inherits parent's old predecessor thread */
new_node->lthread = true;
new_node->right   = parent;
new_node->rthread = true;
parent->left      = new_node;
parent->lthread   = false;
```

#### Pitfall 3: Dangling Thread Pointer After Deleting a Single-Child Node (Case 2)
When deleting a node `target` that has a single real child (say, `target->right`), the leftmost node in `target->right` (`succ`) has a left thread pointing to `target`. If you splice `target->right` into `parent` and `free(target)` without updating `succ->left = target->left`, `succ->left` becomes a **dangling pointer to freed memory**, crashing the next backward traversal or insertion!

```c
/* BUG: Splices child and frees target without redirecting succ->left or pred->right! */
parent->left = target->right;
free(target); /* Dangling pointer: leftmost(target->right)->left still points to target! */

/* FIX: Compute pred and succ BEFORE unlinking, and redirect the inward thread */
ThreadedNode *pred = tbst_inorder_predecessor(target);
ThreadedNode *succ = tbst_inorder_successor(target);
if (target == parent->left) parent->left = child;
else                        parent->right = child;

if (!target->lthread) pred->right = target->right;
else                  succ->left  = target->left;
free(target);
```

### 6.2 Boundary Conditions Checklist

- [ ] **Empty Tree (`root == NULL`)**: Do `tbst_inorder_forward(NULL)`, `tbst_inorder_backward(NULL)`, `tbst_search(NULL, k)`, and `tbst_free(NULL)` return immediately without segmentation faults?
- [ ] **Single-Node Tree**: Does the single root node have `left == NULL`, `right == NULL`, `lthread == true`, and `rthread == true`? Does deleting it return `NULL`?
- [ ] **Global Minimum and Maximum Nodes**: Does `tbst_inorder_predecessor(min_node)` return `NULL`? Does `tbst_inorder_successor(max_node)` return `NULL`?
- [ ] **Deleting the Root Node with 1 Child or 2 Children**: Does `delete_case_one_child` check `if (parent == NULL) root = child;`?
- [ ] **Forward and Backward Symmetry**: After every insertion and deletion, is `tbst_inorder_backward(root)` the exact reversal of `tbst_inorder_forward(root)`? (This verifies that both `lthread` and `rthread` chains are intact!).

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways and Core Invariants

1. **The $n + 1$ Null Pointers Theorem**: An $n$-node binary tree has $2n$ pointer fields and $n - 1$ real child edges, leaving $2n - (n - 1) = n + 1$ `NULL` pointers.
2. **Thread Invariant**:
   - `lthread == true` $\iff$ `node->left` points to `node`'s **Inorder Predecessor** (or `NULL` at the global minimum).
   - `rthread == true` $\iff$ `node->right` points to `node`'s **Inorder Successor** (or `NULL` at the global maximum).
3. **Stackless Traversal**: A Threaded BST supports full Inorder Traversal in **$\Theta(n)$ time** (at most $2n - 1$ pointer transitions) and **$O(1)$ auxiliary space**, without recursion or an explicit stack.
4. **Thread Inheritance on Insertion**:
   - A new **left child** of `parent` inherits `parent->left` as its predecessor thread and sets its successor thread to `parent`.
   - A new **right child** of `parent` sets its predecessor thread to `parent` and inherits `parent->right` as its successor thread.
5. **Thread Redirect on Deletion**:
   - Deleting a leaf restores `parent->lthread = true` (or `rthread = true`) with the leaf's outward thread.
   - Deleting a 1-child node requires redirecting `pred->right = target->right` (if left child exists) or `succ->left = target->left` (if right child exists).

### 7.2 Self-Test Revision Checklist

- [ ] Can I write the 3-line proof that an $n$-node binary tree has $n + 1$ `NULL` pointers?
- [ ] Can I explain why a Threaded BST node requires boolean flags (`lthread`, `rthread`) and why they add zero extra bytes to a 64-bit C struct?
- [ ] Can I trace `tbst_inorder_successor(node)` for both `node->rthread == true` and `node->rthread == false`?
- [ ] Can I write the 6 pointer/flag assignments required to attach `new_node` as the left child or right child of `parent`?
- [ ] Can I explain which thread inside a subtree becomes dangling when a 1-child node is deleted, and how to fix it?

### 7.3 Navigation and Related Topics

- **Previous Topic in Unit 3**: [02 — Expression Trees](./02-Expression-Trees.md)
- **First Topic in Unit 3**: [01 — Binary Search Trees: Array and Dynamic Implementations](./01-BST-Array-and-Dynamic-Implementations.md)
- **Next Topic in Unit 3**: [04 — AVL Trees and Height-Balanced BSTs](./05-AVL-Trees.md)
