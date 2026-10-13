# Unit 3 — Topic 6: Self-Adjusting Binary Search Trees (Splay Trees)

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [What Is a Splay Tree? (Sleator and Tarjan, 1985)](#11-what-is-a-splay-tree-sleator-and-tarjan-1985)
   - 1.2 [Motivation: Temporal Locality (The 80/20 Rule) and Zero Metadata Overhead](#12-motivation-temporal-locality-the-8020-rule-and-zero-metadata-overhead)
   - 1.3 [Why Naive Single Bottom-Up Rotations ("Move-to-Root") Fail ($\Theta(n)$ Amortized)](#13-why-naive-single-bottom-up-rotations-move-to-root-fail-thetan-amortized)
   - 1.4 [Real-World and Systems Applications](#14-real-world-and-systems-applications)
   - 1.5 [Learning Objectives](#15-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Formal Definition of Splaying and Self-Adjustment](#21-formal-definition-of-splaying-and-self-adjustment)
   - 2.2 [Core Terminology Table](#22-core-terminology-table)
   - 2.3 [Amortized Complexity and the Banker's / Potential Method Intuition](#23-amortized-complexity-and-the-bankers--potential-method-intuition)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [`SplayNode` Structure Declaration (Zero Metadata Bytes)](#31-splaynode-structure-declaration-zero-metadata-bytes)
   - 3.2 [Three Implementation Strategies for Splaying in C](#32-three-implementation-strategies-for-splaying-in-c)
   - 3.3 [Memory Layout Comparison: `AVLNode` vs. `SplayNode`](#33-memory-layout-comparison-avlnode-vs-splaynode)
4. [The Three Splay Steps (Six Symmetric Cases)](#4-the-three-splay-steps-six-symmetric-cases)
   - 4.1 [Case 1: Zig / Zag Step (Terminal Step When Parent $p$ Is the Root)](#41-case-1-zig--zag-step-terminal-step-when-parent-p-is-the-root)
   - 4.2 [Case 2: Zig-Zig / Zag-Zag Step (Homogeneous Grandparent — Rotate $g$ FIRST, Then $p$!)](#42-case-2-zig-zig--zag-zag-step-homogeneous-grandparent--rotate-g-first-then-p)
   - 4.3 [Why Rotating $g$ First in Zig-Zig Is Crucial (Halving Path Depth vs. Classic Exam Error)](#43-why-rotating-g-first-in-zig-zig-is-crucial-halving-path-depth-vs-classic-exam-error)
   - 4.4 [Case 3: Zig-Zag / Zag-Zig Step (Heterogeneous Grandparent — Rotate $p$ FIRST, Then $g$!)](#44-case-3-zig-zag--zag-zig-step-heterogeneous-grandparent--rotate-p-first-then-g)
   - 4.5 [Master Summary Table of All 6 Splay Cases](#45-master-summary-table-of-all-6-splay-cases)
5. [Operations and Algorithms](#5-operations-and-algorithms)
   - 5.1 [The Core `splay(root, key)` Operation](#51-the-core-splayroot-key-operation)
   - 5.2 [`splay_search(root, key)`: Successful vs. Unsuccessful Search](#52-splay_searchroot-key-successful-vs-unsuccessful-search)
   - 5.3 [`splay_insert(root, key)`: Bottom-Up Insertion vs. Split-and-Splay Insertion](#53-splay_insertroot-key-bottom-up-insertion-vs-split-and-splay-insertion)
   - 5.4 [`splay_delete(root, key)`: Splay-to-Root and `Join(T_L, T_R)`](#54-splay_deleteroot-key-splay-to-root-and-joint_l-t_r)
   - 5.5 [Complete C11 Implementation: Splay Tree (`splay`, `search`, `insert`, `delete`, 2D Display)](#55-complete-c11-implementation-splay-tree-splay-search-insert-delete-2d-display)
6. [Complexity Analysis and Comparison: AVL Tree vs. Splay Tree](#6-complexity-analysis-and-comparison-avl-tree-vs-splay-tree)
   - 6.1 [Worst-Case Single Operation vs. Amortized $O(\log n)$ Guarantee](#61-worst-case-single-operation-vs-amortized-olog-n-guarantee)
   - 6.2 [Comprehensive Comparison Table: AVL Tree vs. Splay Tree vs. Unbalanced BST](#62-comprehensive-comparison-table-avl-tree-vs-splay-tree-vs-unbalanced-bst)
7. [Common Errors, Pitfalls, and Edge Cases](#7-common-errors-pitfalls-and-edge-cases)
   - 7.1 [Pitfall 1: Rotating Parent $p$ Before Grandparent $g$ in a Zig-Zig Step](#71-pitfall-1-rotating-parent-p-before-grandparent-g-in-a-zig-zig-step)
   - 7.2 [Pitfall 2: Failing to Splay the Last Visited Node on an Unsuccessful Search](#72-pitfall-2-failing-to-splay-the-last-visited-node-on-an-unsuccessful-search)
   - 7.3 [Pitfall 3: Using `Zig` Inside the Loop Instead of Only as the Terminal Root Step](#73-pitfall-3-using-zig-inside-the-loop-instead-of-only-as-the-terminal-root-step)
   - 7.4 [Pitfall 4: Forgetting to Update the Caller's `root` Pointer After `splay_search`](#74-pitfall-4-forgetting-to-update-the-callers-root-pointer-after-splay_search)
   - 7.5 [Boundary Conditions Checklist](#75-boundary-conditions-checklist)
8. [Summary and Revision Checklist](#8-summary-and-revision-checklist)
   - 8.1 [Key Takeaways and Core Invariants](#81-key-takeaways-and-core-invariants)
   - 8.2 [Self-Test Revision Questions](#82-self-test-revision-questions)
   - 8.3 [Navigation Links](#83-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 What Is a Splay Tree? (Sleator and Tarjan, 1985)

Invented by **Daniel Sleator** and **Robert Endre Tarjan** in 1985, a **Splay Tree** is a **self-adjusting Binary Search Tree** with a radical design philosophy:
1. **Zero balance metadata:** Unlike AVL trees (which store subtree `height` or balance factors) and Red-Black trees (which store color bits), a Splay Tree stores **no height, no balance factor, and no color bit** in its nodes.
2. **Restructuring on every access ("Splaying"):** Whenever any node $x$ is accessed—whether by **search**, **insertion**, or **deletion**—the tree executes a sequence of specialized two-level rotations (called **splaying**) that moves $x$ all the way to the **root of the tree**.

```
       BEFORE SEARCH(60)                                AFTER SEARCH(60) (Splayed to Root!)
   (Deep skewed path of height 5)                    (60 is now at the root; tree height halved!)

        10                                                        60
          \                                                      /
           20                                                  10
             \                                                   \
              30             --- splay(root, 60) --->             30
                \              (2 Zag-Zags + 1 Zag)              /  \
                 40                                            20    50
                   \                                                /
                    50                                            40
                      \
                       60
```

### 1.2 Motivation: Temporal Locality (The 80/20 Rule) and Zero Metadata Overhead

Strict height-balanced trees like AVL trees optimize for a **uniform access distribution**, keeping every key within $1.44 \log_2 n$ comparisons of the root. However, real-world software workloads almost never access keys uniformly; instead, they exhibit strong **Temporal Locality** (the **80/20 Pareto Rule**—roughly 80% of accesses hit 20% of the active working set of keys, such as hot network flows, recently resolved DNS names, or active memory pages).

A **Splay Tree** exploits temporal locality automatically:
- **Hot keys stay near the root:** Moving the accessed node to the root ensures that if the same key (or nearby keys in its immediate neighborhood) is accessed again soon, the lookup completes in **$O(1)$ or near-$O(1)$ time**!
- **Self-healing along deep paths:** Even if an adversary inserts keys in sorted order to create a skewed chain of height $n - 1$, the very first access to the deepest leaf executes **Zig-Zig** rotations that **halve the depth of every node along that path**, paying for the expensive $O(n)$ search by leaving behind a balanced tree!

### 1.3 Why Naive Single Bottom-Up Rotations ("Move-to-Root") Fail ($\Theta(n)$ Amortized)

Before Sleator and Tarjan's 1985 paper, researchers (Allen and Munro, 1978) tried moving the accessed node $x$ to the root by simply performing **single rotations between $x$ and its parent $p$ repeatedly** until $x$ reached the root.

> [!CAUTION]
> **Why Naive Single Rotations Fail:** If you rotate $x$ with its parent $p$ one level at a time (`Move-to-Root`), accessing the deepest node in a skewed chain pushes all the other nodes back down into a new skewed chain of height $n - 1$! Accessing keys $1, 2, 3, \dots, n$ in sequence under naive single rotations takes **$\Theta(n^2)$ total time ($\Theta(n)$ amortized per operation)**!

Sleator and Tarjan discovered that looking **two levels up** at both parent $p$ and grandparent $g$—and rotating the **grandparent $g$ first** before rotating the parent $p$ when $x$ and $p$ are on the same side (**Zig-Zig**)—halves the depth of all nodes on the access path and guarantees **$O(\log n)$ amortized time** over any sequence of operations!

### 1.4 Real-World and Systems Applications

1. **Network Routers & Firewalls:** IP packet classification caches, NAT connection tables, and intrusion detection flow tables where a small subset of high-bandwidth flows accounts for the vast majority of packets.
2. **Memory Allocators & Garbage Collectors:** `gcc` (`libiberty` `splay-tree.c`), the FreeBSD virtual memory map (`vm_map_entry` in earlier BSD kernels), and conservative garbage collectors use splay trees to track allocated address intervals with zero metadata overhead per node.
3. **Data Compression (Dynamic Markov / Arithmetic Coding):** Adaptive symbol-frequency trees where frequently occurring characters migrate to the root for short encoding paths.
4. **Rope / String Editors & Link-Cut Trees:** Sleator and Tarjan's **Link-Cut Trees** (for $O(\log n)$ dynamic graph connectivity and network flow) use splay trees as their core auxiliary data structure.

### 1.5 Learning Objectives

After completing this chapter, you will be able to:
- Explain why Splay Trees require zero balance metadata per node and how they exploit temporal locality.
- Trace and implement all **3 Splay Steps (6 symmetric cases)**: **Zig / Zag**, **Zig-Zig / Zag-Zag**, and **Zig-Zag / Zag-Zig**.
- Contrast the **Zig-Zig** rotation order (**rotate grandparent $g$ first, then parent $p$**) against the **Zig-Zag** rotation order (**rotate parent $p$ first, then grandparent $g$**).
- Implement Splay Tree **Search** (including splaying the last visited node on a miss), **Insertion**, and **Deletion** (via `Split` and `Join`) in C11.
- Distinguish **worst-case $O(\log n)$ per operation (AVL Tree)** from **amortized $O(\log n)$ per operation (Splay Tree)**.

---

## 2. Definitions and Terminology

### 2.1 Formal Definition of Splaying and Self-Adjustment

Given a target node $x$ in a Binary Search Tree $T$, **$\text{Splay}(x, T)$** is a restructuring procedure that repeatedly applies two-level rotations (`Zig-Zig`/`Zag-Zag` or `Zig-Zag`/`Zag-Zig`) along the ancestor path from $x$ to the root—followed by at most one single rotation (`Zig`/`Zag`) if $x$ becomes a direct child of the root—until $x$ is the new root of $T$.

### 2.2 Core Terminology Table

| Term | Formal Definition |
| :--- | :--- |
| **Splaying (`splay(x)`)** | Moving node $x$ to the root of the BST via a sequence of `Zig-Zig`, `Zig-Zag`, and at most one terminal `Zig` step. |
| **Zig (or Zag) Step** | A **single rotation** performed **only** when target $x$'s parent $p$ is the root of the tree (no grandparent exists). |
| **Zig-Zig (or Zag-Zag) Step** | A **homogeneous two-level rotation** when $x$ and its parent $p$ are both left children (LL) or both right children (RR) of grandparent $g$. **Rotates $g$ first, then $p$.** |
| **Zig-Zag (or Zag-Zig) Step** | A **heterogeneous two-level rotation** when $x$ is a right child of a left child (LR) or a left child of a right child (RL). **Rotates $p$ first, then $g$.** |
| **Split($T, k$)** | Splaying key $k$ to the root of $T$ and severing a child link to partition $T$ into two trees: $T_L$ (all keys $\le k$) and $T_R$ (all keys $> k$). |
| **Join($T_L, T_R$)** | Combining two splay trees $T_L$ and $T_R$ (where every key in $T_L$ is strictly less than every key in $T_R$) by splaying the maximum element of $T_L$ to its root and attaching $T_R$ as its right child. |
| **Amortized Time Complexity** | The average worst-case time per operation guaranteed over **any** sequence of $m$ operations (regardless of input distribution), proven via a potential function $\Phi(T)$. |

### 2.3 Amortized Complexity and the Banker's / Potential Method Intuition

In a Splay Tree of $n$ nodes, a **single** operation can take **$O(n)$ time in the worst case** (for example, searching for the deepest leaf when the tree happens to be a skewed chain of height $n - 1$). Yet over **any sequence of $m$ operations** starting from an empty tree, the **total worst-case time is guaranteed to be $O(m \log n)$**—meaning the **amortized cost per operation is $O(\log n)$**!

#### How the Potential Function Proof Works (Sleator & Tarjan's Access Lemma)
Define for each node $v \in T$:
- $\text{size}(v)$ = the number of nodes in the subtree rooted at $v$.
- $\text{rank}(v) = r(v) = \lfloor \log_2 \text{size}(v) \rfloor$.
- **Potential of tree $T$:**
  $$\Phi(T) = \sum_{v \in T} r(v) = \sum_{v \in T} \lfloor \log_2 \text{size}(v) \rfloor$$

Notice what $\Phi(T)$ measures:
- When $T$ is **well-balanced**, most nodes live near the leaves with small subtrees ($\text{size}(v) \approx 1 \implies r(v) = 0$), so **$\Phi(T)$ is small** ($O(n)$).
- When $T$ is a **degenerate skewed chain**, nodes have sizes $1, 2, 3, \dots, n$, so **$\Phi(T)$ is large** ($\Theta(n \log n)$).

Sleator and Tarjan proved the **Access Lemma**: the amortized cost $\hat{c}_{\text{step}} = c_{\text{actual}} + \Delta \Phi$ of a single `Zig-Zig` or `Zig-Zag` step moving $x$ from rank $r(x)$ to new rank $r'(x)$ satisfies:

$$\hat{c}_{\text{step}} \le 3 \bigl(r'(x) - r(x)\bigr)$$

When we sum this telescopic bound over all steps that lift $x$ from its initial rank $r_0(x) \ge 0$ to the root rank $r(\text{root}) = \lfloor \log_2 n \rfloor$ (plus $1$ for a possible terminal `Zig` step), all intermediate ranks cancel out:

$$\text{Amortized Cost of } \text{Splay}(x) \le 3\bigl(r(\text{root}) - r_0(x)\bigr) + 1 \le 3 \log_2 n + 1 = O(\log n)$$

Whenever an actual splay takes $O(n)$ rotations on a deep skewed chain, its `Zig-Zig` steps compress the tree so drastically that $\Delta \Phi$ is **heavily negative** ($\approx -O(n)$), paying for the expensive traversal out of the stored potential!

---

## 3. Representation and Memory Layout in C

### 3.1 `SplayNode` Structure Declaration (Zero Metadata Bytes)

Because a Splay Tree maintains no explicit balance invariants, its node structure is identical to a minimal Binary Search Tree node:

```c
typedef struct SplayNode {
    int               key;   /* Search key                                */
    struct SplayNode *left;  /* Pointer to left child  (all keys < key)   */
    struct SplayNode *right; /* Pointer to right child (all keys > key)   */
} SplayNode;
```

### 3.2 Three Implementation Strategies for Splaying in C

To perform bottom-up rotations on $x$, $p$, and $g$, we need access to $x$'s ancestors along the search path. In C, there are three standard ways to implement this without wasting memory:

| Strategy | Extra Space in `SplayNode` | Auxiliary Space During `splay()` | Description |
| :--- | :--- | :--- | :--- |
| **1. Explicit Ancestor Stack (`path[]`)** *(Used in Our C11 Code)* | **0 bytes** | Small local array `SplayNode *path[128]` on stack | Records ancestors during downward BST search, then pops `(p, g)` pairs bottom-up to execute exact `Zig-Zig`, `Zig-Zag`, and terminal `Zig` steps! |
| **2. Top-Down Splaying (Sleator & Tarjan)** | **0 bytes** | **$O(1)$** (two temporary header trees $L$ and $R$) | Splits and rotates on the way *down* the tree, reassembling $L$, $R$, and the middle tree at the end. |
| **3. Parent Pointer (`SplayNode *parent`)** | **8 bytes per node** | $O(1)$ | Walks down to $x$, then follows `x->parent` and `x->parent->parent` upward. Simple, but adds 8 bytes per node. |

### 3.3 Memory Layout Comparison: `AVLNode` vs. `SplayNode`

```
    AVLNode (24 bytes on 64-bit):
    +-------------+---------------+--------------------+--------------------+
    | int key (4) | int height(4) |  AVLNode *left (8) | AVLNode *right (8) |
    +-------------+---------------+--------------------+--------------------+

    SplayNode (24 bytes with alignment, or 16 bytes with 32-bit pointers; ZERO metadata):
    +-------------+---------------+--------------------+--------------------+
    | int key (4) |  (4B unused   | SplayNode *left(8) | SplayNode *right(8)|
    |             |  or payload)  |                    |                    |
    +-------------+---------------+--------------------+--------------------+
```

---

## 4. The Three Splay Steps (Six Symmetric Cases)

Let $x$ be the target node currently being splayed upward, $p$ be $x$'s parent, and $g$ be $x$'s grandparent (if $p$ is not the root). At each iteration of bottom-up splaying, $x$ falls into one of **3 structural cases** (6 counting left/right mirror symmetry):

---

### 4.1 Case 1: Zig / Zag Step (Terminal Step When Parent $p$ Is the Root)

- **Trigger Condition:** Target $x$ is a **direct child of the tree root $p$** (i.e., $p$ has no parent: $\text{parent}(p) == \text{NULL}$).
- **Important Rule:** A **Zig** (or **Zag**) step is executed **at most once**, at the very **end** of a splay operation when the initial depth of $x$ was odd!
- **Action:**
  - **Case 1a — Zig ($x$ is the left child of root $p$):** Perform a single **Right Rotation** on root $p$ (`splay_rotate_right(p)`).
  - **Case 1b — Zag ($x$ is the right child of root $p$):** Perform a single **Left Rotation** on root $p$ (`splay_rotate_left(p)`).

```
      CASE 1a: ZIG STEP (x is left child of root p)        CASE 1b: ZAG STEP (x is right child of root p)
      =============================================        ==============================================

              p (root)                x (new root)                 p (root)                x (new root)
             / \                     / \                          / \                     / \
            x   T3   -- Zig(p) -->  T1  p                        T1  x    -- Zag(p) -->  p   T3
           / \      (Rotate Right)     / \                          / \   (Rotate Left) / \
          T1  T2                      T2  T3                       T2  T3              T1  T2
```

---

### 4.2 Case 2: Zig-Zig / Zag-Zag Step (Homogeneous Grandparent — Rotate $g$ FIRST, Then $p$!)

- **Trigger Condition:** Target $x$ has both a parent $p$ and a grandparent $g$, and **$x$ and $p$ are on the SAME side** (both are left children, or both are right children).
- **Action:**
  - **Case 2a — Zig-Zig (Left-Left: $p = g\to\text{left}$ and $x = p\to\text{left}$):**
    1. **FIRST:** Rotate **Right** around **grandparent $g$** (`splay_rotate_right(g)`), promoting $p$ above $g$.
    2. **SECOND:** Rotate **Right** around **parent $p$** (`splay_rotate_right(p)`), promoting $x$ above $p$.
  - **Case 2b — Zag-Zag (Right-Right: $p = g\to\text{right}$ and $x = p\to\text{right}$):**
    1. **FIRST:** Rotate **Left** around **grandparent $g$** (`splay_rotate_left(g)`), promoting $p$ above $g$.
    2. **SECOND:** Rotate **Left** around **parent $p$** (`splay_rotate_left(p)`), promoting $x$ above $p$.

```
  CASE 2a: ZIG-ZIG (Left-Left Homogeneous Step)
  =============================================
  CRITICAL ORDER: 1st Rotate Grandparent g Right,  2nd Rotate Parent p Right!

          g                        p                       x (new subtree root)
         / \                      / \                     / \
        p   T4   1. Right(g)     x   g    2. Right(p)    T1  p
       / \       ===========>   / \ / \   ===========>      / \
      x   T3                   T1 T2 T3 T4                 T2  g
     / \                                                      / \
    T1  T2                                                   T3  T4

  CASE 2b: ZAG-ZAG (Right-Right Homogeneous Step)
  ===============================================
  CRITICAL ORDER: 1st Rotate Grandparent g Left,   2nd Rotate Parent p Left!

      g                            p                           x (new subtree root)
     / \                          / \                         / \
    T1  p        1. Left(g)      g   x     2. Left(p)        p   T4
       / \       ==========>    / \ / \    ==========>      / \
      T2  x                    T1 T2 T3 T4                 g   T3
         / \                                              / \
        T3  T4                                           T1  T2
```

---

### 4.3 Why Rotating $g$ First in Zig-Zig Is Crucial (Halving Path Depth vs. Classic Exam Error)

> [!IMPORTANT]
> **Classic Exam & Interview Question:** In a Left-Left (`Zig-Zig`) configuration `g -> p -> x`, what is the difference between:
> - **Correct `Zig-Zig`:** Rotate grandparent $g$ right **first**, then rotate parent $p$ right.
> - **Incorrect (Two Single Rotations):** Rotate parent $p$ right **first**, then rotate grandparent $g$ right.

Look at the resulting trees when we apply both methods along a 4-node left chain `40 -> 30 -> 20 -> 10` to bring `10` to the root:

```
   INITIAL SKEWED CHAIN           INCORRECT: Rotate p FIRST, then g          CORRECT ZIG-ZIG: Rotate g FIRST, then p
     (Height = 3)                 (Naive "Move-to-Root" single rotations:    (Grandparent g rotates before p:
                                   40, 30, 20 stay in a deep right chain!)    30 becomes a parent of 40, compacting!)

          40                                 10                                         10
         /                                     \                                          \
        30                                      40                                         30
       /                                       /                                          /  \
      20                                      30                                        20    40
     /                                       /
    10                                      20
```

When you rotate **$g$ first, then $p$**, $p$ becomes the parent of $g$ (instead of $g$ remaining above $p$), folding the chain in half like an accordion! Across a skewed chain of length $k$, true **Zig-Zig** steps reduce the height from $k$ to $\approx k/2$, whereas rotating $p$ first leaves the height at $k$!

---

### 4.4 Case 3: Zig-Zag / Zag-Zig Step (Heterogeneous Grandparent — Rotate $p$ FIRST, Then $g$!)

- **Trigger Condition:** Target $x$ has both a parent $p$ and a grandparent $g$, and **$x$ and $p$ are on OPPOSITE sides** (bent path: Left-Right or Right-Left).
- **Action (Identical to AVL Double Rotation!):**
  - **Case 3a — Zig-Zag (Left-Right: $p = g\to\text{left}$ and $x = p\to\text{right}$):**
    1. **FIRST:** Rotate **Left** around **parent $p$** (`g->left = splay_rotate_left(p)`), lifting $x$ to become left child of $g$.
    2. **SECOND:** Rotate **Right** around **grandparent $g$** (`splay_rotate_right(g)`), lifting $x$ above both $p$ and $g$.
  - **Case 3b — Zag-Zig (Right-Left: $p = g\to\text{right}$ and $x = p\to\text{left}$):**
    1. **FIRST:** Rotate **Right** around **parent $p$** (`g->right = splay_rotate_right(p)`), lifting $x$ to become right child of $g$.
    2. **SECOND:** Rotate **Left** around **grandparent $g$** (`splay_rotate_left(g)`), lifting $x$ above both $g$ and $p$.

```
  CASE 3a: ZIG-ZAG (Left-Right Heterogeneous Step)
  ================================================
  ORDER: 1st Rotate Parent p Left,  2nd Rotate Grandparent g Right!

          g                        g                         x (new subtree root)
         / \                      / \                       /   \
        p   T4   1. Left(p)      x   T4   2. Right(g)      p     g
       / \       ==========>    / \       ===========>    / \   / \
      T1  x                    p   T3                    T1 T2 T3  T4
         / \                  / \
        T2  T3               T1  T2

  CASE 3b: ZAG-ZIG (Right-Left Heterogeneous Step)
  ================================================
  ORDER: 1st Rotate Parent p Right, 2nd Rotate Grandparent g Left!

        g                        g                             x (new subtree root)
       / \                      / \                           /   \
      T1  p      1. Right(p)   T1  x      2. Left(g)         g     p
         / \     ===========>     / \     ==========>       / \   / \
        x   T4                   T2  p                     T1 T2 T3  T4
       / \                          / \
      T2  T3                       T3  T4
```

---

### 4.5 Master Summary Table of All 6 Splay Cases

| Splay Step Name | Geometric Path ($g \to p \to x$) | When Applied | **1st Rotation** | **2nd Rotation** | Resulting Subtree Structure |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **1a. Zig** | `p -> left = x` | Only when $p$ is **root** | `rotate_right(p)` | *(None)* | $x$ at root, $p = x\to\text{right}$ |
| **1b. Zag** | `p -> right = x` | Only when $p$ is **root** | `rotate_left(p)` | *(None)* | $x$ at root, $p = x\to\text{left}$ |
| **2a. Zig-Zig** | **Left-Left** (`g->left = p`, `p->left = x`) | Homogeneous grandparent | **`rotate_right(g)`** *(Grandparent first!)* | **`rotate_right(p)`** *(Parent second!)* | $x$ at top, $p = x\to\text{right}$, $g = p\to\text{right}$ |
| **2b. Zag-Zag** | **Right-Right** (`g->right = p`, `p->right = x`) | Homogeneous grandparent | **`rotate_left(g)`** *(Grandparent first!)* | **`rotate_left(p)`** *(Parent second!)* | $x$ at top, $p = x\to\text{left}$, $g = p\to\text{left}$ |
| **3a. Zig-Zag** | **Left-Right** (`g->left = p`, `p->right = x`) | Heterogeneous grandparent | **`rotate_left(p)`** *(Parent first!)* | **`rotate_right(g)`** *(Grandparent second!)* | $x$ at top, $p = x\to\text{left}$, $g = x\to\text{right}$ |
| **3b. Zag-Zig** | **Right-Left** (`g->right = p`, `p->left = x`) | Heterogeneous grandparent | **`rotate_right(p)`** *(Parent first!)* | **`rotate_left(g)`** *(Grandparent second!)* | $x$ at top, $g = x\to\text{left}$, $p = x\to\text{right}$ |

---

## 5. Operations and Algorithms

### 5.1 The Core `splay(root, key)` Operation

Every operation in a Splay Tree (`search`, `insert`, `delete`, `split`, `join`) delegates its heavy lifting to a single universal primitive: **`root = splay(root, key)`**.

#### Specification of `splay(root, key)`
- **If `key` is present in the tree:** Splay the node containing `key` to the root and return it.
- **If `key` is NOT present in the tree:** Splay the **last non-`NULL` node visited** during the downward search (which is either the inorder predecessor or inorder successor of `key`) to the root and return it!

> [!WARNING]
> **Why You Must Splay Even on an Unsuccessful Search:**
> Suppose an adversary repeatedly searches for a non-existent key `999999` at the bottom of a right-skewed chain of height $n$. If you did **not** splay the last visited node (`max_node`) to the root on a search miss, every failed search would take $\Theta(n)$ time without ever repairing the skewed chain! Splaying the last visited node ensures that **both successful and unsuccessful searches** run in $O(\log n)$ amortized time!

---

### 5.2 `splay_search(root, key)`: Successful vs. Unsuccessful Search

1. Call `root = splay(root, key)`.
2. Check if `root != NULL && root->key == key`:
   - If `true`: the search succeeded, and the found node is now `root`.
   - If `false`: the search missed, and the closest neighbor key (last visited node) is now `root`.

---

### 5.3 `splay_insert(root, key)`: Bottom-Up Insertion vs. Split-and-Splay Insertion

There are two equivalent ways to insert `key` into a Splay Tree:

#### Method A: Standard BST Insert + Bottom-Up Splay
1. Insert `key` as a new leaf using standard BST insertion.
2. Splay the newly inserted leaf (or existing duplicate node) up to the root via `splay(root, key)`.

#### Method B: Split-and-Splay Insertion (Sleator & Tarjan's Single-Pass Method)
1. If `root == NULL`, allocate and return a single-node tree `new_node(key)`.
2. Call `root = splay(root, key)` to bring the closest existing key to the root.
3. If `root->key == key`, `key` is already in the tree (return `root`).
4. Otherwise, allocate `new_root = splay_create_node(key)` and split `root` around `key`:
   - **If `key < root->key`:** Because `root` was the last node visited when searching for `key`, every key in `root->left` is strictly $< \text{key}$! Attach `new_root->left = root->left`, `new_root->right = root`, and set `root->left = NULL`.
   - **If `key > root->key`:** Every key in `root->right` is strictly $> \text{key}$! Attach `new_root->right = root->right`, `new_root->left = root`, and set `root->right = NULL`.

```
     SPLIT-AND-SPLAY INSERTION (Case: key < root->key after splay(root, key))
     =======================================================================

             root (> key)                              new_root (key)
            /    \                                    /              \
          T_L     T_R       --- Split around --->   T_L              root (> key)
        (< key)  (> root)          new_root        (< key)          /    \
                                                                  NULL   T_R
```

---

### 5.4 `splay_delete(root, key)`: Splay-to-Root and `Join(T_L, T_R)`

Deletion in a Splay Tree is remarkably clean because splaying eliminates the messy 0-child / 1-child / 2-child case analysis of standard BST deletion:

#### Step-by-Step Algorithm (`splay_delete(root, key)`)
1. **Splay `key` to the root:** Call `root = splay(root, key)`.
2. **Check for existence:** If `root == NULL || root->key != key`, the target key is not in the tree! Simply return `root` (the closest visited node is already splayed to the root, satisfying the amortized bound).
3. **Remove the root node (`Split` into $T_L$ and $T_R$):**
   Save `T_L = root->left` (all keys $< \text{key}$) and `T_R = root->right` (all keys $> \text{key}$), and call `free(root)`.
4. **Rejoin $T_L$ and $T_R$ via `Join(T_L, T_R)`:**
   - If `T_L == NULL`, return `T_R` as the new root.
   - Otherwise, splay the **maximum element of $T_L$** to the root of $T_L$ by calling `T_L = splay(T_L, key)` (since every key in $T_L$ is $< \text{key}$, searching for `key` in $T_L$ visits and splays the rightmost maximum node of $T_L$!).
   - Because the new root of $T_L$ is the largest element in $T_L$, its right child pointer **`T_L->right` is guaranteed to be `NULL`**!
   - Attach `T_L->right = T_R` and return `T_L`!

```
   STEP 1: Splay target key (30) to root      STEP 2: Free root (30), then splay max(T_L)
           and split into T_L and T_R                 to root of T_L (its right child is NULL!)
                                              STEP 3: Attach T_R as right child of max(T_L)!

                30 (target)                        max(T_L)                     max(T_L)
               /  \                               /        \                   /        \
             T_L   T_R      ===>                T_L'       NULL     ===>     T_L'       T_R
           (< 30) (> 30)                                                               (> 30)
```

---

### 5.5 Complete C11 Implementation: Splay Tree (`splay`, `search`, `insert`, `delete`, 2D Display)

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_PATH_DEPTH 128

/* =========================================================================
 * SPLAY TREE NODE (Zero Height, Balance Factor, or Color Metadata!)
 * ========================================================================= */
typedef struct SplayNode {
    int               key;
    struct SplayNode *left;
    struct SplayNode *right;
} SplayNode;

SplayNode *splay_create_node(int key) {
    SplayNode *node = (SplayNode *)malloc(sizeof(SplayNode));
    if (node == NULL) {
        perror("malloc failed");
        exit(EXIT_FAILURE);
    }
    node->key   = key;
    node->left  = NULL;
    node->right = NULL;
    return node;
}

/* Single Right Rotation (Zig) around pivot u */
SplayNode *splay_rotate_right(SplayNode *u) {
    SplayNode *v = u->left;
    u->left      = v->right;
    v->right     = u;
    return v; /* v is the new root of this rotated subtree */
}

/* Single Left Rotation (Zag) around pivot u */
SplayNode *splay_rotate_left(SplayNode *u) {
    SplayNode *v = u->right;
    u->right     = v->left;
    v->left      = u;
    return v; /* v is the new root of this rotated subtree */
}

/* =========================================================================
 * CORE OPERATION: BOTTOM-UP SPLAYING USING AN EXPLICIT ANCESTOR STACK
 * Splays target key (or last visited non-NULL node if key is absent) to root
 * ========================================================================= */
SplayNode *splay(SplayNode *root, int key, bool verbose) {
    if (root == NULL) {
        return NULL;
    }

    SplayNode *path[MAX_PATH_DEPTH];
    int top = 0;
    SplayNode *curr = root;

    /* 1. Standard BST search path down to key (or last non-NULL node) */
    while (curr != NULL && curr->key != key) {
        if (key < curr->key) {
            if (curr->left == NULL) break; /* key not found; splay curr! */
            path[top++] = curr;
            curr = curr->left;
        } else {
            if (curr->right == NULL) break; /* key not found; splay curr! */
            path[top++] = curr;
            curr = curr->right;
        }
    }

    /* curr is now the node x to splay to the root */
    SplayNode *x = curr;

    /* 2. Bottom-Up Splay Loop: Rotate x up to the root 1 or 2 levels at a time */
    while (top > 0) {
        if (top == 1) {
            /* TERMINAL STEP: Parent p is the root (no grandparent) */
            SplayNode *p = path[--top];
            if (p->left == x) {
                if (verbose) printf("  [Splay] Zig (Terminal Right Rotate on root p=%d) for x=%d\n", p->key, x->key);
                x = splay_rotate_right(p);
            } else {
                if (verbose) printf("  [Splay] Zag (Terminal Left Rotate on root p=%d) for x=%d\n", p->key, x->key);
                x = splay_rotate_left(p);
            }
        } else {
            /* 2-LEVEL STEP: Both Parent p and Grandparent g exist */
            SplayNode *p = path[--top];
            SplayNode *g = path[--top];

            if (g->left == p && p->left == x) {
                /* Case 2a: Zig-Zig (Left-Left) -> Rotate g FIRST, then p! */
                if (verbose) printf("  [Splay] Zig-Zig (LL): Rotate grandparent g=%d right, then parent p=%d right (x=%d)\n", g->key, p->key, x->key);
                SplayNode *after_g = splay_rotate_right(g); /* after_g is p */
                x = splay_rotate_right(after_g);            /* x becomes new subtree root */
            } else if (g->right == p && p->right == x) {
                /* Case 2b: Zag-Zag (Right-Right) -> Rotate g FIRST, then p! */
                if (verbose) printf("  [Splay] Zag-Zag (RR): Rotate grandparent g=%d left, then parent p=%d left (x=%d)\n", g->key, p->key, x->key);
                SplayNode *after_g = splay_rotate_left(g);  /* after_g is p */
                x = splay_rotate_left(after_g);             /* x becomes new subtree root */
            } else if (g->left == p && p->right == x) {
                /* Case 3a: Zig-Zag (Left-Right) -> Rotate p FIRST, then g! */
                if (verbose) printf("  [Splay] Zig-Zag (LR): Rotate parent p=%d left, then grandparent g=%d right (x=%d)\n", p->key, g->key, x->key);
                g->left = splay_rotate_left(p);
                x = splay_rotate_right(g);
            } else {
                /* Case 3b: Zag-Zig (Right-Left) -> Rotate p FIRST, then g! */
                if (verbose) printf("  [Splay] Zag-Zig (RL): Rotate parent p=%d right, then grandparent g=%d left (x=%d)\n", p->key, g->key, x->key);
                g->right = splay_rotate_right(p);
                x = splay_rotate_left(g);
            }

            /* Reattach rotated subtree rooted at x to great-grandparent gg (if any) */
            if (top > 0) {
                SplayNode *gg = path[top - 1];
                if (gg->left == g) {
                    gg->left = x;
                } else {
                    gg->right = x;
                }
            }
        }
    }

    return x; /* x is now the root of the entire Splay Tree! */
}

/* =========================================================================
 * SPLAY TREE OPERATIONS: SEARCH, INSERT, DELETE
 * ========================================================================= */

/* Search for key and splay accessed (or last visited) node to root */
SplayNode *splay_search(SplayNode *root, int key, bool *out_found, bool verbose) {
    root = splay(root, key, verbose);
    *out_found = (root != NULL && root->key == key);
    return root;
}

/* Insert key via Split-and-Splay */
SplayNode *splay_insert(SplayNode *root, int key, bool verbose) {
    if (root == NULL) {
        return splay_create_node(key);
    }

    /* Splay closest existing key to root */
    root = splay(root, key, verbose);
    if (root->key == key) {
        return root; /* Duplicate key already at root */
    }

    SplayNode *new_root = splay_create_node(key);
    if (key < root->key) {
        new_root->right = root;
        new_root->left  = root->left;
        root->left      = NULL;
    } else {
        new_root->left  = root;
        new_root->right = root->right;
        root->right     = NULL;
    }
    return new_root;
}

/* Delete key via Splay-to-Root + Join(T_L, T_R) */
SplayNode *splay_delete(SplayNode *root, int key, bool verbose) {
    if (root == NULL) {
        return NULL;
    }

    /* Step 1: Splay target key to root */
    root = splay(root, key, verbose);
    if (root->key != key) {
        return root; /* Key not present; closest node remains splayed at root */
    }

    SplayNode *T_L = root->left;
    SplayNode *T_R = root->right;
    free(root);

    /* Step 2: Join T_L and T_R */
    if (T_L == NULL) {
        return T_R;
    }
    /* Splay maximum of T_L (searching for key > all elements of T_L) to root of T_L */
    T_L = splay(T_L, key, verbose);
    /* Since T_L's new root is the maximum of T_L, its right child is guaranteed NULL! */
    T_L->right = T_R;
    return T_L;
}

void splay_print_2d(const SplayNode *root, int depth) {
    if (root == NULL) return;
    splay_print_2d(root->right, depth + 1);
    for (int i = 0; i < depth; i++) printf("      ");
    printf("%d\n", root->key);
    splay_print_2d(root->left, depth + 1);
}

void splay_free(SplayNode *root) {
    if (root == NULL) return;
    splay_free(root->left);
    splay_free(root->right);
    free(root);
}

int main(void) {
    SplayNode *root = NULL;

    printf("=== 1. Building a Skewed Chain (Inserting 60, 50, 40, 30, 20, 10 in Reverse) ===\n");
    int init_keys[] = { 60, 50, 40, 30, 20, 10 };
    for (int i = 0; i < 6; i++) {
        root = splay_insert(root, init_keys[i], false);
    }
    printf("Initial Right-Skewed Splay Tree (Root = 10, Depth of 60 = 5):\n");
    splay_print_2d(root, 0);

    printf("\n=== 2. Splaying Deepest Node (Search 60) -> Watch Zag-Zag Halve the Tree Height! ===\n");
    bool found = false;
    root = splay_search(root, 60, &found, true);
    printf("Search(60) Found? %s | New Root = %d\n", found ? "YES" : "NO", root->key);
    printf("Tree After Splaying 60 (Height halved from 5 to 3!):\n");
    splay_print_2d(root, 0);

    printf("\n=== 3. Splaying Internal Node (Search 30) -> Triggers Zig-Zag ===\n");
    root = splay_search(root, 30, &found, true);
    printf("Search(30) Found? %s | New Root = %d\n", found ? "YES" : "NO", root->key);
    splay_print_2d(root, 0);

    printf("\n=== 4. Searching for Non-Existent Key (Search 25) -> Splays Last Visited Node! ===\n");
    root = splay_search(root, 25, &found, true);
    printf("Search(25) Found? %s | New Root (Last Visited) = %d\n", found ? "YES" : "NO", root->key);
    splay_print_2d(root, 0);

    printf("\n=== 5. Deleting Key 30 via Splay + Join(T_L, T_R) ===\n");
    root = splay_delete(root, 30, true);
    printf("Tree After Deleting 30 (New Root = %d):\n", root->key);
    splay_print_2d(root, 0);

    splay_free(root);
    return 0;
}
```

#### Program Output:
```text
=== 1. Building a Skewed Chain (Inserting 60, 50, 40, 30, 20, 10 in Reverse) ===
Initial Right-Skewed Splay Tree (Root = 10, Depth of 60 = 5):
                              60
                        50
                  40
            30
      20
10

=== 2. Splaying Deepest Node (Search 60) -> Watch Zag-Zag Halve the Tree Height! ===
  [Splay] Zag-Zag (RR): Rotate grandparent g=40 left, then parent p=50 left (x=60)
  [Splay] Zag-Zag (RR): Rotate grandparent g=20 left, then parent p=30 left (x=60)
  [Splay] Zag (Terminal Left Rotate on root p=10) for x=60
Search(60) Found? YES | New Root = 60
Tree After Splaying 60 (Height halved from 5 to 3!):
60
                  50
                        40
            30
                  20
      10

=== 3. Splaying Internal Node (Search 30) -> Triggers Zig-Zag ===
  [Splay] Zig-Zag (LR): Rotate parent p=10 left, then grandparent g=60 right (x=30)
Search(30) Found? YES | New Root = 30
      60
            50
                  40
30
            20
      10

=== 4. Searching for Non-Existent Key (Search 25) -> Splays Last Visited Node! ===
  [Splay] Zig-Zag (LR): Rotate parent p=10 left, then grandparent g=30 right (x=20)
Search(25) Found? NO | New Root (Last Visited) = 20
            60
                  50
                        40
      30
20
      10

=== 5. Deleting Key 30 via Splay + Join(T_L, T_R) ===
  [Splay] Zag (Terminal Left Rotate on root p=20) for x=30
Tree After Deleting 30 (New Root = 20):
      60
            50
                  40
20
      10
```

---

## 6. Complexity Analysis and Comparison: AVL Tree vs. Splay Tree

### 6.1 Worst-Case Single Operation vs. Amortized $O(\log n)$ Guarantee

> [!IMPORTANT]
> **Critical Exam Distinction — AVL Tree vs. Splay Tree Complexity:**
> - **AVL Tree:** Guarantees **$O(\log n)$ worst-case time on EVERY individual operation** because the tree height is strictly maintained below $1.44 \log_2(n + 2)$ at all times.
> - **Splay Tree:** A **single operation can take $O(n)$ time in the worst case** (because the tree height can temporarily reach $n - 1$). However, over **any sequence of $m$ operations**, the total time is bounded by $O((m + n) \log n)$, guaranteeing **$O(\log n)$ amortized time per operation**!

Furthermore, Splay Trees satisfy remarkable distribution-sensitive optimality theorems that AVL trees cannot match without modifications:
1. **Static Optimality Theorem:** Over any sequence of accesses, a Splay Tree runs within a constant factor of the optimal static binary search tree built with advance knowledge of all access frequencies.
2. **Working-Set Property:** If only $k \ll n$ distinct keys are accessed during a phase of computation, each access takes $O(\log k)$ amortized time rather than $O(\log n)$.
3. **Sequential Traversal Theorem (Tarjan, 1985):** Accessing all $n$ keys of an arbitrary $n$-node Splay Tree in sorted order $k_1 < k_2 < \dots < k_n$ takes **$O(n)$ total time** ($O(1)$ amortized per access!).

### 6.2 Comprehensive Comparison Table: AVL Tree vs. Splay Tree vs. Unbalanced BST

| Dimension | Unbalanced BST | **AVL Tree** | **Splay Tree** |
| :--- | :--- | :--- | :--- |
| **Balance Strategy** | None | Strict local height balance ($|\text{BF}| \le 1$) | Self-adjusting splay to root on every access |
| **Extra Metadata per Node** | `0` bytes | `int height` (4 bytes) or 2-bit BF | **`0` bytes** |
| **Worst-Case Tree Height ($h$)** | $n - 1$ ($O(n)$) | **$< 1.44 \log_2(n + 2)$ ($O(\log n)$)** | $n - 1$ ($O(n)$) |
| **Single `search(k)` Worst Case** | $O(n)$ | **$O(\log n)$ guaranteed** | $O(n)$ worst-case; **$O(\log n)$ amortized** |
| **Single `insert(k)` Worst Case** | $O(n)$ | **$O(\log n)$ guaranteed** | $O(n)$ worst-case; **$O(\log n)$ amortized** |
| **Single `delete(k)` Worst Case** | $O(n)$ | **$O(\log n)$ guaranteed** | $O(n)$ worst-case; **$O(\log n)$ amortized** |
| **Repeated Access to Same Key** | $O(h)$ (up to $O(n)$) | $O(\log n)$ (if key is a leaf) | **$O(1)$** (key is already at the root!) |
| **Does `search(k)` Modify Tree?** | No (read-only) | No (read-only) | **Yes! (Mutates pointers on every read)** |
| **Multi-Threaded Read Concurrency** | Lock-free concurrent reads | Lock-free concurrent reads | **Requires write-locking / synchronization on reads!** |
| **Hard Real-Time Suitability** | Unsuitable ($O(n)$ spikes) | **Ideal** (predictable $O(\log n)$ ceiling) | Unsuitable for hard real-time ($O(n)$ single spike) |

---

## 7. Common Errors, Pitfalls, and Edge Cases

### 7.1 Pitfall 1: Rotating Parent $p$ Before Grandparent $g$ in a Zig-Zig Step

This is the single most common coding and exam error on Splay Trees. In a homogeneous **Zig-Zig** (LL) or **Zag-Zag** (RR) step, rotating $p$ before $g$ turns splaying into naive `Move-to-Root` single rotations, destroying the $O(\log n)$ amortized complexity!

```c
/* BUG: Rotating parent p BEFORE grandparent g in a Zig-Zig (LL) step! */
if (g->left == p && p->left == x) {
    g->left = splay_rotate_right(p); /* WRONG! This is Move-to-Root, NOT Zig-Zig! */
    x       = splay_rotate_right(g);
}

/* FIX: In Zig-Zig (LL), rotate GRANDPARENT g first, then PARENT p! */
if (g->left == p && p->left == x) {
    SplayNode *after_g = splay_rotate_right(g); /* 1. Rotate grandparent g right (returns p) */
    x                  = splay_rotate_right(after_g); /* 2. Rotate parent p right (returns x) */
}
```

### 7.2 Pitfall 2: Failing to Splay the Last Visited Node on an Unsuccessful Search

If `splay_search(root, key)` returns without splaying when `key` is not found (or when `splay_delete` searches for a missing key), repeated unsuccessful searches along a deep path will take $\Theta(n)$ time every single call!

```c
/* BUG: Stopping and returning NULL without splaying when key is missing */
if (curr == NULL) {
    return root; /* Tree remains skewed! Repeated misses take O(n) forever! */
}

/* FIX: Stop at the last non-NULL node before NULL and splay it to the root! */
if (key < curr->key) {
    if (curr->left == NULL) break; /* curr is the last visited node -> splay curr! */
    path[top++] = curr;
    curr = curr->left;
}
```

### 7.3 Pitfall 3: Using `Zig` Inside the Loop Instead of Only as the Terminal Root Step

A single rotation (`Zig` or `Zag`) is **only** permitted when $x$'s parent $p$ is the **root of the entire tree** (`top == 1`). Always consume ancestors in pairs `(p, g)` from the bottom up so any single remaining ancestor is guaranteed to be the root.

### 7.4 Pitfall 4: Forgetting to Update the Caller's `root` Pointer After `splay_search`

In an ordinary BST or AVL tree, `search(root, key)` does not modify the tree structure, so `root` never changes. In a Splay Tree, **every search changes the root of the tree**! Callers must always reassign `root = splay_search(root, key, &found, verbose);`.

### 7.5 Boundary Conditions Checklist

- [ ] **Empty Tree (`root == NULL`):** Do `splay`, `splay_search`, and `splay_delete` return `NULL` safely?
- [ ] **Target Key Already at Root (`root->key == key`):** Does `splay` perform `0` rotations (`top == 0`) and return `root` immediately in $O(1)$ time?
- [ ] **Deleting a Node When $T_L == \text{NULL}$ (Minimum Element of Tree):** When `root->left == NULL` after splaying the target to the root, does `splay_delete` return `T_R` directly without attempting to splay an empty $T_L$?
- [ ] **Reattaching to Great-Grandparent (`top > 0` after popping $p$ and $g$):** After a `Zig-Zig` or `Zig-Zag` replaces subtree root $g$ with $x$, does the loop update `gg->left = x` or `gg->right = x` on the great-grandparent `gg = path[top - 1]`?

---

## 8. Summary and Revision Checklist

### 8.1 Key Takeaways and Core Invariants

1. **Self-Adjusting Philosophy:** A Splay Tree stores **zero balance metadata** per node and moves every accessed, inserted, or deleted node (or last visited node on a miss) to the root via `splay(root, key)`.
2. **The Three Splay Steps:**
   - **Zig / Zag (Terminal Step):** $x$'s parent $p$ is the root $\implies$ single rotation on $p$.
   - **Zig-Zig / Zag-Zag (LL / RR):** $x$ and $p$ are on the same side of $g$ $\implies$ **rotate grandparent $g$ FIRST, then parent $p$**.
   - **Zig-Zag / Zag-Zig (LR / RL):** $x$ and $p$ are on opposite sides of $g$ $\implies$ **rotate parent $p$ FIRST, then grandparent $g$**.
3. **Join-Based Deletion:** Splay target `key` to root, remove root leaving $T_L$ and $T_R$, splay $\max(T_L)$ to the root of $T_L$ (so its right child is `NULL`), and set `T_L->right = T_R`.
4. **AVL vs. Splay Complexity:** AVL trees guarantee **worst-case $O(\log n)$ per individual operation**; Splay trees have $O(n)$ worst-case single-operation time but guarantee **$O(\log n)$ amortized time per operation** and $O(1)$ repeated access to hot keys.

### 8.2 Self-Test Revision Questions

1. **Rotation Order Question:** Suppose node `10` is the left child of `20`, and `20` is the left child of root `30`. Trace the exact sequence of rotations during `splay(root, 10)` and draw the resulting tree. How does it differ if you mistakenly rotated `20` before `30`?
   *(Answer: Correct `Zig-Zig` rotates `30` right first (making `20` root with children `10` and `30`), then rotates `20` right (making `10` root with right child `20` and `20->right = 30`). In this 3-node case without subtrees the final chain is `10 -> 20 -> 30`, but when `30` has a parent `40` (`40 -> 30 -> 20 -> 10`), `Zig-Zig` makes `30` the parent of `40` and `20`, reducing tree height from `3` to `2`!).*
2. **Unsuccessful Search Question:** In the Splay Tree from our C11 program (`root = 30`, left subtree `10 -> right = 20`, right subtree `60 -> 50 -> 40`), why did `splay_search(root, 25)` splay node `20` to the root?
   *(Answer: Searching for `25` visits `30` (go left), `10` (go right), and `20` (go right $\to \text{NULL}$). Because `20` is the last non-`NULL` node visited before hitting `NULL`, `splay` brings `20` to the root via a `Zig-Zag (LR)` step).*
3. **Architecture Choice Question:** You are designing an in-memory lookup table read concurrently by 64 CPU threads without locks, and latency must never exceed 5 microseconds on any single lookup. Should you choose an AVL Tree or a Splay Tree? Why?
   *(Answer: Choose an **AVL Tree**, for two reasons: (1) AVL `search` is read-only and thread-safe for concurrent readers without locks, whereas Splay `search` mutates pointers on every read; (2) AVL guarantees strict $O(\log n)$ worst-case latency on every single lookup, whereas a Splay Tree can suffer an $O(n)$ single-operation latency spike).*

### 8.3 Navigation Links

- **Previous Topic:** [05-AVL-Trees.md](./05-AVL-Trees.md) — Height-Balanced AVL Binary Search Trees and Rotations
- **Current File:** `06-Splay-Trees.md` (Unit 3 — Topic 6)
- **Next Topic:** [07-Graph-Fundamentals-and-Representations.md](./07-Graph-Fundamentals-and-Representations.md) — Graph Fundamentals and Representations
