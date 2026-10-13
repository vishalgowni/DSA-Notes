# 08 — Stack Applications and Recursion

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [Why Stacks Underpin Subroutine Execution and Nested Syntax](#11-why-stacks-underpin-subroutine-execution-and-nested-syntax)
   - 1.2 [Relationship to Prior Concepts](#12-relationship-to-prior-concepts)
   - 1.3 [Learning Objectives](#13-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [System Call Stack and Activation Records](#21-system-call-stack-and-activation-records)
   - 2.2 [Recursion Classifications and Terminology](#22-recursion-classifications-and-terminology)
   - 2.3 [Balanced Delimiters and the LIFO Nesting Property](#23-balanced-delimiters-and-the-lifo-nesting-property)
   - 2.4 [Terminology Reference Table](#24-terminology-reference-table)
   - 2.5 [Mathematical Properties and Invariants](#25-mathematical-properties-and-invariants)
   - 2.6 [Step-by-Step ASCII Diagrams: Call Stack Frames and Recursion Trees](#26-step-by-step-ascii-diagrams-call-stack-frames-and-recursion-trees)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Process Memory Layout and Hardware Stack Frames](#31-process-memory-layout-and-hardware-stack-frames)
   - 3.2 [Explicit Activation-Record Structs for Iterative Conversion](#32-explicit-activation-record-structs-for-iterative-conversion)
   - 3.3 [Delimiter Stack Representation for Syntax Validation](#33-delimiter-stack-representation-for-syntax-validation)
   - 3.4 [Implicit OS Call Stack vs. Explicit Heap/Array Stack](#34-implicit-os-call-stack-vs-explicit-heaparray-stack)
4. [Operations and Algorithms](#4-operations-and-algorithms)
   - 4.1 [Recursion Analysis: Factorial, Fibonacci, Tower of Hanoi, Binary Search](#41-recursion-analysis-factorial-fibonacci-tower-of-hanoi-binary-search)
   - 4.2 [Tail Recursion vs. Non-Tail Recursion & Converting Recursion to an Explicit Stack](#42-tail-recursion-vs-non-tail-recursion--converting-recursion-to-an-explicit-stack)
   - 4.3 [Complete C11 Implementation: Call-Stack Trace, Tail Recursion, and Explicit-Stack Hanoi](#43-complete-c11-implementation-call-stack-trace-tail-recursion-and-explicit-stack-hanoi)
   - 4.4 [Matching Parentheses and Brackets (Balanced Delimiters) Algorithm & Full Trace Tables](#44-matching-parentheses-and-brackets-balanced-delimiters-algorithm--full-trace-tables)
   - 4.5 [Complete C11 Implementation: Multi-Delimiter Balanced Parentheses Checker](#45-complete-c11-implementation-multi-delimiter-balanced-parentheses-checker)
5. [Comparison Tables & Complexity Summary](#5-comparison-tables--complexity-summary)
   - 5.1 [Recursion vs. Explicit-Stack Iteration Comparison](#51-recursion-vs-explicit-stack-iteration-comparison)
   - 5.2 [Tail Recursion vs. Non-Tail Recursion Comparison](#52-tail-recursion-vs-non-tail-recursion-comparison)
   - 5.3 [Algorithmic Time and Call-Stack Space Complexity Summary](#53-algorithmic-time-and-call-stack-space-complexity-summary)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Frequent C Bugs in Recursion and Bracket Matching](#61-frequent-c-bugs-in-recursion-and-bracket-matching)
   - 6.2 [Boundary Conditions Checklist](#62-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)
   - 7.1 [Key Takeaways and Core Invariants](#71-key-takeaways-and-core-invariants)
   - 7.2 [Self-Test Revision Checklist](#72-self-test-revision-checklist)
   - 7.3 [Navigation Links](#73-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 Why Stacks Underpin Subroutine Execution and Nested Syntax

Whenever a function $A()$ invokes a function $B()$, and $B()$ invokes $C()$, execution must finish $C()$ before resuming $B()$, and must finish $B()$ before resuming $A()$. The **last** function called is always the **first** function to complete. Because subroutine invocations strictly obey the **LIFO (Last-In, First-Out)** principle, every modern CPU architecture and programming language runtime uses a **System Call Stack** to manage function calls, local variables, and return addresses.

This same LIFO mechanism makes **recursion**—a function calling itself with smaller inputs—possible: each recursive invocation receives its own isolated **activation record (stack frame)** on the call stack. Furthermore, whenever human-written code or data contains **nested scopes** such as parentheses `()`, braces `{}`, and brackets `[]`, the most recently opened delimiter must be closed first, making an explicit stack the canonical data structure for lexical and syntactic validation.

### 1.2 Relationship to Prior Concepts

In [07-Stacks-Array-and-Linked-List.md](07-Stacks-Array-and-Linked-List.md), we constructed the Stack ADT (`push`, `pop`, `peek`, `isEmpty`) using arrays and singly linked lists. In this chapter, we apply that exact LIFO discipline to three foundational problems:
1. Understanding how C executes nested and recursive functions via hardware stack frames.
2. Translating recursive algorithms into iterative algorithms backed by an explicit user-managed stack.
3. Validating multi-delimiter balanced expressions in $O(n)$ time.

### 1.3 Learning Objectives

After completing this chapter, you will be able to:
- Describe the internal fields of an **Activation Record (Stack Frame)** and trace how the call stack grows and unwinds across nested function calls.
- Distinguish between **base cases** and **recursive cases**, **direct** vs. **indirect (mutual)** recursion, and **tail** vs. **non-tail** recursion.
- Draw recursion trees and compute time and call-stack space complexity for **Factorial**, **Fibonacci**, **Tower of Hanoi**, and **Binary Search**.
- Convert a non-tail recursive algorithm into an iterative C11 function using an explicit stack of activation frames.
- Trace and implement the **Balanced Delimiters** algorithm handling all three failure modes: *mismatched closer*, *closer on empty stack*, and *unclosed opener at end of input*.

---

## 2. Definitions and Terminology

### 2.1 System Call Stack and Activation Records

When any function is called in C, the runtime environment pushes a contiguous block of metadata and local storage onto the **System Call Stack** called an **Activation Record** (or **Stack Frame**). When the function executes `return`, its stack frame is popped, restoring the caller's context.

An activation record typically stores:
1. **Actual Parameters (Arguments)**: Values passed by the caller (or spilled from argument registers).
2. **Return Address**: The instruction pointer (`RIP`/`PC`) address in the caller to jump back to when the callee finishes.
3. **Saved Frame Pointer (Dynamic Link)**: The caller's base/frame pointer (`RBP`/`FP`) so the caller's local variable offsets are restored upon return.
4. **Saved Registers**: Callee-saved CPU registers that must be preserved across the call.
5. **Local (Automatic) Variables**: Variables declared inside the function body without `static`.

### 2.2 Recursion Classifications and Terminology

A function is **recursive** if its definition contains a call to itself (directly or indirectly). Every terminating recursive function requires two components:
- **Base Case (Stopping Condition)**: One or more simple input instances solved directly **without** making any further recursive calls.
- **Recursive Case (Inductive Step)**: Decomposes the problem of size $n$ into one or more smaller subproblems (e.g., $n - 1$ or $n / 2$) that strictly progress toward a base case.

#### Structural Classifications of Recursion
1. **Direct Recursion**: Function $f()$ explicitly calls $f()$ inside its own body.
2. **Indirect (Mutual) Recursion**: Function $f()$ calls $g()$, and $g()$ calls $f()$ (forming a cycle of two or more functions).
3. **Linear Recursion**: Each activation makes **at most one** recursive call (e.g., `factorial(n)` or `binary_search()`). The recursion tree is a single chain.
4. **Tree (Multiple) Recursion**: An activation makes **two or more** recursive calls (e.g., `fibonacci(n)` or `hanoi(n)`), branching into a tree of calls.
5. **Tail Recursion**: A special case of linear recursion in which the recursive call is the **very last operation** performed before returning, with **zero pending computation** (such as multiplication or addition) after the recursive call returns.

### 2.3 Balanced Delimiters and the LIFO Nesting Property

Given a string containing opening delimiters $O = \{\text{'('}, \text{'['}, \text{'\{'}\}$ and closing delimiters $C = \{\text{')'}, \text{']'}, \text{'\}'}\}$, the delimiters are **balanced (well-formed)** if and only if:
1. Every closing delimiter matches the **type** of the most recently seen **unclosed** opening delimiter.
2. Every closing delimiter is preceded by a corresponding unclosed opening delimiter.
3. No opening delimiter remains unclosed when the end of the string is reached.

> [!IMPORTANT]
> **Why Counting Alone Fails and LIFO Is Mandatory:**
> Simply counting `open_parens == close_parens` and `open_brackets == close_brackets` would falsely accept `[(])`! In `[(])`, `'['` is opened first and `'('` is opened second. Because `'('` is the **inner (most recent)** scope, it **must** be closed by `')'` before `'['` can be closed by `']'`. A LIFO stack naturally enforces this innermost-first nesting rule.

### 2.4 Terminology Reference Table

| Term | Definition | Role in Stack / Execution |
| :--- | :--- | :--- |
| **Activation Record (Stack Frame)** | Memory block pushed onto the system call stack for a single function invocation. | Holds parameters, return address, saved `FP`, and local variables. |
| **Stack Pointer (`SP` / `RSP`)** | Hardware register pointing to the current top of the system call stack. | Decremented on push (frame allocation), incremented on pop (return). |
| **Frame Pointer (`FP` / `RBP`)** | Hardware register pointing to a fixed anchor address within the current stack frame. | Used to access parameters and local variables at constant byte offsets. |
| **Return Address** | Code address of the instruction immediately following the `call` instruction in the caller. | Popped into the instruction pointer (`PC`/`RIP`) when the callee returns. |
| **Base Case** | Input condition where a recursive function returns directly without recurring. | Stops call-stack growth and initiates stack unwinding. |
| **Recursion Depth** | Maximum number of simultaneous active frames on the call stack at any single moment. | Determines the **auxiliary space complexity** $O(d)$ of a recursive algorithm. |
| **Tail Call Elimination (TCE)** | Compiler optimization that overwrites the current stack frame for a tail call instead of pushing a new frame. | Reduces auxiliary call-stack space of tail recursion from $O(n)$ to $O(1)$. |
| **Stack Overflow (Runtime)** | Fatal fault (`SIGSEGV`) when the system call stack exceeds its OS thread limit (typically 1–8 MB). | Caused by missing base cases or excessive recursion depth $O(n)$. |

### 2.5 Mathematical Properties and Invariants

- **Call-Stack Space vs. Total Recursive Calls**:
  - In **Tree Recursion** (such as `fib(n)` or `hanoi(n)`), the total number of calls is $\Theta(2^n)$ (the total number of nodes in the recursion tree), whereas the **maximum stack space** at any instant is proportional to the **height of the recursion tree**, which is only $\Theta(n)$ because a finished branch is popped before its sibling branch is pushed!
- **Balanced Delimiters Failure Mode Completeness**:
  Every invalid delimiter string violates one of exactly three mutually exhaustive conditions:
  1. **Closer on Empty Stack** (`())`): Stack underflows while scanning input.
  2. **Mismatched Closer** (`[(])`): `pop()` returns an opener of a different bracket family than the current closer.
  3. **Unclosed Opener at End** (`((()`): Input finishes with `isEmpty(stack) == false`.

### 2.6 Step-by-Step ASCII Diagrams: Call Stack Frames and Recursion Trees

#### A. System Call Stack Growth and Unwinding Across Nested Calls
Consider `main()` calling `sum_of_squares(3, 4)`, which calls `square(3)` and then `square(4)`:

```
  1. Inside main()         2. main calls             3. sum_of_squares calls   4. square(3) returns 9;
                              sum_of_squares(3, 4)      square(3)                 sum_of_squares calls square(4)
  +------------------+     +------------------+      +------------------+      +------------------+
  | main() Frame     |     | main() Frame     |      | main() Frame     |      | main() Frame     |
  |  result = ?      |     |  result = ?      |      |  result = ?      |      |  result = ?      |
  +------------------+     +------------------+      +------------------+      +------------------+
         ^ SP              | sum_of_squares   |      | sum_of_squares   |      | sum_of_squares   |
                           |  params: a=3,b=4 |      |  params: a=3,b=4 |      |  params: a=3,b=4 |
                           |  ret_addr: main  |      |  ret_addr: main  |      |  ret_addr: main  |
                           |  locals: s1=?,s2 |      |  locals: s1=?,s2 |      |  locals: s1=9,s2 |
                           +------------------+      +------------------+      +------------------+
                                  ^ SP               | square(x=3)      |      | square(x=4)      |
                                                     |  ret_addr: sum.. |      |  ret_addr: sum.. |
                                                     |  return 3*3 = 9  |      |  return 4*4 = 16 |
                                                     +------------------+      +------------------+
                                                            ^ SP                      ^ SP
  (Note: Once square(3) returns 9, its frame is POPPED, and square(4) reuses that exact stack space!)
```

#### B. Recursion Trees: Linear vs. Tree Recursion

```
  1. Factorial: fact(4) — Linear Recursion (Time: O(n), Max Stack Depth: O(n))
  ============================================================================
     fact(4)  ---> waits for 4 * fact(3)           [Unwind: 4 * 6  = 24]
       |
     fact(3)  ---> waits for 3 * fact(2)           [Unwind: 3 * 2  = 6 ]
       |
     fact(2)  ---> waits for 2 * fact(1)           [Unwind: 2 * 1  = 2 ]
       |
     fact(1)  ---> BASE CASE returns 1             [Unwind: returns 1  ]


  2. Fibonacci: fib(4) — Tree Recursion (Time: O(2^n), Max Stack Depth: O(n))
  ===========================================================================
                              fib(4) [returns 3]
                             /                  \
                   fib(3) [returns 2]            fib(2) [returns 1]
                  /                  \          /                  \
        fib(2) [returns 1]         fib(1)    fib(1)              fib(0)
       /                  \         [=1]      [=1]                [=0]
    fib(1)              fib(0)
     [=1]                [=0]

  Notice: fib(2) is computed TWICE, and fib(1) is computed THREE times!
  Max active stack frames at once = 4 (path: fib(4) -> fib(3) -> fib(2) -> fib(1)).


  3. Tower of Hanoi: H(3, A, B, C) — 2^3 - 1 = 7 Moves (Time: O(2^n), Space: O(n))
  ================================================================================
                           H(3, A->C via B) [Move #4: Disk 3 A->C]
                          /                                       \
            H(2, A->B via C) [Move #2: D2 A->B]         H(2, B->C via A) [Move #6: D2 B->C]
             /                               \           /                               \
      H(1, A->C)                           H(1, C->B) H(1, B->A)                       H(1, A->C)
   [Move #1: D1 A->C]                   [Move #3: D1] [Move #5: D1]                 [Move #7: D1 A->C]
```

---

## 3. Representation and Memory Layout in C

### 3.1 Process Memory Layout and Hardware Stack Frames

In a standard C process virtual address space (x86-64 / ARM64), the **System Call Stack** starts near the top of user-space memory (high addresses) and **grows downward** toward lower memory addresses as functions are called, while the **Heap** (`malloc`/`realloc`) grows upward from lower addresses.

```
  High Memory Addresses (e.g., 0x7FFFFFFF_FFFF)
  +---------------------------------------------------+
  | Command-Line Arguments (argv) & Environment       |
  +---------------------------------------------------+
  | caller Frame (e.g., factorial(4))                 |
  |   [RBP + 16] Return Address (to main)             |
  |   [RBP + 0 ] Saved Caller RBP                     |
  |   [RBP - 4 ] Local / Parameter: n = 4             |
  +---------------------------------------------------+
  | callee Frame (e.g., factorial(3))                 |  <-- Pushed by CALL
  |   [RBP + 16] Return Address (into factorial(4))   |
  |   [RBP + 0 ] Saved Caller RBP (factorial(4)'s FP) |  <-- Current RBP (Frame Pointer)
  |   [RBP - 4 ] Local / Parameter: n = 3             |  <-- Current RSP (Stack Pointer)
  +---------------------------------------------------+
  |                        |                          |
  |                        v  (Call Stack grows DOWN) |
  |                                                   |
  |                  Unmapped Guard Pages             |
  |                                                   |
  |                        ^  (Heap grows UP)         |
  |                        |                          |
  +---------------------------------------------------+
  | Heap Segment (malloc / calloc / realloc)          |
  +---------------------------------------------------+
  | BSS & Data Segments (Global & static variables)   |
  +---------------------------------------------------+
  | Text Segment (Compiled Machine Code Instructions) |
  +---------------------------------------------------+
  Low Memory Addresses (e.g., 0x00400000)
```

### 3.2 Explicit Activation-Record Structs for Iterative Conversion

To convert any recursive function into an iterative function that cannot trigger a hardware call-stack overflow, we define an explicit `struct` mirroring the parameters, local variables, and resumption stage (program counter state) of the recursive function:

```c
typedef struct {
    int n;      /* Saved parameter for this invocation */
    int stage;  /* Execution stage: 0 = before recursive call, 1 = after return */
} FactFrame;
```

### 3.3 Delimiter Stack Representation for Syntax Validation

When validating balanced brackets, storing both the opening `char bracket` and its `int index` in each stack slot allows our validator to report the **exact character offset** of any unclosed or mismatched bracket:

```c
typedef struct {
    char bracket; /* Opening delimiter: '(', '[', or '{' */
    int index;    /* 0-based position in the input string */
} DelimToken;

typedef struct {
    DelimToken data[MAX_EXPR_LEN];
    int top;      /* -1 when empty */
} DelimStack;
```

### 3.4 Implicit OS Call Stack vs. Explicit Heap/Array Stack

| Characteristic | Implicit OS System Call Stack (Recursion) | Explicit User-Managed Stack (Iteration) |
| :--- | :--- | :--- |
| **Frame Allocation** | Automatic CPU instructions (`call` / `ret`, `push` / `pop`) | Manual `stack[++top] = frame` on array or heap |
| **Per-Frame Size** | 16–64+ bytes (ABI alignment, return address, saved registers, locals) | Minimal—only the fields explicitly needed (e.g., 4–16 bytes) |
| **Maximum Capacity** | Strictly bounded by OS thread stack limit (typically 1 MB on Windows, 8 MB on Linux) | Bounded only by available system RAM (gigabytes) when heap-allocated |
| **Failure Mode on Overflow** | Immediate process crash (`STATUS_stack_overflow` / `SIGSEGV`) | Graceful error handling (`malloc`/`realloc` returns `NULL`) |

---

## 4. Operations and Algorithms

### 4.1 Recursion Analysis: Factorial, Fibonacci, Tower of Hanoi, Binary Search

1. **Factorial ($n!$)**
   - **Recurrence**: $T(n) = T(n - 1) + O(1)$ with base case $0! = 1! = 1$.
   - **Complexity**: Time $\Theta(n)$, Call-Stack Space $\Theta(n)$.
2. **Naive Recursive Fibonacci ($F_n = F_{n-1} + F_{n-2}$)**
   - **Recurrence**: $T(n) = T(n - 1) + T(n - 2) + O(1)$ with base cases $F_0 = 0, F_1 = 1$.
   - **Complexity**: Time $\Theta(\phi^n) \approx O(2^n)$ (where $\phi = \frac{1+\sqrt{5}}{2} \approx 1.618$) due to overlapping subproblems; Call-Stack Space $\Theta(n)$ (maximum tree depth).
3. **Tower of Hanoi ($n$ disks from `src` to `dest` using `aux`)**
   - **Algorithm**:
     1. If $n == 1$, move disk 1 from `src` to `dest` and return.
     2. Recursively move top $n - 1$ disks from `src` to `aux` using `dest`.
     3. Move disk $n$ from `src` to `dest`.
     4. Recursively move $n - 1$ disks from `aux` to `dest` using `src`.
   - **Recurrence**: $T(n) = 2T(n - 1) + 1 \implies T(n) = 2^n - 1$ moves.
   - **Complexity**: Time $\Theta(2^n)$, Call-Stack Space $\Theta(n)$.
4. **Recursive Binary Search (Sorted Array `arr[low..high]`)**
   - **Algorithm**:
     1. Base Case 1 (Empty Range): If `low > high`, return `-1` (target not found).
     2. Compute `mid = low + (high - low) / 2`.
     3. Base Case 2 (Found): If `arr[mid] == target`, return `mid`.
     4. Recursive Case: If `target < arr[mid]`, return `bsearch(arr, low, mid - 1, target)`; else return `bsearch(arr, mid + 1, high, target)`.
   - **Recurrence**: $T(n) = T(n / 2) + O(1)$.
   - **Complexity**: Time $O(\log n)$, Call-Stack Space $O(\log n)$ (or $O(1)$ with tail-call optimization).

---

### 4.2 Tail Recursion vs. Non-Tail Recursion & Converting Recursion to an Explicit Stack

#### Why Standard Recursive Factorial Is NOT Tail-Recursive
In `return n * factorial(n - 1);`, the caller **cannot** return immediately when `factorial(n - 1)` finishes—it must stay alive on the call stack holding its local variable `n` so it can multiply `n` by the child's result!

By introducing an **accumulator parameter** `acc` initialized to `1`, we perform the multiplication *before* the recursive call:

```c
long long factorial_tail_helper(int n, long long acc) {
    if (n <= 1) return acc;
    return factorial_tail_helper(n - 1, (long long)n * acc); /* Pure tail call! */
}
```

#### How Tail-Call Elimination (TCE) Reuses a Single Stack Frame
```
  Non-Tail factorial(4):                Tail-Recursive factorial_tail_helper(4, 1):
  Must keep 4 frames alive!             Compiler overwrites parameters in-place (O(1) space):
  +--------------------+                +----------------------------------+
  | fact(4): waits 4*? |                | Frame 0: n = 4, acc = 1          |
  | fact(3): waits 3*? |                |   -> overwritten with n=3, acc=4 |
  | fact(2): waits 2*? |                |   -> overwritten with n=2, acc=12|
  | fact(1): returns 1 |                |   -> overwritten with n=1, acc=24|
  +--------------------+                +----------------------------------+
```

#### Converting Non-Tail Recursion to an Explicit Stack
When a recursive function has multiple recursive calls (like Tower of Hanoi or Tree Traversals) or post-call work, we eliminate recursion systematically:
1. Define a frame `struct` containing the function parameters and a `stage` state variable.
2. Push the initial call frame onto an explicit stack.
3. Loop `while (!isEmpty(stack))`:
   - Pop or inspect the top frame.
   - To execute sub-tasks $A$, then $B$, then $C$ sequentially, **push them in reverse order ($C$, then $B$, then $A$)** onto the LIFO stack so that $A$ is popped and executed first!

---

### 4.3 Complete C11 Implementation: Call-Stack Trace, Tail Recursion, and Explicit-Stack Hanoi

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

/* =========================================================================
 * 1. RECURSIVE, TAIL-RECURSIVE, AND EXPLICIT-STACK FACTORIAL
 * ========================================================================= */
long long factorial_recursive(int n, int depth) {
    printf("  %*s-> CALL factorial_recursive(n=%d) [Frame Depth = %d]\n",
           depth * 2, "", n, depth);
    if (n <= 1) {
        printf("  %*s<- BASE CASE reached (n=%d), returning 1\n",
               depth * 2, "", n);
        return 1;
    }
    long long sub_result = factorial_recursive(n - 1, depth + 1);
    long long total = (long long)n * sub_result;
    printf("  %*s<- UNWIND factorial_recursive(n=%d): %d * %lld = %lld\n",
           depth * 2, "", n, n, sub_result, total);
    return total;
}

long long factorial_tail_helper(int n, long long acc) {
    if (n <= 1) {
        return acc;
    }
    /* Pure tail call: no pending multiplication after the recursive call returns */
    return factorial_tail_helper(n - 1, (long long)n * acc);
}

long long factorial_tail(int n) {
    return factorial_tail_helper(n, 1);
}

/* Explicit Activation Record for simulating recursion iteratively */
typedef struct {
    int n;
    int stage; /* 0 = descending toward base case, 1 = multiplying after return */
} FactFrame;

long long factorial_explicit_stack(int n) {
    FactFrame stack[64];
    int top = -1;
    long long return_reg = 1; /* Simulates CPU return-value register (RAX) */

    stack[++top] = (FactFrame){ .n = n, .stage = 0 };

    while (top >= 0) {
        FactFrame *curr = &stack[top];
        if (curr->stage == 0) {
            if (curr->n <= 1) {
                return_reg = 1;
                top--; /* Pop base-case frame */
            } else {
                curr->stage = 1; /* Pause current frame; resume after child returns */
                stack[++top] = (FactFrame){ .n = curr->n - 1, .stage = 0 };
            }
        } else {
            /* Child call has returned in return_reg; resume post-call multiplication */
            return_reg = (long long)curr->n * return_reg;
            top--; /* Pop completed frame */
        }
    }
    return return_reg;
}

/* =========================================================================
 * 2. EXPLICIT-STACK ITERATIVE TOWER OF HANOI
 * ========================================================================= */
typedef struct {
    int disks;
    char src;
    char aux;
    char dest;
    int stage; /* 0: decompose subproblems, 1: print single move */
} HanoiFrame;

void hanoi_explicit_stack(int n, char src, char aux, char dest) {
    HanoiFrame stack[64];
    int top = -1;
    int move_num = 0;

    /* Push initial call hanoi(n, src, aux, dest) */
    stack[++top] = (HanoiFrame){ n, src, aux, dest, 0 };

    while (top >= 0) {
        HanoiFrame frame = stack[top--]; /* Pop current activation record */

        if (frame.disks == 1) {
            printf("  Move #%d: Disk 1 from %c -> %c\n", ++move_num, frame.src, frame.dest);
        } else if (frame.stage == 0) {
            /*
             * Recursive order is:
             *   Step A: hanoi(n-1, src, dest, aux)
             *   Step B: print move disk n from src -> dest
             *   Step C: hanoi(n-1, aux, src, dest)
             * Because a Stack is LIFO, we push Step C, then Step B, then Step A!
             */
            stack[++top] = (HanoiFrame){ frame.disks - 1, frame.aux, frame.src, frame.dest, 0 };
            stack[++top] = (HanoiFrame){ frame.disks,     frame.src, frame.aux, frame.dest, 1 };
            stack[++top] = (HanoiFrame){ frame.disks - 1, frame.src, frame.dest, frame.aux, 0 };
        } else if (frame.stage == 1) {
            printf("  Move #%d: Disk %d from %c -> %c\n",
                   ++move_num, frame.disks, frame.src, frame.dest);
        }
    }
}

/* =========================================================================
 * 3. INDIRECT (MUTUAL) RECURSION DEMO
 * ========================================================================= */
bool is_even_mutual(unsigned int n);
bool is_odd_mutual(unsigned int n);

bool is_even_mutual(unsigned int n) {
    if (n == 0) return true;
    return is_odd_mutual(n - 1);
}

bool is_odd_mutual(unsigned int n) {
    if (n == 0) return false;
    return is_even_mutual(n - 1);
}

int main(void) {
    printf("=== 1. Call-Stack Frame Trace: factorial_recursive(4) ===\n");
    long long f_rec  = factorial_recursive(4, 0);
    long long f_tail = factorial_tail(4);
    long long f_iter = factorial_explicit_stack(4);
    printf("  Results -> Recursive: %lld | Tail-Recursive: %lld | Explicit-Stack: %lld\n\n",
           f_rec, f_tail, f_iter);

    printf("=== 2. Explicit-Stack Tower of Hanoi (3 Disks: A -> C via B) ===\n");
    hanoi_explicit_stack(3, 'A', 'B', 'C');

    printf("\n=== 3. Mutual Recursion Check ===\n");
    printf("  is_even_mutual(7) = %s, is_odd_mutual(7) = %s\n",
           is_even_mutual(7) ? "true" : "false",
           is_odd_mutual(7)  ? "true" : "false");
    return EXIT_SUCCESS;
}
```

#### Program Output:

```text
=== 1. Call-Stack Frame Trace: factorial_recursive(4) ===
  -> CALL factorial_recursive(n=4) [Frame Depth = 0]
    -> CALL factorial_recursive(n=3) [Frame Depth = 1]
      -> CALL factorial_recursive(n=2) [Frame Depth = 2]
        -> CALL factorial_recursive(n=1) [Frame Depth = 3]
        <- BASE CASE reached (n=1), returning 1
      <- UNWIND factorial_recursive(n=2): 2 * 1 = 2
    <- UNWIND factorial_recursive(n=3): 3 * 2 = 6
  <- UNWIND factorial_recursive(n=4): 4 * 6 = 24
  Results -> Recursive: 24 | Tail-Recursive: 24 | Explicit-Stack: 24

=== 2. Explicit-Stack Tower of Hanoi (3 Disks: A -> C via B) ===
  Move #1: Disk 1 from A -> C
  Move #2: Disk 2 from A -> B
  Move #3: Disk 1 from C -> B
  Move #4: Disk 3 from A -> C
  Move #5: Disk 1 from B -> A
  Move #6: Disk 2 from B -> C
  Move #7: Disk 1 from A -> C

=== 3. Mutual Recursion Check ===
  is_even_mutual(7) = false, is_odd_mutual(7) = true
```

---

### 4.4 Matching Parentheses and Brackets (Balanced Delimiters) Algorithm & Full Trace Tables

#### Step-by-Step Algorithm & Pseudocode
- **Purpose**: Given a string `expr` of length $n$, determine whether all `()`, `[]`, and `{}` delimiters are properly matched and nested.
- **Pseudocode**:
  ```text
  Algorithm IsBalancedDelimiters(expr):
      S <- CreateEmptyStack()
      for i <- 0 to length(expr) - 1 do
          ch <- expr[i]
          if ch is in {'(', '[', '{'} then
              Push(S, ch)
          else if ch is in {')', ']', '}'} then
              if IsEmpty(S) then
                  return FALSE   // Failure Mode 1: Closer on empty stack
              open_ch <- Pop(S)
              if not Matches(open_ch, ch) then
                  return FALSE   // Failure Mode 2: Mismatched closer type
          else
              continue           // Ignore non-delimiter characters
      if not IsEmpty(S) then
          return FALSE           // Failure Mode 3: Unclosed opener(s) remain
      return TRUE
  ```
- **Complexity**:
  - **Time Complexity**: $O(n)$ — each character is scanned once, pushed at most once, and popped at most once.
  - **Auxiliary Space Complexity**: $O(n)$ — in the worst case (e.g., `"(((((((("`), all $n$ characters are pushed onto the stack.

---

#### Trace 1: Valid Nested & Adjacent String `"{[()()]}"`

| Step | Index `i` | Scanned `ch` | Action Taken | Stack Contents (Bottom $\to$ Top) | Status |
| :---: | :---: | :---: | :--- | :--- | :--- |
| 0 | — | — | Initialize empty stack | `[ ]` *(empty)* | Active |
| 1 | 0 | `'{'` | Opening delimiter $\to$ `push('{')` | `['{']` | Active |
| 2 | 1 | `'['` | Opening delimiter $\to$ `push('[')` | `['{', '[']` | Active |
| 3 | 2 | `'('` | Opening delimiter $\to$ `push('(')` | `['{', '[', '(']` | Active |
| 4 | 3 | `')'` | Closing delimiter $\to$ `pop()` yields `'('`; matches `')'` | `['{', '[']` | Match OK |
| 5 | 4 | `'('` | Opening delimiter $\to$ `push('(')` | `['{', '[', '(']` | Active |
| 6 | 5 | `')'` | Closing delimiter $\to$ `pop()` yields `'('`; matches `')'` | `['{', '[']` | Match OK |
| 7 | 6 | `']'` | Closing delimiter $\to$ `pop()` yields `'['`; matches `']'` | `['{']` | Match OK |
| 8 | 7 | `'}'` | Closing delimiter $\to$ `pop()` yields `'{'`; matches `'}'` | `[ ]` *(empty)* | Match OK |
| End | 8 | `'\0'` | End of string; check `isEmpty(S)` $\to$ `true` | `[ ]` *(empty)* | **VALID (`true`)** |

---

#### Trace 2: Invalid String `"[(])"` — Failure Mode: Mismatched Closer

| Step | Index `i` | Scanned `ch` | Action Taken | Stack Contents (Bottom $\to$ Top) | Status |
| :---: | :---: | :---: | :--- | :--- | :--- |
| 0 | — | — | Initialize empty stack | `[ ]` *(empty)* | Active |
| 1 | 0 | `'['` | Opening delimiter $\to$ `push('[')` | `['[']` | Active |
| 2 | 1 | `'('` | Opening delimiter $\to$ `push('(')` | `['[', '(']` | Active |
| 3 | 2 | `']'` | Closing delimiter $\to$ `pop()` yields `'('`; **`'('` does NOT match `']'`**! | `['[']` | **INVALID (`false`): Mismatched Closer at index 2** |

---

#### Trace 3: Invalid String `"((()"` — Failure Mode: Unclosed Opener at End

| Step | Index `i` | Scanned `ch` | Action Taken | Stack Contents (Bottom $\to$ Top) | Status |
| :---: | :---: | :---: | :--- | :--- | :--- |
| 0 | — | — | Initialize empty stack | `[ ]` *(empty)* | Active |
| 1 | 0 | `'('` | Opening delimiter $\to$ `push('(')` | `['(']` | Active |
| 2 | 1 | `'('` | Opening delimiter $\to$ `push('(')` | `['(', '(']` | Active |
| 3 | 2 | `'('` | Opening delimiter $\to$ `push('(')` | `['(', '(', '(']` | Active |
| 4 | 3 | `')'` | Closing delimiter $\to$ `pop()` yields `'('`; matches `')'` | `['(', '(']` | Match OK |
| End | 4 | `'\0'` | End of string; check `isEmpty(S)` $\to$ **`false` (2 openers remain!)** | `['(', '(']` | **INVALID (`false`): Unclosed Openers Remain** |

---

#### Trace 4: Invalid String `"())"` — Failure Mode: Closer on Empty Stack

| Step | Index `i` | Scanned `ch` | Action Taken | Stack Contents (Bottom $\to$ Top) | Status |
| :---: | :---: | :---: | :--- | :--- | :--- |
| 0 | — | — | Initialize empty stack | `[ ]` *(empty)* | Active |
| 1 | 0 | `'('` | Opening delimiter $\to$ `push('(')` | `['(']` | Active |
| 2 | 1 | `')'` | Closing delimiter $\to$ `pop()` yields `'('`; matches `')'` | `[ ]` *(empty)* | Match OK |
| 3 | 2 | `')'` | Closing delimiter $\to$ **Stack is EMPTY (`top == -1`)!** | `[ ]` *(empty)* | **INVALID (`false`): Extra Closer on Empty Stack at index 2** |

---

### 4.5 Complete C11 Implementation: Multi-Delimiter Balanced Parentheses Checker

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <string.h>

#define MAX_EXPR_LEN 256

typedef struct {
    char bracket; /* Opening delimiter: '(', '[', or '{' */
    int index;    /* 0-based position in the input string */
} DelimToken;

typedef struct {
    DelimToken data[MAX_EXPR_LEN];
    int top;
} DelimStack;

void delim_stack_init(DelimStack *s) {
    s->top = -1;
}

bool delim_stack_is_empty(const DelimStack *s) {
    return s->top == -1;
}

bool delim_stack_push(DelimStack *s, char bracket, int index) {
    if (s->top >= MAX_EXPR_LEN - 1) {
        return false;
    }
    s->top++;
    s->data[s->top].bracket = bracket;
    s->data[s->top].index   = index;
    return true;
}

bool delim_stack_pop(DelimStack *s, DelimToken *out) {
    if (delim_stack_is_empty(s)) {
        return false;
    }
    *out = s->data[(s->top)--];
    return true;
}

static bool is_opening_delim(char ch) {
    return ch == '(' || ch == '[' || ch == '{';
}

static bool is_closing_delim(char ch) {
    return ch == ')' || ch == ']' || ch == '}';
}

static bool is_matching_pair(char open, char close) {
    return (open == '(' && close == ')') ||
           (open == '[' && close == ']') ||
           (open == '{' && close == '}');
}

bool validate_balanced_delimiters(const char *expr) {
    DelimStack stack;
    delim_stack_init(&stack);

    printf("Checking expression: \"%s\"\n", expr);

    for (int i = 0; expr[i] != '\0'; i++) {
        char ch = expr[i];

        if (is_opening_delim(ch)) {
            delim_stack_push(&stack, ch, i);
        } else if (is_closing_delim(ch)) {
            /* Failure Mode 1: Closing delimiter encountered on an empty stack */
            if (delim_stack_is_empty(&stack)) {
                printf("  -> [INVALID: Extra Closer] Unmatched '%c' at index %d (stack is empty).\n\n",
                       ch, i);
                return false;
            }

            DelimToken top_tok;
            delim_stack_pop(&stack, &top_tok);

            /* Failure Mode 2: Closing delimiter does not match the most recent opener */
            if (!is_matching_pair(top_tok.bracket, ch)) {
                printf("  -> [INVALID: Mismatched Pair] Opener '%c' at index %d cannot be closed by '%c' at index %d.\n\n",
                       top_tok.bracket, top_tok.index, ch, i);
                return false;
            }
        }
    }

    /* Failure Mode 3: One or more opening delimiters remain unclosed at end of input */
    if (!delim_stack_is_empty(&stack)) {
        DelimToken unclosed = stack.data[stack.top];
        printf("  -> [INVALID: Unclosed Opener] Opener '%c' at index %d was never closed (%d opener(s) left on stack).\n\n",
               unclosed.bracket, unclosed.index, stack.top + 1);
        return false;
    }

    printf("  -> [VALID] All delimiters are properly matched and nested.\n\n");
    return true;
}

int main(void) {
    const char *test_cases[] = {
        "{[()()]}",                    /* Valid nested & adjacent delimiters */
        "a[i] = (b + {c * (d - e)});", /* Valid code expression */
        "[(])",                        /* Failure Mode 2: Mismatched closer */
        "((()",                        /* Failure Mode 3: Unclosed opener at end */
        "())"                          /* Failure Mode 1: Closer on empty stack */
    };

    size_t num_tests = sizeof(test_cases) / sizeof(test_cases[0]);
    for (size_t t = 0; t < num_tests; t++) {
        validate_balanced_delimiters(test_cases[t]);
    }

    return EXIT_SUCCESS;
}
```

#### Program Output:

```text
Checking expression: "{[()()]}"
  -> [VALID] All delimiters are properly matched and nested.

Checking expression: "a[i] = (b + {c * (d - e)});"
  -> [VALID] All delimiters are properly matched and nested.

Checking expression: "[(])"
  -> [INVALID: Mismatched Pair] Opener '(' at index 1 cannot be closed by ']' at index 2.

Checking expression: "((()"
  -> [INVALID: Unclosed Opener] Opener '(' at index 1 was never closed (2 opener(s) left on stack).

Checking expression: "())"
  -> [INVALID: Extra Closer] Unmatched ')' at index 2 (stack is empty).
```

---

## 5. Comparison Tables & Complexity Summary

### 5.1 Recursion vs. Explicit-Stack Iteration Comparison

| Dimension | Recursive Implementation (System Call Stack) | Iterative Implementation (Explicit Stack / Loop) |
| :--- | :--- | :--- |
| **State Management** | Implicitly managed by compiler & CPU (`CALL`/`RET`, stack frames) | Explicitly managed by programmer (loop variables or custom `Frame` struct stack) |
| **Code Clarity** | Concise and natural for divide-and-conquer, tree, and combinatorial problems | Often more verbose when simulating multi-branch non-tail recursion |
| **Time Overhead per Step** | Function call prologue/epilogue, register saving, frame setup | Faster loop branching and lightweight array/struct indexing |
| **Memory Overhead per Step** | Full ABI stack frame (return address, saved `FP`, registers, padding) | Only the essential state fields defined in the explicit frame struct |
| **Stack Overflow Risk** | **High** for deep inputs ($n \ge 10^5$ can crash the fixed 1–8 MB OS call stack) | **Low** (an explicit dynamic stack on the heap can grow to gigabytes) |

### 5.2 Tail Recursion vs. Non-Tail Recursion Comparison

| Aspect | Non-Tail Recursion (`return n * fact(n - 1)`) | Tail Recursion (`return fact_tail(n - 1, n * acc)`) |
| :--- | :--- | :--- |
| **Position of Recursive Call** | Embedded inside a larger expression; work remains *after* the call returns | The **final action** of the function; returns the callee's value directly |
| **Pending Caller State** | Caller must retain local variables (`n`) until child returns | Caller has zero work remaining; its frame is no longer needed |
| **Compiler Optimization** | Cannot reuse caller's stack frame | Optimizing compiler (`gcc -O2`) performs **Tail-Call Elimination (TCE)**, replacing `call` with a `jmp` |
| **Call-Stack Space Complexity** | $\Theta(d)$ where $d$ is recursion depth | $\Theta(1)$ with TCE enabled ($\Theta(d)$ if compiled at `-O0` without TCE) |

### 5.3 Algorithmic Time and Call-Stack Space Complexity Summary

Let $n$ represent the problem size (input integer $n$, number of disks $n$, array length $n$, or string length $n$).

| Algorithm / Problem | Recurrence Relation | Time Complexity | Call-Stack / Explicit-Stack Space | Notes |
| :--- | :---: | :---: | :---: | :--- |
| **Factorial (Standard Recursive)** | $T(n) = T(n-1) + O(1)$ | $\Theta(n)$ | $\Theta(n)$ | $n$ frames active at base case |
| **Factorial (Tail-Recursive + TCE)** | $T(n) = T(n-1) + O(1)$ | $\Theta(n)$ | $\Theta(1)$ | Reuses single activation record |
| **Fibonacci (Naive Tree Recursive)** | $T(n) = T(n-1) + T(n-2) + O(1)$ | $\Theta(\phi^n) \approx O(2^n)$ | $\Theta(n)$ | Space is tree height $\Theta(n)$, NOT $\Theta(2^n)$ |
| **Fibonacci (Iterative / Memoized)** | — | $\Theta(n)$ | $\Theta(1)$ / $\Theta(n)$ | Avoids recomputing overlapping subproblems |
| **Tower of Hanoi ($n$ disks)** | $T(n) = 2T(n-1) + O(1)$ | $\Theta(2^n)$ | $\Theta(n)$ | Exactly $2^n - 1$ disk moves |
| **Recursive Binary Search** | $T(n) = T(n/2) + O(1)$ | $O(\log n)$ | $O(\log n)$ | Halves search space at each depth |
| **Balanced Delimiters Validation** | Single pass over string | $\Theta(n)$ | $O(n)$ | At most $n$ pushes and $n$ pops |

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Frequent C Bugs in Recursion and Bracket Matching

#### Pitfall 1: Missing or Unreachable Base Case Causing Infinite Recursion (`SIGSEGV`)
If a recursive function fails to handle negative inputs or omits the base case, it pushes stack frames indefinitely until the OS guard page is hit.

```c
/* BUG: Calling fact_buggy(-1) or fact_buggy(0) skips n == 1 and recurses forever! */
long long fact_buggy(int n) {
    if (n == 1) return 1;
    return (long long)n * fact_buggy(n - 1); /* n=0 -> -1 -> -2 -> ... Stack Overflow! */
}

/* FIX: Use n <= 1 (and guard against negative inputs if needed). */
long long fact_fixed(int n) {
    if (n <= 1) return 1;
    return (long long)n * fact_fixed(n - 1);
}
```

#### Pitfall 2: Mistaking `return n * f(n - 1)` for Tail Recursion
A function is **not** tail-recursive merely because the recursive call appears on the `return` line! If any arithmetic operation (`n * ...` or `1 + ...`) wraps the recursive call, the multiplication happens *after* the call returns.

```c
/* BUG (Conceptual): NOT tail-recursive because "+ 1" happens AFTER length_rec returns! */
size_t list_length_not_tail(const Node *head) {
    if (head == NULL) return 0;
    return 1 + list_length_not_tail(head->next); /* Pending addition prevents TCE */
}

/* FIX: Pass an accumulator so the recursive call is the pure return expression. */
size_t list_length_tail(const Node *head, size_t acc) {
    if (head == NULL) return acc;
    return list_length_tail(head->next, acc + 1); /* Pure tail call! */
}
```

#### Pitfall 3: Forgetting the Final `isEmpty(&stack)` Check in Balanced Delimiters
Returning `true` immediately after the `for` loop finishes falsely marks strings like `"((("` or `"{[()"` as balanced!

```c
/* BUG: Returns true at end of loop even when unclosed openers remain on the stack! */
bool is_balanced_buggy(const char *s) {
    DelimStack st;
    delim_stack_init(&st);
    for (int i = 0; s[i] != '\0'; i++) {
        if (s[i] == '(') delim_stack_push(&st, '(', i);
        else if (s[i] == ')') {
            DelimToken tok;
            if (!delim_stack_pop(&st, &tok)) return false;
        }
    }
    return true; /* BUG: Returns true for "(((" because loop ended without error! */
}

/* FIX: Return true ONLY if the stack is completely empty after scanning all chars. */
bool is_balanced_fixed(const char *s) {
    DelimStack st;
    delim_stack_init(&st);
    for (int i = 0; s[i] != '\0'; i++) {
        if (s[i] == '(') delim_stack_push(&st, '(', i);
        else if (s[i] == ')') {
            DelimToken tok;
            if (!delim_stack_pop(&st, &tok)) return false;
        }
    }
    return delim_stack_is_empty(&st); /* Must be empty! */
}
```

#### Pitfall 4: Pushing Sub-Tasks in Forward Order Instead of Reverse Order on an Explicit Stack
When converting a recursive function that performs `StepA(); StepB(); StepC();` into an explicit stack loop, pushing `StepA`, then `StepB`, then `StepC` puts `StepC` on top of the stack, executing them backwards (`C -> B -> A`)!
- **Fix**: Always push sub-tasks in **reverse execution order** (`push(StepC); push(StepB); push(StepA);`) so `StepA` sits at the top of the LIFO stack and executes first.

### 6.2 Boundary Conditions Checklist

- [ ] **Empty String `""` in Balanced Delimiters**: Loop executes 0 times, `isEmpty(&stack)` is `true` $\to$ returns `true` (an empty string is vacuously balanced).
- [ ] **Single Opening Bracket `"["`**: Pushed onto stack; loop ends with `size == 1` $\to$ rejected by final `isEmpty` check.
- [ ] **Single Closing Bracket `"]"`**: Encounters empty stack on first character $\to$ rejected immediately without underflow crash.
- [ ] **Alternating Mismatched Brackets `"[(])"`**: Equal counts of each bracket type, but rejected at `']'` because top of stack is `'('`.
- [ ] **Base Cases $n = 0$ and $n = 1$ in Recursion**: Tested explicitly so recursion terminates without underflowing toward $-\infty$.

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways and Core Invariants

1. **Activation Records**: Every active function invocation owns a stack frame on the system call stack containing its parameters, return address, saved frame pointer, and local variables.
2. **Recursion Space Invariant**: The auxiliary call-stack space of any recursive algorithm is proportional to the **maximum depth of the recursion tree** (not the total number of calls). Thus, naive `fib(n)` takes $\Theta(2^n)$ time but only $\Theta(n)$ stack space.
3. **Tail Recursion & TCE**: A recursive call is tail-recursive only if zero operations follow it in the caller. Optimizing compilers can execute tail recursion in $\Theta(1)$ stack space.
4. **Balanced Delimiters Three Failure Modes**: A delimiter validator must check (1) stack not empty before popping on a closer, (2) popped opener matches the closer's bracket family, and (3) stack is completely empty at the end of the string.

### 7.2 Self-Test Revision Checklist

- [ ] Can I list the contents of a function's activation record on the system call stack?
- [ ] Can I explain why naive recursive `fib(n)` uses $O(n)$ stack space rather than $O(2^n)$ stack space?
- [ ] Can I convert `factorial(n)` into a tail-recursive function using an accumulator parameter?
- [ ] Can I explain why children subproblems must be pushed in **reverse order** when converting recursion to an explicit LIFO stack?
- [ ] Can I trace the delimiter stack step-by-step on `"{[()()]}"`, `"[(])"`, `"((()"`, and `"())"`?

### 7.3 Navigation Links

- **Previous Topic**: [07 — Stacks: Array and Linked-List Implementations](07-Stacks-Array-and-Linked-List.md)
- **Next Topic**: [09 — Expression Conversion and Evaluation](09-Expression-Conversion-and-Evaluation.md)
- **Unit Review**: [10 — Unit 1 Comprehensive Review & Exam Preparation](10-Unit-1-Review.md)
