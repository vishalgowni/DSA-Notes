# Unit 4 — Topic 2: Trie Trees (Prefix Trees / Digital Search Trees)

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [What Is a Trie? Origin and Etymology](#11-what-is-a-trie-origin-and-etymology)
   - 1.2 [Why Do We Need Tries? The Problem They Solve](#12-why-do-we-need-tries-the-problem-they-solve)
   - 1.3 [Relationship to Prior Concepts: Comparison-Based Trees vs. Character-Indexed Digital Trees](#13-relationship-to-prior-concepts-comparison-based-trees-vs-character-indexed-digital-trees)
   - 1.4 [Real-World Engineering and Computer Science Applications](#14-real-world-engineering-and-computer-science-applications)
   - 1.5 [Learning Objectives](#15-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Formal Definition of a Trie Over Alphabet $\Sigma$](#21-formal-definition-of-a-trie-over-alphabet-sigma)
   - 2.2 [Structural Properties: Root, Edges, Shared Prefixes, and Terminal Markers](#22-structural-properties-root-edges-shared-prefixes-and-terminal-markers)
   - 2.3 [Core Terminology Table](#23-core-terminology-table)
   - 2.4 [Mathematical Properties, Invariants, and Formulas](#24-mathematical-properties-invariants-and-formulas)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Alphabet Specification ($\Sigma = \{\text{'a'}\dots\text{'z'}\}$, `ALPHABET_SIZE = 26`) and Index Mapping](#31-alphabet-specification-sigma--textadotstextz-alphabet_size--26-and-index-mapping)
   - 3.2 [Standard Array-of-Pointers `TrieNode` Structure in C](#32-standard-array-of-pointers-trienode-structure-in-c)
   - 3.3 [ASCII Memory Layout Diagram of a Trie Node and Pointer Array](#33-ascii-memory-layout-diagram-of-a-trie-node-and-pointer-array)
   - 3.4 [Alternative Node Representations: Fixed Array vs. Linked Children (LCRS) vs. Hash Map](#34-alternative-node-representations-fixed-array-vs-linked-children-lcrs-vs-hash-map)
4. [Operations and Algorithms](#4-operations-and-algorithms)
   - 4.1 [Insertion (`trieInsert`) with Step-by-Step ASCII Trace](#41-insertion-trieinsert-with-step-by-step-ascii-trace)
   - 4.2 [Exact Word Search (`trieSearch`) vs. Prefix Trap](#42-exact-word-search-triesearch-vs-prefix-trap)
   - 4.3 [Prefix Search (`trieStartsWith`, `trieCountPrefix`) and Autocomplete / Lexicographical Traversal](#43-prefix-search-triestartswith-triecountprefix-and-autocomplete--lexicographical-traversal)
   - 4.4 [Deletion (`trieDelete`): Recursive Postorder Handling of All 4 Structural Cases](#44-deletion-triedelete-recursive-postorder-handling-of-all-4-structural-cases)
   - 4.5 [Complete C11 Implementation: Insertion, Search, Prefix Count, Autocomplete, and 4-Case Deletion](#45-complete-c11-implementation-insertion-search-prefix-count-autocomplete-and-4-case-deletion)
5. [Comparison Tables and Complexity Summary](#5-comparison-tables-and-complexity-summary)
   - 5.1 [Time and Space Complexity Table for Trie Operations](#51-time-and-space-complexity-table-for-trie-operations)
   - 5.2 [Data Structure Comparison: Trie vs. Balanced BST vs. Hash Table for String Keys](#52-data-structure-comparison-trie-vs-balanced-bst-vs-hash-table-for-string-keys)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Pitfall 1: Returning `true` in Exact Search Without Checking `is_end_of_word`](#61-pitfall-1-returning-true-in-exact-search-without-checking-is_end_of_word)
   - 6.2 [Pitfall 2: Freeing Shared Prefix Nodes During Deletion (Breaking Longer or Shorter Words)](#62-pitfall-2-freeing-shared-prefix-nodes-during-deletion-breaking-longer-or-shorter-words)
   - 6.3 [Pitfall 3: Out-of-Bounds Array Indexing on Uppercase or Non-Alphabetic Characters](#63-pitfall-3-out-of-bounds-array-indexing-on-uppercase-or-non-alphabetic-characters)
   - 6.4 [Pitfall 4: Accidentally Freeing the Root Node When Deleting the Last Word](#64-pitfall-4-accidentally-freeing-the-root-node-when-deleting-the-last-word)
   - 6.5 [Boundary Conditions Checklist](#65-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)
   - 7.1 [Key Takeaways and Core Invariants](#71-key-takeaways-and-core-invariants)
   - 7.2 [Self-Test Revision Questions](#72-self-test-revision-questions)
   - 7.3 [Navigation Links](#73-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 What Is a Trie? Origin and Etymology

A **Trie** (also known as a **Prefix Tree**, **Radix Tree**, or **Digital Search Tree**) is an ordered, multi-way tree data structure specialized for storing and retrieving a dynamic set or associative array of **strings** (or sequences of digits/bits) over a finite alphabet $\Sigma$.

- **Origin:** First described by René de la Briandais in 1959 for file searching and named **"Trie"** by **Edward Fredkin in 1960** from the middle syllable of the word **re-TRIE-val**.
- **Pronunciation:** Fredkin originally pronounced it *"tree"*, though in modern computer science it is almost universally pronounced *"try"* (rhyming with *pie*) to verbally distinguish it from a general tree.
- **Defining Characteristic:** Unlike a Binary Search Tree (BST)—where each node stores an entire key string and branches left or right based on whole-key comparisons (`strcmp`)—**a node in a Trie does not store the key itself**. Instead, a node's **position (path from the root)** defines the prefix string associated with it! Each edge leaving a node corresponds to a single character $c \in \Sigma$.

```
         BALANCED BST OF STRINGS                         TRIE (PREFIX TREE) OF THE SAME 4 STRINGS
    (Whole-key strcmp() at each node)                  (Shared prefixes share the exact same path!)

               ["apply"]                                                (root: "")
               /       \                                                 /      \
          ["app"]     ["bat"]                                        'a'/        \'b'
              \                                                       (a)        (b)
           ["apple"]                                                'p'|          |'a'
                                                                      (ap)       (ba)
    Searching "apple" compares "apple"                              'p'|          |'t'
    character-by-character at EVERY level!                           [app]*     [bat]*
                                                                    'l'|
                                                                     (appl)
                                                                   'e'/  \'y'
                                                                [apple]* [apply]*
                                                        (* denotes is_end_of_word = true)
```

### 1.2 Why Do We Need Tries? The Problem They Solve

Suppose we maintain a dictionary of $n = 1{,}000{,}000$ English words and want to look up a query word $W$ of length $m = 6$ characters (e.g., `"system"`):

1. **The Bottleneck of Balanced BSTs ($O(m \log n)$):** In an AVL or Red-Black Tree with $n$ strings, a search traverses $O(\log_2 n) \approx 20$ tree levels. At each level, `strcmp(W, node->key)` compares up to $m$ characters. Total worst-case time: **$O(m \log n)$** character comparisons.
2. **The Bottleneck of Hash Tables for Prefix Queries ($O(n \cdot m)$):** While a Hash Table can look up an *exact* word in $O(m)$ average time (computing the hash of $m$ characters), hashing scrambles key order completely. If a user types `"sys"` into a search bar and asks for **all words starting with `"sys"` (Autocomplete)** or **how many words start with `"sys"`**, a Hash Table must scan all $n$ buckets in **$O(n \cdot m)$** time!
3. **The Trie Breakthrough ($O(m)$ Independent of $n$!):** In a Trie, looking up, inserting, or deleting a word of length $m$ follows exactly $m$ character pointers from the root. Whether the Trie contains $n = 10$ words or $n = 10{,}000{,}000$ words, exact search and prefix lookup take strictly **$O(m)$ worst-case time**—**completely independent of $n$**!

### 1.3 Relationship to Prior Concepts: Comparison-Based Trees vs. Character-Indexed Digital Trees

| Dimension | Binary Search Tree (BST / AVL) | Trie (Prefix Tree) |
| :--- | :--- | :--- |
| **Branching Mechanism** | Whole-key comparison (`strcmp(k1, k2) < 0`) | Direct array indexing by character $c_i$ (`children[c_i - 'a']`) |
| **Maximum Fan-Out (Degree)** | $2$ (`left`, `right`) | $|\Sigma|$ (e.g., $26$ for lowercase English, $256$ for ASCII) |
| **Where Keys Are Stored** | Explicitly inside each node (`char *key`) | Implicitly along root-to-node paths + `is_end_of_word` flag |
| **Tree Height** | $O(\log n)$ (balanced) or $O(n)$ (skewed) | Strictly $\max_{i} |W_i|$ (length of longest word $M$) |
| **Depends on Insertion Order?** | **Yes** (unbalanced BST shape varies wildly) | **No!** A Trie's shape is **canonical** (unique for any set of keys) |

### 1.4 Real-World Engineering and Computer Science Applications

1. **Search Engine Autocomplete & IDE IntelliSense:** Typing `"pri"` in an IDE or browser immediately navigates $3$ steps to the `"pri"` node and enumerates `"print"`, `"printf"`, `"private"` from its subtree.
2. **IP Routing Tables (Longest Prefix Match in Hardware Routers):** Internet routers store CIDR IP prefixes (binary strings over $\Sigma = \{0, 1\}$ of length $\le 32$ for IPv4 or $\le 128$ for IPv6) in binary/compressed Tries to forward packets in $O(\text{bit-length})$ time.
3. **Spell Checkers & Word Games (Scrabble / Boggle / Wordle Solvers):** Pruning invalid character combinations on a board immediately as soon as `trieStartsWith(prefix)` returns `false`.
4. **T9 Predictive Text & Bioinformatics Sequence Indexing:** Mapping digit sequences or DNA $k$-mers ($\Sigma = \{\text{A, C, G, T}\}$, $|\Sigma| = 4$) to matching words or genomic loci.

### 1.5 Learning Objectives

By the end of this topic, you will be able to:
- Define a Trie formally over an alphabet $\Sigma$ and explain why its operation time $O(m)$ is independent of the dictionary size $n$.
- Design and lay out a `TrieNode` in C using character-to-index mapping (`idx = ch - 'a'`), `is_end_of_word`, `prefix_count`, and `word_count`.
- Trace and implement **Insertion**, **Exact Word Search** (avoiding the prefix false-positive bug), **Prefix Search**, and **Lexicographical Autocomplete**.
- Trace and implement recursive postorder **Deletion** across all **4 structural cases** without leaking memory or corrupting shared prefixes.

---

## 2. Definitions and Terminology

### 2.1 Formal Definition of a Trie Over Alphabet $\Sigma$

Let $\Sigma$ be a finite, totally ordered **alphabet** of size $|\Sigma|$ (for example, the lowercase English alphabet $\Sigma = \{\text{'a'}, \text{'b'}, \dots, \text{'z'}\}$ with $|\Sigma| = 26$). Let $\mathcal{D} = \{W_1, W_2, \dots, W_n\} \subseteq \Sigma^*$ be a finite set of strings over $\Sigma$.

A **Trie** $T(\mathcal{D})$ representing $\mathcal{D}$ is a rooted, edge-labeled (or slot-indexed) ordered tree satisfying:
1. **Root Node:** The root $r$ corresponds to the **empty string** $\varepsilon = \text{""}$ at depth $0$.
2. **Unique Outgoing Character Edges:** Every node $u \in T$ has at most $|\Sigma|$ children, where each outgoing edge $(u, v)$ is labeled with a distinct character $c \in \Sigma$. No two edges leaving the same parent $u$ can share the same character label.
3. **Path-Prefix Bijection:** Every node $u$ at depth $d$ corresponds uniquely to the string prefix $P(u) = c_1 c_2 \dots c_d \in \Sigma^d$ formed by concatenating the character labels along the unique path from the root $r$ to $u$.
4. **Terminal Word Marking:** Each node $u$ stores a boolean predicate $\text{is\_end\_of\_word}(u) \in \{\text{true}, \text{false}\}$, where:
   $$\text{is\_end\_of\_word}(u) = \text{true} \iff P(u) \in \mathcal{D}$$

### 2.2 Structural Properties: Root, Edges, Shared Prefixes, and Terminal Markers

Consider inserting the key set $\mathcal{D} = \{\text{"app"}, \text{"apple"}, \text{"apply"}, \text{"apt"}, \text{"bad"}, \text{"bat"}, \text{"cat"}\}$. Observe how **shared prefixes are stored only once**:

```
                                       (root: "") [prefix_count = 7]
                                      /     |     \
                                 'a' /  'b' |      \ 'c'
                                    /       |       \
                              (a) [p=4]  (b) [p=2]  (c) [p=1]
                              'p'|       'a'|       'a'|
                             (ap) [p=4] (ba) [p=2] (ca) [p=1]
                            'p'/   \'t' 'd'/ \'t'   't'|
                              /     \     /   \     [cat]* (w=1, p=1)
                      [app]* (w=1) [apt]*[bad]*[bat]*
                             (p=3)
                             'l'|
                            (appl) [p=2]
                           'e'/   \'y'
                             /     \
                       [apple]*   [apply]*
                       (w=1,p=1)  (w=1,p=1)

  Legend:
    (x)    : Non-terminal internal node (is_end_of_word == false)
    [xyz]* : Terminal word node (is_end_of_word == true, word_count >= 1)
    p=k    : prefix_count (number of inserted words in this node's subtree, including itself)
```

Notice two crucial facts from this diagram:
1. **A word node does NOT have to be a leaf node!** The node for `"app"` has `is_end_of_word = true`, yet it has a child `'l'` leading to `"apple"` and `"apply"`.
2. **Every leaf node in a trimmed Trie MUST be a word node** (`is_end_of_word == true`), because any node with `is_end_of_word == false` and zero children represents no word and can be pruned.

### 2.3 Core Terminology Table

| Term | Definition | C Representation / Invariant |
| :--- | :--- | :--- |
| **Alphabet ($\Sigma$)** | Finite set of allowed characters in keys | `ALPHABET_SIZE = 26` for `'a'..'z'` |
| **Empty Prefix ($\varepsilon$)** | Zero-length string `""` | Represented by the `root` node at depth $0$ |
| **Character Index (`idx`)** | 0-based array slot mapped from character `ch` | `idx = ch - 'a'` ($\in \{0 \dots 25\}$) |
| **Terminal Flag (`is_end_of_word`)** | Boolean indicating the path from root to this node is a complete stored word | `node->is_end_of_word == true` |
| **Prefix Count (`prefix_count`)** | Count of words in the Trie that have $P(u)$ as a prefix | Incremented on `insert`, decremented on `delete` |
| **Word Count (`word_count`)** | Multiplicity of exact word $P(u)$ in a multiset Trie | `node->word_count > 0 <==> node->is_end_of_word` |

### 2.4 Mathematical Properties, Invariants, and Formulas

1. **Node Depth and Height Invariant:**
   The depth of the node representing prefix $P$ is exactly $|P|$ (the number of characters in $P$). If $\mathcal{D} = \{W_1, \dots, W_n\}$, the height $H$ of the Trie is strictly the length of the longest string in $\mathcal{D}$:
   $$H = \max_{1 \le i \le n} |W_i|$$

2. **Bounds on Total Number of Nodes ($N_{\text{nodes}}$):**
   Let $M = \sum_{i=1}^{n} |W_i|$ be the total number of characters across all $n$ distinct non-empty words in $\mathcal{D}$, and let $L_{\max} = \max_i |W_i|$.
   - **Best Case (Maximum Prefix Sharing):** All $n$ words are prefixes of a single longest word of length $L_{\max}$ (e.g., `"a"`, `"ab"`, `"abc"`, $\dots$). Then:
     $$N_{\text{nodes}}^{\min} = L_{\max} + 1 \quad (\text{including root})$$
   - **Worst Case (Zero Prefix Sharing):** No two words share even their first character (or diverge immediately), creating a new node for every character of every word:
     $$N_{\text{nodes}}^{\max} = 1 + \sum_{i=1}^{n} |W_i| = M + 1$$
   - In general:
     $$1 + \max_{1 \le i \le n} |W_i| \le N_{\text{nodes}} \le 1 + \sum_{i=1}^{n} |W_i|$$

3. **Subtree Prefix-Count Conservation Invariant:**
   For every node $u$ in the Trie, its `prefix_count` equals its own `word_count` plus the sum of `prefix_count` across all of its non-NULL children:
   $$\text{prefix\_count}(u) = \text{word\_count}(u) + \sum_{c \in \Sigma, \, u.\text{child}[c] \neq \text{NULL}} \text{prefix\_count}(u.\text{child}[c])$$

---

## 3. Representation and Memory Layout in C

### 3.1 Alphabet Specification ($\Sigma = \{\text{'a'}\dots\text{'z'}\}$, `ALPHABET_SIZE = 26`) and Index Mapping

In C, characters are small integers (ASCII codes: `'a' = 97`, `'b' = 98`, ..., `'z' = 122`). Rather than searching a list of children to find the edge for character `ch`, we subtract `'a'` to map `'a'..'z'` directly onto array indices `0..25` in **$O(1)$ time**:

$$\text{idx}(ch) = \text{ASCII}(ch) - \text{ASCII}(\text{'a'}) \in \{0, 1, \dots, 25\}$$
$$\text{char}(\text{idx}) = \text{'a'} + \text{idx}$$

```
  Character ch:   'a'   'b'   'c'   'd'  ...  'p'  ...  'y'   'z'
  ASCII Code:      97    98    99   100  ...  112  ...  121   122
  ch - 'a':       [ 0]  [ 1]  [ 2]  [ 3] ... [15]  ... [24]  [25]
```

### 3.2 Standard Array-of-Pointers `TrieNode` Structure in C

```c
#include <stdbool.h>

#define ALPHABET_SIZE 26

typedef struct TrieNode {
    struct TrieNode *children[ALPHABET_SIZE]; /* 26 pointers for 'a'..'z'       */
    bool is_end_of_word;                      /* true if path forms a full word */
    int prefix_count;                         /* Words sharing this prefix      */
    int word_count;                           /* Exact occurrences of this word */
} TrieNode;
```

- **Memory Size of a Single `TrieNode` on a 64-bit System:**
  - `children[26]`: $26 \times 8\text{ bytes} = 208\text{ bytes}$.
  - `is_end_of_word` (`bool`, 1 byte + 3 bytes padding) + `prefix_count` (4 bytes) + `word_count` (4 bytes) + struct alignment padding = $16\text{ bytes}$.
  - **Total per node:** **$224\text{ bytes}$** (for $|\Sigma| = 26$). If full 8-bit ASCII ($|\Sigma| = 256$) were used, each node would consume $256 \times 8 + 16 = 2064\text{ bytes}$ ($\approx 2\text{ KB}$ per node).

### 3.3 ASCII Memory Layout Diagram of a Trie Node and Pointer Array

Below is how the path for `"ab"` is physically stored in heap memory:

```
  root (TrieNode @ 0x1000)
  +-------------------------------------------------------------------+
  | children[0..25]:                                                  |
  |   [0 ('a')]      [1 ('b')]      [2 ('c')]   ...   [25 ('z')]      |
  |   0x2040         NULL           NULL        ...   NULL            |
  +-----|-------------------------------------------------------------+
  | is_end_of_word = false | prefix_count = 1 | word_count = 0        |
  +-----|-------------------------------------------------------------+
        |
        v
  node_a (TrieNode @ 0x2040)  <-- Represents prefix "a"
  +-------------------------------------------------------------------+
  | children[0..25]:                                                  |
  |   [0 ('a')]      [1 ('b')]      [2 ('c')]   ...   [25 ('z')]      |
  |   NULL           0x3080         NULL        ...   NULL            |
  +--------------------|----------------------------------------------+
  | is_end_of_word = false | prefix_count = 1 | word_count = 0        |
  +--------------------|----------------------------------------------+
                       |
                       v
                 node_ab (TrieNode @ 0x3080) <-- Represents word "ab"
                 +----------------------------------------------------+
                 | children[0..25]: [ ALL 26 SLOTS == NULL ]          |
                 +----------------------------------------------------+
                 | is_end_of_word = true  | prefix_count = 1 | w = 1  |
                 +----------------------------------------------------+
```

### 3.4 Alternative Node Representations: Fixed Array vs. Linked Children (LCRS) vs. Hash Map

When $|\Sigma|$ is large (e.g., Unicode UTF-16/UTF-32 or full 256-byte ASCII) and most nodes near the leaves have only $1$ or $2$ non-NULL children, 95% of the `children[]` array pointers are wasted `NULL`s. Three representations trade speed against space:

| Representation Strategy | Child Lookup Time | Space per Node | Lexicographical Order Preserved? | Best Use Case |
| :--- | :---: | :---: | :---: | :--- |
| **1. Fixed Pointer Array (`children[|\Sigma|]`)** | **$O(1)$** (direct index) | $O(|\Sigma|)$ pointers | **Yes** (`0 .. |\Sigma|-1` is sorted) | Small alphabets: DNA ($4$), digits ($10$), `'a'..'z'` ($26$) |
| **2. Sorted Linked List (Left-Child Right-Sibling)** | $O(|\Sigma|)$ scan | **$O(1)$** (`first_child`, `next_sibling`, `char ch`) | **Yes** (if siblings kept sorted) | Memory-constrained embedded dictionaries |
| **3. Per-Node Hash Table / Dynamic Map** | $O(1)$ average | $O(\text{deg}(u))$ (proportional to actual children) | **No** (requires sorting keys to traverse alphabetically) | Large alphabets (Unicode, CJK characters) |

---

## 4. Operations and Algorithms

### 4.1 Insertion (`trieInsert`) with Step-by-Step ASCII Trace

#### 1. Purpose and Preconditions
Insert a null-terminated string `word` of length $m$ into the Trie rooted at `root`. If `word` already exists, increment its `word_count` and keep `is_end_of_word = true`.

#### 2. Step-by-Step Algorithm
1. Start at `curr = root` and increment `curr->prefix_count++`.
2. For each character `word[i]` from $i = 0$ to $m - 1$:
   - Compute `idx = word[i] - 'a'`.
   - If `curr->children[idx] == NULL`, allocate a new zero-initialized `TrieNode` and assign it to `curr->children[idx]`.
   - Advance `curr = curr->children[idx]`.
   - Increment `curr->prefix_count++`.
3. After the loop finishes (at depth $m$), mark the terminal node:
   - `curr->is_end_of_word = true;`
   - `curr->word_count++;`

#### 3. Pseudocode

```text
ALGORITHM TrieInsert(root, word)
  Input:  Pointer to Trie root, string word[0 .. m-1]
  Output: Modifies Trie in-place to include word

  curr <- root
  curr.prefix_count <- curr.prefix_count + 1

  for i <- 0 to m - 1 do
      idx <- word[i] - 'a'
      if curr.children[idx] == NULL then
          curr.children[idx] <- CreateTrieNode()
      curr <- curr.children[idx]
      curr.prefix_count <- curr.prefix_count + 1

  curr.is_end_of_word <- true
  curr.word_count <- curr.word_count + 1
```

#### 4. Step-by-Step ASCII Trace: Inserting `"app"`, `"apple"`, and `"apt"`

```
  STEP 1: Insert("app")                  STEP 2: Insert("apple")                STEP 3: Insert("apt")
  Creates 3 new nodes: a -> p -> p       Reuses a -> p -> p; adds l -> e        Reuses a -> p; branches 't' at (ap)

          (root)                                 (root)                                  (root)
          'a'|                                   'a'|                                    'a'|
            (a)                                    (a)                                     (a)
          'p'|                                   'p'|                                    'p'|
           (ap)                                   (ap)                                    (ap) <-- Shared!
          'p'|                                   'p'|                                   'p'/ \'t'
          [app]*                                 [app]*  <-- stays true!              [app]* [apt]*
                                                 'l'|                                 'l'|
                                                 (appl)                               (appl)
                                                 'e'|                                 'e'|
                                                [apple]*                             [apple]*
```

---

### 4.2 Exact Word Search (`trieSearch`) vs. Prefix Trap

#### 1. Purpose and The Prefix Trap
Determine whether the exact string `word` was previously inserted into the Trie.
- **Why Checking `curr != NULL` Alone Is a Bug:** Suppose we only inserted `"apple"` into an empty Trie, and we call `trieSearch(root, "app")`. Walking `'a' -> 'p' -> 'p'` succeeds and lands on a valid, non-NULL node `(app)`! However, `"app"` was never inserted as a standalone word—it is merely an internal ancestor along the path to `"apple"`.
- **The Exact Search Rule:** `trieSearch(root, word)` returns `true` **if and only if**:
  1. Every character `word[i]` has a non-NULL child pointer along the path from `root`, **AND**
  2. The final node reached at the end of `word` satisfies **`curr->is_end_of_word == true`**!

```
  Trie containing ONLY "apple":
     (root) --'a'--> (a) --'p'--> (ap) --'p'--> (app) --'l'--> (appl) --'e'--> [apple]*
                                                  ^                              ^
                                    is_end_of_word == FALSE        is_end_of_word == TRUE
                                    trieSearch("app")   -> FALSE   trieSearch("apple") -> TRUE
                                    trieStartsWith("app")-> TRUE
```

#### 2. Pseudocode

```text
ALGORITHM TrieSearch(root, word)
  Input:  Pointer to Trie root, query string word[0 .. m-1]
  Output: true if word is stored as a complete word in the Trie, false otherwise

  curr <- root
  for i <- 0 to m - 1 do
      idx <- word[i] - 'a'
      if idx < 0 or idx >= ALPHABET_SIZE or curr.children[idx] == NULL then
          return false                  // Path breaks early: word not in Trie
      curr <- curr.children[idx]

  return (curr != NULL and curr.is_end_of_word == true)
```

---

### 4.3 Prefix Search (`trieStartsWith`, `trieCountPrefix`) and Autocomplete / Lexicographical Traversal

#### 1. Prefix Existence (`trieStartsWith`) and Prefix Frequency (`trieCountPrefix`)
- **`trieStartsWith(root, prefix)`:** Follows the exact same loop as `trieSearch`, except at the end of `prefix` it simply returns `curr != NULL` (ignoring `is_end_of_word`). Time complexity: $O(m)$ where $m = |\text{prefix}|$.
- **`trieCountPrefix(root, prefix)`:** Walks to the node `curr` at the end of `prefix` in $O(m)$ steps and returns `curr->prefix_count` in **$O(1)$ additional time**—telling us immediately how many dictionary words start with `prefix` without visiting a single descendant node!

#### 2. Autocomplete and Automatic Lexicographical Enumeration (`trieAutocomplete`)
Why does a preorder DFS over a Trie naturally output all words in **strictly alphabetical (lexicographical) order** without running any sorting algorithm (`qsort`)?
- At every node `u`, we iterate child index `i` from `0` (`'a'`) to `25` (`'z'`).
- Any word that checks `is_end_of_word` at `u` itself (shorter prefix) is printed **before** its longer extensions (`"app"` before `"apple"`).
- Any word branching through child `i` (`'e'` in `"apple"`) is fully explored **before** any word branching through child `j > i` (`'y'` in `"apply"`).

```text
ALGORITHM TrieAutocomplete(root, prefix)
  curr <- root
  buffer <- EmptyCharArray()
  depth <- 0

  // Phase 1: Walk down to the prefix node in O(|prefix|) time
  for i <- 0 to |prefix| - 1 do
      idx <- prefix[i] - 'a'
      if curr.children[idx] == NULL then
          print "No completions found"
          return
      buffer[depth] <- prefix[i]
      depth <- depth + 1
      curr <- curr.children[idx]

  // Phase 2: Preorder DFS from curr to enumerate all words in its subtree
  CollectWordsDFS(curr, buffer, depth)

PROCEDURE CollectWordsDFS(node, buffer, depth)
  if node.is_end_of_word == true then
      buffer[depth] <- '\0'
      print buffer

  for i <- 0 to ALPHABET_SIZE - 1 do
      if node.children[i] != NULL then
          buffer[depth] <- 'a' + i
          CollectWordsDFS(node.children[i], buffer, depth + 1)
```

---

### 4.4 Deletion (`trieDelete`): Recursive Postorder Handling of All 4 Structural Cases

Deleting a key `word` of length $m$ from a Trie is the most subtle operation because nodes along `word`'s path may be **shared** with shorter words (prefixes of `word`) or longer/sibling words (extensions sharing a prefix with `word`). We must **walk down to the terminal node first** and then **clean up unshared nodes bottom-up (postorder)** on the way back up the recursion stack.

#### The 4 Structural Cases of Trie Deletion

Let us trace all **4 structural cases** with concrete examples and ASCII state diagrams:

1. **Case 1: `word` Is Not Present in the Trie**
   - *Condition:* Either a character pointer along `word` is `NULL` (e.g., deleting `"dog"`), OR the path exists as a proper prefix of another word but `terminal_node->is_end_of_word == false` (e.g., deleting `"ap"` when only `"app"` and `"apple"` are stored).
   - *Action:* Do nothing (`return false`). Modify zero nodes and zero `prefix_count` fields.

2. **Case 2: `word` Is a Proper Prefix of Another Longer Word in the Trie**
   - *Example:* Deleting `"app"` when `"apple"` and `"apply"` are also in the Trie.
   - *Condition at depth $m$:* The terminal node for `"app"` has `hasAnyChild(curr) == true` (its `'l'` child is non-NULL).
   - *Action:* Unmark `curr->is_end_of_word = false` and `curr->word_count = 0`, decrement `prefix_count` along `"app"`, and return `should_free = false` so **no nodes are freed**!

```
  CASE 2: Delete("app") when "apple" exists
  BEFORE:                                     AFTER:
      (ap)                                        (ap)
      'p'|                                        'p'|
      [app]*  <-- is_end_of_word = TRUE           (app)   <-- is_end_of_word = FALSE (NOT freed!)
      'l'|                                        'l'|
     (appl)                                      (appl)
      'e'|                                        'e'|
     [apple]*                                    [apple]* <-- "apple" remains intact!
```

3. **Case 3: `word` Shares a Prefix with Another Shorter or Sibling Word**
   - *Example:* Deleting `"apple"` when `"apply"` (or `"app"`) exists in the Trie.
   - *Condition:* At the terminal node `'e'` of `"apple"`, `hasAnyChild(e) == false`, so `'e'` is freed! As recursion unwinds upward from depth $m$ toward the root, we continue freeing ancestor nodes **until** we hit the first ancestor `u` that either:
     - has another remaining child (`hasAnyChild(u) == true`, such as node `(appl)` which still has child `'y'` for `"apply"`), OR
     - is itself the end of a shorter stored word (`u->is_end_of_word == true`, such as `[app]*`).
   - *Action:* Free only the unshared suffix tail (`'e'`), set `(appl)->children['e' - 'a'] = NULL`, and stop freeing upward (`return false`).

```
  CASE 3: Delete("apple") when "apply" exists
  BEFORE:                                     AFTER:
        (appl)                                      (appl)  <-- Stops freeing here! (still has child 'y')
       'e'/  \'y'                                  'e'/  \'y'
         /    \                                    NULL   \
    [apple]* [apply]*                             (freed) [apply]*
```

4. **Case 4: `word` Shares No Characters with Any Other Word (Independent Branch)**
   - *Example:* Deleting `"cat"` when no other word in the Trie begins with `'c'`.
   - *Condition:* Terminal node `'t'` has no children, and neither `(ca)` nor `(c)` has any other children or `is_end_of_word == true`.
   - *Action:* Postorder recursion frees `'t'`, then `(ca)`, then `(c)`, setting `root->children['c' - 'a'] = NULL`. Note that **`root` itself is NEVER freed** (`depth > 0` check)!

```
  CASE 4: Delete("cat") when no other 'c'-word exists
  BEFORE:                                     AFTER:
        (root)                                      (root)  <-- Root is NEVER freed!
        'c'|                                        'c'|
          (c)   <-- freed                             NULL
        'a'|
         (ca)   <-- freed
        't'|
        [cat]*  <-- freed
```

#### Pseudocode for Recursive Postorder Deletion

```text
ALGORITHM TrieDelete(root, word)
  if not TrieSearch(root, word) then
      return false                            // Case 1: word not in Trie
  DeleteHelper(root, word, 0, |word|)
  return true

FUNCTION DeleteHelper(curr, word, depth, len) -> returns boolean (should parent free curr?)
  curr.prefix_count <- curr.prefix_count - 1

  if depth == len then                        // Reached terminal node of word
      curr.word_count <- 0
      curr.is_end_of_word <- false
      return (not HasAnyChild(curr))          // Case 2: false if children exist; true if leaf

  idx <- word[depth] - 'a'
  should_free_child <- DeleteHelper(curr.children[idx], word, depth + 1, len)

  if should_free_child == true then
      free(curr.children[idx])
      curr.children[idx] <- NULL
      // Case 3 vs Case 4: Continue freeing upward ONLY if curr is not root,
      // not the end of another shorter word, and has no other remaining children!
      return (depth > 0 and not curr.is_end_of_word and not HasAnyChild(curr))

  return false
```

---

### 4.5 Complete C11 Implementation: Insertion, Search, Prefix Count, Autocomplete, and 4-Case Deletion

```c
/**
 * @file trie_operations.c
 * @brief Complete, textbook-grade C11 implementation of a 26-ary Trie (Prefix Tree)
 *        supporting Insertion, Exact Search, Prefix Search, Prefix Counting,
 *        Lexicographical Autocomplete, and 4-Case Recursive Postorder Deletion.
 *        Compile: gcc -std=c11 -Wall -Wextra trie_operations.c -o trie_ops
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <string.h>

#define ALPHABET_SIZE 26
#define MAX_WORD_LEN  128

/* ============================================================================
 * 1. TRIE NODE DEFINITION & INITIALIZATION
 * ============================================================================ */

typedef struct TrieNode {
    struct TrieNode *children[ALPHABET_SIZE];
    bool is_end_of_word;
    int prefix_count;   /* Number of words sharing the prefix up to this node */
    int word_count;     /* Multiplicity of exact word ending at this node     */
} TrieNode;

TrieNode *createTrieNode(void) {
    TrieNode *node = (TrieNode *)malloc(sizeof(TrieNode));
    if (!node) {
        perror("Failed to allocate TrieNode");
        exit(EXIT_FAILURE);
    }
    for (int i = 0; i < ALPHABET_SIZE; i++) {
        node->children[i] = NULL;
    }
    node->is_end_of_word = false;
    node->prefix_count = 0;
    node->word_count = 0;
    return node;
}

static inline int charToIndex(char ch) {
    if (ch >= 'a' && ch <= 'z') return ch - 'a';
    if (ch >= 'A' && ch <= 'Z') return ch - 'A';
    return -1;
}

static bool hasAnyChild(const TrieNode *node) {
    if (!node) return false;
    for (int i = 0; i < ALPHABET_SIZE; i++) {
        if (node->children[i] != NULL) {
            return true;
        }
    }
    return false;
}

/* ============================================================================
 * 2. INSERTION, EXACT SEARCH, AND PREFIX SEARCH
 * ============================================================================ */

void trieInsert(TrieNode *root, const char *word) {
    if (!root || !word) return;

    /* Validate all characters first */
    for (size_t i = 0; word[i] != '\0'; i++) {
        if (charToIndex(word[i]) == -1) {
            fprintf(stderr, "Error: Invalid character '%c' in \"%s\"\n", word[i], word);
            return;
        }
    }

    TrieNode *curr = root;
    curr->prefix_count++;

    for (size_t i = 0; word[i] != '\0'; i++) {
        int idx = charToIndex(word[i]);
        if (curr->children[idx] == NULL) {
            curr->children[idx] = createTrieNode();
        }
        curr = curr->children[idx];
        curr->prefix_count++;
    }

    curr->is_end_of_word = true;
    curr->word_count++;
}

bool trieSearch(const TrieNode *root, const char *word) {
    if (!root || !word) return false;

    const TrieNode *curr = root;
    for (size_t i = 0; word[i] != '\0'; i++) {
        int idx = charToIndex(word[i]);
        if (idx == -1 || curr->children[idx] == NULL) {
            return false;
        }
        curr = curr->children[idx];
    }
    /* CRITICAL: Must check is_end_of_word, not just curr != NULL! */
    return (curr != NULL && curr->is_end_of_word);
}

bool trieStartsWith(const TrieNode *root, const char *prefix) {
    if (!root || !prefix) return false;

    const TrieNode *curr = root;
    for (size_t i = 0; prefix[i] != '\0'; i++) {
        int idx = charToIndex(prefix[i]);
        if (idx == -1 || curr->children[idx] == NULL) {
            return false;
        }
        curr = curr->children[idx];
    }
    return (curr != NULL);
}

int trieCountPrefix(const TrieNode *root, const char *prefix) {
    if (!root || !prefix) return 0;

    const TrieNode *curr = root;
    for (size_t i = 0; prefix[i] != '\0'; i++) {
        int idx = charToIndex(prefix[i]);
        if (idx == -1 || curr->children[idx] == NULL) {
            return 0;
        }
        curr = curr->children[idx];
    }
    return curr->prefix_count;
}

/* ============================================================================
 * 3. LEXICOGRAPHICAL TRAVERSAL & AUTOCOMPLETE
 * ============================================================================ */

static void collectWordsDFS(const TrieNode *node, char buffer[], int depth) {
    if (!node) return;

    if (node->is_end_of_word) {
        buffer[depth] = '\0';
        printf("    - \"%s\" (word_count=%d, prefix_count=%d)\n",
               buffer, node->word_count, node->prefix_count);
    }

    for (int i = 0; i < ALPHABET_SIZE; i++) {
        if (node->children[i] != NULL) {
            buffer[depth] = (char)('a' + i);
            collectWordsDFS(node->children[i], buffer, depth + 1);
        }
    }
}

void trieAutocomplete(const TrieNode *root, const char *prefix) {
    if (!root || !prefix) return;

    const TrieNode *curr = root;
    char buffer[MAX_WORD_LEN];
    int depth = 0;

    for (size_t i = 0; prefix[i] != '\0'; i++) {
        int idx = charToIndex(prefix[i]);
        if (idx == -1 || curr->children[idx] == NULL) {
            printf("  Autocomplete for \"%s\": (no matching words)\n", prefix);
            return;
        }
        buffer[depth++] = (char)('a' + idx);
        curr = curr->children[idx];
    }

    printf("  Autocomplete for \"%s\" (%d matching word(s)):\n",
           prefix, curr->prefix_count);
    collectWordsDFS(curr, buffer, depth);
}

/* ============================================================================
 * 4. RECURSIVE POSTORDER DELETION (ALL 4 STRUCTURAL CASES)
 * ============================================================================ */

/*
 * Recursive helper that returns true iff 'curr' should be freed by its parent.
 * Precondition: 'word' has already been verified to exist in the Trie via trieSearch,
 * so prefix_count can be safely decremented along the path.
 */
static bool deleteHelper(TrieNode *curr, const char *word, int depth, int len) {
    if (!curr) return false;

    curr->prefix_count--;

    /* Base case: reached the terminal node of 'word' */
    if (depth == len) {
        curr->word_count = 0;
        curr->is_end_of_word = false;

        /* Case 2: If curr still has children, it is a prefix of a longer word -> DO NOT FREE!
         * Case 3/4: If curr has no children, tell parent to free this leaf node. */
        return !hasAnyChild(curr);
    }

    int idx = charToIndex(word[depth]);
    bool should_free_child = deleteHelper(curr->children[idx], word, depth + 1, len);

    if (should_free_child) {
        free(curr->children[idx]);
        curr->children[idx] = NULL;

        /* Propagate deletion upward ONLY IF:
         * 1. curr is NOT the root (depth > 0), AND
         * 2. curr is NOT the end of another shorter word (!curr->is_end_of_word), AND
         * 3. curr has no other remaining children (!hasAnyChild(curr)). */
        return (depth > 0 && !curr->is_end_of_word && !hasAnyChild(curr));
    }

    return false;
}

bool trieDelete(TrieNode *root, const char *word) {
    /* Case 1: Word is not present in the Trie -> do nothing */
    if (!trieSearch(root, word)) {
        return false;
    }

    int len = (int)strlen(word);
    deleteHelper(root, word, 0, len);
    return true;
}

void freeTrie(TrieNode *node) {
    if (!node) return;
    for (int i = 0; i < ALPHABET_SIZE; i++) {
        if (node->children[i] != NULL) {
            freeTrie(node->children[i]);
        }
    }
    free(node);
}

/* ============================================================================
 * 5. MAIN DRIVER DEMONSTRATION
 * ============================================================================ */

int main(void) {
    TrieNode *root = createTrieNode();

    const char *words[] = {"apple", "app", "apply", "apt", "bat", "bad", "cat"};
    size_t n = sizeof(words) / sizeof(words[0]);

    printf("=== 1. INSERTING %zu WORDS INTO TRIE ===\n  Inserted: ", n);
    for (size_t i = 0; i < n; i++) {
        trieInsert(root, words[i]);
        printf("\"%s\"%s", words[i], (i + 1 < n) ? ", " : "\n\n");
    }

    printf("=== 2. EXACT WORD SEARCH vs. PREFIX SEARCH ===\n");
    const char *queries[] = {"app", "apple", "ap", "apt", "bat", "ban", "cat"};
    for (size_t i = 0; i < sizeof(queries) / sizeof(queries[0]); i++) {
        const char *q = queries[i];
        printf("  Query %-7s -> ExactWord: %-5s | StartsWith: %-5s | PrefixCount: %d\n",
               q,
               trieSearch(root, q) ? "FOUND" : "NO",
               trieStartsWith(root, q) ? "YES" : "NO",
               trieCountPrefix(root, q));
    }
    printf("\n");

    printf("=== 3. AUTOCOMPLETE & LEXICOGRAPHICAL ENUMERATION ===\n");
    trieAutocomplete(root, "");
    trieAutocomplete(root, "ap");
    trieAutocomplete(root, "ba");
    printf("\n");

    printf("=== 4. TESTING ALL 4 DELETION CASES ===\n");

    /* Case 1: Delete a word not in the Trie ("ap" is only a prefix, "dog" is absent) */
    printf("[Case 1] Delete non-existent \"ap\":  %s\n",
           trieDelete(root, "ap") ? "DELETED" : "NOT FOUND (Unchanged)");
    printf("[Case 1] Delete non-existent \"dog\": %s\n",
           trieDelete(root, "dog") ? "DELETED" : "NOT FOUND (Unchanged)");

    /* Case 2: Delete "app" (which is a proper prefix of "apple" and "apply") */
    printf("[Case 2] Delete \"app\" (prefix of \"apple\", \"apply\"): %s\n",
           trieDelete(root, "app") ? "DELETED" : "NOT FOUND");
    printf("         Search(\"app\") = %s, Search(\"apple\") = %s, Search(\"apply\") = %s\n",
           trieSearch(root, "app") ? "FOUND" : "NO",
           trieSearch(root, "apple") ? "FOUND" : "NO",
           trieSearch(root, "apply") ? "FOUND" : "NO");

    /* Case 3: Delete "apple" (shares prefix "appl" with "apply") */
    printf("[Case 3] Delete \"apple\" (shares \"appl\" with \"apply\"): %s\n",
           trieDelete(root, "apple") ? "DELETED" : "NOT FOUND");
    printf("         Search(\"apple\") = %s, Search(\"apply\") = %s, Search(\"apt\") = %s\n",
           trieSearch(root, "apple") ? "FOUND" : "NO",
           trieSearch(root, "apply") ? "FOUND" : "NO",
           trieSearch(root, "apt") ? "FOUND" : "NO");

    /* Case 4: Delete "cat" (unique branch sharing no characters with any other word) */
    printf("[Case 4] Delete \"cat\" (independent branch): %s\n",
           trieDelete(root, "cat") ? "DELETED" : "NOT FOUND");
    printf("         Search(\"cat\") = %s, root->children['c'-'a'] == %s\n\n",
           trieSearch(root, "cat") ? "FOUND" : "NO",
           (root->children['c' - 'a'] == NULL) ? "NULL (Freed!)" : "NOT NULL");

    printf("=== 5. REMAINING WORDS IN TRIE (LEXICOGRAPHICAL ORDER) ===\n");
    trieAutocomplete(root, "");

    freeTrie(root);
    return 0;
}
```

#### Program Output:

```text
=== 1. INSERTING 7 WORDS INTO TRIE ===
  Inserted: "apple", "app", "apply", "apt", "bat", "bad", "cat"

=== 2. EXACT WORD SEARCH vs. PREFIX SEARCH ===
  Query app     -> ExactWord: FOUND | StartsWith: YES   | PrefixCount: 3
  Query apple   -> ExactWord: FOUND | StartsWith: YES   | PrefixCount: 1
  Query ap      -> ExactWord: NO    | StartsWith: YES   | PrefixCount: 4
  Query apt     -> ExactWord: FOUND | StartsWith: YES   | PrefixCount: 1
  Query bat     -> ExactWord: FOUND | StartsWith: YES   | PrefixCount: 1
  Query ban     -> ExactWord: NO    | StartsWith: NO    | PrefixCount: 0
  Query cat     -> ExactWord: FOUND | StartsWith: YES   | PrefixCount: 1

=== 3. AUTOCOMPLETE & LEXICOGRAPHICAL ENUMERATION ===
  Autocomplete for "" (7 matching word(s)):
    - "app" (word_count=1, prefix_count=3)
    - "apple" (word_count=1, prefix_count=1)
    - "apply" (word_count=1, prefix_count=1)
    - "apt" (word_count=1, prefix_count=1)
    - "bad" (word_count=1, prefix_count=1)
    - "bat" (word_count=1, prefix_count=1)
    - "cat" (word_count=1, prefix_count=1)
  Autocomplete for "ap" (4 matching word(s)):
    - "app" (word_count=1, prefix_count=3)
    - "apple" (word_count=1, prefix_count=1)
    - "apply" (word_count=1, prefix_count=1)
    - "apt" (word_count=1, prefix_count=1)
  Autocomplete for "ba" (2 matching word(s)):
    - "bad" (word_count=1, prefix_count=1)
    - "bat" (word_count=1, prefix_count=1)

=== 4. TESTING ALL 4 DELETION CASES ===
[Case 1] Delete non-existent "ap":  NOT FOUND (Unchanged)
[Case 1] Delete non-existent "dog": NOT FOUND (Unchanged)
[Case 2] Delete "app" (prefix of "apple", "apply"): DELETED
         Search("app") = NO, Search("apple") = FOUND, Search("apply") = FOUND
[Case 3] Delete "apple" (shares "appl" with "apply"): DELETED
         Search("apple") = NO, Search("apply") = FOUND, Search("apt") = FOUND
[Case 4] Delete "cat" (independent branch): DELETED
         Search("cat") = NO, root->children['c'-'a'] == NULL (Freed!)

=== 5. REMAINING WORDS IN TRIE (LEXICOGRAPHICAL ORDER) ===
  Autocomplete for "" (4 matching word(s)):
    - "apply" (word_count=1, prefix_count=1)
    - "apt" (word_count=1, prefix_count=1)
    - "bad" (word_count=1, prefix_count=1)
    - "bat" (word_count=1, prefix_count=1)
```

---

## 5. Comparison Tables and Complexity Summary

### 5.1 Time and Space Complexity Table for Trie Operations

Let $m = |W|$ be the length of the target word/prefix being operated on, $n$ be the total number of words stored in the Trie, $|\Sigma|$ be the alphabet size (`26` for `'a'..'z'`), $N_{\text{nodes}} \le 1 + \sum_{i=1}^{n} |W_i|$ be the total number of Trie nodes, and $K$ be the total number of nodes in the subtree of `prefix` during autocomplete.

| Operation | Best-Case Time | Average-Case Time | Worst-Case Time | Auxiliary Space |
| :--- | :---: | :---: | :---: | :---: |
| **Insertion (`trieInsert`)** | $\Theta(m)$ | $\Theta(m)$ | **$O(m)$** | $O(m \cdot |\Sigma|)$ new nodes max |
| **Exact Word Search (`trieSearch`)** | $O(1)$ (1st char miss) | $O(m)$ | **$O(m)$** | $O(1)$ |
| **Prefix Existence (`trieStartsWith`)** | $O(1)$ (1st char miss) | $O(m)$ | **$O(m)$** | $O(1)$ |
| **Prefix Count (`trieCountPrefix`)** | $O(1)$ (1st char miss) | $O(m)$ | **$O(m)$** | $O(1)$ |
| **Deletion (`trieDelete`)** | $O(1)$ (if not found) | $O(m)$ | **$O(m \cdot |\Sigma|)$** | $O(m)$ recursion stack |
| **Autocomplete (`trieAutocomplete`)** | $O(1)$ (prefix miss) | $O(m + K \cdot |\Sigma|)$ | **$O(m + K \cdot |\Sigma|)$** | $O(H)$ recursion stack |
| **Total Space of Trie Structure** | — | — | **$O(|\Sigma| \cdot N_{\text{nodes}})$** | $\le O\bigl(|\Sigma| \cdot \sum_{i=1}^n |W_i|\bigr)$ |

*(Note: During deletion, `hasAnyChild(curr)` scans `ALPHABET_SIZE` slots in $O(|\Sigma|) = O(1)$ constant time for fixed $|\Sigma| = 26$. If each node also maintains an integer `active_children_count`, `hasAnyChild` becomes strictly $O(1)$ independent of $|\Sigma|$).*

### 5.2 Data Structure Comparison: Trie vs. Balanced BST vs. Hash Table for String Keys

Suppose we store $n$ strings, where the query string has length $m$ and the average/maximum string length is $M$:

| Feature / Operation | Trie (Prefix Tree) | Balanced BST (AVL / Red-Black) | Hash Table (Chaining / Open Addressing) |
| :--- | :--- | :--- | :--- |
| **Exact Search (`search(W)`)** | **$O(m)$ worst-case** | $O(m \log n)$ worst-case | $O(m)$ average, $O(n \cdot m)$ worst-case |
| **Insertion (`insert(W)`)** | **$O(m)$ worst-case** | $O(m \log n)$ worst-case | $O(m)$ average, $O(n \cdot m)$ worst-case (rehash) |
| **Deletion (`delete(W)`)** | **$O(m)$ worst-case** | $O(m \log n)$ worst-case | $O(m)$ average, $O(n \cdot m)$ worst-case |
| **Prefix Search (`startsWith(P)`)** | **$O(|P|)$ worst-case** | $O(|P| \log n)$ | **$O(n \cdot M)$** (requires full table scan!) |
| **Autocomplete (All words with prefix $P$)** | **$O(|P| + K)$** (optimal subtree walk) | $O(|P| \log n + K)$ | **$O(n \cdot M + K \log K)$** (scan all + sort!) |
| **Alphabetical Iteration of All Keys** | **$O(N_{\text{nodes}} \cdot |\Sigma|)$** (no sort needed!) | $O(n)$ inorder traversal | $O(n \log n \cdot M)$ (must extract & sort) |
| **Hash Collisions Possible?** | **Zero** (deterministic character edges) | Zero | Yes (requires collision resolution) |
| **Memory Overhead** | High ($|\Sigma|$ pointers per node, unless compressed) | Low ($2$ child pointers + string per node) | Low-Moderate (bucket array + load factor slack) |

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Pitfall 1: Returning `true` in Exact Search Without Checking `is_end_of_word`

```c
/* BUG: Returning true just because the path of characters exists in the Trie */
bool trieSearchBuggy(const TrieNode *root, const char *word) {
    const TrieNode *curr = root;
    for (size_t i = 0; word[i] != '\0'; i++) {
        int idx = word[i] - 'a';
        if (curr->children[idx] == NULL) return false;
        curr = curr->children[idx];
    }
    return true; /* BUG! Returns true for "app" even when only "apple" was inserted! */
}

/* FIX: Always check curr->is_end_of_word at the end of the character loop */
return (curr != NULL && curr->is_end_of_word);
```

### 6.2 Pitfall 2: Freeing Shared Prefix Nodes During Deletion (Breaking Longer or Shorter Words)

```c
/* BUG: Blindly freeing nodes along the word's path without checking children or is_end_of_word */
/* If "apple" and "apply" are in the Trie and we delete "apple", freeing (a), (ap), (app), (appl)
 * destroys the shared prefix path and leaves dangling pointers for "apply"! */

/* FIX: Use postorder recursion and ONLY free a node if:
 *   1. depth > 0 (never free root)
 *   2. !curr->is_end_of_word (not the terminal node of a shorter word like "app")
 *   3. !hasAnyChild(curr) (no other sibling branch like 'y' in "apply" depends on curr) */
return (depth > 0 && !curr->is_end_of_word && !hasAnyChild(curr));
```

### 6.3 Pitfall 3: Out-of-Bounds Array Indexing on Uppercase or Non-Alphabetic Characters

In C, if `word[i]` is uppercase `'A'` (ASCII 65), space `' '` (ASCII 32), or hyphen `'-'` (ASCII 45), then `int idx = word[i] - 'a'` evaluates to a **negative integer** (`-32`, `-65`, `-52`), causing `curr->children[idx]` to read/write out-of-bounds memory before the struct!

```c
/* BUG: Unchecked subtraction assumes every input character is strictly 'a'..'z' */
int idx = word[i] - 'a';
curr = curr->children[idx]; /* Undefined Behavior (segfault/heap corruption) if word[i] == 'A'! */

/* FIX: Always normalize and bounds-check the character index via a helper function */
int idx = charToIndex(word[i]);
if (idx < 0 || idx >= ALPHABET_SIZE) {
    return false; /* Safely reject out-of-alphabet characters */
}
```

### 6.4 Pitfall 4: Accidentally Freeing the Root Node When Deleting the Last Word

When the very last word in the Trie is deleted (e.g., deleting `"cat"` when `"cat"` is the only word in the Trie), after `root->children['c' - 'a']` is freed and set to `NULL`, `root` itself now has `!root->is_end_of_word && !hasAnyChild(root)`. If `deleteHelper` returns `true` at `depth == 0` and the caller frees `root`, subsequent insertions will dereference a freed `root` pointer! Always guard upward propagation with `depth > 0` so `root` remains allocated until `freeTrie(root)` is explicitly called.

### 6.5 Boundary Conditions Checklist

- [ ] **Empty String (`""`):** Does `trieInsert(root, "")` mark `root->is_end_of_word = true` cleanly, and does `trieSearch(root, "")` return `root->is_end_of_word`?
- [ ] **Duplicate Word Insertion:** If `"apple"` is inserted twice, does `prefix_count` increment appropriately or is duplicate insertion guarded so `trieDelete` doesn't desynchronize `prefix_count`?
- [ ] **Prefix vs. Word Deletion (Case 1):** If `"apple"` is in the Trie and the user calls `trieDelete(root, "ap")`, does `trieDelete` verify `trieSearch(root, "ap")` *first* so it doesn't corrupt `prefix_count` along `"ap"`?
- [ ] **Single-Character Words (`"a"`):** Do insertion, search, and deletion work properly when `len == 1`?

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways and Core Invariants

1. **$O(m)$ Time Independent of $n$:** A Trie indexes by one character of the key per level, achieving worst-case **$O(m)$** time for insertion, exact search, prefix search, and deletion of a length-$m$ string regardless of how many million keys $n$ are stored.
2. **Implicit Key Storage & Canonical Shape:** Nodes do not store strings; the path from the root defines the prefix. Unlike BSTs, a Trie's structure is completely independent of the order in which words are inserted.
3. **Exact Search vs. Prefix Search:** `trieStartsWith(P)` checks `curr != NULL`, whereas `trieSearch(W)` **must** check `curr != NULL && curr->is_end_of_word == true`.
4. **Four Cases of Trie Deletion:**
   - **Case 1 (Not found):** Do nothing.
   - **Case 2 (`W` is a prefix of a longer word):** Set `is_end_of_word = false`; free **zero** nodes.
   - **Case 3 (`W` shares a prefix with another word):** Free nodes bottom-up from the leaf of `W`, stopping at the lowest ancestor with `is_end_of_word == true` or another non-NULL child.
   - **Case 4 (`W` shares no characters with any word):** Free all nodes of `W` up to `root->children[idx] = NULL` (keeping `root` intact).

### 7.2 Self-Test Revision Questions

1. **Trace Question:** How many total `TrieNode` instances (including `root`) exist in a Trie after inserting the 5 words: `"car"`, `"card"`, `"care"`, `"careful"`, `"cat"`?
   *(Answer: `root` (1) + `'c'` (1) + `'a'` (1) + `'r'` (1, for `"car"`) + `'d'` (1, for `"card"`) + `'e'` (1, for `"care"`) + `'f'->'u'->'l'` (3, for `"careful"`) + `'t'` (1, for `"cat"`) = **$10$ nodes total**, compared to $1 + 3 + 4 + 4 + 7 + 3 = 22$ characters if stored without prefix sharing).*
2. **Conceptual Question:** Why can a Trie enumerate all stored words in alphabetical order without calling a sorting algorithm, whereas a Hash Table cannot?
   *(Answer: Because at every `TrieNode`, child pointers `children[0 .. 25]` are ordered by character `'a' .. 'z'`. A preorder traversal visiting `is_end_of_word` first and then recursing on `children[0]` through `children[25]` visits shorter prefixes before extensions and `'a'` branches before `'b'` branches—which is the exact definition of lexicographical order).*
3. **Limitation Question:** A standard Trie excels at `startsWith(prefix)` queries in $O(|P|)$ time. Why is a standard Trie **poor** at answering `containsSubstring(P)` (checking whether $P$ appears *anywhere in the middle* of a large text $S$)—and what specialized Trie variant solves that problem?
   *(Answer: A standard Trie only indexes prefixes starting at index `0` of each inserted word; a substring buried inside $S[i..j]$ is not anchored at the root. Indexing all suffixes of $S$ in a **Suffix Trie** or **Compressed Suffix Tree** makes every substring of $S$ a prefix of some suffix, solving arbitrary substring queries in $O(|P|)$ time—covered in Topic 3!).*

### 7.3 Navigation Links

- **Previous Topic:** [01-Graph-Connectivity-Components-and-Paths.md](./01-Graph-Connectivity-Components-and-Paths.md) — Graph Connectivity, Connected Components, and Path Finding
- **Current File:** `02-Trie-Trees.md` (Unit 4 — Topic 2)
- **Next Topic:** [03-Suffix-Tries-and-Suffix-Trees.md](./03-Suffix-Tries-and-Suffix-Trees.md) — Suffix Tries and Compressed Suffix Trees
