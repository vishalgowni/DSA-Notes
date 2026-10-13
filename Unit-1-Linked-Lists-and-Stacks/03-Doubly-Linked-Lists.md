# Doubly Linked Lists (DLL)
---

## Table of Contents
1. [Introduction](#1-introduction)
   - 1.1 [What Is a Doubly Linked List?](#11-what-is-a-doubly-linked-list)
   - 1.2 [Why Is It Needed and What Problem Does It Solve?](#12-why-is-it-needed-and-what-problem-does-it-solve)
   - 1.3 [Relationship to Prior Concepts](#13-relationship-to-prior-concepts)
   - 1.4 [Learning Objectives](#14-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Formal and Intuitive Definitions](#21-formal-and-intuitive-definitions)
   - 2.2 [Terminology Reference Table](#22-terminology-reference-table)
   - 2.3 [Mathematical Properties and Bidirectional Invariants](#23-mathematical-properties-and-bidirectional-invariants)
   - 2.4 [Conceptual Anatomy Diagram](#24-conceptual-anatomy-diagram)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [`DNode` Structure and List Descriptor (`head`, `tail`, `size`)](#31-dnode-structure-and-list-descriptor-head-tail-size)
   - 3.2 [Bidirectional Heap Memory Layout Diagram](#32-bidirectional-heap-memory-layout-diagram)
   - 3.3 [Canonical States: Empty, Single-Node, and Multi-Node Invariants](#33-canonical-states-empty-single-node-and-multi-node-invariants)
   - 3.4 [NULL-Terminated vs. Sentinel (Dummy Header/Trailer) Representation](#34-null-terminated-vs-sentinel-dummy-headertrailer-representation)
   - 3.5 [Advantages, Limitations, and Real-World Applications](#35-advantages-limitations-and-real-world-applications)
4. [Operations and Algorithms](#4-operations-and-algorithms)
   - 4.1 [Insert at Beginning (Head Insertion)](#41-insert-at-beginning-head-insertion)
   - 4.2 [Insert at End (Tail Insertion)](#42-insert-at-end-tail-insertion)
   - 4.3 [Insert at a Specified Position (1-Based Indexing & 4-Pointer Wiring)](#43-insert-at-a-specified-position-1-based-indexing--4-pointer-wiring)
   - 4.4 [Insert Before and After a Given Node or Key](#44-insert-before-and-after-a-given-node-or-key)
   - 4.5 [Delete from Beginning (Head Deletion)](#45-delete-from-beginning-head-deletion)
   - 4.6 [Delete from End (Tail Deletion)](#46-delete-from-end-tail-deletion)
   - 4.7 [Delete from a Specified Position (1-Based Indexing)](#47-delete-from-a-specified-position-1-based-indexing)
   - 4.8 [Delete by Key and $\mathcal{O}(1)$ Direct Node-Pointer Deletion](#48-delete-by-key-and-mathcalo1-direct-node-pointer-deletion)
   - 4.9 [Forward/Backward Traversal, Searching, Counting, and Updating](#49-forwardbackward-traversal-searching-counting-and-updating)
   - 4.10 [Reversing a Doubly Linked List In-Place](#410-reversing-a-doubly-linked-list-in-place)
   - 4.11 [Complete C11 Demonstration Program](#411-complete-c11-demonstration-program)
5. [Complexity Analysis and Comparison Tables](#5-complexity-analysis-and-comparison-tables)
   - 5.1 [Time and Space Complexity of Doubly Linked List Operations](#51-time-and-space-complexity-of-doubly-linked-list-operations)
   - 5.2 [Comparison: Singly Linked List vs. Doubly Linked List vs. Dynamic Array](#52-comparison-singly-linked-list-vs-doubly-linked-list-vs-dynamic-array)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Clobbering `prev->next` During 4-Pointer Insertion](#61-clobbering-prev-next-during-4-pointer-insertion)
   - 6.2 [Forgetting to Set `head->prev = NULL` After Head Deletion](#62-forgetting-to-set-head-prev--null-after-head-deletion)
   - 6.3 [Unchecked `target->next->prev` Dereference When Deleting the Tail Node](#63-unchecked-target-next-prev-dereference-when-deleting-the-tail-node)
   - 6.4 [Dangling `tail` Pointer When Deleting the Sole Node in a Descriptor](#64-dangling-tail-pointer-when-deleting-the-sole-node-in-a-descriptor)
   - 6.5 [Failing to Update `*head_ref` at the End of In-Place Reversal](#65-failing-to-update-head_ref-at-the-end-of-in-place-reversal)
   - 6.6 [Memory Leak / Use-After-Free During Unlink](#66-memory-leak--use-after-free-during-unlink)
   - 6.7 [Boundary Conditions Checklist](#67-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)
   - 7.1 [Key Takeaways](#71-key-takeaways)
   - 7.2 [Essential Doubly Linked List Invariants](#72-essential-doubly-linked-list-invariants)
   - 7.3 [Quick Self-Test Checklist](#73-quick-self-test-checklist)
   - 7.4 [Related Study Notes](#74-related-study-notes)

---

## 1. Introduction

### 1.1 What Is a Doubly Linked List?
A **Doubly Linked List (DLL)** is a bidirectional linear linked data structure in which every node contains **three** fields:
1. **`prev`**: A pointer to the immediate predecessor node (or `NULL` if the node is the `head`).
2. **`data`**: The payload value stored in the node.
3. **`next`**: A pointer to the immediate successor node (or `NULL` if the node is the `tail`).

Because each node knows both its left neighbor (`prev`) and its right neighbor (`next`), the list can be traversed seamlessly in **both forward and backward directions**.

### 1.2 Why Is It Needed and What Problem Does It Solve?
In [02 — Singly Linked Lists](./02-Singly-Linked-Lists.md), we encountered three major algorithmic limitations caused by one-way `next` links:
1. **$\Theta(n)$ Tail Deletion Even With a `tail` Pointer**: Removing the last node of a Singly Linked List requires knowing the second-to-last node (`second_last->next = NULL`), forcing an $\mathcal{O}(n)$ walk from `head`.
2. **$\Theta(n)$ Deletion Given Only a `Node*` Pointer**: If a hash table or cursor holds a direct pointer `Node *target` to a node in the middle of a Singly Linked List, it *still* cannot delete `target` in $\mathcal{O}(1)$ time because it does not know `target`'s predecessor!
3. **Inability to Traverse Backward or Insert Before a Node in $\mathcal{O}(1)$**: Navigating to a previous item (e.g., browser "Back" button, text editor cursor left, Undo history) is impossible without restarting from `head`.

A **Doubly Linked List** eliminates all three bottlenecks at the cost of one additional pointer (`prev`) per node:
- Given a direct pointer `DNode *target`, **deleting `target` is strictly $\mathcal{O}(1)$** because its predecessor is immediately accessible via `target->prev`.
- **Inserting before `target` is strictly $\mathcal{O}(1)$**.
- **Deleting from the tail (with a `tail` pointer) is strictly $\mathcal{O}(1)$**.

### 1.3 Relationship to Prior Concepts
A Doubly Linked List extends the self-referential structure from [01 — C Structures, Pointers, and Memory](./01-C-Structures-Pointers-and-Memory.md) by adding a second self-referential link (`prev`). It forms the backbone of **Deques (Double-Ended Queues)**, **LRU (Least Recently Used) Caches**, and Linux kernel intrusive lists (`struct list_head`).

### 1.4 Learning Objectives
After completing this module, you will be able to:
1. Define `DNode` in C and verify all bidirectional structural invariants (`u->next == v` $\iff$ `v->prev == u`).
2. Trace the **4-pointer wiring sequence** required to insert a new node between two existing nodes without corrupting links.
3. Implement all insertion variants (at beginning, at end, at 1-based position $k$, before/after a node or key).
4. Implement all deletion variants (from beginning, from end, at position $k$, by key, and $\mathcal{O}(1)$ deletion via direct `DNode*` pointer).
5. Traverse a Doubly Linked List in both forward and reverse order, and reverse a Doubly Linked List in-place in $\mathcal{O}(n)$ time and $\mathcal{O}(1)$ space.

---

## 2. Definitions and Terminology

### 2.1 Formal and Intuitive Definitions
- **Formal Definition**: A Doubly Linked List of length $n \ge 0$ is an ordered sequence of nodes $L = \langle v_1, v_2, \dots, v_n \rangle$ where each node $v_i$ is a 3-tuple $(\text{prev}_i, \text{data}_i, \text{next}_i)$ satisfying:
  - $\text{prev}_1 = \text{NULL}$ and $\text{next}_n = \text{NULL}$.
  - For all $1 \le i < n$: $\text{next}_i = \text{addr}(v_{i+1})$ and $\text{prev}_{i+1} = \text{addr}(v_i)$.
- **Intuitive Analogy**: Think of a Doubly Linked List as a **passenger train**: each train car (`DNode`) has a front coupler connected to the car ahead (`prev`) and a rear coupler connected to the car behind (`next`). A conductor standing in *any* car can walk forward toward the caboose or backward toward the locomotive, or uncouple that specific car directly by releasing its front and rear couplers.

### 2.2 Terminology Reference Table

| Term | Definition |
| :--- | :--- |
| **`DNode`** | A doubly linked node containing `prev`, `data`, and `next` fields. |
| **Previous Pointer (`prev`)** | Pointer to the predecessor node; `NULL` for the `head` node. |
| **Next Pointer (`next`)** | Pointer to the successor node; `NULL` for the `tail` node. |
| **Head (`head`)** | Pointer to the first node ($v_1$), which satisfies `head->prev == NULL`. |
| **Tail (`tail`)** | Pointer to the last node ($v_n$), which satisfies `tail->next == NULL`. |
| **4-Pointer Wiring** | The four pointer assignments required when inserting a new node between `u` and `w`: `new->prev`, `new->next`, `u->next`, and `w->prev`. |
| **2-Pointer Bypass** | The two pointer updates required when unlinking an interior node `v` between `u` and `w`: `u->next = w` and `w->prev = u`. |
| **Bidirectional Symmetry** | The fundamental invariant that `u`'s successor is `v` if and only if `v`'s predecessor is `u`. |

### 2.3 Mathematical Properties and Bidirectional Invariants

#### 1. Bidirectional Reciprocity (Symmetry) Invariant
For every node $u$ in a valid Doubly Linked List:

$$u\to\text{next} \neq \text{NULL} \implies (u\to\text{next})\to\text{prev} == u$$

$$u\to\text{prev} \neq \text{NULL} \implies (u\to\text{prev})\to\text{next} == u$$

#### 2. Boundary Termination Invariants
In any non-empty, non-circular Doubly Linked List with entry pointers `head` and `tail`:

$$\text{head}\to\text{prev} == \text{NULL} \qquad \text{and} \qquad \text{tail}\to\text{next} == \text{NULL}$$

#### 3. Memory Footprint and Pointer Overhead on 64-Bit Systems
With `int data` ($4\text{ bytes}$), $4\text{ bytes}$ of alignment padding, and two 8-byte pointers (`prev` and `next`):

$$\operatorname{sizeof}(\text{DNode}) = 8\text{ (prev)} + 8\text{ (next)} + 4\text{ (data)} + 4\text{ (padding)} = 24\text{ bytes}$$

$$\text{Pointer Overhead Ratio} = \frac{16\text{ bytes}}{24\text{ bytes}} = \frac{2}{3} \approx 66.7\%$$

### 2.4 Conceptual Anatomy Diagram

```
         head                                                        tail
          |                                                           |
          v                                                           v
        +------+------+------+     +------+------+------+     +------+------+------+
NULL <--| prev |  10  | next |---->| prev |  20  | next |---->| prev |  30  | NULL |
        | NULL |      |      |<----|      |      |      |<----|      |      |      |
        +------+------+------+     +------+------+------+     +------+------+------+
```

---

## 3. Representation and Memory Layout in C

### 3.1 `DNode` Structure and List Descriptor (`head`, `tail`, `size`)
By placing the two 8-byte pointers together (or ordering `data` last), we ensure optimal alignment on 64-bit architectures:

```c
typedef struct DNode {
    int data;               /* 4 bytes (+ 4 bytes alignment padding) */
    struct DNode *prev;     /* 8 bytes: pointer to predecessor node  */
    struct DNode *next;     /* 8 bytes: pointer to successor node    */
} DNode;                    /* Total sizeof(DNode) = 24 bytes        */

/* Optional Wrapper Descriptor for O(1) Head and Tail Operations */
typedef struct DoublyLinkedList {
    DNode *head;            /* First node (head->prev == NULL) */
    DNode *tail;            /* Last node  (tail->next == NULL) */
    size_t size;            /* Current number of nodes         */
} DoublyLinkedList;
```

### 3.2 Bidirectional Heap Memory Layout Diagram

```
STACK: head = 0x55a01000, tail = 0x55a03080

HEAP SEGMENT:
  Node 1 @ 0x55a01000            Node 2 @ 0x55a02040            Node 3 @ 0x55a03080
+------------------------+     +------------------------+     +------------------------+
| data : 10              |     | data : 20              |     | data : 30              |
| prev : NULL (0x0)      |     | prev : 0x55a01000 (N1) |     | prev : 0x55a02040 (N2) |
| next : 0x55a02040 (N2) |<--->| next : 0x55a03080 (N3) |<--->| next : NULL (0x0)      |
+------------------------+     +------------------------+     +------------------------+
```

### 3.3 Canonical States: Empty, Single-Node, and Multi-Node Invariants

```
STATE 1: Empty List (n = 0)
  head == NULL, tail == NULL

STATE 2: Single-Node List (n = 1)
  head == tail == 0x55a01000
  head->prev == NULL  AND  head->next == NULL
          head / tail
              |
              v
        +------+------+------+
NULL <--| NULL |  42  | NULL |--> NULL
        +------+------+------+

STATE 3: Two-Node List (n = 2)
  head->next == tail  AND  tail->prev == head
  head->prev == NULL  AND  tail->next == NULL
```

### 3.4 NULL-Terminated vs. Sentinel (Dummy Header/Trailer) Representation
1. **NULL-Terminated Representation (Standard)**: `head` points directly to the first data node (`head->prev == NULL`) and `tail` points to the last data node (`tail->next == NULL`). Requires `if (ptr != NULL)` boundary checks when updating neighbor pointers at the ends.
2. **Sentinel Dummy Header & Trailer Nodes**: Allocates two permanent dummy nodes (`header` and `trailer`) that store no user data, initialized with `header->next = trailer` and `trailer->prev = header`. Every real node is guaranteed to have both a non-NULL `prev` and a non-NULL `next`, eliminating all `if (node == head)` branch checks!

### 3.5 Advantages, Limitations, and Real-World Applications
- **Advantages**:
  - **Bidirectional Traversal**: Can scan forward (`curr = curr->next`) and backward (`curr = curr->prev`).
  - **$\mathcal{O}(1)$ Arbitrary Node Deletion**: Given `DNode *target`, `target` can unhook itself in $\mathcal{O}(1)$ time without searching from `head`.
  - **$\mathcal{O}(1)$ Insertion Before or After Any `DNode*`**.
  - **$\mathcal{O}(1)$ Push/Pop at Both Ends** when `head` and `tail` pointers are maintained.
- **Limitations**:
  - **Higher Memory Overhead**: Each node stores two pointers ($16\text{ bytes}$ on 64-bit) instead of one ($8\text{ bytes}$).
  - **More Pointer Updates per Operation**: Inserting a node in the middle requires updating **4 pointers** (vs. 2 in an SLL); deleting requires updating **2 pointers** (vs. 1 in an SLL).
- **Real-World Applications**:
  - **LRU (Least Recently Used) Cache**: Combines a Hash Map (`key -> DNode*`) with a Doubly Linked List. Accessing a cached key moves its `DNode*` to the `head` in $\mathcal{O}(1)$ time; evicting the oldest entry removes `tail` in $\mathcal{O}(1)$ time!
  - **Web Browser History**: Navigating *Back* (`curr = curr->prev`) and *Forward* (`curr = curr->next`) between visited URLs.
  - **Undo / Redo Systems in Text & Photo Editors**: Moving backward and forward through an action state chain.
  - **Music / Media Playlists**: Previous Track / Next Track navigation and $\mathcal{O}(1)$ track removal.
  - **Operating System Process Runqueues**: Linux Kernel `struct list_head` doubly linked lists for $\mathcal{O}(1)$ task scheduling and wait-queue removal.

---

## 4. Operations and Algorithms

### 4.1 Insert at Beginning (Head Insertion)

#### 1. Purpose
Allocate a new `DNode` with `value` and insert it at the front of the Doubly Linked List in $\mathcal{O}(1)$ time.

#### 2. Preconditions and Assumptions
- `head_ref` (`DNode**`) is non-NULL.

#### 3. Step-by-Step Algorithm
1. Allocate `new_node` via `malloc(sizeof(*new_node))`. If `NULL`, return `false`.
2. Set `new_node->data = value`, `new_node->prev = NULL`, and `new_node->next = *head_ref`.
3. **Non-Empty Check**: If `*head_ref != NULL`, link the old head backward to `new_node`: `(*head_ref)->prev = new_node`.
4. Update `*head_ref = new_node` and return `true`.

#### 4. Pseudocode
```
ALGORITHM DLL_InsertAtBeginning(head_ref, value)
INPUT: head_ref (DNode**), value (int)
OUTPUT: true on success, false on allocation failure
BEGIN
    new_node <- malloc(sizeof(DNode))
    IF new_node == NULL THEN RETURN false END IF

    new_node->data <- value
    new_node->prev <- NULL
    new_node->next <- *head_ref

    IF *head_ref != NULL THEN
        (*head_ref)->prev <- new_node
    END IF

    *head_ref <- new_node
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
BEFORE dll_insert_at_beginning(&head, 5):
         head
          |
          v
        +------+------+------+     +------+------+------+
NULL <--| NULL |  10  | next |<--->| prev |  20  | NULL |
        +------+------+------+     +------+------+------+

STEP 1: new_node->prev = NULL; new_node->next = *head_ref; (*head_ref)->prev = new_node;
        +------+------+------+     +------+------+------+     +------+------+------+
NULL <--| NULL |   5  | next |---->| prev |  10  | next |<--->| prev |  20  | NULL |
        |      |      |      |<----|      |      |      |     |      |      |      |
        +------+------+------+     +------+------+------+     +------+------+------+
          ^
          |
AFTER (*head_ref = new_node):
         head
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct DNode {
    int data;
    struct DNode *prev;
    struct DNode *next;
} DNode;

bool dll_insert_at_beginning(DNode **head_ref, int value) {
    if (head_ref == NULL) {
        return false;
    }
    DNode *new_node = malloc(sizeof(*new_node));
    if (new_node == NULL) {
        return false;
    }
    new_node->data = value;
    new_node->prev = NULL;
    new_node->next = *head_ref;

    if (*head_ref != NULL) {
        (*head_ref)->prev = new_node;
    }
    *head_ref = new_node;
    return true;
}

int main(void) {
    DNode *head = NULL;
    dll_insert_at_beginning(&head, 30);
    dll_insert_at_beginning(&head, 20);
    dll_insert_at_beginning(&head, 10);

    printf("NULL <- ");
    for (DNode *c = head; c != NULL; c = c->next) {
        printf("[%d]%s", c->data, (c->next != NULL) ? " <-> " : " -> NULL\n");
    }

    while (head != NULL) { DNode *t = head; head = head->next; free(t); }
    return 0;
}
```

#### Program Output:
```
NULL <- [10] <-> [20] <-> [30] -> NULL
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: $\mathcal{O}(1)$.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- **Empty List (`*head_ref == NULL`)**: Guarding `if (*head_ref != NULL)` before setting `(*head_ref)->prev = new_node` is mandatory; omitting that guard causes a NULL dereference when inserting the very first node!

---

### 4.2 Insert at End (Tail Insertion)

#### 1. Purpose
Append a newly allocated `DNode` containing `value` at the tail of the Doubly Linked List.

#### 2. Preconditions and Assumptions
- `head_ref` (`DNode**`) is non-NULL.

#### 3. Step-by-Step Algorithm
1. Allocate `new_node`, set `new_node->data = value` and `new_node->next = NULL`.
2. **Empty List Check**: If `*head_ref == NULL`, set `new_node->prev = NULL`, `*head_ref = new_node`, and return `true`.
3. Otherwise, traverse `DNode *tail = *head_ref` while `tail->next != NULL`.
4. Wire bidirectional links: `tail->next = new_node` and `new_node->prev = tail`.

#### 4. Pseudocode
```
ALGORITHM DLL_InsertAtEnd(head_ref, value)
INPUT: head_ref (DNode**), value (int)
OUTPUT: true on success, false on allocation failure
BEGIN
    new_node <- malloc(sizeof(DNode))
    IF new_node == NULL THEN RETURN false END IF
    new_node->data <- value
    new_node->next <- NULL

    IF *head_ref == NULL THEN
        new_node->prev <- NULL
        *head_ref <- new_node
        RETURN true
    END IF

    tail <- *head_ref
    WHILE tail->next != NULL DO
        tail <- tail->next
    END WHILE

    tail->next <- new_node
    new_node->prev <- tail
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
BEFORE dll_insert_at_end(&head, 30):
         head                       tail
          |                          |
          v                          v
        +------+------+------+     +------+------+------+       +------+------+------+
NULL <--| NULL |  10  | next |<--->| prev |  20  | NULL |       |  ?   |  30  | NULL | (new_node)
        +------+------+------+     +------+------+------+       +------+------+------+

AFTER (tail->next = new_node; new_node->prev = tail):
         head                                                    New Tail
          |                                                         |
          v                                                         v
        +------+------+------+     +------+------+------+     +------+------+------+
NULL <--| NULL |  10  | next |<--->| prev |  20  | next |<--->| prev |  30  | NULL |
        +------+------+------+     +------+------+------+     +------+------+------+
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct DNode {
    int data;
    struct DNode *prev;
    struct DNode *next;
} DNode;

bool dll_insert_at_end(DNode **head_ref, int value) {
    if (head_ref == NULL) {
        return false;
    }
    DNode *new_node = malloc(sizeof(*new_node));
    if (new_node == NULL) {
        return false;
    }
    new_node->data = value;
    new_node->next = NULL;

    if (*head_ref == NULL) {
        new_node->prev = NULL;
        *head_ref = new_node;
        return true;
    }

    DNode *tail = *head_ref;
    while (tail->next != NULL) {
        tail = tail->next;
    }
    tail->next = new_node;
    new_node->prev = tail;
    return true;
}

int main(void) {
    DNode *head = NULL;
    dll_insert_at_end(&head, 10);
    dll_insert_at_end(&head, 20);
    dll_insert_at_end(&head, 30);

    printf("Forward : ");
    DNode *tail = NULL;
    for (DNode *c = head; c != NULL; c = c->next) {
        printf("[%d] ", c->data);
        if (c->next == NULL) tail = c;
    }
    printf("\nBackward: ");
    for (DNode *c = tail; c != NULL; c = c->prev) {
        printf("[%d] ", c->data);
    }
    printf("\n");

    while (head != NULL) { DNode *t = head; head = head->next; free(t); }
    return 0;
}
```

#### Program Output:
```
Forward : [10] [20] [30] 
Backward: [30] [20] [10] 
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: $\mathcal{O}(n)$ with `head` only; **$\mathcal{O}(1)$ if a `tail` pointer is maintained**.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- Forgetting `new_node->prev = tail` leaves `new_node->prev` uninitialized or `NULL`, breaking backward traversal from the new tail!

---

### 4.3 Insert at a Specified Position (1-Based Indexing & 4-Pointer Wiring)

#### 1. Purpose
Insert a new `DNode` at 1-based position `pos` ($1 \le \text{pos} \le n + 1$), properly wiring all **4 bidirectional pointers** when inserting between two existing nodes.

#### 2. Preconditions and Assumptions
- `pos >= 1`. `pos == 1` inserts at the head; `pos == n + 1` inserts at the tail.

#### 3. Step-by-Step Algorithm (The Safe 4-Pointer Wiring Rule)
1. Validate `head_ref != NULL && pos >= 1`.
2. If `pos == 1`, delegate to `dll_insert_at_beginning(head_ref, value)`.
3. Traverse `DNode *prev_node = *head_ref` for `pos - 2` steps to reach the node at position `pos - 1`.
4. If `prev_node == NULL`, return `false` (`pos > n + 1`).
5. Allocate `new_node` and set `new_node->data = value`.
6. **Execute the 4-Pointer Wiring in Safe Order** (wire `new_node` first before modifying `prev_node->next`!):
   - **Link 1**: `new_node->next = prev_node->next;`
   - **Link 2**: `new_node->prev = prev_node;`
   - **Link 3**: If `prev_node->next != NULL`, set `prev_node->next->prev = new_node;`
   - **Link 4**: `prev_node->next = new_node;`

#### 4. Pseudocode
```
ALGORITHM DLL_InsertAtPosition(head_ref, pos, value)
INPUT: head_ref (DNode**), pos (int, 1-based), value (int)
OUTPUT: true if inserted, false if out of bounds
BEGIN
    IF head_ref == NULL OR pos < 1 THEN RETURN false END IF
    IF pos == 1 THEN
        RETURN DLL_InsertAtBeginning(head_ref, value)
    END IF

    prev_node <- *head_ref
    FOR i <- 1 TO pos - 2 DO
        IF prev_node == NULL THEN RETURN false END IF
        prev_node <- prev_node->next
    END FOR

    IF prev_node == NULL THEN RETURN false END IF

    new_node <- malloc(sizeof(DNode))
    IF new_node == NULL THEN RETURN false END IF
    new_node->data <- value

    /* 4-Pointer Wiring (New node's outgoing links FIRST) */
    new_node->next <- prev_node->next           /* Link (1) */
    new_node->prev <- prev_node                 /* Link (2) */
    IF prev_node->next != NULL THEN
        prev_node->next->prev <- new_node       /* Link (3) */
    END IF
    prev_node->next <- new_node                 /* Link (4) */
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams (4-Pointer Wiring)
```
Inserting new_node (25) at pos = 3 between prev_node (20) and next_node (30):

      prev_node (pos=2)                               next_node (pos=3)
   +------+------+------+                         +------+------+------+
   | prev |  20  | next | ----------------------->| prev |  30  | NULL |
   |      |      |      | <-----------------------|      |      |      |
   +------+------+------+                         +------+------+------+
             ^       |                               |      ^
             |       | (4) prev_node->next           |      | (1) new_node->next
    (2)      |       v       = new_node              |      |       = prev_node->next
new_node     |    +------+------+------+             |      |
 ->prev      +----| prev |  25  | next |-------------+------+
 = prev_node      +------+------+------+
                            ^
                            | (3) prev_node->next->prev = new_node
                            +-------------------------------------+
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct DNode {
    int data;
    struct DNode *prev;
    struct DNode *next;
} DNode;

bool dll_insert_at_position(DNode **head_ref, int pos, int value) {
    if (head_ref == NULL || pos < 1) {
        return false;
    }
    if (pos == 1) {
        DNode *n = malloc(sizeof(*n));
        if (n == NULL) return false;
        n->data = value;
        n->prev = NULL;
        n->next = *head_ref;
        if (*head_ref != NULL) (*head_ref)->prev = n;
        *head_ref = n;
        return true;
    }

    DNode *prev_node = *head_ref;
    for (int i = 1; i < pos - 1 && prev_node != NULL; i++) {
        prev_node = prev_node->next;
    }
    if (prev_node == NULL) {
        return false;
    }

    DNode *new_node = malloc(sizeof(*new_node));
    if (new_node == NULL) {
        return false;
    }
    new_node->data = value;

    /* Safe 4-Pointer Wiring Sequence */
    new_node->next = prev_node->next;          /* (1) */
    new_node->prev = prev_node;                /* (2) */
    if (prev_node->next != NULL) {
        prev_node->next->prev = new_node;      /* (3) */
    }
    prev_node->next = new_node;                /* (4) */
    return true;
}

int main(void) {
    DNode *head = NULL;
    dll_insert_at_position(&head, 1, 10); /* [10] */
    dll_insert_at_position(&head, 2, 30); /* [10 <-> 30] */
    dll_insert_at_position(&head, 2, 20); /* [10 <-> 20 <-> 30] */
    dll_insert_at_position(&head, 3, 25); /* [10 <-> 20 <-> 25 <-> 30] */

    printf("List after 1-based positional insertions:\n  NULL <- ");
    for (DNode *c = head; c != NULL; c = c->next) {
        printf("[%d]%s", c->data, (c->next != NULL) ? " <-> " : " -> NULL\n");
    }

    while (head != NULL) { DNode *t = head; head = head->next; free(t); }
    return 0;
}
```

#### Program Output:
```
List after 1-based positional insertions:
  NULL <- [10] <-> [20] <-> [25] <-> [30] -> NULL
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: Best Case $\mathcal{O}(1)$ (`pos == 1`); Average/Worst Case $\mathcal{O}(n)$.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- **Inserting at `pos == n + 1` (after the tail)**: `prev_node` is the tail node, so `prev_node->next == NULL`. The `if (prev_node->next != NULL)` check prevents dereferencing `NULL` on Link (3)!

---

### 4.4 Insert Before and After a Given Node or Key

#### 1. Purpose
Insert a new node immediately **before** or **after** a specified node (`DNode *target`) in $\mathcal{O}(1)$ time (or $\mathcal{O}(n)$ time if searching by integer `key` first).

#### 2. Preconditions and Assumptions
- `target != NULL` is a valid node currently belonging to the list.

#### 3. Step-by-Step Algorithm
- **Insert After `DNode *target` ($\mathcal{O}(1)$)**:
  1. Allocate `new_node(value)`.
  2. Set `new_node->next = target->next` and `new_node->prev = target`.
  3. If `target->next != NULL`, set `target->next->prev = new_node`.
  4. Set `target->next = new_node`.
- **Insert Before `DNode *target` ($\mathcal{O}(1)$ — Unique Advantage of DLL over SLL!)**:
  1. Allocate `new_node(value)`.
  2. Set `new_node->prev = target->prev` and `new_node->next = target`.
  3. If `target->prev != NULL`, set `target->prev->next = new_node`; **else** (meaning `target` was the `head`!), update `*head_ref = new_node`.
  4. Set `target->prev = new_node`.

#### 4. Pseudocode
```
ALGORITHM DLL_InsertBeforeNode(head_ref, target, value)
INPUT: head_ref (DNode**), target (DNode*), value (int)
OUTPUT: true on success, false on error
BEGIN
    IF head_ref == NULL OR target == NULL THEN RETURN false END IF
    new_node <- malloc(sizeof(DNode))
    IF new_node == NULL THEN RETURN false END IF
    new_node->data <- value

    new_node->prev <- target->prev
    new_node->next <- target

    IF target->prev != NULL THEN
        target->prev->next <- new_node
    ELSE
        *head_ref <- new_node
    END IF
    target->prev <- new_node
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
Inserting new_node (15) BEFORE target (20) in O(1) time using target->prev:
  target->prev (10)               new_node (15)               target (20)
  +------+------+------+      +------+------+------+      +------+------+------+
  | NULL |  10  | next |----->| prev |  15  | next |----->| prev |  20  | NULL |
  |      |      |      |<-----|      |      |      |<-----|      |      |      |
  +------+------+------+      +------+------+------+      +------+------+------+
```

#### 6. Reusable C Functions and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct DNode {
    int data;
    struct DNode *prev;
    struct DNode *next;
} DNode;

bool dll_insert_after_node(DNode *target, int value) {
    if (target == NULL) return false;
    DNode *new_node = malloc(sizeof(*new_node));
    if (new_node == NULL) return false;

    new_node->data = value;
    new_node->next = target->next;
    new_node->prev = target;
    if (target->next != NULL) {
        target->next->prev = new_node;
    }
    target->next = new_node;
    return true;
}

bool dll_insert_before_node(DNode **head_ref, DNode *target, int value) {
    if (head_ref == NULL || target == NULL) return false;
    DNode *new_node = malloc(sizeof(*new_node));
    if (new_node == NULL) return false;

    new_node->data = value;
    new_node->prev = target->prev;
    new_node->next = target;
    if (target->prev != NULL) {
        target->prev->next = new_node;
    } else {
        *head_ref = new_node; /* target was the head node! */
    }
    target->prev = new_node;
    return true;
}

DNode *dll_find_key(DNode *head, int key) {
    for (DNode *c = head; c != NULL; c = c->next) {
        if (c->data == key) return c;
    }
    return NULL;
}

int main(void) {
    DNode *head = malloc(sizeof(*head));
    head->data = 20; head->prev = NULL; head->next = NULL;

    /* O(1) insert before head (20) -> [10 <-> 20] */
    dll_insert_before_node(&head, head, 10);
    /* O(1) insert after head (10)  -> [10 <-> 15 <-> 20] */
    dll_insert_after_node(head, 15);
    /* Search key 20 and insert 25 after it -> [10 <-> 15 <-> 20 <-> 25] */
    dll_insert_after_node(dll_find_key(head, 20), 25);

    for (DNode *c = head; c != NULL; c = c->next) {
        printf("[%d]%s", c->data, (c->next != NULL) ? " <-> " : " -> NULL\n");
    }

    while (head != NULL) { DNode *t = head; head = head->next; free(t); }
    return 0;
}
```

#### Program Output:
```
[10] <-> [15] <-> [20] <-> [25] -> NULL
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: $\mathcal{O}(1)$ given `DNode *target`; $\mathcal{O}(n)$ if searching by `key` first.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- In `dll_insert_before_node`, when `target == *head_ref` (`target->prev == NULL`), you **must** update `*head_ref = new_node`.

---

### 4.5 Delete from Beginning (Head Deletion)

#### 1. Purpose
Remove and deallocate the first node of the Doubly Linked List in $\mathcal{O}(1)$ time and restore the `head->prev == NULL` invariant on the new head.

#### 2. Preconditions and Assumptions
- `head_ref != NULL` and `*head_ref != NULL`.

#### 3. Step-by-Step Algorithm
1. If `head_ref == NULL || *head_ref == NULL`, return `false`.
2. Save `DNode *victim = *head_ref` and optionally extract `victim->data`.
3. Advance `*head_ref = victim->next`.
4. **Restore Head Invariant**: If `*head_ref != NULL`, set `(*head_ref)->prev = NULL`.
5. Call `free(victim)` and return `true`.

#### 4. Pseudocode
```
ALGORITHM DLL_DeleteFromBeginning(head_ref, removed_out)
INPUT: head_ref (DNode**), removed_out (int* optional)
OUTPUT: true if deleted, false if list was empty
BEGIN
    IF head_ref == NULL OR *head_ref == NULL THEN
        RETURN false
    END IF
    victim <- *head_ref
    IF removed_out != NULL THEN *removed_out <- victim->data END IF

    *head_ref <- victim->next
    IF *head_ref != NULL THEN
        (*head_ref)->prev <- NULL
    END IF

    free(victim)
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
BEFORE dll_delete_from_beginning(&head, &val):
         head (victim)
          |
          v
        +------+------+------+     +------+------+------+
NULL <--| NULL |  10  | next |<--->| prev |  20  | NULL |
        +------+------+------+     +------+------+------+

AFTER (*head_ref = victim->next; (*head_ref)->prev = NULL; free(victim)):
                                    head
                                     |
                                     v
        (freed: [10])              +------+------+------+
                           NULL <--| NULL |  20  | NULL |
                                   +------+------+------+
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct DNode {
    int data;
    struct DNode *prev;
    struct DNode *next;
} DNode;

bool dll_delete_from_beginning(DNode **head_ref, int *removed_out) {
    if (head_ref == NULL || *head_ref == NULL) {
        return false;
    }
    DNode *victim = *head_ref;
    if (removed_out != NULL) {
        *removed_out = victim->data;
    }
    *head_ref = victim->next;
    if (*head_ref != NULL) {
        (*head_ref)->prev = NULL; /* Critical: sever backward dangling link! */
    }
    free(victim);
    return true;
}

int main(void) {
    DNode *n1 = malloc(sizeof(*n1)), *n2 = malloc(sizeof(*n2));
    n1->data = 10; n1->prev = NULL; n1->next = n2;
    n2->data = 20; n2->prev = n1;   n2->next = NULL;
    DNode *head = n1;

    int val = 0;
    dll_delete_from_beginning(&head, &val);
    printf("Deleted %d. New head=%d, new head->prev=%s\n",
           val, head->data, (head->prev == NULL) ? "NULL" : "DANGLING!");

    dll_delete_from_beginning(&head, &val);
    printf("Deleted %d. List is now %s\n", val, (head == NULL) ? "EMPTY" : "non-empty");
    return 0;
}
```

#### Program Output:
```
Deleted 10. New head=20, new head->prev=NULL
Deleted 20. List is now EMPTY
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: $\mathcal{O}(1)$.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- **Forgetting `(*head_ref)->prev = NULL`**: Leaves the new head's `prev` pointer pointing to the freed old head—a classic **dangling pointer**!

---

### 4.6 Delete from End (Tail Deletion)

#### 1. Purpose
Remove and deallocate the last node (tail) of the Doubly Linked List.
- With a `head`-only pointer: $\mathcal{O}(n)$ to locate the tail, then unlink via `tail->prev->next = NULL`.
- With a `DoublyLinkedList` (`head` + `tail`) descriptor: **strictly $\mathcal{O}(1)$ time**!

#### 2. Preconditions and Assumptions
- List is non-empty.

#### 3. Step-by-Step Algorithm
1. If `*head_ref == NULL`, return `false`.
2. Locate `tail` (either directly via `list->tail` in $\mathcal{O}(1)$, or by advancing `while (tail->next != NULL)` in $\mathcal{O}(n)$).
3. Save `tail->data` into `*removed_out` if requested.
4. If `tail->prev != NULL` (list has $\ge 2$ nodes):
   - Set `tail->prev->next = NULL` (new tail's `next` is `NULL`).
5. Else (`tail->prev == NULL`, meaning `tail` was the *only* node in the list):
   - Set `*head_ref = NULL`.
6. Call `free(tail)` and return `true`.

#### 4. Pseudocode
```
ALGORITHM DLL_DeleteFromEnd(head_ref, removed_out)
INPUT: head_ref (DNode**), removed_out (int* optional)
OUTPUT: true if deleted, false if list was empty
BEGIN
    IF head_ref == NULL OR *head_ref == NULL THEN
        RETURN false
    END IF

    tail <- *head_ref
    WHILE tail->next != NULL DO
        tail <- tail->next
    END WHILE

    IF removed_out != NULL THEN *removed_out <- tail->data END IF

    IF tail->prev != NULL THEN
        tail->prev->next <- NULL
    ELSE
        *head_ref <- NULL
    END IF

    free(tail)
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
BEFORE dll_delete_from_end(&head, &val):
         head                      tail->prev                  tail
          |                            |                        |
          v                            v                        v
        +------+------+------+     +------+------+------+     +------+------+------+
NULL <--| NULL |  10  | next |<--->| prev |  20  | next |<--->| prev |  30  | NULL |
        +------+------+------+     +------+------+------+     +------+------+------+

AFTER (tail->prev->next = NULL; free(tail)):
         head                      New Tail
          |                            |
          v                            v
        +------+------+------+     +------+------+------+
NULL <--| NULL |  10  | next |<--->| prev |  20  | NULL |     (freed: [30])
        +------+------+------+     +------+------+------+
```

#### 6. Reusable C Functions (Both Head-Only and $\mathcal{O}(1)$ Tail-Pointer Version) and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct DNode {
    int data;
    struct DNode *prev;
    struct DNode *next;
} DNode;

/* O(1) Tail Deletion when both head_ref and tail_ref are maintained */
bool dll_pop_back_O1(DNode **head_ref, DNode **tail_ref, int *removed_out) {
    if (head_ref == NULL || tail_ref == NULL || *tail_ref == NULL) {
        return false;
    }
    DNode *victim = *tail_ref;
    if (removed_out != NULL) {
        *removed_out = victim->data;
    }
    *tail_ref = victim->prev;
    if (*tail_ref != NULL) {
        (*tail_ref)->next = NULL;
    } else {
        *head_ref = NULL; /* List is now empty */
    }
    free(victim);
    return true;
}

int main(void) {
    DNode *n1 = malloc(sizeof(*n1)), *n2 = malloc(sizeof(*n2));
    n1->data = 100; n1->prev = NULL; n1->next = n2;
    n2->data = 200; n2->prev = n1;   n2->next = NULL;
    DNode *head = n1, *tail = n2;

    int val = 0;
    dll_pop_back_O1(&head, &tail, &val);
    printf("O(1) Popped tail: %d | New tail=%d\n", val, tail->data);

    dll_pop_back_O1(&head, &tail, &val);
    printf("O(1) Popped tail: %d | head=%s, tail=%s\n",
           val, (head == NULL) ? "NULL" : "?", (tail == NULL) ? "NULL" : "?");
    return 0;
}
```

#### Program Output:
```
O(1) Popped tail: 200 | New tail=100
O(1) Popped tail: 100 | head=NULL, tail=NULL
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: **$\mathcal{O}(1)$ with a `tail` pointer** (impossible in a Singly Linked List!); $\mathcal{O}(n)$ if only `head` is stored.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- When the last remaining node is deleted (`victim->prev == NULL`), **both** `*head_ref` and `*tail_ref` must be set to `NULL`.

---

### 4.7 Delete from a Specified Position (1-Based Indexing)

#### 1. Purpose
Locate, unlink, and deallocate the node at 1-based position `pos` ($1 \le \text{pos} \le n$).

#### 2. Preconditions and Assumptions
- `pos >= 1` and `*head_ref != NULL`.

#### 3. Step-by-Step Algorithm
1. Validate `head_ref != NULL && *head_ref != NULL && pos >= 1`.
2. Walk `DNode *target = *head_ref` forward `pos - 1` steps to land directly on the node at position `pos` (unlike SLL, we do **not** need to stop at `pos - 1` because `target` carries its own `target->prev` pointer!).
3. If `target == NULL`, return `false` (`pos > n`).
4. **Bypass `target`'s backward neighbor**:
   - If `target->prev != NULL`, set `target->prev->next = target->next`;
   - Else (`target` is `head`), set `*head_ref = target->next`.
5. **Bypass `target`'s forward neighbor**:
   - If `target->next != NULL`, set `target->next->prev = target->prev`.
6. Free `target` and return `true`.

#### 4. Pseudocode
```
ALGORITHM DLL_DeleteAtPosition(head_ref, pos, removed_out)
INPUT: head_ref (DNode**), pos (int, 1-based), removed_out (int* optional)
OUTPUT: true if deleted, false if pos is out of bounds
BEGIN
    IF head_ref == NULL OR *head_ref == NULL OR pos < 1 THEN
        RETURN false
    END IF

    target <- *head_ref
    FOR i <- 1 TO pos - 1 DO
        IF target == NULL THEN RETURN false END IF
        target <- target->next
    END FOR

    IF target == NULL THEN RETURN false END IF
    IF removed_out != NULL THEN *removed_out <- target->data END IF

    IF target->prev != NULL THEN
        target->prev->next <- target->next
    ELSE
        *head_ref <- target->next
    END IF

    IF target->next != NULL THEN
        target->next->prev <- target->prev
    END IF

    free(target)
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
Deleting target (20) at pos = 2 from [10 <-> 20 <-> 30]:

  target->prev (10)           target (20)           target->next (30)
  +------+------+------+     +------+------+------+     +------+------+------+
  | NULL |  10  | next |---->| prev |  20  | next |---->| prev |  30  | NULL |
  |      |      |      |<----|      |      |      |<----|      |      |      |
  +------+------+------+     +------+------+------+     +------+------+------+
         |         ^                                       |      ^
         |         +---------------------------------------+      |
         |               (2) target->next->prev = target->prev    |
         +--------------------------------------------------------+
                         (1) target->prev->next = target->next
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct DNode {
    int data;
    struct DNode *prev;
    struct DNode *next;
} DNode;

bool dll_delete_at_position(DNode **head_ref, int pos, int *removed_out) {
    if (head_ref == NULL || *head_ref == NULL || pos < 1) {
        return false;
    }
    DNode *target = *head_ref;
    for (int i = 1; i < pos && target != NULL; i++) {
        target = target->next;
    }
    if (target == NULL) {
        return false; /* Position out of range */
    }
    if (removed_out != NULL) {
        *removed_out = target->data;
    }
    if (target->prev != NULL) {
        target->prev->next = target->next;
    } else {
        *head_ref = target->next;
    }
    if (target->next != NULL) {
        target->next->prev = target->prev;
    }
    free(target);
    return true;
}

int main(void) {
    DNode *n1 = malloc(sizeof(*n1)), *n2 = malloc(sizeof(*n2)), *n3 = malloc(sizeof(*n3));
    n1->data = 10; n1->prev = NULL; n1->next = n2;
    n2->data = 20; n2->prev = n1;   n2->next = n3;
    n3->data = 30; n3->prev = n2;   n3->next = NULL;
    DNode *head = n1;

    int val = 0;
    dll_delete_at_position(&head, 2, &val);
    printf("Deleted pos 2 (val=%d). Remaining: ", val);
    for (DNode *c = head; c != NULL; c = c->next) printf("[%d] ", c->data);
    printf("\n");

    while (head != NULL) { DNode *t = head; head = head->next; free(t); }
    return 0;
}
```

#### Program Output:
```
Deleted pos 2 (val=20). Remaining: [10] [30] 
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: Best Case $\mathcal{O}(1)$ (`pos == 1`); Average/Worst Case $\mathcal{O}(n)$.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- Notice how unified the DLL deletion logic is: checking `if (target->prev != NULL)` and `if (target->next != NULL)` handles deleting the head (`pos = 1`), an interior node (`1 < pos < n`), the tail (`pos = n`), and the sole node in a 1-node list with the exact same 6 lines of code!

---

### 4.8 Delete by Key and $\mathcal{O}(1)$ Direct Node-Pointer Deletion

#### 1. Purpose
- **Direct Node-Pointer Deletion (`dll_delete_node`)**: Given a direct pointer `DNode *target` to any node in the list, unlink and `free(target)` in **$\mathcal{O}(1)$ time**.
- **Delete by Key (`dll_delete_by_key`)**: Locate the first node with `data == key` ($\mathcal{O}(n)$ search) and pass its pointer to `dll_delete_node` ($\mathcal{O}(1)$ unlink).

#### 2. Preconditions and Assumptions
- `target` is a valid non-NULL node currently in the list pointed to by `*head_ref`.

#### 3. Step-by-Step Algorithm
1. In `dll_delete_node(DNode **head_ref, DNode *target)`:
   - If `head_ref == NULL || *head_ref == NULL || target == NULL`, return `false`.
   - If `target->prev != NULL`, set `target->prev->next = target->next`; else set `*head_ref = target->next`.
   - If `target->next != NULL`, set `target->next->prev = target->prev`.
   - Call `free(target)` and return `true`.
2. In `dll_delete_by_key(DNode **head_ref, int key)`:
   - Scan `curr` from `*head_ref` until `curr == NULL` or `curr->data == key`.
   - Call `return dll_delete_node(head_ref, curr);`.

#### 4. Pseudocode
```
ALGORITHM DLL_DeleteNode_O1(head_ref, target)
INPUT: head_ref (DNode**), target (DNode* - direct node pointer)
OUTPUT: Unlinks and frees target in O(1) time
BEGIN
    IF head_ref == NULL OR *head_ref == NULL OR target == NULL THEN
        RETURN false
    END IF

    IF target->prev != NULL THEN
        target->prev->next <- target->next
    ELSE
        *head_ref <- target->next
    END IF

    IF target->next != NULL THEN
        target->next->prev <- target->prev
    END IF

    free(target)
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
O(1) Deletion Given Direct Pointer target -> [20]:
No traversal from head is needed! We directly access target->prev [10] and target->next [30]:

  [10 (target->prev)] <=======X=======> [20 (target)] <=======X=======> [30 (target->next)]
          |                                                                     ^
          +---------------------------------------------------------------------+
                     target->prev->next = target->next;
                     target->next->prev = target->prev;
                     free(target);
```

#### 6. Reusable C Functions and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct DNode {
    int data;
    struct DNode *prev;
    struct DNode *next;
} DNode;

/* O(1) Deletion given direct pointer to the node */
bool dll_delete_node(DNode **head_ref, DNode *target) {
    if (head_ref == NULL || *head_ref == NULL || target == NULL) {
        return false;
    }
    if (target->prev != NULL) {
        target->prev->next = target->next;
    } else {
        *head_ref = target->next;
    }
    if (target->next != NULL) {
        target->next->prev = target->prev;
    }
    free(target);
    return true;
}

bool dll_delete_by_key(DNode **head_ref, int key) {
    if (head_ref == NULL) return false;
    for (DNode *curr = *head_ref; curr != NULL; curr = curr->next) {
        if (curr->data == key) {
            return dll_delete_node(head_ref, curr);
        }
    }
    return false;
}

int main(void) {
    DNode *n1 = malloc(sizeof(*n1)), *n2 = malloc(sizeof(*n2)), *n3 = malloc(sizeof(*n3));
    n1->data = 10; n1->prev = NULL; n1->next = n2;
    n2->data = 20; n2->prev = n1;   n2->next = n3;
    n3->data = 30; n3->prev = n2;   n3->next = NULL;
    DNode *head = n1;

    /* Direct O(1) deletion of n2 without searching! */
    dll_delete_node(&head, n2);
    printf("After O(1) direct delete of n2 (20): ");
    for (DNode *c = head; c != NULL; c = c->next) printf("[%d] ", c->data);
    printf("\n");

    /* Delete by key 10 (head) */
    dll_delete_by_key(&head, 10);
    printf("After deleting key 10              : ");
    for (DNode *c = head; c != NULL; c = c->next) printf("[%d] ", c->data);
    printf("\n");

    while (head != NULL) { DNode *t = head; head = head->next; free(t); }
    return 0;
}
```

#### Program Output:
```
After O(1) direct delete of n2 (20): [10] [30] 
After deleting key 10              : [30] 
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**:
  - `dll_delete_node(head_ref, target)`: **$\mathcal{O}(1)$ worst-case** (the defining superpower of Doubly Linked Lists!).
  - `dll_delete_by_key(head_ref, key)`: $\mathcal{O}(n)$ to search + $\mathcal{O}(1)$ to unlink = $\mathcal{O}(n)$.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- Never use `target` after calling `dll_delete_node(&head, target)`. If iterating through the list and deleting matching nodes in a loop, always save `DNode *next_node = curr->next;` **before** calling `dll_delete_node(&head, curr)`.

---

### 4.9 Forward/Backward Traversal, Searching, Counting, and Updating

#### 1. Purpose
- **Forward Traversal**: Visit nodes from `head` to `tail` via `curr = curr->next`.
- **Backward Traversal**: Visit nodes from `tail` down to `head` via `curr = curr->prev`.
- **Search / Count / Update**: Locate keys, count nodes, and modify payloads.

#### 2. Preconditions and Assumptions
- Bidirectional invariants hold across all nodes.

#### 3. Step-by-Step Algorithm
1. For forward traversal, start at `curr = head` and advance `curr = curr->next` until `curr == NULL`.
2. For backward traversal (when only `head` is given), first advance to the `tail` (`while (curr->next != NULL)`), then walk backward via `curr = curr->prev` until `curr == NULL`.

#### 4. Pseudocode
```
ALGORITHM DLL_TraverseBackward(head)
INPUT: head (const DNode*)
OUTPUT: Prints elements from tail to head
BEGIN
    IF head == NULL THEN RETURN END IF
    curr <- head
    WHILE curr->next != NULL DO
        curr <- curr->next
    END WHILE
    WHILE curr != NULL DO
        PRINT curr->data
        curr <- curr->prev
    END WHILE
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
Forward Walk (curr = curr->next):
  head ---> [10] ===> [20] ===> [30] ---> NULL

Backward Walk (curr = curr->prev):
  NULL <--- [10] <=== [20] <=== [30] <--- tail
```

#### 6. Reusable C Functions and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct DNode {
    int data;
    struct DNode *prev;
    struct DNode *next;
} DNode;

void dll_print_forward(const DNode *head) {
    printf("Forward : NULL <- ");
    for (const DNode *c = head; c != NULL; c = c->next) {
        printf("[%d]%s", c->data, (c->next != NULL) ? " <-> " : " -> ");
    }
    printf("NULL\n");
}

void dll_print_backward(const DNode *head) {
    printf("Backward: NULL <- ");
    if (head == NULL) {
        printf("NULL\n");
        return;
    }
    const DNode *tail = head;
    while (tail->next != NULL) {
        tail = tail->next;
    }
    for (const DNode *c = tail; c != NULL; c = c->prev) {
        printf("[%d]%s", c->data, (c->prev != NULL) ? " <-> " : " -> ");
    }
    printf("NULL\n");
}

size_t dll_count(const DNode *head) {
    size_t count = 0;
    for (const DNode *c = head; c != NULL; c = c->next) count++;
    return count;
}

bool dll_update_key(DNode *head, int old_key, int new_key) {
    for (DNode *c = head; c != NULL; c = c->next) {
        if (c->data == old_key) {
            c->data = new_key;
            return true;
        }
    }
    return false;
}

int main(void) {
    DNode *n1 = malloc(sizeof(*n1)), *n2 = malloc(sizeof(*n2)), *n3 = malloc(sizeof(*n3));
    n1->data = 10; n1->prev = NULL; n1->next = n2;
    n2->data = 20; n2->prev = n1;   n2->next = n3;
    n3->data = 30; n3->prev = n2;   n3->next = NULL;
    DNode *head = n1;

    printf("Count = %zu\n", dll_count(head));
    dll_update_key(head, 20, 25);
    dll_print_forward(head);
    dll_print_backward(head);

    while (head != NULL) { DNode *t = head; head = head->next; free(t); }
    return 0;
}
```

#### Program Output:
```
Count = 3
Forward : NULL <- [10] <-> [25] <-> [30] -> NULL
Backward: NULL <- [30] <-> [25] <-> [10] -> NULL
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: $\Theta(n)$.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- Backward traversal is the ultimate **integrity check** for a Doubly Linked List: if forward traversal prints `[10, 25, 30]` and backward traversal prints `[30, 25, 10]`, every single `prev` and `next` pointer in the list is verified to be wired correctly!

---

### 4.10 Reversing a Doubly Linked List In-Place

#### 1. Purpose
Reverse a Doubly Linked List in-place in $\Theta(n)$ time and $\mathcal{O}(1)$ auxiliary space by swapping the `prev` and `next` pointers of every node and updating `head`.

#### 2. Preconditions and Assumptions
- `head_ref` is non-NULL.

#### 3. Step-by-Step Algorithm
1. If `head_ref == NULL || *head_ref == NULL || (*head_ref)->next == NULL`, return (empty or 1-node list is already reversed).
2. Initialize `DNode *curr = *head_ref` and `DNode *temp = NULL`.
3. While `curr != NULL`:
   - Save `temp = curr->prev` (note: before swapping the last node, `curr->prev` points to the second-to-last node).
   - Swap `curr->prev` and `curr->next`:
     - `curr->prev = curr->next;`
     - `curr->next = temp;`
   - **Advance `curr` to the next node in the original order**: Since we just swapped `prev` and `next`, the old `curr->next` is now stored in `curr->prev`! So advance via `curr = curr->prev;`.
4. After the loop finishes, `temp` holds the `prev` pointer of the original tail node, so `temp->prev` points to the **original tail node** (the new head). Update `*head_ref = temp->prev`.

#### 4. Pseudocode
```
ALGORITHM DLL_ReverseInPlace(head_ref)
INPUT: head_ref (DNode**)
OUTPUT: Reverses the Doubly Linked List in-place
BEGIN
    IF head_ref == NULL OR *head_ref == NULL OR (*head_ref)->next == NULL THEN
        RETURN
    END IF

    temp <- NULL
    curr <- *head_ref

    WHILE curr != NULL DO
        temp       <- curr->prev
        curr->prev <- curr->next
        curr->next <- temp
        curr       <- curr->prev   /* Move to old curr->next! */
    END WHILE

    IF temp != NULL THEN
        *head_ref <- temp->prev
    END IF
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
BEFORE Reverse:
  Node 10: (prev=NULL, next=20)
  Node 20: (prev=10,   next=30)
  Node 30: (prev=20,   next=NULL)

AFTER Swapping (prev <-> next) inside every node:
  Node 10: (prev=20,   next=NULL)
  Node 20: (prev=30,   next=10)
  Node 30: (prev=NULL, next=20)  <--- New Head (*head_ref = temp->prev)

Resulting Chain from New Head:
  NULL <- [30] <-> [20] <-> [10] -> NULL
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct DNode {
    int data;
    struct DNode *prev;
    struct DNode *next;
} DNode;

void dll_reverse(DNode **head_ref) {
    if (head_ref == NULL || *head_ref == NULL || (*head_ref)->next == NULL) {
        return;
    }
    DNode *temp = NULL;
    DNode *curr = *head_ref;

    while (curr != NULL) {
        temp = curr->prev;
        curr->prev = curr->next;
        curr->next = temp;
        curr = curr->prev; /* Advance using swapped prev pointer */
    }
    if (temp != NULL) {
        *head_ref = temp->prev;
    }
}

int main(void) {
    DNode *n1 = malloc(sizeof(*n1)), *n2 = malloc(sizeof(*n2)), *n3 = malloc(sizeof(*n3));
    n1->data = 10; n1->prev = NULL; n1->next = n2;
    n2->data = 20; n2->prev = n1;   n2->next = n3;
    n3->data = 30; n3->prev = n2;   n3->next = NULL;
    DNode *head = n1;

    dll_reverse(&head);

    printf("Reversed Forward : ");
    for (DNode *c = head; c != NULL; c = c->next) printf("[%d] ", c->data);
    printf("\n");

    while (head != NULL) { DNode *t = head; head = head->next; free(t); }
    return 0;
}
```

#### Program Output:
```
Reversed Forward : [30] [20] [10] 
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: $\Theta(n)$ — Visits each node once and performs 3 pointer assignments.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$ — Only 2 local pointer variables (`curr`, `temp`).

#### 8. Edge Cases and Common Mistakes
- **Advancing via `curr = curr->next` after swapping**: After `curr->prev` and `curr->next` are swapped, `curr->next` points *backward*! You must advance via `curr = curr->prev`.
- **Single-node list (`n = 1`)**: For a 1-node list, `temp` remains `NULL` after the 1st iteration because `head->prev` was `NULL`. Guarding `if (temp != NULL)` (or returning early when `(*head_ref)->next == NULL`) prevents dereferencing `temp->prev` when `temp == NULL`!

---

### 4.11 Complete C11 Demonstration Program

Below is a complete, compilable C11 program showcasing a full-featured Doubly Linked List with head/tail insertion, positional insertion, $\mathcal{O}(1)$ node-pointer deletion, bidirectional verification, and in-place reversal.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct DNode {
    int data;
    struct DNode *prev;
    struct DNode *next;
} DNode;

DNode *dll_create_node(int value) {
    DNode *node = malloc(sizeof(*node));
    if (node != NULL) {
        node->data = value;
        node->prev = NULL;
        node->next = NULL;
    }
    return node;
}

bool dll_push_front(DNode **head_ref, int value) {
    DNode *n = dll_create_node(value);
    if (n == NULL) return false;
    n->next = *head_ref;
    if (*head_ref != NULL) (*head_ref)->prev = n;
    *head_ref = n;
    return true;
}

bool dll_push_back(DNode **head_ref, int value) {
    DNode *n = dll_create_node(value);
    if (n == NULL) return false;
    if (*head_ref == NULL) {
        *head_ref = n;
        return true;
    }
    DNode *tail = *head_ref;
    while (tail->next != NULL) tail = tail->next;
    tail->next = n;
    n->prev = tail;
    return true;
}

bool dll_insert_pos(DNode **head_ref, int pos, int value) {
    if (pos < 1) return false;
    if (pos == 1) return dll_push_front(head_ref, value);
    DNode *prev = *head_ref;
    for (int i = 1; i < pos - 1 && prev != NULL; i++) prev = prev->next;
    if (prev == NULL) return false;
    DNode *n = dll_create_node(value);
    if (n == NULL) return false;
    n->next = prev->next;
    n->prev = prev;
    if (prev->next != NULL) prev->next->prev = n;
    prev->next = n;
    return true;
}

bool dll_delete_node_O1(DNode **head_ref, DNode *target) {
    if (head_ref == NULL || *head_ref == NULL || target == NULL) return false;
    if (target->prev != NULL) target->prev->next = target->next;
    else *head_ref = target->next;
    if (target->next != NULL) target->next->prev = target->prev;
    free(target);
    return true;
}

void dll_reverse_list(DNode **head_ref) {
    if (head_ref == NULL || *head_ref == NULL || (*head_ref)->next == NULL) return;
    DNode *temp = NULL, *curr = *head_ref;
    while (curr != NULL) {
        temp = curr->prev;
        curr->prev = curr->next;
        curr->next = temp;
        curr = curr->prev;
    }
    if (temp != NULL) *head_ref = temp->prev;
}

void dll_display_both_ways(const char *title, const DNode *head) {
    printf("%s\n", title);
    printf("  Fwd: NULL <- ");
    const DNode *tail = NULL;
    for (const DNode *c = head; c != NULL; c = c->next) {
        printf("[%d]%s", c->data, (c->next != NULL) ? " <-> " : " -> ");
        if (c->next == NULL) tail = c;
    }
    printf("NULL\n  Bwd: NULL <- ");
    for (const DNode *c = tail; c != NULL; c = c->prev) {
        printf("[%d]%s", c->data, (c->prev != NULL) ? " <-> " : " -> ");
    }
    printf("NULL\n");
}

void dll_destroy(DNode **head_ref) {
    while (*head_ref != NULL) {
        DNode *tmp = *head_ref;
        *head_ref = tmp->next;
        free(tmp);
    }
}

int main(void) {
    DNode *head = NULL;

    dll_push_back(&head, 20);
    dll_push_back(&head, 40);
    dll_push_front(&head, 10);
    dll_insert_pos(&head, 3, 30);
    dll_display_both_ways("1. After Building [10, 20, 30, 40]:", head);

    /* Delete second node (20) in O(1) time via direct node pointer */
    dll_delete_node_O1(&head, head->next);
    dll_display_both_ways("2. After O(1) Direct Delete of head->next (20):", head);

    dll_reverse_list(&head);
    dll_display_both_ways("3. After In-Place Reversal:", head);

    dll_destroy(&head);
    printf("4. After Destroy: head == %s\n", (head == NULL) ? "NULL" : "non-NULL");
    return 0;
}
```

#### Program Output:
```
1. After Building [10, 20, 30, 40]:
  Fwd: NULL <- [10] <-> [20] <-> [30] <-> [40] -> NULL
  Bwd: NULL <- [40] <-> [30] <-> [20] <-> [10] -> NULL
2. After O(1) Direct Delete of head->next (20):
  Fwd: NULL <- [10] <-> [30] <-> [40] -> NULL
  Bwd: NULL <- [40] <-> [30] <-> [10] -> NULL
3. After In-Place Reversal:
  Fwd: NULL <- [40] <-> [30] <-> [10] -> NULL
  Bwd: NULL <- [10] <-> [30] <-> [40] -> NULL
4. After Destroy: head == NULL
```

---

## 5. Complexity Analysis and Comparison Tables

### 5.1 Time and Space Complexity of Doubly Linked List Operations
Let $n$ be the number of nodes in the Doubly Linked List and $k$ be the 1-based position.

| Operation | Best Case Time | Average Case Time | Worst Case Time | Auxiliary Space | Total Space |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Insert at Beginning (Head)** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Insert at End (With `tail` ptr)** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Insert at End (No `tail` ptr)** | $\mathcal{O}(1)$ ($n=0$) | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Insert at Position $k$** | $\mathcal{O}(1)$ ($k=1$) | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Insert Before Given `DNode*`** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Insert After Given `DNode*`** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Delete from Beginning (Head)** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Delete from End (With `tail` ptr)** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Delete from End (No `tail` ptr)** | $\mathcal{O}(1)$ ($n=1$) | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Delete Given Direct `DNode*`** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Delete by Key (Search + Unlink)** | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Forward / Backward Traversal** | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Reverse List In-Place** | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |

*Note: If the list descriptor maintains `size` and `tail`, positional access at index $k$ can be optimized to $\mathcal{O}(\min(k, n - k))$ by starting from `head` when $k \le n/2$ and from `tail` when $k > n/2$ (at most $n/2$ hops, still $\mathcal{O}(n)$ asymptotically).*

### 5.2 Comparison: Singly Linked List vs. Doubly Linked List vs. Dynamic Array

| Feature / Operation | Singly Linked List (SLL) | Doubly Linked List (DLL) | Dynamic Array (`realloc`) |
| :--- | :--- | :--- | :--- |
| **Links per Node** | $1$ (`next`) | $2$ (`prev`, `next`) | $0$ (contiguous buffer) |
| **64-Bit Node Size (`int` payload)** | $16\text{ bytes}$ | $24\text{ bytes}$ | $4\text{ bytes}$ per element |
| **Traversal Direction** | Forward only | Forward and Backward | Forward, Backward, and Random Jump |
| **Index Access (`get(i)`)** | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ (max $n/2$ hops with `tail`) | $\mathcal{O}(1)$ |
| **Insert at Head (`push_front`)** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Insert at Tail (with `tail` ptr)** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ amortized, $\mathcal{O}(n)$ worst |
| **Delete at Head (`pop_front`)** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **Delete at Tail (with `tail` ptr)** | **$\mathcal{O}(n)$** (must find predecessor) | **$\mathcal{O}(1)$** (`tail = tail->prev`) | $\mathcal{O}(1)$ |
| **Insert Before Given `Node*`** | **$\mathcal{O}(n)$** | **$\mathcal{O}(1)$** | $\mathcal{O}(n)$ |
| **Delete Given Direct `Node*`** | **$\mathcal{O}(n)$** | **$\mathcal{O}(1)$** | $\mathcal{O}(n)$ |
| **Best Suited For** | Stacks, Hash Chains, Simple Queues | LRU Caches, Deques, Undo/Redo, OS Schedulers | Random access, Sorting, Cache-critical loops |

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Clobbering `prev->next` During 4-Pointer Insertion
If you assign `prev_node->next = new_node` *before* setting `new_node->next = prev_node->next` and `prev_node->next->prev = new_node`, you lose the address of the successor node!

```c
/* BUG: Overwrites prev_node->next prematurely! */
prev_node->next = new_node;
new_node->next = prev_node->next; /* Points new_node->next to new_node itself! */

/* FIX: Always wire new_node's two pointers FIRST, then neighbor back-link, then prev->next */
new_node->next = prev_node->next;     /* 1. Outgoing forward link  */
new_node->prev = prev_node;           /* 2. Outgoing backward link */
if (prev_node->next != NULL) {
    prev_node->next->prev = new_node; /* 3. Successor's prev link  */
}
prev_node->next = new_node;           /* 4. Predecessor's next link*/
```

### 6.2 Forgetting to Set `head->prev = NULL` After Head Deletion
When deleting the `head` node of a Doubly Linked List, advancing `*head_ref = victim->next` is only half the job. The new head's `prev` pointer still points to `victim`!

```c
/* BUG: Leaves (*head_ref)->prev pointing to freed memory (Dangling Pointer!) */
DNode *victim = *head_ref;
*head_ref = victim->next;
free(victim);

/* FIX: Explicitly null out the new head's prev pointer if list is non-empty */
DNode *victim = *head_ref;
*head_ref = victim->next;
if (*head_ref != NULL) {
    (*head_ref)->prev = NULL;
}
free(victim);
```

### 6.3 Unchecked `target->next->prev` Dereference When Deleting the Tail Node
When unlinking `target`, assuming `target->next` is always non-NULL crashes whenever `target` happens to be the last (tail) node.

```c
/* BUG: Crashes with SIGSEGV when target is the tail node (target->next == NULL) */
target->prev->next = target->next;
target->next->prev = target->prev; /* NULL dereference if target->next == NULL! */

/* FIX: Guard both target->prev and target->next against NULL */
if (target->prev != NULL) target->prev->next = target->next;
else *head_ref = target->next;

if (target->next != NULL) target->next->prev = target->prev;
```

### 6.4 Dangling `tail` Pointer When Deleting the Sole Node in a Descriptor
If your list struct maintains both `head` and `tail`, deleting the last remaining node via `pop_front` must also set `list->tail = NULL`.

```c
/* BUG: list->tail still points to the freed node when list becomes empty! */
DNode *victim = list->head;
list->head = victim->next;
if (list->head != NULL) list->head->prev = NULL;
free(victim);

/* FIX: When list->head becomes NULL, list->tail MUST also be set to NULL */
DNode *victim = list->head;
list->head = victim->next;
if (list->head != NULL) {
    list->head->prev = NULL;
} else {
    list->tail = NULL; /* Synchronize tail on empty transition! */
}
free(victim);
```

### 6.5 Failing to Update `*head_ref` at the End of In-Place Reversal
Swapping `curr->prev` and `curr->next` inside the `while` loop reverses all internal links, but leaves `*head_ref` pointing to the original first node (which is now the tail!).

```c
/* BUG: Reverses links, but *head_ref still points to the old head (now tail)! */
while (curr != NULL) {
    temp = curr->prev;
    curr->prev = curr->next;
    curr->next = temp;
    curr = curr->prev;
}
/* Missing *head_ref update! Forward traversal now prints only 1 node! */

/* FIX: Update *head_ref to temp->prev (guarded for 0/1-node lists) */
if (temp != NULL) {
    *head_ref = temp->prev;
}
```

### 6.6 Memory Leak / Use-After-Free During Unlink
Calling `free(target)` before reading `target->prev` and `target->next` invokes undefined Use-After-Free behavior. Always complete all pointer rewiring *before* calling `free(target)`.

### 6.7 Boundary Conditions Checklist
- [ ] **Empty List (`head == NULL`)**: Does every insert, delete, search, and reverse function handle `head == NULL` safely?
- [ ] **Single-Node List (`head->prev == NULL && head->next == NULL`)**:
  - Does deleting the sole node leave `head == NULL` (and `tail == NULL`)?
  - Does reversing a 1-node list avoid dereferencing `temp->prev` when `temp == NULL`?
- [ ] **Head Boundary (`target->prev == NULL`)**: Does inserting before `head` or deleting `head` update `*head_ref` and maintain `(*head_ref)->prev == NULL`?
- [ ] **Tail Boundary (`target->next == NULL`)**: Does inserting after `tail` or deleting `tail` guard `if (target->next != NULL)` and maintain `new_tail->next == NULL`?
- [ ] **Bidirectional Symmetry Check**: After any mutation, does a forward walk followed by a backward walk visit the exact same nodes in reverse order?

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways
1. Each **Doubly Linked List (`DNode`)** stores `prev`, `data`, and `next`, trading one extra pointer per node ($24\text{ bytes}$ vs. $16\text{ bytes}$ on 64-bit) for full bidirectional navigation.
2. Given a direct pointer `DNode *target`, **deleting `target` or inserting before/after `target` takes $\mathcal{O}(1)$ worst-case time**—making Doubly Linked Lists essential for LRU caches and OS schedulers.
3. Inserting a node between `prev_node` and `prev_node->next` requires **4 pointer updates**; always wire `new_node->next` and `new_node->prev` *before* overwriting `prev_node->next`.
4. Unlinking any node `target` requires **2 guarded neighbor updates**: `if (target->prev) target->prev->next = target->next; else *head_ref = target->next;` and `if (target->next) target->next->prev = target->prev;`.
5. In-place reversal swaps `curr->prev` and `curr->next` for each node, advances via `curr = curr->prev`, and sets `*head_ref = temp->prev`.

### 7.2 Essential Doubly Linked List Invariants
- **Head Boundary Invariant**: $\text{head} \neq \text{NULL} \implies \text{head}\to\text{prev} == \text{NULL}$.
- **Tail Boundary Invariant**: $\text{tail} \neq \text{NULL} \implies \text{tail}\to\text{next} == \text{NULL}$.
- **Reciprocity Invariant**: For every pair of adjacent nodes $(u, v)$ in the list, $u\to\text{next} == v \iff v\to\text{prev} == u$.

### 7.3 Quick Self-Test Checklist
- [ ] Why can a Doubly Linked List delete a node in $\mathcal{O}(1)$ time when given `DNode *target`, whereas a Singly Linked List requires $\mathcal{O}(n)$ time?
- [ ] Can you write down the exact 4 lines of C code to insert `new_node` after `prev_node` in the correct order?
- [ ] Why do we advance via `curr = curr->prev` instead of `curr = curr->next` inside the `dll_reverse` loop?
- [ ] What happens if you forget `if (*head_ref != NULL) (*head_ref)->prev = NULL;` when deleting the first node of a Doubly Linked List?
- [ ] How do an LRU Cache's Hash Map and Doubly Linked List work together to achieve $\mathcal{O}(1)$ `get` and `put` operations?

### 7.4 Related Study Notes
- **Foundations**: [01 — C Structures, Pointers, and Memory](./01-C-Structures-Pointers-and-Memory.md)
- **Previous Module**: [02 — Singly Linked Lists](./02-Singly-Linked-Lists.md)
- **Next Module**: [04 — Circular Linked Lists](./04-Circular-Linked-Lists.md)
- **Stacks**: [05 — Stacks](./07-Stacks-Array-and-Linked-List.md)
- **Course Index**: [DSA Notes Root README](../README.md)
