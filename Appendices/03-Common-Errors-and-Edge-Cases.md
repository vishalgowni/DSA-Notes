# Appendix 03: Master Diagnostic Guide to Common C DSA Errors & Edge Cases

> ** Navigation**
> - **Parent Directory**: [DSA Notes Root](../README.md)
> - **Previous Appendix**: [Appendix 02: C Programming Patterns & Idioms](./02-C-Programming-Patterns.md)
> - **Next Appendix**: [Appendix 04: Practice Questions & Answers](./04-Practice-Questions-and-Answers.md)
> - **Related Appendix**: [Appendix 01: Master Big-O Complexity Cheat Sheet](./01-Complexity-Cheat-Sheet.md)

---

## Overview: How to Use This Diagnostic Manual

In C Data Structures exams and lab implementations, 95% of crashes, silent data corruption, and lost marks stem from **20 recurring structural bugs**. This appendix catalogs all 20 bugs across Units 1–4, showing for each:
1. **Symptom / Consequence** (What happens at runtime).
2. **The Buggy Code (`/* ❌ BUG */`) & Memory Corruption Diagram**.
3. **The Corrected Code (`/* ✅ FIX */`) & Prevention Rule**.

---

## Category A: Dynamic Memory & Pointer Lifecycle Bugs (Bugs 1–4)

### Bug 1: Direct `ptr = realloc(ptr, ...)` Assignment & Unchecked `malloc`

- **Symptom / Consequence**: Memory leak of the entire existing buffer plus immediate `NULL` dereference (`SIGSEGV`) under low-memory conditions.
- **Why It Happens**: If `realloc()` cannot allocate the new block, it returns `NULL` **and leaves the original memory block allocated**. Overwriting `ptr` with `NULL` loses the only address to the old block.

```
Before realloc:
  arr ---> [ 10 | 20 | 30 | 40 ] (Heap block at 0x5000)

After buggy `arr = realloc(arr, huge_size)` fails:
  arr ---> NULL                  [ 10 | 20 | 30 | 40 ] (0x5000 ORPHANED / LEAKED!)
```

```c
/* ❌ BUG: Overwrites 'arr' with NULL if realloc fails; also doesn't check malloc */
int *grow_buggy(int *arr, size_t new_cap) {
    arr = realloc(arr, new_cap * sizeof(int));
    arr[0] = 99; /* SEGFAULT if realloc returned NULL! */
    return arr;
}

/* ✅ FIX: Use an intermediate temporary pointer and verify non-NULL before committing */
bool grow_safe(int **arr_ref, size_t new_cap) {
    int *tmp = realloc(*arr_ref, new_cap * sizeof(**arr_ref));
    if (tmp == NULL) {
        return false; /* Original *arr_ref is still intact and reachable! */
    }
    *arr_ref = tmp;
    return true;
}
```
- **Prevention Rule**: Never write `p = realloc(p, ...)`. Always capture `void *tmp = realloc(p, ...)` first.

---

### Bug 2: Preorder `free()` (Use-After-Free During List/Tree Destruction)

- **Symptom / Consequence**: Undefined Behavior (`SIGSEGV`, heap allocator corruption, or infinite loop) when freeing a linked list or binary tree.
- **Why It Happens**: Calling `free(curr)` immediately invalidates `*curr`. Reading `curr->next` or `curr->left` on the next line reads poisoned/reclaimed heap memory.

```
[curr: 0x100] --next--> [0x200]
1. free(curr);          --> 0x100 returned to OS/allocator (contents scrambled!)
2. curr = curr->next;   --> ❌ READS FREED MEMORY AT 0x100!
```

```c
/* ❌ BUG: Frees node BEFORE reading its child/next pointers! */
void free_tree_buggy(TreeNode *root) {
    if (!root) return;
    free(root);                    /* Destroyed root! */
    free_tree_buggy(root->left);   /* ❌ Use-After-Free: reading root->left! */
    free_tree_buggy(root->right);  /* ❌ Use-After-Free: reading root->right! */
}

/* ✅ FIX: Free children FIRST (Postorder LRN), then free parent last */
void free_tree_safe(TreeNode *root) {
    if (!root) return;
    free_tree_safe(root->left);    /* 1. Free left subtree  */
    free_tree_safe(root->right);   /* 2. Free right subtree */
    free(root);                    /* 3. Free root LAST     */
}
```
- **Prevention Rule**: Linked lists require saving `Node *next = curr->next;` before `free(curr)`. Trees and Tries require strict **postorder** destruction.

---

### Bug 3: Double Free & Dangling Pointer Aliasing

- **Symptom / Consequence**: Immediate `abort()` (`free(): double free detected in tcache`) or silent corruption of `malloc`'s internal free-list bin pointers.
- **Why It Happens**: Freeing a structure without setting the caller's pointer to `NULL`, or shallow-copying node pointers between two structures and freeing both.

