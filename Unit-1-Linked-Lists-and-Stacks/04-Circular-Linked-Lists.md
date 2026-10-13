# Circular Linked Lists
---

## Table of Contents
1. [Introduction](#1-introduction)
   - 1.1 [What is a Circular Linked List?](#11-what-is-a-circular-linked-list)
   - 1.2 [Why Circular Linked Lists Are Needed](#12-why-circular-linked-lists-are-needed)
   - 1.3 [Relationship to Prior Linked List Structures](#13-relationship-to-prior-linked-list-structures)
   - 1.4 [Learning Objectives](#14-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Formal Definition and Structural Variants](#21-formal-definition-and-structural-variants)
   - 2.2 [Terminology Table](#22-terminology-table)
   - 2.3 [Mathematical Properties and Structural Invariants](#23-mathematical-properties-and-structural-invariants)
   - 2.4 [Conceptual ASCII Diagrams](#24-conceptual-ascii-diagrams)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [C Structure Declarations](#31-c-structure-declarations)
   - 3.2 [Head-Pointer vs. Tail-Pointer (`last`) Representation](#32-head-pointer-vs-tail-pointer-last-representation)
   - 3.3 [Memory Layout in the Heap](#33-memory-layout-in-the-heap)
   - 3.4 [Static vs. Dynamic Storage, Allocation, and Deallocation](#34-static-vs-dynamic-storage-allocation-and-deallocation)
   - 3.5 [Pros and Cons of Representations](#35-pros-and-cons-of-representations)
4. [Operations and Algorithms](#4-operations-and-algorithms)
   - 4.1 [Insertion at the Beginning](#41-insertion-at-the-beginning)
   - 4.2 [Insertion at the End](#42-insertion-at-the-end)
   - 4.3 [Insertion at a Specified Position](#43-insertion-at-a-specified-position)
   - 4.4 [Insertion After a Given Key/Value](#44-insertion-after-a-given-keyvalue)
   - 4.5 [Deletion from the Beginning](#45-deletion-from-the-beginning)
   - 4.6 [Deletion from the End](#46-deletion-from-the-end)
   - 4.7 [Deletion from a Specified Position](#47-deletion-from-a-specified-position)
   - 4.8 [Deletion by Value/Key](#48-deletion-by-valuekey)
   - 4.9 [Traversal (`do-while`), Display, and Counting Nodes](#49-traversal-do-while-display-and-counting-nodes)
   - 4.10 [Searching and Updating a Node](#410-searching-and-updating-a-node)
   - 4.11 [Reversing a Circular Linked List](#411-reversing-a-circular-linked-list)
   - 4.12 [Complete, Compilable C11 Program](#412-complete-compilable-c11-program)
5. [Complexity Analysis and Comparison Tables](#5-complexity-analysis-and-comparison-tables)
   - 5.1 [Operation Complexity Summary](#51-operation-complexity-summary)
   - 5.2 [Comparison: Singly vs. Doubly vs. Circular Linked Lists](#52-comparison-singly-vs-doubly-vs-circular-linked-lists)
   - 5.3 [Real-World Applications](#53-real-world-applications)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Pitfall 1: Infinite Loops Using `while (curr != NULL)`](#61-pitfall-1-infinite-loops-using-while-curr--null)
   - 6.2 [Pitfall 2: Forgetting to Update `tail->next` on Head Modification](#62-pitfall-2-forgetting-to-update-tail-next-on-head-modification)
   - 6.3 [Pitfall 3: Mishandling the Single-Node Deletion Case](#63-pitfall-3-mishandling-the-single-node-deletion-case)
   - 6.4 [Pitfall 4: Infinite Loop or Use-After-Free During List Deallocation](#64-pitfall-4-infinite-loop-or-use-after-free-during-list-deallocation)
   - 6.5 [Boundary Conditions Checklist](#65-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)
   - 7.1 [Key Takeaways](#71-key-takeaways)
   - 7.2 [Essential Invariants](#72-essential-invariants)
   - 7.3 [Quick Self-Test Checklist](#73-quick-self-test-checklist)
   - 7.4 [Related Notes and Navigation](#74-related-notes-and-navigation)

---

## 1. Introduction

### 1.1 What is a Circular Linked List?
A **Circular Linked List (CLL)** is a variation of a linked list in which the last node (the **tail**) does not store a `NULL` pointer to mark the end of the sequence. Instead, its `next` pointer points back to the first node (the **head**) of the list, forming a closed, continuous ring of nodes.

In a **Singly Circular Linked List (SCLL)**, traversal proceeds in one direction around the circle. In a **Doubly Circular Linked List (DCLL)**, the tail's `next` pointer points to the head AND the head's `prev` pointer points back to the tail, allowing bidirectional circular traversal.

### 1.2 Why Circular Linked Lists Are Needed
In a standard singly linked list (`head -> N1 -> N2 -> ... -> NULL`), once a traversal pointer advances past a node, earlier nodes cannot be reached without restarting from `head`. Furthermore:
1. **Cyclic / Round-Robin Workloads**: Many systems require cycling repeatedly through a collection of active entities—such as CPU time-slicing (Round-Robin scheduling), multiplayer game turn management, or repeating media playlists—without special-casing the "end of list" reset.
2. **Reachability from Any Node**: Starting from *any* node in a circular linked list, every other node in the list can be visited merely by following `next` pointers.
3. **$O(1)$ Insertion at Both Ends Using a Single Pointer**: In a standard singly linked list with only a `head` pointer, inserting at the tail requires an $O(n)$ traversal. In a circular linked list maintained via a **`tail` (`last`) pointer**, both the tail (`tail`) and the head (`tail->next`) are immediately accessible in $O(1)$ time! Consequently, **insertion at the beginning, insertion at the end, and deletion from the beginning all execute in $O(1)$ worst-case time** using only a single external pointer.

### 1.3 Relationship to Prior Linked List Structures
- **Singly Linked List (SLL)**: Terminates with `NULL`. Has a strict linear order with a distinct beginning and end.
- **Doubly Linked List (DLL)**: Adds a `prev` pointer per node so predecessors can be accessed in $O(1)$ time, paying an extra pointer of memory per node.
- **Circular Linked List (CLL)**: Eliminates `NULL` pointers inside non-empty lists by wiring the boundary pointers into a cycle. Every node has a well-defined successor.

### 1.4 Learning Objectives
After studying these notes, you will be able to:
- Define singly and doubly circular linked lists and state their structural invariants.
- Explain why maintaining an external `tail` (`last`) pointer is superior to maintaining an external `head` pointer for Singly Circular Linked Lists.
- Implement all core operations—insertion (front, end, position, after key), deletion (front, end, position, by key), `do-while` traversal, search, update, in-place reversal, and safe memory deallocation—in standard C11.
- Analyze the best-, average-, and worst-case time and space complexities of every operation.
- Avoid classic circular-list bugs such as infinite loops and single-node dangling pointers.

---

## 2. Definitions and Terminology

### 2.1 Formal Definition and Structural Variants
Let $L = (v_1, v_2, \dots, v_n)$ be a finite sequence of $n \ge 0$ dynamically allocated nodes.
- **Singly Circular Linked List (SCLL)**: Each node $v_i$ stores a data payload $\text{data}(v_i)$ and a single link pointer $\text{next}(v_i)$ such that:
  $$\text{next}(v_i) = \begin{cases} v_{i+1} & \text{if } 1 \le i < n \\ v_1 & \text{if } i = n \end{cases}$$
- **Doubly Circular Linked List (DCLL)**: Each node $v_i$ additionally stores a predecessor link $\text{prev}(v_i)$ such that:
  $$\text{prev}(v_i) = \begin{cases} v_{i-1} & \text{if } 1 < i \le n \\ v_n & \text{if } i = 1 \end{cases}$$

### 2.2 Terminology Table

| Term | Definition |
| :--- | :--- |
| **Circular Linked List (CLL)** | A linked list in which the last node's `next` pointer points back to the first node instead of `NULL`. |
| **Tail / Last Pointer (`tail`)** | The external pointer pointing to the last node $v_n$ of the list. When `tail != NULL`, `tail->next` is the first node (head). |
| **Head Node (`tail->next`)** | The logically first node $v_1$ in the circular sequence. |
| **Self-Loop (Single-Node Cycle)** | A circular list with $n = 1$ node where `node->next == node`. |
| **Empty Circular List** | A circular list with $n = 0$ nodes, represented by `tail == NULL` (or `head == NULL`). |
| **Cycle Period ($n$)** | The minimum positive integer $n$ such that applying `->next` $n$ times from any node $v$ returns to $v$ itself. |
| **Sentinel / Header Node** | An optional dummy node placed permanently in the ring to eliminate `NULL` checks for empty lists. |

### 2.3 Mathematical Properties and Structural Invariants
For a valid Singly Circular Linked List of size $n$ managed via an external pointer `tail`:

1. **Empty List Invariant ($n = 0$)**:
   $$\text{tail} == \text{NULL}$$
2. **Non-Empty List No-NULL Invariant ($n \ge 1$)**:
   $$\forall v \in L, \quad v\text{->next} \neq \text{NULL}$$
   No node in a non-empty circular linked list ever contains a `NULL` `next` pointer.
3. **Single-Node Invariant ($n = 1$)**:
   $$\text{tail} \neq \text{NULL} \;\land\; \text{tail->next} == \text{tail}$$
4. **Head-Tail Duality Invariant ($n \ge 1$)**:
   $$\text{head} = \text{tail->next}$$
5. **Periodic Cycle Property ($n \ge 1$)**:
   Define $\text{next}^0(v) = v$ and $\text{next}^k(v) = \text{next}(\text{next}^{k-1}(v))$. Then for every node $v \in L$:
   $$\text{next}^n(v) = v \quad \text{and} \quad \forall k \in \{1, 2, \dots, n-1\}, \;\text{next}^k(v) \neq v$$

### 2.4 Conceptual ASCII Diagrams

#### 1. Empty Circular Linked List ($n = 0$)
```text
tail ──► NULL
```

#### 2. Single-Node Singly Circular Linked List ($n = 1$)
```text
         ┌──────────────┐
         ▼              │
      +------+------+   │
      |  42  | next |───┘
      +------+------+
         ▲
         │
        tail  (Note: head == tail->next == tail)
```

#### 3. Multi-Node Singly Circular Linked List ($n = 4$)
```text
       (head = tail->next)
              │
              ▼
         +----+----+     +----+----+     +----+----+     +----+----+
    ┌───►| 10 | o──|────►| 20 | o──|────►| 30 | o──|────►| 40 | o──|───┐
    │    +----+----+     +----+----+     +----+----+     +----+----+   │
    │                                                         ▲        │
    │                                                         │        │
    │                                                        tail      │
    └──────────────────────────────────────────────────────────────────┘
```

#### 4. Doubly Circular Linked List ($n = 3$)
```text
       ┌───────────────────────────────────────────────────────────────┐
       │                                                               ▼
       │    +----+----+----+     +----+----+----+     +----+----+----+ │
  head─┼───►| o  | 10 | o──|────►| o  | 20 | o──|────►| o  | 30 | o──|─┼─┐
       │    +-|--+----+----+     +-|--+----+----+     +-|--+----+----+ │ │
       │      │                    │   ▲                │   ▲          │ │
       │      │                    └───┼────────────────┘   │          │ │
       │      │                        └────────────────────┘          │ │
       │      └────────────────────────────────────────────────────────┘ │
       └─────────────────────────────────────────────────────────────────┘
```

---

## 3. Representation and Memory Layout in C

### 3.1 C Structure Declarations
A Singly Circular Linked List uses the exact same node `struct` as a standard Singly Linked List. The circularity is a **structural invariant of how pointers are wired**, not a different struct field layout.

```c
/* Node definition for a Singly Circular Linked List (SCLL) */
typedef struct CLLNode {
    int data;                   /* Payload stored in the node */
    struct CLLNode *next;       /* Pointer to the next node in the circle */
} CLLNode;

/* Optional: Node definition for a Doubly Circular Linked List (DCLL) */
typedef struct DCLLNode {
    int data;                   /* Payload stored in the node */
    struct DCLLNode *prev;      /* Pointer to the predecessor node */
    struct DCLLNode *next;      /* Pointer to the successor node */
} DCLLNode;
```

### 3.2 Head-Pointer vs. Tail-Pointer (`last`) Representation

A critical engineering decision when implementing a Singly Circular Linked List is whether to represent the list using an external `head` pointer or an external `tail` (`last`) pointer:

#### Case A: Maintaining Only a `head` Pointer
Suppose we only keep `CLLNode *head`.
- To insert a new node at the **beginning** (or at the **end**), we must update the last node's `next` pointer (`tail->next = newNode`) so the circle remains closed.
- Because we only have `head`, finding the last node requires traversing all $n$ nodes until `curr->next == head`.
- Therefore, **both `insertAtBeginning` and `insertAtEnd` take $O(n)$ time** if only `head` is stored!

#### Case B: Maintaining Only a `tail` (`last`) Pointer (Recommended Standard)
Suppose we keep `CLLNode *tail` pointing to the last node:
- The **last node** is directly accessible via `tail` in $O(1)$ time.
- The **first node (head)** is directly accessible via `tail->next` in $O(1)$ time!
- Inserting at the beginning simply splices the new node between `tail` and `tail->next` in $O(1)$ time.
- Inserting at the end splices the new node between `tail` and `tail->next`, and then updates `tail = newNode` in $O(1)$ time.
- Deleting from the beginning removes `tail->next` and updates `tail->next = oldHead->next` in $O(1)$ time.

```text
Why `tail` gives O(1) access to BOTH ends:

               tail->next (O(1) access to Head!)
                    │
                    ▼
               +----+----+     +----+----+     +----+----+
          ┌───►| 10 | next────►| 20 | next────►| 30 | next───┐
          │    +----+----+     +----+----+     +----+----+   │
          │                                         ▲        │
          │                                         │        │
          │                                        tail      │
          │                                  (O(1) access!)  │
          └──────────────────────────────────────────────────┘
```

### 3.3 Memory Layout in the Heap
Nodes are allocated dynamically on the heap via `malloc` and may reside at non-contiguous memory addresses. On a 64-bit system (`sizeof(int) == 4`, 4 bytes of struct alignment padding, `sizeof(CLLNode *) == 8`), each `CLLNode` occupies $16\text{ bytes}$.

Suppose the circular list stores `(10, 20, 30)` at heap addresses `0x5010`, `0x5080`, and `0x5040`:

```text
Stack Frame (main)
+------------------+
| tail = 0x5040    | ──────────────────────────────────────────────┐
+------------------+                                               │
                                                                   ▼
Heap Memory (Non-contiguous 16-byte blocks)                        │
  Address: 0x5010 (Head Node)   Address: 0x5080 (Node 2)     Address: 0x5040 (Tail Node)
  +---------+------------+      +---------+------------+     +---------+------------+
  | data:10 | next:0x5080|─────►| data:20 | next:0x5040|────►| data:30 | next:0x5010|──┐
  +---------+------------+      +---------+------------+     +---------+------------+  │
       ▲                                                                               │
       └────────────────────── (Closes the ring: 0x5040->next == 0x5010) ──────────────┘
```

### 3.4 Static vs. Dynamic Storage, Allocation, and Deallocation
- **Dynamic Allocation**: Each node is created at runtime using `malloc(sizeof(CLLNode))`. Always check if `malloc` returns `NULL` (heap exhaustion) before dereferencing.
- **Helper Constructor (`createNode`)**:
  ```c
  CLLNode* createNode(int value) {
      CLLNode *newNode = (CLLNode *)malloc(sizeof(CLLNode));
      if (newNode == NULL) {
          fprintf(stderr, "Error: Heap memory allocation failed for value %d.\n", value);
          return NULL;
      }
      newNode->data = value;
      newNode->next = newNode; /* Safe default: self-loop */
      return newNode;
  }
  ```
- **Safe Deallocation**: Because there is no `NULL` pointer in a non-empty circular list, freeing a list requires **either** recording the starting address and stopping when `curr` returns to `head`, **or** breaking the cycle first (`tail->next = NULL`) so the circular list turns into a standard `NULL`-terminated singly linked list and can be freed with a simple `while (head != NULL)` loop!

### 3.5 Pros and Cons of Representations

| Representation Variant | Advantages | Disadvantages |
| :--- | :--- | :--- |
| **SCLL with `head` pointer only** | Intuitive direct pointer to first element. | Both `insertAtBeginning` and `insertAtEnd` require $O(n)$ traversal to update the last node's `next`. |
| **SCLL with `tail` (`last`) pointer** | $O(1)$ `insertAtBeginning`, $O(1)$ `insertAtEnd`, and $O(1)$ `deleteFromBeginning` with only one pointer variable. | `deleteFromEnd` still costs $O(n)$ because finding the node before `tail` requires traversal. |
| **DCLL (Doubly Circular Linked List)** | $O(1)$ insertion and deletion at **both** head and tail (`tail->prev` is accessed in $O(1)$). Bidirectional traversal. | Extra memory overhead of one `prev` pointer ($8\text{ bytes}$ on 64-bit) per node; twice as many pointer updates per mutation. |

---

## 4. Operations and Algorithms

Throughout this section, we use the **Tail-Pointer Representation** (`CLLNode *tail`), where:
- If the list is empty, `tail == NULL`.
- If the list is non-empty, `tail` points to the last node and `tail->next` points to the first node (head).

---

### 4.1 Insertion at the Beginning

#### 1. Purpose
Insert a new node containing `value` as the new first node (head) of the circular linked list, without changing `tail` (unless the list was initially empty).

#### 2. Preconditions and Assumptions
- `tailRef` is a valid non-NULL double pointer (`CLLNode **tailRef`) holding the address of the caller's `tail` pointer.
- Heap memory is available for allocating one `CLLNode`.

#### 3. Step-by-Step Algorithm
1. Allocate a new node `newNode` with `newNode->data = value`. If allocation fails, return `false`.
2. Check if the list is empty (`*tailRef == NULL`):
   - Set `newNode->next = newNode` (points to itself).
   - Set `*tailRef = newNode` (the new node is both head and tail).
3. Otherwise (`*tailRef != NULL`):
   - Set `newNode->next = (*tailRef)->next` (new node points to current head).
   - Set `(*tailRef)->next = newNode` (tail now points to the new head).
4. Return `true`.

#### 4. Pseudocode
```text
ALGORITHM InsertAtBeginning(tail, value)
Input: Reference to tail pointer, integer value
Output: Updated tail pointer with new node at front

1. newNode <- AllocateNode(value)
2. IF tail == NULL THEN
3.     newNode.next <- newNode
4.     tail <- newNode
5. ELSE
6.     newNode.next <- tail.next
7.     tail.next <- newNode
8. END IF
9. RETURN tail
```

#### 5. Before-and-After ASCII Pointer Diagram
Inserting `5` at the beginning of `(10 -> 20 -> 30)`:
```text
BEFORE:
           tail->next (head)
                │
                ▼
           +----+----+     +----+----+     +----+----+
      ┌───►| 10 | o──|────►| 20 | o──|────►| 30 | o──|───┐
      │    +----+----+     +----+----+     +----+----+   │
      │                                         ▲        │
      └─────────────────────────────────────────┼────────┘
                                               tail

STEP 1 & 2: newNode->next = tail->next;  tail->next = newNode;

AFTER:
   (new head = tail->next)
        │
        ▼
   +----+----+     +----+----+     +----+----+     +----+----+
┌─►|  5 | o──|────►| 10 | o──|────►| 20 | o──|────►| 30 | o──|─┐
│  +----+----+     +----+----+     +----+----+     +----+----+ │
│                                                       ▲      │
└───────────────────────────────────────────────────────┼──────┘
                                                       tail (unchanged!)
```

#### 6. Reusable C Function
```c
bool insertAtBeginning(CLLNode **tailRef, int value) {
    CLLNode *newNode = createNode(value);
    if (newNode == NULL) {
        return false;
    }
    if (*tailRef == NULL) {
        newNode->next = newNode;
        *tailRef = newNode;
    } else {
        newNode->next = (*tailRef)->next;
        (*tailRef)->next = newNode;
    }
    return true;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $O(1)$ in best, average, and worst cases (only 2 pointer assignments).
- **Auxiliary Space Complexity**: $O(1)$ (excluding the single newly allocated node).

#### 8. Edge Cases and Common Mistakes
- **Empty List (`*tailRef == NULL`)**: `*tailRef` must be updated to `newNode`. In non-empty lists, `*tailRef` does **not** change during `insertAtBeginning`.
- **Order of Pointer Assignment**: Always execute `newNode->next = (*tailRef)->next` *before* overwriting `(*tailRef)->next = newNode`, or you will lose the address of the old head!

---

### 4.2 Insertion at the End

#### 1. Purpose
Append a new node containing `value` at the end of the circular linked list so that it becomes the new `tail` node while pointing forward to the existing head.

#### 2. Preconditions and Assumptions
- `tailRef` is a valid non-NULL pointer to the caller's `tail` pointer.

#### 3. Step-by-Step Algorithm
1. Allocate `newNode` with `newNode->data = value`. If `NULL`, return `false`.
2. If `*tailRef == NULL` (empty list):
   - Set `newNode->next = newNode`.
   - Set `*tailRef = newNode`.
3. Else (`*tailRef != NULL`):
   - Set `newNode->next = (*tailRef)->next` (new node points to current head).
   - Set `(*tailRef)->next = newNode` (old tail points to new node).
   - Update `*tailRef = newNode` (new node becomes the new tail).
4. Return `true`.

> **Key Insight**: Notice that Steps 2 and 3 of `insertAtEnd` are **identical** to `insertAtBeginning`, followed by a single extra statement: `*tailRef = newNode`! Because the list is a circle, inserting between `tail` and `head` puts the node at the boundary; whether that node is treated as the new first node or the new last node depends solely on whether we advance `tail` to point to it.

#### 4. Pseudocode
```text
ALGORITHM InsertAtEnd(tail, value)
Input: Reference to tail pointer, integer value
Output: Updated tail pointer with new node at end

1. newNode <- AllocateNode(value)
2. IF tail == NULL THEN
3.     newNode.next <- newNode
4.     tail <- newNode
5. ELSE
6.     newNode.next <- tail.next
7.     tail.next <- newNode
8.     tail <- newNode
9. END IF
10. RETURN tail
```

#### 5. Before-and-After ASCII Pointer Diagram
Inserting `40` at the end of `(10 -> 20 -> 30)`:
```text
BEFORE:
         +----+----+     +----+----+     +----+----+
    ┌───►| 10 | o──|────►| 20 | o──|────►| 30 | o──|───┐
    │    +----+----+     +----+----+     +----+----+   │
    │                                         ▲        │
    └─────────────────────────────────────────┼────────┘
                                             tail

AFTER:
         +----+----+     +----+----+     +----+----+     +----+----+
    ┌───►| 10 | o──|────►| 20 | o──|────►| 30 | o──|────►| 40 | o──|───┐
    │    +----+----+     +----+----+     +----+----+     +----+----+   │
    │                                                         ▲        │
    └─────────────────────────────────────────────────────────┼────────┘
                                                             tail (advanced!)
```

#### 6. Reusable C Function
```c
bool insertAtEnd(CLLNode **tailRef, int value) {
    if (!insertAtBeginning(tailRef, value)) {
        return false;
    }
    /* Advancing tail to the newly inserted boundary node makes it the tail */
    *tailRef = (*tailRef)->next;
    return true;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $O(1)$ worst-case time.
- **Auxiliary Space Complexity**: $O(1)$.

#### 8. Edge Cases and Common Mistakes
- **Forgetting to update `*tailRef`**: If you link `newNode` between `tail` and `tail->next` without setting `*tailRef = newNode`, you have performed `insertAtBeginning` instead of `insertAtEnd`.

---

### 4.3 Insertion at a Specified Position

#### 1. Purpose
Insert a new node with `value` at 1-based position `pos` ($1 \le \text{pos} \le n + 1$).

#### 2. Preconditions and Assumptions
- Positions are 1-indexed (`pos == 1` means insert at the beginning).
- If `pos < 1` or `pos > n + 1`, the operation rejects the invalid position and returns `false`.

#### 3. Step-by-Step Algorithm
1. If `pos < 1`, print an error and return `false`.
2. If `pos == 1`, delegate directly to `insertAtBeginning(tailRef, value)`.
3. If `*tailRef == NULL` (and `pos > 1`), position is out of range; return `false`.
4. Initialize `curr = (*tailRef)->next` (head) and traverse `pos - 2` steps to reach the predecessor node at index `pos - 1`:
   - In each step, if `curr == *tailRef` before completing `pos - 2` steps, `pos` exceeds $n + 1$; return `false`.
   - Advance `curr = curr->next`.
5. Allocate `newNode`. Set `newNode->next = curr->next` and `curr->next = newNode`.
6. If `curr == *tailRef` (meaning we inserted right after the old tail, at position $n + 1$), update `*tailRef = newNode`.
7. Return `true`.

#### 4. Pseudocode
```text
ALGORITHM InsertAtPosition(tail, value, pos)
1. IF pos < 1 THEN RETURN ERROR
2. IF pos == 1 THEN RETURN InsertAtBeginning(tail, value)
3. IF tail == NULL THEN RETURN ERROR
4. curr <- tail.next
5. FOR i <- 1 TO pos - 2 DO
6.     IF curr == tail THEN RETURN ERROR (pos > n + 1)
7.     curr <- curr.next
8. END FOR
9. newNode <- AllocateNode(value)
10. newNode.next <- curr.next
11. curr.next <- newNode
12. IF curr == tail THEN
13.     tail <- newNode
14. END IF
15. RETURN tail
```

#### 5. Before-and-After ASCII Pointer Diagram
Inserting `25` at `pos = 3` in `(10 -> 20 -> 30)`:
```text
BEFORE (curr stops at pos - 1 = 2, which is node 20):
                        curr
                         │
                         ▼
         +----+----+   +----+----+   +----+----+
    ┌───►| 10 | o──|──►| 20 | o──|──►| 30 | o──|───┐
    │    +----+----+   +----+----+   +----+----+   │
    │                                     ▲        │
    └─────────────────────────────────────┼────────┘
                                         tail

AFTER (newNode 25 spliced after curr):
                        curr          newNode
                         │               │
                         ▼               ▼
         +----+----+   +----+----+   +----+----+   +----+----+
    ┌───►| 10 | o──|──►| 20 | o──|──►| 25 | o──|──►| 30 | o──|───┐
    │    +----+----+   +----+----+   +----+----+   +----+----+   │
    │                                                   ▲        │
    └───────────────────────────────────────────────────┼────────┘
                                                       tail
```

#### 6. Reusable C Function
```c
bool insertAtPosition(CLLNode **tailRef, int value, int pos) {
    if (pos < 1) {
        fprintf(stderr, "Error: Invalid position %d (must be >= 1).\n", pos);
        return false;
    }
    if (pos == 1) {
        return insertAtBeginning(tailRef, value);
    }
    if (*tailRef == NULL) {
        fprintf(stderr, "Error: Position %d out of bounds for empty list.\n", pos);
        return false;
    }

    CLLNode *curr = (*tailRef)->next; /* Start at head (position 1) */
    for (int i = 1; i < pos - 1; i++) {
        if (curr == *tailRef) {
            fprintf(stderr, "Error: Position %d exceeds list length + 1.\n", pos);
            return false;
        }
        curr = curr->next;
    }

    CLLNode *newNode = createNode(value);
    if (newNode == NULL) {
        return false;
    }

    newNode->next = curr->next;
    curr->next = newNode;

    /* If inserted after the tail node (position n + 1), update tail */
    if (curr == *tailRef) {
        *tailRef = newNode;
    }
    return true;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**:
  - Best Case ($\text{pos} = 1$): $O(1)$.
  - Average & Worst Case ($1 < \text{pos} \le n + 1$): $O(n)$.
- **Auxiliary Space Complexity**: $O(1)$.

#### 8. Edge Cases and Common Mistakes
- **Inserting at `pos = n + 1`**: `curr` lands on `*tailRef`. If you forget `if (curr == *tailRef) *tailRef = newNode;`, the new node will end up at the **beginning** of the list instead of the end!
- **Wrap-Around Bug**: Without the check `if (curr == *tailRef)` inside the loop, passing `pos = 100` on a 3-node list would silently wrap around the circle multiple times and insert at a modulo position instead of reporting out-of-bounds.

---

### 4.4 Insertion After a Given Key/Value

#### 1. Purpose
Search for the first node containing `targetKey` and insert a new node with `newValue` immediately after it.

#### 2. Preconditions and Assumptions
- List must be non-empty (`*tailRef != NULL`).
- If `targetKey` occurs multiple times, insertion happens after the first occurrence starting from `head`.

#### 3. Step-by-Step Algorithm
1. If `*tailRef == NULL`, return `false`.
2. Set `curr = (*tailRef)->next` (head).
3. Using a `do-while` loop:
   - If `curr->data == targetKey`:
     a. Allocate `newNode(newValue)`.
     b. Set `newNode->next = curr->next` and `curr->next = newNode`.
     c. If `curr == *tailRef`, update `*tailRef = newNode`.
     d. Return `true`.
   - Advance `curr = curr->next`.
   - Continue while `curr != (*tailRef)->next`.
4. Return `false` (`targetKey` not found).

#### 4. Pseudocode
```text
ALGORITHM InsertAfterKey(tail, targetKey, newValue)
1. IF tail == NULL THEN RETURN FALSE
2. head <- tail.next
3. curr <- head
4. DO
5.     IF curr.data == targetKey THEN
6.         newNode <- AllocateNode(newValue)
7.         newNode.next <- curr.next
8.         curr.next <- newNode
9.         IF curr == tail THEN tail <- newNode
10.        RETURN TRUE
11.    END IF
12.    curr <- curr.next
13. WHILE curr != head
14. RETURN FALSE
```

#### 5. Before-and-After ASCII Pointer Diagram
Inserting `35` after `targetKey = 30` (where `30` is currently the `tail`):
```text
BEFORE:
                                        curr == tail
                                             │
                                             ▼
         +----+----+     +----+----+     +----+----+
    ┌───►| 10 | o──|────►| 20 | o──|────►| 30 | o──|───┐
    │    +----+----+     +----+----+     +----+----+   │
    └──────────────────────────────────────────────────┘

AFTER (curr == tail triggered *tailRef = newNode):
         +----+----+     +----+----+     +----+----+     +----+----+
    ┌───►| 10 | o──|────►| 20 | o──|────►| 30 | o──|────►| 35 | o──|───┐
    │    +----+----+     +----+----+     +----+----+     +----+----+   │
    │                                                         ▲        │
    └─────────────────────────────────────────────────────────┼────────┘
                                                             tail
```

#### 6. Reusable C Function
```c
bool insertAfterKey(CLLNode **tailRef, int targetKey, int newValue) {
    if (*tailRef == NULL) {
        return false;
    }
    CLLNode *head = (*tailRef)->next;
    CLLNode *curr = head;
    do {
        if (curr->data == targetKey) {
            CLLNode *newNode = createNode(newValue);
            if (newNode == NULL) {
                return false;
            }
            newNode->next = curr->next;
            curr->next = newNode;
            if (curr == *tailRef) {
                *tailRef = newNode;
            }
            return true;
        }
        curr = curr->next;
    } while (curr != head);

    return false; /* targetKey not found */
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $O(1)$ best case (`targetKey` at head), $O(n)$ average and worst case.
- **Auxiliary Space Complexity**: $O(1)$.

#### 8. Edge Cases and Common Mistakes
- **`targetKey` at `tail`**: Updating `*tailRef = newNode` is mandatory when `curr == *tailRef`.

---

### 4.5 Deletion from the Beginning

#### 1. Purpose
Remove and deallocate the first node (`head = tail->next`) of the circular linked list and store its value in `*removedOut`.

#### 2. Preconditions and Assumptions
- List must be non-empty (`*tailRef != NULL`).

#### 3. Step-by-Step Algorithm
1. If `*tailRef == NULL`, return `false` (underflow).
2. Let `head = (*tailRef)->next`.
3. Store `head->data` into `*removedOut` (if `removedOut != NULL`).
4. Check if there is only a single node (`head == *tailRef`):
   - Set `*tailRef = NULL`.
5. Otherwise (`head != *tailRef`):
   - Bypass `head` by setting `(*tailRef)->next = head->next`.
6. Call `free(head)` and return `true`.

#### 4. Pseudocode
```text
ALGORITHM DeleteFromBeginning(tail)
1. IF tail == NULL THEN RETURN ERROR (Underflow)
2. head <- tail.next
3. removedValue <- head.data
4. IF head == tail THEN
5.     tail <- NULL
6. ELSE
7.     tail.next <- head.next
8. END IF
9. FreeNode(head)
10. RETURN removedValue
```

#### 5. Before-and-After ASCII Pointer Diagram
Deleting the head node `10` from `(10 -> 20 -> 30)`:
```text
BEFORE:
           head = tail->next
                │
                ▼
           +----+----+     +----+----+     +----+----+
      ┌───►| 10 | o──|────►| 20 | o──|────►| 30 | o──|───┐
      │    +----+----+     +----+----+     +----+----+   │
      │                                         ▲        │
      └─────────────────────────────────────────┼────────┘
                                               tail

STEP: tail->next = head->next;  free(head);

AFTER:
            new head = tail->next
                 │
                 ▼
            +----+----+     +----+----+
       ┌───►| 20 | o──|────►| 30 | o──|───┐
       │    +----+----+     +----+----+   │
       │                         ▲        │
       └─────────────────────────┼────────┘
                                tail
```

#### 6. Reusable C Function
```c
bool deleteFromBeginning(CLLNode **tailRef, int *removedOut) {
    if (*tailRef == NULL) {
        fprintf(stderr, "Error: Cannot delete from an empty circular list.\n");
        return false;
    }

    CLLNode *head = (*tailRef)->next;
    if (removedOut != NULL) {
        *removedOut = head->data;
    }

    if (head == *tailRef) {
        /* Single-node list becomes empty */
        *tailRef = NULL;
    } else {
        (*tailRef)->next = head->next;
    }

    free(head);
    return true;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $O(1)$ in best, average, and worst cases!
- **Auxiliary Space Complexity**: $O(1)$.

#### 8. Edge Cases and Common Mistakes
- **Single-Node List (`head == *tailRef`)**: If you omit the `if (head == *tailRef)` check and execute `(*tailRef)->next = head->next` followed by `free(head)`, `*tailRef` becomes a **dangling pointer** to freed memory!

---

### 4.6 Deletion from the End

#### 1. Purpose
Remove and deallocate the last node (`tail`) of the circular linked list and update `tail` to point to its predecessor.

#### 2. Preconditions and Assumptions
- List is non-empty (`*tailRef != NULL`).

#### 3. Step-by-Step Algorithm
1. If `*tailRef == NULL`, return `false`.
2. Let `oldTail = *tailRef` and `head = oldTail->next`.
3. Store `oldTail->data` into `*removedOut` (if non-NULL).
4. If `head == oldTail` (single-node list):
   - Set `*tailRef = NULL`.
   - `free(oldTail)` and return `true`.
5. Otherwise, find the second-to-last node (`prev`):
   - Set `prev = head`.
   - While `prev->next != oldTail`, advance `prev = prev->next`.
6. Set `prev->next = head` (closing the circle without `oldTail`).
7. Update `*tailRef = prev`.
8. `free(oldTail)` and return `true`.

#### 4. Pseudocode
```text
ALGORITHM DeleteFromEnd(tail)
1. IF tail == NULL THEN RETURN ERROR (Underflow)
2. oldTail <- tail
3. head <- oldTail.next
4. IF head == oldTail THEN
5.     tail <- NULL
6.     FreeNode(oldTail)
7.     RETURN
8. END IF
9. prev <- head
10. WHILE prev.next != oldTail DO
11.     prev <- prev.next
12. END WHILE
13. prev.next <- head
14. tail <- prev
15. FreeNode(oldTail)
```

#### 5. Before-and-After ASCII Pointer Diagram
Deleting `30` from `(10 -> 20 -> 30)`:
```text
BEFORE (traverse until prev->next == tail):
                           prev           oldTail (tail)
                            │               │
                            ▼               ▼
         +----+----+     +----+----+     +----+----+
    ┌───►| 10 | o──|────►| 20 | o──|────►| 30 | o──|───┐
    │    +----+----+     +----+----+     +----+----+   │
    └──────────────────────────────────────────────────┘

AFTER (prev->next = head; tail = prev; free(oldTail)):
                           tail
                            │
                            ▼
         +----+----+     +----+----+
    ┌───►| 10 | o──|────►| 20 | o──|───┐
    │    +----+----+     +----+----+   │
    └──────────────────────────────────┘
```

#### 6. Reusable C Function
```c
bool deleteFromEnd(CLLNode **tailRef, int *removedOut) {
    if (*tailRef == NULL) {
        fprintf(stderr, "Error: Cannot delete from an empty circular list.\n");
        return false;
    }

    CLLNode *oldTail = *tailRef;
    CLLNode *head = oldTail->next;

    if (removedOut != NULL) {
        *removedOut = oldTail->data;
    }

    if (head == oldTail) {
        *tailRef = NULL;
        free(oldTail);
        return true;
    }

    CLLNode *prev = head;
    while (prev->next != oldTail) {
        prev = prev->next;
    }

    prev->next = head;
    *tailRef = prev;
    free(oldTail);
    return true;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $O(1)$ when $n = 1$; $O(n)$ in average and worst cases because a Singly Circular Linked List has no backward `prev` pointer. (In a Doubly Circular Linked List, `deleteFromEnd` is $O(1)$).
- **Auxiliary Space Complexity**: $O(1)$.

#### 8. Edge Cases and Common Mistakes
- **Why `deleteFromEnd` is $O(n)$ in SCLL**: Even though we have a pointer to `tail`, we must modify the `next` pointer of the node *immediately preceding* `tail`, which requires traversing $n - 1$ links from `head`.

---

### 4.7 Deletion from a Specified Position

#### 1. Purpose
Delete the node at 1-based position `pos` ($1 \le \text{pos} \le n$).

#### 2. Preconditions and Assumptions
- `pos >= 1` and `*tailRef != NULL`.

#### 3. Step-by-Step Algorithm
1. If `*tailRef == NULL` or `pos < 1`, return `false`.
2. If `pos == 1`, delegate to `deleteFromBeginning(tailRef, removedOut)`.
3. Set `prev = (*tailRef)->next` (head).
4. Advance `prev` for `pos - 2` steps:
   - Before advancing (or after), check if `prev->next == (*tailRef)->next` (meaning `pos` exceeds list length $n$). If so, return `false`.
5. Let `target = prev->next`. If `target == (*tailRef)->next`, `pos` is out of bounds; return `false`.
6. Set `prev->next = target->next`.
7. If `target == *tailRef` (deleting the last node), update `*tailRef = prev`.
8. Save `target->data` to `*removedOut`, `free(target)`, and return `true`.

#### 4. Pseudocode
```text
ALGORITHM DeleteAtPosition(tail, pos)
1. IF tail == NULL OR pos < 1 THEN RETURN ERROR
2. IF pos == 1 THEN RETURN DeleteFromBeginning(tail)
3. prev <- tail.next
4. FOR i <- 1 TO pos - 2 DO
5.     prev <- prev.next
6.     IF prev == tail THEN RETURN ERROR (pos > n)
7. END FOR
8. target <- prev.next
9. prev.next <- target.next
10. IF target == tail THEN
11.     tail <- prev
12. END IF
13. FreeNode(target)
```

#### 5. Before-and-After ASCII Pointer Diagram
Deleting `pos = 2` (node `20`) from `(10 -> 20 -> 30)`:
```text
BEFORE:
           prev           target
            │               │
            ▼               ▼
         +----+----+     +----+----+     +----+----+
    ┌───►| 10 | o──|────►| 20 | o──|────►| 30 | o──|───┐
    │    +----+----+     +----+----+     +----+----+   │
    │                                         ▲        │
    └─────────────────────────────────────────┼────────┘
                                             tail

AFTER (prev->next = target->next; free(target)):
           prev
            │
            ▼
         +----+----+                     +----+----+
    ┌───►| 10 | o──|────────────────────►| 30 | o──|───┐
    │    +----+----+                     +----+----+   │
    │                                         ▲        │
    └─────────────────────────────────────────┼────────┘
                                             tail
```

#### 6. Reusable C Function
```c
bool deleteAtPosition(CLLNode **tailRef, int pos, int *removedOut) {
    if (*tailRef == NULL || pos < 1) {
        fprintf(stderr, "Error: Invalid deletion position %d.\n", pos);
        return false;
    }
    if (pos == 1) {
        return deleteFromBeginning(tailRef, removedOut);
    }

    CLLNode *prev = (*tailRef)->next; /* Start at head (pos 1) */
    for (int i = 1; i < pos - 1; i++) {
        prev = prev->next;
        if (prev == *tailRef) {
            fprintf(stderr, "Error: Position %d exceeds list length.\n", pos);
            return false;
        }
    }

    CLLNode *target = prev->next;
    if (target == (*tailRef)->next) {
        fprintf(stderr, "Error: Position %d exceeds list length.\n", pos);
        return false;
    }

    if (removedOut != NULL) {
        *removedOut = target->data;
    }

    prev->next = target->next;
    if (target == *tailRef) {
        *tailRef = prev;
    }
    free(target);
    return true;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $O(1)$ for `pos == 1`; $O(\text{pos}) = O(n)$ for general position.
- **Auxiliary Space Complexity**: $O(1)$.

#### 8. Edge Cases and Common Mistakes
- **Deleting the `tail` via `deleteAtPosition(tailRef, n)`**: When `target == *tailRef`, `*tailRef` must be reassigned to `prev`.

---

### 4.8 Deletion by Value/Key

#### 1. Purpose
Find and delete the first node whose `data` equals `key`.

#### 2. Preconditions and Assumptions
- List is non-empty (`*tailRef != NULL`).

#### 3. Step-by-Step Algorithm
1. If `*tailRef == NULL`, return `false`.
2. Initialize `prev = *tailRef` and `curr = (*tailRef)->next` (head).
3. Loop through the circle using `do-while`:
   - If `curr->data == key`:
     a. If `curr == prev` (single-node list where `head == tail`):
        Set `*tailRef = NULL`.
     b. Else:
        Set `prev->next = curr->next`.
        If `curr == *tailRef`, update `*tailRef = prev`.
     c. `free(curr)` and return `true`.
   - Advance `prev = curr` and `curr = curr->next`.
   - Continue while `curr != (*tailRef)->next`.
4. Return `false` (`key` not found).

#### 4. Pseudocode
```text
ALGORITHM DeleteByKey(tail, key)
1. IF tail == NULL THEN RETURN FALSE
2. prev <- tail
3. curr <- tail.next
4. head <- curr
5. DO
6.     IF curr.data == key THEN
7.         IF curr == prev THEN
8.             tail <- NULL
9.         ELSE
10.            prev.next <- curr.next
11.            IF curr == tail THEN tail <- prev
12.        END IF
13.        FreeNode(curr)
14.        RETURN TRUE
15.    END IF
16.    prev <- curr
17.    curr <- curr.next
18. WHILE curr != head
19. RETURN FALSE
```

#### 5. Before-and-After ASCII Pointer Diagram
Deleting `key = 30` (the tail node) from `(10 -> 20 -> 30)`:
```text
BEFORE (curr matches 30 at tail; prev is at 20):
                           prev             curr (tail)
                            │                │
                            ▼                ▼
         +----+----+     +----+----+     +----+----+
    ┌───►| 10 | o──|────►| 20 | o──|────►| 30 | o──|───┐
    │    +----+----+     +----+----+     +----+----+   │
    └──────────────────────────────────────────────────┘

AFTER (prev->next = curr->next; tail = prev; free(curr)):
         +----+----+     +----+----+
    ┌───►| 10 | o──|────►| 20 | o──|───┐
    │    +----+----+     +----+----+   │
    │                         ▲        │
    └─────────────────────────┼────────┘
                             tail
```

#### 6. Reusable C Function
```c
bool deleteByKey(CLLNode **tailRef, int key) {
    if (*tailRef == NULL) {
        return false;
    }

    CLLNode *head = (*tailRef)->next;
    CLLNode *prev = *tailRef;
    CLLNode *curr = head;

    do {
        if (curr->data == key) {
            if (curr == prev) {
                /* Single node in list */
                *tailRef = NULL;
            } else {
                prev->next = curr->next;
                if (curr == *tailRef) {
                    *tailRef = prev;
                }
            }
            free(curr);
            return true;
        }
        prev = curr;
        curr = curr->next;
    } while (curr != head);

    return false; /* Key not found */
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $O(1)$ best case (key at head), $O(n)$ average and worst case.
- **Auxiliary Space Complexity**: $O(1)$.

#### 8. Edge Cases and Common Mistakes
- **Initializing `prev = *tailRef`**: Starting `prev` at `*tailRef` and `curr` at `(*tailRef)->next` is a clean idiom in circular lists because `tail` is genuinely the predecessor of `head`! This unifies head deletion and interior deletion into a single code path.

---

### 4.9 Traversal (`do-while`), Display, and Counting Nodes

#### 1. Purpose
Visit every node in the circular linked list exactly once to print its contents or count the total number of nodes $n$.

#### 2. Preconditions and Assumptions
- Because `curr` starts at `head` AND stops when `curr == head`, a standard `while (curr != head)` loop would terminate *immediately* on the first check (0 iterations). Therefore, after checking for an empty list, a **`do { ... } while (curr != head);`** loop is the canonical pattern.

#### 3. Step-by-Step Algorithm
1. If `tail == NULL`, print `[Empty Circular List]` (or return count `0`).
2. Set `head = tail->next` and `curr = head`.
3. `do`:
   - Process `curr->data` (print value or increment `count`).
   - Advance `curr = curr->next`.
   `while (curr != head);`
4. Print wrap-around indicator `(back to head: <head->data>)`.

#### 4. Pseudocode
```text
ALGORITHM TraverseAndCount(tail)
1. IF tail == NULL THEN RETURN 0
2. count <- 0
3. head <- tail.next
4. curr <- head
5. DO
6.     Print(curr.data)
7.     count <- count + 1
8.     curr <- curr.next
9. WHILE curr != head
10. RETURN count
```

#### 5. ASCII Trace of `do-while` Termination
For `(10 -> 20 -> 30)` where `head` is at `10`:
```text
Init:       curr = head (10)
Iter 1:     visit 10; curr = curr->next (20); check (20 != 10) -> true
Iter 2:     visit 20; curr = curr->next (30); check (30 != 10) -> true
Iter 3:     visit 30; curr = curr->next (10); check (10 != 10) -> FALSE (Loop terminates!)
```

#### 6. Reusable C Functions
```c
void displayList(const CLLNode *tail) {
    if (tail == NULL) {
        printf("List: [Empty]\n");
        return;
    }
    const CLLNode *head = tail->next;
    const CLLNode *curr = head;
    printf("List: ");
    do {
        printf("[%d] -> ", curr->data);
        curr = curr->next;
    } while (curr != head);
    printf("(back to head [%d])\n", head->data);
}

int countNodes(const CLLNode *tail) {
    if (tail == NULL) {
        return 0;
    }
    int count = 0;
    const CLLNode *head = tail->next;
    const CLLNode *curr = head;
    do {
        count++;
        curr = curr->next;
    } while (curr != head);
    return count;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $\Theta(n)$ — visits each of the $n$ nodes once.
- **Auxiliary Space Complexity**: $O(1)$.

#### 8. Edge Cases and Common Mistakes
- **Calling `do-while` without `if (tail == NULL)`**: Will immediately dereference a `NULL` pointer on `tail->next`. Always guard the `do-while` loop with an empty-list check.

---

### 4.10 Searching and Updating a Node

#### 1. Purpose
- **Search**: Determine whether `key` exists in the list and return its 1-based position (or `-1` if absent).
- **Update**: Replace the first occurrence of `oldVal` with `newVal`.

#### 2. Preconditions and Assumptions
- If `tail == NULL`, search returns `-1` and update returns `false`.

#### 3. Step-by-Step Algorithm
1. Check `if (tail == NULL)` and return `-1` (or `false`).
2. Set `head = tail->next`, `curr = head`, and `pos = 1`.
3. In a `do-while (curr != head)` loop:
   - If `curr->data == key`:
     - For Search: return `pos`.
     - For Update: set `curr->data = newVal` and return `true`.
   - Advance `curr = curr->next` and `pos++`.
4. Return `-1` (or `false`).

#### 4. Pseudocode
```text
ALGORITHM SearchCircularList(tail, key)
1. IF tail == NULL THEN RETURN -1
2. head <- tail.next
3. curr <- head
4. pos <- 1
5. DO
6.     IF curr.data == key THEN RETURN pos
7.     curr <- curr.next
8.     pos <- pos + 1
9. WHILE curr != head
10. RETURN -1
```

#### 5. Worked Trace
Searching for `30` in `(10 -> 20 -> 30 -> 40)`:
```text
pos = 1: curr->data = 10 != 30 -> advance
pos = 2: curr->data = 20 != 30 -> advance
pos = 3: curr->data = 30 == 30 -> MATCH! Return 3.
```

#### 6. Reusable C Functions
```c
int searchNode(const CLLNode *tail, int key) {
    if (tail == NULL) {
        return -1;
    }
    const CLLNode *head = tail->next;
    const CLLNode *curr = head;
    int pos = 1;
    do {
        if (curr->data == key) {
            return pos;
        }
        curr = curr->next;
        pos++;
    } while (curr != head);
    return -1;
}

bool updateNode(CLLNode *tail, int oldVal, int newVal) {
    if (tail == NULL) {
        return false;
    }
    CLLNode *head = tail->next;
    CLLNode *curr = head;
    do {
        if (curr->data == oldVal) {
            curr->data = newVal;
            return true;
        }
        curr = curr->next;
    } while (curr != head);
    return false;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $O(1)$ best case, $O(n)$ average and worst case.
- **Auxiliary Space Complexity**: $O(1)$.

#### 8. Edge Cases and Common Mistakes
- **Modifying `head` pointer during search**: Never mutate `tail->next` while iterating; use a separate local pointer `curr`.

---

### 4.11 Reversing a Circular Linked List

#### 1. Purpose
Reverse the direction of all `next` links in the circular linked list in-place so that `(v_1 -> v_2 -> ... -> v_n)` becomes `(v_n -> v_{n-1} -> ... -> v_1)`, and update `tail` to point to the old head $v_1$.

#### 2. Preconditions and Assumptions
- Empty (`tail == NULL`) or single-node (`tail->next == tail`) lists are already reversed and require no changes.

#### 3. Step-by-Step Algorithm
1. If `*tailRef == NULL` or `(*tailRef)->next == *tailRef`, return immediately.
2. Let `head = (*tailRef)->next`.
3. Initialize three pointers:
   - `prev = *tailRef`
   - `curr = head`
   - `nextNode = NULL`
4. Use a `do-while` loop:
   - Save `nextNode = curr->next`.
   - Reverse the link: `curr->next = prev`.
   - Advance `prev = curr` and `curr = nextNode`.
   `while (curr != head);`
5. Update `*tailRef = head` (the original head is now the last node, and its `next` pointer was already set to the original `tail` in iteration 1!).

#### 4. Pseudocode
```text
ALGORITHM ReverseCircularList(tail)
1. IF tail == NULL OR tail.next == tail THEN RETURN tail
2. head <- tail.next
3. prev <- tail
4. curr <- head
5. DO
6.     nextNode <- curr.next
7.     curr.next <- prev
8.     prev <- curr
9.     curr <- nextNode
10. WHILE curr != head
11. tail <- head
12. RETURN tail
```

#### 5. Before-and-After ASCII Pointer Diagram
Reversing `(10 -> 20 -> 30)`:
```text
BEFORE:
         head                                tail
          │                                   │
          ▼                                   ▼
         +----+----+     +----+----+     +----+----+
    ┌───►| 10 | o──|────►| 20 | o──|────►| 30 | o──|───┐
    │    +----+----+     +----+----+     +----+----+   │
    └──────────────────────────────────────────────────┘

DURING REVERSAL (all arrows flip direction):
         +----+----+     +----+----+     +----+----+
    ┌────|──o | 10 |◄────|──o | 20 |◄────|──o | 30 |◄──┐
    │    +----+----+     +----+----+     +----+----+   │
    │         ▲                               ▲        │
    │        tail                         new head     │
    └──────────────────────────────────────────────────┘

LOGICAL ORDER STARTING FROM NEW HEAD (tail->next = 30):
  [30] -> [20] -> [10] -> (back to head [30])
```

#### 6. Reusable C Function
```c
void reverseCircularList(CLLNode **tailRef) {
    if (*tailRef == NULL || (*tailRef)->next == *tailRef) {
        return; /* Empty or 1-node list needs no work */
    }

    CLLNode *head = (*tailRef)->next;
    CLLNode *prev = *tailRef;
    CLLNode *curr = head;
    CLLNode *nextNode = NULL;

    do {
        nextNode = curr->next;
        curr->next = prev;
        prev = curr;
        curr = nextNode;
    } while (curr != head);

    /* Original head is now the new tail; its ->next points to original tail (new head) */
    *tailRef = head;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $\Theta(n)$ — single pass through all $n$ nodes.
- **Auxiliary Space Complexity**: $O(1)$ — uses 4 local pointer variables.

#### 8. Edge Cases and Common Mistakes
- **Forgetting `*tailRef = head` at the end**: If you reverse all links but leave `*tailRef` pointing to the old tail (`30`), then `(*tailRef)->next` will be `20`, rotating the list out of place! Setting `*tailRef = head` (`10`) ensures `(*tailRef)->next` is `30` (the new head).

---

### 4.12 Complete, Compilable C11 Program

The program below integrates every operation covered in Section 4, including safe memory deallocation (`freeCircularList`) without infinite loops.

```c
/**
 * File: circular_linked_list_demo.c
 * Standard: C11
 * Description: Complete implementation of a Singly Circular Linked List (SCLL)
 *              using the O(1) Tail-Pointer (last) representation.
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct CLLNode {
    int data;
    struct CLLNode *next;
} CLLNode;

/* Allocate and initialize a new circular node */
CLLNode* createNode(int value) {
    CLLNode *newNode = (CLLNode *)malloc(sizeof(CLLNode));
    if (newNode == NULL) {
        fprintf(stderr, "Error: Memory allocation failed for value %d.\n", value);
        return NULL;
    }
    newNode->data = value;
    newNode->next = newNode;
    return newNode;
}

/* 1. Insert at the beginning: O(1) */
bool insertAtBeginning(CLLNode **tailRef, int value) {
    CLLNode *newNode = createNode(value);
    if (newNode == NULL) {
        return false;
    }
    if (*tailRef == NULL) {
        newNode->next = newNode;
        *tailRef = newNode;
    } else {
        newNode->next = (*tailRef)->next;
        (*tailRef)->next = newNode;
    }
    return true;
}

/* 2. Insert at the end: O(1) */
bool insertAtEnd(CLLNode **tailRef, int value) {
    if (!insertAtBeginning(tailRef, value)) {
        return false;
    }
    *tailRef = (*tailRef)->next;
    return true;
}

/* 3. Insert at 1-based position: O(n) */
bool insertAtPosition(CLLNode **tailRef, int value, int pos) {
    if (pos < 1) {
        fprintf(stderr, "Error: Invalid position %d.\n", pos);
        return false;
    }
    if (pos == 1) {
        return insertAtBeginning(tailRef, value);
    }
    if (*tailRef == NULL) {
        fprintf(stderr, "Error: Position %d out of bounds (list is empty).\n", pos);
        return false;
    }

    CLLNode *curr = (*tailRef)->next;
    for (int i = 1; i < pos - 1; i++) {
        if (curr == *tailRef) {
            fprintf(stderr, "Error: Position %d exceeds list length + 1.\n", pos);
            return false;
        }
        curr = curr->next;
    }

    CLLNode *newNode = createNode(value);
    if (newNode == NULL) {
        return false;
    }

    newNode->next = curr->next;
    curr->next = newNode;
    if (curr == *tailRef) {
        *tailRef = newNode;
    }
    return true;
}

/* 4. Insert after a given key: O(n) */
bool insertAfterKey(CLLNode **tailRef, int targetKey, int newValue) {
    if (*tailRef == NULL) {
        return false;
    }
    CLLNode *head = (*tailRef)->next;
    CLLNode *curr = head;
    do {
        if (curr->data == targetKey) {
            CLLNode *newNode = createNode(newValue);
            if (newNode == NULL) {
                return false;
            }
            newNode->next = curr->next;
            curr->next = newNode;
            if (curr == *tailRef) {
                *tailRef = newNode;
            }
            return true;
        }
        curr = curr->next;
    } while (curr != head);
    return false;
}

/* 5. Delete from the beginning: O(1) */
bool deleteFromBeginning(CLLNode **tailRef, int *removedOut) {
    if (*tailRef == NULL) {
        return false;
    }
    CLLNode *head = (*tailRef)->next;
    if (removedOut != NULL) {
        *removedOut = head->data;
    }
    if (head == *tailRef) {
        *tailRef = NULL;
    } else {
        (*tailRef)->next = head->next;
    }
    free(head);
    return true;
}

/* 6. Delete from the end: O(n) */
bool deleteFromEnd(CLLNode **tailRef, int *removedOut) {
    if (*tailRef == NULL) {
        return false;
    }
    CLLNode *oldTail = *tailRef;
    CLLNode *head = oldTail->next;
    if (removedOut != NULL) {
        *removedOut = oldTail->data;
    }
    if (head == oldTail) {
        *tailRef = NULL;
        free(oldTail);
        return true;
    }
    CLLNode *prev = head;
    while (prev->next != oldTail) {
        prev = prev->next;
    }
    prev->next = head;
    *tailRef = prev;
    free(oldTail);
    return true;
}

/* 7. Delete at 1-based position: O(n) */
bool deleteAtPosition(CLLNode **tailRef, int pos, int *removedOut) {
    if (*tailRef == NULL || pos < 1) {
        return false;
    }
    if (pos == 1) {
        return deleteFromBeginning(tailRef, removedOut);
    }
    CLLNode *prev = (*tailRef)->next;
    for (int i = 1; i < pos - 1; i++) {
        prev = prev->next;
        if (prev == *tailRef) {
            return false;
        }
    }
    CLLNode *target = prev->next;
    if (target == (*tailRef)->next) {
        return false;
    }
    if (removedOut != NULL) {
        *removedOut = target->data;
    }
    prev->next = target->next;
    if (target == *tailRef) {
        *tailRef = prev;
    }
    free(target);
    return true;
}

/* 8. Delete by value/key: O(n) */
bool deleteByKey(CLLNode **tailRef, int key) {
    if (*tailRef == NULL) {
        return false;
    }
    CLLNode *head = (*tailRef)->next;
    CLLNode *prev = *tailRef;
    CLLNode *curr = head;
    do {
        if (curr->data == key) {
            if (curr == prev) {
                *tailRef = NULL;
            } else {
                prev->next = curr->next;
                if (curr == *tailRef) {
                    *tailRef = prev;
                }
            }
            free(curr);
            return true;
        }
        prev = curr;
        curr = curr->next;
    } while (curr != head);
    return false;
}

/* 9. Display and Count: O(n) */
void displayList(const CLLNode *tail) {
    if (tail == NULL) {
        printf("List: [Empty]\n");
        return;
    }
    const CLLNode *head = tail->next;
    const CLLNode *curr = head;
    printf("List: ");
    do {
        printf("[%d] -> ", curr->data);
        curr = curr->next;
    } while (curr != head);
    printf("(back to head [%d])\n", head->data);
}

int countNodes(const CLLNode *tail) {
    if (tail == NULL) {
        return 0;
    }
    int count = 0;
    const CLLNode *head = tail->next;
    const CLLNode *curr = head;
    do {
        count++;
        curr = curr->next;
    } while (curr != head);
    return count;
}

/* 10. Search and Update: O(n) */
int searchNode(const CLLNode *tail, int key) {
    if (tail == NULL) {
        return -1;
    }
    const CLLNode *head = tail->next;
    const CLLNode *curr = head;
    int pos = 1;
    do {
        if (curr->data == key) {
            return pos;
        }
        curr = curr->next;
        pos++;
    } while (curr != head);
    return -1;
}

bool updateNode(CLLNode *tail, int oldVal, int newVal) {
    if (tail == NULL) {
        return false;
    }
    CLLNode *head = tail->next;
    CLLNode *curr = head;
    do {
        if (curr->data == oldVal) {
            curr->data = newVal;
            return true;
        }
        curr = curr->next;
    } while (curr != head);
    return false;
}

/* 11. Reverse in-place: O(n) */
void reverseCircularList(CLLNode **tailRef) {
    if (*tailRef == NULL || (*tailRef)->next == *tailRef) {
        return;
    }
    CLLNode *head = (*tailRef)->next;
    CLLNode *prev = *tailRef;
    CLLNode *curr = head;
    CLLNode *nextNode = NULL;
    do {
        nextNode = curr->next;
        curr->next = prev;
        prev = curr;
        curr = nextNode;
    } while (curr != head);
    *tailRef = head;
}

/* 12. Safe Deallocation: Break the circle first, then free linearly */
void freeCircularList(CLLNode **tailRef) {
    if (*tailRef == NULL) {
        return;
    }
    CLLNode *head = (*tailRef)->next;
    (*tailRef)->next = NULL; /* Break circular link into a NULL-terminated SLL */

    CLLNode *curr = head;
    while (curr != NULL) {
        CLLNode *next = curr->next;
        free(curr);
        curr = next;
    }
    *tailRef = NULL;
}

int main(void) {
    CLLNode *tail = NULL;
    int removed = 0;

    printf("=== 1. Insertions (Front, End, Position, After Key) ===\n");
    insertAtEnd(&tail, 20);
    insertAtEnd(&tail, 30);
    insertAtBeginning(&tail, 10);
    insertAtEnd(&tail, 50);
    insertAtPosition(&tail, 40, 4);      /* Insert 40 at pos 4 */
    insertAfterKey(&tail, 50, 60);       /* Insert 60 after tail (50) */
    displayList(tail);
    printf("Node count: %d (Head: %d, Tail: %d)\n\n",
           countNodes(tail), tail->next->data, tail->data);

    printf("=== 2. Search and Update ===\n");
    int pos = searchNode(tail, 40);
    printf("Search for 40: found at 1-based position %d\n", pos);
    updateNode(tail, 40, 45);
    printf("After updating 40 -> 45:\n");
    displayList(tail);
    printf("\n");

    printf("=== 3. Deletions (Beginning, End, Position, By Key) ===\n");
    deleteFromBeginning(&tail, &removed);
    printf("Deleted from beginning (%d):\n", removed);
    displayList(tail);

    deleteFromEnd(&tail, &removed);
    printf("Deleted from end (%d):\n", removed);
    displayList(tail);

    deleteAtPosition(&tail, 2, &removed);
    printf("Deleted at position 2 (%d):\n", removed);
    displayList(tail);

    deleteByKey(&tail, 45);
    printf("Deleted by key (45):\n");
    displayList(tail);
    printf("\n");

    printf("=== 4. Reversing the Circular Linked List ===\n");
    insertAtEnd(&tail, 70);
    insertAtEnd(&tail, 80);
    printf("Before reversal:\n");
    displayList(tail);
    reverseCircularList(&tail);
    printf("After reversal:\n");
    displayList(tail);
    printf("New Head: %d, New Tail: %d\n\n", tail->next->data, tail->data);

    printf("=== 5. Safe Deallocation ===\n");
    freeCircularList(&tail);
    displayList(tail);
    printf("Node count after free: %d\n", countNodes(tail));

    return 0;
}
```

#### Program Output:
```text
=== 1. Insertions (Front, End, Position, After Key) ===
List: [10] -> [20] -> [30] -> [40] -> [50] -> [60] -> (back to head [10])
Node count: 6 (Head: 10, Tail: 60)

=== 2. Search and Update ===
Search for 40: found at 1-based position 4
After updating 40 -> 45:
List: [10] -> [20] -> [30] -> [45] -> [50] -> [60] -> (back to head [10])

=== 3. Deletions (Beginning, End, Position, By Key) ===
Deleted from beginning (10):
List: [20] -> [30] -> [45] -> [50] -> [60] -> (back to head [20])
Deleted from end (60):
List: [20] -> [30] -> [45] -> [50] -> (back to head [20])
Deleted at position 2 (30):
List: [20] -> [45] -> [50] -> (back to head [20])
Deleted by key (45):
List: [20] -> [50] -> (back to head [20])

=== 4. Reversing the Circular Linked List ===
Before reversal:
List: [20] -> [50] -> [70] -> [80] -> (back to head [20])
After reversal:
List: [80] -> [70] -> [50] -> [20] -> (back to head [80])
New Head: 80, New Tail: 20

=== 5. Safe Deallocation ===
List: [Empty]
Node count after free: 0
```

---

## 5. Complexity Analysis and Comparison Tables

### 5.1 Operation Complexity Summary
Let $n$ denote the number of nodes currently stored in the Circular Linked List.
- **Total Space Complexity** of the data structure is $\Theta(n)$ words ($\approx 16n\text{ bytes}$ for SCLL on 64-bit systems).
- **Auxiliary Space Complexity** is the extra temporary stack/heap memory required per operation.

| Operation | SCLL (Head Pointer Only) Worst Time | SCLL (Tail Pointer `last`) Best Time | SCLL (Tail Pointer `last`) Average Time | SCLL (Tail Pointer `last`) Worst Time | Doubly Circular (DCLL) Worst Time | Auxiliary Space |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Access Head** | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **Access Tail** | $O(n)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **Insert at Beginning** | $O(n)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **Insert at End** | $O(n)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **Insert at Position `pos`** | $O(n)$ | $O(1)$ ($\text{pos}=1$) | $O(n)$ | $O(n)$ | $O(n)$ | $O(1)$ |
| **Insert After Key** | $O(n)$ | $O(1)$ (at head) | $O(n)$ | $O(n)$ | $O(n)$ | $O(1)$ |
| **Delete from Beginning** | $O(n)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **Delete from End** | $O(n)$ | $O(1)$ ($n=1$) | $O(n)$ | $O(n)$ | $O(1)$ | $O(1)$ |
| **Delete at Position `pos`** | $O(n)$ | $O(1)$ ($\text{pos}=1$) | $O(n)$ | $O(n)$ | $O(n)$ | $O(1)$ |
| **Delete by Key** | $O(n)$ | $O(1)$ (at head) | $O(n)$ | $O(n)$ | $O(n)$ | $O(1)$ |
| **Search / Update by Key** | $O(n)$ | $O(1)$ (at head) | $O(n)$ | $O(n)$ | $O(n)$ | $O(1)$ |
| **Traverse / Count Nodes** | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $O(1)$ |
| **Reverse In-Place** | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $O(1)$ |

### 5.2 Comparison: Singly vs. Doubly vs. Circular Linked Lists

| Feature / Metric | Singly Linked List (SLL) (`head` only) | Doubly Linked List (DLL) (`head` & `tail`) | Singly Circular Linked List (SCLL) (`tail` only) | Doubly Circular Linked List (DCLL) (`head` or `tail`) |
| :--- | :--- | :--- | :--- | :--- |
| **Pointers per Node** | $1$ (`next`) | $2$ (`prev`, `next`) | $1$ (`next`) | $2$ (`prev`, `next`) |
| **Bytes per Node (64-bit, `int`)** | $16\text{ B}$ | $24\text{ B}$ | $16\text{ B}$ | $24\text{ B}$ |
| **Contains `NULL` Pointers?** | Yes (`tail->next == NULL`) | Yes (`head->prev`, `tail->next`) | No (when $n \ge 1$) | No (when $n \ge 1$) |
| **Insert at Beginning** | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **Insert at End** | $O(n)$ (or $O(1)$ with extra `tail` ptr) | $O(1)$ | $O(1)$ (with single `tail` ptr!) | $O(1)$ |
| **Delete from Beginning** | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **Delete from End** | $O(n)$ | $O(1)$ | $O(n)$ | $O(1)$ |
| **Reach All Nodes from Any Node?** | No (only forward nodes) | Yes (by reversing direction) | Yes (by continuing around circle) | Yes (in either direction) |
| **Concatenate Two Lists** | $O(n)$ | $O(1)$ | $O(1)$ (swap `tail1->next` and `tail2->next`) | $O(1)$ |

> **Bonus Property — $O(1)$ List Concatenation**: Two Singly Circular Linked Lists with tail pointers `tail1` and `tail2` can be concatenated into a single circular list in **$O(1)$ time**:
> ```c
> CLLNode *head1 = tail1->next;
> tail1->next = tail2->next; /* List 1 tail points to List 2 head */
> tail2->next = head1;       /* List 2 tail points to List 1 head */
> tail1 = tail2;             /* New combined tail is tail2 */
> ```

### 5.3 Real-World Applications
1. **Operating System Round-Robin CPU Scheduling**: Ready processes share CPU time slices in a circular queue. The scheduler executes the process at `curr`, and when its time quantum expires, simply advances `curr = curr->next` in $O(1)$ time.
2. **Multiplayer Turn-Based Games**: Players in board games (Monopoly, Poker, Ludo) are linked in a circular ring. Advancing turns is `currentPlayer = currentPlayer->next`, and eliminating a bankrupt player is a node deletion.
3. **Circular Buffers / Streaming Queues**: A Queue ADT can be implemented using a Singly Circular Linked List with **only a single `tail` pointer** (`enqueue` = `insertAtEnd` in $O(1)$, `dequeue` = `deleteFromBeginning` in $O(1)$).
4. **Fibonacci Heaps**: Advanced priority queues (used in Dijkstra's and Prim's algorithms) store sibling trees using Doubly Circular Linked Lists so two heaps can be melded in $O(1)$ time.

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Pitfall 1: Infinite Loops Using `while (curr != NULL)`
In a non-empty circular linked list, no node has `next == NULL`. Using a standard SLL traversal loop causes an endless loop.

```c
/* BUG: Infinite loop on any non-empty circular list! */
void printListBuggy(const CLLNode *tail) {
    const CLLNode *curr = tail->next;
    while (curr != NULL) { /* curr is NEVER NULL! */
        printf("%d ", curr->data);
        curr = curr->next;
    }
}

/* FIX: Check for NULL tail first, then use do-while stopping at head */
void printListFixed(const CLLNode *tail) {
    if (tail == NULL) return;
    const CLLNode *head = tail->next;
    const CLLNode *curr = head;
    do {
        printf("%d ", curr->data);
        curr = curr->next;
    } while (curr != head);
}
```

### 6.2 Pitfall 2: Forgetting to Update `tail->next` on Head Modification
If you maintain a `head` pointer (or copy `head = tail->next` into a local variable) and insert/delete at the front without updating `tail->next`, the last node still points to the old head (which may now be freed memory!).

```c
/* BUG: Modifying a local head pointer without updating tail->next */
void deleteHeadBuggy(CLLNode *tail) {
    CLLNode *head = tail->next;
    head = head->next; /* Only changes local variable! tail->next still points to freed node! */
    /* free(oldHead) would corrupt the ring */
}

/* FIX: Update tail->next directly (and handle the 1-node case) */
void deleteHeadFixed(CLLNode **tailRef) {
    if (*tailRef == NULL) return;
    CLLNode *head = (*tailRef)->next;
    if (head == *tailRef) {
        *tailRef = NULL;
    } else {
        (*tailRef)->next = head->next;
    }
    free(head);
}
```

### 6.3 Pitfall 3: Mishandling the Single-Node Deletion Case
When $n = 1$, `head == tail` and `tail->next == tail`. If you execute `tail->next = head->next` and `free(head)` without setting `tail = NULL`, `tail` continues pointing to the deallocated block.

```c
/* BUG: Fails to set *tailRef = NULL when deleting the last remaining node */
void removeFirstBuggy(CLLNode **tailRef) {
    CLLNode *head = (*tailRef)->next;
    (*tailRef)->next = head->next; /* When n == 1, this is a no-op self-assignment */
    free(head);                    /* Frees *tailRef itself! *tailRef is now DANGLING! */
}

/* FIX: Explicitly check (head == *tailRef) */
void removeFirstFixed(CLLNode **tailRef) {
    if (*tailRef == NULL) return;
    CLLNode *head = (*tailRef)->next;
    if (head == *tailRef) {
        *tailRef = NULL;
    } else {
        (*tailRef)->next = head->next;
    }
    free(head);
}
```

### 6.4 Pitfall 4: Infinite Loop or Use-After-Free During List Deallocation
If you try to free a circular list with `do { ... } while (curr != head)` and free `head` on the first iteration, some compilers/debuggers (like AddressSanitizer or Valgrind) may flag pointer comparisons or accidental dereferences, or if you forget the stop condition, you will double-free `head`.

```c
/* BUG: Freeing without breaking the cycle or tracking original head properly */
void freeListBuggy(CLLNode *tail) {
    CLLNode *curr = tail->next;
    while (curr != tail) {
        CLLNode *next = curr->next;
        free(curr);
        curr = next;
    }
    /* Wait: what if tail was NULL? Crash on tail->next! */
}

/* FIX: Guard NULL, break the circle at tail->next = NULL, then free like a normal SLL */
void freeListFixed(CLLNode **tailRef) {
    if (*tailRef == NULL) return;
    CLLNode *curr = (*tailRef)->next; /* Start at head */
    (*tailRef)->next = NULL;          /* Break the ring! */
    while (curr != NULL) {
        CLLNode *next = curr->next;
        free(curr);
        curr = next;
    }
    *tailRef = NULL;
}
```

### 6.5 Boundary Conditions Checklist
Before finalizing any Circular Linked List function, verify its behavior on these five boundary states:
- [ ] **Case 1: Empty List (`tail == NULL`)** — Does traversal/search/deletion return immediately without dereferencing `tail->next`? Does insertion set `newNode->next = newNode`?
- [ ] **Case 2: Single-Node List (`tail->next == tail`)** — Does deletion set `*tailRef = NULL`? Does reversal leave the node pointing to itself?
- [ ] **Case 3: Operation at the Head (`tail->next`)** — Is `tail->next` updated to point to the new first node?
- [ ] **Case 4: Operation at the Tail (`tail`)** — If a node is inserted after `tail` or if `tail` itself is deleted, is `*tailRef` updated to the new last node?
- [ ] **Case 5: Out-of-Bounds Position / Missing Key** — Does the loop stop after exactly one full revolution (`curr == head` or `curr == tail`) instead of wrapping around infinitely?

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways
1. **No `NULL` Pointers in Non-Empty Lists**: In a Circular Linked List, the last node's `next` pointer points back to the first node.
2. **Always Prefer a `tail` (`last`) Pointer Over a `head` Pointer for SCLL**: Storing `tail` gives $O(1)$ access to both the last node (`tail`) and the first node (`tail->next`), enabling **$O(1)$ `insertAtBeginning`, $O(1)$ `insertAtEnd`, and $O(1)$ `deleteFromBeginning`**.
3. **`do-while` Is the Standard Traversal Idiom**: After checking `if (tail != NULL)`, start `curr = head` and iterate with `do { ... curr = curr->next; } while (curr != head);`.
4. **Deleting the Tail Takes $O(n)$ in SCLL**: Finding the predecessor of `tail` requires traversing $n - 1$ links unless a Doubly Circular Linked List (DCLL) is used.
5. **Break the Ring to Free**: Setting `tail->next = NULL` converts a circular list into a standard `NULL`-terminated singly linked list for clean, bug-free deallocation.

### 7.2 Essential Invariants
- **Empty State**: `tail == NULL`
- **Single-Node State**: `tail != NULL && tail->next == tail`
- **Head Access**: `head = tail->next`
- **Cycle Identity**: Starting from any node `p`, advancing `p = p->next` $n$ times brings `p` back to its initial address.

### 7.3 Quick Self-Test Checklist
- [ ] Can you explain why `insertAtEnd` is simply `insertAtBeginning` followed by `tail = tail->next`?
- [ ] Can you write a `do-while` loop to count the nodes in a circular linked list without an off-by-one error?
- [ ] Can you trace `deleteByKey` when the key matches the only node in a 1-node circular list?
- [ ] Can you concatenate two non-empty circular linked lists in $O(1)$ time using their `tail` pointers?
- [ ] Can you reverse a circular linked list in-place in $O(n)$ time and $O(1)$ auxiliary space?

### 7.4 Related Notes and Navigation
- **Previous**: [03 - Doubly Linked Lists](./03-Doubly-Linked-Lists.md)
- **Next**: [05 - Multilists, Sparse Matrices, and Skip Lists](./05-Multilists-Sparse-Matrices-and-Skip-Lists.md)
- **Case Study**: [06 - Polynomial Representation and Operations](./06-Polynomial-Representation-and-Operations.md)
- **Course Index**: [DSA Notes README](../README.md)
