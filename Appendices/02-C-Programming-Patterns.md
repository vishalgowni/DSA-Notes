# Appendix 02: C11 Data Structures Idioms & Programming Patterns

> ** Navigation**
> - **Parent Directory**: [DSA Notes Root](../README.md)
> - **Previous Appendix**: [Appendix 01: Master Big-O Complexity Cheat Sheet](./01-Complexity-Cheat-Sheet.md)
> - **Next Appendix**: [Appendix 03: Common Errors & Edge Cases](./03-Common-Errors-and-Edge-Cases.md)
> - **Related Appendix**: [Appendix 04: Practice Questions & Answers](./04-Practice-Questions-and-Answers.md)

---

## 1. Self-Referential Structs & Opaque / Forward Declarations

Every linked data structure in C (Linked Lists, Trees, Tries, Skip Lists, Adjacency Lists) relies on **self-referential structures**—structs containing one or more pointer fields that refer to the same struct type.

### 1.1 Why Value Self-Reference Is Illegal vs. Pointer Self-Reference

A `struct` in C cannot contain an **instance** of itself by value because the compiler must know the exact `sizeof(struct Node)` at compile time, and a value field would create an infinite recursive size ($\text{size} = 4 + \text{size} = \infty$).

```c
/* ❌ ILLEGAL: Incomplete type has infinite size */
struct BadNode {
    int data;
    struct BadNode next; /* COMPILER ERROR: field 'next' has incomplete type */
};

/* ✅ LEGAL: A pointer has a fixed, known size (8 bytes on 64-bit, 4 bytes on 32-bit) */
typedef struct Node {
    int data;
    struct Node *next;   /* Must use struct tag 'struct Node *' inside body */
} Node;                  /* 'Node' typedef alias becomes active here */
```

### 1.2 Forward Declaration Pattern (Mutually Referential Structs)

In graphs (where a `Vertex` holds a list of `Edge` nodes, and each `Edge` points back to a `Vertex`) or orthogonal sparse matrices, use **forward declarations** before defining the struct bodies:

```c
/* Step 1: Forward-declare the struct tags and typedef aliases */
typedef struct Vertex Vertex;
typedef struct Edge   Edge;

/* Step 2: Define the bodies using the clean typedef names */
struct Edge {
    int     weight;
    Vertex *dest;      /* Points to a Vertex */
    Edge   *next_edge; /* Next edge in adjacency list */
};

struct Vertex {
    int   id;
    Edge *adj_head;    /* Head of outgoing edges */
};
```

### 1.3 Opaque Pointer (Information Hiding / ADT Encapsulation)

In production C libraries, expose only an incomplete `typedef` in the header file (`.h`) so callers cannot corrupt internal pointers (`head`, `size`, `top`), and define the `struct` inside the implementation file (`.c`):

```c
/* stack.h — Public Header */
typedef struct Stack Stack;        /* Opaque type: caller only holds 'Stack *' */
Stack *stack_create(size_t cap);
bool   stack_push(Stack *s, int val);
void   stack_destroy(Stack **s_ref);
```

---

## 2. Safe Dynamic Memory Management Patterns

### 2.1 The Constructor / Factory Pattern (`create_node`)

Never scatter raw `malloc()` calls and field initializations across twenty places in your codebase. Centralize allocation inside a single factory function using the **`sizeof(*ptr)` idiom**:

```c
typedef struct TreeNode {
    int key;
    int height;
    struct TreeNode *left;
    struct TreeNode *right;
} TreeNode;

/* ✅ IDIOM: Use sizeof(*node) so changing the pointer's type automatically
 * updates the allocation size without risking a sizeof mismatch! */
TreeNode *create_tree_node(int key) {
    TreeNode *node = malloc(sizeof(*node));
    if (node == NULL) {
        perror("Fatal: malloc failed in create_tree_node");
        exit(EXIT_FAILURE); /* Or return NULL if caller checks for OOM */
    }
    node->key    = key;
    node->height = 0;       /* Leaf height = 0 (or 1 depending on convention) */
    node->left   = NULL;
    node->right  = NULL;
    return node;
}
```

