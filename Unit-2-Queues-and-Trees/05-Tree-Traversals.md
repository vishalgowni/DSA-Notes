# Unit 2: Tree Traversals (Preorder, Inorder, Postorder, Level-Order, and N-Ary)

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [What is Tree Traversal and Why is it Needed?](#11-what-is-tree-traversal-and-why-is-it-needed)
   - 1.2 [Relationship to Prior Concepts](#12-relationship-to-prior-concepts)
   - 1.3 [Learning Objectives](#13-learning-objectives)
2. [Definitions, Terminology, and Mathematical Foundations](#2-definitions-terminology-and-mathematical-foundations)
   - 2.1 [Formal Definition and Classification of Tree Traversals](#21-formal-definition-and-classification-of-tree-traversals)
   - 2.2 [The Unified Euler Tour (Walk-Around) Intuition](#22-the-unified-euler-tour-walk-around-intuition)
   - 2.3 [Mathematical Properties, Theorems, and Proofs](#23-mathematical-properties-theorems-and-proofs)
3. [Representation and Memory Layout in C for Traversals](#3-representation-and-memory-layout-in-c-for-traversals)
   - 3.1 [Implicit OS Call Stack vs. Explicit `NodeStack` and `NodeQueue`](#31-implicit-os-call-stack-vs-explicit-nodestack-and-nodequeue)
   - 3.2 [ASCII Memory Layout of Iterative Traversal Structures](#32-ascii-memory-layout-of-iterative-traversal-structures)
4. [Operations and Traversal Algorithms](#4-operations-and-traversal-algorithms)
   - 4.1 [Preorder Traversal ($\text{Root} \to \text{Left} \to \text{Right}$) — Recursive & Iterative](#41-preorder-traversal-textroot-to-textleft-to-textright--recursive--iterative)
   - 4.2 [Inorder Traversal ($\text{Left} \to \text{Root} \to \text{Right}$) — Recursive & Iterative](#42-inorder-traversal-textleft-to-textroot-to-textright--recursive--iterative)
   - 4.3 [Postorder Traversal ($\text{Left} \to \text{Right} \to \text{Root}$) — Recursive, Two-Stack, & Single-Stack](#43-postorder-traversal-textleft-to-textright-to-textroot--recursive-two-stack--single-stack)
   - 4.4 [Level-Order (Breadth-First) Traversal Using a FIFO Queue](#44-level-order-breadth-first-traversal-using-a-fifo-queue)
   - 4.5 [Unique Binary Tree Reconstruction from Traversal Sequences](#45-unique-binary-tree-reconstruction-from-traversal-sequences)
   - 4.6 [General N-Ary Tree Traversals and the LCRS Isomorphism Theorem](#46-general-n-ary-tree-traversals-and-the-lcrs-isomorphism-theorem)
   - 4.7 [Complete C11 Implementation and Verified Execution Trace](#47-complete-c11-implementation-and-verified-execution-trace)
5. [Comparison Tables and Complexity Summary](#5-comparison-tables-and-complexity-summary)
   - 5.1 [Master Comparison of Binary Tree Traversals](#51-master-comparison-of-binary-tree-traversals)
   - 5.2 [N-Ary Tree Traversal vs. Binary Tree Traversal](#52-n-ary-tree-traversal-vs-binary-tree-traversal)
   - 5.3 [Time and Space Complexity Summary](#53-time-and-space-complexity-summary)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Concrete Bug vs. Fix Examples in C](#61-concrete-bug-vs-fix-examples-in-c)
   - 6.2 [Boundary Conditions Checklist](#62-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)

---

## 1. Introduction and Learning Objectives

### 1.1 What is Tree Traversal and Why is it Needed?

In linear data structures (arrays, linked lists, stacks, queues), there is only one natural way to iterate through all elements: sequentially from front to back (or back to front). Because a **tree** is a non-linear, branching hierarchy where each node may have multiple children, there is no single obvious "next" node.

**Tree Traversal** is the systematic algorithmic process of visiting (inspecting, printing, or updating) **every node in a tree exactly once** in a well-defined deterministic sequence. Linearizing a tree via traversal solves fundamental problems across computer science:
- **Compiler Expression Evaluation & Code Generation:** Generating Prefix (Polish), Infix, or Postfix (Reverse Polish) representations of arithmetic syntax trees.
- **Ordered Retrieval in Search Trees:** Extracting keys from a Binary Search Tree (BST) in sorted ascending order in $O(n)$ time.
- **Safe Heap Deallocation & Bottom-Up Aggregation:** Computing directory sizes, subtree heights, or calling `free()` on every child before freeing its parent.
- **Tree Serialization & Cloning:** Saving a tree to disk or network and reconstructing an exact copy in memory.

### 1.2 Relationship to Prior Concepts

- **Stacks and Recursion (Unit 1):** Depth-First Traversals (**Preorder, Inorder, Postorder**) must remember ancestor nodes while exploring deep subtrees. Recursive implementations use the **OS call stack** implicitly; iterative implementations use an explicit **LIFO Stack** of `BTNode *` pointers.
- **Queues (Unit 2A):** Breadth-First (**Level-Order**) traversal visits nodes level by level ($d = 0, 1, \dots, h$) and requires a **FIFO Queue** of `BTNode *` pointers.
- **LCRS Binary Representation (Unit 2C-04):** Every $N$-ary tree (or forest) maps to a unique LCRS Binary Tree, creating a remarkable algebraic correspondence between $N$-ary traversals and binary traversals.

### 1.3 Learning Objectives

After completing this chapter, you will be able to:
1. Classify tree traversals into **Depth-First (Preorder, Inorder, Postorder)** and **Breadth-First (Level-Order)** and trace all four on any tree using the **Euler Tour** walk-around method.
2. Write both **recursive** and **iterative (explicit-stack)** algorithms and C11 implementations for **Preorder**, **Inorder**, and **Postorder** (using both the **Two-Stack** and **Single-Stack `last_visited`** methods).
3. Explain why Iterative Preorder pushes the **right child before the left child**, why Inorder sorts a **BST**, and why Postorder is mandatory for **`free()`** cleanup.
4. Reconstruct a unique Binary Tree from **`(Inorder + Preorder)`** or **`(Inorder + Postorder)`**, and prove why **`(Preorder + Postorder)`** is ambiguous for general binary trees.
5. Distinguish **N-ary Tree traversals** from **Binary Tree traversals** and prove that $\text{Preorder}(T) \equiv \text{Preorder}(B(T))$ and $\text{Postorder}(T) \equiv \text{Inorder}(B(T))$.

---

## 2. Definitions, Terminology, and Mathematical Foundations

### 2.1 Formal Definition and Classification of Tree Traversals

> **Definition 1 (Tree Traversal):**
> Given a rooted tree $T = (V, E)$ with $n = |V|$ vertices, a **traversal** is a bijection $\pi: \{1, 2, \dots, n\} \to V$ that defines a linear sequence $(\pi(1), \pi(2), \dots, \pi(n))$ in which every vertex $v \in V$ is visited **exactly once**.

For a **Binary Tree** where each node consists of a **Root / Node ($\text{N}$)**, a **Left Subtree ($\text{L}$)**, and a **Right Subtree ($\text{R}$)**, keeping the convention that the left subtree is always explored before the right subtree ($\text{L}$ before $\text{R}$) yields three canonical **Depth-First** orders based on *when* the root node $\text{N}$ is visited:

```text
                       +----------------------------+
                       |      TREE TRAVERSALS       |
                       +-------------+--------------+
                                     |
                 +-------------------+-------------------+
                 |                                       |
                 v                                       v
     +-----------------------+               +-----------------------+
     |  Depth-First (DFS)    |               |  Breadth-First (BFS)  |
     | (Stack / Recursion)   |               |     (FIFO Queue)      |
     +-----------+-----------+               +-----------+-----------+
                 |                                       |
     +-----------+-----------+                           v
     |           |           |               +-----------------------+
     v           v           v               |     4. Level-Order    |
+----------+ +---------+ +-----------+       |  Visit d = 0, 1, .. h |
|1.Preorder| |2.Inorder| |3.Postorder|       |   Left-to-Right per   |
|  N -> L  | |  L -> N | |  L -> R   |       |        level          |
|    -> R  | |    -> R | |    -> N   |       +-----------------------+
+----------+ +---------+ +-----------+
```

| Traversal Name | Mnemonic | Visiting Rule at Each Subtree | Expression Notation Produced |
| :--- | :---: | :--- | :--- |
| **Preorder** | $\text{NLR}$ | **1.** Visit **N**ode $\to$ **2.** Traverse **L**eft $\to$ **3.** Traverse **R**ight | **Prefix (Polish):** `+ A * B C` |
| **Inorder (Symmetric)** | $\text{LNR}$ | **1.** Traverse **L**eft $\to$ **2.** Visit **N**ode $\to$ **3.** Traverse **R**ight | **Infix:** `A + B * C` |
| **Postorder** | $\text{LRN}$ | **1.** Traverse **L**eft $\to$ **2.** Traverse **R**ight $\to$ **3.** Visit **N**ode | **Postfix (RPN):** `A B C * +` |
| **Level-Order (BFS)** | Level-by-Level | Visit all nodes at depth $d = 0$, then $d = 1$, $\dots$, up to $d = h$ (left to right) | Breadth-first serialization |

---

### 2.2 The Unified Euler Tour (Walk-Around) Intuition

Every recursive DFS traversal follows the **exact same continuous path** around the outer perimeter of the tree, starting at the root and hugging the edges counter-clockwise until returning to the root. This perimeter walk is called an **Euler Tour** of the binary tree (replacing missing children with small `NULL` stubs).

During an Euler Tour, we pass by every node $u$ **three times**:
1. **First Pass (On the LEFT of $u$, moving downward):** Recording nodes here yields **Preorder ($\text{NLR}$)**.
2. **Second Pass (UNDERNEATH $u$, after returning from its left subtree):** Recording nodes here yields **Inorder ($\text{LNR}$)**.
3. **Third Pass (On the RIGHT of $u$, after returning from its right subtree):** Recording nodes here yields **Postorder ($\text{LRN}$)**.

```text
Euler Tour Perimeter Walk Around our Canonical 6-Node Binary Tree:

                  (1) Preorder: Pass on LEFT going down
                        |
                        v
                     *[ A ]+  <-- (3) Postorder: Pass on RIGHT going up
                     /  ^  \
                   /   (2)   \
                 v   Inorder   \
              *[ B ]+          *[ C ]+
              /  ^  \          /  ^  \
            /   (2)   \     (NULL)    \
          v            v               v
       *[ D ]+       *[ E ]+         *[ F ]+
          ^             ^               ^
         (2)           (2)             (2)

Legend:
  * (Left side of node)   -> Visit for PREORDER  : A, B, D, E, C, F
  ^ (Underneath node)     -> Visit for INORDER   : D, B, E, A, C, F
  + (Right side of node)  -> Visit for POSTORDER : D, E, B, F, C, A
```

---

### 2.3 Mathematical Properties, Theorems, and Proofs

#### Theorem 1: Time and Auxiliary Space Complexity of DFS Traversals
> **Statement:** For any binary tree with $n$ nodes and height $h$ ($\lfloor \log_2 n \rfloor \le h \le n - 1$), both recursive and iterative (single-stack) Preorder, Inorder, and Postorder traversals run in:
> $$\text{Time Complexity} = \Theta(n), \qquad \text{Auxiliary Space Complexity} = \Theta(h)$$

**Proof:**
- **Time ($\Theta(n)$):** In the recursive formulation, each of the $n$ non-null nodes makes exactly $2$ recursive calls (one for `left`, one for `right`), plus $1$ initial call for the root. Out of $2n + 1$ total calls, $n$ calls visit a real node in $O(1)$ time and $n + 1$ calls hit a `NULL` pointer and return immediately in $O(1)$ time. Thus total work is $c_1 n + c_2(n + 1) = \Theta(n)$. In the iterative formulation, each of the $n$ nodes is pushed onto the stack once and popped once ($2n$ stack operations at $O(1)$ each $= \Theta(n)$).
- **Space ($\Theta(h)$):** At any instant during DFS, the call stack (or explicit stack) contains only nodes along the active root-to-current-node path (plus at most one right-sibling per ancestor in Iterative Preorder). Since the longest root-to-leaf path has $h + 1$ nodes, maximum stack depth is $\Theta(h)$, which is $\Theta(\log n)$ for balanced trees and $\Theta(n)$ for skewed trees. $\blacksquare$

---

#### Theorem 2: Binary Search Tree (BST) Inorder Sorting Theorem
> **Statement:** Performing an **Inorder ($\text{LNR}$) traversal** on a Binary Search Tree $T$ with $n$ distinct keys outputs the keys in **strictly increasing (sorted) order**.

**Proof (By Strong Structural Induction on $n$):**
- **Base Cases ($n = 0$ and $n = 1$):** An empty tree outputs an empty sequence, and a 1-node tree outputs its single key; both are trivially sorted.
- **Inductive Hypothesis:** Assume that for every BST with $k < n$ nodes, Inorder traversal produces its keys in strictly increasing order.
- **Inductive Step ($n$ nodes):** Let $T$ have root $r$, left subtree $T_L$ (of size $n_L < n$), and right subtree $T_R$ (of size $n_R < n$).
  1. Inorder traversal of $T$ outputs: $[\text{Inorder}(T_L)],\ \text{key}(r),\ [\text{Inorder}(T_R)]$.
  2. By the Inductive Hypothesis, $\text{Inorder}(T_L)$ is sorted in strictly increasing order, and $\text{Inorder}(T_R)$ is sorted in strictly increasing order.
  3. By the **BST Ordering Invariant**, every key in $T_L$ is strictly less than $\text{key}(r)$, and $\text{key}(r)$ is strictly less than every key in $T_R$.
  4. Therefore, concatenating $[\text{Inorder}(T_L)]$, $\text{key}(r)$, and $[\text{Inorder}(T_R)]$ produces a globally strictly increasing sequence of all $n$ keys. $\blacksquare$

---

#### Theorem 3: Unique Binary Tree Reconstruction Theorem
> **Statement:**
> 1. A binary tree with $n$ distinct node labels is **uniquely determined** given its **Inorder** sequence together with **either** its **Preorder** sequence **or** its **Postorder** sequence.
> 2. A general binary tree is **NOT uniquely determined** by its **Preorder** and **Postorder** sequences alone (unless the tree is guaranteed to be a **Full / Strict Binary Tree** where every internal node has $2$ children).

**Proof:**
1. **Sufficiency of (Inorder + Preorder / Postorder) by Induction on $n$:**
   - For $n \le 1$, the tree is unique.
   - For $n > 1$, the **first** element of $\text{Preorder}(T)$ (or the **last** element of $\text{Postorder}(T)$) is guaranteed to be the **root** $r$.
   - Because all node labels are distinct, locating $r$ at index $i$ in $\text{Inorder}(T)$ uniquely partitions $\text{Inorder}(T)$ into:
     - Elements to the **left** of $r$ ($\text{indices } 0 \dots i - 1$): the exact set of $i$ nodes in the **left subtree** $T_L$.
     - Elements to the **right** of $r$ ($\text{indices } i + 1 \dots n - 1$): the exact set of $n - 1 - i$ nodes in the **right subtree** $T_R$.
   - Knowing the exact size $i$ of $T_L$ allows us to partition the remaining elements of $\text{Preorder}(T)$ (indices $1 \dots i$ for $T_L$, and $i + 1 \dots n - 1$ for $T_R$). By the inductive hypothesis, $T_L$ and $T_R$ are uniquely reconstructed.
2. **Ambiguity of (Preorder + Postorder) Without Inorder (Counterexample):**
   Consider the two distinct 2-node binary trees below:
   ```text
       Tree B1 (B is LEFT child)          Tree B2 (B is RIGHT child)
                [ A ]                              [ A ]
                /                                    \
              [ B ]                                  [ B ]
       Preorder(B1)  = [A, B]             Preorder(B2)  = [A, B]
       Postorder(B1) = [B, A]             Postorder(B2) = [B, A]
       Inorder(B1)   = [B, A]             Inorder(B2)   = [A, B]  <-- Only Inorder distinguishes them!
   ```
   Both $B_1$ and $B_2$ have the **exact same Preorder `[A, B]` and Postorder `[B, A]`**! Without Inorder, whenever a parent has only a single child, it is impossible to tell whether that child is a **left child** or a **right child**. $\blacksquare$

---

## 3. Representation and Memory Layout in C for Traversals

### 3.1 Implicit OS Call Stack vs. Explicit `NodeStack` and `NodeQueue`

When we write `preorder_recursive(root)`, the C compiler allocates a **stack frame** on the operating system call stack for every recursive call. In embedded systems or when traversing deep degenerate trees ($n = 10^5$), recursive calls can exhaust the OS stack (`Stack Overflow`).

By maintaining an explicit **`NodeStack`** (an array or linked list of `const BTNode *` pointers) or **`NodeQueue`** (for Level-Order), we store only an 8-byte pointer per active node and gain complete control over memory allocation:

```c
typedef struct BTNode {
    char data;
    struct BTNode *left;
    struct BTNode *right;
} BTNode;

/* Explicit LIFO Stack of BTNode pointers for Iterative DFS Traversals */
typedef struct {
    const BTNode *items[MAX_CAP];
    int top;
} NodeStack;

/* Explicit Circular FIFO Queue of BTNode pointers for Level-Order (BFS) */
typedef struct {
    const BTNode *items[MAX_CAP];
    int front;
    int rear;
    int size;
} NodeQueue;
```

### 3.2 ASCII Memory Layout of Iterative Traversal Structures

```text
Heap-Allocated Binary Tree Nodes:                 Explicit NodeStack (LIFO):
          0x100 [ 'A' | 0x200 | 0x300 ]           Index    BTNode* Pointer
                        /           \             +-----+-------------------+
                       v             v            |  2  |       NULL        |
      0x200 [ 'B' | 0x400 | 0x500 ]  0x300 ['C']  +-----+-------------------+
                                                  |  1  | 0x200 (&Node 'B') | <-- top = 1
                                                  +-----+-------------------+
                                                  |  0  | 0x300 (&Node 'C') |
                                                  +-----+-------------------+
                                                  (Right child 'C' pushed first,
                                                   Left child 'B' on top to pop next!)
```

---

## 4. Operations and Traversal Algorithms

Throughout Sections 4.1–4.4, we will trace all recursive and iterative algorithms on the following **Canonical 6-Node Binary Tree** $T_6$:

```text
                [ A ]          Level / Depth d = 0
                /   \
              [B]   [C]        Level / Depth d = 1
              / \     \
            [D] [E]   [F]      Level / Depth d = 2
            (C has NO left child; F is C's RIGHT child)
```

---

### 4.1 Preorder Traversal ($\text{Root} \to \text{Left} \to \text{Right}$) — Recursive & Iterative

#### Purpose & Preconditions
- **Purpose:** Visit each parent node **before** visiting either of its subtrees.
- **Preconditions:** `root` points to a valid `BTNode` or `NULL`.

#### Step-by-Step Iterative Preorder Algorithm (Using an Explicit Stack)
1. If `root == NULL`, return immediately.
2. Initialize an empty stack `S` and `push(S, root)`.
3. While `S` is not empty:
   - **Step 3a:** `curr = pop(S)` and **visit `curr`** (print `curr->data`).
   - **Step 3b:** If `curr->right != NULL`, `push(S, curr->right)`.
   - **Step 3c:** If `curr->left != NULL`, `push(S, curr->left)`.

> **Why Push `curr->right` BEFORE `curr->left`?**
> Because a Stack is a **LIFO (Last-In, First-Out)** structure! We want to visit `curr->left` immediately after `curr`, and defer `curr->right` until the entire left subtree finishes. Pushing `curr->right` first places it at the bottom, and pushing `curr->left` second places `curr->left` at the **top of the stack**, so `curr->left` is popped next!

#### Pseudocode (Recursive & Iterative Preorder)

```text
ALGORITHM PreorderRecursive(u)
1.  if u == NULL then return
2.  Visit(u)
3.  PreorderRecursive(u.left)
4.  PreorderRecursive(u.right)

ALGORITHM PreorderIterative(root)
1.  if root == NULL then return
2.  S <- EmptyStack()
3.  Push(S, root)
4.  while not IsEmpty(S) do
5.      curr <- Pop(S)
6.      Visit(curr)
7.      if curr.right != NULL then Push(S, curr.right)   // Push RIGHT first!
8.      if curr.left  != NULL then Push(S, curr.left)    // Push LEFT second (on top)!
```

#### Step-by-Step Stack Trace Table (Iterative Preorder on $T_6$)

| Step | Action Performed | Popped `curr` | Pushed Right | Pushed Left | Stack `S` (Bottom $\to$ Top) | Output So Far |
| :---: | :--- | :---: | :---: | :---: | :--- | :--- |
| **Init** | Push root `A` | — | — | — | `[A]` | *(empty)* |
| **1** | Pop `A`, visit `A`, push `C`, push `B` | `A` | `C` | `B` | `[C, B]` | `A` |
| **2** | Pop `B`, visit `B`, push `E`, push `D` | `B` | `E` | `D` | `[C, E, D]` | `A B` |
| **3** | Pop `D`, visit `D` (leaf: no pushes) | `D` | — | — | `[C, E]` | `A B D` |
| **4** | Pop `E`, visit `E` (leaf: no pushes) | `E` | — | — | `[C]` | `A B D E` |
| **5** | Pop `C`, visit `C`, push `F` (`left` is NULL) | `C` | `F` | — | `[F]` | `A B D E C` |
| **6** | Pop `F`, visit `F` (leaf: no pushes) | `F` | — | — | `[]` *(empty — terminate)* | **`A B D E C F`** |

#### Key Applications of Preorder Traversal
- **Tree Cloning / Deep Copying:** You must allocate the parent node before you can attach its left and right child clones.
- **Prefix (Polish) Expression Generation:** Visiting an expression tree in Preorder outputs `+ a * b c`, which requires zero parentheses to be unambiguous.
- **Directory Tree Listing (`tree` command):** Prints the parent folder name before indenting its sub-items.

---

### 4.2 Inorder Traversal ($\text{Left} \to \text{Root} \to \text{Right}$) — Recursive & Iterative

#### Purpose & Preconditions
- **Purpose:** Visit the entire left subtree first, then the root node, and finally the entire right subtree.
- **Preconditions:** `root` points to a valid `BTNode` or `NULL`.

#### Step-by-Step Iterative Inorder Algorithm (Using an Explicit Stack)
Unlike Preorder, we **cannot** immediately visit the root when we first see it; we must defer visiting each node until its entire left subtree has been processed.
1. Initialize an empty stack `S` and a traversal pointer `curr = root`.
2. Loop while `curr != NULL` **or** `!IsEmpty(S)`:
   - **Phase A (Plunge Down Left Spine):** While `curr != NULL`, `push(S, curr)` and advance `curr = curr->left`.
   - **Phase B (Backtrack, Visit, and Pivot Right):** Once `curr == NULL` (we hit the bottom of a left spine), set `curr = pop(S)`, **visit `curr`**, and then set `curr = curr->right` to explore `curr`'s right subtree on the next iteration.

#### Pseudocode (Recursive & Iterative Inorder)

```text
ALGORITHM InorderRecursive(u)
1.  if u == NULL then return
2.  InorderRecursive(u.left)
3.  Visit(u)
4.  InorderRecursive(u.right)

ALGORITHM InorderIterative(root)
1.  S <- EmptyStack()
2.  curr <- root
3.  while curr != NULL or not IsEmpty(S) do
4.      while curr != NULL do
5.          Push(S, curr)
6.          curr <- curr.left
7.      curr <- Pop(S)
8.      Visit(curr)
9.      curr <- curr.right
```

#### Step-by-Step Stack Trace Table (Iterative Inorder on $T_6$)

| Iter | `curr` at Start | Inner `while (curr != NULL)` Pushes | Stack Before Pop | Popped & Visited | `curr = popped->right` | Output So Far |
| :---: | :---: | :--- | :--- | :---: | :---: | :--- |
| **1** | `A` | Push `A` (`curr=B`), Push `B` (`curr=D`), Push `D` (`curr=NULL`) | `[A, B, D]` | `D` | `D->right = NULL` | `D` |
| **2** | `NULL` | *(none)* | `[A, B]` | `B` | `B->right = E` | `D B` |
| **3** | `E` | Push `E` (`curr=E->left=NULL`) | `[A, E]` | `E` | `E->right = NULL` | `D B E` |
| **4** | `NULL` | *(none)* | `[A]` | `A` | `A->right = C` | `D B E A` |
| **5** | `C` | Push `C` (`curr=C->left=NULL`) | `[C]` | `C` | `C->right = F` | `D B E A C` |
| **6** | `F` | Push `F` (`curr=F->left=NULL`) | `[F]` | `F` | `F->right = NULL` | **`D B E A C F`** |
| **End** | `NULL` | `curr == NULL` and `S` is `[]` $\implies$ **Terminate** | `[]` | — | — | **`D B E A C F`** |

---

### 4.3 Postorder Traversal ($\text{Left} \to \text{Right} \to \text{Root}$) — Recursive, Two-Stack, & Single-Stack

#### Purpose & Preconditions
- **Purpose:** Visit a node **only after** both its left and right subtrees have been completely visited.
- **Why Iterative Postorder is the Trickiest DFS Traversal:** In Inorder, when a node is popped from the stack, we know its left subtree is finished, so we immediately visit it and move to `right`. In Postorder, when we return to a parent node on the stack from its left child, we **still cannot pop and visit the parent** if it has an unvisited right child! We must distinguish whether we are returning from the **left subtree** or from the **right subtree**.

There are two classic iterative approaches to solve this:

---

#### Method 1: Iterative Postorder Using Two Stacks ($O(n)$ Auxiliary Space)
- **Key Insight:** Notice that Postorder is $\text{Left} \to \text{Right} \to \text{Root}$. If we reverse this sequence from back to front, we get:
  $$\text{Reverse}(\text{Left} \to \text{Right} \to \text{Root}) \;=\; \text{Root} \to \text{Right} \to \text{Left}$$
  And $\text{Root} \to \text{Right} \to \text{Left}$ is simply **Preorder with Left and Right swapped**!
- **Algorithm:**
  1. Push `root` onto `S1`.
  2. While `S1` is not empty:
     - Pop `curr` from `S1` and push `curr` onto `S2`.
     - If `curr->left != NULL`, push `curr->left` onto `S1`.
     - If `curr->right != NULL`, push `curr->right` onto `S1` (so `right` is on top of `S1` and popped before `left`!).
  3. Once `S1` is empty, `S2` holds the nodes in $\text{Root} \to \text{Right} \to \text{Left}$ order from bottom to top. Popping and printing all elements from `S2` reverses them into exact $\text{Left} \to \text{Right} \to \text{Root}$ **Postorder**!

##### Step-by-Step Two-Stack Trace Table (on $T_6$)

| Step | Popped from `S1` | Pushed to `S1` (`L` then `R`) | Stack `S1` (Bottom $\to$ Top) | Stack `S2` (Bottom $\to$ Top) |
| :---: | :---: | :--- | :--- | :--- |
| **Init** | — | — | `[A]` | `[]` |
| **1** | `A` | `B`, `C` | `[B, C]` | `[A]` |
| **2** | `C` | `F` (`C->left` is NULL) | `[B, F]` | `[A, C]` |
| **3** | `F` | *(none)* | `[B]` | `[A, C, F]` |
| **4** | `B` | `D`, `E` | `[D, E]` | `[A, C, F, B]` |
| **5** | `E` | *(none)* | `[D]` | `[A, C, F, B, E]` |
| **6** | `D` | *(none)* | `[]` | `[A, C, F, B, E, D]` |
| **Drain `S2`** | Pop `S2` until empty | — | `[]` | Output: **`D E B F C A`** |

---

#### Method 2: Iterative Postorder Using a Single Stack + `last_visited` Pointer ($O(h)$ Auxiliary Space)
- **Key Insight:** To avoid the $O(n)$ memory of a second stack `S2`, we keep a single stack `S` and a pointer `last_visited` initialized to `NULL`. When `curr == NULL`, we **peek** (do not pop yet!) at `peek_node = Top(S)`:
  - If `peek_node->right != NULL` **AND** `last_visited != peek_node->right`, it means the right subtree of `peek_node` exists and has **not** been visited yet. So we set `curr = peek_node->right` to plunge into the right subtree.
  - Otherwise (either `peek_node->right == NULL`, or `last_visited == peek_node->right` meaning we just finished the right subtree!), both subtrees of `peek_node` are complete. We **visit `peek_node`** and set `last_visited = pop(S)`.

#### Pseudocode (Recursive & Single-Stack Iterative Postorder)

```text
ALGORITHM PostorderRecursive(u)
1.  if u == NULL then return
2.  PostorderRecursive(u.left)
3.  PostorderRecursive(u.right)
4.  Visit(u)

ALGORITHM PostorderIterativeSingleStack(root)
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

##### Step-by-Step Single-Stack (`last_visited`) Trace Table (on $T_6$)

| Step | `curr` | `last_visited` | Stack `S` (Bottom $\to$ Top) | Action Taken (`if curr != NULL` vs `else` Peek Check) | Output So Far |
| :---: | :---: | :---: | :--- | :--- | :--- |
| **1** | `A` | `NULL` | `[A]` | Push `A`, `curr = A->left = B` | |
| **2** | `B` | `NULL` | `[A, B]` | Push `B`, `curr = B->left = D` | |
| **3** | `D` | `NULL` | `[A, B, D]` | Push `D`, `curr = D->left = NULL` | |
| **4** | `NULL` | `NULL` | `[A, B]` | Peek `D`: `D->right == NULL` $\implies$ **Visit `D`**, pop `D`, `last_visited = D` | `D` |
| **5** | `NULL` | `D` | `[A, B]` | Peek `B`: `B->right (E) != D` $\implies$ descend right: `curr = E` | `D` |
| **6** | `E` | `D` | `[A, B, E]` | Push `E`, `curr = E->left = NULL` | `D` |
| **7** | `NULL` | `D` | `[A, B]` | Peek `E`: `E->right == NULL` $\implies$ **Visit `E`**, pop `E`, `last_visited = E` | `D E` |
| **8** | `NULL` | `E` | `[A]` | Peek `B`: `B->right (E) == last_visited (E)` $\implies$ **Visit `B`**, pop `B`, `last_visited = B` | `D E B` |
| **9** | `NULL` | `B` | `[A]` | Peek `A`: `A->right (C) != B` $\implies$ descend right: `curr = C` | `D E B` |
| **10** | `C` | `B` | `[A, C]` | Push `C`, `curr = C->left = NULL` | `D E B` |
| **11** | `NULL` | `B` | `[A, C]` | Peek `C`: `C->right (F) != B` $\implies$ descend right: `curr = F` | `D E B` |
| **12** | `F` | `B` | `[A, C, F]` | Push `F`, `curr = F->left = NULL` | `D E B` |
| **13** | `NULL` | `B` | `[A, C]` | Peek `F`: `F->right == NULL` $\implies$ **Visit `F`**, pop `F`, `last_visited = F` | `D E B F` |
| **14** | `NULL` | `F` | `[A]` | Peek `C`: `C->right (F) == last_visited (F)` $\implies$ **Visit `C`**, pop `C`, `last_visited = C` | `D E B F C` |
| **15** | `NULL` | `C` | `[]` | Peek `A`: `A->right (C) == last_visited (C)` $\implies$ **Visit `A`**, pop `A`, `last_visited = A` | **`D E B F C A`** |

#### Why Postorder is Mandatory for `free()` and Bottom-Up Aggregations
1. **Heap Memory Deallocation (`bt_free`):** A parent node holds the only pointers (`u->left`, `u->right`) to its children. If you call `free(u)` before freeing `u->left` and `u->right`, accessing `u->left` afterward is a **Use-After-Free bug** (or if you don't access them, the entire subtrees become unreachable **memory leaks**). Postorder guarantees children are freed before their parent.
2. **Bottom-Up Tree Aggregations:** Computing a node's height $h(u) = 1 + \max(h(u.\text{left}), h(u.\text{right}))$, subtree node count, or evaluating an arithmetic expression tree (`+` needs the evaluated values of its left and right operands first) all require **Postorder**.

---

### 4.4 Level-Order (Breadth-First) Traversal Using a FIFO Queue

#### Step-by-Step Algorithm
1. If `root == NULL`, return.
2. Initialize a FIFO Queue `Q` and `Enqueue(Q, root)`.
3. While `Q` is not empty:
   - `curr = Dequeue(Q)` and **visit `curr`**.
   - If `curr->left != NULL`, `Enqueue(Q, curr->left)`.
   - If `curr->right != NULL`, `Enqueue(Q, curr->right)`.

#### Level-Order Queue Trace Table (on $T_6$)

| Step | Dequeued & Visited | Enqueued Children | Queue `Q` (Front $\to$ Rear) | Output So Far |
| :---: | :---: | :---: | :--- | :--- |
| **Init** | — | `A` | `[A]` | *(empty)* |
| **1** | `A` | `B`, `C` | `[B, C]` | `A` |
| **2** | `B` | `D`, `E` | `[C, D, E]` | `A B` |
| **3** | `C` | `F` | `[D, E, F]` | `A B C` |
| **4** | `D` | *(none)* | `[E, F]` | `A B C D` |
| **5** | `E` | *(none)* | `[F]` | `A B C D E` |
| **6** | `F` | *(none)* | `[]` *(empty — terminate)* | **`A B C D E F`** |

---

### 4.5 Unique Binary Tree Reconstruction from Traversal Sequences

#### Worked Example: Reconstructing $T_6$ from `Inorder` and `Preorder`
Suppose we are given:
- **`Preorder = [A, B, D, E, C, F]`** *(Root is always first!)*
- **`Inorder  = [D, B, E, A, C, F]`** *(Splits Left Subtree | Root | Right Subtree!)*

**Step-by-Step Trace:**
1. **Call 1 (`in[0..5]`):** Next in `Preorder` is **`A`**. Locate `A` in `Inorder` at index `3`:
   - Left of `A` in `Inorder`: `[D, B, E]` (`in[0..2]`, 3 nodes).
   - Right of `A` in `Inorder`: `[C, F]` (`in[4..5]`, 2 nodes).
2. **Call 2 (Left of `A`, `in[0..2] = [D, B, E]`):** Next in `Preorder` is **`B`**. Locate `B` in `Inorder` at index `1`:
   - Left of `B`: `[D]` (`in[0..0]`) $\to$ Next in `Preorder` is **`D`** (leaf).
   - Right of `B`: `[E]` (`in[2..2]`) $\to$ Next in `Preorder` is **`E`** (leaf).
3. **Call 3 (Right of `A`, `in[4..5] = [C, F]`):** Next in `Preorder` is **`C`**. Locate `C` in `Inorder` at index `4`:
   - Left of `C`: `in[4..3]` (empty range `start > end` $\to$ **`NULL`**!).
   - Right of `C`: `[F]` (`in[5..5]`) $\to$ Next in `Preorder` is **`F`** (leaf).

```text
Step 1 (Root A splits Inorder)     Step 2 (Root B splits [D,B,E])     Step 3 (Root C splits [C,F])
          [ A ]                                [ A ]                              [ A ]
         /     \                              /     \                            /     \
   [D, B, E]  [C, F]                       [ B ]   [C, F]                     [ B ]   [ C ]
                                           /   \                              /   \   /   \
                                         [D]   [E]                          [D]  [E] NULL [F]
```

> **Note on `Inorder + Postorder` Reconstruction:**
> If given `Inorder` and `Postorder = [D, E, B, F, C, A]`, we scan `Postorder` **from right to left** (`post_idx` starting at $n - 1$ down to $0$). Because scanning `Postorder` backwards visits $\text{Root} \to \text{Right} \to \text{Left}$, you must recursively build `root->right` **before** `root->left`!

---

### 4.6 General N-Ary Tree Traversals and the LCRS Isomorphism Theorem

#### Traversals Defined on an N-Ary Tree
Let $u$ be a node in an ordered $N$-ary tree $T$ with children $c_1, c_2, \dots, c_d$ ($d \ge 0$):
1. **N-Ary Preorder Traversal:** Visit $u$ first, then recursively perform N-ary Preorder on $c_1, c_2, \dots, c_d$ in left-to-right order.
2. **N-Ary Postorder Traversal:** Recursively perform N-ary Postorder on $c_1, c_2, \dots, c_d$ in left-to-right order, then visit $u$ last.
3. **N-Ary Level-Order Traversal:** Visit nodes level by level from $d = 0$ to $h$ using a FIFO Queue.
4. **Why Standard Inorder is Undefined for General N-Ary Trees:** In a binary tree, every node has a clear division between its $1^{\text{st}}$ (left) and $2^{\text{nd}}$ (right) child slots. In an $N$-ary tree where one node may have $3$ children and another $5$ children, there is no universal position among $c_1, \dots, c_d$ at which to place the parent $u$.

---

#### Theorem 4: The LCRS Traversal Correspondence Theorem
> **Statement:** Let $T$ be any ordered $N$-ary tree (or ordered forest $\mathcal{F}$) and let $B(T)$ be its **Left-Child Right-Sibling (LCRS)** Binary Tree representation. Then:
> 1. $\text{Preorder}_{\text{N-ary}}(T) \;\equiv\; \text{Preorder}_{\text{Binary}}(B(T))$
> 2. $\text{Postorder}_{\text{N-ary}}(T) \;\equiv\; \mathbf{\text{Inorder}_{\text{Binary}}}(B(T))$
>
> *(Warning: $\text{Postorder}_{\text{N-ary}}(T)$ is **NOT** equal to $\text{Postorder}_{\text{Binary}}(B(T))$!)*

**Proof:**
Consider a node $u$ with subtrees $T_1, T_2, \dots, T_d$ in the $N$-ary tree (and right-siblings $S$ after $u$). In the LCRS binary tree $B(T)$:
- `u->left` contains the LCRS representation of $u$'s children $T_1, T_2, \dots, T_d$.
- `u->right` contains the LCRS representation of $u$'s right siblings $S$.

1. **Preorder Correspondence ($\text{NLR}$ on $B(T)$):**
   - Binary Preorder on `u` visits **$\text{N}$ ($u$) first**, then traverses **$\text{L}$ (`u->left`, which visits all children/descendants of $u$ in $T_1 \dots T_d$)**, and finally traverses **$\text{R}$ (`u->right`, which visits $u$'s right siblings $S$)**.
   - Visiting $u$, then all of $u$'s children, then moving to $u$'s next sibling is the **exact definition of N-Ary Preorder**!

2. **Postorder $\leftrightarrow$ Binary Inorder Correspondence ($\text{LNR}$ on $B(T)$):**
   - Binary **Inorder** ($\text{LNR}$) on `u` first traverses **$\text{L}$ (`u->left`, which completely visits all children $T_1, \dots, T_d$ of $u$)**, **then visits $\text{N}$ ($u$ itself!)**, and finally traverses **$\text{R}$ (`u->right`, moving to $u$'s next sibling $S$)**!
   - Visiting all children of $u$ *before* visiting $u$, and only then moving to $u$'s next sibling, is the **exact definition of N-Ary Postorder**!
   - *(By contrast, Binary Postorder $\text{LRN}$ on $B(T)$ would visit `u->right`—$u$'s siblings—**before** visiting $u$, reversing the order of siblings!)* $\blacksquare$

```text
Visual Verification on N-Ary Tree T and its LCRS Binary Tree B(T):

        N-Ary Tree T                        LCRS Binary Tree B(T)
           [ A ]                                    [ A ]
         /   |   \                                  /
       [B]  [C]  [D]                              [B]
       / \        |                               / \
     [E] [F]     [G]                            [E] [C]
                                                  \   \
                                                  [F] [D]
                                                      /
                                                    [G]

  N-Ary Preorder(T)   = A B E F C D G   <===>   Binary Preorder(B(T))  = A B E F C D G  (MATCH!)
  N-Ary Postorder(T)  = E F B C G D A   <===>   Binary Inorder(B(T))   = E F B C G D A  (MATCH!)
                                                Binary Postorder(B(T)) = F E G D C B A  (DIFFERENT!)
```

---

### 4.7 Complete C11 Implementation and Verified Execution Trace

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

/* =========================================================================
 * 1. BINARY TREE NODE & CREATION HELPERS
 * ========================================================================= */
typedef struct BTNode {
    char data;
    struct BTNode *left;
    struct BTNode *right;
} BTNode;

BTNode *bt_create(char data) {
    BTNode *node = (BTNode *)malloc(sizeof(BTNode));
    if (node == NULL) {
        fprintf(stderr, "Error: malloc failed\n");
        exit(EXIT_FAILURE);
    }
    node->data = data;
    node->left = NULL;
    node->right = NULL;
    return node;
}

void bt_free(BTNode *root) {
    if (root == NULL) return;
    bt_free(root->left);
    bt_free(root->right);
    free(root);
}

/* =========================================================================
 * 2. EXPLICIT STACK & QUEUE FOR ITERATIVE TRAVERSALS
 * ========================================================================= */
#define MAX_CAP 64

typedef struct {
    const BTNode *items[MAX_CAP];
    int top;
} NodeStack;

void stack_init(NodeStack *s) { s->top = -1; }
bool stack_empty(const NodeStack *s) { return s->top == -1; }
void stack_push(NodeStack *s, const BTNode *node) {
    if (s->top + 1 >= MAX_CAP) {
        fprintf(stderr, "Error: Stack overflow\n");
        exit(EXIT_FAILURE);
    }
    s->items[++(s->top)] = node;
}
const BTNode *stack_pop(NodeStack *s) {
    return stack_empty(s) ? NULL : s->items[(s->top)--];
}
const BTNode *stack_peek(const NodeStack *s) {
    return stack_empty(s) ? NULL : s->items[s->top];
}

typedef struct {
    const BTNode *items[MAX_CAP];
    int front;
    int rear;
    int size;
} NodeQueue;

void queue_init(NodeQueue *q) { q->front = 0; q->rear = -1; q->size = 0; }
bool queue_empty(const NodeQueue *q) { return q->size == 0; }
void queue_enqueue(NodeQueue *q, const BTNode *node) {
    if (q->size >= MAX_CAP) {
        fprintf(stderr, "Error: Queue overflow\n");
        exit(EXIT_FAILURE);
    }
    q->rear = (q->rear + 1) % MAX_CAP;
    q->items[q->rear] = node;
    q->size++;
}
const BTNode *queue_dequeue(NodeQueue *q) {
    if (queue_empty(q)) return NULL;
    const BTNode *front_node = q->items[q->front];
    q->front = (q->front + 1) % MAX_CAP;
    q->size--;
    return front_node;
}

/* =========================================================================
 * 3. RECURSIVE BINARY TREE TRAVERSALS (DFS)
 * ========================================================================= */
void preorder_recursive(const BTNode *root) {
    if (root == NULL) return;
    printf("%c ", root->data);
    preorder_recursive(root->left);
    preorder_recursive(root->right);
}

void inorder_recursive(const BTNode *root) {
    if (root == NULL) return;
    inorder_recursive(root->left);
    printf("%c ", root->data);
    inorder_recursive(root->right);
}

void postorder_recursive(const BTNode *root) {
    if (root == NULL) return;
    postorder_recursive(root->left);
    postorder_recursive(root->right);
    printf("%c ", root->data);
}

/* =========================================================================
 * 4. ITERATIVE BINARY TREE TRAVERSALS (EXPLICIT STACK & QUEUE)
 * ========================================================================= */

/* 4.1 Iterative Preorder (Root -> Left -> Right)
 * Crucial rule: Push RIGHT child first, then LEFT child so LEFT is popped first! */
void preorder_iterative(const BTNode *root) {
    if (root == NULL) return;
    NodeStack s;
    stack_init(&s);
    stack_push(&s, root);

    while (!stack_empty(&s)) {
        const BTNode *curr = stack_pop(&s);
        printf("%c ", curr->data);

        if (curr->right != NULL) stack_push(&s, curr->right);
        if (curr->left != NULL)  stack_push(&s, curr->left);
    }
}

/* 4.2 Iterative Inorder (Left -> Root -> Right)
 * Plunge down left spine pushing nodes; pop & visit; pivot to right child. */
void inorder_iterative(const BTNode *root) {
    NodeStack s;
    stack_init(&s);
    const BTNode *curr = root;

    while (curr != NULL || !stack_empty(&s)) {
        while (curr != NULL) {
            stack_push(&s, curr);
            curr = curr->left;
        }
        curr = stack_pop(&s);
        printf("%c ", curr->data);
        curr = curr->right;
    }
}

/* 4.3A Iterative Postorder — Method 1: Two-Stack Method
 * S1 generates Root -> Right -> Left into S2; popping S2 yields Left -> Right -> Root. */
void postorder_iterative_two_stacks(const BTNode *root) {
    if (root == NULL) return;
    NodeStack s1, s2;
    stack_init(&s1);
    stack_init(&s2);
    stack_push(&s1, root);

    while (!stack_empty(&s1)) {
        const BTNode *curr = stack_pop(&s1);
        stack_push(&s2, curr);

        /* Push left first, then right onto S1 so right is processed first into S2 */
        if (curr->left != NULL)  stack_push(&s1, curr->left);
        if (curr->right != NULL) stack_push(&s1, curr->right);
    }

    while (!stack_empty(&s2)) {
        printf("%c ", stack_pop(&s2)->data);
    }
}

/* 4.3B Iterative Postorder — Method 2: Single-Stack with last_visited Pointer
 * Uses O(h) space by tracking the most recently visited node. */
void postorder_iterative_one_stack(const BTNode *root) {
    NodeStack s;
    stack_init(&s);
    const BTNode *curr = root;
    const BTNode *last_visited = NULL;

    while (curr != NULL || !stack_empty(&s)) {
        if (curr != NULL) {
            stack_push(&s, curr);
            curr = curr->left;
        } else {
            const BTNode *peek_node = stack_peek(&s);
            /* If right child exists and we haven't visited it yet, descend right */
            if (peek_node->right != NULL && last_visited != peek_node->right) {
                curr = peek_node->right;
            } else {
                printf("%c ", peek_node->data);
                last_visited = stack_pop(&s);
            }
        }
    }
}

/* 4.4 Level-Order (Breadth-First) Traversal Using a FIFO Queue */
void level_order_traversal(const BTNode *root) {
    if (root == NULL) return;
    NodeQueue q;
    queue_init(&q);
    queue_enqueue(&q, root);

    while (!queue_empty(&q)) {
        const BTNode *curr = queue_dequeue(&q);
        printf("%c ", curr->data);
        if (curr->left != NULL)  queue_enqueue(&q, curr->left);
        if (curr->right != NULL) queue_enqueue(&q, curr->right);
    }
}

/* =========================================================================
 * 5. BINARY TREE RECONSTRUCTION FROM INORDER + PREORDER
 * ========================================================================= */
BTNode *build_tree_in_pre(const char *inorder, const char *preorder,
                          int in_start, int in_end, int *pre_idx) {
    if (in_start > in_end) {
        return NULL;
    }
    char root_val = preorder[(*pre_idx)++];
    BTNode *root = bt_create(root_val);

    if (in_start == in_end) {
        return root;
    }

    /* Locate root_val in inorder[in_start .. in_end] */
    int in_root_idx = in_start;
    while (in_root_idx <= in_end && inorder[in_root_idx] != root_val) {
        in_root_idx++;
    }

    root->left  = build_tree_in_pre(inorder, preorder, in_start, in_root_idx - 1, pre_idx);
    root->right = build_tree_in_pre(inorder, preorder, in_root_idx + 1, in_end, pre_idx);
    return root;
}

/* =========================================================================
 * 6. GENERAL N-ARY TREE TRAVERSALS & LCRS ISOMORPHISM VERIFICATION
 * ========================================================================= */
typedef struct NAryNode {
    char data;
    struct NAryNode *children[4];
    int child_count;
} NAryNode;

NAryNode *nary_create(char data) {
    NAryNode *node = (NAryNode *)malloc(sizeof(NAryNode));
    node->data = data;
    node->child_count = 0;
    return node;
}

void nary_add(NAryNode *parent, NAryNode *child) {
    parent->children[parent->child_count++] = child;
}

void nary_preorder(const NAryNode *root) {
    if (root == NULL) return;
    printf("%c ", root->data);
    for (int i = 0; i < root->child_count; i++) {
        nary_preorder(root->children[i]);
    }
}

void nary_postorder(const NAryNode *root) {
    if (root == NULL) return;
    for (int i = 0; i < root->child_count; i++) {
        nary_postorder(root->children[i]);
    }
    printf("%c ", root->data);
}

BTNode *nary_to_lcrs(const NAryNode *root) {
    if (root == NULL) return NULL;
    BTNode *b = bt_create(root->data);
    if (root->child_count > 0) {
        b->left = nary_to_lcrs(root->children[0]);
        BTNode *curr = b->left;
        for (int i = 1; i < root->child_count; i++) {
            curr->right = nary_to_lcrs(root->children[i]);
            curr = curr->right;
        }
    }
    return b;
}

void nary_free(NAryNode *root) {
    if (root == NULL) return;
    for (int i = 0; i < root->child_count; i++) {
        nary_free(root->children[i]);
    }
    free(root);
}

int main(void) {
    /* Construct 6-Node Binary Tree Used in Step-by-Step Stack Traces:
     *          [ A ]
     *          /   \
     *        [B]   [C]
     *        / \     \
     *      [D] [E]   [F]
     */
    BTNode *root = bt_create('A');
    root->left = bt_create('B');
    root->right = bt_create('C');
    root->left->left = bt_create('D');
    root->left->right = bt_create('E');
    root->right->right = bt_create('F');

    printf("=== 1. Binary Tree Traversals (Recursive vs. Iterative) ===\n");
    printf("Preorder  (Recursive)          : "); preorder_recursive(root); printf("\n");
    printf("Preorder  (Iterative - Stack)  : "); preorder_iterative(root); printf("\n\n");

    printf("Inorder   (Recursive)          : "); inorder_recursive(root); printf("\n");
    printf("Inorder   (Iterative - Stack)  : "); inorder_iterative(root); printf("\n\n");

    printf("Postorder (Recursive)          : "); postorder_recursive(root); printf("\n");
    printf("Postorder (Iterative - 2-Stack): "); postorder_iterative_two_stacks(root); printf("\n");
    printf("Postorder (Iterative - 1-Stack): "); postorder_iterative_one_stack(root); printf("\n\n");

    printf("Level-Order (BFS - Queue)      : "); level_order_traversal(root); printf("\n\n");

    printf("=== 2. Tree Reconstruction from Inorder + Preorder ===\n");
    const char inorder_seq[]  = {'D', 'B', 'E', 'A', 'C', 'F'};
    const char preorder_seq[] = {'A', 'B', 'D', 'E', 'C', 'F'};
    int pre_idx = 0;
    BTNode *rebuilt = build_tree_in_pre(inorder_seq, preorder_seq, 0, 5, &pre_idx);
    printf("Rebuilt Tree Postorder Check   : "); postorder_recursive(rebuilt); printf("\n\n");

    printf("=== 3. N-Ary Tree Traversals & LCRS Binary Correspondence ===\n");
    /* N-Ary Tree T:
     *         A
     *      /  |  \
     *     B   C   D
     *    / \      |
     *   E   F     G
     */
    NAryNode *nA = nary_create('A');
    NAryNode *nB = nary_create('B');
    NAryNode *nC = nary_create('C');
    NAryNode *nD = nary_create('D');
    nary_add(nB, nary_create('E'));
    nary_add(nB, nary_create('F'));
    nary_add(nD, nary_create('G'));
    nary_add(nA, nB);
    nary_add(nA, nC);
    nary_add(nA, nD);

    BTNode *lcrs = nary_to_lcrs(nA);
    printf("N-Ary Preorder(T)              : "); nary_preorder(nA); printf("\n");
    printf("LCRS  Preorder(B(T))           : "); preorder_recursive(lcrs); printf("\n");
    printf("N-Ary Postorder(T)             : "); nary_postorder(nA); printf("\n");
    printf("LCRS  Inorder(B(T))   [MATCH!] : "); inorder_recursive(lcrs); printf("\n");
    printf("LCRS  Postorder(B(T)) [DIFF!]  : "); postorder_recursive(lcrs); printf("\n");

    bt_free(root);
    bt_free(rebuilt);
    bt_free(lcrs);
    nary_free(nA);
    return 0;
}
```

#### Program Output:

```text
=== 1. Binary Tree Traversals (Recursive vs. Iterative) ===
Preorder  (Recursive)          : A B D E C F 
Preorder  (Iterative - Stack)  : A B D E C F 

Inorder   (Recursive)          : D B E A C F 
Inorder   (Iterative - Stack)  : D B E A C F 

Postorder (Recursive)          : D E B F C A 
Postorder (Iterative - 2-Stack): D E B F C A 
Postorder (Iterative - 1-Stack): D E B F C A 

Level-Order (BFS - Queue)      : A B C D E F 

=== 2. Tree Reconstruction from Inorder + Preorder ===
Rebuilt Tree Postorder Check   : D E B F C A 

=== 3. N-Ary Tree Traversals & LCRS Binary Correspondence ===
N-Ary Preorder(T)              : A B E F C D G 
LCRS  Preorder(B(T))           : A B E F C D G 
N-Ary Postorder(T)             : E F B C G D A 
LCRS  Inorder(B(T))   [MATCH!] : E F B C G D A 
LCRS  Postorder(B(T)) [DIFF!]  : F E G D C B A 
```

---

## 5. Comparison Tables and Complexity Summary

### 5.1 Master Comparison of Binary Tree Traversals

| Dimension | Preorder ($\text{NLR}$) | Inorder ($\text{LNR}$) | Postorder ($\text{LRN}$) | Level-Order (BFS) |
| :--- | :--- | :--- | :--- | :--- |
| **Order of Operations** | Root $\to$ Left $\to$ Right | Left $\to$ Root $\to$ Right | Left $\to$ Right $\to$ Root | Depth $d = 0, 1, \dots, h$ (L to R) |
| **Position of Root** | **First** element in output | Between Left and Right subtrees | **Last** element in output | **First** element in output |
| **Auxiliary Data Structure** | LIFO Stack | LIFO Stack | LIFO Stack (1 or 2 stacks) | FIFO Queue |
| **Euler Tour Encounter** | $1^{\text{st}}$ pass (left of node) | $2^{\text{nd}}$ pass (underneath node) | $3^{\text{rd}}$ pass (right of node) | Not a DFS Euler Tour |
| **Expression Tree Output** | Prefix (`+ A * B C`) | Infix (`A + B * C`) | Postfix / RPN (`A B C * +`) | Level-wise operators/operands |
| **Primary Applications** | Tree cloning, serialization, prefix export | Sorted output of a **BST** | `free()` cleanup, height/size calculation, RPN evaluation | Shortest path in unweighted tree, level printing |

---

### 5.2 N-Ary Tree Traversal vs. Binary Tree Traversal

| Feature | General $N$-Ary Tree $T$ | Binary Tree $B$ | LCRS Binary Tree $B(T)$ of $N$-Ary Tree $T$ |
| :--- | :--- | :--- | :--- |
| **Preorder Traversal** | Visit $u$, then loop $c_1 \dots c_d$ | Visit $u$, then `left`, then `right` | $\text{Preorder}(B(T)) \equiv \text{Preorder}(T)$ |
| **Inorder Traversal** | **Undefined** in general (ambiguous child split) | Traverse `left`, visit $u$, traverse `right` | $\text{Inorder}(B(T)) \equiv \mathbf{\text{Postorder}}(T)$ |
| **Postorder Traversal** | Loop $c_1 \dots c_d$, then visit $u$ | Traverse `left`, traverse `right`, visit $u$ | $\text{Postorder}(B(T)) \ne \text{Postorder}(T)$ |

---

### 5.3 Time and Space Complexity Summary

**Variable Definitions:**
- $n$: Total number of nodes in the tree.
- $h$: Height of the tree ($\lfloor \log_2 n \rfloor$ for balanced; $n - 1$ for skewed).
- $w$: Maximum width of the tree (maximum nodes at any single depth $d$; $w \le \lceil n / 2 \rceil$ for binary trees).

| Algorithm | Time Complexity | Space (Balanced Tree) | Space (Skewed Tree) | Space Formula |
| :--- | :---: | :---: | :---: | :---: |
| **Recursive Preorder / Inorder / Postorder** | $\Theta(n)$ | $\Theta(\log n)$ | $\Theta(n)$ | $\Theta(h)$ call stack frames |
| **Iterative Preorder (1 Stack)** | $\Theta(n)$ | $\Theta(\log n)$ | $\Theta(n)$ | $\Theta(h)$ pointers |
| **Iterative Inorder (1 Stack)** | $\Theta(n)$ | $\Theta(\log n)$ | $\Theta(n)$ | $\Theta(h)$ pointers |
| **Iterative Postorder (2 Stacks)** | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ (`S2` holds all $n$ nodes) |
| **Iterative Postorder (1 Stack + `last_visited`)** | $\Theta(n)$ | $\Theta(\log n)$ | $\Theta(n)$ | $\Theta(h)$ pointers |
| **Level-Order Traversal (FIFO Queue)** | $\Theta(n)$ | $\Theta(n)$ (last level $\approx n/2$) | $\Theta(1)$ ($1$ node per level) | $\Theta(w)$ pointers |
| **Tree Reconstruction (`Inorder + Preorder` — Linear Scan)** | $O(n^2)$ worst / $O(n \log n)$ avg | $\Theta(\log n)$ | $\Theta(n)$ | $O(n)$ time if using a Hash Map / LUT for Inorder indices |

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Concrete Bug vs. Fix Examples in C

#### Pitfall 1: Pushing `left` Before `right` in Iterative Preorder
Because a stack is LIFO, pushing `left` first and `right` second puts `right` on top of the stack, visiting the right subtree before the left subtree ($\text{NRL}$ instead of $\text{NLR}$)!

```c
/* BUG: Pushes left first, then right. Because Stack is LIFO, right is popped
 *      first, producing Root -> Right -> Left instead of Preorder! */
void preorder_iter_buggy(const BTNode *root) {
    if (root == NULL) return;
    NodeStack s; stack_init(&s); stack_push(&s, root);
    while (!stack_empty(&s)) {
        const BTNode *curr = stack_pop(&s);
        printf("%c ", curr->data);
        if (curr->left != NULL)  stack_push(&s, curr->left);  /* Pushed first (bottom) */
        if (curr->right != NULL) stack_push(&s, curr->right); /* Pushed second (TOP!) */
    }
}

/* FIX: Push RIGHT child first, then LEFT child so LEFT sits at the top of the stack. */
void preorder_iter_fixed(const BTNode *root) {
    if (root == NULL) return;
    NodeStack s; stack_init(&s); stack_push(&s, root);
    while (!stack_empty(&s)) {
        const BTNode *curr = stack_pop(&s);
        printf("%c ", curr->data);
        if (curr->right != NULL) stack_push(&s, curr->right); /* Pushed first */
        if (curr->left != NULL)  stack_push(&s, curr->left);  /* Pushed second (popped next!) */
    }
}
```

#### Pitfall 2: Infinite Loop in Single-Stack Iterative Postorder by Forgetting `last_visited`
If you forget to update `last_visited = stack_pop(&s)` after visiting a right child, the parent node on the stack will repeatedly think its right child has not been visited yet and re-enter `curr = peek_node->right` forever!

```c
/* BUG: Pops the node after visiting, but forgets to record last_visited!
 *      When backtracking to parent, peek_node->right != last_visited is ALWAYS true! */
void postorder_1stack_buggy(const BTNode *root) {
    NodeStack s; stack_init(&s);
    const BTNode *curr = root, *last_visited = NULL;
    while (curr != NULL || !stack_empty(&s)) {
        if (curr != NULL) { stack_push(&s, curr); curr = curr->left; }
        else {
            const BTNode *peek_node = stack_peek(&s);
            if (peek_node->right != NULL && last_visited != peek_node->right) {
                curr = peek_node->right;
            } else {
                printf("%c ", peek_node->data);
                stack_pop(&s); /* BUG: last_visited is never updated! Infinite loop! */
            }
        }
    }
}

/* FIX: Always assign last_visited = stack_pop(&s) when finishing a node. */
void postorder_1stack_fixed(const BTNode *root) {
    NodeStack s; stack_init(&s);
    const BTNode *curr = root, *last_visited = NULL;
    while (curr != NULL || !stack_empty(&s)) {
        if (curr != NULL) { stack_push(&s, curr); curr = curr->left; }
        else {
            const BTNode *peek_node = stack_peek(&s);
            if (peek_node->right != NULL && last_visited != peek_node->right) {
                curr = peek_node->right;
            } else {
                printf("%c ", peek_node->data);
                last_visited = stack_pop(&s);
            }
        }
    }
}
```

#### Pitfall 3: Building Left Subtree Before Right Subtree When Reconstructing from `Inorder + Postorder`
When reconstructing a tree from `Inorder` and `Postorder`, decrementing `post_idx` from $n - 1$ down to $0$ visits $\text{Root}$, then the **Right Subtree**, then the **Left Subtree**. Calling `build(left)` before `build(right)` consumes the right subtree's nodes into the left subtree!

```c
/* BUG: Decrements post_idx from n-1, but builds root->left BEFORE root->right! */
BTNode *build_in_post_buggy(const char *in, const char *post,
                            int in_start, int in_end, int *post_idx) {
    if (in_start > in_end) return NULL;
    char val = post[(*post_idx)--];
    BTNode *root = bt_create(val);
    int mid = in_start;
    while (in[mid] != val) mid++;
    /* WRONG ORDER: post_idx is currently pointing at the RIGHT subtree's root! */
    root->left  = build_in_post_buggy(in, post, in_start, mid - 1, post_idx);
    root->right = build_in_post_buggy(in, post, mid + 1, in_end, post_idx);
    return root;
}

/* FIX: When consuming Postorder backwards, build root->right FIRST, then root->left! */
BTNode *build_in_post_fixed(const char *in, const char *post,
                            int in_start, int in_end, int *post_idx) {
    if (in_start > in_end) return NULL;
    char val = post[(*post_idx)--];
    BTNode *root = bt_create(val);
    int mid = in_start;
    while (in[mid] != val) mid++;
    root->right = build_in_post_fixed(in, post, mid + 1, in_end, post_idx);
    root->left  = build_in_post_fixed(in, post, in_start, mid - 1, post_idx);
    return root;
}
```

---

### 6.2 Boundary Conditions Checklist

- [ ] **Empty Tree (`root == NULL`):**
  - Do all recursive and iterative functions (`preorder_iterative`, `inorder_iterative`, `postorder_iterative`, `level_order_traversal`) exit cleanly without pushing `NULL` onto the stack/queue and dereferencing it?
- [ ] **Single-Node Tree (`root->left == NULL && root->right == NULL`):**
  - Preorder, Inorder, Postorder, and Level-Order must all output `[root->data]`.
- [ ] **Left-Skewed vs. Right-Skewed Degenerate Trees:**
  - On a **left-skewed** tree ($A \xrightarrow{L} B \xrightarrow{L} C$), verify:
    - $\text{Preorder} = [A, B, C]$, while $\text{Inorder} = \text{Postorder} = [C, B, A]$ (exact reverse!).
  - On a **right-skewed** tree ($A \xrightarrow{R} B \xrightarrow{R} C$), verify:
    - $\text{Preorder} = \text{Inorder} = [A, B, C]$, while $\text{Postorder} = [C, B, A]$.
- [ ] **Duplicate Keys in Tree Reconstruction:**
  - Standard reconstruction from `(Inorder + Preorder)` requires **distinct node keys** so `in_root_idx` is unique.

---

## 7. Summary and Revision Checklist

### Key Takeaways
1. **DFS vs. BFS:** Depth-First traversals (**Preorder $\text{NLR}$, Inorder $\text{LNR}$, Postorder $\text{LRN}$**) use a **Stack** in $\Theta(h)$ auxiliary space; Breadth-First (**Level-Order**) uses a **Queue** in $\Theta(w)$ auxiliary space. All visit $n$ nodes in $\Theta(n)$ time.
2. **Iterative Stack Rules:**
   - **Preorder:** Pop & visit; push `right` first, then `left`.
   - **Inorder:** Push left spine while `curr != NULL`; pop, visit, set `curr = curr->right`.
   - **Postorder:** Either use **Two Stacks** (reversing $\text{Root} \to \text{Right} \to \text{Left}$) or **One Stack + `last_visited`** (checking `peek->right != NULL && last_visited != peek->right`).
3. **Tree Reconstruction:** `Inorder + Preorder` or `Inorder + Postorder` uniquely determines any binary tree with distinct keys; `Preorder + Postorder` is ambiguous whenever degree-1 nodes can exist.
4. **LCRS Traversal Isomorphism:** For any $N$-ary tree $T$ and its LCRS Binary Tree $B(T)$:
   $$\text{Preorder}(T) \equiv \text{Preorder}(B(T)) \qquad \text{and} \qquad \text{Postorder}(T) \equiv \text{Inorder}(B(T))$$

### Self-Test Revision Checklist
- [ ] Can I trace Iterative Preorder, Inorder, and Postorder (both 1-stack and 2-stack) with a stack table on an arbitrary binary tree?
- [ ] Can I explain why `curr->right` must be pushed before `curr->left` in Iterative Preorder?
- [ ] Can I reconstruct a binary tree step-by-step given `Inorder` and `Preorder` (or `Inorder` and `Postorder`)?
- [ ] Can I prove why `Postorder` of an $N$-ary tree equals `Inorder` (and NOT `Postorder`) of its LCRS Binary Tree?

### Navigation Links
- **Previous Topic:** [04 - N-Ary Trees, Binary Trees, and Forests](./04-N-Ary-Binary-Trees-and-Forests.md)
- **Next Topic:** [06 - Unit 2 Comprehensive Review & Exam Preparation](./06-Unit-2-Review.md)
