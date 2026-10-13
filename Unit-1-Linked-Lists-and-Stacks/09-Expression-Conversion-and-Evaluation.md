# 09 — Expression Conversion and Evaluation

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [Why Computers Prefer Postfix and Prefix Over Infix](#11-why-computers-prefer-postfix-and-prefix-over-infix)
   - 1.2 [Relationship to Prior Concepts](#12-relationship-to-prior-concepts)
   - 1.3 [Learning Objectives](#13-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Infix, Postfix (Reverse Polish Notation), and Prefix (Polish Notation)](#21-infix-postfix-reverse-polish-notation-and-prefix-polish-notation)
   - 2.2 [Operator Precedence, Associativity Rules, and Unary Minus Handling](#22-operator-precedence-associativity-rules-and-unary-minus-handling)
   - 2.3 [Terminology Reference Table](#23-terminology-reference-table)
   - 2.4 [Mathematical Properties and Structural Invariants](#24-mathematical-properties-and-structural-invariants)
   - 2.5 [Conceptual ASCII Diagrams: Expression Trees and the Shunting-Yard Railway](#25-conceptual-ascii-diagrams-expression-trees-and-the-shunting-yard-railway)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Operator Stack (`CharStack`) vs. Operand Stack (`ValStack`)](#31-operator-stack-charstack-vs-operand-stack-valstack)
   - 3.2 [Tokenization of Multi-Digit Integers vs. Single-Character Identifiers](#32-tokenization-of-multi-digit-integers-vs-single-character-identifiers)
   - 3.3 [Memory Layout During Conversion vs. Evaluation](#33-memory-layout-during-conversion-vs-evaluation)
4. [Operations and Algorithms](#4-operations-and-algorithms)
   - 4.1 [Infix-to-Postfix Conversion (Dijkstra's Shunting-Yard Algorithm) & Full Tabular Trace](#41-infix-to-postfix-conversion-dijkstras-shunting-yard-algorithm--full-tabular-trace)
   - 4.2 [Infix-to-Prefix Conversion (Both Methods) & Full Tabular Trace](#42-infix-to-prefix-conversion-both-methods--full-tabular-trace)
   - 4.3 [Postfix Evaluation Algorithm & Full Tabular Trace](#43-postfix-evaluation-algorithm--full-tabular-trace)
   - 4.4 [Prefix Evaluation Algorithm & Full Tabular Trace](#44-prefix-evaluation-algorithm--full-tabular-trace)
   - 4.5 [Complete C11 Implementation: Conversion and Evaluation Suite](#45-complete-c11-implementation-conversion-and-evaluation-suite)
5. [Comparison Tables & Complexity Summary](#5-comparison-tables--complexity-summary)
   - 5.1 [Comparison of Infix, Postfix, and Prefix Notations](#51-comparison-of-infix-postfix-and-prefix-notations)
   - 5.2 [Conversion and Evaluation Rules Comparison Matrix](#52-conversion-and-evaluation-rules-comparison-matrix)
   - 5.3 [Time and Space Complexity Summary Table](#53-time-and-space-complexity-summary-table)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Frequent C Implementation Bugs and Fixes](#61-frequent-c-implementation-bugs-and-fixes)
   - 6.2 [Boundary Conditions Checklist](#62-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)
   - 7.1 [Key Takeaways and Core Invariants](#71-key-takeaways-and-core-invariants)
   - 7.2 [Self-Test Revision Checklist](#72-self-test-revision-checklist)
   - 7.3 [Navigation Links](#73-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 Why Computers Prefer Postfix and Prefix Over Infix

Humans naturally write arithmetic expressions in **Infix notation**, where each binary operator sits *between* its operands—for example, `A + B * C` or `(A + B) * C`. While familiar to human readers, infix notation presents three severe obstacles for single-pass computer evaluation:
1. **Operator Precedence Ambiguity**: To evaluate `A + B * C`, an evaluator cannot immediately add `A` and `B` upon reading `+`; it must look ahead to see if a higher-precedence operator (`*`) follows `B`.
2. **Associativity Tie-Breaking**: For operators of equal precedence, `A - B - C` must group left-to-right as `(A - B) - C`, whereas exponentiation `A ^ B ^ C` must group right-to-left as `A ^ (B ^ C)`.
3. **Parenthesis Overrides**: Nested parentheses `((...))` arbitrarily override default precedence rules, requiring recursive or stack-based sub-expression tracking.

In the 1920s, Polish logician **Jan Łukasiewicz** proved that placing the operator either **before** its operands (**Prefix / Polish Notation**) or **after** its operands (**Postfix / Reverse Polish Notation — RPN**) renders parentheses completely unnecessary! In both Postfix and Prefix notations:
- **Zero parentheses** are ever needed, regardless of how deeply sub-expressions are nested.
- The exact order of evaluation is **uniquely and unambiguously encoded** by token positions alone.
- A computer can evaluate any Postfix or Prefix expression in a **single linear pass ($O(n)$ time)** using a single operand stack without ever consulting a precedence table.

### 1.2 Relationship to Prior Concepts

In [07-Stacks-Array-and-Linked-List.md](07-Stacks-Array-and-Linked-List.md) and [08-Stack-Applications-and-Recursion.md](08-Stack-Applications-and-Recursion.md), we saw how a LIFO stack suspends outer contexts while inner nested contexts finish first. Arithmetic expression conversion and evaluation use two distinct varieties of stacks:
- During **Infix-to-Postfix / Infix-to-Prefix conversion**, an **Operator Stack** holds pending operators and opening parentheses until higher-precedence sub-expressions are emitted.
- During **Postfix / Prefix evaluation**, an **Operand Stack** holds intermediate numeric values until their governing operator is encountered.

### 1.3 Learning Objectives

After completing this chapter, you will be able to:
- Convert any parenthesized infix expression with mixed-precedence and mixed-associativity operators (`+`, `-`, `*`, `/`, `%`, `^`) into **Postfix (RPN)** and **Prefix (Polish)** notations.
- Explain and apply **Dijkstra's Shunting-Yard Algorithm**, including the exact tie-breaking rules for **left-associative** vs. **right-associative** operators.
- Explain why reversing an infix string for **Infix-to-Prefix conversion** flips the equal-precedence tie-breaking rule between left-associative and right-associative operators.
- Trace and implement **Postfix Evaluation** (left-to-right scan, `op2 = pop(); op1 = pop()`) and **Prefix Evaluation** (right-to-left scan, `op1 = pop(); op2 = pop()`).

---

## 2. Definitions and Terminology

### 2.1 Infix, Postfix (Reverse Polish Notation), and Prefix (Polish Notation)

For a binary operator $\oplus$ and two operands $X$ and $Y$:

| Notation | Syntax Pattern | Example 1: Simple | Example 2: Precedence | Example 3: Parenthesized |
| :--- | :---: | :---: | :---: | :---: |
| **Infix** | $X \oplus Y$ | `A + B` | `A + B * C` | `(A + B) * C` |
| **Postfix (RPN)** | $X\ Y\ \oplus$ | `A B +` | `A B C * +` | `A B + C *` |
| **Prefix (Polish)** | $\oplus\ X\ Y$ | `+ A B` | `+ A * B C` | `* + A B C` |

Notice how `A + B * C` and `(A + B) * C` have completely distinct Postfix (`A B C * +` vs. `A B + C *`) and Prefix (`+ A * B C` vs. `* + A B C`) representations **without a single parenthesis**.

### 2.2 Operator Precedence, Associativity Rules, and Unary Minus Handling

#### Operator Precedence and Associativity Table

| Operator(s) | Operation | Arity | Precedence Level | Associativity | Grouping of `X op Y op Z` | Postfix of `X op Y op Z` | Prefix of `X op Y op Z` |
| :---: | :--- | :---: | :---: | :---: | :---: | :---: | :---: |
| `()` | Parentheses (Grouping) | — | Highest (Override) | — | Explicit | *(Removed)* | *(Removed)* |
| `~` or `u-` | Unary Negation (`-X`) | Unary (1) | **4** | **Right-to-Left** | `~(~X)` | `X ~ ~` | `~ ~ X` |
| `^` | Exponentiation ($X^Y$) | Binary (2) | **3** | **Right-to-Left** | `X ^ (Y ^ Z)` | `X Y Z ^ ^` | `^ X ^ Y Z` |
| `*`, `/`, `%` | Multiply, Divide, Modulo | Binary (2) | **2** | **Left-to-Right** | `(X / Y) * Z` | `X Y / Z *` | `* / X Y Z` |
| `+`, `-` | Addition, Subtraction | Binary (2) | **1** (Lowest) | **Left-to-Right** | `(X - Y) - Z` | `X Y - Z -` | `- - X Y Z` |

#### Handling Unary Minus (`-`) in Expressions
In standard mathematical syntax, the symbol `'-'` is overloaded:
1. **Binary Subtraction** (`A - B`, `(X + Y) - Z`): Takes **two** operands, has **low precedence (1)**, and is **left-associative**.
2. **Unary Negation** (`-5`, `-A * B`, `A + (-B)`): Takes **one** operand to its right, has **high precedence (4)**, and is **right-associative** (`--A` means `-(-A)`).

**How Compilers and Evaluators Disambiguate Unary Minus**:
- **Lexical Context Rule**: A `'-'` token is **unary** if and only if it occurs at the **very beginning of the expression** OR immediately after an **opening parenthesis `'('`** or another **operator** (`+`, `-`, `*`, `/`, `%`, `^`). If it immediately follows an **operand** or a **closing parenthesis `')'`**, it is **binary subtraction**.
- **Representation in Postfix/Prefix**:
  - *For numeric literals* (e.g., `-5` in `"-5 3 +"`): If `'-'` is immediately followed by a digit with no intervening space, the tokenizer parses `-5` directly as a signed integer operand.
  - *For general symbolic expressions*: Replace unary `'-'` with a distinct unary operator symbol such as `'~'` (precedence 4, right-associative, popping only **1** operand during evaluation: `val = -pop()`) or rewrite `-X` algebraically as `(0 - X)`.

### 2.3 Terminology Reference Table

| Term | Definition |
| :--- | :--- |
| **Infix Notation** | Operator appears *between* its operands (`A + B`). Requires precedence rules, associativity rules, and parentheses. |
| **Postfix / RPN** | Operator appears immediately *after* its operands (`A B +`). Evaluated left-to-right using an operand stack. |
| **Prefix / Polish** | Operator appears immediately *before* its operands (`+ A B`). Evaluated right-to-left using an operand stack. |
| **Precedence** | Priority level determining which operator binds more tightly when two different operators share an operand (`*` binds tighter than `+`). |
| **Associativity** | Tie-breaking rule determining how adjacent operators of **equal precedence** group in the absence of parentheses (Left-to-Right vs. Right-to-Left). |
| **Shunting-Yard Algorithm** | Edsger Dijkstra's $O(n)$ stack-based algorithm for converting infix expressions to postfix (or AST) by routing operators through a side-track stack. |

### 2.4 Mathematical Properties and Structural Invariants

1. **Parenthesis-Free Token Count Identity**:
   If a valid infix expression contains $m$ binary operators, $m + 1$ operands, and $k$ pairs of parentheses (total $2m + 2k + 1$ tokens), then its equivalent Postfix and Prefix expressions both contain **extrinsically $2m + 1$ tokens** ($m$ operators, $m + 1$ operands, and $0$ parentheses).
2. **Relative Operand Order Invariant**:
   In **all three notations** (Infix, Postfix, and Prefix), the **left-to-right relative order of operands is strictly identical**! Only the positions of the operators move.
   - Example: In `(A - B / C) * (D ^ E ^ F)`, the operands appear in the exact sequence `A, B, C, D, E, F` in Infix, in Postfix (`A B C / - D E F ^ ^ *`), and in Prefix (`* - A / B C ^ D ^ E F`).
3. **Postfix Stack-Rank Validity Invariant**:
   Assign every operand a rank of $+1$ and every binary operator a rank of $-1$ (since it pops $2$ operands and pushes $1$ result, net change $1 - 2 = -1$). A postfix sequence of tokens $t_1, t_2, \dots, t_N$ is syntactically valid if and only if:
   - Every proper prefix sum $\sum_{i=1}^{j} \text{rank}(t_i) \ge 1$ for all $1 \le j < N$ (never underflows), and
   - The total sum $\sum_{i=1}^{N} \text{rank}(t_i) == 1$ (leaves exactly 1 final result on the stack).

### 2.5 Conceptual ASCII Diagrams: Expression Trees and the Shunting-Yard Railway

```
  Correspondence Between Expression Tree Traversals and Notations
  ===============================================================
  Expression: (A + B) * (C - D ^ E)

                       [ * ]
                      /     \
                  [ + ]     [ - ]
                  /   \     /   \
                 A     B   C    [ ^ ]
                                /   \
                               D     E

  1. Inorder Traversal (Left, Root, Right + parens) : (A + B) * (C - (D ^ E))  [INFIX]
  2. Postorder Traversal (Left, Right, Root)        : A B + C D E ^ - *        [POSTFIX]
  3. Preorder Traversal (Root, Left, Right)         : * + A B - C ^ D E        [PREFIX]


  Dijkstra's Shunting-Yard Railway Analogy (Infix -> Postfix)
  ===========================================================
  Output Track (Postfix) <--- [ A  B  C ] <--- Input Track (Infix: A + B * C)
                                  ^
                                  |  (Operands roll straight through to Output!)
                                  |  (Operators detour onto the vertical Operator Stack)
                               |  *  | <-- Top of Operator Stack (higher precedence)
                               |  +  |
                               +-----+
                            Operator Stack
```

---

## 3. Representation and Memory Layout in C

### 3.1 Operator Stack (`CharStack`) vs. Operand Stack (`ValStack`)

It is critical not to confuse the stack used during **conversion** with the stack used during **evaluation**:
- **Conversion (`infix_to_postfix` / `infix_to_prefix`)** pushes and pops **operator characters** (`'+'`, `'*'`, `'('`), so it uses a `CharStack`.
- **Evaluation (`evaluate_postfix` / `evaluate_prefix`)** pushes and pops **numeric operand values** (`100`, `20`, `864`), so it uses an integer/floating-point `ValStack`.

```c
/* Used during Infix -> Postfix / Prefix conversion */
typedef struct {
    char data[MAX_LEN]; /* Holds '+', '-', '*', '/', '%', '^', '(', ')' */
    int top;
} CharStack;

/* Used during Postfix / Prefix numerical evaluation */
typedef struct {
    long long data[MAX_LEN]; /* Holds intermediate numeric results */
    int top;
} ValStack;
```

### 3.2 Tokenization of Multi-Digit Integers vs. Single-Character Identifiers

In textbook hand-traces, operands are often single letters (`A`, `B`, `C`) or single digits (`3`, `5`). However, in real C programs, an expression like `100 20 5 / -` contains **multi-digit integers**:
- If we omit spaces in Postfix (`100205/-`), it is impossible to tell whether `100205` is one number or three numbers `100`, `20`, and `5`!
- Therefore, our C11 implementation emits a **single space `' '`** after every multi-digit operand and operator token (`"100 20 5 / -"`), and uses `isalnum()` / `strtoll()` during scanning to read complete multi-character tokens cleanly.

### 3.3 Memory Layout During Conversion vs. Evaluation

```
  During Infix-to-Postfix Conversion of "100 - 20 / 5":
  +-----------------------------------+       +-----------------------------------+
  | CharStack op_stack (1 byte/slot)  |       | Output Buffer postfix_out[]       |
  |   data: [ '-', '/' ]   top = 1    |       |   "100 20 5 "                     |
  +-----------------------------------+       +-----------------------------------+

  During Postfix Evaluation of "100 20 5 / -":
  +---------------------------------------------------+
  | ValStack vs (8 bytes per long long slot)          |
  |   Step 1..3 (push operands): [ 100, 20, 5 ] top=2 |
  |   Step 4 (apply '/'):        [ 100, 4 ]     top=1 |
  |   Step 5 (apply '-'):        [ 96 ]         top=0 |
  +---------------------------------------------------+
```

---

## 4. Operations and Algorithms

### 4.1 Infix-to-Postfix Conversion (Dijkstra's Shunting-Yard Algorithm) & Full Tabular Trace

#### Complete Step-by-Step Rules (Left-to-Right Scan)
Initialize an empty operator stack `op_stack` and an empty output string `postfix`. Scan the infix expression from **left to right**, one token `T` at a time:

1. **If `T` is an Operand (variable or number)**:
   - Append `T` immediately to `postfix`.
2. **If `T` is an Opening Parenthesis `'('`**:
   - Push `'('` onto `op_stack`. (Note: While on the stack, `'('` acts as a barrier—no operator below `'('` can be popped until the matching `')'` arrives.)
3. **If `T` is a Closing Parenthesis `')'`**:
   - Pop operators from `op_stack` and append them to `postfix` until `'('` is at the top of `op_stack`.
   - Pop and **discard** `'('` (do not append `'('` or `')'` to `postfix`).
   - *(Error Check)*: If `op_stack` becomes empty before finding `'('`, the expression has a mismatched `')'`.
4. **If `T` is a Binary Operator `op`**:
   - While `op_stack` is not empty, `peek(op_stack) != '('`, and:
     - `precedence(peek(op_stack)) > precedence(op)`, **OR**
     - `precedence(peek(op_stack)) == precedence(op)` **AND `op` is Left-Associative** (`+`, `-`, `*`, `/`, `%`):
     - Pop the top operator from `op_stack` and append it to `postfix`.
   - Push `op` onto `op_stack`.
5. **When End of Input Is Reached**:
   - While `op_stack` is not empty, pop each remaining operator and append it to `postfix`. (If any `'('` is found on `op_stack`, report an unclosed `'('` error.)

> [!IMPORTANT]
> **Why Associativity Determines the Equal-Precedence Tie-Break:**
> - **Left-Associative (`A - B - C`)**: When the second `'-'` arrives, `'-'` is already on top of `op_stack`. Because the first subtraction `(A - B)` must execute *before* the second subtraction, we **pop** the first `'-'` immediately, yielding `A B - C -`.
> - **Right-Associative (`A ^ B ^ C`)**: When the second `'^'` arrives, `'^'` is already on top of `op_stack`. Because the right-hand exponentiation `(B ^ C)` must execute *before* the first `'^'`, we **do NOT pop** the first `'^'`; instead we push the second `'^'` on top of it, yielding `A B C ^ ^`.

---

#### Full Step-by-Step Tabular Trace: Complex Expression `A + B * (C ^ D - E) ^ (F + G * H) - I`

| Step | Scanned Token | Action Taken | Operator Stack (Bottom $\to$ Top) | Postfix Output So Far |
| :---: | :---: | :--- | :--- | :--- |
| 0 | — | Initialize empty stack and output | `[ ]` | `""` |
| 1 | `A` | Operand $\to$ append to output | `[ ]` | `A` |
| 2 | `+` | Stack empty $\to$ push `+` | `+` | `A` |
| 3 | `B` | Operand $\to$ append to output | `+` | `A B` |
| 4 | `*` | `prec(*) = 2 > prec(+) = 1` $\to$ push `*` | `+ *` | `A B` |
| 5 | `(` | Opening parenthesis $\to$ push `(` | `+ * (` | `A B` |
| 6 | `C` | Operand $\to$ append to output | `+ * (` | `A B C` |
| 7 | `^` | Top is `(` $\to$ push `^` | `+ * ( ^` | `A B C` |
| 8 | `D` | Operand $\to$ append to output | `+ * ( ^` | `A B C D` |
| 9 | `-` | `prec(^) = 3 > prec(-) = 1` $\to$ pop `^`, then push `-` | `+ * ( -` | `A B C D ^` |
| 10 | `E` | Operand $\to$ append to output | `+ * ( -` | `A B C D ^ E` |
| 11 | `)` | Pop until `(`: pop `-`, then pop & discard `(` | `+ *` | `A B C D ^ E -` |
| 12 | `^` | `prec(^) = 3 > prec(*) = 2` $\to$ push `^` | `+ * ^` | `A B C D ^ E -` |
| 13 | `(` | Opening parenthesis $\to$ push `(` | `+ * ^ (` | `A B C D ^ E -` |
| 14 | `F` | Operand $\to$ append to output | `+ * ^ (` | `A B C D ^ E - F` |
| 15 | `+` | Top is `(` $\to$ push `+` | `+ * ^ ( +` | `A B C D ^ E - F` |
| 16 | `G` | Operand $\to$ append to output | `+ * ^ ( +` | `A B C D ^ E - F G` |
| 17 | `*` | `prec(*) = 2 > prec(+) = 1` $\to$ push `*` | `+ * ^ ( + *` | `A B C D ^ E - F G` |
| 18 | `H` | Operand $\to$ append to output | `+ * ^ ( + *` | `A B C D ^ E - F G H` |
| 19 | `)` | Pop until `(`: pop `*`, pop `+`, discard `(` | `+ * ^` | `A B C D ^ E - F G H * +` |
| 20 | `-` | Pop `^` ($3>1$), pop `*` ($2>1$), pop `+` ($1==1$, left-assoc); push `-` | `-` | `A B C D ^ E - F G H * + ^ * +` |
| 21 | `I` | Operand $\to$ append to output | `-` | `A B C D ^ E - F G H * + ^ * + I` |
| 22 | `EOF` | End of input $\to$ pop remaining `-` | `[ ]` | `A B C D ^ E - F G H * + ^ * + I -` |

**Final Postfix Result**: `A B C D ^ E - F G H * + ^ * + I -`

---

### 4.2 Infix-to-Prefix Conversion (Both Methods) & Full Tabular Trace

There are two standard textbook methods to convert an Infix expression into **Prefix (Polish) Notation**:

#### Method 1: Reverse-Scan Shunting-Yard Algorithm (Standard Exam & Implementation Method)
1. **Scan Right-to-Left (or Reverse the Infix String)**: Read tokens from the rightmost character down to index `0`.
2. **Swap Parentheses Roles**: When scanning right-to-left, a closing parenthesis `')'` opens a parenthesized group (so push `')'` onto `op_stack`), and an opening parenthesis `'('` closes that group (so pop operators until `')'` is reached, then discard `')'`).
3. **Modified Operator Tie-Breaking Rule**:
   - Because we are scanning from **right to left**, if two operators have **equal precedence** (`prec(top) == prec(op)`):
     - If `op` is **Left-Associative** (`+`, `-`, `*`, `/`, `%`), the rightmost operator must be evaluated **second** (outer), so we **do NOT pop** `top`; we push `op` onto the stack.
     - If `op` is **Right-Associative** (`^`), the rightmost `^` must be evaluated **first** (inner), so we **DO pop** `top` before pushing `op`!
   - Summary of pop condition during right-to-left scan:
     $$\text{Pop } \text{top} \iff \text{prec}(\text{top}) > \text{prec}(\text{op}) \quad \text{OR} \quad \big(\text{prec}(\text{top}) == \text{prec}(\text{op}) \text{ AND } \text{op is Right-Associative}\big)$$
4. **Reverse the Output Token Sequence**: After popping all remaining operators at the end of the right-to-left scan, reverse the collected output tokens to obtain the final **Prefix** expression.

#### Method 2: Direct Left-to-Right Two-Stack Method
Maintain two stacks while scanning left-to-right: an **Operator Stack** `op_stack` (using exact Postfix Shunting-Yard precedence rules) and a **Prefix Expression Stack** `expr_stack` of strings:
- When an operand is read, push it as a string onto `expr_stack`.
- Whenever an operator `op` is popped from `op_stack`, pop the top two strings `right_str = pop(expr_stack)` and `left_str = pop(expr_stack)`, concatenate them in prefix order `"op " + left_str + " " + right_str`, and push that combined string back onto `expr_stack`.

---

#### Full Step-by-Step Tabular Trace (Method 1): `(A - B / C) * (D ^ E ^ F)`

First, note that scanning `(A - B / C) * (D ^ E ^ F)` from **right to left** visits the tokens in the sequence:
`)`, `F`, `^`, `E`, `^`, `D`, `(`, `*`, `)`, `C`, `/`, `B`, `-`, `A`, `(`.

| Step | Right-to-Left Token | Action Taken (Right-to-Left Rules) | Operator Stack (Bottom $\to$ Top) | Reversed Output Collected |
| :---: | :---: | :--- | :--- | :--- |
| 1 | `)` | Right-to-left opener $\to$ push `)` | `)` | `""` |
| 2 | `F` | Operand $\to$ append `F` | `)` | `F` |
| 3 | `^` | Top is `)` $\to$ push `^` | `) ^` | `F` |
| 4 | `E` | Operand $\to$ append `E` | `) ^` | `F E` |
| 5 | `^` | `prec(^) == prec(^)` and `^` is **Right-Assoc** $\to$ **pop `^`**, then push `^` | `) ^` | `F E ^` |
| 6 | `D` | Operand $\to$ append `D` | `) ^` | `F E ^ D` |
| 7 | `(` | Right-to-left closer $\to$ pop until `)`: pop `^`, discard `)` | `[ ]` | `F E ^ D ^` |
| 8 | `*` | Stack empty $\to$ push `*` | `*` | `F E ^ D ^` |
| 9 | `)` | Right-to-left opener $\to$ push `)` | `* )` | `F E ^ D ^` |
| 10 | `C` | Operand $\to$ append `C` | `* )` | `F E ^ D ^ C` |
| 11 | `/` | Top is `)` $\to$ push `/` | `* ) /` | `F E ^ D ^ C` |
| 12 | `B` | Operand $\to$ append `B` | `* ) /` | `F E ^ D ^ C B` |
| 13 | `-` | `prec(/) = 2 > prec(-) = 1` $\to$ pop `/`, then push `-` | `* ) -` | `F E ^ D ^ C B /` |
| 14 | `A` | Operand $\to$ append `A` | `* ) -` | `F E ^ D ^ C B / A` |
| 15 | `(` | Right-to-left closer $\to$ pop until `)`: pop `-`, discard `)` | `*` | `F E ^ D ^ C B / A -` |
| 16 | `EOF` | Pop remaining `*` from stack | `[ ]` | `F E ^ D ^ C B / A - *` |
| **Final** | **Reverse** | **Reverse the collected output tokens** | — | **`* - A / B C ^ D ^ E F`** |

---

### 4.3 Postfix Evaluation Algorithm & Full Tabular Trace

#### Step-by-Step Algorithm (Left-to-Right Scan)
1. Initialize an empty numeric operand stack `val_stack`.
2. Scan the Postfix expression from **left to right**, token by token:
   - **If token is an Operand (number)**: Convert to number and `push(val_stack, num)`.
   - **If token is a Binary Operator `op`**:
     1. Pop the **second (right-hand) operand** first: `op2 = pop(val_stack)`.
     2. Pop the **first (left-hand) operand** second: `op1 = pop(val_stack)`.
     3. Compute `res = op1 op op2` (e.g., `op1 - op2`, `op1 / op2`, $\text{op1}^{\text{op2}}$).
     4. Push `res` onto `val_stack`.
3. When the scan completes, `pop(val_stack)` returns the final result.

> [!CAUTION]
> **Operand Pop Order in Postfix Evaluation (`op2` First, `op1` Second!):**
> In `"20 5 /"`, `20` is pushed first and `5` is pushed second (`5` sits on top of `20`). Therefore, the **first** `pop()` yields `op2 = 5` (divisor) and the **second** `pop()` yields `op1 = 20` (dividend), computing `op1 / op2 = 20 / 5 = 4`. Reversing them computes `5 / 20 = 0`!

#### Full Step-by-Step Tabular Trace: Postfix `"100 20 5 / - 3 2 ^ *"`
*(Corresponds to Infix `(100 - 20 / 5) * 3 ^ 2`)*

| Step | Scanned Token | Action Taken | Operand Stack (Bottom $\to$ Top) |
| :---: | :---: | :--- | :--- |
| 1 | `100` | Operand $\to$ `push(100)` | `[ 100 ]` |
| 2 | `20` | Operand $\to$ `push(20)` | `[ 100, 20 ]` |
| 3 | `5` | Operand $\to$ `push(5)` | `[ 100, 20, 5 ]` |
| 4 | `/` | Operator `/`: `op2 = pop() = 5`, `op1 = pop() = 20`; `push(20 / 5 = 4)` | `[ 100, 4 ]` |
| 5 | `-` | Operator `-`: `op2 = pop() = 4`, `op1 = pop() = 100`; `push(100 - 4 = 96)` | `[ 96 ]` |
| 6 | `3` | Operand $\to$ `push(3)` | `[ 96, 3 ]` |
| 7 | `2` | Operand $\to$ `push(2)` | `[ 96, 3, 2 ]` |
| 8 | `^` | Operator `^`: `op2 = pop() = 2`, `op1 = pop() = 3`; `push(3 ^ 2 = 9)` | `[ 96, 9 ]` |
| 9 | `*` | Operator `*`: `op2 = pop() = 9`, `op1 = pop() = 96`; `push(96 * 9 = 864)` | `[ 864 ]` |
| **End** | `EOF` | Pop final answer from stack | **Result = `864`** |

---

### 4.4 Prefix Evaluation Algorithm & Full Tabular Trace

#### Step-by-Step Algorithm (Right-to-Left Scan)
1. Initialize an empty numeric operand stack `val_stack`.
2. Scan the Prefix expression from **right to left** (from the last token back to the first token):
   - **If token is an Operand (number)**: `push(val_stack, num)`.
   - **If token is a Binary Operator `op`**:
     1. Pop the **first (left-hand) operand** first: `op1 = pop(val_stack)`. *(Because we scanned right-to-left, `op2` was pushed earlier and `op1` was pushed most recently, so `op1` sits at the top of the stack!)*
     2. Pop the **second (right-hand) operand** second: `op2 = pop(val_stack)`.
     3. Compute `res = op1 op op2`.
     4. Push `res` onto `val_stack`.
3. When the scan reaches the front of the string, `pop(val_stack)` returns the final result.

#### Full Step-by-Step Tabular Trace: Prefix `"* - 100 / 20 5 ^ 3 2"`
*(Scanning right-to-left visits tokens: `2`, `3`, `^`, `5`, `20`, `/`, `100`, `-`, `*`)*

| Step | Right-to-Left Token | Action Taken | Operand Stack (Bottom $\to$ Top) |
| :---: | :---: | :--- | :--- |
| 1 | `2` | Operand $\to$ `push(2)` | `[ 2 ]` |
| 2 | `3` | Operand $\to$ `push(3)` | `[ 2, 3 ]` |
| 3 | `^` | Operator `^`: `op1 = pop() = 3`, `op2 = pop() = 2`; `push(3 ^ 2 = 9)` | `[ 9 ]` |
| 4 | `5` | Operand $\to$ `push(5)` | `[ 9, 5 ]` |
| 5 | `20` | Operand $\to$ `push(20)` | `[ 9, 5, 20 ]` |
| 6 | `/` | Operator `/`: `op1 = pop() = 20`, `op2 = pop() = 5`; `push(20 / 5 = 4)` | `[ 9, 4 ]` |
| 7 | `100` | Operand $\to$ `push(100)` | `[ 9, 4, 100 ]` |
| 8 | `-` | Operator `-`: `op1 = pop() = 100`, `op2 = pop() = 4`; `push(100 - 4 = 96)` | `[ 9, 96 ]` |
| 9 | `*` | Operator `*`: `op1 = pop() = 96`, `op2 = pop() = 9`; `push(96 * 9 = 864)` | `[ 864 ]` |
| **End** | `Start` | Pop final answer from stack | **Result = `864`** |

---

### 4.5 Complete C11 Implementation: Conversion and Evaluation Suite

#### Supported Token Types and Operators
- **`infix_to_postfix` and `infix_to_prefix`**: Support single-letter or multi-character alphanumeric variables (`A`, `x1`), **multi-digit integers** (`100`, `20`), nested parentheses `()`, and binary operators `+`, `-`, `*`, `/`, `%`, `^` (with left-associativity for `+ - * / %` and right-associativity for `^`).
- **`evaluate_postfix` and `evaluate_prefix`**: Support space-separated **multi-digit integers** (including signed negative literals like `-5`) and binary operators `+`, `-`, `*`, `/`, `%`, `^` with division-by-zero and stack-underflow error checking.

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <string.h>
#include <ctype.h>

#define MAX_LEN 512

/* =========================================================================
 * CHARACTER STACK (FOR OPERATOR TOKENS)
 * ========================================================================= */
typedef struct {
    char data[MAX_LEN];
    int top;
} CharStack;

static void cs_init(CharStack *s) { s->top = -1; }
static bool cs_empty(const CharStack *s) { return s->top == -1; }
static char cs_peek(const CharStack *s) { return s->data[s->top]; }
static void cs_push(CharStack *s, char c) {
    if (s->top < MAX_LEN - 1) s->data[++(s->top)] = c;
}
static char cs_pop(CharStack *s) {
    return cs_empty(s) ? '\0' : s->data[(s->top)--];
}

/* =========================================================================
 * INTEGER STACK (FOR OPERAND EVALUATION)
 * ========================================================================= */
typedef struct {
    long long data[MAX_LEN];
    int top;
} ValStack;

static void vs_init(ValStack *s) { s->top = -1; }
static bool vs_empty(const ValStack *s) { return s->top == -1; }
static void vs_push(ValStack *s, long long v) {
    if (s->top < MAX_LEN - 1) s->data[++(s->top)] = v;
}
static bool vs_pop(ValStack *s, long long *out) {
    if (vs_empty(s)) return false;
    *out = s->data[(s->top)--];
    return true;
}

/* =========================================================================
 * OPERATOR PRECEDENCE, ASSOCIATIVITY, AND ARITHMETIC HELPERS
 * ========================================================================= */
static bool is_operator(char c) {
    return c == '+' || c == '-' || c == '*' || c == '/' || c == '%' || c == '^';
}

static int precedence(char op) {
    switch (op) {
        case '^': return 3;
        case '*':
        case '/':
        case '%': return 2;
        case '+':
        case '-': return 1;
        default:  return 0;
    }
}

static bool is_right_associative(char op) {
    return op == '^';
}

static long long int_pow(long long base, long long exp) {
    if (exp < 0) return 0;
    long long result = 1;
    while (exp > 0) {
        if (exp & 1) result *= base;
        base *= base;
        exp >>= 1;
    }
    return result;
}

static bool apply_operator(char op, long long op1, long long op2, long long *res) {
    switch (op) {
        case '+': *res = op1 + op2; return true;
        case '-': *res = op1 - op2; return true;
        case '*': *res = op1 * op2; return true;
        case '/':
            if (op2 == 0) { printf("  [ERROR] Division by zero!\n"); return false; }
            *res = op1 / op2;
            return true;
        case '%':
            if (op2 == 0) { printf("  [ERROR] Modulo by zero!\n"); return false; }
            *res = op1 % op2;
            return true;
        case '^': *res = int_pow(op1, op2); return true;
        default:  return false;
    }
}

/* =========================================================================
 * 1. INFIX TO POSTFIX (DIJKSTRA'S SHUNTING-YARD ALGORITHM)
 * ========================================================================= */
bool infix_to_postfix(const char *infix, char *postfix_out) {
    CharStack op_stack;
    cs_init(&op_stack);
    int out_idx = 0;

    for (int i = 0; infix[i] != '\0'; i++) {
        char ch = infix[i];
        if (isspace((unsigned char)ch)) {
            continue;
        }

        /* Case 1: Multi-digit number or variable identifier */
        if (isalnum((unsigned char)ch)) {
            while (isalnum((unsigned char)infix[i])) {
                postfix_out[out_idx++] = infix[i++];
            }
            postfix_out[out_idx++] = ' ';
            i--; /* Step back one index to compensate for outer loop i++ */
        }
        /* Case 2: Opening parenthesis */
        else if (ch == '(') {
            cs_push(&op_stack, ch);
        }
        /* Case 3: Closing parenthesis */
        else if (ch == ')') {
            while (!cs_empty(&op_stack) && cs_peek(&op_stack) != '(') {
                postfix_out[out_idx++] = cs_pop(&op_stack);
                postfix_out[out_idx++] = ' ';
            }
            if (cs_empty(&op_stack)) {
                printf("  [ERROR] Mismatched ')' in expression.\n");
                return false;
            }
            cs_pop(&op_stack); /* Discard '(' */
        }
        /* Case 4: Binary operator */
        else if (is_operator(ch)) {
            while (!cs_empty(&op_stack) && cs_peek(&op_stack) != '(') {
                char top_op = cs_peek(&op_stack);
                int p_top = precedence(top_op);
                int p_cur = precedence(ch);

                /* Pop if higher precedence, OR equal precedence and LEFT-associative */
                if (p_top > p_cur || (p_top == p_cur && !is_right_associative(ch))) {
                    postfix_out[out_idx++] = cs_pop(&op_stack);
                    postfix_out[out_idx++] = ' ';
                } else {
                    break;
                }
            }
            cs_push(&op_stack, ch);
        }
    }

    /* Case 5: Pop all remaining operators */
    while (!cs_empty(&op_stack)) {
        char top_op = cs_pop(&op_stack);
        if (top_op == '(') {
            printf("  [ERROR] Unclosed '(' in expression.\n");
            return false;
        }
        postfix_out[out_idx++] = top_op;
        postfix_out[out_idx++] = ' ';
    }

    if (out_idx > 0 && postfix_out[out_idx - 1] == ' ') {
        out_idx--; /* Trim trailing space */
    }
    postfix_out[out_idx] = '\0';
    return true;
}

/* =========================================================================
 * 2. INFIX TO PREFIX (REVERSE-SCAN SHUNTING-YARD METHOD)
 * ========================================================================= */
bool infix_to_prefix(const char *infix, char *prefix_out) {
    char tokens[MAX_LEN][32];
    int tok_count = 0;
    CharStack op_stack;
    cs_init(&op_stack);

    int len = (int)strlen(infix);
    for (int i = len - 1; i >= 0; i--) {
        char ch = infix[i];
        if (isspace((unsigned char)ch)) {
            continue;
        }

        /* Case 1: Operand (collect full multi-digit number or variable right-to-left) */
        if (isalnum((unsigned char)ch)) {
            int end = i;
            while (i >= 0 && isalnum((unsigned char)infix[i])) {
                i--;
            }
            int start = i + 1;
            int tlen = end - start + 1;
            memcpy(tokens[tok_count], &infix[start], (size_t)tlen);
            tokens[tok_count][tlen] = '\0';
            tok_count++;
            i++; /* Compensate for outer loop i-- */
        }
        /* Case 2: Closing parenthesis acts as opener when scanning right-to-left */
        else if (ch == ')') {
            cs_push(&op_stack, ch);
        }
        /* Case 3: Opening parenthesis pops until matching ')' */
        else if (ch == '(') {
            while (!cs_empty(&op_stack) && cs_peek(&op_stack) != ')') {
                tokens[tok_count][0] = cs_pop(&op_stack);
                tokens[tok_count][1] = '\0';
                tok_count++;
            }
            if (cs_empty(&op_stack)) return false;
            cs_pop(&op_stack); /* Discard ')' */
        }
        /* Case 4: Operator (inverted tie-breaking for reversed scan!) */
        else if (is_operator(ch)) {
            while (!cs_empty(&op_stack) && cs_peek(&op_stack) != ')') {
                char top_op = cs_peek(&op_stack);
                int p_top = precedence(top_op);
                int p_cur = precedence(ch);

                /* In right-to-left scan: pop if p_top > p_cur, OR (p_top == p_cur and RIGHT-associative) */
                if (p_top > p_cur || (p_top == p_cur && is_right_associative(ch))) {
                    tokens[tok_count][0] = cs_pop(&op_stack);
                    tokens[tok_count][1] = '\0';
                    tok_count++;
                } else {
                    break;
                }
            }
            cs_push(&op_stack, ch);
        }
    }

    while (!cs_empty(&op_stack)) {
        char top_op = cs_pop(&op_stack);
        if (top_op == ')') return false;
        tokens[tok_count][0] = top_op;
        tokens[tok_count][1] = '\0';
        tok_count++;
    }

    /* Reverse the token array to build the final prefix string */
    prefix_out[0] = '\0';
    for (int k = tok_count - 1; k >= 0; k--) {
        strcat(prefix_out, tokens[k]);
        if (k > 0) strcat(prefix_out, " ");
    }
    return true;
}

/* =========================================================================
 * 3. POSTFIX EVALUATION (LEFT-TO-RIGHT SCAN)
 *    Crucial Pop Order: op2 = pop(), op1 = pop() -> push(op1 op op2)
 * ========================================================================= */
bool evaluate_postfix(const char *postfix, long long *result_out) {
    ValStack vs;
    vs_init(&vs);

    for (int i = 0; postfix[i] != '\0'; i++) {
        if (isspace((unsigned char)postfix[i])) continue;

        /* Multi-digit integer (or negative number like -5 followed by digit) */
        if (isdigit((unsigned char)postfix[i]) ||
            (postfix[i] == '-' && isdigit((unsigned char)postfix[i + 1]))) {
            char *endptr = NULL;
            long long val = strtoll(&postfix[i], &endptr, 10);
            vs_push(&vs, val);
            i = (int)(endptr - postfix) - 1;
        } else if (is_operator(postfix[i])) {
            long long op2 = 0, op1 = 0, res = 0;
            if (!vs_pop(&vs, &op2) || !vs_pop(&vs, &op1)) {
                printf("  [ERROR] Postfix operand underflow!\n");
                return false;
            }
            if (!apply_operator(postfix[i], op1, op2, &res)) return false;
            vs_push(&vs, res);
        }
    }

    if (vs.top != 0) {
        printf("  [ERROR] Malformed postfix expression (stack size = %d).\n", vs.top + 1);
        return false;
    }
    return vs_pop(&vs, result_out);
}

/* =========================================================================
 * 4. PREFIX EVALUATION (RIGHT-TO-LEFT SCAN)
 *    Crucial Pop Order: op1 = pop(), op2 = pop() -> push(op1 op op2)
 * ========================================================================= */
bool evaluate_prefix(const char *prefix, long long *result_out) {
    ValStack vs;
    vs_init(&vs);

    int len = (int)strlen(prefix);
    for (int i = len - 1; i >= 0; i--) {
        if (isspace((unsigned char)prefix[i])) continue;

        if (isdigit((unsigned char)prefix[i])) {
            while (i >= 0 && (isdigit((unsigned char)prefix[i]) ||
                   (prefix[i] == '-' && (i == 0 || isspace((unsigned char)prefix[i - 1]))))) {
                i--;
            }
            long long val = strtoll(&prefix[i + 1], NULL, 10);
            vs_push(&vs, val);
            i++; /* Compensate for loop i-- */
        } else if (is_operator(prefix[i])) {
            long long op1 = 0, op2 = 0, res = 0;
            /* In Right-to-Left scan, FIRST pop is op1, SECOND pop is op2! */
            if (!vs_pop(&vs, &op1) || !vs_pop(&vs, &op2)) {
                printf("  [ERROR] Prefix operand underflow!\n");
                return false;
            }
            if (!apply_operator(prefix[i], op1, op2, &res)) return false;
            vs_push(&vs, res);
        }
    }

    if (vs.top != 0) return false;
    return vs_pop(&vs, result_out);
}

int main(void) {
    char postfix[MAX_LEN];
    char prefix[MAX_LEN];

    printf("=== 1. Symbolic Expression Conversion ===\n");
    const char *symbolic_tests[] = {
        "A + B * (C ^ D - E) ^ (F + G * H) - I",
        "(A - B / C) * (D ^ E ^ F)",
        "A - B - C",
        "A ^ B ^ C"
    };

    for (size_t i = 0; i < sizeof(symbolic_tests) / sizeof(symbolic_tests[0]); i++) {
        infix_to_postfix(symbolic_tests[i], postfix);
        infix_to_prefix(symbolic_tests[i], prefix);
        printf("  Infix  : %s\n", symbolic_tests[i]);
        printf("  Postfix: %s\n", postfix);
        printf("  Prefix : %s\n\n", prefix);
    }

    printf("=== 2. Numeric Conversion & Evaluation (Multi-Digit Integers) ===\n");
    const char *numeric_infix = "(100 - 20 / 5) * 3 ^ 2";
    infix_to_postfix(numeric_infix, postfix);
    infix_to_prefix(numeric_infix, prefix);

    long long post_val = 0, pre_val = 0;
    evaluate_postfix(postfix, &post_val);
    evaluate_prefix(prefix, &pre_val);

    printf("  Infix Expression  : %s\n", numeric_infix);
    printf("  Converted Postfix : \"%s\" -> Evaluated = %lld\n", postfix, post_val);
    printf("  Converted Prefix  : \"%s\" -> Evaluated = %lld\n", prefix, pre_val);

    return EXIT_SUCCESS;
}
```

#### Program Output:

```text
=== 1. Symbolic Expression Conversion ===
  Infix  : A + B * (C ^ D - E) ^ (F + G * H) - I
  Postfix: A B C D ^ E - F G H * + ^ * + I -
  Prefix : - + A * B ^ - ^ C D E + F * G H I

  Infix  : (A - B / C) * (D ^ E ^ F)
  Postfix: A B C / - D E F ^ ^ *
  Prefix : * - A / B C ^ D ^ E F

  Infix  : A - B - C
  Postfix: A B - C -
  Prefix : - - A B C

  Infix  : A ^ B ^ C
  Postfix: A B C ^ ^
  Prefix : ^ A ^ B C

=== 2. Numeric Conversion & Evaluation (Multi-Digit Integers) ===
  Infix Expression  : (100 - 20 / 5) * 3 ^ 2
  Converted Postfix : "100 20 5 / - 3 2 ^ *" -> Evaluated = 864
  Converted Prefix  : "* - 100 / 20 5 ^ 3 2" -> Evaluated = 864
```

---

## 5. Comparison Tables & Complexity Summary

### 5.1 Comparison of Infix, Postfix, and Prefix Notations

| Criterion | Infix Notation (`A + B * C`) | Postfix / RPN (`A B C * +`) | Prefix / Polish (`+ A * B C`) |
| :--- | :--- | :--- | :--- |
| **Operator Position** | Between operands (`X op Y`) | After operands (`X Y op`) | Before operands (`op X Y`) |
| **Parentheses Required?** | **Yes** (mandatory to override precedence) | **No** (never needed) | **No** (never needed) |
| **Precedence Rules Needed to Evaluate?** | **Yes** | **No** (encoded in token positions) | **No** (encoded in token positions) |
| **Single-Pass Evaluation Direction** | Cannot evaluate in 1 pass without operator + operand stacks | **Left-to-Right** | **Right-to-Left** |
| **Corresponding AST Traversal** | Inorder (with parentheses at subtrees) | Postorder (`Left, Right, Root`) | Preorder (`Root, Left, Right`) |
| **Human Readability** | High (standard algebra) | Moderate | Moderate (resembles function calls `add(A, mul(B, C))`) |

### 5.2 Conversion and Evaluation Rules Comparison Matrix

| Algorithm | Scan Direction | Stack Type Used | Equal-Precedence (`prec(top) == prec(op)`) Tie-Break | Pop Order on Operator |
| :--- | :---: | :---: | :--- | :--- |
| **Infix $\to$ Postfix** | Left $\to$ Right | Operator Stack (`CharStack`) | Pop `top` if `op` is **Left-Associative** (`+ - * / %`); push if Right-Assoc (`^`) | N/A |
| **Infix $\to$ Prefix (Reversed Scan)** | Right $\to$ Left | Operator Stack (`CharStack`) | Pop `top` if `op` is **Right-Associative** (`^`); push if Left-Assoc (`+ - * / %`) | Reverse output tokens at end |
| **Postfix Evaluation** | Left $\to$ Right | Operand Stack (`ValStack`) | N/A | **1st pop = `op2`**, **2nd pop = `op1`** $\to$ `op1 op op2` |
| **Prefix Evaluation** | Right $\to$ Left | Operand Stack (`ValStack`) | N/A | **1st pop = `op1`**, **2nd pop = `op2`** $\to$ `op1 op op2` |

### 5.3 Time and Space Complexity Summary Table

Let $n$ be the number of characters (or tokens) in the input expression.

| Operation | Time Complexity | Auxiliary Stack Space | Output Buffer Space | Justification |
| :--- | :---: | :---: | :---: | :--- |
| **Infix $\to$ Postfix (Shunting-Yard)** | $\Theta(n)$ | $O(n)$ | $\Theta(n)$ | Each token is scanned once, pushed at most once, and popped at most once. |
| **Infix $\to$ Prefix (Reverse Shunting-Yard)** | $\Theta(n)$ | $O(n)$ | $\Theta(n)$ | Single right-to-left pass + $O(n)$ token reversal. |
| **Postfix Evaluation** | $\Theta(n)$ | $O(n)$ | $O(1)$ | Each operand pushed once; each binary operator pops 2 and pushes 1. |
| **Prefix Evaluation** | $\Theta(n)$ | $O(n)$ | $O(1)$ | Single right-to-left pass; each operand pushed once. |

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Frequent C Implementation Bugs and Fixes

#### Pitfall 1: Reversing `op1` and `op2` in Postfix Evaluation (`op1 - op2` vs. `op2 - op1`)
For commutative operators (`+` and `*`), pop order does not change the result. For **non-commutative** operators (`-`, `/`, `%`, `^`), assigning the first popped value to `op1` computes `op2 - op1` and `op2 / op1`!

```c
/* BUG: Assigns first pop to op1 and second pop to op2 in Postfix evaluation! */
long long eval_sub_buggy(ValStack *s) {
    long long op1, op2;
    vs_pop(s, &op1); /* WRONG: Top of stack is the RIGHT operand (op2)! */
    vs_pop(s, &op2);
    return op1 - op2; /* Computes (B - A) instead of (A - B)! */
}

/* FIX: In Postfix (L->R scan), pop op2 FIRST, then pop op1 SECOND. */
long long eval_sub_fixed(ValStack *s) {
    long long op2, op1;
    vs_pop(s, &op2); /* 1st pop = Right operand */
    vs_pop(s, &op1); /* 2nd pop = Left operand  */
    return op1 - op2; /* Correctly computes (A - B) */
}
```

#### Pitfall 2: Treating Exponentiation `^` as Left-Associative in Shunting-Yard
If you pop on `prec(top) >= prec(op)` unconditionally for all operators, then `A ^ B ^ C` will pop the first `^` when the second `^` arrives, producing `A B ^ C ^` (`(A ^ B) ^ C`) instead of `A B C ^ ^` (`A ^ (B ^ C)`). For $2^{3^2}$, $(2^3)^2 = 64 \ne 2^{(3^2)} = 512$!

```c
/* BUG: Pops unconditionally on >=, breaking right-associative '^'! */
if (precedence(top_op) >= precedence(ch)) {
    cs_pop(&op_stack);
}

/* FIX: Check associativity when precedences are equal! */
if (precedence(top_op) > precedence(ch) ||
    (precedence(top_op) == precedence(ch) && !is_right_associative(ch))) {
    cs_pop(&op_stack);
}
```

#### Pitfall 3: Appending `'('` to Postfix Output When Popping Until `'('`
When a closing parenthesis `')'` is encountered, all operators above `'('` are popped to output, and `'('` itself must be popped and **discarded**. Leaving `'('` on the stack causes subsequent operators to sit above `'('` and fails at the end of the expression.

#### Pitfall 4: Concatenating Multi-Digit Numbers Without Spaces in Postfix Output
Converting `"12 + 34"` without token delimiters produces `"1234+"`, which is indistinguishable from `"1 + 234"` or `"123 + 4"`. Always separate multi-digit tokens with a delimiter space (`"12 34 +"`).

### 6.2 Boundary Conditions Checklist

- [ ] **Single Operand Expression (`"42"`)**: Converts to `"42"` and evaluates to `42` with 0 operators pushed.
- [ ] **Deeply Nested Redundant Parentheses (`"(((A + B)))"`)**: All `'('` and `')'` are cleanly matched and discarded, producing `"A B +"`.
- [ ] **Chained Left-Associative Subtraction (`"10 - 3 - 2"`)**: Converts to Postfix `"10 3 - 2 -"` and Prefix `"- - 10 3 2"`, evaluating to $(10 - 3) - 2 = 5$ (NOT $10 - (3 - 2) = 9$).
- [ ] **Chained Right-Associative Exponentiation (`"2 ^ 3 ^ 2"`)**: Converts to Postfix `"2 3 2 ^ ^"` and Prefix `"^ 2 ^ 3 2"`, evaluating to $2^{(3^2)} = 2^9 = 512$ (NOT $(2^3)^2 = 64$).
- [ ] **Division or Modulo by Zero (`"10 0 /"`)**: Checked explicitly before executing `op1 / op2` or `op1 % op2`.
- [ ] **Malformed Expression (`"10 + * 5"` or `"10 20 30 +"`)**: Caught by operand stack underflow check or final `vs.top == 0` check.

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways and Core Invariants

1. **Parenthesis Elimination**: Postfix and Prefix notations encode operator precedence and associativity directly into token order, eliminating parentheses and enabling single-pass $O(n)$ evaluation.
2. **Operand Order Invariant**: Converting between Infix, Postfix, and Prefix **never alters the left-to-right sequence of operands**—only operators change position.
3. **Associativity Tie-Breaking**:
   - In **Infix-to-Postfix** (left-to-right scan), pop equal-precedence operators iff the operator is **left-associative**.
   - In **Infix-to-Prefix** (right-to-left scan), pop equal-precedence operators iff the operator is **right-associative**.
4. **Evaluation Pop Order**:
   - **Postfix Evaluation** (left-to-right scan): `op2 = pop()`, then `op1 = pop()` $\to$ compute `op1 op op2`.
   - **Prefix Evaluation** (right-to-left scan): `op1 = pop()`, then `op2 = pop()` $\to$ compute `op1 op op2`.

### 7.2 Self-Test Revision Checklist

- [ ] Can I convert `A + B * (C ^ D - E) ^ (F + G * H) - I` into Postfix and Prefix by hand using a stack table?
- [ ] Can I explain why `A - B - C` becomes `A B - C -` whereas `A ^ B ^ C` becomes `A B C ^ ^`?
- [ ] Can I explain why the equal-precedence rule flips when scanning right-to-left for Infix-to-Prefix conversion?
- [ ] Can I trace Postfix evaluation on `"100 20 5 / - 3 2 ^ *"` and Prefix evaluation on `"* - 100 / 20 5 ^ 3 2"` without mixing up `op1` and `op2`?

### 7.3 Navigation Links

- **Previous Topic**: [08 — Stack Applications and Recursion](08-Stack-Applications-and-Recursion.md)
- **Next Topic (Unit Review)**: [10 — Unit 1 Comprehensive Review & Exam Preparation](10-Unit-1-Review.md)
