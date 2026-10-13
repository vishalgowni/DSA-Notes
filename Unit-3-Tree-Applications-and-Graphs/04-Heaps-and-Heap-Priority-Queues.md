# Unit 3 — Topic 4: Binary Heaps and Heap-Based Priority Queues

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [What Is a Binary Heap?](#11-what-is-a-binary-heap)
   - 1.2 [Why Do We Need Heaps? The Problem They Solve](#12-why-do-we-need-heaps-the-problem-they-solve)
   - 1.3 [Relationship to Prior Concepts: Binary Trees, BSTs, and Priority Queues](#13-relationship-to-prior-concepts-binary-trees-bsts-and-priority-queues)
   - 1.4 [Real-World and Systems Applications](#14-real-world-and-systems-applications)
   - 1.5 [Learning Objectives](#15-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Structural Invariant 1: The Shape Property (Complete Binary Tree)](#21-structural-invariant-1-the-shape-property-complete-binary-tree)
   - 2.2 [Structural Invariant 2: The Heap-Order Property (Max-Heap vs. Min-Heap)](#22-structural-invariant-2-structural-invariant-2-the-heap-order-property-max-heap-vs-min-heap)
   - 2.3 [Why a Binary Heap Is NOT a Binary Search Tree (BST)](#23-why-a-binary-heap-is-not-a-binary-search-tree-bst)
   - 2.4 [Core Terminology Table](#24-core-terminology-table)
   - 2.5 [Mathematical Properties, Invariants, and Formulas](#25-mathematical-properties-invariants-and-formulas)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Why Heaps Are Implicit Array Data Structures (Zero Pointer Overhead)](#31-why-heaps-are-implicit-array-data-structures-zero-pointer-overhead)
   - 3.2 [0-Based vs. 1-Based Array Index Formulas Compared](#32-0-based-vs-1-based-array-index-formulas-compared)
   - 3.3 [Tree-to-Array Mapping Diagram](#33-tree-to-array-mapping-diagram)
   - 3.4 [C `typedef struct` Declarations and Memory Layout](#34-c-typedef-struct-declarations-and-memory-layout)
   - 3.5 [Static Capacity vs. Dynamically Resized Heap Storage](#35-static-capacity-vs-dynamically-resized-heap-storage)
4. [Operations and Algorithms](#4-operations-and-algorithms)
   - 4.1 [Up-Heap Restoration (`sift_up`) and Heap Insertion ($O(\log n)$)](#41-up-heap-restoration-sift_up-and-heap-insertion-olog-n)
   - 4.2 [Down-Heap Restoration (`sift_down` / `heapify`) and Deleting the Root (`extract_max` / `extract_min`, $O(\log n)$)](#42-down-heap-restoration-sift_down--heapify-and-deleting-the-root-extract_max--extract_min-olog-n)
   - 4.3 [Priority Updates (`increase_key` / `decrease_key`) and Arbitrary Index Deletion ($O(\log n)$)](#43-priority-updates-increase_key--decrease_key-and-arbitrary-index-deletion-olog-n)
   - 4.4 [Floyd's Bottom-Up Heap Construction (`build_heap`) and $O(n)$ Linear-Time Proof](#44-floyds-bottom-up-heap-construction-build_heap-and-on-linear-time-proof)
   - 4.5 [[Supplementary Topic] In-Place Heap Sort ($O(n \log n)$ Time, $O(1)$ Space)](#45-supplementary-topic-in-place-heap-sort-on-log-n-time-o1-space)
   - 4.6 [Priority Queues Using Binary Heaps (Max-PQ and Min-PQ)](#46-priority-queues-using-binary-heaps-max-pq-and-min-pq)
   - 4.7 [Complete C11 Implementation: Max-Heap, Heap Sort, and Min-Heap Priority Queue](#47-complete-c11-implementation-max-heap-heap-sort-and-min-heap-priority-queue)
5. [Comparison Tables and Complexity Summary](#5-comparison-tables-and-complexity-summary)
   - 5.1 [Binary Heap Operations Complexity Summary](#51-binary-heap-operations-complexity-summary)
   - 5.2 [Priority Queue Implementations Compared: Array vs. Linked List vs. BST vs. Binary Heap](#52-priority-queue-implementations-compared-array-vs-linked-list-vs-bst-vs-binary-heap)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Pitfall 1: Confusing 0-Based and 1-Based Index Formulas (`2*i` vs. `2*i + 1`)](#61-pitfall-1-confusing-0-based-and-1-based-index-formulas-2i-vs-2i--1)
   - 6.2 [Pitfall 2: Swapping with an Unchecked Child Instead of the Larger of Both Children](#62-pitfall-2-swapping-with-an-unchecked-child-instead-of-the-larger-of-both-children)
   - 6.3 [Pitfall 3: Running Only `sift_down` After Arbitrary-Index Deletion](#63-pitfall-3-running-only-sift_down-after-arbitrary-index-deletion)
   - 6.4 [Pitfall 4: Building a Heap Top-Down in $O(n \log n)$ Instead of Bottom-Up in $O(n)$](#64-pitfall-4-building-a-heap-top-down-in-on-log-n-instead-of-bottom-up-in-on)
   - 6.5 [Boundary Conditions Checklist](#65-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)
   - 7.1 [Key Takeaways and Core Invariants](#71-key-takeaways-and-core-invariants)
   - 7.2 [Self-Test Revision Questions](#72-self-test-revision-questions)
   - 7.3 [Navigation Links](#73-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 What Is a Binary Heap?

A **Binary Heap** (introduced by J. W. J. Williams in 1964 for *Heapsort*) is a specialized **Complete Binary Tree** that satisfies a local vertical ordering rule—called the **Heap-Order Property**—between every parent node and its children. Because its shape is strictly a complete binary tree without any structural gaps, a binary heap is stored **implicitly inside a flat contiguous array** using simple arithmetic index formulas rather than heap-allocated nodes and `left`/`right` pointers.

```
                   MAX-HEAP (Root = Global Maximum = 95)

                              [0] 95
                             /      \
                       [1] 80        [2] 70
                       /    \        /    \
                  [3] 50  [4] 60  [5] 40  [6] 30
                   /
              [7] 10

    Contiguous Array Storage (0-based indices):
    +----+----+----+----+----+----+----+----+
    | 95 | 80 | 70 | 50 | 60 | 40 | 30 | 10 |
    +----+----+----+----+----+----+----+----+
      0    1    2    3    4    5    6    7
```

### 1.2 Why Do We Need Heaps? The Problem They Solve

In many real-time and algorithmic workloads, we do **not** need all elements sorted at every instant; instead, we repeatedly need to:
1. **Insert** new items dynamically with arbitrary priorities.
2. **Inspect** the highest-priority (maximum or minimum) item in $O(1)$ time.
3. **Extract** (remove) that extreme item rapidly so the next highest-priority item takes its place.

If we use an **unsorted array**, insertion is $O(1)$ but finding/removing the maximum costs $O(n)$. If we use a **sorted array** or **sorted linked list**, extraction from one end is $O(1)$ but every insertion requires $O(n)$ shifting or traversal. A **Binary Heap** hits the algorithmic sweet spot by relaxing total ordering into a partial vertical ordering:
- **Find-Max / Find-Min (`peek`):** $O(1)$ worst-case time (always at index `0`).
- **Insert (`push`):** $O(\log n)$ worst-case time.
- **Extract-Max / Extract-Min (`pop`):** $O(\log n)$ worst-case time.
- **Bulk Construction (`build_heap`):** $O(n)$ worst-case time from an arbitrary array of $n$ elements.

### 1.3 Relationship to Prior Concepts: Binary Trees, BSTs, and Priority Queues

- **Priority Queue (ADT) vs. Binary Heap (Data Structure):** A **Priority Queue** is an *Abstract Data Type (ADT)* specifying *what* operations are supported (`insert`, `peek`, `extract_extreme`, `change_key`), whereas a **Binary Heap** is the canonical *concrete data structure* used to implement a Priority Queue efficiently.
- **Binary Tree vs. Complete Binary Tree:** In Unit 2, we saw that a general binary tree of $n$ nodes can degenerate into a skewed chain of height $h = n - 1$. By enforcing the **Complete Binary Tree** shape property, a binary heap guarantees height $h = \lfloor \log_2 n \rfloor$ at all times without ever needing tree rotations!

### 1.4 Real-World and Systems Applications

1. **Graph Shortest-Path & Minimum Spanning Tree Algorithms:** **Dijkstra's Algorithm**, **A\* Search**, and **Prim's MST Algorithm** use a **Min-Heap Priority Queue** to repeatedly extract the unvisited vertex with the minimum tentative distance/edge weight in $O(\log V)$ time.
2. **Data Compression (Huffman Coding):** Building an optimal prefix-free Huffman tree repeatedly extracts the two lowest-frequency symbols from a **Min-Heap** of size $k$ and re-inserts their merged parent node.
3. **Operating System Process & Timer Management:** Linux kernel high-resolution timers (`timerqueue`), discrete-event simulators, and real-time packet schedulers store events in a min-heap keyed by expiration timestamp.
4. **Top-$K$ Streaming Queries & Median Maintenance:** Finding the $k$ largest items in a massive stream of $N \gg k$ items uses a **Min-Heap** of size $k$ in $O(N \log k)$ time and $O(k)$ memory; maintaining a running median uses a paired Max-Heap (lower half) and Min-Heap (upper half).

### 1.5 Learning Objectives

After completing this chapter, you will be able to:
- State and verify the two invariants of a Binary Heap (**Shape Property** and **Heap-Order Property**).
- Derive and apply both **0-based** and **1-based** array index formulas for `parent`, `left`, `right`, internal nodes, and leaf ranges.
- Trace and implement `sift_up`, `sift_down` (`heapify`), `insert`, `extract_max`/`extract_min`, `increase_key`/`decrease_key`, and arbitrary index deletion in C11.
- Prove mathematically why Floyd's bottom-up `build_heap` runs in $O(n)$ linear time rather than $O(n \log n)$.
- Implement in-place **Heap Sort** and a production-quality **Min-Heap Priority Queue** in C11.

---

## 2. Definitions and Terminology

### 2.1 Structural Invariant 1: The Shape Property (Complete Binary Tree)

> [!IMPORTANT]
> **The Shape Property:** A binary heap of $n$ elements must always form a **Complete Binary Tree**—every level $0, 1, \dots, h - 1$ is completely filled with $2^\ell$ nodes, and the bottom level $h$ has all of its nodes packed as far to the **left** as possible with no gaps.

```
     VALID COMPLETE BINARY TREE                  INVALID (NOT COMPLETE)
       (All bottom nodes packed                 (Gap at left child of 20
        strictly left-to-right)                  violates completeness!)

                 (10)                                     (10)
                /    \                                   /    \
             (20)    (30)                             (20)    (30)
             /  \    /                                   \    /  \
           (40)(50)(60)                                 (50)(60)(70)
                                                         ^
                                                   Missing left child!
```

Because a complete binary tree of height $h$ contains between $2^h$ and $2^{h+1} - 1$ nodes, its height for any $n \ge 1$ is strictly fixed at:

$$h = \lfloor \log_2 n \rfloor$$

### 2.2 Structural Invariant 2: The Heap-Order Property (Max-Heap vs. Min-Heap)

The **Heap-Order Property** constrains the relative priority of every node $i$ ($i > 0$) and its parent $\text{parent}(i)$:

1. **Max-Heap Property:** Every node's key is **less than or equal to** its parent's key:
   $$\forall i \in \{1, 2, \dots, n-1\}: \quad A[\text{parent}(i)] \ge A[i]$$
   By transitivity along every root-to-leaf path, the **maximum element of the entire heap is always stored at the root ($A[0]$)**, and the minimum element must reside among the leaves.

2. **Min-Heap Property:** Every node's key is **greater than or equal to** its parent's key:
   $$\forall i \in \{1, 2, \dots, n-1\}: \quad A[\text{parent}(i)] \le A[i]$$
   By transitivity, the **minimum element of the entire heap is always stored at the root ($A[0]$)**, and the maximum element must reside among the leaves.

```
            MAX-HEAP                                    MIN-HEAP
   (Every parent >= its children)              (Every parent <= its children)
     Root = 90 (Global Maximum)                  Root = 10 (Global Minimum)

               (90)                                        (10)
              /    \                                      /    \
           (70)    (80)                                (30)    (20)
           /  \    /  \                                /  \    /  \
         (40)(65)(50)(75)                            (40)(65)(50)(25)
```

### 2.3 Why a Binary Heap Is NOT a Binary Search Tree (BST)

A common misconception is confusing a Binary Heap with a Binary Search Tree. Their ordering invariants are fundamentally orthogonal:

| Property | Binary Search Tree (BST) | Binary Heap (Max-Heap / Min-Heap) |
| :--- | :--- | :--- |
| **Ordering Axis** | **Horizontal (Left $<$ Root $<$ Right):** All keys in left subtree $<$ root $<$ all keys in right subtree. | **Vertical (Parent vs. Children):** Parent $\ge$ both children (Max-Heap) or Parent $\le$ both children (Min-Heap). |
| **Left vs. Right Sibling Order** | Strictly ordered ($\text{left} < \text{right}$). | **Completely unconstrained** (left child can be $>$, $<$, or $=$ right child). |
| **Location of Extreme Key** | Minimum at leftmost node ($O(h)$); Maximum at rightmost node ($O(h)$). | Global extreme is **always at the root** ($O(1)$). |
| **Shape Constraint** | Arbitrary binary tree ($h \in [\lfloor \log_2 n \rfloor, n-1]$). | Strictly a **Complete Binary Tree** ($h = \lfloor \log_2 n \rfloor$). |
| **Search for Arbitrary Key $k$** | **$O(h)$** ($O(\log n)$ if balanced) via binary decision at each node. | **$O(n)$ linear scan** (cannot prune one whole subtree safely in general). |
| **Memory Representation** | Node structs with `left` and `right` pointers. | Flat contiguous array (zero pointers). |

### 2.4 Core Terminology Table

| Term | Formal Definition |
| :--- | :--- |
| **Complete Binary Tree** | A binary tree in which every level except possibly the deepest is completely filled, and all nodes at the deepest level are as far left as possible. |
| **Heap Height ($h$)** | Length (number of edges) of the longest path from the root to a leaf: $h = \lfloor \log_2 n \rfloor$ for $n \ge 1$. |
| **Node Height ($\text{height}(i)$)** | Number of edges on the longest downward simple path from node $i$ to a leaf in its subtree. Leaves have height $0$; the root has height $h = \lfloor \log_2 n \rfloor$. |
| **Node Depth ($\text{depth}(i)$)** | Number of edges on the unique upward path from node $i$ to the root: $\text{depth}(i) = \lfloor \log_2(i + 1) \rfloor$ (0-based). The root has depth $0$. |
| **Internal Node** | Any node that has at least one child (in a 0-based heap of size $n$, indices $0 \dots \lfloor n/2 \rfloor - 1$). |
| **Leaf Node** | Any node with no children (in a 0-based heap of size $n$, indices $\lfloor n/2 \rfloor \dots n - 1$). |
| **`sift_up` (Percolate-Up / Bubble-Up)** | Restoring heap order when a node's key becomes too large (Max-Heap) or too small (Min-Heap) relative to its parent by repeatedly swapping it upward with its parent. |
| **`sift_down` (`heapify` / Percolate-Down)** | Restoring heap order when a node's key violates heap order with its children by repeatedly swapping it downward with its larger child (Max-Heap) or smaller child (Min-Heap). |

### 2.5 Mathematical Properties, Invariants, and Formulas

For any Binary Heap containing $n \ge 1$ elements stored in a 0-based array $A[0 \dots n-1]$:

1. **Height Bound:**
   $$h = \lfloor \log_2 n \rfloor \quad \Longleftrightarrow \quad 2^h \le n \le 2^{h+1} - 1$$
2. **Exact Partition into Internal Nodes and Leaves:**
   - **Internal nodes:** $\lfloor n/2 \rfloor$ nodes occupying indices $0, 1, \dots, \lfloor n/2 \rfloor - 1$.
   - **Leaf nodes:** $\lceil n/2 \rceil = n - \lfloor n/2 \rfloor$ nodes occupying indices $\lfloor n/2 \rfloor, \dots, n - 1$.
   *(Proof: Node $i$ is a leaf iff its left child index $2i + 1 \ge n \iff 2i \ge n - 1 \iff i \ge \lfloor n/2 \rfloor$.)*
3. **Number of Nodes at Height $h'$:**
   In an $n$-element heap, the number of nodes of height $h'$ (where leaves are at $h' = 0$) is bounded above by:
   $$\text{nodes at height } h' \le \left\lceil \frac{n}{2^{h'+1}} \right\rceil$$
   Consequently, at least **half of all nodes** ($\lceil n/2 \rceil$) are leaves at height $0$, at most $\lceil n/4 \rceil$ nodes are at height $1$, at most $\lceil n/8 \rceil$ are at height $2$, and so on.

---

## 3. Representation and Memory Layout in C

### 3.1 Why Heaps Are Implicit Array Data Structures (Zero Pointer Overhead)

In a general binary tree, missing nodes at arbitrary positions force us to allocate individual `Node` structures connected via two 8-byte pointers (`left` and `right` on 64-bit architectures)—imposing a **16-byte pointer overhead per 4-byte `int` key** (400% memory overhead!) and scattering nodes across the heap with poor CPU cache locality.

Because a Binary Heap is a **Complete Binary Tree**, reading its nodes in **level-order (top-to-bottom, left-to-right)** maps every node onto a unique, contiguous array index with **zero empty slots**. Parent-child navigation becomes a single CPU bit-shift and addition.

### 3.2 0-Based vs. 1-Based Array Index Formulas Compared

Standard C arrays are **0-based** (`A[0 .. n-1]`), whereas some classic algorithm textbooks (such as early editions of CLRS) use **1-based** indexing (`A[1 .. n]`, wasting `A[0]` or storing `size` in `A[0]`). Mastering both conventions—and never mixing them—is essential:

| Relationship / Range | **0-Based Array Indexing (`A[0 .. n-1]`)** *(Standard in C)* | **1-Based Array Indexing (`A[1 .. n]`)** *(Textbook / Bit-Shift Style)* |
| :--- | :--- | :--- |
| **Root Index** | `0` | `1` |
| **Last Element Index** | `n - 1` | `n` |
| **Parent of Node `i`** | `(i - 1) / 2` *(for `i > 0`)* | `i / 2` *(or `i >> 1`, for `i > 1`)* |
| **Left Child of Node `i`** | `2 * i + 1` | `2 * i` *(or `i << 1`)* |
| **Right Child of Node `i`** | `2 * i + 2` | `2 * i + 1` *(or `(i << 1) \| 1`)* |
| **Internal Node Indices** | `0` to `(n / 2) - 1` | `1` to `n / 2` |
| **Last Internal Node** | `(n / 2) - 1` | `n / 2` |
| **Leaf Node Indices** | `n / 2` to `n - 1` | `(n / 2) + 1` to `n` |
| **Has Left Child?** | `2 * i + 1 < n` | `2 * i <= n` |
| **Has Right Child?** | `2 * i + 2 < n` | `2 * i + 1 <= n` |

### 3.3 Tree-to-Array Mapping Diagram

Consider a 10-element Max-Heap ($n = 10$). Notice how internal nodes occupy indices `0 .. 4` ($\lfloor 10/2 \rfloor - 1 = 4$) and leaves occupy indices `5 .. 9`:

```
                       COMPLETE BINARY TREE VIEW (n = 10)

                                   [0] 95                  <-- Level 0 (h'=3)
                                 /        \
                          [1] 85            [2] 78         <-- Level 1 (h'=2)
                         /      \          /      \
                    [3] 64      [4] 72  [5] 45    [6] 53   <-- Level 2 (h'=1)
                    /    \      /
                [7] 31 [8] 50 [9] 28                       <-- Level 3 (h'=0)

    0-BASED ARRAY MEMORY LAYOUT (Contiguous Cache-Friendly Block):
         |<------- Internal Nodes (0..4) ------->|<------ Leaf Nodes (5..9) ------>|
    Idx: |   0  |   1  |   2  |   3  |   4  |   5  |   6  |   7  |   8  |   9  |
         +------+------+------+------+------+------+------+------+------+------+
    Val: |  95  |  85  |  78  |  64  |  72  |  45  |  53  |  31  |  50  |  28  |
         +------+------+------+------+------+------+------+------+------+------+
             |      ^      ^      ^      ^                    ^      ^
             +------+------+      +------+--------------------+------+
             left=1, right=2      For i=3: left = 2(3)+1 = 7, right = 2(3)+2 = 8
```

### 3.4 C `typedef struct` Declarations and Memory Layout

In C11, we define both a fixed-capacity array heap and a dynamically resizable heap, as well as a Priority Queue struct pairing a priority key with a satellite payload:

```c
#include <stddef.h>
#include <stdbool.h>

#define HEAP_CAPACITY 128

/* 1. Static-capacity Max-Heap of integer keys */
typedef struct {
    int data[HEAP_CAPACITY]; /* Contiguous 0-based array [0 .. size-1] */
    int size;                /* Current number of active elements      */
} MaxHeap;

/* 2. Dynamically resizable Heap (grows via realloc when size == capacity) */
typedef struct {
    int *data;               /* Heap-allocated array of length capacity */
    int  size;               /* Current number of elements              */
    int  capacity;           /* Allocated slots in data[]               */
} DynamicHeap;

/* 3. Priority Queue Item (Key + Satellite Data Payload) */
typedef struct {
    int         priority;    /* Ordering key (e.g., smaller = more urgent in Min-PQ) */
    int         task_id;     /* Unique identifier                                    */
    const char *description; /* Satellite payload pointer                            */
} PQItem;

typedef struct {
    PQItem items[HEAP_CAPACITY];
    int    size;
} MinPriorityQueue;
```

### 3.5 Static Capacity vs. Dynamically Resized Heap Storage

| Criterion | Static Array Heap (`int data[CAPACITY]`) | Dynamic Array Heap (`int *data` + `realloc`) | Pointer-Based Binary Tree Heap |
| :--- | :--- | :--- | :--- |
| **Memory Overhead per Element** | **0 bytes** beyond key/item size | **0 bytes** per element (up to $2\times$ slack capacity) | **16–24 bytes** (`left`, `right`, `parent` pointers) |
| **Cache Locality** | **Excellent** (contiguous stack/struct memory) | **Excellent** (contiguous heap block) | **Poor** (pointer chasing across heap) |
| **Overflow Behavior** | Bounded by `CAPACITY` (`insert` returns `false`) | Grows by $2\times$ via `realloc` in $O(1)$ amortized time | Limited only by system RAM |
| **Best Use Case** | Embedded systems, real-time OS kernels, known upper bound | General-purpose library containers | Almost never used for standard binary heaps |

---

## 4. Operations and Algorithms

### 4.1 Up-Heap Restoration (`sift_up`) and Heap Insertion ($O(\log n)$)

#### Purpose & Preconditions
- **Purpose:** Insert a new key `k` into a Max-Heap of size $n$ while preserving both the **Shape Property** and the **Max-Heap Order Property**.
- **Preconditions:** `h != NULL` and `h->size < MAX_CAPACITY`.

#### Step-by-Step Algorithm
1. **Preserve the Shape Property first:** Place the new key at the very next available leaf position at the bottom-right of the complete binary tree: `i = h->size`, `h->data[i] = key`, and increment `h->size`.
2. **Restore the Heap-Order Property via `sift_up(h, i)`:**
   - While `i > 0` (not at the root) and `h->data[parent(i)] < h->data[i]` (parent is smaller than child, violating Max-Heap order):
     - Swap `h->data[parent(i)]` and `h->data[i]`.
     - Move upward: `i = (i - 1) / 2`.
   - Terminate as soon as `i == 0` or `h->data[parent(i)] >= h->data[i]`.

#### Pseudocode
```text
ALGORITHM MaxHeap-Insert(H, key)
Input:  Max-Heap H with array H.data[0 .. H.size - 1], new value key
Output: true if inserted, false if H is full

1. if H.size == MAX_CAPACITY then
2.     return false
3. i <- H.size
4. H.data[i] <- key
5. H.size <- H.size + 1
6. while i > 0 and H.data[(i - 1) / 2] < H.data[i] do
7.     Swap(H.data[(i - 1) / 2], H.data[i])
8.     i <- (i - 1) / 2
9. return true
```

#### Before-and-After ASCII Trace: `max_heap_insert(&h, 95)`

Suppose our Max-Heap initially holds `[80, 50, 40, 10, 20, 15, 30]` ($n = 7$) and we insert `95`:

```
  STEP 1: Append 95 at index i = 7               STEP 2: Compare i=7 (95) with parent
          (Shape property satisfied;                     p=(7-1)/2 = 3 (10). Since 95 > 10,
           violates parent 10 < 95)                      SWAP index 7 and index 3!

                  [0] 80                                         [0] 80
                 /      \                                       /      \
           [1] 50        [2] 40                           [1] 50        [2] 40
           /    \        /    \                           /    \        /    \
      [3] 10  [4] 20  [5] 15  [6] 30                 [3] 95* [4] 20  [5] 15  [6] 30
       /                                              /
  [7] 95*                                        [7] 10

  Array: [80, 50, 40, 10, 20, 15, 30, 95*]       Array: [80, 50, 40, 95*, 20, 15, 30, 10]

  STEP 3: Compare i=3 (95) with parent           STEP 4: Compare i=1 (95) with parent
          p=(3-1)/2 = 1 (50). Since 95 > 50,             p=(1-1)/2 = 0 (80). Since 95 > 80,
          SWAP index 3 and index 1!                      SWAP index 1 and index 0! (i=0 -> STOP)

                  [0] 80                                         [0] 95*
                 /      \                                       /      \
           [1] 95*       [2] 40                           [1] 80        [2] 40
           /    \        /    \                           /    \        /    \
      [3] 50  [4] 20  [5] 15  [6] 30                 [3] 50  [4] 20  [5] 15  [6] 30
       /                                              /
  [7] 10                                         [7] 10

  Array: [80, 95*, 40, 50, 20, 15, 30, 10]       Final: [95, 80, 40, 50, 20, 15, 30, 10]
```

#### Complexity & Edge Cases
- **Time Complexity:**
  - **Best Case:** $\Theta(1)$ — when the inserted key is $\le$ its immediate parent at index $\lfloor (n-1)/2 \rfloor$, requiring $1$ comparison and $0$ swaps (indeed, because half the nodes in a heap are leaves, random insertions require $O(1)$ swaps on average!).
  - **Worst Case:** $O(\log n)$ — when the new key is the new global maximum and bubbles all the way from leaf depth $h = \lfloor \log_2 n \rfloor$ to index `0`.
- **Space Complexity:** $O(1)$ auxiliary space (iterative loop).
- **Edge Cases:** Inserting into an empty heap (`size == 0` $\implies$ `i = 0`, loop condition `i > 0` is immediately false); duplicate keys (`parent >= child` stops immediately without unnecessary swaps).

---

### 4.2 Down-Heap Restoration (`sift_down` / `heapify`) and Deleting the Root (`extract_max` / `extract_min`, $O(\log n)$)

#### Purpose & Preconditions
- **Purpose:** Remove and return the extreme element at the root (`data[0]`) of a Binary Heap while preserving the Complete Binary Tree shape and Heap-Order invariant.
- **Preconditions:** `h != NULL` and `h->size > 0`.

#### Why We Cannot Simply Delete `data[0]` and Shift Elements Left
If we deleted `data[0]` and shifted `data[1 .. n-1]` left by one slot, not only would shifting cost $O(n)$ time, it would completely scramble parent-child relationships across the tree! Instead, we remove the **last leaf** at `data[size - 1]` (preserving the Complete Binary Tree shape in $O(1)$ time), move its value into the root `data[0]`, and push it down via `sift_down(h, 0)`.

#### Step-by-Step Algorithm (`extract_max` + `sift_down`)
1. Save the maximum key from the root: `*out_max = h->data[0]`.
2. Decrement `h->size`. If `h->size == 0`, return `true` immediately.
3. Move the former last leaf into the root: `h->data[0] = h->data[h->size]`.
4. **Restore Heap Order via `sift_down(h, 0)`:**
   - Set `i = 0`.
   - Loop:
     - Compute `left = 2 * i + 1` and `right = 2 * i + 2`.
     - Find `largest` among `{i, left (if left < size), right (if right < size)}`.
     - If `largest == i`, `h->data[i]` is already $\ge$ both children; `break`.
     - Swap `h->data[i]` with `h->data[largest]` (crucial: **always swap with the larger of the two children** in a Max-Heap so the promoted child is $\ge$ its sibling!).
     - Advance downward: `i = largest`.

#### Pseudocode
```text
ALGORITHM MaxHeap-SiftDown(H, i)
Input:  Max-Heap H, root index i whose left and right subtrees are already valid Max-Heaps
Output: Subtree rooted at i becomes a valid Max-Heap

1. loop
2.     largest <- i
3.     left    <- 2 * i + 1
4.     right   <- 2 * i + 2
5.     if left < H.size and H.data[left] > H.data[largest] then
6.         largest <- left
7.     if right < H.size and H.data[right] > H.data[largest] then
8.         largest <- right
9.     if largest == i then
10.        break
11.    Swap(H.data[i], H.data[largest])
12.    i <- largest
```

#### Before-and-After ASCII Trace: `max_heap_extract_max(&h, &max_val)`

Starting from `[95, 80, 40, 50, 20, 10, 30]` ($n = 7$):

```
  INITIAL HEAP (size = 7)                        STEP 1: Save root 95, move last leaf
                                                         data[6] = 30 to root, size = 6

               [0] 95  <-- extract                            [0] 30*
              /      \                                       /      \
        [1] 80        [2] 40                           [1] 80        [2] 40
        /    \        /    \                           /    \        /
   [3] 50  [4] 20  [5] 10  [6] 30 <-- last        [3] 50  [4] 20  [5] 10

  Array: [95, 80, 40, 50, 20, 10, 30]            Array: [30*, 80, 40, 50, 20, 10]

  STEP 2: At i=0 (30), children are              STEP 3: At i=1 (30), children are
          left=1 (80) and right=2 (40).                  left=3 (50) and right=4 (20).
          Largest in {30, 80, 40} is 80 (idx 1).         Largest in {30, 50, 20} is 50 (idx 3).
          SWAP index 0 and index 1!                      SWAP index 1 and index 3! (idx 3 is leaf -> STOP)

               [0] 80                                         [0] 80
              /      \                                       /      \
        [1] 30*       [2] 40                           [1] 50        [2] 40
        /    \        /                                /    \        /
   [3] 50  [4] 20  [5] 10                         [3] 30* [4] 20  [5] 10

  Array: [80, 30*, 40, 50, 20, 10]               Final: [80, 50, 40, 30, 20, 10]
```

#### Complexity & Edge Cases
- **Time Complexity:** $O(\log n)$ worst-case time (at most $h = \lfloor \log_2 n \rfloor$ levels of descent, performing at most $2$ key comparisons per level $\implies \le 2 \lfloor \log_2 n \rfloor$ comparisons).
- **Space Complexity:** $O(1)$ auxiliary space using the iterative loop (a recursive `heapify` implementation would use $O(\log n)$ call-stack frames).
- **Edge Cases:** Single-element heap (`size == 1` $\implies$ `size` becomes `0`, no `sift_down` needed); node with only a left child (`left == size - 1`, `right == size` $\implies$ `right < size` check prevents out-of-bounds read!).

---

### 4.3 Priority Updates (`increase_key` / `decrease_key`) and Arbitrary Index Deletion ($O(\log n)$)

#### 1. `increase_key(h, i, new_key)` in a Max-Heap (or `decrease_key` in a Min-Heap)
- **Purpose:** Raise the priority of the element at index `i` (`new_key >= h->data[i]` in a Max-Heap; `new_key <= pq->items[i].priority` in a Min-Heap).
- **Algorithm:** Because increasing a key in a Max-Heap cannot violate the heap-order property with `i`'s children (since `new_key >= old_key >= children`), it can **only violate heap order with `i`'s ancestors**. Thus, we simply overwrite `h->data[i] = new_key` and call `sift_up(h, i)` in $O(\log n)$ time.

#### 2. Deleting an Arbitrary Index `delete_at(h, i)` in $O(\log n)$ Time
- **Purpose:** Remove the element at arbitrary index `i` ($0 \le i < n$) from a heap.
- **Algorithm:**
  1. Replace `h->data[i]` with the last leaf `replacement = h->data[h->size - 1]` and decrement `h->size`.
  2. If `i == h->size` (we just deleted the last leaf itself), we are done!
  3. Otherwise, compare `replacement` against `old_key` (or against `h->data[parent(i)]`):
     - In a Max-Heap, if `replacement > old_key` (or `i > 0 && replacement > h->data[parent(i)]`), call `sift_up(h, i)`.
     - Otherwise, call `sift_down(h, i)`.

> [!WARNING]
> Why might `delete_at(h, i)` need `sift_up`? Because a Binary Heap is **only vertically ordered**! A leaf in the right subtree of the root can easily have a key **larger** than a deep internal node in the left subtree of the root. Moving that right-subtree leaf into a deep left-subtree position can make it larger than its new parent!

---

### 4.4 Floyd's Bottom-Up Heap Construction (`build_heap`) and $O(n)$ Linear-Time Proof

#### Purpose & Preconditions
Given an unsorted array `arr[0 .. n-1]` of $n$ elements, convert it into a valid Binary Heap in-place.

#### Naive Top-Down Approach ($O(n \log n)$) vs. Floyd's Bottom-Up Algorithm ($O(n)$)
- **Naive Top-Down Approach:** Start with an empty heap and call `insert` for each of the $n$ elements one by one: $\sum_{i=1}^{n} O(\log i) = O(n \log n)$ worst-case time.
- **Floyd's Bottom-Up Algorithm (Robert W. Floyd, 1964):**
  1. Treat the entire array `arr[0 .. n-1]` as an already-shaped Complete Binary Tree.
  2. Observe that **all leaf nodes** at indices $\lfloor n/2 \rfloor \dots n - 1$ have no children, so each leaf is trivially a valid 1-element heap!
  3. Iterate **backward** from the last internal node `i = (n / 2) - 1` down to `0`, calling `sift_down(h, i)` on each node. Because we process bottom-up, whenever `sift_down(h, i)` is called, both the left subtree (`2*i + 1`) and right subtree (`2*i + 2`) of `i` are **already valid heaps**—satisfying the exact precondition of `sift_down`!

#### Step-by-Step ASCII Trace: `build_max_heap` on `[15, 10, 30, 50, 20, 40, 80]` ($n = 7$)

Here $n = 7$, so leaves are indices `3, 4, 5, 6` and internal nodes are `i = (7/2) - 1 = 2` down to `0`:

```
  INITIAL UNORDERED TREE                         STEP 1: i = 2 (val 30). Children are
  Internal nodes: i = 2, 1, 0                            idx 5 (40) and idx 6 (80).
  Leaves (already heaps): 3, 4, 5, 6                     Max child is 80 (idx 6) -> Swap(30, 80)

               [0] 15                                         [0] 15
              /      \                                       /      \
        [1] 10        [2] 30  <-- start i=2            [1] 10        [2] 80*
        /    \        /    \                           /    \        /    \
   [3] 50  [4] 20  [5] 40  [6] 80                 [3] 50  [4] 20  [5] 40  [6] 30*

  Array: [15, 10, 30, 50, 20, 40, 80]            Array: [15, 10, 80, 50, 20, 40, 30]

  STEP 2: i = 1 (val 10). Children are           STEP 3: i = 0 (val 15). Sifts down via
          idx 3 (50) and idx 4 (20).                     idx 2 (80), then compares with
          Max child is 50 (idx 3) -> Swap(10, 50)        idx 5 (40) and idx 6 (30) -> Swap(15, 40)

               [0] 15                                         [0] 80*
              /      \                                       /      \
        [1] 50*       [2] 80                           [1] 50        [2] 40*
        /    \        /    \                           /    \        /    \
   [3] 10* [4] 20  [5] 40  [6] 30                 [3] 10  [4] 20  [5] 15* [6] 30

  Array: [15, 50, 80, 10, 20, 40, 30]            Final: [80, 50, 40, 10, 20, 15, 30]
```

#### Mathematical Proof That `build_heap` Runs in $O(n)$ Linear Time

> [!TIP]
> **Exam & Interview Essential Proof:** Why is calling an $O(\log n)$ function (`sift_down`) $\lfloor n/2 \rfloor$ times $O(n)$ instead of $O(n \log n)$? Because the work done by `sift_down(h, i)` is proportional to the **height $h'$ of node $i$** (distance down to the leaves), and the vast majority of nodes in a complete binary tree live near the bottom with tiny height!

1. **Cost per node at height $h'$:** A node at height $h'$ can sift down at most $h'$ levels, costing $O(h') \le c \cdot h'$ operations for some constant $c > 0$.
2. **Node count at height $h'$:** An $n$-element complete binary tree has tree height $H = \lfloor \log_2 n \rfloor$ and contains at most $\left\lceil \frac{n}{2^{h'+1}} \right\rceil$ nodes at height $h'$.
3. **Total work $T(n)$ summed across all heights $h' = 0 \dots H$:**
   $$T(n) = \sum_{h'=0}^{\lfloor \log_2 n \rfloor} \left\lceil \frac{n}{2^{h'+1}} \right\rceil c \cdot h' \le \sum_{h'=0}^{\lfloor \log_2 n \rfloor} \left( \frac{n}{2^{h'+1}} + 1 \right) c \cdot h' \le \frac{c \cdot n}{2} \sum_{h'=0}^{\infty} \frac{h'}{2^{h'}} + c \sum_{h'=0}^{\lfloor \log_2 n \rfloor} h'$$
4. **Evaluating the infinite geometric-arithmetic series:**
   Recall the standard identity for $|x| < 1$:
   $$\sum_{k=0}^{\infty} x^k = \frac{1}{1 - x} \quad \stackrel{\frac{d}{dx}}{\Longrightarrow} \quad \sum_{k=0}^{\infty} k x^k = \frac{x}{(1 - x)^2}$$
   Substituting $x = \frac{1}{2}$ gives:
   $$\sum_{h'=0}^{\infty} \frac{h'}{2^{h'}} = \frac{1/2}{(1 - 1/2)^2} = \frac{1/2}{1/4} = 2$$
5. **Concluding the bound:**
   $$T(n) \le \frac{c \cdot n}{2} (2) + O(\log^2 n) = c \cdot n + O(\log^2 n) = O(n)$$
   More specifically, each `sift_down` at height $h'$ performs at most $2h'$ key comparisons, so `build_heap` performs **at most $2n$ comparisons** in the worst case—strictly linear $O(n)$ time! $\blacksquare$

---

### 4.5 [Supplementary Topic] In-Place Heap Sort ($O(n \log n)$ Time, $O(1)$ Space)

> [!NOTE]
> **Supplementary Topic:** Although **Heap Sort** is a sorting algorithm, it is the direct algorithmic application of a Binary Max-Heap and is frequently tested alongside `build_heap` and `sift_down`.

#### How Heap Sort Works
To sort an array `arr[0 .. n-1]` in **ascending order** in-place without allocating any auxiliary array:
1. **Phase 1 — Build a Max-Heap ($O(n)$):** Rearrange `arr[0 .. n-1]` into a valid Max-Heap using Floyd's bottom-up construction so `arr[0]` holds the largest element.
2. **Phase 2 — Repeated Extraction into the Sorted Suffix ($O(n \log n)$):**
   For `end = n - 1` down to `1`:
   - Swap the current maximum at the root `arr[0]` with `arr[end]`. Now `arr[end .. n-1]` holds the largest elements in final sorted order!
   - Shrink the active heap boundary to `0 .. end - 1` (size `end`) and call `sift_down(arr, end, 0)` to restore the Max-Heap property in $O(\log(\text{end}))$ time.

```
   In-Place Partition of arr[0 .. n-1] During Heap Sort:
   +-----------------------------------+-----------------------------------+
   |   Active Max-Heap: arr[0..end-1]  |   Sorted Suffix: arr[end..n-1]    |
   |   (arr[0] is max of remaining)    |   (Final ascending order!)        |
   +-----------------------------------+-----------------------------------+
     |                               ^
     +---- swap(arr[0], arr[end]) ---+   then sift_down(arr, end, 0)
```

- **Time Complexity:** $O(n) + \sum_{k=1}^{n-1} O(\log k) = O(n \log n)$ in the **best, average, and worst cases**!
- **Space Complexity:** $O(1)$ auxiliary space (100% in-place).
- **Stability:** **Not stable** (long-distance swaps between root and leaf can reorder equal keys).

---

### 4.6 Priority Queues Using Binary Heaps (Max-PQ and Min-PQ)

A **Priority Queue** implemented with a Binary Heap supports two symmetric variants depending on the application domain:
- **Max-Priority Queue (Max-Heap):** Higher numeric key = higher urgency. Used in job schedulers where priority weight `100` preempts weight `10`, or in bandwidth allocation.
- **Min-Priority Queue (Min-Heap):** Smaller numeric key = higher urgency. Used in Dijkstra's shortest-path algorithm (smallest distance $d[u]$ first), OS timer queues (earliest deadline timestamp first), and UNIX `nice` levels (`-20` to `+19`).

### 4.7 Complete C11 Implementation: Max-Heap, Heap Sort, and Min-Heap Priority Queue

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <string.h>

#define MAX_CAPACITY 32

/* =========================================================================
 * PART 1: ARRAY-BASED BINARY MAX-HEAP (0-Based Indexing)
 * ========================================================================= */
typedef struct {
    int data[MAX_CAPACITY];
    int size;
} MaxHeap;

static inline int parent_idx(int i) { return (i - 1) / 2; }
static inline int left_idx(int i)   { return 2 * i + 1; }
static inline int right_idx(int i)  { return 2 * i + 2; }

static inline void swap_int(int *a, int *b) {
    int tmp = *a;
    *a = *b;
    *b = tmp;
}

void max_heap_init(MaxHeap *h) {
    h->size = 0;
}

/* Sift-Up (Percolate-Up): Restore Max-Heap order from index i up to root */
void max_heap_sift_up(MaxHeap *h, int i) {
    while (i > 0 && h->data[parent_idx(i)] < h->data[i]) {
        swap_int(&h->data[parent_idx(i)], &h->data[i]);
        i = parent_idx(i);
    }
}

/* Sift-Down (Max-Heapify): Restore Max-Heap order at subtree rooted at i */
void max_heap_sift_down(MaxHeap *h, int i) {
    while (true) {
        int largest = i;
        int left    = left_idx(i);
        int right   = right_idx(i);

        if (left < h->size && h->data[left] > h->data[largest]) {
            largest = left;
        }
        if (right < h->size && h->data[right] > h->data[largest]) {
            largest = right;
        }
        if (largest == i) {
            break; /* Heap-order property satisfied */
        }
        swap_int(&h->data[i], &h->data[largest]);
        i = largest;
    }
}

/* Insert a new key into Max-Heap in O(log n) */
bool max_heap_insert(MaxHeap *h, int key) {
    if (h->size >= MAX_CAPACITY) {
        return false;
    }
    int i = h->size++;
    h->data[i] = key;
    max_heap_sift_up(h, i);
    return true;
}

/* Extract the maximum key (root) in O(log n) */
bool max_heap_extract_max(MaxHeap *h, int *out_max) {
    if (h->size <= 0) {
        return false;
    }
    *out_max = h->data[0];
    h->size--;
    if (h->size > 0) {
        h->data[0] = h->data[h->size];
        max_heap_sift_down(h, 0);
    }
    return true;
}

/* Increase key at index i to new_key in O(log n) */
bool max_heap_increase_key(MaxHeap *h, int i, int new_key) {
    if (i < 0 || i >= h->size || new_key < h->data[i]) {
        return false;
    }
    h->data[i] = new_key;
    max_heap_sift_up(h, i);
    return true;
}

/* Delete arbitrary element at index i in O(log n) */
bool max_heap_delete_at(MaxHeap *h, int i, int *out_removed) {
    if (i < 0 || i >= h->size) {
        return false;
    }
    *out_removed = h->data[i];
    int replacement = h->data[h->size - 1];
    h->size--;
    if (i < h->size) {
        int old_key = h->data[i];
        h->data[i] = replacement;
        if (replacement > old_key) {
            max_heap_sift_up(h, i);
        } else {
            max_heap_sift_down(h, i);
        }
    }
    return true;
}

/* Floyd's Bottom-Up Heap Construction in O(n) linear time */
void build_max_heap(MaxHeap *h, const int *arr, int n) {
    if (n > MAX_CAPACITY) n = MAX_CAPACITY;
    memcpy(h->data, arr, (size_t)n * sizeof(int));
    h->size = n;
    for (int i = (h->size / 2) - 1; i >= 0; i--) {
        max_heap_sift_down(h, i);
    }
}

/* In-Place Heap Sort in O(n log n) time and O(1) auxiliary space */
void heap_sort_raw_sift_down(int *arr, int n, int i) {
    while (true) {
        int largest = i;
        int left    = 2 * i + 1;
        int right   = 2 * i + 2;
        if (left < n && arr[left] > arr[largest])   largest = left;
        if (right < n && arr[right] > arr[largest]) largest = right;
        if (largest == i) break;
        swap_int(&arr[i], &arr[largest]);
        i = largest;
    }
}

void heap_sort(int *arr, int n) {
    /* Phase 1: Build Max-Heap in-place in O(n) */
    for (int i = (n / 2) - 1; i >= 0; i--) {
        heap_sort_raw_sift_down(arr, n, i);
    }
    /* Phase 2: Repeatedly swap max root to end and shrink heap in O(n log n) */
    for (int end = n - 1; end > 0; end--) {
        swap_int(&arr[0], &arr[end]);
        heap_sort_raw_sift_down(arr, end, 0);
    }
}

void max_heap_print(const MaxHeap *h, const char *label) {
    printf("%s (size=%d): [", label, h->size);
    for (int i = 0; i < h->size; i++) {
        printf("%d%s", h->data[i], (i + 1 < h->size) ? ", " : "");
    }
    printf("]\n");
}

/* =========================================================================
 * PART 2: MIN-HEAP PRIORITY QUEUE FOR TASK SCHEDULING
 * (Lower numeric priority value = Higher urgency)
 * ========================================================================= */
typedef struct {
    int         task_id;
    int         priority; /* Key: smaller value = higher priority */
    const char *name;
} Task;

typedef struct {
    Task items[MAX_CAPACITY];
    int  size;
} MinPriorityQueue;

static inline void swap_task(Task *a, Task *b) {
    Task tmp = *a;
    *a = *b;
    *b = tmp;
}

void pq_init(MinPriorityQueue *pq) {
    pq->size = 0;
}

void pq_sift_up(MinPriorityQueue *pq, int i) {
    while (i > 0 && pq->items[parent_idx(i)].priority > pq->items[i].priority) {
        swap_task(&pq->items[parent_idx(i)], &pq->items[i]);
        i = parent_idx(i);
    }
}

void pq_sift_down(MinPriorityQueue *pq, int i) {
    while (true) {
        int smallest = i;
        int left     = left_idx(i);
        int right    = right_idx(i);

        if (left < pq->size && pq->items[left].priority < pq->items[smallest].priority) {
            smallest = left;
        }
        if (right < pq->size && pq->items[right].priority < pq->items[smallest].priority) {
            smallest = right;
        }
        if (smallest == i) {
            break;
        }
        swap_task(&pq->items[i], &pq->items[smallest]);
        i = smallest;
    }
}

bool pq_push(MinPriorityQueue *pq, int task_id, int priority, const char *name) {
    if (pq->size >= MAX_CAPACITY) return false;
    int i = pq->size++;
    pq->items[i] = (Task){ .task_id = task_id, .priority = priority, .name = name };
    pq_sift_up(pq, i);
    return true;
}

bool pq_pop(MinPriorityQueue *pq, Task *out_task) {
    if (pq->size <= 0) return false;
    *out_task = pq->items[0];
    pq->size--;
    if (pq->size > 0) {
        pq->items[0] = pq->items[pq->size];
        pq_sift_down(pq, 0);
    }
    return true;
}

bool pq_decrease_key(MinPriorityQueue *pq, int task_id, int new_priority) {
    for (int i = 0; i < pq->size; i++) {
        if (pq->items[i].task_id == task_id) {
            if (new_priority > pq->items[i].priority) return false;
            pq->items[i].priority = new_priority;
            pq_sift_up(pq, i);
            return true;
        }
    }
    return false;
}

int main(void) {
    printf("=== 1. Floyd's O(n) Bottom-Up Max-Heap Construction ===\n");
    int raw[] = { 15, 10, 30, 50, 20, 40, 80 };
    int n = (int)(sizeof(raw) / sizeof(raw[0]));
    MaxHeap h;
    build_max_heap(&h, raw, n);
    max_heap_print(&h, "After build_max_heap");

    printf("\n=== 2. Max-Heap Insertion, Increase-Key, and Extract-Max ===\n");
    max_heap_insert(&h, 95);
    max_heap_print(&h, "After insert(95)    ");

    max_heap_increase_key(&h, 5, 90); /* Index 5 was 15 -> promoted to 90 */
    max_heap_print(&h, "After increase_key(idx=5, 90)");

    int removed_at;
    max_heap_delete_at(&h, 2, &removed_at);
    printf("Deleted element at index 2 (value=%d)\n", removed_at);
    max_heap_print(&h, "After delete_at(2)  ");

    int max_val;
    max_heap_extract_max(&h, &max_val);
    printf("Extracted Max Root = %d\n", max_val);
    max_heap_print(&h, "After extract_max() ");

    printf("\n=== 3. In-Place O(n log n) Heap Sort ===\n");
    int sort_arr[] = { 42, 15, 73, 8, 91, 27, 64, 19 };
    int sn = (int)(sizeof(sort_arr) / sizeof(sort_arr[0]));
    heap_sort(sort_arr, sn);
    printf("Sorted ascending: [");
    for (int i = 0; i < sn; i++) {
        printf("%d%s", sort_arr[i], (i + 1 < sn) ? ", " : "");
    }
    printf("]\n");

    printf("\n=== 4. Min-Heap Priority Queue (Task Scheduler) ===\n");
    MinPriorityQueue pq;
    pq_init(&pq);
    pq_push(&pq, 101, 40, "Background Backup");
    pq_push(&pq, 102, 15, "Database Query");
    pq_push(&pq, 103, 50, "Log Compaction");
    pq_push(&pq, 104,  5, "Kernel Page Fault");
    pq_push(&pq, 105, 25, "HTTP API Response");

    printf("Boosting Task #103 (Log Compaction) priority from 50 -> 2...\n");
    pq_decrease_key(&pq, 103, 2);

    printf("Dispatching tasks in priority order:\n");
    Task t;
    while (pq_pop(&pq, &t)) {
        printf("  -> Dispatched [priority=%2d] Task #%d: %s\n",
               t.priority, t.task_id, t.name);
    }
    return 0;
}
```

#### Program Output:
```text
=== 1. Floyd's O(n) Bottom-Up Max-Heap Construction ===
After build_max_heap (size=7): [80, 50, 40, 10, 20, 15, 30]

=== 2. Max-Heap Insertion, Increase-Key, and Extract-Max ===
After insert(95)     (size=8): [95, 80, 40, 50, 20, 15, 30, 10]
After increase_key(idx=5, 90) (size=8): [95, 80, 90, 50, 20, 40, 30, 10]
Deleted element at index 2 (value=90)
After delete_at(2)   (size=7): [95, 80, 40, 50, 20, 10, 30]
Extracted Max Root = 95
After extract_max()  (size=6): [80, 50, 40, 30, 20, 10]

=== 3. In-Place O(n log n) Heap Sort ===
Sorted ascending: [8, 15, 19, 27, 42, 64, 73, 91]

=== 4. Min-Heap Priority Queue (Task Scheduler) ===
Boosting Task #103 (Log Compaction) priority from 50 -> 2...
Dispatching tasks in priority order:
  -> Dispatched [priority= 2] Task #103: Log Compaction
  -> Dispatched [priority= 5] Task #104: Kernel Page Fault
  -> Dispatched [priority=15] Task #102: Database Query
  -> Dispatched [priority=25] Task #105: HTTP API Response
  -> Dispatched [priority=40] Task #101: Background Backup
```

---

## 5. Comparison Tables and Complexity Summary

Let $n$ denote the number of elements in the heap and $h = \lfloor \log_2 n \rfloor$ denote the height of the heap.

### 5.1 Binary Heap Operations Complexity Summary

| Operation | Best-Case Time | Average-Case Time | Worst-Case Time | Auxiliary Space |
| :--- | :--- | :--- | :--- | :--- |
| **`peek_max` / `peek_min`** (`data[0]`) | $\Theta(1)$ | $\Theta(1)$ | $O(1)$ | $O(1)$ |
| **`insert` (`sift_up`)** | $\Theta(1)$ | $\Theta(1)$ | $O(\log n)$ | $O(1)$ |
| **`extract_max` / `extract_min` (`sift_down`)** | $\Theta(1)$ *(all equal)* | $\Theta(\log n)$ | $O(\log n)$ | $O(1)$ |
| **`increase_key` / `decrease_key` (given index `i`)** | $\Theta(1)$ | $\Theta(\log n)$ | $O(\log n)$ | $O(1)$ |
| **`delete_at(i)` (given index `i`)** | $\Theta(1)$ | $\Theta(\log n)$ | $O(\log n)$ | $O(1)$ |
| **`build_heap` (Floyd's Bottom-Up)** | $\Theta(n)$ | $\Theta(n)$ | **$O(n)$** | $O(1)$ |
| **Search for Arbitrary Key $k$** | $\Theta(1)$ | $\Theta(n)$ | $O(n)$ | $O(1)$ |
| **In-Place `heap_sort`** | $\Theta(n \log n)$ | $\Theta(n \log n)$ | $O(n \log n)$ | $O(1)$ |

### 5.2 Priority Queue Implementations Compared: Array vs. Linked List vs. BST vs. Binary Heap

| Priority Queue Implementation | `insert(x)` | `peek_max()` / `peek_min()` | `extract_max()` / `extract_min()` | `build_pq(n items)` | `change_key(ptr/idx)` | Memory Overhead |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Unsorted Array** | $O(1)$ | $O(n)$ | $O(n)$ | $O(n)$ | $O(1)$ | Zero pointer overhead |
| **Unsorted Doubly Linked List** | $O(1)$ | $O(n)$ | $O(n)$ | $O(n)$ | $O(1)$ | 2 pointers/node |
| **Sorted Array (Ascending for Max-PQ)** | $O(n)$ *(shift)* | $O(1)$ *(`A[n-1]`)* | $O(1)$ *(`n--`)* | $O(n \log n)$ | $O(n)$ *(shift)* | Zero pointer overhead |
| **Sorted Singly Linked List** | $O(n)$ *(scan)* | $O(1)$ *(`head`)* | $O(1)$ *(`head`)* | $O(n^2)$ | $O(n)$ | 1 pointer/node |
| **Unbalanced BST** | $O(n)$ worst | $O(n)$ worst | $O(n)$ worst | $O(n^2)$ worst | $O(n)$ worst | 2 pointers/node |
| **Balanced BST (AVL / Red-Black)** | $O(\log n)$ | $O(\log n)$ *(or $O(1)$ cached)* | $O(\log n)$ | $O(n \log n)$ | $O(\log n)$ | 2–3 pointers + metadata |
| **Binary Heap (Array-Based)** | **$O(\log n)$** | **$O(1)$** | **$O(\log n)$** | **$O(n)$** | **$O(\log n)$** | **Zero pointer overhead** |

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Pitfall 1: Confusing 0-Based and 1-Based Index Formulas (`2*i` vs. `2*i + 1`)

In a 0-based C array, writing `left = 2 * i` sets `left = 0` when `i = 0`, causing the root to compare with itself in an infinite loop!

```c
/* BUG: Using 1-based child formulas on a 0-based C array (i = 0 -> left = 0!) */
int parent = i / 2;       /* Wrong for even indices in 0-based array! */
int left   = 2 * i;       /* Infinite self-loop when i == 0!          */
int right  = 2 * i + 1;

/* FIX: Always use 0-based formulas for 0-indexed C arrays */
int parent = (i - 1) / 2; /* Valid for i > 0 */
int left   = 2 * i + 1;
int right  = 2 * i + 2;
```

### 6.2 Pitfall 2: Swapping with an Unchecked Child Instead of the Larger of Both Children

During `sift_down` in a Max-Heap, if you immediately swap `data[i]` with `data[left]` whenever `data[left] > data[i]` without checking whether `data[right] > data[left]`, you can promote a smaller left child above a larger right child, destroying the Max-Heap property!

```c
/* BUG: Swapping with left child before comparing left vs. right child */
if (left < h->size && h->data[left] > h->data[i]) {
    swap_int(&h->data[i], &h->data[left]); /* WRONG if data[right] > data[left]! */
    i = left;
}

/* FIX: Find the largest among {i, left, right} FIRST, then swap once */
int largest = i;
if (left < h->size && h->data[left] > h->data[largest])   largest = left;
if (right < h->size && h->data[right] > h->data[largest]) largest = right;
if (largest != i) {
    swap_int(&h->data[i], &h->data[largest]);
    i = largest;
}
```

### 6.3 Pitfall 3: Running Only `sift_down` After Arbitrary-Index Deletion

When deleting the element at index `i` ($i > 0$) and replacing it with `last = h->data[h->size - 1]`, the replacement leaf came from another branch of the tree and may be **larger than `i`'s parent**. Calling only `sift_down` leaves a parent-child violation above `i`.

```c
/* BUG: Assuming replacement leaf only needs to move downward */
h->data[i] = h->data[--h->size];
max_heap_sift_down(h, i); /* Fails if new h->data[i] > h->data[(i - 1) / 2]! */

/* FIX: Check whether replacement needs to sift UP or sift DOWN */
int old_key = h->data[i];
h->data[i] = h->data[--h->size];
if (i < h->size) {
    if (h->data[i] > old_key) {
        max_heap_sift_up(h, i);
    } else {
        max_heap_sift_down(h, i);
    }
}
```

### 6.4 Pitfall 4: Building a Heap Top-Down in $O(n \log n)$ Instead of Bottom-Up in $O(n)$

Calling `sift_up` from `i = 1` to `n - 1` forces the $\lceil n/2 \rceil$ leaves at the bottom level to pay the maximum $O(\log n)$ cost, yielding $\Theta(n \log n)$ worst-case time instead of Floyd's $O(n)$ bottom-up construction.

```c
/* SUBOPTIMAL O(n log n): Top-down sift_up on every node */
for (int i = 1; i < h->size; i++) {
    max_heap_sift_up(h, i);
}

/* OPTIMAL O(n): Floyd's bottom-up sift_down from last internal node to 0 */
for (int i = (h->size / 2) - 1; i >= 0; i--) {
    max_heap_sift_down(h, i);
}
```

### 6.5 Boundary Conditions Checklist

- [ ] **Empty Heap (`size == 0`):** Does `extract_max` / `pq_pop` return `false` without decrementing `size` into `-1`?
- [ ] **Single-Element Extraction (`size == 1`):** After `size--`, does `extract_max` skip overwriting `data[0]` and calling `sift_down`?
- [ ] **Deleting the Last Leaf (`delete_at(h, h->size - 1)`):** Does `delete_at` guard with `if (i < h->size)` after `h->size--` so it does not sift an out-of-bounds slot?
- [ ] **Node with Only One Child (`2*i + 1 == size - 1`):** Does `sift_down` check `right < h->size` separately from `left < h->size`?
- [ ] **Integer Division on Index `0`:** In C, `(0 - 1) / 2` evaluates to `0` (`-1 / 2 == 0` via truncation toward zero). Always guard `sift_up` with `while (i > 0 && ...)` so index `0` never computes a fake parent!

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways and Core Invariants

1. **Two Defining Invariants:** A Binary Heap is simultaneously (a) a **Complete Binary Tree** of height $h = \lfloor \log_2 n \rfloor$ (Shape Property) and (b) vertically ordered so every parent dominates its children (Heap-Order Property).
2. **Implicit Array Representation (0-Based):** `parent(i) = (i - 1) / 2`, `left(i) = 2*i + 1`, `right(i) = 2*i + 2`. Internal nodes occupy `0 .. (n/2) - 1`; leaves occupy `(n/2) .. n - 1`.
3. **Core Complexities:** `peek` is $O(1)$; `insert`, `extract_max`/`extract_min`, `increase_key`/`decrease_key`, and `delete_at` are all $O(\log n)$; Floyd's bottom-up `build_heap` is **$O(n)$**; in-place `heap_sort` is **$O(n \log n)$**.
4. **Why a Heap Is Not a BST:** A heap imposes no ordering between left and right siblings, trading $O(\log n)$ arbitrary search for $O(1)$ extreme access, $O(n)$ bulk construction, and zero pointer overhead.

### 7.2 Self-Test Revision Questions

1. **Index Arithmetic Question:** In a 0-based binary heap of $n = 15$ elements, what are the indices of (a) the parent of node `6`, (b) the left and right children of node `6`, (c) the last internal node, and (d) all leaf nodes?
   *(Answer: (a) `(6 - 1)/2 = 2`; (b) `left = 13`, `right = 14`; (c) `(15/2) - 1 = 6`; (d) leaves are indices `7 .. 14`).*
2. **Location Question:** In a Max-Heap of $n = 10$ distinct elements, where can the **second-largest** element be located? Where can the **smallest** element be located?
   *(Answer: The second-largest element must be a direct child of the root—index `1` or index `2`. The smallest element must be one of the $\lceil 10/2 \rceil = 5$ leaves at indices `5 .. 9`).*
3. **Trace Question:** Trace Floyd's `build_max_heap` on the array `[4, 1, 3, 2, 16, 9, 10, 14, 8, 7]`. Which index does the loop start at, and what is the final array?
   *(Answer: Starts at `i = (10/2) - 1 = 4` and iterates down to `0`, producing `[16, 14, 10, 8, 7, 9, 3, 2, 4, 1]`).*
4. **Proof Question:** Explain in two sentences why calling `sift_down` from `(n/2) - 1` down to `0` takes $O(n)$ time rather than $O(n \log n)$.
   *(Answer: At height $h'$, there are at most $\lceil n / 2^{h'+1} \rceil$ nodes and each takes $O(h')$ work. Summing $\sum_{h'=0}^{\lfloor \log_2 n \rfloor} \frac{n}{2^{h'+1}} h'$ converges to $O(n)$ because $\sum_{h'=0}^{\infty} \frac{h'}{2^{h'}} = 2$).*

### 7.3 Navigation Links

- **Previous Topic:** [03-Threaded-Binary-Search-Trees.md](./03-Threaded-Binary-Search-Trees.md) — Single and Double Threaded Binary Search Trees
- **Current File:** `04-Heaps-and-Heap-Priority-Queues.md` (Unit 3 — Topic 4)
- **Next Topic:** [05-AVL-Trees.md](./05-AVL-Trees.md) — Height-Balanced AVL Binary Search Trees and Rotations
