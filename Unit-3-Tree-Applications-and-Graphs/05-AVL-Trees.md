# Unit 3 — Topic 5: Height-Balanced Binary Search Trees (AVL Trees)

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [Why Ordinary Binary Search Trees Fail in the Worst Case](#11-why-ordinary-binary-search-trees-fail-in-the-worst-case)
   - 1.2 [What Is an AVL Tree? (Adelson-Velsky and Landis, 1962)](#12-what-is-an-avl-tree-adelson-velsky-and-landis-1962)
   - 1.3 [Relationship to Prior Concepts: Unbalanced BSTs vs. Heaps vs. AVL Trees](#13-relationship-to-prior-concepts-unbalanced-bsts-vs-heaps-vs-avl-trees)
   - 1.4 [Real-World and Systems Applications](#14-real-world-and-systems-applications)
   - 1.5 [Learning Objectives](#15-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Subtree Height Conventions (`height(NULL) = 0` vs. `height(NULL) = -1`)](#21-subtree-height-conventions-heightnull--0-vs-heightnull---1)
   - 2.2 [Formal Definition of Balance Factor ($\text{BF}$) and the AVL Invariant](#22-formal-definition-of-balance-factor-textbf-and-the-avl-invariant)
   - 2.3 [Core Terminology Table](#23-core-terminology-table)
   - 2.4 [Mathematical Proof of Worst-Case AVL Height ($h < 1.44 \log_2(n+2)$)](#24-mathematical-proof-of-worst-case-avl-height-h--144-log_2n2)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [`AVLNode` Structure Declaration](#31-avlnode-structure-declaration)
   - 3.2 [Why Storing `height` Is Superior to Recomputing Height on the Fly](#32-why-storing-height-is-superior-to-recomputing-height-on-the-fly)
   - 3.3 [Memory Layout Diagram of an AVL Tree Node](#33-memory-layout-diagram-of-an-avl-tree-node)
4. [The Four Imbalance Cases and Tree Rotations](#4-the-four-imbalance-cases-and-tree-rotations)
   - 4.1 [What Is a Tree Rotation? Preserving the Inorder BST Invariant in $O(1)$ Time](#41-what-is-a-tree-rotation-preserving-the-inorder-bst-invariant-in-o1-time)
   - 4.2 [Case 1: Left-Left (LL) Imbalance $\implies$ Single Right Rotation (`rotate_right(z)`)](#42-case-1-left-left-ll-imbalance-implies-single-right-rotation-rotate_rightz)
   - 4.3 [Case 2: Right-Right (RR) Imbalance $\implies$ Single Left Rotation (`rotate_left(z)`)](#43-case-2-right-right-rr-imbalance-implies-single-left-rotation-rotate_leftz)
   - 4.4 [Case 3: Left-Right (LR) Imbalance $\implies$ Double Rotation (`rotate_left(y)`, then `rotate_right(z)`)](#44-case-3-left-right-lr-imbalance-implies-double-rotation-rotate_lefty-then-rotate_rightz)
   - 4.5 [Case 4: Right-Left (RL) Imbalance $\implies$ Double Rotation (`rotate_right(y)`, then `rotate_left(z)`)](#45-case-4-right-left-rl-imbalance-implies-double-rotation-rotate_righty-then-rotate_leftz)
   - 4.6 [Decision Table for All 4 Imbalance Cases](#46-decision-table-for-all-4-imbalance-cases)
5. [Operations and Algorithms](#5-operations-and-algorithms)
   - 5.1 [AVL Insertion Algorithm ($O(\log n)$ Time, At Most 1 Rotation)](#51-avl-insertion-algorithm-olog-n-time-at-most-1-rotation)
   - 5.2 [Complete Step-by-Step Insertion Trace Triggering LL, RR, LR, and RL](#52-complete-step-by-step-insertion-trace-triggering-ll-rr-lr-and-rl)
   - 5.3 [AVL Deletion Algorithm ($O(\log n)$ Time, Up to $O(\log n)$ Rotations)](#53-avl-deletion-algorithm-olog-n-time-up-to-olog-n-rotations)
   - 5.4 [Complete C11 Implementation: AVL Insertion, Deletion, Search, and Validation](#54-complete-c11-implementation-avl-insertion-deletion-search-and-validation)
6. [Comparison Tables and Complexity Summary](#6-comparison-tables-and-complexity-summary)
   - 6.1 [Time and Space Complexity Summary](#61-time-and-space-complexity-summary)
   - 6.2 [Unbalanced BST vs. AVL Tree vs. Red-Black Tree vs. Binary Heap](#62-unbalanced-bst-vs-avl-tree-vs-red-black-tree-vs-binary-heap)
7. [Common Errors, Pitfalls, and Edge Cases](#7-common-errors-pitfalls-and-edge-cases)
   - 7.1 [Pitfall 1: Updating `y->height` Before `z->height` After a Rotation](#71-pitfall-1-updating-y-height-before-z-height-after-a-rotation)
   - 7.2 [Pitfall 2: Using `< 0` Instead of `<= 0` for Child Balance Factor During Deletion](#72-pitfall-2-using--0-instead-of--0-for-child-balance-factor-during-deletion)
   - 7.3 [Pitfall 3: Discarding the New Subtree Root Returned by Rotation](#73-pitfall-3-discarding-the-new-subtree-root-returned-by-rotation)
   - 7.4 [Pitfall 4: Calling `height(node)` Recursively in $O(n)$ Instead of Reading `node->height` in $O(1)$](#74-pitfall-4-calling-heightnode-recursively-in-on-instead-of-reading-node-height-in-o1)
   - 7.5 [Boundary Conditions Checklist](#75-boundary-conditions-checklist)
8. [Summary and Revision Checklist](#8-summary-and-revision-checklist)
   - 8.1 [Key Takeaways and Core Invariants](#81-key-takeaways-and-core-invariants)
   - 8.2 [Self-Test Revision Questions](#82-self-test-revision-questions)
   - 8.3 [Navigation Links](#83-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 Why Ordinary Binary Search Trees Fail in the Worst Case

In Unit 2, we saw that every primary operation on a Binary Search Tree (`search`, `insert`, `delete`, `min`, `max`, `successor`) runs in $O(h)$ time, where $h$ is the height of the tree. When keys are inserted in uniformly random order, the expected height is $O(\log n)$. However, when keys arrive in **sorted**, **reverse-sorted**, or **alternating extreme** order—a routine occurrence in timestamped logs, auto-incrementing database IDs, and sequential file dumps—an ordinary BST degenerates into a **skewed linear chain** of height $h = n - 1$:

```
   DEGENERATE SKEWED BST (Inserted: 10, 20, 30, 40, 50)      HEIGHT-BALANCED AVL TREE (Same Keys!)
                Height h = n - 1 = 4                                  Height h = 2
                Worst-Case Search = O(n)                              Worst-Case Search = O(log n)

                  10 (BF=-4)                                              20 (BF=-1)
                    \                                                    /  \
                     20 (BF=-3)                                  (BF=0) 10   40 (BF=0)
                       \                                                    /  \
                        30 (BF=-2)                                  (BF=0) 30  50 (BF=0)
                          \
                           40 (BF=-1)
                             \
                              50 (BF=0)
```

In a degenerate BST of $n = 1{,}000{,}000$ sorted keys, searching for the last key requires **$1{,}000{,}000$ comparisons**. In an AVL tree holding the exact same $1{,}000{,}000$ keys, the height is strictly bounded below $1.44 \log_2(10^6 + 2) \approx 28$, so searching requires **at most 28 comparisons**!

### 1.2 What Is an AVL Tree? (Adelson-Velsky and Landis, 1962)

Named after Soviet mathematicians **Georgy Adelson-Velsky** and **Evgenii Landis** (who published it in 1962), the **AVL Tree** is the oldest **self-balancing Binary Search Tree** in computer science.

An AVL tree is a Binary Search Tree that enforces an additional local **Height-Balance Invariant** at **every** node $v$ in the tree: the heights of the left and right subtrees of $v$ may differ by **at most 1**. Whenever an insertion or deletion causes any node's left and right subtree heights to differ by $2$, the tree immediately repairs the local imbalance using $O(1)$ pointer rearrangements called **Tree Rotations**.

### 1.3 Relationship to Prior Concepts: Unbalanced BSTs vs. Heaps vs. AVL Trees

- **Unbalanced BST vs. AVL Tree:** Every AVL tree is a valid BST and supports exact searches, range queries, and sorted inorder traversal—capabilities a Binary Heap lacks—while eliminating the $O(n)$ worst-case pathology of an unbalanced BST.
- **Binary Heap vs. AVL Tree:** A Binary Heap maintains perfect balance ($h = \lfloor \log_2 n \rfloor$) in an array by sacrificing horizontal ordering (only the max/min is accessible in $O(1)$). An AVL tree maintains **full total ordering** of all keys (`left < root < right`) in a pointer-based tree while keeping $h < 1.44 \log_2(n + 2)$ via local rotations.

### 1.4 Real-World and Systems Applications

1. **Read-Intensive In-Memory Dictionaries & Databases:** Because AVL trees are more strictly balanced ($h \le 1.44 \log_2 n$) than Red-Black trees ($h \le 2 \log_2 n$), lookup-intensive workloads (such as symbol tables, routing caches, and geometry sweep-line status structures) achieve faster worst-case lookups in AVL trees.
2. **Operating System Virtual Memory Managers:** Windows NT/10/11 kernel **Virtual Address Descriptors (`MM_AVL_TABLE`)** use AVL trees to manage non-overlapping virtual memory regions per process for rapid page-fault address lookup.
3. **Real-Time Embedded & Safety-Critical Systems:** Avionics and real-time control loops require strict **worst-case** execution time ($O(\log n)$ guaranteed per single operation) rather than merely amortized bounds.

### 1.5 Learning Objectives

After completing this chapter, you will be able to:
- Define the **Balance Factor ($\text{BF}$)** of any node and verify whether a given BST is a valid AVL tree.
- Prove mathematically using Fibonacci recurrence relations why an AVL tree of $n$ nodes has height $h < 1.44 \log_2(n + 2) = O(\log n)$.
- Identify and execute all four AVL imbalance cases (**LL**, **RR**, **LR**, and **RL**) using single and double rotations.
- Trace and implement AVL tree **Insertion** and **Deletion** in C11, explaining why insertion needs at most 1 rotation whereas deletion may need $O(\log n)$ rotations.

---

## 2. Definitions and Terminology

### 2.1 Subtree Height Conventions (`height(NULL) = 0` vs. `height(NULL) = -1`)

In textbooks and C implementations, two equivalent conventions are used for the height of a node $v$. **Both conventions produce the exact same Balance Factor $\text{BF}(v)$ for every node!**

| Node Type | **Convention A: Node-Count Height** *(Standard in C Code)* | **Convention B: Edge-Count Height** *(Standard in Mathematical Proofs)* |
| :--- | :--- | :--- |
| **Empty Subtree (`NULL`)** | $\text{height}(\text{NULL}) = 0$ | $\text{height}(\text{NULL}) = -1$ |
| **Leaf Node ($L = \text{NULL}, R = \text{NULL}$)** | $\text{height}(\text{leaf}) = 1 + \max(0, 0) = 1$ | $\text{height}(\text{leaf}) = 1 + \max(-1, -1) = 0$ |
| **General Node $v$** | $\text{height}(v) = 1 + \max(\text{height}(v\to\text{left}), \text{height}(v\to\text{right}))$ | $\text{height}(v) = 1 + \max(\text{height}(v\to\text{left}), \text{height}(v\to\text{right}))$ |
| **Balance Factor $\text{BF}(v)$** | $\text{height}(v\to\text{left}) - \text{height}(v\to\text{right})$ | $\text{height}(v\to\text{left}) - \text{height}(v\to\text{right})$ *(Identical!)* |

In our C11 implementation, we use **Convention A (`height(NULL) = 0`, leaf height `= 1`)** because `0` is the natural sentinel for an empty pointer and avoids `-1` arithmetic.

### 2.2 Formal Definition of Balance Factor ($\text{BF}$) and the AVL Invariant

For any node $v$ in a binary tree, its **Balance Factor**, denoted $\text{BF}(v)$, is the height of its left subtree minus the height of its right subtree:

$$\text{BF}(v) = \text{height}(v\to\text{left}) - \text{height}(v\to\text{right})$$

> [!IMPORTANT]
> **The AVL Balance Invariant:** A Binary Search Tree $T$ is an **AVL Tree** if and only if every node $v \in T$ satisfies:
> $$\text{BF}(v) \in \{-1, \, 0, \, +1\} \quad \Longleftrightarrow \quad |\text{height}(v\to\text{left}) - \text{height}(v\to\text{right})| \le 1$$

- **$\text{BF}(v) = +1$:** Node $v$ is **left-heavy** (its left subtree is 1 level taller than its right subtree).
- **$\text{BF}(v) = 0$:** Node $v$ is **perfectly balanced** (both subtrees have equal height).
- **$\text{BF}(v) = -1$:** Node $v$ is **right-heavy** (its right subtree is 1 level taller than its left subtree).
- **$|\text{BF}(v)| = 2$ ($\text{BF}(v) \in \{+2, -2\}$):** Node $v$ is **unbalanced** and violates the AVL property, requiring an immediate rotation.

### 2.3 Core Terminology Table

| Term | Formal Definition |
| :--- | :--- |
| **Balance Factor ($\text{BF}(v)$)** | $\text{height}(v\to\text{left}) - \text{height}(v\to\text{right})$. Must remain in $\{-1, 0, +1\}$. |
| **Critical Node ($z$)** | The **lowest (deepest) ancestor** of the newly inserted or deleted node whose Balance Factor becomes $+2$ or $-2$. |
| **Single Rotation** | A single $O(1)$ pointer rotation (`rotate_right` for **LL**, `rotate_left` for **RR**) applied when the imbalance path from $z$ through $y$ to $x$ is straight (collinear). |
| **Double Rotation** | Two consecutive $O(1)$ single rotations (`rotate_left(y)` then `rotate_right(z)` for **LR**; `rotate_right(y)` then `rotate_left(z)` for **RL**) applied when the imbalance path bends (zig-zag). |
| **Fibonacci AVL Tree ($T_h$)** | An AVL tree of height $h$ containing the **minimum possible number of nodes** $N(h)$, where every internal node has balance factor $+1$ (or $-1$). |

### 2.4 Mathematical Proof of Worst-Case AVL Height ($h < 1.44 \log_2(n+2)$)

To find the **maximum possible height $h$** of an AVL tree with $n$ nodes, we solve the dual problem: *What is the **minimum number of nodes $N(h)$** required to construct a valid AVL tree of height $h$ (using edge-count height where a single root leaf has $h = 0$)?*

1. **Base Cases:**
   - For $h = 0$ (a single node): $N(0) = 1$.
   - For $h = 1$ (a root and one child): $N(1) = 2$.
2. **Recurrence Relation for Minimal AVL Trees ($T_h$):**
   For an AVL tree to have height $h$ with as few nodes as possible, its root must have one subtree of height $h - 1$ and the other subtree of height $h - 2$ (so $|\text{BF}(\text{root})| = 1$), and both subtrees must themselves be minimal AVL trees:
   $$N(h) = 1 + N(h - 1) + N(h - 2) \quad \text{for } h \ge 2$$

```
        MINIMAL AVL TREE OF HEIGHT h (Fibonacci AVL Tree T_h)
        N(0) = 1,  N(1) = 2,  N(2) = 4,  N(3) = 7,  N(4) = 12

           h=0 (N=1)       h=1 (N=2)         h=2 (N=4)            h=3 (N=7)
              (o)             (o)               (o)                  (o)
                              /                 / \                 /   \
                            (o)               (o) (o)             (o)   (o)
                                              /                   / \   /
                                            (o)                 (o) (o)(o)
                                                                /
                                                              (o)
```

3. **Connection to Fibonacci Numbers ($F_0 = 0, F_1 = 1, F_2 = 1, F_3 = 2, F_4 = 3, F_5 = 5, \dots$):**
   Add $1$ to both sides of the recurrence:
   $$[N(h) + 1] = [N(h - 1) + 1] + [N(h - 2) + 1]$$
   Since $N(0) + 1 = 2 = F_3$ and $N(1) + 1 = 3 = F_4$, induction immediately yields:
   $$N(h) = F_{h+3} - 1$$
4. **Solving for $h$ in Terms of $n$:**
   By Binet's formula, $F_k > \frac{\varphi^k}{\sqrt{5}} - \frac{1}{2}$ where $\varphi = \frac{1 + \sqrt{5}}{2} \approx 1.61803$ is the Golden Ratio. Since an $n$-node AVL tree of height $h$ must satisfy $n \ge N(h)$, we have:
   $$n + 1 \ge F_{h+3} \approx \frac{\varphi^{h+3}}{\sqrt{5}} \implies \varphi^{h+3} \le \sqrt{5}(n + 2)$$
   Taking $\log_2$ on both sides and using $\frac{1}{\log_2 \varphi} \approx 1.4404$:
   $$h < 1.4404 \log_2(n + 2) - 1.328 = O(\log n)$$
   Therefore, an AVL tree's height is **never more than 44% taller than a perfectly full binary tree**, guaranteeing worst-case $O(\log n)$ time for all operations! $\blacksquare$

---

## 3. Representation and Memory Layout in C

### 3.1 `AVLNode` Structure Declaration

Each node in an AVL tree stores its `key`, its cached subtree `height` (an `int`), and pointers to its `left` and `right` children:

```c
typedef struct AVLNode {
    int             key;    /* Search key                                    */
    int             height; /* Cached subtree height (leaf = 1, NULL = 0)    */
    struct AVLNode *left;   /* Pointer to left subtree  (all keys < key)     */
    struct AVLNode *right;  /* Pointer to right subtree (all keys > key)     */
} AVLNode;
```

### 3.2 Why Storing `height` Is Superior to Recomputing Height on the Fly

If we did **not** store `int height` inside `AVLNode` and instead computed `height(node)` recursively whenever we needed $\text{BF}(node)$, each balance factor check would take $O(\text{subtree size})$ time, degrading insertion from $O(\log n)$ to $O(n)$! By caching `node->height` inside every struct:
- Reading `avl_height(node)` takes **$O(1)$ time** (`node ? node->height : 0`).
- Computing `avl_balance_factor(node)` takes **$O(1)$ time** (`avl_height(node->left) - avl_height(node->right)`).
- Updating `node->height` after modifying its children takes **$O(1)$ time** (`1 + max(avl_height(node->left), avl_height(node->right))`).

### 3.3 Memory Layout Diagram of an AVL Tree Node

On a 64-bit platform (`sizeof(int) == 4`, `sizeof(void *) == 8`), placing `key` and `height` adjacent to each other packs both integers into a single 8-byte word with zero wasted padding:

```
    AVLNode Memory Layout (Total = 24 bytes on 64-bit C11):
    +-------------------+-------------------+-----------------------------------+-----------------------------------+
    |   int key (4 B)   |  int height (4 B) |     struct AVLNode *left (8 B)    |    struct AVLNode *right (8 B)    |
    +-------------------+-------------------+-----------------------------------+-----------------------------------+
    Byte 0              Byte 4              Byte 8                              Byte 16                             Byte 24
```

---

## 4. The Four Imbalance Cases and Tree Rotations

### 4.1 What Is a Tree Rotation? Preserving the Inorder BST Invariant in $O(1)$ Time

Suppose an insertion or deletion causes some ancestor node $z$ to become the lowest unbalanced node with $|\text{BF}(z)| = 2$. Let:
- **$z$** = the lowest unbalanced ancestor ($|\text{BF}(z)| = 2$),
- **$y$** = the taller child of $z$ along the path to the modified subtree,
- **$x$** = the taller child of $y$.

Depending on whether $y$ is the left or right child of $z$, and whether $x$ is the left or right child of $y$, there are **exactly 4 geometric configurations**: **LL**, **RR**, **LR**, and **RL**.

A **Tree Rotation** locally rearranges at most 3 nodes ($z, y, x$) and re-attaches their 4 external subtrees ($T_1, T_2, T_3, T_4$) in $O(1)$ pointer assignments. Because the horizontal left-to-right ordering of $T_1 < \text{node}_1 < T_2 < \text{node}_2 < T_3 < \text{node}_3 < T_4$ is **identically preserved**, a rotation **never violates the BST property** while reducing the subtree height by $1$!

---

### 4.2 Case 1: Left-Left (LL) Imbalance $\implies$ Single Right Rotation (`rotate_right(z)`)

- **Trigger Condition:** $\text{BF}(z) = +2$ and $\text{BF}(y) \ge 0$ (where $y = z\to\text{left}$). The excess height lies in the **left** subtree of the **left** child of $z$.
- **Fix:** Perform a **Single Right Rotation** around $z$ (`avl_rotate_right(z)`):
  1. Promote $y = z\to\text{left}$ to become the new root of the subtree.
  2. Attach $z$ as the right child of $y$ (`y->right = z`).
  3. Re-parent $y$'s old right subtree $T_3$ (whose keys satisfy $y < T_3 < z$) as the new left child of $z$ (`z->left = T3`).
  4. Update `z->height` **first**, then `y->height` **second**.

```
       BEFORE: Left-Left (LL) Imbalance                 AFTER: Single Right Rotation on z
         BF(z) = +2, BF(y) = +1 (or 0)                      Subtree height drops by 1!
         Inorder: T1 < x < T2 < y < T3 < z < T4             Inorder: T1 < x < T2 < y < T3 < z < T4

                     z (BF=+2)                                        y (BF=0)
                    / \                                             /   \
          (BF=+1)  y   T4 (h)        rotate_right(z)               x     z (BF=0)
                  / \                 =============>              / \   / \
             (h) x   T3 (h)                                      T1 T2 T3  T4
                / \                                             (h)(h)(h) (h)
          (h)  T1  T2 (h)

       Pointer Rewiring:  y = z->left;  T3 = y->right;  y->right = z;  z->left = T3;
```

---

### 4.3 Case 2: Right-Right (RR) Imbalance $\implies$ Single Left Rotation (`rotate_left(z)`)

- **Trigger Condition:** $\text{BF}(z) = -2$ and $\text{BF}(y) \le 0$ (where $y = z\to\text{right}$). The excess height lies in the **right** subtree of the **right** child of $z$ (exact mirror image of LL).
- **Fix:** Perform a **Single Left Rotation** around $z$ (`avl_rotate_left(z)`):
  1. Promote $y = z\to\text{right}$ to become the new root of the subtree.
  2. Attach $z$ as the left child of $y$ (`y->left = z`).
  3. Re-parent $y$'s old left subtree $T_2$ (whose keys satisfy $z < T_2 < y$) as the new right child of $z$ (`z->right = T2`).
  4. Update `z->height` **first**, then `y->height` **second**.

```
       BEFORE: Right-Right (RR) Imbalance               AFTER: Single Left Rotation on z
         BF(z) = -2, BF(y) = -1 (or 0)                      Subtree height drops by 1!
         Inorder: T1 < z < T2 < y < T3 < x < T4             Inorder: T1 < z < T2 < y < T3 < x < T4

               z (BF=-2)                                              y (BF=0)
              / \                                                   /   \
        (h)  T1  y (BF=-1)           rotate_left(z)        (BF=0)  z     x
                / \                   ============>               / \   / \
          (h)  T2  x (h)                                         T1 T2 T3  T4
                  / \                                           (h)(h)(h) (h)
            (h)  T3  T4 (h)

       Pointer Rewiring:  y = z->right;  T2 = y->left;  y->left = z;  z->right = T2;
```

---

### 4.4 Case 3: Left-Right (LR) Imbalance $\implies$ Double Rotation (`rotate_left(y)`, then `rotate_right(z)`)

- **Trigger Condition:** $\text{BF}(z) = +2$ and $\text{BF}(y) = -1 < 0$ (where $y = z\to\text{left}$ and $x = y\to\text{right}$). The excess height lies in the **right** subtree ($x$) of the **left** child ($y$) of $z$.
- **Why a Single Right Rotation on $z$ Fails:** A single right rotation on $z$ moves $x$ (the tallest grandchild!) from `y->right` to `z->left` without reducing $x$'s depth, leaving the rotated tree with $\text{BF}(y) = -2$!
- **Fix — Left-Right Double Rotation:**
  1. **Step 1 (`z->left = avl_rotate_left(y)`):** Rotate **left** around child $y$ to straighten the bent `z -> y -> x` path into a collinear Left-Left chain `z -> x -> y`.
  2. **Step 2 (`return avl_rotate_right(z)`):** Rotate **right** around grandparent $z$, promoting grandchild $x$ to the root of the subtree with $y$ as its left child and $z$ as its right child!

```
   INITIAL: Left-Right (LR)           STEP 1: Left Rotate on y           STEP 2: Right Rotate on z
    BF(z) = +2, BF(y) = -1             Straightens path into LL!          Grandchild x becomes new root!

            z (BF=+2)                          z (BF=+2)                             x (BF=0)
           / \                                / \                                  /   \
   (BF=-1)y   T4                      (BF=+1)x   T4                               y     z
         / \       rotate_left(y)           / \        rotate_right(z)           / \   / \
        T1  x      =============>          y   T3      ==============>          T1 T2 T3  T4
           / \                            / \
          T2  T3                         T1  T2
```

---

### 4.5 Case 4: Right-Left (RL) Imbalance $\implies$ Double Rotation (`rotate_right(y)`, then `rotate_left(z)`)

- **Trigger Condition:** $\text{BF}(z) = -2$ and $\text{BF}(y) = +1 > 0$ (where $y = z\to\text{right}$ and $x = y\to\text{left}$). The excess height lies in the **left** subtree ($x$) of the **right** child ($y$) of $z$ (exact mirror image of LR).
- **Fix — Right-Left Double Rotation:**
  1. **Step 1 (`z->right = avl_rotate_right(y)`):** Rotate **right** around child $y$ to straighten the bent `z -> y -> x` path into a collinear Right-Right chain `z -> x -> y`.
  2. **Step 2 (`return avl_rotate_left(z)`):** Rotate **left** around grandparent $z$, promoting grandchild $x$ to the root of the subtree with $z$ as its left child and $y$ as its right child!

```
   INITIAL: Right-Left (RL)           STEP 1: Right Rotate on y          STEP 2: Left Rotate on z
    BF(z) = -2, BF(y) = +1             Straightens path into RR!          Grandchild x becomes new root!

        z (BF=-2)                          z (BF=-2)                                 x (BF=0)
       / \                                / \                                      /   \
      T1  y (BF=+1)                      T1  x (BF=-1)                            z     y
         / \       rotate_right(y)          / \        rotate_left(z)            / \   / \
        x   T4     ==============>         T2  y       =============>           T1 T2 T3  T4
       / \                                    / \
      T2  T3                                 T3  T4
```

---

### 4.6 Decision Table for All 4 Imbalance Cases

| Imbalance Case | Unbalanced Node $\text{BF}(z)$ | Taller Child $y$ and Its $\text{BF}(y)$ | Geometric Shape | Rebalancing Action in C | New Subtree Root |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Left-Left (LL)** | $\text{BF}(z) = +2$ | $y = z\to\text{left}$, $\text{BF}(y) \ge 0$ | Straight left (`/`) | `return avl_rotate_right(z);` | $y$ |
| **Left-Right (LR)** | $\text{BF}(z) = +2$ | $y = z\to\text{left}$, $\text{BF}(y) < 0$ | Bent left-right (`<`) | `z->left = avl_rotate_left(z->left);`<br>`return avl_rotate_right(z);` | $x = y\to\text{right}$ |
| **Right-Right (RR)** | $\text{BF}(z) = -2$ | $y = z\to\text{right}$, $\text{BF}(y) \le 0$ | Straight right (`\`) | `return avl_rotate_left(z);` | $y$ |
| **Right-Left (RL)** | $\text{BF}(z) = -2$ | $y = z\to\text{right}$, $\text{BF}(y) > 0$ | Bent right-left (`>`) | `z->right = avl_rotate_right(z->right);`<br>`return avl_rotate_left(z);` | $x = y\to\text{left}$ |

> [!IMPORTANT]
> **Why Bottom-Up Height Update Order Inside Rotations Is Mandatory:**
> In `avl_rotate_right(z)` and `avl_rotate_left(z)`, node $z$ moves **downward** to become a child of $y$. Because `y->height` depends on `avl_height(z)`, you **MUST** call `avl_update_height(z)` **before** calling `avl_update_height(y)`! Reversing those two lines computes `y->height` using $z$'s stale pre-rotation height and corrupts every ancestor's balance factor!

---

## 5. Operations and Algorithms

### 5.1 AVL Insertion Algorithm ($O(\log n)$ Time, At Most 1 Rotation)

#### Step-by-Step Algorithm
1. **Standard Recursive BST Descent ($O(\log n)$):** Compare `key` with `root->key` and recursively insert into `root->left` or `root->right`. If `root == NULL`, allocate and return a new leaf with `height = 1`.
2. **Bottom-Up Height Update ($O(1)$ per ancestor):** As recursion unwinds along the insertion path, update `root->height = 1 + max(height(left), height(right))`.
3. **Balance Factor Check & Rotation ($O(1)$):** Compute `bf = height(left) - height(right)`. If `bf > 1` or `bf < -1`, execute the matching case (**LL**, **LR**, **RR**, or **RL**) and return the new subtree root.

> [!TIP]
> **Key Theorem — Why Insertion Requires At Most ONE (Single or Double) Rotation:**
> Suppose inserting a new leaf causes node $z$ to become the lowest unbalanced ancestor ($|\text{BF}(z)| = 2$). Before the insertion, $z$'s subtree had height $h$; the insertion increased $z$'s taller child to height $h$, temporarily raising $z$'s height to $h + 1$. Performing a single (LL/RR) or double (LR/RL) rotation at $z$ **restores the subtree height back to $h$**—the exact height it had before the insertion! Consequently, **no ancestor above $z$ sees any change in subtree height**, so no further rotations can ever be triggered higher up the tree!

---

### 5.2 Complete Step-by-Step Insertion Trace Triggering LL, RR, LR, and RL

Let us trace inserting the sequence **`[30, 20, 10, 40, 50, 5, 8, 60, 55]`** into an initially empty AVL tree:

```
  1. Insert 30, then 20, then 10  ===>  Triggers LL Imbalance at 30 (BF = +2, BF(20) = +1)
  ----------------------------------------------------------------------------------------
        30 (BF=+2)                               20 (BF=0)
       /                                        /  \
      20 (BF=+1)     --- rotate_right(30) -->  10   30
     /                                       (BF=0) (BF=0)
    10 (BF=0)

  2. Insert 40, then 50           ===>  Triggers RR Imbalance at 30 (BF = -2, BF(40) = -1)
  ----------------------------------------------------------------------------------------
        20 (BF=-2)                               20 (BF=-1)
       /  \                                     /  \
     10    30 (BF=-2) --- rotate_left(30) --> 10    40 (BF=0)
             \                                     /  \
              40 (BF=-1)                         30    50
                \
                 50 (BF=0)

  3. Insert 5, then 8             ===>  Triggers LR Imbalance at 10 (BF = +2, BF(5) = -1)
  ----------------------------------------------------------------------------------------
          20                                      20                                    20 (BF=0)
         /  \                                    /  \                                  /  \
  (+2) 10    40                           (+2) 10    40                         (0)   8    40 (0)
       /    /  \    -- rotate_left(5) ->       /    /  \   -- rotate_right(10) ->    / \   / \
 (-1) 5   30    50                            8   30    50                          5  10 30  50
       \                                     /
        8                                   5

  4. Insert 60, then 55           ===>  Triggers RL Imbalance at 50 (BF = -2, BF(60) = +1)
  ----------------------------------------------------------------------------------------
          20                                      20                                    20 (BF=-1)
         /  \                                    /  \                                  /  \
        8    40                                 8    40 (-1)                          8    40 (BF=-1)
       / \   / \                               / \   / \                             / \   / \
      5  10 30  50 (-2) -- rotate_right(60)-> 5  10 30  50 (-2) -- rotate_left(50)->5  10 30  55 (BF=0)
                  \                                       \                                   / \
                   60 (+1)                                 55                                50  60
                  /                                          \
                 55                                           60
```

---

### 5.3 AVL Deletion Algorithm ($O(\log n)$ Time, Up to $O(\log n)$ Rotations)

#### Step-by-Step Algorithm
1. **Standard BST Deletion ($O(\log n)$):**
   - Recursively locate `key`.
   - **0 or 1 child:** Free the target node and return its single child (or `NULL`).
   - **2 children:** Find the **inorder successor** $s$ (minimum key in `root->right`), copy `s->key` into `root->key`, and recursively delete `s->key` from `root->right`.
2. **Bottom-Up Rebalancing Along the Entire Path to the Root:**
   - As recursion unwinds from the physically deleted node all the way to the root, call `avl_rebalance(root)` at **every ancestor**.

> [!WARNING]
> **Why Deletion Can Require $O(\log n)$ Rotations (Unlike Insertion's Single Rotation):**
> When a rotation repairs an unbalanced node $z$ after a **deletion**, the height of the rotated subtree rooted at $y$ (or $x$) often **decreases by 1** compared to $z$'s pre-deletion height! That height reduction can propagate upward and cause $z$'s parent (or grandparent) to also reach $|\text{BF}| = 2$, triggering a cascade of up to $\lfloor h/2 \rfloor = O(\log n)$ rotations along the path to the root!

---

### 5.4 Complete C11 Implementation: AVL Insertion, Deletion, Search, and Validation

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <limits.h>

/* =========================================================================
 * AVL TREE NODE DEFINITION (Height Convention: height(NULL) = 0, Leaf = 1)
 * ========================================================================= */
typedef struct AVLNode {
    int             key;
    int             height; /* 1 + max(height(left), height(right)) */
    struct AVLNode *left;
    struct AVLNode *right;
} AVLNode;

static inline int max_int(int a, int b) {
    return (a > b) ? a : b;
}

static inline int avl_height(const AVLNode *node) {
    return (node == NULL) ? 0 : node->height;
}

static inline int avl_balance_factor(const AVLNode *node) {
    return (node == NULL) ? 0 : (avl_height(node->left) - avl_height(node->right));
}

static inline void avl_update_height(AVLNode *node) {
    if (node != NULL) {
        node->height = 1 + max_int(avl_height(node->left), avl_height(node->right));
    }
}

AVLNode *avl_create_node(int key) {
    AVLNode *node = (AVLNode *)malloc(sizeof(AVLNode));
    if (node == NULL) {
        perror("malloc failed");
        exit(EXIT_FAILURE);
    }
    node->key    = key;
    node->height = 1;
    node->left   = NULL;
    node->right  = NULL;
    return node;
}

/* =========================================================================
 * TREE ROTATIONS (O(1) Pointer Rewiring + Bottom-Up Height Update)
 * ========================================================================= */
AVLNode *avl_rotate_right(AVLNode *z) {
    AVLNode *y  = z->left;
    AVLNode *T3 = y->right;

    y->right = z;
    z->left  = T3;

    /* CRITICAL: Update lower node (z) height BEFORE upper node (y) height! */
    avl_update_height(z);
    avl_update_height(y);
    return y;
}

AVLNode *avl_rotate_left(AVLNode *z) {
    AVLNode *y  = z->right;
    AVLNode *T2 = y->left;

    y->left  = z;
    z->right = T2;

    /* CRITICAL: Update lower node (z) height BEFORE upper node (y) height! */
    avl_update_height(z);
    avl_update_height(y);
    return y;
}

AVLNode *avl_rebalance(AVLNode *z) {
    avl_update_height(z);
    int bf = avl_balance_factor(z);

    /* Case 1: Left-Left (LL) -> BF(z) == +2 and BF(z->left) >= 0 */
    if (bf > 1 && avl_balance_factor(z->left) >= 0) {
        printf("  [Rebalance] LL Imbalance at key=%d (BF=%+d) -> Single Right Rotate(%d)\n",
               z->key, bf, z->key);
        return avl_rotate_right(z);
    }

    /* Case 2: Left-Right (LR) -> BF(z) == +2 and BF(z->left) < 0 */
    if (bf > 1 && avl_balance_factor(z->left) < 0) {
        printf("  [Rebalance] LR Imbalance at key=%d (BF=%+d) -> Left Rotate(%d), then Right Rotate(%d)\n",
               z->key, bf, z->left->key, z->key);
        z->left = avl_rotate_left(z->left);
        return avl_rotate_right(z);
    }

    /* Case 3: Right-Right (RR) -> BF(z) == -2 and BF(z->right) <= 0 */
    if (bf < -1 && avl_balance_factor(z->right) <= 0) {
        printf("  [Rebalance] RR Imbalance at key=%d (BF=%+d) -> Single Left Rotate(%d)\n",
               z->key, bf, z->key);
        return avl_rotate_left(z);
    }

    /* Case 4: Right-Left (RL) -> BF(z) == -2 and BF(z->right) > 0 */
    if (bf < -1 && avl_balance_factor(z->right) > 0) {
        printf("  [Rebalance] RL Imbalance at key=%d (BF=%+d) -> Right Rotate(%d), then Left Rotate(%d)\n",
               z->key, bf, z->right->key, z->key);
        z->right = avl_rotate_right(z->right);
        return avl_rotate_left(z);
    }

    return z; /* Already balanced: BF in {-1, 0, +1} */
}

/* =========================================================================
 * CORE OPERATIONS: INSERT, DELETE, SEARCH, VALIDATE, DISPLAY
 * ========================================================================= */
AVLNode *avl_insert(AVLNode *root, int key) {
    if (root == NULL) {
        return avl_create_node(key);
    }
    if (key < root->key) {
        root->left = avl_insert(root->left, key);
    } else if (key > root->key) {
        root->right = avl_insert(root->right, key);
    } else {
        return root; /* Duplicate keys ignored */
    }
    return avl_rebalance(root);
}

AVLNode *avl_min_node(AVLNode *node) {
    AVLNode *curr = node;
    while (curr != NULL && curr->left != NULL) {
        curr = curr->left;
    }
    return curr;
}

AVLNode *avl_delete(AVLNode *root, int key) {
    if (root == NULL) {
        return NULL;
    }
    if (key < root->key) {
        root->left = avl_delete(root->left, key);
    } else if (key > root->key) {
        root->right = avl_delete(root->right, key);
    } else {
        if (root->left == NULL || root->right == NULL) {
            AVLNode *child = (root->left != NULL) ? root->left : root->right;
            free(root);
            return child;
        } else {
            AVLNode *succ = avl_min_node(root->right);
            root->key     = succ->key;
            root->right   = avl_delete(root->right, succ->key);
        }
    }
    return avl_rebalance(root);
}

const AVLNode *avl_search(const AVLNode *root, int key) {
    while (root != NULL) {
        if (key == root->key) return root;
        root = (key < root->key) ? root->left : root->right;
    }
    return NULL;
}

bool avl_validate(const AVLNode *root, int min_k, int max_k) {
    if (root == NULL) return true;
    if (root->key <= min_k || root->key >= max_k) return false;
    int bf = avl_balance_factor(root);
    if (bf < -1 || bf > 1) return false;
    int expected_h = 1 + max_int(avl_height(root->left), avl_height(root->right));
    if (root->height != expected_h) return false;
    return avl_validate(root->left, min_k, root->key) &&
           avl_validate(root->right, root->key, max_k);
}

void avl_print_2d(const AVLNode *root, int depth) {
    if (root == NULL) return;
    avl_print_2d(root->right, depth + 1);
    for (int i = 0; i < depth; i++) printf("      ");
    printf("%d (h=%d, BF=%+d)\n", root->key, root->height, avl_balance_factor(root));
    avl_print_2d(root->left, depth + 1);
}

void avl_inorder(const AVLNode *root) {
    if (root == NULL) return;
    avl_inorder(root->left);
    printf("%d(BF=%+d) ", root->key, avl_balance_factor(root));
    avl_inorder(root->right);
}

void avl_free(AVLNode *root) {
    if (root == NULL) return;
    avl_free(root->left);
    avl_free(root->right);
    free(root);
}

int main(void) {
    AVLNode *root = NULL;
    int keys[] = { 30, 20, 10, 40, 50, 5, 8, 60, 55 };
    int n = (int)(sizeof(keys) / sizeof(keys[0]));

    printf("=== 1. Step-by-Step AVL Insertions (Triggering LL, RR, LR, RL) ===\n");
    for (int i = 0; i < n; i++) {
        printf("Inserting %d...\n", keys[i]);
        root = avl_insert(root, keys[i]);
    }

    printf("\n=== 2. 2D AVL Tree Structure (Rotated 90 Degrees Counter-Clockwise) ===\n");
    avl_print_2d(root, 0);

    printf("\nInorder Traversal: ");
    avl_inorder(root);
    printf("\nAVL Invariant Valid? %s\n",
           avl_validate(root, INT_MIN, INT_MAX) ? "YES (All |BF| <= 1)" : "NO");

    printf("\n=== 3. AVL Search & Deletion with Automatic Rebalancing ===\n");
    const AVLNode *found = avl_search(root, 55);
    printf("Search(55): %s (height=%d, BF=%+d)\n",
           found ? "FOUND" : "NOT FOUND",
           found ? found->height : 0,
           found ? avl_balance_factor(found) : 0);

    printf("Deleting 30 (causes RR imbalance at 40)...\n");
    root = avl_delete(root, 30);
    printf("Deleting 20 (root with 2 children; replaced by successor 40)...\n");
    root = avl_delete(root, 20);

    printf("\n2D AVL Tree After Deletions:\n");
    avl_print_2d(root, 0);
    printf("AVL Invariant Valid? %s\n",
           avl_validate(root, INT_MIN, INT_MAX) ? "YES (All |BF| <= 1)" : "NO");

    avl_free(root);
    return 0;
}
```

#### Program Output:
```text
=== 1. Step-by-Step AVL Insertions (Triggering LL, RR, LR, RL) ===
Inserting 30...
Inserting 20...
Inserting 10...
  [Rebalance] LL Imbalance at key=30 (BF=+2) -> Single Right Rotate(30)
Inserting 40...
Inserting 50...
  [Rebalance] RR Imbalance at key=30 (BF=-2) -> Single Left Rotate(30)
Inserting 5...
Inserting 8...
  [Rebalance] LR Imbalance at key=10 (BF=+2) -> Left Rotate(5), then Right Rotate(10)
Inserting 60...
Inserting 55...
  [Rebalance] RL Imbalance at key=50 (BF=-2) -> Right Rotate(60), then Left Rotate(50)

=== 2. 2D AVL Tree Structure (Rotated 90 Degrees Counter-Clockwise) ===
                  60 (h=1, BF=+0)
            55 (h=2, BF=+0)
                  50 (h=1, BF=+0)
      40 (h=3, BF=-1)
            30 (h=1, BF=+0)
20 (h=4, BF=-1)
            10 (h=1, BF=+0)
      8 (h=2, BF=+0)
            5 (h=1, BF=+0)

Inorder Traversal: 5(BF=+0) 8(BF=+0) 10(BF=+0) 20(BF=-1) 30(BF=+0) 40(BF=-1) 50(BF=+0) 55(BF=+0) 60(BF=+0) 
AVL Invariant Valid? YES (All |BF| <= 1)

=== 3. AVL Search & Deletion with Automatic Rebalancing ===
Search(55): FOUND (height=2, BF=+0)
Deleting 30 (causes RR imbalance at 40)...
  [Rebalance] RR Imbalance at key=40 (BF=-2) -> Single Left Rotate(40)
Deleting 20 (root with 2 children; replaced by successor 40)...

2D AVL Tree After Deletions:
            60 (h=1, BF=+0)
      55 (h=2, BF=+0)
            50 (h=1, BF=+0)
40 (h=3, BF=+0)
            10 (h=1, BF=+0)
      8 (h=2, BF=+0)
            5 (h=1, BF=+0)
AVL Invariant Valid? YES (All |BF| <= 1)
```

---

## 6. Comparison Tables and Complexity Summary

Let $n$ denote the number of nodes in the tree and $h < 1.44 \log_2(n + 2)$ denote the tree height.

### 6.1 Time and Space Complexity Summary

| Operation | Average-Case Time | **Worst-Case Time** | Rotations Needed (Worst Case) | Auxiliary Space (Recursive) |
| :--- | :--- | :--- | :--- | :--- |
| **`avl_search(key)`** | $\Theta(\log n)$ | **$O(\log n)$** | $0$ | $O(1)$ *(iterative)* |
| **`avl_insert(key)`** | $\Theta(\log n)$ | **$O(\log n)$** | **At most 1** *(single or double)* | $O(\log n)$ call stack |
| **`avl_delete(key)`** | $\Theta(\log n)$ | **$O(\log n)$** | **Up to $O(\log n)$** | $O(\log n)$ call stack |
| **`avl_min` / `avl_max`** | $\Theta(\log n)$ | **$O(\log n)$** | $0$ | $O(1)$ *(iterative)* |
| **`avl_inorder` Traversal** | $\Theta(n)$ | **$O(n)$** | $0$ | $O(\log n)$ call stack |

### 6.2 Unbalanced BST vs. AVL Tree vs. Red-Black Tree vs. Binary Heap

| Feature | Unbalanced BST | **AVL Tree** | Red-Black Tree | Binary Heap |
| :--- | :--- | :--- | :--- | :--- |
| **Worst-Case Height $h$** | $n - 1$ ($O(n)$) | **$< 1.44 \log_2(n + 2)$** | $\le 2 \log_2(n + 1)$ | $\lfloor \log_2 n \rfloor$ |
| **Balance Invariant** | None | $|\text{height}(L) - \text{height}(R)| \le 1$ | 5 color rules (black-height) | Complete Binary Tree |
| **Worst-Case `search(k)`** | $O(n)$ | **$O(\log n)$** *(fastest tree lookup)* | $O(\log n)$ | $O(n)$ |
| **Worst-Case `insert(k)`** | $O(n)$ | **$O(\log n)$** ($\le 2$ single rotations) | $O(\log n)$ ($\le 2$ rotations) | $O(\log n)$ |
| **Worst-Case `delete(k)`** | $O(n)$ | **$O(\log n)$** (up to $O(\log n)$ rotations) | $O(\log n)$ ($\le 3$ rotations) | $O(\log n)$ |
| **Metadata Stored per Node** | `0` bytes | `int height` (or 2-bit BF) | 1 color bit | `0` bytes (array) |

---

## 7. Common Errors, Pitfalls, and Edge Cases

### 7.1 Pitfall 1: Updating `y->height` Before `z->height` After a Rotation

Inside `avl_rotate_right(z)` and `avl_rotate_left(z)`, $z$ becomes a child of $y$. Updating `y->height` first reads the old, pre-rotation `z->height`, leaving `y->height` too large by 1 or 2!

```c
/* BUG: Updating new root y BEFORE lower child z! */
y->right = z;
z->left  = T3;
avl_update_height(y); /* WRONG! Reads z->height before z was recalculated! */
avl_update_height(z);

/* FIX: Always update the LOWER node (z) first, then the UPPER node (y)! */
y->right = z;
z->left  = T3;
avl_update_height(z); /* 1. Recompute child z first  */
avl_update_height(y); /* 2. Recompute parent y next  */
```

### 7.2 Pitfall 2: Using `< 0` Instead of `<= 0` for Child Balance Factor During Deletion

During **insertion**, when $\text{BF}(z) = +2$, its taller child $y = z\to\text{left}$ always has $\text{BF}(y) \in \{+1, -1\}$ (never $0$). However, during **deletion**, removing a node from $z\to\text{right}$ can uncover a left child $y$ whose subtrees have equal height ($\text{BF}(y) == 0$)! If you write `bf > 1 && avl_balance_factor(z->left) > 0` instead of `>= 0`, the $\text{BF}(y) == 0$ case after deletion will either be skipped or misclassified as LR!

```c
/* BUG: Strict inequality misses BF(z->left) == 0 after a deletion! */
if (bf > 1 && avl_balance_factor(z->left) > 0) {
    return avl_rotate_right(z);
}

/* FIX: Use >= 0 for LL and <= 0 for RR so BF(y) == 0 uses a single rotation */
if (bf > 1 && avl_balance_factor(z->left) >= 0) {
    return avl_rotate_right(z);
}
if (bf < -1 && avl_balance_factor(z->right) <= 0) {
    return avl_rotate_left(z);
}
```

### 7.3 Pitfall 3: Discarding the New Subtree Root Returned by Rotation

Because C passes pointers by value, `avl_insert(root->left, key)` and `avl_rotate_right(z)` return the **new root pointer** of that subtree. Failing to assign `root = avl_insert(root, key)` or `z->left = avl_rotate_left(z->left)` leaves parent pointers pointing at the old root!

```c
/* BUG: Ignoring the return value of rotation in LR double rotation */
if (bf > 1 && avl_balance_factor(z->left) < 0) {
    avl_rotate_left(z->left); /* BUG! z->left still points to old y instead of x! */
    return avl_rotate_right(z);
}

/* FIX: Reassign z->left to the new subtree root returned by avl_rotate_left */
if (bf > 1 && avl_balance_factor(z->left) < 0) {
    z->left = avl_rotate_left(z->left);
    return avl_rotate_right(z);
}
```

### 7.4 Pitfall 4: Calling `height(node)` Recursively in $O(n)$ Instead of Reading `node->height` in $O(1)$

Never write a recursive height function `1 + max(get_h(n->left), get_h(n->right))` inside `avl_balance_factor`—that visits the entire subtree at every level and turns $O(\log n)$ insertion into $O(n)$! Always read the cached `node->height` field in $O(1)$.

### 7.5 Boundary Conditions Checklist

- [ ] **Empty Tree (`root == NULL`):** Do `avl_height(NULL)` and `avl_balance_factor(NULL)` safely return `0` without dereferencing `NULL`?
- [ ] **Root Rotation (`root` itself has $|\text{BF}| == 2$):** Does the caller assign `root = avl_insert(root, key)` and `root = avl_delete(root, key)` so `root` updates when the top of the tree rotates?
- [ ] **Duplicate Key Insertion (`key == root->key`):** Does `avl_insert` return `root` unchanged without allocating a duplicate node?
- [ ] **Deleting a 2-Child Node:** After copying `succ->key` into `root->key` and calling `root->right = avl_delete(root->right, succ->key)`, does `avl_delete` still call `return avl_rebalance(root)` before returning?

---

## 8. Summary and Revision Checklist

### 8.1 Key Takeaways and Core Invariants

1. **AVL Balance Invariant:** Every node $v$ in an AVL tree satisfies $\text{BF}(v) = \text{height}(v\to\text{left}) - \text{height}(v\to\text{right}) \in \{-1, 0, +1\}$.
2. **Worst-Case Height Bound:** A minimal AVL tree of height $h$ has $N(h) = F_{h+3} - 1$ nodes, proving $h < 1.44 \log_2(n + 2) = O(\log n)$ in the worst case.
3. **Four Rotation Cases:**
   - **LL ($\text{BF}(z) = +2, \text{BF}(y) \ge 0$):** Single Right Rotation on $z$.
   - **RR ($\text{BF}(z) = -2, \text{BF}(y) \le 0$):** Single Left Rotation on $z$.
   - **LR ($\text{BF}(z) = +2, \text{BF}(y) < 0$):** Left Rotation on $y$, then Right Rotation on $z$.
   - **RL ($\text{BF}(z) = -2, \text{BF}(y) > 0$):** Right Rotation on $y$, then Left Rotation on $z$.
4. **Insertion vs. Deletion Rebalancing:** Insertion requires **at most 1** (single or double) rotation globally because rotation restores the pre-insertion subtree height; deletion may require **$O(\log n)$** rotations up to the root because rotation can shrink the subtree height by 1.

### 8.2 Self-Test Revision Questions

1. **Formula Question:** Using the recurrence $N(h) = 1 + N(h-1) + N(h-2)$ with $N(0) = 1$ and $N(1) = 2$, what is the minimum number of nodes in an AVL tree of edge-height $h = 5$?
   *(Answer: $N(2) = 4$, $N(3) = 7$, $N(4) = 12$, $N(5) = 1 + 12 + 7 = 20$ nodes).*
2. **Trace Question:** Insert the keys `10, 20, 30, 15, 18` one by one into an empty AVL tree. Which rotations occur and after which insertions?
   *(Answer: Inserting `30` triggers an **RR** single left rotation at `10` (making `20` the root); inserting `18` triggers an **LR** double rotation at `10` (`rotate_left(15)` then `rotate_right(10)`), making `15` the left child of `20` with children `10` and `18`).*
3. **Conceptual Question:** Why does an LR imbalance ($\text{BF}(z) = +2, \text{BF}(y) = -1$) require two rotations instead of a single right rotation on $z$?
   *(Answer: Because the tallest subtree is rooted at grandchild $x = y\to\text{right}$. A single right rotation on $z$ simply shifts $x$ from `y->right` to `z->left` at the exact same depth, leaving the subtree unbalanced with $\text{BF}(y) = -2$).*

### 8.3 Navigation Links

- **Previous Topic:** [04-Heaps-and-Heap-Priority-Queues.md](./04-Heaps-and-Heap-Priority-Queues.md) — Binary Heaps and Heap-Based Priority Queues
- **Current File:** `05-AVL-Trees.md` (Unit 3 — Topic 5)
- **Next Topic:** [06-Splay-Trees.md](./06-Splay-Trees.md) — Self-Adjusting Splay Trees and Amortized Analysis
