# Unit 4 — Topic 3: Suffix Tries and Compressed Suffix Trees

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [From Multi-Word Prefix Indexing to Full-Text Substring Indexing](#11-from-multi-word-prefix-indexing-to-full-text-substring-indexing)
   - 1.2 [Why Do We Need Suffix Tries and Suffix Trees? The Problem They Solve](#12-why-do-we-need-suffix-tries-and-suffix-trees-the-problem-they-solve)
   - 1.3 [Relationship to Prior Concepts: Standard Trie vs. Suffix Trie vs. Suffix Tree](#13-relationship-to-prior-concepts-standard-trie-vs-suffix-trie-vs-suffix-tree)
   - 1.4 [Real-World Bioinformatics, Compression, and Text-Indexing Applications](#14-real-world-bioinformatics-compression-and-text-indexing-applications)
   - 1.5 [Learning Objectives](#15-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Prefixes, Suffixes, Substrings, and the Fundamental Substring-Suffix Identity](#21-prefixes-suffixes-substrings-and-the-fundamental-substring-suffix-identity)
   - 2.2 [The Terminal Sentinel Marker (`$`): Why Prefix-Freeness Is Essential](#22-the-terminal-sentinel-marker--why-prefix-freeness-is-essential)
   - 2.3 [Core Terminology Table](#23-core-terminology-table)
   - 2.4 [Mathematical Properties, Node Bounds, and Space Invariants](#24-mathematical-properties-node-bounds-and-space-invariants)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Uncompressed `SuffixTrieNode` Representation in C](#31-uncompressed-suffixtrienode-representation-in-c)
   - 3.2 [Compressed `SuffixTreeNode` Representation Using `[start, end]` Index Pairs](#32-compressed-suffixtreenode-representation-using-start-end-index-pairs)
   - 3.3 [ASCII Memory Layout Diagram: String-Slice Edges vs. Character Chains](#33-ascii-memory-layout-diagram-string-slice-edges-vs-character-chains)
   - 3.4 [Why Storing Explicit Edge Substrings Still Wastes $\Theta(m^2)$ Memory](#34-why-storing-explicit-edge-substrings-still-wastes-thetam2-memory)
4. [Operations and Algorithms: Uncompressed Suffix Trie](#4-operations-and-algorithms-uncompressed-suffix-trie)
   - 4.1 [Suffix Trie Construction Algorithm ($\Theta(m^2)$ Time and up to $\Theta(m^2)$ Space)](#41-suffix-trie-construction-algorithm-thetam2-time-and-up-to-thetam2-space)
   - 4.2 [Worked Example 1: Step-by-Step Suffix Trie Construction for $S = \text{"aba\$"}$](#42-worked-example-1-step-by-step-suffix-trie-construction-for-s--textaba)
   - 4.3 [Worked Example 2: Full Suffix Trie for $S = \text{"banana\$"}$ (23 Nodes) and $\Theta(m^2)$ Worst-Case Proof](#43-worked-example-2-full-suffix-trie-for-s--textbanana-23-nodes-and-thetam2-worst-case-proof)
   - 4.4 [Substring Search, Occurrence Counting, and Longest Repeated Substring on a Suffix Trie](#44-substring-search-occurrence-counting-and-longest-repeated-substring-on-a-suffix-trie)
5. [Operations and Algorithms: Compressed Suffix Tree](#5-operations-and-algorithms-compressed-suffix-tree)
   - 5.1 [Path Compression and the $\le 2m - 1$ Node Bound Proof](#51-path-compression-and-the-le-2m---1-node-bound-proof)
   - 5.2 [Worked Example: Compressing the Suffix Trie of `"banana$"` into Its 11-Node Suffix Tree](#52-worked-example-compressing-the-suffix-trie-of-banana-into-its-11-node-suffix-tree)
   - 5.3 [Step-by-Step Suffix Tree Construction with Edge Splitting](#53-step-by-step-suffix-tree-construction-with-edge-splitting)
   - 5.4 [Honest Complexity Distinction: $O(m^2)$ Educational Construction vs. $O(m)$ Ukkonen's Algorithm](#54-honest-complexity-distinction-om2-educational-construction-vs-om-ukkonens-algorithm)
   - 5.5 [Complete C11 Implementation: Suffix Trie and Compressed Suffix Tree on `"banana$"`](#55-complete-c11-implementation-suffix-trie-and-compressed-suffix-tree-on-banana)
6. [Comparison Tables and Complexity Summary](#6-comparison-tables-and-complexity-summary)
   - 6.1 [Time and Space Complexity Table](#61-time-and-space-complexity-table)
   - 6.2 [Structural Comparison: Standard Trie vs. Uncompressed Suffix Trie vs. Compressed Suffix Tree](#62-structural-comparison-standard-trie-vs-uncompressed-suffix-trie-vs-compressed-suffix-tree)
7. [Common Errors, Pitfalls, and Edge Cases](#7-common-errors-pitfalls-and-edge-cases)
   - 7.1 [Pitfall 1: Omitting the Terminal Sentinel `$` (Losing Suffixes Inside Internal Nodes)](#71-pitfall-1-omitting-the-terminal-sentinel--losing-suffixes-inside-internal-nodes)
   - 7.2 [Pitfall 2: Copying Substring Characters Onto Compressed Edges Instead of `[start, end]` Indices](#72-pitfall-2-copying-substring-characters-onto-compressed-edges-instead-of-start-end-indices)
   - 7.3 [Pitfall 3: Claiming an $O(m^2)$ Edge-Splitting Implementation Runs in $O(m)$ Time](#73-pitfall-3-claiming-an-om2-edge-splitting-implementation-runs-in-om-time)
   - 7.4 [Boundary Conditions Checklist](#74-boundary-conditions-checklist)
8. [Summary and Revision Checklist](#8-summary-and-revision-checklist)
   - 8.1 [Key Takeaways and Core Invariants](#81-key-takeaways-and-core-invariants)
   - 8.2 [Self-Test Revision Questions](#82-self-test-revision-questions)
   - 8.3 [Navigation Links](#83-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 From Multi-Word Prefix Indexing to Full-Text Substring Indexing

In Topic 2, we saw that a **Standard Trie** indexes a dictionary of $n$ discrete words $\mathcal{D} = \{W_1, \dots, W_n\}$ so that we can check whether a query string $P$ is a **word** or a **prefix of a word** in $O(|P|)$ time.

However, many critical problems in computer science do not involve a dictionary of short, separate words—instead, they involve a **single massive text string $S$ of length $m$** (such as the 3-billion-base-pair human genome, the complete works of Shakespeare, or a multi-megabyte source code repository) against which thousands of arbitrary **substring queries** $P$ arrive continuously:
- *"Does the pattern $P = \text{"ana"}$ appear **anywhere inside** $S = \text{"banana"}$?"*
- *"How many times does $P$ occur in $S$, and at what exact character indices?"*
- *"What is the **Longest Repeated Substring** inside $S$?"*

### 1.2 Why Do We Need Suffix Tries and Suffix Trees? The Problem They Solve

1. **The Limitation of Online Pattern Matching (KMP / Boyer-Moore / Rabin-Karp):**
   Classical string-matching algorithms preprocess the **pattern $P$** in $O(|P|)$ time and then scan the **entire text $S$** in $O(m)$ time per query. If a genomic database $S$ has length $m = 3 \times 10^9$ and we run $Q = 100{,}000$ short queries of length $|P| = 20$, scanning $S$ repeatedly takes $O(Q \cdot m) \approx 3 \times 10^{14}$ operations!
2. **The Suffix Indexing Breakthrough ($O(|P|)$ Query Time Independent of Text Length $m$!):**
   By preprocessing the **text $S$** once into a **Suffix Trie** or **Suffix Tree**—which indexes **all $m$ suffixes of $S$**—we can answer whether *any* substring $P$ exists in $S$ in strictly **$O(|P|)$ time**, and report all $\text{occ}$ starting positions in **$O(|P| + \text{occ})$ time**, **without scanning $S$ at all**!

```
   ONLINE PATTERN MATCHING (KMP / Boyer-Moore)            FULL-TEXT INDEXING (Suffix Trie / Suffix Tree)
  +--------------------------------------------+         +----------------------------------------------+
  | Preprocess Pattern P: O(|P|)               |         | Preprocess Text S ONCE:                      |
  | Scan entire Text S:   O(m) per query!      |         |   - Suffix Trie: O(m^2) time & space         |
  |                                            |         |   - Suffix Tree: O(m) space (O(m) Ukkonen)   |
  | Slow when Text S is huge & queried often!  |         | Every Query P runs in O(|P| + occ) time!     |
  +--------------------------------------------+         +----------------------------------------------+
```

### 1.3 Relationship to Prior Concepts: Standard Trie vs. Suffix Trie vs. Suffix Tree

```
  1. STANDARD TRIE                 2. UNCOMPRESSED SUFFIX TRIE         3. COMPRESSED SUFFIX TREE
  Indexes a set of n words         Indexes all m suffixes of S         Compresses single-child chains
  {W_1, W_2, ..., W_n}             {S[0..m-1], ..., S[m-1..m-1]}       into [start, end] substring edges

         (root)                               (root)                                (root)
          'a'|                                 'b'|                               [0..6]| "banana$"
            (a)                                  (b)                                  [Leaf 0]
          'p'|                                 'a'|
          [app]*                                 (ba)                  Eliminates all non-branching
                                               'n'|                    internal nodes! Guaranteed
  Answers prefix queries               ... (up to O(m^2) nodes!)       <= 2m - 1 nodes and O(m) space!
```

### 1.4 Real-World Bioinformatics, Compression, and Text-Indexing Applications

1. **Bioinformatics & Whole-Genome Read Alignment (MUMmer / Bowtie precursors):** Finding Maximal Unique Matches (MUMs) and aligning short DNA sequencing reads against a reference genome in time proportional to the read length $|P|$.
2. **Data Compression (LZ77 / gzip / Deflate & Burrows-Wheeler Transform):** LZ77 compression replaces repeated substrings with `(distance, length)` back-references by maintaining a sliding-window Suffix Tree; sorting the leaves of a Suffix Tree directly yields the **Suffix Array** used in BWT (`bzip2`).
3. **Plagiarism Detection & Code Clone Analysis:** Building a Generalized Suffix Tree over multiple documents to find the **Longest Common Substring** across files in linear time.

### 1.5 Learning Objectives

By the end of this topic, you will be able to:
- State and prove the **Fundamental Substring-Suffix Identity** (*every substring of $S$ is a prefix of some suffix of $S$*).
- Explain why appending the unique terminal marker `$` ($\notin \Sigma$) guarantees that every suffix terminates at a **distinct leaf node**.
- Construct an **Uncompressed Suffix Trie** for strings such as `"aba$"` and `"banana$"`, trace substring queries, and prove why its worst-case space is $\Theta(m^2)$ on $S = \text{"a}^k\text{b}^k\text{\$}"$.
- Apply **Path Compression** and `[start, end]` index pairs to construct a **Compressed Suffix Tree**, prove that it has **exactly $m$ leaves and at most $2m - 1$ total nodes**, and clearly distinguish **$O(m^2)$ educational edge-splitting construction** from **$O(m)$ Ukkonen's algorithm**.

---

## 2. Definitions and Terminology

### 2.1 Prefixes, Suffixes, Substrings, and the Fundamental Substring-Suffix Identity

Let $S = S[0 \dots m-1]$ be a string of length $m$ over an alphabet $\Sigma$.

1. **Prefix of $S$:** Any contiguous slice $S[0 \dots j]$ anchored at index $0$ ($0 \le j < m$), plus the empty string $\varepsilon$.
2. **Suffix of $S$ ($\text{Suffix}(i)$):** Any contiguous slice $S[i \dots m-1]$ anchored at the end of $S$ ($0 \le i < m$). The integer $i$ is called the **suffix index** (or starting position) of that suffix.
3. **Substring of $S$:** Any contiguous slice $S[i \dots j]$ starting at index $i$ and ending at index $j$ ($0 \le i \le j < m$).

> [!IMPORTANT]
> **The Fundamental Substring-Suffix Identity:**
> Every non-empty substring $P = S[i \dots j]$ of $S$ is a **prefix** of the $i$-th suffix $\text{Suffix}(i) = S[i \dots m-1]$!
>
> $$S[i \dots j] \text{ occurs in } S \text{ at index } i \iff S[i \dots j] \text{ is a prefix of } \text{Suffix}(i) = S[i \dots m-1]$$

```
  Text S = "banana$" (m = 7)
  Index:     0   1   2   3   4   5   6
  Char:    [ b | a | n | a | n | a | $ ]
                 \_______/
                 Substring S[1..3] = "ana"
                 |
                 v
  Suffix(1) =   [ a | n | a | n | a | $ ]  ("anana$")
                  \_______/
                  "ana" is a 3-character PREFIX of Suffix(1)!
```

Because **a Trie is a prefix-search engine**, inserting all $m$ suffixes $\text{Suffix}(0), \text{Suffix}(1), \dots, \text{Suffix}(m-1)$ into a Trie means that **every possible substring of $S$ is now a prefix of at least one key in the Trie**—and can therefore be found by walking from the root in $O(|P|)$ time!

### 2.2 The Terminal Sentinel Marker (`$`): Why Prefix-Freeness Is Essential

Suppose we build a Suffix Trie for $S_0 = \text{"banana"}$ **without** a terminal symbol. Its 6 suffixes are:
- $\text{Suffix}(0) = \text{"banana"}$
- $\text{Suffix}(1) = \text{"anana"}$
- $\text{Suffix}(2) = \text{"nana"}$
- $\text{Suffix}(3) = \text{"ana"}$ $\longleftarrow$ **Proper prefix of $\text{Suffix}(1) = \text{"anana"}$!**
- $\text{Suffix}(4) = \text{"na"}$ $\longleftarrow$ **Proper prefix of $\text{Suffix}(2) = \text{"nana"}$!**
- $\text{Suffix}(5) = \text{"a"}$ $\longleftarrow$ **Proper prefix of $\text{Suffix}(3) = \text{"ana"}$ and $\text{Suffix}(1) = \text{"anana"}$!**

If a suffix $\text{Suffix}(i)$ is a proper prefix of another suffix $\text{Suffix}(j)$, then $\text{Suffix}(i)$ ends at an **internal node** rather than a **leaf node**. In a Compressed Suffix Tree, internal nodes without branching children get merged into edges, which would swallow $\text{Suffix}(3)$, $\text{Suffix}(4)$, and $\text{Suffix}(5)$ completely!

- **The Solution — Appending the Terminal Sentinel `$` ($\notin \Sigma$):**
  We append a special terminal character `$` that does **not** appear anywhere else in $\Sigma$ (and is defined to be lexicographically smaller than every character in $\Sigma$: $\text{'\$'} < \text{'a'} < \text{'b'} < \dots < \text{'z'}$).
  Now, for $S = \text{"banana\$"}$ ($m = 7$), every suffix ends with `$` at index $m-1 = 6$, and `$` appears nowhere else. Therefore, **no suffix can ever be a proper prefix of another suffix** (as soon as the shorter suffix hits `$`, it branches off along edge `'$'` to its own private leaf!).

```
  WITHOUT '$' (S = "aba"):                    WITH '$' (S = "aba$", m = 4):
  Suffix 2 ("a") is trapped inside            Every suffix 0, 1, 2, 3 branches on '$'
  the path of Suffix 0 ("aba")!               to its own distinct LEAF node!

           (root)                                        (root)
          'a'/ \'b'                                  '$'/  'a'|   \'b'
           [a] (b)                                   [L:3]   (a)  (b)
          'b'|  |'a'                                     '$'/ |'b' |'a'
          (ab) [ba]                                      [L:2](ab)(ba)
          'a'|                                                |'a' |'$'
          [aba]                                             (aba) [L:1]
                                                             |'$'
                                                            [L:0]
```

### 2.3 Core Terminology Table

| Term | Formal Definition | Role in Suffix Indexing |
| :--- | :--- | :--- |
| **Suffix $\text{Suffix}(i)$** | Substring $S[i \dots m-1]$ starting at index $i$ | Its leaf stores `suffix_index = i` |
| **Terminal Marker (`$`)** | Special symbol $\$ \notin \Sigma$ appended at $S[m-1]$ | Ensures prefix-freeness: all $m$ suffixes end at $m$ distinct leaves |
| **Suffix Trie** | Standard Trie containing all $m$ suffixes of $S$ | Answers substring queries in $O(|P|)$ time, but uses up to $\Theta(m^2)$ nodes |
| **Path Compression** | Merging each maximal non-branching chain of nodes into 1 edge | Reduces total node count from $\Theta(m^2)$ to $\le 2m - 1$ |
| **Suffix Tree** | Path-compressed Suffix Trie with `[start, end]` edge labels | Achieves **$O(m)$ space** and $O(|P| + \text{occ})$ query time |
| **String Depth of Node $u$** | Total number of characters spelled along the path $r \leadsto u$ | Used to identify the Longest Repeated Substring (deepest internal node) |

### 2.4 Mathematical Properties, Node Bounds, and Space Invariants

Let $S$ be a string of length $m$ ending with the unique terminal marker `$`.

1. **Number of Distinct Non-Empty Suffixes:**
   Exactly $m$ suffixes: $\text{Suffix}(0), \text{Suffix}(1), \dots, \text{Suffix}(m-1)$, with lengths $m, m-1, \dots, 1$.

2. **Number of Distinct Non-Empty Substrings of $S_0$ (of length $n = m - 1$ without `$`):**
   At most $\binom{n + 1}{2} = \frac{n(n + 1)}{2} = \Theta(m^2)$ distinct substrings. In an **Uncompressed Suffix Trie**, every distinct substring of $S$ corresponds to a distinct node, so:
   $$m + 1 \le N_{\text{nodes}}(\text{Suffix Trie}) \le \frac{m(m + 1)}{2} + 1 = \Theta(m^2)$$

3. **Exact Leaf and Internal Node Bounds in a Compressed Suffix Tree:**
   - **Number of Leaves ($L$):** Because of `$`, every suffix $\text{Suffix}(i)$ ($0 \le i < m$) terminates at a unique leaf node, and every leaf node represents a suffix. Thus:
     $$L = m$$
   - **Number of Internal Nodes ($I$):** By path compression, no internal node (not even the root when $m \ge 2$, since `$` alone branches at the root!) has degree $1$; every internal node has **at least $2$ children** ($\text{out-degree}(u) \ge 2$). In any rooted tree where every internal node has $\ge 2$ children:
     $$I \le L - 1 = m - 1$$
     *(Equality $I = m - 1$ holds when the Suffix Tree is strictly binary; when some internal nodes have $\ge 3$ children, $I < m - 1$).*
   - **Total Nodes ($N_{\text{total}}$) and Total Edges ($E_{\text{total}}$):**
     $$N_{\text{total}} = L + I \le m + (m - 1) = \mathbf{2m - 1}$$
     $$E_{\text{total}} = N_{\text{total}} - 1 \le \mathbf{2m - 2}$$

---

## 3. Representation and Memory Layout in C

### 3.1 Uncompressed `SuffixTrieNode` Representation in C

For lowercase letters `'a'..'z'` plus the terminal marker `'$'`, our alphabet has size `SUFFIX_ALPHA_SIZE = 27` (index `0` for `'$'`, indices `1..26` for `'a'..'z'`):

```c
#define SUFFIX_ALPHA_SIZE 27

typedef struct SuffixTrieNode {
    struct SuffixTrieNode *children[SUFFIX_ALPHA_SIZE]; /* '$' at [0], 'a'..'z' at [1..26] */
    int suffix_index; /* -1 for internal nodes; 0 .. m-1 for leaf nodes */
} SuffixTrieNode;
```

### 3.2 Compressed `SuffixTreeNode` Representation Using `[start, end]` Index Pairs

In a Compressed Suffix Tree, each edge $(u, v)$ is labeled with a multi-character substring $S[\text{start} \dots \text{end}]$ of the original text $S$. Since every tree node $v \neq \text{root}$ has **exactly one incoming edge** from its parent $u$, we store the incoming edge's `[start, end]` indices directly inside the child node `v`!

```c
typedef struct SuffixTreeNode {
    struct SuffixTreeNode *children[SUFFIX_ALPHA_SIZE]; /* Indexed by FIRST char S[start] */
    int start;        /* Start index of incoming edge label in S (-1 for root) */
    int end;          /* End index (inclusive) of incoming edge label in S     */
    int suffix_index; /* -1 for internal branching nodes; 0 .. m-1 for leaves  */
} SuffixTreeNode;
```

- **How `children[]` Indexing Works on Compressed Edges:**
  Because no two edges leaving the same parent $u$ can begin with the same first character, the child node $v$ whose incoming edge spells $S[\text{start} \dots \text{end}]$ is stored at slot `u->children[suffixCharToIndex(S[v->start])]`.
- **Edge Length in $O(1)$ Time:**
  $$\text{edgeLength}(v) = v\text{->end} - v\text{->start} + 1$$

### 3.3 ASCII Memory Layout Diagram: String-Slice Edges vs. Character Chains

```
  TEXT ARRAY IN MEMORY:   text[] =  [ 'b' | 'a' | 'n' | 'a' | 'n' | 'a' | '$' | '\0' ]
                          Index:       0     1     2     3     4     5     6

  UNCOMPRESSED SUFFIX TRIE CHAIN FOR "banana$":
  (root) --'b'--> (n1) --'a'--> (n2) --'n'--> (n3) --'a'--> (n4) --'n'--> (n5) --'a'--> (n6) --'$'--> [Leaf 0]
  Requires 7 separate heap allocations along a single non-branching path!

  COMPRESSED SUFFIX TREE EDGE FOR "banana$":
  (root) ---- children[idx('b')] ----> +-------------------------------+
                                       | start        = 0              |  References text[0..6] ("banana$")
                                       | end          = 6              |  in O(1) space (just two ints)!
                                       | suffix_index = 0   (Leaf)     |
                                       | children[]   = { NULL ... }   |
                                       +-------------------------------+
```

### 3.4 Why Storing Explicit Edge Substrings Still Wastes $\Theta(m^2)$ Memory

Suppose a programmer compresses all non-branching chains into single edges ($\le 2m - 1$ nodes), **but** stores a dynamically allocated `char *label` copy of the substring inside every edge.
- On a string where all $m$ suffixes diverge right at the root (or after $O(1)$ characters), the $m$ leaf edges have lengths $m, m-1, m-2, \dots, 1$.
- The sum of the string lengths stored across those $m$ leaf edges is $\sum_{i=1}^{m} i = \frac{m(m+1)}{2} = \Theta(m^2)$ characters!
- **Therefore, achieving true $O(m)$ space for a Suffix Tree requires BOTH:**
  1. **Path Compression** (reducing node count to $\le 2m - 1$), **AND**
  2. **Index-Pair Edge Labels `[start, end]`** into the shared text buffer `S` ($O(1)$ integers per edge)!

---

## 4. Operations and Algorithms: Uncompressed Suffix Trie

### 4.1 Suffix Trie Construction Algorithm ($\Theta(m^2)$ Time and up to $\Theta(m^2)$ Space)

To build an Uncompressed Suffix Trie for a string $S[0 \dots m-1]$ (ending with `$`):
1. Allocate an empty `root` node (`suffix_index = -1`).
2. Loop $i$ from $0$ to $m - 1$ (iterating over all $m$ suffixes $\text{Suffix}(i) = S[i \dots m-1]$):
   - Set `curr = root`.
   - Loop $j$ from $i$ to $m - 1$:
     - Compute `idx = suffixCharToIndex(S[j])`.
     - If `curr->children[idx] == NULL`, allocate a new `SuffixTrieNode`.
     - Advance `curr = curr->children[idx]`.
   - At the end of $\text{Suffix}(i)$ (after the `'$'` transition at $j = m - 1$), mark the leaf node with `curr->suffix_index = i`.

---

### 4.2 Worked Example 1: Step-by-Step Suffix Trie Construction for $S = \text{"aba\$"}$

For $S = \text{"aba\$"}$ ($m = 4$), the $4$ suffixes are:
- $i = 0$: `"aba$"`
- $i = 1$: `"ba$"`
- $i = 2$: `"a$"`
- $i = 3$: `"$"`

```
  Step 1: Insert i=0 ("aba$")   Step 2: Insert i=1 ("ba$")   Step 3: Insert i=2 ("a$")    Step 4: Insert i=3 ("$")
            (root)                        (root)                       (root)                       (root)
            'a'|                        'a'/  \'b'                   'a'/  \'b'               '$'/   'a'|   \'b'
              (a)                         (a) (b)                      (a) (b)                [L:3]    (a)  (b)
            'b'|                        'b'|   |'a'                '$'/ |'b'|'a'                   '$'/ |'b' |'a'
             (ab)                        (ab) (ba)                 [L:2](ab)(ba)                   [L:2](ab)(ba)
            'a'|                        'a'|   |'$'                     |'a' |'$'                       |'a' |'$'
            (aba)                       (aba) [L:1]                   (aba) [L:1]                     (aba) [L:1]
            '$'|                        '$'|                            |'$'                            |'$'
            [L:0]                       [L:0]                          [L:0]                           [L:0]
```

---

### 4.3 Worked Example 2: Full Suffix Trie for $S = \text{"banana\$"}$ (23 Nodes) and $\Theta(m^2)$ Worst-Case Proof

For $S = \text{"banana\$"}$ ($m = 7$), the $7$ suffixes are:
- $i = 0$: `"banana$"`
- $i = 1$: `"anana$"`
- $i = 2$: `"nana$"`
- $i = 3$: `"ana$"`
- $i = 4$: `"na$"`
- $i = 5$: `"a$"`
- $i = 6$: `"$"`

```
                    UNCOMPRESSED SUFFIX TRIE FOR S = "banana$" (23 Total Nodes)

                                           (root)
                              /         /          \          \
                          '$'/      'a'/         'b'\          \'n'
                            /         /              \          \
                        [L:6]       (a)              (b)        (n)
                                 '$'/ \'n'           |'a'       |'a'
                                [L:5] (an)          (ba)       (na)
                                       |'a'          |'n'   '$'/  \'n'
                                     (ana)         (ban)   [L:4] (nan)
                                  '$'/   \'n'        |'a'         |'a'
                                 [L:3]  (anan)     (bana)       (nana)
                                          |'a'       |'n'         |'$'
                                        (anana)   (banan)        [L:2]
                                          |'$'       |'a'
                                         [L:1]   (banana)
                                                     |'$'
                                                    [L:0]

  Count of Nodes: 1 (root) + 1 ("$") + 7 (under 'a') + 7 (under 'b') + 7 (under 'n') = 23 Nodes!
```

#### Why Does an Uncompressed Suffix Trie Require $\Theta(m^2)$ Nodes in the Worst Case?
Consider the string $S = \underbrace{\text{"aa}\dots\text{a"}}_{k}\underbrace{\text{"bb}\dots\text{b"}}_{k}\text{"\$"}$ of length $m = 2k + 1$ (e.g., `"aaabbb$"` for $k = 3$).
- For each $i \in \{0, 1, \dots, k-1\}$, the suffix $\text{Suffix}(i) = \text{"a}^{k-i}\text{b}^k\text{\$}"$ begins with $k - i$ copies of `'a'` and then transitions to a chain of $k$ copies of `'b'`.
- Because each of these $k$ suffixes has a **different** number of leading `'a'`s ($1, 2, \dots, k$), they branch off the `'a'`-spine at $k$ different depths.
- Below each of those $k$ branch points hangs an independent, non-branching chain of $k$ nodes for `"bb...b$"`!
- Total nodes created for the `'b'` chains alone:
  $$k \times k = \left(\frac{m - 1}{2}\right)^2 = \Theta(m^2) \text{ nodes!}$$

---

### 4.4 Substring Search, Occurrence Counting, and Longest Repeated Substring on a Suffix Trie

Given the Suffix Trie (or Suffix Tree) of $S$:
1. **Substring Existence (`search(P)`):** Follow the characters of $P$ from `root`. If every character of $P$ matches an edge/node transition, return `true` in **$O(|P|)$ time**; if any transition fails, return `false`.
2. **Counting and Locating All Occurrences of $P$:**
   - Suppose walking pattern $P$ from `root` lands at node $u$ (or partway along edge $(p, u)$ in a Suffix Tree).
   - Every leaf in the subtree rooted at $u$ represents a suffix $\text{Suffix}(i)$ that has $P$ as a prefix—meaning $P$ occurs in $S$ starting at index $i$!
   - Thus, traversing the subtree of $u$ collects all $\text{occ}$ starting positions `suffix_index` in $O(|P| + \text{occ})$ time (or $O(|P|)$ time to just return the count if each internal node precomputes `leaf_count`).
3. **Longest Repeated Substring (LRS):**
   - A substring $R$ appears **at least twice** in $S$ if and only if at least two distinct suffixes $\text{Suffix}(i)$ and $\text{Suffix}(j)$ ($i \neq j$) share $R$ as a common prefix—which means the path for $R$ reaches an **internal branching node** with $\ge 2$ leaves in its subtree!
   - Therefore, the **Longest Repeated Substring** of $S$ is spelled by the path from `root` to the **internal branching node with the maximum string depth (character count from root)**!
   - In `"banana$"`, the internal branching nodes spell `""` (depth 0), `"a"` (depth 1), `"na"` (depth 2), and `"ana"` (depth 3). The deepest internal node spells **`"ana"`** (length 3, occurring at indices `1` and `3`).

---

## 5. Operations and Algorithms: Compressed Suffix Tree

### 5.1 Path Compression and the $\le 2m - 1$ Node Bound Proof

Look back at the 23-node Uncompressed Suffix Trie of `"banana$"` in Section 4.3. Notice how the branch under `'b'` is a single-child chain of **7 nodes** `(b) -> (ba) -> (ban) -> (bana) -> (banan) -> (banana) -> [L:0]` with zero branching! Similarly, `(anan) -> (anana) -> [L:1]` and `(nan) -> (nana) -> [L:2]` are non-branching chains.

- **Path Compression Rule:** Every maximal chain of nodes $u_0 \to u_1 \to \dots \to u_k$ ($k \ge 2$) where each intermediate node $u_1, \dots, u_{k-1}$ has **exactly one child** (and is not the root) is collapsed into a **single directed edge** $(u_0, u_k)$ labeled with the concatenated substring $S[\text{start} \dots \text{end}]$.

---

### 5.2 Worked Example: Compressing the Suffix Trie of `"banana$"` into Its 11-Node Suffix Tree

```
             COMPRESSED SUFFIX TREE FOR S = "banana$" (m = 7)
             Text Indices:  0:'b'  1:'a'  2:'n'  3:'a'  4:'n'  5:'a'  6:'$'
             Total Nodes :  7 Leaves + 4 Internal Nodes = 11 Nodes (<= 2m - 1 = 13)

                                         (root)
                         /              /      \               \
                  [6..6]/        [1..1]/        \[0..6]         \[2..3]
                    "$"/           "a"/          \"banana$"      \"na"
                      /              /            \               \
                  [Leaf:6]         (I_1)        [Leaf:0]         (I_3)
                                  /     \                       /     \
                           [6..6]/       \[2..3]         [6..6]/       \[4..6]
                             "$"/         \"na"            "$"/         \"na$"
                               /           \                 /           \
                           [Leaf:5]       (I_2)          [Leaf:4]      [Leaf:2]
                                         /     \
                                  [6..6]/       \[4..6]
                                    "$"/         \"na$"
                                      /           \
                                  [Leaf:3]      [Leaf:1]
```

Let us verify every suffix path in this 11-node Compressed Suffix Tree:
- **Suffix 0 (`"banana$"`):** `(root)` $\xrightarrow{[0..6]\text{ "banana\$"}}$ `[Leaf:0]`
- **Suffix 1 (`"anana$"`):** `(root)` $\xrightarrow{[1..1]\text{ "a"}}$ `(I_1)` $\xrightarrow{[2..3]\text{ "na"}}$ `(I_2)` $\xrightarrow{[4..6]\text{ "na\$"}}$ `[Leaf:1]`
- **Suffix 2 (`"nana$"`):** `(root)` $\xrightarrow{[2..3]\text{ "na"}}$ `(I_3)` $\xrightarrow{[4..6]\text{ "na\$"}}$ `[Leaf:2]`
- **Suffix 3 (`"ana$"`):** `(root)` $\xrightarrow{[1..1]\text{ "a"}}$ `(I_1)` $\xrightarrow{[2..3]\text{ "na"}}$ `(I_2)` $\xrightarrow{[6..6]\text{ "\$"}}$ `[Leaf:3]`
- **Suffix 4 (`"na$"`):** `(root)` $\xrightarrow{[2..3]\text{ "na"}}$ `(I_3)` $\xrightarrow{[6..6]\text{ "\$"}}$ `[Leaf:4]`
- **Suffix 5 (`"a$"`):** `(root)` $\xrightarrow{[1..1]\text{ "a"}}$ `(I_1)` $\xrightarrow{[6..6]\text{ "\$"}}$ `[Leaf:5]`
- **Suffix 6 (`"$"`):** `(root)` $\xrightarrow{[6..6]\text{ "\$"}}$ `[Leaf:6]`

---

### 5.3 Step-by-Step Suffix Tree Construction with Edge Splitting

How do we build the Compressed Suffix Tree directly without ever allocating the $\Theta(m^2)$ uncompressed Suffix Trie? We insert suffixes $\text{Suffix}(0), \text{Suffix}(1), \dots, \text{Suffix}(m-1)$ one by one, performing **Edge Splitting** whenever a new suffix matches the first $k$ characters of an existing compressed edge `[start .. end]` and then diverges at character $k+1$:

```
  TRACE OF ITERATIVE SUFFIX INSERTION WITH EDGE SPLITTING ON S = "banana$" (m = 7)

  1. Insert Suffix 0 ("banana$"), i=1 ("anana$"), i=2 ("nana$"):
     All three start with distinct characters ('b', 'a', 'n'), creating 3 leaf edges from root:
                       (root)
             /           |           \
      "anana$"[1..6]  "banana$"[0..6] "nana$"[2..6]
          [L:1]        [L:0]           [L:2]

  2. Insert Suffix 3 ("ana$" = S[3..6]):
     - At root, starts with 'a', so we enter edge [1..6] ("anana$").
     - Compare S[1..6] ("anana$") with S[3..6] ("ana$"):
       k=0: 'a' == 'a'
       k=1: 'n' == 'n'
       k=2: 'a' == 'a'
       k=3: 'n' != '$'  <-- MISMATCH after k = 3 matched characters ("ana")!
     - SPLIT EDGE [1..6] into:
       * Upper edge [1..3] ("ana") -> new internal node (I_2)
       * Old child shortened to [4..6] ("na$") -> [L:1]
       * New leaf edge [6..6] ("$") -> [L:3]

  3. Insert Suffix 4 ("na$" = S[4..6]):
     - Enters edge [2..6] ("nana$"), matches k = 2 chars ("na"), mismatches at 'n' vs '$'.
     - Splits edge [2..6] into [2..3] ("na") -> (I_3) with children [4..6] ("na$" -> [L:2]) and [6..6] ("$" -> [L:4]).

  4. Insert Suffix 5 ("a$" = S[5..6]):
     - Enters edge [1..3] ("ana"), matches k = 1 char ("a"), mismatches at 'n' vs '$'.
     - Splits edge [1..3] into [1..1] ("a") -> (I_1) with children [2..3] ("na" -> (I_2)) and [6..6] ("$" -> [L:5]).

  5. Insert Suffix 6 ("$" = S[6..6]):
     - No '$' edge at root -> adds leaf edge [6..6] ("$" -> [L:6]).
```

---

### 5.4 Honest Complexity Distinction: $O(m^2)$ Educational Construction vs. $O(m)$ Ukkonen's Algorithm

> [!WARNING]
> **Critical Syllabus & Exam Distinction (Do Not Confuse Construction Complexities!):**
> There are **two** fundamentally different ways to construct a Compressed Suffix Tree, and you must state their time complexities honestly:

| Construction Method | Space Complexity | Worst-Case Time Complexity | How It Works |
| :--- | :---: | :---: | :--- |
| **1. Naive Suffix Trie + Path Compression** | $\Theta(m^2)$ peak during build $\to O(m)$ after | **$\Theta(m^2)$** | Builds full $\Theta(m^2)$-node Suffix Trie first, then collapses single-child chains |
| **2. Direct Suffix Insertion with Edge Splitting** *(Implemented below)* | **$O(m)$** ($\le 2m - 1$ nodes at all times) | **$O(m^2)$** | Inserts suffixes $i = 0 \dots m-1$ from `root`, walking & splitting `[start, end]` edges |
| **3. Ukkonen's Online Algorithm (1995)** *(or Weiner 1973 / McCreight 1976)* | **$O(m)$** ($\le 2m - 1$ nodes) | **$O(m)$ (Linear Time!)** | Processes $S$ left-to-right in $m$ phases using **Suffix Links**, an **Active Point**, **Skip/Count Trick**, and **Global Leaf End** |

- **Why Does Direct Edge-Splitting Take $O(m^2)$ Time Even Though the Tree Has Only $O(m)$ Nodes?**
  Because when inserting $\text{Suffix}(i)$ from the root, we still compare characters one by one along edges (`while (text[child->start + k] == text[pos + k]) k++`) from depth $0$ down to the split point. Across all $m$ suffixes, the total number of character comparisons can still be $\sum_{i=0}^{m-1} (m - i) = \Theta(m^2)$ (e.g., on $S = \text{"a}^{m-1}\text{\$}"$).
- **How Does Ukkonen's Algorithm Achieve True $O(m)$ Time?**
  Ukkonen's algorithm avoids restarting at the root for each suffix and avoids scanning characters inside edges via four mechanisms:
  1. **Implicit Open Leaf Edges (`end = &global_end`):** All leaf edges share a pointer to the current phase index `i`; incrementing `global_end++` extends all existing leaves by $S[i]$ in **$O(1)$ total time**.
  2. **Rule 3 ("Showstopper" Rule):** In phase $i$, as soon as the current suffix $S[j \dots i]$ is found to already exist in the tree, $S[j+1 \dots i], \dots, S[i \dots i]$ are guaranteed to exist too—so the phase terminates immediately in $O(1)$ time!
  3. **Suffix Links ($u \to s(u)$):** Every internal node $u$ spelling $c\alpha$ maintains a pointer to the internal node $s(u)$ spelling $\alpha$, allowing the algorithm to jump directly from $\text{Suffix}(j)$ to $\text{Suffix}(j+1)$ in $O(1)$ time without returning to the root.
  4. **Skip/Count Trick:** When jumping along a suffix link, since the target path is guaranteed to exist, the algorithm jumps over entire `[start, end]` edges in $O(1)$ time per edge by adding `edgeLength` instead of comparing individual characters.

---

### 5.5 Complete C11 Implementation: Suffix Trie and Compressed Suffix Tree on `"banana$"`

```c
/**
 * @file suffix_trie_and_tree.c
 * @brief Complete, self-contained C11 implementation of BOTH:
 *        1. Uncompressed Suffix Trie (O(m^2) time and space)
 *        2. Compressed Suffix Tree using [start, end] index pairs and
 *           dynamic Edge Splitting (O(m) space, <= 2m - 1 nodes, O(m^2) build time),
 *        demonstrating structure visualization, O(|P| + occ) substring search,
 *        and Longest Repeated Substring (LRS) on "banana$".
 *        Compile: gcc -std=c11 -Wall -Wextra suffix_trie_and_tree.c -o suffix_demo
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <string.h>

/*
 * Alphabet mapping for lowercase 'a'..'z' plus terminal marker '$':
 *   Index 0      -> '$' (lexicographically smallest terminal symbol)
 *   Index 1..26  -> 'a'..'z'
 */
#define SUFFIX_ALPHA_SIZE 27

static inline int suffixCharToIndex(char ch) {
    if (ch == '$') return 0;
    if (ch >= 'a' && ch <= 'z') return (ch - 'a') + 1;
    if (ch >= 'A' && ch <= 'Z') return (ch - 'A') + 1;
    return -1;
}

/* ============================================================================
 * PART 1: UNCOMPRESSED SUFFIX TRIE
 * ============================================================================ */

typedef struct SuffixTrieNode {
    struct SuffixTrieNode *children[SUFFIX_ALPHA_SIZE];
    int suffix_index; /* -1 for internal nodes; 0..m-1 for leaf nodes */
} SuffixTrieNode;

static SuffixTrieNode *createSuffixTrieNode(int *node_counter) {
    SuffixTrieNode *node = (SuffixTrieNode *)malloc(sizeof(SuffixTrieNode));
    if (!node) {
        perror("Failed to allocate SuffixTrieNode");
        exit(EXIT_FAILURE);
    }
    for (int i = 0; i < SUFFIX_ALPHA_SIZE; i++) {
        node->children[i] = NULL;
    }
    node->suffix_index = -1;
    if (node_counter) (*node_counter)++;
    return node;
}

SuffixTrieNode *buildSuffixTrie(const char *text, int *total_nodes) {
    *total_nodes = 0;
    SuffixTrieNode *root = createSuffixTrieNode(total_nodes);
    int m = (int)strlen(text);

    /* Insert all m suffixes: text[i .. m-1] for i = 0 .. m-1 */
    for (int i = 0; i < m; i++) {
        SuffixTrieNode *curr = root;
        for (int j = i; j < m; j++) {
            int idx = suffixCharToIndex(text[j]);
            if (idx < 0) continue;
            if (curr->children[idx] == NULL) {
                curr->children[idx] = createSuffixTrieNode(total_nodes);
            }
            curr = curr->children[idx];
        }
        curr->suffix_index = i; /* Leaf node stores starting index of suffix */
    }
    return root;
}

static void collectTrieLeaves(const SuffixTrieNode *node, int matches[], int *count) {
    if (!node) return;
    if (node->suffix_index != -1) {
        matches[(*count)++] = node->suffix_index;
    }
    for (int i = 0; i < SUFFIX_ALPHA_SIZE; i++) {
        if (node->children[i] != NULL) {
            collectTrieLeaves(node->children[i], matches, count);
        }
    }
}

int searchSuffixTrie(const SuffixTrieNode *root, const char *pattern, int matches[]) {
    const SuffixTrieNode *curr = root;
    for (size_t i = 0; pattern[i] != '\0'; i++) {
        int idx = suffixCharToIndex(pattern[i]);
        if (idx < 0 || curr->children[idx] == NULL) {
            return 0; /* Pattern not found */
        }
        curr = curr->children[idx];
    }
    int count = 0;
    collectTrieLeaves(curr, matches, &count);
    return count;
}

void freeSuffixTrie(SuffixTrieNode *node) {
    if (!node) return;
    for (int i = 0; i < SUFFIX_ALPHA_SIZE; i++) {
        if (node->children[i]) freeSuffixTrie(node->children[i]);
    }
    free(node);
}

/* ============================================================================
 * PART 2: COMPRESSED SUFFIX TREE (WITH [start, end] EDGES & EDGE SPLITTING)
 * ============================================================================
 * Note on Complexity:
 *   This implementation constructs the exact Compressed Suffix Tree with
 *   at most 2m - 1 nodes and O(m) space using [start, end] substring indices
 *   via direct suffix insertion and edge splitting in O(m^2) worst-case time.
 */

typedef struct SuffixTreeNode {
    struct SuffixTreeNode *children[SUFFIX_ALPHA_SIZE];
    int start;        /* Start index of incoming edge label in text S (-1 for root) */
    int end;          /* End index (inclusive) of incoming edge label in text S     */
    int suffix_index; /* -1 for internal nodes; 0..m-1 for leaf nodes               */
} SuffixTreeNode;

static SuffixTreeNode *createSuffixTreeNode(int start, int end, int suffix_index, int *node_counter) {
    SuffixTreeNode *node = (SuffixTreeNode *)malloc(sizeof(SuffixTreeNode));
    if (!node) {
        perror("Failed to allocate SuffixTreeNode");
        exit(EXIT_FAILURE);
    }
    for (int i = 0; i < SUFFIX_ALPHA_SIZE; i++) {
        node->children[i] = NULL;
    }
    node->start = start;
    node->end = end;
    node->suffix_index = suffix_index;
    if (node_counter) (*node_counter)++;
    return node;
}

static inline int edgeLength(const SuffixTreeNode *node) {
    if (!node || node->start == -1) return 0;
    return node->end - node->start + 1;
}

/*
 * Insert suffix text[suffix_idx .. m-1] into the Compressed Suffix Tree,
 * splitting an existing compressed edge whenever a partial match diverges.
 */
static void insertSuffixCompressed(SuffixTreeNode *root, const char *text,
                                   int suffix_idx, int m, int *node_counter) {
    SuffixTreeNode *curr = root;
    int pos = suffix_idx;

    while (pos < m) {
        int idx = suffixCharToIndex(text[pos]);
        SuffixTreeNode *child = curr->children[idx];

        /* Case A: No outgoing edge starts with text[pos] -> create a new leaf edge [pos, m-1] */
        if (child == NULL) {
            curr->children[idx] = createSuffixTreeNode(pos, m - 1, suffix_idx, node_counter);
            return;
        }

        /* Case B: Outgoing edge [child->start .. child->end] exists; walk along it */
        int edge_len = edgeLength(child);
        int k = 0;
        while (k < edge_len && pos + k < m && text[child->start + k] == text[pos + k]) {
            k++;
        }

        /* Subcase B1: Matched the entire edge [child->start .. child->end] -> descend to child */
        if (k == edge_len) {
            curr = child;
            pos += k;
        } else {
            /* Subcase B2: Mismatch after k < edge_len characters!
             * Split edge [child->start .. child->end] at index (child->start + k - 1):
             *   1. Create new internal node 'split_node' with edge [child->start, child->start + k - 1]
             *   2. Shorten 'child' edge to [child->start + k, child->end] and attach under 'split_node'
             *   3. Create new leaf node for remaining suffix [pos + k, m - 1] under 'split_node'
             */
            SuffixTreeNode *split_node = createSuffixTreeNode(
                child->start, child->start + k - 1, -1, node_counter
            );

            /* Replace child under curr with split_node */
            curr->children[idx] = split_node;

            /* Adjust existing child's start index and attach to split_node */
            child->start += k;
            int old_child_idx = suffixCharToIndex(text[child->start]);
            split_node->children[old_child_idx] = child;

            /* Create new leaf for the newly diverging suffix */
            int new_leaf_idx = suffixCharToIndex(text[pos + k]);
            split_node->children[new_leaf_idx] = createSuffixTreeNode(
                pos + k, m - 1, suffix_idx, node_counter
            );
            return;
        }
    }
}

SuffixTreeNode *buildCompressedSuffixTree(const char *text, int *total_nodes) {
    *total_nodes = 0;
    SuffixTreeNode *root = createSuffixTreeNode(-1, -1, -1, total_nodes);
    int m = (int)strlen(text);

    for (int i = 0; i < m; i++) {
        insertSuffixCompressed(root, text, i, m, total_nodes);
    }
    return root;
}

static void collectTreeLeaves(const SuffixTreeNode *node, int matches[], int *count) {
    if (!node) return;
    if (node->suffix_index != -1) {
        matches[(*count)++] = node->suffix_index;
    }
    for (int i = 0; i < SUFFIX_ALPHA_SIZE; i++) {
        if (node->children[i] != NULL) {
            collectTreeLeaves(node->children[i], matches, count);
        }
    }
}

/*
 * Search for pattern P in Compressed Suffix Tree in O(|P| + occ) time.
 */
int searchCompressedSuffixTree(const SuffixTreeNode *root, const char *text,
                               const char *pattern, int matches[]) {
    const SuffixTreeNode *curr = root;
    int pat_len = (int)strlen(pattern);
    int pos = 0;

    while (pos < pat_len) {
        int idx = suffixCharToIndex(pattern[pos]);
        if (idx < 0 || curr->children[idx] == NULL) {
            return 0; /* No edge starts with pattern[pos] */
        }

        const SuffixTreeNode *child = curr->children[idx];
        int edge_len = edgeLength(child);
        int k = 0;

        while (k < edge_len && pos < pat_len) {
            if (text[child->start + k] != pattern[pos]) {
                return 0; /* Character mismatch along compressed edge */
            }
            k++;
            pos++;
        }

        curr = child;
    }

    /* Pattern completely matched! Collect all leaf suffix indices in curr's subtree */
    int count = 0;
    collectTreeLeaves(curr, matches, &count);
    return count;
}

/*
 * Find Longest Repeated Substring (LRS):
 * Deepest internal node (suffix_index == -1) by total character depth from root.
 */
static void findLRSDFS(const SuffixTreeNode *node, int curr_depth, int *max_len,
                       int *end_pos_in_text) {
    if (!node) return;

    /* Any internal node (except root) in a Suffix Tree has >= 2 leaves in its subtree */
    if (node->suffix_index == -1 && node->start != -1) {
        if (curr_depth > *max_len) {
            *max_len = curr_depth;
            *end_pos_in_text = node->end;
        }
    }

    for (int i = 0; i < SUFFIX_ALPHA_SIZE; i++) {
        SuffixTreeNode *child = node->children[i];
        if (child != NULL && child->suffix_index == -1) {
            findLRSDFS(child, curr_depth + edgeLength(child), max_len, end_pos_in_text);
        }
    }
}

void printSuffixTreeStructure(const SuffixTreeNode *node, const char *text, int depth) {
    if (!node) return;

    if (node->start == -1) {
        printf("  (root)\n");
    } else {
        for (int d = 0; d < depth; d++) printf("    ");
        printf("|-- Edge [%d..%d] \"", node->start, node->end);
        for (int i = node->start; i <= node->end; i++) {
            putchar(text[i]);
        }
        if (node->suffix_index != -1) {
            printf("\" --> [LEAF: suffix_index = %d]\n", node->suffix_index);
        } else {
            printf("\" --> (Internal Branching Node)\n");
        }
    }

    for (int i = 0; i < SUFFIX_ALPHA_SIZE; i++) {
        if (node->children[i] != NULL) {
            printSuffixTreeStructure(node->children[i], text, depth + 1);
        }
    }
}

void freeSuffixTree(SuffixTreeNode *node) {
    if (!node) return;
    for (int i = 0; i < SUFFIX_ALPHA_SIZE; i++) {
        if (node->children[i]) freeSuffixTree(node->children[i]);
    }
    free(node);
}

/* ============================================================================
 * PART 3: MAIN DRIVER DEMONSTRATION
 * ============================================================================ */

int main(void) {
    const char *text = "banana$";
    int m = (int)strlen(text);

    int trie_nodes = 0;
    int tree_nodes = 0;

    SuffixTrieNode *s_trie = buildSuffixTrie(text, &trie_nodes);
    SuffixTreeNode *s_tree = buildCompressedSuffixTree(text, &tree_nodes);

    printf("=== 1. NODE COUNT COMPARISON FOR S = \"%s\" (m = %d) ===\n", text, m);
    printf("  Uncompressed Suffix Trie Total Nodes : %d nodes\n", trie_nodes);
    printf("  Compressed Suffix Tree Total Nodes   : %d nodes (<= 2m - 1 = %d)\n\n",
           tree_nodes, 2 * m - 1);

    printf("=== 2. COMPRESSED SUFFIX TREE STRUCTURE FOR \"%s\" ===\n", text);
    printSuffixTreeStructure(s_tree, text, 0);
    printf("\n");

    printf("=== 3. SUBSTRING SEARCH & OCCURRENCE LOCATIONS ===\n");
    const char *patterns[] = {"ana", "na", "ban", "a", "nan", "apple"};
    int matches[32];

    for (size_t p = 0; p < sizeof(patterns) / sizeof(patterns[0]); p++) {
        const char *pat = patterns[p];
        int count_trie = searchSuffixTrie(s_trie, pat, matches);
        int count_tree = searchCompressedSuffixTree(s_tree, text, pat, matches);

        printf("  Pattern %-7s -> Found: %-3s | Occurrences: %d (Trie=%d, Tree=%d)",
               pat, (count_tree > 0) ? "YES" : "NO", count_tree, count_trie, count_tree);
        if (count_tree > 0) {
            printf(" at indices: [ ");
            for (int i = 0; i < count_tree; i++) {
                printf("%d ", matches[i]);
            }
            printf("]");
        }
        printf("\n");
    }
    printf("\n");

    printf("=== 4. LONGEST REPEATED SUBSTRING (LRS) IN \"%s\" ===\n", text);
    int max_len = 0, end_pos = -1;
    findLRSDFS(s_tree, 0, &max_len, &end_pos);
    printf("  Longest Repeated Substring Length: %d\n", max_len);
    printf("  Longest Repeated Substring       : \"");
    for (int i = end_pos - max_len + 1; i <= end_pos; i++) {
        putchar(text[i]);
    }
    printf("\"\n");

    freeSuffixTrie(s_trie);
    freeSuffixTree(s_tree);
    return 0;
}
```

#### Program Output:

```text
=== 1. NODE COUNT COMPARISON FOR S = "banana$" (m = 7) ===
  Uncompressed Suffix Trie Total Nodes : 23 nodes
  Compressed Suffix Tree Total Nodes   : 11 nodes (<= 2m - 1 = 13)

=== 2. COMPRESSED SUFFIX TREE STRUCTURE FOR "banana$" ===
  (root)
    |-- Edge [6..6] "$" --> [LEAF: suffix_index = 6]
    |-- Edge [1..1] "a" --> (Internal Branching Node)
        |-- Edge [6..6] "$" --> [LEAF: suffix_index = 5]
        |-- Edge [2..3] "na" --> (Internal Branching Node)
            |-- Edge [6..6] "$" --> [LEAF: suffix_index = 3]
            |-- Edge [4..6] "na$" --> [LEAF: suffix_index = 1]
    |-- Edge [0..6] "banana$" --> [LEAF: suffix_index = 0]
    |-- Edge [2..3] "na" --> (Internal Branching Node)
        |-- Edge [6..6] "$" --> [LEAF: suffix_index = 4]
        |-- Edge [4..6] "na$" --> [LEAF: suffix_index = 2]

=== 3. SUBSTRING SEARCH & OCCURRENCE LOCATIONS ===
  Pattern ana     -> Found: YES | Occurrences: 2 (Trie=2, Tree=2) at indices: [ 3 1 ]
  Pattern na      -> Found: YES | Occurrences: 2 (Trie=2, Tree=2) at indices: [ 4 2 ]
  Pattern ban     -> Found: YES | Occurrences: 1 (Trie=1, Tree=1) at indices: [ 0 ]
  Pattern a       -> Found: YES | Occurrences: 3 (Trie=3, Tree=3) at indices: [ 5 3 1 ]
  Pattern nan     -> Found: YES | Occurrences: 1 (Trie=1, Tree=1) at indices: [ 2 ]
  Pattern apple   -> Found: NO  | Occurrences: 0 (Trie=0, Tree=0)

=== 4. LONGEST REPEATED SUBSTRING (LRS) IN "banana$" ===
  Longest Repeated Substring Length: 3
  Longest Repeated Substring       : "ana"
```

---

## 6. Comparison Tables and Complexity Summary

### 6.1 Time and Space Complexity Table

Let $S$ be a text of length $m$ (including `$`), $P$ be a query pattern of length $|P|$, and $\text{occ}$ be the number of occurrences of $P$ in $S$.

| Operation / Metric | Uncompressed Suffix Trie | Compressed Suffix Tree (Educational Edge-Split) | Compressed Suffix Tree (Ukkonen's Algorithm) |
| :--- | :---: | :---: | :---: |
| **Total Leaves** | $m$ | $m$ | $m$ |
| **Total Internal Nodes** | Up to $\Theta(m^2)$ | $\le m - 1$ | $\le m - 1$ |
| **Total Nodes** | $\Theta(m^2)$ worst-case | **$\le 2m - 1 \in O(m)$** | **$\le 2m - 1 \in O(m)$** |
| **Space Complexity** | $\Theta(m^2 \cdot |\Sigma|)$ | **$O(m \cdot |\Sigma|)$** | **$O(m \cdot |\Sigma|)$** |
| **Construction Time** | $\Theta(m^2)$ | $O(m^2)$ | **$O(m)$** (for constant $|\Sigma|$) |
| **Substring Existence (`search(P)`)** | $O(|P|)$ | **$O(|P|)$** | **$O(|P|)$** |
| **Find All Occurrences of $P$** | $O(|P| + \text{occ}$ subtree$)$ | **$O(|P| + \text{occ})$** | **$O(|P| + \text{occ})$** |
| **Longest Repeated Substring** | $O(m^2)$ tree walk | **$O(m)$** tree walk | **$O(m)$** tree walk |

*(Note: Why is finding all $\text{occ}$ occurrences $O(|P| + \text{occ})$ in a Compressed Suffix Tree, whereas in an Uncompressed Suffix Trie the subtree walk can take $O(|P| + m \cdot \text{occ})$? Because in a Compressed Suffix Tree every internal node has degree $\ge 2$, so a subtree with $\text{occ}$ leaves has at most $\text{occ} - 1$ internal nodes! Thus, traversing the subtree below the match point visits at most $2 \cdot \text{occ} - 1 = O(\text{occ})$ nodes!)*

### 6.2 Structural Comparison: Standard Trie vs. Uncompressed Suffix Trie vs. Compressed Suffix Tree

| Characteristic | Standard Trie (Topic 2) | Uncompressed Suffix Trie | Compressed Suffix Tree |
| :--- | :--- | :--- | :--- |
| **Primary Input** | Dictionary of $n$ strings $\{W_1, \dots, W_n\}$ | Single text $S[0 \dots m-1]$ ($m$ suffixes) | Single text $S[0 \dots m-1]$ ($m$ suffixes) |
| **Edge Labels** | Single character $c \in \Sigma$ | Single character $c \in \Sigma \cup \{\$\}$ | Substring slice `[start, end]` of $S$ |
| **Single-Child Internal Nodes Allowed?** | Yes | Yes (causes $\Theta(m^2)$ node blowup) | **No!** (All non-branching chains compressed) |
| **Primary Query Solved** | Exact word & prefix lookup ($W[0 \dots k]$) | Arbitrary substring lookup ($S[i \dots j]$) | Arbitrary substring lookup ($S[i \dots j]$) |
| **Worst-Case Space** | $O\bigl(|\Sigma| \sum |W_i|\bigr)$ | $\Theta(m^2 \cdot |\Sigma|)$ | **$O(m \cdot |\Sigma|)$** |

---

## 7. Common Errors, Pitfalls, and Edge Cases

### 7.1 Pitfall 1: Omitting the Terminal Sentinel `$` (Losing Suffixes Inside Internal Nodes)

```c
/* BUG: Building a Suffix Tree on "banana" WITHOUT appending '$' */
SuffixTreeNode *tree = buildCompressedSuffixTree("banana", &nodes);
/* Suffix 5 ("a"), Suffix 3 ("ana"), and Suffix 1 ("anana"):
 * "a" and "ana" are proper prefixes of "anana"! Without '$', they do NOT branch off to leaves,
 * so collectTreeLeaves() for pattern "a" only finds 1 leaf ("anana") instead of 3 occurrences! */

/* FIX: Always ensure text ends with a unique terminal symbol '$' not present in the pattern alphabet */
SuffixTreeNode *tree = buildCompressedSuffixTree("banana$", &nodes);
```

### 7.2 Pitfall 2: Copying Substring Characters Onto Compressed Edges Instead of `[start, end]` Indices

```c
/* BUG: Storing a heap-allocated substring inside each SuffixTreeNode */
typedef struct BadSuffixTreeNode {
    char *edge_string; /* Takes O(m) bytes per edge -> O(m^2) total space across m leaves! */
    struct BadSuffixTreeNode *children[27];
} BadSuffixTreeNode;

/* FIX: Store only two integers (start, end) indexing into the shared text buffer S */
typedef struct GoodSuffixTreeNode {
    int start;         /* O(1) space per edge -> O(m) total space across <= 2m - 1 nodes! */
    int end;
    int suffix_index;
    struct GoodSuffixTreeNode *children[27];
} GoodSuffixTreeNode;
```

### 7.3 Pitfall 3: Claiming an $O(m^2)$ Edge-Splitting Implementation Runs in $O(m)$ Time

```c
/* ACADEMIC & INTERVIEW PITFALL:
 * Inserting suffixes i = 0 .. m-1 from the root and splitting edges achieves O(m) SPACE
 * (at most 2m - 1 nodes), but its construction time is still O(m^2) in the worst case!
 * True O(m) construction time requires Ukkonen's Algorithm with Suffix Links, Active Point,
 * Global Leaf End pointer, and the Showstopper Rule (Rule 3). */
```

### 7.4 Boundary Conditions Checklist

- [ ] **Pattern Ends Mid-Edge (`pos == pat_len` while `k < edge_len`):** Does `searchCompressedSuffixTree` succeed when the pattern $P$ finishes in the *middle* of a compressed edge (e.g., searching `"ban"` on edge `[0..6] "banana$"`) and then collect all leaves in `child`'s subtree? (Yes! The inner `while (k < edge_len && pos < pat_len)` loop exits with `pos == pat_len` and sets `curr = child`).
- [ ] **Mismatch Mid-Edge:** Does `searchCompressedSuffixTree` return `0` immediately if `text[child->start + k] != pattern[pos]` partway along an edge (e.g., searching `"band"` along `"banana$"`)?
- [ ] **Repeated Single-Character String (`"aaaaa$"`):** Does edge splitting properly chain internal nodes when every suffix shares long prefixes of `'a'`?
- [ ] **Terminal Symbol Excluded from Queries:** Does the user search `"ana"` (without `$`) to find all internal occurrences, or `"ana$"` (with `$`) if they specifically want to test whether `"ana"` is a *suffix* of the original string? (Searching `"ana$"` checks if `"ana"` is an exact suffix!).

---

## 8. Summary and Revision Checklist

### 8.1 Key Takeaways and Core Invariants

1. **Fundamental Substring-Suffix Identity:** Every substring $S[i \dots j]$ of $S$ is a **prefix** of $\text{Suffix}(i) = S[i \dots m-1]$. Indexing all suffixes turns arbitrary substring search into $O(|P|)$ prefix search.
2. **Role of Terminal Marker `$`:** Appending $\$ \notin \Sigma$ ensures no suffix is a proper prefix of another suffix, guaranteeing a 1-to-1 bijection between the $m$ suffixes of $S$ and the $m$ leaves of the tree.
3. **Suffix Trie vs. Suffix Tree Space:** An Uncompressed Suffix Trie can have $\Theta(m^2)$ nodes (on $S = \text{"a}^k\text{b}^k\text{\$}"$). A Compressed Suffix Tree eliminates all degree-1 internal nodes via **Path Compression** and represents edge labels as `[start, end]` index pairs, guaranteeing **$m$ leaves**, **$\le m - 1$ internal nodes**, **$\le 2m - 1$ total nodes**, and **$O(m)$ space**.
4. **Honest Construction Complexity:** Constructing a Suffix Tree by inserting $m$ suffixes from the root with edge splitting uses $O(m)$ space and $O(m^2)$ time. **Ukkonen's algorithm** achieves **$O(m)$ time and $O(m)$ space** using suffix links and online phase extensions.

### 8.2 Self-Test Revision Questions

1. **Combinatorial Question:** Suppose $S = \text{"mississippi\$"}$ has length $m = 12$ (including `$`). Without even drawing the Compressed Suffix Tree, what is the exact number of leaf nodes, the maximum possible number of internal nodes, and the maximum possible total number of nodes?
   *(Answer: Exactly $L = m = 12$ leaves; at most $m - 1 = 11$ internal nodes; and at most $2m - 1 = 23$ total nodes).*
2. **Conceptual Question:** In a Compressed Suffix Tree with $\text{occ}$ occurrences of pattern $P$, why does collecting all matching leaf indices take $O(\text{occ})$ time after matching $P$ in $O(|P|)$ time, whereas in an Uncompressed Suffix Trie it could take $O(m \cdot \text{occ})$ time?
   *(Answer: In a Compressed Suffix Tree, every internal node in the subtree below the match locus has at least $2$ children, so a subtree with $\text{occ}$ leaves contains at most $\text{occ} - 1$ internal nodes and $\le 2\text{occ} - 1$ total nodes. In an Uncompressed Suffix Trie, each of the $\text{occ}$ leaves can hang at the end of a non-branching chain of length $O(m)$).*
3. **Algorithmic Question:** How can you use the Suffix Tree of $S$ (with `$`) to test in $O(|P|)$ time whether a query string $P$ is an **exact suffix** of the original string $S_0$ (without `$`)?
   *(Answer: Search for the pattern $P\text{"\$"}$—i.e., $P$ followed by the terminal marker `$`—from the root, which succeeds iff $P$ appears immediately before `$` at the end of $S$).*

### 8.3 Navigation Links

- **Previous Topic:** [02-Trie-Trees.md](./02-Trie-Trees.md) — Trie Trees (Prefix Trees / Digital Search Trees)
- **Current File:** `03-Suffix-Tries-and-Suffix-Trees.md` (Unit 4 — Topic 3)
- **Next Topic:** [04-Hash-Functions-and-Hash-Tables.md](./04-Hash-Functions-and-Hash-Tables.md) — Hash Functions and Hash Tables
- **Further Ahead:** [05-Collision-Handling-and-Rehashing.md](./05-Collision-Handling-and-Rehashing.md) — Collision Handling and Rehashing
