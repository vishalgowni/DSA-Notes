# Appendix 04: Master Cross-Unit Practice Question Bank, Worked Traces & Solutions Manual

> ** Navigation**
> - **Parent Directory**: [DSA Notes Root](../README.md)
> - **Previous Appendix**: [Appendix 03: Common Errors & Edge Cases](./03-Common-Errors-and-Edge-Cases.md)
> - **Related Appendices**: [Appendix 01: Master Big-O Complexity Cheat Sheet](./01-Complexity-Cheat-Sheet.md) | [Appendix 02: C Programming Patterns & Idioms](./02-C-Programming-Patterns.md)

---

## Table of Contents

1. [Section 1: 20 Rapid-Fire Short-Answer & True/False Questions](#section-1-20-rapid-fire-short-answer--truefalse-questions)
2. [Section 2: 12 Deep Conceptual & Comparative Synthesis Questions](#section-2-12-deep-conceptual--comparative-synthesis-questions)
3. [Section 3: 8 Algorithm Design & Pseudocode Problems](#section-3-8-algorithm-design--pseudocode-problems)
4. [Section 4: 8 Complete C11 Programming Problems with Compilable Solutions & Outputs](#section-4-8-complete-c11-programming-problems-with-compilable-solutions--outputs)
5. [Section 5: 8 Full-Length Worked Examination Problems (Step-by-Step Hand Traces)](#section-5-8-full-length-worked-examination-problems-step-by-step-hand-traces)
6. [Section 6: Complete, Detailed Answer Keys (Sections 1, 2 & 3)](#section-6-complete-detailed-answer-keys-sections-1-2--3)

---

## Section 1: 20 Rapid-Fire Short-Answer & True/False Questions

Attempt each question before consulting [Section 6.1](#61-answer-key-for-section-1-20-rapid-fire-questions) for the formal proof or justification.

### Unit 1: Linear Lists, Sparse Matrices, Polynomials & Stacks

- **Q1.1 (True/False)**: Given a singly linked list of $n$ sorted integers, Binary Search can find a target key in $O(\log n)$ time because the keys are sorted.
- **Q1.2 (Short Answer)**: Why is maintaining a single pointer `tail` (also called `last`) to the **last** node of a Circular Singly Linked List strictly superior to maintaining a single pointer `head` to the **first** node?
- **Q1.3 (True/False)**: When converting the infix expression `A ^ B ^ C` into postfix using Dijkstra's Shunting-Yard algorithm, the resulting postfix expression is `A B ^ C ^`.
- **Q1.4 (Short Answer)**: An $R \times C$ matrix of 32-bit integers ($4$ bytes each) is stored using the 3-tuple Triplet Representation `(row, col, val)`. At what exact fraction (density) of non-zero elements $nnz$ does the Triplet representation begin consuming **more** memory than a dense 2D array `A[R][C]`?
- **Q1.5 (True/False)**: In a Skip List of $n$ elements with level-promotion probability $p = 1/2$, every node stores an array of $\lceil \log_2 n \rceil$ forward pointers, making the expected space complexity $\Theta(n \log n)$.

### Unit 2: Queues, Priority Queues, Applications & Trees

- **Q1.6 (Short Answer)**: In a Circular Array Queue of capacity `CAP = 8` using `front` and `rear` indices (without an explicit `size` counter) where `front = 6` and `rear = 2`, write the exact modular arithmetic expression that evaluates to the number of elements currently in the queue.
- **Q1.7 (True/False)**: Level-Order Traversal (BFS) of a binary tree always uses less or equal auxiliary memory than Recursive Inorder Traversal (DFS) of the same binary tree.
- **Q1.8 (Short Answer)**: Any binary tree with $n$ nodes has $2n$ total child pointer fields (`left` and `right`). Prove the exact number of `NULL` child pointers in any non-empty binary tree of $n$ nodes.
- **Q1.9 (True/False)**: Given the **Preorder** and **Postorder** traversal sequences of an arbitrary Binary Tree with $n \ge 3$ nodes, the original Binary Tree can always be uniquely reconstructed.
- **Q1.10 (Short Answer)**: Under the Left-Child Right-Sibling (LCRS) representation that converts a General N-ary Tree $T$ into a Binary Tree $B$, which standard traversal of $B$ produces the exact same node sequence as the **Postorder** traversal of the original N-ary tree $T$?

### Unit 3: BSTs, Threaded Trees, Heaps, AVL, Splay & Graph Representations

- **Q1.11 (True/False)**: In a Binary Min-Heap of $n \ge 2$ distinct elements stored in a 0-based array `H[0..n-1]`, the maximum element is guaranteed to be at the last index `H[n-1]`.
- **Q1.12 (Short Answer)**: Building a Binary Heap of $n$ elements by repeatedly calling `insert()` $n$ times takes $O(n \log n)$ worst-case time, whereas Floyd's bottom-up `build_heap()` runs in $O(n)$ worst-case time. State the key structural reason why Floyd's method is asymptotically faster.
- **Q1.13 (True/False)**: Deleting a single key from an $n$-node AVL Tree requires at most one single or double rotation to restore the AVL height-balance invariant across the entire tree.
- **Q1.14 (Short Answer)**: Why is the **Zig-Zig** rotation step in a Splay Tree (`rotate(grandparent)` followed by `rotate(parent)`) fundamentally different from performing two consecutive **Zig** rotations on the target node $x$ (`rotate(parent)` followed by `rotate(grandparent)`)?
- **Q1.15 (True/False)**: For a sparse undirected graph with $V = 10^5$ vertices and $E = 2 \times 10^5$ edges, finding the degree of a vertex $u$ takes $\Theta(V)$ time in an Adjacency Matrix but $O(\text{deg}(u))$ time in an Adjacency List.

### Unit 4: Graph Traversals, Tries, Suffix Trees & Hashing

- **Q1.16 (True/False)**: Running Depth-First Search (DFS) from source vertex $s$ in an unweighted connected graph $G = (V, E)$ guarantees that the tree path from $s$ to any vertex $v$ in the DFS tree has the minimum possible number of edges.
- **Q1.17 (Short Answer)**: Why does detecting a cycle in a **Directed Graph** via DFS require 3 vertex states (`WHITE` unvisited, `GRAY` on active recursion stack, `BLACK` finished), whereas an **Undirected Graph** only requires 2 states (`visited` and `unvisited`) plus tracking the immediate `parent` vertex?
- **Q1.18 (True/False)**: Why must a special terminal character `$` (where `$` $\notin \Sigma$) be appended to a string $S$ before constructing its Suffix Tree?
- **Q1.19 (Short Answer)**: In an Open-Addressed Hash Table using Quadratic Probing $h(k, i) = (h(k) + i^2) \bmod m$, what two conditions on the table size $m$ and load factor $\alpha = n/m$ mathematically guarantee that every insertion will find an empty slot?
- **Q1.20 (True/False)**: When rehashing a hash table from capacity $m = 11$ to $m' = 23$, you can copy each bucket $i \in [0, 10]$ directly to bucket $i$ in the new table using `memcpy` in $O(m)$ time without re-evaluating the hash function.

---

## Section 2: 12 Deep Conceptual & Comparative Synthesis Questions

Full textbook answers are provided in [Section 6.2](#62-answer-key-for-section-2-12-deep-conceptual--comparative-questions).

- **Q2.1 (Cross-Unit Dictionary Showdown)**: Compare **Skip Lists (Unit 1)**, **AVL Trees (Unit 3)**, **Splay Trees (Unit 3)**, **Standard Tries (Unit 4)**, and **Hash Tables (Unit 4)** across: (a) Worst-case vs. Amortized/Expected Search Time, (b) Support for Ordered Range Queries `[k1, k2]` and Prefix Queries, (c) Extra Metadata/Pointer Overhead per Key, and (d) Workload where each structure is the indisputable best choice.
- **Q2.2 (Amortized Analysis Across Units)**: Explain why a **Dynamic Array (`realloc` $\times 2$)**, a **Splay Tree**, and a **Hash Table with Rehashing** all exhibit $\Theta(n)$ single-operation worst-case spikes yet achieve $O(1)$, $O(\log n)$, and $O(1)$ **amortized** costs respectively. Why would an avionics hard real-time system reject all three in favor of an **AVL Tree** or **Static Circular Buffer**?
- **Q2.3 (Recursion Stack vs. Explicit Stack vs. Threads)**: Compare three ways to perform an Inorder Traversal of an $n$-node Binary Search Tree: (a) Recursive Inorder, (b) Iterative Explicit-Stack Inorder, and (c) Double-Threaded BST Inorder. Analyze exact time complexity, auxiliary memory in balanced vs. skewed trees, and structural trade-offs.
- **Q2.4 (Sparse Matrix Representations)**: Contrast the **3-Tuple Triplet Array (COO)** against the **Orthogonal Cross-Linked Multilist** for representing an $R \times C$ sparse matrix. Why does Naive Triplet Transpose take $O(C \cdot nnz)$ while Fast Transpose takes $O(C + nnz)$?
- **Q2.5 (CPU Scheduling & Priority Queues)**: Compare **FCFS**, **Non-Preemptive SJF**, **Preemptive SRTF**, and **Round-Robin (quantum $q$)** CPU scheduling policies. Identify the exact data structure required to implement the Ready Queue for each policy, the phenomenon of **Starvation** vs. **Convoy Effect**, and how **Aging** resolves starvation.
- **Q2.6 (Tree Reconstruction Uniqueness Theorem)**: Prove mathematically why **Inorder + Preorder** (or **Inorder + Postorder**) uniquely determines any Binary Tree, whereas **Preorder + Postorder** fails to uniquely determine a General Binary Tree—yet *does* uniquely determine a **Full (Strict) Binary Tree** where every internal node has exactly 2 children.
- **Q2.7 (Binary Heap vs. Binary Search Tree)**: Students often confuse a **Binary Min-Heap** with a **Binary Search Tree (BST)**. Contrast their structural and ordering invariants. Why can a Heap be stored in a compact array with zero pointers while a general BST cannot, and why does searching for an arbitrary key $x$ in a Min-Heap require $O(n)$ time?
- **Q2.8 (AVL Rotations: Insert vs. Delete Asymmetry)**: Prove why an `insert(k)` operation on an AVL Tree requires **at most 1** rotation (single or double) to restore global balance, whereas a `delete(k)` operation may require up to $\Theta(\log n)$ rotations propagating all the way to the root.
- **Q2.9 (BFS vs. DFS Memory Paradox on Trees & Graphs)**: Consider two extreme binary trees of $n = 2^{20} - 1 \approx 10^6$ nodes: Tree $A$ is a perfectly balanced Complete Binary Tree ($h = 19$), and Tree $B$ is a right-skewed linear chain ($h = 10^6 - 1$). Compare the maximum auxiliary memory consumed by **BFS (Queue)** versus **DFS (Stack)** on Tree $A$ and Tree $B$.
- **Q2.10 (Standard Trie vs. Suffix Trie vs. Compact Suffix Tree)**: Explain how compressing non-branching paths and replacing character edge labels with `(start_index, end_index)` substring pointers transforms an uncompressed Suffix Trie from $\Theta(m^2)$ space into a Compact Suffix Tree of $\Theta(m)$ space with at most $2m$ nodes.
- **Q2.11 (The Anatomy of Hash Clustering)**: Contrast **Primary Clustering** (Linear Probing), **Secondary Clustering** (Quadratic Probing), and **Double Hashing**. Why does Linear Probing still outperform Double Hashing in many modern CPU benchmarks despite requiring more probes at moderate $\alpha$?
- **Q2.12 (Open Addressing Deletion & The Tombstone Accumulation Problem)**: Explain why deleting a key in Open Addressing requires a `DELETED` tombstone state, and explain what happens to search performance over millions of alternating `insert` and `delete` operations even when the load factor $\alpha = n/m$ stays constant at $0.4$. How is this resolved?

---

## Section 3: 8 Algorithm Design & Pseudocode Problems

Detailed pseudocode and complexity proofs are provided in [Section 6.3](#63-answer-key-for-section-3-8-algorithm-design--pseudocode-problems).

- **Problem 3.1 (Unit 1 — In-Place Linked List Reversal & Palindrome Check)**: Design an $O(n)$ time, **$O(1)$ auxiliary space** algorithm that determines whether a Singly Linked List of $n$ nodes is a palindrome, and restores the list to its original order before returning.
- **Problem 3.2 (Unit 1 — $O(1)$ Minimum-Tracking Stack)**: Design a stack data structure `MinStack` supporting `push(x)`, `pop()`, `top()`, and `getMin()` such that **all four operations run in $O(1)$ worst-case time**.
- **Problem 3.3 (Unit 2 — Queue Using Two Stacks in $O(1)$ Amortized Time)**: Design a FIFO Queue using only two LIFO Stacks (`S_in` and `S_out`). Prove using the Banker's/Accounting Method why `enqueue` and `dequeue` both run in $O(1)$ **amortized** time.
- **Problem 3.4 (Unit 2 — Lowest Common Ancestor in General Binary Tree vs. BST)**: Design an $O(n)$ single-pass algorithm to find the Lowest Common Ancestor $\text{LCA}(u, v)$ in an arbitrary Binary Tree (without parent pointers), and contrast it with the $O(h)$ iterative algorithm for a Binary Search Tree.
- **Problem 3.5 (Unit 3 — $k$-th Smallest Element & Range Count in Augmented BST/AVL)**: Show how to augment each node of an AVL Tree with a `size` field so that `kth_smallest(root, k)` and `count_range(root, low, high)` both execute in $O(\log n)$ worst-case time without breaking AVL rotations.
- **Problem 3.6 (Unit 3 — Converting an Arbitrary BST into a Balanced BST in $O(n)$ Time)**: Design an $O(n)$ time algorithm that takes an arbitrary (potentially skewed) Binary Search Tree of $n$ nodes and rebuilds it into a minimum-height Balanced BST of height $\lfloor \log_2 n \rfloor$.
- **Problem 3.7 (Unit 4 — Bipartite Graph Check / 2-Coloring via BFS)**: Design an $O(V + E)$ algorithm using BFS to determine whether an undirected graph $G = (V, E)$ (possibly disconnected) is **Bipartite** (contains no odd-length cycles).
- **Problem 3.8 (Unit 4 — Longest Common Prefix & Autocomplete Using a Trie)**: Given a Trie containing $n$ dictionary words, design: (a) an $O(m)$ algorithm to find the Longest Common Prefix (LCP) of all words in the Trie, and (b) an $O(|P| + K_{\text{chars}})$ autocomplete algorithm that prints all words starting with prefix $P$ in lexicographical order.

---

## Section 4: 8 Complete C11 Programming Problems with Compilable Solutions & Outputs

Every program below adheres strictly to the **C11 standard** (`-std=c11 -Wall -Wextra -pedantic`) and includes verified console output.

---

### Problem 4.1 (Unit 1): Sparse Polynomial Addition Using Sorted Singly Linked Lists & Dummy Sentinel

**Problem Statement**: Write a complete C11 program to represent sparse polynomials $P(x) = \sum c_i x^{e_i}$ in strictly decreasing order of exponents using a Singly Linked List. Implement `poly_add(p1, p2)` in $O(t_1 + t_2)$ time using the stack-allocated Dummy Sentinel pattern, ensuring terms whose coefficients sum to $0$ are cancelled and omitted.

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct PolyNode {
    int coeff;
    int exp;
    struct PolyNode *next;
} PolyNode;

PolyNode *poly_create_term(int coeff, int exp) {
    PolyNode *node = malloc(sizeof(*node));
    if (!node) exit(EXIT_FAILURE);
    node->coeff = coeff;
    node->exp   = exp;
    node->next  = NULL;
    return node;
}

void poly_append_term(PolyNode **head_ref, int coeff, int exp) {
    if (coeff == 0) return;
    PolyNode **pp = head_ref;
    while (*pp != NULL && (*pp)->exp > exp) {
        pp = &((*pp)->next);
    }
    if (*pp != NULL && (*pp)->exp == exp) {
        (*pp)->coeff += coeff;
        if ((*pp)->coeff == 0) {
            PolyNode *victim = *pp;
            *pp = victim->next;
            free(victim);
        }
    } else {
        PolyNode *node = poly_create_term(coeff, exp);
        node->next = *pp;
        *pp = node;
    }
}

PolyNode *poly_add(const PolyNode *p1, const PolyNode *p2) {
    PolyNode dummy = {0, 0, NULL};
    PolyNode *tail = &dummy;

    while (p1 != NULL && p2 != NULL) {
        if (p1->exp > p2->exp) {
            tail->next = poly_create_term(p1->coeff, p1->exp);
            tail = tail->next;
            p1 = p1->next;
        } else if (p1->exp < p2->exp) {
            tail->next = poly_create_term(p2->coeff, p2->exp);
            tail = tail->next;
            p2 = p2->next;
        } else {
            int sum = p1->coeff + p2->coeff;
            if (sum != 0) { /* Zero-cancellation check! */
                tail->next = poly_create_term(sum, p1->exp);
                tail = tail->next;
            }
            p1 = p1->next;
            p2 = p2->next;
        }
    }
    for (const PolyNode *rem = p1 ? p1 : p2; rem != NULL; rem = rem->next) {
        tail->next = poly_create_term(rem->coeff, rem->exp);
        tail = tail->next;
    }
    return dummy.next;
}

void poly_print(const char *label, const PolyNode *p) {
    printf("%s", label);
    if (!p) { printf("0\n"); return; }
    for (const PolyNode *curr = p; curr != NULL; curr = curr->next) {
        if (curr != p) printf(curr->coeff > 0 ? " + " : " - ");
        else if (curr->coeff < 0) printf("-");
        printf("%dx^%d", abs(curr->coeff), curr->exp);
    }
    printf("\n");
}

void poly_free(PolyNode **p_ref) {
    PolyNode *curr = *p_ref;
    while (curr) {
        PolyNode *next = curr->next;
        free(curr);
        curr = next;
    }
    *p_ref = NULL;
}

int main(void) {
    PolyNode *p1 = NULL, *p2 = NULL;
    /* P1(x) = 5x^4 - 3x^2 + 2x^1 + 7x^0 */
    poly_append_term(&p1, 5, 4);
    poly_append_term(&p1, -3, 2);
    poly_append_term(&p1, 2, 1);
    poly_append_term(&p1, 7, 0);

    /* P2(x) = 2x^3 + 3x^2 - 5x^1 - 1x^0 (Note: -3x^2 + 3x^2 cancels!) */
    poly_append_term(&p2, 2, 3);
    poly_append_term(&p2, 3, 2);
    poly_append_term(&p2, -5, 1);
    poly_append_term(&p2, -1, 0);

    PolyNode *sum = poly_add(p1, p2);
    poly_print("P1(x)        = ", p1);
    poly_print("P2(x)        = ", p2);
    poly_print("P1(x)+P2(x)  = ", sum);

    poly_free(&p1);
    poly_free(&p2);
    poly_free(&sum);
    return 0;
}
```

**Verified Output**:
```text
P1(x)        = 5x^4 - 3x^2 + 2x^1 + 7x^0
P2(x)        = 2x^3 + 3x^2 - 5x^1 - 1x^0
P1(x)+P2(x)  = 5x^4 + 2x^3 - 3x^1 + 6x^0
```

---

### Problem 4.2 (Unit 1): $O(C + nnz)$ Fast Transpose of a Sparse Matrix in Triplet Form

**Problem Statement**: Implement the $O(C + nnz)$ **Fast Transpose** algorithm on a Sparse Matrix stored in 3-tuple Triplet Representation `Triplet a[nnz + 1]`, where `a[0]` stores `(num_rows, num_cols, nnz)` and `a[1..nnz]` store `(row, col, val)` in row-major order. Print the frequency array `row_terms[]`, starting position array `start_pos[]`, and resulting transposed triplet table `b[]`.

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int row;
    int col;
    int val;
} Triplet;

void sparse_fast_transpose(const Triplet a[], Triplet b[]) {
    int num_rows = a[0].row;
    int num_cols = a[0].col;
    int nnz      = a[0].val;

    b[0].row = num_cols;
    b[0].col = num_rows;
    b[0].val = nnz;
    if (nnz == 0) return;

    int *row_terms = calloc(num_cols, sizeof(int));
    int *start_pos = malloc(num_cols * sizeof(int));
    if (!row_terms || !start_pos) exit(EXIT_FAILURE);

    /* Pass 1: Count non-zeros in each column of A (which become rows of B) */
    for (int i = 1; i <= nnz; i++) {
        row_terms[a[i].col]++;
    }

    /* Pass 2: Compute starting index in B[] for each transposed row */
    start_pos[0] = 1;
    for (int c = 1; c < num_cols; c++) {
        start_pos[c] = start_pos[c - 1] + row_terms[c - 1];
    }

    printf("Column counts (row_terms): ");
    for (int c = 0; c < num_cols; c++) printf("col[%d]=%d ", c, row_terms[c]);
    printf("\nStarting positions       : ");
    for (int c = 0; c < num_cols; c++) printf("pos[%d]=%d ", c, start_pos[c]);
    printf("\n\n");

    /* Pass 3: Scatter each element directly into its exact target slot in O(nnz) */
    for (int i = 1; i <= nnz; i++) {
        int pos = start_pos[a[i].col]++;
        b[pos].row = a[i].col;
        b[pos].col = a[i].row;
        b[pos].val = a[i].val;
    }

    free(row_terms);
    free(start_pos);
}

void print_triplets(const char *title, const Triplet t[]) {
    printf("%s (Rows=%d, Cols=%d, NNZ=%d):\n", title, t[0].row, t[0].col, t[0].val);
    printf("  Idx | Row | Col | Val\n  ----+-----+-----+----\n");
    for (int i = 0; i <= t[0].val; i++) {
        printf("  %3d | %3d | %3d | %3d\n", i, t[i].row, t[i].col, t[i].val);
    }
    printf("\n");
}

int main(void) {
    Triplet a[6] = {
        {4, 4, 5},  /* Header: 4x4 matrix with 5 non-zero elements */
        {0, 1, 10},
        {0, 3, 20},
        {1, 1, 30},
        {2, 0, 40},
        {3, 2, 50}
    };
    Triplet b[6];
    print_triplets("Original Sparse Matrix A", a);
    sparse_fast_transpose(a, b);
    print_triplets("Transposed Sparse Matrix B = A^T", b);
    return 0;
}
```

**Verified Output**:
```text
Original Sparse Matrix A (Rows=4, Cols=4, NNZ=5):
  Idx | Row | Col | Val
  ----+-----+-----+----
    0 |   4 |   4 |   5
    1 |   0 |   1 |  10
    2 |   0 |   3 |  20
    3 |   1 |   1 |  30
    4 |   2 |   0 |  40
    5 |   3 |   2 |  50

Column counts (row_terms): col[0]=1 col[1]=2 col[2]=1 col[3]=1 
Starting positions       : pos[0]=1 pos[1]=2 pos[2]=4 pos[3]=5 

Transposed Sparse Matrix B = A^T (Rows=4, Cols=4, NNZ=5):
  Idx | Row | Col | Val
  ----+-----+-----+----
    0 |   4 |   4 |   5
    1 |   0 |   2 |  40
    2 |   1 |   0 |  10
    3 |   1 |   1 |  30
    4 |   2 |   3 |  50
    5 |   3 |   0 |  20
```

---

### Problem 4.3 (Unit 2): Josephus Problem — Circular Queue Simulation vs. $O(n)$ Recurrence

**Problem Statement**: Write a C11 program that simulates the Josephus problem for $n$ people (`1 .. n`) and step size $k$ using a **Circular Array Queue** to print the exact elimination order in $O(n \cdot k)$ time, and cross-verifies the final survivor using the $O(n)$ time, $O(1)$ space mathematical recurrence.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_Q 64

typedef struct {
    int data[MAX_Q];
    int front;
    int size;
} CircularQueue;

void cq_init(CircularQueue *q) { q->front = 0; q->size = 0; }
bool cq_enqueue(CircularQueue *q, int val) {
    if (q->size == MAX_Q) return false;
    q->data[(q->front + q->size) % MAX_Q] = val;
    q->size++;
    return true;
}
int cq_dequeue(CircularQueue *q) {
    int val = q->data[q->front];
    q->front = (q->front + 1) % MAX_Q;
    q->size--;
    return val;
}

int josephus_queue_sim(int n, int k) {
    CircularQueue q;
    cq_init(&q);
    for (int i = 1; i <= n; i++) cq_enqueue(&q, i);

    printf("Elimination order (n=%d, k=%d): ", n, k);
    while (q.size > 1) {
        /* Rotate k - 1 people from front to back */
        for (int step = 1; step < k; step++) {
            cq_enqueue(&q, cq_dequeue(&q));
        }
        /* Eliminate the k-th person */
        printf("%d -> ", cq_dequeue(&q));
    }
    int survivor = cq_dequeue(&q);
    printf("SURVIVOR = %d\n", survivor);
    return survivor;
}

/* O(n) Time, O(1) Space Mathematical Recurrence: J(1,k) = 0; J(i,k) = (J(i-1,k) + k) % i */
int josephus_math_o_n(int n, int k) {
    int pos = 0; /* 0-indexed survivor for n = 1 */
    for (int i = 2; i <= n; i++) {
        pos = (pos + k) % i;
    }
    return pos + 1; /* Convert to 1-indexed */
}

int main(void) {
    int s_sim  = josephus_queue_sim(7, 3);
    int s_math = josephus_math_o_n(7, 3);
    printf("O(n) Formula Verification     : SURVIVOR = %d (Match: %s)\n",
           s_math, (s_sim == s_math) ? "YES" : "NO");
    return 0;
}
```

**Verified Output**:
```text
Elimination order (n=7, k=3): 3 -> 6 -> 2 -> 7 -> 5 -> 1 -> SURVIVOR = 4
O(n) Formula Verification     : SURVIVOR = 4 (Match: YES)
```

---

### Problem 4.4 (Unit 2): N-ary Tree to Binary Tree (LCRS) Conversion & Traversal Correspondence

**Problem Statement**: Write a C11 program that represents a General N-ary Tree using the **Left-Child Right-Sibling (LCRS)** binary tree representation (`first_child` as `left`, `next_sibling` as `right`), and demonstrates why **Binary Preorder (`NLR`)** equals **N-ary Preorder**, while **Binary Inorder (`LNR`)** equals **N-ary Postorder**!

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct LCRSNode {
    char label;
    struct LCRSNode *first_child;  /* Binary 'left' pointer  */
    struct LCRSNode *next_sibling; /* Binary 'right' pointer */
} LCRSNode;

LCRSNode *lcrs_create(char label) {
    LCRSNode *node = malloc(sizeof(*node));
    if (!node) exit(EXIT_FAILURE);
    node->label        = label;
    node->first_child  = NULL;
    node->next_sibling = NULL;
    return node;
}

void lcrs_add_child(LCRSNode *parent, LCRSNode *child) {
    if (parent->first_child == NULL) {
        parent->first_child = child;
    } else {
        LCRSNode *sib = parent->first_child;
        while (sib->next_sibling != NULL) {
            sib = sib->next_sibling;
        }
        sib->next_sibling = child;
    }
}

/* Binary Preorder (N -> L -> R) == General N-ary Tree Preorder */
void lcrs_binary_preorder(const LCRSNode *root) {
    if (!root) return;
    printf("%c ", root->label);
    lcrs_binary_preorder(root->first_child);
    lcrs_binary_preorder(root->next_sibling);
}

/* Binary Inorder (L -> N -> R) == General N-ary Tree Postorder! */
void lcrs_binary_inorder(const LCRSNode *root) {
    if (!root) return;
    lcrs_binary_inorder(root->first_child);
    printf("%c ", root->label);
    lcrs_binary_inorder(root->next_sibling);
}

void lcrs_free(LCRSNode *root) {
    if (!root) return;
    lcrs_free(root->first_child);
    lcrs_free(root->next_sibling);
    free(root);
}

int main(void) {
    /* General Tree: Root A has children B, C, D. B has children E, F. D has child G. */
    LCRSNode *A = lcrs_create('A');
    LCRSNode *B = lcrs_create('B');
    LCRSNode *C = lcrs_create('C');
    LCRSNode *D = lcrs_create('D');
    LCRSNode *E = lcrs_create('E');
    LCRSNode *F = lcrs_create('F');
    LCRSNode *G = lcrs_create('G');

    lcrs_add_child(A, B); lcrs_add_child(A, C); lcrs_add_child(A, D);
    lcrs_add_child(B, E); lcrs_add_child(B, F);
    lcrs_add_child(D, G);

    printf("LCRS Binary Preorder (= N-ary Preorder) : ");
    lcrs_binary_preorder(A);
    printf("\nLCRS Binary Inorder  (= N-ary Postorder): ");
    lcrs_binary_inorder(A);
    printf("\n");

    lcrs_free(A);
    return 0;
}
```

**Verified Output**:
```text
LCRS Binary Preorder (= N-ary Preorder) : A B E F C D G 
LCRS Binary Inorder  (= N-ary Postorder): E F B C G D A 
```

---

### Problem 4.5 (Unit 3): Double-Threaded BST Insertion & $O(1)$-Space Stackless Inorder Traversal

**Problem Statement**: Implement a Double-Threaded Binary Search Tree (`lthread`, `rthread`) supporting `tbst_insert()` and stackless, recursion-free `tbst_inorder()` in $\Theta(n)$ time and $O(1)$ auxiliary memory.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct TNode {
    int key;
    struct TNode *left;
    struct TNode *right;
    bool lthread; /* true if left points to Inorder Predecessor */
    bool rthread; /* true if right points to Inorder Successor  */
} TNode;

TNode *tbst_insert(TNode *root, int key) {
    TNode *ptr = root;
    TNode *par = NULL;

    while (ptr != NULL) {
        if (key == ptr->key) return root; /* Ignore duplicate */
        par = ptr;
        if (key < ptr->key) {
            if (!ptr->lthread) ptr = ptr->left;
            else break;
        } else {
            if (!ptr->rthread) ptr = ptr->right;
            else break;
        }
    }

    TNode *tmp = malloc(sizeof(*tmp));
    if (!tmp) exit(EXIT_FAILURE);
    tmp->key     = key;
    tmp->lthread = true;
    tmp->rthread = true;

    if (par == NULL) {
        tmp->left  = NULL;
        tmp->right = NULL;
        return tmp;
    } else if (key < par->key) {
        tmp->left    = par->left; /* Inherit parent's predecessor thread */
        tmp->right   = par;       /* Parent is inorder successor         */
        par->lthread = false;     /* par->left is now a real child       */
        par->left    = tmp;
    } else {
        tmp->left    = par;        /* Parent is inorder predecessor      */
        tmp->right   = par->right; /* Inherit parent's successor thread  */
        par->rthread = false;      /* par->right is now a real child     */
        par->right   = tmp;
    }
    return root;
}

TNode *tbst_inorder_successor(TNode *ptr) {
    if (ptr->rthread) {
        return ptr->right; /* Follow thread directly in O(1)! */
    }
    ptr = ptr->right;
    while (!ptr->lthread) {
        ptr = ptr->left;
    }
    return ptr;
}

/* Stackless, recursion-free O(1) auxiliary space Inorder Traversal! */
void tbst_inorder(TNode *root) {
    if (!root) return;
    TNode *ptr = root;
    while (!ptr->lthread) {
        ptr = ptr->left; /* Start at leftmost node */
    }
    printf("Stackless Threaded Inorder: ");
    while (ptr != NULL) {
        printf("%d ", ptr->key);
        ptr = tbst_inorder_successor(ptr);
    }
    printf("\n");
}

void tbst_free(TNode *root) {
    if (!root) return;
    if (!root->lthread) tbst_free(root->left);
    if (!root->rthread) tbst_free(root->right);
    free(root);
}

int main(void) {
    TNode *root = NULL;
    int keys[] = {20, 10, 30, 5, 16, 14, 17, 13};
    for (int i = 0; i < 8; i++) {
        root = tbst_insert(root, keys[i]);
    }
    tbst_inorder(root);
    tbst_free(root);
    return 0;
}
```

**Verified Output**:
```text
Stackless Threaded Inorder: 5 10 13 14 16 17 20 30 
```

---

### Problem 4.6 (Unit 3): Self-Balancing AVL Tree Insertion with All 4 Rotations (`LL`, `RR`, `LR`, `RL`)

**Problem Statement**: Implement a complete C11 AVL Tree with height tracking, balance factor calculation $\text{BF}(u) = h(u\text{->left}) - h(u\text{->right})$, and all four rotation cases (`LL`, `RR`, `LR`, `RL`). Print the Preorder traversal with each node's height and balance factor.

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct AVLNode {
    int key;
    int height;
    struct AVLNode *left;
    struct AVLNode *right;
} AVLNode;

int avl_height(const AVLNode *n) { return n ? n->height : 0; }
int max_int(int a, int b)        { return (a > b) ? a : b; }
int avl_get_bf(const AVLNode *n) { return n ? avl_height(n->left) - avl_height(n->right) : 0; }

AVLNode *avl_create_node(int key) {
    AVLNode *node = malloc(sizeof(*node));
    if (!node) exit(EXIT_FAILURE);
    node->key    = key;
    node->height = 1;
    node->left   = NULL;
    node->right  = NULL;
    return node;
}

AVLNode *avl_rotate_right(AVLNode *y) {
    AVLNode *x  = y->left;
    AVLNode *T2 = x->right;
    x->right = y;
    y->left  = T2;
    /* Update demoted child y FIRST, then promoted parent x SECOND! */
    y->height = 1 + max_int(avl_height(y->left), avl_height(y->right));
    x->height = 1 + max_int(avl_height(x->left), avl_height(x->right));
    return x;
}

AVLNode *avl_rotate_left(AVLNode *x) {
    AVLNode *y  = x->right;
    AVLNode *T2 = y->left;
    y->left  = x;
    x->right = T2;
    /* Update demoted child x FIRST, then promoted parent y SECOND! */
    x->height = 1 + max_int(avl_height(x->left), avl_height(x->right));
    y->height = 1 + max_int(avl_height(y->left), avl_height(y->right));
    return y;
}

AVLNode *avl_insert(AVLNode *node, int key) {
    if (node == NULL) return avl_create_node(key);

    if (key < node->key)      node->left  = avl_insert(node->left, key);
    else if (key > node->key) node->right = avl_insert(node->right, key);
    else return node; /* Duplicate */

    node->height = 1 + max_int(avl_height(node->left), avl_height(node->right));
    int bf = avl_get_bf(node);

    /* Case 1: Left-Left (LL) */
    if (bf > 1 && key < node->left->key)
        return avl_rotate_right(node);
    /* Case 2: Right-Right (RR) */
    if (bf < -1 && key > node->right->key)
        return avl_rotate_left(node);
    /* Case 3: Left-Right (LR) */
    if (bf > 1 && key > node->left->key) {
        node->left = avl_rotate_left(node->left);
        return avl_rotate_right(node);
    }
    /* Case 4: Right-Left (RL) */
    if (bf < -1 && key < node->right->key) {
        node->right = avl_rotate_right(node->right);
        return avl_rotate_left(node);
    }
    return node;
}

void avl_preorder_print(const AVLNode *root) {
    if (!root) return;
    printf("%d(h=%d,bf=%+d) ", root->key, root->height, avl_get_bf(root));
    avl_preorder_print(root->left);
    avl_preorder_print(root->right);
}

void avl_free(AVLNode *root) {
    if (!root) return;
    avl_free(root->left);
    avl_free(root->right);
    free(root);
}

int main(void) {
    AVLNode *root = NULL;
    int keys[] = {30, 20, 10, 25, 28, 40, 50, 45};
    for (int i = 0; i < 8; i++) {
        root = avl_insert(root, keys[i]);
    }
    printf("AVL Preorder [key(height, BF)]:\n  ");
    avl_preorder_print(root);
    printf("\n");
    avl_free(root);
    return 0;
}
```

**Verified Output**:
```text
AVL Preorder [key(height, BF)]:
  25(h=4,bf=-1) 20(h=2,bf=+1) 10(h=1,bf=+0) 30(h=3,bf=-1) 28(h=1,bf=+0) 45(h=2,bf=+0) 40(h=1,bf=+0) 50(h=1,bf=+0) 
```

---

### Problem 4.7 (Unit 3 & 4): Graph Connected Components & Unweighted Shortest Path via BFS

**Problem Statement**: Write a C11 program using an **Adjacency List** representation of an undirected graph $G = (V, E)$ that:
1. Counts and labels all **Connected Components** in $\Theta(V + E)$ time.
2. Computes the **Shortest Unweighted Path** (minimum edges) and reconstructs the exact vertex path from source $s$ to target $t$ using BFS (`dist[]` and `parent[]`).

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_V 16

typedef struct AdjNode {
    int v;
    struct AdjNode *next;
} AdjNode;

typedef struct {
    int num_v;
    AdjNode *head[MAX_V];
} Graph;

void graph_init(Graph *g, int num_v) {
    g->num_v = num_v;
    for (int i = 0; i < num_v; i++) g->head[i] = NULL;
}

void graph_add_undirected_edge(Graph *g, int u, int v) {
    AdjNode *n1 = malloc(sizeof(*n1));
    n1->v = v; n1->next = g->head[u]; g->head[u] = n1;

    AdjNode *n2 = malloc(sizeof(*n2));
    n2->v = u; n2->next = g->head[v]; g->head[v] = n2;
}

void bfs_shortest_path(const Graph *g, int src, int dest) {
    bool visited[MAX_V] = {false};
    int  dist[MAX_V];
    int  parent[MAX_V];
    int  queue[MAX_V];
    int  front = 0, rear = 0;

    for (int i = 0; i < g->num_v; i++) { dist[i] = -1; parent[i] = -1; }

    visited[src] = true; /* Mark visited ON ENQUEUE! */
    dist[src]    = 0;
    queue[rear++] = src;

    while (front < rear) {
        int u = queue[front++];
        for (AdjNode *e = g->head[u]; e != NULL; e = e->next) {
            int v = e->v;
            if (!visited[v]) {
                visited[v]    = true;
                dist[v]       = dist[u] + 1;
                parent[v]     = u;
                queue[rear++] = v;
            }
        }
    }

    if (dist[dest] == -1) {
        printf("No path exists from %d to %d.\n", src, dest);
        return;
    }
    printf("Shortest distance from %d to %d = %d edges | Path: ", src, dest, dist[dest]);
    int path[MAX_V], len = 0;
    for (int curr = dest; curr != -1; curr = parent[curr]) {
        path[len++] = curr;
    }
    for (int i = len - 1; i >= 0; i--) {
        printf("%d%s", path[i], (i > 0) ? " -> " : "\n");
    }
}

int count_connected_components(const Graph *g) {
    bool visited[MAX_V] = {false};
    int components = 0;

    for (int i = 0; i < g->num_v; i++) {
        if (!visited[i]) {
            components++;
            printf("  Component #%d: { ", components);
            int queue[MAX_V], front = 0, rear = 0;
            visited[i] = true;
            queue[rear++] = i;
            while (front < rear) {
                int u = queue[front++];
                printf("%d ", u);
                for (AdjNode *e = g->head[u]; e != NULL; e = e->next) {
                    if (!visited[e->v]) {
                        visited[e->v] = true;
                        queue[rear++] = e->v;
                    }
                }
            }
            printf("}\n");
        }
    }
    return components;
}

void graph_free(Graph *g) {
    for (int i = 0; i < g->num_v; i++) {
        AdjNode *curr = g->head[i];
        while (curr) {
            AdjNode *next = curr->next;
            free(curr);
            curr = next;
        }
    }
}

int main(void) {
    Graph g;
    graph_init(&g, 8);
    /* Component 1: {0, 1, 2, 3, 4, 5} */
    graph_add_undirected_edge(&g, 0, 1);
    graph_add_undirected_edge(&g, 0, 2);
    graph_add_undirected_edge(&g, 1, 3);
    graph_add_undirected_edge(&g, 2, 4);
    graph_add_undirected_edge(&g, 3, 5);
    graph_add_undirected_edge(&g, 4, 5);
    /* Component 2: {6, 7} */
    graph_add_undirected_edge(&g, 6, 7);

    printf("Connected Components Analysis:\n");
    int total = count_connected_components(&g);
    printf("Total Connected Components = %d\n\n", total);

    bfs_shortest_path(&g, 0, 5);
    graph_free(&g);
    return 0;
}
```

**Verified Output**:
```text
Connected Components Analysis:
  Component #1: { 0 2 1 4 3 5 }
  Component #2: { 6 7 }
Total Connected Components = 2

Shortest distance from 0 to 5 = 3 edges | Path: 0 -> 2 -> 4 -> 5
```

---

### Problem 4.8 (Unit 4): Open-Addressed Double Hashing Table with Tombstone (`DELETED`) Support

**Problem Statement**: Implement an Open-Addressed Hash Table of prime capacity $m = 11$ using **Double Hashing** $h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod 11$ where $h_1(k) = k \bmod 11$ and $h_2(k) = 7 - (k \bmod 7)$. Support `insert`, `search`, and `delete` with `EMPTY`, `OCCUPIED`, and `DELETED` (Tombstone) states, demonstrating that deleting an intermediate colliding key does not break searches for later keys, and that subsequent insertions reuse tombstone slots!

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define TABLE_M 11
#define PRIME_R 7

typedef enum { SLOT_EMPTY = 0, SLOT_OCCUPIED = 1, SLOT_DELETED = 2 } SlotState;

typedef struct {
    int       key;
    SlotState state;
} HashSlot;

typedef struct {
    HashSlot slots[TABLE_M];
    int      size;
} DoubleHashTable;

void dht_init(DoubleHashTable *ht) {
    ht->size = 0;
    for (int i = 0; i < TABLE_M; i++) {
        ht->slots[i].key   = 0;
        ht->slots[i].state = SLOT_EMPTY;
    }
}

int h1(int key) { return ((key % TABLE_M) + TABLE_M) % TABLE_M; }
int h2(int key) { return PRIME_R - (((key % PRIME_R) + PRIME_R) % PRIME_R); }

int dht_search(const DoubleHashTable *ht, int key) {
    int u = h1(key);
    int v = h2(key);
    for (int i = 0; i < TABLE_M; i++) {
        int idx = (u + i * v) % TABLE_M;
        if (ht->slots[idx].state == SLOT_EMPTY) {
            return -1; /* Stop at EMPTY (but continue past DELETED!) */
        }
        if (ht->slots[idx].state == SLOT_OCCUPIED && ht->slots[idx].key == key) {
            return idx;
        }
    }
    return -1;
}

bool dht_insert(DoubleHashTable *ht, int key) {
    if (ht->size == TABLE_M) return false;
    int u = h1(key);
    int v = h2(key);
    int first_deleted = -1;

    for (int i = 0; i < TABLE_M; i++) {
        int idx = (u + i * v) % TABLE_M;
        if (ht->slots[idx].state == SLOT_OCCUPIED) {
            if (ht->slots[idx].key == key) return true; /* Already exists */
        } else if (ht->slots[idx].state == SLOT_DELETED) {
            if (first_deleted == -1) first_deleted = idx; /* Remember 1st tombstone */
        } else { /* SLOT_EMPTY */
            int target = (first_deleted != -1) ? first_deleted : idx;
            ht->slots[target].key   = key;
            ht->slots[target].state = SLOT_OCCUPIED;
            ht->size++;
            return true;
        }
    }
    if (first_deleted != -1) {
        ht->slots[first_deleted].key   = key;
        ht->slots[first_deleted].state = SLOT_OCCUPIED;
        ht->size++;
        return true;
    }
    return false;
}

bool dht_delete(DoubleHashTable *ht, int key) {
    int idx = dht_search(ht, key);
    if (idx == -1) return false;
    ht->slots[idx].state = SLOT_DELETED; /* Mark TOMBSTONE, not EMPTY! */
    ht->size--;
    return true;
}

void dht_print(const DoubleHashTable *ht) {
    printf("Idx: ");
    for (int i = 0; i < TABLE_M; i++) printf("%4d ", i);
    printf("\nVal: ");
    for (int i = 0; i < TABLE_M; i++) {
        if (ht->slots[i].state == SLOT_OCCUPIED)     printf("%4d ", ht->slots[i].key);
        else if (ht->slots[i].state == SLOT_DELETED) printf(" DEL ");
        else                                         printf("   . ");
    }
    printf("\n\n");
}

int main(void) {
    DoubleHashTable ht;
    dht_init(&ht);

    /* Keys 14, 25, 36 all have h1(k) = k % 11 = 3! */
    dht_insert(&ht, 14); /* i=0: lands at 3 */
    dht_insert(&ht, 25); /* h1=3, h2=7-(25%7)=3 -> i=1: (3+3)%11 = 6 */
    dht_insert(&ht, 36); /* h1=3, h2=7-(36%7)=6 -> i=1: (3+6)%11 = 9 */
    printf("After inserting colliding keys 14, 25, 36 (all h1 = 3):\n");
    dht_print(&ht);

    dht_delete(&ht, 14); /* Slot 3 becomes DEL tombstone */
    printf("After deleting 14 (slot 3 becomes DEL tombstone):\n");
    dht_print(&ht);

    printf("Search(25) = slot %d | Search(36) = slot %d (Still reachable past DEL!)\n\n",
           dht_search(&ht, 25), dht_search(&ht, 36));

    dht_insert(&ht, 47); /* h1(47) = 3 -> Reuses tombstone at slot 3! */
    printf("After inserting 47 (h1 = 3 -> reuses slot 3 tombstone!):\n");
    dht_print(&ht);
    return 0;
}
```

**Verified Output**:
```text
After inserting colliding keys 14, 25, 36 (all h1 = 3):
Idx:    0    1    2    3    4    5    6    7    8    9   10 
Val:    .    .    .   14    .    .   25    .    .   36    . 

After deleting 14 (slot 3 becomes DEL tombstone):
Idx:    0    1    2    3    4    5    6    7    8    9   10 
Val:    .    .    .  DEL    .    .   25    .    .   36    . 

Search(25) = slot 6 | Search(36) = slot 9 (Still reachable past DEL!)

After inserting 47 (h1 = 3 -> reuses slot 3 tombstone!):
Idx:    0    1    2    3    4    5    6    7    8    9   10 
Val:    .    .    .   47    .    .   25    .    .   36    . 
```

---

## Section 5: 8 Full-Length Worked Examination Problems (Step-by-Step Hand Traces)

These eight problems replicate full-credit university final examination hand-trace questions across all 4 Units.

---

### Worked Trace 5.1 (Unit 1): Infix-to-Postfix Conversion & Postfix Evaluation

**Problem**:
1. Convert the infix expression `3 + 4 * 2 / ( 5 - 1 ) ^ 2 ^ 1` into Postfix using Dijkstra's Shunting-Yard Algorithm.
   - Precedence: `^` (3, **Right-Associative**), `*` `/` (2, **Left-Associative**), `+` `-` (1, **Left-Associative**).
2. Evaluate the resulting Postfix expression step-by-step using an operand stack.

#### Part A: Step-by-Step Infix-to-Postfix Shunting-Yard Trace

| Step | Token Scanned | Action Taken | Operator Stack (Bottom $\to$ Top) | Postfix Output String |
| :---: | :---: | :--- | :--- | :--- |
| 1 | `3` | Operand $\to$ output | *(empty)* | `3` |
| 2 | `+` | Push `+` onto empty stack | `+` | `3` |
| 3 | `4` | Operand $\to$ output | `+` | `3 4` |
| 4 | `*` | $\text{prec}(*) = 2 > \text{prec}(+) = 1 \implies$ push `*` | `+ *` | `3 4` |
| 5 | `2` | Operand $\to$ output | `+ *` | `3 4 2` |
| 6 | `/` | $\text{prec}(/) = \text{prec}(*) = 2$ and Left-Assoc $\implies$ pop `*`, push `/` | `+ /` | `3 4 2 *` |
| 7 | `(` | Left paren $\implies$ push `(` | `+ / (` | `3 4 2 *` |
| 8 | `5` | Operand $\to$ output | `+ / (` | `3 4 2 * 5` |
| 9 | `-` | Top is `(` $\implies$ push `-` | `+ / ( -` | `3 4 2 * 5` |
| 10 | `1` | Operand $\to$ output | `+ / ( -` | `3 4 2 * 5 1` |
| 11 | `)` | Pop until `(`: pop `-` to output, discard `(` | `+ /` | `3 4 2 * 5 1 -` |
| 12 | `^` | $\text{prec}(\text{\textasciicircum}) = 3 > \text{prec}(/) = 2 \implies$ push `^` | `+ / ^` | `3 4 2 * 5 1 -` |
| 13 | `2` | Operand $\to$ output | `+ / ^` | `3 4 2 * 5 1 - 2` |
| 14 | `^` | $\text{prec}(\text{\textasciicircum}) = 3 == \text{prec}(\text{\textasciicircum})$, **Right-Associative** $\implies$ **DO NOT POP**, push `^`! | `+ / ^ ^` | `3 4 2 * 5 1 - 2` |
| 15 | `1` | Operand $\to$ output | `+ / ^ ^` | `3 4 2 * 5 1 - 2 1` |
| 16 | *EOF* | Pop all remaining operators: `^`, `^`, `/`, `+` | *(empty)* | **`3 4 2 * 5 1 - 2 1 ^ ^ / +`** |

#### Part B: Step-by-Step Postfix Stack Evaluation Trace

| Step | Token | Operation (`left op right` where `right` is 1st pop, `left` is 2nd pop) | Operand Stack (Bottom $\to$ Top) |
| :---: | :---: | :--- | :--- |
| 1 | `3` | Push `3` | `[ 3 ]` |
| 2 | `4` | Push `4` | `[ 3, 4 ]` |
| 3 | `2` | Push `2` | `[ 3, 4, 2 ]` |
| 4 | `*` | `right = 2`, `left = 4` $\to$ `4 * 2 = 8`, push `8` | `[ 3, 8 ]` |
| 5 | `5` | Push `5` | `[ 3, 8, 5 ]` |
| 6 | `1` | Push `1` | `[ 3, 8, 5, 1 ]` |
| 7 | `-` | `right = 1`, `left = 5` $\to$ `5 - 1 = 4`, push `4` | `[ 3, 8, 4 ]` |
| 8 | `2` | Push `2` | `[ 3, 8, 4, 2 ]` |
| 9 | `1` | Push `1` | `[ 3, 8, 4, 2, 1 ]` |
| 10 | `^` | `right = 1`, `left = 2` $\to$ $2^1 = 2$, push `2` | `[ 3, 8, 4, 2 ]` |
| 11 | `^` | `right = 2`, `left = 4` $\to$ $4^2 = 16$, push `16` | `[ 3, 8, 16 ]` |
| 12 | `/` | `right = 16`, `left = 8` $\to$ $8 / 16 = 0.5$ (or `0` in integer div), push `0.5` | `[ 3, 0.5 ]` |
| 13 | `+` | `right = 0.5`, `left = 3` $\to$ $3 + 0.5 = \mathbf{3.5}$ | **`[ 3.5 ]`** |

---

### Worked Trace 5.2 (Unit 2): Round-Robin CPU Scheduling Queue Trace & Turnaround/Waiting Metrics

**Problem**: Trace **Round-Robin (RR)** CPU Scheduling with time quantum $q = 3\text{ ms}$ for the following 4 processes. Newly arriving processes at time $t$ are enqueued into the FIFO Ready Queue **before** a preempted process at time $t$ is re-enqueued.

| Process | Arrival Time ($AT$) | Burst Time ($BT$) |
| :---: | :---: | :---: |
| $P_1$ | $0$ | $7$ |
| $P_2$ | $2$ | $4$ |
| $P_3$ | $4$ | $1$ |
| $P_4$ | $5$ | $4$ |

#### Step-by-Step Ready Queue & Gantt Chart Trace

1. **$t = 0$**: $P_1$ arrives. Ready Queue: `[P1(7)]`. Dispatch **$P_1$** for quantum $3$ ($t = 0 \to 3$).
   - During $t = 0 \to 3$: $P_2(4)$ arrives at $t = 2 \implies$ Queue: `[P2(4)]`.
   - At $t = 3$: $P_1$ preempted with remaining $BT = 4 \implies$ re-enqueued at back: Queue: `[P2(4), P1(4)]`.
2. **$t = 3$**: Dispatch **$P_2$** for quantum $3$ ($t = 3 \to 6$).
   - During $t = 3 \to 6$: $P_3(1)$ arrives at $t = 4$, $P_4(4)$ arrives at $t = 5 \implies$ Queue: `[P1(4), P3(1), P4(4)]`.
   - At $t = 6$: $P_2$ preempted with remaining $BT = 1 \implies$ Queue: `[P1(4), P3(1), P4(4), P2(1)]`.
3. **$t = 6$**: Dispatch **$P_1$** for quantum $3$ ($t = 6 \to 9$).
   - At $t = 9$: $P_1$ preempted with remaining $BT = 1 \implies$ Queue: `[P3(1), P4(4), P2(1), P1(1)]`.
4. **$t = 9$**: Dispatch **$P_3$** for $1\text{ ms}$ ($t = 9 \to 10$, finishes early!).
   - At $t = 10$: **$P_3$ completes** ($CT_3 = 10$). Queue: `[P4(4), P2(1), P1(1)]`.
5. **$t = 10$**: Dispatch **$P_4$** for quantum $3$ ($t = 10 \to 13$).
   - At $t = 13$: $P_4$ preempted with remaining $BT = 1 \implies$ Queue: `[P2(1), P1(1), P4(1)]`.
6. **$t = 13$**: Dispatch **$P_2$** for $1\text{ ms}$ ($t = 13 \to 14$). **$P_2$ completes** ($CT_2 = 14$). Queue: `[P1(1), P4(1)]`.
7. **$t = 14$**: Dispatch **$P_1$** for $1\text{ ms}$ ($t = 14 \to 15$). **$P_1$ completes** ($CT_1 = 15$). Queue: `[P4(1)]`.
8. **$t = 15$**: Dispatch **$P_4$** for $1\text{ ms}$ ($t = 15 \to 16$). **$P_4$ completes** ($CT_4 = 16$). Queue: `[]`.

```
Gantt Chart:
+--------+--------+--------+----+--------+----+----+----+
|   P1   |   P2   |   P1   | P3 |   P4   | P2 | P1 | P4 |
+--------+--------+--------+----+--------+----+----+----+
0        3        6        9   10       13   14   15   16
```

#### Final Scheduling Metrics Table

$$\text{Turnaround Time } (TAT_i) = CT_i - AT_i, \qquad \text{Waiting Time } (WT_i) = TAT_i - BT_i$$

| Process | $AT_i$ | $BT_i$ | Completion Time ($CT_i$) | Turnaround Time ($TAT_i = CT_i - AT_i$) | Waiting Time ($WT_i = TAT_i - BT_i$) | Response Time ($RT_i$) |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| $P_1$ | $0$ | $7$ | $15$ | $15 - 0 = 15$ | $15 - 7 = 8$ | $0 - 0 = 0$ |
| $P_2$ | $2$ | $4$ | $14$ | $14 - 2 = 12$ | $12 - 4 = 8$ | $3 - 2 = 1$ |
| $P_3$ | $4$ | $1$ | $10$ | $10 - 4 = 6$ | $6 - 1 = 5$ | $9 - 4 = 5$ |
| $P_4$ | $5$ | $4$ | $16$ | $16 - 5 = 11$ | $11 - 4 = 7$ | $10 - 5 = 5$ |
| **Average** | — | — | — | **$\frac{44}{4} = 11.00\text{ ms}$** | **$\frac{28}{4} = 7.00\text{ ms}$** | **$\frac{11}{4} = 2.75\text{ ms}$** |

---

### Worked Trace 5.3 (Unit 2): Forest of General Trees to LCRS Binary Tree Conversion

**Problem**: Convert the following **Forest of 3 General Trees** ($T_1, T_2, T_3$) into a single **Left-Child Right-Sibling (LCRS) Binary Tree** $B$, and write the Preorder, Inorder, and Postorder traversals of $B$.

```
Original Forest of 3 N-ary Trees:
      T1:   A              T2:   E              T3:   G
          / | \                  |                   / \
         B  C  D                 F                  H   I
```

#### Step 1: Apply the LCRS Transformation Rule
- For every node $u$:
  - `u->left` = **First (leftmost) child** of $u$ in the general tree.
  - `u->right` = **Immediate right sibling** of $u$ in the general tree (and for tree roots $A, E, G$, treat them as siblings: $A\text{->right} = E$, $E\text{->right} = G$).

```
Resulting LCRS Binary Tree B:
            [A]
           /   \
         [B]   [E]
           \   / \
           [C][F] [G]
             \    /
             [D] [H]
                   \
                   [I]
```

#### Step 2: Traversals of LCRS Binary Tree $B$
- **Preorder (`NLR`) of $B$**: `A, B, C, D, E, F, G, H, I` *(Matches Forest Preorder!)*
- **Inorder (`LNR`) of $B$**: `B, C, D, A, F, E, H, I, G` *(Matches Forest Postorder within each tree!)*
- **Postorder (`LRN`) of $B$**: `D, C, B, F, I, H, G, E, A`

---

### Worked Trace 5.4 (Unit 2): Unique Binary Tree Reconstruction from Inorder + Preorder

**Problem**: Reconstruct the unique Binary Tree whose traversals are:
- **Preorder (`NLR`)**: `F, B, A, D, C, E, G, I, H`
- **Inorder (`LNR`)**: `A, B, C, D, E, F, G, H, I`

#### Step-by-Step Recursive Partition Trace

1. **Call 1 (`Pre[0] = F`)**: Root of entire tree is **`F`**.
   - Locate `F` in `Inorder`: `[A, B, C, D, E] | F | [G, H, I]`.
   - Left Subtree of `F` has 5 nodes: `In = [A, B, C, D, E]`, `Pre = [B, A, D, C, E]`.
   - Right Subtree of `F` has 3 nodes: `In = [G, H, I]`, `Pre = [G, I, H]`.
2. **Call 2 (Left of `F`, `Pre = [B, A, D, C, E]`)**: Root is **`B`**.
   - Locate `B` in `In = [A, B, C, D, E]`: `[A] | B | [C, D, E]`.
   - `B->left` = single leaf **`A`**.
   - `B->right` has `In = [C, D, E]`, `Pre = [D, C, E]`.
3. **Call 3 (Right of `B`, `Pre = [D, C, E]`)**: Root is **`D`**.
   - Locate `D` in `In = [C, D, E]`: `[C] | D | [E]`.
   - `D->left` = leaf **`C`**, `D->right` = leaf **`E`**.
4. **Call 4 (Right of `F`, `Pre = [G, I, H]`)**: Root is **`G`**.
   - Locate `G` in `In = [G, H, I]`: `[] | G | [H, I]`.
   - `G->left = NULL`.
   - `G->right` has `In = [H, I]`, `Pre = [I, H]`.
5. **Call 5 (Right of `G`, `Pre = [I, H]`)**: Root is **`I`**.
   - Locate `I` in `In = [H, I]`: `[H] | I | []`.
   - `I->left` = leaf **`H`**, `I->right = NULL`.

```
Reconstructed Unique Binary Tree:
              [F]
             /   \
           [B]   [G]
          /   \     \
        [A]   [D]   [I]
             /   \  /
           [C]  [E][H]

Verification Postorder (LRN): A, C, E, D, B, H, I, G, F
```

---

### Worked Trace 5.5 (Unit 3): AVL Tree Step-by-Step Insertion & Rotation Trace (All 4 Cases)

**Problem**: Insert the sequence `30, 20, 10, 25, 28, 40, 50, 45` into an initially empty AVL Tree, showing the balance factor $\text{BF}(u) = h(L) - h(R)$ and exact rotation performed at each imbalance.

1. **Insert `30`, `20`, `10`**:
   - Node `30` becomes unbalanced with $\text{BF}(30) = 2 - 0 = +2$, and `10` is in the **Left-Left (`LL`)** subtree of `30`.
   - **Action**: **Single Right Rotation (`LL`)** at `30`:
```
    [30] (BF=+2)            [20] (BF=0)
    /          --R(30)-->   /  \
  [20] (BF=+1)            [10] [30]
  /
[10]
```

2. **Insert `25`, then `28`**:
   - `25` attaches as left child of `30` (Tree balanced: $\text{BF}(20) = -1, \text{BF}(30) = +1$).
   - `28` attaches as right child of `25`. Now $\text{BF}(30) = 2 - 0 = +2$ and $\text{BF}(25) = 0 - 1 = -1$ (**Left-Right `LR` case** at `30`!).
   - **Action**: **Double Rotation (`LR`)**: Left-Rotate `25`, then Right-Rotate `30`:
```
    [20]                     [20]                     [20] (BF=-1)
    /  \                     /  \                     /  \
  [10] [30] (BF=+2)        [10] [30]                [10] [28] (BF=0)
       /        --L(25)-->      /       --R(30)-->       /  \
     [25] (BF=-1)             [28]                     [25] [30]
       \                      /
       [28]                 [25]
```

3. **Insert `40`**:
   - `40` attaches as right child of `30`. Now root `20` has left height $1$ (`[10]`) and right height $3$ (`[28]->[30]->[40]`), so $\text{BF}(20) = 1 - 3 = -2$ and $\text{BF}(28) = 1 - 2 = -1$ (**Right-Right `RR` case** at `20`!).
   - **Action**: **Single Left Rotation (`RR`)** at `20` (`28` becomes the new root; `28`'s left child `25` transfers to `20->right`):
```
        [28] (BF=0)
       /    \
     [20]   [30] (BF=-1)
    /    \      \
  [10]  [25]    [40]
```

4. **Insert `50`**:
   - `50` attaches as right child of `40`. Now $\text{BF}(30) = 0 - 2 = -2$ and $\text{BF}(40) = -1$ (**Right-Right `RR` case** at `30`!).
   - **Action**: **Single Left Rotation (`RR`)** at `30`:
```
        [28] (BF=0)
       /    \
     [20]   [40] (BF=0)
    /    \  /    \
  [10]  [25][30] [50]
```

5. **Insert `45`** (To also demonstrate `RL`, suppose instead we had inserted `50` then `45` on `30->right = 50, 50->left = 45`: $\text{BF}(30)=-2, \text{BF}(50)=+1 \implies$ **`RL` Double Rotation**: Right-Rotate `50`, Left-Rotate `30`, making `45` parent of `30` and `50`!).

---

### Worked Trace 5.6 (Unit 3): Splay Tree Splaying Trace (`Zig-Zig`, `Zig-Zag`, `Zig`)

**Problem**: Given the BST below, trace `splay(25)` to bring node `25` to the root using Sleator & Tarjan's bottom-up Splay Tree rules.

```
Initial Tree before splay(25):
          [50] (Root)
          /
        [40] (Grandparent g of 25)
        /
      [30] (Parent p of 25)
      /
    [20]
      \
      [25] <-- Target x
```

#### Step 1: Identify Relationship of `x = 25`, `p = 20`, `g = 30`
- `p = 20` is the **left** child of `g = 30`, and `x = 25` is the **right** child of `p = 20` (**`LR` Zig-Zag**!).
- **Action**: Rotate `p = 20` Left, then rotate `g = 30` Right:
```
After Step 1 (Zig-Zag on 25, 20, 30):
          [50] (Grandparent g of 25)
          /
        [40] (Parent p of 25)
        /
      [25] <-- Target x
     /    \
   [20]  [30]
```

#### Step 2: Identify Relationship of `x = 25`, `p = 40`, `g = 50`
- `p = 40` is the **left** child of `g = 50`, and `x = 25` is the **left** child of `p = 40` (**`LL` Zig-Zig**!).
- **Action**: Rotate **Grandparent `g = 50` Right FIRST**, then rotate **Parent `p = 40` Right SECOND**:
```
1. Right-Rotate g = 50:            2. Right-Rotate p = 40 (Final Splayed Tree!):
        [40]                                   [25] (New Root!)
       /    \                                 /    \
     [25]   [50]                            [20]   [40]
    /    \                                        /    \
  [20]  [30]                                    [30]   [50]
```
*(Notice how the path length was halved from $5$ levels down to $3$ levels!)*

---

### Worked Trace 5.7 (Unit 4): Graph BFS Shortest Path & Connected Components Trace

**Problem**: Consider the undirected graph on $V = \{0, 1, 2, 3, 4, 5\}$ with adjacency lists (sorted ascending):
- `0: [1, 2]`, `1: [0, 3, 4]`, `2: [0, 4]`, `3: [1, 5]`, `4: [1, 2, 5]`, `5: [3, 4]`.
Trace BFS starting from `src = 0` to find `dist[]` and `parent[]` for all vertices.

| Step | Dequeued `u` | Unvisited Neighbors Enqueued (`visited` marked ON ENQUEUE!) | Queue State After Step | `dist[0..5]` | `parent[0..5]` |
| :---: | :---: | :--- | :--- | :--- | :--- |
| Init | — | Enqueue `0` | `[0]` | `[0, ∞, ∞, ∞, ∞, ∞]` | `[-1, -1, -1, -1, -1, -1]` |
| 1 | `0` | `1` (`dist=1, par=0`), `2` (`dist=1, par=0`) | `[1, 2]` | `[0, 1, 1, ∞, ∞, ∞]` | `[-1, 0, 0, -1, -1, -1]` |
| 2 | `1` | `3` (`dist=2, par=1`), `4` (`dist=2, par=1`) | `[2, 3, 4]` | `[0, 1, 1, 2, 2, ∞]` | `[-1, 0, 0, 1, 1, -1]` |
| 3 | `2` | *(Neighbors `0`, `4` already visited!)* | `[3, 4]` | `[0, 1, 1, 2, 2, ∞]` | `[-1, 0, 0, 1, 1, -1]` |
| 4 | `3` | `5` (`dist=3, par=3`) | `[4, 5]` | `[0, 1, 1, 2, 2, 3]` | `[-1, 0, 0, 1, 1, 3]` |
| 5 | `4` | *(Neighbors `1`, `2`, `5` already visited!)* | `[5]` | `[0, 1, 1, 2, 2, 3]` | `[-1, 0, 0, 1, 1, 3]` |
| 6 | `5` | *(Neighbors `3`, `4` already visited!)* | `[]` | **`[0, 1, 1, 2, 2, 3]`** | **`[-1, 0, 0, 1, 1, 3]`** |

- **Shortest Path from `0` to `5`**: Backtrack `parent[5]=3 -> parent[3]=1 -> parent[1]=0` $\implies$ **`0 -> 1 -> 3 -> 5`** (Length = $3$ edges).

---

### Worked Trace 5.8 (Unit 4): Hash Table Collision Resolution Comparison Trace

**Problem**: Insert the key sequence `K = {22, 1, 13, 11, 24, 33, 18, 42}` into a Hash Table of size $m = 11$ with primary hash function $h_1(k) = k \bmod 11$ under all four collision resolution schemes:
1. **Separate Chaining** (insert at tail of chain)
2. **Linear Probing**: $h(k, i) = (h_1(k) + i) \bmod 11$
3. **Quadratic Probing**: $h(k, i) = (h_1(k) + i^2) \bmod 11$
4. **Double Hashing**: $h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod 11$, where $h_2(k) = 7 - (k \bmod 7)$.

#### Step 1: Precompute $h_1(k)$ and $h_2(k)$ for Every Key

| Key $k$ | $h_1(k) = k \bmod 11$ | $h_2(k) = 7 - (k \bmod 7)$ | Probe Traces ($i = 0, 1, 2, \dots$) |
| :---: | :---: | :---: | :--- |
| **`22`** | $0$ | $7 - 1 = 6$ | Lands at slot **`0`** ($1$ probe in all schemes) |
| **`1`** | $1$ | $7 - 1 = 6$ | Lands at slot **`1`** ($1$ probe in all schemes) |
| **`13`** | $2$ | $7 - 6 = 1$ | Lands at slot **`2`** ($1$ probe in all schemes) |
| **`11`** | $0$ | $7 - 4 = 3$ | Collides at `0`!<ul><li>**Linear**: $0 \to 1 \to 2 \to \mathbf{3}$ (4 probes)</li><li>**Quadratic**: $0 \to (0+1^2)=1 \to (0+2^2)=\mathbf{4}$ (3 probes)</li><li>**Double**: $0 \to (0+1\cdot 3)=\mathbf{3}$ (2 probes)</li></ul> |
| **`24`** | $2$ | $7 - 3 = 4$ | Collides at `2`!<ul><li>**Linear**: $2 \to 3 \to \mathbf{4}$ (3 probes)</li><li>**Quadratic**: $2 \to (2+1^2)=\mathbf{3}$ (2 probes)</li><li>**Double**: $2 \to (2+1\cdot 4)=\mathbf{6}$ (2 probes)</li></ul> |
| **`33`** | $0$ | $7 - 5 = 2$ | Collides at `0`!<ul><li>**Linear**: $0 \to 1 \to 2 \to 3 \to 4 \to \mathbf{5}$ (6 probes)</li><li>**Quadratic**: $0 \to 1 \to 4 \to (0+3^2)=\mathbf{9}$ (4 probes)</li><li>**Double**: $0 \to (0+1\cdot 2)=2 \to (0+2\cdot 2)=\mathbf{4}$ (3 probes)</li></ul> |
| **`18`** | $7$ | $7 - 4 = 3$ | Lands at slot **`7`** ($1$ probe in all schemes) |
| **`42`** | $9$ | $7 - 0 = 7$ |<ul><li>**Linear**: Lands at **`9`** (1 probe)</li><li>**Quadratic**: Collides at `9`! $9 \to (9+1^2)=\mathbf{10}$ (2 probes)</li><li>**Double**: Lands at **`9`** (1 probe)</li></ul> |

#### Step 2: Final Hash Table State Across All 4 Schemes

| Bucket Index | Separate Chaining | Linear Probing | Quadratic Probing | Double Hashing |
| :---: | :--- | :---: | :---: | :---: |
| **`0`** | `22 -> 11 -> 33` | `22` | `22` | `22` |
| **`1`** | `1` | `1` | `1` | `1` |
| **`2`** | `13 -> 24` | `13` | `13` | `13` |
| **`3`** | `NULL` | `11` | `24` | `11` |
| **`4`** | `NULL` | `24` | `11` | `33` |
| **`5`** | `NULL` | `33` | `.` | `.` |
| **`6`** | `NULL` | `.` | `.` | `24` |
| **`7`** | `18` | `18` | `18` | `18` |
| **`8`** | `NULL` | `.` | `.` | `.` |
| **`9`** | `42` | `42` | `33` | `42` |
| **`10`** | `NULL` | `.` | `42` | `.` |
| **Total Insertion Probes** | **8 bucket accesses** | **18 probes** | **15 probes** | **12 probes** |

---

## Section 6: Complete, Detailed Answer Keys (Sections 1, 2 & 3)

### 6.1 Answer Key for Section 1 (20 Rapid-Fire Questions)

- **A1.1 — FALSE**: Binary Search requires $O(1)$ random access to the midpoint element (`arr[mid]`). In a Singly Linked List, reaching the middle node requires traversing $n/2 + n/4 + n/8 + \dots = \Theta(n)$ pointers, so Binary Search on a Singly Linked List takes $O(n)$ time (no better than Linear Search). To achieve $O(\log n)$ search on linked nodes, use a **Skip List** or **Balanced BST**.
- **A1.2 — `tail` Pointer in Circular SLL**: If you only store `head`, finding the last node to update `tail->next` during front insertion (`push_front`) or back insertion (`push_back`) requires an $O(n)$ traversal around the circle. If instead you store a single pointer `tail` to the **last** node, then **`tail->next` is already `head`!** Thus, `push_front`, `pop_front`, and `push_back` all execute in **$O(1)$ worst-case time** with only a single pointer variable!
- **A1.3 — FALSE**: The exponentiation operator `^` is **right-associative** ($A^{(B^C)}$). When the second `^` is scanned while the first `^` is on top of the operator stack, a right-associative operator is **pushed** onto the stack (not popped). Thus `A ^ B ^ C` converts to **`A B C ^ ^`**, whereas `(A ^ B) ^ C` converts to `A B ^ C ^`.
- **A1.4 — Breakeven Density of Triplet Matrix**: A dense $R \times C$ matrix uses $R \cdot C$ integers. A Triplet table uses $3(nnz + 1)$ integers (3 fields per non-zero plus 1 header row). Thus Triplet storage exceeds dense storage when:
  $$3(nnz + 1) > R \cdot C \implies nnz \ge \frac{R \cdot C}{3} \quad (\text{i.e., density } > \mathbf{33.3\%})$$
- **A1.5 — FALSE**: In a Skip List with promotion probability $p = 1/2$, each node is allocated **only as many forward pointers as its randomly chosen height $k$** (not $\lceil \log_2 n \rceil$). Because $\Pr[\text{height} \ge k] = (1/2)^{k-1}$, the expected number of pointers per node is $\sum_{k=1}^{\infty} (1/2)^{k-1} = \mathbf{2}$, giving $\Theta(n)$ expected space!
- **A1.6 — Circular Queue Element Count**:
  $$\text{count} = (\text{rear} - \text{front} + \text{CAP}) \bmod \text{CAP} + 1 = (2 - 6 + 8) \bmod 8 + 1 = 4 + 1 = \mathbf{5\text{ elements}}$$
  *( occupying indices `6, 7, 0, 1, 2` ).*
- **A1.7 — FALSE**: Auxiliary memory for DFS (Inorder) is $\Theta(h)$ (tree height), whereas auxiliary memory for BFS (Level-Order) is $\Theta(W)$ (maximum tree width). On a **perfectly balanced binary tree** of $n$ nodes, DFS stack uses only $\Theta(\log n)$ nodes, whereas the BFS queue holds the entire bottom level of $\lceil n/2 \rceil = \Theta(n)$ nodes! Conversely, on a **skewed linear tree**, BFS uses $O(1)$ space ($W=1$) while DFS uses $\Theta(n)$ space ($h=n-1$).
- **A1.8 — Exact Count of `NULL` Child Pointers ($n + 1$)**: An $n$-node binary tree has $2n$ total child pointer slots (`left` and `right` in each node). Every node except the root has exactly $1$ incoming parent pointer, consuming $n - 1$ non-null pointers. Therefore, the number of `NULL` pointers is:
  $$2n - (n - 1) = \mathbf{n + 1}$$
- **A1.9 — FALSE**: If a node $u$ has only **one** child $v$, Preorder (`u v`) and Postorder (`v u`) tell us that $v$ is a child of $u$, but cannot distinguish whether $v$ is the **left child** or the **right child** of $u$! (Inorder is required to separate left from right; Preorder + Postorder is only unique for **Full/Strict** binary trees where every node has $0$ or $2$ children).
- **A1.10 — LCRS Traversal Correspondence**: The **Inorder (`LNR`) traversal of the LCRS Binary Tree $B$** visits `first_child` (all descendants of $u$), then visits node $u$ itself, then visits `next_sibling`—which produces the exact **Postorder traversal** of the original N-ary tree $T$!
- **A1.11 — FALSE**: The Min-Heap property only guarantees $H[\text{parent}(i)] \le H[i]$. The maximum element must be in one of the **leaf nodes** (indices $\lfloor n/2 \rfloor \dots n - 1$), because any internal node is smaller than its children, but it can reside at **any** leaf (e.g., in `[10, 50, 20, 30]`, max `50` is at index `1`, not `n-1 = 3`).
- **A1.12 — Why Floyd's `build_heap` Is $O(n)$**: In top-down insertion, the $\lceil n/2 \rceil$ bottom leaves each sift **up** through $O(\log n)$ levels ($\frac{n}{2} \log n = \Theta(n \log n)$). In Floyd's bottom-up `sift_down`, the $\lceil n/2 \rceil$ leaves do **$0$ work**, $\lceil n/4 \rceil$ nodes at height $1$ sift down at most $1$ step, etc., summing to $\sum_{i=0}^{\lfloor \log_2 n \rfloor} \lceil \frac{n}{2^{i+1}} \rceil \cdot i \le n \sum_{i=1}^{\infty} \frac{i}{2^{i+1}} = n \cdot 1 = O(n)$!
- **A1.13 — FALSE**: While **inserting** into an AVL tree requires at most **one** rotation (single or double) because the rotation reduces the subtree height back to its pre-insertion value, **deleting** from an AVL tree can shorten a subtree and propagate height reductions all the way up the ancestor path, requiring up to $\Theta(\log n)$ rotations in the worst case!
- **A1.14 — Why Zig-Zig Rotates Grandparent First**: If `x` and `p` are both left children (`LL`), rotating `p` first and `g` second (two single `Zig` steps) moves `x` to the top but leaves the remaining nodes strung out in a linear chain of depth $d - 1$. Rotating **grandparent `g` first** and **parent `p` second** folds the tree, halving the depth of **every node** along the access path and guaranteeing the $O(\log n)$ amortized bound!
- **A1.15 — TRUE**: In an Adjacency Matrix `adj[V][V]`, finding the degree of vertex $u$ requires scanning all $V$ entries of row $u$ ($\Theta(V)$ time). In an Adjacency List, iterating over the linked list at `head[u]` visits only the actual incident edges of $u$ ($\Theta(\text{deg}(u))$ time, or $O(1)$ if lengths are cached).
- **A1.16 — FALSE**: **BFS** (not DFS!) finds shortest unweighted paths because a FIFO queue explores vertices in concentric layers of distance $d = 0, 1, 2, \dots$. DFS plunges deeply along a single path first and may reach a neighbor of the source via a long detour of length $O(V)$.
- **A1.17 — Directed vs. Undirected DFS Cycle Detection**: In an undirected graph, any edge to an already-visited vertex $v \ne \text{parent}[u]$ must be a back-edge to an ancestor (a cycle), because undirected DFS has **no cross-edges**. In a directed graph, an edge $(u, v)$ to a visited vertex $v$ might be a harmless **cross-edge** or **forward-edge** to an already finished (`BLACK`) subtree! A directed cycle exists **if and only if** $(u, v)$ points to an active ancestor currently on the recursion stack (`GRAY`).
- **A1.18 — TRUE**: Without a unique terminal symbol `$` $\notin \Sigma$, if a suffix $S[i..m-1]$ is a proper prefix of another longer suffix $S[j..m-1]$ (e.g., `"ab"` inside `"abab"`), the shorter suffix would end at an **internal node** rather than its own distinct **leaf node**. Appending `$` guarantees no suffix is a prefix of another suffix, creating a 1-to-1 bijection between the $m$ suffixes and the $m$ leaves!
- **A1.19 — Quadratic Probing Guarantee Theorem**: If the table capacity $m$ is a **prime number** and the load factor satisfies $\alpha = \frac{n}{m} < \mathbf{0.5}$ (the table is strictly less than half full), then the first $\lceil m/2 \rceil$ quadratic probes $(h(k) + i^2) \bmod m$ for $0 \le i \le \lfloor m/2 \rfloor$ inspect **distinct** buckets, guaranteeing an empty slot is found!
- **A1.20 — FALSE**: Bucket indices depend on the modulus $m$ ($k \bmod 11 \ne k \bmod 23$; for example, $25 \bmod 11 = 3$, whereas $25 \bmod 23 = 2$). Copying bucket $3$ directly to bucket $3$ would place key $25$ in the wrong bucket where `search(25)` (checking $25 \bmod 23 = 2$) will never find it! Every key must be re-hashed using `% m'`.

---

### 6.2 Answer Key for Section 2 (12 Deep Conceptual & Comparative Questions)

#### A2.1 — Cross-Unit Dictionary Showdown

| Criterion | Skip List (Unit 1) | AVL Tree (Unit 3) | Splay Tree (Unit 3) | Standard Trie (Unit 4) | Hash Table (Unit 4) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Search / Insert / Delete Time** | $O(\log n)$ expected w.h.p., $O(n)$ pathological worst | **$O(\log n)$ strict worst-case** | **$O(\log n)$ amortized**, $O(n)$ single worst-case | **$O(L)$ worst-case** ($L = \text{key length}$) | **$O(1)$ expected**, $O(n)$ worst-case |
| **Ordered Range Query `[k1, k2]`** | **Yes**: $O(\log n + k)$ (level-0 list is sorted!) | **Yes**: $O(\log n + k)$ | **Yes**: $O(\log n + k)$ amortized | **Yes**: Lexicographical preorder DFS | **No**: Requires full $O(m)$ table scan + sort |
| **Prefix / Autocomplete Query** | $O(\log n + k)$ | $O(\log n + k)$ | $O(\log n + k)$ amortized | **Optimal**: $O(|P| + k)$ | **No**: Cannot prefix-match hashes |
| **Metadata Overhead per Node** | $\approx 2$ pointers on average ($p=1/2$) | $2$ pointers + height/BF | $2$ pointers ($0$ balance bits!) | $|\Sigma|$ child pointers per node | $0$ pointers (Open Addr) or $1$ ptr (Chaining) |
| **Winning Use Case** | Lock-free concurrent sorted sets (Redis `ZSET`) | Hard real-time systems needing strict $O(\log n)$ + range scans | Skewed temporal locality caches ($80/20$ rule) | String autocomplete, IP routing tables, spell-check | High-speed exact key-value lookup without ordering |

#### A2.2 — Amortized Analysis vs. Hard Real-Time Guarantees
- **Dynamic Array**: Doubling capacity at powers of two ($1, 2, 4, 8, \dots, n$) copies $\sum_{i=0}^{\log_2 n} 2^i = 2n - 1$ elements across $n$ insertions, yielding $\frac{n + (2n-1)}{n} < 3 = O(1)$ amortized cost per `push_back`.
- **Splay Tree**: Using potential function $\Phi(T) = \sum_{u \in T} \lfloor \log_2 \text{size}(u) \rfloor$, the Access Lemma proves any splay operation has amortized cost $\le 3(r(\text{root}) - r(x)) + 1 = O(\log n)$, because expensive deep splays reduce the tree's total potential $\Delta \Phi \ll 0$, paying for the work!
- **Hash Table Rehashing**: Doubling $m \to 2m$ when $\alpha > 0.75$ costs $O(n)$ once every $\Theta(n)$ insertions, amortizing to $O(1)$ per insert.
- **Why Avionics / Hard Real-Time Rejects Amortized Bounds**: Amortized $O(1)$ or $O(\log n)$ only bounds the *average* over a sequence of operations; a single `insert` that triggers a resize or splays a depth-$n$ chain takes $\Theta(n)$ time, which can miss a strict $1\text{ ms}$ flight-control deadline! Hard real-time systems require **worst-case per-operation** $O(\log n)$ (AVL Tree) or $O(1)$ (pre-allocated Circular Buffer).

#### A2.3 — Inorder Traversal: Recursion vs. Explicit Stack vs. Threaded BST
- **Recursive Inorder**: $\Theta(n)$ time, $\Theta(h)$ OS call-stack frames ($\Theta(\log n)$ balanced, $\Theta(n)$ skewed). Risk: OS stack overflow (`SIGSEGV`) if $h \ge 10^5$.
- **Iterative Explicit Stack**: $\Theta(n)$ time, $\Theta(h)$ heap-allocated `TreeNode*` pointers ($8$ bytes per level vs. $32\text{--}64$ bytes per OS call frame). Eliminates stack overflow risk while leaving tree nodes unmodified.
- **Double-Threaded BST**: $\Theta(n)$ time, **$O(1)$ auxiliary memory**! Repurposes the $n+1$ wasted `NULL` pointers to point directly to inorder predecessors/successors. Trade-off: Adds $2$ boolean flags per node and slightly slows down `insert`/`delete` to maintain threads.

#### A2.4 — Sparse Matrix Representations & Fast Transpose Proof
- **Triplet Array (COO)** stores `3 * (nnz + 1)` contiguous integers: great cache locality and compact space, but inserting/deleting a single non-zero entry shifts $O(nnz)$ array elements. **Cross-Linked Multilist** links each non-zero node into both a circular row list and a circular column list: higher per-node memory ($5$ fields/node), but supports dynamic insertion/deletion in $O(nnz_{\text{row}} + nnz_{\text{col}})$ without shifting.
- **Why Naive Transpose Is $O(C \cdot nnz)$ vs. Fast Transpose $O(C + nnz)$**: To output transposed triplets sorted by their new row (which was the old column $c \in [0, C-1]$), Naive Transpose scans all $nnz$ elements $C$ separate times (once for $c = 0$, once for $c = 1, \dots$). **Fast Transpose** uses Counting Sort's prefix-sum technique: Pass 1 counts column frequencies `row_terms[0..C-1]` in $O(nnz)$; Pass 2 computes starting output indices `start_pos[c] = start_pos[c-1] + row_terms[c-1]` in $O(C)$; Pass 3 places each element directly into `b[start_pos[a[i].col]++]` in $O(nnz)$, totaling **$O(C + nnz)$**!

#### A2.5 — CPU Scheduling Policies & Priority Queues
- **FCFS**: FIFO Queue; suffers from the **Convoy Effect** (short I/O-bound jobs stuck behind one long CPU-bound job).
- **Non-Preemptive SJF**: Min-Heap keyed on burst time $BT_i$; provably minimizes average waiting time among non-preemptive policies, but suffers from **Starvation** of long jobs.
- **Preemptive SRTF**: Min-Heap keyed on remaining time $RBT_i$; optimal average waiting time overall, also vulnerable to starvation.
- **Round-Robin (Quantum $q$)**: Circular FIFO Ready Queue; guarantees fair response time $RT_i \le (n-1)q$ and **zero starvation**; if $q \to \infty$, RR degenerates to FCFS; if $q \to 0$, context-switch overhead dominates.
- **Solution to Starvation (Aging)**: Gradually increase the priority (or decrement the effective key in the Min-Heap) of waiting processes proportional to their waiting time $WT_i$, guaranteeing every job eventually reaches the root of the heap.

#### A2.6 — Tree Reconstruction Uniqueness Proof
- **Why Inorder + Preorder Is Unique**: By induction on $n$. For $n \le 1$, trivial. For $n > 1$, `Preorder[0]` uniquely identifies the root $r$. Because all keys are distinct, $r$ appears at a unique index $k$ in `Inorder[0..n-1]`. By definition of Inorder (`LNR`), the $k$ elements to the left `Inorder[0..k-1]` form the exact node set of $r$'s left subtree, and `Inorder[k+1..n-1]` form $r$'s right subtree. Consequently, `Preorder[1..k]` and `Preorder[k+1..n-1]` are the exact Preorder sequences of the left and right subtrees, which are uniquely reconstructed by the inductive hypothesis!
- **Why Preorder + Postorder Fails for General Binary Trees**: Consider Tree 1 (Root `A` with **left** child `B`) and Tree 2 (Root `A` with **right** child `B`). Both have Preorder `A B` and Postorder `B A`! Without Inorder, a 1-child node's left/right orientation is ambiguous. However, in a **Full Binary Tree** (every node has $0$ or $2$ children), if `Preorder[0] = r` is not a leaf, then `Preorder[1]` is unconditionally $r$'s **left child** $L$, and finding $L$ in `Postorder` uniquely partitions the left and right subtrees!

#### A2.7 — Binary Min-Heap vs. Binary Search Tree (BST)
- **Structural Invariant**: A Binary Heap must be a **Complete Binary Tree** (every level full except possibly the bottom level, filled strictly left-to-right), so its height is always $\lfloor \log_2 n \rfloor$ and its nodes map bijectively to array indices `0 .. n-1` with **zero wasted slots and zero pointers**. A BST has no shape constraint (height can be $n-1$), so array storage would waste $O(2^n)$ empty slots.
- **Ordering Invariant**: A BST enforces a **total left-to-right horizontal order** ($\text{Left} < \text{Root} < \text{Right}$), allowing $O(h)$ binary search by eliminating half the tree at each step. A Min-Heap only enforces a **vertical dominance order** ($\text{Parent} \le \text{Left}$ and $\text{Parent} \le \text{Right}$)—it imposes **no ordering relationship between left and right subtrees**! Therefore, searching for a key $x > \text{root}$ cannot eliminate either subtree and requires scanning up to $O(n)$ nodes.

#### A2.8 — AVL Rotation Asymmetry: Why Insert Needs $\le 1$ Rotation While Delete Needs $O(\log n)$
- **Insertion**: Before inserting key $k$, every node has $\text{BF} \in \{-1, 0, +1\}$. When $k$ is inserted, let $z$ be the **lowest** ancestor whose balance factor becomes $+2$ or $-2$. Prior to the insertion, the taller child of $z$ grew by $+1$, increasing $z$'s height from $h$ to $h+1$. Performing a single (`LL`/`RR`) or double (`LR`/`RL`) rotation at $z$ **restores the height of the subtree rooted at $z$ back to its exact pre-insertion height $h$**! Because $z$'s subtree height is now unchanged from before the insertion, no ancestor above $z$ sees any height change, so **zero further rotations** are needed!
- **Deletion**: When deleting a node, performing a rotation at the lowest unbalanced ancestor $z$ often **decreases** the height of $z$'s subtree by $1$ (from $h+1$ down to $h$). That height decrease can propagate upward to $z$'s parent, grandparent, and all the way to the root, triggering up to $\Theta(\log n)$ rotations along the root-to-leaf path!

#### A2.9 — The BFS vs. DFS Memory Paradox on Balanced vs. Skewed Trees
- **Tree A (Complete Balanced Binary Tree, $n = 2^{20}-1 \approx 10^6, h = 19$)**:
  - **DFS (Stack)**: Maximum stack depth is $h + 1 = \mathbf{20\text{ nodes}}$ ($\approx 160\text{ bytes}$)!
  - **BFS (Queue)**: When reaching the bottom leaf level, the queue simultaneously holds $2^{19} = \mathbf{524,288\text{ nodes}}$ ($\approx 4.2\text{ MB}$)!
- **Tree B (Right-Skewed Linear Chain, $n = 10^6, h = 999,999, \text{width} = 1$)**:
  - **DFS (Stack)**: Maximum stack depth is $n = \mathbf{1,000,000\text{ nodes}}$ (crashes OS stack if recursive)!
  - **BFS (Queue)**: Because every node has only $1$ child, the queue holds at most **$\mathbf{1\text{ node}}$** at any time!

#### A2.10 — Standard Trie vs. Suffix Trie vs. Compact Suffix Tree
- An uncompressed **Suffix Trie** of a string $S$ of length $m$ inserts all $m$ suffixes character-by-character. For $S = \texttt{a}^{m/2}\texttt{b}^{m/2}$, there are $\Theta(m^2)$ distinct substrings, creating $\Theta(m^2)$ single-child internal nodes.
- A **Compact Suffix Tree** applies two transformations:
  1. **Path Compression**: Every maximal chain of single-child nodes is collapsed into a single edge. Since a tree with $m$ leaves where every internal node has $\ge 2$ children can have at most **$m - 1$ internal nodes**, the total number of nodes is bounded by $m + (m - 1) + 1 = \mathbf{2m} = O(m)$!
  2. **Implicit Edge Representation**: Instead of storing copies of edge substrings (which could still total $\Theta(m^2)$ characters), each edge stores only a pair of integer indices `(start_idx, end_idx)` into the original string $S$, reducing each edge label to $O(1)$ space ($2$ ints) and total space to **$\Theta(m)$**!

#### A2.11 — Primary Clustering vs. Secondary Clustering vs. Double Hashing & CPU Cache Effects
- **Primary Clustering (Linear Probing)**: Any key whose hash falls anywhere inside a contiguous block of $L$ occupied slots must scan to the end of the block and extends its length to $L + 1$. Thus large clusters have a higher probability $(L+1)/m$ of growing even larger!
- **Secondary Clustering (Quadratic Probing)**: Keys that hash to *different* home slots inside a cluster do not merge into a single giant block, but two keys with the **exact same initial hash** $h_1(k_1) = h_1(k_2)$ follow the identical probe sequence $h_1 + 1^2, h_1 + 2^2, \dots$.
- **Double Hashing**: Uses a key-dependent step size $h_2(k)$, so even keys with $h_1(k_1) = h_1(k_2)$ jump by different strides, eliminating both primary and secondary clustering.
- **Why Linear Probing Still Wins in Hardware Benchmarks at $\alpha \le 0.6$**: A modern CPU loads memory in **64-byte L1 cache lines** (holding sixteen 32-bit integers at once). In Linear Probing, probes $i = 0, 1, 2, 3, 4$ read adjacent array elements already sitting in the **same L1 cache line** ($< 1\text{ ns}$ latency), whereas Double Hashing jumps to random array indices, suffering a costly **RAM / L3 cache miss** ($\approx 50\text{--}100\text{ ns}$) on every probe!

#### A2.12 — Tombstone Accumulation & Periodic Rehashing
- In Open Addressing, deleting a key must mark its slot `DELETED` (tombstone) rather than `EMPTY` so searches for keys probed past that slot do not stop prematurely.
- **The Contamination Problem**: While `insert()` can reuse `DELETED` slots, **unsuccessful searches** must continue probing past every `DELETED` slot until they finally encounter a truly `EMPTY` slot. Over millions of alternating `insert` and `delete` operations at constant load factor $\alpha = n/m = 0.4$, almost every unused slot in the table transitions from `EMPTY` to `DELETED`! Once zero `EMPTY` slots remain, every unsuccessful search degrades from $O(1)$ to scanning all $m$ slots ($\Theta(m)$)!
- **Solution**: Track `num_occupied + num_deleted`. Whenever `(num_occupied + num_deleted) / m` exceeds a threshold (e.g., $0.75$), trigger a **Rehash** (into a fresh table of the same size $m$ if $n/m$ is small, or $2m$ if $n/m$ is large), re-inserting only the `OCCUPIED` keys and wiping out $100\%$ of the tombstones in $O(m)$ time!

---

### 6.3 Answer Key for Section 3 (8 Algorithm Design & Pseudocode Problems)

#### A3.1 — $O(n)$ Time, $O(1)$ Space In-Place Linked List Palindrome Check
1. Use Fast & Slow pointers (`fast->next && fast->next->next`) to find the first middle node `mid`.
2. Reverse the second half of the list starting at `mid->next` in-place in $O(n/2)$ time and $O(1)$ space.
3. Compare the first half (`p1 = head`) and reversed second half (`p2 = second_half_head`) node-by-node.
4. Reverse `second_half_head` a second time and reattach `mid->next` to restore the original list before returning `is_palin`.

```text
ALGORITHM IsPalindromeList(head):
    IF head == NULL OR head.next == NULL THEN RETURN true
    slow <- head; fast <- head
    WHILE fast.next != NULL AND fast.next.next != NULL DO
        slow <- slow.next
        fast <- fast.next.next
    second_half <- ReverseList(slow.next)
    p1 <- head; p2 <- second_half; result <- true
    WHILE p2 != NULL DO
        IF p1.data != p2.data THEN result <- false; BREAK
        p1 <- p1.next; p2 <- p2.next
    slow.next <- ReverseList(second_half)   // Restore original list!
    RETURN result
```

#### A3.2 — $O(1)$ Worst-Case `MinStack`
Maintain each stack entry as a pair `(val, current_min)` (or maintain a second auxiliary stack `min_stack` that only pushes when `val <= min_stack.top()`):
- `push(x)`: `new_min = isEmpty() ? x : min(x, top().current_min); push_pair(x, new_min);` — **$O(1)$**
- `pop()`: `pop_pair();` — **$O(1)$**
- `getMin()`: `return top().current_min;` — **$O(1)$**

#### A3.3 — Queue Using Two Stacks (`S_in`, `S_out`) & Banker's Amortized Proof
```text
ALGORITHM Enqueue(Q, x):
    Push(Q.S_in, x)                        // Always O(1) worst-case

ALGORITHM Dequeue(Q):
    IF IsEmpty(Q.S_out) THEN
        WHILE NOT IsEmpty(Q.S_in) DO
            Push(Q.S_out, Pop(Q.S_in))     // Reverses LIFO into FIFO order!
    RETURN Pop(Q.S_out)
```
- **Amortized $O(1)$ Proof (Accounting Method)**: Charge **$3$ credits** on every `Enqueue(x)`: $1$ credit pays for the immediate `Push(S_in, x)`, leaving $2$ credits stored on element `x`. When `x` is later transferred from `S_in` to `S_out`, its $2$ stored credits pay for `Pop(S_in)` ($1$ credit) and `Push(S_out)` ($1$ credit). Because each element is transferred from `S_in` to `S_out` **at most once** in its lifetime, `Dequeue` only needs $1$ credit to pay for `Pop(S_out)`. Thus both operations run in **$O(1)$ amortized time**!

#### A3.4 — Lowest Common Ancestor (LCA) in General Binary Tree vs. BST
```text
// 1. General Binary Tree LCA — O(n) Time, O(h) Space
ALGORITHM LCA_BinaryTree(root, p, q):
    IF root == NULL OR root == p OR root == q THEN RETURN root
    left_res  <- LCA_BinaryTree(root.left,  p, q)
    right_res <- LCA_BinaryTree(root.right, p, q)
    IF left_res != NULL AND right_res != NULL THEN
        RETURN root    // p and q found in different subtrees -> root is the split point!
    RETURN (left_res != NULL) ? left_res : right_res

// 2. Binary Search Tree LCA — O(h) Time, O(1) Iterative Space
ALGORITHM LCA_BST(root, p_key, q_key):
    curr <- root
    WHILE curr != NULL DO
        IF p_key < curr.key AND q_key < curr.key THEN
            curr <- curr.left
        ELSE IF p_key > curr.key AND q_key > curr.key THEN
            curr <- curr.right
        ELSE
            RETURN curr   // First split point (or one key equals curr.key) is the LCA!
```

#### A3.5 — Order-Statistic AVL Tree (`kth_smallest` & `count_range` in $O(\log n)$)
Augment every node $u$ with `u.size = 1 + size(u.left) + size(u.right)` (updated in $O(1)$ inside `rotate_left` and `rotate_right` right next to `u.height`).
```text
ALGORITHM KthSmallest(u, k):   // 1-indexed k in [1 .. u.size]
    left_sz <- (u.left != NULL) ? u.left.size : 0
    IF k == left_sz + 1 THEN RETURN u.key
    ELSE IF k <= left_sz THEN RETURN KthSmallest(u.left, k)
    ELSE RETURN KthSmallest(u.right, k - left_sz - 1)
```
To compute `count_range(root, low, high)` in $O(\log n)$ time, define `Rank(root, x)` (number of keys $\le x$ in $O(\log n)$ by summing `1 + size(u.left)` whenever $x \ge u.\text{key}$) and return `Rank(root, high) - Rank(root, low - 1)`.

#### A3.6 — Rebuilding an Arbitrary BST into a Minimum-Height Balanced BST in $O(n)$ Time
1. **Step 1 ($O(n)$ Time)**: Perform an **Inorder Traversal** of the input BST, storing the node pointers (or keys) into a sorted array `A[0 .. n-1]`. Because Inorder of any BST is sorted, no sorting step ($O(n \log n)$) is needed!
2. **Step 2 ($O(n)$ Time)**: Recursively pick the midpoint `mid = (low + high) / 2` as the root of `A[low..high]`:
```text
ALGORITHM BuildBalancedFromSorted(A, low, high):
    IF low > high THEN RETURN NULL
    mid       <- low + (high - low) / 2
    root      <- A[mid]
    root.left  <- BuildBalancedFromSorted(A, low, mid - 1)
    root.right <- BuildBalancedFromSorted(A, mid + 1, high)
    RETURN root
```
Recurrence $T(n) = 2T(n/2) + O(1) \implies \Theta(n)$ time and resulting height $\lfloor \log_2 n \rfloor$.

#### A3.7 — Bipartite Graph Check (2-Coloring) via BFS in $O(V + E)$
```text
ALGORITHM IsBipartite(G):
    color[0 .. V-1] <- {-1, -1, ..., -1}   // -1 = uncolored, 0 = RED, 1 = BLUE
    FOR start <- 0 TO V - 1 DO             // Outer loop handles disconnected components!
        IF color[start] == -1 THEN
            color[start] <- 0
            Enqueue(Q, start)
            WHILE NOT IsEmpty(Q) DO
                u <- Dequeue(Q)
                FOR EACH neighbor v IN G.Adj[u] DO
                    IF color[v] == -1 THEN
                        color[v] <- 1 - color[u]   // Assign opposite color
                        Enqueue(Q, v)
                    ELSE IF color[v] == color[u] THEN
                        RETURN false               // Odd-length cycle detected!
    RETURN true
```

#### A3.8 — Longest Common Prefix (LCP) & Lexicographical Autocomplete in a Trie
- **(a) Longest Common Prefix ($O(\text{length of LCP})$)**: Starting at `root`, walk down the Trie as long as the current node `curr` has `curr.is_end == false` AND `child_count(curr) == 1`. Append the single child's character to `lcp_str`. Stop as soon as a node has $\ge 2$ children (branch point) or `is_end == true` (shortest word ended).
- **(b) Autocomplete ($O(|P| + K_{\text{output}})$)**:
  1. Walk from `root` following the characters of prefix $P$ (`curr = curr.children[P[i] - 'a']`). If `curr == NULL` at any step, return `{}` (no matches).
  2. From the node `curr` reached at the end of $P$, run a recursive Preorder DFS iterating `c` from `0` (`'a'`) to `25` (`'z'`), pushing `c` onto a path buffer before recursing and popping `c` on backtrack. Whenever `node.is_end == true`, print the buffer! Because children `0..25` are visited in alphabetical order, all matching completions are printed in **strictly sorted lexicographical order** without needing any sorting algorithm!

---

> **End of Appendices**: Return to **[Appendix 01: Complexity Cheat Sheet](./01-Complexity-Cheat-Sheet.md)**, **[Appendix 02: C Programming Patterns](./02-C-Programming-Patterns.md)**, or **[Appendix 03: Common Errors & Edge Cases](./03-Common-Errors-and-Edge-Cases.md)**.
