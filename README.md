# Data Structures and Algorithms in C — Complete University Study Notes

---

## Table of Contents
1. [Project Overview](#1-project-overview)
2. [Repository Structure and Navigation](#2-repository-structure-and-navigation)
   - 2.1 [Core Navigation and Revision Files](#21-core-navigation-and-revision-files)
   - 2.2 [Unit 1: Linked Lists and Stacks](#22-unit-1-linked-lists-and-stacks)
   - 2.3 [Unit 2: Queues and Trees](#23-unit-2-queues-and-trees)
   - 2.4 [Unit 3: Applications of Trees and Introduction to Graphs](#24-unit-3-applications-of-trees-and-introduction-to-graphs)
   - 2.5 [Unit 4: Applications of Graphs, Suffix Structures, and Hashing](#25-unit-4-applications-of-graphs-suffix-structures-and-hashing)
   - 2.6 [Appendices and Exam Preparation](#26-appendices-and-exam-preparation)
3. [Recommended Learning Order and Topic Dependency Roadmap](#3-recommended-learning-order-and-topic-dependency-roadmap)
4. [Pedagogical Structure of Each Topic Note](#4-pedagogical-structure-of-each-topic-note)
5. [C11 Compilation, Memory Safety, and Coding Conventions](#5-c11-compilation-memory-safety-and-coding-conventions)

---

## 1. Project Overview

Welcome to the **Data Structures and Algorithms in C** comprehensive textbook-grade study repository. Designed for university coursework, competitive examinations, laboratory programming, and technical interviews, these notes develop every topic in the four-unit syllabus from first principles to complete, production-quality **ISO C11** implementations.

### Key Highlights
- **100% Syllabus Coverage**: Every data structure, representation variant, operation, case study, and application in the four-unit curriculum is thoroughly explained.
- **Step-by-Step Visual Traces**: Rich ASCII memory layouts, before-and-after pointer diagrams, tree rotation states, operator-stack traces, graph traversal tables, and hash probing states.
- **Compilable & Verified C11 Programs**: Every topic file includes complete, self-contained C11 programs with `main(void)` drivers, strict `NULL` allocation guards, full dynamic memory cleanup (`free`), and verified program outputs.
- **Rigorous Mathematical & Complexity Analysis**: Formal definitions, proofs of structural theorems, and Best/Average/Worst-case time and auxiliary-space complexity tables.
- **Exam-Focused Preparation**: Each unit concludes with a dedicated **Unit Review** containing definitions, formulas, commonly confused concepts, short-answer, conceptual, algorithm-writing, C coding, and worked problem-solving questions with full answer keys.

---

## 2. Repository Structure and Navigation

```text
DSA Notes/
│
├── README.md
├── SYLLABUS-COVERAGE.md
├── QUICK-REVISION.md
│
├── Unit-1-Linked-Lists-and-Stacks/
│   ├── 01-C-Structures-Pointers-and-Memory.md
│   ├── 02-Singly-Linked-Lists.md
│   ├── 03-Doubly-Linked-Lists.md
│   ├── 04-Circular-Linked-Lists.md
│   ├── 05-Multilists-Sparse-Matrices-and-Skip-Lists.md
│   ├── 06-Polynomial-Representation-and-Operations.md
│   ├── 07-Stacks-Array-and-Linked-List.md
│   ├── 08-Stack-Applications-and-Recursion.md
│   ├── 09-Expression-Conversion-and-Evaluation.md
│   └── 10-Unit-1-Review.md
│
├── Unit-2-Queues-and-Trees/
│   ├── 01-Queue-Fundamentals.md
│   ├── 02-Circular-Priority-and-Deque.md
│   ├── 03-Josephus-Problem-and-CPU-Scheduling.md
│   ├── 04-N-Ary-Binary-Trees-and-Forests.md
│   ├── 05-Tree-Traversals.md
│   └── 06-Unit-2-Review.md
│
├── Unit-3-Tree-Applications-and-Graphs/
│   ├── 01-BST-Array-and-Dynamic-Implementations.md
│   ├── 02-Expression-Trees.md
│   ├── 03-Threaded-Binary-Search-Trees.md
│   ├── 04-Heaps-and-Heap-Priority-Queues.md
│   ├── 05-AVL-Trees.md
│   ├── 06-Splay-Trees.md
│   ├── 07-Graph-Fundamentals-and-Representations.md
│   ├── 08-DFS-and-BFS.md
│   └── 09-Unit-3-Review.md
│
├── Unit-4-Graph-Applications-Tries-and-Hashing/
│   ├── 01-Graph-Connectivity-Components-and-Paths.md
│   ├── 02-Trie-Trees.md
│   ├── 03-Suffix-Tries-and-Suffix-Trees.md
│   ├── 04-Hash-Functions-and-Hash-Tables.md
│   ├── 05-Collision-Handling-and-Rehashing.md
│   └── 06-Unit-4-Review.md
│
└── Appendices/
    ├── 01-Complexity-Cheat-Sheet.md
    ├── 02-C-Programming-Patterns.md
    ├── 03-Common-Errors-and-Edge-Cases.md
    └── 04-Practice-Questions-and-Answers.md
```

---

### 2.1 Core Navigation and Revision Files

| File | Purpose |
| :--- | :--- |
| **[`README.md`](./README.md)** | Master index, learning order, compilation guide, and navigation hub. |
| **[`SYLLABUS-COVERAGE.md`](./SYLLABUS-COVERAGE.md)** | Evidence-based checklist mapping every syllabus item to exact files and section headings. |
| **[`QUICK-REVISION.md`](./QUICK-REVISION.md)** | High-density revision notes: key definitions, mathematical formulas, traversal rules, exam distinctions, and consolidated complexity tables. |

---

### 2.2 Unit 1: Linked Lists and Stacks

| # | Topic File | Key Concepts Covered |
| :---: | :--- | :--- |
| **01** | **[C Structures, Pointers, and Memory](./Unit-1-Linked-Lists-and-Stacks/01-C-Structures-Pointers-and-Memory.md)** | `struct`, `typedef`, `->` operator, pointer manipulation, `NULL`, stack vs. static vs. heap memory, `malloc`, `calloc`, `realloc`, `free`, memory leaks, dangling pointers, self-referential nodes. |
| **02** | **[Singly Linked Lists](./Unit-1-Linked-Lists-and-Stacks/02-Singly-Linked-Lists.md)** | Node layout, head/tail management, empty/single-node cases, insertion (head, tail, position, before/after key), deletion (head, tail, position, by key), search, count, update, iterative & recursive reversal. |
| **03** | **[Doubly Linked Lists](./Unit-1-Linked-Lists-and-Stacks/03-Doubly-Linked-Lists.md)** | Bidirectional `prev`/`next` pointers, 4-pointer insertion/deletion wiring, $O(1)$ deletion given node pointer, forward/backward traversal, reversal, SLL vs. DLL comparison. |
| **04** | **[Circular Linked Lists](./Unit-1-Linked-Lists-and-Stacks/04-Circular-Linked-Lists.md)** | Singly & doubly circular lists, `tail`-pointer $O(1)$ head/tail insertion optimization, `do-while` traversal, deletion, reversal, SLL vs. DLL vs. CLL comparison. |
| **05** | **[Multilists, Sparse Matrices, and Skip Lists](./Unit-1-Linked-Lists-and-Stacks/05-Multilists-Sparse-Matrices-and-Skip-Lists.md)** | Multi-link shared nodes, Sparse Matrix 3-Tuple (Triplet) array & Orthogonal Cross-Linked Multilist, Skip List probabilistic levels, search/insert/delete, $O(\log n)$ expected vs. $O(n)$ worst-case analysis. |
| **06** | **[Polynomial Representation and Operations](./Unit-1-Linked-Lists-and-Stacks/06-Polynomial-Representation-and-Operations.md)** | Case Study: Array vs. Linked List representation, descending exponent invariant, combining like terms, zero-coefficient handling, addition, subtraction, multiplication, and Horner's evaluation. |
| **07** | **[Stacks: Array and Linked List](./Unit-1-Linked-Lists-and-Stacks/07-Stacks-Array-and-Linked-List.md)** | LIFO principle, Stack ADT, fixed array, dynamic resizable array, and singly linked list implementations (`push`, `pop`, `peek`, `isEmpty`, `isFull`), overflow and underflow. |
| **08** | **[Stack Applications and Recursion](./Unit-1-Linked-Lists-and-Stacks/08-Stack-Applications-and-Recursion.md)** | System call stack & activation records, recursive execution traces (Factorial, Fibonacci, Tower of Hanoi), tail recursion, explicit stack conversion, multi-delimiter balanced parentheses matching. |
| **09** | **[Expression Conversion and Evaluation](./Unit-1-Linked-Lists-and-Stacks/09-Expression-Conversion-and-Evaluation.md)** | Infix, Postfix (RPN), and Prefix (Polish) notations, operator precedence & associativity (`^` right-to-left vs. `* / + -` left-to-right), Shunting-Yard Infix-to-Postfix, Infix-to-Prefix, Postfix & Prefix evaluation. |
| **10** | **[Unit 1 Comprehensive Review](./Unit-1-Linked-Lists-and-Stacks/10-Unit-1-Review.md)** | Glossary, formulas, commonly confused concepts, Unit 1 complexity table, 30+ practice questions (short-answer, conceptual, algorithm, C code, worked traces) with full answer keys. |

---

### 2.3 Unit 2: Queues and Trees

| # | Topic File | Key Concepts Covered |
| :---: | :--- | :--- |
| **01** | **[Queue Fundamentals](./Unit-2-Queues-and-Trees/01-Queue-Fundamentals.md)** | FIFO principle, Queue ADT (`enqueue`, `dequeue`, `front`, `rear`, `isEmpty`, `isFull`), simple linear array queue, the "False Overflow" rightward-drift problem, and $O(1)$ `front`/`rear` linked-list queue. |
| **02** | **[Circular Queue, Priority Queue, and Deque](./Unit-2-Queues-and-Trees/02-Circular-Priority-and-Deque.md)** | Circular indexing `(i + 1) % C`, empty vs. full disambiguation, Priority Queue (array & sorted list, tie handling), Double-Ended Queue (Input-Restricted & Output-Restricted Deques). |
| **03** | **[Josephus Problem and CPU Scheduling](./Unit-2-Queues-and-Trees/03-Josephus-Problem-and-CPU-Scheduling.md)** | Case Studies: Josephus $J(n, k)$ queue elimination trace & $O(n)$ recurrence; OS CPU Scheduling (FCFS & Round-Robin with time quantum $q$, Gantt charts, CT, TAT, WT, and RT calculations). |
| **04** | **[N-Ary Trees, Binary Trees, and Forests](./Unit-2-Queues-and-Trees/04-N-Ary-Binary-Trees-and-Forests.md)** | Tree terminology & mathematical proofs ($|E| = |V| - 1$, $L = I_2 + 1$, $|E| = |V| - k$), N-ary representations, Full/Complete/Perfect/Skewed binary trees, BST basics, Forests, LCRS N-ary & Forest to Binary Tree conversions. |
| **05** | **[Tree Traversals](./Unit-2-Queues-and-Trees/05-Tree-Traversals.md)** | Recursive and Iterative (explicit stack) Preorder, Inorder, and Postorder traversals, Level-Order (BFS), tree reconstruction from traversals, and Binary vs. N-ary traversal correspondences. |
| **06** | **[Unit 2 Comprehensive Review](./Unit-2-Queues-and-Trees/06-Unit-2-Review.md)** | Glossary, formulas, commonly confused concepts, Unit 2 complexity table, 30+ practice questions and worked exam problems with complete answer keys. |

---

### 2.4 Unit 3: Applications of Trees and Introduction to Graphs

| # | Topic File | Key Concepts Covered |
| :---: | :--- | :--- |
| **01** | **[BST: Array and Dynamic Implementations](./Unit-3-Tree-Applications-and-Graphs/01-BST-Array-and-Dynamic-Implementations.md)** | BST ordering invariant, implicit array representation (`2i+1`, `2i+2`) vs. dynamic pointer representation, search, insert, min/max, successor/predecessor, 3-case deletion, range queries, and BST validation. |
| **02** | **[Expression Trees](./Unit-3-Tree-Applications-and-Graphs/02-Expression-Trees.md)** | Stack-based construction from postfix expressions, Preorder (Prefix), Postorder (Postfix), and parenthesized Inorder (Infix) generation, recursive bottom-up arithmetic evaluation. |
| **03** | **[Threaded Binary Search Trees](./Unit-3-Tree-Applications-and-Graphs/03-Threaded-Binary-Search-Trees.md)** | Repurposing $n + 1$ `NULL` pointers as Inorder Predecessor/Successor threads, single vs. fully threaded BSTs, `lthread`/`rthread` flags, $O(1)$-space stackless Inorder traversal, threaded insertion & deletion. |
| **04** | **[Heaps and Heap Priority Queues](./Unit-3-Tree-Applications-and-Graphs/04-Heaps-and-Heap-Priority-Queues.md)** | Complete binary tree shape & Min/Max heap-order invariants, array index formulas, `sift_up` (insert), `sift_down` (`heapify` / extract root), $O(n)$ bottom-up Floyd's `build_heap` proof, Heap Sort, and Priority Queues. |
| **05** | **[AVL Trees](./Unit-3-Tree-Applications-and-Graphs/05-AVL-Trees.md)** | Height-balanced BSTs, balance factor $BF \in \{-1, 0, +1\}$, $O(\log n)$ height bound, LL, RR, LR, and RL imbalances and single/double rotations, AVL insertion and deletion. |
| **06** | **[Splay Trees](./Unit-3-Tree-Applications-and-Graphs/06-Splay-Trees.md)** | Self-adjusting BSTs and temporal locality, Zig, Zig-Zig (rotate grandparent first!), and Zig-Zag splaying steps, splay search/insert/delete, amortized $O(\log n)$ vs. worst-case $O(n)$ analysis, AVL vs. Splay comparison. |
| **07** | **[Graph Fundamentals and Representations](./Unit-3-Tree-Applications-and-Graphs/07-Graph-Fundamentals-and-Representations.md)** | Directed/undirected, weighted/unweighted graphs, degree, in/out-degree, Handshaking Lemma, paths, cycles, connectivity, Adjacency Matrix vs. Adjacency List implementations on consistent examples. |
| **08** | **[DFS and BFS Graph Traversals](./Unit-3-Tree-Applications-and-Graphs/08-DFS-and-BFS.md)** | Queue-based BFS and Recursive/Iterative Stack-based DFS, `visited[]` management, step-by-step traversal traces, edge classifications, $O(V + E)$ (Adjacency List) vs. $O(V^2)$ (Adjacency Matrix) complexity. |
| **09** | **[Unit 3 Comprehensive Review](./Unit-3-Tree-Applications-and-Graphs/09-Unit-3-Review.md)** | Glossary, formulas, commonly confused concepts, Unit 3 complexity table, 30+ practice questions and worked traces with complete answer keys. |

---

### 2.5 Unit 4: Applications of Graphs, Suffix Structures, and Hashing

| # | Topic File | Key Concepts Covered |
| :---: | :--- | :--- |
| **01** | **[Graph Connectivity, Components, and Paths](./Unit-4-Graph-Applications-Tries-and-Hashing/01-Graph-Connectivity-Components-and-Paths.md)** | Testing undirected/directed connectivity, counting & labeling connected components, `visited[]` and `parent[]` path reconstruction, DFS reachability vs. BFS shortest path guarantee in unweighted graphs. |
| **02** | **[Trie Trees](./Unit-4-Graph-Applications-Tries-and-Hashing/02-Trie-Trees.md)** | Prefix tree architecture, alphabet indexing, `is_end_of_word` & prefix counters, $O(m)$ insertion, exact search, prefix search, lexicographical autocomplete, and 4-case recursive postorder deletion. |
| **03** | **[Suffix Tries and Suffix Trees](./Unit-4-Graph-Applications-Tries-and-Hashing/03-Suffix-Tries-and-Suffix-Trees.md)** | Substrings as prefixes of suffixes, terminal marker `$`, uncompressed Suffix Trie ($O(m^2)$ space/time) vs. compressed Suffix Tree ($\le 2m - 1$ nodes), pattern search, educational $O(m^2)$ vs. Ukkonen's $O(m)$ distinction. |
| **04** | **[Hash Functions and Hash Tables](./Unit-4-Graph-Applications-Tries-and-Hashing/04-Hash-Functions-and-Hash-Tables.md)** | Direct addressing vs. hashing, keys, buckets, load factor $\alpha = n/m$, uniform hashing properties, Division, Multiplication, Mid-Square, Folding, and Polynomial Rolling String hash functions. |
| **05** | **[Collision Handling and Rehashing](./Unit-4-Graph-Applications-Tries-and-Hashing/05-Collision-Handling-and-Rehashing.md)** | Separate Chaining, Open Addressing (Linear Probing, Quadratic Probing, Double Hashing), Primary vs. Secondary Clustering, Tombstone (`DELETED`) deletion mechanics, and dynamic Rehashing to prime capacities. |
| **06** | **[Unit 4 Comprehensive Review](./Unit-4-Graph-Applications-Tries-and-Hashing/06-Unit-4-Review.md)** | Glossary, formulas, commonly confused concepts, Unit 4 complexity table, 30+ practice questions and worked collision/graph/trie traces with complete answer keys. |

---

### 2.6 Appendices and Exam Preparation

| # | Appendix File | Description |
| :---: | :--- | :--- |
| **A1** | **[Complexity Cheat Sheet](./Appendices/01-Complexity-Cheat-Sheet.md)** | Master Big-O Time and Space Complexity reference across all four units, closed-form mathematical series, and data-structure selection matrix. |
| **A2** | **[C Programming Patterns for DSA](./Appendices/02-C-Programming-Patterns.md)** | Reusable C11 idioms: self-referential structs, safe `realloc`, `Node*` vs. `Node**` double-pointer modification, dummy sentinel nodes, fast/slow pointers, and modular circular indexing. |
| **A3** | **[Common Errors and Edge Cases](./Appendices/03-Common-Errors-and-Edge-Cases.md)** | Diagnostic catalog of 20 classic C DSA bugs (`/* BUG */` vs. `/* FIX */`) across pointers, lists, stacks, queues, trees, heaps, graphs, tries, and hash tables. |
| **A4** | **[Practice Questions and Answers](./Appendices/04-Practice-Questions-and-Answers.md)** | Cross-unit university examination bank: short-answer, conceptual comparisons, algorithm design, C11 coding problems, and full hand-traced worked solutions. |

---

## 3. Recommended Learning Order and Topic Dependency Roadmap

```
+-------------------------------------------------------------------------+
| FOUNDATION: Unit 1.01 (C Structures, Pointers & Dynamic Memory)         |
+-------------------------------------------------------------------------+
       |
       +---> Unit 1.02 - 1.04: Linear Linked Lists (Singly, Doubly, Circular)
       |            |
       |            +---> Unit 1.05: Multilists, Sparse Matrices & Skip Lists
       |            +---> Unit 1.06: Polynomial Representation & Operations
       |
       +---> Unit 1.07 - 1.09: Stacks (LIFO), Recursion & Expression Evaluation
       |
       +---> Unit 2.01 - 2.03: Queues (FIFO), Circular/Priority/Deque,
       |                       Josephus Problem & CPU Scheduling
       |
       v
+-------------------------------------------------------------------------+
| HIERARCHICAL STRUCTURES: Unit 2.04 - 2.05 (Trees, Forests & Traversals) |
+-------------------------------------------------------------------------+
       |
       +---> Unit 3.01: Binary Search Trees (Array & Dynamic Pointers)
       |        |---> Unit 3.02: Expression Trees (Stacks + Binary Trees)
       |        |---> Unit 3.03: Threaded Binary Search Trees (O(1)-Space Inorder)
       |        |---> Unit 3.05: AVL Trees (Strict Height-Balanced Rotations)
       |        +---> Unit 3.06: Splay Trees (Amortized Self-Adjusting BST)
       |
       +---> Unit 3.04: Binary Heaps & Priority Queues (Complete Binary Tree Array)
       +---> Unit 4.02 - 4.03: Trie Trees, Suffix Tries & Suffix Trees
       |
       v
+-------------------------------------------------------------------------+
| NETWORKS & INDEXING: Unit 3.07 - 3.08 & Unit 4.01, 4.04 - 4.05          |
+-------------------------------------------------------------------------+
       |---> Unit 3.07 - 3.08: Graph Representations, BFS (Queue) & DFS (Stack)
       |---> Unit 4.01: Graph Connectivity, Components & Shortest Paths
       +---> Unit 4.04 - 4.05: Hash Functions, Separate Chaining, Open Addressing & Rehashing
```

---

## 4. Pedagogical Structure of Each Topic Note

Every topic file follows a consistent 12-part instructional template:
1. **Introduction & Learning Objectives** — Intuition, motivation, and prerequisites.
2. **Numbered Table of Contents** — Quick internal navigation.
3. **Definitions & Mathematical Properties** — Formal terminology, invariants, and LaTeX proofs.
4. **Representation & Memory Layout in C** — `struct` declarations, pointer diagrams, and storage trade-offs.
5. **Core Operations & Step-by-Step Algorithms** — Purpose, preconditions, steps, and pseudocode explained *before* code.
6. **ASCII Diagrams & Worked Traces** — Visual state transitions for every pointer update, rotation, traversal, or collision.
7. **Complete, Compilable C11 Implementations** — Full working programs with `main(void)` and traced sample output.
8. **Complexity Analysis** — Best, average, and worst-case time and auxiliary space with explicit variable definitions.
9. **Comparison Tables** — Structured side-by-side contrasts of competing designs.
10. **Applications & Case Studies** — Real-world systems and syllabus problem studies.
11. **Common Errors & Edge Cases** — `/* BUG */` vs. `/* FIX */` examples and boundary checklists.
12. **Summary & Revision Checklist** — Key takeaways, self-test questions, and cross-references.

---

## 5. C11 Compilation, Memory Safety, and Coding Conventions

All C programs in these notes conform to **ISO/IEC 9899:2011 (C11)** and use only standard library headers (`<stdio.h>`, `<stdlib.h>`, `<stdbool.h>`, `<string.h>`, `<limits.h>`, `<ctype.h>`).

### Compiling Programs with `gcc` or `clang`
To compile and execute any complete C program from the notes:

```bash
# Using GCC (Linux / macOS / MinGW on Windows)
gcc -std=c11 -Wall -Wextra -pedantic -O2 program.c -o program
./program

# Using Clang
clang -std=c11 -Wall -Wextra -pedantic -O2 program.c -o program
./program
```

### Memory Safety Standards Used Throughout
- Every call to `malloc()`, `calloc()`, or `realloc()` is immediately verified against `NULL`.
- Resizing dynamic arrays always uses a temporary pointer (`void* tmp = realloc(ptr, new_size)`) to avoid leaking the original block on allocation failure.
- Every dynamically allocated list, tree, graph, trie, and hash table is completely freed before `main(void)` terminates.