```
  head (0x100) ----+
                   v
                 [ Freed Heap Chunk 0x100 ]
                   ^
  alias (0x100) ---+  (Second free(alias) corrupts allocator freelist!)
```

```c
/* ❌ BUG: Caller's pointer remains non-NULL after free */
free(head);
/* ... later in cleanup ... */
if (head != NULL) free(head); /* ❌ DOUBLE FREE: head still holds old address! */

/* ✅ FIX: Always nullify the pointer immediately after freeing */
free(head);
head = NULL;
```
- **Prevention Rule**: Pass `Node **head_ref` to destruction routines and set `*head_ref = NULL` as the final statement.

---

### Bug 4: Returning a Pointer to a Stack-Allocated Local Struct

- **Symptom / Consequence**: Garbage values or `SIGSEGV` as soon as the next function call overwrites the popped stack frame.
- **Why It Happens**: Local non-`static` variables live on the **call stack** and cease to exist the instant the function returns.

```
Stack Frame of create_node_buggy():
+--------------------------+
| Node n = {42, NULL}      | <--- Address 0x7fff10 (POPPED on return!)
+--------------------------+
Caller receives 0x7fff10, which is immediately overwritten by printf()!
```

```c
/* ❌ BUG: Returns address of automatic local stack variable 'n' */
Node *create_node_buggy(int val) {
    Node n;
    n.data = val;
    n.next = NULL;
    return &n; /* ❌ Compiler warning: function returns address of local variable */
}

/* ✅ FIX: Allocate on the heap with malloc() so lifetime persists until free() */
Node *create_node_safe(int val) {
    Node *n = malloc(sizeof(*n));
    if (!n) exit(EXIT_FAILURE);
    n->data = val;
    n->next = NULL;
    return n;
}
```
- **Prevention Rule**: Never return `&local_var` from any function. Either return the struct by value or allocate it on the heap via `malloc`.

---

## Category B: Unit 1 — Linked List Pointer Wiring Bugs (Bugs 5–7)

### Bug 5: Losing the Rest of the Linked List During Insertion (Wrong Assignment Order)

- **Symptom / Consequence**: Infinite self-loop (`new_node->next == new_node`) and permanent memory leak of all nodes after `prev`.
- **Why It Happens**: Overwriting `prev->next = new_node` **before** saving `prev->next` into `new_node->next`.

```
Before: [prev: 10] --------next--------> [20] ---> [30]

Buggy Step 1 (`prev->next = new_node`):
        [prev: 10] ---> [new: 15]        [20] ---> [30] (ORPHANED!)

Buggy Step 2 (`new_node->next = prev->next`):
        [prev: 10] ---> [new: 15] --+
                           ^        |
                           +--------+ (INFINITE SELF-LOOP!)
```

```c
/* ❌ BUG: Overwrites prev->next before reading its old value! */
void insert_after_buggy(Node *prev, Node *new_node) {
    prev->next = new_node;
    new_node->next = prev->next; /* ❌ Sets new_node->next = new_node! */
}

/* ✅ FIX: Always wire the NEW node's outgoing pointer(s) FIRST! */
void insert_after_safe(Node *prev, Node *new_node) {
    new_node->next = prev->next; /* 1. Attach new_node to rest of list */
    prev->next = new_node;       /* 2. Redirect prev to new_node       */
}
```
- **Prevention Rule**: **Golden Rule of Pointer Splicing**: Always attach the **new node's outgoing pointers first** before modifying any existing node's pointers.

---

### Bug 6: Incomplete Pointer Wiring in Doubly Linked Lists (DLL) & Circular Lists (CLL)

- **Symptom / Consequence**: Forward traversal (`next`) works, but backward traversal (`prev`) skips nodes or crashes on `NULL`; or inserting at the head of a Circular Linked List breaks the circle (`tail->next` still points to the old head).
- **Why It Happens**: Inserting a node in the middle of a DLL requires updating **4 pointers** (2 on `new_node`, 1 on `prev`, 1 on `prev->next`), and guarded check if `prev->next != NULL`.

```
DLL Middle Insertion requires 4 links (Buggy code forgets link #3!):
               (1) prev                 (2) next
        +-------------------- [new] --------------------+
        v                       ^                       v
     [prev] ----(4) next--------+-------(3) prev---- [next_node]
```

