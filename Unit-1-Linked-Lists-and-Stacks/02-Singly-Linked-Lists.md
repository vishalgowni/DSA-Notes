# Singly Linked Lists (SLL)
---

## Table of Contents
1. [Introduction](#1-introduction)
   - 1.1 [What Is a Singly Linked List?](#11-what-is-a-singly-linked-list)
   - 1.2 [Why Is It Needed and What Problem Does It Solve?](#12-why-is-it-needed-and-what-problem-does-it-solve)
   - 1.3 [Relationship to Prior Concepts](#13-relationship-to-prior-concepts)
   - 1.4 [Learning Objectives](#14-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Formal and Intuitive Definitions](#21-formal-and-intuitive-definitions)
   - 2.2 [Terminology Reference Table](#22-terminology-reference-table)
   - 2.3 [Mathematical Properties and Structural Invariants](#23-mathematical-properties-and-structural-invariants)
   - 2.4 [Conceptual Anatomy Diagram](#24-conceptual-anatomy-diagram)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Node Declaration and Optional List Descriptor (`head` and `tail`)](#31-node-declaration-and-optional-list-descriptor-head-and-tail)
   - 3.2 [Non-Contiguous Heap Memory Layout Diagram](#32-non-contiguous-heap-memory-layout-diagram)
   - 3.3 [Canonical List States: Empty, Single-Node, and Multi-Node](#33-canonical-list-states-empty-single-node-and-multi-node)
   - 3.4 [Dynamic Heap Allocation vs. Static Pool Representation](#34-dynamic-heap-allocation-vs-static-pool-representation)
   - 3.5 [Advantages, Limitations, and Real-World Applications](#35-advantages-limitations-and-real-world-applications)
4. [Operations and Algorithms](#4-operations-and-algorithms)
   - 4.1 [Insert at Beginning (Head Insertion)](#41-insert-at-beginning-head-insertion)
   - 4.2 [Insert at End (Tail Insertion)](#42-insert-at-end-tail-insertion)
   - 4.3 [Insert at a Specified Position (1-Based Indexing)](#43-insert-at-a-specified-position-1-based-indexing)
   - 4.4 [Insert Before and After a Specified Key/Value](#44-insert-before-and-after-a-specified-keyvalue)
   - 4.5 [Delete from Beginning (Head Deletion)](#45-delete-from-beginning-head-deletion)
   - 4.6 [Delete from End (Tail Deletion)](#46-delete-from-end-tail-deletion)
   - 4.7 [Delete from a Specified Position (1-Based Indexing)](#47-delete-from-a-specified-position-1-based-indexing)
   - 4.8 [Delete by Value/Key (First Occurrence and All Occurrences)](#48-delete-by-valuekey-first-occurrence-and-all-occurrences)
   - 4.9 [Searching, Traversal, Counting, and Updating Nodes](#49-searching-traversal-counting-and-updating-nodes)
   - 4.10 [Reversing a Singly Linked List (Iterative 3-Pointer & Recursive)](#410-reversing-a-singly-linked-list-iterative-3-pointer--recursive)
   - 4.11 [Complete C11 Demonstration Program](#411-complete-c11-demonstration-program)
5. [Complexity Analysis and Comparison Tables](#5-complexity-analysis-and-comparison-tables)
   - 5.1 [Time and Space Complexity of Singly Linked List Operations](#51-time-and-space-complexity-of-singly-linked-list-operations)
   - 5.2 [Comparison: Arrays vs. Dynamic Arrays vs. Singly Linked Lists](#52-comparison-arrays-vs-dynamic-arrays-vs-singly-linked-lists)
   - 5.3 [Comparison: Head-Only Representation vs. Head-and-Tail Representation](#53-comparison-head-only-representation-vs-head-and-tail-representation)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Broken Chain: Incorrect Order of Pointer Assignments on Insertion](#61-broken-chain-incorrect-order-of-pointer-assignments-on-insertion)
   - 6.2 [Memory Leak on Deletion (Unlinking Without Calling `free()`)](#62-memory-leak-on-deletion-unlinking-without-calling-free)
   - 6.3 [Null Dereference on Single-Node List in Tail Deletion](#63-null-dereference-on-single-node-list-in-tail-deletion)
   - 6.4 [Losing Head Updates When Deleting All Occurrences of a Key](#64-losing-head-updates-when-deleting-all-occurrences-of-a-key)
   - 6.5 [Infinite Cycle During Recursive Reversal](#65-infinite-cycle-during-recursive-reversal)
   - 6.6 [Off-by-One Traversal Errors (`curr != NULL` vs. `curr->next != NULL`)](#66-off-by-one-traversal-errors-curr--null-vs-curr-next--null)
   - 6.7 [Boundary Conditions Checklist](#67-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)
   - 7.1 [Key Takeaways](#71-key-takeaways)
   - 7.2 [Essential Singly Linked List Invariants](#72-essential-singly-linked-list-invariants)
   - 7.3 [Quick Self-Test Checklist](#73-quick-self-test-checklist)
   - 7.4 [Related Study Notes](#74-related-study-notes)

---

## 1. Introduction

### 1.1 What Is a Singly Linked List?
A **Singly Linked List (SLL)** is a dynamic, linear data structure composed of a sequence of discrete memory blocks called **nodes**. Each node stores two distinct components:
1. **`data`**: The user payload (an integer, structure, string pointer, etc.).
2. **`next`**: A unidirectional pointer storing the memory address of the logical successor node in the sequence, or `NULL` if the node is the last element.

An external pointer called **`head`** stores the address of the first node in the list. If the list contains zero nodes, `head == NULL`.

### 1.2 Why Is It Needed and What Problem Does It Solve?
Contiguous arrays (`int arr[N]`) excel at $\mathcal{O}(1)$ random indexing but suffer from severe structural bottlenecks when sequences undergo frequent insertions and deletions:
- **Costly Element Shifting**: Inserting or deleting an element at the front (`arr[0]`) of an $n$-element array requires shifting $n$ elements in memory, costing $\Theta(n)$ time.
- **Contiguous Allocation Requirement**: A dynamic array (`realloc`) of $100\text{ MB}$ fails if the heap does not have a single *contiguous* $100\text{ MB}$ slab, even if $500\text{ MB}$ of fragmented free blocks exist.
- **Resizing Overhead**: Expanding a full dynamic array requires allocating a larger buffer and copying all $n$ elements ($\mathcal{O}(n)$ worst-case latency spike).

A **Singly Linked List** solves these problems by decoupling logical order from physical memory layout:
- **$\mathcal{O}(1)$ Front Insertion and Deletion**: Prepending or removing the first element requires updating only 1–2 pointers—zero element shifting.
- **Fine-Grained On-Demand Allocation**: Nodes are allocated individually via `malloc(sizeof(Node))` wherever free space exists on the heap.

### 1.3 Relationship to Prior Concepts
In [01 — C Structures, Pointers, and Memory](./01-C-Structures-Pointers-and-Memory.md), we saw how self-referential structures (`struct Node { int data; struct Node *next; }`) and double pointers (`Node **head_ref`) work. Singly Linked Lists are the direct realization of self-referential structures managed systematically through insertion, deletion, search, and reversal algorithms. Furthermore, Singly Linked Lists serve as the underlying implementation engine for **Stacks**, **Queues**, **Hash Table Chaining**, and **Graph Adjacency Lists**.

### 1.4 Learning Objectives
After completing this module, you will be able to:
1. Represent Singly Linked Lists in C using `Node` structures and trace non-contiguous heap layouts.
2. Implement all four insertion variants (at head, at tail, at 1-based position $k$, and before/after a target key) with exact pointer diagrams.
3. Implement all four deletion variants (from head, from tail, at position $k$, and first/all occurrences of a key) without memory leaks or dangling pointers.
4. Perform linear search, counting, value updates, and both iterative ($\mathcal{O}(1)$ space) and recursive ($\mathcal{O}(n)$ stack space) list reversal.
5. Analyze best-, average-, and worst-case time and space complexities for every operation.

---

## 2. Definitions and Terminology

### 2.1 Formal and Intuitive Definitions
- **Formal Definition**: A Singly Linked List of length $n \ge 0$ over domain $D$ is an ordered sequence of nodes $L = \langle v_1, v_2, \dots, v_n \rangle$ where each node $v_i$ is a pair $(\text{data}_i, \text{next}_i)$ with $\text{data}_i \in D$, such that $\text{next}_i = \text{addr}(v_{i+1})$ for $1 \le i < n$, and $\text{next}_n = \text{NULL}$. An entry pointer $\text{head}$ satisfies $\text{head} = \text{addr}(v_1)$ if $n \ge 1$, and $\text{head} = \text{NULL}$ if $n = 0$.
- **Intuitive Analogy**: Think of a Singly Linked List as a **treasure hunt**: you hold only the clue to the first location (`head`). At each location, you find a prize (`data`) and the exact GPS coordinates of the *next* location (`next`), until the final location holds a note saying *"End of hunt"* (`NULL`). You can never walk backward unless you start over from the first clue.

### 2.2 Terminology Reference Table

| Term | Definition |
| :--- | :--- |
| **Node** | A self-referential heap-allocated structure containing `data` and a `next` pointer. |
| **Data / Payload** | The information stored inside a node (e.g., `int data`). |
| **Next Pointer (`next`)** | The link field storing the memory address of the successor node, or `NULL`. |
| **Head Pointer (`head`)** | The external pointer variable pointing to the first node in the list (`NULL` when empty). |
| **Tail Node** | The last node in a non-empty list, uniquely identified by `node->next == NULL`. |
| **Predecessor** | For node $v_i$ ($i > 1$), the node $v_{i-1}$ whose `next` pointer points to $v_i$ (`pred->next == curr`). |
| **Successor** | For node $v_i$ ($i < n$), the node $v_{i+1}$ pointed to by `v_i->next`. |
| **Sentinel / Dummy Node** | An optional permanent header node whose `data` is ignored and whose `next` points to the first real node, simplifying edge cases at the head. |
| **Traversal** | Sequentially visiting every node from `head` to `NULL` by repeatedly executing `curr = curr->next`. |

### 2.3 Mathematical Properties and Structural Invariants

#### 1. Acyclic Reachability Invariant
In a valid, non-circular Singly Linked List of $n$ nodes starting at `head`, applying the `.next` dereference operator $k$ times ($\text{next}^k(\text{head})$) yields:
- The $(k+1)$-th node $v_{k+1}$ for $0 \le k < n$.
- $\text{NULL}$ for $k = n$.

$$\text{next}^0(\text{head}) = v_1, \quad \text{next}^{n-1}(\text{head}) = v_n \text{ (tail)}, \quad \text{next}^n(\text{head}) = \text{NULL}$$

#### 2. Total Memory Footprint Formula
Let $S_{\text{data}} = \operatorname{sizeof}(\text{data})$, $S_{\text{ptr}} = \operatorname{sizeof}(\text{Node*})$, and $P_{\text{align}}$ be the internal alignment padding per node. The total heap memory $M(n)$ consumed by an $n$-node Singly Linked List (excluding allocator header metadata) is:

$$M(n) = n \times \operatorname{sizeof}(\text{Node}) = n \times \left(S_{\text{data}} + P_{\text{align}} + S_{\text{ptr}}\right)$$

On a 64-bit platform with `int data` ($4\text{ bytes}$), $P_{\text{align}} = 4\text{ bytes}$, and $S_{\text{ptr}} = 8\text{ bytes}$:

$$\operatorname{sizeof}(\text{Node}) = 16\text{ bytes}, \qquad \text{Pointer Overhead Ratio} = \frac{S_{\text{ptr}}}{\operatorname{sizeof}(\text{Node})} = \frac{8}{16} = 50\%$$

### 2.4 Conceptual Anatomy Diagram

```
  External Pointer        Node 1 (Head)         Node 2               Node 3 (Tail)
  +--------------+       +------+------+       +------+------+       +------+------+
  |     head     | ----> |  10  | next | ----> |  20  | next | ----> |  30  | NULL |
  +--------------+       +------+------+       +------+------+       +------+------+
  1-Based Position:          pos = 1               pos = 2               pos = 3
  0-Based Index   :          idx = 0               idx = 1               idx = 2
```

---

## 3. Representation and Memory Layout in C

### 3.1 Node Declaration and Optional List Descriptor (`head` and `tail`)
There are two standard idioms for representing a Singly Linked List in C:

```c
/* Idiom 1: Direct Head Pointer (Standard Academic & Interview Representation) */
typedef struct Node {
    int data;
    struct Node *next;
} Node;
/* An empty list is simply: Node *head = NULL; */

/* Idiom 2: List Descriptor Wrapper Maintaining Both Head and Tail + Size */
typedef struct SinglyLinkedList {
    Node *head;     /* Points to first node (or NULL if empty) */
    Node *tail;     /* Points to last node  (or NULL if empty) */
    size_t size;    /* Cached node count for O(1) length query */
} SinglyLinkedList;
```

### 3.2 Non-Contiguous Heap Memory Layout Diagram
Unlike arrays, consecutively linked nodes can reside at arbitrary, non-adjacent heap addresses (even decreasing addresses depending on heap free-list state):

```
STACK SEGMENT                     HEAP SEGMENT (Non-Contiguous Addresses)
+-------------------+
| head (0x7ffc0010) |
|   [ 0x55a04080 ]  | --------->  Node 1 @ 0x55a04080
+-------------------+            +------------+------------------+
                                 | data: 10   | next: 0x55a01020 |
                                 +------------+--------+---------+
                                                       |
                 +-------------------------------------+
                 v
                 Node 2 @ 0x55a01020 (Lower heap address than Node 1!)
                +------------+------------------+
                | data: 20   | next: 0x55a09000 |
                +------------+--------+---------+
                                      |
                                      v
                                      Node 3 (Tail) @ 0x55a09000
                                     +------------+------------------+
                                     | data: 30   | next: NULL (0x0) |
                                     +------------+------------------+
```

### 3.3 Canonical List States: Empty, Single-Node, and Multi-Node
Every Singly Linked List algorithm must correctly handle three fundamental structural configurations:

```
STATE 1: Empty List (n = 0)
  head (and tail, if used)
  +------+
  | NULL |
  +------+

STATE 2: Single-Node List (n = 1 — Head and Tail are the SAME node!)
  head ---------+
                v
             +------+------+
             |  42  | NULL |
             +------+------+
                ^
  tail ---------+

STATE 3: Multi-Node List (n >= 2)
  head                                       tail
   |                                          |
   v                                          v
  +------+------+     +------+------+     +------+------+
  |  10  | next | --> |  20  | next | --> |  30  | NULL |
  +------+------+     +------+------+     +------+------+
```

### 3.4 Dynamic Heap Allocation vs. Static Pool Representation
1. **Dynamic Heap Allocation (`malloc`/`free`)**: The standard approach where each node is allocated on the heap via `malloc(sizeof(Node))` and released via `free(node)`.
2. **Static Array-Pool (Cursor-Based Linked List)**: In safety-critical embedded systems (e.g., MISRA-C automotive firmware) where `malloc()` is banned due to fragmentation and non-deterministic latency, a fixed array `Node pool[MAX_NODES]` is used, and `int next` stores the array index of the next node (`-1` for `NULL`).

### 3.5 Advantages, Limitations, and Real-World Applications
- **Advantages**:
  - True $\mathcal{O}(1)$ insertion and deletion at the head.
  - Dynamic size without contiguous memory requirements or `realloc` copy spikes.
  - Lower per-node memory overhead ($1\text{ pointer}$ per node) compared to Doubly Linked Lists ($2\text{ pointers}$ per node).
- **Limitations**:
  - **No $\mathcal{O}(1)$ Random Access**: Accessing the $k$-th element requires $\mathcal{O}(k)$ sequential pointer hops.
  - **Unidirectional Traversal Only**: Even if given a direct pointer to the tail node, deleting the tail still requires $\mathcal{O}(n)$ time to find its predecessor.
  - **Poor CPU Cache Locality**: Pointer chasing across scattered heap addresses causes frequent L1/L2 CPU cache misses compared to contiguous arrays.
- **Real-World Applications**:
  - **Separate Chaining in Hash Tables** (collision resolution buckets).
  - **Dynamic Stacks and FIFO Queues** (with head + tail pointers).
  - **Graph Adjacency Lists** (storing outgoing edges per vertex).
  - **OS Free-Memory Lists** and **Slab Allocators** (chaining free memory blocks).
  - **Sparse Polynomial Arithmetic** (each node stores non-zero `(coefficient, exponent)`).

---

## 4. Operations and Algorithms

> [!NOTE]
> **Position Indexing Convention**: Throughout Sections 4.3 and 4.7, we use **1-based position indexing** (`pos = 1` refers to the `head` node, `pos = n` refers to the last node). To convert any algorithm to **0-based indexing**, simply replace `pos == 1` with `idx == 0` and loop `idx - 1` times instead of `pos - 2` times.

---

### 4.1 Insert at Beginning (Head Insertion)

#### 1. Purpose
Allocate a new node with `data = value` and place it at the very front of the list so it becomes the new `head` in $\mathcal{O}(1)$ time.

#### 2. Preconditions and Assumptions
- `head_ref` (`Node**`) is a non-NULL address of the caller's `head` pointer (`*head_ref` itself may be `NULL` if the list is currently empty).

#### 3. Step-by-Step Algorithm
1. Allocate a new node `new_node` on the heap via `malloc(sizeof(*new_node))`. If `NULL`, return `false`.
2. Set `new_node->data = value`.
3. Link `new_node->next = *head_ref` (points to old head, or `NULL` if list was empty).
4. Update `*head_ref = new_node` so `head` now points to the newly inserted node.

#### 4. Pseudocode
```
ALGORITHM InsertAtBeginning(head_ref, value)
INPUT: head_ref (Node**), value (int)
OUTPUT: true on success, false on allocation failure
BEGIN
    new_node <- malloc(sizeof(Node))
    IF new_node == NULL THEN
        RETURN false
    END IF
    new_node->data <- value
    new_node->next <- *head_ref
    *head_ref <- new_node
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
BEFORE insert_at_beginning(&head, 5):
  head
   |
   v
  +------+------+     +------+------+
  |  10  | next | --> |  20  | NULL |
  +------+------+     +------+------+

STEP 1 & 2: Allocate new_node (5) and wire new_node->next = *head_ref
  new_node            head
   |                   |
   v                   v
  +------+------+     +------+------+     +------+------+
  |   5  | next | --> |  10  | next | --> |  20  | NULL |
  +------+------+     +------+------+     +------+------+

AFTER (*head_ref = new_node):
  head
   |
   v
  +------+------+     +------+------+     +------+------+
  |   5  | next | --> |  10  | next | --> |  20  | NULL |
  +------+------+     +------+------+     +------+------+
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct Node {
    int data;
    struct Node *next;
} Node;

bool insert_at_beginning(Node **head_ref, int value) {
    if (head_ref == NULL) {
        return false;
    }
    Node *new_node = malloc(sizeof(*new_node));
    if (new_node == NULL) {
        return false;
    }
    new_node->data = value;
    new_node->next = *head_ref; /* Step 1: Point new node to old head */
    *head_ref = new_node;       /* Step 2: Update head to new node    */
    return true;
}

void print_list(const Node *head) {
    for (const Node *curr = head; curr != NULL; curr = curr->next) {
        printf("[%d] -> ", curr->data);
    }
    printf("NULL\n");
}

void free_list(Node **head_ref) {
    Node *curr = *head_ref;
    while (curr != NULL) {
        Node *next = curr->next;
        free(curr);
        curr = next;
    }
    *head_ref = NULL;
}

int main(void) {
    Node *head = NULL;
    insert_at_beginning(&head, 20);
    insert_at_beginning(&head, 10);
    insert_at_beginning(&head, 5);
    print_list(head);
    free_list(&head);
    return 0;
}
```

#### Program Output:
```
[5] -> [10] -> [20] -> NULL
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: $\mathcal{O}(1)$ — Always executes two pointer assignments regardless of list length $n$.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$ auxiliary stack space; $\mathcal{O}(1)$ new heap node.

#### 8. Edge Cases and Common Mistakes
- **Empty list (`*head_ref == NULL`)**: Handled seamlessly without an `if` branch because `new_node->next = *head_ref` sets `new_node->next = NULL`.
- **Mistake**: Assigning `*head_ref = new_node` *before* `new_node->next = *head_ref` overwrites `*head_ref`, causing `new_node->next` to point to itself (self-loop) and leaking the entire existing list!

---

### 4.2 Insert at End (Tail Insertion)

#### 1. Purpose
Append a newly allocated node containing `value` at the end of the Singly Linked List.

#### 2. Preconditions and Assumptions
- `head_ref` (`Node**`) is non-NULL.

#### 3. Step-by-Step Algorithm
1. Allocate `new_node`, set `new_node->data = value` and `new_node->next = NULL`.
2. **Empty List Check**: If `*head_ref == NULL`, set `*head_ref = new_node` and return `true`.
3. Otherwise, initialize `Node *curr = *head_ref` and advance `curr = curr->next` while `curr->next != NULL` to stop at the current tail node.
4. Set `curr->next = new_node`.

#### 4. Pseudocode
```
ALGORITHM InsertAtEnd(head_ref, value)
INPUT: head_ref (Node**), value (int)
OUTPUT: true on success, false on allocation failure
BEGIN
    new_node <- malloc(sizeof(Node))
    IF new_node == NULL THEN
        RETURN false
    END IF
    new_node->data <- value
    new_node->next <- NULL

    IF *head_ref == NULL THEN
        *head_ref <- new_node
        RETURN true
    END IF

    curr <- *head_ref
    WHILE curr->next != NULL DO
        curr <- curr->next
    END WHILE
    curr->next <- new_node
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
BEFORE insert_at_end(&head, 30):
  head                curr (stops where curr->next == NULL)
   |                   |
   v                   v
  +------+------+     +------+------+       +------+------+
  |  10  | next | --> |  20  | NULL |       |  30  | NULL | (new_node)
  +------+------+     +------+------+       +------+------+

AFTER (curr->next = new_node):
  head
   |
   v
  +------+------+     +------+------+     +------+------+
  |  10  | next | --> |  20  | next | --> |  30  | NULL |
  +------+------+     +------+------+     +------+------+
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct Node {
    int data;
    struct Node *next;
} Node;

bool insert_at_end(Node **head_ref, int value) {
    if (head_ref == NULL) {
        return false;
    }
    Node *new_node = malloc(sizeof(*new_node));
    if (new_node == NULL) {
        return false;
    }
    new_node->data = value;
    new_node->next = NULL;

    if (*head_ref == NULL) {
        *head_ref = new_node;
        return true;
    }

    Node *curr = *head_ref;
    while (curr->next != NULL) {
        curr = curr->next;
    }
    curr->next = new_node;
    return true;
}

int main(void) {
    Node *head = NULL;
    insert_at_end(&head, 10);
    insert_at_end(&head, 20);
    insert_at_end(&head, 30);

    for (Node *c = head; c != NULL; c = c->next) {
        printf("[%d] -> ", c->data);
    }
    printf("NULL\n");

    while (head != NULL) {
        Node *tmp = head;
        head = head->next;
        free(tmp);
    }
    return 0;
}
```

#### Program Output:
```
[10] -> [20] -> [30] -> NULL
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**:
  - Best Case (Empty list, $n = 0$): $\mathcal{O}(1)$.
  - Average & Worst Case ($n \ge 1$, without tail pointer): $\Theta(n)$ to traverse $n$ nodes.
  - With an explicit `tail` pointer maintained in a wrapper struct: $\mathcal{O}(1)$ in all cases.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- Writing `while (curr != NULL)` instead of `while (curr->next != NULL)` advances `curr` *past* the last node until `curr == NULL`, and then `curr = new_node` only modifies the local variable `curr` while leaving the list unchanged and leaking `new_node`!

---

### 4.3 Insert at a Specified Position (1-Based Indexing)

#### 1. Purpose
Insert a new node with `value` at 1-based position `pos` ($1 \le \text{pos} \le n + 1$), shifting existing nodes from position `pos` onward one position to the right.

#### 2. Preconditions and Assumptions
- **1-Based Convention**: `pos == 1` inserts at the head; `pos == n + 1` appends at the tail; any `pos < 1` or `pos > n + 1` is rejected as out-of-bounds.

#### 3. Step-by-Step Algorithm
1. If `head_ref == NULL` or `pos < 1`, return `false`.
2. If `pos == 1`, allocate `new_node`, set `new_node->next = *head_ref`, update `*head_ref = new_node`, and return `true`.
3. Otherwise, start `Node *prev = *head_ref` at position 1 and advance `pos - 2` steps (stopping at position `pos - 1`) while `prev != NULL`.
4. If `prev == NULL` (meaning `pos > n + 1`), return `false` without allocating memory.
5. Allocate `new_node`, set `new_node->data = value`, wire `new_node->next = prev->next`, and then wire `prev->next = new_node`.

#### 4. Pseudocode
```
ALGORITHM InsertAtPosition(head_ref, pos, value)
INPUT: head_ref (Node**), pos (int, 1-based), value (int)
OUTPUT: true if inserted, false if pos is out of range or malloc fails
BEGIN
    IF head_ref == NULL OR pos < 1 THEN
        RETURN false
    END IF
    IF pos == 1 THEN
        RETURN InsertAtBeginning(head_ref, value)
    END IF

    prev <- *head_ref
    FOR step <- 1 TO pos - 2 DO
        IF prev == NULL THEN
            RETURN false
        END IF
        prev <- prev->next
    END FOR

    IF prev == NULL THEN
        RETURN false
    END IF

    new_node <- malloc(sizeof(Node))
    IF new_node == NULL THEN
        RETURN false
    END IF
    new_node->data <- value
    new_node->next <- prev->next
    prev->next <- new_node
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
Inserting 25 at pos = 3 into [10 -> 20 -> 30]:
Stop at predecessor prev at pos - 1 = 2 (node 20):

  pos=1               pos=2 (prev)                pos=3 (prev->next)
  +------+------+     +------+------+             +------+------+
  |  10  | next | --> |  20  | next | ----------> |  30  | NULL |
  +------+------+     +------+------+             +------+------+
                         |                           ^
                         | (2) prev->next = new_node | (1) new_node->next = prev->next
                         v                           |
                      +------+------+----------------+
                      |  25  | next | (new_node)
                      +------+------+

AFTER:
  pos=1               pos=2               pos=3               pos=4
  +------+------+     +------+------+     +------+------+     +------+------+
  |  10  | next | --> |  20  | next | --> |  25  | next | --> |  30  | NULL |
  +------+------+     +------+------+     +------+------+     +------+------+
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct Node {
    int data;
    struct Node *next;
} Node;

bool insert_at_position(Node **head_ref, int pos, int value) {
    if (head_ref == NULL || pos < 1) {
        return false;
    }
    if (pos == 1) {
        Node *new_node = malloc(sizeof(*new_node));
        if (new_node == NULL) return false;
        new_node->data = value;
        new_node->next = *head_ref;
        *head_ref = new_node;
        return true;
    }

    Node *prev = *head_ref;
    for (int i = 1; i < pos - 1 && prev != NULL; i++) {
        prev = prev->next;
    }
    if (prev == NULL) {
        return false; /* Position exceeds list length + 1 */
    }

    Node *new_node = malloc(sizeof(*new_node));
    if (new_node == NULL) {
        return false;
    }
    new_node->data = value;
    new_node->next = prev->next; /* Step 1: Connect new_node to successor */
    prev->next = new_node;       /* Step 2: Connect predecessor to new_node */
    return true;
}

int main(void) {
    Node *head = NULL;
    insert_at_position(&head, 1, 10); /* [10] */
    insert_at_position(&head, 2, 30); /* [10 -> 30] */
    insert_at_position(&head, 2, 20); /* [10 -> 20 -> 30] */
    insert_at_position(&head, 3, 25); /* [10 -> 20 -> 25 -> 30] */

    printf("Valid insertions  : ");
    for (Node *c = head; c != NULL; c = c->next) printf("[%d] -> ", c->data);
    printf("NULL\n");

    bool ok = insert_at_position(&head, 10, 99);
    printf("Insert at pos=10  : %s\n", ok ? "Success" : "Rejected (Out of bounds)");

    while (head != NULL) { Node *t = head; head = head->next; free(t); }
    return 0;
}
```

#### Program Output:
```
Valid insertions  : [10] -> [20] -> [25] -> [30] -> NULL
Insert at pos=10  : Rejected (Out of bounds)
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: Best Case $\mathcal{O}(1)$ (`pos == 1`); Average/Worst Case $\mathcal{O}(n)$ (traversing up to `pos - 1` nodes).
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- Allocating `new_node` *before* verifying that `pos` is within bounds leaks `new_node` if `prev == NULL` returns early. Always validate `prev != NULL` first or `free(new_node)` before returning `false`.

---

### 4.4 Insert Before and After a Specified Key/Value

#### 1. Purpose
- **Insert After Key**: Locate the first node containing `target_key` and insert `new_value` immediately after it.
- **Insert Before Key**: Locate the first node containing `target_key` (tracking its predecessor) and insert `new_value` immediately before it.

#### 2. Preconditions and Assumptions
- List must be non-empty (`*head_ref != NULL`), and `target_key` must exist in the list for the insertion to succeed.

#### 3. Step-by-Step Algorithm
- **Insert After Key (`target_key`)**:
  1. Search from `curr = head` until `curr == NULL` or `curr->data == target_key`.
  2. If `curr == NULL`, return `false` (key not found).
  3. Allocate `new_node(new_value)`, set `new_node->next = curr->next`, and `curr->next = new_node`.
- **Insert Before Key (`target_key`)**:
  1. If `*head_ref == NULL`, return `false`.
  2. If `(*head_ref)->data == target_key`, prepend at the head (`new_node->next = *head_ref; *head_ref = new_node;`).
  3. Otherwise, traverse with `prev = *head_ref` while `prev->next != NULL && prev->next->data != target_key`.
  4. If `prev->next == NULL`, return `false` (key not found).
  5. Allocate `new_node(new_value)`, set `new_node->next = prev->next`, and `prev->next = new_node`.

#### 4. Pseudocode
```
ALGORITHM InsertBeforeKey(head_ref, target_key, new_value)
INPUT: head_ref (Node**), target_key (int), new_value (int)
OUTPUT: true if target_key found and new_value inserted, false otherwise
BEGIN
    IF head_ref == NULL OR *head_ref == NULL THEN
        RETURN false
    END IF
    IF (*head_ref)->data == target_key THEN
        RETURN InsertAtBeginning(head_ref, new_value)
    END IF

    prev <- *head_ref
    WHILE prev->next != NULL AND prev->next->data != target_key DO
        prev <- prev->next
    END WHILE

    IF prev->next == NULL THEN
        RETURN false
    END IF

    new_node <- malloc(sizeof(Node))
    IF new_node == NULL THEN RETURN false END IF
    new_node->data <- new_value
    new_node->next <- prev->next
    prev->next <- new_node
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
1. INSERT BEFORE KEY (target_key = 20, new_value = 15):
   prev stops at node [10] where prev->next->data == 20:
   [10 (prev)] ---> [15 (new)] ---> [20 (target)] ---> [30] ---> NULL

2. INSERT AFTER KEY (target_key = 20, new_value = 25):
   curr stops directly on node [20] where curr->data == 20:
   [10] ---> [15] ---> [20 (curr)] ---> [25 (new)] ---> [30] ---> NULL
```

#### 6. Reusable C Functions and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct Node {
    int data;
    struct Node *next;
} Node;

bool insert_after_key(Node *head, int target_key, int new_value) {
    Node *curr = head;
    while (curr != NULL && curr->data != target_key) {
        curr = curr->next;
    }
    if (curr == NULL) {
        return false; /* Key not found */
    }
    Node *new_node = malloc(sizeof(*new_node));
    if (new_node == NULL) {
        return false;
    }
    new_node->data = new_value;
    new_node->next = curr->next;
    curr->next = new_node;
    return true;
}

bool insert_before_key(Node **head_ref, int target_key, int new_value) {
    if (head_ref == NULL || *head_ref == NULL) {
        return false;
    }
    if ((*head_ref)->data == target_key) {
        Node *new_node = malloc(sizeof(*new_node));
        if (new_node == NULL) return false;
        new_node->data = new_value;
        new_node->next = *head_ref;
        *head_ref = new_node;
        return true;
    }
    Node *prev = *head_ref;
    while (prev->next != NULL && prev->next->data != target_key) {
        prev = prev->next;
    }
    if (prev->next == NULL) {
        return false; /* Key not found */
    }
    Node *new_node = malloc(sizeof(*new_node));
    if (new_node == NULL) {
        return false;
    }
    new_node->data = new_value;
    new_node->next = prev->next;
    prev->next = new_node;
    return true;
}

int main(void) {
    Node *head = NULL;
    /* Seed list with [10 -> 20] */
    Node *n1 = malloc(sizeof(*n1)), *n2 = malloc(sizeof(*n2));
    n1->data = 10; n1->next = n2;
    n2->data = 20; n2->next = NULL;
    head = n1;

    insert_before_key(&head, 10, 5);  /* Before head: [5 -> 10 -> 20] */
    insert_before_key(&head, 20, 15); /* Before 20  : [5 -> 10 -> 15 -> 20] */
    insert_after_key(head, 20, 25);   /* After 20   : [5 -> 10 -> 15 -> 20 -> 25] */

    for (Node *c = head; c != NULL; c = c->next) printf("[%d] -> ", c->data);
    printf("NULL\n");

    while (head != NULL) { Node *t = head; head = head->next; free(t); }
    return 0;
}
```

#### Program Output:
```
[5] -> [10] -> [15] -> [20] -> [25] -> NULL
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: $\mathcal{O}(n)$ to search for `target_key` (Note: if a direct pointer `Node *target` is already provided, `insert_after_node` is $\mathcal{O}(1)$, whereas `insert_before_node` still requires $\mathcal{O}(n)$ in an SLL to find the predecessor!).
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- When `target_key` is at the `head` node, `insert_before_key` must update `*head_ref` itself because the `head` node has no predecessor node.

---

### 4.5 Delete from Beginning (Head Deletion)

#### 1. Purpose
Remove and deallocate the first node of the list in $\mathcal{O}(1)$ time, advancing `head` to the second node (or `NULL` if the list had only 1 node).

#### 2. Preconditions and Assumptions
- `head_ref` is non-NULL and `*head_ref != NULL` (list is non-empty).

#### 3. Step-by-Step Algorithm
1. Check if `head_ref == NULL || *head_ref == NULL`; if so, return `false`.
2. Save a temporary pointer to the current head: `Node *to_delete = *head_ref`.
3. Optionally store `to_delete->data` into `*removed_out` if `removed_out != NULL`.
4. Advance the head pointer: `*head_ref = to_delete->next`.
5. Call `free(to_delete)` and return `true`.

#### 4. Pseudocode
```
ALGORITHM DeleteFromBeginning(head_ref, removed_out)
INPUT: head_ref (Node**), removed_out (int* optional)
OUTPUT: true if deleted, false if list was empty
BEGIN
    IF head_ref == NULL OR *head_ref == NULL THEN
        RETURN false
    END IF
    to_delete <- *head_ref
    IF removed_out != NULL THEN
        *removed_out <- to_delete->data
    END IF
    *head_ref <- to_delete->next
    free(to_delete)
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
BEFORE delete_from_beginning(&head, &val):
  head (and to_delete)
   |
   v
  +------+------+     +------+------+     +------+------+
  |  10  | next | --> |  20  | next | --> |  30  | NULL |
  +------+------+     +------+------+     +------+------+

AFTER (*head_ref = to_delete->next; free(to_delete)):
                      head
                       |
                       v
  (freed: [10])       +------+------+     +------+------+
                      |  20  | next | --> |  30  | NULL |
                      +------+------+     +------+------+
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct Node {
    int data;
    struct Node *next;
} Node;

bool delete_from_beginning(Node **head_ref, int *removed_out) {
    if (head_ref == NULL || *head_ref == NULL) {
        return false;
    }
    Node *to_delete = *head_ref;
    if (removed_out != NULL) {
        *removed_out = to_delete->data;
    }
    *head_ref = to_delete->next;
    free(to_delete);
    return true;
}

int main(void) {
    Node *n1 = malloc(sizeof(*n1)), *n2 = malloc(sizeof(*n2));
    n1->data = 10; n1->next = n2;
    n2->data = 20; n2->next = NULL;
    Node *head = n1;

    int popped = 0;
    while (delete_from_beginning(&head, &popped)) {
        printf("Deleted from head: %d | Remaining head: %s\n",
               popped, (head != NULL) ? "non-NULL" : "NULL");
    }
    return 0;
}
```

#### Program Output:
```
Deleted from head: 10 | Remaining head: non-NULL
Deleted from head: 20 | Remaining head: NULL
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: $\mathcal{O}(1)$.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- **Single-node list (`(*head_ref)->next == NULL`)**: Automatically sets `*head_ref = NULL`, leaving a valid empty list.

---

### 4.6 Delete from End (Tail Deletion)

#### 1. Purpose
Remove and deallocate the last node (tail) of the Singly Linked List and set the new tail's `next` pointer to `NULL`.

#### 2. Preconditions and Assumptions
- List is non-empty (`*head_ref != NULL`).

#### 3. Step-by-Step Algorithm
1. If `head_ref == NULL || *head_ref == NULL`, return `false`.
2. **Single-Node Special Case**: If `(*head_ref)->next == NULL`, save `*removed_out = (*head_ref)->data`, `free(*head_ref)`, set `*head_ref = NULL`, and return `true`.
3. **Multi-Node Case ($n \ge 2$)**: Initialize `Node *second_last = *head_ref` and advance while `second_last->next->next != NULL`.
4. Now `second_last->next` is the tail node. Save its data, `free(second_last->next)`, and set `second_last->next = NULL`.

#### 4. Pseudocode
```
ALGORITHM DeleteFromEnd(head_ref, removed_out)
INPUT: head_ref (Node**), removed_out (int* optional)
OUTPUT: true if deleted, false if list was empty
BEGIN
    IF head_ref == NULL OR *head_ref == NULL THEN
        RETURN false
    END IF

    /* Case 1: Single-node list */
    IF (*head_ref)->next == NULL THEN
        IF removed_out != NULL THEN *removed_out <- (*head_ref)->data END IF
        free(*head_ref)
        *head_ref <- NULL
        RETURN true
    END IF

    /* Case 2: 2 or more nodes — stop at second-to-last node */
    second_last <- *head_ref
    WHILE second_last->next->next != NULL DO
        second_last <- second_last->next
    END WHILE

    tail_node <- second_last->next
    IF removed_out != NULL THEN *removed_out <- tail_node->data END IF
    free(tail_node)
    second_last->next <- NULL
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
BEFORE delete_from_end(&head, &val):
  head                second_last         tail_node (second_last->next)
   |                   |                   |
   v                   v                   v
  +------+------+     +------+------+     +------+------+
  |  10  | next | --> |  20  | next | --> |  30  | NULL |
  +------+------+     +------+------+     +------+------+

AFTER (free(tail_node); second_last->next = NULL):
  head                second_last (New Tail)
   |                   |
   v                   v
  +------+------+     +------+------+
  |  10  | next | --> |  20  | NULL |     (freed: [30])
  +------+------+     +------+------+
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct Node {
    int data;
    struct Node *next;
} Node;

bool delete_from_end(Node **head_ref, int *removed_out) {
    if (head_ref == NULL || *head_ref == NULL) {
        return false;
    }
    if ((*head_ref)->next == NULL) {
        if (removed_out != NULL) {
            *removed_out = (*head_ref)->data;
        }
        free(*head_ref);
        *head_ref = NULL;
        return true;
    }
    Node *second_last = *head_ref;
    while (second_last->next->next != NULL) {
        second_last = second_last->next;
    }
    Node *tail = second_last->next;
    if (removed_out != NULL) {
        *removed_out = tail->data;
    }
    free(tail);
    second_last->next = NULL;
    return true;
}

int main(void) {
    Node *n1 = malloc(sizeof(*n1)), *n2 = malloc(sizeof(*n2));
    n1->data = 100; n1->next = n2;
    n2->data = 200; n2->next = NULL;
    Node *head = n1;

    int val = 0;
    delete_from_end(&head, &val);
    printf("1st tail delete removed: %d | New tail data: %d\n", val, head->data);
    delete_from_end(&head, &val);
    printf("2nd tail delete removed: %d | head == %s\n",
           val, (head == NULL) ? "NULL" : "non-NULL");
    return 0;
}
```

#### Program Output:
```
1st tail delete removed: 200 | New tail data: 100
2nd tail delete removed: 100 | head == NULL
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: $\Theta(n)$ — Even if a `tail` pointer is maintained, a Singly Linked List has no `prev` pointer, so finding `second_last` always requires traversing $n - 1$ nodes!
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- **Forgetting the single-node check (`(*head_ref)->next == NULL`)**: Evaluating `second_last->next->next` when `second_last->next == NULL` immediately causes a NULL-pointer dereference crash.

---

### 4.7 Delete from a Specified Position (1-Based Indexing)

#### 1. Purpose
Delete and free the node at 1-based position `pos` ($1 \le \text{pos} \le n$).

#### 2. Preconditions and Assumptions
- `pos >= 1` and list is non-empty (`*head_ref != NULL`).

#### 3. Step-by-Step Algorithm
1. If `head_ref == NULL || *head_ref == NULL || pos < 1`, return `false`.
2. If `pos == 1`, unlink `*head_ref`, advance `*head_ref = (*head_ref)->next`, `free` the old head, and return `true`.
3. Otherwise, traverse `prev` from `*head_ref` for `pos - 2` steps while `prev != NULL`.
4. If `prev == NULL || prev->next == NULL` (meaning no node exists at position `pos`), return `false`.
5. Let `Node *target = prev->next`. Bypass `target` via `prev->next = target->next`, `free(target)`, and return `true`.

#### 4. Pseudocode
```
ALGORITHM DeleteAtPosition(head_ref, pos, removed_out)
INPUT: head_ref (Node**), pos (int, 1-based), removed_out (int* optional)
OUTPUT: true if deleted, false if pos is out of range
BEGIN
    IF head_ref == NULL OR *head_ref == NULL OR pos < 1 THEN
        RETURN false
    END IF
    IF pos == 1 THEN
        RETURN DeleteFromBeginning(head_ref, removed_out)
    END IF

    prev <- *head_ref
    FOR i <- 1 TO pos - 2 DO
        IF prev == NULL THEN RETURN false END IF
        prev <- prev->next
    END FOR

    IF prev == NULL OR prev->next == NULL THEN
        RETURN false
    END IF

    target <- prev->next
    IF removed_out != NULL THEN *removed_out <- target->data END IF
    prev->next <- target->next
    free(target)
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
Deleting pos = 2 from [10 -> 20 -> 30]:
  pos=1 (prev)        pos=2 (target)      pos=3 (target->next)
  +------+------+     +------+------+     +------+------+
  |  10  | next | --> |  20  | next | --> |  30  | NULL |
  +------+------+     +------+------+     +------+------+
            |                               ^
            +-------------------------------+
               prev->next = target->next;
               free(target);
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct Node {
    int data;
    struct Node *next;
} Node;

bool delete_at_position(Node **head_ref, int pos, int *removed_out) {
    if (head_ref == NULL || *head_ref == NULL || pos < 1) {
        return false;
    }
    if (pos == 1) {
        Node *to_del = *head_ref;
        if (removed_out != NULL) *removed_out = to_del->data;
        *head_ref = to_del->next;
        free(to_del);
        return true;
    }
    Node *prev = *head_ref;
    for (int i = 1; i < pos - 1 && prev != NULL; i++) {
        prev = prev->next;
    }
    if (prev == NULL || prev->next == NULL) {
        return false;
    }
    Node *target = prev->next;
    if (removed_out != NULL) {
        *removed_out = target->data;
    }
    prev->next = target->next;
    free(target);
    return true;
}

int main(void) {
    Node *n1 = malloc(sizeof(*n1)), *n2 = malloc(sizeof(*n2)), *n3 = malloc(sizeof(*n3));
    n1->data = 10; n1->next = n2;
    n2->data = 20; n2->next = n3;
    n3->data = 30; n3->next = NULL;
    Node *head = n1;

    int val = 0;
    delete_at_position(&head, 2, &val);
    printf("Deleted pos 2 (value=%d). Remaining: ", val);
    for (Node *c = head; c != NULL; c = c->next) printf("[%d] -> ", c->data);
    printf("NULL\n");

    while (head != NULL) { Node *t = head; head = head->next; free(t); }
    return 0;
}
```

#### Program Output:
```
Deleted pos 2 (value=20). Remaining: [10] -> [30] -> NULL
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: Best Case $\mathcal{O}(1)$ (`pos == 1`); Average/Worst Case $\mathcal{O}(n)$.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- Failing to check `prev->next == NULL` when `pos == n + 1`: `prev` points to the tail node (`prev != NULL`), so `prev->next` is `NULL`, and `target->next` would crash!

---

### 4.8 Delete by Value/Key (First Occurrence and All Occurrences)

#### 1. Purpose
- **Delete First Occurrence**: Find the first node with `data == key`, unlink and `free` it, and return `true`.
- **Delete All Occurrences**: Remove and `free` *every* node with `data == key` across the entire list, returning the count of deleted nodes.

#### 2. Preconditions and Assumptions
- `head_ref` is non-NULL.

#### 3. Step-by-Step Algorithm (Pointer-to-Pointer `Node **indirect` Idiom)
Using Linus Torvalds's classic **indirect pointer (`Node **curr_pp = head_ref`)** technique unifies head deletion and interior deletion without a separate special-case loop:
1. Initialize `Node **indirect = head_ref`.
2. While `*indirect != NULL`:
   - If `(*indirect)->data == key`:
     - Save `Node *victim = *indirect`.
     - Bypass `victim` by updating `*indirect = victim->next` (if `indirect == head_ref`, this updates `head`; otherwise it updates `prev->next`!).
     - Call `free(victim)`.
     - For *first occurrence*, return `true` immediately. For *all occurrences*, increment `deleted_count` and **do not advance `indirect`** (since `*indirect` now holds the next node to inspect).
   - Else (`(*indirect)->data != key`):
     - Advance `indirect = &((*indirect)->next)`.

#### 4. Pseudocode
```
ALGORITHM DeleteAllByKey(head_ref, key)
INPUT: head_ref (Node**), key (int)
OUTPUT: count of deleted nodes (size_t)
BEGIN
    IF head_ref == NULL THEN RETURN 0 END IF
    count <- 0
    indirect <- head_ref
    WHILE *indirect != NULL DO
        IF (*indirect)->data == key THEN
            victim <- *indirect
            *indirect <- victim->next
            free(victim)
            count <- count + 1
        ELSE
            indirect <- &((*indirect)->next)
        END IF
    END WHILE
    RETURN count
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
Deleting ALL occurrences of key = 7 from [7 -> 10 -> 7 -> 20]:
1. At head: indirect == &head, *indirect == [7].
   *indirect = victim->next updates head directly to [10]; frees first [7].
2. At [10]: (*indirect)->data != 7, so indirect advances to &(node10->next).
3. At second [7]: *indirect == [7].
   *indirect = victim->next updates node10->next to [20]; frees second [7].

Result: head ---> [10] ---> [20] ---> NULL (2 nodes freed)
```

#### 6. Reusable C Functions and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct Node {
    int data;
    struct Node *next;
} Node;

bool delete_first_by_key(Node **head_ref, int key) {
    if (head_ref == NULL) {
        return false;
    }
    Node **indirect = head_ref;
    while (*indirect != NULL) {
        if ((*indirect)->data == key) {
            Node *victim = *indirect;
            *indirect = victim->next;
            free(victim);
            return true;
        }
        indirect = &((*indirect)->next);
    }
    return false;
}

size_t delete_all_by_key(Node **head_ref, int key) {
    if (head_ref == NULL) {
        return 0;
    }
    size_t count = 0;
    Node **indirect = head_ref;
    while (*indirect != NULL) {
        if ((*indirect)->data == key) {
            Node *victim = *indirect;
            *indirect = victim->next;
            free(victim);
            count++;
        } else {
            indirect = &((*indirect)->next);
        }
    }
    return count;
}

void push_back(Node **head_ref, int val) {
    Node **p = head_ref;
    while (*p != NULL) p = &((*p)->next);
    *p = malloc(sizeof(**p));
    (*p)->data = val;
    (*p)->next = NULL;
}

int main(void) {
    Node *head = NULL;
    int vals[] = {7, 7, 10, 7, 20, 7};
    for (size_t i = 0; i < 6; i++) push_back(&head, vals[i]);

    delete_first_by_key(&head, 10);
    size_t removed_sevens = delete_all_by_key(&head, 7);

    printf("Removed %zu occurrences of 7. Remaining list: ", removed_sevens);
    for (Node *c = head; c != NULL; c = c->next) printf("[%d] -> ", c->data);
    printf("NULL\n");

    while (head != NULL) { Node *t = head; head = head->next; free(t); }
    return 0;
}
```

#### Program Output:
```
Removed 4 occurrences of 7. Remaining list: [20] -> NULL
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**:
  - `delete_first_by_key`: Best Case $\mathcal{O}(1)$ (key at head); Average/Worst Case $\mathcal{O}(n)$.
  - `delete_all_by_key`: Always $\Theta(n)$ because the entire list must be scanned.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- **Consecutive matching nodes (e.g., `[7 -> 7 -> 10]`)**: If you advance `prev = prev->next` unconditionally after deleting a node, you skip the second `7`! Only advance the predecessor pointer in the `else` branch when the current node is *not* deleted.

---

### 4.9 Searching, Traversal, Counting, and Updating Nodes

#### 1. Purpose
Inspect or mutate node payloads across the list:
- **Linear Search**: Return the 1-based position (or `Node*`) of the first node matching `key` (`-1` / `NULL` if absent).
- **Count Nodes**: Return the total number of nodes $n$.
- **Update Node Value**: Replace the first occurrence of `old_val` with `new_val`.

#### 2. Preconditions and Assumptions
- Functions receiving `const Node *head` do not modify list structure.

#### 3. Step-by-Step Algorithm
1. Initialize `curr = head` and `pos = 1`.
2. Loop while `curr != NULL`:
   - For search: if `curr->data == key`, return `pos` (or `curr`).
   - For count: increment `count`.
   - Advance `curr = curr->next` and `pos++`.

#### 4. Pseudocode
```
ALGORITHM SearchByKey(head, key)
INPUT: head (const Node*), key (int)
OUTPUT: 1-based position if found, -1 if not found
BEGIN
    pos <- 1
    curr <- head
    WHILE curr != NULL DO
        IF curr->data == key THEN
            RETURN pos
        END IF
        curr <- curr->next
        pos <- pos + 1
    END WHILE
    RETURN -1
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
Linear Search for key = 30 in [10 -> 20 -> 30 -> 40]:
  Step 1 (pos=1): curr -> [10] (10 != 30) -> advance curr = curr->next
  Step 2 (pos=2): curr -> [20] (20 != 30) -> advance curr = curr->next
  Step 3 (pos=3): curr -> [30] (30 == 30) -> MATCH! Return pos = 3
```

#### 6. Reusable C Functions and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct Node {
    int data;
    struct Node *next;
} Node;

int search_key_position(const Node *head, int key) {
    int pos = 1;
    for (const Node *curr = head; curr != NULL; curr = curr->next, pos++) {
        if (curr->data == key) {
            return pos;
        }
    }
    return -1;
}

size_t count_nodes(const Node *head) {
    size_t count = 0;
    for (const Node *curr = head; curr != NULL; curr = curr->next) {
        count++;
    }
    return count;
}

bool update_first_key(Node *head, int old_val, int new_val) {
    for (Node *curr = head; curr != NULL; curr = curr->next) {
        if (curr->data == old_val) {
            curr->data = new_val;
            return true;
        }
    }
    return false;
}

int main(void) {
    Node *n1 = malloc(sizeof(*n1)), *n2 = malloc(sizeof(*n2)), *n3 = malloc(sizeof(*n3));
    n1->data = 10; n1->next = n2;
    n2->data = 20; n2->next = n3;
    n3->data = 30; n3->next = NULL;
    Node *head = n1;

    printf("Node count: %zu\n", count_nodes(head));
    printf("Position of 20: %d\n", search_key_position(head, 20));
    update_first_key(head, 20, 200);
    printf("After update (20 -> 200), position of 200: %d\n",
           search_key_position(head, 200));

    while (head != NULL) { Node *t = head; head = head->next; free(t); }
    return 0;
}
```

#### Program Output:
```
Node count: 3
Position of 20: 2
After update (20 -> 200), position of 200: 2
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: Search/Update: Best Case $\mathcal{O}(1)$, Average/Worst Case $\mathcal{O}(n)$. Counting: $\Theta(n)$.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- Binary search is **not** advantageous on a Singly Linked List because finding the middle element still requires $\mathcal{O}(n)$ pointer hops, making binary search on an SLL $\mathcal{O}(n)$ time instead of $\mathcal{O}(\log n)$.

---

### 4.10 Reversing a Singly Linked List (Iterative 3-Pointer & Recursive)

#### 1. Purpose
Reverse the direction of all `next` links in-place so the original tail becomes the new `head` and the original `head` becomes the new tail (`next == NULL`), without allocating any new heap nodes.

#### 2. Preconditions and Assumptions
- `head_ref` is non-NULL. Empty (`n = 0`) and single-node (`n = 1`) lists remain unchanged.

#### 3. Step-by-Step Algorithm
- **Method A: Iterative 3-Pointer Algorithm (`prev`, `curr`, `next_node`)**:
  1. Initialize `Node *prev = NULL`, `Node *curr = *head_ref`, `Node *next_node = NULL`.
  2. While `curr != NULL`:
     - Save successor: `next_node = curr->next` (so we don't lose the rest of the list).
     - Reverse link: `curr->next = prev`.
     - Advance `prev = curr`.
     - Advance `curr = next_node`.
  3. Update `*head_ref = prev` (`prev` now points to the old tail / new head).
- **Method B: Recursive Reversal**:
  1. Base case: if `head == NULL || head->next == NULL`, return `head`.
  2. Recursively reverse the sublist starting at `head->next`: `Node *new_head = reverse_recursive(head->next)`.
  3. Make `head`'s successor point back to `head`: `head->next->next = head`.
  4. Break `head`'s old forward link: `head->next = NULL`.
  5. Return `new_head`.

#### 4. Pseudocode
```
ALGORITHM ReverseIterative(head_ref)
INPUT: head_ref (Node**)
OUTPUT: Reverses list in-place in O(n) time and O(1) auxiliary space
BEGIN
    prev <- NULL
    curr <- *head_ref
    WHILE curr != NULL DO
        next_node  <- curr->next
        curr->next <- prev
        prev       <- curr
        curr       <- next_node
    END WHILE
    *head_ref <- prev
END
```

#### 5. Before-and-After ASCII Pointer Diagrams (Step-by-Step Iterative Trace)
```
Initial State:
  prev = NULL      curr -> [10] -> [20] -> [30] -> NULL

Iteration 1 (next_node = [20]; [10]->next = NULL; prev = [10]; curr = [20]):
  NULL <-- [10]    [20] -> [30] -> NULL
            ^       ^
           prev    curr

Iteration 2 (next_node = [30]; [20]->next = [10]; prev = [20]; curr = [30]):
  NULL <-- [10] <-- [20]    [30] -> NULL
                     ^       ^
                    prev    curr

Iteration 3 (next_node = NULL; [30]->next = [20]; prev = [30]; curr = NULL):
  NULL <-- [10] <-- [20] <-- [30]    curr = NULL
                              ^
                             prev

Final Update (*head_ref = prev):
  head ---> [30] ---> [20] ---> [10] ---> NULL
```

#### 6. Reusable C Functions and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct Node {
    int data;
    struct Node *next;
} Node;

void reverse_iterative(Node **head_ref) {
    if (head_ref == NULL) {
        return;
    }
    Node *prev = NULL;
    Node *curr = *head_ref;
    while (curr != NULL) {
        Node *next_node = curr->next; /* 1. Save remainder of list */
        curr->next = prev;            /* 2. Reverse current link   */
        prev = curr;                  /* 3. Advance prev           */
        curr = next_node;             /* 4. Advance curr           */
    }
    *head_ref = prev;
}

Node *reverse_recursive(Node *head) {
    if (head == NULL || head->next == NULL) {
        return head;
    }
    Node *new_head = reverse_recursive(head->next);
    head->next->next = head; /* Make next node point back to head */
    head->next = NULL;       /* Prevent cycle at original head    */
    return new_head;
}

void print_list(const Node *head) {
    for (const Node *c = head; c != NULL; c = c->next) printf("[%d] -> ", c->data);
    printf("NULL\n");
}

int main(void) {
    Node *n1 = malloc(sizeof(*n1)), *n2 = malloc(sizeof(*n2)), *n3 = malloc(sizeof(*n3));
    n1->data = 10; n1->next = n2;
    n2->data = 20; n2->next = n3;
    n3->data = 30; n3->next = NULL;
    Node *head = n1;

    printf("Original           : "); print_list(head);
    reverse_iterative(&head);
    printf("Reversed (Iterative): "); print_list(head);
    head = reverse_recursive(head);
    printf("Reversed (Recursive): "); print_list(head);

    while (head != NULL) { Node *t = head; head = head->next; free(t); }
    return 0;
}
```

#### Program Output:
```
Original           : [10] -> [20] -> [30] -> NULL
Reversed (Iterative): [30] -> [20] -> [10] -> NULL
Reversed (Recursive): [10] -> [20] -> [30] -> NULL
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: $\Theta(n)$ for both iterative and recursive approaches.
- **Auxiliary Space Complexity**:
  - **Iterative**: $\mathcal{O}(1)$ (only 3 local pointer variables).
  - **Recursive**: $\mathcal{O}(n)$ call-stack frames (risks stack overflow for very large $n$).

#### 8. Edge Cases and Common Mistakes
- **Recursive Reversal Cycle Bug**: Omitting `head->next = NULL` in `reverse_recursive` leaves the original first node pointing to the original second node while the second node also points back to the first node, creating an infinite cycle `[10 <-> 20]`!

---

### 4.11 Complete C11 Demonstration Program

This self-contained, compilable C11 program demonstrates every Singly Linked List operation in sequence with full memory cleanup.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct Node {
    int data;
    struct Node *next;
} Node;

Node *create_node(int value) {
    Node *node = malloc(sizeof(*node));
    if (node != NULL) {
        node->data = value;
        node->next = NULL;
    }
    return node;
}

bool sll_insert_front(Node **head_ref, int value) {
    Node *n = create_node(value);
    if (n == NULL) return false;
    n->next = *head_ref;
    *head_ref = n;
    return true;
}

bool sll_insert_end(Node **head_ref, int value) {
    Node *n = create_node(value);
    if (n == NULL) return false;
    if (*head_ref == NULL) {
        *head_ref = n;
        return true;
    }
    Node *curr = *head_ref;
    while (curr->next != NULL) curr = curr->next;
    curr->next = n;
    return true;
}

bool sll_insert_at_pos(Node **head_ref, int pos, int value) {
    if (pos < 1) return false;
    if (pos == 1) return sll_insert_front(head_ref, value);
    Node *prev = *head_ref;
    for (int i = 1; i < pos - 1 && prev != NULL; i++) prev = prev->next;
    if (prev == NULL) return false;
    Node *n = create_node(value);
    if (n == NULL) return false;
    n->next = prev->next;
    prev->next = n;
    return true;
}

bool sll_delete_front(Node **head_ref, int *out) {
    if (*head_ref == NULL) return false;
    Node *victim = *head_ref;
    if (out != NULL) *out = victim->data;
    *head_ref = victim->next;
    free(victim);
    return true;
}

bool sll_delete_end(Node **head_ref, int *out) {
    if (*head_ref == NULL) return false;
    if ((*head_ref)->next == NULL) return sll_delete_front(head_ref, out);
    Node *prev = *head_ref;
    while (prev->next->next != NULL) prev = prev->next;
    if (out != NULL) *out = prev->next->data;
    free(prev->next);
    prev->next = NULL;
    return true;
}

bool sll_delete_key(Node **head_ref, int key) {
    Node **indirect = head_ref;
    while (*indirect != NULL) {
        if ((*indirect)->data == key) {
            Node *victim = *indirect;
            *indirect = victim->next;
            free(victim);
            return true;
        }
        indirect = &((*indirect)->next);
    }
    return false;
}

void sll_reverse(Node **head_ref) {
    Node *prev = NULL, *curr = *head_ref;
    while (curr != NULL) {
        Node *next = curr->next;
        curr->next = prev;
        prev = curr;
        curr = next;
    }
    *head_ref = prev;
}

void sll_display(const char *label, const Node *head) {
    printf("%-26s: ", label);
    for (const Node *c = head; c != NULL; c = c->next) {
        printf("[%d] -> ", c->data);
    }
    printf("NULL\n");
}

void sll_destroy(Node **head_ref) {
    while (*head_ref != NULL) {
        Node *tmp = *head_ref;
        *head_ref = tmp->next;
        free(tmp);
    }
}

int main(void) {
    Node *head = NULL;
    int removed = 0;

    sll_insert_end(&head, 20);
    sll_insert_end(&head, 40);
    sll_insert_front(&head, 10);
    sll_insert_at_pos(&head, 3, 30);
    sll_display("1. After Insertions", head);

    sll_reverse(&head);
    sll_display("2. After Iterative Reverse", head);

    sll_delete_front(&head, &removed);
    printf("3. Deleted Front (%d)\n", removed);
    sll_display("   List State", head);

    sll_delete_end(&head, &removed);
    printf("4. Deleted End (%d)\n", removed);
    sll_display("   List State", head);

    sll_delete_key(&head, 20);
    sll_display("5. After Deleting Key 20", head);

    sll_destroy(&head);
    sll_display("6. After Destroy", head);
    return 0;
}
```

#### Program Output:
```
1. After Insertions       : [10] -> [20] -> [30] -> [40] -> NULL
2. After Iterative Reverse: [40] -> [30] -> [20] -> [10] -> NULL
3. Deleted Front (40)
   List State             : [30] -> [20] -> [10] -> NULL
4. Deleted End (10)
   List State             : [30] -> [20] -> NULL
5. After Deleting Key 20  : [30] -> NULL
6. After Destroy          : NULL
```

---

## 5. Complexity Analysis and Comparison Tables

### 5.1 Time and Space Complexity of Singly Linked List Operations
Let $n$ be the number of nodes currently in the list, and $k$ be the 1-based target position ($1 \le k \le n$).

| Operation | Best Case Time | Average Case Time | Worst Case Time | Auxiliary Space | Total Space |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Insert at Beginning (Head)** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Insert at End (No `tail` ptr)** | $\mathcal{O}(1)$ ($n=0$) | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Insert at End (With `tail` ptr)** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Insert at Position $k$** | $\mathcal{O}(1)$ ($k=1$) | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ ($k=n+1$) | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Insert After Given `Node*`** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Insert Before Given Key/Node** | $\mathcal{O}(1)$ (at head) | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Delete from Beginning (Head)** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Delete from End (With or Without `tail`)** | $\mathcal{O}(1)$ ($n=1$) | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Delete at Position $k$** | $\mathcal{O}(1)$ ($k=1$) | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ ($k=n$) | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Delete by Key (First / All)** | $\mathcal{O}(1)$ / $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Linear Search / Update** | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Reverse List (Iterative)** | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Reverse List (Recursive)** | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ (stack) | $\mathcal{O}(n)$ |

### 5.2 Comparison: Arrays vs. Dynamic Arrays vs. Singly Linked Lists

| Criterion | Static Array (`int a[N]`) | Dynamic Array (`realloc`) | Singly Linked List (`Node*`) |
| :--- | :--- | :--- | :--- |
| **Memory Layout** | Contiguous (Stack or BSS/Data) | Contiguous (Heap) | Non-contiguous scattered nodes (Heap) |
| **Random Access (`i`-th item)** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Insert / Delete at Front** | $\mathcal{O}(n)$ (shift elements) | $\mathcal{O}(n)$ (shift elements) | $\mathcal{O}(1)$ |
| **Insert at End** | $\mathcal{O}(1)$ (up to capacity $N$) | $\mathcal{O}(1)$ amortized, $\mathcal{O}(n)$ worst | $\mathcal{O}(n)$ (or $\mathcal{O}(1)$ with `tail` ptr) |
| **Delete from End** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ (must find `second_last`) |
| **Search (Unsorted / Sorted)** | $\mathcal{O}(n)$ / $\mathcal{O}(\log n)$ | $\mathcal{O}(n)$ / $\mathcal{O}(\log n)$ | $\mathcal{O}(n)$ / $\mathcal{O}(n)$ |
| **Extra Memory Overhead** | $0$ bytes (or unused capacity) | Unused spare capacity ($0$–$50\%$) | $8\text{ bytes}$ (`next` ptr) + padding per node |
| **CPU Cache Locality** | Excellent (spatial prefetching) | Excellent (spatial prefetching) | Poor (pointer chasing across heap) |

### 5.3 Comparison: Head-Only Representation vs. Head-and-Tail Representation

| Feature | Head-Only (`Node *head`) | Head + Tail Descriptor (`head`, `tail`, `size`) |
| :--- | :--- | :--- |
| **Descriptor Footprint** | $8\text{ bytes}$ | $24\text{ bytes}$ |
| **Append at End (`push_back`)** | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ |
| **Concatenate Two Lists** | $\mathcal{O}(n_1)$ | $\mathcal{O}(1)$ (`L1.tail->next = L2.head`) |
| **Delete from End (`pop_back`)** | $\mathcal{O}(n)$ | Still $\mathcal{O}(n)$ (no backward link from `tail`) |
| **Query List Length** | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ |
| **Implementation Complexity** | Simpler (only `head` updated) | Must keep `tail` synchronized when $n \in \{0, 1\}$ |

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Broken Chain: Incorrect Order of Pointer Assignments on Insertion
If you update `prev->next = new_node` *before* saving `prev->next` into `new_node->next`, the address of the rest of the list is overwritten and permanently leaked.

```c
/* BUG: Overwrites prev->next first; new_node->next points to new_node itself! */
prev->next = new_node;
new_node->next = prev->next; /* Infinite self-loop; rest of list is leaked! */

/* FIX: Always wire the NEW node's outgoing link FIRST */
new_node->next = prev->next; /* Step 1: Connect new_node to successor */
prev->next = new_node;       /* Step 2: Connect predecessor to new_node */
```

### 6.2 Memory Leak on Deletion (Unlinking Without Calling `free()`)
Simply bypassing a node (`prev->next = prev->next->next`) removes it from traversal, but the node remains allocated on the heap forever.

```c
/* BUG: Unlinks node at prev->next without saving or freeing its address */
prev->next = prev->next->next; /* Heap memory leak! */

/* FIX: Save pointer to victim node, unlink it, and then free(victim) */
Node *victim = prev->next;
prev->next = victim->next;
free(victim);
```

### 6.3 Null Dereference on Single-Node List in Tail Deletion
When a list has only 1 node (`head->next == NULL`), the expression `curr->next->next` dereferences `NULL`.

```c
/* BUG: Crashes when list has 0 or 1 node */
Node *curr = *head_ref;
while (curr->next->next != NULL) { /* SIGSEGV if curr == NULL or curr->next == NULL */
    curr = curr->next;
}

/* FIX: Explicitly guard empty (n=0) and single-node (n=1) cases first */
if (*head_ref == NULL) return false;
if ((*head_ref)->next == NULL) {
    free(*head_ref);
    *head_ref = NULL;
    return true;
}
```

### 6.4 Losing Head Updates When Deleting All Occurrences of a Key
Using an `if` instead of a `while` (or indirect pointer) at the head fails when the first two or more nodes match `key` (e.g., `[7 -> 7 -> 10]`).

```c
/* BUG: Only deletes the first 7 if head starts with [7 -> 7 -> 10] */
if (*head_ref != NULL && (*head_ref)->data == key) {
    Node *tmp = *head_ref;
    *head_ref = tmp->next;
    free(tmp);
}

/* FIX: Use a while loop for leading matches (or the Node** indirect pattern) */
while (*head_ref != NULL && (*head_ref)->data == key) {
    Node *tmp = *head_ref;
    *head_ref = tmp->next;
    free(tmp);
}
```

### 6.5 Infinite Cycle During Recursive Reversal
Forgetting to null out `head->next` after reversing the link creates a 2-node cycle between the original first and second nodes.

```c
/* BUG: Original head still points to original second node */
Node *new_head = reverse_recursive(head->next);
head->next->next = head;
return new_head; /* Cycle between old 1st and 2nd nodes! */

/* FIX: Explicitly terminate the new tail with NULL */
Node *new_head = reverse_recursive(head->next);
head->next->next = head;
head->next = NULL;
return new_head;
```

### 6.6 Off-by-One Traversal Errors (`curr != NULL` vs. `curr->next != NULL`)
Confusing the loop termination condition is the single most common bug in linked list exams:
- Use `while (curr != NULL)` when you want to **process every node** (printing, counting, searching) and end with `curr == NULL`.
- Use `while (curr->next != NULL)` (after checking `curr != NULL`) when you want `curr` to **stop on the last (tail) node**.
- Use `while (curr->next->next != NULL)` (after checking `curr != NULL && curr->next != NULL`) when you want `curr` to **stop on the second-to-last node**.

### 6.7 Boundary Conditions Checklist
- [ ] **Empty List (`head == NULL`)**: Tested for every insertion, deletion, search, and reversal function?
- [ ] **Single-Node List (`head->next == NULL`)**: Tested for tail deletion, position deletion (`pos = 1`), and reversal?
- [ ] **Two-Node List**: Tested for tail deletion and reversal?
- [ ] **Operation at Head (`pos = 1`)**: Does `*head_ref` update properly in the caller?
- [ ] **Operation at Tail (`pos = n` or `pos = n + 1`)**: Does the new tail always have `tail->next == NULL`?
- [ ] **Out-of-Bounds Position (`pos <= 0` or `pos > n + 1`)**: Does the function reject cleanly without leaking memory or crashing?

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways
1. A **Singly Linked List** chains heap-allocated `Node` structures via a single forward pointer (`next`), providing $\mathcal{O}(1)$ insertion and deletion at the head.
2. Any operation in a Singly Linked List that inserts before or deletes an arbitrary node requires access to the **predecessor node** (`prev`), because links cannot be traversed backward.
3. Even with a `tail` pointer, **tail deletion is $\Theta(n)$** in a Singly Linked List because finding the second-to-last node requires a full scan from `head`.
4. **Iterative 3-pointer reversal** (`prev`, `curr`, `next_node`) reverses an $n$-node list in-place in $\Theta(n)$ time and $\mathcal{O}(1)$ auxiliary space.
5. The **pointer-to-pointer (`Node **indirect`)** idiom eliminates separate special-case code for the `head` node during conditional deletions.

### 7.2 Essential Singly Linked List Invariants
- **Empty List Invariant**: $n = 0 \iff \text{head} = \text{NULL}$ (and $\text{tail} = \text{NULL}$ if maintained).
- **Single-Node Invariant**: $n = 1 \iff \text{head} \neq \text{NULL} \land \text{head}\to\text{next} = \text{NULL}$ (and $\text{head} = \text{tail}$).
- **Tail Termination Invariant**: For any non-empty acyclic list, there exists a unique tail node $v_n$ reachable in $n - 1$ hops from `head` satisfying $v_n\to\text{next} = \text{NULL}$.

### 7.3 Quick Self-Test Checklist
- [ ] Why does inserting a node after `prev` require executing `new_node->next = prev->next;` *before* `prev->next = new_node;`?
- [ ] Why does `delete_from_end` take $\Theta(n)$ time even if we maintain a `Node *tail` pointer?
- [ ] Can you trace the 4 lines inside the `while (curr != NULL)` loop of `reverse_iterative` on a 3-node list without looking at the notes?
- [ ] How does `Node **indirect = head_ref` handle deleting the `head` node without an `if (curr == *head_ref)` branch?
- [ ] What is the time complexity of finding the middle node of a Singly Linked List?

### 7.4 Related Study Notes
- **Previous Module**: [01 — C Structures, Pointers, and Memory](./01-C-Structures-Pointers-and-Memory.md)
- **Next Module**: [03 — Doubly Linked Lists](./03-Doubly-Linked-Lists.md)
- **Circular Lists**: [04 — Circular Linked Lists](./04-Circular-Linked-Lists.md)
- **Stacks**: [05 — Stacks](./07-Stacks-Array-and-Linked-List.md)
- **Course Index**: [DSA Notes Root README](../README.md)
