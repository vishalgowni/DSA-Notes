# C Structures, Pointers, and Dynamic Memory Management
---

## Table of Contents
1. [Introduction](#1-introduction)
   - 1.1 [What Are Structures, Pointers, and Dynamic Memory?](#11-what-are-structures-pointers-and-dynamic-memory)
   - 1.2 [Why Are They Needed in Data Structures?](#12-why-are-they-needed-in-data-structures)
   - 1.3 [Relationship to Prior Concepts](#13-relationship-to-prior-concepts)
   - 1.4 [Learning Objectives](#14-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Core Definitions](#21-core-definitions)
   - 2.2 [Terminology Reference Table](#22-terminology-reference-table)
   - 2.3 [Mathematical Properties and Memory Formulas](#23-mathematical-properties-and-memory-formulas)
   - 2.4 [Conceptual Overview Diagram](#24-conceptual-overview-diagram)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Process Memory Segments (Text, Data, BSS, Heap, Stack)](#31-process-memory-segments-text-data-bss-heap-stack)
   - 3.2 [Structures, `typedef`, and Memory Alignment / Padding](#32-structures-typedef-and-memory-alignment--padding)
   - 3.3 [Structure Pointers and Member Access (`.` vs. `->`)](#33-structure-pointers-and-member-access--vs--)
   - 3.4 [Pointer Basics, `void*`, `NULL`, and Pointer-to-Pointer (`Node**`)](#34-pointer-basics-void-null-and-pointer-to-pointer-node)
   - 3.5 [Self-Referential Structures: The Building Block of Linked Lists](#35-self-referential-structures-the-building-block-of-linked-lists)
   - 3.6 [Static vs. Automatic vs. Dynamic Storage Representations](#36-static-vs-automatic-vs-dynamic-storage-representations)
4. [Operations and Algorithms](#4-operations-and-algorithms)
   - 4.1 [Structure Definition, Initialization, and Pointer Access](#41-structure-definition-initialization-and-pointer-access)
   - 4.2 [Pointer Arithmetic and Double-Pointer (`Node**`) Mutation](#42-pointer-arithmetic-and-double-pointer-node-mutation)
   - 4.3 [Dynamic Allocation with `malloc()` and `calloc()`](#43-dynamic-allocation-with-malloc-and-calloc)
   - 4.4 [Safe Dynamic Resizing with `realloc()` and Deallocation with `free()`](#44-safe-dynamic-resizing-with-realloc-and-deallocation-with-free)
   - 4.5 [Constructing and Linking Self-Referential Heap Nodes](#45-constructing-and-linking-self-referential-heap-nodes)
   - 4.6 [Complete C11 Demonstration Program](#46-complete-c11-demonstration-program)
5. [Complexity Analysis and Comparison Tables](#5-complexity-analysis-and-comparison-tables)
   - 5.1 [Complexity of Memory and Pointer Operations](#51-complexity-of-memory-and-pointer-operations)
   - 5.2 [Storage Duration Comparison: Static vs. Stack vs. Heap](#52-storage-duration-comparison-static-vs-stack-vs-heap)
   - 5.3 [Heap Allocator Comparison: `malloc` vs. `calloc` vs. `realloc`](#53-heap-allocator-comparison-malloc-vs-calloc-vs-realloc)
   - 5.4 [Parameter Passing: By Value vs. Single Pointer vs. Double Pointer](#54-parameter-passing-by-value-vs-single-pointer-vs-double-pointer)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Uninitialized (Wild) Pointers](#61-uninitialized-wild-pointers)
   - 6.2 [Unchecked `NULL` Return from `malloc()` / `calloc()`](#62-unchecked-null-return-from-malloc--calloc)
   - 6.3 [Unsafe `realloc()` Pattern Causing Memory Leaks](#63-unsafe-realloc-pattern-causing-memory-leaks)
   - 6.4 [Memory Leaks via Lost Heap References](#64-memory-leaks-via-lost-heap-references)
   - 6.5 [Dangling Pointers (Use-After-Free) and Double-Free Errors](#65-dangling-pointers-use-after-free-and-double-free-errors)
   - 6.6 [Modifying a Caller's Pointer Using `Node*` Instead of `Node**`](#66-modifying-a-callers-pointer-using-node-instead-of-node)
   - 6.7 [Illegal Self-Referential Structure by Value](#67-illegal-self-referential-structure-by-value)
   - 6.8 [Boundary Conditions Checklist](#68-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)
   - 7.1 [Key Takeaways](#71-key-takeaways)
   - 7.2 [Essential Memory Invariants](#72-essential-memory-invariants)
   - 7.3 [Quick Self-Test Checklist](#73-quick-self-test-checklist)
   - 7.4 [Related Study Notes](#74-related-study-notes)

---

## 1. Introduction

### 1.1 What Are Structures, Pointers, and Dynamic Memory?
Every non-trivial data structure in C rests upon a triad of core language mechanisms:
1. **Structures (`struct`)**: Composite user-defined data types that group heterogeneous variables (fields or members) under a single contiguous memory block.
2. **Pointers (`T*`)**: Variables whose values are virtual memory addresses pointing to other objects or functions in memory.
3. **Dynamic Memory Management (`malloc`, `calloc`, `realloc`, `free`)**: Runtime manual allocation and deallocation of memory blocks on the **heap**, enabling data structures whose size and lifetime are not known at compile time.

### 1.2 Why Are They Needed in Data Structures?
Primitive types (`int`, `double`, `char`) and fixed-size arrays suffer from three fundamental limitations when modeling real-world computational problems:
- **Homogeneity Constraint**: Arrays can only store elements of the *same* data type. A student record or a graph vertex requires bundling an integer key, a string label, and one or more links into a single logical unit.
- **Compile-Time Size Rigidity**: Static and stack-allocated arrays require fixed capacities or stack-frame bounds. Over-allocating wastes memory; under-allocating causes buffer overflows.
- **Function-Scoped Lifetimes**: Automatic variables created inside a helper function are destroyed the instant that function returns. Dynamic data structures (linked lists, trees, hash tables, graphs) must create nodes inside helper functions that persist across the program's execution until explicitly destroyed.

By combining **structures** (to bundle data and links), **pointers** (to connect non-contiguous nodes in memory), and **dynamic heap allocation** (to create and destroy nodes on demand), we unlock **self-referential structures**—the foundation of all linked data structures.

### 1.3 Relationship to Prior Concepts
- **Variables and Memory Addresses**: Just as an `int x = 42;` occupies 4 bytes at some hardware address (e.g., `0x7ffc1000`), a pointer stores that exact address as a typed scalar value.
- **Arrays and Contiguous Indexing**: In C, array indexing `arr[i]` is syntactic sugar for pointer arithmetic `*(arr + i)`. Understanding pointer arithmetic bridges static arrays to dynamic arrays and linked structures.
- **Function Call Stack**: Passing variables by value copies bytes onto a callee's stack frame. Pointers allow functions to mutate caller state in place and return newly allocated heap structures without expensive copying.

### 1.4 Learning Objectives
After mastering this module, you will be able to:
1. Define composite types using `struct` and `typedef`, and calculate exact memory footprints including hardware alignment padding.
2. Distinguish between the direct member access operator (`.`) and the structure pointer indirection operator (`->`).
3. Trace single pointers (`T*`), double pointers (`T**`), generic pointers (`void*`), and `NULL` pointers using address-level ASCII diagrams.
4. Explain the five segments of a C process's virtual address space (Text, Data, BSS, Heap, Stack).
5. Write memory-safe C11 code using `malloc()`, `calloc()`, `realloc()`, and `free()` with proper allocation-failure handling.
6. Construct self-referential `Node` structures and diagnose memory leaks, dangling pointers, wild pointers, and double-free bugs.

---

## 2. Definitions and Terminology

### 2.1 Core Definitions
- **Structure (`struct`)**: An aggregate type consisting of a sequence of named members allocated in the exact order of declaration, potentially separated by compiler-inserted alignment padding bytes.
- **Pointer**: A strongly typed variable that holds the starting memory address of another object (or `NULL`). On a 64-bit architecture, all object pointers (`char*`, `int*`, `struct Node*`, `void*`) occupy exactly $8\text{ bytes}$ ($64\text{ bits}$), regardless of the size of the pointed-to object.
- **Self-Referential Structure**: A structure that contains at least one member that is a **pointer to a structure of the same type** (e.g., `struct Node *next;`).
- **Heap (Free Store)**: A pool of process memory managed at runtime via `<stdlib.h>` allocation functions (`malloc`, `calloc`, `realloc`, `free`) where blocks persist until explicitly freed or the process terminates.

### 2.2 Terminology Reference Table

| Term | Formal Definition |
| :--- | :--- |
| **`struct` Tag** | The identifier following the `struct` keyword (e.g., `struct Node`) used to reference the structure type before or inside its own body. |
| **`typedef` Alias** | A type synonym created via `typedef` so programmers can write `Node` instead of `struct Node`. |
| **Alignment ($A$)** | A hardware requirement that an object of size $k$ be stored at a memory address divisible by $A$ (typically $A \in \{1, 2, 4, 8\}$ bytes). |
| **Structure Padding** | Unused bytes inserted by the compiler between structure members or at the end of a structure to satisfy member alignment requirements. |
| **Dereferencing (`*ptr`)** | Following the memory address stored in `ptr` to read or modify the target lvalue stored at that address. |
| **Arrow Operator (`->`)** | Shorthand operator `ptr->member` that dereferences a structure pointer and accesses a member, equivalent to `(*ptr).member`. |
| **Null Pointer (`NULL`)** | A special reserved pointer constant (`(void*)0`) guaranteed not to point to any valid object or function. |
| **Void Pointer (`void*`)** | A generic object pointer type that can hold the address of any data object but cannot be dereferenced or used in pointer arithmetic without casting. |
| **Wild Pointer** | An uninitialized pointer containing indeterminate garbage bits from the stack or heap. |
| **Dangling Pointer** | A pointer that still holds the address of a memory block that has already been deallocated (`free`d) or gone out of scope. |
| **Memory Leak** | Allocated heap memory that is no longer reachable by any live pointer in the program and was never released via `free()`. |
| **Double Pointer (`T**`)** | A pointer that stores the memory address of another pointer variable (`T*`), required when a function must modify the caller's pointer itself. |

### 2.3 Mathematical Properties and Memory Formulas

#### 1. Pointer Arithmetic Scaling Formula
Given a typed pointer $p$ of type `T*` stored at base address $\text{addr}(p)$ and an integer offset $k \in \mathbb{Z}$, the address resulting from pointer addition $p + k$ is scaled by the byte size of type `T`:

$$\text{addr}(p + k) = \text{addr}(p) + k \times \operatorname{sizeof}(T)$$

Similarly, subtracting two pointers $p_2$ and $p_1$ pointing into the same array yields the number of elements (`ptrdiff_t`) between them:

$$p_2 - p_1 = \frac{\text{addr}(p_2) - \text{addr}(p_1)}{\operatorname{sizeof}(T)}$$

#### 2. Memory Alignment Invariant
Every primitive type $T_i$ has an alignment requirement $A_i = \operatorname{alignof}(T_i)$ (usually equal to $\operatorname{sizeof}(T_i)$ on 64-bit platforms). A memory address $M$ is properly aligned for type $T_i$ if and only if:

$$M \bmod A_i = 0$$

#### 3. Structure Size and Tail Padding Formula
Let a structure $S$ have members $m_1, m_2, \dots, m_k$ with alignments $A_1, A_2, \dots, A_k$. The overall alignment of the structure is the maximum alignment among its members:

$$A_S = \max_{1 \le i \le k} A_i$$

If the raw byte offset after placing member $m_k$ is $R$ bytes, the total size of the structure $\operatorname{sizeof}(S)$ is rounded up to the nearest multiple of $A_S$:

$$\operatorname{sizeof}(S) = \left\lceil \frac{R}{A_S} \right\rceil \times A_S$$

### 2.4 Conceptual Overview Diagram

```
+-------------------------------------------------------------------------+
|                    STACK FRAME (main / helper)                          |
|                                                                         |
|   headPtr (Node**)             head (Node*)                             |
|   Address: 0x7ffc0010          Address: 0x7ffc0020                      |
|  +--------------------+       +--------------------+                    |
|  |     0x7ffc0020     | ----> |     0x55a01000     |                    |
|  +--------------------+       +---------+----------+                    |
+-----------------------------------------|-------------------------------+
                                          |
                                          | Points across segments to Heap
                                          v
+-------------------------------------------------------------------------+
|                    HEAP SEGMENT (Dynamically Allocated)                 |
|                                                                         |
|   Node 1 (Address: 0x55a01000)         Node 2 (Address: 0x55a01040)     |
|  +------------+---------------+       +------------+---------------+    |
|  | data: 100  | next:0x55a01040| ----> | data: 200  | next: NULL    |    |
|  +------------+---------------+       +------------+---------------+    |
+-------------------------------------------------------------------------+
```

---

## 3. Representation and Memory Layout in C

### 3.1 Process Memory Segments (Text, Data, BSS, Heap, Stack)
When the operating system loads a compiled C program into virtual memory, it partitions the process address space into five distinct segments:

```
High Virtual Addresses (e.g., 0x7fffffffffff)
+---------------------------------------------------+
| Command-Line Arguments (argv) & Environment (envp)|
+---------------------------------------------------+
|                   STACK SEGMENT                   |
|  - Automatic local variables, function parameters |
|  - Return addresses & saved registers             |
|  - LIFO growth: Grows DOWNWARD toward low memory  |
+------------------------+--------------------------+
                         |  |
                         v  v
                 (Unmapped Guard Space)
                         ^  ^
                         |  |
+------------------------+--------------------------+
|                    HEAP SEGMENT                   |
|  - Dynamic memory: malloc(), calloc(), realloc()  |
|  - Manual lifetime: persists until free()         |
|  - Grows UPWARD toward high memory                |
+---------------------------------------------------+
|            BSS SEGMENT (Uninitialized Data)       |
|  - Zero-initialized global and static variables   |
|  - e.g., static int count;                        |
+---------------------------------------------------+
|            DATA SEGMENT (Initialized Data)        |
|  - Explicitly initialized global/static variables |
|  - e.g., int max_nodes = 1024;                    |
+---------------------------------------------------+
|                 TEXT / CODE SEGMENT               |
|  - Read-only compiled machine instructions        |
|  - String literals (e.g., "Hello")                |
+---------------------------------------------------+
Low Virtual Addresses (e.g., 0x00400000)
```

### 3.2 Structures, `typedef`, and Memory Alignment / Padding
In C, we declare structures using `struct` and commonly wrap them in a `typedef` so the type can be referenced cleanly.

```c
/* Standard idiom supporting both self-reference and clean alias */
typedef struct Student {
    int id;             /* 4 bytes */
    char grade;         /* 1 byte  */
    double gpa;         /* 8 bytes */
} Student;
```

#### Why `sizeof(Student)` is 16 Bytes (Not 13 Bytes)
On a 64-bit system (`sizeof(int) = 4`, `sizeof(char) = 1`, `sizeof(double) = 8`), the CPU reads memory in 8-byte words. If `gpa` (`double`, $A = 8$) started at offset 5, fetching it would require two separate bus cycles across an 8-byte boundary. Therefore, the compiler inserts **3 bytes of internal padding** after `grade`:

```
Base Address: 0x1000
Offset:   0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
        +---+---+---+---+---+-----------+---+---+---+---+---+---+---+---+
Field:  |      id       |grd|  PADDING  |              gpa              |
        |   (4 bytes)   |(1)| (3 bytes) |           (8 bytes)           |
        +---+---+---+---+---+-----------+---+---+---+---+---+---+---+---+
```

Compare two orderings of the exact same fields when an extra `char section` is added:

```c
/* Suboptimal Field Ordering: 24 bytes total */
typedef struct PaddedRecord {
    char grade;      /* Offset 0 (1 byte) + 7 bytes padding */
    double gpa;      /* Offset 8 (8 bytes)                  */
    int id;          /* Offset 16 (4 bytes)                 */
    char section;    /* Offset 20 (1 byte) + 3 bytes tail   */
} PaddedRecord;      /* Total = 24 bytes                    */

/* Optimal Field Ordering (Largest-to-Smallest Alignment): 16 bytes total */
typedef struct CompactRecord {
    double gpa;      /* Offset 0 (8 bytes)                  */
    int id;          /* Offset 8 (4 bytes)                  */
    char grade;      /* Offset 12 (1 byte)                  */
    char section;    /* Offset 13 (1 byte) + 2 bytes tail   */
} CompactRecord;     /* Total = 16 bytes (33% memory saved) */
```

#### Nested Structures
A structure may contain another full structure by value or by pointer:

```c
typedef struct Date {
    int day;
    int month;
    int year;
} Date;

typedef struct Employee {
    int emp_id;
    Date join_date;  /* Nested structure embedded contiguously */
} Employee;
```

### 3.3 Structure Pointers and Member Access (`.` vs. `->`)
C provides two operators for accessing structure fields depending on whether you hold the **structure value** or a **pointer to the structure**:
1. **Direct Member Access (`.`)**: Used on a structure variable or dereferenced pointer: `s.id = 101;` or `(*ptr).id = 101;`.
2. **Indirect Member Access (`->`)**: Used directly on a structure pointer: `ptr->id = 101;`.

> [!IMPORTANT]
> Operator Precedence Trap: The dot operator (`.`) has **higher precedence** than the dereference operator (`*`). Writing `*ptr.id` is parsed by the compiler as `*(ptr.id)`, which fails to compile because `ptr` is a pointer, not a struct! Parentheses in `(*ptr).id` are mandatory, which is why C provides the cleaner `ptr->id` operator.

```
Stack Variable s (struct Student)          Stack Pointer ptr (Student*)
Address: 0x7ffc0100                        Address: 0x7ffc0200
+--------+---------+----------+            +------------------+
| id:101 | grade:A | gpa:3.95 | <--------- |    0x7ffc0100    |
+--------+---------+----------+            +------------------+
Access via: s.id                           Access via: ptr->id  or  (*ptr).id
```

### 3.4 Pointer Basics, `void*`, `NULL`, and Pointer-to-Pointer (`Node**`)

#### 1. Single Pointer (`int*`), Double Pointer (`int**`)
A pointer has its own memory address on the stack and holds the address of its target:

```
Variable:     int x = 42;          int *p = &x;           int **pp = &p;
Address:      0x7ffc1000           0x7ffc1008             0x7ffc1010
            +------------+       +------------+         +------------+
Value:      |     42     | <---- | 0x7ffc1000 | <------ | 0x7ffc1008 |
            +------------+       +------------+         +------------+
Expression:   x == 42              *p == 42               **pp == 42
              &x == 0x7ffc1000     p == 0x7ffc1000        *pp == 0x7ffc1000
```

#### 2. Generic Pointer (`void*`)
In C11, `void*` is the universal object pointer returned by `malloc()`, `calloc()`, and `realloc()`.
- **Implicit Conversion in C**: Unlike C++, C allows implicit conversion between `void*` and any typed object pointer (`Node *n = malloc(sizeof(*n));` requires no cast).
- **Restrictions**: You **cannot** dereference a `void*` (`*vptr` is illegal because the compiler does not know how many bytes to read) and standard C **forbids** pointer arithmetic on `void*` (`sizeof(void)` is undefined).

#### 3. Null Pointer (`NULL`)
`NULL` (defined in `<stddef.h>`, `<stdio.h>`, `<stdlib.h>`) represents a pointer that points to **nothing**. In linked data structures, `NULL` marks:
- An empty list (`head == NULL`)
- The end of a chain (`tail->next == NULL`)
- A failed heap allocation (`ptr == NULL`)

### 3.5 Self-Referential Structures: The Building Block of Linked Lists
A **self-referential structure** contains a pointer member that points to another instance of the same structure type:

```c
typedef struct Node {
    int data;           /* Payload (4 bytes + 4 bytes alignment padding) */
    struct Node *next;  /* Link pointer to next Node (8 bytes on 64-bit) */
} Node;
```

Why must we write `struct Node *next;` inside the body instead of `Node *next;`?
Because the `typedef` alias `Node` is only completed at the closing semicolon `} Node;`. Inside the curly braces, only the structure tag `struct Node` is in scope.

```
Memory Layout of One 64-Bit Node (sizeof(Node) == 16 bytes):
Base Address: 0x55a02000
Offset:   0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
        +---+---+---+---+---------------+---+---+---+---+---+---+---+---+
Field:  |   data (int)  |    PADDING    |      next (struct Node*)      |
        |    4 bytes    |    4 bytes    |            8 bytes            |
        +---+---+---+---+---------------+---+---+---+---+---+---+---+---+
```

### 3.6 Static vs. Automatic vs. Dynamic Storage Representations
- **Static Storage (`static Node pool[100];`)**: Pre-allocated in BSS/Data segment before `main()` runs. Fast and fragmentation-free, but capped at a hard compile-time limit.
- **Automatic Storage (`Node local_node;`)**: Allocated on the call stack upon block entry and automatically popped when the block exits. Extremely fast ($O(1)$ stack-pointer shift), but cannot outlive the creating function.
- **Dynamic Heap Storage (`Node *n = malloc(sizeof(Node));`)**: Allocated at runtime from the heap. Survives across function returns and scales up to available system RAM, at the cost of allocator bookkeeping overhead and manual `free()` responsibility.

---

## 4. Operations and Algorithms

### 4.1 Structure Definition, Initialization, and Pointer Access

#### 1. Purpose
Define a composite structure, initialize it using C99/C11 designated initializers, and manipulate its fields through both value semantics (`.`) and pointer semantics (`->`).

#### 2. Preconditions and Assumptions
- Pointer `ptr` must hold the valid address of an allocated `SensorNode` object (`ptr != NULL`) before `ptr->field` is evaluated.

#### 3. Step-by-Step Algorithm
1. Define `typedef struct SensorNode` containing `id`, `reading`, and `status`.
2. Initialize a stack instance `s1` using designated initializers `{ .id = 101, ... }`.
3. Declare a pointer `SensorNode *ptr = &s1` storing the address of `s1`.
4. Read and update fields through `ptr->reading` and `(*ptr).status`.

#### 4. Pseudocode
```
ALGORITHM UpdateSensorViaPointer(ptr, new_reading, new_status)
INPUT: ptr (Pointer to SensorNode), new_reading (double), new_status (char)
OUTPUT: Modifies target SensorNode in place
BEGIN
    IF ptr == NULL THEN
        RETURN
    END IF
    ptr->reading <- new_reading
    (*ptr).status <- new_status
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
BEFORE UpdateSensorViaPointer(&s1, 98.6, 'A'):
Stack: ptr (0x7ffc0010)          Stack: s1 (0x7ffc0020)
+--------------------+          +--------+--------------+------------+
|     0x7ffc0020     | -------> | id:101 | reading:25.0 | status:'I' |
+--------------------+          +--------+--------------+------------+

AFTER UpdateSensorViaPointer(&s1, 98.6, 'A'):
Stack: ptr (0x7ffc0010)          Stack: s1 (0x7ffc0020)
+--------------------+          +--------+--------------+------------+
|     0x7ffc0020     | -------> | id:101 | reading:98.6 | status:'A' |
+--------------------+          +--------+--------------+------------+
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stddef.h>

typedef struct SensorNode {
    double reading;
    int id;
    char status;
} SensorNode;

void update_sensor(SensorNode *ptr, double new_reading, char new_status) {
    if (ptr == NULL) {
        return;
    }
    ptr->reading = new_reading;   /* Equivalent to (*ptr).reading */
    (*ptr).status = new_status;   /* Explicit dereference + dot   */
}

int main(void) {
    SensorNode s1 = { .reading = 25.0, .id = 101, .status = 'I' };
    SensorNode *ptr = &s1;

    printf("sizeof(SensorNode) = %zu bytes\n", sizeof(SensorNode));
    printf("Offsets -> reading: %zu, id: %zu, status: %zu\n",
           offsetof(SensorNode, reading),
           offsetof(SensorNode, id),
           offsetof(SensorNode, status));

    printf("Before: id=%d, reading=%.1f, status=%c\n",
           s1.id, s1.reading, s1.status);

    update_sensor(ptr, 98.6, 'A');

    printf("After : id=%d, reading=%.1f, status=%c\n",
           ptr->id, ptr->reading, ptr->status);
    return 0;
}
```

#### Program Output:
```
sizeof(SensorNode) = 16 bytes
Offsets -> reading: 0, id: 8, status: 12
Before: id=101, reading=25.0, status=I
After : id=101, reading=98.6, status=A
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: $\mathcal{O}(1)$ — Direct memory offset calculation (`base_address + offset`) executes in a single CPU instruction.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$ — Only a single 8-byte pointer parameter is placed on the stack.

#### 8. Edge Cases and Common Mistakes
- Passing a `NULL` pointer and evaluating `ptr->reading` triggers an immediate segmentation fault (`SIGSEGV`).
- Passing `SensorNode s` by value instead of `SensorNode *ptr` mutates only the local stack copy and leaves the caller's struct unchanged.

---

### 4.2 Pointer Arithmetic and Double-Pointer (`Node**`) Mutation

#### 1. Purpose
Traverse contiguous memory using typed pointer arithmetic (`p + i`, `p++`) and redirect a caller's pointer variable using a double pointer (`int**`).

#### 2. Preconditions and Assumptions
- Pointer arithmetic is strictly valid only within the bounds of a single allocated array object (plus one past the last element for comparison).
- Double pointer `pp` must satisfy `pp != NULL`.

#### 3. Step-by-Step Algorithm
1. Point `const int *p` to the first element of an array `arr`.
2. Advance `p` across `n` elements; each `p + i` advances the byte address by $i \times \operatorname{sizeof}(\text{int})$.
3. Pass `&target_ptr` (`int**`) to `redirect_pointer(int **pp, int *new_target)`.
4. Inside `redirect_pointer`, assign `*pp = new_target` to overwrite the caller's pointer variable.

#### 4. Pseudocode
```
ALGORITHM RedirectPointer(pp, new_target)
INPUT: pp (int** - address of caller's pointer), new_target (int*)
OUTPUT: Caller's pointer now points to new_target
BEGIN
    IF pp == NULL THEN
        RETURN
    END IF
    *pp <- new_target
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
BEFORE redirect_pointer(&active, &b):
   pp (0x7ffc0008)          active (0x7ffc0010)        a (0x7ffc0020)    b (0x7ffc0024)
  +---------------+        +-------------------+      +--------------+  +--------------+
  |  0x7ffc0010   | -----> |    0x7ffc0020     | ---> |     100      |  |     200      |
  +---------------+        +-------------------+      +--------------+  +--------------+

AFTER redirect_pointer(&active, &b) executes (*pp = new_target):
   pp (0x7ffc0008)          active (0x7ffc0010)        a (0x7ffc0020)    b (0x7ffc0024)
  +---------------+        +-------------------+      +--------------+  +--------------+
  |  0x7ffc0010   | -----> |    0x7ffc0024     | ---------------------> |     200      |
  +---------------+        +-------------------+      +--------------+  +--------------+
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>

void redirect_pointer(int **pp, int *new_target) {
    if (pp == NULL) {
        return;
    }
    *pp = new_target;
}

int main(void) {
    int arr[4] = {10, 20, 30, 40};
    const int *p = arr; /* Decays to &arr[0] */

    printf("Pointer arithmetic traversal:\n");
    for (size_t i = 0; i < 4; i++) {
        printf("  *(p + %zu) = %d (byte offset: +%td)\n",
               i, *(p + i), (const char *)(p + i) - (const char *)p);
    }

    int a = 100;
    int b = 200;
    int *active = &a;
    printf("Before redirect: *active = %d\n", *active);

    redirect_pointer(&active, &b);
    printf("After redirect : *active = %d\n", *active);
    return 0;
}
```

#### Program Output:
```
Pointer arithmetic traversal:
  *(p + 0) = 10 (byte offset: +0)
  *(p + 1) = 20 (byte offset: +4)
  *(p + 2) = 30 (byte offset: +8)
  *(p + 3) = 40 (byte offset: +12)
Before redirect: *active = 100
After redirect : *active = 200
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: $\mathcal{O}(1)$ for pointer redirection (`*pp = new_target`); $\mathcal{O}(n)$ for array traversal of $n$ elements.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$.

#### 8. Edge Cases and Common Mistakes
- Assuming `p + 1` adds 1 byte instead of `sizeof(*p)` bytes.
- Dereferencing one-past-the-end (`*(arr + 4)`): C allows computing `arr + 4` as a loop sentinel, but dereferencing it invokes undefined behavior.

---

### 4.3 Dynamic Allocation with `malloc()` and `calloc()`

#### 1. Purpose
Allocate dynamic memory blocks on the heap using `malloc()` (uninitialized bytes) and `calloc()` (zero-initialized bytes with overflow checking).

#### 2. Preconditions and Assumptions
- Requested element count $n > 0$.
- Caller must check whether the returned pointer is `NULL` before dereferencing.

#### 3. Step-by-Step Algorithm
1. Call `malloc(n * sizeof(*arr))` to request $n \times \operatorname{sizeof}(T)$ uninitialized bytes from the heap.
2. Verify `arr != NULL`. Populate each element explicitly before reading.
3. Call `calloc(n, sizeof(*zeroed))` to request $n$ elements cleared to all-bits-zero.
4. Verify `zeroed != NULL`.
5. Release both heap blocks using `free()` when finished.

#### 4. Pseudocode
```
ALGORITHM AllocateZeroedArray(n, element_size)
INPUT: n (size_t count), element_size (size_t bytes)
OUTPUT: Pointer to zero-initialized heap array or NULL
BEGIN
    IF n == 0 OR element_size == 0 THEN
        RETURN NULL
    END IF
    ptr <- calloc(n, element_size)
    IF ptr == NULL THEN
        PRINT "Error: Out of heap memory"
        RETURN NULL
    END IF
    RETURN ptr
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
1. malloc(4 * sizeof(int)) -> Uninitialized garbage bytes on Heap:
   Stack: m_arr                 Heap (0x55a01000, 16 bytes)
  +--------------+             +-----+-----+-----+-----+
  |  0x55a01000  | ----------> | ??? | ??? | ??? | ??? | (Indeterminate values)
  +--------------+             +-----+-----+-----+-----+

2. calloc(4, sizeof(int)) -> Zero-initialized bytes on Heap:
   Stack: c_arr                 Heap (0x55a01040, 16 bytes)
  +--------------+             +-----+-----+-----+-----+
  |  0x55a01040  | ----------> |  0  |  0  |  0  |  0  | (Guaranteed 0)
  +--------------+             +-----+-----+-----+-----+
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>

int *create_sequence(size_t n, int start, int step) {
    if (n == 0) {
        return NULL;
    }
    /* Using sizeof(*arr) automatically tracks type changes */
    int *arr = malloc(n * sizeof(*arr));
    if (arr == NULL) {
        return NULL;
    }
    for (size_t i = 0; i < n; i++) {
        arr[i] = start + (int)i * step;
    }
    return arr;
}

int main(void) {
    size_t n = 4;
    int *seq = create_sequence(n, 10, 5);
    int *counts = calloc(n, sizeof(*counts));

    if (seq == NULL || counts == NULL) {
        free(seq);
        free(counts);
        fprintf(stderr, "Allocation failed\n");
        return EXIT_FAILURE;
    }

    printf("malloc initialized sequence: ");
    for (size_t i = 0; i < n; i++) {
        printf("%d ", seq[i]);
    }
    printf("\ncalloc zeroed array        : ");
    for (size_t i = 0; i < n; i++) {
        printf("%d ", counts[i]);
    }
    printf("\n");

    free(seq);
    seq = NULL;
    free(counts);
    counts = NULL;
    return 0;
}
```

#### Program Output:
```
malloc initialized sequence: 10 15 20 25 
calloc zeroed array        : 0 0 0 0 
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**:
  - `malloc(B)`: $\mathcal{O}(1)$ amortized (depends on heap free-list search, does not touch payload bytes).
  - `calloc(n, s)`: $\mathcal{O}(n \times s)$ worst-case to zero-fill memory (often optimized to $\mathcal{O}(1)$ via OS copy-on-write zero pages for large allocations).
- **Auxiliary Space Complexity**: $\mathcal{O}(n)$ heap space allocated.

#### 8. Edge Cases and Common Mistakes
- Reading from a `malloc()` block before writing to it reads indeterminate garbage (Undefined Behavior).
- Integer overflow in `malloc(n * sizeof(int))`: if `n` is huge, `n * sizeof(int)` can wrap around modulo $2^{64}$, allocating a tiny buffer. `calloc(n, sizeof(int))` checks for multiplication overflow automatically.

---

### 4.4 Safe Dynamic Resizing with `realloc()` and Deallocation with `free()`

#### 1. Purpose
Expand or shrink an existing dynamically allocated memory block safely using `realloc()` without losing the original pointer if allocation fails, and release memory cleanly with `free()`.

#### 2. Preconditions and Assumptions
- The pointer passed to `realloc(ptr, new_size)` and `free(ptr)` must either be `NULL` or a pointer previously returned by `malloc`, `calloc`, or `realloc` that has not yet been freed.

#### 3. Step-by-Step Algorithm
1. Assign the result of `realloc(*arr_ptr, new_cap * sizeof(int))` to a **temporary pointer** `int *temp`.
2. Check if `temp == NULL`.
   - If `temp == NULL`, return `false` immediately; `*arr_ptr` still points to the intact original heap block so the caller can clean it up without leaking memory.
3. If `temp != NULL`, update `*arr_ptr = temp` and return `true`.
4. When done, call `free(*arr_ptr)` and set `*arr_ptr = NULL`.

#### 4. Pseudocode
```
ALGORITHM SafeResizeIntArray(arr_ptr, new_capacity)
INPUT: arr_ptr (int**), new_capacity (size_t)
OUTPUT: true if resize succeeded, false if allocation failed (original array preserved)
BEGIN
    IF arr_ptr == NULL OR new_capacity == 0 THEN
        RETURN false
    END IF
    temp <- realloc(*arr_ptr, new_capacity * sizeof(int))
    IF temp == NULL THEN
        /* Original block at *arr_ptr is untouched and NOT leaked */
        RETURN false
    END IF
    *arr_ptr <- temp
    RETURN true
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
CASE A: In-Place Expansion (Contiguous free space available after block)
BEFORE: [ 10 | 20 ] (at 0x55a01000)
AFTER : [ 10 | 20 | ?? | ?? ] (still at 0x55a01000)

CASE B: Relocated Expansion (Block moved to new heap address; old block auto-freed)
BEFORE:
  *arr_ptr ---> [ 10 | 20 ] (at 0x55a01000) [Allocated neighbor blocks here]

AFTER temp = realloc(*arr_ptr, 4 * sizeof(int)):
  Old Block (0x55a01000): Automatically freed by realloc()!
  New Block (0x55a02080): [ 10 | 20 | ?? | ?? ] (Copied first 2 ints)
  *arr_ptr = temp ------> Points to 0x55a02080
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

bool safe_resize(int **arr_ptr, size_t new_capacity) {
    if (arr_ptr == NULL || new_capacity == 0) {
        return false;
    }
    int *temp = realloc(*arr_ptr, new_capacity * sizeof(**arr_ptr));
    if (temp == NULL) {
        return false; /* Original *arr_ptr remains valid */
    }
    *arr_ptr = temp;
    return true;
}

void safe_free(void **ptr) {
    if (ptr != NULL && *ptr != NULL) {
        free(*ptr);
        *ptr = NULL;
    }
}

int main(void) {
    size_t cap = 2;
    int *data = malloc(cap * sizeof(*data));
    if (data == NULL) {
        return EXIT_FAILURE;
    }
    data[0] = 100;
    data[1] = 200;

    printf("Initial capacity %zu: [%d, %d]\n", cap, data[0], data[1]);

    if (safe_resize(&data, 5)) {
        cap = 5;
        data[2] = 300;
        data[3] = 400;
        data[4] = 500;
    }

    printf("Resized capacity %zu: [", cap);
    for (size_t i = 0; i < cap; i++) {
        printf("%d%s", data[i], (i + 1 < cap) ? ", " : "]\n");
    }

    safe_free((void **)&data);
    printf("After safe_free, data == %s\n", (data == NULL) ? "NULL" : "non-NULL");
    return 0;
}
```

#### Program Output:
```
Initial capacity 2: [100, 200]
Resized capacity 5: [100, 200, 300, 400, 500]
After safe_free, data == NULL
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**:
  - Best Case (in-place expansion or shrinking): $\mathcal{O}(1)$.
  - Worst Case (relocation to a new block of size $n$): $\mathcal{O}(n)$ to copy existing bytes.
- **Auxiliary Space Complexity**: $\mathcal{O}(n)$ temporary peak memory during block relocation.

#### 8. Edge Cases and Common Mistakes
- **Any pointers into the old block become dangling** if `realloc()` relocates the block to a new address! Never keep separate raw pointers to elements inside a `realloc`-managed array; store integer indices instead.
- `realloc(NULL, size)` behaves identically to `malloc(size)`.

---

### 4.5 Constructing and Linking Self-Referential Heap Nodes

#### 1. Purpose
Create independent self-referential `Node` structures on the heap, wire their `next` pointers to form a chain, traverse the chain, and deallocate every node without memory leaks.

#### 2. Preconditions and Assumptions
- System heap has sufficient memory for node allocations.

#### 3. Step-by-Step Algorithm
1. Define `create_node(int value)`:
   - Allocate `Node *new_node = malloc(sizeof(*new_node))`.
   - If `new_node == NULL`, return `NULL`.
   - Initialize `new_node->data = value` and `new_node->next = NULL`.
   - Return `new_node`.
2. Allocate three nodes (`first`, `second`, `third`) and link `first->next = second`, `second->next = third`.
3. Traverse using a cursor pointer `curr = first`, advancing via `curr = curr->next` until `curr == NULL`.
4. Free the chain by saving `Node *next_node = curr->next` *before* calling `free(curr)`.

#### 4. Pseudocode
```
ALGORITHM CreateAndFreeThreeNodes()
BEGIN
    first  <- CreateNode(10)
    second <- CreateNode(20)
    third  <- CreateNode(30)

    first->next  <- second
    second->next <- third
    third->next  <- NULL

    /* Safe deallocation loop */
    curr <- first
    WHILE curr != NULL DO
        next_node <- curr->next
        free(curr)
        curr <- next_node
    END WHILE
END
```

#### 5. Before-and-After ASCII Pointer Diagrams
```
STEP 1: Separate Heap Nodes After create_node()
  first                   second                  third
  +------+------+         +------+------+         +------+------+
  |  10  | NULL |         |  20  | NULL |         |  30  | NULL |
  +------+------+         +------+------+         +------+------+

STEP 2: Linked Chain After Wiring next Pointers
  first (0x55a01000)      second (0x55a01040)     third (0x55a01080)
  +------+------------+   +------+------------+   +------+------+
  |  10  | 0x55a01040 |-->|  20  | 0x55a01080 |-->|  30  | NULL |
  +------+------------+   +------+------------+   +------+------+

STEP 3: After Freeing Loop Completes
  first = NULL (All three heap blocks returned to OS/allocator)
```

#### 6. Reusable C Function and Complete C11 Program
```c
#include <stdio.h>
#include <stdlib.h>

typedef struct Node {
    int data;
    struct Node *next;
} Node;

Node *create_node(int data) {
    Node *new_node = malloc(sizeof(*new_node));
    if (new_node == NULL) {
        return NULL;
    }
    new_node->data = data;
    new_node->next = NULL;
    return new_node;
}

void free_chain(Node **head_ref) {
    if (head_ref == NULL) {
        return;
    }
    Node *curr = *head_ref;
    while (curr != NULL) {
        Node *next_node = curr->next; /* Save next pointer BEFORE freeing curr */
        free(curr);
        curr = next_node;
    }
    *head_ref = NULL;
}

int main(void) {
    Node *n1 = create_node(10);
    Node *n2 = create_node(20);
    Node *n3 = create_node(30);

    if (n1 == NULL || n2 == NULL || n3 == NULL) {
        free(n1);
        free(n2);
        free(n3);
        return EXIT_FAILURE;
    }

    /* Wire the self-referential pointers */
    n1->next = n2;
    n2->next = n3;

    printf("Traversing self-referential node chain:\n  ");
    for (const Node *curr = n1; curr != NULL; curr = curr->next) {
        printf("[%d] -> ", curr->data);
    }
    printf("NULL\n");

    free_chain(&n1);
    printf("Chain freed. Head pointer is now: %s\n",
           (n1 == NULL) ? "NULL" : "non-NULL");
    return 0;
}
```

#### Program Output:
```
Traversing self-referential node chain:
  [10] -> [20] -> [30] -> NULL
Chain freed. Head pointer is now: NULL
```

#### 7. Time and Auxiliary Space Complexity
- **Time Complexity**: $\mathcal{O}(1)$ per `create_node` call; $\mathcal{O}(n)$ to traverse and free an $n$-node chain.
- **Auxiliary Space Complexity**: $\mathcal{O}(1)$ auxiliary pointers (`curr`, `next_node`); $\mathcal{O}(n)$ total heap space for the $n$ nodes.

#### 8. Edge Cases and Common Mistakes
- Calling `free(curr)` *before* reading `curr->next` causes a Use-After-Free bug (`curr->next` may already be overwritten by heap allocator metadata).

---

### 4.6 Complete C11 Demonstration Program

Below is a comprehensive, self-contained C11 program integrating structure padding inspection, nested structures, `malloc`/`calloc`/`realloc`, double-pointer (`Node**`) head insertion, and leak-free memory cleanup.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <stddef.h>

/* 1. Alignment & Padding Demonstration Structures */
typedef struct InefficientLayout {
    char tag;       /* 1 byte + 7 bytes padding */
    double metric;  /* 8 bytes                  */
    int code;       /* 4 bytes + 4 bytes tail   */
} InefficientLayout;

typedef struct OptimalLayout {
    double metric;  /* 8 bytes                  */
    int code;       /* 4 bytes                  */
    char tag;       /* 1 byte + 3 bytes tail    */
} OptimalLayout;

/* 2. Self-Referential Node Structure */
typedef struct PacketNode {
    int packet_id;
    int *payload;           /* Dynamically allocated array inside node */
    size_t payload_len;
    struct PacketNode *next;
} PacketNode;

/* Create a PacketNode with a zero-initialized payload array via calloc */
PacketNode *create_packet(int id, size_t len) {
    PacketNode *node = malloc(sizeof(*node));
    if (node == NULL) {
        return NULL;
    }
    node->payload = calloc(len, sizeof(*(node->payload)));
    if (node->payload == NULL) {
        free(node); /* Clean up partially allocated struct! */
        return NULL;
    }
    node->packet_id = id;
    node->payload_len = len;
    node->next = NULL;

    for (size_t i = 0; i < len; i++) {
        node->payload[i] = id * 10 + (int)(i + 1);
    }
    return node;
}

/* Resize a packet's internal payload array using safe realloc pattern */
bool resize_packet_payload(PacketNode *node, size_t new_len) {
    if (node == NULL || new_len == 0) {
        return false;
    }
    int *temp = realloc(node->payload, new_len * sizeof(*(node->payload)));
    if (temp == NULL) {
        return false;
    }
    /* Initialize any newly added slots */
    for (size_t i = node->payload_len; i < new_len; i++) {
        temp[i] = node->packet_id * 10 + (int)(i + 1);
    }
    node->payload = temp;
    node->payload_len = new_len;
    return true;
}

/* Prepend node using a double pointer (PacketNode**) */
bool push_packet_front(PacketNode **head_ref, int id, size_t len) {
    if (head_ref == NULL) {
        return false;
    }
    PacketNode *new_node = create_packet(id, len);
    if (new_node == NULL) {
        return false;
    }
    new_node->next = *head_ref;
    *head_ref = new_node;
    return true;
}

/* Print all packets and their internal dynamic arrays */
void print_packets(const PacketNode *head) {
    const PacketNode *curr = head;
    while (curr != NULL) {
        printf("  Packet #%d (len=%zu): [", curr->packet_id, curr->payload_len);
        for (size_t i = 0; i < curr->payload_len; i++) {
            printf("%d%s", curr->payload[i],
                   (i + 1 < curr->payload_len) ? ", " : "");
        }
        printf("] -> %s\n", (curr->next != NULL) ? "next" : "NULL");
        curr = curr->next;
    }
}

/* Deep-free: free inner payload array BEFORE freeing outer node */
void destroy_all_packets(PacketNode **head_ref) {
    if (head_ref == NULL) {
        return;
    }
    PacketNode *curr = *head_ref;
    while (curr != NULL) {
        PacketNode *next_node = curr->next;
        free(curr->payload); /* Free inner heap buffer first */
        free(curr);          /* Free outer node struct second */
        curr = next_node;
    }
    *head_ref = NULL;
}

int main(void) {
    printf("=== 1. Structure Alignment & Padding ===\n");
    printf("sizeof(InefficientLayout) = %zu bytes\n", sizeof(InefficientLayout));
    printf("sizeof(OptimalLayout)     = %zu bytes\n\n", sizeof(OptimalLayout));

    printf("=== 2. Dynamic Self-Referential Packet List ===\n");
    PacketNode *head = NULL;

    push_packet_front(&head, 3, 2);
    push_packet_front(&head, 2, 3);
    push_packet_front(&head, 1, 2);

    printf("Initial Packet Chain:\n");
    print_packets(head);

    printf("\n=== 3. Resizing Head Packet Payload via realloc() ===\n");
    resize_packet_payload(head, 5);
    print_packets(head);

    printf("\n=== 4. Deep Deallocation ===\n");
    destroy_all_packets(&head);
    printf("All packets freed. head == %s\n", (head == NULL) ? "NULL" : "non-NULL");

    return 0;
}
```

#### Program Output:
```
=== 1. Structure Alignment & Padding ===
sizeof(InefficientLayout) = 24 bytes
sizeof(OptimalLayout)     = 16 bytes

=== 2. Dynamic Self-Referential Packet List ===
Initial Packet Chain:
  Packet #1 (len=2): [11, 12] -> next
  Packet #2 (len=3): [21, 22, 23] -> next
  Packet #3 (len=2): [31, 32] -> NULL

=== 3. Resizing Head Packet Payload via realloc() ===
  Packet #1 (len=5): [11, 12, 13, 14, 15] -> next
  Packet #2 (len=3): [21, 22, 23] -> next
  Packet #3 (len=2): [31, 32] -> NULL

=== 4. Deep Deallocation ===
All packets freed. head == NULL
```

---

## 5. Complexity Analysis and Comparison Tables

### 5.1 Complexity of Memory and Pointer Operations
Let $n$ be the number of elements/nodes and $B$ be the total number of bytes requested.

| Operation | Best Case Time | Average Case Time | Worst Case Time | Auxiliary Space | Total Space |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Struct Member Access (`s.m` or `p->m`)** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ |
| **Pointer Arithmetic (`p + k`)** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ |
| **Double-Pointer Redirect (`*pp = target`)** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ |
| **`malloc(B)`** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(\text{free-list})$ | $\mathcal{O}(1)$ | $\mathcal{O}(B)$ |
| **`calloc(n, sizeof(T))`** | $\mathcal{O}(1)^*$ | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |
| **`realloc(ptr, new_B)`** | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(\text{new\_B})$ |
| **`free(ptr)`** | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ | $\mathcal{O}(1)$ |
| **Freeing an $n$-Node Linked Chain** | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(n)$ | $\mathcal{O}(1)$ | $\mathcal{O}(n)$ |

*\*Note: For very large allocations, `calloc` may map virtual pages to a shared OS zero page in $\mathcal{O}(1)$ time, deferring physical zeroing until page faults occur.*

### 5.2 Storage Duration Comparison: Static vs. Stack vs. Heap

| Feature | Static (`static` / Global) | Automatic (Stack) | Dynamic (Heap) |
| :--- | :--- | :--- | :--- |
| **Memory Segment** | Data / BSS Segment | Stack Segment | Heap Segment |
| **Allocation Time** | Compile / Load time | Runtime (Function/Block entry) | Runtime (`malloc`, `calloc`, `realloc`) |
| **Deallocation** | Process termination | Automatic at block exit (`}`) | Manual via `free()` |
| **Allocation Speed** | Zero runtime cost | Ultra-fast ($\mathcal{O}(1)$ `RSP` register shift) | Slower (free-list search, metadata, thread locks) |
| **Size Flexibility** | Fixed at compile time | Fixed upon declaration (small limits, ~1–8 MB stack) | Flexible; can grow/shrink up to system RAM |
| **Initial Contents** | Zero-initialized by default | Indeterminate garbage (unless initialized) | Garbage (`malloc`) or Zero (`calloc`) |
| **Primary Failure Mode** | Binary bloat / fixed capacity overflow | Stack Overflow (`SIGSEGV`) | Returns `NULL`; Memory Leaks; Fragmentation |

### 5.3 Heap Allocator Comparison: `malloc` vs. `calloc` vs. `realloc`

| Function | Prototype in `<stdlib.h>` | Initializes Memory? | Overflow Check on Count $\times$ Size? | Behavior on Failure |
| :--- | :--- | :--- | :--- | :--- |
| **`malloc`** | `void *malloc(size_t size);` | **No** (contains indeterminate garbage) | **No** (caller does `n * sizeof(T)`) | Returns `NULL` |
| **`calloc`** | `void *calloc(size_t nmemb, size_t size);` | **Yes** (all bits zeroed) | **Yes** (checks `nmemb * size` overflow) | Returns `NULL` |
| **`realloc`** | `void *realloc(void *ptr, size_t size);` | **No** (existing prefix copied; new tail uninitialized) | **No** | Returns `NULL`; **original `ptr` remains allocated** |

### 5.4 Parameter Passing: By Value vs. Single Pointer vs. Double Pointer

| Passing Mode | Parameter Type | Copies Placed on Stack | Can Modify Struct Fields? | Can Redirect Caller's Pointer (`head`)? |
| :--- | :--- | :--- | :---: | :---: |
| **Pass Struct by Value** | `Node n` | Entire `sizeof(Node)` bytes | No (modifies local copy only) | No |
| **Pass Single Pointer** | `Node *p` | $8\text{ bytes}$ (address of `Node`) | **Yes** (`p->data = 99;`) | No (`p = new_node;` only changes local `p`) |
| **Pass Read-Only Pointer** | `const Node *p` | $8\text{ bytes}$ (address of `Node`) | No (compiler-enforced read-only) | No |
| **Pass Double Pointer** | `Node **pp` | $8\text{ bytes}$ (address of `Node*`) | **Yes** (`(*pp)->data = 99;`) | **Yes** (`*pp = new_node;`) |

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Uninitialized (Wild) Pointers
Declaring a pointer without initializing it leaves random stack bits inside the pointer variable. Dereferencing it writes to an arbitrary memory address.

```c
/* BUG: ptr is uninitialized (wild pointer) */
Node *ptr;
ptr->data = 42; /* Undefined Behavior / Segmentation Fault! */

/* FIX: Always initialize pointers to NULL or a valid allocated object */
Node *ptr = malloc(sizeof(*ptr));
if (ptr != NULL) {
    ptr->data = 42;
    ptr->next = NULL;
}
```

### 6.2 Unchecked `NULL` Return from `malloc()` / `calloc()`
When the heap is exhausted or an excessive size is requested, `malloc()` returns `NULL`.

```c
/* BUG: Dereferencing ptr without checking if malloc succeeded */
int *arr = malloc(n * sizeof(*arr));
arr[0] = 10; /* Crashes if malloc returned NULL */

/* FIX: Always validate allocation before dereferencing */
int *arr = malloc(n * sizeof(*arr));
if (arr == NULL) {
    fprintf(stderr, "Fatal: Out of memory\n");
    return EXIT_FAILURE;
}
arr[0] = 10;
```

### 6.3 Unsafe `realloc()` Pattern Causing Memory Leaks
Assigning `ptr = realloc(ptr, new_size)` directly overwrites `ptr` with `NULL` if `realloc` fails, permanently orphaning the original heap block.

```c
/* BUG: If realloc returns NULL, the original 'buf' address is lost forever! */
buf = realloc(buf, new_size * sizeof(*buf));

/* FIX: Assign to a temporary pointer first */
int *temp = realloc(buf, new_size * sizeof(*buf));
if (temp == NULL) {
    /* 'buf' is still valid; we can save state or free(buf) cleanly */
    free(buf);
    return EXIT_FAILURE;
}
buf = temp;
```

### 6.4 Memory Leaks via Lost Heap References
Reassigning a pointer that holds the sole reference to a heap block without freeing it first creates a permanent memory leak.

```c
/* BUG: First allocation is leaked when head is overwritten */
Node *head = malloc(sizeof(*head));
head = malloc(sizeof(*head)); /* Address of first node is lost! */

/* FIX: Free the old block or link it before reassigning */
Node *head = malloc(sizeof(*head));
free(head);
head = malloc(sizeof(*head));
```

### 6.5 Dangling Pointers (Use-After-Free) and Double-Free Errors
After `free(ptr)` is called, `ptr` still holds the old numeric address. Dereferencing it (Use-After-Free) or passing it to `free()` a second time (Double-Free) corrupts the heap allocator's internal free lists.

```c
/* BUG: Use-after-free and Double-free */
Node *p = malloc(sizeof(*p));
p->data = 50;
free(p);
printf("%d\n", p->data); /* BUG 1: Use-after-free (Undefined Behavior) */
free(p);                 /* BUG 2: Double-free (Heap corruption / abort) */

/* FIX: Immediately set freed pointers to NULL; free(NULL) is a safe no-op */
Node *p = malloc(sizeof(*p));
if (p != NULL) {
    p->data = 50;
    free(p);
    p = NULL;            /* Prevents dangling access; free(NULL) does nothing */
}
```

### 6.6 Modifying a Caller's Pointer Using `Node*` Instead of `Node**`
In C, all function arguments are passed by value. Passing `Node *head` passes a **copy of the pointer**. Modifying that copy does not update the caller's `head`.

```c
/* BUG: Modifies only the local parameter 'head'; caller's head stays NULL */
void bad_init(Node *head, int val) {
    head = malloc(sizeof(*head));
    if (head != NULL) {
        head->data = val;
        head->next = NULL;
    }
} /* Memory leaked here too! */

/* FIX: Accept Node **head_ref (or return Node*) to update caller's pointer */
void good_init(Node **head_ref, int val) {
    if (head_ref == NULL) return;
    *head_ref = malloc(sizeof(**head_ref));
    if (*head_ref != NULL) {
        (*head_ref)->data = val;
        (*head_ref)->next = NULL;
    }
}
```

### 6.7 Illegal Self-Referential Structure by Value
A structure cannot contain an instance of itself **by value**, because computing `sizeof(struct BadNode)` would require infinite recursion ($\operatorname{sizeof}(\text{BadNode}) = 4 + \operatorname{sizeof}(\text{BadNode}) = \infty$).

```c
/* BUG: Incomplete type error — infinite size */
struct BadNode {
    int data;
    struct BadNode next;  /* Compile Error: field 'next' has incomplete type */
};

/* FIX: A self-referential member MUST be a pointer (fixed 8-byte size) */
typedef struct GoodNode {
    int data;
    struct GoodNode *next; /* Valid: pointer size is always known (8 bytes) */
} GoodNode;
```

### 6.8 Boundary Conditions Checklist
- [ ] **Null Pointer Guard**: Does every function accepting a pointer verify `ptr != NULL` before dereferencing?
- [ ] **Allocation Check**: Is every `malloc()`, `calloc()`, and `realloc()` return value checked against `NULL`?
- [ ] **Deep Cleanup**: When freeing a struct that owns dynamically allocated member pointers (e.g., `node->payload`), are the inner pointers freed *before* the outer struct?
- [ ] **Order of Link Traversal During `free()`**: Is `next_node = curr->next` saved *before* calling `free(curr)`?
- [ ] **Post-Free Nulling**: Are pointers set to `NULL` after `free()` so they cannot become dangling pointers?
- [ ] **Zero-Size Requests**: Are `n == 0` allocation requests guarded before calling `malloc(0)` or `realloc(ptr, 0)` (whose behavior is implementation-defined in C11)?

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways
1. **Structures (`struct`)** group related fields contiguously in memory. Ordering members from largest alignment (`double`, pointers) to smallest (`char`) minimizes compiler padding bytes.
2. **Structure Pointers** access members using `ptr->member`, which is identical to `(*ptr).member`.
3. **Pointers** store virtual addresses. Pointer arithmetic `p + k` automatically scales by $k \times \operatorname{sizeof}(*p)$.
4. **Double Pointers (`Node**`)** allow functions to modify the caller's pointer variable itself (essential for updating `head` in linked lists when inserting or deleting at the front).
5. **Dynamic Heap Memory** (`malloc`, `calloc`, `realloc`, `free`) enables data structures to grow and shrink dynamically at runtime and persist across function calls.
6. **Self-Referential Structures** combine `struct` and a pointer of the same struct type (`struct Node *next;`) to form the nodes of linked lists, trees, and graphs.

### 7.2 Essential Memory Invariants
- **Heap Balance Invariant**: Every successful execution of `malloc()`, `calloc()`, or `realloc()` (that creates a net block) must be paired with exactly one execution of `free()` before program termination:
  $$N_{\text{allocations}} - N_{\text{frees}} = 0$$
- **Pointer Validity Invariant**: Every live pointer variable in a data structure must either equal `NULL` or point to a currently allocated, initialized memory object of compatible type.
- **Chain Termination Invariant**: Every non-circular linked sequence of self-referential nodes must terminate at a node whose `next` pointer is `NULL`.

### 7.3 Quick Self-Test Checklist
- [ ] Can you explain why `sizeof(struct { char a; double b; char c; })` is 24 bytes on a 64-bit machine, and how to reorder its fields to 16 bytes?
- [ ] Can you explain why `*ptr.next` fails to compile and why `(*ptr).next` or `ptr->next` is required?
- [ ] Can you trace on paper why passing `Node *head` to an insertion function fails to update `head` in `main()`, whereas `Node **head_ref` succeeds?
- [ ] Can you write the safe 4-line `realloc()` pattern using a `temp` pointer from memory?
- [ ] Can you write a `while` loop that frees a chain of `Node` structs without triggering a Use-After-Free error?

### 7.4 Related Study Notes
- **Next Module**: [02 — Singly Linked Lists](./02-Singly-Linked-Lists.md)
- **Subsequent Module**: [03 — Doubly Linked Lists](./03-Doubly-Linked-Lists.md)
- **Circular Lists**: [04 — Circular Linked Lists](./04-Circular-Linked-Lists.md)
- **Stacks**: [05 — Stacks](./07-Stacks-Array-and-Linked-List.md)
- **Course Index**: [DSA Notes Root README](../README.md)