```c
/* ❌ BUG: Updates 3 pointers in DLL insert, forgetting next_node->prev! */
void dll_insert_after_buggy(DNode *prev, DNode *new_node) {
    new_node->next = prev->next;
    new_node->prev = prev;
    prev->next = new_node;
    /* ❌ Forgot: if (new_node->next) new_node->next->prev = new_node; */
}

/* ✅ FIX: Wire all 4 pointers and guard against tail insertion (new_node->next == NULL) */
void dll_insert_after_safe(DNode *prev, DNode *new_node) {
    new_node->next = prev->next;              /* Link 1 */
    new_node->prev = prev;                    /* Link 2 */
    if (prev->next != NULL) {
        prev->next->prev = new_node;          /* Link 3 (guarded for tail!) */
    }
    prev->next = new_node;                    /* Link 4 */
}
```
- **Prevention Rule**: In a DLL, every `a->next = b` (where `b != NULL`) must have a matching `b->prev = a`. In a CLL, changing `head` requires updating `tail->next = head`.

---

### Bug 7: Infinite Loop Traversing a Circular Linked List with `while (curr != NULL)`

- **Symptom / Consequence**: Program hangs in an infinite loop on any non-empty Circular Linked List (or crashes on empty list if `do-while` isn't guarded).
- **Why It Happens**: By definition, a Circular Linked List has **no `NULL` pointer** (`tail->next == head`).

```
  head
   v
 [10] ---> [20] ---> [30]
   ^                   |
   +-------------------+  (curr != NULL is ALWAYS true!)
```

```c
/* ❌ BUG: Never terminates on a non-empty Circular Linked List! */
void print_cll_buggy(Node *head) {
    Node *curr = head;
    while (curr != NULL) { /* ❌ Infinite loop! */
        printf("%d ", curr->data);
        curr = curr->next;
    }
}

/* ✅ FIX: Guard against NULL head first, then use a do-while loop stopping at head */
void print_cll_safe(const Node *head) {
    if (head == NULL) return; /* Essential guard before do-while! */
    const Node *curr = head;
    do {
        printf("%d ", curr->data);
        curr = curr->next;
    } while (curr != head);
}
```
- **Prevention Rule**: Always traverse a Circular Linked List with `if (!head) return; Node *curr = head; do { ... curr = curr->next; } while (curr != head);`.

---

## Category C: Unit 1 & Unit 2 — Stack & Queue Bugs (Bugs 8–11)

### Bug 8: Array Stack Off-by-One Overflow (`top == MAX` vs `top == MAX - 1`)

- **Symptom / Consequence**: Out-of-bounds write to `data[MAX]` (`stack buffer overflow`), corrupting adjacent struct fields (often overwriting `top` itself!).
- **Why It Happens**: In C, an array `int data[MAX]` has valid indices `0` through `MAX - 1`. When `top` holds the index of the current top element (initialized to `-1`), the stack is already full when `top == MAX - 1`.

```
Index:   0    1    2    3    (MAX = 4)
       +----+----+----+----+
data:  | 10 | 20 | 30 | 40 |   [ Out-of-bounds: data[4] ]
       +----+----+----+----+             ^
                        ^                |
                   top = 3 (FULL!)   ++top writes here if checked against MAX!
```

```c
/* ❌ BUG: Allows ++top to reach MAX, writing 1 element past array end! */
bool push_buggy(Stack *s, int val) {
    if (s->top == MAX) return false; /* ❌ Too late! Valid indices are 0..MAX-1 */
    s->data[++(s->top)] = val;       /* Writes to s->data[MAX] when top was MAX-1! */
    return true;
}

/* ✅ FIX: Check top == MAX - 1 before pre-incrementing top */
bool push_safe(Stack *s, int val) {
    if (s->top == MAX - 1) return false;
    s->data[++(s->top)] = val;
    return true;
}
```
- **Prevention Rule**: If `top = -1` when empty, full is `top == MAX - 1`.

---

### Bug 9: Popping Operands in Reverse Order During Postfix Evaluation

- **Symptom / Consequence**: Addition and multiplication pass tests, but subtraction (`-`), division (`/`), and exponentiation (`^`) produce inverted results (e.g., `"10 3 -"` evaluates to `-7` instead of `7`!).
- **Why It Happens**: In postfix expression `A B op`, `A` is pushed first and `B` is pushed second. Because a stack is **LIFO**, the **first `pop()` returns the right operand `B`**, and the **second `pop()` returns the left operand `A`**!

```
Expression: "10 3 -" (meaning 10 - 3 = 7)
Stack before '-':
  |  3  | <-- top (Popped 1st = RIGHT operand 'b')
  | 10  |         (Popped 2nd = LEFT  operand 'a')
  +-----+
```

```c
/* ❌ BUG: First pop assigned to left operand 'a'! Computes 3 - 10 = -7! */
int a = pop(&s);
int b = pop(&s);
push(&s, a - b); /* ❌ WRONG for '-', '/', '%', '^'! */

/* ✅ FIX: First pop is RIGHT operand 'b'; second pop is LEFT operand 'a' */
int right_op = pop(&s);
int left_op  = pop(&s);
push(&s, left_op - right_op); /* ✅ Computes 10 - 3 = 7 */
```
- **Prevention Rule**: Name your popped variables `right` and `left` (not `a` and `b`) in Postfix evaluation! (Note: In right-to-left **Prefix** evaluation, the top of stack *is* `left`!)

---

### Bug 10: Simple Linear Array Queue "False Overflow"

- **Symptom / Consequence**: Queue reports "Queue Overflow" and rejects new items even when the queue is completely empty or holds only 1 element!
- **Why It Happens**: In a non-circular queue, `front` and `rear` only march rightward (`0 -> 1 -> ... -> CAP-1`) and never wrap around to reuse vacated slots `0 .. front-1`.

```
CAP = 5, after 5 enqueues and 4 dequeues (Only 1 item in queue, 4 empty slots!):
Index:     0       1       2       3       4
        +-------+-------+-------+-------+-------+
data:   | (dead)| (dead)| (dead)| (dead)|  50   |
        +-------+-------+-------+-------+-------+
                                         ^
                                  front=4, rear=4 --> rear == CAP-1 ("FULL"!)
```

```c
/* ❌ BUG: Linear increment causes false overflow once rear hits CAP - 1 */
bool enqueue_buggy(Queue *q, int val) {
    if (q->rear == CAP - 1) return false; /* ❌ Rejects even if front == 4! */
    q->data[++(q->rear)] = val;
    return true;
}

/* ✅ FIX: Use Circular Array Queue with modular wrap-around (% CAP) */
bool enqueue_circular(CircularQueue *q, int val) {
    if (q->size == CAP) return false; /* True overflow only when all CAP slots used */
    int rear = (q->front + q->size) % CAP;
    q->data[rear] = val;
    q->size++;
    return true;
}
```
- **Prevention Rule**: Always implement array-backed queues as **Circular Queues** using `% CAP`.

---

### Bug 11: Dangling `rear` Pointer When Dequeuing the Last Node of a Linked Queue

- **Symptom / Consequence**: Dequeuing the last element leaves `rear` pointing to freed heap memory. The very next `enqueue()` writes `q->rear->next = new_node` into freed memory (Use-After-Free) and loses `new_node`!
- **Why It Happens**: Updating `q->front = q->front->next` makes `q->front == NULL` when the single remaining node is removed, but `q->rear` still holds the old node's address unless explicitly reset.

```
Before last dequeue (1 node left):
  front ---> [0x100: 42 | NULL] <--- rear

After buggy dequeue (`front = front->next; free(temp);`):
  front ---> NULL
  rear  ---> [0x100: FREED MEMORY!] ❌ Dangling pointer!
```

```c
/* ❌ BUG: Forgets to reset q->rear = NULL when queue becomes empty! */
int dequeue_buggy(LinkedQueue *q) {
    Node *temp = q->front;
    int val = temp->data;
    q->front = q->front->next;
    free(temp);
    return val; /* ❌ If q->front is now NULL, q->rear is a dangling pointer! */
}

/* ✅ FIX: Reset q->rear = NULL whenever q->front becomes NULL */
int dequeue_safe(LinkedQueue *q) {
    Node *temp = q->front;
    int val = temp->data;
    q->front = q->front->next;
    if (q->front == NULL) {
        q->rear = NULL; /* ✅ Critical reset for 1-element-to-empty transition! */
    }
    free(temp);
    return val;
}
```
- **Prevention Rule**: Whenever a container maintains both `head`/`front` and `tail`/`rear` pointers, test the **1-element $\to$ 0-element deletion transition** and ensure both pointers become `NULL`.

---

## Category D: Unit 2 & Unit 3 — Tree, BST, Heap, AVL & Splay Bugs (Bugs 12–16)

### Bug 12: The Local-Only `is_bst()` Trap

- **Symptom / Consequence**: Function returns `true` for invalid Binary Search Trees where a grandchild violates an ancestor's bound.
- **Why It Happens**: The BST invariant requires **every** node in `u`'s left subtree to be strictly less than `u->key`, not just `u`'s immediate left child!

```
         [10]
        /    \
      [5]    [20]
     /   \
   [2]   [15]  <-- ❌ 15 > 5 (valid locally for parent 5),
                   but 15 > 10 (INVALID in left subtree of root 10!)
```

```c
/* ❌ BUG: Only checks immediate children; returns true for the invalid tree above! */
bool is_bst_buggy(const TreeNode *root) {
    if (!root) return true;
    if (root->left  && root->left->key  >= root->key) return false;
    if (root->right && root->right->key <= root->key) return false;
    return is_bst_buggy(root->left) && is_bst_buggy(root->right);
}

/* ✅ FIX: Propagate strict (min_allowed, max_allowed) range bounds down the tree */
bool is_bst_range(const TreeNode *node, long min_val, long max_val) {
    if (!node) return true;
    if (node->key <= min_val || node->key >= max_val) {
        return false;
    }
    return is_bst_range(node->left,  min_val,   node->key) &&
           is_bst_range(node->right, node->key, max_val);
}
/* Call as: is_bst_range(root, LONG_MIN, LONG_MAX) */
```
- **Prevention Rule**: Validate BSTs using either recursive `(min, max)` range clamping or by verifying that an Inorder traversal is strictly increasing.

---

### Bug 13: Updating New Root Height Before Old Root Height in AVL Rotations

- **Symptom / Consequence**: Balance factors become corrupted after an AVL rotation, causing valid AVL trees to miss needed rotations or trigger wrong rotations.
- **Why It Happens**: After a right rotation at `y` (promoting `x = y->left` to be the new subtree root, with `y` becoming `x->right`), **`y` is now the child of `x`**. Since a parent's height depends on its children's heights, `y->height` MUST be recomputed **before** `x->height`!

```
Before Right Rotate(y):          After Right Rotate(y):
        [y]                              [x]   <-- 2. Update NEW parent (x) SECOND!
       /   \                            /   \
     [x]    T3                        T1    [y] <-- 1. Update NEW child  (y) FIRST!
    /   \                                  /   \
   T1   T2                                T2   T3
```

```c
/* ❌ BUG: Recomputes x->height using y's STALE pre-rotation height! */
AVLNode *rotate_right_buggy(AVLNode *y) {
    AVLNode *x  = y->left;
    AVLNode *T2 = x->right;
    x->right = y;
    y->left  = T2;
    x->height = 1 + max(height(x->left), height(x->right)); /* ❌ y->height is stale! */
    y->height = 1 + max(height(y->left), height(y->right));
    return x;
}

/* ✅ FIX: Always update the demoted node (y, now lower) BEFORE the promoted node (x) */
AVLNode *rotate_right_safe(AVLNode *y) {
    AVLNode *x  = y->left;
    AVLNode *T2 = x->right;
    x->right = y;
    y->left  = T2;
    y->height = 1 + max(height(y->left), height(y->right)); /* 1. Lower node FIRST  */
    x->height = 1 + max(height(x->left), height(x->right)); /* 2. Upper node SECOND */
    return x;
}
```
- **Prevention Rule**: In any tree rotation, recompute metadata (height, subtree size) bottom-up: **demoted child first, promoted parent second**.

---

### Bug 14: Rotating Parent Before Grandparent in Splay Tree **Zig-Zig**

- **Symptom / Consequence**: Splay tree fails to halve the height of skewed paths, degrading to $\Theta(n)$ amortized time instead of $O(\log n)$!
- **Why It Happens**: When node `x` and parent `p` are both left children (`LL` Zig-Zig) or both right children (`RR` Zig-Zig), doing two simple rotations on `x` (rotate `p` then rotate `g`, i.e., `Zig(p)` then `Zig(g)`) simply pushes the chain down without folding it. Tarjan & Sleator's **Zig-Zig** rule requires rotating **grandparent `g` first**, and **then rotating parent `p`**!

```
Initial LL Chain:      ❌ WRONG (Rotate p then g):     ✅ CORRECT ZIG-ZIG (Rotate g FIRST, then p):
      [g]                       [x]                                  [x]
      /                           \                                   \
    [p]                           [g]                                 [p]
    /                             /                                     \
  [x]                           [p]                                     [g]
```

```c
/* ❌ BUG: Rotates x with p first, then x with g (Double-Zig instead of Zig-Zig!) */
/* On a skewed tree, this leaves the tree nearly as deep as before! */
rotate_right(p);
rotate_right(g);

/* ✅ FIX: In Zig-Zig (same-direction LL or RR), rotate GRANDPARENT g first, then p! */
/* (Note: In Zig-Zag LR or RL, you DO rotate p first, then g!) */
rotate_right(g); /* 1. Rotate grandparent g FIRST */
rotate_right(p); /* 2. Rotate parent p SECOND     */
```
- **Prevention Rule**: Memorize the Splay Tree distinction:
  - **Zig-Zag (`LR` / `RL`)**: Rotate `p` first, then `g` (identical to AVL double rotation).
  - **Zig-Zig (`LL` / `RR`)**: Rotate **`g` first**, then `p`!

---

### Bug 15: Confusing Child Pointer with Thread in a Threaded BST

- **Symptom / Consequence**: Infinite loop during traversal or insertion, or corrupting ancestor nodes by treating a thread pointer as a real child subtree.
- **Why It Happens**: In a Threaded BST, leaf pointers are **not `NULL`**—they point to the inorder predecessor or successor! Checking `if (curr->left != NULL)` instead of `if (!curr->lthread)` follows threads back up the tree forever.

```
        [20]
       /    \
     [10]   [30]
       \
        +--- (rthread=true: points back to 20, NOT a right child!)
```

```c
/* ❌ BUG: Uses != NULL on a threaded tree, following threads in an infinite cycle! */
TNode *leftmost_buggy(TNode *n) {
    while (n != NULL && n->left != NULL) { /* ❌ Follows predecessor thread! */
        n = n->left;
    }
    return n;
}

/* ✅ FIX: Always inspect the boolean thread flag (lthread / rthread) before descending! */
TNode *leftmost_safe(TNode *n) {
    if (n == NULL) return NULL;
    while (n->lthread == false) { /* ✅ Only descend while left is a REAL child */
        n = n->left;
    }
    return n;
}
```
- **Prevention Rule**: In a Threaded BST, never write `node->left != NULL` or `node->right != NULL` to test for children; always test `!node->lthread` and `!node->rthread`.

---

### Bug 16: Mixing 0-Based and 1-Based Binary Heap Index Formulas

- **Symptom / Consequence**: Infinite loop at root `i = 0` (`2 * 0 = 0`, so root is treated as its own left child!) or off-by-one corruption of the heap property.
- **Why It Happens**: Textbook pseudocode often uses 1-based indexing (`arr[1..n]`) where `left = 2i`, `right = 2i+1`, `parent = i/2`. Using those formulas on a C 0-based array (`arr[0..n-1]`) maps index `0`'s left child to `2*0 = 0`!

| Relationship | **0-Based C Array (`arr[0 .. n-1]`, Root at `0`)** | **1-Based Array (`arr[1 .. n]`, Root at `1`)** |
| :--- | :--- | :--- |
| **Parent of `i`** | `(i - 1) / 2` | `i / 2` |
| **Left Child of `i`** | `2 * i + 1` | `2 * i` |
| **Right Child of `i`** | `2 * i + 2` | `2 * i + 1` |
| **Last Internal Node (Start of `build_heap`)** | `(n / 2) - 1` | `n / 2` |

```c
/* ❌ BUG: 1-based formulas applied to 0-based C array (2*0 == 0!) */
int left_buggy(int i)   { return 2 * i; }     /* ❌ Returns 0 when i == 0! */
int parent_buggy(int i) { return i / 2; }     /* ❌ Parent of right child 2 is 1, not 0! */

/* ✅ FIX: 0-based C array index macros/functions */
static inline int heap_parent(int i) { return (i - 1) / 2; }
static inline int heap_left(int i)   { return 2 * i + 1;   }
static inline int heap_right(int i)  { return 2 * i + 2;   }
```
- **Prevention Rule**: Always write helper functions `heap_parent(i)`, `heap_left(i)`, `heap_right(i)` at the top of your file and verify them for `i = 0` (`left = 1, right = 2`) and `i = 2` (`parent = (2-1)/2 = 0`).

---

## Category E: Unit 3 & Unit 4 — Graph, Trie & Hashing Bugs (Bugs 17–20)

### Bug 17: Marking BFS Vertices `visited` on Dequeue Instead of Enqueue

- **Symptom / Consequence**: The same unvisited vertex is pushed onto the BFS queue multiple times by different neighbors before it gets dequeued, causing duplicate output, corrupted `parent[]`/`dist[]` arrays, and queue overflow ($O(E)$ entries in a queue sized for $V$!).
- **Why It Happens**: Between the time vertex `w` is enqueued and when `w` is finally dequeued, other vertices currently in the queue may also inspect their edges to `w`. If `visited[w]` is still `false`, they enqueue `w` again!

```
Graph: Diamond 0 -> {1, 2}, and both 1 -> 3 and 2 -> 3
If marked on dequeue:
  Pop 0: pushes 1, 2 (Queue: [1, 2])
  Pop 1: sees 3 unvisited -> pushes 3 (Queue: [2, 3], visited[3] STILL FALSE!)
  Pop 2: sees 3 unvisited -> pushes 3 AGAIN! (Queue: [3, 3] ❌ DUPLICATE!)
```

```c
/* ❌ BUG: Marks visited when popping from queue -> duplicate enqueues! */
while (!queue_empty(&q)) {
    int u = dequeue(&q);
    visited[u] = true; /* ❌ TOO LATE! Neighbors already enqueued 'u' multiple times! */
    for (Edge *e = adj[u]; e != NULL; e = e->next) {
        if (!visited[e->to]) enqueue(&q, e->to);
    }
}

/* ✅ FIX: Always mark visited[v] = true AT THE EXACT MOMENT v IS ENQUEUED! */
visited[start] = true;
enqueue(&q, start);
while (!queue_empty(&q)) {
    int u = dequeue(&q);
    for (Edge *e = adj[u]; e != NULL; e = e->next) {
        int v = e->to;
        if (!visited[v]) {
            visited[v] = true; /* ✅ Marked BEFORE enqueueing: never enqueued twice! */
            dist[v] = dist[u] + 1;
            enqueue(&q, v);
        }
    }
}
```
- **Prevention Rule**: In **BFS**, mark `visited` **before `enqueue()`** (so each vertex enters the queue at most once). (In iterative stack-based DFS, you either mark on pop with a `if (visited[u]) continue;` guard or mark on push).

---

### Bug 18: Destroying Shared Prefix Nodes During Trie Deletion

- **Symptom / Consequence**: Deleting `"CAR"` from a Trie that also contains `"CART"` or `"CARD"` destroys the `'C' -> 'A' -> 'R'` path, silently deleting `"CART"` and `"CARD"` and causing dangling pointers!
- **Why It Happens**: Naive deletion frees every node along the word's path without checking whether the node has other children (`has_children(node)`) or marks the end of another shorter word (`is_end_of_word`).

```
Trie containing "CAR" and "CART":
  (root) --C--> ( ) --A--> ( ) --R--> [is_end=true] --T--> [is_end=true]
                                            ^
   Deleting "CAR" must ONLY set is_end=false here; it MUST NOT free the 'R' node!
```

```c
/* ❌ BUG: Unconditionally frees nodes along the path of 'word'! */
void trie_delete_buggy(TrieNode *curr, const char *word) {
    if (*word == '\0') return;
    int idx = *word - 'a';
    trie_delete_buggy(curr->children[idx], word + 1);
    free(curr->children[idx]); /* ❌ Destroys shared prefixes used by longer words! */
    curr->children[idx] = NULL;
}

/* ✅ FIX: Postorder recursive delete that ONLY frees a node if it has NO children
 *         and is NOT the end of another word! */
bool trie_has_children(const TrieNode *node) {
    for (int i = 0; i < 26; i++) {
        if (node->children[i] != NULL) return true;
    }
    return false;
}

bool trie_delete_safe(TrieNode *curr, const char *word) {
    if (curr == NULL) return false;
    if (*word == '\0') {
        if (!curr->is_end) return false; /* Word wasn't in Trie */
        curr->is_end = false;            /* Unmark word end     */
        return !trie_has_children(curr); /* True iff caller can safely free this node */
    }
    int idx = *word - 'a';
    if (trie_delete_safe(curr->children[idx], word + 1)) {
        free(curr->children[idx]);
        curr->children[idx] = NULL;
        /* Prune upward ONLY if curr is not another word's end and has no other children */
        return !curr->is_end && !trie_has_children(curr);
    }
    return false;
}
```
- **Prevention Rule**: Three cases in Trie deletion of $W$:
  1. $W$ is a prefix of a longer word (`has_children == true` at terminal node): Only set `is_end = false`; free **0** nodes.
  2. $W$ shares a prefix with a shorter word (`is_end == true` on an ancestor): Free bottom-up only until the lowest ancestor with `is_end == true` or `children > 1`.
  3. $W$ shares no nodes with any other word: Free all $|W|$ nodes bottom-up.

---

### Bug 19: Setting Open-Addressed Hash Slot to `EMPTY` Instead of `DELETED` (Tombstone)

- **Symptom / Consequence**: Silent data loss! Searching for a key $k_2$ that was inserted *after* a colliding key $k_1$ returns `NOT_FOUND` as soon as $k_1$ is deleted.
- **Why It Happens**: Open addressing (Linear/Quadratic Probing, Double Hashing) stops searching when it hits an `EMPTY` slot. If deleting $k_1$ resets its slot to `EMPTY`, it breaks the probe chain bridge needed to reach $k_2$!

```
Table size m = 7, h(k) = k % 7. Insert 10 (slot 3), Insert 17 (collides at 3 -> slot 4):
Index:   0     1     2      3        4       5     6
       +-----+-----+-----+-------+-------+-----+-----+
       | EMP | EMP | EMP |  10   |  17   | EMP | EMP |
       +-----+-----+-----+-------+-------+-----+-----+

If Delete(10) sets slot 3 to EMPTY:
       | EMP | EMP | EMP |  EMP  |  17   | EMP | EMP |
                             ^
  Search(17) hashes to 17 % 7 = 3, sees EMPTY at slot 3, and aborts! 17 is LOST!
```

```c
/* ❌ BUG: Marks deleted slot as EMPTY, breaking probe chains for later keys! */
void hash_delete_buggy(HashTable *ht, int key) {
    int idx = find_slot(ht, key);
    if (idx != -1) {
        ht->state[idx] = SLOT_EMPTY; /* ❌ Breaks search for keys probed past idx! */
    }
}

/* ✅ FIX: Use 3 slot states (EMPTY, OCCUPIED, DELETED/TOMBSTONE):
 * - Search continues probing past DELETED slots, stopping only at EMPTY.
 * - Insert can reuse the FIRST DELETED slot encountered (after verifying key isn't present further down). */
void hash_delete_safe(HashTable *ht, int key) {
    int idx = find_slot(ht, key);
    if (idx != -1) {
        ht->state[idx] = SLOT_DELETED; /* ✅ Tombstone preserves probe chain! */
        ht->size--;
    }
}
```
- **Prevention Rule**: Open-addressed hash tables with deletion **must** use three states: `EMPTY`, `OCCUPIED`, and `DELETED` (Tombstone).

---

### Bug 20: $h_2(k) == 0$ in Double Hashing & Naive `memcpy` During Rehashing

- **Symptom / Consequence**:
  1. If $h_2(k) = 0$, the probe sequence $(h_1(k) + i \cdot 0) \bmod m = h_1(k)$ checks the **exact same slot forever** in an infinite loop!
  2. If rehashing copies buckets directly (`new_buckets[i] = old_buckets[i]`) without re-evaluating `hash(key) % new_m`, every key lands in the wrong bucket ($k \bmod 7 \ne k \bmod 17$)!

```c
/* ❌ BUG 20A: Secondary hash using modulo can return 0 when key is a multiple of R! */
int h2_buggy(int key) {
    return key % 7; /* ❌ Returns 0 for key = 0, 7, 14, 21 -> INFINITE LOOP! */
}

/* ✅ FIX 20A: Use R - (key % R) where R < m is prime; range is [1 .. R], NEVER 0! */
int h2_safe(int key, int R) {
    return R - (abs(key) % R); /* ✅ Strictly in [1, R], coprime to prime m! */
}

/* ❌ BUG 20B: Resizing hash table via realloc/memcpy without re-hashing keys! */
ht->buckets = realloc(ht->buckets, new_m * sizeof(Slot)); /* ❌ Keys are at key % old_m! */

/* ✅ FIX 20B: Allocate a fresh table of size new_m and re-insert every OCCUPIED key
 * using new_idx = hash(key) % new_m (simultaneously purging all DELETED tombstones!) */
```
- **Prevention Rule**:
  1. $h_2(k)$ in Double Hashing must **never evaluate to $0$** and must be coprime to $m$.
  2. Rehashing **always** requires allocating a new array and re-computing `hash(key) % new_m` for every active key.

---

## 6. Master Edge-Case Checklist Before Submitting Any Exam/Lab Program

Before handing in any C Data Structures code, trace your function mentally against these **10 mandatory boundary inputs**:

| # | Boundary Scenario | What to Verify in Your Code |
| :--- | :--- | :--- |
| **1** | **Empty Structure (`head == NULL`, `root == NULL`, `size == 0`)** | Does `search`, `delete`, `peek`, or `traverse` immediately check for `NULL`/`0` without dereferencing `head->next` or `root->key`? |
| **2** | **Single-Element Structure ($n = 1$)** | When deleting the **only** node, do **both** `head` AND `tail` (or `front` AND `rear`) become `NULL`? In CLL, does `head->next == head` free cleanly and set `head = NULL`? |
| **3** | **Two-Element Structure ($n = 2$)** | In Fast/Slow middle finding, does `slow` stop at index `0` (`1st` node) so Merge Sort doesn't infinitely recurse on a 2-node list? |
| **4** | **Operation at the Very First Element (`head` / index `0`)** | Does inserting/deleting at `head` update the caller's `head` pointer (and `new_head->prev = NULL` in DLL, `tail->next = new_head` in CLL)? |
| **5** | **Operation at the Very Last Element (`tail` / index `n-1`)** | Does inserting/deleting at `tail` avoid `NULL->prev` dereference and properly set the new `tail->next = NULL`? |
| **6** | **Full Fixed-Capacity Structure (`size == CAP`, `top == CAP - 1`)** | Does `push`/`enqueue`/`insert` reject or resize *before* writing to `arr[CAP]`? |
| **7** | **Wrap-Around Boundary in Circular Structures (`idx == 0` or `CAP - 1`)** | Does stepping backward from `0` use `(idx - 1 + CAP) % CAP` (giving `CAP - 1`) instead of `(0 - 1) % CAP` (giving `-1`)? |
| **8** | **Duplicate Keys & Non-Existent Keys** | What happens if `insert(k)` is called with an existing key (in BST/AVL/Trie/Hash Table)? What if `delete(k)` is called for a key not in the structure? |
| **9** | **Degenerate / Skewed Input ($1, 2, 3, \dots, n$ sorted order)** | Does your BST/AVL/Splay/Heap handle sorted or reverse-sorted insertions without corrupting parent/child pointers? |
| **10** | **Disconnected Graph / Self-Loops / Cycles** | Does your graph algorithm loop over all $u \in [0, V-1]$ to visit disconnected components, and does `visited[]` prevent infinite loops on cycles? |

---

> **Next Step**: Continue to **[Appendix 04: Practice Questions & Answers](./04-Practice-Questions-and-Answers.md)** for the complete Final Examination Question Bank, C11 Programming Solutions, and Step-by-Step Worked Hand Traces.