> [!TIP]
> **When to Use `calloc` Instead of `malloc`**: When allocating **Trie nodes** (`TrieNode *children[26]`), **Hash Table bucket arrays** (`Node **buckets`), or **Graph `visited` arrays**, `calloc(count, sizeof(*ptr))` zero-initializes all bytes in one step, guaranteeing every integer is `0` and every pointer is `NULL` on all POSIX/modern architectures.

---

### 2.2 The Safe `realloc` Intermediate Pointer Pattern

When dynamically doubling a stack, heap, or dynamic array, **never assign the return value of `realloc()` directly back to the original pointer (`arr = realloc(arr, ...)`):**
- If `realloc()` fails to find a contiguous memory block, it returns `NULL`, **but it does NOT free the original block!**
- Overwriting `arr` with `NULL` immediately leaks the entire original array and loses all existing data.

```c
/* ❌ BUG: Leaks original 'vec->data' block if realloc returns NULL! */
vec->data = realloc(vec->data, new_cap * sizeof(*vec->data));

/* ✅ SAFE IDIOM: Assign to a temporary pointer first, check for NULL, then commit */
bool dynamic_array_reserve(int **arr_ref, size_t *cap_ref, size_t min_cap) {
    if (*cap_ref >= min_cap) return true;

    size_t new_cap = (*cap_ref == 0) ? 8 : (*cap_ref * 2);
    while (new_cap < min_cap) {
        new_cap *= 2;
    }

    /* Check for integer multiplication overflow before calling realloc */
    if (new_cap > SIZE_MAX / sizeof(**arr_ref)) {
        return false;
    }

    int *tmp = realloc(*arr_ref, new_cap * sizeof(**arr_ref));
    if (tmp == NULL) {
        /* Original *arr_ref is STILL VALID and intact here! */
        return false;
    }
    *arr_ref = tmp;
    *cap_ref = new_cap;
    return true;
}
```

---

### 2.3 Safe Deallocation: Iterative Lists, Circular Lists, and Postorder Trees

You must **never read `node->next` or `node->left` after calling `free(node)`** (Undefined Behavior / Use-After-Free).

#### A. Linear Singly/Doubly Linked List (Save `next` Before `free`)

```c
void free_linear_list(ListNode **head_ref) {
    if (!head_ref) return;
    ListNode *curr = *head_ref;
    while (curr != NULL) {
        ListNode *next = curr->next; /* 1. Save next pointer BEFORE freeing! */
        free(curr);                  /* 2. Free current node                 */
        curr = next;                 /* 3. Advance to saved next             */
    }
    *head_ref = NULL;                /* 4. Zero caller's pointer (no dangling ptr) */
}
```

#### B. Circular Linked List (Break the Cycle First or Stop at `head`)

```c
void free_circular_list(ListNode **head_ref) {
    if (!head_ref || !*head_ref) return;
    ListNode *head = *head_ref;
    ListNode *curr = head->next;
    while (curr != head) {           /* Stop when we wrap back to original head */
        ListNode *next = curr->next;
        free(curr);
        curr = next;
    }
    free(head);
    *head_ref = NULL;
}
```

#### C. Binary Tree / N-ary Tree / Trie (Strictly Postorder `LRN`!)

```
       [Root]          3. Free Root LAST (after both subtrees are freed)
       /    \
   [Left]  [Right]     1. Free Left Subtree  -->  2. Free Right Subtree
```

```c
void free_tree_postorder(TreeNode **root_ref) {
    if (!root_ref || !*root_ref) return;
    free_tree_postorder(&((*root_ref)->left));  /* 1. Left  */
    free_tree_postorder(&((*root_ref)->right)); /* 2. Right */
    free(*root_ref);                            /* 3. Node  */
    *root_ref = NULL;                           /* Nullify caller/parent pointer */
}
```

