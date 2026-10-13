# Polynomial Representation and Operations
---

## Table of Contents
1. [Introduction](#1-introduction)
   - 1.1 [What is Symbolic Polynomial Manipulation?](#11-what-is-symbolic-polynomial-manipulation)
   - 1.2 [Why Linked Lists Are Ideal for Sparse Polynomials](#12-why-linked-lists-are-ideal-for-sparse-polynomials)
   - 1.3 [Relationship to Prior Linked List Concepts](#13-relationship-to-prior-linked-list-concepts)
   - 1.4 [Learning Objectives](#14-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Mathematical Formulation](#21-mathematical-formulation)
   - 2.2 [Canonical Polynomial Invariants](#22-canonical-polynomial-invariants)
   - 2.3 [Terminology Table](#23-terminology-table)
   - 2.4 [Conceptual ASCII Diagrams](#24-conceptual-ascii-diagrams)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [C Structure Declarations (`typedef struct`)](#31-c-structure-declarations-typedef-struct)
   - 3.2 [Heap Memory Layout of a Linked Polynomial](#32-heap-memory-layout-of-a-linked-polynomial)
   - 3.3 [Static vs. Dynamic Storage, Allocation, and Deallocation](#33-static-vs-dynamic-storage-allocation-and-deallocation)
   - 3.4 [Pros and Cons of Representations](#34-pros-and-cons-of-representations)
4. [Operations and Algorithms](#4-operations-and-algorithms)
   - 4.1 [Canonical Term Insertion and Polynomial Creation (`insertTerm`)](#41-canonical-term-insertion-and-polynomial-creation-insertterm)
   - 4.2 [Displaying a Polynomial in Algebraic Format (`displayPoly`)](#42-displaying-a-polynomial-in-algebraic-format-displaypoly)
   - 4.3 [Polynomial Addition (`addPolynomials`)](#43-polynomial-addition-addpolynomials)
   - 4.4 [Polynomial Subtraction (`subtractPolynomials`)](#44-polynomial-subtraction-subtractpolynomials)
   - 4.5 [Polynomial Multiplication (`multiplyPolynomials`)](#45-polynomial-multiplication-multiplypolynomials)
   - 4.6 [Polynomial Evaluation at $x = x_0$ (`evaluatePolynomial`)](#46-polynomial-evaluation-at-x--x_0-evaluatepolynomial)
   - 4.7 [Complete, Compilable C11 Program](#47-complete-compilable-c11-program)
5. [Complexity Analysis and Comparison Tables](#5-complexity-analysis-and-comparison-tables)
   - 5.1 [Operation Complexity Summary](#51-operation-complexity-summary)
   - 5.2 [Comparison: Dense Array vs. Term-Struct Array vs. Linked List](#52-comparison-dense-array-vs-term-struct-array-vs-linked-list)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Pitfall 1: Retaining Zero-Coefficient Terms After Cancellation](#61-pitfall-1-retaining-zero-coefficient-terms-after-cancellation)
   - 6.2 [Pitfall 2: Breaking Descending Exponent Order During Creation](#62-pitfall-2-breaking-descending-exponent-order-during-creation)
   - 6.3 [Pitfall 3: Shallow Pointer Aliasing in Addition (Double-Free Hazard)](#63-pitfall-3-shallow-pointer-aliasing-in-addition-double-free-hazard)
   - 6.4 [Pitfall 4: Failing to Combine Like Terms During Multiplication](#64-pitfall-4-failing-to-combine-like-terms-during-multiplication)
   - 6.5 [Boundary Conditions Checklist](#65-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)
   - 7.1 [Key Takeaways](#71-key-takeaways)
   - 7.2 [Essential Invariants](#72-essential-invariants)
   - 7.3 [Quick Self-Test Checklist](#73-quick-self-test-checklist)
   - 7.4 [Related Notes and Navigation](#74-related-notes-and-navigation)

---

## 1. Introduction

### 1.1 What is Symbolic Polynomial Manipulation?
In numerical computing and computer algebra systems (such as Mathematica, Maple, SymPy, and compiler loop-index analyzers), a polynomial is not merely a function to evaluate at a single number—it is an **algebraic object** that must be stored symbolically and manipulated via exact operations such as **addition, subtraction, multiplication, differentiation, and evaluation**.

A univariate polynomial $P(x)$ of degree $d$ has the general form:
$$P(x) = a_d x^d + a_{d-1} x^{d-1} + \dots + a_1 x^1 + a_0 x^0$$

### 1.2 Why Linked Lists Are Ideal for Sparse Polynomials
Suppose we store a polynomial of degree $d$ using a **dense coefficient array** `int coeff[d + 1]`, where `coeff[k]` holds the coefficient of $x^k$:
- For a **dense polynomial** like $P(x) = 4x^3 - 2x^2 + 7x + 9$ ($d = 3$, $4$ non-zero terms), an array `coeff[0..3] = {9, 7, -2, 4}` is compact and efficient.
- However, consider a **sparse high-degree polynomial** such as:
  $$Q(x) = 5x^{10000} - 3x^2 + 7$$
  Here, the degree is $d = 10{,}000$, but there are only $m = 3$ non-zero terms! A dense array `coeff[10001]` wastes **$9{,}998$ zero entries ($99.97\%$ wasted memory)**, and adding two such polynomials forces an $O(d)$ loop over $10{,}001$ elements instead of $3$ elements.

By representing each non-zero term $c_i x^{e_i}$ as a node in a **Singly Linked List sorted in strictly descending order of exponents**, memory usage drops from $\Theta(d)$ to $\Theta(m)$ (where $m$ is the number of non-zero terms), and addition/subtraction execute in optimal $\Theta(m + n)$ time via a two-pointer merge!

### 1.3 Relationship to Prior Linked List Concepts
Polynomial manipulation is the classic **capstone case study** for Singly Linked Lists:
- **Sorted Insertion (`insertTerm`)** directly applies ordered linked-list insertion, extended with an in-place **merge/delete rule** when an exponent already exists in the list.
- **Polynomial Addition & Subtraction** directly apply the **Two-Pointer Sorted List Merge** pattern.
- **Polynomial Multiplication** combines nested list traversal with repeated ordered insertion/merging.

### 1.4 Learning Objectives
After studying these notes, you will be able to:
- State the three **Canonical Polynomial Invariants** for linked-list representation.
- Compare Dense Coefficient Arrays, Sparse Term-Struct Arrays, and Singly Linked Lists in terms of time and space complexity.
- Implement `insertTerm` to build canonical polynomials from arbitrary (even unsorted or duplicate-exponent) inputs while automatically eliminating zero-coefficient terms.
- Trace and implement **Polynomial Addition** ($\Theta(m+n)$), **Subtraction** ($\Theta(m+n)$), **Multiplication**, and **Evaluation** (direct & Horner's method) in C11.

---

## 2. Definitions and Terminology

### 2.1 Mathematical Formulation
A non-zero univariate polynomial $P(x)$ with $m \ge 1$ non-zero terms is written in **canonical sparse form** as:
$$P(x) = \sum_{i=1}^{m} c_i x^{e_i} = c_1 x^{e_1} + c_2 x^{e_2} + \dots + c_m x^{e_m}$$
where:
- $c_i \in \mathbb{Z} \setminus \{0\}$ (or $\mathbb{R} \setminus \{0\}$) is the **coefficient** of the $i$-th term.
- $e_i \in \mathbb{Z}_{\ge 0}$ is the non-negative integer **exponent** (power) of the $i$-th term.
- $e_1 > e_2 > \dots > e_m \ge 0$, so the **degree** of $P(x)$ is $\deg(P) = e_1$.

### 2.2 Canonical Polynomial Invariants
To guarantee that every polynomial has a **unique representation** and that two-pointer merge algorithms work in linear time, every linked-list polynomial must satisfy three invariants at all times:

1. **Strictly Descending Exponent Invariant**:
   $$\forall \text{ node } u \text{ with } u\text{->next} \neq \text{NULL}, \quad u\text{->exp} > u\text{->next->exp}$$
2. **Unique Exponent (Combined Like Terms) Invariant**:
   No two nodes in the list may have the same exponent (`u->exp != v->exp` for $u \neq v$).
3. **Non-Zero Coefficient Invariant**:
   $$\forall \text{ node } u \in P, \quad u\text{->coeff} \neq 0$$
   The **Zero Polynomial** $P(x) = 0$ is uniquely represented by an empty linked list (`head == NULL`).

### 2.3 Terminology Table

| Term | Definition |
| :--- | :--- |
| **Monomial / Term** | A single expression of the form $c \cdot x^e$, characterized by its coefficient $c$ and exponent $e$. |
| **Coefficient (`coeff`)** | The numerical multiplier $c$ of a term $c x^e$. |
| **Exponent (`exp`)** | The non-negative integer power $e$ to which the variable $x$ is raised. |
| **Degree of Polynomial ($\deg(P)$)** | The highest exponent $e_1$ among all non-zero terms in $P(x)$. In canonical descending order, $\deg(P) = \text{head->exp}$. |
| **Like Terms** | Two terms that have the exact same exponent ($e_1 == e_2$), which can be combined into $(c_1 + c_2)x^{e_1}$. |
| **Dense Polynomial** | A polynomial of degree $d$ where most of the $d + 1$ possible terms have non-zero coefficients ($m \approx d + 1$). |
| **Sparse Polynomial** | A polynomial of degree $d$ where the number of non-zero terms $m$ is much smaller than $d + 1$ ($m \ll d + 1$). |
| **Zero Polynomial ($P(x) = 0$)** | A polynomial with no non-zero terms ($m = 0$), represented by `head == NULL`. |
| **Horner's Rule** | An optimal factoring scheme $P(x) = (\dots((a_d x + a_{d-1})x + a_{d-2})x + \dots + a_0)$ for evaluating a polynomial with minimal multiplications. |

### 2.4 Conceptual ASCII Diagrams

Consider $P(x) = 4x^7 - 6x^3 + 2x - 9$ ($d = 7$, $m = 4$ non-zero terms):

#### 1. Representation A: Dense Coefficient Array (`coeff[0..7]`)
```text
Index (exp):   [0]  [1]  [2]  [3]  [4]  [5]  [6]  [7]
              +----+----+----+----+----+----+----+----+
coeff[exp]:   | -9 |  2 |  0 | -6 |  0 |  0 |  0 |  4 |   (4 zero slots wasted)
              +----+----+----+----+----+----+----+----+
```

#### 2. Representation B: Array of `(coeff, exp)` Structs (`terms[0..3]`)
```text
Index:     [0]          [1]          [2]          [3]
      +-----+-----+-----+-----+-----+-----+-----+-----+
      |c: 4 |e: 7 |c:-6 |e: 3 |c: 2 |e: 1 |c:-9 |e: 0 |
      +-----+-----+-----+-----+-----+-----+-----+-----+
```

#### 3. Representation C: Canonical Singly Linked List (`PolyNode *head`)
```text
        +----------+----------+     +----------+----------+     +----------+----------+     +----------+----------+
head ──►| coeff: 4 |  exp: 7  |────►| coeff:-6 |  exp: 3  |────►| coeff: 2 |  exp: 1  |────►| coeff:-9 |  exp: 0  |────► NULL
        +----------+----------+     +----------+----------+     +----------+----------+     +----------+----------+
         (Leading Term: deg=7)                                                               (Constant Term: x^0)
```

---

## 3. Representation and Memory Layout in C

### 3.1 C Structure Declarations (`typedef struct`)

```c
/* =========================================================================
 * 1. DENSE COEFFICIENT ARRAY REPRESENTATION
 * ========================================================================= */
#define MAX_DEGREE 1000

typedef struct DensePoly {
    int degree;                     /* Highest exponent with non-zero coeff */
    int coeff[MAX_DEGREE + 1];      /* coeff[e] stores coefficient of x^e */
} DensePoly;

/* =========================================================================
 * 2. SPARSE ARRAY OF TERM STRUCTS REPRESENTATION
 * ========================================================================= */
#define MAX_POLY_TERMS 100

typedef struct PolyTerm {
    int coeff;
    int exp;
} PolyTerm;

typedef struct TermArrayPoly {
    int numTerms;                   /* Number of active non-zero terms (m) */
    PolyTerm terms[MAX_POLY_TERMS]; /* Sorted in descending order of exp */
} TermArrayPoly;

/* =========================================================================
 * 3. SINGLY LINKED LIST REPRESENTATION (Primary Syllabus Standard)
 * ========================================================================= */
typedef struct PolyNode {
    int coeff;                      /* Non-zero coefficient (c_i != 0) */
    int exp;                        /* Non-negative exponent (e_i >= 0) */
    struct PolyNode *next;          /* Pointer to next lower-degree term */
} PolyNode;
```

### 3.2 Heap Memory Layout of a Linked Polynomial
On a 64-bit platform (`sizeof(int) = 4`, `sizeof(PolyNode *) = 8`), the two `int` fields (`coeff` and `exp`) pack cleanly into $8\text{ bytes}$ with **zero alignment padding** before the $8$-byte `next` pointer! Thus each `PolyNode` occupies exactly $16\text{ bytes}$.

For $P(x) = 5x^4 + 3x^2 - 8$:
```text
Stack Frame (main)
+------------------+
| polyP = 0x4100   | ───┐
+------------------+    │
                        ▼
Heap Memory (16-byte nodes, zero internal padding!)
  Address: 0x4100              Address: 0x4190              Address: 0x4140
  +--------+-------+--------+  +--------+-------+--------+  +--------+-------+--------+
  |coeff: 5| exp: 4|0x4190 ─┼─►|coeff: 3| exp: 2|0x4140 ─┼─►|coeff:-8| exp: 0| NULL   |
  +--------+-------+--------+  +--------+-------+--------+  +--------+-------+--------+
```

### 3.3 Static vs. Dynamic Storage, Allocation, and Deallocation
- **Node Allocation (`createPolyNode`)**:
  ```c
  PolyNode* createPolyNode(int coeff, int exp) {
      PolyNode *node = (PolyNode *)malloc(sizeof(PolyNode));
      if (node == NULL) {
          fprintf(stderr, "Error: Heap allocation failed for term %dx^%d.\n", coeff, exp);
          return NULL;
      }
      node->coeff = coeff;
      node->exp = exp;
      node->next = NULL;
      return node;
  }
  ```
- **Polynomial Deallocation (`freePoly`)**:
  Every polynomial produced by `addPolynomials`, `subtractPolynomials`, or `multiplyPolynomials` allocates its own independent heap nodes. When a polynomial is no longer needed, traverse from `head` and free each `PolyNode` to avoid memory leaks.

### 3.4 Pros and Cons of Representations

| Representation | Advantages | Disadvantages |
| :--- | :--- | :--- |
| **Dense Coefficient Array (`coeff[0..d]`)** | $O(1)$ access to any exponent $e$; zero pointer/exponent storage overhead; simple $O(d)$ Horner evaluation. | Catastrophic $\Theta(d)$ space and time waste on sparse polynomials like $x^{10000} + 1$; fixed `MAX_DEGREE` ceiling. |
| **Sparse Term-Struct Array (`terms[0..m-1]`)** | $\Theta(m)$ compact contiguous memory ($8\text{ bytes/term}$); good cache locality for addition merge. | Inserting or canceling a term in the middle during multiplication requires $O(m)$ array shifts; fixed `MAX_POLY_TERMS` capacity. |
| **Singly Linked List (`PolyNode *head`)** | $\Theta(m)$ space; grows/shrinks dynamically; $O(1)$ localized node insertion or removal when like terms cancel to $0$. | $8\text{ bytes}$ pointer overhead per term on 64-bit; sequential access only (no binary search on exponents). |

---

## 4. Operations and Algorithms

### 4.1 Canonical Term Insertion and Polynomial Creation (`insertTerm`)

#### 1. Purpose
Insert a term $c \cdot x^e$ into a polynomial linked list `*headRef` while preserving all three **Canonical Polynomial Invariants**:
1. Terms remain sorted in strictly descending order of `exp`.
2. If a node with `exp == e` already exists, their coefficients are summed (`curr->coeff += c`).
3. If `c == 0` initially, or if `curr->coeff + c == 0` after combining like terms, no zero node is kept in the list (the existing node is unlinked and freed!).

#### 2. Preconditions and Assumptions
- `headRef` is a valid non-NULL pointer (`PolyNode **headRef`).
- `exp >= 0`.

#### 3. Step-by-Step Algorithm
1. If `coeff == 0`, return `true` immediately (never insert a zero term).
2. Initialize `prev = NULL` and `curr = *headRef`.
3. Traverse while `curr != NULL && curr->exp > exp`:
   - Set `prev = curr` and `curr = curr->next`.
4. **Case 1 — Like Term Found (`curr != NULL && curr->exp == exp`)**:
   - Add coefficients: `curr->coeff += coeff`.
   - **Subcase 1a — Cancellation (`curr->coeff == 0`)**:
     - If `prev == NULL`, update `*headRef = curr->next` (head cancelled).
     - Else update `prev->next = curr->next`.
     - `free(curr)`.
   - Return `true`.
5. **Case 2 — New Exponent (`curr == NULL || curr->exp < exp`)**:
   - Allocate `newNode = createPolyNode(coeff, exp)`.
   - Set `newNode->next = curr`.
   - If `prev == NULL` (inserting new highest degree at head), set `*headRef = newNode`.
   - Else set `prev->next = newNode`.
   - Return `true`.

#### 4. Pseudocode
```text
ALGORITHM InsertTerm(head, coeff, exp)
1. IF coeff == 0 THEN RETURN head
2. prev <- NULL; curr <- head
3. WHILE curr != NULL AND curr.exp > exp DO
4.     prev <- curr
5.     curr <- curr.next
6. END WHILE
7. IF curr != NULL AND curr.exp == exp THEN
8.     curr.coeff <- curr.coeff + coeff
9.     IF curr.coeff == 0 THEN
10.        IF prev == NULL THEN head <- curr.next
11.        ELSE prev.next <- curr.next
12.        FreeNode(curr)
13.    END IF
14.    RETURN head
15. END IF
16. newNode <- AllocatePolyNode(coeff, exp)
17. newNode.next <- curr
18. IF prev == NULL THEN head <- newNode
19. ELSE prev.next <- newNode
20. RETURN head
```

#### 5. Before-and-After ASCII Pointer Diagram
Suppose $P(x) = 5x^4 + 3x^2 + 7$:
- **Operation A**: `insertTerm(&P, 4, 3)` (inserts $4x^3$ between $5x^4$ and $3x^2$).
- **Operation B**: `insertTerm(&P, -3, 2)` (combines with $+3x^2 \to 0x^2$, automatically deleting the node!).

```text
INITIAL:
head ──► [5 | x^4] ───────────────────► [3 | x^2] ──► [7 | x^0] ──► NULL

AFTER Operation A: insertTerm(&P, 4, 3) (New exponent 3 spliced between 4 and 2):
head ──► [5 | x^4] ──► [4 | x^3] ────► [3 | x^2] ──► [7 | x^0] ──► NULL

AFTER Operation B: insertTerm(&P, -3, 2) (3 + (-3) == 0 -> Node x^2 unlinked & freed!):
head ──► [5 | x^4] ──► [4 | x^3] ───────────────────► [7 | x^0] ──► NULL
```

#### 6. Reusable C Function
```c
bool insertTerm(PolyNode **headRef, int coeff, int exp) {
    if (coeff == 0) {
        return true; /* Zero terms are ignored */
    }
    if (exp < 0) {
        fprintf(stderr, "Error: Negative exponent %d not supported.\n", exp);
        return false;
    }

    PolyNode *prev = NULL;
    PolyNode *curr = *headRef;

    /* Find position in descending order of exp */
    while (curr != NULL && curr->exp > exp) {
        prev = curr;
        curr = curr->next;
    }

    /* Case 1: Exponent already exists -> combine like terms */
    if (curr != NULL && curr->exp == exp) {
        curr->coeff += coeff;
        if (curr->coeff == 0) {
            /* Coefficient cancelled to 0 -> remove node */
            if (prev == NULL) {
                *headRef = curr->next;
            } else {
                prev->next = curr->next;
            }
            free(curr);
        }
        return true;
    }

    /* Case 2: Exponent does not exist -> allocate and splice new node */
    PolyNode *newNode = createPolyNode(coeff, exp);
    if (newNode == NULL) {
        return false;
    }
    newNode->next = curr;
    if (prev == NULL) {
        *headRef = newNode;
    } else {
        prev->next = newNode;
    }
    return true;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $O(1)$ best case (inserting a new highest degree at head), $O(m)$ average and worst case (where $m$ is the current number of terms in the polynomial).
- **Auxiliary Space Complexity**: $O(1)$.

#### 8. Edge Cases and Common Mistakes
- **Failing to delete the node when `curr->coeff += coeff` becomes `0`**: Leaving `[0 | x^2]` in the list violates the Non-Zero Coefficient Invariant and corrupts degree checks if the leading term cancels!

---

### 4.2 Displaying a Polynomial in Algebraic Format (`displayPoly`)

#### 1. Purpose
Print a polynomial linked list in clean, human-readable mathematical notation (e.g., `5x^4 - 3x^2 + 2x - 8`, or `0` for an empty list).

#### 2. Preconditions and Assumptions
- `head` is a valid canonical polynomial pointer (or `NULL` for the zero polynomial).

#### 3. Step-by-Step Algorithm
1. If `head == NULL`, print `"0\n"` and return.
2. Iterate `curr` from `head` to `NULL`:
   - **Sign Formatting**:
     - If `curr == head` (first term): if `curr->coeff < 0`, print `"-"`.
     - Else (`curr != head`): print `" + "` if `curr->coeff > 0`, or `" - "` if `curr->coeff < 0`.
   - **Magnitude & Variable Formatting**:
     - Let `absCoeff = abs(curr->coeff)`.
     - If `curr->exp == 0`, print `absCoeff` (constant term).
     - Else if `curr->exp == 1`:
       - If `absCoeff == 1`, print `"x"`, else print `"%dx", absCoeff`.
     - Else (`curr->exp > 1`):
       - If `absCoeff == 1`, print `"x^%d", curr->exp`, else print `"%dx^%d", absCoeff, curr->exp`.
3. Print newline `"\n"`.

#### 4. Pseudocode
```text
ALGORITHM DisplayPoly(head)
1. IF head == NULL THEN Print("0"); RETURN
2. FOR curr <- head TO NULL DO
3.     IF curr == head THEN
4.         IF curr.coeff < 0 THEN Print("-")
5.     ELSE
6.         IF curr.coeff > 0 THEN Print(" + ") ELSE Print(" - ")
7.     END IF
8.     c <- ABS(curr.coeff)
9.     IF curr.exp == 0 THEN Print(c)
10.    ELSE IF curr.exp == 1 THEN
11.        IF c == 1 THEN Print("x") ELSE Print(c, "x")
12.    ELSE
13.        IF c == 1 THEN Print("x^", curr.exp) ELSE Print(c, "x^", curr.exp)
14.    END IF
15. END FOR
```

#### 5. Worked Formatting Examples
```text
Nodes: [(-1, 3) -> (4, 1) -> (-7, 0)]   ===>  -x^3 + 4x - 7
Nodes: [(3, 5) -> (1, 2)]               ===>  3x^5 + x^2
Nodes: NULL                             ===>  0
```

#### 6. Reusable C Function
```c
void displayPoly(const PolyNode *head) {
    if (head == NULL) {
        printf("0\n");
        return;
    }
    const PolyNode *curr = head;
    while (curr != NULL) {
        int c = curr->coeff;
        int absC = (c < 0) ? -c : c;

        if (curr == head) {
            if (c < 0) printf("-");
        } else {
            printf(c > 0 ? " + " : " - ");
        }

        if (curr->exp == 0) {
            printf("%d", absC);
        } else if (curr->exp == 1) {
            if (absC == 1) printf("x");
            else printf("%dx", absC);
        } else {
            if (absC == 1) printf("x^%d", curr->exp);
            else printf("%dx^%d", absC, curr->exp);
        }
        curr = curr->next;
    }
    printf("\n");
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $\Theta(m)$.
- **Auxiliary Space Complexity**: $O(1)$.

#### 8. Edge Cases and Common Mistakes
- **Printing `1x^0` as empty**: When `exp == 0` and `absC == 1`, the constant `1` **must** be printed (`1`, not empty string!). Our `if (curr->exp == 0)` branch handles this first.

---

### 4.3 Polynomial Addition (`addPolynomials`)

#### 1. Purpose
Given two canonical polynomials $P(x)$ (with $m$ terms) and $Q(x)$ (with $n$ terms), construct and return a new canonical polynomial $R(x) = P(x) + Q(x)$ in optimal $\Theta(m + n)$ time.

#### 2. Preconditions and Assumptions
- Both `poly1` and `poly2` are sorted in strictly descending order of exponents.
- Instead of calling `insertTerm` repeatedly from the head (which would degrade to $O((m+n)^2)$), we maintain a `tail` pointer on the result list so each merged term is appended in **$O(1)$ time**!

#### 3. Step-by-Step Algorithm
1. Initialize `resHead = NULL`, `resTail = NULL`, `p1 = poly1`, and `p2 = poly2`.
2. While `p1 != NULL && p2 != NULL`:
   - **Case 1 (`p1->exp > p2->exp`)**:
     - Append a new node `(p1->coeff, p1->exp)` to `(resHead, resTail)`.
     - Advance `p1 = p1->next`.
   - **Case 2 (`p2->exp > p1->exp`)**:
     - Append a new node `(p2->coeff, p2->exp)` to `(resHead, resTail)`.
     - Advance `p2 = p2->next`.
   - **Case 3 (`p1->exp == p2->exp`)**:
     - Compute `sum = p1->coeff + p2->coeff`.
     - If `sum != 0`, append `(sum, p1->exp)` to `(resHead, resTail)`.
     - Advance both `p1 = p1->next` and `p2 = p2->next`.
3. While `p1 != NULL`: append `(p1->coeff, p1->exp)` and advance `p1 = p1->next`.
4. While `p2 != NULL`: append `(p2->coeff, p2->exp)` and advance `p2 = p2->next`.
5. Return `resHead`.

#### 4. Pseudocode
```text
ALGORITHM AddPolynomials(p1, p2)
1. resHead <- NULL; resTail <- NULL
2. WHILE p1 != NULL AND p2 != NULL DO
3.     IF p1.exp > p2.exp THEN
4.         AppendNode(resHead, resTail, p1.coeff, p1.exp)
5.         p1 <- p1.next
6.     ELSE IF p2.exp > p1.exp THEN
7.         AppendNode(resHead, resTail, p2.coeff, p2.exp)
8.         p2 <- p2.next
9.     ELSE
10.        sum <- p1.coeff + p2.coeff
11.        IF sum != 0 THEN
12.            AppendNode(resHead, resTail, sum, p1.exp)
13.        END IF
14.        p1 <- p1.next; p2 <- p2.next
15.    END IF
16. END WHILE
17. WHILE p1 != NULL DO AppendNode(resHead, resTail, p1.coeff, p1.exp); p1 <- p1.next
18. WHILE p2 != NULL DO AppendNode(resHead, resTail, p2.coeff, p2.exp); p2 <- p2.next
19. RETURN resHead
```

#### 5. Step-by-Step Worked Trace & ASCII Diagram
Let $P(x) = 5x^4 + 3x^2 + 2x$ and $Q(x) = 4x^3 - 3x^2 + 6x + 9$:

```text
p1:  [5 | x^4] ──► [ 3 | x^2] ──► [2 | x^1] ──► NULL
p2:  [4 | x^3] ──► [-3 | x^2] ──► [6 | x^1] ──► [9 | x^0] ──► NULL

Merge Trace:
- Step 1: p1->exp (4) > p2->exp (3)  ==> Append [5 | x^4], advance p1
- Step 2: p1->exp (2) < p2->exp (3)  ==> Append [4 | x^3], advance p2
- Step 3: p1->exp (2) == p2->exp (2) ==> sum = 3 + (-3) = 0 (CANCELLED! Skip node), advance p1 & p2
- Step 4: p1->exp (1) == p2->exp (1) ==> sum = 2 + 6 = 8  ==> Append [8 | x^1], advance p1 & p2
- Step 5: p1 is NULL; copy remaining p2 ==> Append [9 | x^0], advance p2

Result R(x) = P(x) + Q(x):
resHead ──► [5 | x^4] ──► [4 | x^3] ──► [8 | x^1] ──► [9 | x^0] ──► NULL
```

#### 6. Reusable C Function
```c
/* Helper: O(1) append at the tail of a polynomial list */
static bool appendTermFast(PolyNode **headRef, PolyNode **tailRef, int coeff, int exp) {
    if (coeff == 0) return true;
    PolyNode *newNode = createPolyNode(coeff, exp);
    if (newNode == NULL) return false;
    if (*headRef == NULL) {
        *headRef = newNode;
        *tailRef = newNode;
    } else {
        (*tailRef)->next = newNode;
        *tailRef = newNode;
    }
    return true;
}

PolyNode* addPolynomials(const PolyNode *p1, const PolyNode *p2) {
    PolyNode *resHead = NULL;
    PolyNode *resTail = NULL;

    while (p1 != NULL && p2 != NULL) {
        if (p1->exp > p2->exp) {
            appendTermFast(&resHead, &resTail, p1->coeff, p1->exp);
            p1 = p1->next;
        } else if (p2->exp > p1->exp) {
            appendTermFast(&resHead, &resTail, p2->coeff, p2->exp);
            p2 = p2->next;
        } else {
            int sum = p1->coeff + p2->coeff;
            if (sum != 0) {
                appendTermFast(&resHead, &resTail, sum, p1->exp);
            }
            p1 = p1->next;
            p2 = p2->next;
        }
    }

    while (p1 != NULL) {
        appendTermFast(&resHead, &resTail, p1->coeff, p1->exp);
        p1 = p1->next;
    }
    while (p2 != NULL) {
        appendTermFast(&resHead, &resTail, p2->coeff, p2->exp);
        p2 = p2->next;
    }

    return resHead;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $\Theta(m + n)$ in the worst case (when all exponents are distinct) and $O(m + n)$ in all cases.
- **Auxiliary Space Complexity**: $O(1)$ temporary pointers, plus $\le m + n$ newly allocated nodes for the output polynomial.

#### 8. Edge Cases and Common Mistakes
- **Appending via `insertTerm` instead of `resTail`**: If you call `insertTerm(&resHead, ...)` for each merged term, every append traverses from `resHead` to the end of the growing list, turning an $O(m + n)$ merge into $O((m + n)^2)$! Maintaining `resTail` keeps each append strictly $O(1)$.

---

### 4.4 Polynomial Subtraction (`subtractPolynomials`)

#### 1. Purpose
Given two canonical polynomials $P(x)$ and $Q(x)$, compute $D(x) = P(x) - Q(x)$ in $\Theta(m + n)$ time.

#### 2. Preconditions and Assumptions
- Both `poly1` and `poly2` are in canonical descending exponent order.

#### 3. Step-by-Step Algorithm
1. Initialize `resHead = NULL`, `resTail = NULL`, `p1 = poly1`, `p2 = poly2`.
2. While `p1 != NULL && p2 != NULL`:
   - If `p1->exp > p2->exp`: append `(p1->coeff, p1->exp)`, advance `p1`.
   - Else if `p2->exp > p1->exp`: append `(-p2->coeff, p2->exp)` (negate `p2`'s coefficient!), advance `p2`.
   - Else (`p1->exp == p2->exp`):
     - Compute `diff = p1->coeff - p2->coeff`.
     - If `diff != 0`, append `(diff, p1->exp)`.
     - Advance both `p1` and `p2`.
3. Copy remaining `p1` terms as `(p1->coeff, p1->exp)`.
4. Copy remaining `p2` terms with negated coefficients `(-p2->coeff, p2->exp)`.
5. Return `resHead`.

#### 4. Pseudocode
```text
ALGORITHM SubtractPolynomials(p1, p2)
1. resHead <- NULL; resTail <- NULL
2. WHILE p1 != NULL AND p2 != NULL DO
3.     IF p1.exp > p2.exp THEN
4.         AppendNode(resHead, resTail, p1.coeff, p1.exp); p1 <- p1.next
5.     ELSE IF p2.exp > p1.exp THEN
6.         AppendNode(resHead, resTail, -p2.coeff, p2.exp); p2 <- p2.next
7.     ELSE
8.         diff <- p1.coeff - p2.coeff
9.         IF diff != 0 THEN AppendNode(resHead, resTail, diff, p1.exp)
10.        p1 <- p1.next; p2 <- p2.next
11.    END IF
12. END WHILE
13. WHILE p1 != NULL DO AppendNode(resHead, resTail, p1.coeff, p1.exp); p1 <- p1.next
14. WHILE p2 != NULL DO AppendNode(resHead, resTail, -p2.coeff, p2.exp); p2 <- p2.next
15. RETURN resHead
```

#### 5. Worked Trace
Subtracting $P(x) - Q(x)$ where $P(x) = 5x^4 + 3x^2 + 2x$ and $Q(x) = 5x^4 - 3x^2 + 2x$:
```text
- x^4: 5 - 5 = 0       (Cancelled!)
- x^2: 3 - (-3) = +6   (Appended: 6x^2)
- x^1: 2 - 2 = 0       (Cancelled!)
Result: D(x) = 6x^2.
```

#### 6. Reusable C Function
```c
PolyNode* subtractPolynomials(const PolyNode *p1, const PolyNode *p2) {
    PolyNode *resHead = NULL;
    PolyNode *resTail = NULL;

    while (p1 != NULL && p2 != NULL) {
        if (p1->exp > p2->exp) {
            appendTermFast(&resHead, &resTail, p1->coeff, p1->exp);
            p1 = p1->next;
        } else if (p2->exp > p1->exp) {
            appendTermFast(&resHead, &resTail, -p2->coeff, p2->exp);
            p2 = p2->next;
        } else {
            int diff = p1->coeff - p2->coeff;
            if (diff != 0) {
                appendTermFast(&resHead, &resTail, diff, p1->exp);
            }
            p1 = p1->next;
            p2 = p2->next;
        }
    }

    while (p1 != NULL) {
        appendTermFast(&resHead, &resTail, p1->coeff, p1->exp);
        p1 = p1->next;
    }
    while (p2 != NULL) {
        appendTermFast(&resHead, &resTail, -p2->coeff, p2->exp);
        p2 = p2->next;
    }

    return resHead;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $\Theta(m + n)$.
- **Auxiliary Space Complexity**: $O(1)$ beyond the output list.

#### 8. Edge Cases and Common Mistakes
- **Forgetting to negate remaining `p2` terms**: In the second cleanup loop (`while (p2 != NULL)`), you must append `-p2->coeff`, NOT `+p2->coeff`!
- **Self-Subtraction ($P(x) - P(x)$)**: Must cleanly cancel every term and return `NULL` (printed as `0`).

---

### 4.5 Polynomial Multiplication (`multiplyPolynomials`)

#### 1. Purpose
Given $P(x) = \sum_{i=1}^{m} a_i x^{e_i}$ and $Q(x) = \sum_{j=1}^{n} b_j x^{f_j}$, compute their product:
$$M(x) = P(x) \cdot Q(x) = \sum_{i=1}^{m} \sum_{j=1}^{n} (a_i \cdot b_j) x^{e_i + f_j}$$
with all like terms combined and zero cancellations removed.

#### 2. Preconditions and Assumptions
- If either `p1 == NULL` or `p2 == NULL` (multiplying by the zero polynomial $0$), the product is immediately `NULL` ($0$).

#### 3. Step-by-Step Algorithm
1. If `p1 == NULL || p2 == NULL`, return `NULL`.
2. Initialize `result = NULL`.
3. For each node `t1` in `p1`:
   - For each node `t2` in `p2`:
     - Compute `prodCoeff = t1->coeff * t2->coeff`.
     - Compute `sumExp = t1->exp + t2->exp`.
     - Call `insertTerm(&result, prodCoeff, sumExp)` (which inserts in descending order, merges like terms with identical `sumExp`, and removes terms that cancel to `0`).
4. Return `result`.

#### 4. Pseudocode
```text
ALGORITHM MultiplyPolynomials(p1, p2)
1. IF p1 == NULL OR p2 == NULL THEN RETURN NULL
2. result <- NULL
3. FOR t1 <- p1 TO NULL DO
4.     FOR t2 <- p2 TO NULL DO
5.         prodCoeff <- t1.coeff * t2.coeff
6.         sumExp <- t1.exp + t2.exp
7.         result <- InsertTerm(result, prodCoeff, sumExp)
8.     END FOR
9. END FOR
10. RETURN result
```

#### 5. Worked Trace & ASCII Diagram
Multiply $P(x) = 3x^2 + 2x$ by $Q(x) = 3x^2 - 2x + 5$:
```text
t1 = 3x^2:
  * 3x^2 -> insertTerm( 9, 4)  ==> [9x^4]
  * -2x  -> insertTerm(-6, 3)  ==> [9x^4] -> [-6x^3]
  * 5    -> insertTerm(15, 2)  ==> [9x^4] -> [-6x^3] -> [15x^2]

t1 = 2x:
  * 3x^2 -> insertTerm( 6, 3)  ==> -6x^3 + 6x^3 = 0x^3 (CANCELLED & FREED!)
                                   List is now: [9x^4] -> [15x^2]
  * -2x  -> insertTerm(-4, 2)  ==> 15x^2 + (-4x^2) = 11x^2
                                   List is now: [9x^4] -> [11x^2]
  * 5    -> insertTerm(10, 1)  ==> [9x^4] -> [11x^2] -> [10x^1]

Final Product: 9x^4 + 11x^2 + 10x
```

#### 6. Reusable C Function
```c
PolyNode* multiplyPolynomials(const PolyNode *p1, const PolyNode *p2) {
    if (p1 == NULL || p2 == NULL) {
        return NULL; /* Multiplication by zero polynomial yields 0 */
    }

    PolyNode *result = NULL;
    for (const PolyNode *t1 = p1; t1 != NULL; t1 = t1->next) {
        for (const PolyNode *t2 = p2; t2 != NULL; t2 = t2->next) {
            int prodCoeff = t1->coeff * t2->coeff;
            int sumExp = t1->exp + t2->exp;
            insertTerm(&result, prodCoeff, sumExp);
        }
    }
    return result;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**:
  - Generating $m \cdot n$ partial product terms and inserting each into `result` (which has up to $m \cdot n$ terms) takes $O(m^2 n^2)$ in the worst case if all $m \cdot n$ exponents are distinct, or $O(m \cdot n \cdot (m + n))$ when exponents are dense.
  - **Optimization Note**: If we instead build a temporary polynomial $T_i(x) = t_i \cdot Q(x)$ in $O(n)$ time for each of the $m$ terms of $P(x)$ and merge it into `result` using `addPolynomials`, the worst-case time improves to $O(m^2 n)$ (or $O(m n \log m)$ with divide-and-conquer pairwise merging).
- **Auxiliary Space Complexity**: $O(1)$ beyond the output polynomial (which has at most $m \cdot n$ nodes).

#### 8. Edge Cases and Common Mistakes
- **Difference of Squares Cancellation**: E.g., $(x - 2)(x + 2) = x^2 - 4$. Because `insertTerm` deletes nodes whose combined coefficient hits `0`, the middle terms $-2x + 2x$ are automatically removed.

---

### 4.6 Polynomial Evaluation at $x = x_0$ (`evaluatePolynomial`)

#### 1. Purpose
Compute the numerical scalar value $P(x_0)$ for a given real or integer input $x = x_0$.

#### 2. Preconditions and Assumptions
- If `head == NULL`, $P(x_0) = 0$.

#### 3. Step-by-Step Algorithm (Direct vs. Sparse Horner Adaptation)
- **Method A (Term-by-Term Exponentiation)**:
  For each node `(coeff, exp)`, accumulate `total += coeff * pow(x0, exp)`.
- **Method B (Sparse Horner's Rule Using Exponent Gaps)**:
  Let the terms be $(c_1, e_1), (c_2, e_2), \dots, (c_m, e_m)$. We can factor $P(x)$ by exponent differences $\Delta_i = e_i - e_{i+1}$:
  $$P(x) = \left(\dots\left((c_1 x^{e_1 - e_2} + c_2) x^{e_2 - e_3} + c_3\right)\dots + c_m\right) x^{e_m}$$
  This requires only $\deg(P) = e_1$ total multiplications (or $m$ calls to fast exponentiation on the gaps) and avoids recomputing powers of $x_0$ from scratch!

#### 4. Pseudocode
```text
ALGORITHM EvaluatePolynomialSparseHorner(head, x0)
1. IF head == NULL THEN RETURN 0
2. result <- 0
3. curr <- head
4. WHILE curr != NULL DO
5.     result <- result + curr.coeff
6.     IF curr.next != NULL THEN
7.         gap <- curr.exp - curr.next.exp
8.     ELSE
9.         gap <- curr.exp
10.    END IF
11.    result <- result * (x0 ^ gap)
12.    curr <- curr.next
13. END WHILE
14. RETURN result
```

#### 5. Worked Trace
Evaluate $P(x) = 2x^4 - 5x^2 + 8$ at $x_0 = 3$:
```text
- Term 1 (2x^4, next exp = 2, gap = 4 - 2 = 2):
  result = (0 + 2) * 3^2 = 2 * 9 = 18
- Term 2 (-5x^2, next exp = 0, gap = 2 - 0 = 2):
  result = (18 + (-5)) * 3^2 = 13 * 9 = 117
- Term 3 (8x^0, next = NULL, gap = 0):
  result = (117 + 8) * 3^0 = 125 * 1 = 125.
Check: 2(81) - 5(9) + 8 = 162 - 45 + 8 = 125. Exact match!
```

#### 6. Reusable C Function
```c
long long intPower(long long base, int exp) {
    long long res = 1;
    while (exp > 0) {
        if (exp & 1) res *= base;
        base *= base;
        exp >>= 1;
    }
    return res;
}

long long evaluatePolynomial(const PolyNode *head, long long x0) {
    if (head == NULL) {
        return 0;
    }
    long long result = 0;
    const PolyNode *curr = head;
    while (curr != NULL) {
        result += curr->coeff;
        int gap = (curr->next != NULL) ? (curr->exp - curr->next->exp) : curr->exp;
        result *= intPower(x0, gap);
        curr = curr->next;
    }
    return result;
}
```

#### 7. Time and Auxiliary-Space Complexity
- **Time Complexity**: $O(m \log d)$ using binary exponentiation on gaps (or $O(d)$ with simple multiplication loops).
- **Auxiliary Space Complexity**: $O(1)$.

#### 8. Edge Cases and Common Mistakes
- **Last term has `curr->exp > 0`**: For example, $P(x) = 3x^2$ at $x_0 = 4$. Because our algorithm uses `gap = curr->exp` ($2$) when `curr->next == NULL`, it multiplies `result` by $x_0^2 = 16$, giving $3 \times 16 = 48$!

---

### 4.7 Complete, Compilable C11 Program

```c
/**
 * File: polynomial_operations_demo.c
 * Standard: C11
 * Description: Comprehensive case study of Univariate Polynomial Representation
 *              and Operations using a Canonical Singly Linked List.
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

typedef struct PolyNode {
    int coeff;
    int exp;
    struct PolyNode *next;
} PolyNode;

PolyNode* createPolyNode(int coeff, int exp) {
    PolyNode *node = (PolyNode *)malloc(sizeof(PolyNode));
    if (node == NULL) {
        fprintf(stderr, "Error: Memory allocation failed for %dx^%d.\n", coeff, exp);
        return NULL;
    }
    node->coeff = coeff;
    node->exp = exp;
    node->next = NULL;
    return node;
}

/* Insert term in canonical descending exponent order; merge like terms & drop zeros */
bool insertTerm(PolyNode **headRef, int coeff, int exp) {
    if (coeff == 0 || exp < 0) {
        return (exp >= 0);
    }

    PolyNode *prev = NULL;
    PolyNode *curr = *headRef;

    while (curr != NULL && curr->exp > exp) {
        prev = curr;
        curr = curr->next;
    }

    if (curr != NULL && curr->exp == exp) {
        curr->coeff += coeff;
        if (curr->coeff == 0) {
            if (prev == NULL) {
                *headRef = curr->next;
            } else {
                prev->next = curr->next;
            }
            free(curr);
        }
        return true;
    }

    PolyNode *newNode = createPolyNode(coeff, exp);
    if (newNode == NULL) return false;
    newNode->next = curr;
    if (prev == NULL) {
        *headRef = newNode;
    } else {
        prev->next = newNode;
    }
    return true;
}

/* Display polynomial in standard algebraic notation */
void displayPoly(const PolyNode *head) {
    if (head == NULL) {
        printf("0\n");
        return;
    }
    const PolyNode *curr = head;
    while (curr != NULL) {
        int c = curr->coeff;
        int absC = (c < 0) ? -c : c;

        if (curr == head) {
            if (c < 0) printf("-");
        } else {
            printf(c > 0 ? " + " : " - ");
        }

        if (curr->exp == 0) {
            printf("%d", absC);
        } else if (curr->exp == 1) {
            if (absC == 1) printf("x");
            else printf("%dx", absC);
        } else {
            if (absC == 1) printf("x^%d", curr->exp);
            else printf("%dx^%d", absC, curr->exp);
        }
        curr = curr->next;
    }
    printf("\n");
}

static bool appendTermFast(PolyNode **headRef, PolyNode **tailRef, int coeff, int exp) {
    if (coeff == 0) return true;
    PolyNode *newNode = createPolyNode(coeff, exp);
    if (newNode == NULL) return false;
    if (*headRef == NULL) {
        *headRef = newNode;
        *tailRef = newNode;
    } else {
        (*tailRef)->next = newNode;
        *tailRef = newNode;
    }
    return true;
}

/* Polynomial Addition: O(m + n) two-pointer merge */
PolyNode* addPolynomials(const PolyNode *p1, const PolyNode *p2) {
    PolyNode *resHead = NULL, *resTail = NULL;
    while (p1 != NULL && p2 != NULL) {
        if (p1->exp > p2->exp) {
            appendTermFast(&resHead, &resTail, p1->coeff, p1->exp);
            p1 = p1->next;
        } else if (p2->exp > p1->exp) {
            appendTermFast(&resHead, &resTail, p2->coeff, p2->exp);
            p2 = p2->next;
        } else {
            int sum = p1->coeff + p2->coeff;
            if (sum != 0) {
                appendTermFast(&resHead, &resTail, sum, p1->exp);
            }
            p1 = p1->next;
            p2 = p2->next;
        }
    }
    while (p1 != NULL) {
        appendTermFast(&resHead, &resTail, p1->coeff, p1->exp);
        p1 = p1->next;
    }
    while (p2 != NULL) {
        appendTermFast(&resHead, &resTail, p2->coeff, p2->exp);
        p2 = p2->next;
    }
    return resHead;
}

/* Polynomial Subtraction: O(m + n) two-pointer merge */
PolyNode* subtractPolynomials(const PolyNode *p1, const PolyNode *p2) {
    PolyNode *resHead = NULL, *resTail = NULL;
    while (p1 != NULL && p2 != NULL) {
        if (p1->exp > p2->exp) {
            appendTermFast(&resHead, &resTail, p1->coeff, p1->exp);
            p1 = p1->next;
        } else if (p2->exp > p1->exp) {
            appendTermFast(&resHead, &resTail, -p2->coeff, p2->exp);
            p2 = p2->next;
        } else {
            int diff = p1->coeff - p2->coeff;
            if (diff != 0) {
                appendTermFast(&resHead, &resTail, diff, p1->exp);
            }
            p1 = p1->next;
            p2 = p2->next;
        }
    }
    while (p1 != NULL) {
        appendTermFast(&resHead, &resTail, p1->coeff, p1->exp);
        p1 = p1->next;
    }
    while (p2 != NULL) {
        appendTermFast(&resHead, &resTail, -p2->coeff, p2->exp);
        p2 = p2->next;
    }
    return resHead;
}

/* Polynomial Multiplication: Term-by-term with canonical merging */
PolyNode* multiplyPolynomials(const PolyNode *p1, const PolyNode *p2) {
    if (p1 == NULL || p2 == NULL) return NULL;
    PolyNode *result = NULL;
    for (const PolyNode *t1 = p1; t1 != NULL; t1 = t1->next) {
        for (const PolyNode *t2 = p2; t2 != NULL; t2 = t2->next) {
            insertTerm(&result, t1->coeff * t2->coeff, t1->exp + t2->exp);
        }
    }
    return result;
}

static long long intPower(long long base, int exp) {
    long long res = 1;
    while (exp > 0) {
        if (exp & 1) res *= base;
        base *= base;
        exp >>= 1;
    }
    return res;
}

/* Polynomial Evaluation using Sparse Horner's Rule */
long long evaluatePolynomial(const PolyNode *head, long long x0) {
    if (head == NULL) return 0;
    long long result = 0;
    const PolyNode *curr = head;
    while (curr != NULL) {
        result += curr->coeff;
        int gap = (curr->next != NULL) ? (curr->exp - curr->next->exp) : curr->exp;
        result *= intPower(x0, gap);
        curr = curr->next;
    }
    return result;
}

void freePoly(PolyNode **headRef) {
    PolyNode *curr = *headRef;
    while (curr != NULL) {
        PolyNode *next = curr->next;
        free(curr);
        curr = next;
    }
    *headRef = NULL;
}

int main(void) {
    PolyNode *P = NULL;
    PolyNode *Q = NULL;

    printf("=== 1. Creating Polynomials (Unsorted Input & Like-Term Merging) ===\n");
    /* Build P(x) = 5x^4 + 3x^2 + 2x (inserted out of order with duplicate x^2) */
    insertTerm(&P, 2, 1);
    insertTerm(&P, 5, 4);
    insertTerm(&P, 1, 2);
    insertTerm(&P, 2, 2);   /* Combines with 1x^2 -> 3x^2 */
    insertTerm(&P, 0, 9);   /* Zero coefficient ignored */

    /* Build Q(x) = 4x^3 - 3x^2 + 6x + 9 */
    insertTerm(&Q, 4, 3);
    insertTerm(&Q, -3, 2);
    insertTerm(&Q, 6, 1);
    insertTerm(&Q, 9, 0);

    printf("P(x) = ");
    displayPoly(P);
    printf("Q(x) = ");
    displayPoly(Q);
    printf("\n");

    printf("=== 2. Polynomial Addition & Subtraction ===\n");
    PolyNode *sum = addPolynomials(P, Q);
    printf("P(x) + Q(x) = ");
    displayPoly(sum);

    PolyNode *diff = subtractPolynomials(P, Q);
    printf("P(x) - Q(x) = ");
    displayPoly(diff);

    PolyNode *selfDiff = subtractPolynomials(P, P);
    printf("P(x) - P(x) = ");
    displayPoly(selfDiff);
    printf("\n");

    printf("=== 3. Polynomial Multiplication ===\n");
    PolyNode *A = NULL, *B = NULL;
    insertTerm(&A, 3, 2);
    insertTerm(&A, 2, 1);           /* A(x) = 3x^2 + 2x */
    insertTerm(&B, 3, 2);
    insertTerm(&B, -2, 1);
    insertTerm(&B, 5, 0);           /* B(x) = 3x^2 - 2x + 5 */

    printf("A(x) = ");
    displayPoly(A);
    printf("B(x) = ");
    displayPoly(B);

    PolyNode *prod = multiplyPolynomials(A, B);
    printf("A(x) * B(x) = ");
    displayPoly(prod);
    printf("\n");

    printf("=== 4. Polynomial Evaluation ===\n");
    long long x0 = 2;
    printf("P(%lld) = %lld\n", x0, evaluatePolynomial(P, x0));
    printf("Q(%lld) = %lld\n", x0, evaluatePolynomial(Q, x0));
    printf("(P + Q)(%lld) = %lld\n", x0, evaluatePolynomial(sum, x0));
    printf("A(%lld) * B(%lld) = %lld\n", x0, x0, evaluatePolynomial(prod, x0));

    freePoly(&P);
    freePoly(&Q);
    freePoly(&sum);
    freePoly(&diff);
    freePoly(&selfDiff);
    freePoly(&A);
    freePoly(&B);
    freePoly(&prod);

    return 0;
}
```

#### Program Output:
```text
=== 1. Creating Polynomials (Unsorted Input & Like-Term Merging) ===
P(x) = 5x^4 + 3x^2 + 2x
Q(x) = 4x^3 - 3x^2 + 6x + 9

=== 2. Polynomial Addition & Subtraction ===
P(x) + Q(x) = 5x^4 + 4x^3 + 8x + 9
P(x) - Q(x) = 5x^4 - 4x^3 + 6x^2 - 4x - 9
P(x) - P(x) = 0

=== 3. Polynomial Multiplication ===
A(x) = 3x^2 + 2x
B(x) = 3x^2 - 2x + 5
A(x) * B(x) = 9x^4 + 11x^2 + 10x

=== 4. Polynomial Evaluation ===
P(2) = 96
Q(2) = 41
(P + Q)(2) = 137
A(2) * B(2) = 208
```

---

## 5. Complexity Analysis and Comparison Tables

### 5.1 Operation Complexity Summary
Let:
- $d_1 = \deg(P)$, $d_2 = \deg(Q)$, and $d = \max(d_1, d_2)$.
- $m$ = number of non-zero terms in $P(x)$, and $n$ = number of non-zero terms in $Q(x)$ (note that $m \le d_1 + 1$ and $n \le d_2 + 1$; for sparse polynomials, $m, n \ll d$).

| Operation | Dense Array (`coeff[0..d]`) Time | Linked List (`PolyNode`) Best Time | Linked List (`PolyNode`) Average Time | Linked List (`PolyNode`) Worst Time | Linked List Auxiliary Space | Linked List Total Space (Output) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| **Insert 1 Term (`insertTerm`)** | $O(1)$ | $O(1)$ (new max exp) | $O(m)$ | $O(m)$ | $O(1)$ | $\Theta(m)$ |
| **Create from $m$ Unsorted Terms** | $O(d + m)$ | $O(m)$ (if ascending) | $O(m^2)$ | $O(m^2)$ | $O(1)$ | $\Theta(m)$ |
| **Display Polynomial** | $\Theta(d)$ | $\Theta(m)$ | $\Theta(m)$ | $\Theta(m)$ | $O(1)$ | $O(1)$ |
| **Addition ($P + Q$)** | $\Theta(d_1 + d_2)$ | $\Theta(m + n)$ | $\Theta(m + n)$ | $\Theta(m + n)$ | $O(1)$ | $O(m + n)$ |
| **Subtraction ($P - Q$)** | $\Theta(d_1 + d_2)$ | $\Theta(m + n)$ | $\Theta(m + n)$ | $\Theta(m + n)$ | $O(1)$ | $O(m + n)$ |
| **Multiplication ($P \cdot Q$)** | $\Theta(d_1 \cdot d_2)$ | $O(m \cdot n)$ | $O(m^2 n)$ | $O(m^2 n^2)$ (via `insertTerm`) | $O(1)$ | $O(m \cdot n)$ |
| **Evaluation at $x = x_0$** | $\Theta(d)$ (Horner) | $O(m)$ | $O(m \log d)$ | $O(m \log d)$ | $O(1)$ | $O(1)$ |
| **Differentiation $P'(x)$** | $\Theta(d)$ | $\Theta(m)$ | $\Theta(m)$ | $\Theta(m)$ | $O(1)$ | $O(m)$ |

### 5.2 Comparison: Dense Array vs. Term-Struct Array vs. Linked List

| Feature / Metric | Dense Coefficient Array (`int coeff[d+1]`) | Sparse Term-Struct Array (`PolyTerm terms[m]`) | Singly Linked List (`PolyNode *head`) |
| :--- | :--- | :--- | :--- |
| **Storage Space** | $\Theta(d)$ ($4(d+1)\text{ B}$) | $\Theta(m)$ ($8m\text{ B}$) | $\Theta(m)$ ($16m\text{ B}$ on 64-bit) |
| **Memory on $5x^{10000} + 1$** | $40{,}004\text{ bytes}$ (99.98% wasted) | $16\text{ bytes}$ | $32\text{ bytes}$ |
| **Access Coefficient of $x^k$** | $O(1)$ direct indexing | $O(\log m)$ binary search | $O(m)$ linear scan |
| **Insert / Cancel Term in Middle** | $O(1)$ | $O(m)$ array shift | $O(1)$ pointer splice (once position found) |
| **Addition / Subtraction ($P \pm Q$)** | $\Theta(\max(d_1, d_2))$ | $\Theta(m + n)$ merge | $\Theta(m + n)$ merge |
| **Dynamic Size Flexibility** | Fixed `MAX_DEGREE` | Fixed `MAX_POLY_TERMS` | Unbounded (up to heap capacity) |

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Pitfall 1: Retaining Zero-Coefficient Terms After Cancellation
When adding two polynomials (or combining like terms in `insertTerm`), if $c_1 + c_2 = 0$, storing a node with `coeff == 0` breaks display formatting (`+ 0x^2`), misreports the polynomial's degree if the leading term cancels, and wastes memory.

```c
/* BUG: Appending a node even when p1->coeff + p2->coeff == 0 */
int sum = p1->coeff + p2->coeff;
appendTermFast(&resHead, &resTail, sum, p1->exp); /* Stores [0 | x^k]! */

/* FIX: Only append when sum != 0 */
int sum = p1->coeff + p2->coeff;
if (sum != 0) {
    appendTermFast(&resHead, &resTail, sum, p1->exp);
}
```

### 6.2 Pitfall 2: Breaking Descending Exponent Order During Creation
If a programmer builds a polynomial using a simple `insertAtEnd` function instead of `insertTerm`, and the user supplies terms out of order (e.g., $2x + 5x^4 + 3x^2$), the two-pointer merge in `addPolynomials` will fail to match like terms!

```c
/* BUG: Appending unsorted input directly to the tail */
appendTermFast(&poly, &tail, 2, 1);
appendTermFast(&poly, &tail, 5, 4); /* List is [2x^1 -> 5x^4]: NOT canonical! */

/* FIX: Always use insertTerm(&poly, coeff, exp) when building from arbitrary input */
insertTerm(&poly, 2, 1);
insertTerm(&poly, 5, 4); /* List is [5x^4 -> 2x^1]: Strictly descending! */
```

### 6.3 Pitfall 3: Shallow Pointer Aliasing in Addition (Double-Free Hazard)
When one list finishes during `addPolynomials` (`p1 == NULL` while `p2 != NULL`), it is tempting to set `resTail->next = p2` instead of copying the remaining nodes of `p2`. Doing so causes `resHead` and `poly2` to **share the same heap nodes**, leading to a **double-free crash** when both polynomials are later freed!

```c
/* BUG: Shallow-linking remaining nodes of p2 into the result list */
if (p1 == NULL) {
    resTail->next = (PolyNode *)p2; /* Shares heap nodes with input polynomial p2! */
}

/* FIX: Deep-copy each remaining node so every polynomial owns its own nodes */
while (p2 != NULL) {
    appendTermFast(&resHead, &resTail, p2->coeff, p2->exp);
    p2 = p2->next;
}
```

### 6.4 Pitfall 4: Failing to Combine Like Terms During Multiplication
If `multiplyPolynomials` simply appends every pair product $a_i b_j x^{e_i + f_j}$ to the end of a list without combining like exponents, multiplying two $m$-term polynomials produces $m^2$ unmerged, unsorted nodes.

```c
/* BUG: Appending raw pair products without sorting or merging like exponents */
appendTermFast(&resHead, &resTail, t1->coeff * t2->coeff, t1->exp + t2->exp);

/* FIX: Use insertTerm(&result, ...) so equal exponents are summed and sorted */
insertTerm(&result, t1->coeff * t2->coeff, t1->exp + t2->exp);
```

### 6.5 Boundary Conditions Checklist
- [ ] **Zero Polynomial (`head == NULL`)**:
  - Does `displayPoly(NULL)` print `0`?
  - Does `addPolynomials(P, NULL)` return a deep copy of `P`?
  - Does `multiplyPolynomials(P, NULL)` return `NULL`?
  - Does `evaluatePolynomial(NULL, x0)` return `0`?
- [ ] **Complete Cancellation ($P(x) + (-P(x)) = 0$)**: Does `addPolynomials` or `subtractPolynomials(P, P)` return `NULL`?
- [ ] **Constant Polynomial ($P(x) = c x^0$)**: Does `displayPoly` print `c` without `x^0`?
- [ ] **Unit Coefficients ($c = \pm 1$)**: Does `displayPoly` format $1x^3 - 1x + 1$ cleanly as `x^3 - x + 1` (keeping the `1` on the constant term)?

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways
1. **Sparse Polynomials Require Linked or Term-Array Representations**: Storing a degree-$d$ polynomial with $m$ non-zero terms in a linked list uses $\Theta(m)$ space instead of $\Theta(d)$ space.
2. **Canonical Form Is Mandatory**: Every polynomial linked list must keep terms in **strictly descending exponent order**, **combine like terms**, and **never store zero-coefficient nodes**.
3. **Addition & Subtraction Are $\Theta(m + n)$ Two-Pointer Merges**: By maintaining a `resTail` pointer, each merged term is appended in $O(1)$ time.
4. **Multiplication Distributes All Pairs**: Multiplying an $m$-term polynomial by an $n$-term polynomial computes $m \cdot n$ products $(a_i b_j) x^{e_i + f_j}$ and merges terms with equal exponents.

### 7.2 Essential Invariants
- **Descending Order**: `curr->exp > curr->next->exp` for all nodes.
- **Non-Zero Payload**: `curr->coeff != 0` for all nodes.
- **Zero Polynomial Identity**: $P(x) = 0 \iff \text{head} == \text{NULL}$.
- **Deep Ownership**: Every polynomial variable owns a disjoint set of heap-allocated `PolyNode` structs.

### 7.3 Quick Self-Test Checklist
- [ ] Why does `addPolynomials` use `appendTermFast(&resHead, &resTail, ...)` instead of `insertTerm(&resHead, ...)`? *(Answer: Because the two-pointer merge already produces terms in strictly descending exponent order; using `resTail` takes $O(1)$ per term instead of $O(m+n)$ per term!)*
- [ ] What happens in `insertTerm(&head, -5, 3)` if the list currently contains `+5x^3` at the head?
- [ ] How many nodes are allocated to represent $P(x) = 9x^{50000} - 4x^{100} + 1$ in a singly linked list? *(Answer: 3 nodes)*
- [ ] Why is `resTail->next = p2` dangerous at the end of `addPolynomials`?

### 7.4 Related Notes and Navigation
- **Previous**: [05 - Multilists, Sparse Matrices, and Skip Lists](./05-Multilists-Sparse-Matrices-and-Skip-Lists.md)
- **Circular Lists**: [04 - Circular Linked Lists](./04-Circular-Linked-Lists.md)
- **Course Index**: [DSA Notes README](../README.md)
