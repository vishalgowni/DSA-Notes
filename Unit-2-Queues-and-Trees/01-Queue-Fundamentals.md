# Unit 2 — Topic 1: Queue Fundamentals (ADT, Simple Array Queue, and Linked-List Queue)

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [What Is a Queue?](#11-what-is-a-queue)
   - 1.2 [Why Do We Need Queues? The Problem They Solve](#12-why-do-we-need-queues-the-problem-they-solve)
   - 1.3 [Relationship to Prior Concepts: FIFO Queues vs. LIFO Stacks](#13-relationship-to-prior-concepts-fifo-queues-vs-lifo-stacks)
   - 1.4 [Real-World Analogies and Computer Science Applications](#14-real-world-analogies-and-computer-science-applications)
   - 1.5 [Learning Objectives](#15-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Formal Definition of the Queue Abstract Data Type (ADT)](#21-formal-definition-of-the-queue-abstract-data-type-adt)
   - 2.2 [Intuitive Explanation and Two-Ended Access Model](#22-intuitive-explanation-and-two-ended-access-model)
   - 2.3 [Core Terminology Table](#23-core-terminology-table)
   - 2.4 [Mathematical Properties, Invariants, and Formulas](#24-mathematical-properties-invariants-and-formulas)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Simple (Linear) Array-Based Queue Representation](#31-simple-linear-array-based-queue-representation)
   - 3.2 [`front` and `rear` Index Conventions Compared](#32-front-and-rear-index-conventions-compared)
   - 3.3 [Linked-List-Based Queue Representation](#33-linked-list-based-queue-representation)
   - 3.4 [Static vs. Dynamic Storage: Architectural Trade-Offs](#34-static-vs-dynamic-storage-architectural-trade-offs)
4. [Operations and Algorithms: Simple (Linear) Array Queue](#4-operations-and-algorithms-simple-linear-array-queue)
   - 4.1 [Initialization (`initArrayQueue`) and State Predicates (`isEmpty`, `isFull`, `size`)](#41-initialization-initarrayqueue-and-state-predicates-isempty-isfull-size)
   - 4.2 [Insertion at Rear (`enqueue`)](#42-insertion-at-rear-enqueue)
   - 4.3 [Deletion from Front (`dequeue`)](#43-deletion-from-front-dequeue)
   - 4.4 [Inspection Operations (`peekFront`, `peekRear`, `display`)](#44-inspection-operations-peekfront-peekrear-display)
   - 4.5 [Step-by-Step Index and Array Trace](#45-step-by-step-index-and-array-trace)
   - 4.6 [The "False Overflow" (Rightward Drift) Problem](#46-the-false-overflow-rightward-drift-problem)
5. [Operations and Algorithms: Linked-List Queue](#5-operations-and-algorithms-linked-list-queue)
   - 5.1 [Why Both `front` and `rear` Pointers Are Essential for $O(1)$ Performance](#51-why-both-front-and-rear-pointers-are-essential-for-o1-performance)
   - 5.2 [Initialization (`initLinkedQueue`) and Empty Check](#52-initialization-initlinkedqueue-and-empty-check)
   - 5.3 [Dynamic Enqueue (`enqueueLinked`) and the Empty-Queue Edge Case](#53-dynamic-enqueue-enqueuelinked-and-the-empty-queue-edge-case)
   - 5.4 [Dynamic Dequeue (`dequeueLinked`) and the Single-Node Edge Case](#54-dynamic-dequeue-dequeuelinked-and-the-single-node-edge-case)
   - 5.5 [Inspection and Teardown (`peekFront`, `peekRear`, `display`, `freeQueue`)](#55-inspection-and-teardown-peekfront-peekrear-display-freequeue)
   - 5.6 [Complete C11 Implementation: Simple Array Queue & Linked-List Queue](#56-complete-c11-implementation-simple-array-queue--linked-list-queue)
6. [Comparison Tables and Complexity Summary](#6-comparison-tables-and-complexity-summary)
   - 6.1 [Time and Space Complexity Table](#61-time-and-space-complexity-table)
   - 6.2 [Architectural Comparison: Simple Array Queue vs. Linked-List Queue vs. Stack](#62-architectural-comparison-simple-array-queue-vs-linked-list-queue-vs-stack)
7. [Common Errors, Pitfalls, and Edge Cases](#7-common-errors-pitfalls-and-edge-cases)
   - 7.1 [Pitfall 1: Dangling `rear` Pointer When Dequeueing the Last Node](#71-pitfall-1-dangling-rear-pointer-when-dequeueing-the-last-node)
   - 7.2 [Pitfall 2: Not Resetting `front = rear = -1` When Last Array Element Is Dequeued](#72-pitfall-2-not-resetting-front--rear---1-when-last-array-element-is-dequeued)
   - 7.3 [Pitfall 3: Off-by-One Size Calculation in Linear Array Queues](#73-pitfall-3-off-by-one-size-calculation-in-linear-array-queues)
   - 7.4 [Pitfall 4: Confusing `front = 0` Initialization with `front = -1`](#74-pitfall-4-confusing-front--0-initialization-with-front---1)
   - 7.5 [Boundary Conditions Checklist](#75-boundary-conditions-checklist)
8. [Summary and Revision Checklist](#8-summary-and-revision-checklist)
   - 8.1 [Key Takeaways and Core Invariants](#81-key-takeaways-and-core-invariants)
   - 8.2 [Self-Test Revision Questions](#82-self-test-revision-questions)
   - 8.3 [Navigation Links](#83-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 What Is a Queue?

A **Queue** is an ordered, linear Abstract Data Type (ADT) in which all insertions occur strictly at one end—called the **`rear`** (or *tail* / *back*)—and all deletions occur strictly at the opposite end—called the **`front`** (or *head*). Because the first element inserted into the queue is always the first element removed from it, a queue enforces the **FIFO (First-In, First-Out)**—also called **FCFS (First-Come, First-Served)**—access discipline.

```
                        QUEUE (FIFO: First-In, First-Out)

              +--------+--------+--------+--------+--------+
   dequeue <--|   10   |   20   |   30   |   40   |   50   |<-- enqueue(60)
  (removes    +--------+--------+--------+--------+--------+    (inserts at
   oldest)      ^                                   ^            newest end)
                |                                   |
              front                               rear
          (head / exit)                      (tail / entry)
```

### 1.2 Why Do We Need Queues? The Problem They Solve

In computing systems, resources such as a CPU core, a network interface card (NIC), a disk controller, or an office printer are **finite** and often slower than the burst rate at which requests arrive. Without an orderly buffer between a fast producer and a slower consumer, requests would either be dropped or served in an arbitrary, unfair order.

A **queue** solves two fundamental engineering problems simultaneously:
1. **Temporal Fairness (Starvation Prevention):** By serving items strictly in arrival order ($t_1 < t_2 \implies \text{served}(t_1) < \text{served}(t_2)$), no request waits indefinitely behind newer requests.
2. **Asynchronous Rate Decoupling (Buffering):** A producer process can place data items into the `rear` of a queue at its own speed while a consumer process extracts items from the `front` at its own speed, preserving the exact chronological sequence of events.

### 1.3 Relationship to Prior Concepts: FIFO Queues vs. LIFO Stacks

Both **Stacks** and **Queues** are restricted linear data structures: unlike a general array or linked list where insertions and deletions are permitted at any arbitrary index $i$, stacks and queues restrict modifications to their boundary endpoints so that every core operation runs in worst-case $O(1)$ time.

| Characteristic | Stack (LIFO) | Queue (FIFO) |
| :--- | :--- | :--- |
| **Ordering Discipline** | **Last-In, First-Out (LIFO)** | **First-In, First-Out (FIFO)** |
| **Active Endpoints** | **1 endpoint** (`top`) for both push and pop | **2 endpoints** (`rear` for insert, `front` for delete) |
| **Insertion Operation** | `push(x)` at `top` | `enqueue(x)` at `rear` |
| **Deletion Operation** | `pop()` from `top` | `dequeue()` from `front` |
| **Inspection Operation** | `peek()` / `top()` | `peekFront()` and `peekRear()` |
| **Primary Tree/Graph Traversal** | Depth-First Search (**DFS**), Backtracking | Breadth-First Search (**BFS**), Level-Order Traversal |

### 1.4 Real-World Analogies and Computer Science Applications

- **Real-World Analogies:**
  - Customers standing in a single-file checkout line at a supermarket or ticket counter.
  - Vehicles passing through a single-lane toll booth or highway tunnel.
- **Systems & Algorithmic Applications:**
  1. **Operating System Scheduling:** The OS **Ready Queue** holds processes waiting for CPU execution; **I/O Device Queues** hold pending read/write requests.
  2. **Graph & Tree Algorithms:** **Breadth-First Search (BFS)** and binary tree **Level-Order Traversal** rely on a FIFO queue to explore nodes layer by layer according to shortest unweighted edge distance.
  3. **Producer-Consumer I/O Buffers:** Keyboard keystroke buffers, print spoolers, UART serial rings, and TCP/IP packet transmit/receive rings.
  4. **Inter-Process Communication (IPC):** POSIX pipes (`|`) and message queues transfer byte streams in strict FIFO order.

### 1.5 Learning Objectives

After studying this note, you will be able to:
1. Formally define the **Queue ADT**, its core operations, and mathematical FIFO invariants.
2. Implement a **Simple (Linear) Array Queue** in C11 and compare the `front = -1, rear = -1` and `front = 0, rear = -1` index conventions.
3. Diagnose and prove why a simple linear array queue suffers from **False Overflow (Rightward Drift)** and why shifting elements degrades `dequeue` to $O(n)$.
4. Implement a dynamic **Linked-List Queue** in C11 maintaining both `front` and `rear` pointers for worst-case $O(1)$ `enqueue` and `dequeue`, handling the empty-queue and single-element boundary cases without memory leaks or dangling pointers.

---

## 2. Definitions and Terminology

### 2.1 Formal Definition of the Queue Abstract Data Type (ADT)

Let $Q = \langle a_1, a_2, \dots, a_k \rangle$ be a finite sequence of $k \ge 0$ elements drawn from a type universe $\mathcal{T}$.
- When $k = 0$, $Q = \langle \rangle$ is called the **empty queue**.
- When $k \ge 1$, $a_1$ is the **`front`** element (the oldest element currently in $Q$), and $a_k$ is the **`rear`** element (the most recently inserted element in $Q$).
- **FIFO Ordering Axiom:** If element $x$ is inserted before element $y$ ($t_{\text{enqueue}}(x) < t_{\text{enqueue}}(y)$), then $x$ must be removed before $y$ ($t_{\text{dequeue}}(x) < t_{\text{dequeue}}(y)$).

### 2.2 Intuitive Explanation and Two-Ended Access Model

Unlike a stack where `top` moves back and forth along a single axis, a queue acts like a **moving window** with two tracking markers:
- Every `enqueue(x)` advances **`rear`** forward to accommodate the new arrival.
- Every `dequeue()` advances **`front`** forward as the oldest element departs.

```
  Index:     0      1      2      3      4      5      6      7
          +------+------+------+------+------+------+------+------+
  Array:  |  --  |  --  |  A   |  B   |  C   |  D   |  --  |  --  |
          +------+------+------+------+------+------+------+------+
                           ^                    ^
                           |                    |
                         front = 2            rear = 5
                         (next to exit)       (last inserted)
                         |<--- Active Window -->|
                          size = 5 - 2 + 1 = 4
```

### 2.3 Core Terminology Table

| Term | Symbol / Identifier | Definition |
| :--- | :--- | :--- |
| **Front (Head)** | `front` / `head` | Index or pointer identifying the oldest element in the queue (deletion point). |
| **Rear (Tail / Back)** | `rear` / `tail` | Index or pointer identifying the newest element in the queue (insertion point). |
| **Enqueue** | `enqueue(Q, x)` | Operation that appends element `x` at the `rear` of queue `Q`. |
| **Dequeue** | `dequeue(Q, &out)` | Operation that removes and returns the element at the `front` of queue `Q`. |
| **Peek Front** | `peekFront(Q)` | Reads the `front` element without removing it from `Q`. |
| **Peek Rear** | `peekRear(Q)` | Reads the `rear` element without removing it from `Q`. |
| **Queue Overflow** | `isFull(Q) == true` | Attempt to `enqueue` into a queue that has no available storage capacity. |
| **Queue Underflow** | `isEmpty(Q) == true` | Attempt to `dequeue` or `peek` from an empty queue ($k = 0$). |
| **False Overflow** | Rightward Drift | Condition in a linear array queue where `rear == CAPACITY - 1` even though indices $0 \dots \text{front}-1$ are unoccupied. |

### 2.4 Mathematical Properties, Invariants, and Formulas

For a **Simple (Linear) Array Queue** of fixed capacity $N = \text{CAPACITY}$ (indices $0, 1, \dots, N-1$) using the standard `-1` sentinel convention:

1. **Empty State Invariant:**
   $$\text{isEmpty}(Q) \iff (\text{front} == -1) \land (\text{rear} == -1)$$

2. **Single-Element State Invariant ($k = 1$):**
   $$\text{size}(Q) = 1 \iff (\text{front} \neq -1) \land (\text{front} == \text{rear})$$

3. **Current Element Count ($k$):**
   $$\text{size}(Q) = \begin{cases} 0 & \text{if } \text{front} == -1 \\ \text{rear} - \text{front} + 1 & \text{if } \text{front} \ge 0 \end{cases}$$

4. **Linear Array Full Condition (True or False Overflow):**
   $$\text{isFull}(Q) \iff \text{rear} == N - 1$$

5. **Total Enqueue vs. Dequeue Count Invariant:**
   Let $E$ be the total number of successful `enqueue` operations since initialization and $D$ be the total number of successful `dequeue` operations. At every point in the queue's lifetime:
   $$0 \le D \le E, \qquad \text{size}(Q) = E - D$$

---

## 3. Representation and Memory Layout in C

### 3.1 Simple (Linear) Array-Based Queue Representation

In C, a static linear array queue bundles a fixed-size array `data[CAPACITY]` with two integer indices `front` and `rear`:

```c
#define CAPACITY 5

typedef struct {
    int data[CAPACITY];  /* Contiguous storage slots [0 .. CAPACITY - 1] */
    int front;           /* Index of oldest element (-1 when empty)      */
    int rear;            /* Index of newest element (-1 when empty)      */
} ArrayQueue;
```

#### Memory Layout in RAM (`CAPACITY = 5`, assuming `sizeof(int) = 4` bytes, base address `0x1000`)

```
  Struct ArrayQueue in Memory (Total size = 5 * 4 + 4 + 4 = 28 bytes):
  +---------------------------------------------------------------+---------+---------+
  | data[0]  | data[1]  | data[2]  | data[3]  | data[4]  | front   | rear    |
  |  (free)  |   15     |   25     |   35     |  (free)  |    1    |    3    |
  +----------+----------+----------+----------+----------+---------+---------+
  0x1000     0x1004     0x1008     0x100C     0x1010     0x1014    0x1018
                 ^                     ^
                 |                     |
               front                 rear
```

### 3.2 `front` and `rear` Index Conventions Compared

Textbooks and exams use two common initialization conventions for linear array queues. You must know both so you never misread exam code:

| State / Operation | Convention A: **`front = -1, rear = -1`** (Standard Sentinel) | Convention B: **`front = 0, rear = -1`** (Crossed-Index) |
| :--- | :--- | :--- |
| **Initial State** | `front = -1`, `rear = -1` | `front = 0`, `rear = -1` |
| **Empty Condition** | `front == -1` (or `front == -1 && rear == -1`) | `front > rear` (initially `0 > -1`) |
| **Single Element ($k=1$)** | `front == rear && front != -1` (e.g., `0 == 0`) | `front == rear` (e.g., `0 == 0`) |
| **Full Condition** | `rear == CAPACITY - 1` | `rear == CAPACITY - 1` |
| **Size Formula** | `(front == -1) ? 0 : (rear - front + 1)` | `rear - front + 1` (works even when empty: `-1 - 0 + 1 = 0`!) |
| **On First `enqueue(x)`** | Set `front = 0`, `rear = 0`, `data[0] = x` | Increment `rear` to `0`, `data[0] = x` (`front` is already `0`) |
| **On Last `dequeue()`** | Reset `front = -1, rear = -1` when `front == rear` | Increment `front` so `front == rear + 1` (can reset to `0, -1`) |

> **Why We Standardize on Convention A (`front = -1, rear = -1`):**
> Convention A extends seamlessly to **Circular Queues** and **Deques** (Topic 2), where `front > rear` can happen legitimately during circular wraparound and therefore *cannot* be used as an empty check. Furthermore, resetting `front = rear = -1` whenever the last element is dequeued immediately reclaims all $N$ slots of the array whenever the queue becomes completely empty.

### 3.3 Linked-List-Based Queue Representation

A linked-list queue eliminates fixed capacity constraints and the rightward-drift problem by allocating a heap node per element and maintaining two pointers in a descriptor struct: `front` (pointing to the head node) and `rear` (pointing to the tail node), plus an explicit `size` counter.

```c
typedef struct QueueNode {
    int data;
    struct QueueNode *next;
} QueueNode;

typedef struct {
    QueueNode *front;    /* Points to oldest node (head of list) */
    QueueNode *rear;     /* Points to newest node (tail of list) */
    size_t size;         /* Number of currently active nodes     */
} LinkedQueue;
```

#### ASCII Memory Diagram of a Linked-List Queue

```
  LinkedQueue Header (Stack / Heap)
  +-------------------+-------------------+----------+
  | front: 0x5010     | rear:  0x5090     | size: 3  |
  +---------+---------+---------+---------+----------+
            |                   |
            v                   +---------------------------------------+
       [Node @ 0x5010]             [Node @ 0x5040]             [Node @ 0x5090]
       +------+----------+         +------+----------+         +------+----------+
       |  10  | 0x5040 --+-------->|  20  | 0x5090 --+-------->|  30  |   NULL   |
       +------+----------+         +------+----------+         +------+----------+
       (Oldest: dequeue here)                                  (Newest: enqueue here)
```

### 3.4 Static vs. Dynamic Storage: Architectural Trade-Offs

| Dimension | Simple (Linear) Array Queue | Singly Linked-List Queue |
| :--- | :--- | :--- |
| **Memory Allocation** | Static/contiguous block allocated once (`CAPACITY * sizeof(int)`). | Dynamic per-node heap allocation (`malloc` on `enqueue`, `free` on `dequeue`). |
| **Memory Overhead per Element** | $0$ bytes of structural overhead per occupied slot; wasted space in unused slots. | `sizeof(QueueNode*)` (8 bytes on 64-bit systems) + heap allocator metadata per node. |
| **Cache Locality** | **Excellent:** Sequential memory addresses prefetch cleanly into L1/L2 CPU cache lines. | **Poor:** Non-contiguous heap nodes cause pointer-chasing cache misses. |
| **Capacity Limit** | Hard compile-time/init-time limit; vulnerable to **False Overflow**. | Bounded only by available system RAM; **no False Overflow**. |

---

## 4. Operations and Algorithms: Simple (Linear) Array Queue

### 4.1 Initialization (`initArrayQueue`) and State Predicates (`isEmpty`, `isFull`, `size`)

- **Purpose:** Set the queue to a known empty state and query its occupancy in $O(1)$ time.
- **Preconditions:** Pointer `q` must be non-`NULL`.
- **Complexity:** Time $O(1)$, Auxiliary Space $O(1)$.

```text
ALGORITHM InitArrayQueue(Q)
1. Q.front <- -1
2. Q.rear  <- -1

ALGORITHM IsEmpty(Q)
1. RETURN (Q.front == -1)

ALGORITHM IsFull(Q)
1. RETURN (Q.rear == CAPACITY - 1)

ALGORITHM Size(Q)
1. IF Q.front == -1 THEN RETURN 0
2. RETURN (Q.rear - Q.front + 1)
```

### 4.2 Insertion at Rear (`enqueue`)

- **Purpose:** Insert a new element `value` at the `rear` of the linear array queue.
- **Preconditions:** Queue `Q` is initialized and not full (`rear < CAPACITY - 1`).
- **Step-by-Step Algorithm:**
  1. **Overflow Check:** If `rear == CAPACITY - 1`, report **Queue Overflow** and return `false`.
  2. **First-Element Check:** If the queue is currently empty (`front == -1`), set `front = 0` so `front` points to the first valid slot.
  3. **Advance Rear:** Increment `rear` by 1 (`rear = rear + 1`).
  4. **Store Value:** Write `data[rear] = value` and return `true`.
- **Complexity:** Time $O(1)$, Auxiliary Space $O(1)$.

```text
ALGORITHM EnqueueArray(Q, value)
Input:  ArrayQueue Q, element value
Output: Boolean indicating success (true) or overflow (false)
1. IF Q.rear == CAPACITY - 1 THEN
2.     PRINT "Queue Overflow"
3.     RETURN false
4. END IF
5. IF Q.front == -1 THEN
6.     Q.front <- 0
7. END IF
8. Q.rear <- Q.rear + 1
9. Q.data[Q.rear] <- value
10. RETURN true
```

#### Before-and-After ASCII Diagram for `enqueue`

```
  Case 1: Enqueue(10) into an EMPTY Queue (front = -1, rear = -1)
  BEFORE: [ -- | -- | -- | -- | -- ]   front = -1, rear = -1
  AFTER:  [ 10 | -- | -- | -- | -- ]   front =  0, rear =  0   (Both set to 0!)
            ^
         front,rear

  Case 2: Enqueue(20) into a NON-EMPTY Queue (front = 0, rear = 0)
  BEFORE: [ 10 | -- | -- | -- | -- ]   front =  0, rear =  0
  AFTER:  [ 10 | 20 | -- | -- | -- ]   front =  0, rear =  1   (Only rear advances)
            ^    ^
          front rear
```

### 4.3 Deletion from Front (`dequeue`)

- **Purpose:** Remove and return the oldest element located at `data[front]`.
- **Preconditions:** Queue `Q` is initialized and not empty (`front != -1`).
- **Step-by-Step Algorithm:**
  1. **Underflow Check:** If `front == -1`, report **Queue Underflow** and return `false`.
  2. **Extract Value:** Copy `data[front]` into `*out_value`.
  3. **Update Markers:**
     - **Single-Element Case (`front == rear`):** The element being removed is the *only* element left in the queue. Reset both `front = -1` and `rear = -1` so the queue returns to its clean initial empty state.
     - **Multi-Element Case (`front < rear`):** Increment `front = front + 1`.
  4. Return `true`.
- **Complexity:** Time $O(1)$, Auxiliary Space $O(1)$.

```text
ALGORITHM DequeueArray(Q, out_value)
Input:  ArrayQueue Q, output reference out_value
Output: Boolean indicating success (true) or underflow (false)
1. IF Q.front == -1 THEN
2.     PRINT "Queue Underflow"
3.     RETURN false
4. END IF
5. out_value <- Q.data[Q.front]
6. IF Q.front == Q.rear THEN
7.     Q.front <- -1      /* Queue is now completely empty; reclaim all slots */
8.     Q.rear  <- -1
9. ELSE
10.    Q.front <- Q.front + 1
11. END IF
12. RETURN true
```

#### Before-and-After ASCII Diagram for `dequeue`

```
  Case 1: Dequeue() from a Multi-Element Queue (front = 0, rear = 1)
  BEFORE: [ 10 | 20 | -- | -- | -- ]   front =  0, rear =  1
            ^    ^
          front rear
  AFTER:  [ xx | 20 | -- | -- | -- ]   front =  1, rear =  1, returned = 10
                 ^
              front,rear

  Case 2: Dequeue() the LAST Remaining Element (front = 1, rear = 1)
  BEFORE: [ xx | 20 | -- | -- | -- ]   front =  1, rear =  1
  AFTER:  [ xx | xx | -- | -- | -- ]   front = -1, rear = -1, returned = 20
```

### 4.4 Inspection Operations (`peekFront`, `peekRear`, `display`)

- **`peekFront(Q, &out)`:** Checks `front != -1`, copies `Q->data[Q->front]` into `*out` in $O(1)$ time without modifying `front`.
- **`peekRear(Q, &out)`:** Checks `rear != -1`, copies `Q->data[Q->rear]` into `*out` in $O(1)$ time without modifying `rear`.
- **`display(Q)`:** Iterates index `i` from `front` to `rear` inclusive in $O(k)$ time (where $k = \text{rear} - \text{front} + 1$).

### 4.5 Step-by-Step Index and Array Trace

Let `CAPACITY = 4`. Trace the following sequence of operations from initialization:

```
  Step 0: initArrayQueue(Q)
          Index:   0    1    2    3
                 +----+----+----+----+
          Array: | -- | -- | -- | -- |   front = -1, rear = -1, size = 0 (EMPTY)
                 +----+----+----+----+

  Step 1: enqueue(Q, 10)
                 +----+----+----+----+
          Array: | 10 | -- | -- | -- |   front =  0, rear =  0, size = 1
                 +----+----+----+----+
                   ^
                 F, R

  Step 2: enqueue(Q, 20), enqueue(Q, 30)
                 +----+----+----+----+
          Array: | 10 | 20 | 30 | -- |   front =  0, rear =  2, size = 3
                 +----+----+----+----+
                   ^         ^
                   F         R

  Step 3: dequeue(Q) -> returns 10; dequeue(Q) -> returns 20
                 +----+----+----+----+
          Array: | xx | xx | 30 | -- |   front =  2, rear =  2, size = 1
                 +----+----+----+----+
                             ^
                           F, R

  Step 4: enqueue(Q, 40)
                 +----+----+----+----+
          Array: | xx | xx | 30 | 40 |   front =  2, rear =  3, size = 2 (rear == CAPACITY - 1!)
                 +----+----+----+----+
                             ^    ^
                             F    R
```

### 4.6 The "False Overflow" (Rightward Drift) Problem

Look closely at **Step 4** in the trace above. Suppose we now call `enqueue(Q, 50)`:

```
                 |<-- 2 VACANT SLOTS -->|
  Index:            0      1               2      3
                 +------+------+        +------+------+
  Array State:   | free | free |        |  30  |  40  |   <-- rear == 3 (CAPACITY - 1)
                 +------+------+        +------+------+
                                           ^      ^
                                         front   rear
```

- **What happens?** `isFull(Q)` checks `rear == CAPACITY - 1` ($3 == 3$), which evaluates to **`true`**, so `enqueue(Q, 50)` fails with **Queue Overflow**!
- **Why is this called "False Overflow" (or Rightward Drift)?**
  The queue has a capacity of $N = 4$ slots, and it currently holds only $k = 2$ elements (`30` and `40`). Slots `0` and `1` are completely vacant because elements `10` and `20` were dequeued earlier. However, because `front` and `rear` only ever move **rightward** ($\text{index} + 1$) while the queue stays non-empty, the active window drifts to the right edge of the array and hits the wall at `CAPACITY - 1`.

#### Why Not Just Shift Elements Left on Every `dequeue`?
One naive idea is to shift all remaining elements one position to the left (`data[i] = data[i + 1]`) whenever `dequeue()` is called so that `front` always stays at index `0`:

```c
/* NAIVE SHIFTING DEQUEUE — DO NOT USE IN PRODUCTION */
int val = q->data[0];
for (int i = 0; i < q->rear; i++) {
    q->data[i] = q->data[i + 1];  /* Shifts k - 1 elements on EVERY dequeue! */
}
q->rear--;
```

- **Why this fails algorithmically:** Shifting $k - 1$ elements degrades `dequeue()` from **$O(1)$** to **$O(n)$** worst-case time! Processing $n$ `dequeue` operations would then take $\sum_{i=1}^{n} O(i) = O(n^2)$ total time instead of $O(n)$.
- **The Two Proper $O(1)$ Solutions:**
  1. **Circular Array Queue (Topic 2):** Use modular arithmetic (`rear = (rear + 1) % CAPACITY`) so `rear` wraps around to index `0` and reuses vacant prefix slots in $O(1)$ time.
  2. **Linked-List Queue (Section 5 below):** Allocate and free nodes dynamically on the heap so there is no fixed array boundary.

---

## 5. Operations and Algorithms: Linked-List Queue

### 5.1 Why Both `front` and `rear` Pointers Are Essential for $O(1)$ Performance

Suppose we tried to implement a queue using a singly linked list with **only** a `head` (`front`) pointer:
- Deleting from `head` takes $O(1)$ time (`head = head->next`).
- However, inserting at the tail requires traversing all $n$ nodes from `head` to find the last node, costing **$O(n)$** time!

Conversely, if we maintained a `rear` pointer at the head of a singly linked list and `front` at the tail:
- Inserting at `head` takes $O(1)$ time, but deleting from the tail requires $O(n)$ traversal to find the **second-to-last** node!

Therefore, a singly linked list queue **must** orient its links from **`front` (head) $\to$ `rear` (tail)**:
- **`front` (`head`):** Supports $O(1)$ deletion (`front = front->next`).
- **`rear` (`tail`):** Supports $O(1)$ insertion (`rear->next = new_node; rear = new_node`).

### 5.2 Initialization (`initLinkedQueue`) and Empty Check

```text
ALGORITHM InitLinkedQueue(Q)
1. Q.front <- NULL
2. Q.rear  <- NULL
3. Q.size  <- 0

ALGORITHM IsLinkedQueueEmpty(Q)
1. RETURN (Q.front == NULL)
```

### 5.3 Dynamic Enqueue (`enqueueLinked`) and the Empty-Queue Edge Case

- **Purpose:** Allocate a new node on the heap and append it at `rear` in $O(1)$ time.
- **Step-by-Step Algorithm:**
  1. Allocate `new_node = malloc(sizeof(QueueNode))`. If `new_node == NULL`, return `false` (Heap Overflow).
  2. Set `new_node->data = value` and `new_node->next = NULL`.
  3. **Critical Edge Case — Empty Queue (`Q->rear == NULL`):**
     Both `Q->front` and `Q->rear` must be updated to point to `new_node`.
  4. **Standard Case — Non-Empty Queue (`Q->rear != NULL`):**
     Link the current tail to the new node (`Q->rear->next = new_node`), then advance `Q->rear = new_node`.
  5. Increment `Q->size` and return `true`.

```
  Case A: Enqueue(10) into an EMPTY LinkedQueue (front == NULL, rear == NULL)
  BEFORE:  front -> NULL,  rear -> NULL
  AFTER:   front -+
                  v
                +----+------+
                | 10 | NULL |
                +----+------+
                  ^
           rear --+

  Case B: Enqueue(20) into a NON-EMPTY LinkedQueue
  BEFORE:  front -> [ 10 | NULL ] <- rear
  STEP 1:  Create new_node [ 20 | NULL ]
  STEP 2:  rear->next = new_node:   front -> [ 10 | * ] ---> [ 20 | NULL ]
  STEP 3:  rear = new_node:         front -> [ 10 | * ] ---> [ 20 | NULL ] <- rear
```

### 5.4 Dynamic Dequeue (`dequeueLinked`) and the Single-Node Edge Case

- **Purpose:** Remove the node at `front`, extract its value, free its heap memory, and maintain pointer invariants in $O(1)$ time.
- **Step-by-Step Algorithm:**
  1. **Underflow Check:** If `Q->front == NULL`, return `false`.
  2. Save a temporary pointer to the victim node: `QueueNode *temp = Q->front`.
  3. Copy `temp->data` into `*out_value`.
  4. Advance `Q->front = Q->front->next`.
  5. **Critical Edge Case — Dequeueing the Last Remaining Node (`Q->front == NULL`):**
     If advancing `Q->front` made it `NULL`, the queue is now empty! However, `Q->rear` is still pointing to `temp` (which is about to be freed). We **must** explicitly set `Q->rear = NULL` to prevent a **dangling pointer**.
  6. Call `free(temp)`, decrement `Q->size`, and return `true`.

```
  Dequeueing the ONLY Node in LinkedQueue:
  BEFORE:  front -+
                  v
                [ 10 | NULL ]  <-- temp
                  ^
           rear --+

  STEP 4:  front = front->next  ==>  front == NULL (but rear still points to temp!)
  STEP 5:  IF front == NULL THEN rear = NULL   <-- ESSENTIAL! Prevents dangling rear!
  STEP 6:  free(temp)
  AFTER:   front -> NULL,  rear -> NULL,  size = 0
```

### 5.5 Inspection and Teardown (`peekFront`, `peekRear`, `display`, `freeQueue`)

- **`peekFront` / `peekRear`:** Dereference `Q->front->data` or `Q->rear->data` in $O(1)$ time after checking `Q->front != NULL`.
- **`freeQueue`:** Repeatedly dequeues or walks `curr = Q->front` freeing every node in $O(n)$ time and resets `front = rear = NULL` and `size = 0`.

### 5.6 Complete C11 Implementation: Simple Array Queue & Linked-List Queue

The following self-contained C11 program implements both the **Simple (Linear) Array Queue** (including an explicit demonstration of the **False Overflow** problem and recovery when completely emptied) and the **Linked-List Queue** (verifying single-node and empty-queue edge cases).

```c
/*
 * File: queue_fundamentals_demo.c
 * Standard: C11
 * Description: Complete implementation of Simple (Linear) Array Queue
 *              (demonstrating FIFO operations and False Overflow) and
 *              Dynamic Singly Linked-List Queue in O(1) time.
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

/* =========================================================================
 * PART 1: SIMPLE (LINEAR) ARRAY QUEUE
 * ========================================================================= */

#define ARRAY_QUEUE_CAPACITY 4

typedef struct {
    int data[ARRAY_QUEUE_CAPACITY];
    int front;
    int rear;
} ArrayQueue;

void arrayQueueInit(ArrayQueue *q) {
    q->front = -1;
    q->rear  = -1;
}

bool arrayQueueIsEmpty(const ArrayQueue *q) {
    return (q->front == -1);
}

bool arrayQueueIsFull(const ArrayQueue *q) {
    return (q->rear == ARRAY_QUEUE_CAPACITY - 1);
}

int arrayQueueSize(const ArrayQueue *q) {
    if (arrayQueueIsEmpty(q)) {
        return 0;
    }
    return (q->rear - q->front + 1);
}

bool arrayQueueEnqueue(ArrayQueue *q, int value) {
    if (arrayQueueIsFull(q)) {
        printf("  [ArrayQueue] OVERFLOW: Cannot enqueue %d (front=%d, rear=%d, size=%d/%d)\n",
               value, q->front, q->rear, arrayQueueSize(q), ARRAY_QUEUE_CAPACITY);
        return false;
    }
    if (arrayQueueIsEmpty(q)) {
        q->front = 0;
    }
    q->rear++;
    q->data[q->rear] = value;
    printf("  [ArrayQueue] Enqueued %d -> (front=%d, rear=%d, size=%d)\n",
           value, q->front, q->rear, arrayQueueSize(q));
    return true;
}

bool arrayQueueDequeue(ArrayQueue *q, int *out_value) {
    if (arrayQueueIsEmpty(q)) {
        printf("  [ArrayQueue] UNDERFLOW: Cannot dequeue from empty queue.\n");
        return false;
    }
    *out_value = q->data[q->front];
    if (q->front == q->rear) {
        /* Last remaining element removed: reset to initial empty state */
        q->front = -1;
        q->rear  = -1;
    } else {
        q->front++;
    }
    printf("  [ArrayQueue] Dequeued %d -> (front=%d, rear=%d, size=%d)\n",
           *out_value, q->front, q->rear, arrayQueueSize(q));
    return true;
}

bool arrayQueuePeekFront(const ArrayQueue *q, int *out_value) {
    if (arrayQueueIsEmpty(q)) return false;
    *out_value = q->data[q->front];
    return true;
}

bool arrayQueuePeekRear(const ArrayQueue *q, int *out_value) {
    if (arrayQueueIsEmpty(q)) return false;
    *out_value = q->data[q->rear];
    return true;
}

void arrayQueueDisplay(const ArrayQueue *q) {
    printf("  [ArrayQueue State] Slots: [");
    for (int i = 0; i < ARRAY_QUEUE_CAPACITY; i++) {
        if (!arrayQueueIsEmpty(q) && i >= q->front && i <= q->rear) {
            printf(" %2d ", q->data[i]);
        } else {
            printf(" -- ");
        }
        if (i < ARRAY_QUEUE_CAPACITY - 1) printf("|");
    }
    printf("] (front=%d, rear=%d, size=%d)\n",
           q->front, q->rear, arrayQueueSize(q));
}

/* =========================================================================
 * PART 2: DYNAMIC LINKED-LIST QUEUE
 * ========================================================================= */

typedef struct QueueNode {
    int data;
    struct QueueNode *next;
} QueueNode;

typedef struct {
    QueueNode *front;
    QueueNode *rear;
    size_t size;
} LinkedQueue;

void linkedQueueInit(LinkedQueue *q) {
    q->front = NULL;
    q->rear  = NULL;
    q->size  = 0;
}

bool linkedQueueIsEmpty(const LinkedQueue *q) {
    return (q->front == NULL);
}

size_t linkedQueueSize(const LinkedQueue *q) {
    return q->size;
}

bool linkedQueueEnqueue(LinkedQueue *q, int value) {
    QueueNode *new_node = (QueueNode *)malloc(sizeof(QueueNode));
    if (new_node == NULL) {
        fprintf(stderr, "  [LinkedQueue] Memory allocation failed!\n");
        return false;
    }
    new_node->data = value;
    new_node->next = NULL;

    if (q->rear == NULL) {
        /* Queue was empty: both front and rear point to the new node */
        q->front = new_node;
        q->rear  = new_node;
    } else {
        /* Append after current rear and advance rear */
        q->rear->next = new_node;
        q->rear       = new_node;
    }
    q->size++;
    printf("  [LinkedQueue] Enqueued %d (size=%zu, front=%d, rear=%d)\n",
           value, q->size, q->front->data, q->rear->data);
    return true;
}

bool linkedQueueDequeue(LinkedQueue *q, int *out_value) {
    if (linkedQueueIsEmpty(q)) {
        printf("  [LinkedQueue] UNDERFLOW: Queue is empty.\n");
        return false;
    }
    QueueNode *victim = q->front;
    *out_value        = victim->data;
    q->front          = victim->next;

    if (q->front == NULL) {
        /* Last node was removed: MUST reset rear to NULL! */
        q->rear = NULL;
    }
    free(victim);
    q->size--;

    if (q->front != NULL) {
        printf("  [LinkedQueue] Dequeued %d (size=%zu, front=%d, rear=%d)\n",
               *out_value, q->size, q->front->data, q->rear->data);
    } else {
        printf("  [LinkedQueue] Dequeued %d (size=%zu, queue is now EMPTY: front=NULL, rear=NULL)\n",
               *out_value, q->size);
    }
    return true;
}

bool linkedQueuePeekFront(const LinkedQueue *q, int *out_value) {
    if (linkedQueueIsEmpty(q)) return false;
    *out_value = q->front->data;
    return true;
}

bool linkedQueuePeekRear(const LinkedQueue *q, int *out_value) {
    if (linkedQueueIsEmpty(q)) return false;
    *out_value = q->rear->data;
    return true;
}

void linkedQueueDisplay(const LinkedQueue *q) {
    printf("  [LinkedQueue State] front -> ");
    const QueueNode *curr = q->front;
    while (curr != NULL) {
        printf("[%d] -> ", curr->data);
        curr = curr->next;
    }
    printf("NULL (size=%zu)\n", q->size);
}

void linkedQueueFree(LinkedQueue *q) {
    QueueNode *curr = q->front;
    while (curr != NULL) {
        QueueNode *next = curr->next;
        free(curr);
        curr = next;
    }
    q->front = NULL;
    q->rear  = NULL;
    q->size  = 0;
}

/* =========================================================================
 * MAIN DRIVER: DEMONSTRATING ARRAY QUEUE & LINKED-LIST QUEUE
 * ========================================================================= */

int main(void) {
    int val = 0;

    printf("=== 1. SIMPLE (LINEAR) ARRAY QUEUE & FALSE OVERFLOW DEMO ===\n");
    ArrayQueue aq;
    arrayQueueInit(&aq);
    arrayQueueDisplay(&aq);

    arrayQueueEnqueue(&aq, 10);
    arrayQueueEnqueue(&aq, 20);
    arrayQueueEnqueue(&aq, 30);
    arrayQueueEnqueue(&aq, 40);
    arrayQueueDisplay(&aq);

    /* Dequeue two elements: frees slots 0 and 1 */
    arrayQueueDequeue(&aq, &val);
    arrayQueueDequeue(&aq, &val);
    arrayQueueDisplay(&aq);

    /* Attempt to enqueue 50: triggers FALSE OVERFLOW because rear == 3! */
    printf("\n  Attempting enqueue(50) when 2 slots (indices 0, 1) are free:\n");
    arrayQueueEnqueue(&aq, 50);

    /* Empty the remaining two elements so front == rear resets to -1 */
    printf("\n  Draining remaining elements to trigger full reset (front = rear = -1):\n");
    arrayQueueDequeue(&aq, &val);
    arrayQueueDequeue(&aq, &val);
    arrayQueueDisplay(&aq);

    printf("\n=== 2. DYNAMIC LINKED-LIST QUEUE DEMO ===\n");
    LinkedQueue lq;
    linkedQueueInit(&lq);

    linkedQueueEnqueue(&lq, 100);
    linkedQueueEnqueue(&lq, 200);
    linkedQueueEnqueue(&lq, 300);
    linkedQueueDisplay(&lq);

    int f_val = 0, r_val = 0;
    if (linkedQueuePeekFront(&lq, &f_val) && linkedQueuePeekRear(&lq, &r_val)) {
        printf("  [LinkedQueue Peek] front = %d, rear = %d\n", f_val, r_val);
    }

    linkedQueueDequeue(&lq, &val);
    linkedQueueDequeue(&lq, &val);
    linkedQueueDisplay(&lq);

    /* Dequeue the last remaining node (300) and verify clean empty state */
    linkedQueueDequeue(&lq, &val);
    linkedQueueDisplay(&lq);

    /* Verify enqueue works cleanly after becoming empty */
    linkedQueueEnqueue(&lq, 400);
    linkedQueueDisplay(&lq);

    linkedQueueFree(&lq);
    return 0;
}
```

#### Program Output:

```text
=== 1. SIMPLE (LINEAR) ARRAY QUEUE & FALSE OVERFLOW DEMO ===
  [ArrayQueue State] Slots: [ -- | -- | -- | -- ] (front=-1, rear=-1, size=0)
  [ArrayQueue] Enqueued 10 -> (front=0, rear=0, size=1)
  [ArrayQueue] Enqueued 20 -> (front=0, rear=1, size=2)
  [ArrayQueue] Enqueued 30 -> (front=0, rear=2, size=3)
  [ArrayQueue] Enqueued 40 -> (front=0, rear=3, size=4)
  [ArrayQueue State] Slots: [ 10 | 20 | 30 | 40 ] (front=0, rear=3, size=4)
  [ArrayQueue] Dequeued 10 -> (front=1, rear=3, size=3)
  [ArrayQueue] Dequeued 20 -> (front=2, rear=3, size=2)
  [ArrayQueue State] Slots: [ -- | -- | 30 | 40 ] (front=2, rear=3, size=2)

  Attempting enqueue(50) when 2 slots (indices 0, 1) are free:
  [ArrayQueue] OVERFLOW: Cannot enqueue 50 (front=2, rear=3, size=2/4)

  Draining remaining elements to trigger full reset (front = rear = -1):
  [ArrayQueue] Dequeued 30 -> (front=3, rear=3, size=1)
  [ArrayQueue] Dequeued 40 -> (front=-1, rear=-1, size=0)
  [ArrayQueue State] Slots: [ -- | -- | -- | -- ] (front=-1, rear=-1, size=0)

=== 2. DYNAMIC LINKED-LIST QUEUE DEMO ===
  [LinkedQueue] Enqueued 100 (size=1, front=100, rear=100)
  [LinkedQueue] Enqueued 200 (size=2, front=100, rear=200)
  [LinkedQueue] Enqueued 300 (size=3, front=100, rear=300)
  [LinkedQueue State] front -> [100] -> [200] -> [300] -> NULL (size=3)
  [LinkedQueue Peek] front = 100, rear = 300
  [LinkedQueue] Dequeued 100 (size=2, front=200, rear=300)
  [LinkedQueue] Dequeued 200 (size=1, front=300, rear=300)
  [LinkedQueue State] front -> [300] -> NULL (size=1)
  [LinkedQueue] Dequeued 300 (size=0, queue is now EMPTY: front=NULL, rear=NULL)
  [LinkedQueue State] front -> NULL (size=0)
  [LinkedQueue] Enqueued 400 (size=1, front=400, rear=400)
  [LinkedQueue State] front -> [400] -> NULL (size=1)
```

---

## 6. Comparison Tables and Complexity Summary

### 6.1 Time and Space Complexity Table

Let $n$ be the number of elements currently in the queue and $N = \text{CAPACITY}$ be the static array size.

| Operation | Simple Array Queue (No Shift) | Simple Array Queue (With Shift on Dequeue) | Singly Linked-List Queue (`front` + `rear`) | Singly Linked-List (`front` only, no `rear`) |
| :--- | :---: | :---: | :---: | :---: |
| **`enqueue(x)`** | $O(1)$ | $O(1)$ | $O(1)$ | $O(n)$ |
| **`dequeue()`** | $O(1)$ | $O(n)$ | $O(1)$ | $O(1)$ |
| **`peekFront()`** | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **`peekRear()`** | $O(1)$ | $O(1)$ | $O(1)$ | $O(n)$ |
| **`isEmpty()` / `size()`** | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| **`display()`** | $O(n)$ | $O(n)$ | $O(n)$ | $O(n)$ |
| **Total Space** | $\Theta(N)$ static | $\Theta(N)$ static | $\Theta(n)$ dynamic | $\Theta(n)$ dynamic |
| **False Overflow?** | **Yes** | No | No | No |

### 6.2 Architectural Comparison: Simple Array Queue vs. Linked-List Queue vs. Stack

| Feature | Simple Array Queue | Linked-List Queue | Array/Linked Stack |
| :--- | :--- | :--- | :--- |
| **Ordering Policy** | FIFO (First-In, First-Out) | FIFO (First-In, First-Out) | LIFO (Last-In, First-Out) |
| **Pointers / Indices Maintained** | Two indices: `front`, `rear` | Two node pointers: `front`, `rear` | Single marker: `top` |
| **Slot Reuse Before Empty** | **No** — slots $0 \dots \text{front}-1$ are unusable until queue completely empties | **Yes** — `free()` immediately returns memory to heap | **Yes** — `top--` immediately reclaims popped slot |
| **Best Suited For** | Single-batch fixed-size buffering where total insertions $\le N$ | Unbounded or unpredictable request streams | Recursion, undo history, expression parsing, DFS |

---

## 7. Common Errors, Pitfalls, and Edge Cases

### 7.1 Pitfall 1: Dangling `rear` Pointer When Dequeueing the Last Node

When a `LinkedQueue` has only $1$ node (`front == rear`), advancing `front = front->next` sets `front` to `NULL`. If you forget to also reset `rear = NULL`, `rear` remains pointing to the `free()`d node! The very next `enqueue()` will execute `q->rear->next = new_node` (**use-after-free** undefined behavior) and leave `q->front == NULL`.

```c
/* BUG: Forgetting to update q->rear when the last node is dequeued */
QueueNode *victim = q->front;
*out_value = victim->data;
q->front = victim->next;
free(victim); /* BUG: If victim was the last node, q->rear now points to freed memory! */

/* FIX: Always check if q->front became NULL and reset q->rear = NULL */
QueueNode *victim = q->front;
*out_value = victim->data;
q->front = victim->next;
if (q->front == NULL) {
    q->rear = NULL; /* Cleanly resets tail pointer on last-node removal */
}
free(victim);
```

### 7.2 Pitfall 2: Not Resetting `front = rear = -1` When Last Array Element Is Dequeued

```c
/* BUG: Only incrementing front without checking front == rear */
*out_value = q->data[q->front];
q->front++; /* BUG: When last element is removed, front becomes rear + 1,
               and rear stays at its old index instead of resetting to -1! */

/* FIX: Reset both indices to -1 when the last element is removed */
*out_value = q->data[q->front];
if (q->front == q->rear) {
    q->front = -1;
    q->rear  = -1;
} else {
    q->front++;
}
```

### 7.3 Pitfall 3: Off-by-One Size Calculation in Linear Array Queues

Because both `front` and `rear` are **inclusive** indices of occupied elements, the number of elements in `[front .. rear]` is `rear - front + 1`, **not** `rear - front`. Moreover, when the queue is empty (`front == -1, rear == -1`), blindly evaluating `rear - front + 1` yields `(-1) - (-1) + 1 = 1` instead of `0`!

```c
/* BUG: Returns 1 when queue is empty (-1 - (-1) + 1 == 1)! */
int badSize(const ArrayQueue *q) {
    return q->rear - q->front + 1;
}

/* FIX: Explicitly guard the empty state first */
int goodSize(const ArrayQueue *q) {
    return (q->front == -1) ? 0 : (q->rear - q->front + 1);
}
```

### 7.4 Pitfall 4: Confusing `front = 0` Initialization with `front = -1`

If you initialize `front = -1, rear = -1`, you **must** set `front = 0` during the first `enqueue`. If you forget, `front` remains `-1`, and the first `dequeue` will read `q->data[-1]` (out-of-bounds memory corruption!).

```c
/* BUG: Leaving front at -1 on the first enqueue */
q->rear++;
q->data[q->rear] = value; /* front is still -1! */

/* FIX: Transition front from -1 to 0 when inserting into an empty queue */
if (q->front == -1) {
    q->front = 0;
}
q->rear++;
q->data[q->rear] = value;
```

### 7.5 Boundary Conditions Checklist

- [ ] **Empty Queue (`size == 0`):** Does `dequeue`, `peekFront`, and `peekRear` safely reject the call without dereferencing `NULL` or indexing `data[-1]`?
- [ ] **Transition $0 \to 1$ (`enqueue` into empty queue):** Are **both** `front` and `rear` updated to point to the single element (`0` in array, `new_node` in linked list)?
- [ ] **Transition $1 \to 0$ (`dequeue` from 1-element queue):** Are **both** `front` and `rear` reset to the empty sentinel (`-1` in array, `NULL` in linked list)?
- [ ] **Full Array Queue (`rear == CAPACITY - 1`):** Does `enqueue` check for overflow *before* incrementing `rear`?
- [ ] **Heap Exhaustion (`malloc == NULL`):** Does `linkedQueueEnqueue` check `new_node != NULL` before dereferencing it?

---

## 8. Summary and Revision Checklist

### 8.1 Key Takeaways and Core Invariants

1. **FIFO Discipline:** A queue inserts at `rear` (`enqueue`) and removes from `front` (`dequeue`), guaranteeing First-In, First-Out service order.
2. **Simple Array Queue Invariants (`front = -1, rear = -1` convention):**
   - Empty iff `front == -1`.
   - Single element iff `front != -1 && front == rear`.
   - Size $= (\text{front} == -1) \ ? \ 0 : (\text{rear} - \text{front} + 1)$.
3. **False Overflow:** In a simple linear array queue, `rear == CAPACITY - 1` blocks further insertions even when slots $0 \dots \text{front}-1$ are vacant. Shifting elements costs $O(n)$; **Circular Queues** solve this in $O(1)$.
4. **Linked-List Queue Invariants:** Maintaining both `front` (head) and `rear` (tail) pointers achieves worst-case $O(1)$ `enqueue` and `dequeue` with zero False Overflow. Always update both `front` and `rear` on $0 \to 1$ and $1 \to 0$ transitions.

### 8.2 Self-Test Revision Questions

1. **Trace Question:** Given a linear array queue of `CAPACITY = 5` initialized with `front = -1, rear = -1`, what are the values of `front`, `rear`, and `size` after: `enqueue(5)`, `enqueue(8)`, `enqueue(12)`, `dequeue()`, `enqueue(15)`, `dequeue()`?
   *(Answer: `front = 2`, `rear = 3`, `size = 2`, active elements: `12` at index 2 and `15` at index 3).*
2. **Conceptual Question:** Why do we place `front` at the head of a singly linked list and `rear` at the tail, rather than `rear` at the head and `front` at the tail?
   *(Answer: Because deleting the tail node of a singly linked list requires finding its predecessor in $O(n)$ time, whereas deleting the head node (`front = front->next`) and appending after the tail node (`rear->next = new_node`) both run in $O(1)$ time).*
3. **Bug-Hunting Question:** In a linked-list queue, what specific bug occurs if `dequeue()` does not execute `if (q->front == NULL) q->rear = NULL;` when removing the last node?
   *(Answer: `q->rear` becomes a dangling pointer to freed memory; a subsequent `enqueue()` writes to freed heap memory via `q->rear->next = new_node` and leaves `q->front == NULL`).*

### 8.3 Navigation Links

- **Current File:** `01-Queue-Fundamentals.md` (Unit 2 — Topic 1)
- **Next Topic:** [02-Circular-Priority-and-Deque.md](./02-Circular-Priority-and-Deque.md) — Circular Queues, Priority Queues, and Double-Ended Queues (Deques)
- **Applications Case Studies:** [03-Josephus-Problem-and-CPU-Scheduling.md](./03-Josephus-Problem-and-CPU-Scheduling.md) — The Josephus Problem and OS CPU Scheduling Simulations