---

## 3. Single Pointer (`Node*`) vs. Double Pointer (`Node**`) Modification Idioms

In C, **all function arguments are passed by value**. If a function takes `Node *head` and assigns `head = new_node;`, it only modifies its own local stack parameter; the caller's `head` pointer remains untouched!

```
CALLER STACK FRAME                         CALLEE STACK FRAME (Single Pointer)
+--------------------+                     +--------------------+
| Node *head = 0x100 | --(copied by val)-> | Node *head = 0x100 |
+--------------------+                     +--------------------+
  (Unchanged if callee                       head = 0x200; (Only changes local copy!)
   modifies its copy!)
```

### 3.1 Comparing the Three Pointer Modification Styles

| Style | Function Signature | Caller Syntax | Pros & Cons |
| :--- | :--- | :--- | :--- |
| **1. Return New Head/Root** | `Node *insert(Node *head, int val)` | `head = insert(head, 42);` | **Pros**: Composable for recursive trees (`root->left = insert(root->left, k)`).<br>**Cons**: Caller can forget to assign return value (`insert(head, 42);` leaks node!). |
| **2. Double Pointer (`Node **`)** | `bool insert(Node **head_ref, int val)` | `insert(&head, 42);` | **Pros**: Updates caller's `head` in-place; return value can report `bool` success/error.<br>**Cons**: Requires explicit `*head_ref` dereferencing. |
| **3. Indirect Traversal (`Node **pp`)** | `void delete(Node **head_ref, int key)` | `delete(&head, 42);` | **Pros**: Eliminates `prev` pointer and eliminates the special `if (head)` branch! |

---

### 3.2 Linus Torvalds' Indirect Double-Pointer (`Node **pp`) Traversal Idiom

In standard linked list deletion or sorted insertion, beginners write two separate code paths:
1. A special case if the target is at `head` (`prev == NULL`).
2. A general loop maintaining two pointers (`curr` and `prev`).

By instead iterating with a **pointer-to-pointer `Node **pp = head_ref`**, `pp` points to the **exact pointer variable that will be modified**—initially the caller's `head` variable itself, and subsequently each node's `.next` field!

```
Initial State: pp = &head
  head
+-------+       +------+------+       +------+------+       +------+------+
| 0x100 | ----> |  10  | 0x200| ----> |  20  | 0x300| ----> |  30  | NULL |
+-------+       +------+------+       +------+------+       +------+------+
    ^                  |                     ^
    |                  +---------------------+
   pp (initially &head)     After 1 step: pp = &((*pp)->next)
                            Now *pp IS the 'next' field inside Node(10) (value 0x200, pointing to Node 20)!
```

When we find the node to delete (`*pp` points to `Node(20)`), executing `*pp = victim->next` writes `0x300` directly into `Node(10)->next` (or into `head` if deleting the first node)—**with zero `if (prev == NULL)` branches!**

```c
/* Branchless Sorted Insertion using Indirect Pointer **pp */
void sorted_insert_indirect(ListNode **head_ref, int val) {
    ListNode **pp = head_ref;
    while (*pp != NULL && (*pp)->data < val) {
        pp = &((*pp)->next); /* Advance pp to point to current node's .next field */
    }
    /* Splice new node right into the pointer slot addressed by pp! */
    *pp = create_node(val, *pp);
}

/* Branchless Key Deletion using Indirect Pointer **pp */
bool delete_key_indirect(ListNode **head_ref, int key) {
    ListNode **pp = head_ref;
    while (*pp != NULL && (*pp)->data != key) {
        pp = &((*pp)->next);
    }
    if (*pp == NULL) {
        return false; /* Key not found */
    }
    ListNode *victim = *pp;
    *pp = victim->next; /* Updates 'head' OR 'prev->next' identically! */
    free(victim);
    return true;
}
```

---

## 4. Dummy Sentinel / Header Node Pattern

