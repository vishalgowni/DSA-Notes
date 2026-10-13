# Expression Trees: Construction, Traversals, and Evaluation

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [What Is an Expression Tree?](#11-what-is-an-expression-tree)
   - 1.2 [Why Do We Need Expression Trees?](#12-why-do-we-need-expression-trees)
   - 1.3 [Relationship to Prior Concepts](#13-relationship-to-prior-concepts)
   - 1.4 [Learning Objectives](#14-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Formal Structural Definition](#21-formal-structural-definition)
   - 2.2 [Why Expression Trees Do Not Store Parentheses](#22-why-expression-trees-do-not-store-parentheses)
   - 2.3 [Terminology Table](#23-terminology-table)
   - 2.4 [Mathematical Properties and Structural Invariants](#24-mathematical-properties-and-structural-invariants)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Node and Subtree-Stack Data Structures in C11](#31-node-and-subtree-stack-data-structures-in-c11)
   - 3.2 [Heap Memory Layout and Subtree Stack Diagram](#32-heap-memory-layout-and-subtree-stack-diagram)
4. [Operations and Algorithms](#4-operations-and-algorithms)
   - 4.1 [Construction of an Expression Tree from a Postfix Expression](#41-construction-of-an-expression-tree-from-a-postfix-expression)
   - 4.2 [Step-by-Step Worked Trace 1: Symbolic Postfix `a b + c d e + * *`](#42-step-by-step-worked-trace-1-symbolic-postfix-a-b--c-d-e---)
   - 4.3 [Step-by-Step Worked Trace 2: Arithmetic Postfix `15 7 1 1 + - / 3 * 2 1 1 + + -`](#43-step-by-step-worked-trace-2-arithmetic-postfix-15-7-1-1---3--2-1-1---)
   - 4.4 [Traversals and Algebraic Notation Relationships (Prefix, Postfix, Infix)](#44-traversals-and-algebraic-notation-relationships-prefix-postfix-infix)
   - 4.5 [Bottom-Up Recursive Postorder Evaluation](#45-bottom-up-recursive-postorder-evaluation)
   - 4.6 [Complete, Compilable C11 Implementation](#46-complete-compilable-c11-implementation)
5. [Comparison Tables and Complexity Summary](#5-comparison-tables-and-complexity-summary)
   - 5.1 [Comparison of Algebraic Representations](#51-comparison-of-algebraic-representations)
   - 5.2 [Time and Space Complexity Summary](#52-time-and-space-complexity-summary)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Concrete Bug vs. Fix Examples in C](#61-concrete-bug-vs-fix-examples-in-c)
   - 6.2 [Boundary Conditions Checklist](#62-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)
   - 7.1 [Key Takeaways and Core Invariants](#71-key-takeaways-and-core-invariants)
   - 7.2 [Self-Test Revision Checklist](#72-self-test-revision-checklist)
   - 7.3 [Navigation and Related Topics](#73-navigation-and-related-topics)

---

## 1. Introduction and Learning Objectives

### 1.1 What Is an Expression Tree?

An **Expression Tree** is a specialized binary tree (a concrete form of an **Abstract Syntax Tree**, or **AST**) that represents an algebraic, arithmetic, or boolean expression hierarchically:
- Every **leaf node (external node)** represents an **operand**—either a numeric constant (such as `15`, `3.14`) or a symbolic variable (such as `a`, `x`, `count`).
- Every **internal node** represents an **operator**—either a binary operator (such as `+`, `-`, `*`, `/`, `^`) with two children (`left` and `right` operands) or a unary operator (such as unary negation `-` or logical NOT `!`) with a single operand child.

### 1.2 Why Do We Need Expression Trees?

Linear strings written in human-friendly **Infix Notation**—such as `a + b * c` or `(a + b) * c`—are inherently ambiguous without external rules for **operator precedence**, **associativity**, and **explicit parentheses**. While a linear Postfix (Reverse Polish) string allows left-to-right stack evaluation, it destroys the hierarchical view needed when we want to inspect, transform, differentiate, or optimize sub-expressions.

An **Expression Tree** solves both problems simultaneously and serves as the foundational intermediate representation in:
1. **Compilers and Interpreters** (GCC, Clang, JVM, Python): Parsing source code expressions into ASTs to perform type checking, constant folding (e.g., replacing `3 + 5` with `8` at compile time), and machine-code register allocation.
2. **Computer Algebra Systems (SymPy, Mathematica, Maple)**: Performing symbolic differentiation (e.g., applying the product rule $\frac{d}{dx}(u \cdot v) = u'v + uv'$ recursively on subtrees) and algebraic simplification.
3. **Spreadsheet & Database Query Engines**: Representing formula dependencies and filter predicates (`WHERE (salary > 50000) AND (dept = 'CS')`) for repeated fast evaluation.

### 1.3 Relationship to Prior Concepts

- **Stacks and Postfix Evaluation (Unit 1)**: In Unit 1, we evaluated a postfix expression by pushing numbers onto a stack and popping two numbers whenever an operator appeared. Constructing an Expression Tree uses the **exact same stack discipline**, except the stack holds **pointers to tree nodes (`ExprNode*`)** instead of raw numbers!
- **Binary Tree Traversals (Unit 2)**: Applying the three classic depth-first traversals (`Preorder`, `Inorder`, `Postorder`) to an Expression Tree directly yields the three classic algebraic notations (**Prefix / Polish**, **Infix**, and **Postfix / Reverse Polish**).

### 1.4 Learning Objectives

After studying this module, you will be able to:

1. Explain why Expression Trees encode operator precedence and associativity purely through tree topology without storing parentheses.
2. Trace and implement the **$O(n)$ Stack-Based Postfix-to-Expression-Tree Construction Algorithm** for both symbolic and multi-digit arithmetic expressions.
3. Prove why a naive Inorder Traversal loses necessary parentheses, and implement both **Fully-Parenthesized Inorder** and **Precedence-Aware Minimal-Parentheses Inorder** traversals.
4. Implement **Recursive Postorder Evaluation** of an Expression Tree in C11 with error handling for division by zero and malformed inputs.

---

## 2. Definitions and Terminology

### 2.1 Formal Structural Definition

> **Definition (Expression Tree):**
> An **Expression Tree** $T$ over an operand set $\Sigma_{\text{opd}}$ and an operator set $\Sigma_{\text{opr}} = \Sigma_{\text{bin}} \cup \Sigma_{\text{un}}$ is a rooted binary tree defined inductively:
> 1. **Base Case (Operand Leaf)**: A single node $v$ labeled with an operand $x \in \Sigma_{\text{opd}}$ (where $v.\text{left} = \text{NULL}$ and $v.\text{right} = \text{NULL}$) is a valid Expression Tree.
> 2. **Inductive Case 1 (Unary Operator)**: If $T_1$ is a valid Expression Tree and $\odot \in \Sigma_{\text{un}}$ is a unary operator, then a root node $r$ labeled $\odot$ with $r.\text{left} = \text{NULL}$ and $r.\text{right} = \text{root}(T_1)$ is a valid Expression Tree representing $(\odot \, T_1)$.
> 3. **Inductive Case 2 (Binary Operator)**: If $T_L$ and $T_R$ are valid Expression Trees and $\oplus \in \Sigma_{\text{bin}}$ is a binary operator, then a root node $r$ labeled $\oplus$ with $r.\text{left} = \text{root}(T_L)$ and $r.\text{right} = \text{root}(T_R)$ is a valid Expression Tree representing $(T_L \oplus T_R)$.

### 2.2 Why Expression Trees Do Not Store Parentheses

In an Expression Tree, **evaluation flows strictly bottom-up from the leaves to the root**: a parent operator node cannot be evaluated until both of its child subtrees have been completely evaluated. Therefore, **any operator placed deeper in the tree is evaluated before its ancestors**, regardless of standard operator precedence!

Compare how `a + b * c` and `(a + b) * c` are represented:

```
   Expression 1:  a + b * c                        Expression 2:  (a + b) * c
   (Multiplication binds tighter than +)           (Parentheses force + to evaluate first)

             [+]  <-- evaluated 2nd                          [*]  <-- evaluated 2nd
            /   \                                           /   \
          [a]   [*]  <-- deeper: evaluated 1st            [+]   [c]
               /   \                                     /   \
             [b]   [c]                                 [a]   [b]  <-- deeper: evaluated 1st

   Notice: NEITHER tree stores '(' or ')' nodes!
   The tree topology alone uniquely dictates the order of operations.
```

Similarly, associativity for repeated operators of equal precedence is captured by whether the tree leans left or right:

```
   Left-Associative Subtraction: (a - b) - c       Right-Associative Exponentiation: a ^ (b ^ c)

                [-]                                             [^]
               /   \                                           /   \
             [-]   [c]                                       [a]   [^]
            /   \                                                 /   \
          [a]   [b]                                             [b]   [c]
```

### 2.3 Terminology Table

| Term | Formal / Structural Meaning | Example |
| :--- | :--- | :--- |
| **Operand Node (Leaf)** | External node ($d_{\text{out}} = 0$) storing a constant literal or variable identifier. | `15`, `3.14`, `a`, `x` |
| **Binary Operator Node** | Internal node ($d_{\text{out}} = 2$) storing an operator that combines `left` and `right` subtrees. | `+`, `-`, `*`, `/`, `^` |
| **Unary Operator Node** | Internal node ($d_{\text{out}} = 1$) operating on a single child subtree (stored in `right` by convention). | Unary negation `~` (representing `-x`) |
| **Prefix (Polish) Notation** | Operator precedes its operands (`op A B`); produced by **Preorder Traversal**. | `* + a b c` for `(a + b) * c` |
| **Postfix (Reverse Polish / RPN)** | Operator follows its operands (`A B op`); produced by **Postorder Traversal**. | `a b + c *` for `(a + b) * c` |
| **Infix Notation** | Operator appears between operands (`A op B`); produced by **Parenthesized Inorder Traversal**. | `((a + b) * c)` |

### 2.4 Mathematical Properties and Structural Invariants

For any valid Expression Tree containing **only binary operators** and operands:
1. **Strict (Full / Proper) Binary Tree Property**: Every node has either $0$ children (an operand leaf) or $2$ children (a binary operator). No node has degree $1$.
2. **Operand-to-Operator Count Identity**:
   Let $I$ be the number of binary operator nodes (internal nodes) and $L$ be the number of operand nodes (leaves). By the Full Binary Tree Theorem:
   $$L = I + 1$$
   and the total number of tokens (nodes) in the expression is:
   $$n = L + I = 2I + 1 \quad (\text{always an odd integer!})$$
3. **Postfix Validity Condition (Running Stack Depth Invariant)**:
   When scanning a binary-operator postfix expression of $k$ tokens left-to-right, let $s_i$ be the number of subtrees on the stack after processing token $i$ ($1 \le i \le k$):
   - Every operand increments the stack size by $+1$ ($\Delta s = +1$).
   - Every binary operator pops $2$ subtrees and pushes $1$ new subtree, changing the stack size by $-2 + 1 = -1$ ($\Delta s = -1$).
   - A postfix expression is structurally valid if and only if:
     $$\forall i \in \{1, \dots, k - 1\}, \quad s_i \ge 1 \qquad \text{and} \qquad s_k = 1$$

---

## 3. Representation and Memory Layout in C

### 3.1 Node and Subtree-Stack Data Structures in C11

To support numeric constants, symbolic variables, binary operators, and unary negation within a single unified C program, we define `ExprNode` and an explicit array-backed pointer stack `ExprStack`:

```c
#include <stdbool.h>

typedef enum {
    NODE_OPERAND_NUM,      /* Numeric literal leaf (e.g., 15, 3.5) */
    NODE_OPERAND_VAR,      /* Symbolic variable leaf (e.g., "a", "x") */
    NODE_OPERATOR_BINARY,  /* Binary operator internal node (+, -, *, /, ^) */
    NODE_OPERATOR_UNARY    /* Unary operator internal node (~ for unary minus) */
} ExprNodeType;

typedef struct ExprNode {
    ExprNodeType type;
    double num_val;          /* Numeric value when type == NODE_OPERAND_NUM */
    char symbol[16];         /* Printable token string ("15", "a", "+", "^") */
    char op;                 /* Operator character ('+', '-', '*', '/', '^', '~') */
    struct ExprNode *left;   /* Left operand subtree (NULL for leaves & unary ops) */
    struct ExprNode *right;  /* Right operand subtree (or sole operand for unary) */
} ExprNode;

#define MAX_STACK_CAP 128

typedef struct {
    ExprNode *items[MAX_STACK_CAP]; /* Stack of pointers to heap-allocated subtree roots */
    int top;                        /* Index of top element (-1 when empty) */
} ExprStack;
```

### 3.2 Heap Memory Layout and Subtree Stack Diagram

During postfix construction, `ExprStack` does **not** copy nodes by value; its `items[]` array holds `8-byte` `ExprNode*` pointers referencing subtree roots on the heap.

```
ExprStack (on main/function stack frame):          Heap-Allocated Subtrees:
+-------+------------+
|  top  |     1      |                             0x6000 (Subtree for "a + b"):
+-------+------------+                                    [+]
| items | [0] 0x6000 | ----------------------------->    /   \
|       | [1] 0x6090 | --------------+                 [a]   [b]
|       | [2]  ...   |               |
+-------+------------+               |             0x6090 (Subtree for "c * (d + e)"):
                                     +-------------->     [*]
                                                         /   \
                                                       [c]   [+]
                                                            /   \
                                                          [d]   [e]
```

---

## 4. Operations and Algorithms

### 4.1 Construction of an Expression Tree from a Postfix Expression

- **Purpose**: Convert a whitespace-delimited Postfix (RPN) string into its corresponding linked `ExprNode*` Expression Tree in $O(n)$ time.
- **Preconditions**: Input string is a valid postfix expression where tokens are separated by spaces.
- **Step-by-Step Algorithm**:
  1. Initialize an empty stack `S` of `ExprNode*` pointers.
  2. Tokenize the postfix expression from **left to right**. For each token `tok`:
     - **Case A (`tok` is an Operand—number or variable)**:
       1. Allocate a new leaf `ExprNode` containing `tok` with `left = NULL` and `right = NULL`.
       2. Push the pointer to this leaf node onto `S`.
     - **Case B (`tok` is a Unary Operator `~`)**:
       1. Pop **one** pointer `operand = Pop(S)`. (If `S` was empty, abort: malformed expression).
       2. Allocate a new unary operator `ExprNode` with `left = NULL` and `right = operand`.
       3. Push the new operator node pointer onto `S`.
     - **Case C (`tok` is a Binary Operator `+`, `-`, `*`, `/`, `^`)**:
       1. **CRITICAL ORDER**: Pop the first pointer from `S` and assign it to **`right`** (`right = Pop(S)`).
       2. Pop the second pointer from `S` and assign it to **`left`** (`left = Pop(S)`).
          *(Why? Because in postfix `left right op`, `left` was pushed first and `right` was pushed second, so `right` sits at the top of the LIFO stack!)*
       3. Allocate a new binary operator `ExprNode` with `node->left = left` and `node->right = right`.
       4. Push the new operator node pointer onto `S`.
  3. After all tokens are consumed, pop `root = Pop(S)`.
  4. Verify that `S` is now empty. If `S` is not empty, the postfix string had too many operands (malformed); free all remaining nodes and return `NULL`. Otherwise, return `root`.

#### Pseudocode

```
ALGORITHM Build-Expression-Tree(postfix_tokens[1..n])
Input:  Array of n tokens in Postfix (Reverse Polish) notation
Output: Pointer to root of the constructed Expression Tree

1. S <- Empty-Stack()
2. FOR i <- 1 TO n DO
3.     tok <- postfix_tokens[i]
4.     IF Is-Operand(tok) THEN
5.         node <- Create-Leaf-Node(tok)
6.         Push(S, node)
7.     ELSE IF Is-Unary-Operator(tok) THEN
8.         right_child <- Pop(S)
9.         node <- Create-Operator-Node(tok, NULL, right_child)
10.        Push(S, node)
11.    ELSE IF Is-Binary-Operator(tok) THEN
12.        right_child <- Pop(S)      // Top of stack is RIGHT operand!
13.        left_child  <- Pop(S)      // Second from top is LEFT operand!
14.        node <- Create-Operator-Node(tok, left_child, right_child)
15.        Push(S, node)
16. root <- Pop(S)
17. IF NOT Is-Empty(S) THEN
18.     ERROR "Malformed Postfix Expression: excess operands"
19. RETURN root
```

### 4.2 Step-by-Step Worked Trace 1: Symbolic Postfix `a b + c d e + * *`

Let us trace `a b + c d e + * *` (which represents $(a + b) \times (c \times (d + e))$) through all 9 tokens, showing the exact contents of the subtree stack `S` (from bottom to top) at every step:

```
Token 1: 'a' (Operand) -> Push leaf [a]
  Stack S:  [a]

Token 2: 'b' (Operand) -> Push leaf [b]
  Stack S:  [a] , [b]

Token 3: '+' (Binary Op) -> Pop right=[b], Pop left=[a], Push subtree (+)
  Stack S:      [+]
               /   \
             [a]   [b]

Token 4: 'c' (Operand) -> Push leaf [c]
  Stack S:      [+]    ,   [c]
               /   \
             [a]   [b]

Token 5: 'd' (Operand) -> Push leaf [d]
  Stack S:      [+]    ,   [c]  ,  [d]
               /   \
             [a]   [b]

Token 6: 'e' (Operand) -> Push leaf [e]
  Stack S:      [+]    ,   [c]  ,  [d]  ,  [e]
               /   \
             [a]   [b]

Token 7: '+' (Binary Op) -> Pop right=[e], Pop left=[d], Push subtree (+)
  Stack S:      [+]    ,   [c]  ,      [+]
               /   \                  /   \
             [a]   [b]              [d]   [e]

Token 8: '*' (Binary Op) -> Pop right=(d+e), Pop left=[c], Push subtree (*)
  Stack S:      [+]    ,        [*]
               /   \           /   \
             [a]   [b]       [c]   [+]
                                  /   \
                                [d]   [e]

Token 9: '*' (Binary Op) -> Pop right=(c*(d+e)), Pop left=(a+b), Push root (*)
  Stack S (Final Single Tree Root):
                         [*]
                       /     \
                     [+]     [*]
                    /   \   /   \
                  [a]  [b] [c]  [+]
                               /   \
                             [d]   [e]
```

### 4.3 Step-by-Step Worked Trace 2: Arithmetic Postfix `15 7 1 1 + - / 3 * 2 1 1 + + -`

Let us trace how the multi-digit arithmetic postfix expression `15 7 1 1 + - / 3 * 2 1 1 + + -` builds its tree and evaluates:

| Step | Token | Action Taken | Stack `S` Subtrees (shown as equivalent infix sub-expressions) |
| :---: | :---: | :--- | :--- |
| 1 | `15` | Push leaf `15` | `[ 15 ]` |
| 2 | `7` | Push leaf `7` | `[ 15, 7 ]` |
| 3 | `1` | Push leaf `1` | `[ 15, 7, 1 ]` |
| 4 | `1` | Push leaf `1` | `[ 15, 7, 1, 1 ]` |
| 5 | `+` | Pop `1`, `1`; push `(1 + 1)` | `[ 15, 7, (1 + 1) ]` |
| 6 | `-` | Pop `right=(1+1)`, `left=7`; push `(7 - (1 + 1))` | `[ 15, (7 - (1 + 1)) ]` |
| 7 | `/` | Pop `right=(7-(1+1))`, `left=15`; push `(15 / (7 - (1 + 1)))` | `[ (15 / (7 - (1 + 1))) ]` |
| 8 | `3` | Push leaf `3` | `[ (15 / (7 - (1 + 1))), 3 ]` |
| 9 | `*` | Pop `3` and `(15/...)`; push `((15 / (7 - (1 + 1))) * 3)` | `[ ((15 / (7 - (1 + 1))) * 3) ]` |
| 10 | `2` | Push leaf `2` | `[ ((15 / (7 - (1 + 1))) * 3), 2 ]` |
| 11 | `1` | Push leaf `1` | `[ ((15 / (7 - (1 + 1))) * 3), 2, 1 ]` |
| 12 | `1` | Push leaf `1` | `[ ((15 / (7 - (1 + 1))) * 3), 2, 1, 1 ]` |
| 13 | `+` | Pop `1`, `1`; push `(1 + 1)` | `[ ((15 / (7 - (1 + 1))) * 3), 2, (1 + 1) ]` |
| 14 | `+` | Pop `(1+1)`, `2`; push `(2 + (1 + 1))` | `[ ((15 / (7 - (1 + 1))) * 3), (2 + (1 + 1)) ]` |
| 15 | `-` | Pop `right=(2+(1+1))`, `left=((15/..)*3)`; push root `-` | `[ (((15 / (7 - (1 + 1))) * 3) - (2 + (1 + 1))) ]` |

### 4.4 Traversals and Algebraic Notation Relationships (Prefix, Postfix, Infix)

Given an Expression Tree rooted at `root`:

1. **Preorder Traversal (`Root -> Left -> Right`) $\implies$ Prefix (Polish) Notation**:
   - Visits the operator before its left and right subtrees:
     - On `(a + b) * (c * (d + e))`, Preorder outputs: `* + a b * c + d e`
   - **Remarkable Property**: Prefix notation is completely unambiguous **without any parentheses**!

2. **Postorder Traversal (`Left -> Right -> Root`) $\implies$ Postfix (Reverse Polish) Notation**:
   - Visits both operand subtrees before their combining operator:
     - On `(a + b) * (c * (d + e))`, Postorder outputs: `a b + c d e + * *`
   - **Remarkable Property**: Postfix notation is also completely unambiguous **without any parentheses**!

3. **Inorder Traversal (`Left -> Root -> Right`) $\implies$ Infix Notation (and the Missing-Parentheses Trap!)**:
   - **Why Naive Inorder Fails**:
     Suppose we run a standard unparenthesized Inorder Traversal on the tree for `(a + b) * (c * (d + e))`:
     $$\text{Naive Inorder Output: } \quad \texttt{a + b * c * d + e}$$
     Under standard algebra rules, `a + b * c * d + e` means $a + (b \cdot c \cdot d) + e$, which is **mathematically wrong**!
   - **Solution 1 — Fully-Parenthesized Inorder Traversal**:
     Whenever visiting an internal operator node, print `(` **before** recursing into `node->left`, print `node->symbol`, and print `)` **after** recursing into `node->right`:
     ```c
     void print_infix_full_parens(const ExprNode *root) {
         if (!root) return;
         if (root->type == NODE_OPERATOR_BINARY) {
             printf("(");
             print_infix_full_parens(root->left);
             printf(" %s ", root->symbol);
             print_infix_full_parens(root->right);
             printf(")");
         } else if (root->type == NODE_OPERATOR_UNARY) {
             printf("(-");
             print_infix_full_parens(root->right);
             printf(")");
         } else {
             printf("%s", root->symbol); /* Leaf operand: no parens */
         }
     }
     ```
     Output: `((a + b) * (c * (d + e)))` — 100% unambiguous!
   - **Solution 2 — Precedence- and Associativity-Aware Minimal-Parentheses Inorder**:
     Only emit parentheses around a child subtree when:
     - The child operator has **strictly lower precedence** than the parent operator (e.g., `+` child under `*` parent), OR
     - The child operator has **equal precedence** and sits on the **non-associative side** of a non-commutative operator (e.g., right child of `-` or `/`, as in `a - (b - c)` or `a - (b + c)`, or left child of right-associative `^`, as in `(a ^ b) ^ c`), OR
     - A unary negation `-x` is the base (left child) of an exponentiation `^` (e.g., `(-3) ^ 2` vs. `-3 ^ 2`).

### 4.5 Bottom-Up Recursive Postorder Evaluation

- **Purpose**: Compute the numerical value of an Expression Tree whose leaves store numeric constants.
- **Key Insight**: An operator at `root` can only be applied **after** we know the numerical values of `root->left` and `root->right`. Therefore, **evaluation of an Expression Tree is inherently a Postorder Traversal**!

```
Bottom-Up Postorder Evaluation of: 15 7 1 1 + - / 3 * 2 1 1 + + -

                         [-]  ===================> 9 - 4 = 5.00
                       /     \
       (3 * 3 = 9)   [*]     [+]   (2 + 2 = 4)
                    /   \   /   \
     (15 / 5 = 3) [/]  [3] [2]  [+] (1 + 1 = 2)
                 /   \         /   \
              [15]   [-]     [1]   [1]
                    /   \
                  [7]   [+] (1 + 1 = 2) -> (7 - 2 = 5)
                       /   \
                     [1]   [1]
```

#### Pseudocode

```
ALGORITHM Evaluate-Expression-Tree(root)
Input:  Pointer root to an arithmetic Expression Tree
Output: Floating-point numerical value of the expression

1. IF root.type == NODE_OPERAND_NUM THEN
2.     RETURN root.num_val
3. IF root.type == NODE_OPERATOR_UNARY THEN
4.     rval <- Evaluate-Expression-Tree(root.right)
5.     RETURN -rval
6. lval <- Evaluate-Expression-Tree(root.left)     // Postorder: evaluate left subtree
7. rval <- Evaluate-Expression-Tree(root.right)    // Postorder: evaluate right subtree
8. SWITCH root.op DO                               // Visit root: apply operator
9.     CASE '+': RETURN lval + rval
10.    CASE '-': RETURN lval - rval
11.    CASE '*': RETURN lval * rval
12.    CASE '/':
13.        IF rval == 0 THEN ERROR "Division by zero"
14.        RETURN lval / rval
15.    CASE '^': RETURN Power(lval, rval)
```

### 4.6 Complete, Compilable C11 Implementation

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <string.h>
#include <ctype.h>
#include <math.h>

typedef enum {
    NODE_OPERAND_NUM,
    NODE_OPERAND_VAR,
    NODE_OPERATOR_BINARY,
    NODE_OPERATOR_UNARY
} ExprNodeType;

typedef struct ExprNode {
    ExprNodeType type;
    double num_val;          /* Used when type == NODE_OPERAND_NUM */
    char symbol[16];         /* Display token: e.g., "15", "a", "+", "^", "~" */
    char op;                 /* '+', '-', '*', '/', '^', or '~' (unary minus) */
    struct ExprNode *left;   /* Left operand (NULL for unary operator or leaf) */
    struct ExprNode *right;  /* Right operand (or single operand for unary '~') */
} ExprNode;

#define MAX_STACK_CAP 128

typedef struct {
    ExprNode *items[MAX_STACK_CAP];
    int top;
} ExprStack;

void stack_init(ExprStack *s) { s->top = -1; }
bool stack_is_empty(const ExprStack *s) { return s->top == -1; }

bool stack_push(ExprStack *s, ExprNode *node) {
    if (s->top >= MAX_STACK_CAP - 1) return false;
    s->items[++(s->top)] = node;
    return true;
}

ExprNode *stack_pop(ExprStack *s) {
    if (stack_is_empty(s)) return NULL;
    return s->items[(s->top)--];
}

ExprNode *create_num_node(double val, const char *raw_token) {
    ExprNode *node = (ExprNode *)malloc(sizeof(ExprNode));
    if (!node) { perror("malloc"); exit(EXIT_FAILURE); }
    node->type = NODE_OPERAND_NUM;
    node->num_val = val;
    strncpy(node->symbol, raw_token, sizeof(node->symbol) - 1);
    node->symbol[sizeof(node->symbol) - 1] = '\0';
    node->op = '\0';
    node->left = node->right = NULL;
    return node;
}

ExprNode *create_var_node(const char *var_name) {
    ExprNode *node = (ExprNode *)malloc(sizeof(ExprNode));
    if (!node) { perror("malloc"); exit(EXIT_FAILURE); }
    node->type = NODE_OPERAND_VAR;
    node->num_val = 0.0;
    strncpy(node->symbol, var_name, sizeof(node->symbol) - 1);
    node->symbol[sizeof(node->symbol) - 1] = '\0';
    node->op = '\0';
    node->left = node->right = NULL;
    return node;
}

ExprNode *create_op_node(char op, ExprNode *left, ExprNode *right, bool is_unary) {
    ExprNode *node = (ExprNode *)malloc(sizeof(ExprNode));
    if (!node) { perror("malloc"); exit(EXIT_FAILURE); }
    node->type = is_unary ? NODE_OPERATOR_UNARY : NODE_OPERATOR_BINARY;
    node->num_val = 0.0;
    node->op = op;
    node->symbol[0] = op;
    node->symbol[1] = '\0';
    node->left = left;
    node->right = right;
    return node;
}

bool is_binary_op_token(const char *tok) {
    return (strlen(tok) == 1) &&
           (tok[0] == '+' || tok[0] == '-' || tok[0] == '*' ||
            tok[0] == '/' || tok[0] == '^');
}

bool is_unary_op_token(const char *tok) {
    return (strcmp(tok, "~") == 0 || strcmp(tok, "neg") == 0);
}

void free_expr_tree(ExprNode *root) {
    if (!root) return;
    free_expr_tree(root->left);
    free_expr_tree(root->right);
    free(root);
}

/* Construct Expression Tree from whitespace-delimited Postfix string */
ExprNode *build_expr_tree_from_postfix(const char *postfix) {
    ExprStack stack;
    stack_init(&stack);

    char buffer[512];
    strncpy(buffer, postfix, sizeof(buffer) - 1);
    buffer[sizeof(buffer) - 1] = '\0';

    char *tok = strtok(buffer, " \t\r\n");
    while (tok != NULL) {
        if (is_unary_op_token(tok)) {
            ExprNode *operand = stack_pop(&stack);
            if (!operand) {
                fprintf(stderr, "[Error] Malformed postfix: missing operand for unary '%s'\n", tok);
                while (!stack_is_empty(&stack)) free_expr_tree(stack_pop(&stack));
                return NULL;
            }
            ExprNode *op_node = create_op_node('~', NULL, operand, true);
            stack_push(&stack, op_node);
        } else if (is_binary_op_token(tok)) {
            ExprNode *right = stack_pop(&stack);
            ExprNode *left  = stack_pop(&stack);
            if (!left || !right) {
                fprintf(stderr, "[Error] Malformed postfix: insufficient operands for '%s'\n", tok);
                free_expr_tree(left);
                free_expr_tree(right);
                while (!stack_is_empty(&stack)) free_expr_tree(stack_pop(&stack));
                return NULL;
            }
            ExprNode *op_node = create_op_node(tok[0], left, right, false);
            stack_push(&stack, op_node);
        } else {
            char *endptr = NULL;
            double val = strtod(tok, &endptr);
            if (endptr != tok && *endptr == '\0') {
                stack_push(&stack, create_num_node(val, tok));
            } else {
                stack_push(&stack, create_var_node(tok));
            }
        }
        tok = strtok(NULL, " \t\r\n");
    }

    ExprNode *root = stack_pop(&stack);
    if (!stack_is_empty(&stack)) {
        fprintf(stderr, "[Error] Malformed postfix: leftover subtrees on stack!\n");
        free_expr_tree(root);
        while (!stack_is_empty(&stack)) free_expr_tree(stack_pop(&stack));
        return NULL;
    }
    return root;
}

/* Preorder Traversal -> Prefix (Polish) Notation */
void print_prefix(const ExprNode *root) {
    if (!root) return;
    printf("%s ", root->symbol);
    print_prefix(root->left);
    print_prefix(root->right);
}

/* Postorder Traversal -> Postfix (Reverse Polish) Notation */
void print_postfix(const ExprNode *root) {
    if (!root) return;
    print_postfix(root->left);
    print_postfix(root->right);
    printf("%s ", root->symbol);
}

/* Fully-Parenthesized Inorder Traversal -> Unambiguous Infix Notation */
void print_infix_full_parens(const ExprNode *root) {
    if (!root) return;
    if (root->type == NODE_OPERATOR_BINARY) {
        printf("(");
        print_infix_full_parens(root->left);
        printf(" %s ", root->symbol);
        print_infix_full_parens(root->right);
        printf(")");
    } else if (root->type == NODE_OPERATOR_UNARY) {
        printf("(-");
        print_infix_full_parens(root->right);
        printf(")");
    } else {
        printf("%s", root->symbol);
    }
}

/* Precedence-Aware Minimal-Parentheses Infix Traversal */
int op_precedence(const ExprNode *node) {
    if (!node) return 100;
    if (node->type == NODE_OPERATOR_UNARY) return 4;
    if (node->type != NODE_OPERATOR_BINARY) return 100;
    switch (node->op) {
        case '+': case '-': return 1;
        case '*': case '/': return 2;
        case '^':           return 3;
        default:            return 100;
    }
}

void print_infix_minimal(const ExprNode *root) {
    if (!root) return;
    if (root->type == NODE_OPERAND_NUM || root->type == NODE_OPERAND_VAR) {
        printf("%s", root->symbol);
        return;
    }
    if (root->type == NODE_OPERATOR_UNARY) {
        printf("-");
        bool parens = (op_precedence(root->right) < op_precedence(root));
        if (parens) printf("(");
        print_infix_minimal(root->right);
        if (parens) printf(")");
        return;
    }

    int p_root  = op_precedence(root);
    int p_left  = op_precedence(root->left);
    int p_right = op_precedence(root->right);

    /* Left child needs parens if lower precedence, or equal for right-assoc '^',
     * or if a unary negation is the base of an exponentiation ((-3)^2 vs -3^2) */
    bool left_parens = (p_left < p_root) ||
                       (p_left == p_root && root->op == '^') ||
                       (root->op == '^' && root->left->type == NODE_OPERATOR_UNARY);

    /* Right child needs parens if lower precedence, or equal for non-commutative '-', '/' */
    bool right_parens = (p_right < p_root) ||
                        (p_right == p_root && (root->op == '-' || root->op == '/'));

    if (left_parens) printf("(");
    print_infix_minimal(root->left);
    if (left_parens) printf(")");

    printf(" %s ", root->symbol);

    if (right_parens) printf("(");
    print_infix_minimal(root->right);
    if (right_parens) printf(")");
}

/* Bottom-up Postorder Evaluation */
double evaluate_expr_tree(const ExprNode *root, bool *ok) {
    if (!root) { *ok = false; return 0.0; }
    if (root->type == NODE_OPERAND_NUM) {
        return root->num_val;
    }
    if (root->type == NODE_OPERAND_VAR) {
        fprintf(stderr, "[Eval Error] Cannot numerically evaluate symbolic variable '%s'\n", root->symbol);
        *ok = false;
        return 0.0;
    }
    if (root->type == NODE_OPERATOR_UNARY) {
        double rval = evaluate_expr_tree(root->right, ok);
        return (*ok) ? -rval : 0.0;
    }

    double lval = evaluate_expr_tree(root->left, ok);
    if (!(*ok)) return 0.0;
    double rval = evaluate_expr_tree(root->right, ok);
    if (!(*ok)) return 0.0;

    switch (root->op) {
        case '+': return lval + rval;
        case '-': return lval - rval;
        case '*': return lval * rval;
        case '/':
            if (fabs(rval) < 1e-12) {
                fprintf(stderr, "[Eval Error] Division by zero!\n");
                *ok = false;
                return 0.0;
            }
            return lval / rval;
        case '^': return pow(lval, rval);
        default:
            *ok = false;
            return 0.0;
    }
}

void print_tree_2d(const ExprNode *root, int depth) {
    if (!root) return;
    print_tree_2d(root->right, depth + 1);
    for (int i = 0; i < depth; i++) printf("    ");
    printf("[%s]\n", root->symbol);
    print_tree_2d(root->left, depth + 1);
}

int main(void) {
    const char *symbolic_postfix = "a b + c d e + * *";
    printf("=== 1. SYMBOLIC EXPRESSION TREE: \"%s\" ===\n", symbolic_postfix);
    ExprNode *sym_root = build_expr_tree_from_postfix(symbolic_postfix);
    if (sym_root) {
        printf("Prefix  (Preorder)          : "); print_prefix(sym_root); printf("\n");
        printf("Postfix (Postorder)         : "); print_postfix(sym_root); printf("\n");
        printf("Infix   (Full Parentheses)  : "); print_infix_full_parens(sym_root); printf("\n");
        printf("Infix   (Minimal Precedence): "); print_infix_minimal(sym_root); printf("\n\n");
        printf("2D Rotated Tree View (Right subtree at top, Left at bottom):\n");
        print_tree_2d(sym_root, 0);
        free_expr_tree(sym_root);
    }

    printf("\n=== 2. ARITHMETIC EXPRESSION TREE ===\n");
    const char *arith_postfix = "15 7 1 1 + - / 3 * 2 1 1 + + -";
    printf("Input Postfix               : %s\n", arith_postfix);
    ExprNode *arith_root = build_expr_tree_from_postfix(arith_postfix);
    if (arith_root) {
        printf("Prefix  (Preorder)          : "); print_prefix(arith_root); printf("\n");
        printf("Postfix (Postorder)         : "); print_postfix(arith_root); printf("\n");
        printf("Infix   (Full Parentheses)  : "); print_infix_full_parens(arith_root); printf("\n");
        printf("Infix   (Minimal Precedence): "); print_infix_minimal(arith_root); printf("\n");

        bool ok = true;
        double result = evaluate_expr_tree(arith_root, &ok);
        if (ok) {
            printf("Evaluated Numerical Result  : %.2f\n", result);
        }
        free_expr_tree(arith_root);
    }

    printf("\n=== 3. UNARY NEGATION & EXPONENTIATION DEMO ===\n");
    const char *exp_postfix = "3 ~ 2 ^ 4 2 ^ +";
    printf("Input Postfix               : %s\n", exp_postfix);
    ExprNode *exp_root = build_expr_tree_from_postfix(exp_postfix);
    if (exp_root) {
        printf("Infix   (Full Parentheses)  : "); print_infix_full_parens(exp_root); printf("\n");
        printf("Infix   (Minimal Precedence): "); print_infix_minimal(exp_root); printf("\n");
        bool ok = true;
        double result = evaluate_expr_tree(exp_root, &ok);
        if (ok) {
            printf("Evaluated Numerical Result  : %.2f\n", result);
        }
        free_expr_tree(exp_root);
    }
    return 0;
}
```

#### Program Output:

```text
=== 1. SYMBOLIC EXPRESSION TREE: "a b + c d e + * *" ===
Prefix  (Preorder)          : * + a b * c + d e 
Postfix (Postorder)         : a b + c d e + * * 
Infix   (Full Parentheses)  : ((a + b) * (c * (d + e)))
Infix   (Minimal Precedence): (a + b) * c * (d + e)

2D Rotated Tree View (Right subtree at top, Left at bottom):
            [e]
        [+]
            [d]
    [*]
        [c]
[*]
        [b]
    [+]
        [a]

=== 2. ARITHMETIC EXPRESSION TREE ===
Input Postfix               : 15 7 1 1 + - / 3 * 2 1 1 + + -
Prefix  (Preorder)          : - * / 15 - 7 + 1 1 3 + 2 + 1 1 
Postfix (Postorder)         : 15 7 1 1 + - / 3 * 2 1 1 + + - 
Infix   (Full Parentheses)  : (((15 / (7 - (1 + 1))) * 3) - (2 + (1 + 1)))
Infix   (Minimal Precedence): 15 / (7 - (1 + 1)) * 3 - (2 + 1 + 1)
Evaluated Numerical Result  : 5.00

=== 3. UNARY NEGATION & EXPONENTIATION DEMO ===
Input Postfix               : 3 ~ 2 ^ 4 2 ^ +
Infix   (Full Parentheses)  : (((-3) ^ 2) + (4 ^ 2))
Infix   (Minimal Precedence): (-3) ^ 2 + 4 ^ 2
Evaluated Numerical Result  : 25.00
```

---

## 5. Comparison Tables and Complexity Summary

### 5.1 Comparison of Algebraic Representations

| Dimension | Infix String (`(a + b) * c`) | Prefix / Polish (`* + a b c`) | Postfix / RPN (`a b + c *`) | Expression Tree (`ExprNode*`) |
| :--- | :--- | :--- | :--- | :--- |
| **Needs Parentheses?** | **Yes** (mandatory to override precedence) | **No** (unambiguous) | **No** (unambiguous) | **No** (encoded in tree depth) |
| **Human Readability** | Highest (standard textbook math) | Moderate | Low | High (visual 2D hierarchy) |
| **Direct Evaluation Method** | Two stacks (Shunting-Yard) | Right-to-left stack or recursion | Left-to-right operand stack | Recursive Postorder traversal |
| **Sub-expression Manipulation (Simplification / Calculus)** | Difficult (requires reparsing string) | Difficult | Difficult | **Easy ($O(1)$ pointer replacement)** |
| **Corresponding Tree Traversal** | Parenthesized **Inorder** (`L, Root, R`) | **Preorder** (`Root, L, R`) | **Postorder** (`L, R, Root`) | Native Tree Structure |

### 5.2 Time and Space Complexity Summary

Let $n$ be the total number of tokens (operands + operators) in the expression, and let $h$ be the height of the resulting Expression Tree ($\lfloor \log_2 n \rfloor \le h \le n - 1$).

| Operation | Time Complexity | Auxiliary Space Complexity | Notes |
| :--- | :---: | :---: | :--- |
| **`build_expr_tree_from_postfix`** | $\Theta(n)$ | $O(n)$ stack space (+ $\Theta(n)$ heap nodes) | Each token is pushed once and popped at most once. |
| **`print_prefix` (Preorder)** | $\Theta(n)$ | $O(h)$ call stack | Visits every node exactly once. |
| **`print_postfix` (Postorder)** | $\Theta(n)$ | $O(h)$ call stack | Visits every node exactly once. |
| **`print_infix_full_parens` (Inorder)** | $\Theta(n)$ | $O(h)$ call stack | Prints `(` and `)` at each of the $I = (n-1)/2$ internal nodes. |
| **`evaluate_expr_tree` (Postorder)** | $\Theta(n)$ | $O(h)$ call stack | Performs $I$ arithmetic operations at internal nodes. |
| **`free_expr_tree` (Postorder)** | $\Theta(n)$ | $O(h)$ call stack | Frees all $n$ heap nodes bottom-up. |

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Concrete Bug vs. Fix Examples in C

#### Pitfall 1: Popping `left` Before `right` When Building a Binary Operator Node
Because the stack is Last-In-First-Out (LIFO), for a postfix sequence `A B -` (representing $A - B$), `A` is pushed first and `B` is pushed second. Thus, the **first pop returns the right operand `B`** and the **second pop returns the left operand `A`**. Reversing them silently passes tests for commutative operators (`+`, `*`) but computes $B - A$, $B / A$, and $B^A$ for non-commutative operators!

```c
/* BUG: Popping left first reverses non-commutative operands ("10 3 -" becomes 3 - 10 = -7)! */
ExprNode *left  = stack_pop(&stack);
ExprNode *right = stack_pop(&stack);
ExprNode *node  = create_op_node(op, left, right, false);

/* FIX: Always pop RIGHT operand first, then LEFT operand second! */
ExprNode *right = stack_pop(&stack);
ExprNode *left  = stack_pop(&stack);
ExprNode *node  = create_op_node(op, left, right, false);
```

#### Pitfall 2: Naive Unparenthesized Inorder Traversal
Printing `left`, `root`, `right` without parentheses collapses `(a + b) * c` into `a + b * c`.

```c
/* BUG: Naive inorder loses parentheses and changes operator precedence! */
void print_infix_buggy(const ExprNode *root) {
    if (!root) return;
    print_infix_buggy(root->left);
    printf("%s ", root->symbol);
    print_infix_buggy(root->right);
}

/* FIX: Emit '(' before left child and ')' after right child for binary operator nodes */
void print_infix_fixed(const ExprNode *root) {
    if (!root) return;
    if (root->type == NODE_OPERATOR_BINARY) printf("(");
    print_infix_fixed(root->left);
    printf("%s", root->symbol);
    print_infix_fixed(root->right);
    if (root->type == NODE_OPERATOR_BINARY) printf(")");
}
```

#### Pitfall 3: Leaking Partially Constructed Subtrees on Malformed Postfix Input
If an input string is malformed (e.g., `"10 20 30 +"` which leaves two subtrees `10` and `20+30` on the stack, or `"10 +"` which underflows on the second pop), returning `NULL` without draining and freeing the stack leaks all previously allocated subtrees!

```c
/* BUG: Returning NULL on error leaks subtrees still sitting inside stack! */
ExprNode *right = stack_pop(&stack);
ExprNode *left  = stack_pop(&stack);
if (!left || !right) {
    return NULL; /* Memory leak: 'right' and any deeper stack subtrees are leaked! */
}

/* FIX: Free both popped subtrees AND drain the entire stack via free_expr_tree() */
ExprNode *right = stack_pop(&stack);
ExprNode *left  = stack_pop(&stack);
if (!left || !right) {
    free_expr_tree(left);
    free_expr_tree(right);
    while (!stack_is_empty(&stack)) {
        free_expr_tree(stack_pop(&stack));
    }
    return NULL;
}
```

### 6.2 Boundary Conditions Checklist

- [ ] **Single-Operand Expression (`"42"` or `"x"`)**: Does `build_expr_tree_from_postfix` return a 1-node leaf tree that evaluates to `42.0` and prints `42` without extraneous parentheses?
- [ ] **Multi-Digit and Negative/Decimal Numbers (`"15"`, `"-3.5"`)**: Does the tokenizer split tokens by whitespace (`strtok`) rather than scanning character-by-character (which would wrongly split `"15"` into `'1'` and `'5'`)?
- [ ] **Non-Commutative Right-Child Associativity (`"10 5 2 - -"` $\implies 10 - (5 - 2) = 7$)**: Does minimal-parenthesis infix printing preserve the parentheses around the right child `10 - (5 - 2)` instead of printing `10 - 5 - 2` ($= 3$)?
- [ ] **Division by Zero (`"10 5 5 - /"`)**: Does `evaluate_expr_tree` check `fabs(rval) < 1e-12` before dividing and set `*ok = false` cleanly?
- [ ] **Malformed Postfix Inputs (`"5 +"` or `"5 6 7 +"`)**: Are both stack underflow (too few operands) and leftover stack elements (too many operands) caught and freed without memory leaks?

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways and Core Invariants

1. **Structural Invariant**: In an Expression Tree, **all operands are leaves** and **all operators are internal nodes**.
2. **No Parentheses Needed in the Tree**: Operator precedence and associativity are encoded directly by **node depth**—deeper subtrees are always evaluated before their parent operator nodes.
3. **Postfix Construction Algorithm**: Scan postfix tokens left-to-right using a stack of `ExprNode*`. Push operands as leaves; on a binary operator, pop **`right` first**, pop **`left` second**, link them under the new operator node, and push the operator node back onto the stack.
4. **Traversals $\leftrightarrow$ Notations**:
   - **Preorder** $\implies$ **Prefix (Polish)**
   - **Postorder** $\implies$ **Postfix (Reverse Polish)**
   - **Parenthesized Inorder** $\implies$ **Infix**
5. **Bottom-Up Evaluation**: Evaluating an Expression Tree is a **Postorder Traversal** (`eval(left) op eval(right)`) running in $\Theta(n)$ time and $O(h)$ recursion stack space.

### 7.2 Self-Test Revision Checklist

- [ ] Can I draw the step-by-step stack of subtrees when constructing the Expression Tree for `a b + c d e + * *`?
- [ ] Can I explain why the first `Pop()` on a binary operator must become `node->right` and the second `Pop()` must become `node->left`?
- [ ] Can I demonstrate with a 3-node example (`(a + b) * c`) why unparenthesized Inorder Traversal produces an incorrect infix expression?
- [ ] Can I state the exact relationship between the number of binary operators $I$ and operand leaves $L$ ($L = I + 1$)?
- [ ] Can I trace the recursive postorder evaluation of `15 7 1 1 + - / 3 * 2 1 1 + + -` to obtain `5.00`?

### 7.3 Navigation and Related Topics

- **Previous Topic in Unit 3**: [01 — Binary Search Trees: Array and Dynamic Implementations](./01-BST-Array-and-Dynamic-Implementations.md)
- **Next Topic in Unit 3**: [03 — Threaded Binary Search Trees](./03-Threaded-Binary-Search-Trees.md)
- **Related Topic in Unit 1**: [Unit 1 — Stack Applications (Infix to Postfix and Evaluation)](../Unit-1-Linked-Lists-and-Stacks/09-Expression-Conversion-and-Evaluation.md)
