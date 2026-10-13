# Unit 2 — Topic 2: Queue Variations — Circular Queue, Priority Queue, and Double-Ended Queue (Deque)

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [Limitations of the Simple FIFO Queue](#11-limitations-of-the-simple-fifo-queue)
   - 1.2 [The Three Essential Queue Variations](#12-the-three-essential-queue-variations)
   - 1.3 [Learning Objectives](#13-learning-objectives)
2. [Definitions, Mathematical Properties, and Terminology](#2-definitions-mathematical-properties-and-terminology)
   - 2.1 [Circular Queue (Ring Buffer) and Modular Arithmetic](#21-circular-queue-ring-buffer-and-modular-arithmetic)
   - 2.2 [Resolving the `front == rear` Ambiguity: All 3 Standard Approaches Compared](#22-resolving-the-front--rear-ambiguity-all-3-standard-approaches-compared)
   - 2.3 [Priority Queue: Policies and FIFO Tie-Breaking Stability](#23-priority-queue-policies-and-fifo-tie-breaking-stability)
   - 2.4 [Double-Ended Queue (Deque) and Restricted Variants](#24-double-ended-queue-deque-and-restricted-variants)
   - 2.5 [Core Terminology Table](#25-core-terminology-table)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Circular Array Queue and Single-Pointer Circular Linked-List Queue](#31-circular-array-queue-and-single-pointer-circular-linked-list-queue)
   - 3.2 [Priority Queue Representations (Array vs. Linked List)](#32-priority-queue-representations-array-vs-linked-list)
   - 3.3 [Deque Representations (Circular Array vs. Doubly Linked List)](#33-deque-representations-circular-array-vs-doubly-linked-list)
4. [Circular Queue: Operations, Algorithms, and C11 Implementation](#4-circular-queue-operations-algorithms-and-c11-implementation)
   - 4.1 [Circular Array `enqueue`, `dequeue`, `size`, and `display` Algorithms](#41-circular-array-enqueue-dequeue-size-and-display-algorithms)
   - 4.2 [Step-by-Step Wraparound ASCII Trace](#42-step-by-step-wraparound-ascii-trace)
   - 4.3 [Circular Linked-List Queue Using Only a Single `rear` Pointer](#43-circular-linked-list-queue-using-only-a-single-rear-pointer)
   - 4.4 [Complete C11 Program: Array Circular Queue & Single-Pointer Circular Linked Queue](#44-complete-c11-program-array-circular-queue--single-pointer-circular-linked-queue)
5. [Priority Queue: Operations, Algorithms, and C11 Implementation](#5-priority-queue-operations-algorithms-and-c11-implementation)
   - 5.1 [Unordered vs. Ordered vs. Binary Heap Implementations](#51-unordered-vs-ordered-vs-binary-heap-implementations)
   - 5.2 [Ordered Linked-List Priority Queue with FIFO Tie-Breaking](#52-ordered-linked-list-priority-queue-with-fifo-tie-breaking)
   - 5.3 [Ordered Array Priority Queue Implementation](#53-ordered-array-priority-queue-implementation)
   - 5.4 [Complete C11 Program: Linked-List & Array Priority Queues](#54-complete-c11-program-linked-list--array-priority-queues)
6. [Double-Ended Queue (Deque): Operations, Algorithms, and C11 Implementation](#6-double-ended-queue-deque-operations-algorithms-and-c11-implementation)
   - 6.1 [Circular Array Deque Algorithms (`insertFront`, `insertRear`, `deleteFront`, `deleteRear`)](#61-circular-array-deque-algorithms-insertfront-insertrear-deletefront-deleterear)
   - 6.2 [Restricted Deques (Input-Restricted & Output-Restricted) and Simulating Stacks/Queues](#62-restricted-deques-input-restricted--output-restricted-and-simulating-stacksqueues)
   - 6.3 [Doubly Linked List Deque Representation and Operations](#63-doubly-linked-list-deque-representation-and-operations)
   - 6.4 [Complete C11 Program: Circular Array Deque & Doubly Linked List Deque](#64-complete-c11-program-circular-array-deque--doubly-linked-list-deque)
7. [Comparison Tables and Complexity Summary](#7-comparison-tables-and-complexity-summary)
   - 7.1 [Priority Queue Implementation Complexity Comparison](#71-priority-queue-implementation-complexity-comparison)
   - 7.2 [Master Comparison Table: Simple Queue vs. Circular Queue vs. Priority Queue vs. Deque](#72-master-comparison-table-simple-queue-vs-circular-queue-vs-priority-queue-vs-deque)
8. [Common Errors, Pitfalls, and Edge Cases](#8-common-errors-pitfalls-and-edge-cases)
   - 8.1 [Pitfall 1: C's Remainder Operator (`%`) on Negative Indices in Deque `insertFront` / `deleteRear`](#81-pitfall-1-cs-remainder-operator--on-negative-indices-in-deque-insertfront--deleterear)
   - 8.2 [Pitfall 2: Breaking FIFO Stability on Equal Priorities (`<` vs. `<=`)](#82-pitfall-2-breaking-fifo-stability-on-equal-priorities--vs-)
   - 8.3 [Pitfall 3: Infinite Loop When Traversing a Circular Array Queue with `i <= rear`](#83-pitfall-3-infinite-loop-when-traversing-a-circular-array-queue-with-i--rear)
   - 8.4 [Pitfall 4: Single-Node Self-Loop Removal in a Circular Linked-List Queue](#84-pitfall-4-single-node-self-loop-removal-in-a-circular-linked-list-queue)
   - 8.5 [Boundary Conditions Checklist](#85-boundary-conditions-checklist)
9. [Summary and Revision Checklist](#9-summary-and-revision-checklist)
   - 9.1 [Key Takeaways and Core Formulas](#91-key-takeaways-and-core-formulas)
   - 9.2 [Self-Test Revision Questions](#92-self-test-revision-questions)
   - 9.3 [Navigation Links](#93-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 Limitations of the Simple FIFO Queue

In [01-Queue-Fundamentals.md](./01-Queue-Fundamentals.md), we saw that a simple linear array queue suffers from a critical structural flaw alongside two functional constraints:
1. **Structural Flaw — False Overflow (Rightward Drift):** When `rear` reaches `CAPACITY - 1`, no new elements can be inserted even if earlier `dequeue` operations have left slots `0 .. front - 1` completely vacant.
2. **Urgency Blindness:** A strict FIFO queue treats all items as equally urgent. In an operating system or hospital emergency room, a critical kernel interrupt or cardiac arrest patient cannot wait behind dozens of low-priority background tasks.
3. **Unidirectional Rigidity:** A standard queue forbids inserting an urgent item at the `front` or cancelling/removing the most recently added item from the `rear` (such as work-stealing schedulers or sliding-window algorithms require).

### 1.2 The Three Essential Queue Variations

To overcome these three limitations, Computer Science introduces three specialized queue designs:

1. **Circular Queue (Ring Buffer):** Connects the last array slot `CAPACITY - 1` logically back to slot `0` using modular arithmetic ($\bmod \text{CAPACITY}$), reclaiming vacated prefix slots in worst-case **$O(1)$** time without ever shifting elements.
2. **Priority Queue:** Associates each element $x$ with a numerical priority $p(x)$ and always dequeues the element with the **highest priority**, breaking ties among equal priorities in **FIFO arrival order**.
3. **Double-Ended Queue (Deque, pronounced *"deck"*):** Generalizes both stacks and queues by permitting **$O(1)$ insertion and deletion at both `front` and `rear`**.

### 1.3 Learning Objectives

After studying this note, you will be able to:
1. Derive and apply the modular arithmetic wraparound formulas for a **Circular Queue** and compare all three standard strategies for resolving the `front == rear` empty/full ambiguity.
2. Implement a **Circular Linked-List Queue** using only a **single `rear` pointer** in $O(1)$ time.
3. Design and implement **Priority Queues** using unordered arrays/lists and ordered arrays/linked lists, preserve **FIFO tie-breaking stability**, and contrast their $O(1)$ / $O(n)$ bounds against the $O(\log n)$ Binary Heap previewed for Unit 3.
4. Implement a **Double-Ended Queue (Deque)** using both a circular array (with backward modular wraparound) and a doubly linked list, explain **Input-Restricted** and **Output-Restricted** Deques, and show how a Deque simulates both a Stack and a FIFO Queue.

---

## 2. Definitions, Mathematical Properties, and Terminology

### 2.1 Circular Queue (Ring Buffer) and Modular Arithmetic

A **Circular Queue** of capacity $N = \text{CAPACITY}$ views the array indices $0, 1, \dots, N-1$ as a closed ring $\mathbb{Z}_N$ where index $0$ immediately follows index $N-1$.

```
                     Logical Ring View (CAPACITY = 6)

                                [0]  [1]
                             100        200
                        [5]                  [2]
                        600                  300
                             500        400
                                [4]  [3]

    Linear Memory View:
    Index:     0      1      2      3      4      5
            +------+------+------+------+------+------+
    Array:  | 100  | 200  | 300  | 400  | 500  | 600  |
            +------+------+------+------+------+------+
                                                  |
              ^-----------------------------------+
                 Wraparound: (5 + 1) % 6 == 0
```

#### Core Modular Arithmetic Formulas ($N = \text{CAPACITY}$)

1. **Forward Index Advance (used by Circular Queue `enqueue` and `dequeue`):**
   $$\text{next}(i) = (i + 1) \bmod N$$
   Specifically, when $i = N - 1$, $\text{next}(N - 1) = (N - 1 + 1) \bmod N = N \bmod N = 0$.

2. **Backward Index Retreat (used by Deque `insertFront` and `deleteRear`):**
   $$\text{prev}(i) = (i - 1 + N) \bmod N$$
   *(Adding $+N$ before applying `% N` is mandatory in C because `(-1) % N` evaluates to `-1`, not `N - 1`!)*

3. **Current Element Count ($k$) When Non-Empty (`front != -1`):**
   $$\text{size}(Q) = \big((\text{rear} - \text{front} + N) \bmod N\big) + 1$$
   - *Verification when not wrapped (`front = 1, rear = 4, N = 6`):*
     $((4 - 1 + 6) \bmod 6) + 1 = (9 \bmod 6) + 1 = 3 + 1 = 4$.
   - *Verification when wrapped around (`front = 4, rear = 1, N = 6`, occupying indices `4, 5, 0, 1`):*
     $((1 - 4 + 6) \bmod 6) + 1 = (3 \bmod 6) + 1 = 3 + 1 = 4$.

### 2.2 Resolving the `front == rear` Ambiguity: All 3 Standard Approaches Compared

In a circular array of capacity $N$, if we advance `rear` on `enqueue` and `front` on `dequeue`, the relation between `front` and `rear` when the queue is **completely full** ($k = N$) is identical to when the queue is **completely empty** ($k = 0$): `rear` sits immediately behind `front` (i.e., `(rear + 1) % N == front`), or if `rear` points to the next free slot, `front == rear` in both states!

There are **three standard techniques** used in textbooks and systems programming to distinguish Empty from Full:

| Criterion | Approach 1: **Reset `front = rear = -1` on Empty** | Approach 2: **Explicit `count` (`size`) Field** | Approach 3: **Sacrifice 1 Array Slot** |
| :--- | :--- | :--- | :--- |
| **Initial State** | `front = -1, rear = -1` | `front = 0, rear = -1, count = 0` | `front = 0, rear = 0` |
| **Meaning of `rear`** | Index of **last occupied** slot | Index of **last occupied** slot | Index of **next vacant** slot |
| **Empty Condition** | `front == -1` | `count == 0` | `front == rear` |
| **Full Condition** | `(rear + 1) % N == front` | `count == N` | `(rear + 1) % N == front` |
| **Usable Capacity** | **$N$ elements** ($100\%$ utilization) | **$N$ elements** ($100\%$ utilization) | **$N - 1$ elements** ($1$ slot always kept empty) |
| **Size Calculation** | `front == -1 ? 0 : (rear - front + N) % N + 1` | Simply return `count` | `(rear - front + N) % N` |
| **Primary Advantage** | Uses all $N$ slots without extra struct fields | Simplest logic for `isEmpty`, `isFull`, and `size` | No shared `count` variable; lock-free SPSC ring buffers |

### 2.3 Priority Queue: Policies and FIFO Tie-Breaking Stability

A **Priority Queue** is an abstract data type storing ordered pairs $(x_i, p_i)$, where $x_i$ is the payload data and $p_i \in \mathbb{Z}$ is its priority key.

1. **Ascending (Min-Priority) Policy:** Smaller numeric key = higher urgency. `dequeueMin()` extracts the element with $\min_i p_i$. *(Used in Dijkstra's algorithm, A\* search, Huffman coding, and UNIX `nice` process priorities where `0` beats `19`).*
2. **Descending (Max-Priority) Policy:** Larger numeric key = higher urgency. `dequeueMax()` extracts the element with $\max_i p_i$. *(Used in hospital triage scales or task managers where priority `10` beats priority `1`).*
3. **FIFO Stability on Equal Priorities:**
   If two elements $x_a$ and $x_b$ have identical priority ($p_a = p_b$) and $x_a$ was enqueued before $x_b$ ($t(x_a) < t(x_b)$), then **$x_a$ must be dequeued before $x_b$**.

### 2.4 Double-Ended Queue (Deque) and Restricted Variants

A **Double-Ended Queue (Deque)** is a homogeneous linear collection supporting $O(1)$ insertion and deletion at **both** the `front` and `rear` ends:

```
                      DOUBLE-ENDED QUEUE (DEQUE)

   insertFront(x) --> +------+------+------+------+ <-- insertRear(y)
                      |  10  |  20  |  30  |  40  |
   deleteFront()  <-- +------+------+------+------+ --> deleteRear()
                         ^                    ^
                       front                rear
```

#### Restricted Deque Variants
1. **Input-Restricted Deque (IRD):**
   - **Insertions** allowed at **one end only** (conventionally `insertRear`).
   - **Deletions** allowed at **both ends** (`deleteFront` and `deleteRear`).
   - *Application:* Job-stealing thread pools where the owner thread pushes/pops from one end and idle worker threads steal from the opposite end.
2. **Output-Restricted Deque (ORD):**
   - **Insertions** allowed at **both ends** (`insertFront` and `insertRear`).
   - **Deletions** allowed at **one end only** (conventionally `deleteFront`).
   - *Application:* Print spoolers or network routers where normal jobs enter at `rear` (`insertRear`), urgent jobs cut to the `front` (`insertFront`), and the printer/NIC always consumes from `front` (`deleteFront`).

### 2.5 Core Terminology Table

| Term | Definition |
| :--- | :--- |
| **Ring Buffer / Circular Queue** | Fixed-size array queue where indices wrap around modulo `CAPACITY` (`(i + 1) % N`). |
| **Min-Priority Queue** | Priority queue where the smallest numeric priority value ($p_{\min}$) has highest urgency. |
| **Max-Priority Queue** | Priority queue where the largest numeric priority value ($p_{\max}$) has highest urgency. |
| **Stable Priority Queue** | Priority queue that resolves equal-priority ties in strict FIFO arrival order. |
| **Deque** | Double-Ended Queue supporting `insertFront`, `insertRear`, `deleteFront`, and `deleteRear` in $O(1)$ time. |
| **Input-Restricted Deque** | Deque restricting insertions to one end while allowing deletions from both ends. |
| **Output-Restricted Deque** | Deque allowing insertions at both ends while restricting deletions to one end. |

---

## 3. Representation and Memory Layout in C

### 3.1 Circular Array Queue and Single-Pointer Circular Linked-List Queue

#### A. Circular Array Queue Struct
```c
#define CQ_CAPACITY 5

typedef struct {
    int data[CQ_CAPACITY];
    int front;   /* -1 when empty; otherwise [0 .. CQ_CAPACITY - 1] */
    int rear;    /* -1 when empty; otherwise [0 .. CQ_CAPACITY - 1] */
    int count;   /* Explicit element count [0 .. CQ_CAPACITY]       */
} CircularQueue;
```

#### B. Circular Linked-List Queue Using Only a Single `rear` Pointer
Why does a **Circular Singly Linked List** need **only one pointer (`rear`)** instead of both `front` and `rear`?
Because the tail node's `next` pointer wraps around to point directly to the head node!
- **`rear`** points to the **newest (tail) node** in $O(1)$.
- **`rear->next`** points to the **oldest (`front` / head) node** in $O(1)$!

```
  CircularLinkedQueue (Single pointer: rear)
  +--------------+
  | rear: 0x3080 |-----------------------------------------+
  +--------------+                                         |
                                                           v
       +---> [Node @ 0x3010]     [Node @ 0x3040]     [Node @ 0x3080]
       |     +------+--------+   +------+--------+   +------+--------+
       |     |  10  | 0x3040 +-->|  20  | 0x3080 +-->|  30  | 0x3010 +---+
       |     +------+--------+   +------+--------+   +------+--------+   |
       |       ^                                       ^                 |
       |     front = rear->next                      rear                |
       +-----------------------------------------------------------------+
```

### 3.2 Priority Queue Representations (Array vs. Linked List)

```c
/* Ordered Singly Linked-List Priority Queue Node */
typedef struct PQNode {
    int data;               /* Payload value                        */
    int priority;           /* Smaller integer = Higher urgency     */
    struct PQNode *next;
} PQNode;

typedef struct {
    PQNode *front;          /* Always points to highest-priority node */
    size_t size;
} LinkedPriorityQueue;
```

```
  LinkedPriorityQueue (Sorted in Ascending Priority Order: Min-Priority at front)
  front
    |
    v
  +-------------+----+     +-------------+----+     +-------------+----+     +-------------+------+
  | data: 500   |    |     | data: 100   |    |     | data: 200   |    |     | data: 300   |      |
  | prio: 1     |  --+---->| prio: 2 (A) |  --+---->| prio: 2 (B) |  --+---->| prio: 5     | NULL |
  +-------------+----+     +-------------+----+     +-------------+----+     +-------------+------+
  (Highest urgency:        (Arrived first with      (Arrived second with     (Lowest urgency)
   dequeued in O(1))        prio 2 -> before B!)     prio 2 -> FIFO stable!)
```

### 3.3 Deque Representations (Circular Array vs. Doubly Linked List)

A **Doubly Linked List** is the natural dynamic representation for a Deque because `deleteRear()` must update `rear = rear->prev` in $O(1)$ time (a singly linked list cannot step backward from `rear` in $O(1)$ time).

```c
typedef struct DequeNode {
    int data;
    struct DequeNode *prev;
    struct DequeNode *next;
} DequeNode;

typedef struct {
    DequeNode *front;
    DequeNode *rear;
    size_t size;
} LinkedDeque;
```

```
  Doubly Linked List Deque Memory Layout:
           front                                                rear
             |                                                    |
             v                                                    v
  NULL <-- [prev | 10 | next] <==> [prev | 20 | next] <==> [prev | 30 | next] --> NULL
```

---

## 4. Circular Queue: Operations, Algorithms, and C11 Implementation

### 4.1 Circular Array `enqueue`, `dequeue`, `size`, and `display` Algorithms

Using **Approach 1** (`front = -1, rear = -1` when empty, full when `(rear + 1) % N == front`):

```text
ALGORITHM CircularEnqueue(Q, value)
1. IF (Q.rear + 1) MOD N == Q.front THEN
2.     PRINT "Circular Queue Overflow"
3.     RETURN false
4. END IF
5. IF Q.front == -1 THEN          /* Queue was empty */
6.     Q.front <- 0
7.     Q.rear  <- 0
8. ELSE
9.     Q.rear <- (Q.rear + 1) MOD N
10. END IF
11. Q.data[Q.rear] <- value
12. RETURN true

ALGORITHM CircularDequeue(Q, out_value)
1. IF Q.front == -1 THEN
2.     PRINT "Circular Queue Underflow"
3.     RETURN false
4. END IF
5. out_value <- Q.data[Q.front]
6. IF Q.front == Q.rear THEN      /* Last remaining element removed */
7.     Q.front <- -1
8.     Q.rear  <- -1
9. ELSE
10.    Q.front <- (Q.front + 1) MOD N
11. END IF
12. RETURN true
```

### 4.2 Step-by-Step Wraparound ASCII Trace

Let `CAPACITY = 4`. Observe how `enqueue(50)` and `enqueue(60)` wrap `rear` around to indices `0` and `1`, solving the False Overflow problem of Topic 1:

```
  1. After enqueue(10), enqueue(20), enqueue(30), enqueue(40):
     Index:   0    1    2    3
            +----+----+----+----+
     Array: | 10 | 20 | 30 | 40 |   front = 0, rear = 3, size = 4
            +----+----+----+----+   isFull? (3 + 1) % 4 == 0 == front -> TRUE!
              ^              ^
              F              R

  2. After dequeue() -> 10, dequeue() -> 20:
            +----+----+----+----+
     Array: | -- | -- | 30 | 40 |   front = 2, rear = 3, size = 2
            +----+----+----+----+   isFull? (3 + 1) % 4 == 0 != 2 -> FALSE!
                        ^    ^
                        F    R

  3. After enqueue(50):  rear = (3 + 1) % 4 = 0  (WRAPAROUND!)
            +----+----+----+----+
     Array: | 50 | -- | 30 | 40 |   front = 2, rear = 0, size = (0 - 2 + 4) % 4 + 1 = 3
            +----+----+----+----+
              ^         ^
              R         F

  4. After enqueue(60):  rear = (0 + 1) % 4 = 1
            +----+----+----+----+
     Array: | 50 | 60 | 30 | 40 |   front = 2, rear = 1, size = (1 - 2 + 4) % 4 + 1 = 4
            +----+----+----+----+   isFull? (1 + 1) % 4 == 2 == front -> TRUE!
                   ^    ^
                   R    F
     Logical FIFO Order from front to rear: 30 (idx 2) -> 40 (idx 3) -> 50 (idx 0) -> 60 (idx 1)
```

### 4.3 Circular Linked-List Queue Using Only a Single `rear` Pointer

- **`enqueueCSLL(Q, value)` ($O(1)$):**
  1. Allocate `new_node` with `data = value`.
  2. If `Q->rear == NULL` (empty): set `new_node->next = new_node` (points to itself!) and `Q->rear = new_node`.
  3. Else (non-empty):
     - `new_node->next = Q->rear->next` (new node points to current `front`).
     - `Q->rear->next = new_node` (old `rear` points to `new_node`).
     - `Q->rear = new_node` (advance `rear` to `new_node`).
- **`dequeueCSLL(Q, &out)` ($O(1)$):**
  1. If `Q->rear == NULL`, return `false` (Underflow).
  2. Let `front_node = Q->rear->next`. Extract `*out = front_node->data`.
  3. **Single-Node Edge Case (`front_node == Q->rear`):** Reset `Q->rear = NULL`.
  4. **Multi-Node Case:** Bypass `front_node` via `Q->rear->next = front_node->next`.
  5. `free(front_node)` and return `true`.

### 4.4 Complete C11 Program: Array Circular Queue & Single-Pointer Circular Linked Queue

```c
/*
 * File: circular_queue_demo.c
 * Standard: C11
 * Description: Complete implementation of (1) Array-Based Circular Queue
 *              with modular wraparound and (2) Circular Singly Linked-List
 *              Queue using a single rear pointer.
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define CQ_CAP 4

/* =========================================================================
 * PART 1: ARRAY-BASED CIRCULAR QUEUE
 * ========================================================================= */

typedef struct {
    int data[CQ_CAP];
    int front;
    int rear;
} CircularQueue;

void cqInit(CircularQueue *q) {
    q->front = -1;
    q->rear  = -1;
}

bool cqIsEmpty(const CircularQueue *q) {
    return (q->front == -1);
}

bool cqIsFull(const CircularQueue *q) {
    return ((q->rear + 1) % CQ_CAP == q->front);
}

int cqSize(const CircularQueue *q) {
    if (cqIsEmpty(q)) return 0;
    return ((q->rear - q->front + CQ_CAP) % CQ_CAP) + 1;
}

bool cqEnqueue(CircularQueue *q, int value) {
    if (cqIsFull(q)) {
        printf("  [CircularQueue] OVERFLOW: Queue is full (size=%d/%d), cannot enqueue %d\n",
               cqSize(q), CQ_CAP, value);
        return false;
    }
    if (cqIsEmpty(q)) {
        q->front = 0;
        q->rear  = 0;
    } else {
        q->rear = (q->rear + 1) % CQ_CAP;
    }
    q->data[q->rear] = value;
    printf("  [CircularQueue] Enqueued %d at idx %d (front=%d, rear=%d, size=%d)\n",
           value, q->rear, q->front, q->rear, cqSize(q));
    return true;
}

bool cqDequeue(CircularQueue *q, int *out_value) {
    if (cqIsEmpty(q)) {
        printf("  [CircularQueue] UNDERFLOW: Queue is empty\n");
        return false;
    }
    int removed_idx = q->front;
    *out_value = q->data[removed_idx];
    if (q->front == q->rear) {
        q->front = -1;
        q->rear  = -1;
    } else {
        q->front = (q->front + 1) % CQ_CAP;
    }
    printf("  [CircularQueue] Dequeued %d from idx %d (front=%d, rear=%d, size=%d)\n",
           *out_value, removed_idx, q->front, q->rear, cqSize(q));
    return true;
}

void cqDisplay(const CircularQueue *q) {
    if (cqIsEmpty(q)) {
        printf("  [CircularQueue] Logical FIFO Order: <EMPTY>\n");
        return;
    }
    int count = cqSize(q);
    printf("  [CircularQueue] Logical FIFO Order (%d items): ", count);
    for (int step = 0; step < count; step++) {
        int idx = (q->front + step) % CQ_CAP;
        printf("%d[idx=%d]%s", q->data[idx], idx, (step < count - 1) ? " -> " : "\n");
    }
}

/* =========================================================================
 * PART 2: SINGLE-POINTER CIRCULAR LINKED-LIST QUEUE
 * ========================================================================= */

typedef struct CSLLNode {
    int data;
    struct CSLLNode *next;
} CSLLNode;

typedef struct {
    CSLLNode *rear;  /* rear->next is ALWAYS front! */
    size_t size;
} CircularLinkedQueue;

void clqInit(CircularLinkedQueue *q) {
    q->rear = NULL;
    q->size = 0;
}

bool clqEnqueue(CircularLinkedQueue *q, int value) {
    CSLLNode *node = (CSLLNode *)malloc(sizeof(CSLLNode));
    if (!node) return false;
    node->data = value;

    if (q->rear == NULL) {
        node->next = node;   /* Self-loop for single node */
        q->rear    = node;
    } else {
        node->next    = q->rear->next; /* Point to front */
        q->rear->next = node;          /* Link old rear to new node */
        q->rear       = node;          /* Advance rear */
    }
    q->size++;
    printf("  [SinglePtr-CLQ] Enqueued %d (front=%d, rear=%d, size=%zu)\n",
           value, q->rear->next->data, q->rear->data, q->size);
    return true;
}

bool clqDequeue(CircularLinkedQueue *q, int *out_value) {
    if (q->rear == NULL) return false;
    CSLLNode *front_node = q->rear->next;
    *out_value = front_node->data;

    if (front_node == q->rear) {
        /* Only 1 node in circular list */
        q->rear = NULL;
    } else {
        q->rear->next = front_node->next;
    }
    free(front_node);
    q->size--;
    printf("  [SinglePtr-CLQ] Dequeued %d (remaining size=%zu)\n", *out_value, q->size);
    return true;
}

void clqFree(CircularLinkedQueue *q) {
    int dummy;
    while (q->rear != NULL) {
        clqDequeue(q, &dummy);
    }
}

int main(void) {
    int val = 0;
    printf("=== 1. ARRAY CIRCULAR QUEUE WRAPAROUND DEMO ===\n");
    CircularQueue cq;
    cqInit(&cq);

    cqEnqueue(&cq, 10);
    cqEnqueue(&cq, 20);
    cqEnqueue(&cq, 30);
    cqEnqueue(&cq, 40);
    cqDisplay(&cq);

    /* Free indices 0 and 1 */
    cqDequeue(&cq, &val);
    cqDequeue(&cq, &val);
    cqDisplay(&cq);

    /* Wrap around into indices 0 and 1! */
    cqEnqueue(&cq, 50);
    cqEnqueue(&cq, 60);
    cqDisplay(&cq);

    /* Now truly full: 4/4 slots occupied */
    cqEnqueue(&cq, 70);

    printf("\n=== 2. SINGLE-POINTER CIRCULAR LINKED QUEUE DEMO ===\n");
    CircularLinkedQueue clq;
    clqInit(&clq);
    clqEnqueue(&clq, 100);
    clqEnqueue(&clq, 200);
    clqEnqueue(&clq, 300);
    clqDequeue(&clq, &val);
    clqDequeue(&clq, &val);
    clqDequeue(&clq, &val);
    return 0;
}
```

#### Program Output:

```text
=== 1. ARRAY CIRCULAR QUEUE WRAPAROUND DEMO ===
  [CircularQueue] Enqueued 10 at idx 0 (front=0, rear=0, size=1)
  [CircularQueue] Enqueued 20 at idx 1 (front=0, rear=1, size=2)
  [CircularQueue] Enqueued 30 at idx 2 (front=0, rear=2, size=3)
  [CircularQueue] Enqueued 40 at idx 3 (front=0, rear=3, size=4)
  [CircularQueue] Logical FIFO Order (4 items): 10[idx=0] -> 20[idx=1] -> 30[idx=2] -> 40[idx=3]
  [CircularQueue] Dequeued 10 from idx 0 (front=1, rear=3, size=3)
  [CircularQueue] Dequeued 20 from idx 1 (front=2, rear=3, size=2)
  [CircularQueue] Logical FIFO Order (2 items): 30[idx=2] -> 40[idx=3]
  [CircularQueue] Enqueued 50 at idx 0 (front=2, rear=0, size=3)
  [CircularQueue] Enqueued 60 at idx 1 (front=2, rear=1, size=4)
  [CircularQueue] Logical FIFO Order (4 items): 30[idx=2] -> 40[idx=3] -> 50[idx=0] -> 60[idx=1]
  [CircularQueue] OVERFLOW: Queue is full (size=4/4), cannot enqueue 70

=== 2. SINGLE-POINTER CIRCULAR LINKED QUEUE DEMO ===
  [SinglePtr-CLQ] Enqueued 100 (front=100, rear=100, size=1)
  [SinglePtr-CLQ] Enqueued 200 (front=100, rear=200, size=2)
  [SinglePtr-CLQ] Enqueued 300 (front=100, rear=300, size=3)
  [SinglePtr-CLQ] Dequeued 100 (remaining size=2)
  [SinglePtr-CLQ] Dequeued 200 (remaining size=1)
  [SinglePtr-CLQ] Dequeued 300 (remaining size=0)
```

---

## 5. Priority Queue: Operations, Algorithms, and C11 Implementation

### 5.1 Unordered vs. Ordered vs. Binary Heap Implementations

A Priority Queue can be implemented using linear structures (Unit 2) or tree-based heaps (Unit 3):

1. **Unordered Array / Unordered Linked List (Lazy Approach):**
   - **`enqueue(x, p)` is $O(1)$:** Simply append the new element at the end without sorting.
   - **`dequeue()` / `peek()` is $O(n)$:** Scan all $n$ elements to find the highest-priority element (and in an array, shift or swap to close the gap).
2. **Ordered Linked List / Ordered Array (Eager Approach):**
   - **`enqueue(x, p)` is $O(n)$:** Traverse or shift elements to insert $(x, p)$ into its exact priority-sorted position.
   - **`dequeue()` and `peek()` are $O(1)$:** The highest-priority element is always positioned directly at `front` (or the active array end).
3. **Binary Heap (Preview of Unit 3):**
   - Uses a complete binary tree stored in an array to achieve **$O(\log n)$** for both `enqueue` and `dequeue`, and **$O(1)$** for `peek`.

### 5.2 Ordered Linked-List Priority Queue with FIFO Tie-Breaking

Assume a **Min-Priority Policy** (smaller priority number = higher urgency, e.g., Priority `1` beats Priority `2`).
- **How to Guarantee FIFO Stability on Ties:**
  When inserting `(value, prio)`, advance the traversal pointer `curr` as long as `curr->next != NULL` **AND `curr->next->priority <= prio`** (using `<=`, **not** `<`!).
  Using `<=` skips past all existing nodes of equal priority so the newly arrived node is placed **after** all earlier nodes with the same priority `prio`.

```text
ALGORITHM PriorityQueueEnqueueLinked(PQ, value, prio)
1. Allocate new_node with data = value, priority = prio, next = NULL
2. IF PQ.front == NULL OR prio < PQ.front.priority THEN
3.     /* Strictly higher urgency than current front: insert at head */
4.     new_node.next <- PQ.front
5.     PQ.front      <- new_node
6. ELSE
7.     curr <- PQ.front
8.     /* Use <= so equal priorities stay in FIFO arrival order! */
9.     WHILE curr.next != NULL AND curr.next.priority <= prio DO
10.        curr <- curr.next
11.    END WHILE
12.    new_node.next <- curr.next
13.    curr.next     <- new_node
14. END IF
15. PQ.size <- PQ.size + 1
```

### 5.3 Ordered Array Priority Queue Implementation

In an **Ordered Array Priority Queue**, if we want **`dequeue()` to be $O(1)$ without shifting**, we sort the array so that the **highest-priority element sits at the highest index `size - 1`** (the `rear` end). Then:
- **`dequeue()` is $O(1)$:** Simply return `items[--size]`!
- **`enqueue(x, p)` is $O(n)$:** Shift higher-urgency elements right by 1 slot until finding the correct slot for `(x, p)`.

### 5.4 Complete C11 Program: Linked-List & Array Priority Queues

```c
/*
 * File: priority_queue_demo.c
 * Standard: C11
 * Description: Complete implementation of (1) Ordered Linked-List Min-Priority
 *              Queue with FIFO stability and (2) Ordered Array Priority Queue.
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

/* =========================================================================
 * PART 1: ORDERED LINKED-LIST MIN-PRIORITY QUEUE (O(n) Enqueue, O(1) Dequeue)
 * ========================================================================= */

typedef struct PQNode {
    const char *task_name;
    int priority;          /* Smaller value = Higher urgency (1 beats 2) */
    struct PQNode *next;
} PQNode;

typedef struct {
    PQNode *front;
    size_t size;
} LinkedPriorityQueue;

void lpqInit(LinkedPriorityQueue *pq) {
    pq->front = NULL;
    pq->size  = 0;
}

bool lpqEnqueue(LinkedPriorityQueue *pq, const char *task_name, int priority) {
    PQNode *new_node = (PQNode *)malloc(sizeof(PQNode));
    if (!new_node) return false;
    new_node->task_name = task_name;
    new_node->priority  = priority;
    new_node->next      = NULL;

    /* Strictly '<' here preserves FIFO order if new priority == front->priority */
    if (pq->front == NULL || priority < pq->front->priority) {
        new_node->next = pq->front;
        pq->front      = new_node;
    } else {
        PQNode *curr = pq->front;
        /* '<=' skips existing nodes with equal priority for FIFO stability */
        while (curr->next != NULL && curr->next->priority <= priority) {
            curr = curr->next;
        }
        new_node->next = curr->next;
        curr->next     = new_node;
    }
    pq->size++;
    printf("  [LinkedPQ] Enqueued (\"%s\", prio=%d)\n", task_name, priority);
    return true;
}

bool lpqDequeue(LinkedPriorityQueue *pq, const char **out_task, int *out_prio) {
    if (pq->front == NULL) return false;
    PQNode *victim = pq->front;
    *out_task      = victim->task_name;
    *out_prio      = victim->priority;
    pq->front      = victim->next;
    free(victim);
    pq->size--;
    return true;
}

void lpqDisplay(const LinkedPriorityQueue *pq) {
    printf("  [LinkedPQ Order] front -> ");
    for (const PQNode *c = pq->front; c != NULL; c = c->next) {
        printf("(\"%s\", p=%d) -> ", c->task_name, c->priority);
    }
    printf("NULL\n");
}

/* =========================================================================
 * PART 2: ORDERED ARRAY MIN-PRIORITY QUEUE (O(n) Enqueue, O(1) Dequeue)
 * ========================================================================= */

#define APQ_CAP 8

typedef struct {
    int data;
    int priority;
} PQItem;

typedef struct {
    PQItem items[APQ_CAP];
    int size;
} ArrayPriorityQueue;

void apqInit(ArrayPriorityQueue *apq) {
    apq->size = 0;
}

/*
 * Keep items sorted in DESCENDING order of priority number (so the SMALLEST
 * priority number = HIGHEST urgency sits at index [size - 1] for O(1) pop!).
 * Equal priorities: newer items are placed to the LEFT of older equal items
 * so older items remain closer to [size - 1] and are dequeued first (FIFO).
 */
bool apqEnqueue(ArrayPriorityQueue *apq, int data, int priority) {
    if (apq->size == APQ_CAP) return false;
    int i = apq->size - 1;
    /* Shift items that have higher urgency (smaller or equal priority number) right */
    while (i >= 0 && apq->items[i].priority <= priority) {
        apq->items[i + 1] = apq->items[i];
        i--;
    }
    apq->items[i + 1].data     = data;
    apq->items[i + 1].priority = priority;
    apq->size++;
    return true;
}

bool apqDequeue(ArrayPriorityQueue *apq, int *out_data, int *out_prio) {
    if (apq->size == 0) return false;
    apq->size--;
    *out_data = apq->items[apq->size].data;
    *out_prio = apq->items[apq->size].priority;
    return true;
}

int main(void) {
    printf("=== 1. ORDERED LINKED-LIST PRIORITY QUEUE (WITH FIFO TIE-BREAKING) ===\n");
    LinkedPriorityQueue lpq;
    lpqInit(&lpq);

    lpqEnqueue(&lpq, "Backup-DB",       3);
    lpqEnqueue(&lpq, "Net-Packet-A",    2);
    lpqEnqueue(&lpq, "Kernel-Panic",    1);
    lpqEnqueue(&lpq, "Net-Packet-B",    2); /* Tie with Net-Packet-A (prio 2) */
    lpqEnqueue(&lpq, "UI-Refresh",      3); /* Tie with Backup-DB (prio 3)    */
    lpqDisplay(&lpq);

    const char *task = NULL;
    int prio = 0;
    printf("\n  Dequeueing in strict Priority + FIFO Order:\n");
    while (lpqDequeue(&lpq, &task, &prio)) {
        printf("  -> Served: %-14s (priority = %d)\n", task, prio);
    }

    printf("\n=== 2. ORDERED ARRAY PRIORITY QUEUE (O(1) DEQUEUE) ===\n");
    ArrayPriorityQueue apq;
    apqInit(&apq);
    apqEnqueue(&apq, 300, 3);
    apqEnqueue(&apq, 201, 2);
    apqEnqueue(&apq, 100, 1);
    apqEnqueue(&apq, 202, 2);

    int d = 0, p = 0;
    while (apqDequeue(&apq, &d, &p)) {
        printf("  -> ArrayPQ Dequeued data=%d (priority=%d)\n", d, p);
    }
    return 0;
}
```

#### Program Output:

```text
=== 1. ORDERED LINKED-LIST PRIORITY QUEUE (WITH FIFO TIE-BREAKING) ===
  [LinkedPQ] Enqueued ("Backup-DB", prio=3)
  [LinkedPQ] Enqueued ("Net-Packet-A", prio=2)
  [LinkedPQ] Enqueued ("Kernel-Panic", prio=1)
  [LinkedPQ] Enqueued ("Net-Packet-B", prio=2)
  [LinkedPQ] Enqueued ("UI-Refresh", prio=3)
  [LinkedPQ Order] front -> ("Kernel-Panic", p=1) -> ("Net-Packet-A", p=2) -> ("Net-Packet-B", p=2) -> ("Backup-DB", p=3) -> ("UI-Refresh", p=3) -> NULL

  Dequeueing in strict Priority + FIFO Order:
  -> Served: Kernel-Panic   (priority = 1)
  -> Served: Net-Packet-A   (priority = 2)
  -> Served: Net-Packet-B   (priority = 2)
  -> Served: Backup-DB      (priority = 3)
  -> Served: UI-Refresh     (priority = 3)

=== 2. ORDERED ARRAY PRIORITY QUEUE (O(1) DEQUEUE) ===
  -> ArrayPQ Dequeued data=100 (priority=1)
  -> ArrayPQ Dequeued data=201 (priority=2)
  -> ArrayPQ Dequeued data=202 (priority=2)
  -> ArrayPQ Dequeued data=300 (priority=3)
```

---

## 6. Double-Ended Queue (Deque): Operations, Algorithms, and C11 Implementation

### 6.1 Circular Array Deque Algorithms (`insertFront`, `insertRear`, `deleteFront`, `deleteRear`)

In a Circular Array Deque of capacity $N$, `front` and `rear` move in **opposite circular directions**:
- **`insertFront(x)`:** Moves `front` **backward** (counter-clockwise):
  $$\text{front} = (\text{front} - 1 + N) \bmod N$$
- **`insertRear(x)`:** Moves `rear` **forward** (clockwise):
  $$\text{rear} = (\text{rear} + 1) \bmod N$$
- **`deleteFront()`:** Moves `front` **forward** (clockwise):
  $$\text{front} = (\text{front} + 1) \bmod N$$
- **`deleteRear()`:** Moves `rear` **backward** (counter-clockwise):
  $$\text{rear} = (\text{rear} - 1 + N) \bmod N$$

```
  Circular Array Deque Index Movement Summary (N = CAPACITY):

               insertFront: front = (front - 1 + N) % N   (Backward)
               deleteFront: front = (front + 1) % N       (Forward)
                                   |
                                   v
            +-------+-------+-------+-------+-------+
            | idx 0 | idx 1 | idx 2 | idx 3 | idx 4 |
            +-------+-------+-------+-------+-------+
                                           ^
                                           |
               insertRear:  rear = (rear + 1) % N         (Forward)
               deleteRear:  rear = (rear - 1 + N) % N     (Backward)
```

### 6.2 Restricted Deques (Input-Restricted & Output-Restricted) and Simulating Stacks/Queues

| Emulated Structure / Variant | Allowed Deque Operations | Disabled Deque Operations |
| :--- | :--- | :--- |
| **LIFO Stack (Front-Based)** | `insertFront(x)` as `push(x)`, `deleteFront()` as `pop()` | `insertRear`, `deleteRear` |
| **LIFO Stack (Rear-Based)** | `insertRear(x)` as `push(x)`, `deleteRear()` as `pop()` | `insertFront`, `deleteFront` |
| **Standard FIFO Queue** | `insertRear(x)` as `enqueue(x)`, `deleteFront()` as `dequeue()` | `insertFront`, `deleteRear` |
| **Input-Restricted Deque** | `insertRear`, `deleteFront`, `deleteRear` | `insertFront` disabled |
| **Output-Restricted Deque** | `insertFront`, `insertRear`, `deleteFront` | `deleteRear` disabled |

### 6.3 Doubly Linked List Deque Representation and Operations

In a Doubly Linked List Deque (`LinkedDeque`):
- **`insertFront(x)`:** Allocates `node`, sets `node->prev = NULL`, `node->next = front`. If empty, `rear = node`; else `front->prev = node`. Updates `front = node`.
- **`insertRear(x)`:** Allocates `node`, sets `node->next = NULL`, `node->prev = rear`. If empty, `front = node`; else `rear->next = node`. Updates `rear = node`.
- **`deleteFront()`:** Extracts `front->data`, advances `front = front->next`. If `front == NULL`, sets `rear = NULL`; else sets `front->prev = NULL`.
- **`deleteRear()`:** Extracts `rear->data`, retreats `rear = rear->prev`. If `rear == NULL`, sets `front = NULL`; else sets `rear->next = NULL`.

### 6.4 Complete C11 Program: Circular Array Deque & Doubly Linked List Deque

```c
/*
 * File: deque_demo.c
 * Standard: C11
 * Description: Complete implementation of (1) Circular Array Deque and
 *              (2) Doubly Linked List Deque supporting O(1) operations at both ends.
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define DQ_CAP 5

/* =========================================================================
 * PART 1: CIRCULAR ARRAY DEQUE
 * ========================================================================= */

typedef struct {
    int data[DQ_CAP];
    int front;
    int rear;
    int count;
} ArrayDeque;

void adqInit(ArrayDeque *dq) {
    dq->front = -1;
    dq->rear  = -1;
    dq->count = 0;
}

bool adqIsEmpty(const ArrayDeque *dq) { return (dq->count == 0); }
bool adqIsFull(const ArrayDeque *dq)  { return (dq->count == DQ_CAP); }

bool adqInsertFront(ArrayDeque *dq, int value) {
    if (adqIsFull(dq)) return false;
    if (adqIsEmpty(dq)) {
        dq->front = 0;
        dq->rear  = 0;
    } else {
        /* Backward modular wraparound: wraps 0 -> DQ_CAP - 1 */
        dq->front = (dq->front - 1 + DQ_CAP) % DQ_CAP;
    }
    dq->data[dq->front] = value;
    dq->count++;
    printf("  [ArrayDeque] insertFront(%d) -> idx=%d (front=%d, rear=%d, count=%d)\n",
           value, dq->front, dq->front, dq->rear, dq->count);
    return true;
}

bool adqInsertRear(ArrayDeque *dq, int value) {
    if (adqIsFull(dq)) return false;
    if (adqIsEmpty(dq)) {
        dq->front = 0;
        dq->rear  = 0;
    } else {
        /* Forward modular wraparound: wraps DQ_CAP - 1 -> 0 */
        dq->rear = (dq->rear + 1) % DQ_CAP;
    }
    dq->data[dq->rear] = value;
    dq->count++;
    printf("  [ArrayDeque] insertRear(%d)  -> idx=%d (front=%d, rear=%d, count=%d)\n",
           value, dq->rear, dq->front, dq->rear, dq->count);
    return true;
}

bool adqDeleteFront(ArrayDeque *dq, int *out_value) {
    if (adqIsEmpty(dq)) return false;
    *out_value = dq->data[dq->front];
    if (dq->count == 1) {
        dq->front = -1;
        dq->rear  = -1;
    } else {
        dq->front = (dq->front + 1) % DQ_CAP;
    }
    dq->count--;
    printf("  [ArrayDeque] deleteFront() -> %d (front=%d, rear=%d, count=%d)\n",
           *out_value, dq->front, dq->rear, dq->count);
    return true;
}

bool adqDeleteRear(ArrayDeque *dq, int *out_value) {
    if (adqIsEmpty(dq)) return false;
    *out_value = dq->data[dq->rear];
    if (dq->count == 1) {
        dq->front = -1;
        dq->rear  = -1;
    } else {
        dq->rear = (dq->rear - 1 + DQ_CAP) % DQ_CAP;
    }
    dq->count--;
    printf("  [ArrayDeque] deleteRear()  -> %d (front=%d, rear=%d, count=%d)\n",
           *out_value, dq->front, dq->rear, dq->count);
    return true;
}

bool adqGetFront(const ArrayDeque *dq, int *out_value) {
    if (adqIsEmpty(dq)) return false;
    *out_value = dq->data[dq->front];
    return true;
}

bool adqGetRear(const ArrayDeque *dq, int *out_value) {
    if (adqIsEmpty(dq)) return false;
    *out_value = dq->data[dq->rear];
    return true;
}

void adqDisplay(const ArrayDeque *dq) {
    printf("  [ArrayDeque] Front-to-Rear: [");
    for (int i = 0; i < dq->count; i++) {
        int idx = (dq->front + i) % DQ_CAP;
        printf("%d(idx=%d)%s", dq->data[idx], idx, (i < dq->count - 1) ? ", " : "");
    }
    printf("]\n");
}

/* =========================================================================
 * PART 2: DOUBLY LINKED LIST DEQUE
 * ========================================================================= */

typedef struct DequeNode {
    int data;
    struct DequeNode *prev;
    struct DequeNode *next;
} DequeNode;

typedef struct {
    DequeNode *front;
    DequeNode *rear;
    size_t size;
} LinkedDeque;

void ldqInit(LinkedDeque *dq) {
    dq->front = NULL;
    dq->rear  = NULL;
    dq->size  = 0;
}

bool ldqInsertFront(LinkedDeque *dq, int value) {
    DequeNode *node = (DequeNode *)malloc(sizeof(DequeNode));
    if (!node) return false;
    node->data = value;
    node->prev = NULL;
    node->next = dq->front;

    if (dq->front == NULL) {
        dq->rear = node;
    } else {
        dq->front->prev = node;
    }
    dq->front = node;
    dq->size++;
    return true;
}

bool ldqInsertRear(LinkedDeque *dq, int value) {
    DequeNode *node = (DequeNode *)malloc(sizeof(DequeNode));
    if (!node) return false;
    node->data = value;
    node->next = NULL;
    node->prev = dq->rear;

    if (dq->rear == NULL) {
        dq->front = node;
    } else {
        dq->rear->next = node;
    }
    dq->rear = node;
    dq->size++;
    return true;
}

bool ldqDeleteFront(LinkedDeque *dq, int *out_value) {
    if (dq->front == NULL) return false;
    DequeNode *victim = dq->front;
    *out_value = victim->data;
    dq->front  = victim->next;
    if (dq->front == NULL) {
        dq->rear = NULL;
    } else {
        dq->front->prev = NULL;
    }
    free(victim);
    dq->size--;
    return true;
}

bool ldqDeleteRear(LinkedDeque *dq, int *out_value) {
    if (dq->rear == NULL) return false;
    DequeNode *victim = dq->rear;
    *out_value = victim->data;
    dq->rear   = victim->prev;
    if (dq->rear == NULL) {
        dq->front = NULL;
    } else {
        dq->rear->next = NULL;
    }
    free(victim);
    dq->size--;
    return true;
}

int main(void) {
    int val = 0, f = 0, r = 0;
    printf("=== 1. CIRCULAR ARRAY DEQUE DEMO ===\n");
    ArrayDeque adq;
    adqInit(&adq);

    adqInsertRear(&adq, 20);
    adqInsertRear(&adq, 30);
    adqInsertFront(&adq, 10);  /* Wraps front backward from 0 to 4! */
    adqInsertFront(&adq, 5);   /* Moves front backward from 4 to 3! */
    adqDisplay(&adq);

    adqGetFront(&adq, &f);
    adqGetRear(&adq, &r);
    printf("  [ArrayDeque] getFront() = %d, getRear() = %d\n", f, r);

    adqDeleteFront(&adq, &val);
    adqDeleteRear(&adq, &val);
    adqDisplay(&adq);

    printf("\n=== 2. DOUBLY LINKED LIST DEQUE DEMO ===\n");
    LinkedDeque ldq;
    ldqInit(&ldq);
    ldqInsertRear(&ldq, 100);
    ldqInsertFront(&ldq, 50);
    ldqInsertRear(&ldq, 150);
    ldqDeleteFront(&ldq, &f);
    ldqDeleteRear(&ldq, &r);
    printf("  [LinkedDeque] deleteFront -> %d, deleteRear -> %d, remaining front -> %d\n",
           f, r, ldq.front->data);
    ldqDeleteFront(&ldq, &val);
    return 0;
}
```

#### Program Output:

```text
=== 1. CIRCULAR ARRAY DEQUE DEMO ===
  [ArrayDeque] insertRear(20)  -> idx=0 (front=0, rear=0, count=1)
  [ArrayDeque] insertRear(30)  -> idx=1 (front=0, rear=1, count=2)
  [ArrayDeque] insertFront(10) -> idx=4 (front=4, rear=1, count=3)
  [ArrayDeque] insertFront(5)  -> idx=3 (front=3, rear=1, count=4)
  [ArrayDeque] Front-to-Rear: [5(idx=3), 10(idx=4), 20(idx=0), 30(idx=1)]
  [ArrayDeque] getFront() = 5, getRear() = 30
  [ArrayDeque] deleteFront() -> 5 (front=4, rear=1, count=3)
  [ArrayDeque] deleteRear()  -> 30 (front=4, rear=0, count=2)
  [ArrayDeque] Front-to-Rear: [10(idx=4), 20(idx=0)]

=== 2. DOUBLY LINKED LIST DEQUE DEMO ===
  [LinkedDeque] deleteFront -> 50, deleteRear -> 150, remaining front -> 100
```

---

## 7. Comparison Tables and Complexity Summary

### 7.1 Priority Queue Implementation Complexity Comparison

Let $n$ be the number of elements currently in the Priority Queue.

| Representation | `enqueue(x, p)` (Insert) | `dequeue()` (Extract Highest Priority) | `peek()` (Inspect Highest Priority) | Space Overhead | Best Use Case |
| :--- | :---: | :---: | :---: | :---: | :--- |
| **Unordered Array** | $O(1)$ | $O(n)$ (search + shift/swap) | $O(n)$ | $O(N)$ static | Very high write rate, rare reads |
| **Unordered Linked List** | $O(1)$ | $O(n)$ (scan for min/max) | $O(n)$ | $O(n)$ pointers | Dynamic stream with rare extractions |
| **Ordered Array** | $O(n)$ (search + shift) | $O(1)$ (if best at `size-1`) | $O(1)$ | $O(N)$ static | Small fixed-capacity priority tables |
| **Ordered Linked List** | $O(n)$ (traverse to spot) | $O(1)$ (remove `front`) | $O(1)$ | $O(n)$ pointers | Moderate $n$, frequent `peek`/`dequeue` |
| **Binary Heap (Unit 3)** | **$O(\log n)$** | **$O(\log n)$** | **$O(1)$** | $O(n)$ array | **General-purpose large-scale workloads** |

### 7.2 Master Comparison Table: Simple Queue vs. Circular Queue vs. Priority Queue vs. Deque

| Dimension | Simple (Linear) Queue | Circular Queue | Priority Queue (Ordered List) | Double-Ended Queue (Deque) |
| :--- | :--- | :--- | :--- | :--- |
| **Service Order** | Strict FIFO | Strict FIFO | Priority Order (ties in FIFO) | User-controlled (FIFO or LIFO) |
| **Insertion Point(s)** | `rear` only | `rear` only (wraps modulo $N$) | Priority-sorted position | Both `front` and `rear` |
| **Deletion Point(s)** | `front` only | `front` only (wraps modulo $N$) | `front` (highest priority) | Both `front` and `rear` |
| **Insert Complexity** | $O(1)$ | $O(1)$ | $O(n)$ (or $O(\log n)$ heap) | $O(1)$ at both ends |
| **Delete Complexity** | $O(1)$ | $O(1)$ | $O(1)$ (or $O(\log n)$ heap) | $O(1)$ at both ends |
| **False Overflow?** | **Yes** | **No** ($100\%$ reuse) | **No** | **No** ($100\%$ reuse) |
| **Typical OS / CS Use** | Simple batch pipe | UART/Network ring, RR scheduler | CPU Priority Scheduler, Dijkstra | Work-stealing scheduler, Sliding window |

---

## 8. Common Errors, Pitfalls, and Edge Cases

### 8.1 Pitfall 1: C's Remainder Operator (`%`) on Negative Indices in Deque `insertFront` / `deleteRear`

In C11, the `%` operator is a **remainder** operator (sign matches dividend), **not** a mathematical Euclidean modulo! Thus, when `front == 0` and `CAPACITY == 5`, evaluating `(0 - 1) % 5` produces **`-1`**, causing an out-of-bounds write `data[-1]`!

```c
/* BUG: Evaluates to -1 when dq->front == 0! */
dq->front = (dq->front - 1) % DQ_CAP;

/* FIX: Always add + DQ_CAP before applying % DQ_CAP */
dq->front = (dq->front - 1 + DQ_CAP) % DQ_CAP;
```

### 8.2 Pitfall 2: Breaking FIFO Stability on Equal Priorities (`<` vs. `<=`)

```c
/* BUG: Using '<' stops BEFORE existing nodes of equal priority,
 * inserting the newer node AHEAD of older nodes with the same priority (LIFO on ties)! */
while (curr->next != NULL && curr->next->priority < priority) {
    curr = curr->next;
}

/* FIX: Use '<=' so traversal moves PAST all existing nodes of equal priority */
while (curr->next != NULL && curr->next->priority <= priority) {
    curr = curr->next;
}
```

### 8.3 Pitfall 3: Infinite Loop When Traversing a Circular Array Queue with `i <= rear`

When a Circular Queue has wrapped around (`front = 3, rear = 1`), writing a standard linear loop `for (int i = front; i <= rear; i++)` immediately terminates ($3 \le 1$ is false!), or if written `for (int i = front; i != rear; i = (i + 1) % N)` misses the `rear` element!

```c
/* BUG: Fails whenever rear < front (wrapped state) */
for (int i = q->front; i <= q->rear; i++) {
    printf("%d ", q->data[i]);
}

/* FIX: Iterate over the exact element count using modular indexing */
int count = cqSize(q);
for (int step = 0; step < count; step++) {
    int idx = (q->front + step) % CQ_CAP;
    printf("%d ", q->data[idx]);
}
```

### 8.4 Pitfall 4: Single-Node Self-Loop Removal in a Circular Linked-List Queue

In a single-pointer Circular Linked Queue (`rear`), when only $1$ node remains, `rear->next == rear`. If `dequeue` blindly executes `rear->next = front_node->next; free(front_node);` without checking `front_node == rear`, `rear` continues pointing to the freed node!

```c
/* BUG: Does not reset q->rear to NULL when the single self-looping node is removed */
CSLLNode *front_node = q->rear->next;
q->rear->next = front_node->next;
free(front_node); /* q->rear is now a dangling pointer! */

/* FIX: Check if front_node == q->rear first */
CSLLNode *front_node = q->rear->next;
if (front_node == q->rear) {
    q->rear = NULL;
} else {
    q->rear->next = front_node->next;
}
free(front_node);
```

### 8.5 Boundary Conditions Checklist

- [ ] **Circular Wraparound (`rear == CAPACITY - 1` $\to$ `0`):** Does `(rear + 1) % CAPACITY` properly wrap without false overflow when `front > 0`?
- [ ] **Backward Wraparound (`front == 0` $\to$ `CAPACITY - 1`):** Does Deque `insertFront` and `deleteRear` use `(idx - 1 + CAPACITY) % CAPACITY`?
- [ ] **Single-Element Deque Deletion (`count == 1`):** Do both `deleteFront` and `deleteRear` reset `front = rear = -1` when removing the last element?
- [ ] **Priority Queue Head Insertion:** Does `lpqEnqueue` handle inserting a node with strictly higher urgency than `pq->front` (`priority < pq->front->priority`)?

---

## 9. Summary and Revision Checklist

### 9.1 Key Takeaways and Core Formulas

1. **Circular Queue Wraparound:**
   - Advance: $i \gets (i + 1) \bmod N$
   - Retreat (Deque): $i \gets (i - 1 + N) \bmod N$
   - Size (non-empty): $\text{size} = ((\text{rear} - \text{front} + N) \bmod N) + 1$
2. **`front == rear` Disambiguation:** Use either (a) `front = rear = -1` on empty, (b) an explicit `count` field, or (c) sacrifice 1 slot (`(rear + 1) % N == front` for Full, `front == rear` for Empty).
3. **Priority Queue Trade-Off:** Unordered list gives $O(1)$ insert / $O(n)$ delete; Ordered list gives $O(n)$ insert / $O(1)$ delete; Binary Heap (Unit 3) gives $O(\log n)$ for both.
4. **Deque Versatility:** A Deque supports $O(1)$ insertion and deletion at both `front` and `rear`, subsuming both LIFO stacks and FIFO queues.

### 9.2 Self-Test Revision Questions

1. **Calculation Question:** In a Circular Queue with `CAPACITY = 8`, `front = 6`, and `rear = 2`, how many elements are currently in the queue, and which array indices are occupied?
   *(Answer: $\text{size} = ((2 - 6 + 8) \bmod 8) + 1 = 4 + 1 = 5$ elements, occupying indices `6, 7, 0, 1, 2`).*
2. **Design Question:** Why can a Circular Singly Linked List implement a $O(1)$ FIFO queue with only a `rear` pointer, whereas a non-circular Singly Linked List with only a `rear` pointer cannot?
   *(Answer: In a circular singly linked list, `rear->next` points directly to `front` in $O(1)$ time; in a non-circular singly linked list, `rear->next` is `NULL`, so `front` is unreachable from `rear`).*
3. **Code Trace Question:** In an Array Deque of `CAPACITY = 6` initialized with `front = -1, rear = -1`, trace `front` and `rear` after: `insertFront(10)`, `insertFront(20)`, `insertRear(30)`, `deleteRear()`.
   *(Answer: `insertFront(10)` $\to (0, 0)$; `insertFront(20)` $\to (5, 0)$; `insertRear(30)` $\to (5, 1)$; `deleteRear()` $\to (5, 0)$).*

### 9.3 Navigation Links

- **Previous Topic:** [01-Queue-Fundamentals.md](./01-Queue-Fundamentals.md) — Queue ADT, Simple Array Queue, and Linked-List Queue
- **Current File:** `02-Circular-Priority-and-Deque.md` (Unit 2 — Topic 2)
- **Next Topic:** [03-Josephus-Problem-and-CPU-Scheduling.md](./03-Josephus-Problem-and-CPU-Scheduling.md) — The Josephus Problem and OS CPU Scheduling Simulations