When building a new linked list node-by-node (such as merging two sorted lists, adding two polynomials, or partitioning a list), appending to the tail normally requires checking `if (result_head == NULL)` on every iteration.

Instead, allocate a **temporary `dummy` node on the function call stack**:
- `ListNode dummy = {0, NULL};`
- `ListNode *tail = &dummy;`
- Every append simply writes `tail->next = node; tail = tail->next;`.
- When finished, return `dummy.next` (no `free(&dummy)` needed because `dummy` is automatic stack memory!).

```
Stack Memory                Heap Memory
+-------------+             +------+------+    +------+------+    +------+------+
| dummy       |             |  10  |  *---+--->|  20  |  *---+--->|  30  | NULL |
| data: 0     |             +------+------+    +------+------+    +------+------+
| next: 0x100 | ------------^                                     ^
+-------------+                                                   |
                                                                 tail
Return value: dummy.next (0x100)
```

```c
/* Merging two sorted lists in O(n + m) time and O(1) auxiliary space */
ListNode *merge_sorted_lists(ListNode *a, ListNode *b) {
    ListNode dummy = {0, NULL}; /* Stack-allocated sentinel node */
    ListNode *tail = &dummy;    /* Tail always points to a valid node! */

    while (a != NULL && b != NULL) {
        if (a->data <= b->data) {
            tail->next = a;
            a = a->next;
        } else {
            tail->next = b;
            b = b->next;
        }
        tail = tail->next;
    }
    /* Attach the remaining non-empty list in O(1) */
    tail->next = (a != NULL) ? a : b;
    return dummy.next;
}
```

---

## 5. Two-Pointer (Fast & Slow / Tortoise & Hare) Idioms

### 5.1 Finding the Middle Node (Merge Sort Split)

To split a linked list of length $n$ into two halves for Merge Sort, we want `slow` to stop at node $\lfloor (n-1)/2 \rfloor$ (the **first middle** on even-length lists, so a 2-node list `[10 -> 20]` splits into `[10]` and `[20]`, preventing infinite recursion!):

```c
ListNode *find_first_middle(ListNode *head) {
    if (head == NULL) return NULL;
    ListNode *slow = head;
    ListNode *fast = head;

    /* Check fast->next AND fast->next->next so slow stops at 1st middle on even n */
    while (fast->next != NULL && fast->next->next != NULL) {
        slow = slow->next;       /* 1 step  */
        fast = fast->next->next; /* 2 steps */
    }
    return slow;
}
```

| List Elements | Condition `fast->next && fast->next->next` (`slow` stops at) | Condition `fast && fast->next` (`slow` stops at) |
| :--- | :--- | :--- |
| `[10, 20]` ($n=2$) | **`10` (Index 0 — Safe for Merge Sort split!)** | `20` (Index 1 — Left half has 2 nodes, right has 0 $\to$ Infinite recursion!) |
| `[10, 20, 30, 40]` ($n=4$) | **`20` (Index 1)** | `30` (Index 2) |
| `[10, 20, 30, 40, 50]` ($n=5$) | **`30` (Index 2)** | **`30` (Index 2)** |

---

### 5.2 Floyd's Cycle Detection & Finding the Exact Cycle Entry

```
               L steps                 k steps (Meeting Point M)
[Head] ----------------------> [Entry] ------------------------> [Meet M]
                                  ^                                 |
                                  |_________________________________|
                                            C - k steps
                                     (Total Cycle Length = C)
```

**Mathematical Proof of Cycle Entry**:
1. Let $L$ be the distance from `head` to the cycle entry, $C$ be the cycle length, and $k$ be the distance from the cycle entry to the first meeting point $M$.
2. When `slow` and `fast` meet at $M$:
   - `slow` has traveled $d_{\text{slow}} = L + k$ steps.
   - `fast` has traveled $d_{\text{fast}} = L + k + m \cdot C$ steps (for some integer $m \ge 1$).
