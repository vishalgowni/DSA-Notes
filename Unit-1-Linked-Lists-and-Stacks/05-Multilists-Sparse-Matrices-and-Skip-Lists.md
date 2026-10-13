# Multilists, Sparse Matrices, and Skip Lists
---

## Table of Contents
1. [Introduction](#1-introduction)
   - 1.1 [Overview and Motivation](#11-overview-and-motivation)
   - 1.2 [Core Syllabus Requirements vs. Optional Extensions](#12-core-syllabus-requirements-vs-optional-extensions)
   - 1.3 [Learning Objectives](#13-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Multilists (Multiply Linked Lists)](#21-multilists-multiply-linked-lists)
   - 2.2 [Sparse Matrices: Triplet and Cross-Linked Representations](#22-sparse-matrices-triplet-and-cross-linked-representations)
   - 2.3 [Skip Lists: Probabilistic Multi-Level Express Lanes](#23-skip-lists-probabilistic-multi-level-express-lanes)
   - 2.4 [Terminology Table](#24-terminology-table)
   - 2.5 [Mathematical Properties, Invariants, and Formulas](#25-mathematical-properties-invariants-and-formulas)
   - 2.6 [Conceptual ASCII Diagrams](#26-conceptual-ascii-diagrams)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [C Structure Declarations (`typedef struct`)](#31-c-structure-declarations-typedef-struct)
   - 3.2 [Pointer Relationships and Heap Memory Layouts](#32-pointer-relationships-and-heap-memory-layouts)
   - 3.3 [Static vs. Dynamic Storage, Allocation, and Deallocation](#33-static-vs-dynamic-storage-allocation-and-deallocation)
   - 3.4 [Pros and Cons of Representations](#34-pros-and-cons-of-representations)
4. [Operations and Algorithms](#4-operations-and-algorithms)
   - 4.1 [Sparse Matrix Triplet (3-Tuple): Creation, Display, and Fast Transpose](#41-sparse-matrix-triplet-3-tuple-creation-display-and-fast-transpose)
   - 4.2 [Orthogonal Cross-Linked Sparse Matrix: Sorted Insertion and Display](#42-orthogonal-cross-linked-sparse-matrix-sorted-insertion-and-display)
   - 4.3 [Sparse Matrix Addition](#43-sparse-matrix-addition)
   - 4.4 [Complete, Compilable C11 Program: Sparse Matrices (Triplet & Multilist)](#44-complete-compilable-c11-program-sparse-matrices-triplet--multilist)
   - 4.5 [Skip List Search](#45-skip-list-search)
   - 4.6 [Skip List Insertion (with Random Level Generation and `update[]`)](#46-skip-list-insertion-with-random-level-generation-and-update)
   - 4.7 [Skip List Deletion](#47-skip-list-deletion)
   - 4.8 [Complete, Compilable C11 Program: Skip List](#48-complete-compilable-c11-program-skip-list)
5. [Complexity Analysis and Comparison Tables](#5-complexity-analysis-and-comparison-tables)
   - 5.1 [Sparse Matrix Operation Complexity Table](#51-sparse-matrix-operation-complexity-table)
   - 5.2 [Comparison: 2D Array vs. Triplet Array vs. Cross-Linked Multilist](#52-comparison-2d-array-vs-triplet-array-vs-cross-linked-multilist)
   - 5.3 [Skip List Operation Complexity Table](#53-skip-list-operation-complexity-table)
   - 5.4 [Comparison: Sorted Linked List vs. Skip List vs. Balanced BST](#54-comparison-sorted-linked-list-vs-skip-list-vs-balanced-bst)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Pitfall 1: Retaining Zero-Sum Entries After Sparse Matrix Addition](#61-pitfall-1-retaining-zero-sum-entries-after-sparse-matrix-addition)
   - 6.2 [Pitfall 2: Updating Only Row Links and Forgetting Column Links in a Multilist](#62-pitfall-2-updating-only-row-links-and-forgetting-column-links-in-a-multilist)
   - 6.3 [Pitfall 3: Out-of-Bounds Level Indexing in Skip List `forward[]` and `update[]`](#63-pitfall-3-out-of-bounds-level-indexing-in-skip-list-forward-and-update)
   - 6.4 [Pitfall 4: Failing to Reduce Skip List `level` After Deleting the Tallest Node](#64-pitfall-4-failing-to-reduce-skip-list-level-after-deleting-the-tallest-node)
   - 6.5 [Boundary Conditions Checklist](#65-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)
   - 7.1 [Key Takeaways](#71-key-takeaways)
   - 7.2 [Essential Invariants](#72-essential-invariants)
   - 7.3 [Quick Self-Test Checklist](#73-quick-self-test-checklist)
   - 7.4 [Related Notes and Navigation](#74-related-notes-and-navigation)

---

## 1. Introduction

### 1.1 Overview and Motivation
Standard singly, doubly, and circular linked lists organize elements along a **single one-dimensional linear chain**. However, real-world engineering problems frequently run into two fundamental bottlenecks with simple 1D linked lists and 2D arrays:

1. **Multi-Dimensional & Many-to-Many Relationships (Multilists & Sparse Matrices)**:
   - How do we represent a university registration database where each enrollment record belongs simultaneously to a *student's schedule* AND a *course's roster* without duplicating nodes?
   - How do we store a $100{,}000 \times 100{,}000$ scientific or web-graph matrix ($10^{10}$ entries $\approx 40\text{ GB}$ of `int`s) when $99.99\%$ of its entries are $0$?
   - **Solution**: **Multilists (Multiply Linked Lists)** and **Sparse Matrix representations** (the array-based **3-Tuple/Triplet** format and the dynamic **Orthogonal Cross-Linked List** format) store *only* the non-zero elements ($\text{nnz}$), reducing memory from $\Theta(r \cdot c)$ to $\Theta(\text{nnz})$.

2. **The $\Theta(n)$ Search Bottleneck of Sorted Linked Lists (Skip Lists)**:
   - Binary search achieves $O(\log n)$ time on a sorted array via $O(1)$ random index access, but insertion and deletion in an array cost $O(n)$ element shifts.
   - Conversely, a sorted singly linked list supports $O(1)$ localized pointer insertion once the position is known, but finding that position requires $O(n)$ sequential traversal because pointers cannot jump to the middle.
   - **Solution**: Invented by William Pugh in 1989/1990, a **Skip List** augments a sorted linked list with probabilistic **multi-level forward pointers ("express lanes")**, achieving **$O(\log n)$ expected time for search, insertion, and deletion** without the complex rotations of AVL or Red-Black trees.

### 1.2 Core Syllabus Requirements vs. Optional Extensions

| Topic Area | Core Syllabus Requirements (Must Master) | Optional Extensions (Enrichment) |
| :--- | :--- | :--- |
| **Multilists** | Concept of shared nodes with multiple independent `next` pointers; student-course and orthogonal grid models. | General $k$-dimensional relational multilist indexing. |
| **Sparse Matrices** | Sparsity definition; **Triplet (3-Tuple) Array** representation; **Orthogonal Cross-Linked List** representation; creation, display, insertion, addition. | Fast Transpose ($O(c + \text{nnz})$); Compressed Sparse Row (CSR) format used in high-performance BLAS libraries. |
| **Skip Lists** | Motivation; multi-level node structure; promotion probability $p=1/2$; $L_{\max} = \lceil \log_2 n \rceil$; Search, Insertion (`update[]`), Deletion; Expected $O(\log n)$ vs. Worst-Case $O(n)$ analysis. | Deterministic skip lists; indexable (rank/order-statistic) skip lists; lock-free concurrent skip lists. |

### 1.3 Learning Objectives
After studying these notes, you will be able to:
- Explain how a **Multilist** threads a single node across two or more independent logical lists.
- Calculate when a matrix is **sparse** and derive the exact memory break-even threshold between a dense 2D array, a 3-Tuple Triplet array, and an Orthogonal Cross-Linked Multilist.
- Implement Sparse Matrix creation, display, transpose, insertion, and addition in C11.
- Explain how **Skip Lists** use coin-flip level promotion ($p = 1/2$) to achieve $O(\log n)$ expected search, insertion, and deletion time with $O(n)$ expected space.
- Trace and implement Skip List `search`, `insert`, and `delete` using the `update[]` predecessor array in C11.

---

## 2. Definitions and Terminology

### 2.1 Multilists (Multiply Linked Lists)
A **Multilist** (or **Multiply Linked List**) is a linked data structure in which each node contains **two or more independent link pointers**, allowing a single node to participate simultaneously in multiple distinct logical lists.

> **Important Distinction**: A **Doubly Linked List (DLL)** is *not* considered a true multilist in the structural sense, because `next` and `prev` traverse the *exact same sequence* in opposite directions. In a true **Multilist**, each pointer field (`nextRow`, `nextCol`, or `nextCourse`, `nextStudent`) traverses a **different logical subset or dimension** of the data.

### 2.2 Sparse Matrices: Triplet and Cross-Linked Representations
Let $A = [a_{i,j}]$ be an $r \times c$ matrix with $0 \le i < r$ and $0 \le j < c$.
- **Sparse Matrix**: A matrix in which the number of non-zero elements ($\text{nnz}$) is much smaller than the total number of elements ($\text{nnz} \ll r \cdot c$).
- **Triplet (3-Tuple / Coordinate COO) Representation**: An array of $(\text{nnz} + 1)$ rows and $3$ columns `(row, col, value)` sorted in **row-major order** (ascending by `row`, and ascending by `col` within the same row).
  - **Row `0` (Header Triplet)**: Stores `(r, c, nnz)` — total rows, total columns, and total non-zero elements.
  - **Rows `1 .. nnz`**: Store `(i, j, A[i][j])` for each non-zero entry $A[i][j] \neq 0$.
- **Orthogonal Cross-Linked List (Multilist) Representation**: A dynamic 2D linked structure where every non-zero element $A[i][j]$ is represented by a single node that belongs simultaneously to:
  1. **Row $i$'s sorted linked list** via a horizontal pointer `right`.
  2. **Column $j$'s sorted linked list** via a vertical pointer `down`.

### 2.3 Skip Lists: Probabilistic Multi-Level Express Lanes
A **Skip List** for a sorted set of $n$ keys is a hierarchy of sorted singly linked lists layered across levels $0, 1, \dots, L_{\max}$:
- **Level $0$ (Base Layer)**: Contains **all $n$ elements** in strictly ascending key order.
- **Level $k$ ($k \ge 1$, Express Lanes)**: Contains a random subset of the nodes from Level $k - 1$. Specifically, each node present at Level $k - 1$ is promoted to Level $k$ independently with probability $p$ (typically $p = 1/2$, equivalent to flipping a fair coin until tails appears).
- **Tower Height**: A node promoted up to level $k$ has a "tower" of $k + 1$ forward pointers (`forward[0 .. k]`), where `forward[i]` points to the next node whose tower reaches at least level $i$.

### 2.4 Terminology Table

| Term | Definition |
| :--- | :--- |
| **Multilist** | A linked structure where each node contains multiple pointers belonging to independent logical lists. |
| **Non-Zero Count ($\text{nnz}$)** | The number of elements $a_{i,j} \neq 0$ in a matrix $A \in \mathbb{R}^{r \times c}$. |
| **Matrix Density ($\delta$)** | The fraction of non-zero elements: $\delta = \frac{\text{nnz}}{r \cdot c}$. |
| **Sparsity Ratio** | The fraction of zero elements: $\text{Sparsity} = 1 - \delta = \frac{r \cdot c - \text{nnz}}{r \cdot c}$. |
| **3-Tuple / Triplet** | A 3-element record `(row, col, value)` representing one non-zero entry of a sparse matrix. |
| **Orthogonal List** | A 2D multilist where nodes are linked horizontally by row (`right`) and vertically by column (`down`). |
| **Skip List** | A probabilistic multi-level linked list supporting $O(\log n)$ expected search, insert, and delete. |
| **Sentinel Header (`header`)** | A dummy node at the left end of a Skip List with maximum height $L_{\max}$ and key $-\infty$ (`INT_MIN`). |
| **`forward[i]`** | The pointer at level $i$ of a Skip List node pointing to the next node at level $i$. |
| **`update[i]`** | Temporary array used during Skip List insert/delete to record the rightmost predecessor at level $i$ whose key is $< \text{targetKey}$. |

### 2.5 Mathematical Properties, Invariants, and Formulas

#### 1. Sparse Matrix Storage Break-Even Analysis
Suppose an $r \times c$ matrix stores 4-byte integers (`sizeof(int) = 4`) on a 64-bit machine (`sizeof(void*) = 8`):
- **Dense 2D Array Space**:
  $$S_{\text{dense}} = 4 \cdot r \cdot c \text{ bytes}$$
- **Triplet Array Space** ($(\text{nnz} + 1)$ rows $\times 3$ ints):
  $$S_{\text{triplet}} = 12 \cdot (\text{nnz} + 1) \text{ bytes}$$
  Thus, the Triplet representation saves memory whenever:
  $$12(\text{nnz} + 1) < 4 r c \iff \text{nnz} < \frac{r \cdot c}{3} - 1 \quad (\text{i.e., Density } \delta < 33.3\%)$$
- **Orthogonal Cross-Linked List Space**:
  Each node stores `row` (4B), `col` (4B), `val` (4B), plus 4B padding and two 8B pointers (`right`, `down`) $= 32\text{ bytes/node}$, plus header arrays $8(r + c)\text{ bytes}$:
  $$S_{\text{cross}} = 32 \cdot \text{nnz} + 8(r + c) \text{ bytes}$$
  For large matrices, the Cross-Linked list saves memory over a dense 2D array when $\delta = \frac{\text{nnz}}{r \cdot c} < \frac{1}{8} = 12.5\%$.

#### 2. Skip List Probabilistic Properties
Let $p \in (0, 1)$ be the level promotion probability (standard $p = \frac{1}{2}$):
1. **Maximum Level Cap**:
   $$L_{\max} = \lceil \log_{1/p} n \rceil = \lceil \log_2 n \rceil \quad (\text{for } p = 1/2)$$
2. **Probability of Reaching Level $k$**:
   $$P(\text{node height} \ge k) = p^k = \left(\frac{1}{2}\right)^k$$
3. **Expected Number of Nodes at Level $k$**:
   $$E[N_k] = n \cdot p^k = \frac{n}{2^k}$$
4. **Expected Number of Forward Pointers per Node**:
   $$E[\text{pointers per node}] = \sum_{k=0}^{L_{\max}} p^k \le \sum_{k=0}^{\infty} p^k = \frac{1}{1 - p}$$
   For $p = \frac{1}{2}$, each node stores on average **only $\frac{1}{1 - 1/2} = 2$ pointers**! Thus total expected space is $\Theta(n)$.
5. **Expected Search Cost**:
   Tracing the search path backward from the target node at level $0$ up to the header at level $L_{\max}$, at each step we move UP a level with probability $p$ or LEFT along the current level with probability $1 - p$. The expected number of steps to climb $\log_{1/p} n$ levels is:
   $$E[\text{comparisons}] = O\left(\frac{\log_{1/p} n}{p}\right) = O(\log n)$$

### 2.6 Conceptual ASCII Diagrams

#### 1. Student-Course Registration Multilist
Suppose Student `S1` takes Courses `C1, C2`, and Student `S2` takes Courses `C1, C3`. Each enrollment node `(Si, Cj)` is shared between a horizontal Student list and a vertical Course list:
```text
             Course C1          Course C2          Course C3
             +-------+          +-------+          +-------+
             |  C1   |          |  C2   |          |  C3   |
             +---+---+          +---+---+          +---+---+
                 |                  |                  |
                 ▼                  ▼                  |
+------+     +-------+          +-------+              |
|  S1  |────►| S1,C1 |─────────►| S1,C2 |────► NULL    |
+------+     +---+---+          +---+---+              |
                 |                  |                  |
                 ▼                  ▼                  ▼
+------+     +-------+            NULL             +-------+
|  S2  |────►| S2,C1 |────────────────────────────►| S2,C3 |────► NULL
+------+     +---+---+                             +---+---+
                 |                                     |
                 ▼                                     ▼
               NULL                                  NULL
```

#### 2. Sparse Matrix: 2D Grid vs. Triplet Table vs. Cross-Linked Multilist
Consider the $4 \times 4$ sparse matrix $A$ with $\text{nnz} = 4$:
```text
Dense 4x4 Matrix A:                3-Tuple / Triplet Array T[0..4]:
Col:   0   1   2   3               Index | row | col | value
Row 0 [0, 15,  0,  0]              ------+-----+-----+-------
Row 1 [0,  0,  0, 22]              [0]   |  4  |  4  |   4    <-- Header (r, c, nnz)
Row 2 [9,  0,  0,  0]              [1]   |  0  |  1  |  15
Row 3 [0,  0,  0, -5]              [2]   |  1  |  3  |  22
                                   [3]   |  2  |  0  |   9
                                   [4]   |  3  |  3  |  -5
```

**Orthogonal Cross-Linked List (Multilist) for Matrix $A$**:
```text
colHead:     [Col 0]          [Col 1]          [Col 2]          [Col 3]
                |                |                |                |
                |                ▼                ▼                |
rowHead[0] ─────|───────────► (0,1: 15) ──► NULL NULL              |
                |                |                                 ▼
rowHead[1] ─────|────────────────|────────────────────────────► (1,3: 22) ──► NULL
                ▼                ▼                                 |
rowHead[2] ──► (2,0: 9) ───► NULL NULL                             |
                |                                                  ▼
rowHead[3] ─────|─────────────────────────────────────────────► (3,3: -5) ──► NULL
                ▼                                                  |
              NULL                                               NULL
```

#### 3. Skip List with Multi-Level Express Lanes
Searching for key `50` starts at `header` (top level $L=3$) and drops down whenever the next node's key exceeds `50`:
```text
Level 3: [HDR] ═══════════════════════════════► [30] ══════════════════════════════════════► NULL
           ║                                      ║
Level 2: [HDR] ══════════════► [20] ══════════► [30] ═════════════════════════► [60] ══════► NULL
                                                  ║
Level 1: [HDR] ═════► [10] ══► [20] ══════════► [30] ═══════════════► [50] ═══► [60] ══════► NULL
                                                                        ║
Level 0: [HDR] ═════► [10] ══► [20] ══► [25] ═► [30] ══► [40] ══════► [50] ═══► [60] ═► [70]► NULL

Search trace for 50 (marked by ═ and ║):
  1. At Level 3: HDR->forward[3] is 30 (30 < 50) -> advance to [30]. Next is NULL -> drop to Level 2.
  2. At Level 2: [30]->forward[2] is 60 (60 > 50) -> cannot advance -> drop to Level 1.
  3. At Level 1: [30]->forward[1] is 50 (50 < 50 is false) -> drop to Level 0, check [30]->forward[0] = 40.
  4. At Level 0: [30] -> [40] (40 < 50) -> advance to [40]. Next is [50] -> FOUND 50!
```

---

## 3. Representation and Memory Layout in C

### 3.1 C Structure Declarations (`typedef struct`)

```c
/* =========================================================================
 * 1. GENERAL MULTILIST NODE (Student-Course Enrollment Example)
 * ========================================================================= */
typedef struct EnrollmentNode {
    int studentId;
    int courseId;
    char grade;
    struct EnrollmentNode *nextCourseForStudent; /* Horizontal link (Student's list) */
    struct EnrollmentNode *nextStudentInCourse;  /* Vertical link (Course's list) */
} EnrollmentNode;

/* =========================================================================
 * 2. SPARSE MATRIX: ARRAY-BASED TRIPLET (3-TUPLE) REPRESENTATION
 * ========================================================================= */
#define MAX_TERMS 100

typedef struct Triplet {
    int row;    /* Row index (or total rows in index 0) */
    int col;    /* Column index (or total cols in index 0) */
    int val;    /* Non-zero value (or total nnz in index 0) */
} Triplet;

/* =========================================================================
 * 3. SPARSE MATRIX: ORTHOGONAL CROSS-LINKED LIST (MULTILIST)
 * ========================================================================= */
typedef struct SMNode {
    int row;
    int col;
    int val;
    struct SMNode *right;   /* Next non-zero element in the SAME row */
    struct SMNode *down;    /* Next non-zero element in the SAME column */
} SMNode;

typedef struct CrossLinkedMatrix {
    int rows;
    int cols;
    int nnz;
    SMNode **rowHead;       /* Array of r row-head pointers */
    SMNode **colHead;       /* Array of c column-head pointers */
} CrossLinkedMatrix;

/* =========================================================================
 * 4. SKIP LIST REPRESENTATION
 * ========================================================================= */
#define MAX_LEVEL 6         /* Supports up to 2^6 = 64+ elements efficiently */

typedef struct SkipNode {
    int key;
    int value;
    int nodeLevel;                          /* Highest level index (0 .. MAX_LEVEL) */
    struct SkipNode *forward[MAX_LEVEL + 1];/* Express-lane forward pointers */
} SkipNode;

typedef struct SkipList {
    int level;              /* Current highest active level in the list */
    int size;               /* Total number of keys stored */
    SkipNode *header;       /* Sentinel header node */
} SkipList;
```

### 3.2 Pointer Relationships and Heap Memory Layouts

#### Orthogonal Cross-Linked Sparse Matrix Node Layout
Each `SMNode` resides on the heap and contains two orthogonal pointers:
```text
Heap Address: 0x6100 (SMNode for A[1][3] = 22)
+---------+---------+---------+----------------+----------------+
| row: 1  | col: 3  | val: 22 | right: 0x0     | down:  0x6180  |
+---------+---------+---------+----------------+----------------+
                                  │                │
                                  ▼                ▼
                             Next node in     Next node in
                               Row 1            Column 3
```

#### Skip List Node Memory Layout
In our C definition, `forward` is a fixed-capacity array `forward[MAX_LEVEL + 1]` inside `SkipNode` (alternatively allocated as a flexible array member `malloc(sizeof(SkipNode*) * (level + 1))`).
```text
SkipNode (key = 30, nodeLevel = 3):
+----------+----------+--------------+
| key: 30  | val: 300 | nodeLevel: 3 |
+----------+----------+--------------+
| forward[5]: NULL                   |  (Unused levels above nodeLevel)
| forward[4]: NULL                   |
| forward[3]: 0x0 (NULL)             | ──► Level 3 express link
| forward[2]: 0x7090 (Node 60)       | ──► Level 2 express link
| forward[1]: 0x7050 (Node 50)       | ──► Level 1 express link
| forward[0]: 0x7020 (Node 40)       | ──► Level 0 base link
+------------------------------------+
```

### 3.3 Static vs. Dynamic Storage, Allocation, and Deallocation
- **Triplet Array (`Triplet T[MAX_TERMS + 1]`)**: Contiguous memory block (static or single `malloc`). Extremely cache-friendly and requires zero pointer overhead, making it ideal when the matrix is static or only undergoes batch operations (like Transpose or Matrix Addition).
- **Cross-Linked Multilist (`CrossLinkedMatrix`)**: Requires dynamically allocating `rowHead` ($r$ pointers), `colHead` ($c$ pointers), and $\text{nnz}$ `SMNode` structs. When freeing a `CrossLinkedMatrix`, **traverse only along the rows (`rowHead[i]` via `right` pointers) to `free()` each `SMNode`**, and then `free(mat->rowHead)` and `free(mat->colHead)`. Never free along both `right` and `down` or every node will be double-freed!
- **Skip List (`SkipList`)**: Even though each node is linked across multiple levels `forward[0 .. nodeLevel]`, **Level 0 (`forward[0]`) visits every single node in the Skip List**. Therefore, deallocating a Skip List is a simple linear walk along `curr = curr->forward[0]`!

### 3.4 Pros and Cons of Representations

| Representation | Pros | Cons |
| :--- | :--- | :--- |
| **Triplet (3-Tuple) Array** | Compact ($3\text{ ints/entry}$); contiguous cache locality; fast sequential scan and $O(c + \text{nnz})$ transpose. | Inserting or deleting a single non-zero entry in the middle requires $O(\text{nnz})$ array shifts; no fast column traversal without transposing. |
| **Cross-Linked Multilist** | Dynamic $O(r + c)$ worst-case single-element insertion/deletion without shifting; simultaneous row and column traversal. | Higher memory per non-zero entry (2 pointers + coordinates); non-contiguous heap allocations. |
| **Skip List** | $O(\log n)$ expected search, insert, and delete; simple iterative implementation compared to balanced trees; easy range queries via Level 0. | Probabilistic bounds ($O(n)$ worst case with tiny probability); uses slightly more pointers on average ($2$ pointers/node for $p=1/2$) than a 1D SLL. |

---

## 4. Operations and Algorithms

### 4.1 Sparse Matrix Triplet (3-Tuple): Creation, Display, and Fast Transpose

#### 1. Purpose
Convert a 2D matrix into a row-major 3-Tuple array `T[0 .. nnz]` and compute its transpose $A^T$ in optimal $O(c + \text{nnz})$ time using **Fast Transpose** (counting sort on column indices).

#### 2. Preconditions and Assumptions
- Input triplets in `A[1 .. nnz]` are ordered in row-major order.
- `A[0].row = r`, `A[0].col = c`, `A[0].val = nnz`.

#### 3. Step-by-Step Algorithm (Fast Transpose)
A naive transpose scans all $\text{nnz}$ terms for each column $0 \dots c-1$, costing $O(c \cdot \text{nnz})$. **Fast Transpose** uses two auxiliary arrays of size $c$:
1. Set header `B[0].row = A[0].col`, `B[0].col = A[0].row`, `B[0].val = A[0].val`.
2. Initialize `rowTerms[0 .. c-1] = 0`.
3. Count how many non-zero entries exist in each column of $A$:
   - For $i = 1 \dots \text{nnz}$: `rowTerms[A[i].col]++`.
4. Compute the starting output index `startPos[j]` for each row $j$ of $B$:
   - `startPos[0] = 1`
   - For $j = 1 \dots c-1$: `startPos[j] = startPos[j - 1] + rowTerms[j - 1]`.
5. Place each triplet `A[i]` directly into its exact destination in `B`:
   - For $i = 1 \dots \text{nnz}$:
     - `pos = startPos[A[i].col]++`
     - `B[pos] = (A[i].col, A[i].row, A[i].val)`.

#### 4. Pseudocode
```text
ALGORITHM FastTranspose(A, B)
1. r <- A[0].row; c <- A[0].col; nnz <- A[0].val
2. B[0] <- (c, r, nnz)
3. IF nnz == 0 THEN RETURN
4. FOR j <- 0 TO c - 1 DO rowTerms[j] <- 0
5. FOR i <- 1 TO nnz DO rowTerms[A[i].col] <- rowTerms[A[i].col] + 1
6. startPos[0] <- 1
7. FOR j <- 1 TO c - 1 DO startPos[j] <- startPos[j - 1] + rowTerms[j - 1]
8. FOR i <- 1 TO nnz DO
9.     pos <- startPos[A[i].col]
10.    startPos[A[i].col] <- pos + 1
11.    B[pos] <- (A[i].col, A[i].row, A[i].val)
12. END FOR
```

#### 5. Worked Trace
For our $4 \times 4$ matrix with triplets `(0,1,15), (1,3,22), (2,0,9), (3,3,-5)`:
```text
Col j:           0   1   2   3
rowTerms[j]:   [ 1,  1,  0,  2 ]   (Col 0 has 1, Col 1 has 1, Col 2 has 0, Col 3 has 2)
startPos[j]:   [ 1,  2,  3,  3 ]   (Col 0 starts at B[1], Col 1 at B[2], Col 3 at B[3])

Placement pass:
- A[1]=(0,1,15): col=1 -> B[startPos[1]=2] = (1,0,15), startPos[1] becomes 3
- A[2]=(1,3,22): col=3 -> B[startPos[3]=3] = (3,1,22), startPos[3] becomes 4
- A[3]=(2,0,9) : col=0 -> B[startPos[0]=1] = (0,2,9),  startPos[0] becomes 2
- A[4]=(3,3,-5): col=3 -> B[startPos[3]=4] = (3,3,-5), startPos[3] becomes 5
```

#### 6. Reusable C Function
```c
void fastTransposeTriplet(const Triplet A[], Triplet B[]) {
    int r = A[0].row;
    int c = A[0].col;
    int nnz = A[0].val;

    B[0].row = c;
    B[0].col = r;
    B[0].val = nnz;

    if (nnz <= 0) {
        return;
    }

    int *rowTerms = (int *)calloc((size_t)c, sizeof(int));
    int *startPos = (int *)malloc((size_t)c * sizeof(int));
    if (rowTerms == NULL || startPos == NULL) {
        free(rowTerms);
        free(startPos);
        return;
    }

    for (int i = 1; i <= nnz; i++) {
        rowTerms[A[i].col]++;
    }

    startPos[0] = 1;
    for (int j = 1; j < c; j++) {
        startPos[j] = startPos[j - 1] + rowTerms[j - 1];
    }

    for (int i = 1; i <= nnz; i++) {
        int col = A[i].col;
        int pos = startPos[col]++;
        B[pos].row = A[i].col;
        B[pos].col = A[i].row;
        B[pos].val = A[i].val;
    }

    free(rowTerms);
    free(startPos);
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $\Theta(c + \text{nnz})$, which is dramatically faster than $O(r \cdot c)$ when $\text{nnz} \ll r \cdot c$.
- **Auxiliary Space Complexity**: $\Theta(c)$ for `rowTerms` and `startPos`.

#### 8. Edge Cases and Common Mistakes
- **Zero Matrix ($\text{nnz} == 0$)**: Must return immediately after setting `B[0] = (c, r, 0)`.

---

### 4.2 Orthogonal Cross-Linked Sparse Matrix: Sorted Insertion and Display

#### 1. Purpose
Insert or update an entry `(r, c, val)` in an Orthogonal Cross-Linked Sparse Matrix while keeping both Row `r`'s horizontal list sorted by `col` and Column `c`'s vertical list sorted by `row`.

#### 2. Preconditions and Assumptions
- $0 \le r < \text{mat->rows}$ and $0 \le c < \text{mat->cols}$.
- `val != 0` (zero entries are not stored in a sparse matrix).

#### 3. Step-by-Step Algorithm
1. Validate coordinates `(r, c)` and verify `val != 0`.
2. Allocate a new `SMNode` with `(r, c, val, right=NULL, down=NULL)`.
3. **Link Horizontally into `rowHead[r]`** (sorted by ascending `col`):
   - If `rowHead[r] == NULL` or `rowHead[r]->col > c`:
     - Insert at the front of Row `r`: `newNode->right = rowHead[r]; rowHead[r] = newNode;`.
   - Else traverse with `curr = rowHead[r]` while `curr->right != NULL && curr->right->col < c`.
   - If an existing node already has `col == c`, update its value and free `newNode`.
   - Otherwise, splice `newNode` after `curr`: `newNode->right = curr->right; curr->right = newNode;`.
4. **Link Vertically into `colHead[c]`** (sorted by ascending `row`):
   - If `colHead[c] == NULL` or `colHead[c]->row > r`:
     - Insert at the top of Column `c`: `newNode->down = colHead[c]; colHead[c] = newNode;`.
   - Else traverse down Column `c` while `curr->down != NULL && curr->down->row < r`, then splice `newNode->down = curr->down; curr->down = newNode;`.
5. Increment `mat->nnz`.

#### 4. Pseudocode
```text
ALGORITHM InsertCrossMatrix(mat, r, c, val)
1. IF val == 0 THEN RETURN
2. newNode <- AllocateSMNode(r, c, val)
3. // Horizontal splice into rowHead[r]
4. IF mat.rowHead[r] == NULL OR mat.rowHead[r].col > c THEN
5.     newNode.right <- mat.rowHead[r]
6.     mat.rowHead[r] <- newNode
7. ELSE
8.     curr <- mat.rowHead[r]
9.     WHILE curr.right != NULL AND curr.right.col < c DO curr <- curr.right
10.    newNode.right <- curr.right
11.    curr.right <- newNode
12. END IF
13. // Vertical splice into colHead[c]
14. IF mat.colHead[c] == NULL OR mat.colHead[c].row > r THEN
15.    newNode.down <- mat.colHead[c]
16.    mat.colHead[c] <- newNode
17. ELSE
18.    curr <- mat.colHead[c]
19.    WHILE curr.down != NULL AND curr.down.row < r DO curr <- curr.down
20.    newNode.down <- curr.down
21.    curr.down <- newNode
22. END IF
23. mat.nnz <- mat.nnz + 1
```

#### 5. Before-and-After ASCII Pointer Diagram
Inserting `(1, 2, 99)` into Row 1 (containing `(1,0,10) -> (1,3,22)`) and Col 2 (containing `(0,2,5) -> (3,2,40)`):
```text
                         colHead[2]
                             │
                             ▼
                         (0,2:  5)
                             ║ (down pointer updated)
                             ▼
rowHead[1] ──► (1,0: 10) ══► [1,2: 99] ══► (1,3: 22) ──► NULL
                             ║
                             ▼
                         (3,2: 40)
                             │
                             ▼
                           NULL
```

#### 6. Reusable C Function
```c
bool insertCrossMatrix(CrossLinkedMatrix *mat, int r, int c, int val) {
    if (mat == NULL || r < 0 || r >= mat->rows || c < 0 || c >= mat->cols || val == 0) {
        return false;
    }

    SMNode *newNode = (SMNode *)malloc(sizeof(SMNode));
    if (newNode == NULL) {
        return false;
    }
    newNode->row = r;
    newNode->col = c;
    newNode->val = val;
    newNode->right = NULL;
    newNode->down = NULL;

    /* 1. Insert horizontally into row r (sorted by col) */
    if (mat->rowHead[r] == NULL || mat->rowHead[r]->col > c) {
        newNode->right = mat->rowHead[r];
        mat->rowHead[r] = newNode;
    } else {
        SMNode *curr = mat->rowHead[r];
        while (curr->right != NULL && curr->right->col < c) {
            curr = curr->right;
        }
        newNode->right = curr->right;
        curr->right = newNode;
    }

    /* 2. Insert vertically into column c (sorted by row) */
    if (mat->colHead[c] == NULL || mat->colHead[c]->row > r) {
        newNode->down = mat->colHead[c];
        mat->colHead[c] = newNode;
    } else {
        SMNode *curr = mat->colHead[c];
        while (curr->down != NULL && curr->down->row < r) {
            curr = curr->down;
        }
        newNode->down = curr->down;
        curr->down = newNode;
    }

    mat->nnz++;
    return true;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $O(\text{nnz}_{\text{row } r} + \text{nnz}_{\text{col } c}) = O(r + c)$ worst case.
- **Auxiliary Space Complexity**: $O(1)$.

#### 8. Edge Cases and Common Mistakes
- **Both links must be wired**: Every node in an orthogonal list must be inserted into *both* `rowHead[r]` and `colHead[c]`.

---

### 4.3 Sparse Matrix Addition

#### 1. Purpose
Compute $C = A + B$ for two $r \times c$ sparse matrices represented as Triplet arrays in $O(\text{nnz}_A + \text{nnz}_B)$ time using a two-pointer merge.

#### 2. Preconditions and Assumptions
- Dimensions match: `A[0].row == B[0].row` and `A[0].col == B[0].col`.
- Both `A` and `B` have their triplets sorted in row-major order.

#### 3. Step-by-Step Algorithm
1. Verify that dimensions of `A` and `B` match. Set `C[0].row = A[0].row`, `C[0].col = A[0].col`.
2. Initialize two read indices `i = 1` (for `A`), `j = 1` (for `B`), and write index `k = 1` (for `C`).
3. While `i <= A[0].val` and `j <= B[0].val`:
   - Compare lexicographical coordinates `(A[i].row, A[i].col)` vs. `(B[j].row, B[j].col)`:
   - If `A[i]` precedes `B[j]`: copy `C[k++] = A[i++]`.
   - Else if `B[j]` precedes `A[i]`: copy `C[k++] = B[j++]`.
   - Else (same `row` and `col`):
     - Compute `sum = A[i].val + B[j].val`.
     - **If `sum != 0`**, write `C[k++] = (A[i].row, A[i].col, sum)`. (If `sum == 0`, skip it so zero entries are never stored!).
     - Advance both `i++` and `j++`.
4. Copy any remaining triplets from `A` or `B` into `C`.
5. Set `C[0].val = k - 1` (total non-zero count in $C$).

#### 4. Pseudocode
```text
ALGORITHM AddSparseTriplets(A, B, C)
1. IF A[0].row != B[0].row OR A[0].col != B[0].col THEN RETURN ERROR
2. i <- 1; j <- 1; k <- 1
3. WHILE i <= A[0].val AND j <= B[0].val DO
4.     IF (A[i].row < B[j].row) OR (A[i].row == B[j].row AND A[i].col < B[j].col) THEN
5.         C[k++] <- A[i++]
6.     ELSE IF (B[j].row < A[i].row) OR (B[j].row == A[i].row AND B[j].col < A[i].col) THEN
7.         C[k++] <- B[j++]
8.     ELSE
9.         sum <- A[i].val + B[j].val
10.        IF sum != 0 THEN C[k++] <- (A[i].row, A[i].col, sum)
11.        i <- i + 1; j <- j + 1
12.    END IF
13. END WHILE
14. WHILE i <= A[0].val DO C[k++] <- A[i++]
15. WHILE j <= B[0].val DO C[k++] <- B[j++]
16. C[0] <- (A[0].row, A[0].col, k - 1)
```

#### 5. Worked Trace
Adding $A$ and $B$:
```text
A[1..3]: (0,1, 15), (1,3, 22), (2,0,  9)
B[1..3]: (0,1, -5), (1,3,-22), (3,2, 11)

Step 1: Compare (0,1) vs (0,1) -> Equal! sum = 15 + (-5) = 10 != 0 -> C[1] = (0,1, 10)
Step 2: Compare (1,3) vs (1,3) -> Equal! sum = 22 + (-22) = 0      -> CANCELLED (Skipped!)
Step 3: Compare (2,0) vs (3,2) -> (2,0) < (3,2)                    -> C[2] = (2,0,  9)
Step 4: A exhausted; copy remaining B:                             -> C[3] = (3,2, 11)
Result Header C[0] = (4, 4, 3).
```

#### 6. Reusable C Function
```c
bool addSparseTriplets(const Triplet A[], const Triplet B[], Triplet C[]) {
    if (A[0].row != B[0].row || A[0].col != B[0].col) {
        fprintf(stderr, "Error: Matrix dimensions do not match for addition.\n");
        return false;
    }

    C[0].row = A[0].row;
    C[0].col = A[0].col;

    int i = 1, j = 1, k = 1;
    int nnzA = A[0].val, nnzB = B[0].val;

    while (i <= nnzA && j <= nnzB) {
        if (A[i].row < B[j].row || (A[i].row == B[j].row && A[i].col < B[j].col)) {
            C[k++] = A[i++];
        } else if (B[j].row < A[i].row || (B[j].row == A[i].row && B[j].col < A[i].col)) {
            C[k++] = B[j++];
        } else {
            int sum = A[i].val + B[j].val;
            if (sum != 0) {
                C[k].row = A[i].row;
                C[k].col = A[i].col;
                C[k].val = sum;
                k++;
            }
            i++;
            j++;
        }
    }

    while (i <= nnzA) {
        C[k++] = A[i++];
    }
    while (j <= nnzB) {
        C[k++] = B[j++];
    }

    C[0].val = k - 1;
    return true;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $\Theta(\text{nnz}_A + \text{nnz}_B)$ — single linear merge pass! Contrast this with $\Theta(r \cdot c)$ for dense 2D array addition.
- **Auxiliary Space Complexity**: $O(1)$ beyond the output array `C`.

#### 8. Edge Cases and Common Mistakes
- **Exact Cancellation (`A[i].val + B[j].val == 0`)**: Must NOT be written into `C`.

---

### 4.4 Complete, Compilable C11 Program: Sparse Matrices (Triplet & Multilist)

```c
/**
 * File: sparse_matrix_demo.c
 * Standard: C11
 * Description: Demonstrates Sparse Matrix representations and operations:
 *              1. 3-Tuple (Triplet) Creation, Display, Addition, and Fast Transpose.
 *              2. Orthogonal Cross-Linked List (Multilist) Creation, Row/Col Display, Free.
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_TERMS 50

/* --- Part A: 3-Tuple (Triplet) Representation --- */
typedef struct Triplet {
    int row;
    int col;
    int val;
} Triplet;

void createTripletFromDense(int rows, int cols, const int dense[rows][cols], Triplet T[]) {
    T[0].row = rows;
    T[0].col = cols;
    int k = 1;
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            if (dense[i][j] != 0) {
                T[k].row = i;
                T[k].col = j;
                T[k].val = dense[i][j];
                k++;
            }
        }
    }
    T[0].val = k - 1;
}

void displayTripletTable(const char *label, const Triplet T[]) {
    printf("%s (Rows=%d, Cols=%d, NNZ=%d):\n", label, T[0].row, T[0].col, T[0].val);
    printf("  Idx | Row | Col | Value\n");
    printf("  ----+-----+-----+------\n");
    for (int i = 0; i <= T[0].val; i++) {
        printf("  [%d] |  %2d |  %2d |  %4d%s\n",
               i, T[i].row, T[i].col, T[i].val, (i == 0) ? "  <-- Header" : "");
    }
    printf("\n");
}

void fastTransposeTriplet(const Triplet A[], Triplet B[]) {
    int r = A[0].row, c = A[0].col, nnz = A[0].val;
    B[0].row = c;
    B[0].col = r;
    B[0].val = nnz;
    if (nnz <= 0) return;

    int *rowTerms = (int *)calloc((size_t)c, sizeof(int));
    int *startPos = (int *)malloc((size_t)c * sizeof(int));
    if (rowTerms == NULL || startPos == NULL) {
        free(rowTerms);
        free(startPos);
        return;
    }

    for (int i = 1; i <= nnz; i++) {
        rowTerms[A[i].col]++;
    }
    startPos[0] = 1;
    for (int j = 1; j < c; j++) {
        startPos[j] = startPos[j - 1] + rowTerms[j - 1];
    }
    for (int i = 1; i <= nnz; i++) {
        int pos = startPos[A[i].col]++;
        B[pos].row = A[i].col;
        B[pos].col = A[i].row;
        B[pos].val = A[i].val;
    }
    free(rowTerms);
    free(startPos);
}

bool addSparseTriplets(const Triplet A[], const Triplet B[], Triplet C[]) {
    if (A[0].row != B[0].row || A[0].col != B[0].col) return false;
    C[0].row = A[0].row;
    C[0].col = A[0].col;
    int i = 1, j = 1, k = 1;
    while (i <= A[0].val && j <= B[0].val) {
        if (A[i].row < B[j].row || (A[i].row == B[j].row && A[i].col < B[j].col)) {
            C[k++] = A[i++];
        } else if (B[j].row < A[i].row || (B[j].row == A[i].row && B[j].col < A[i].col)) {
            C[k++] = B[j++];
        } else {
            int sum = A[i].val + B[j].val;
            if (sum != 0) {
                C[k].row = A[i].row;
                C[k].col = A[i].col;
                C[k].val = sum;
                k++;
            }
            i++;
            j++;
        }
    }
    while (i <= A[0].val) C[k++] = A[i++];
    while (j <= B[0].val) C[k++] = B[j++];
    C[0].val = k - 1;
    return true;
}

/* --- Part B: Orthogonal Cross-Linked Multilist Representation --- */
typedef struct SMNode {
    int row;
    int col;
    int val;
    struct SMNode *right;
    struct SMNode *down;
} SMNode;

typedef struct CrossLinkedMatrix {
    int rows;
    int cols;
    int nnz;
    SMNode **rowHead;
    SMNode **colHead;
} CrossLinkedMatrix;

CrossLinkedMatrix* createCrossMatrix(int rows, int cols) {
    CrossLinkedMatrix *mat = (CrossLinkedMatrix *)malloc(sizeof(CrossLinkedMatrix));
    if (mat == NULL) return NULL;
    mat->rows = rows;
    mat->cols = cols;
    mat->nnz = 0;
    mat->rowHead = (SMNode **)calloc((size_t)rows, sizeof(SMNode *));
    mat->colHead = (SMNode **)calloc((size_t)cols, sizeof(SMNode *));
    if (mat->rowHead == NULL || mat->colHead == NULL) {
        free(mat->rowHead);
        free(mat->colHead);
        free(mat);
        return NULL;
    }
    return mat;
}

bool insertCrossMatrix(CrossLinkedMatrix *mat, int r, int c, int val) {
    if (mat == NULL || r < 0 || r >= mat->rows || c < 0 || c >= mat->cols || val == 0) {
        return false;
    }
    SMNode *newNode = (SMNode *)malloc(sizeof(SMNode));
    if (newNode == NULL) return false;
    newNode->row = r;
    newNode->col = c;
    newNode->val = val;
    newNode->right = NULL;
    newNode->down = NULL;

    if (mat->rowHead[r] == NULL || mat->rowHead[r]->col > c) {
        newNode->right = mat->rowHead[r];
        mat->rowHead[r] = newNode;
    } else {
        SMNode *curr = mat->rowHead[r];
        while (curr->right != NULL && curr->right->col < c) curr = curr->right;
        newNode->right = curr->right;
        curr->right = newNode;
    }

    if (mat->colHead[c] == NULL || mat->colHead[c]->row > r) {
        newNode->down = mat->colHead[c];
        mat->colHead[c] = newNode;
    } else {
        SMNode *curr = mat->colHead[c];
        while (curr->down != NULL && curr->down->row < r) curr = curr->down;
        newNode->down = curr->down;
        curr->down = newNode;
    }
    mat->nnz++;
    return true;
}

void displayCrossMatrix(const CrossLinkedMatrix *mat) {
    printf("Orthogonal Cross-Linked Matrix (%dx%d, NNZ=%d):\n", mat->rows, mat->cols, mat->nnz);
    printf("  [Horizontal Row Lists]:\n");
    for (int i = 0; i < mat->rows; i++) {
        printf("    Row %d: ", i);
        const SMNode *curr = mat->rowHead[i];
        while (curr != NULL) {
            printf("(%d,%d: %d) -> ", curr->row, curr->col, curr->val);
            curr = curr->right;
        }
        printf("NULL\n");
    }
    printf("  [Vertical Column Lists]:\n");
    for (int j = 0; j < mat->cols; j++) {
        printf("    Col %d: ", j);
        const SMNode *curr = mat->colHead[j];
        while (curr != NULL) {
            printf("(%d,%d: %d) v ", curr->row, curr->col, curr->val);
            curr = curr->down;
        }
        printf("NULL\n");
    }
}

void freeCrossMatrix(CrossLinkedMatrix *mat) {
    if (mat == NULL) return;
    /* Free nodes by traversing ONLY rows (never free along both rows and cols!) */
    for (int i = 0; i < mat->rows; i++) {
        SMNode *curr = mat->rowHead[i];
        while (curr != NULL) {
            SMNode *next = curr->right;
            free(curr);
            curr = next;
        }
    }
    free(mat->rowHead);
    free(mat->colHead);
    free(mat);
}

int main(void) {
    int denseA[4][4] = {
        { 0, 15,  0,  0},
        { 0,  0,  0, 22},
        { 9,  0,  0,  0},
        { 0,  0,  0, -5}
    };
    int denseB[4][4] = {
        { 0, -5,  0,  0},
        { 0,  0,  0,-22},
        { 0,  0,  0,  0},
        { 0,  0, 11,  5}
    };

    Triplet A[MAX_TERMS], B[MAX_TERMS], C[MAX_TERMS], AT[MAX_TERMS];

    createTripletFromDense(4, 4, denseA, A);
    createTripletFromDense(4, 4, denseB, B);

    displayTripletTable("Matrix A Triplet Table", A);
    fastTransposeTriplet(A, AT);
    displayTripletTable("Fast Transpose A^T Triplet Table", AT);

    addSparseTriplets(A, B, C);
    displayTripletTable("Sum Matrix C = A + B (Note (1,3) and (3,3) cancelled to 0!)", C);

    CrossLinkedMatrix *crossA = createCrossMatrix(4, 4);
    for (int i = 1; i <= A[0].val; i++) {
        insertCrossMatrix(crossA, A[i].row, A[i].col, A[i].val);
    }
    displayCrossMatrix(crossA);
    freeCrossMatrix(crossA);

    return 0;
}
```

#### Program Output:
```text
Matrix A Triplet Table (Rows=4, Cols=4, NNZ=4):
  Idx | Row | Col | Value
  ----+-----+-----+------
  [0] |   4 |   4 |     4  <-- Header
  [1] |   0 |   1 |    15
  [2] |   1 |   3 |    22
  [3] |   2 |   0 |     9
  [4] |   3 |   3 |    -5

Fast Transpose A^T Triplet Table (Rows=4, Cols=4, NNZ=4):
  Idx | Row | Col | Value
  ----+-----+-----+------
  [0] |   4 |   4 |     4  <-- Header
  [1] |   0 |   2 |     9
  [2] |   1 |   0 |    15
  [3] |   3 |   1 |    22
  [4] |   3 |   3 |    -5

Sum Matrix C = A + B (Note (1,3) and (3,3) cancelled to 0!) (Rows=4, Cols=4, NNZ=3):
  Idx | Row | Col | Value
  ----+-----+-----+------
  [0] |   4 |   4 |     3  <-- Header
  [1] |   0 |   1 |    10
  [2] |   2 |   0 |     9
  [3] |   3 |   2 |    11

Orthogonal Cross-Linked Matrix (4x4, NNZ=4):
  [Horizontal Row Lists]:
    Row 0: (0,1: 15) -> NULL
    Row 1: (1,3: 22) -> NULL
    Row 2: (2,0: 9) -> NULL
    Row 3: (3,3: -5) -> NULL
  [Vertical Column Lists]:
    Col 0: (2,0: 9) v NULL
    Col 1: (0,1: 15) v NULL
    Col 2: NULL
    Col 3: (1,3: 22) v (3,3: -5) v NULL
```

---

### 4.5 Skip List Search

#### 1. Purpose
Locate a node with `targetKey` in a Skip List in $O(\log n)$ expected time by traversing top-down from the highest active express lane (`list->level`) down to base `level 0`.

#### 2. Preconditions and Assumptions
- `list != NULL` and `list->header != NULL`.
- All keys at every level $i \in [0, \text{list->level}]$ are sorted in strictly ascending order.

#### 3. Step-by-Step Algorithm
1. Initialize `curr = list->header`.
2. Loop `i` from `list->level` down to `0`:
   - While `curr->forward[i] != NULL` and `curr->forward[i]->key < targetKey`:
     - Advance `curr = curr->forward[i]` (move right along level `i`).
   - (When the inner loop ends, either `curr->forward[i]` is `NULL` or its key is $\ge \text{targetKey}$, so we drop down to level `i - 1`).
3. Having reached level `0` right before the candidate position, step forward once: `curr = curr->forward[0]`.
4. If `curr != NULL && curr->key == targetKey`, return `curr`; otherwise return `NULL`.

#### 4. Pseudocode
```text
ALGORITHM SkipListSearch(list, targetKey)
1. curr <- list.header
2. FOR i <- list.level DOWNTO 0 DO
3.     WHILE curr.forward[i] != NULL AND curr.forward[i].key < targetKey DO
4.         curr <- curr.forward[i]
5.     END WHILE
6. END FOR
7. curr <- curr.forward[0]
8. IF curr != NULL AND curr.key == targetKey THEN
9.     RETURN curr
10. END IF
11. RETURN NULL
```

#### 5. ASCII Diagram / Worked Trace
Searching for `targetKey = 40` in a Skip List with keys `{10, 20, 30, 40, 50}`:
```text
Level 2: [HDR] ────────────────────────► [30] ───────────────────────► NULL
                                           │ (next is NULL -> drop to L1)
Level 1: [HDR] ──────────► [20] ───────► [30] ───────────────► [50] ─► NULL
                                           │ (50 >= 40 -> drop to L0)
Level 0: [HDR] ─► [10] ──► [20] ───────► [30] ───► [40] ─────► [50] ─► NULL
                                           └───► curr->forward[0] is [40] (MATCH!)
```

#### 6. Reusable C Function
```c
SkipNode* skipListSearch(const SkipList *list, int targetKey) {
    if (list == NULL || list->header == NULL) {
        return NULL;
    }
    SkipNode *curr = list->header;
    for (int i = list->level; i >= 0; i--) {
        while (curr->forward[i] != NULL && curr->forward[i]->key < targetKey) {
            curr = curr->forward[i];
        }
    }
    curr = curr->forward[0];
    if (curr != NULL && curr->key == targetKey) {
        return curr;
    }
    return NULL;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $O(\log n)$ expected / average case; $O(n)$ worst case.
- **Auxiliary Space Complexity**: $O(1)$.

#### 8. Edge Cases and Common Mistakes
- **Why use `< targetKey` instead of `<= targetKey` in the `while` loop?** Stopping while the next key is strictly less than `targetKey` guarantees that `curr` stops at the *predecessor* of `targetKey` at every level. This exact invariant is reused by both `insert` and `delete`!

---

### 4.6 Skip List Insertion (with Random Level Generation and `update[]`)

#### 1. Purpose
Insert `(key, value)` into the Skip List (or update `value` if `key` already exists), promoting the new node's tower height probabilistically via coin flips ($p = 1/2$).

#### 2. Preconditions and Assumptions
- `MAX_LEVEL` is defined as the upper bound on tower height.

#### 3. Step-by-Step Algorithm
1. Declare `SkipNode *update[MAX_LEVEL + 1]` and set `curr = list->header`.
2. For `i = list->level` down to `0`:
   - Advance `curr = curr->forward[i]` while `curr->forward[i] != NULL && curr->forward[i]->key < key`.
   - Record `update[i] = curr` (the rightmost node at level `i` preceding `key`).
3. Advance `curr = curr->forward[0]`.
4. If `curr != NULL && curr->key == key`:
   - Key already exists; update `curr->value = value` and return `true`.
5. Otherwise, generate a random tower height `newLevel = randomLevel()` ($0 \le \text{newLevel} \le \text{MAX\_LEVEL}$):
   - Start `lvl = 0`. While `(rand() & 1) == 1 && lvl < MAX_LEVEL`, increment `lvl++`.
6. If `newLevel > list->level`:
   - For `i = list->level + 1` to `newLevel`, set `update[i] = list->header`.
   - Update `list->level = newLevel`.
7. Allocate `newNode(key, value, newLevel)`.
8. Splice `newNode` into levels `0 .. newLevel`:
   - For `i = 0` to `newLevel`:
     - `newNode->forward[i] = update[i]->forward[i]`
     - `update[i]->forward[i] = newNode`
9. Increment `list->size` and return `true`.

#### 4. Pseudocode
```text
ALGORITHM SkipListInsert(list, key, value)
1. curr <- list.header
2. FOR i <- list.level DOWNTO 0 DO
3.     WHILE curr.forward[i] != NULL AND curr.forward[i].key < key DO
4.         curr <- curr.forward[i]
5.     END WHILE
6.     update[i] <- curr
7. END FOR
8. curr <- curr.forward[0]
9. IF curr != NULL AND curr.key == key THEN
10.    curr.value <- value; RETURN
11. END IF
12. newLevel <- RandomLevel()
13. IF newLevel > list.level THEN
14.    FOR i <- list.level + 1 TO newLevel DO update[i] <- list.header
15.    list.level <- newLevel
16. END IF
17. newNode <- AllocateSkipNode(key, value, newLevel)
18. FOR i <- 0 TO newLevel DO
19.    newNode.forward[i] <- update[i].forward[i]
20.    update[i].forward[i] <- newNode
21. END FOR
```

#### 5. Before-and-After ASCII Pointer Diagram
Inserting `key = 35` with `randomLevel() = 1` between `30` and `50`:
```text
BEFORE (update[1] = [30], update[0] = [30]):
Level 2: [HDR] ────────────────────────► [30] ───────────────────────► [60] ─► NULL
Level 1: [HDR] ──────────► [20] ───────► [30] ───────────────► [50] ─► [60] ─► NULL
Level 0: [HDR] ─► [10] ──► [20] ───────► [30] ───────────────► [50] ─► [60] ─► NULL
                                          ▲
                                  update[0], update[1]

AFTER SPLICING [35] AT LEVELS 0 AND 1:
Level 2: [HDR] ────────────────────────► [30] ───────────────────────► [60] ─► NULL
Level 1: [HDR] ──────────► [20] ───────► [30] ──► [35] ──────► [50] ─► [60] ─► NULL
Level 0: [HDR] ─► [10] ──► [20] ───────► [30] ──► [35] ──────► [50] ─► [60] ─► NULL
```

#### 6. Reusable C Functions
```c
int randomLevel(void) {
    int lvl = 0;
    while ((rand() & 1) == 1 && lvl < MAX_LEVEL) {
        lvl++;
    }
    return lvl;
}

bool skipListInsert(SkipList *list, int key, int value) {
    SkipNode *update[MAX_LEVEL + 1];
    SkipNode *curr = list->header;

    for (int i = list->level; i >= 0; i--) {
        while (curr->forward[i] != NULL && curr->forward[i]->key < key) {
            curr = curr->forward[i];
        }
        update[i] = curr;
    }

    curr = curr->forward[0];
    if (curr != NULL && curr->key == key) {
        curr->value = value; /* Update existing key */
        return true;
    }

    int newLevel = randomLevel();
    if (newLevel > list->header->nodeLevel) {
        newLevel = list->header->nodeLevel;
    }

    if (newLevel > list->level) {
        for (int i = list->level + 1; i <= newLevel; i++) {
            update[i] = list->header;
        }
        list->level = newLevel;
    }

    SkipNode *newNode = createSkipNode(key, value, newLevel);
    if (newNode == NULL) {
        return false;
    }

    for (int i = 0; i <= newLevel; i++) {
        newNode->forward[i] = update[i]->forward[i];
        update[i]->forward[i] = newNode;
    }

    list->size++;
    return true;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $O(\log n)$ expected time; $O(n)$ worst case.
- **Auxiliary Space Complexity**: $O(L_{\max}) = O(\log n)$ for the `update[]` array on the stack.

#### 8. Edge Cases and Common Mistakes
- **Initializing `update[list->level + 1 .. newLevel]`**: When `newLevel > list->level`, those higher levels were not visited during the search loop, so `update[i]` must be explicitly set to `list->header` before splicing!

---

### 4.7 Skip List Deletion

#### 1. Purpose
Remove and deallocate the node with `targetKey` across all levels it participates in, and shrink `list->level` if the highest levels become empty.

#### 2. Preconditions and Assumptions
- `list != NULL`.

#### 3. Step-by-Step Algorithm
1. Declare `SkipNode *update[MAX_LEVEL + 1]` and set `curr = list->header`.
2. For `i = list->level` down to `0`:
   - While `curr->forward[i] != NULL && curr->forward[i]->key < targetKey`, advance `curr = curr->forward[i]`.
   - Set `update[i] = curr`.
3. Set `curr = curr->forward[0]`.
4. If `curr == NULL || curr->key != targetKey`, return `false` (key not found).
5. For `i = 0` to `list->level`:
   - If `update[i]->forward[i] != curr`, `break` (the target node's tower does not reach level `i`).
   - Bypass `curr` at level `i`: `update[i]->forward[i] = curr->forward[i]`.
6. Call `free(curr)`.
7. While `list->level > 0 && list->header->forward[list->level] == NULL`, decrement `list->level--`.
8. Decrement `list->size--` and return `true`.

#### 4. Pseudocode
```text
ALGORITHM SkipListDelete(list, targetKey)
1. curr <- list.header
2. FOR i <- list.level DOWNTO 0 DO
3.     WHILE curr.forward[i] != NULL AND curr.forward[i].key < targetKey DO
4.         curr <- curr.forward[i]
5.     END WHILE
6.     update[i] <- curr
7. END FOR
8. curr <- curr.forward[0]
9. IF curr == NULL OR curr.key != targetKey THEN RETURN FALSE
10. FOR i <- 0 TO list.level DO
11.    IF update[i].forward[i] != curr THEN BREAK
12.    update[i].forward[i] <- curr.forward[i]
13. END FOR
14. FreeNode(curr)
15. WHILE list.level > 0 AND list.header.forward[list.level] == NULL DO
16.    list.level <- list.level - 1
17. END WHILE
18. RETURN TRUE
```

#### 5. Before-and-After ASCII Pointer Diagram
Deleting `[35]` (which had height 1) from `{10, 20, 30, 35, 50, 60}`:
```text
BEFORE:
Level 1: ... ──► [30] ──► [35] ──► [50] ──► ...
Level 0: ... ──► [30] ──► [35] ──► [50] ──► ...

AFTER (update[0]->forward[0] = [50]; update[1]->forward[1] = [50]; free([35])):
Level 1: ... ──► [30] ───────────► [50] ──► ...
Level 0: ... ──► [30] ───────────► [50] ──► ...
```

#### 6. Reusable C Function
```c
bool skipListDelete(SkipList *list, int targetKey) {
    SkipNode *update[MAX_LEVEL + 1];
    SkipNode *curr = list->header;

    for (int i = list->level; i >= 0; i--) {
        while (curr->forward[i] != NULL && curr->forward[i]->key < targetKey) {
            curr = curr->forward[i];
        }
        update[i] = curr;
    }

    curr = curr->forward[0];
    if (curr == NULL || curr->key != targetKey) {
        return false; /* Key not found */
    }

    for (int i = 0; i <= list->level; i++) {
        if (update[i]->forward[i] != curr) {
            break;
        }
        update[i]->forward[i] = curr->forward[i];
    }

    free(curr);

    while (list->level > 0 && list->header->forward[list->level] == NULL) {
        list->level--;
    }

    list->size--;
    return true;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $O(\log n)$ expected; $O(n)$ worst case.
- **Auxiliary Space Complexity**: $O(L_{\max}) = O(\log n)$.

#### 8. Edge Cases and Common Mistakes
- **Shrinking empty top levels**: If the deleted node was the *only* node reaching `list->level`, `list->header->forward[list->level]` becomes `NULL`. The `while` loop at Step 7 cleans up empty top levels so future searches don't waste iterations scanning empty levels.

---

### 4.8 Complete, Compilable C11 Program: Skip List

To make output deterministic and reproducible across runs while using genuine coin-flip level generation, we seed `srand(42)` in `main(void)`.

```c
/**
 * File: skip_list_demo.c
 * Standard: C11
 * Description: Complete Skip List implementation with Search, Insert,
 *              Delete, Multi-Level Display, and Safe Memory Cleanup.
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <limits.h>

#define MAX_LEVEL 5

typedef struct SkipNode {
    int key;
    int value;
    int nodeLevel;
    struct SkipNode *forward[MAX_LEVEL + 1];
} SkipNode;

typedef struct SkipList {
    int level;
    int size;
    SkipNode *header;
} SkipList;

SkipNode* createSkipNode(int key, int value, int level) {
    SkipNode *node = (SkipNode *)malloc(sizeof(SkipNode));
    if (node == NULL) {
        fprintf(stderr, "Error: SkipNode allocation failed.\n");
        return NULL;
    }
    node->key = key;
    node->value = value;
    node->nodeLevel = level;
    for (int i = 0; i <= MAX_LEVEL; i++) {
        node->forward[i] = NULL;
    }
    return node;
}

SkipList* createSkipList(void) {
    SkipList *list = (SkipList *)malloc(sizeof(SkipList));
    if (list == NULL) return NULL;
    list->level = 0;
    list->size = 0;
    list->header = createSkipNode(INT_MIN, 0, MAX_LEVEL);
    if (list->header == NULL) {
        free(list);
        return NULL;
    }
    return list;
}

/* Deterministic pseudo-random coin flip (p = 1/2) for reproducible trace */
static unsigned int rng_state = 109u;
int randomLevel(void) {
    int lvl = 0;
    while (lvl < MAX_LEVEL) {
        rng_state = rng_state * 1103515245u + 12345u;
        if (((rng_state >> 16) & 1u) == 1u) {
            lvl++;
        } else {
            break;
        }
    }
    return lvl;
}

bool skipListInsert(SkipList *list, int key, int value) {
    SkipNode *update[MAX_LEVEL + 1];
    SkipNode *curr = list->header;

    for (int i = list->level; i >= 0; i--) {
        while (curr->forward[i] != NULL && curr->forward[i]->key < key) {
            curr = curr->forward[i];
        }
        update[i] = curr;
    }

    curr = curr->forward[0];
    if (curr != NULL && curr->key == key) {
        curr->value = value;
        return true;
    }

    int newLevel = randomLevel();
    if (newLevel > list->level) {
        for (int i = list->level + 1; i <= newLevel; i++) {
            update[i] = list->header;
        }
        list->level = newLevel;
    }

    SkipNode *newNode = createSkipNode(key, value, newLevel);
    if (newNode == NULL) return false;

    for (int i = 0; i <= newLevel; i++) {
        newNode->forward[i] = update[i]->forward[i];
        update[i]->forward[i] = newNode;
    }

    list->size++;
    return true;
}

SkipNode* skipListSearch(const SkipList *list, int targetKey, int *stepsOut) {
    int steps = 0;
    SkipNode *curr = list->header;
    for (int i = list->level; i >= 0; i--) {
        while (curr->forward[i] != NULL && curr->forward[i]->key < targetKey) {
            curr = curr->forward[i];
            steps++;
        }
        steps++; /* Count level drop */
    }
    curr = curr->forward[0];
    if (stepsOut != NULL) *stepsOut = steps;
    if (curr != NULL && curr->key == targetKey) {
        return curr;
    }
    return NULL;
}

bool skipListDelete(SkipList *list, int targetKey) {
    SkipNode *update[MAX_LEVEL + 1];
    SkipNode *curr = list->header;

    for (int i = list->level; i >= 0; i--) {
        while (curr->forward[i] != NULL && curr->forward[i]->key < targetKey) {
            curr = curr->forward[i];
        }
        update[i] = curr;
    }

    curr = curr->forward[0];
    if (curr == NULL || curr->key != targetKey) {
        return false;
    }

    for (int i = 0; i <= list->level; i++) {
        if (update[i]->forward[i] != curr) break;
        update[i]->forward[i] = curr->forward[i];
    }

    free(curr);
    while (list->level > 0 && list->header->forward[list->level] == NULL) {
        list->level--;
    }
    list->size--;
    return true;
}

void displaySkipList(const SkipList *list) {
    printf("Skip List (Size=%d, Max Active Level=%d):\n", list->size, list->level);
    for (int i = list->level; i >= 0; i--) {
        printf("  Level %d: [HDR] -> ", i);
        const SkipNode *curr = list->header->forward[i];
        while (curr != NULL) {
            printf("[%d] -> ", curr->key);
            curr = curr->forward[i];
        }
        printf("NULL\n");
    }
    printf("\n");
}

void freeSkipList(SkipList *list) {
    if (list == NULL) return;
    /* Level 0 links every single node in the Skip List! */
    SkipNode *curr = list->header;
    while (curr != NULL) {
        SkipNode *next = curr->forward[0];
        free(curr);
        curr = next;
    }
    free(list);
}

int main(void) {
    SkipList *list = createSkipList();
    int keys[] = {10, 20, 30, 40, 50, 60, 70, 25, 35};
    int n = (int)(sizeof(keys) / sizeof(keys[0]));

    printf("=== 1. Inserting %d Keys into Skip List ===\n", n);
    for (int i = 0; i < n; i++) {
        skipListInsert(list, keys[i], keys[i] * 10);
    }
    displaySkipList(list);

    printf("=== 2. Searching Keys ===\n");
    int steps = 0;
    SkipNode *found = skipListSearch(list, 35, &steps);
    if (found != NULL) {
        printf("Search(35): Found key=%d, val=%d (height=%d) in %d steps\n",
               found->key, found->value, found->nodeLevel, steps);
    }
    found = skipListSearch(list, 45, &steps);
    printf("Search(45): %s (checked in %d steps)\n\n",
           found ? "Found" : "Not Found", steps);

    printf("=== 3. Deleting Keys (30 and 70) ===\n");
    skipListDelete(list, 30);
    skipListDelete(list, 70);
    displaySkipList(list);

    freeSkipList(list);
    return 0;
}
```

#### Program Output:
```text
=== 1. Inserting 9 Keys into Skip List ===
Skip List (Size=9, Max Active Level=4):
  Level 4: [HDR] -> [10] -> NULL
  Level 3: [HDR] -> [10] -> [30] -> NULL
  Level 2: [HDR] -> [10] -> [30] -> [70] -> NULL
  Level 1: [HDR] -> [10] -> [30] -> [70] -> NULL
  Level 0: [HDR] -> [10] -> [20] -> [25] -> [30] -> [35] -> [40] -> [50] -> [60] -> [70] -> NULL

=== 2. Searching Keys ===
Search(35): Found key=35, val=350 (height=0) in 7 steps
Search(45): Not Found (checked in 9 steps)

=== 3. Deleting Keys (30 and 70) ===
Skip List (Size=7, Max Active Level=4):
  Level 4: [HDR] -> [10] -> NULL
  Level 3: [HDR] -> [10] -> NULL
  Level 2: [HDR] -> [10] -> NULL
  Level 1: [HDR] -> [10] -> NULL
  Level 0: [HDR] -> [10] -> [20] -> [25] -> [35] -> [40] -> [50] -> [60] -> NULL
```

---

## 5. Complexity Analysis and Comparison Tables

### 5.1 Sparse Matrix Operation Complexity Table
Let $r$ = rows, $c$ = columns, and $\text{nnz}$ = number of non-zero elements ($\text{nnz} \ll r \cdot c$).

| Operation | 2D Dense Array Time | Triplet (3-Tuple) Array Time | Orthogonal Cross-Linked Multilist Time | Triplet Aux Space | Multilist Aux Space |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Create from Coordinates** | $O(r \cdot c)$ (init) | $O(\text{nnz} \log \text{nnz})$ (sort) | $O(\text{nnz} \cdot (r + c))$ | $O(1)$ | $O(1)$ |
| **Access $(i, j)$** | $O(1)$ | $O(\log \text{nnz})$ (binary search) | $O(\min(\text{nnz}_{\text{row } i}, \text{nnz}_{\text{col } j}))$ | $O(1)$ | $O(1)$ |
| **Insert / Delete Single $(i, j)$** | $O(1)$ | $O(\text{nnz})$ (array shift) | $O(\text{nnz}_{\text{row } i} + \text{nnz}_{\text{col } j})$ | $O(1)$ | $O(1)$ |
| **Traverse Row $i$** | $O(c)$ | $O(\log \text{nnz} + \text{nnz}_{\text{row } i})$ | $\Theta(\text{nnz}_{\text{row } i})$ | $O(1)$ | $O(1)$ |
| **Traverse Column $j$** | $O(r)$ | $\Theta(\text{nnz})$ (full scan) | $\Theta(\text{nnz}_{\text{col } j})$ | $O(1)$ | $O(1)$ |
| **Matrix Addition ($A + B$)** | $\Theta(r \cdot c)$ | $\Theta(\text{nnz}_A + \text{nnz}_B)$ | $\Theta(r + \text{nnz}_A + \text{nnz}_B)$ | $O(1)$ | $O(1)$ |
| **Transpose ($A^T$)** | $\Theta(r \cdot c)$ | $\Theta(c + \text{nnz})$ (Fast Transpose) | $\Theta(r + c + \text{nnz})$ (swap pointers) | $O(c)$ | $O(1)$ |
| **Total Storage Space** | $\Theta(r \cdot c)$ | $\Theta(\text{nnz})$ ($3(\text{nnz}+1)$ ints) | $\Theta(r + c + \text{nnz})$ | — | — |

### 5.2 Comparison: 2D Array vs. Triplet Array vs. Cross-Linked Multilist

| Criterion | 2D Dense Array (`int A[r][c]`) | Triplet / 3-Tuple Array (`Triplet T[nnz+1]`) | Orthogonal Cross-Linked Multilist |
| :--- | :--- | :--- | :--- |
| **Best Use Case** | Dense matrices ($\delta > 33\%$) or frequent $O(1)$ random access. | Static sparse matrices ($\delta < 33\%$); batch addition & fast transpose. | Dynamic sparse matrices ($\delta < 12.5\%$) with frequent single-entry updates and both row & column scans. |
| **Memory per Non-Zero Entry** | $4\text{ B}$ (for *all* $r \times c$ cells, even zeros!) | $12\text{ B}$ (only for $\text{nnz}$ entries) | $\approx 32\text{ B}$ (node) $+ 8(r+c)\text{ B}$ (headers) |
| **Row AND Column Traversal** | $O(c)$ and $O(r)$ | Fast row scan; slow $O(\text{nnz})$ column scan | Fast $O(\text{nnz}_{\text{row}})$ and $O(\text{nnz}_{\text{col}})$ |
| **In-Place Point Mutation** | $O(1)$ | Poor ($O(\text{nnz})$ shifts to insert/remove) | Good (pointer splicing without shifting) |

### 5.3 Skip List Operation Complexity Table
Let $n$ be the number of elements in the Skip List, $p = 1/2$ be the promotion probability, and $L_{\max} = \lceil \log_2 n \rceil$.

| Operation | Best Case Time | Expected / Average Case Time | Worst Case Time (Pathological Coin Flips) | Auxiliary Space | Total Space (Expected) | Total Space (Worst Case) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Search (`skipListSearch`)** | $O(1)$ | $O(\log n)$ | $O(n)$ | $O(1)$ | $\Theta(n)$ | $O(n \log n)$ |
| **Insert (`skipListInsert`)** | $O(\log n)$ | $O(\log n)$ | $O(n)$ | $O(\log n)$ (`update[]`) | $\Theta(n)$ | $O(n \log n)$ |
| **Delete (`skipListDelete`)** | $O(\log n)$ | $O(\log n)$ | $O(n)$ | $O(\log n)$ (`update[]`) | $\Theta(n)$ | $O(n \log n)$ |
| **Find Min (`header->forward[0]`)** | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $\Theta(n)$ | $O(n \log n)$ |
| **Range Query $[k_1, k_2]$ ($m$ hits)** | $O(\log n + m)$ | $O(\log n + m)$ | $O(n)$ | $O(1)$ | $\Theta(n)$ | $O(n \log n)$ |

### 5.4 Comparison: Sorted Linked List vs. Skip List vs. Balanced BST

| Feature / Metric | Sorted Singly Linked List | Skip List ($p = 1/2$) | Balanced BST (AVL / Red-Black Tree) |
| :--- | :--- | :--- | :--- |
| **Search Time** | $\Theta(n)$ average & worst | $O(\log n)$ expected, $O(n)$ worst | $O(\log n)$ guaranteed worst case |
| **Insertion Time** | $\Theta(n)$ (to find position) | $O(\log n)$ expected | $O(\log n)$ guaranteed (with rotations) |
| **Deletion Time** | $\Theta(n)$ | $O(\log n)$ expected | $O(\log n)$ guaranteed (with rebalancing) |
| **Average Pointers per Node** | $1$ | $\frac{1}{1 - p} = 2$ (for $p = 1/2$) | $2$ (`left`, `right`) $+ 1$ (`parent`/color/height) |
| **Implementation Complexity** | Very Low | Low / Moderate (no tree rotations!) | High (single/double rotations, color cases) |
| **Range Traversal $[k_1, k_2]$** | Simple linear `.next` walk | Simple linear `.forward[0]` walk | Requires in-order tree traversal / stack |
| **Lock-Free Concurrency** | Difficult ($O(n)$ search) | Very natural (used in Java `ConcurrentSkipListMap`, Redis `ZSET`) | Extremely difficult (rotations touch multiple levels) |

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Pitfall 1: Retaining Zero-Sum Entries After Sparse Matrix Addition
When adding two sparse matrices $A$ and $B$, if $A[i,j] + B[i,j] == 0$, storing `(i, j, 0)` violates the sparse matrix invariant and corrupts `nnz`.

```c
/* BUG: Unconditionally writing the sum when coordinates match */
C[k].row = A[i].row;
C[k].col = A[i].col;
C[k].val = A[i].val + B[j].val; /* What if A[i].val == 5 and B[j].val == -5? */
k++;

/* FIX: Guard with (sum != 0) before writing to C[] */
int sum = A[i].val + B[j].val;
if (sum != 0) {
    C[k].row = A[i].row;
    C[k].col = A[i].col;
    C[k].val = sum;
    k++;
}
```

### 6.2 Pitfall 2: Updating Only Row Links and Forgetting Column Links in a Multilist
In an Orthogonal Cross-Linked Matrix, every node belongs to two lists (`rowHead[r]` and `colHead[c]`). Deleting or inserting a node in `rowHead[r]` without updating `colHead[c]` leaves a dangling pointer or invisible node in Column `c`.

```c
/* BUG: Freeing a node from rowHead[r] while colHead[c] still points to it! */
SMNode *target = mat->rowHead[r];
mat->rowHead[r] = target->right;
free(target); /* CRASH when traversing colHead[target->col] later! */

/* FIX: Unlink from BOTH rowHead[r] AND colHead[c] before calling free(target) */
unlinkFromRow(mat, r, target);
unlinkFromCol(mat, target->col, target);
free(target);
```

### 6.3 Pitfall 3: Out-of-Bounds Level Indexing in Skip List `forward[]` and `update[]`
If `MAX_LEVEL` is the highest valid level index (`0 .. MAX_LEVEL`), the array size must be `MAX_LEVEL + 1`. Furthermore, when `newLevel > list->level`, `update[list->level + 1 .. newLevel]` is uninitialized garbage unless explicitly set to `list->header`.

```c
/* BUG: Using uninitialized update[i] when newLevel > list->level */
int newLevel = randomLevel();
for (int i = 0; i <= newLevel; i++) {
    /* If newLevel > list->level, update[i] contains stack garbage! */
    newNode->forward[i] = update[i]->forward[i];
    update[i]->forward[i] = newNode;
}

/* FIX: Initialize newly activated upper levels of update[] to list->header */
if (newLevel > list->level) {
    for (int i = list->level + 1; i <= newLevel; i++) {
        update[i] = list->header;
    }
    list->level = newLevel;
}
```

### 6.4 Pitfall 4: Failing to Reduce Skip List `level` After Deleting the Tallest Node
If a single node reached level 5 and is deleted, leaving `list->level == 5` when levels 5, 4, and 3 are now empty causes subsequent operations to waste iterations on empty levels.

```c
/* BUG: Leaving list->level unchanged after deleting the sole top-level node */
free(curr);
list->size--;

/* FIX: Decrement list->level while the top header pointer is NULL */
free(curr);
while (list->level > 0 && list->header->forward[list->level] == NULL) {
    list->level--;
}
list->size--;
```

### 6.5 Boundary Conditions Checklist
- [ ] **All-Zero Sparse Matrix ($\text{nnz} = 0$)**: Does Triplet `T[0]` store `(r, c, 0)` and do `rowHead[]` / `colHead[]` contain all `NULL` pointers?
- [ ] **Opposite Matrices ($B = -A$)**: Does `addSparseTriplets(A, B, C)` produce `C[0].val == 0` with zero stored terms?
- [ ] **Duplicate Key Insertion in Skip List**: Does `skipListInsert` update `curr->value` instead of inserting a duplicate key?
- [ ] **Deleting from a 1-Element Skip List**: Does `list->level` return cleanly to `0` and `header->forward[0]` become `NULL`?
- [ ] **Multilist Deallocation**: Does `freeCrossMatrix` free nodes along rows *only* (never both rows and columns)?

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways
1. **Multilists** share a single dynamically allocated node across multiple independent logical lists using distinct pointer fields (`right`, `down`, etc.), eliminating data duplication.
2. **Sparse Matrices** save memory when $\text{nnz} \ll r \cdot c$. The **3-Tuple (Triplet) Array** uses $3(\text{nnz} + 1)$ integers (saving space when density $\delta < 1/3$) and supports **Fast Transpose** in $\Theta(c + \text{nnz})$ time and **Addition** in $\Theta(\text{nnz}_A + \text{nnz}_B)$ time.
3. **Orthogonal Cross-Linked Lists** link each non-zero entry into both a sorted row list (`right`) and a sorted column list (`down`), supporting dynamic insertions and fast row/column traversals.
4. **Skip Lists** layer probabilistic express lanes ($p = 1/2$) above a sorted Level-0 linked list, achieving **$O(\log n)$ expected time** for search, insertion, and deletion with an average of **2 pointers per node** ($\Theta(n)$ space).

### 7.2 Essential Invariants
- **Triplet Row-Major Order**: For $1 \le i < k \le \text{nnz}$, either $T[i].\text{row} < T[k].\text{row}$, or $(T[i].\text{row} == T[k].\text{row} \land T[i].\text{col} < T[k].\text{col})$.
- **No-Zero Invariant**: Neither a Triplet array nor an Orthogonal Cross-Linked Matrix ever stores an element with `val == 0`.
- **Skip List Tower Contiguity**: If a Skip List node has `nodeLevel = k`, it participates in every level $0, 1, \dots, k$ without gaps.
- **Skip List Predecessor Invariant**: After the top-down search loop for `key`, `update[i]->key < key` and `(update[i]->forward[i] == NULL || update[i]->forward[i]->key >= key)` for all $0 \le i \le \text{list->level}$.

### 7.3 Quick Self-Test Checklist
- [ ] For a $1000 \times 1000$ matrix of 4-byte `int`s, what is the maximum number of non-zero elements ($\text{nnz}$) for which the 3-Tuple representation uses less memory than a 2D array? *(Answer: $\text{nnz} < 333{,}332$)*
- [ ] How does Fast Transpose achieve $O(c + \text{nnz})$ time instead of $O(c \cdot \text{nnz})$?
- [ ] Why must you never traverse *both* `rowHead` and `colHead` when calling `free()` on an Orthogonal Cross-Linked Sparse Matrix?
- [ ] Why does a Skip List with $p = 1/2$ use an average of only $2$ pointers per node regardless of how large $n$ grows?
- [ ] What is the role of the `update[]` array during `skipListInsert` and `skipListDelete`?

### 7.4 Related Notes and Navigation
- **Previous**: [04 - Circular Linked Lists](./04-Circular-Linked-Lists.md)
- **Next**: [06 - Polynomial Representation and Operations](./06-Polynomial-Representation-and-Operations.md)
- **Course Index**: [DSA Notes README](../README.md)
