# 07 — Stacks: Array and Linked-List Implementations

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [What Is a Stack?](#11-what-is-a-stack)
   - 1.2 [Why Stacks Are Needed and Problems They Solve](#12-why-stacks-are-needed-and-problems-they-solve)
   - 1.3 [Relationship to Prior Concepts](#13-relationship-to-prior-concepts)
   - 1.4 [Learning Objectives](#14-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Formal ADT Specification and the LIFO Principle](#21-formal-adt-specification-and-the-lifo-principle)
   - 2.2 [Intuitive Real-World Analogies](#22-intuitive-real-world-analogies)
   - 2.3 [Terminology Reference Table](#23-terminology-reference-table)
   - 2.4 [Mathematical Properties and Structural Invariants](#24-mathematical-properties-and-structural-invariants)
   - 2.5 [Conceptual ASCII Diagrams](#25-conceptual-ascii-diagrams)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Static Fixed-Capacity Array Stack](#31-static-fixed-capacity-array-stack)
   - 3.2 [Dynamically Allocated Resizable Array Stack](#32-dynamically-allocated-resizable-array-stack)
   - 3.3 [Linked-List-Based Stack (Why Top Must Be the Head)](#33-linked-list-based-stack-why-top-must-be-the-head)
   - 3.4 [Static vs. Dynamic Storage Tradeoffs](#34-static-vs-dynamic-storage-tradeoffs)
4. [Operations and Algorithms](#4-operations-and-algorithms)
   - 4.1 [Array-Based Stack Operations (`push`, `pop`, `peek`, `isEmpty`, `isFull`, `size`, `display`)](#41-array-based-stack-operations-push-pop-peek-isempty-isfull-size-display)
   - 4.2 [Complete C11 Implementation: Static and Dynamic Array Stacks](#42-complete-c11-implementation-static-and-dynamic-array-stacks)
   - 4.3 [Linked-List-Based Stack Operations (`push`, `pop`, `peek`, `isEmpty`, `display`, `free_stack`)](#43-linked-list-based-stack-operations-push-pop-peek-isempty-display-free_stack)
   - 4.4 [Complete C11 Implementation: Linked-List Stack](#44-complete-c11-implementation-linked-list-stack)
5. [Comparison Tables & Complexity Summary](#5-comparison-tables--complexity-summary)
   - 5.1 [Architectural Comparison: Fixed Array vs. Dynamic Array vs. Linked-List Stack](#51-architectural-comparison-fixed-array-vs-dynamic-array-vs-linked-list-stack)
   - 5.2 [Time and Space Complexity Summary Table](#52-time-and-space-complexity-summary-table)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Frequent C Implementation Bugs and Fixes](#61-frequent-c-implementation-bugs-and-fixes)
   - 6.2 [Boundary Conditions Checklist](#62-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)
   - 7.1 [Key Takeaways and Core Invariants](#71-key-takeaways-and-core-invariants)
   - 7.2 [Self-Test Revision Checklist](#72-self-test-revision-checklist)
   - 7.3 [Navigation Links](#73-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 What Is a Stack?

A **Stack** is a fundamental linear **Abstract Data Type (ADT)** in which all insertions and deletions are restricted to a single designated end called the **top**. Because the most recently inserted element is always the first one to be removed, a stack operates strictly under the **LIFO (Last-In, First-Out)**—or equivalently **FILO (First-In, Last-Out)**—discipline.

Unlike general arrays or linked lists that allow reading, inserting, or deleting elements at arbitrary indices or positions, a stack intentionally **restricts** its interface. Clients can only:
1. **Push** a new item onto the top of the stack.
2. **Pop** (remove and return) the item currently at the top of the stack.
3. **Peek** (inspect without removing) the item currently at the top of the stack.

### 1.2 Why Stacks Are Needed and Problems They Solve

In computer science, restricting access to the most recent item models any process that exhibits **nested structure**, **temporal reversal**, or **hierarchical backtracking**:

- **Subroutine Execution and Recursion**: Hardware processors and language runtimes maintain a *system call stack* of activation records (stack frames) so that nested function calls return in the exact reverse order of their invocation.
- **Syntax Validation and Compiler Parsing**: Checking balanced delimiters (`()`, `{}`, `[]`) in source code requires matching the most recently opened delimiter with the next closing delimiter encountered.
- **Expression Conversion and Evaluation**: Translating arithmetic expressions from human-readable infix notation (`A + B * C`) into parenthesis-free postfix (`A B C * +`) or prefix (`+ A * B C`) notations—and evaluating them—relies directly on operator and operand stacks.
- **Undo/Redo Subsystems and Backtracking**: Text editors, browser navigation histories ("Back" button), and depth-first search (DFS) maze/graph explorers push states onto a stack so they can unwind to the most recent decision point in $O(1)$ time.

### 1.3 Relationship to Prior Concepts

In earlier sections of Unit 1, we studied **arrays** and **linked lists** ([02-Singly-Linked-Lists.md](02-Singly-Linked-Lists.md), [03-Doubly-Linked-Lists.md](03-Doubly-Linked-Lists.md)). A stack is not a brand-new physical memory layout; rather, it is an **abstract behavioral contract (ADT)** implemented *on top of* either:
1. A **contiguous array** (static fixed-size or dynamically resized via `realloc`), where an integer index `top` tracks the last occupied slot.
2. A **singly linked list**, where the `head` pointer serves as the `top` of the stack so that both insertion (`push`) and removal (`pop`) execute in worst-case $O(1)$ time.

### 1.4 Learning Objectives

After completing this chapter, you will be able to:
- Define the Stack ADT formally using algebraic axioms and structural invariants.
- Implement a **Fixed-Capacity Static Array Stack** and a **Resizable Dynamic Array Stack** in C11 with proper overflow and underflow protection.
- Prove why a **Linked-List Stack** must place the stack top at the **head** of the singly linked list ($O(1)$ push and pop) rather than the tail ($O(n)$ pop).
- Analyze worst-case vs. amortized time complexity when doubling/halving a dynamic array stack.
- Compare Fixed Array, Dynamic Array, and Linked-List stacks across time complexity, per-element memory overhead, CPU cache locality, and failure modes.

---

## 2. Definitions and Terminology

### 2.1 Formal ADT Specification and the LIFO Principle

Formally, a **Stack** $S$ of elements drawn from a type universe $T$ is a finite ordered sequence:

$$S = (a_0, a_1, a_2, \dots, a_{n-1})$$

where:
- $n = |S| \ge 0$ is the current **size** (number of elements) of the stack.
- When $n = 0$, $S = ()$ is the **empty stack**.
- When $n > 0$, $a_0$ is the **bottom** element (the earliest element inserted still remaining in $S$) and $a_{n-1}$ is the **top** element (the most recently inserted element).

#### Algebraic Axioms of the Stack ADT
For any stack $S$ and element $x \in T$:
1. $\text{isEmpty}(\text{create}()) = \text{true}$
2. $\text{isEmpty}(\text{push}(S, x)) = \text{false}$
3. $\text{size}(\text{create}()) = 0$
4. $\text{size}(\text{push}(S, x)) = \text{size}(S) + 1$
5. $\text{peek}(\text{push}(S, x)) = x$
6. $\text{pop}(\text{push}(S, x)) = (S, x)$ *(restores stack state $S$ and yields $x$)*

Property (6) mathematically captures the **LIFO (Last-In, First-Out)** principle: if a sequence of elements $x_1, x_2, \dots, x_k$ is pushed onto $S$ without intervening pops, then $k$ consecutive `pop` operations yield $x_k, x_{k-1}, \dots, x_1$ in exact reverse order.

### 2.2 Intuitive Real-World Analogies

1. **Spring-Loaded Cafeteria Plate Dispenser**: Clean plates are pushed down into a spring-loaded cylinder. Only the topmost plate is visible and accessible. Adding a new plate pushes all existing plates down; taking a plate removes the top one and allows the plate immediately beneath it to rise to the top.
2. **Stack of Shipping Crates in a Narrow Alley**: If crates `A`, `B`, and `C` are stacked vertically, you cannot pull crate `A` from the bottom without first removing `C` and then `B`.

### 2.3 Terminology Reference Table

| Term | Definition | Array Stack Equivalent | Linked-List Stack Equivalent |
| :--- | :--- | :--- | :--- |
| **Top (`top`)** | The active end of the stack where all insertions and removals take place. | Integer index `s->top` (`-1` when empty). | Pointer `s->top` to the head node (`NULL` when empty). |
| **Bottom** | The oldest element in the stack; the last element to be popped. | Element at index `0` (`s->data[0]`). | The tail node whose `next` pointer is `NULL`. |
| **`push(S, x)`** | Inserts element `x` onto the top of stack `S`. | `s->data[++(s->top)] = x` | Allocate node, set `node->next = s->top`, `s->top = node`. |
| **`pop(S)`** | Removes and returns the element at the top of stack `S`. | Return `s->data[(s->top)--]` | Save `s->top`, advance `s->top = s->top->next`, `free` old top. |
| **`peek(S)` / `top(S)`** | Returns the top element of `S` without modifying the stack. | Return `s->data[s->top]` | Return `s->top->data` |
| **`isEmpty(S)`** | Returns `true` if stack `S` contains zero elements. | `s->top == -1` | `s->top == NULL` |
| **`isFull(S)`** | Returns `true` if stack `S` has reached its maximum capacity. | `s->top == capacity - 1` | Only true if heap memory is exhausted (`malloc == NULL`). |
| **Stack Overflow** | Error condition when attempting to `push` onto a full stack. | `s->top == MAX - 1` (or `realloc` fails). | Heap exhaustion (`malloc` returns `NULL`). |
| **Stack Underflow** | Error condition when attempting to `pop` or `peek` on an empty stack. | `s->top == -1` | `s->top == NULL` |

### 2.4 Mathematical Properties and Structural Invariants

#### Array-Based Stack Invariants (0-Indexed C Arrays)
Let `capacity` $= C \ge 1$ be the allocated array length and `top` be the index of the current top element:
- **Range Invariant**: $-1 \le \text{top} \le C - 1$.
- **Size Identity**: The number of elements currently in the stack is always $n = \text{top} + 1$.
- **Empty State**: $n = 0 \iff \text{top} == -1$.
- **Full State**: $n = C \iff \text{top} == C - 1$.
- **Valid Elements**: Only array indices $0, 1, \dots, \text{top}$ contain live stack elements. Indices $\text{top} + 1, \dots, C - 1$ contain uninitialized or stale ("garbage") values that are logically outside the stack.

#### Linked-List Stack Invariants
Let `s->top` be a pointer to the first `StackNode` and `s->size` be the stored element counter:
- **Empty State**: `s->top == NULL` $\iff$ `s->size == 0`.
- **Non-Empty State**: `s->top != NULL` $\iff$ `s->size >= 1`, and `s->top->data` is the most recently pushed value.
- **Acyclicity & Length**: Following `next` pointers starting from `s->top` visits exactly `s->size` distinct nodes before terminating at `NULL`.

### 2.5 Conceptual ASCII Diagrams

```
  LIFO Discipline Visualization:
  ==============================

   1. Initial      2. push(10)     3. push(20)     4. push(30)     5. pop() -> 30
      Empty           top = 0         top = 1         top = 2         top = 1

     |      |        |      |        |      |        |  30  | <--top |      |
     |      |        |      |        |  20  | <--top |  20  |        |  20  | <--top
     |      |        |  10  | <--top |  10  |        |  10  |        |  10  |
     +------+        +------+        +------+        +------+        +------+
     top = -1
```

---

## 3. Representation and Memory Layout in C

### 3.1 Static Fixed-Capacity Array Stack

In a static array stack, the maximum capacity `MAX_CAPACITY` is fixed at compile time. The array lives directly inside the `struct` (either on the caller's stack frame or in static memory), requiring zero heap allocations (`malloc`/`free`).

```c
#define MAX_CAPACITY 100

typedef struct {
    int data[MAX_CAPACITY]; /* Contiguous fixed-size storage buffer */
    int top;                /* Index of top element; -1 when empty  */
} StaticStack;
```

#### Memory Layout Diagram (Static Array Stack with `MAX_CAPACITY = 6`, after pushing `10, 20, 30`)

```
  StaticStack struct in contiguous memory (e.g., base address 0x7FFC00):
  +------------+------------+------------+------------+------------+------------+------------+
  |  data[0]   |  data[1]   |  data[2]   |  data[3]   |  data[4]   |  data[5]   |    top     |
  |     10     |     20     |     30     |   (junk)   |   (junk)   |   (junk)   |     2      |
  +------------+------------+------------+------------+------------+------------+------------+
    0x7FFC00     0x7FFC04     0x7FFC08     0x7FFC0C     0x7FFC10     0x7FFC14     0x7FFC18
       ^                         ^            ^                                 
     BOTTOM                     TOP      Next push writes here (index top + 1 = 3)
  |<------- Active Stack (size=3) ------>|<-------- Unused Capacity ----------->|
```

### 3.2 Dynamically Allocated Resizable Array Stack

When the maximum number of elements is unknown at compile time, we store a pointer `int *data` to a heap-allocated array alongside `top` and the current `capacity`. When `top == capacity - 1`, we double `capacity` using `realloc`.

```c
typedef struct {
    int *data;      /* Pointer to heap-allocated array of length 'capacity' */
    int top;        /* Index of current top element (-1 when empty)         */
    int capacity;   /* Current allocated capacity in number of elements     */
} DynamicStack;
```

#### Memory Layout Diagram (Dynamic Array Stack)

```
  Stack Frame (Automatic Storage)          Heap Memory (Contiguous Buffer via malloc/realloc)
  +------------------------------+         +---------+---------+---------+---------+
  | DynamicStack ds              |         | data[0] | data[1] | data[2] | data[3] |
  |   data     = 0x55A01000 -----+-------->|   100   |   200   |   300   | (unused)|
  |   top      = 2               |         +---------+---------+---------+---------+
  |   capacity = 4               |          0x55A01000          0x55A01008
  +------------------------------+             ^                   ^
                                             BOTTOM               TOP (index 2)
```

### 3.3 Linked-List-Based Stack (Why Top Must Be the Head)

In a linked-list stack, each element is stored in an independently allocated heap node containing a payload (`data`) and a pointer (`next`) to the element immediately below it in the stack.

```c
typedef struct StackNode {
    int data;
    struct StackNode *next; /* Points to the element BELOW this node */
} StackNode;

typedef struct {
    StackNode *top;         /* Points to the HEAD node of the singly linked list */
    size_t size;            /* Tracks current number of nodes in O(1)            */
} LinkedStack;
```

#### Crucial Design Question: Why MUST Top-of-Stack Be the Head (and Not the Tail) of a Singly Linked List?

Suppose a singly linked list has $n$ nodes: $\text{head} \to N_1 \to N_2 \to \dots \to N_{n-1} \to \text{tail} \to \text{NULL}$.

1. **Option A — Top at the Head (`s->top == head`) [CORRECT]**:
   - **`push(x)` (Insert at Head)**: Allocate `new_node`, set `new_node->next = s->top`, and update `s->top = new_node`. Takes **$O(1)$ worst-case time**.
   - **`pop()` (Delete from Head)**: Save `victim = s->top`, advance `s->top = s->top->next`, and `free(victim)`. Takes **$O(1)$ worst-case time**.

2. **Option B — Top at the Tail (`s->top == tail`) [WRONG FOR SINGLY LINKED LIST]**:
   - **`push(x)` (Append at Tail)**: Even if we maintain a `tail` pointer, appending takes $O(1)$ time.
   - **`pop()` (Delete from Tail)**: To remove the `tail` node and update `tail` to point to the predecessor node $N_{n-1}$ (and set $N_{n-1}\text{->next} = \text{NULL}$), we need a pointer to $N_{n-1}$. Because a singly linked list has no backward (`prev`) pointers, finding $N_{n-1}$ requires traversing the entire list from `head`! Thus, `pop()` degrades from $O(1)$ to **$O(n)$ time**.

```
  Option A: Top at HEAD of Singly Linked List (Both push & pop are O(1))
  ======================================================================
  s->top
    |
    v
  +----+------+    +----+------+    +----+------+
  | 60 | next-+--->| 45 | next-+--->| 30 | NULL |   (Bottom of stack)
  +----+------+    +----+------+    +----+------+
  ^
  Push prepends here in O(1); Pop removes here in O(1).


  Option B: Top at TAIL of Singly Linked List (Pop degrades to O(n)!)
  ===================================================================
  head                                s->top (tail)
    |                                   |
    v                                   v
  +----+------+    +----+------+    +----+------+
  | 30 | next-+--->| 45 | next-+--->| 60 | NULL |
  +----+------+    +----+------+    +----+------+
                     ^
                     To pop 60, we must update this node's 'next' to NULL
                     and move s->top here. Without backward links, finding
                     node 45 requires an O(n) scan from head!
```

### 3.4 Static vs. Dynamic Storage Tradeoffs

| Storage Strategy | Pros | Cons |
| :--- | :--- | :--- |
| **Static Fixed Array** (`int data[MAX]`) | Zero heap overhead; fastest possible $O(1)$ access; excellent CPU cache locality; no memory leaks possible. | Fixed upper bound `MAX`; wastes memory if largely empty; suffers **Stack Overflow** if elements exceed `MAX`. |
| **Dynamic Resizable Array** (`int *data` + `realloc`) | Grows on demand; retains contiguous memory and high cache locality; zero per-element pointer overhead; $O(1)$ amortized `push`. | Occasional $O(n)$ resize spikes during `realloc`; may waste up to $50\%$–$75\%$ slack capacity; must call `free()`. |
| **Singly Linked List** (`StackNode *top`) | Strict worst-case $O(1)$ `push` and `pop` (no resize spikes); capacity grows smoothly node-by-node until heap is full; zero slack capacity. | High per-element memory overhead (8-byte pointer + 8–16 byte allocator header per node on 64-bit systems); poor cache locality (pointer chasing); `malloc`/`free` call on every `push`/`pop`. |

---

## 4. Operations and Algorithms

### 4.1 Array-Based Stack Operations (`push`, `pop`, `peek`, `isEmpty`, `isFull`, `size`, `display`)

#### 1. `push(S, value)` — Array Stack
- **Purpose**: Insert `value` as the new top element of stack `S`.
- **Preconditions**:
  - For a Fixed Array Stack: Stack must not be full (`s->top < MAX - 1`).
  - For a Dynamic Array Stack: If `s->top == s->capacity - 1`, `realloc` must succeed in doubling `s->capacity`.
- **Step-by-Step Algorithm**:
  1. Check if `s->top == s->capacity - 1`.
  2. If fixed-capacity, report **Stack Overflow** and return `false`. If dynamic, invoke `resize(s, s->capacity * 2)`; if `realloc` returns `NULL`, report **Heap Overflow** and return `false`.
  3. Increment `s->top` by 1 (`++(s->top)`).
  4. Store `value` at `s->data[s->top]`.
  5. Return `true`.
- **Pseudocode**:
  ```text
  Algorithm ArrayStack_Push(S, value):
      if S.top == S.capacity - 1 then
          if S is StaticStack then
              error "Stack Overflow"
              return false
          else if not Resize(S, S.capacity * 2) then
              error "Heap Overflow"
              return false
      S.top <- S.top + 1
      S.data[S.top] <- value
      return true
  ```
- **ASCII State Transition (`push(30)` onto `[10, 20]`)**:
  ```
  BEFORE push(30):                         AFTER push(30):
  index:   0    1    2    3                index:   0    1    2    3
         +----+----+----+----+                    +----+----+----+----+
  data:  | 10 | 20 | -- | -- |             data:  | 10 | 20 | 30 | -- |
         +----+----+----+----+                    +----+----+----+----+
                ^                                             ^
              top = 1                                       top = 2
  Step 1: ++top changes top from 1 to 2.   Step 2: data[2] = 30.
  ```
- **Complexity**:
  - Fixed Array: Worst-case Time $O(1)$, Auxiliary Space $O(1)$.
  - Dynamic Array: Worst-case Time $O(n)$ (when resizing copies $n$ elements), **Amortized Time $O(1)$** (since doubling after $n$ insertions performs $\sum_{i=0}^{\log_2 n} 2^i < 2n$ copies across $n$ pushes, averaging $< 2$ copies per push).

#### 2. `pop(S, &out_value)` — Array Stack
- **Purpose**: Remove the top element from `S` and store it in `*out_value`.
- **Preconditions**: Stack must not be empty (`s->top >= 0`).
- **Step-by-Step Algorithm**:
  1. Check if `s->top == -1`. If so, report **Stack Underflow** and return `false`.
  2. Copy `s->data[s->top]` into `*out_value`.
  3. Decrement `s->top` by 1 (`(s->top)--`).
  4. *(Optional for Dynamic Stack)*: If `size <= capacity / 4` and `capacity > MIN_CAP`, halve `capacity` via `realloc` to avoid **thrashing** (alternating `push`/`pop` at the resize boundary).
  5. Return `true`.
- **ASCII State Transition (`pop()` from `[10, 20, 30]`)**:
  ```
  BEFORE pop():                            AFTER pop() -> returns 30:
  index:   0    1    2    3                index:   0    1    2    3
         +----+----+----+----+                    +----+----+----+----+
  data:  | 10 | 20 | 30 | -- |             data:  | 10 | 20 | (30)| -- |
         +----+----+----+----+                    +----+----+----+----+
                     ^                                   ^
                   top = 2                             top = 1
  Note: 30 physically remains in data[2] as a stale value, but is logically
  excluded because top is now 1 (next push will overwrite data[2]).
  ```
- **Complexity**: Time $O(1)$ worst-case (static) / $O(1)$ amortized (dynamic with quarter-full shrinking); Space $O(1)$.

#### 3. `peek(S, &out_value)` / `isEmpty(S)` / `isFull(S)` / `size(S)` / `display(S)`
- **`peek`**: If `s->top == -1`, return `false` (Underflow); otherwise set `*out_value = s->data[s->top]` in $O(1)$ time without changing `s->top`.
- **`isEmpty`**: Returns `s->top == -1` in $O(1)$ time.
- **`isFull`**: Returns `s->top == s->capacity - 1` in $O(1)$ time.
- **`size`**: Returns `s->top + 1` in $O(1)$ time.
- **`display`**: Loops index `i` from `s->top` down to `0` in $O(n)$ time and $O(1)$ space.

---

### 4.2 Complete C11 Implementation: Static and Dynamic Array Stacks

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_STATIC_CAP 4

/* =========================================================================
 * PART 1: FIXED-CAPACITY STATIC ARRAY STACK
 * ========================================================================= */
typedef struct {
    int data[MAX_STATIC_CAP];
    int top;
} StaticStack;

void static_stack_init(StaticStack *s) {
    s->top = -1;
}

bool static_stack_is_empty(const StaticStack *s) {
    return s->top == -1;
}

bool static_stack_is_full(const StaticStack *s) {
    return s->top == MAX_STATIC_CAP - 1;
}

int static_stack_size(const StaticStack *s) {
    return s->top + 1;
}

bool static_stack_push(StaticStack *s, int value) {
    if (static_stack_is_full(s)) {
        printf("  [OVERFLOW] Cannot push %d: StaticStack is full (capacity = %d).\n",
               value, MAX_STATIC_CAP);
        return false;
    }
    s->data[++(s->top)] = value;
    printf("  Pushed %d -> top index = %d, size = %d\n",
           value, s->top, static_stack_size(s));
    return true;
}

bool static_stack_pop(StaticStack *s, int *out_value) {
    if (static_stack_is_empty(s)) {
        printf("  [UNDERFLOW] Cannot pop: StaticStack is empty (top = -1).\n");
        return false;
    }
    *out_value = s->data[(s->top)--];
    return true;
}

bool static_stack_peek(const StaticStack *s, int *out_value) {
    if (static_stack_is_empty(s)) {
        printf("  [UNDERFLOW] Cannot peek: StaticStack is empty.\n");
        return false;
    }
    *out_value = s->data[s->top];
    return true;
}

void static_stack_display(const StaticStack *s) {
    if (static_stack_is_empty(s)) {
        printf("  StaticStack: [ EMPTY ] (top = -1)\n");
        return;
    }
    printf("  StaticStack (top=%d, size=%d/%d): TOP -> ",
           s->top, static_stack_size(s), MAX_STATIC_CAP);
    for (int i = s->top; i >= 0; i--) {
        printf("[%d]%s", s->data[i], (i > 0) ? " " : "");
    }
    printf(" <- BOTTOM\n");
}

/* =========================================================================
 * PART 2: DYNAMICALLY RESIZABLE ARRAY STACK
 * ========================================================================= */
typedef struct {
    int *data;
    int top;
    int capacity;
} DynamicStack;

bool dyn_stack_init(DynamicStack *s, int initial_capacity) {
    if (initial_capacity <= 0) {
        initial_capacity = 2;
    }
    s->data = (int *)malloc((size_t)initial_capacity * sizeof(int));
    if (s->data == NULL) {
        return false;
    }
    s->top = -1;
    s->capacity = initial_capacity;
    return true;
}

bool dyn_stack_is_empty(const DynamicStack *s) {
    return s->top == -1;
}

int dyn_stack_size(const DynamicStack *s) {
    return s->top + 1;
}

static bool dyn_stack_resize(DynamicStack *s, int new_capacity) {
    int *new_block = (int *)realloc(s->data, (size_t)new_capacity * sizeof(int));
    if (new_block == NULL) {
        return false; /* Original s->data remains intact */
    }
    printf("  [RESIZE] Capacity changed: %d -> %d\n", s->capacity, new_capacity);
    s->data = new_block;
    s->capacity = new_capacity;
    return true;
}

bool dyn_stack_push(DynamicStack *s, int value) {
    if (s->top == s->capacity - 1) {
        if (!dyn_stack_resize(s, s->capacity * 2)) {
            printf("  [HEAP OVERFLOW] Failed to expand DynamicStack for value %d.\n", value);
            return false;
        }
    }
    s->data[++(s->top)] = value;
    printf("  Pushed %d -> top = %d, size = %d, capacity = %d\n",
           value, s->top, dyn_stack_size(s), s->capacity);
    return true;
}

bool dyn_stack_pop(DynamicStack *s, int *out_value) {
    if (dyn_stack_is_empty(s)) {
        printf("  [UNDERFLOW] Cannot pop: DynamicStack is empty.\n");
        return false;
    }
    *out_value = s->data[(s->top)--];
    /* Shrink capacity by half when load factor drops to 25% (minimum capacity 2) */
    if (s->capacity > 2 && dyn_stack_size(s) <= s->capacity / 4) {
        dyn_stack_resize(s, s->capacity / 2);
    }
    return true;
}

void dyn_stack_free(DynamicStack *s) {
    free(s->data);
    s->data = NULL;
    s->top = -1;
    s->capacity = 0;
}

int main(void) {
    printf("=== 1. Fixed-Capacity StaticStack Demo (MAX = %d) ===\n", MAX_STATIC_CAP);
    StaticStack st;
    static_stack_init(&st);
    static_stack_display(&st);

    int popped = 0;
    static_stack_pop(&st, &popped); /* Test underflow on empty stack */

    static_stack_push(&st, 10);
    static_stack_push(&st, 20);
    static_stack_push(&st, 30);
    static_stack_push(&st, 40);
    static_stack_display(&st);

    static_stack_push(&st, 50); /* Test overflow on full stack */

    if (static_stack_peek(&st, &popped)) {
        printf("  Peeked top element: %d (size unchanged = %d)\n",
               popped, static_stack_size(&st));
    }

    while (!static_stack_is_empty(&st)) {
        static_stack_pop(&st, &popped);
        printf("  Popped %d -> remaining size = %d\n", popped, static_stack_size(&st));
    }
    static_stack_display(&st);

    printf("\n=== 2. Resizable DynamicStack Demo (Initial Capacity = 2) ===\n");
    DynamicStack ds;
    if (!dyn_stack_init(&ds, 2)) {
        return EXIT_FAILURE;
    }

    for (int v = 100; v <= 500; v += 100) {
        dyn_stack_push(&ds, v);
    }

    while (!dyn_stack_is_empty(&ds)) {
        dyn_stack_pop(&ds, &popped);
        printf("  Popped %d -> size = %d, capacity = %d\n",
               popped, dyn_stack_size(&ds), ds.capacity);
    }

    dyn_stack_free(&ds);
    return EXIT_SUCCESS;
}
```

#### Program Output:

```text
=== 1. Fixed-Capacity StaticStack Demo (MAX = 4) ===
  StaticStack: [ EMPTY ] (top = -1)
  [UNDERFLOW] Cannot pop: StaticStack is empty (top = -1).
  Pushed 10 -> top index = 0, size = 1
  Pushed 20 -> top index = 1, size = 2
  Pushed 30 -> top index = 2, size = 3
  Pushed 40 -> top index = 3, size = 4
  StaticStack (top=3, size=4/4): TOP -> [40] [30] [20] [10] <- BOTTOM
  [OVERFLOW] Cannot push 50: StaticStack is full (capacity = 4).
  Peeked top element: 40 (size unchanged = 4)
  Popped 40 -> remaining size = 3
  Popped 30 -> remaining size = 2
  Popped 20 -> remaining size = 1
  Popped 10 -> remaining size = 0
  StaticStack: [ EMPTY ] (top = -1)

=== 2. Resizable DynamicStack Demo (Initial Capacity = 2) ===
  Pushed 100 -> top = 0, size = 1, capacity = 2
  Pushed 200 -> top = 1, size = 2, capacity = 2
  [RESIZE] Capacity changed: 2 -> 4
  Pushed 300 -> top = 2, size = 3, capacity = 4
  Pushed 400 -> top = 3, size = 4, capacity = 4
  [RESIZE] Capacity changed: 4 -> 8
  Pushed 500 -> top = 4, size = 5, capacity = 8
  Popped 500 -> size = 4, capacity = 8
  Popped 400 -> size = 3, capacity = 8
  [RESIZE] Capacity changed: 8 -> 4
  Popped 300 -> size = 2, capacity = 4
  [RESIZE] Capacity changed: 4 -> 2
  Popped 200 -> size = 1, capacity = 2
  Popped 100 -> size = 0, capacity = 2
```

---

### 4.3 Linked-List-Based Stack Operations (`push`, `pop`, `peek`, `isEmpty`, `display`, `free_stack`)

#### 1. `push(S, value)` — Linked-List Stack
- **Purpose**: Allocate a new `StackNode` containing `value` and prepend it at `s->top` in $O(1)$ time.
- **Preconditions**: Heap must have sufficient memory for `malloc(sizeof(StackNode))`.
- **Step-by-Step Algorithm**:
  1. Call `new_node = malloc(sizeof(StackNode))`.
  2. If `new_node == NULL`, report **Heap Overflow** and return `false`.
  3. Set `new_node->data = value`.
  4. Link `new_node->next = s->top` (points to the former top node, or `NULL` if stack was empty).
  5. Update `s->top = new_node` and increment `s->size`.
  6. Return `true`.
- **Pseudocode**:
  ```text
  Algorithm LinkedStack_Push(S, value):
      new_node <- allocate StackNode
      if new_node == NULL then
          error "Heap Overflow"
          return false
      new_node.data <- value
      new_node.next <- S.top
      S.top <- new_node
      S.size <- S.size + 1
      return true
  ```
- **ASCII State Transition (`push(45)` onto `[30, 15]`)**:
  ```
  BEFORE push(45):
    s->top
      |
      v
    [30 | next] ---> [15 | NULL]

  STEP 1 & 2 (Allocate node 45 and link new_node->next = s->top):
    new_node         s->top
      |                |
      v                v
    [45 | next] ---> [30 | next] ---> [15 | NULL]

  STEP 3 (Update s->top = new_node):
    s->top
      |
      v
    [45 | next] ---> [30 | next] ---> [15 | NULL]
  ```
- **Complexity**: Worst-case Time $O(1)$, Auxiliary Space $O(1)$ per push.

#### 2. `pop(S, &out_value)` — Linked-List Stack
- **Purpose**: Remove the node at `s->top`, save its `data` into `*out_value`, and `free` the removed node in $O(1)$ time.
- **Preconditions**: Stack is non-empty (`s->top != NULL`).
- **Step-by-Step Algorithm**:
  1. Check if `s->top == NULL`. If so, report **Stack Underflow** and return `false`.
  2. Save a temporary pointer `StackNode *victim = s->top`.
  3. Copy `victim->data` into `*out_value`.
  4. Advance `s->top = victim->next`.
  5. Call `free(victim)` and decrement `s->size`.
  6. Return `true`.
- **ASCII State Transition (`pop()` from `[45, 30, 15]`)**:
  ```
  BEFORE pop():
    s->top
      |
      v
    [45 | next] ---> [30 | next] ---> [15 | NULL]

  STEP 1 & 2 (victim = s->top; s->top = victim->next):
    victim           s->top
      |                |
      v                v
    [45 | next] ---> [30 | next] ---> [15 | NULL]

  STEP 3 (free(victim)):
                     s->top
                       |
                       v
                     [30 | next] ---> [15 | NULL]
  ```
- **Edge Case**: When popping the **last remaining node** (`s->size == 1`), `victim->next` is `NULL`, so `s->top = victim->next` automatically resets `s->top` to `NULL` without needing a special branch!

#### 3. `free_stack(S)` — Deallocating All Nodes
- **Purpose**: Prevent memory leaks when destroying a non-empty `LinkedStack`.
- **Algorithm**: Iterate `curr = s->top`; while `curr != NULL`, save `next_node = curr->next`, call `free(curr)`, and advance `curr = next_node`. Reset `s->top = NULL` and `s->size = 0`. Runs in $O(n)$ time and $O(1)$ auxiliary space.

---

### 4.4 Complete C11 Implementation: Linked-List Stack

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct StackNode {
    int data;
    struct StackNode *next;
} StackNode;

typedef struct {
    StackNode *top;
    size_t size;
} LinkedStack;

void linked_stack_init(LinkedStack *s) {
    s->top = NULL;
    s->size = 0;
}

bool linked_stack_is_empty(const LinkedStack *s) {
    return s->top == NULL;
}

size_t linked_stack_size(const LinkedStack *s) {
    return s->size;
}

bool linked_stack_push(LinkedStack *s, int value) {
    StackNode *new_node = (StackNode *)malloc(sizeof(StackNode));
    if (new_node == NULL) {
        printf("  [HEAP OVERFLOW] malloc failed when pushing %d.\n", value);
        return false;
    }
    new_node->data = value;
    new_node->next = s->top; /* Prepend at head in O(1) */
    s->top = new_node;
    s->size++;
    printf("  Pushed %d at head -> size = %zu\n", value, s->size);
    return true;
}

bool linked_stack_pop(LinkedStack *s, int *out_value) {
    if (linked_stack_is_empty(s)) {
        printf("  [UNDERFLOW] Cannot pop: LinkedStack is empty (top = NULL).\n");
        return false;
    }
    StackNode *victim = s->top;
    *out_value = victim->data;
    s->top = victim->next; /* Advance head in O(1) */
    free(victim);
    s->size--;
    return true;
}

bool linked_stack_peek(const LinkedStack *s, int *out_value) {
    if (linked_stack_is_empty(s)) {
        printf("  [UNDERFLOW] Cannot peek: LinkedStack is empty.\n");
        return false;
    }
    *out_value = s->top->data;
    return true;
}

void linked_stack_display(const LinkedStack *s) {
    printf("  LinkedStack (size=%zu): TOP -> ", s->size);
    const StackNode *curr = s->top;
    while (curr != NULL) {
        printf("[%d] -> ", curr->data);
        curr = curr->next;
    }
    printf("NULL\n");
}

void linked_stack_free(LinkedStack *s) {
    StackNode *curr = s->top;
    size_t freed_count = 0;
    while (curr != NULL) {
        StackNode *next_node = curr->next;
        free(curr);
        curr = next_node;
        freed_count++;
    }
    s->top = NULL;
    s->size = 0;
    printf("  Freed %zu remaining node(s). Stack reset to empty.\n", freed_count);
}

int main(void) {
    printf("=== Linked-List Stack Demo (Head = Top) ===\n");
    LinkedStack ls;
    linked_stack_init(&ls);
    linked_stack_display(&ls);

    int val = 0;
    linked_stack_pop(&ls, &val); /* Underflow test */

    linked_stack_push(&ls, 15);
    linked_stack_push(&ls, 30);
    linked_stack_push(&ls, 45);
    linked_stack_push(&ls, 60);
    linked_stack_display(&ls);

    if (linked_stack_peek(&ls, &val)) {
        printf("  Peeked top element: %d (size = %zu)\n", val, linked_stack_size(&ls));
    }

    if (linked_stack_pop(&ls, &val)) {
        printf("  Popped %d -> size = %zu\n", val, linked_stack_size(&ls));
    }
    if (linked_stack_pop(&ls, &val)) {
        printf("  Popped %d -> size = %zu\n", val, linked_stack_size(&ls));
    }
    linked_stack_display(&ls);

    linked_stack_free(&ls);
    linked_stack_display(&ls);
    return EXIT_SUCCESS;
}
```

#### Program Output:

```text
=== Linked-List Stack Demo (Head = Top) ===
  LinkedStack (size=0): TOP -> NULL
  [UNDERFLOW] Cannot pop: LinkedStack is empty (top = NULL).
  Pushed 15 at head -> size = 1
  Pushed 30 at head -> size = 2
  Pushed 45 at head -> size = 3
  Pushed 60 at head -> size = 4
  LinkedStack (size=4): TOP -> [60] -> [45] -> [30] -> [15] -> NULL
  Peeked top element: 60 (size = 4)
  Popped 60 -> size = 3
  Popped 45 -> size = 2
  LinkedStack (size=2): TOP -> [30] -> [15] -> NULL
  Freed 2 remaining node(s). Stack reset to empty.
  LinkedStack (size=0): TOP -> NULL
```

---

## 5. Comparison Tables & Complexity Summary

### 5.1 Architectural Comparison: Fixed Array vs. Dynamic Array vs. Linked-List Stack

Let $n$ denote the number of elements currently in the stack, and $C$ denote the allocated array capacity.

| Feature / Dimension | Fixed-Capacity Array Stack | Dynamic Resizable Array Stack | Linked-List Stack (Head = Top) |
| :--- | :--- | :--- | :--- |
| **Underlying Storage** | Static array `int data[C]` | Heap array `int *data` via `malloc`/`realloc` | Non-contiguous heap nodes (`StackNode`) |
| **`push(x)` Time (Worst-Case)** | $O(1)$ | $O(n)$ *(when `realloc` copies $n$ elements)* | $O(1)$ *(single `malloc` + pointer link)* |
| **`push(x)` Time (Amortized)** | $O(1)$ | $O(1)$ *(via geometric doubling $C \leftarrow 2C$)* | $O(1)$ |
| **`pop()` Time (Worst-Case)** | $O(1)$ | $O(n)$ *(if shrinking at $n = C/4$, else $O(1)$)* | $O(1)$ *(single pointer advance + `free`)* |
| **`pop()` Time (Amortized)** | $O(1)$ | $O(1)$ | $O(1)$ |
| **`peek()` / `isEmpty()` / `size()`** | $O(1)$ worst-case | $O(1)$ worst-case | $O(1)$ worst-case |
| **Worst-Case Resize Cost** | None ($O(0)$, cannot resize) | $O(n)$ element copies during doubling/halving | None ($O(0)$, grows 1 node at a time) |
| **Per-Element Memory Overhead** | $0$ bytes per live element *(unused slots $C - n$ wasted)* | $0$ bytes per live element *(slack capacity $\le 3n$ when shrinking at $C/4$)* | `sizeof(StackNode*)` (8B) + heap allocator metadata (8–16B) $\approx 16\text{–}24$ bytes per node |
| **CPU Cache Locality** | **Excellent** (contiguous L1/L2 cache lines; hardware prefetcher friendly) | **Excellent** (contiguous heap block) | **Poor** (nodes scattered across heap; each access can trigger a cache miss) |
| **Overflow Behavior** | Fails immediately when $n = C$ (even if system RAM is plentiful) | Grows until contiguous heap block of size $2C$ cannot be allocated | Grows until system heap is completely exhausted (`malloc == NULL`) |

### 5.2 Time and Space Complexity Summary Table

Let $n$ be the number of elements currently stored in the stack, and $C$ be the capacity of the fixed array.

| Operation | Fixed Array Stack (Time) | Dynamic Array Stack (Time) | Linked-List Stack (Time) | Auxiliary Space (Per Call) |
| :--- | :---: | :---: | :---: | :---: |
| `init` / `create` | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| `push(x)` | $O(1)$ | $O(1)$ amortized / $O(n)$ worst | $O(1)$ worst-case | $O(1)$ |
| `pop()` | $O(1)$ | $O(1)$ amortized / $O(n)$ worst | $O(1)$ worst-case | $O(1)$ |
| `peek()` / `top()` | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| `isEmpty()` / `isFull()` | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ |
| `size()` | $O(1)$ | $O(1)$ | $O(1)$ *(with `size` field)* | $O(1)$ |
| `display()` | $O(n)$ | $O(n)$ | $O(n)$ | $O(1)$ |
| `free_stack()` | $O(1)$ | $O(1)$ *(single `free(s->data)`)* | $O(n)$ *(loop freeing $n$ nodes)* | $O(1)$ |
| **Total Structure Space** | $\Theta(C)$ | $\Theta(n)$ *(since $n \le C \le 4n$)* | $\Theta(n)$ | — |

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Frequent C Implementation Bugs and Fixes

#### Pitfall 1: Post-Increment Instead of Pre-Increment in `push` (Out-of-Bounds Write at Index `-1`)
When `top` is initialized to `-1`, using `s->data[s->top++] = val` indexes `s->data[-1]` **before** incrementing `top`, corrupting memory immediately preceding the array!

```c
/* BUG: Post-increment evaluates s->top (-1) BEFORE incrementing! */
void push_buggy(StaticStack *s, int val) {
    s->data[s->top++] = val; /* Writes to s->data[-1] when stack is empty -> UB! */
}

/* FIX: Use pre-increment (++s->top) so -1 becomes 0 before indexing. */
void push_fixed(StaticStack *s, int val) {
    s->data[++(s->top)] = val; /* Increments top to 0 first, then writes s->data[0] */
}
```

#### Pitfall 2: Pre-Decrement Instead of Post-Decrement in `pop` (Skipping the Top Element)
Conversely, during `pop()`, `s->top` already points to the valid top element. Using `--(s->top)` decrements `top` too early, returning the element *below* the top and reading `s->data[-1]` when popping a 1-element stack.

```c
/* BUG: Pre-decrement decrements top BEFORE reading the top element! */
int pop_buggy(StaticStack *s) {
    return s->data[--(s->top)]; /* Misses current top; reads data[-1] when top == 0! */
}

/* FIX: Use post-decrement ((s->top)--) to read s->data[top] first, then decrement. */
int pop_fixed(StaticStack *s) {
    return s->data[(s->top)--];
}
```

#### Pitfall 3: Direct `s->data = realloc(s->data, ...)` (Memory Leak on Allocation Failure)
If `realloc` fails to allocate the expanded block, it returns `NULL` **without freeing the original block**. Overwriting `s->data` directly loses the only pointer to the existing heap array.

```c
/* BUG: Overwrites s->data with NULL if realloc fails, leaking the old buffer! */
bool resize_buggy(DynamicStack *s, int new_cap) {
    s->data = (int *)realloc(s->data, (size_t)new_cap * sizeof(int));
    return s->data != NULL;
}

/* FIX: Assign realloc result to a temporary pointer first. */
bool resize_fixed(DynamicStack *s, int new_cap) {
    int *tmp = (int *)realloc(s->data, (size_t)new_cap * sizeof(int));
    if (tmp == NULL) {
        return false; /* s->data is still valid and uncorrupted */
    }
    s->data = tmp;
    s->capacity = new_cap;
    return true;
}
```

#### Pitfall 4: Memory Leak in Linked-List Stack `pop`
Advancing `s->top = s->top->next` without saving the old `s->top` pointer makes it impossible to `free()` the popped node.

```c
/* BUG: Advances s->top directly, leaking the popped heap node on every pop! */
int linked_pop_buggy(LinkedStack *s) {
    int val = s->top->data;
    s->top = s->top->next; /* Old s->top node is now leaked forever! */
    return val;
}

/* FIX: Save old top in a victim pointer, advance s->top, then free(victim). */
int linked_pop_fixed(LinkedStack *s) {
    StackNode *victim = s->top;
    int val = victim->data;
    s->top = victim->next;
    free(victim);
    s->size--;
    return val;
}
```

#### Pitfall 5: Resize Thrashing When Shrinking Dynamic Stack at `size == capacity / 2`
If a dynamic stack doubles when `size == capacity` and halves immediately when `size == capacity / 2`, alternating `push` and `pop` at the boundary triggers an $O(n)$ `realloc` on **every single operation**, degrading amortized complexity from $O(1)$ to $O(n)$.
- **Fix**: Only halve `capacity` when `size <= capacity / 4` (25% load factor). After halving, the stack is 50% full, requiring at least $C/2$ pushes or $C/4$ pops before the next resize can occur.

### 6.2 Boundary Conditions Checklist

- [ ] **Empty Stack (`top == -1` or `top == NULL`)**: Do both `pop()` and `peek()` check `isEmpty()` before dereferencing `s->data[s->top]` or `s->top->data`?
- [ ] **Single-Element Stack (`size == 1`)**: Does `pop()` cleanly transition `s->top` back to `-1` (array) or `NULL` (linked list)?
- [ ] **Full Stack (`top == capacity - 1`)**: Does `push()` check `isFull()` *before* incrementing `s->top`?
- [ ] **Allocation Failure (`malloc` / `realloc == NULL`)**: Does `push()` leave the existing stack state intact and return `false` if heap allocation fails?
- [ ] **Destruction of Non-Empty Stack**: Does `linked_stack_free()` free every remaining node and reset `s->top = NULL` and `s->size = 0`?

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways and Core Invariants

1. **LIFO Discipline**: A Stack restricts all insertions (`push`), removals (`pop`), and inspections (`peek`) to a single end called `top`, guaranteeing Last-In, First-Out ordering in $O(1)$ time.
2. **Array Stack Convention**: Setting `top = -1` for an empty stack ensures `size == top + 1`, `push` uses `s->data[++(s->top)] = x`, and `pop` uses `x = s->data[(s->top)--]`.
3. **Linked-List Stack Convention**: The top of a linked-list stack **must** be the `head` of the singly linked list so that both `push` (prepend at head) and `pop` (remove head) run in $O(1)$ worst-case time.
4. **Amortized Dynamic Resizing**: Doubling capacity at $100\%$ full and halving capacity at $25\%$ full yields $O(1)$ amortized `push` and `pop` while bounding memory usage to $\Theta(n)$.

### 7.2 Self-Test Revision Checklist

- [ ] Can I trace `top` and array contents after a mixed sequence of `push`, `pop`, and `peek` calls?
- [ ] Can I explain why `s->data[s->top++] = x` causes undefined behavior when `top` starts at `-1`?
- [ ] Can I prove why using the tail of a singly linked list as the stack top makes `pop()` an $O(n)$ operation?
- [ ] Can I write a leak-free `linked_stack_pop()` and `linked_stack_free()` from memory in C11?
- [ ] Can I explain why shrinking a dynamic array at $25\%$ occupancy instead of $50\%$ prevents resize thrashing?

### 7.3 Navigation Links

- **Previous Topic**: [06 — Polynomial Representation and Operations](06-Polynomial-Representation-and-Operations.md)
- **Next Topic**: [08 — Stack Applications and Recursion](08-Stack-Applications-and-Recursion.md)
- **Unit Review**: [10 — Unit 1 Comprehensive Review & Exam Preparation](10-Unit-1-Review.md)