3. Since $d_{\text{fast}} = 2 \cdot d_{\text{slow}}$:
   $$2(L + k) = L + k + mC \implies L + k = mC \implies L = mC - k = (m - 1)C + (C - k)$$
4. Therefore, advancing one pointer from `head` ($L$ steps) and the other from $M$ ($C - k$ steps plus $(m-1)$ full loops) at **1 step per iteration** guarantees they collide at the **exact cycle entry node**!

```c
ListNode *detect_cycle_entry(ListNode *head) {
    ListNode *slow = head, *fast = head;
    while (fast != NULL && fast->next != NULL) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) {
            slow = head; /* Reset slow to head; keep fast at meeting point */
            while (slow != fast) {
                slow = slow->next; /* Both advance 1 step at a time */
                fast = fast->next;
            }
            return slow; /* Exact cycle start node */
        }
    }
    return NULL; /* No cycle */
}
```

---

### 5.3 Finding the $k$-th Node from the End (Lead-Lag Gap Pattern)

Advance `lead` $k$ steps ahead of `lag`. Then move both one step at a time until `lead == NULL`. Because the distance between `lead` and `lag` is invariant at $k$, `lag` lands directly on the $k$-th node from the end in a single $O(n)$ pass!

```c
ListNode *find_kth_from_end(ListNode *head, int k) {
    if (head == NULL || k <= 0) return NULL;
    ListNode *lead = head, *lag = head;

    for (int i = 0; i < k; i++) {
        if (lead == NULL) return NULL; /* Error: k > length of list */
        lead = lead->next;
    }
    while (lead != NULL) {
        lead = lead->next;
        lag  = lag->next;
    }
    return lag;
}
```

---

## 6. Circular Indexing & Modular Arithmetic Patterns

In C, the `%` operator computes the **remainder of truncated division**, NOT the mathematical Euclidean modulus! Specifically, in C:
$$\texttt{-1 \% 8 == -1} \quad (\text{NOT } 7!)$$

Indexing `arr[(-1) % CAP]` causes an immediate **out-of-bounds buffer underflow (`arr[-1]`)**. Always add `CAP` before taking `% CAP` when decrementing indices!

### Master Table of Circular Buffer / Deque Modular Formulas

| Operation | Formula (`front` + `size` Design — Recommended!) | Formula (`front` + `rear` Design, `empty` at `-1`) |
| :--- | :--- | :--- |
| **Initialize Empty** | `front = 0; size = 0;` | `front = -1; rear = -1;` |
| **Check Empty** | `size == 0` | `front == -1` |
| **Check Full** | `size == CAP` | `(rear + 1) % CAP == front` |
| **Current Element Count** | `size` | `(front == -1) ? 0 : (rear - front + CAP) % CAP + 1` |
| **Advance Index Forward (+1)** | `idx = (idx + 1) % CAP` | `idx = (idx + 1) % CAP` |
| **Step Index Backward (-1)** | `idx = (idx - 1 + CAP) % CAP` | `idx = (idx - 1 + CAP) % CAP` |
| **Compute `rear` from `front` & `size`** | `rear = (front + size - 1) % CAP` (last valid item)<br>`next_slot = (front + size) % CAP` (next write slot) | Explicit `rear` variable |
| **Access $i$-th Logical Item ($0 \le i < \text{size}$)** | `arr[(front + i) % CAP]` | `arr[(front + i) % CAP]` |

> [!TIP]
> **Why the `front` + `size` Design Is Superior in Exams & Production**: Storing `front` and `size` (instead of `front` and `rear`) eliminates the `-1` special initialization state, eliminates the ambiguity between full and empty states (`front == rear`), and makes `isEmpty()` (`size == 0`) and `isFull()` (`size == CAP`) trivial!

---

## 7. Explicit Stack / Queue State-Machine Pattern for Eliminating Recursion

Recursive functions on deep trees or graphs ($V, n \ge 10^5$) can overflow the OS call stack (typically 1 MB – 8 MB). Any recursive algorithm can be mechanically converted into an iterative algorithm using an explicit heap-allocated stack.

