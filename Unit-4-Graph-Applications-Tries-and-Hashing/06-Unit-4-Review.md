# Unit 4 — Comprehensive Review & Exam Preparation: Graph Applications, Tries, Suffix Trees, and Hashing

---

## Table of Contents

1. [Important Definitions Glossary (All of Unit 4)](#1-important-definitions-glossary-all-of-unit-4)
   - 1.1 [Graph Applications Terminology](#11-graph-applications-terminology)
   - 1.2 [Tries, Suffix Tries, and Suffix Trees Terminology](#12-tries-suffix-tries-and-suffix-trees-terminology)
   - 1.3 [Hash Functions, Collision Resolution, and Rehashing Terminology](#13-hash-functions-collision-resolution-and-rehashing-terminology)
2. [Essential Diagrams and Mathematical Formulas](#2-essential-diagrams-and-mathematical-formulas)
   - 2.1 [Graph Applications: Connected Components & BFS Shortest Unweighted Path](#21-graph-applications-connected-components--bfs-shortest-unweighted-path)
   - 2.2 [Trie and Suffix Tree Node & Space Bounds](#22-trie-and-suffix-tree-node--space-bounds)
   - 2.3 [Hash Functions, Probing Formulas, and Expected Probe Counts](#23-hash-functions-probing-formulas-and-expected-probe-counts)
3. [Commonly Confused Concepts (Exam Pitfall Clarifications)](#3-commonly-confused-concepts-exam-pitfall-clarifications)
   - 3.1 [BFS vs. DFS for Shortest Unweighted Path](#31-bfs-vs-dfs-for-shortest-unweighted-path)
   - 3.2 [Standard Trie vs. Suffix Trie vs. Compressed Suffix Tree](#32-standard-trie-vs-suffix-trie-vs-compressed-suffix-tree)
   - 3.3 [Primary Clustering (Linear Probing) vs. Secondary Clustering (Quadratic Probing)](#33-primary-clustering-linear-probing-vs-secondary-clustering-quadratic-probing)
   - 3.4 [Why Open Addressing Deletion Requires `DELETED` Tombstones Instead of `EMPTY`](#34-why-open-addressing-deletion-requires-deleted-tombstones-instead-of-empty)
   - 3.5 [Why `realloc` / `memcpy` Cannot Resize a Hash Table Without Rehashing](#35-why-realloc--memcpy-cannot-resize-a-hash-table-without-rehashing)
4. [Consolidated Operation-Complexity Table for Unit 4](#4-consolidated-operation-complexity-table-for-unit-4)
5. [Short-Answer Questions (10 Exam Questions)](#5-short-answer-questions-10-exam-questions)
6. [Conceptual Questions (8 Deep-Thinking Questions)](#6-conceptual-questions-8-deep-thinking-questions)
7. [Algorithm-Writing Questions (5 Pseudocode Problems)](#7-algorithm-writing-questions-5-pseudocode-problems)
8. [C Programming Questions (5 Coding Problems)](#8-c-programming-questions-5-coding-problems)
9. [Worked Problem-Solving Questions (Step-by-Step Traces)](#9-worked-problem-solving-questions-step-by-step-traces)
   - 9.1 [Problem 1: Connected Components & BFS Shortest Unweighted Path Trace](#91-problem-1-connected-components--bfs-shortest-unweighted-path-trace)
   - 9.2 [Problem 2: Standard Trie Insertion, Prefix Search, and Bottom-Up Deletion Trace](#92-problem-2-standard-trie-insertion-prefix-search-and-bottom-up-deletion-trace)
   - 9.3 [Problem 3: Suffix Tree Construction and Pattern Matching for `"banana$"`](#93-problem-3-suffix-tree-construction-and-pattern-matching-for-banana)
   - 9.4 [Problem 4: Master Hash Table Collision Trace (Chaining vs. Linear vs. Quadratic vs. Double Hashing)](#94-problem-4-master-hash-table-collision-trace-chaining-vs-linear-vs-quadratic-vs-double-hashing)
   - 9.5 [Problem 5: Dynamic Rehashing Step-by-Step Trace ($m = 5 \to m' = 11$)](#95-problem-5-dynamic-rehashing-step-by-step-trace-m--5-to-m--11)
10. [Complete Answer Keys with Detailed Explanations](#10-complete-answer-keys-with-detailed-explanations)
    - 10.1 [Answers to Short-Answer Questions (Section 5)](#101-answers-to-short-answer-questions-section-5)
    - 10.2 [Answers to Conceptual Questions (Section 6)](#102-answers-to-conceptual-questions-section-6)
    - 10.3 [Solutions to Algorithm-Writing Questions (Section 7)](#103-solutions-to-algorithm-writing-questions-section-7)
    - 10.4 [Complete C11 Solutions to C Programming Questions (Section 8)](#104-complete-c11-solutions-to-c-programming-questions-section-8)
11. [Navigation Links](#11-navigation-links)

---

## 1. Important Definitions Glossary (All of Unit 4)

### 1.1 Graph Applications Terminology

| Term | Formal Definition |
| :--- | :--- |
| **Connected Graph (Undirected)** | An undirected graph $G = (V, E)$ in which there exists a path between every pair of vertices $u, v \in V$. |
| **Connected Component** | A **maximal** connected subgraph of an undirected graph $G$. The connected components partition $V$ into disjoint equivalence classes $C_1 \sqcup C_2 \sqcup \dots \sqcup C_k = V$ under path reachability. |
| **Strongly Connected Component (SCC)** | A maximal subset of vertices $C \subseteq V$ in a **directed** graph such that for every pair $u, v \in C$, there is both a directed path $u \leadsto v$ and a directed path $v \leadsto u$. |
| **Unweighted Path Length** | The number of edges $k$ along a path $P = \langle v_0, v_1, \dots, v_k \rangle$. |
| **Shortest Unweighted Path ($\delta(s, v)$)** | The minimum number of edges on any path from source $s$ to target $v$ in $G$ (or $+\infty$ if $v$ is unreachable from $s$). |
| **Breadth-First Tree (BFS Tree)** | The spanning tree of reachable vertices rooted at source $s$ formed by the predecessor edges $\{(\text{parent}[v], v) : v \neq s\}$ discovered during BFS, where every tree path from $s$ to $v$ has length exactly $\delta(s, v)$. |
| **Bipartite Graph** | A graph whose vertices can be partitioned into two disjoint independent sets $V = V_1 \sqcup V_2$ such that every edge connects a vertex in $V_1$ to one in $V_2$ (equivalent to having **no odd-length cycles**, verifiable in $O(V + E)$ via 2-coloring BFS). |

### 1.2 Tries, Suffix Tries, and Suffix Trees Terminology

| Term | Formal Definition |
| :--- | :--- |
| **Trie (Prefix Tree / Digital Search Tree)** | An $m$-ary rooted tree ($m = |\Sigma|$) storing a dynamic set of strings over alphabet $\Sigma$, where each edge is labeled with a single character, root-to-node paths represent prefixes, and nodes carry an `isEndOfWord` boolean flag. |
| **Compressed Trie (Radix Tree / Patricia Trie)** | A space-optimized Trie obtained by contracting every non-branching chain of single-child internal nodes (where `isEndOfWord == false`) into a single edge labeled with a multi-character substring. |
| **Suffix of String $T$** | Any substring $T[i \,..\, m-1]$ ($0 \le i < m$) starting at index $i$ and extending to the end of $T$. A string of length $m$ has $m$ non-empty suffixes. |
| **Terminal Sentinel (`$`)** | A special character $\$ \notin \Sigma$ appended to the end of text $T$ before building a Suffix Trie/Tree so that **no suffix of $T\$$ is a prefix of any other suffix**, guaranteeing that every suffix ends at its own distinct leaf node. |
| **Suffix Trie (Uncompressed)** | A standard character-by-character Trie containing all $m$ suffixes of $T\$$. Requires $O(m^2)$ worst-case nodes and space. |
| **Suffix Tree (Compressed Suffix Trie)** | The compressed Trie of all $m$ suffixes of $T\$$, where every internal node (except possibly the root) has $\ge 2$ children and edge labels are represented in $O(1)$ space as index pairs `(start, end)` into $T\$$, yielding **$O(m)$ total nodes and $O(m)$ total space**. |

### 1.3 Hash Functions, Collision Resolution, and Rehashing Terminology

| Term | Formal Definition |
| :--- | :--- |
| **Hash Function ($h(k)$)** | A deterministic function $h : U \to \{0, 1, \dots, m-1\}$ mapping a key $k$ from universe $U$ into one of $m$ bucket indices. |
| **Hash Code ($h_1(k)$) vs. Compression Map ($h_2(z)$)** | Two-stage decomposition $h(k) = h_2(h_1(k))$ where $h_1 : U \to \mathbb{Z}_{\ge 0}$ converts an arbitrary key to an integer and $h_2 : \mathbb{Z}_{\ge 0} \to \{0, \dots, m-1\}$ compresses it into the table range. |
| **Collision** | The event where two distinct keys $k_1 \neq k_2$ map to the exact same bucket index: $h(k_1) = h(k_2)$. |
| **Load Factor ($\alpha$)** | The ratio $\alpha = \frac{n}{m}$ of stored elements $n$ to table capacity $m$. In Separate Chaining, $\alpha \ge 0$ is the average chain length; in Open Addressing, $0 \le \alpha \le 1$ is the fraction of occupied slots. |
| **Simple Uniform Hashing Assumption (SUHA)** | The assumption that every key $k \in U$ is equally likely to hash to any of the $m$ slots ($\Pr[h(k) = j] = 1/m$) independently of where other keys hash. |
| **Separate Chaining (Open Hashing)** | Collision resolution where each bucket $T[i]$ points to a singly linked list containing all keys that hash to $i$. |
| **Open Addressing (Closed Hashing)** | Collision resolution where all elements reside directly inside the array $T[0 \dots m-1]$ and collisions are resolved by probing a sequence of alternative slots $h(k, 0), h(k, 1), \dots, h(k, m-1)$. |
| **Linear Probing** | Open addressing with $h(k, i) = (h'(k) + i) \bmod m$; suffers from **Primary Clustering**. |
| **Quadratic Probing** | Open addressing with $h(k, i) = (h'(k) + i^2) \bmod m$; eliminates Primary Clustering, exhibits milder **Secondary Clustering**, and is guaranteed to find an empty slot if $m$ is prime and $\alpha \le 0.5$. |
| **Double Hashing** | Open addressing with $h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod m$, where $h_2(k) \neq 0$ and $\gcd(h_2(k), m) = 1$; eliminates both Primary and Secondary Clustering. |
| **Tombstone (`SLOT_DELETED`)** | A lazy-deletion marker placed in an open-addressed slot when a key is deleted so that searches for keys probed past that slot do not terminate prematurely, while insertions can reuse the slot. |
| **Rehashing** | Resizing a hash table when $\alpha > \alpha_{\max}$ by allocating a new table of prime capacity $m' \approx 2m$ and re-inserting every active key using the new modulus $k \bmod m'$ (purging all tombstones). |

---

## 2. Essential Diagrams and Mathematical Formulas

### 2.1 Graph Applications: Connected Components & BFS Shortest Unweighted Path

```
  BFS LAYER-BY-LAYER WAVEFRONT PROPERTY (Shortest Unweighted Path)
  Layer d=0        Layer d=1           Layer d=2           Layer d=3
    [ s ] ---------> [ u ] ------------> [ x ] ------------> [ dest ]
      |                |                                        ^
      +------------> [ v ] ------------> [ y ] -----------------+
  dist[s] = 0      dist = 1            dist = 2            dist[dest] = 3

  Queue FIFO Invariant: At any moment, the BFS queue contains vertices of at most
  two consecutive distances: [ d, d, ..., d, d+1, d+1, ..., d+1 ].
```

- **BFS Shortest Path Recurrence:** When edge $(u, v)$ discovers an unvisited vertex $v$ (`dist[v] == -1`):
  $$\text{dist}[v] = \text{dist}[u] + 1, \qquad \text{parent}[v] = u$$
- **Number of Edges in a Forest of $k$ Connected Components:** An acyclic undirected graph with $|V|$ vertices and $k$ connected components has exactly $|E| = |V| - k$ edges. Any graph with $k$ connected components satisfies:
  $$|V| - k \le |E| \le \frac{(|V| - k)(|V| - k + 1)}{2}$$

### 2.2 Trie and Suffix Tree Node & Space Bounds

```
  STANDARD TRIE vs. COMPRESSED SUFFIX TREE (Edge Compression via Index Pairs)

  Uncompressed Chain (O(m^2) nodes):       Compressed Suffix Tree Edge (O(1) space):
  (o) --'b'--> (o) --'a'--> (o) --'n'-->   (o) ---- "banana$" [start=0, end=6] ----> [Leaf 0]
```

- **Standard Trie Space & Time ($N$ strings, max length $L$, alphabet size $|\Sigma|$):**
  - Search / Insert / Delete a word of length $L$: **$\Theta(L)$ worst-case time** (independent of $N$!).
  - Worst-case node count: $1 + N \cdot L$ nodes; space **$O(N \cdot L \cdot |\Sigma|)$**.
- **Compressed Suffix Tree Bounds (Text $T\$$ of length $m = |T\$$|):**
  - Number of leaves: **Exactly $m$ leaves** (one per suffix $T[i \dots m-1]$ for $i \in \{0, \dots, m-1\}$).
  - Number of internal nodes (since every internal node has $\ge 2$ children): **At most $m - 1$ internal nodes**.
  - Total nodes: **$\le 2m - 1$ nodes**; total space with `(start, end)` index pairs: **$\Theta(m)$** (built in $O(m)$ time via Ukkonen's algorithm).
  - Pattern matching query for pattern $P$ of length $p = |P|$:
    - Existence / Longest Prefix Match: **$O(p)$ time**.
    - Finding all $occ$ occurrences of $P$ in $T$: **$O(p + occ)$ time** (totally independent of text length $m$!).

### 2.3 Hash Functions, Probing Formulas, and Expected Probe Counts

| Formula Name | Mathematical Expression | Key Constraints / Notes |
| :--- | :--- | :--- |
| **Load Factor** | $\alpha = \dfrac{n}{m}$ | Separate Chaining: $\alpha \ge 0$; Open Addressing: $0 \le \alpha \le 1$ |
| **Division Hash** | $h(k) = k \bmod m$ | $m$ should be **prime**, not close to $2^p$ or $10^p$ |
| **Multiplication Hash** | $h(k) = \lfloor m \cdot ((k \cdot A) \bmod 1) \rfloor$ | $A = \frac{\sqrt{5}-1}{2} \approx 0.6180339887$; $m = 2^p$ works well |
| **Polynomial String Hash** | $h(S) = \left(\sum_{i=0}^{L-1} S[i]\cdot p^{L-1-i}\right) \bmod m$ | Horner's rule: `h = (h * p + c) % m` ($p \in \{31, 53, 131\}$) |
| **Linear Probing** | $h(k, i) = (h'(k) + i) \bmod m$ | $i \in \{0, 1, \dots, m-1\}$; suffers from Primary Clustering |
| **Quadratic Probing** | $h(k, i) = (h'(k) + i^2) \bmod m$ | Guaranteed to find empty slot if **$m$ is prime and $\alpha \le 0.5$** |
| **Double Hashing** | $h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod m$ | Requires **$h_2(k) \neq 0$** and **$\gcd(h_2(k), m) = 1$** (e.g., $R - (k \bmod R)$) |
| **Chaining Expected Probes** | Unsucc: $1 + \alpha$, Succ: $1 + \dfrac{\alpha}{2}$ | Worst case: $\Theta(n)$ if all keys collide in one bucket |
| **Linear Probing Expected Probes** | Unsucc: $\dfrac{1}{2}\left(1 + \dfrac{1}{(1-\alpha)^2}\right)$, Succ: $\dfrac{1}{2}\left(1 + \dfrac{1}{1-\alpha}\right)$ | Degrades quadratically as $\alpha \to 1$ due to Primary Clustering |
| **Double Hashing Expected Probes** | Unsucc: $\dfrac{1}{1 - \alpha}$, Succ: $\dfrac{1}{\alpha}\ln\left(\dfrac{1}{1-\alpha}\right)$ | Matches ideal Uniform Hashing |

---

## 3. Commonly Confused Concepts (Exam Pitfall Clarifications)

### 3.1 BFS vs. DFS for Shortest Unweighted Path

| Question / Misconception | Technical Clarification |
| :--- | :--- |
| *"Both BFS and DFS visit every reachable vertex in $O(V + E)$ time, so can't DFS also find the shortest unweighted path from $s$ to $t$?"* | **NO!** DFS plunges as deeply as possible along the first branch it sees. In a triangle graph with edges $(s, u), (u, t), (s, t)$, if DFS explores $u$ before $t$, it reaches $t$ via the 2-edge path $s \to u \to t$, marks $t$ as visited, and never discovers the 1-edge direct path $s \to t$! Only **BFS** explores vertices in strict non-decreasing order of distance $d = 0, 1, 2, \dots$ using a FIFO queue, guaranteeing that the **first** time $t$ is visited is via a shortest unweighted path. (However, for counting **Connected Components**, *both* BFS and DFS work equally well in $O(V + E)$ time because component membership only depends on reachability, not path length!). |

### 3.2 Standard Trie vs. Suffix Trie vs. Compressed Suffix Tree

| Feature | Standard Trie | Uncompressed Suffix Trie | Compressed Suffix Tree |
| :--- | :--- | :--- | :--- |
| **What Is Stored?** | A dictionary of $N$ distinct strings $\{W_1, \dots, W_N\}$ | All $m$ suffixes of a **single text** $T\$$ of length $m$ | All $m$ suffixes of a **single text** $T\$$ of length $m$ |
| **Edge Labels** | Single character $c \in \Sigma$ | Single character $c \in \Sigma \cup \{\$\}$ | Substring slice `(start, end)` of $T\$$ |
| **Number of Nodes** | $O(N \cdot L)$ | $\Theta(m^2)$ in worst case (e.g., $a^{m/2}b^{m/2}\$$) | **$\le 2m - 1 = \Theta(m)$** always! |
| **Primary Query** | Exact word lookup or prefix autocomplete in $O(|P|)$ | Arbitrary **substring** search in $T$ in $O(|P|)$ | Arbitrary **substring** search, LRS, LCS in $O(|P|)$ |

### 3.3 Primary Clustering (Linear Probing) vs. Secondary Clustering (Quadratic Probing)

- **Primary Clustering (Linear Probing):** Keys with **different** initial hash values (e.g., $h'(k_1) = 4$ and $h'(k_2) = 5$) merge into the **same contiguous block** of occupied slots. Any new key hashing anywhere inside or immediately after the block extends the block further.
- **Secondary Clustering (Quadratic Probing):** Keys with *different* initial hash values follow different quadratic trajectories and do **not** coalesce into contiguous blocks. However, two keys with the **exact same initial hash** ($h'(k_1) = h'(k_2)$) follow the **identical** sequence of quadratic jumps $(h' + 1, h' + 4, h' + 9, \dots)$.
- **Double Hashing:** Eliminates **both** Primary and Secondary Clustering because the step size $h_2(k)$ is computed independently from the starting slot $h_1(k)$.

### 3.4 Why Open Addressing Deletion Requires `DELETED` Tombstones Instead of `EMPTY`

In Open Addressing, a collision during `insert(k2)` causes `k2` to be placed at a later probe index after an occupied slot containing `k1`. If `delete(k1)` resets `k1`'s slot to `EMPTY`, a subsequent `search(k2)` will hit that `EMPTY` slot before reaching `k2` and **falsely conclude that `k2` is not in the table**! Marking `k1`'s slot as `DELETED` (a **Tombstone**) instructs `search` to keep probing past it, while allowing `insert` to reuse the slot once duplicate checking is complete.

### 3.5 Why `realloc` / `memcpy` Cannot Resize a Hash Table Without Rehashing

In a hash table of capacity $m$, every key $k$ resides in bucket $k \bmod m$ (or along a probe path starting at $k \bmod m$). When resized to capacity $m'$, future lookups will compute $k \bmod m'$. Since $k \bmod m \neq k \bmod m'$ in general (e.g., $10 \bmod 7 = 3$, whereas $10 \bmod 17 = 10$), simply copying the array bytes leaves keys at their old indices where `search(k)` in the new table will never find them! Every active key **must** have its hash re-evaluated modulo $m'$.

---

## 4. Consolidated Operation-Complexity Table for Unit 4

Let $|V|$ be vertices, $|E|$ be edges, $N$ be dictionary word count, $L$ be word length, $m$ be text length (for suffix structures) or bucket count (for hash tables), $p = |P|$ be pattern length, $occ$ be occurrence count, and $\alpha = n/m$ be hash table load factor:

| Data Structure / Algorithm | Construction / Init | Search / Query (Average) | Search / Query (Worst-Case) | Insert (Average) | Insert (Worst-Case) | Delete (Average) | Delete (Worst-Case) | Auxiliary Space |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Connected Components (BFS/DFS, Adj List)** | $O(V + E)$ | $O(1)$ (`comp[u]==comp[v]`) | $O(1)$ | — | — | — | — | $\Theta(V)$ |
| **Connected Components (Adj Matrix)** | $\Theta(V^2)$ | $O(1)$ | $O(1)$ | — | — | — | — | $\Theta(V)$ |
| **Unweighted Shortest Path (BFS, Adj List)** | $O(V + E)$ | $O(\delta(s, t))$ path print | $O(V)$ path print | — | — | — | — | $\Theta(V)$ |
| **Standard Trie ($N$ words, alphabet $\Sigma$)** | $O(N \cdot L)$ | $\Theta(L)$ | $\Theta(L)$ | $\Theta(L)$ | $\Theta(L)$ | $\Theta(L)$ | $\Theta(L)$ | $O(N \cdot L \cdot |\Sigma|)$ |
| **Uncompressed Suffix Trie (Text len $m$)** | $O(m^2)$ | $O(p)$ | $O(p)$ | — | — | — | — | $O(m^2 \cdot |\Sigma|)$ |
| **Compressed Suffix Tree (Text len $m$)** | $O(m)$ (Ukkonen) | $O(p + occ)$ | $O(p + occ)$ | — | — | — | — | $\Theta(m)$ |
| **Direct Address Table (Universe $U$)** | $\Theta(|U|)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $\Theta(|U|)$ |
| **Hash Table: Separate Chaining** | $\Theta(m)$ | $\Theta(1 + \alpha)$ | $\Theta(n)$ | $O(1)$ (head) / $\Theta(1+\alpha)$ (upsert) | $\Theta(n)$ | $\Theta(1 + \alpha)$ | $\Theta(n)$ | $\Theta(n + m)$ |
| **Hash Table: Linear Probing ($\alpha < 1$)** | $\Theta(m)$ | $O\left(\frac{1}{1-\alpha}\right)$ | $\Theta(n)$ | $O\left(\frac{1}{(1-\alpha)^2}\right)$ | $\Theta(n)$ | $O\left(\frac{1}{1-\alpha}\right)$ | $\Theta(n)$ | $\Theta(m)$ |
| **Hash Table: Quadratic Probing ($\alpha \le 0.5$)** | $\Theta(m)$ | $O(1)$ | $\Theta(n)$ | $O(1)$ | $\Theta(n)$ | $O(1)$ | $\Theta(n)$ | $\Theta(m)$ |
| **Hash Table: Double Hashing ($\alpha < 1$)** | $\Theta(m)$ | $O\left(\frac{1}{1-\alpha}\right)$ | $\Theta(n)$ | $O\left(\frac{1}{1-\alpha}\right)$ | $\Theta(n)$ | $O\left(\frac{1}{1-\alpha}\right)$ | $\Theta(n)$ | $\Theta(m)$ |
| **Hash Table: Dynamic Rehashing ($m \to 2m$)** | $\Theta(n + m')$ | — | — | **$O(1)$ amortized** | $\Theta(n + m')$ | — | — | $\Theta(m')$ |

---

## 5. Short-Answer Questions (10 Exam Questions)

1. **Q5.1:** In an undirected graph $G = (V, E)$ represented by an adjacency list, what is the exact time and space complexity of finding all connected components, and how many times is `BFS` (or `DFS`) invoked by the outer loop if $G$ has $k$ connected components?
2. **Q5.2:** During a BFS shortest-path search from source $s$, what initial values should be assigned to `dist[v]` and `parent[v]` for all $v \in V$, and how do we reconstruct the shortest path from $s$ to $t$ once BFS finishes?
3. **Q5.3:** Why do every node in a standard Trie carry a boolean flag `isEndOfWord` instead of simply treating leaf nodes as complete words? Give a two-word example that fails without `isEndOfWord`.
4. **Q5.4:** Why must a unique terminal symbol `$` ($\notin \Sigma$) be appended to a string $T$ before constructing its Suffix Tree? Give an example string where a suffix would not have its own leaf node without `$`.
5. **Q5.5:** What is the maximum number of leaves and the maximum number of internal nodes in a compressed Suffix Tree for a string $T\$$ of total length $m$ (including `$`)?
6. **Q5.6:** Define the **Load Factor** $\alpha$ of a hash table. For which collision resolution method can $\alpha$ exceed $1.0$, and why can it never exceed $1.0$ in Open Addressing?
7. **Q5.7:** Why is choosing a table capacity $m = 2^p$ (a power of two) a terrible choice for the Division Method $h(k) = k \bmod m$?
8. **Q5.8:** Why does the naive string hash function $h(S) = \left(\sum_{i=0}^{L-1} S[i]\right) \bmod m$ perform poorly on English text dictionaries? State two distinct reasons.
9. **Q5.9:** Under what two mathematical conditions is Quadratic Probing $h(k, i) = (h'(k) + i^2) \bmod m$ guaranteed to find an empty slot during insertion?
10. **Q5.10:** In Double Hashing $h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod m$, state the two mandatory requirements that the secondary hash function $h_2(k)$ must satisfy.

---

## 6. Conceptual Questions (8 Deep-Thinking Questions)

1. **Q6.1 (BFS Optimality vs. Edge Weights):** Prove why Breadth-First Search (BFS) guarantees finding the shortest path from $s$ to every reachable vertex $v$ when all edges are unweighted (or have identical positive weight $w > 0$), but fails if edges can have different positive weights such as $1$ and $5$. *(Give a 3-vertex counterexample for weighted graphs).*
2. **Q6.2 (Trie Deletion Safety):** When deleting a word $W$ of length $L$ from a standard Trie, explain the three distinct cases that can occur at the terminal node of $W$ and along its upward path toward the root. Why must deletion proceed **bottom-up** (post-order recursion) rather than top-down?
3. **Q6.3 (Suffix Tree Space Linearization):** An uncompressed Suffix Trie for a string of length $m$ can have $\Theta(m^2)$ nodes and $\Theta(m^2)$ characters stored along its edges. Even after compressing non-branching paths into a Suffix Tree with $\le 2m - 1$ nodes, the sum of substring lengths across all edges can still be $\Theta(m^2)$. How does a Suffix Tree achieve **$\Theta(m)$ total memory space**?
4. **Q6.4 (Trie vs. Hash Table Trade-Offs):** Compare a **Standard Trie** against a **Hash Table** for storing a dictionary of $N$ string keys of average length $L$. Under what four specific application requirements is a Trie strictly superior to a Hash Table, and when is a Hash Table superior?
5. **Q6.5 (Primary vs. Secondary Clustering Mechanics):** Explain mathematically why a contiguous cluster of length $L$ in **Linear Probing** has probability $\frac{L+1}{m}$ of growing to length $L+1$ on the next random insertion, whereas **Quadratic Probing** prevents adjacent buckets from merging in this manner. Why does Quadratic Probing still suffer from Secondary Clustering?
6. **Q6.6 (The Open Addressing Tombstone Lifecycle):** Trace the complete lifecycle of a slot in Open Addressing (`EMPTY` $\to$ `OCCUPIED` $\to$ `DELETED` $\to$ `OCCUPIED`). Explain why a table subjected to millions of alternating insertions and deletions can degrade to $\Theta(m)$ time for unsuccessful searches even when the active load factor $\alpha = n/m$ remains below $0.20$, and state how this degradation is cured.
7. **Q6.7 (Quadratic Probing Number-Theoretic Proof):** Prove by contradiction that if $m$ is an odd prime and $0 \le i < j \le \lfloor m/2 \rfloor$, then $(h'(k) + i^2) \bmod m \neq (h'(k) + j^2) \bmod m$. Where specifically does this proof break down if $m$ is a composite number (for example, $m = 12$ with $i = 1, j = 5$)?
8. **Q6.8 (Amortized Analysis of Rehashing):** A student argues: *"Because rehashing copies all $n$ elements into a new table in $\Theta(n)$ time, inserting into a dynamically resized hash table takes $\Theta(n)$ average time per insertion, so $N$ insertions take $\Theta(N^2)$ total time."* Refute this claim using amortized analysis when the table capacity doubles ($m' \approx 2m$) whenever $\alpha > 0.75$.

---

## 7. Algorithm-Writing Questions (5 Pseudocode Problems)

1. **Q7.1 (Shortest Unweighted Path & Cycle Detection via BFS):**
   Write clear pseudocode for an algorithm `BFS-Shortest-Path(G, s, dest)` on an undirected adjacency-list graph $G = (V, E)$ that computes `dist[0..V-1]` and `parent[0..V-1]`, and returns the shortest path from `s` to `dest` as an ordered list of vertices `[s, ..., dest]` (or `EMPTY_LIST` if `dest` is unreachable). State its time and space complexity.
2. **Q7.2 (Trie Prefix Autocomplete — Finding All Words with Given Prefix):**
   Write pseudocode for `Trie-Autocomplete(root, prefix)` that takes the root of a standard Trie and a query string `prefix`, locates the node corresponding to `prefix` in $O(|\text{prefix}|)$ time, and outputs all complete dictionary words stored in the Trie that begin with `prefix` in lexicographical order.
3. **Q7.3 (Suffix Tree Substring Count — Counting All Occurrences of Pattern $P$):**
   Suppose a compressed Suffix Tree has already been built for text $T\$$, where each leaf stores the starting suffix index and each internal node $u$ has been annotated (or can be traversed) to count the number of leaves in its subtree. Write pseudocode for `SuffixTree-PatternSearch(root, T, P)` that determines whether pattern $P[0..p-1]$ occurs as a substring of $T$ and returns all starting indices of $P$ in $T$ in $O(p + occ)$ time.
4. **Q7.4 (Open Addressing Insert & Search with Tombstone Reuse):**
   Write complete pseudocode for `OpenAddress-Insert(T, m, k, v)` and `OpenAddress-Search(T, m, k)` using a generic probe function `Probe(k, i, m)` where each slot `T[idx]` has fields `key`, `value`, and `state` $\in \{\text{EMPTY}, \text{OCCUPIED}, \text{DELETED}\}$. Ensure `OpenAddress-Insert` properly reuses the first `DELETED` slot while preventing duplicate keys!
5. **Q7.5 (Dynamic Rehashing Algorithm for Open Addressing):**
   Write complete pseudocode for `Rehash-OpenTable(H)` that resizes an open-addressed hash table `H` from capacity $m$ to $m' = \text{NextPrime}(2m)$, re-inserts all active `OCCUPIED` entries using the new capacity $m'$, and discards all `DELETED` tombstones.

---

## 8. C Programming Questions (5 Coding Problems)

1. **Q8.1 (C11 Graph Connected Components & Shortest Path):**
   Write two C11 functions operating on an adjacency-list `Graph`:
   - `int count_connected_components_bfs(const Graph *g, int *comp_id)` that labels every vertex `0 .. V-1` with its 1-based component ID and returns the total number of connected components $k$.
   - `bool bfs_shortest_path(const Graph *g, int src, int dest, int *dist, int *parent)` that computes the shortest unweighted path from `src` to `dest`.
2. **Q8.2 (C11 Trie Recursive Deletion & Prefix Word Counter):**
   Write a C11 implementation of a lowercase English `TrieNode` that maintains `bool is_end_of_word` and `int prefix_count` (the number of words in the subtree of that node). Implement `trie_insert`, `int trie_count_prefix(const TrieNode *root, const char *prefix)` (in $O(|P|)$ time without traversing the subtree!), and `TrieNode *trie_delete_rec(TrieNode *curr, const char *word, size_t depth, bool *deleted)` that unmarks the word, decrements `prefix_count` along the path, and frees any nodes that become completely unused.
3. **Q8.3 (C11 Polynomial Rolling Hash & Separate Chaining Word Frequency Counter):**
   Write a C11 function `uint32_t poly_hash(const char *s, uint32_t m)` using Horner's rule with radix $p = 31$, and implement `void word_freq_increment(WordFreqTable *ht, const char *word)` that inserts `word` with frequency `1` if new, or increments `curr->freq++` in-place if `word` already exists in the Separate Chaining table.
4. **Q8.4 (C11 Double Hashing Insert with Tombstones):**
   Write a C11 function `bool dh_insert(DoubleHashTable *ht, int key, int value)` for a Double Hashing table with primary hash $h_1(k) = k \bmod m$ and secondary hash $h_2(k) = R - (k \bmod R)$, supporting in-place value updates for existing keys and reuse of `DELETED` tombstone slots.
5. **Q8.5 (C11 Dynamic Rehashing for Linear Probing):**
   Write a C11 function `void lp_rehash_table(DHSlot **table_ptr, int *m_ptr, int *n_ptr, int new_m)` that dynamically allocates a new Linear Probing array of size `new_m`, re-hashes and re-inserts all `OCCUPIED` entries from the old table while purging `DELETED` tombstones, frees the old array, and updates `*table_ptr`, `*m_ptr`, and `*n_ptr`.

---

## 9. Worked Problem-Solving Questions (Step-by-Step Traces)

### 9.1 Problem 1: Connected Components & BFS Shortest Unweighted Path Trace

**Problem Statement:**
Consider an undirected graph $G = (V, E)$ with $V = \{0, 1, 2, 3, 4, 5, 6, 7\}$ (8 vertices) and edge set:
$$E = \{(0, 1),\ (0, 2),\ (1, 3),\ (2, 3),\ (3, 4),\ (5, 6),\ (6, 7)\}$$
(Assume adjacency lists are ordered in ascending vertex order).
1. Trace the Connected Components algorithm across all vertices $0 \dots 7$ and state the connected components partition.
2. Trace BFS starting from source $s = 0$ to find the shortest unweighted path to target $dest = 4$, showing the FIFO queue, `dist[]`, and `parent[]` arrays at each step, and reconstruct the shortest path.

#### Step-by-Step Solution:

```
          Component 1                                Component 2
          [1] ----- [3] ----- [4]                [5] --- [6] --- [7]
         /           |
       [0] --------- [2]
```

**Part 1 — Connected Components Trace:**
- Initialize `comp_id[0..7] = [-1, -1, -1, -1, -1, -1, -1, -1]`, `count = 0`.
- **Outer Loop `v = 0` (`comp_id[0] == -1`):**
  - Increment `count = 1`. Start BFS from `0`:
  - Visit `0` $\to$ neighbors `1, 2` $\to$ neighbor `3` $\to$ neighbor `4`.
  - All of $\{0, 1, 2, 3, 4\}$ receive `comp_id = 1`.
- **Outer Loop `v = 1, 2, 3, 4`:** All already have `comp_id != -1` $\implies$ skipped in $O(1)$.
- **Outer Loop `v = 5` (`comp_id[5] == -1`):**
  - Increment `count = 2`. Start BFS from `5`:
  - Visit `5` $\to$ neighbor `6` $\to$ neighbor `7`.
  - All of $\{5, 6, 7\}$ receive `comp_id = 2`.
- **Outer Loop `v = 6, 7`:** Already visited $\implies$ skipped.
- **Final Result:** **$k = 2$ Connected Components:** $C_1 = \{0, 1, 2, 3, 4\}$ and $C_2 = \{5, 6, 7\}$.

**Part 2 — BFS Shortest Path Trace from $s = 0$ to $dest = 4$:**

| Step | Dequeued $u$ | Unvisited Neighbors Enqueued | Queue State (Front $\to$ Rear) | `dist[0..4]` | `parent[0..4]` |
| :---: | :---: | :--- | :--- | :--- | :--- |
| Init | — | Start at $s = 0$ | `[ 0 ]` | `[0, -1, -1, -1, -1]` | `[-1, -1, -1, -1, -1]` |
| 1 | `0` | `1` ($\text{dist}=1, \text{p}=0$), `2` ($\text{dist}=1, \text{p}=0$) | `[ 1, 2 ]` | `[0,  1,  1, -1, -1]` | `[-1,  0,  0, -1, -1]` |
| 2 | `1` | `3` ($\text{dist}=2, \text{p}=1$) | `[ 2, 3 ]` | `[0,  1,  1,  2, -1]` | `[-1,  0,  0,  1, -1]` |
| 3 | `2` | *(0 and 3 already visited)* | `[ 3 ]` | `[0,  1,  1,  2, -1]` | `[-1,  0,  0,  1, -1]` |
| 4 | `3` | `4` ($\text{dist}=3, \text{p}=3$) | `[ 4 ]` | `[0,  1,  1,  2,  3]` | `[-1,  0,  0,  1,  3]` |
| 5 | `4` | Reached `dest == 4` (Stop) | `[ ]` | `[0,  1,  1,  2,  3]` | `[-1,  0,  0,  1,  3]` |

- **Backtracking `parent[]` from `dest = 4`:**
  $$4 \xrightarrow{\text{parent}[4]=3} 3 \xrightarrow{\text{parent}[3]=1} 1 \xrightarrow{\text{parent}[1]=0} 0 \xrightarrow{\text{parent}[0]=-1} \text{STOP}$$
- **Reversing the Backtracked Sequence:** Shortest path is **$0 \to 1 \to 3 \to 4$** with length $\text{dist}[4] = \mathbf{3}$ edges.

---

### 9.2 Problem 2: Standard Trie Insertion, Prefix Search, and Bottom-Up Deletion Trace

**Problem Statement:**
1. Construct a standard Trie by inserting the five words: `{"cat", "car", "cart", "dog", "dot"}`. Mark every `isEndOfWord == true` node with `*`.
2. Trace the deletion of:
   - **(a)** `"car"` (a word that is a proper prefix of `"cart"`).
   - **(b)** `"cart"` (after `"car"` has already been deleted, where `"cat"` still shares prefix `"ca"`).

#### Step-by-Step Solution:

**1. Initial Trie After Inserting `{"cat", "car", "cart", "dog", "dot"}`:**

```
                        (root)
                        /    \
                      'c'    'd'
                      /        \
                    (c)        (d)
                    /            \
                  'a'            'o'
                  /                \
                (a)                (o)
               /   \              /   \
             't'   'r'          'g'   't'
             /       \          /       \
          (t)*      (r)*     (g)*      (t)*
                     |
                    't'
                     |
                    (t)*
```

**2(a). Deleting `"car"` (Internal Prefix Node Case):**
- Traverse root $\to$ `'c'` $\to$ `'a'` $\to$ `'r'`.
- At node `(r)*`, we find `isEndOfWord == true`. We set `isEndOfWord = false`.
- **Check children of `(r)`:** Does `(r)` have any non-NULL children? **Yes!** `children['t']` points to `(t)*` (for `"cart"`).
- Therefore, **zero nodes are `free()`d**! Only the `*` flag on `(r)` is cleared. `"cart"` and `"cat"` remain completely intact.

**2(b). Deleting `"cart"` Next (Partial Branch Pruning Case):**
- Traverse root $\to$ `'c'` $\to$ `'a'` $\to$ `'r'` $\to$ `'t'`.
- At leaf `(t)*` (depth 4): set `isEndOfWord = false`. Since `(t)` has **no children**, `free(t)` and return `NULL` to parent `(r)`.
- Unwind to `(r)` (depth 3): `r->children['t']` becomes `NULL`. Since `(r)` now has `isEndOfWord == false` (from step 2a!) and **no children**, `free(r)` and return `NULL` to parent `(a)`.
- Unwind to `(a)` (depth 2): `a->children['r']` becomes `NULL`. Check `(a)`: does `(a)` have any remaining children? **Yes!** `a->children['t']` still points to `(t)*` for the word `"cat"`!
- Pruning **stops** at node `(a)`. Nodes `(t)` and `(r)` of `"cart"` were freed, while `"cat"` remains untouched!

---

### 9.3 Problem 3: Suffix Tree Construction and Pattern Matching for `"banana$"`

**Problem Statement:**
Given the string $T = \mathtt{"banana"}$:
1. List all $7$ suffixes of $T\$ = \mathtt{"banana\$"}$ with their starting indices $0 \dots 6$.
2. Explain why the terminal `$` is necessary by examining the suffix at index $5$ (`"a"`).
3. Draw the compressed **Suffix Tree** for `"banana$"`, showing all compressed edge labels, internal branching nodes, and leaf suffix indices.
4. Trace how the Suffix Tree finds all occurrences of pattern $P = \mathtt{"ana"}$.

#### Step-by-Step Solution:

**1. All 7 Suffixes of $T\$ = \mathtt{"banana\$"}$ ($m = 7$):**
- Index `0`: `"banana$"`
- Index `1`: `"anana$"`
- Index `2`: `"nana$"`
- Index `3`: `"ana$"`
- Index `4`: `"na$"`
- Index `5`: `"a$"`
- Index `6`: `"$"`

**2. Why `$` Is Necessary:**
Without `$`, suffix `5` (`"a"`) is a proper prefix of suffix `3` (`"ana"`) and suffix `1` (`"anana"`), and suffix `4` (`"na"`) is a proper prefix of suffix `2` (`"nana"`). Without `$`, suffixes `5`, `3`, and `4` would end at **internal nodes** rather than distinct leaves! Appending `$` ensures all $7$ suffixes terminate at $7$ distinct leaves.

**3. Compressed Suffix Tree for `"banana$"`:**

```
                                    [Root]
                     /        |               |          \
            "banana$"         "$"             "a"        "na"
           (idx 0..6)      (idx 6..6)      (idx 1..1)   (idx 2..3)
               /              |               |            \
          [Leaf 0]        [Leaf 6]         (Node A)      (Node B)
                                           /      \       /     \
                                         "$"     "na$"  "$"    "na$"
                                          |        |     |       |
                                      [Leaf 5]  (Node C)[Leaf 4][Leaf 2]
                                                /      \
                                              "$"     "na$"
                                               |        |
                                           [Leaf 3]  [Leaf 1]
```
- **Total Leaves:** $7$ (`[Leaf 0]` through `[Leaf 6]`).
- **Internal Branching Nodes (excluding root):** $3$ (`Node A` representing prefix `"a"`, `Node C` representing `"ana"`, and `Node B` representing `"na"`). Note that $7 + 3 + 1 = 11 \le 2(7) - 1 = 13$.

**4. Pattern Search Trace for $P = \mathtt{"ana"}$ ($p = 3$):**
- Start at `[Root]`, match `P[0] = 'a'` $\to$ take edge `"a"` to `Node A` ($1$ character matched).
- From `Node A`, match `P[1..2] = "na"` $\to$ take edge `"na$"` ($2$ characters `"na"` matched; pattern $P = \mathtt{"ana"}$ is **exhausted** along the edge leading into `Node C`!).
- Every leaf in the subtree of `Node C` is an occurrence of `"ana"`:
  - DFS from `Node C` visits `[Leaf 3]` and `[Leaf 1]`.
- **Result:** Pattern `"ana"` occurs **$2$ times**, at starting indices **`1`** and **`3`** in `"banana"`, found in $O(p + occ) = O(3 + 2)$ time!

---

### 9.4 Problem 4: Master Hash Table Collision Trace (Chaining vs. Linear vs. Quadratic vs. Double Hashing)

**Problem Statement:**
Given the sequence of 7 keys $K = \langle 12,\ 23,\ 34,\ 45,\ 56,\ 26,\ 37 \rangle$:
1. Insert $K$ into a **Separate Chaining** table of size $m = 5$ with $h(k) = k \bmod 5$ (using head insertion).
2. Insert $K$ into an **Open Addressing** table of size $m = 11$ with primary hash $h_1(k) = k \bmod 11$ using:
   - **(a) Linear Probing:** $h(k, i) = (h_1(k) + i) \bmod 11$
   - **(b) Quadratic Probing:** $h(k, i) = (h_1(k) + i^2) \bmod 11$
   - **(c) Double Hashing:** $h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod 11$, where $h_2(k) = 7 - (k \bmod 7)$
3. In the Linear Probing table from 2(a), trace `Delete(23)` followed by `Search(45)` and `Insert(67)`.

#### Step-by-Step Solution:

**1. Separate Chaining ($m = 5, h(k) = k \bmod 5$):**
- $12 \bmod 5 = 2 \implies [2]: 12$
- $23 \bmod 5 = 3 \implies [3]: 23$
- $34 \bmod 5 = 4 \implies [4]: 34$
- $45 \bmod 5 = 0 \implies [0]: 45$
- $56 \bmod 5 = 1 \implies [1]: 56$
- $26 \bmod 5 = 1 \implies [1]: 26 \to 56$
- $37 \bmod 5 = 2 \implies [2]: 37 \to 12$

```
  [0]: 45 -> NULL
  [1]: 26 -> 56 -> NULL
  [2]: 37 -> 12 -> NULL
  [3]: 23 -> NULL
  [4]: 34 -> NULL
```

**2. Open Addressing Comparison ($m = 11, h_1(k) = k \bmod 11$):**
Notice that the first five keys $\langle 12, 23, 34, 45, 56 \rangle$ all have **$h_1(k) = k \bmod 11 = 1$**, while $26 \bmod 11 = 4$ and $37 \bmod 11 = 4$!

| Key $k$ | $h_1(k)$ | $h_2(k) = 7 - (k \bmod 7)$ | **(a) Linear Probes** $(h_1 + i) \bmod 11$ | **(b) Quadratic Probes** $(h_1 + i^2) \bmod 11$ | **(c) Double Hashing Probes** $(h_1 + i\cdot h_2) \bmod 11$ |
| :---: | :---: | :---: | :--- | :--- | :--- |
| `12` | `1` | $7 - 5 = 2$ | `1` $\to$ **`[1]`** | `1` $\to$ **`[1]`** | `1` $\to$ **`[1]`** |
| `23` | `1` | $7 - 2 = 5$ | `1` $\to$ `2` $\to$ **`[2]`** | `1` $\to$ $(1+1)=\mathbf{2}$ $\to$ **`[2]`** | `1` $\to$ $(1+5)=\mathbf{6}$ $\to$ **`[6]`** |
| `34` | `1` | $7 - 1 = 6$ | `1`$\to$`2`$\to$`3` $\to$ **`[3]`** | `1`$\to$`2`$\to$$(1+4)=\mathbf{5}$ $\to$ **`[5]`** | `1` $\to$ $(1+6)=\mathbf{7}$ $\to$ **`[7]`** |
| `45` | `1` | $7 - 3 = 4$ | `1`$\to$`2`$\to$`3`$\to$`4` $\to$ **`[4]`** | `1`$\to$`2`$\to$`5`$\to$$(1+9)=\mathbf{10}$ $\to$ **`[10]`** | `1` $\to$ $(1+4)=\mathbf{5}$ $\to$ **`[5]`** |
| `56` | `1` | $7 - 0 = 7$ | `1`$\to$`2`$\to$`3`$\to$`4`$\to$`5` $\to$ **`[5]`** | `1`$\to$`2`$\to$`5`$\to$`10`$\to$$(1+16)\bmod 11=\mathbf{6}$ $\to$ **`[6]`** | `1` $\to$ $(1+7)=\mathbf{8}$ $\to$ **`[8]`** |
| `26` | `4` | $7 - 5 = 2$ | `4`$\to$`5`$\to$`6` $\to$ **`[6]`** | `4` $\to$ **`[4]`** | `4` $\to$ **`[4]`** |
| `37` | `4` | $7 - 2 = 5$ | `4`$\to$`5`$\to$`6`$\to$`7` $\to$ **`[7]`** | `4`$\to$$(4+1)=5$$\to$$(4+4)=\mathbf{8}$ $\to$ **`[8]`** | `4` $\to$ $(4+5)=\mathbf{9}$ $\to$ **`[9]`** |

**Final Arrays `[0 .. 10]` After All 7 Insertions:**
- **(a) Linear Probing:**
  `[ 0: . | 1: 12 | 2: 23 | 3: 34 | 4: 45 | 5: 56 | 6: 26 | 7: 37 | 8: . | 9: . | 10: . ]`
  *(A single massive 7-element primary cluster `[1..7]` formed!)*
- **(b) Quadratic Probing:**
  `[ 0: . | 1: 12 | 2: 23 | 3: . | 4: 26 | 5: 34 | 6: 56 | 7: . | 8: 37 | 9: . | 10: 45 ]`
  *(Notice `26` landed cleanly at its home slot `[4]` on the first try because `45` had jumped quadratically to `[10]`!)*
- **(c) Double Hashing:**
  `[ 0: . | 1: 12 | 2: . | 3: . | 4: 26 | 5: 45 | 6: 23 | 7: 34 | 8: 56 | 9: 37 | 10: . ]`
  *(Every single colliding key with $h_1(k)=1$ resolved on its very first retry $i=1$ because their step sizes $h_2(k) \in \{5, 6, 4, 7\}$ were all distinct!)*

**3. Tombstone Trace in Linear Probing Table 2(a):**
- **`Delete(23)`:** Key `23` is at slot `[2]`. Mark `T[2] = DEL`:
  `[ 0: . | 1: 12 | 2: DEL | 3: 34 | 4: 45 | 5: 56 | 6: 26 | 7: 37 | 8: . | 9: . | 10: . ]`
- **`Search(45)`:** $h_1(45) = 1$. Probes `1` (12) $\to$ `2` (`DEL` — **skip tombstone!**) $\to$ `3` (34) $\to$ `4` (**45 == 45: FOUND** in 4 probes).
- **`Insert(67)`:** $h_1(67) = 67 \bmod 11 = 1$.
  - $i=0$: `T[1]` is `12`.
  - $i=1$: `T[2]` is `DEL` $\implies$ save `first_tombstone = 2`, continue probing `[3..8]` until `T[8] == EMPTY` to verify `67` is not already present.
  - Place `67` into `T[2]`:
    `[ 0: . | 1: 12 | 2: 67 | 3: 34 | 4: 45 | 5: 56 | 6: 26 | 7: 37 | 8: . | 9: . | 10: . ]`

---

### 9.5 Problem 5: Dynamic Rehashing Step-by-Step Trace ($m = 5 \to m' = 11$)

**Problem Statement:**
A Linear Probing hash table starts with capacity $m = 5$, $h(k) = k \bmod 5$, and rehash threshold $\alpha_{\max} = 0.70$ (meaning a rehash to $m' = \text{next\_prime}(2m) = 11$ is triggered whenever inserting a new key would cause $\frac{n}{m} > 0.70$, i.e., on the 4th active key since $4/5 = 0.80 > 0.70$).
Trace the following sequence of operations:
`Insert(12)`, `Insert(7)`, `Insert(22)`, `Delete(7)`, `Insert(17)`, `Insert(27)`.

#### Step-by-Step Solution:

1. **`Insert(12)` ($n = 1, \alpha = 1/5 = 0.20 \le 0.70$):**
   $12 \bmod 5 = 2 \implies$ `T[2] = 12`.
   Table ($m=5$): `[ 0: . | 1: . | 2: 12 | 3: . | 4: . ]`
2. **`Insert(7)` ($n = 2, \alpha = 2/5 = 0.40 \le 0.70$):**
   $7 \bmod 5 = 2$ (hit 12) $\to$ `T[3] = 7`.
   Table ($m=5$): `[ 0: . | 1: . | 2: 12 | 3: 7 | 4: . ]`
3. **`Insert(22)` ($n = 3, \alpha = 3/5 = 0.60 \le 0.70$):**
   $22 \bmod 5 = 2$ (hit 12) $\to$ `3` (hit 7) $\to$ `T[4] = 22`.
   Table ($m=5$): `[ 0: . | 1: . | 2: 12 | 3: 7 | 4: 22 ]`
4. **`Delete(7)` ($n = 2, \text{tombstones} = 1, \alpha = 2/5 = 0.40$):**
   Key `7` at `T[3]` becomes `DEL`.
   Table ($m=5$): `[ 0: . | 1: . | 2: 12 | 3: DEL | 4: 22 ]`
5. **`Insert(17)` ($n$ becomes $3$, $\alpha = 3/5 = 0.60 \le 0.70$ — NO rehash yet!):**
   $17 \bmod 5 = 2$ (hit 12) $\to$ `3` (`DEL` $\implies$ `first_tombstone = 3`) $\to$ `4` (hit 22) $\to$ `0` (`EMPTY`). Reuses `T[3] = 17`!
   Table ($m=5$): `[ 0: . | 1: . | 2: 12 | 3: 17 | 4: 22 ]`
6. **`Insert(27)` (Would make $n = 4 \implies \alpha = 4/5 = 0.80 > 0.70 \implies$ REHASH TRIGGERED!):**
   - Allocate new table of capacity $m' = \text{next\_prime}(2 \times 5) = \mathbf{11}$ initialized to `EMPTY`.
   - Scan old table `[0..4]` and re-insert each `OCCUPIED` key using $k \bmod 11$:
     - `12` (from old `[2]`): $12 \bmod 11 = \mathbf{1} \implies \text{New}[1] = 12$
     - `17` (from old `[3]`): $17 \bmod 11 = \mathbf{6} \implies \text{New}[6] = 17$
     - `22` (from old `[4]`): $22 \bmod 11 = \mathbf{0} \implies \text{New}[0] = 22$
   - Now insert triggering key `27` into new table:
     - $27 \bmod 11 = \mathbf{5} \implies \text{New}[5] = 27$.
   - **Final Rehashed Table ($m' = 11, n = 4, \alpha = 4/11 \approx 0.36$):**
     `[ 0: 22 | 1: 12 | 2: . | 3: . | 4: . | 5: 27 | 6: 17 | 7: . | 8: . | 9: . | 10: . ]`

---

## 10. Complete Answer Keys with Detailed Explanations

### 10.1 Answers to Short-Answer Questions (Section 5)

- **A5.1:** Time complexity is **$\Theta(V + E)$** and auxiliary space is **$\Theta(V)$** (for the `visited[]`/`comp_id[]` array and BFS queue or DFS recursion stack). The outer loop checks all $|V|$ vertices, but only invokes `BFS`/`DFS` on an unvisited vertex—which happens **exactly $k$ times** (once per connected component).
- **A5.2:** Initialize `dist[v] = -1` (or `INT_MAX` for $\infty$) and `parent[v] = -1` for all $v \in V$, with `dist[s] = 0`. Once BFS finishes, if `dist[t] != -1`, reconstruct the shortest path by following `curr = parent[curr]` backward from `t` to `s` and reversing the sequence (or pushing onto a stack).
- **A5.3:** Because one valid dictionary word can be a **proper prefix** of another longer dictionary word! For example, if both `"car"` and `"cart"` are inserted, the node for `'r'` is an internal non-leaf node (since it has child `'t'`). Without `isEndOfWord == true` on `'r'`, we could not tell whether `"car"` is stored in the dictionary or merely a prefix of `"cart"`.
- **A5.4:** Without `$`, any suffix that is a prefix of another suffix ends inside an internal node rather than at its own leaf. For example, in $T = \mathtt{"aba"}$, the suffix `"a"` (index 2) is a prefix of `"aba"` (index 0); appending `$` yields `"aba$"`, where `"a$"` branches off from `"aba$"` at the second character (`'$'` vs. `'b'`).
- **A5.5:** For $T\$$ of length $m$, the compressed Suffix Tree has **exactly $m$ leaves** (one per suffix $0 \dots m-1$) and **at most $m - 1$ internal nodes** (since every internal node has at least $2$ children), giving at most **$2m - 1$ total nodes**.
- **A5.6:** $\alpha = n/m$, where $n$ is the number of stored elements and $m$ is the number of buckets/slots. In **Separate Chaining**, $\alpha$ can exceed $1.0$ because colliding items are stored in dynamically allocated linked list nodes outside the bucket array. In **Open Addressing**, each of the $m$ array slots holds at most $1$ item, so $n \le m \implies 0 \le \alpha \le 1.0$.
- **A5.7:** Because $k \bmod 2^p = k \ \& \ (2^p - 1)$, which depends **only on the lowest $p$ bits** of key $k$ and ignores all upper bits! If keys share low-order bit patterns (such as 8- or 16-byte aligned memory addresses), all keys collide into a tiny fraction of buckets.
- **A5.8:** (1) Addition is commutative, so **all anagrams** (e.g., `"stop"`, `"tops"`, `"pots"`) sum to the exact same integer and collide in every table size $m$. (2) For short strings of length $L$, the sum of ASCII values spans a very narrow range, leaving most buckets of a large table unreachable.
- **A5.9:** (1) The table capacity $m$ must be a **prime number**, and (2) the load factor must satisfy **$\alpha = n/m \le 0.5$** (the table is at least half empty).
- **A5.10:** (1) $h_2(k)$ must **never evaluate to $0$** ($h_2(k) \in \{1, \dots, m-1\}$, otherwise step size is $0 \implies$ infinite loop on $h_1(k)$), and (2) $h_2(k)$ must be **relatively prime (coprime) to $m$** ($\gcd(h_2(k), m) = 1$) so the probe sequence visits all $m$ slots.

---

### 10.2 Answers to Conceptual Questions (Section 6)

- **A6.1 (BFS Optimality vs. Edge Weights):**
  - *Why BFS works on unweighted graphs:* By induction on queue operations, the BFS FIFO queue maintains the invariant that vertices are dequeued in non-decreasing order of unweighted edge count $d = 0, 1, 2, \dots$ from $s$. When $v$ is first discovered from $u$, no path of length $< \text{dist}[u] + 1$ can exist.
  - *Counterexample on weighted graphs:* Let $V = \{s, u, t\}$ with weighted edges $w(s, t) = 10$, $w(s, u) = 1$, and $w(u, t) = 1$. Starting from $s$, BFS inspects neighbor $t$ directly via the 1-edge path $(s, t)$ of weight $10$, marks $t$ as visited, and never updates $t$ when the 2-edge path $s \to u \to t$ of total weight $1 + 1 = 2 < 10$ arrives! (Weighted shortest paths require **Dijkstra's Algorithm** with a Min-Priority Queue).
- **A6.2 (Trie Deletion Safety):**
  - *Case 1 ($W$ is a prefix of a longer word, e.g., deleting `"car"` when `"cart"` exists):* The terminal node of $W$ has non-NULL children. We only set `isEndOfWord = false` and free no nodes.
  - *Case 2 ($W$ shares a prefix with another word, e.g., deleting `"cart"` when `"cat"` exists):* We free nodes from the bottom leaf upward until reaching the lowest ancestor that either has `isEndOfWord == true` or has another non-NULL child (`'t'` in `"cat"`).
  - *Case 3 ($W$ shares no characters with any other word):* All nodes of $W$ up to the children of `root` are freed (never freeing `root` itself).
  - *Why bottom-up:* You cannot know whether an ancestor node still has remaining children after deleting $W$'s suffix until the recursive deletion of the child subtree finishes and returns `NULL`!
- **A6.3 (Suffix Tree Space Linearization):**
  Instead of storing raw character strings (like `"banana$"`) inside each edge struct, each edge stores only **two integers**: `(start_index, end_index)` pointing into the single immutable global text array $T\$$! Because there are at most $2m - 1$ nodes and at most $2m - 2$ edges, storing two `int` indices per edge uses $O(1)$ space per edge, reducing total Suffix Tree space from $\Theta(m^2)$ to **$\Theta(m)$**!
- **A6.4 (Trie vs. Hash Table Trade-Offs):**
  - *When a Trie is strictly superior:* (1) **Prefix-based queries** (autocomplete, `startsWith`, counting words with prefix $P$ in $O(|P|)$ time—impossible in a hash table without scanning all $N$ buckets!); (2) **Lexicographical / alphabetical sorted iteration** in $O(N \cdot L)$ time; (3) **Deterministic worst-case $\Theta(L)$ lookup** with zero collisions, no hash function computation, and no worst-case $\Theta(N)$ rehash spikes (critical in hard real-time routers/avionics); (4) **Longest-Prefix Matching (LPM)** (e.g., IP routing tables).
  - *When a Hash Table is superior:* When only exact-match `get(k)`/`put(k, v)`/`remove(k)` queries are needed and memory footprint or CPU cache locality is critical (since standard Trie nodes with 26 or 256 pointers consume significantly more memory than flat hash table buckets).
- **A6.5 (Primary vs. Secondary Clustering Mechanics):**
  In Linear Probing, a contiguous cluster occupying $L$ slots $[a \dots a+L-1]$ grows to length $L+1$ whenever the next inserted key has home hash $h'(k) \in \{a, a+1, \dots, a+L-1, a+L\}$, which has probability $\frac{L+1}{m}$ under SUHA. In Quadratic Probing, a key landing at $a$ probes $a + 1, a + 4, a + 9$, while a key landing at $a + 1$ probes $a + 2, a + 5, a + 10$—their trajectories do not merge into a single contiguous block. However, if two keys have the **exact same** initial hash $h'(k_1) = h'(k_2) = a$, both evaluate $(a + i^2) \bmod m$ for all $i$, colliding at every probe step (**Secondary Clustering**).
- **A6.6 (The Open Addressing Tombstone Lifecycle):**
  A slot starts `EMPTY`, becomes `OCCUPIED` on insertion, becomes `DELETED` (Tombstone) on deletion, and can become `OCCUPIED` again when a future insertion reuses the tombstone. However, `search(k)` for a non-existent key **can only stop at `EMPTY`**, never at `DELETED`! If millions of insertions and deletions turn almost every non-occupied slot into `DELETED` so zero `EMPTY` slots remain, every unsuccessful search (and every insertion duplicate-check!) must inspect all $m$ slots in $\Theta(m)$ time even when $\alpha = 0.10$. This is cured by **Rehashing** whenever $\frac{n + \text{tombstones}}{m} > 0.75$, which resets all non-occupied slots to fresh `EMPTY`.
- **A6.7 (Quadratic Probing Number-Theoretic Proof):**
  If $(h'(k) + i^2) \equiv (h'(k) + j^2) \pmod m$ for $0 \le i < j \le \lfloor m/2 \rfloor$, then $m \mid (j - i)(j + i)$. When $m$ is prime, Euclid's Lemma requires $m \mid (j - i)$ or $m \mid (j + i)$, which is impossible since $1 \le j - i < j + i \le m - 1$. When $m$ is **composite**, Euclid's Lemma fails because divisors of $m$ can split across $(j - i)$ and $(j + i)$! For $m = 12$ and $i = 1, j = 5 \le 6$: $(5 - 1)(5 + 1) = 4 \times 6 = 24 \equiv 0 \pmod{12}$, so $1^2 \equiv 5^2 \equiv 1 \pmod{12}$ even though $12 \nmid 4$ and $12 \nmid 6$!
- **A6.8 (Amortized Analysis of Rehashing):**
  Rehashing does **not** happen on every insertion; it only happens when $n$ doubles ($n = 1, 2, 4, 8, \dots, 2^k$). Across a sequence of $N$ insertions, the $N$ normal probes cost $O(N)$ expected time, and the rehash operations copy $1 + 2 + 4 + \dots + 2^{\lfloor \log_2 N \rfloor} < 2N$ elements in total. Thus, the total cost of $N$ insertions is $O(N) + O(2N) = O(N)$, making the **amortized cost per insertion $\frac{O(N)}{N} = O(1)$**, not $\Theta(N)$!

---

### 10.3 Solutions to Algorithm-Writing Questions (Section 7)

#### Solution 7.1: `BFS-Shortest-Path(G, s, dest)`

```text
ALGORITHM BFS-Shortest-Path(G, s, dest)
Input: Adjacency-list graph G = (V, E), source vertex s, target vertex dest
Output: Ordered list of vertices on a shortest path from s to dest (or EMPTY_LIST)

1. for v <- 0 to |V| - 1 do
2.     dist[v] <- -1
3.     parent[v] <- -1
4.
5. Q <- CreateEmptyQueue()
6. dist[s] <- 0
7. Enqueue(Q, s)
8.
9. while not IsEmpty(Q) do
10.    u <- Dequeue(Q)
11.    if u == dest then
12.        break
13.    for each neighbor v in G.adj[u] do
14.        if dist[v] == -1 then
15.            dist[v] <- dist[u] + 1
16.            parent[v] <- u
17.            Enqueue(Q, v)
18.
19. if dist[dest] == -1 then
20.    return EMPTY_LIST  // dest is unreachable from s
21.
22. path <- CreateEmptyList()
23. curr <- dest
24. while curr != -1 do
25.    Prepend(path, curr)
26.    curr <- parent[curr]
27. return path
```
- **Complexity:** Time $O(V + E)$, Auxiliary Space $\Theta(V)$.

#### Solution 7.2: `Trie-Autocomplete(root, prefix)`

```text
ALGORITHM Trie-Autocomplete(root, prefix)
Input: Trie root pointer, query string prefix[0 .. p-1]
Output: Prints all complete words in the Trie starting with prefix in alphabetical order

1. curr <- root
2. for i <- 0 to length(prefix) - 1 do
3.     idx <- Ord(prefix[i]) - Ord('a')
4.     if curr.children[idx] == NULL then
5.         return  // No words in Trie start with this prefix
6.     curr <- curr.children[idx]
7.
8. buffer <- CopyString(prefix)
9. Trie-Collect-DFS(curr, buffer)

ALGORITHM Trie-Collect-DFS(node, buffer)
1. if node == NULL then return
2. if node.isEndOfWord == true then
3.     Print(buffer)
4. for idx <- 0 to ALPHABET_SIZE - 1 do
5.     if node.children[idx] != NULL then
6.         ch <- CharFromIndex(idx)
7.         AppendChar(buffer, ch)
8.         Trie-Collect-DFS(node.children[idx], buffer)
9.         RemoveLastChar(buffer)  // Backtrack
```

#### Solution 7.3: `SuffixTree-PatternSearch(root, T, P)`

```text
ALGORITHM SuffixTree-PatternSearch(root, T, P)
Input: Suffix tree root, text T[0..m-1], pattern P[0..p-1]
Output: List of all starting indices of P in T

1. curr <- root
2. i <- 0  // Current index in pattern P
3. while i < p do
4.     edge <- FindOutgoingEdgeStartingWith(curr, P[i])
5.     if edge == NULL then
6.         return EMPTY_LIST  // Pattern P does not occur in T
7.     // Compare characters along edge label T[edge.start .. edge.end]
8.     k <- edge.start
9.     while k <= edge.end and i < p do
10.        if T[k] != P[i] then
11.            return EMPTY_LIST  // Mismatch inside compressed edge label
12.        k <- k + 1
13.        i <- i + 1
14.    curr <- edge.targetNode
15.
16. // Pattern P[0..p-1] is completely matched! Collect all leaf suffix indices below curr
17. results <- CreateEmptyList()
18. Collect-Leaf-Suffix-Indices-DFS(curr, results)
19. return results
```

#### Solution 7.4: `OpenAddress-Insert(T, m, k, v)` and `OpenAddress-Search(T, m, k)`

```text
ALGORITHM OpenAddress-Insert(T, m, k, v)
1. first_tombstone <- -1
2. for i <- 0 to m - 1 do
3.     idx <- Probe(k, i, m)
4.     if T[idx].state == OCCUPIED then
5.         if T[idx].key == k then
6.             T[idx].value <- v  // Key already exists; update in-place
7.             return true
8.     else if T[idx].state == DELETED then
9.         if first_tombstone == -1 then
10.            first_tombstone <- idx  // Save first tombstone, keep probing!
11.    else  // T[idx].state == EMPTY
12.        target <- (first_tombstone != -1) ? first_tombstone : idx
13.        T[target].key <- k
14.        T[target].value <- v
15.        T[target].state <- OCCUPIED
16.        return true
17.
18. if first_tombstone != -1 then
19.    T[first_tombstone].key <- k
20.    T[first_tombstone].value <- v
21.    T[first_tombstone].state <- OCCUPIED
22.    return true
23. return false  // Hash table overflow

ALGORITHM OpenAddress-Search(T, m, k)
1. for i <- 0 to m - 1 do
2.     idx <- Probe(k, i, m)
3.     if T[idx].state == EMPTY then
4.         return NIL  // Empty slot terminates search
5.     if T[idx].state == OCCUPIED and T[idx].key == k then
6.         return T[idx].value
7.     // If T[idx].state == DELETED, continue loop to i + 1!
8. return NIL
```

#### Solution 7.5: `Rehash-OpenTable(H)`

```text
ALGORITHM Rehash-OpenTable(H)
1. old_m <- H.m
2. old_slots <- H.slots
3. new_m <- NextPrime(2 * old_m)
4.
5. H.m <- new_m
6. H.n <- 0
7. H.tombstones <- 0
8. H.slots <- AllocateArray(new_m)
9. for j <- 0 to new_m - 1 do
10.    H.slots[j].state <- EMPTY
11.
12. for j <- 0 to old_m - 1 do
13.    if old_slots[j].state == OCCUPIED then
14.        // Recomputes hash modulo new_m and inserts into H.slots; tombstones are dropped
15.        OpenAddress-Insert(H.slots, H.m, old_slots[j].key, old_slots[j].value)
16.        H.n <- H.n + 1
17.
18. FreeArray(old_slots)
```

---

### 10.4 Complete C11 Solutions to C Programming Questions (Section 8)

All five C11 solutions below are self-contained, follow strict C11 standards, and handle all edge cases cleanly:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdint.h>
#include <stdbool.h>

/* =========================================================================
 * SOLUTION 8.1: Connected Components & BFS Shortest Unweighted Path
 * ========================================================================= */
typedef struct AdjNode {
    int             vertex;
    struct AdjNode *next;
} AdjNode;

typedef struct {
    int       V;
    AdjNode **head;
} Graph;

int count_connected_components_bfs(const Graph *g, int *comp_id) {
    for (int i = 0; i < g->V; i++) comp_id[i] = -1;
    int *queue = (int *)malloc((size_t)g->V * sizeof(int));
    int count = 0;

    for (int start = 0; start < g->V; start++) {
        if (comp_id[start] != -1) continue;
        count++;
        int front = 0, rear = 0;
        comp_id[start] = count;
        queue[rear++] = start;

        while (front < rear) {
            int u = queue[front++];
            for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
                int v = curr->vertex;
                if (comp_id[v] == -1) {
                    comp_id[v] = count;
                    queue[rear++] = v;
                }
            }
        }
    }
    free(queue);
    return count;
}

bool bfs_shortest_path(const Graph *g, int src, int dest, int *dist, int *parent) {
    for (int i = 0; i < g->V; i++) {
        dist[i] = -1;
        parent[i] = -1;
    }
    int *queue = (int *)malloc((size_t)g->V * sizeof(int));
    int front = 0, rear = 0;

    dist[src] = 0;
    queue[rear++] = src;

    while (front < rear) {
        int u = queue[front++];
        if (u == dest) break;
        for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
            int v = curr->vertex;
            if (dist[v] == -1) {
                dist[v] = dist[u] + 1;
                parent[v] = u;
                queue[rear++] = v;
            }
        }
    }
    free(queue);
    return dist[dest] != -1;
}

/* =========================================================================
 * SOLUTION 8.2: Trie Recursive Deletion & O(|P|) Prefix Word Counter
 * ========================================================================= */
#define ALPHABET_SIZE 26

typedef struct TrieNode {
    struct TrieNode *children[ALPHABET_SIZE];
    bool             is_end_of_word;
    int              prefix_count;
} TrieNode;

TrieNode *trie_create_node(void) {
    return (TrieNode *)calloc(1, sizeof(TrieNode));
}

void trie_insert(TrieNode *root, const char *word) {
    TrieNode *curr = root;
    curr->prefix_count++;
    for (size_t i = 0; word[i] != '\0'; i++) {
        int idx = word[i] - 'a';
        if (!curr->children[idx]) {
            curr->children[idx] = trie_create_node();
        }
        curr = curr->children[idx];
        curr->prefix_count++;
    }
    curr->is_end_of_word = true;
}

int trie_count_prefix(const TrieNode *root, const char *prefix) {
    const TrieNode *curr = root;
    for (size_t i = 0; prefix[i] != '\0'; i++) {
        int idx = prefix[i] - 'a';
        if (!curr->children[idx]) return 0;
        curr = curr->children[idx];
    }
    return curr->prefix_count;
}

static bool trie_has_no_children(const TrieNode *node) {
    for (int i = 0; i < ALPHABET_SIZE; i++) {
        if (node->children[i] != NULL) return false;
    }
    return true;
}

TrieNode *trie_delete_rec(TrieNode *curr, const char *word, size_t depth, bool *deleted) {
    if (!curr) return NULL;

    if (word[depth] == '\0') {
        if (curr->is_end_of_word) {
            curr->is_end_of_word = false;
            curr->prefix_count--;
            *deleted = true;
            if (trie_has_no_children(curr)) {
                free(curr);
                return NULL;
            }
        }
        return curr;
    }

    int idx = word[depth] - 'a';
    curr->children[idx] = trie_delete_rec(curr->children[idx], word, depth + 1, deleted);
    if (*deleted) {
        curr->prefix_count--;
    }
    if (!curr->is_end_of_word && trie_has_no_children(curr) && depth > 0) {
        free(curr);
        return NULL;
    }
    return curr;
}

/* =========================================================================
 * SOLUTION 8.3: Polynomial Rolling Hash & Separate Chaining Word Frequency
 * ========================================================================= */
typedef struct WordNode {
    char            *word;
    int              freq;
    struct WordNode *next;
} WordNode;

typedef struct {
    WordNode **buckets;
    uint32_t   m;
    uint32_t   n;
} WordFreqTable;

uint32_t poly_hash(const char *s, uint32_t m) {
    uint32_t h = 0;
    for (size_t i = 0; s[i] != '\0'; i++) {
        h = (h * 31U + (unsigned char)s[i]) % m;
    }
    return h;
}

void word_freq_increment(WordFreqTable *ht, const char *word) {
    uint32_t idx = poly_hash(word, ht->m);
    for (WordNode *curr = ht->buckets[idx]; curr != NULL; curr = curr->next) {
        if (strcmp(curr->word, word) == 0) {
            curr->freq++;
            return;
        }
    }
    WordNode *node = (WordNode *)malloc(sizeof(WordNode));
    size_t len = strlen(word) + 1;
    node->word = (char *)malloc(len);
    memcpy(node->word, word, len);
    node->freq = 1;
    node->next = ht->buckets[idx];
    ht->buckets[idx] = node;
    ht->n++;
}

/* =========================================================================
 * SOLUTION 8.4: Double Hashing Insert with Tombstone Reuse
 * ========================================================================= */
typedef enum { EMPTY = 0, OCCUPIED, DELETED } State;

typedef struct {
    int   key;
    int   value;
    State state;
} DHSlot;

typedef struct {
    DHSlot *table;
    int     m;
    int     R;
    int     n;
} DoubleHashTable;

int dh_h1(int k, int m) { return ((k % m) + m) % m; }
int dh_h2(int k, int R) { return R - (((k % R) + R) % R); }

bool dh_insert(DoubleHashTable *ht, int key, int value) {
    int first_del = -1;
    int u = dh_h1(key, ht->m);
    int v = dh_h2(key, ht->R);

    for (int i = 0; i < ht->m; i++) {
        int idx = (u + i * v) % ht->m;
        if (ht->table[idx].state == OCCUPIED) {
            if (ht->table[idx].key == key) {
                ht->table[idx].value = value;
                return true;
            }
        } else if (ht->table[idx].state == DELETED) {
            if (first_del == -1) first_del = idx;
        } else { /* EMPTY */
            int target = (first_del != -1) ? first_del : idx;
            ht->table[target].key = key;
            ht->table[target].value = value;
            ht->table[target].state = OCCUPIED;
            ht->n++;
            return true;
        }
    }
    if (first_del != -1) {
        ht->table[first_del].key = key;
        ht->table[first_del].value = value;
        ht->table[first_del].state = OCCUPIED;
        ht->n++;
        return true;
    }
    return false;
}

/* =========================================================================
 * SOLUTION 8.5: Dynamic Rehashing for Linear Probing
 * ========================================================================= */
void lp_rehash_table(DHSlot **table_ptr, int *m_ptr, int *n_ptr, int new_m) {
    DHSlot *old_table = *table_ptr;
    int old_m = *m_ptr;

    DHSlot *new_table = (DHSlot *)calloc((size_t)new_m, sizeof(DHSlot));
    for (int i = 0; i < new_m; i++) new_table[i].state = EMPTY;

    int new_n = 0;
    for (int i = 0; i < old_m; i++) {
        if (old_table[i].state == OCCUPIED) {
            int k = old_table[i].key;
            int val = old_table[i].value;
            int base = ((k % new_m) + new_m) % new_m;
            for (int probe = 0; probe < new_m; probe++) {
                int idx = (base + probe) % new_m;
                if (new_table[idx].state == EMPTY) {
                    new_table[idx].key = k;
                    new_table[idx].value = val;
                    new_table[idx].state = OCCUPIED;
                    new_n++;
                    break;
                }
            }
        }
    }
    free(old_table);
    *table_ptr = new_table;
    *m_ptr = new_m;
    *n_ptr = new_n;
}
```

---

## 11. Navigation Links

- **Unit 4 — Topic 1:** [01-Graph-Connectivity-Components-and-Paths.md](./01-Graph-Connectivity-Components-and-Paths.md) — Graph Applications: Connected Components and Shortest Unweighted Paths
- **Unit 4 — Topic 2:** [02-Trie-Trees.md](./02-Trie-Trees.md) — Standard Tries (Prefix Trees) and Compressed Tries
- **Unit 4 — Topic 3:** [03-Suffix-Tries-and-Suffix-Trees.md](./03-Suffix-Tries-and-Suffix-Trees.md) — Suffix Tries and Compressed Suffix Trees
- **Unit 4 — Topic 4:** [04-Hash-Functions-and-Hash-Tables.md](./04-Hash-Functions-and-Hash-Tables.md) — Hash Functions and Hash Tables
- **Unit 4 — Topic 5:** [05-Collision-Handling-and-Rehashing.md](./05-Collision-Handling-and-Rehashing.md) — Collision Handling and Rehashing
- **Current File:** `06-Unit-4-Review.md` (Unit 4 — Comprehensive Exam Preparation & Review)