### 7.1 Iterative Tree Traversals (Inorder & Single-Stack Postorder)

```c
/* Iterative Inorder (LNR): Push left spine, pop & visit, step right */
void iterative_inorder(TreeNode *root) {
    TreeNode *stack[128];
    int top = -1;
    TreeNode *curr = root;

    while (curr != NULL || top >= 0) {
        while (curr != NULL) {
            stack[++top] = curr;
            curr = curr->left;  /* Plunge down left spine */
        }
        curr = stack[top--];    /* Pop deepest left node */
        printf("%d ", curr->key);
        curr = curr->right;     /* Now explore its right subtree */
    }
}

/* Iterative Single-Stack Postorder (LRN) using 'last_visited' tracker */
void iterative_postorder(TreeNode *root) {
    TreeNode *stack[128];
    int top = -1;
    TreeNode *curr = root;
    TreeNode *last_visited = NULL;

    while (curr != NULL || top >= 0) {
        if (curr != NULL) {
            stack[++top] = curr;
            curr = curr->left;
        } else {
            TreeNode *peek_node = stack[top];
            /* If right child exists and hasn't been processed yet, descend right */
            if (peek_node->right != NULL && last_visited != peek_node->right) {
                curr = peek_node->right;
            } else {
                printf("%d ", peek_node->key);
                last_visited = stack[top--];
            }
        }
    }
}
```

### 7.2 General Call-Frame State Machine Pattern

When converting a complex recursive function (such as postorder height calculation or DFS finish times) where work happens **between** and **after** recursive calls, define an explicit `Frame` struct with a `stage` program counter:

```c
typedef struct {
    TreeNode *node;
    int       stage; /* 0: before left, 1: before right, 2: after both (postorder) */
} CallFrame;
```

---

## 8. Complete, Compilable C11 Showcase Program

The complete C11 program below demonstrates all seven patterns working together cleanly with zero memory leaks.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <assert.h>

/* =========================================================================
 * 1. SELF-REFERENTIAL STRUCT & CONSTRUCTOR PATTERN
 * ========================================================================= */
typedef struct ListNode {
    int data;
    struct ListNode *next;
} ListNode;

ListNode *create_node(int data, ListNode *next) {
    ListNode *node = malloc(sizeof(*node));
    if (!node) {
        perror("malloc failed in create_node");
        exit(EXIT_FAILURE);
    }
    node->data = data;
    node->next = next;
    return node;
}

/* =========================================================================
 * 2. LINUS TORVALDS' INDIRECT DOUBLE-POINTER (**pp) INSERT & DELETE
 * ========================================================================= */
void sorted_insert_indirect(ListNode **head_ref, int val) {
    ListNode **pp = head_ref;
    while (*pp != NULL && (*pp)->data < val) {
        pp = &((*pp)->next);
    }
    *pp = create_node(val, *pp);
}

bool delete_key_indirect(ListNode **head_ref, int key) {
    ListNode **pp = head_ref;
    while (*pp != NULL && (*pp)->data != key) {
        pp = &((*pp)->next);
    }
    if (*pp == NULL) {
        return false; /* Key not found */
    }
    ListNode *victim = *pp;
    *pp = victim->next; /* Seamlessly updates head OR prev->next! */
    free(victim);
    return true;
}

/* =========================================================================
 * 3. STACK-ALLOCATED DUMMY SENTINEL MERGE PATTERN
 * ========================================================================= */
ListNode *merge_sorted_lists(ListNode *a, ListNode *b) {
    ListNode dummy = {0, NULL};
    ListNode *tail = &dummy;

    while (a != NULL && b != NULL) {
        if (a->data <= b->data) {
            tail->next = a;
            a = a->next;
        } else {
            tail->next = b;
            b = b->next;
        }
        tail = tail->next;
    }
    tail->next = (a != NULL) ? a : b;
    return dummy.next;
}

/* =========================================================================
 * 4. TWO-POINTER (TORTOISE & HARE + LEAD-LAG) IDIOMS
 * ========================================================================= */
ListNode *find_middle(ListNode *head) {
    if (!head) return NULL;
    ListNode *slow = head;
    ListNode *fast = head;
    while (fast->next != NULL && fast->next->next != NULL) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow; /* Returns first middle on even length */
}

ListNode *find_kth_from_end(ListNode *head, int k) {
    if (!head || k <= 0) return NULL;
    ListNode *lead = head;
    ListNode *lag  = head;
    for (int i = 0; i < k; i++) {
        if (lead == NULL) return NULL; /* List has fewer than k nodes */
        lead = lead->next;
    }
    while (lead != NULL) {
        lead = lead->next;
        lag  = lag->next;
    }
    return lag;
}

ListNode *detect_cycle_entry(ListNode *head) {
    ListNode *slow = head, *fast = head;
    while (fast != NULL && fast->next != NULL) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) {
            /* Cycle detected! Reset slow to head and advance both by 1 */
            slow = head;
            while (slow != fast) {
                slow = slow->next;
                fast = fast->next;
            }
            return slow; /* Exact cycle entry node */
        }
    }
    return NULL;
}

/* =========================================================================
 * 5. CIRCULAR DEQUE WITH MODULAR ARITHMETIC PATTERN
 * ========================================================================= */
#define DEQUE_CAP 5
typedef struct {
    int data[DEQUE_CAP];
    int front;
    int size;
} CircularDeque;

void deque_init(CircularDeque *dq) {
    dq->front = 0;
    dq->size  = 0;
}

bool deque_push_front(CircularDeque *dq, int val) {
    if (dq->size == DEQUE_CAP) return false;
    dq->front = (dq->front - 1 + DEQUE_CAP) % DEQUE_CAP;
    dq->data[dq->front] = val;
    dq->size++;
    return true;
}

bool deque_push_back(CircularDeque *dq, int val) {
    if (dq->size == DEQUE_CAP) return false;
    int rear = (dq->front + dq->size) % DEQUE_CAP;
    dq->data[rear] = val;
    dq->size++;
    return true;
}

bool deque_pop_front(CircularDeque *dq, int *out) {
    if (dq->size == 0) return false;
    *out = dq->data[dq->front];
    dq->front = (dq->front + 1) % DEQUE_CAP;
    dq->size--;
    return true;
}

/* =========================================================================
 * 6. EXPLICIT STATE-MACHINE STACK FOR ITERATIVE POSTORDER & SAFE FREE
 * ========================================================================= */
typedef struct TreeNode {
    int key;
    struct TreeNode *left;
    struct TreeNode *right;
} TreeNode;

TreeNode *create_tree_node(int key, TreeNode *left, TreeNode *right) {
    TreeNode *node = malloc(sizeof(*node));
    if (!node) exit(EXIT_FAILURE);
    node->key = key;
    node->left = left;
    node->right = right;
    return node;
}

void iterative_postorder(TreeNode *root) {
    if (!root) return;
    TreeNode *stack[64];
    int top = -1;
    TreeNode *curr = root;
    TreeNode *last_visited = NULL;

    printf("Iterative Postorder: ");
    while (curr != NULL || top >= 0) {
        if (curr != NULL) {
            stack[++top] = curr;
            curr = curr->left;
        } else {
            TreeNode *peek_node = stack[top];
            if (peek_node->right != NULL && last_visited != peek_node->right) {
                curr = peek_node->right;
            } else {
                printf("%d ", peek_node->key);
                last_visited = stack[top--];
            }
        }
    }
    printf("\n");
}

void free_tree_postorder(TreeNode **root_ref) {
    if (!root_ref || !*root_ref) return;
    free_tree_postorder(&((*root_ref)->left));
    free_tree_postorder(&((*root_ref)->right));
    free(*root_ref);
    *root_ref = NULL;
}

void free_list(ListNode **head_ref) {
    ListNode *curr = *head_ref;
    while (curr != NULL) {
        ListNode *next = curr->next;
        free(curr);
        curr = next;
    }
    *head_ref = NULL;
}

void print_list(const char *label, const ListNode *head) {
    printf("%s: ", label);
    for (const ListNode *p = head; p != NULL; p = p->next) {
        printf("%d -> ", p->data);
    }
    printf("NULL\n");
}

int main(void) {
    /* 1 & 2. Indirect double-pointer sorted insert & delete */
    ListNode *list1 = NULL;
    int vals1[] = {30, 10, 50, 20, 40};
    for (int i = 0; i < 5; i++) {
        sorted_insert_indirect(&list1, vals1[i]);
    }
    print_list("After indirect sorted inserts", list1);

    delete_key_indirect(&list1, 10); /* Delete head without special branch */
    delete_key_indirect(&list1, 30); /* Delete middle */
    print_list("After deleting 10 and 30     ", list1);

    /* 3. Dummy sentinel merge */
    ListNode *list2 = NULL;
    sorted_insert_indirect(&list2, 15);
    sorted_insert_indirect(&list2, 25);
    sorted_insert_indirect(&list2, 45);
    ListNode *merged = merge_sorted_lists(list1, list2);
    print_list("Merged sorted list (dummy)   ", merged);

    /* 4. Two-pointer idioms */
    ListNode *mid = find_middle(merged);
    ListNode *kth = find_kth_from_end(merged, 2);
    printf("Middle node: %d | 2nd from end: %d\n", mid->data, kth->data);

    /* Create temporary cycle: tail (50) -> node (25) */
    ListNode *tail = merged;
    while (tail->next != NULL) tail = tail->next;
    tail->next = mid; /* 50 -> 25 */
    ListNode *entry = detect_cycle_entry(merged);
    printf("Detected cycle entry at node : %d\n", entry ? entry->data : -1);
    tail->next = NULL; /* Restore linear list before freeing */

    /* 5. Circular Deque modular wrap-around */
    CircularDeque dq;
    deque_init(&dq);
    deque_push_back(&dq, 100);
    deque_push_back(&dq, 200);
    deque_push_front(&dq, 50);
    deque_push_front(&dq, 25);
    printf("Circular Deque pop_front     : ");
    int val;
    while (deque_pop_front(&dq, &val)) {
        printf("%d ", val);
    }
    printf("\n");

    /* 6. Iterative Postorder & Postorder Tree Free */
    TreeNode *root = create_tree_node(40,
        create_tree_node(20, create_tree_node(10, NULL, NULL), create_tree_node(30, NULL, NULL)),
        create_tree_node(60, create_tree_node(50, NULL, NULL), create_tree_node(70, NULL, NULL))
    );
    iterative_postorder(root);
    free_tree_postorder(&root);
    assert(root == NULL);

    free_list(&merged);
    assert(merged == NULL);
    printf("All memory freed cleanly (root == NULL, merged == NULL).\n");
    return 0;
}
```

### Verified Program Output

```text
After indirect sorted inserts: 10 -> 20 -> 30 -> 40 -> 50 -> NULL
After deleting 10 and 30     : 20 -> 40 -> 50 -> NULL
Merged sorted list (dummy)   : 15 -> 20 -> 25 -> 40 -> 45 -> 50 -> NULL
Middle node: 25 | 2nd from end: 45
Detected cycle entry at node : 25
Circular Deque pop_front     : 25 50 100 200 
Iterative Postorder: 10 30 20 50 70 60 40 
All memory freed cleanly (root == NULL, merged == NULL).
```

---

> **Next Step**: Continue to **[Appendix 03: Common Errors & Edge Cases](./03-Common-Errors-and-Edge-Cases.md)** to see the 20 deadliest C DSA bugs, their memory corruption diagrams, and exact fixes.
