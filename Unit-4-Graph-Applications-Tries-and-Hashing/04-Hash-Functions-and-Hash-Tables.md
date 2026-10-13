# Unit 4 — Topic 4: Hash Functions and Hash Tables

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [From Direct Address Tables to Hash Tables](#11-from-direct-address-tables-to-hash-tables)
   - 1.2 [Two-Stage Hash Table Architecture: Hash Code + Compression Map](#12-two-stage-hash-table-architecture-hash-code--compression-map)
   - 1.3 [Relationship to Prior Search Structures](#13-relationship-to-prior-search-structures)
   - 1.4 [Real-World Systems and Algorithmic Applications](#14-real-world-systems-and-algorithmic-applications)
   - 1.5 [Learning Objectives](#15-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Formal Definition of the Dictionary (Map) ADT and Hash Table](#21-formal-definition-of-the-dictionary-map-adt-and-hash-table)
   - 2.2 [Core Terminology Table](#22-core-terminology-table)
   - 2.3 [Collisions, the Pigeonhole Principle, and the Birthday Paradox](#23-collisions-the-pigeonhole-principle-and-the-birthday-paradox)
   - 2.4 [Load Factor ($\alpha$) in Separate Chaining vs. Open Addressing](#24-load-factor-alpha-in-separate-chaining-vs-open-addressing)
   - 2.5 [Properties of a Good Hash Function and Simple Uniform Hashing (SUHA)](#25-properties-of-a-good-hash-function-and-simple-uniform-hashing-suha)
3. [Standard Hash Function Families](#3-standard-hash-function-families)
   - 3.1 [The Division Method](#31-the-division-method)
   - 3.2 [The Multiplication Method and Fibonacci Hashing](#32-the-multiplication-method-and-fibonacci-hashing)
   - 3.3 [The Mid-Square Method](#33-the-mid-square-method)
   - 3.4 [The Folding Method (Shift Folding and Boundary Folding)](#34-the-folding-method-shift-folding-and-boundary-folding)
   - 3.5 [Polynomial Rolling Hash for Strings (Horner's Method)](#35-polynomial-rolling-hash-for-strings-horners-method)
4. [Representation and Memory Layout in C](#4-representation-and-memory-layout-in-c)
   - 4.1 [Key-Value Entry and Hash Table `typedef struct` Declarations](#41-key-value-entry-and-hash-table-typedef-struct-declarations)
   - 4.2 [ASCII Memory Layout and Key Ownership Semantics](#42-ascii-memory-layout-and-key-ownership-semantics)
   - 4.3 [Static Fixed-Capacity vs. Dynamic Heap-Allocated Tables](#43-static-fixed-capacity-vs-dynamic-heap-allocated-tables)
5. [Basic Hash Table Operations, Algorithms, and C11 Implementation](#5-basic-hash-table-operations-algorithms-and-c11-implementation)
   - 5.1 [Insertion and Update (`ht_put` / Upsert)](#51-insertion-and-update-ht_put--upsert)
   - 5.2 [Lookup (`ht_get`) and Deletion (`ht_remove`)](#52-lookup-ht_get-and-deletion-ht_remove)
   - 5.3 [Expected $O(1 + \alpha)$ vs. Worst-Case $O(n)$ Complexity Analysis](#53-expected-o1--alpha-vs-worst-case-on-complexity-analysis)
   - 5.4 [Complete C11 Implementation: Hash Function Suite & Key-Value Hash Table](#54-complete-c11-implementation-hash-function-suite--key-value-hash-table)
6. [Comparison Tables and Complexity Summary](#6-comparison-tables-and-complexity-summary)
   - 6.1 [Comparison of Standard Hash Function Families](#61-comparison-of-standard-hash-function-families)
   - 6.2 [Dictionary ADT Implementations Complexity Comparison](#62-dictionary-adt-implementations-complexity-comparison)
7. [Common Errors, Pitfalls, and Edge Cases](#7-common-errors-pitfalls-and-edge-cases)
   - 7.1 [Pitfall 1: Negative Remainders from Signed Modulo (`k % m`) in C](#71-pitfall-1-negative-remainders-from-signed-modulo-k--m-in-c)
   - 7.2 [Pitfall 2: Choosing a Power-of-Two Table Size ($m = 2^p$) with Division Hashing](#72-pitfall-2-choosing-a-power-of-two-table-size-m--2p-with-division-hashing)
   - 7.3 [Pitfall 3: Storing Raw `char *` Key Pointers Without Deep-Copying](#73-pitfall-3-storing-raw-char--key-pointers-without-deep-copying)
   - 7.4 [Pitfall 4: Failing to Check for Duplicate Keys During Insertion](#74-pitfall-4-failing-to-check-for-duplicate-keys-during-insertion)
   - 7.5 [Boundary Conditions Checklist](#75-boundary-conditions-checklist)
8. [Summary and Revision Checklist](#8-summary-and-revision-checklist)
   - 8.1 [Key Takeaways and Core Invariants](#81-key-takeaways-and-core-invariants)
   - 8.2 [Self-Test Revision Questions](#82-self-test-revision-questions)
   - 8.3 [Navigation Links](#83-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 From Direct Address Tables to Hash Tables

Suppose an application needs to maintain a dynamic collection of records indexed by unique keys so that **Insert**, **Search (Lookup)**, and **Delete** each execute as fast as possible.

If the universe of all possible keys is a small integer range $U = \{0, 1, 2, \dots, M-1\}$ (for example, employee IDs from $0$ to $999$), we can use a **Direct Address Table**: an ordinary array `T[0 .. M-1]` in which key $k$ is stored directly at array index `T[k]`.

```
                  DIRECT ADDRESS TABLE (|U| = 10, n = 3 active keys)
    Key Universe U                  Array T[0 .. 9]
   +--------------+                +---------------+
   |   0, 1       |        T[0]    |     NULL      |
   |   2  --------+------->T[2]    | Ptr -> Rec(2) |   O(1) worst-case lookup,
   |   3, 4       |        ...     |     NULL      |   insertion, and deletion!
   |   5  --------+------->T[5]    | Ptr -> Rec(5) |
   |   6, 7       |        ...     |     NULL      |   BUT: requires |U| slots
   |   8  --------+------->T[8]    | Ptr -> Rec(8) |   regardless of how small n is!
   |   9          |        T[9]    |     NULL      |
   +--------------+                +---------------+
```

**Why Direct Addressing Fails in General:**
1. **Astronomical Universe Size ($|U| \gg n$):**
   - If keys are 64-bit unsigned integers (`uint64_t`), then $|U| = 2^{64} \approx 1.84 \times 10^{19}$. Allocating an array of $2^{64}$ pointers would require **147 exabytes** of RAM—even if our program only stores $n = 1{,}000$ active keys!
   - If keys are 20-character lowercase ASCII strings, $|U| = 26^{20} \approx 1.99 \times 10^{28}$.
2. **Non-Integer Keys:**
   - Keys in real software are often variable-length strings (variable names in a compiler, URLs in a browser cache), floating-point coordinates, IPv6 addresses, or composite structs that cannot directly index a C array `T[k]`.

A **Hash Table** solves both problems by allocating an array of size $m$ proportional to the number of *actually stored* keys $n$ (so $m = \Theta(n)$ with $m \ll |U|$), and using a deterministic mathematical function $h : U \to \{0, 1, \dots, m-1\}$—called a **hash function**—to map any key $k \in U$ to an array index $h(k)$.

### 1.2 Two-Stage Hash Table Architecture: Hash Code + Compression Map

In modern systems engineering and data structure design, mapping an arbitrary key $k \in U$ into a valid bucket index in $\{0, 1, \dots, m-1\}$ is conceptually decomposed into **two stages**:

$$U \xrightarrow{\quad h_1 \text{ (Hash Code Map)} \quad} \mathbb{Z}_{\ge 0} \xrightarrow{\quad h_2 \text{ (Compression Map)} \quad} \{0, 1, \dots, m-1\}$$

$$h(k) = h_2(h_1(k))$$

```
  Arbitrary Key k in U              32/64-bit Unsigned Int         Bucket Index in [0 .. m-1]
 +---------------------+   h1(k)   +----------------------+  h2(z)  +-----------------------+
 | "printf" (string)   | --------> | z = 2089451723       | ------> | i = z mod m = 4       |
 | 192.168.1.1 (IPv4)  |  (Hash    | (Machine-word integer| (Comp-  | (Valid array subscript|
 | struct Point {x, y} |   Code)   |  independent of m)   | ression)|  in T[0 .. m-1])      |
 +---------------------+           +----------------------+         +-----------------------+
```

1. **Stage 1 — Hash Code Map ($h_1 : U \to \mathbb{Z}_{\ge 0}$):** Converts an arbitrary key (such as a string, struct, or 64-bit ID) into a fixed-width non-negative integer (`uint32_t` or `uint64_t`), independent of the table capacity $m$. Examples include Polynomial Rolling Hash, `djb2`, FNV-1a, or MurmurHash3.
2. **Stage 2 — Compression Map ($h_2 : \mathbb{Z}_{\ge 0} \to \{0, 1, \dots, m-1\}$):** Compresses that potentially huge integer hash code into the valid C array index range $[0, m-1]$. Examples include the Division Method ($z \bmod m$) and the Multiplication Method ($\lfloor m ((z A) \bmod 1) \rfloor$).

### 1.3 Relationship to Prior Search Structures

Before hash tables, we studied linear and tree-based search structures:
- **Unsorted Arrays / Linked Lists:** Insert is $O(1)$, but Search and Delete require scanning $O(n)$ elements.
- **Sorted Arrays:** Binary search achieves $O(\log n)$ lookup, but inserting or deleting requires shifting $O(n)$ elements.
- **Balanced Binary Search Trees (AVL / Red-Black Trees):** Guarantee $O(\log n)$ worst-case search, insertion, and deletion while maintaining sorted key order.
- **Tries (Prefix Trees):** Achieve $O(L)$ operations for strings of length $L$ and support prefix queries, at the cost of high node pointer overhead.
- **Hash Tables:** By sacrificing sorted order and prefix traversal, hash tables break the $\Omega(\log n)$ comparison-based search barrier and achieve **$O(1)$ expected-time** search, insertion, and deletion!

### 1.4 Real-World Systems and Algorithmic Applications

1. **Compiler Symbol Tables:** Mapping identifier names (`"main"`, `"total_count"`) to their type, scope, and stack offset metadata during lexical/semantic analysis.
2. **Database Indexing & Hash Joins:** In-memory hash indexes and equijoin execution (`SELECT * FROM A JOIN B ON A.id = B.id`) in PostgreSQL, MySQL, and Oracle.
3. **In-Memory Key-Value Stores & Caches:** Redis, Memcached, CPU page-table Translation Lookaside Buffers (TLBs), and the Linux kernel VFS directory entry (`dentry`) cache.
4. **Network Routers & Firewalls:** Connection tracking tables mapping 5-tuples `(src_ip, dst_ip, src_port, dst_port, protocol)` to NAT or firewall state in $O(1)$ per packet.
5. **Algorithmic Problem Solving:** $O(n)$ Two-Sum, frequency counting, cycle detection, subgraph isomorphism, and Rabin-Karp substring matching.

### 1.5 Learning Objectives

After studying this note, you will be able to:
1. Explain why Direct Address Tables fail when $|U| \gg n$ and how the two-stage Hash Table architecture ($h = h_2 \circ h_1$) resolves this in $\Theta(n + m)$ space.
2. Define **collisions**, the **Pigeonhole Principle**, the **Birthday Paradox**, the **Load Factor** ($\alpha = n/m$), and the **Simple Uniform Hashing Assumption (SUHA)**.
3. Compute and analyze the five classic hash function families: **Division Method**, **Multiplication / Fibonacci Method**, **Mid-Square Method**, **Folding Method (Shift & Boundary)**, and **Polynomial Rolling String Hash (Horner's Rule)**.
4. Prove why $m = 2^p$ fails under the Division Method and why naive character-sum string hashing fails on anagrams.
5. Implement a complete, memory-safe **Hash Table** in C11 supporting $O(1 + \alpha)$ expected insertion/update, search, and deletion.

---

## 2. Definitions and Terminology

### 2.1 Formal Definition of the Dictionary (Map) ADT and Hash Table

A **Dictionary** (or **Associative Array / Map**) ADT manages a dynamic set $S$ of key-value pairs $(k, v)$ where each key $k \in U$ is unique, supporting three primary operations:
- $\text{INSERT}(S, k, v)$: Insert pair $(k, v)$ into $S$; if key $k$ already exists in $S$, update its associated value to $v$.
- $\text{SEARCH}(S, k)$: Return the value $v$ associated with key $k$ if $(k, v) \in S$, or a special sentinel $\text{NIL}$ if $k \notin S$.
- $\text{DELETE}(S, k)$: Remove the pair with key $k$ from $S$ if present.

A **Hash Table** implements the Dictionary ADT using:
1. A **bucket array** $T[0 \dots m-1]$ of capacity $m > 0$.
2. A **hash function** $h : U \to \{0, 1, \dots, m-1\}$ that computes the home bucket index $i = h(k)$ for any key $k \in U$.
3. A **collision resolution mechanism** (such as Separate Chaining or Open Addressing, covered in full depth in Topic 5) to handle multiple keys mapping to the same bucket index.

### 2.2 Core Terminology Table

| Term | Symbol / Notation | Formal Definition |
| :--- | :--- | :--- |
| **Key** | $k$ | The unique identifier used to index, insert, search, or delete a record. |
| **Value (Satellite Data)** | $v$ | The payload data associated with key $k$ in a key-value pair $(k, v)$. |
| **Universe of Keys** | $U$ | The set of all theoretically possible keys allowed by the key data type (with cardinality $|U|$). |
| **Active Key Set** | $K \subseteq U$ | The subset of keys currently stored in the hash table at a given moment. |
| **Number of Stored Elements** | $n = |K|$ | The count of active key-value pairs currently residing in the hash table. |
| **Table Capacity (Bucket Count)** | $m$ | The total number of slots (buckets) allocated in the underlying array $T[0 \dots m-1]$. |
| **Hash Function** | $h(k)$ | A deterministic function $h : U \to \{0, 1, \dots, m-1\}$ mapping key $k$ to its home slot index. |
| **Hash Code** | $h_1(k)$ | An intermediate non-negative integer representation of key $k$ prior to modulo/compression. |
| **Bucket / Slot** | $T[i]$ | Entry $i$ of the hash table array ($0 \le i < m$), which holds one item or a chain of colliding items. |
| **Collision** | $h(k_1) = h(k_2)$ | The event where two distinct keys $k_1 \neq k_2$ map to the exact same bucket index. |
| **Synonyms** | $k_1 \sim_h k_2$ | Two distinct keys $k_1 \neq k_2$ that satisfy $h(k_1) = h(k_2)$ are called *synonyms* with respect to $h$. |
| **Load Factor** | $\alpha = \dfrac{n}{m}$ | The ratio of stored elements $n$ to table capacity $m$; measures table density/fullness. |

### 2.3 Collisions, the Pigeonhole Principle, and the Birthday Paradox

A **collision** occurs whenever two distinct keys $k_1, k_2 \in U$ ($k_1 \neq k_2$) produce the same hash index:

$$k_1 \neq k_2 \quad \text{and} \quad h(k_1) = h(k_2)$$

#### 1. The Pigeonhole Principle (Worst-Case Certainty)
By Dirichlet's **Pigeonhole Principle**, if $|U| > m$ (there are more possible keys in the universe than buckets in the table), then **every** hash function $h : U \to \{0, 1, \dots, m-1\}$ must map at least $\lceil |U| / m \rceil$ distinct keys to the same bucket. For instance, if $|U| = 10{,}000$ and $m = 100$, at least one bucket is guaranteed to be the home bucket for at least $100$ distinct keys in $U$. Thus, collisions cannot be eliminated by "finding a magical collision-free hash function" unless $K$ is static and known in advance (*perfect hashing*).

#### 2. The Birthday Paradox (Probabilistic Earliness of Collisions)
Even if keys are hashed uniformly and independently at random into $m$ buckets, collisions occur **much earlier** than $n = m$. By the **Birthday Paradox**, the probability that $n$ randomly chosen keys have *zero* collisions in a table of size $m$ is:

$$P(\text{no collision}) = 1 \cdot \left(1 - \frac{1}{m}\right) \cdot \left(1 - \frac{2}{m}\right) \cdots \left(1 - \frac{n-1}{m}\right) = \prod_{i=0}^{n-1} \left(1 - \frac{i}{m}\right)$$

Using the Taylor approximation $1 - x \approx e^{-x}$ for small $x = i/m$:

$$P(\text{at least one collision}) = 1 - \prod_{i=0}^{n-1} \left(1 - \frac{i}{m}\right) \approx 1 - \exp\left(-\frac{n(n-1)}{2m}\right)$$

Setting $P(\text{collision}) = 0.5$ gives $n \approx \sqrt{2 \ln(2) \cdot m} \approx 1.177\sqrt{m}$:
- For $m = 365$ (days in a year), just **$n = 23$** keys ($\alpha = 23/365 \approx 6.3\%$) yield a **$50.73\%$** chance of a collision!
- For $m = 1{,}000{,}000$ buckets, a collision exceeds $50\%$ probability after inserting only **$n = 1{,}178$** keys ($\alpha \approx 0.12\%$ full)!

Therefore, **every general-purpose hash table MUST include a deterministic collision resolution strategy.**

### 2.4 Load Factor ($\alpha$) in Separate Chaining vs. Open Addressing

The **load factor** $\alpha$ of a hash table with $m$ buckets currently storing $n$ elements is defined as:

$$\alpha = \frac{n}{m}$$

The operational meaning and legal range of $\alpha$ depend critically on the collision resolution paradigm:

| Property of Load Factor $\alpha = n/m$ | Separate Chaining (Open Hashing) | Open Addressing (Closed Hashing) |
| :--- | :--- | :--- |
| **Physical Meaning of $\alpha$** | **Average number of elements per linked list chain** | **Fraction of array slots currently occupied** |
| **Mathematical Range** | $\alpha \ge 0$ (**can be $< 1$, $= 1$, or $> 1$**) | **$0 \le \alpha \le 1$ strictly** (since $n \le m$) |
| **Can $n$ Exceed $m$ ($\alpha > 1$)?** | **Yes** — new nodes are heap-allocated on linked lists | **No** — table overflows when $n = m$ ($\alpha = 1.0$) |
| **Typical Rehash Threshold ($\alpha_{\max}$)** | $\alpha_{\max} \approx 0.75 \text{ to } 1.0$ | $\alpha_{\max} \approx 0.50$ (Quadratic) to $0.70$ (Linear/Double) |

### 2.5 Properties of a Good Hash Function and Simple Uniform Hashing (SUHA)

A high-quality hash function $h : U \to \{0, 1, \dots, m-1\}$ must satisfy five core engineering and mathematical properties:

1. **Determinism (Referential Transparency):**
   For any key $k \in U$, evaluating $h(k)$ must always return the exact same bucket index throughout the lifetime of the hash table:
   $$k_1 = k_2 \implies h(k_1) = h(k_2)$$
   *(Note: A hash function must never depend on mutable fields of a key, the current time, or unseeded `rand()` calls during lookup!)*
2. **Computational Efficiency:**
   Evaluating $h(k)$ must take $O(1)$ time for fixed-width numeric keys (e.g., 32/64-bit integers) and $O(|k|)$ time for variable-length keys of length $|k|$ (e.g., strings). If $h(k)$ took $\Theta(n)$ time, hashing would be no faster than a linear search!
3. **Uniform Distribution (Simple Uniform Hashing Assumption — SUHA):**
   In theoretical analysis, an ideal hash function satisfies the **Simple Uniform Hashing Assumption (SUHA)**: any key $k$ drawn from $U$ is equally likely to hash to any of the $m$ available slots, independently of where any other key hashes:
   $$\Pr_{k \in U}[h(k) = j] = \frac{1}{m} \quad \text{for all } j \in \{0, 1, \dots, m-1\}$$
4. **Full Key Dependency (Sensitivity to All Bits and Order):**
   Every bit, digit, or character of $k$—as well as its **positional order**—must influence $h(k)$. Ignoring upper bits or ignoring character positions causes catastrophic clustering on real-world data (which often shares common prefixes like `"http://"` or common suffixes like `".com"`, or aligned memory addresses ending in `0000`).
5. **Avalanche Effect:**
   Flipping a single bit in the input key $k$ should cause each bit of the resulting hash code $h_1(k)$ to flip independently with probability $\approx 0.5$ (so even consecutive keys like `1001`, `1002`, `1003` scatter widely across the table).

---

## 3. Standard Hash Function Families

### 3.1 The Division Method

#### Mathematical Definition
The **Division Method** maps an integer key $k$ into one of $m$ buckets by taking the remainder of $k$ divided by $m$:

$$h(k) = k \bmod m$$

It requires only a single integer division/modulo instruction (`%` in C) and is the most widely used compression map when $m$ is chosen carefully.

#### Critical Rules for Choosing the Table Size $m$

1. **Rule 1 — Never Choose a Power of Two ($m = 2^p$):**
   If $m = 2^p$, then $k \bmod 2^p$ is identical to extracting the **lowest $p$ bits** of $k$ via bitwise AND (`k & ((1 << p) - 1)`):

   ```
   Suppose k is a 16-bit key and m = 2^4 = 16:

   Key k1 = 0101 1100 0011 1010 (23610)  --->  k1 mod 16 = 1010_2 = 10
   Key k2 = 1111 0000 1001 1010 (61594)  --->  k2 mod 16 = 1010_2 = 10
   Key k3 = 0000 0001 0100 1010 (  330)  --->  k3 mod 16 = 1010_2 = 10
            \_________________/  \____/
             Upper 12 bits are    ONLY the lowest 4 bits
             COMPLETELY IGNORED!  determine the bucket!
   ```
   For example, if keys are 64-bit pointers to 16-byte-aligned heap structs (`malloc` aligns addresses to multiples of $16$), then the lowest $4$ bits of every pointer are `0000`! With $m = 16$, **$100\%$ of all keys collide at bucket `0`**, leaving buckets `1..15` permanently empty!

2. **Rule 2 — Never Choose a Power of Ten ($m = 10^p$):**
   If $m = 10^3 = 1000$, then $k \bmod 1000$ depends *only* on the last three decimal digits of $k$. Keys that share decimal suffixes (e.g., prices ending in `99` or years ending in `2026`) all collide.

3. **Rule 3 — Avoid $m = 2^p - 1$ When Keys Are Strings Hashed with Radix $2^p$:**
   If characters are weighted by powers of $2^p$ and compressed modulo $m = 2^p - 1$, since $2^p \equiv 1 \pmod{2^p - 1}$, swapping any two characters in the string yields the **exact same remainder** modulo $2^p - 1$!

4. **Best Practice for Division Method:**
   Choose $m$ to be a **prime number not too close to an exact power of $2$ or $10$**.
   - **Example:** If you expect $n \approx 2{,}000$ elements and want load factor $\alpha \approx 0.7$, you need $m \approx 2000 / 0.7 \approx 2857$. Since $2^{11} = 2048$ and $2^{12} = 4096$, a prime near the middle such as **$m = 2999$** or **$m = 3001$** is an excellent choice.

#### Worked Numerical Example (Division Method)
Let $m = 97$ (a prime not close to $2^6 = 64$ or $2^7 = 128$):
- For $k = 1234$: $1234 = 12 \times 97 + 70 \implies h(1234) = 1234 \bmod 97 = \mathbf{70}$.
- For $k = 4321$: $4321 = 44 \times 97 + 53 \implies h(4321) = 4321 \bmod 97 = \mathbf{53}$.
- For $k = 5678$: $5678 = 58 \times 97 + 52 \implies h(5678) = 5678 \bmod 97 = \mathbf{52}$.

---

### 3.2 The Multiplication Method and Fibonacci Hashing

#### Mathematical Definition
The **Multiplication Method** operates in two steps:
1. Multiply the key $k$ by a constant real number $A$ in the range $0 < A < 1$, and extract the **fractional part** $(k \cdot A) \bmod 1 = k \cdot A - \lfloor k \cdot A \rfloor$.
2. Multiply this fractional part by $m$ and take the floor:

$$h(k) = \lfloor m \cdot ((k \cdot A) \bmod 1) \rfloor = \lfloor m \cdot (k \cdot A - \lfloor k \cdot A \rfloor) \rfloor$$

#### Key Advantages & Knuth's Golden Ratio Constant $A$
- Unlike the Division Method, the exact value of $m$ is **not critical** in the Multiplication Method. We can safely choose $m = 2^p$ to enable ultra-fast bit-shifting hardware execution!
- Donald Knuth proved that the optimal choice of $A$ for dispersing consecutive keys (Fibonacci hashing, via the Three-Distance Theorem) is the **Golden Ratio conjugate**:

$$A = \frac{\sqrt{5} - 1}{2} = \frac{1}{\phi} \approx 0.6180339887498949\dots$$

#### Fast Integer Implementation (32-Bit Fibonacci Hashing)
Instead of slow floating-point `floor()` operations, suppose keys fit in a $w = 32$-bit unsigned machine word ($2^w = 2^{32}$) and $m = 2^p$. We precompute the $32$-bit unsigned integer multiplier:

$$s = \lfloor A \cdot 2^{32} \rfloor = \left\lfloor \frac{\sqrt{5}-1}{2} \cdot 4294967296 \right\rfloor = 2654435769 = \mathtt{0x9E3779B9}_{\text{16}}$$

When we compute the 32-bit unsigned product `uint32_t prod = k * 2654435769U`, integer overflow naturally discards the integer part $\lfloor k \cdot A \rfloor$ (the bits above bit 31), leaving the fractional part scaled by $2^{32}$ inside the 32-bit register! Extracting the top $p$ bits via a right shift by $(32 - p)$ yields the bucket index in $[0, 2^p - 1]$:

$$h_{\text{Fib}}(k) = \left((k \times 2654435769) \bmod 2^{32}\right) \gg (32 - p)$$

```
                  32-BitUnsigned Product: (k * 2654435769U) mod 2^32
   Bit 31                  Bit (32 - p)                               Bit 0
  +----------------------------+-------------------------------------------+
  |   Top p bits = h_Fib(k)    |        Remaining (32 - p) bits            |
  +----------------------------+-------------------------------------------+
  \____________________________/
    Right-shift by (32 - p) gives bucket index in [0 .. 2^p - 1] in 2 CPU cycles!
```

#### Worked Numerical Examples (Multiplication Method)
1. **Floating-Point Formula ($m = 97$, $k = 1234$, $A = 0.61803398875$):**
   - $k \cdot A = 1234 \times 0.61803398875 = 762.6539421175$.
   - Fractional part: $(k \cdot A) \bmod 1 = 762.6539421175 - 762 = 0.6539421175$.
   - Multiply by $m = 97$: $97 \times 0.6539421175 = 63.432385\dots$
   - Take floor: $h(1234) = \lfloor 63.432385 \rfloor = \mathbf{63}$.
2. **32-Bit Integer Fibonacci Formula ($m = 2^7 = 128$, $p = 7$, $k = 1234$):**
   - $(1234 \times 2654435769) \bmod 2^{32} = 3275573738946 \bmod 4294967296 = 2796787634$.
   - Shift right by $(32 - 7) = 25$ bits: $\lfloor 2796787634 / 2^{25} \rfloor = \lfloor 2796787634 / 33554432 \rfloor = \mathbf{83}$.

---

### 3.3 The Mid-Square Method

#### Mathematical Definition
Developed by John von Neumann, the **Mid-Square Method** computes the hash address in three steps:
1. **Square the key:** Compute $k^2 = k \times k$.
2. **Extract the middle $r$ digits (or bits):** Discard the leading and trailing digits/bits of $k^2$, keeping $r$ central digits/bits.
3. **Compress (if needed):** If $m$ is $10^r$ (decimal) or $2^r$ (binary), the $r$ middle digits/bits directly lie in $[0, m-1]$; otherwise take $(\text{middle digits}) \bmod m$.

**Why the Middle Digits?** When two multi-digit numbers are multiplied, the bottom digits of $k^2$ depend only on the bottom digits of $k$, and the top digits of $k^2$ depend mostly on the top digits of $k$. Only the **middle digits** of $k^2$ receive cross-product contributions from **all** digits of $k$!

#### Worked Numerical Example (Mid-Square Method)
Suppose we extract the **middle $3$ decimal digits** (by discarding the rightmost $2$ digits and keeping the next $3$ digits: $\lfloor k^2 / 100 \rfloor \bmod 1000$):
- **Key $k = 1234$:**
  - $k^2 = 1234^2 = 1{,}522{,}756 = \mathtt{01\,\mathbf{522}\,75\,6}$.
  - Discard rightmost $2$ digits (`56`) and keep middle $3$ digits: $\mathbf{522}$.
  - For a table of size $m = 1000$, $h(1234) = \mathbf{522}$. For $m = 97$, $522 \bmod 97 = \mathbf{33}$.
- **Key $k = 5678$:**
  - $k^2 = 5678^2 = 32{,}239{,}684 = \mathtt{322\,\mathbf{396}\,84}$.
  - Middle $3$ digits: $\mathbf{396}$.
  - For $m = 1000$, $h(5678) = \mathbf{396}$. For $m = 97$, $396 \bmod 97 = \mathbf{8}$.

---

### 3.4 The Folding Method (Shift Folding and Boundary Folding)

When a key $k$ is a long multi-digit identifier (such as a 9-digit Social Security Number, a 12-digit Aadhaar number, an ISBN, or a 16-digit credit card number) that may overflow standard arithmetic or exhibits localized patterns, the **Folding Method** partitions $k$ into $c$ smaller chunks of $r$ digits each: $k = P_1 \| P_2 \| \dots \| P_c$.

There are two standard variants:

#### 1. Shift Folding
Each $r$-digit partition $P_1, P_2, \dots, P_c$ is shifted directly underneath the others and added together without reversing:

$$S_{\text{shift}} = \sum_{i=1}^{c} P_i, \qquad h_{\text{shift}}(k) = S_{\text{shift}} \bmod m$$

#### 2. Boundary Folding
Imagine the key written on a strip of paper and folded back-and-forth like an accordion along the partition boundaries. Every **even-indexed** partition ($P_2, P_4, \dots$) is **digit-reversed** ($\text{rev}(P_{2j})$) before summing:

$$S_{\text{boundary}} = P_1 + \text{rev}(P_2) + P_3 + \text{rev}(P_4) + \dots, \qquad h_{\text{boundary}}(k) = S_{\text{boundary}} \bmod m$$

**Why Boundary Folding Is Superior to Shift Folding:** In Shift Folding, swapping the first and second chunks ($P_1 \leftrightarrow P_2$) or swapping two digits in the same column across chunks leaves the sum $P_1 + P_2 + P_3$ completely unchanged! Reversing alternate chunks in Boundary Folding destroys this column-wise commutativity.

#### Worked Side-by-Side Numerical Example
Consider the 9-digit key $k = \mathtt{987654321}$ partitioned into three 3-digit chunks:
- $P_1 = \mathtt{987}$
- $P_2 = \mathtt{654}$ (reversed for Boundary Folding: $\text{rev}(654) = \mathtt{456}$)
- $P_3 = \mathtt{321}$

```
          SHIFT FOLDING                           BOUNDARY FOLDING
   Chunk P1 :        9 8 7                 Chunk P1      :        9 8 7
   Chunk P2 :      + 6 5 4   (unchanged)   rev(Chunk P2) :      + 4 5 6   (reversed!)
   Chunk P3 :      + 3 2 1                 Chunk P3      :      + 3 2 1
                   -------                                      -------
   Raw Sum  :      1 9 6 2                 Raw Sum       :      1 7 6 4

   Mod 1000 :          962                 Mod 1000      :          764
   Mod 97   :  1962 mod 97 = 22            Mod 97        :  1764 mod 97 = 18
```

---

### 3.5 Polynomial Rolling Hash for Strings (Horner's Method)

Strings $S = S[0 \,..\, L-1]$ of length $L$ are the most common keys in compilers, databases, and web servers. How should we convert a string $S$ into an integer hash code?

#### Why Naive Additive Hashing Fails Catastrophically
A common beginner mistake is to sum the ASCII values of all characters in $S$:

$$h_{\text{naive}}(S) = \left(\sum_{i=0}^{L-1} \text{ASCII}(S[i])\right) \bmod m$$

This function has **two fatal flaws**:
1. **Zero Positional Sensitivity (Anagram Collisions):** Addition is commutative ($a + b = b + a$). Therefore, **every anagram** of a word produces the exact same sum!
   - For `"stop"`, `"tops"`, `"pots"`, `"spot"`, `"post"`, `"opts"`:
     $$\text{ASCII}(\mathtt{'s'}) + \text{ASCII}(\mathtt{'t'}) + \text{ASCII}(\mathtt{'o'}) + \text{ASCII}(\mathtt{'p'}) = 115 + 116 + 111 + 112 = 454$$
     In a table of size $m = 101$, all six words collide at bucket $454 \bmod 101 = \mathbf{50}$!
2. **Tiny Output Range on Short Strings:** If keys are 6-letter lowercase English words ($\text{ASCII} \in [97, 122]$), the sum is strictly bounded between $6 \times 97 = 582$ and $6 \times 122 = 732$—a span of only **$151$ possible integers**! In a hash table with $m = 10{,}007$ buckets, **over $98.5\%$ of the table can never be reached!**

#### The Solution: Polynomial Rolling Hash
To make every character's **position** matter and spread short strings across the entire 32/64-bit integer range, we treat the string $S[0 \,..\, L-1]$ as a polynomial evaluated at a prime radix $p$:

$$h_{\text{poly}}(S) = \left( S[0]\cdot p^{L-1} + S[1]\cdot p^{L-2} + \dots + S[L-2]\cdot p^1 + S[L-1]\cdot p^0 \right) \bmod m = \left(\sum_{i=0}^{L-1} S[i] \cdot p^{L-1-i}\right) \bmod m$$

- **Choosing Radix $p$:** Choose $p$ to be a prime roughly equal to or larger than the input alphabet size:
  - $p = 31$ for lowercase English letters (`31 * x` can also be optimized by compilers to `(x << 5) - x`).
  - $p = 53$ for mixed-case English letters.
  - $p = 131$ or $257$ for arbitrary full ASCII / byte strings.

#### Fast $O(L)$ Evaluation via Horner's Rule
Instead of computing $p^{L-1-i}$ separately, **Horner's Rule** factors the polynomial into nested multiplications:

$$S[0]\cdot p^3 + S[1]\cdot p^2 + S[2]\cdot p + S[3] = \Big(\big((0 \cdot p + S[0]) \cdot p + S[1]\big) \cdot p + S[2]\Big) \cdot p + S[3]$$

By applying $\bmod m$ at each step using modular arithmetic identities, we prevent signed integer overflow and compute $h_{\text{poly}}(S)$ in exactly $L$ loop iterations ($O(L)$ time, $O(1)$ space):

```c
uint32_t hash = 0;
for (size_t i = 0; S[i] != '\0'; i++) {
    hash = (hash * 31U + (unsigned char)S[i]) % m;
}
```

#### Worked Trace: Polynomial Rolling Hash ($p = 31, m = 101$) on `"stop"` vs. `"tops"`

1. **Tracing $S = \mathtt{"stop"}$ ($\mathtt{'s'}=115, \mathtt{'t'}=116, \mathtt{'o'}=111, \mathtt{'p'}=112$):**
   - Start: $\text{hash}_0 = 0$
   - $i = 0$ (`'s'` = 115): $\text{hash}_1 = (0 \times 31 + 115) \bmod 101 = 115 \bmod 101 = \mathbf{14}$
   - $i = 1$ (`'t'` = 116): $\text{hash}_2 = (14 \times 31 + 116) \bmod 101 = 550 \bmod 101 = \mathbf{45}$
   - $i = 2$ (`'o'` = 111): $\text{hash}_3 = (45 \times 31 + 111) \bmod 101 = 1506 \bmod 101 = \mathbf{92}$
   - $i = 3$ (`'p'` = 112): $\text{hash}_4 = (92 \times 31 + 112) \bmod 101 = 2964 \bmod 101 = \mathbf{35}$
   - **Result:** $h_{\text{poly}}(\mathtt{"stop"}) = \mathbf{35}$.

2. **Tracing $S = \mathtt{"tops"}$ ($\mathtt{'t'}=116, \mathtt{'o'}=111, \mathtt{'p'}=112, \mathtt{'s'}=115$):**
   - Start: $\text{hash}_0 = 0$
   - $i = 0$ (`'t'` = 116): $\text{hash}_1 = 116 \bmod 101 = \mathbf{15}$
   - $i = 1$ (`'o'` = 111): $\text{hash}_2 = (15 \times 31 + 111) \bmod 101 = 576 \bmod 101 = \mathbf{71}$
   - $i = 2$ (`'p'` = 112): $\text{hash}_3 = (71 \times 31 + 112) \bmod 101 = 2313 \bmod 101 = \mathbf{91}$
   - $i = 3$ (`'s'` = 115): $\text{hash}_4 = (91 \times 31 + 115) \bmod 101 = 2936 \bmod 101 = \mathbf{7}$
   - **Result:** $h_{\text{poly}}(\mathtt{"tops"}) = \mathbf{7}$ (completely distinct from $35$!).

---

## 4. Representation and Memory Layout in C

### 4.1 Key-Value Entry and Hash Table `typedef struct` Declarations

To store key-value pairs in C11 and support dynamic bucket chains, we define two structures: a `HashNode` representing a single key-value entry, and a `HashTable` manager struct holding the array of bucket pointers, capacity $m$, and element count $n$:

```c
#include <stdint.h>

/* A single key-value node in a bucket chain */
typedef struct HashNode {
    char            *key;    /* Heap-allocated deep copy of the string key */
    int              value;  /* Satellite data associated with the key     */
    struct HashNode *next;   /* Pointer to the next colliding entry        */
} HashNode;

/* Hash Table descriptor */
typedef struct {
    HashNode **buckets;      /* Dynamic array of m bucket head pointers    */
    uint32_t   capacity;     /* m: total number of buckets in table        */
    uint32_t   size;         /* n: total number of stored key-value pairs  */
} HashTable;
```

### 4.2 ASCII Memory Layout and Key Ownership Semantics

```
  HashTable *ht (on Heap)            ht->buckets[0 .. 6] (m = 7)       Heap-Allocated HashNodes
 +----------------------+           +-----------+
 | buckets  ------------+---------> | [0]: NULL |
 | capacity : 7         |           +-----------+     +-----------------------+
 | size     : 3         |           | [1]: o----+---> | key: o--> "delta\0"   |
 +----------------------+           +-----------+     | value: 5              |
                                    | [2]: NULL |     | next: NULL            |
                                    +-----------+     +-----------------------+
                                    | [3]: o----+---> +-----------------------+     +-----------------------+
                                    +-----------+     | key: o--> "alpha\0"   |     | key: o--> "limit\0"   |
                                    | [4]: NULL |     | value: 42             |     | value: 50             |
                                    +-----------+     | next: o---------------+---> | next: NULL            |
                                    | [5]: NULL |     +-----------------------+     +-----------------------+
                                    +-----------+
                                    | [6]: NULL |
                                    +-----------+
```

**Why Deep-Copying Keys (`strdup`) Is Mandatory:**
If `node->key` simply stored the caller's pointer `node->key = caller_buf`, and the caller passed a stack-allocated buffer `char buf[64]` that gets overwritten on the next loop iteration (or goes out of scope), every node in the hash table would end up pointing to corrupted or dangling stack memory! Therefore:
- `ht_put` allocates a private heap copy of `key` (`strlen(key) + 1` bytes) when creating a new `HashNode`.
- `ht_remove` and `ht_destroy` call `free(node->key)` before calling `free(node)`.

### 4.3 Static Fixed-Capacity vs. Dynamic Heap-Allocated Tables

| Architectural Aspect | Static Fixed-Capacity Array (`HashNode *buckets[M]`) | Dynamic Heap-Allocated Bucket Array (`HashNode **buckets`) |
| :--- | :--- | :--- |
| **Capacity $m$ Determination** | Compile-time constant `#define M 101` | Runtime parameter `ht_create(capacity)` |
| **Stack/BSS vs. Heap Footprint** | Can live in `.bss` or stack without `malloc` for bucket array | Allocated on heap via `calloc(capacity, sizeof(HashNode *))` |
| **Can Be Resized (Rehashed)?** | **No** — array size is fixed at compile time | **Yes** — table can allocate a larger prime $m'$ and rehash |
| **Best Use Case** | Embedded firmware with strict no-heap policies | General systems software, compilers, databases |

---

## 5. Basic Hash Table Operations, Algorithms, and C11 Implementation

### 5.1 Insertion and Update (`ht_put` / Upsert)

- **Purpose:** Associate `key` with `value` in the hash table. Because dictionary keys are unique, if `key` is already present in bucket $i = h(k)$, we **update** its existing `value` in-place rather than inserting a duplicate node.
- **Step-by-Step Algorithm:**
  1. Compute bucket index $i = h(\text{key}) \in \{0, \dots, m-1\}$.
  2. Traverse the linked list starting at `ht->buckets[i]`:
     - For each node `curr`, if `strcmp(curr->key, key) == 0`, set `curr->value = value` and return `true` (`ht->size` does **not** change).
  3. If the end of the chain (`NULL`) is reached without finding `key`:
     - Allocate a new `HashNode` and deep-copy `key`.
     - Prepend the new node at the **head** of bucket $i$ in $O(1)$ time (`node->next = ht->buckets[i]; ht->buckets[i] = node;`).
     - Increment `ht->size++` and return `true`.

### 5.2 Lookup (`ht_get`) and Deletion (`ht_remove`)

- **Lookup (`ht_get`):**
  1. Compute $i = h(\text{key})$.
  2. Walk `curr = ht->buckets[i]`. If `strcmp(curr->key, key) == 0`, write `curr->value` to `*out_value` and return `true`.
  3. If `curr` reaches `NULL`, return `false` (key not found).
- **Deletion (`ht_remove`):**
  1. Compute $i = h(\text{key})$.
  2. Walk `curr = ht->buckets[i]` while maintaining `prev = NULL`.
  3. When `strcmp(curr->key, key) == 0`:
     - If `prev == NULL` (deleting head of bucket chain), set `ht->buckets[i] = curr->next`.
     - Else, bypass `curr` via `prev->next = curr->next`.
     - Free `curr->key`, free `curr`, decrement `ht->size--`, and return `true`.
  4. If `curr` reaches `NULL`, return `false`.

### 5.3 Expected $O(1 + \alpha)$ vs. Worst-Case $O(n)$ Complexity Analysis

Let $n$ be the number of stored elements and $m$ be the number of buckets, with load factor $\alpha = n/m$:
1. **Expected-Case Analysis (Under Simple Uniform Hashing — SUHA):**
   - Computing $h(k)$ takes $O(1)$ time (assuming bounded key length).
   - Because each of the $n$ keys is equally likely to land in any of the $m$ buckets independently with probability $1/m$, the expected number of keys in bucket $i$ is:
     $$E[N_i] = \sum_{j=1}^{n} \frac{1}{m} = \frac{n}{m} = \alpha$$
   - Therefore, searching, inserting, or deleting a key requires $O(1)$ time to compute $h(k)$ plus $O(\alpha)$ expected comparisons along chain $T[h(k)]$, totaling **$\Theta(1 + \alpha)$ expected time**.
   - When $m = \Theta(n)$ so that $\alpha \le C$ is bounded by a small constant (e.g., $\alpha \le 1.0$), $\Theta(1 + \alpha) = \mathbf{O(1)}$ **expected time**!
2. **Worst-Case Analysis (Adversarial or Degenerate Hash Function):**
   - If all $n$ keys collide into the **single bucket** $T[i]$ (producing one chain of length $n$ and $m - 1$ empty buckets), searching for a missing key or checking for duplicates during insertion traverses all $n$ nodes, degrading to **$\Theta(n)$ worst-case time**.

### 5.4 Complete C11 Implementation: Hash Function Suite & Key-Value Hash Table

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <stdint.h>
#include <stdbool.h>
#include <math.h>

/* =========================================================================
 * PART 1: STANDARD HASH FUNCTION IMPLEMENTATIONS
 * ========================================================================= */

/* 1. Division Method: h(k) = ((k % m) + m) % m (safe for negative keys) */
uint32_t hash_division(int64_t key, uint32_t m) {
    int64_t rem = key % (int64_t)m;
    if (rem < 0) {
        rem += (int64_t)m;
    }
    return (uint32_t)rem;
}

/* 2a. Multiplication Method (Floating-Point Knuth Golden Ratio Conjugate)
 *     h(k) = floor(m * ((k * A) mod 1)), where A = (sqrt(5) - 1) / 2
 */
uint32_t hash_multiplication_float(uint32_t key, uint32_t m) {
    const double A = 0.6180339887498949; /* (sqrt(5) - 1) / 2 */
    double product = (double)key * A;
    double fractional = product - floor(product);
    return (uint32_t)floor((double)m * fractional);
}

/* 2b. Fibonacci Multiplicative Hashing (Fast 32-bit Integer Arithmetic)
 *     Uses s = floor(A * 2^32) = 2654435769U (0x9E3779B9U) for m = 2^p
 */
uint32_t hash_fibonacci_32(uint32_t key, uint32_t p_bits) {
    const uint32_t FIBONACCI_MULT = 2654435769U;
    return (key * FIBONACCI_MULT) >> (32U - p_bits);
}

/* 3. Mid-Square Method (Decimal: square 4-digit key, extract middle 3 digits mod m) */
uint32_t hash_mid_square_dec(uint32_t key, uint32_t m) {
    uint64_t sq = (uint64_t)key * (uint64_t)key;
    /* Extract middle 3 digits from an up-to-8-digit square: (sq / 100) % 1000 */
    uint32_t mid3 = (uint32_t)((sq / 100ULL) % 1000ULL);
    return mid3 % m;
}

/* 4a. Shift Folding: Partition 9-digit key into three 3-digit chunks and sum */
uint32_t hash_folding_shift(uint32_t key, uint32_t m) {
    uint32_t p1 = key / 1000000U;         /* Leftmost 3 digits  */
    uint32_t p2 = (key / 1000U) % 1000U;  /* Middle 3 digits    */
    uint32_t p3 = key % 1000U;            /* Rightmost 3 digits */
    uint32_t sum = p1 + p2 + p3;
    return sum % m;
}

/* Helper to reverse a 3-digit number (e.g., 654 -> 456, 042 -> 240) */
static uint32_t reverse_3digits(uint32_t x) {
    uint32_t d1 = x / 100U;
    uint32_t d2 = (x / 10U) % 10U;
    uint32_t d3 = x % 10U;
    return d3 * 100U + d2 * 10U + d1;
}

/* 4b. Boundary Folding: Reverse alternating chunks before summing */
uint32_t hash_folding_boundary(uint32_t key, uint32_t m) {
    uint32_t p1 = key / 1000000U;
    uint32_t p2 = reverse_3digits((key / 1000U) % 1000U); /* Folded boundary */
    uint32_t p3 = key % 1000U;
    uint32_t sum = p1 + p2 + p3;
    return sum % m;
}

/* 5a. Naive Additive String Hash (BAD: Collides on all anagrams!) */
uint32_t hash_string_additive(const char *str, uint32_t m) {
    uint32_t sum = 0;
    for (size_t i = 0; str[i] != '\0'; i++) {
        sum += (unsigned char)str[i];
    }
    return sum % m;
}

/* 5b. Polynomial Rolling String Hash via Horner's Rule (p = 31) */
uint32_t hash_string_polynomial(const char *str, uint32_t m) {
    const uint32_t P = 31U;
    uint32_t hash = 0;
    for (size_t i = 0; str[i] != '\0'; i++) {
        hash = (hash * P + (unsigned char)str[i]) % m;
    }
    return hash;
}

/* =========================================================================
 * PART 2: BASIC HASH TABLE (KEY-VALUE DICTIONARY) IMPLEMENTATION
 * ========================================================================= */

typedef struct HashNode {
    char            *key;
    int              value;
    struct HashNode *next;
} HashNode;

typedef struct {
    HashNode **buckets;
    uint32_t   capacity; /* m: number of buckets */
    uint32_t   size;     /* n: number of stored key-value pairs */
} HashTable;

static char *my_strdup(const char *s) {
    size_t len = strlen(s) + 1;
    char *copy = (char *)malloc(len);
    if (copy != NULL) {
        memcpy(copy, s, len);
    }
    return copy;
}

HashTable *ht_create(uint32_t capacity) {
    if (capacity == 0) return NULL;
    HashTable *ht = (HashTable *)malloc(sizeof(HashTable));
    if (!ht) return NULL;

    ht->buckets = (HashNode **)calloc(capacity, sizeof(HashNode *));
    if (!ht->buckets) {
        free(ht);
        return NULL;
    }
    ht->capacity = capacity;
    ht->size = 0;
    return ht;
}

double ht_load_factor(const HashTable *ht) {
    if (!ht || ht->capacity == 0) return 0.0;
    return (double)ht->size / (double)ht->capacity;
}

/* Insert or Update (Upsert): O(1 + alpha) expected */
bool ht_put(HashTable *ht, const char *key, int value) {
    if (!ht || !key) return false;
    uint32_t index = hash_string_polynomial(key, ht->capacity);

    /* Step 1: Scan bucket chain to check if key already exists */
    HashNode *curr = ht->buckets[index];
    while (curr != NULL) {
        if (strcmp(curr->key, key) == 0) {
            curr->value = value; /* Update existing key's satellite data */
            return true;
        }
        curr = curr->next;
    }

    /* Step 2: Key not found; allocate and prepend new node at bucket head */
    HashNode *node = (HashNode *)malloc(sizeof(HashNode));
    if (!node) return false;
    node->key = my_strdup(key);
    if (!node->key) {
        free(node);
        return false;
    }
    node->value = value;
    node->next = ht->buckets[index];
    ht->buckets[index] = node;
    ht->size++;
    return true;
}

/* Search / Lookup: O(1 + alpha) expected */
bool ht_get(const HashTable *ht, const char *key, int *out_value) {
    if (!ht || !key) return false;
    uint32_t index = hash_string_polynomial(key, ht->capacity);

    const HashNode *curr = ht->buckets[index];
    while (curr != NULL) {
        if (strcmp(curr->key, key) == 0) {
            if (out_value) *out_value = curr->value;
            return true;
        }
        curr = curr->next;
    }
    return false;
}

/* Delete Key: O(1 + alpha) expected */
bool ht_remove(HashTable *ht, const char *key, int *out_removed_val) {
    if (!ht || !key) return false;
    uint32_t index = hash_string_polynomial(key, ht->capacity);

    HashNode *curr = ht->buckets[index];
    HashNode *prev = NULL;
    while (curr != NULL) {
        if (strcmp(curr->key, key) == 0) {
            if (prev == NULL) {
                ht->buckets[index] = curr->next;
            } else {
                prev->next = curr->next;
            }
            if (out_removed_val) *out_removed_val = curr->value;
            free(curr->key);
            free(curr);
            ht->size--;
            return true;
        }
        prev = curr;
        curr = curr->next;
    }
    return false;
}

void ht_print(const HashTable *ht) {
    printf("Hash Table (capacity m = %u, size n = %u, load factor alpha = %.2f):\n",
           ht->capacity, ht->size, ht_load_factor(ht));
    for (uint32_t i = 0; i < ht->capacity; i++) {
        printf("  Bucket [%2u]: ", i);
        const HashNode *curr = ht->buckets[i];
        if (!curr) {
            printf("NULL\n");
        } else {
            while (curr != NULL) {
                printf("(\"%s\" : %d) -> ", curr->key, curr->value);
                curr = curr->next;
            }
            printf("NULL\n");
        }
    }
}

void ht_destroy(HashTable *ht) {
    if (!ht) return;
    for (uint32_t i = 0; i < ht->capacity; i++) {
        HashNode *curr = ht->buckets[i];
        while (curr != NULL) {
            HashNode *next = curr->next;
            free(curr->key);
            free(curr);
            curr = next;
        }
    }
    free(ht->buckets);
    free(ht);
}

int main(void) {
    printf("=== 1. Numeric Hash Functions Comparison (m = 97) ===\n");
    uint32_t m = 97;
    uint32_t sample_keys[] = {1234, 4321, 5678, 987654321U};
    for (size_t i = 0; i < 4; i++) {
        uint32_t k = sample_keys[i];
        printf("Key = %9u | Div: %2u | Mult(Float): %2u | Fib(p=7): %3u | MidSq: %2u | ShiftFold: %2u | BoundFold: %2u\n",
               k,
               hash_division(k, m),
               hash_multiplication_float(k, m),
               hash_fibonacci_32(k, 7),
               hash_mid_square_dec(k, m),
               hash_folding_shift(k, m),
               hash_folding_boundary(k, m));
    }

    printf("\n=== 2. String Hashing: Naive Additive vs. Polynomial Rolling (m = 101) ===\n");
    const char *anagrams[] = {"stop", "tops", "pots", "spot", "post", "opts"};
    for (size_t i = 0; i < 6; i++) {
        printf("Word = \"%s\" | Additive Hash (mod 101): %2u | Polynomial Hash (p=31, mod 101): %2u\n",
               anagrams[i],
               hash_string_additive(anagrams[i], 101),
               hash_string_polynomial(anagrams[i], 101));
    }

    printf("\n=== 3. Basic Key-Value Hash Table Operations (m = 7) ===\n");
    HashTable *symtab = ht_create(7);

    ht_put(symtab, "count", 10);
    ht_put(symtab, "index", 25);
    ht_put(symtab, "total", 100);
    ht_put(symtab, "limit", 50);
    ht_put(symtab, "delta", 5);
    ht_put(symtab, "alpha", 42);
    ht_print(symtab);

    /* Update an existing key */
    printf("\nUpdating key \"total\" from 100 to 250...\n");
    ht_put(symtab, "total", 250);

    int val = 0;
    if (ht_get(symtab, "total", &val)) {
        printf("Lookup \"total\": found value = %d\n", val);
    }
    if (!ht_get(symtab, "missing_var", &val)) {
        printf("Lookup \"missing_var\": NOT FOUND\n");
    }

    /* Remove a key */
    int removed_val = 0;
    if (ht_remove(symtab, "index", &removed_val)) {
        printf("Removed key \"index\" (had value = %d). New load factor = %.2f\n",
               removed_val, ht_load_factor(symtab));
    }

    ht_destroy(symtab);
    return 0;
}
```

#### Program Output:

```text
=== 1. Numeric Hash Functions Comparison (m = 97) ===
Key =      1234 | Div: 70 | Mult(Float): 63 | Fib(p=7):  83 | MidSq: 33 | ShiftFold: 41 | BoundFold: 43
Key =      4321 | Div: 53 | Mult(Float): 50 | Fib(p=7):  67 | MidSq: 31 | ShiftFold: 34 | BoundFold: 42
Key =      5678 | Div: 52 | Mult(Float): 19 | Fib(p=7):  25 | MidSq:  8 | ShiftFold:  4 | BoundFold: 14
Key = 987654321 | Div: 30 | Mult(Float): 49 | Fib(p=7):  51 | MidSq: 31 | ShiftFold: 22 | BoundFold: 18

=== 2. String Hashing: Naive Additive vs. Polynomial Rolling (m = 101) ===
Word = "stop" | Additive Hash (mod 101): 50 | Polynomial Hash (p=31, mod 101): 35
Word = "tops" | Additive Hash (mod 101): 50 | Polynomial Hash (p=31, mod 101):  7
Word = "pots" | Additive Hash (mod 101): 50 | Polynomial Hash (p=31, mod 101): 46
Word = "spot" | Additive Hash (mod 101): 50 | Polynomial Hash (p=31, mod 101): 33
Word = "post" | Additive Hash (mod 101): 50 | Polynomial Hash (p=31, mod 101): 16
Word = "opts" | Additive Hash (mod 101): 50 | Polynomial Hash (p=31, mod 101):  1

=== 3. Basic Key-Value Hash Table Operations (m = 7) ===
Hash Table (capacity m = 7, size n = 6, load factor alpha = 0.86):
  Bucket [ 0]: NULL
  Bucket [ 1]: ("delta" : 5) -> NULL
  Bucket [ 2]: ("index" : 25) -> NULL
  Bucket [ 3]: ("alpha" : 42) -> ("limit" : 50) -> NULL
  Bucket [ 4]: ("total" : 100) -> NULL
  Bucket [ 5]: NULL
  Bucket [ 6]: ("count" : 10) -> NULL

Updating key "total" from 100 to 250...
Lookup "total": found value = 250
Lookup "missing_var": NOT FOUND
Removed key "index" (had value = 25). New load factor = 0.71
```

---

## 6. Comparison Tables and Complexity Summary

### 6.1 Comparison of Standard Hash Function Families

| Hash Function Family | Mathematical Formula | Optimal Choice of $m$ | Primary Strengths | Primary Weaknesses / Caveats |
| :--- | :--- | :--- | :--- | :--- |
| **Division Method** | $h(k) = k \bmod m$ | **Prime** not close to $2^p$ or $10^p$ | Simple; single instruction (`%`); exact uniformity if keys are uniform mod $m$ | Bad if $m = 2^p$ (ignores upper bits); integer division is slower than bit shifts |
| **Multiplication / Fibonacci** | $\lfloor m ((k A) \bmod 1) \rfloor$, $A = \frac{\sqrt{5}-1}{2}$ | **$m = 2^p$** (power of two) | Fast integer multiply + bit shift (`>>`); spreads consecutive keys evenly | Requires careful word-size multiplier selection (`0x9E3779B9U` for 32-bit) |
| **Mid-Square Method** | Middle $r$ digits/bits of $k^2$ | $m = 2^r$ (binary) or $10^r$ (decimal) | Middle bits of $k^2$ depend on all bits of $k$ | Computing $k^2$ can overflow if key width equals machine register width |
| **Folding Method (Shift / Boundary)** | Partition $k = P_1 \| \dots \| P_c$ and sum modulo $m$ | Prime $m$ or $10^r$ | Handles multi-word numeric keys (SSNs, ISBNs, account IDs) easily | Shift folding ignores chunk order; Boundary folding fixes this via reversal |
| **Polynomial Rolling Hash** | $\left(\sum_{i=0}^{L-1} S[i]\cdot p^{L-1-i}\right) \bmod m$ | Prime $m$, prime radix $p \in \{31, 53, 131\}$ | $O(L)$ via Horner's rule; distinguishes anagrams and short strings | Requires unsigned arithmetic (`uint32_t`/`uint64_t`) to avoid signed UB on overflow |

### 6.2 Dictionary ADT Implementations Complexity Comparison

Let $n$ be the number of stored elements, $m$ be the hash table capacity ($\alpha = n/m$), $|U|$ be the key universe size, and $L$ be the length of a string key (for integer comparisons, assume $O(1)$ key comparison):

| Data Structure | Search (Average) | Search (Worst) | Insert (Average) | Insert (Worst) | Delete (Average) | Delete (Worst) | Extra Space | Ordered Traversal ($O(n)$)? |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Unsorted Array** | $\Theta(n)$ | $\Theta(n)$ | $O(1)$ | $O(1)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | No ($O(n \log n)$ sort) |
| **Sorted Array** | $O(\log n)$ | $O(\log n)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | **Yes** |
| **Singly Linked List** | $\Theta(n)$ | $\Theta(n)$ | $O(1)$ | $O(1)$ | $\Theta(n)$ | $\Theta(n)$ | $\Theta(n)$ | No |
| **Balanced BST (AVL / Red-Black)** | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | $O(\log n)$ | $\Theta(n)$ | **Yes** (In-order) |
| **Standard Trie (Alphabet $\Sigma$)** | $O(L)$ | $O(L)$ | $O(L)$ | $O(L)$ | $O(L)$ | $O(L)$ | $O(n \cdot L \cdot |\Sigma|)$ | **Yes** (Lexicographical) |
| **Direct Address Table** | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $O(1)$ | $\Theta(|U|)$ | **Yes** (Scan $0 \dots |U|-1$) |
| **Hash Table ($\alpha = O(1)$)** | **$O(1)$** | $\Theta(n)$ | **$O(1)$** | $\Theta(n)$ | **$O(1)$** | $\Theta(n)$ | $\Theta(n + m)$ | **No** (Keys are scrambled) |

---

## 7. Common Errors, Pitfalls, and Edge Cases

### 7.1 Pitfall 1: Negative Remainders from Signed Modulo (`k % m`) in C

In C99/C11, the remainder operator `%` is defined by truncating division toward zero (`(a / b) * b + a % b == a`). Consequently, if `k` is a negative signed integer (or if a signed `int` accumulator overflows during string hashing!), `k % m` evaluates to a **negative number** in $[-(m-1), -1]$! Indexing `ht->buckets[k % m]` immediately reads out-of-bounds memory before the array start.

```c
/* BUG: Signed modulo returns negative index when key < 0 or signed hash overflows! */
int bad_hash(int key, int m) {
    return key % m; /* If key = -15 and m = 7, returns -1 -> buckets[-1] UB! */
}

/* FIX 1: Normalize signed remainder into [0, m - 1] */
uint32_t safe_hash_signed(int64_t key, uint32_t m) {
    int64_t rem = key % (int64_t)m;
    if (rem < 0) rem += (int64_t)m;
    return (uint32_t)rem;
}

/* FIX 2: Always use unsigned integers (uint32_t / uint64_t) and (unsigned char) casts in string hashes */
uint32_t safe_string_hash(const char *s, uint32_t m) {
    uint32_t h = 0;
    for (size_t i = 0; s[i] != '\0'; i++) {
        h = (h * 31U + (unsigned char)s[i]) % m; /* Unsigned arithmetic never becomes negative */
    }
    return h;
}
```

### 7.2 Pitfall 2: Choosing a Power-of-Two Table Size ($m = 2^p$) with Division Hashing

```c
/* BUG: Using m = 1024 (2^10) with Division Method on pointer/aligned keys */
uint32_t bad_ptr_hash(void *ptr) {
    uintptr_t addr = (uintptr_t)ptr;
    return (uint32_t)(addr % 1024U); /* Only looks at lowest 10 bits; aligned addresses collide! */
}

/* FIX: Either use a prime capacity m = 1021 with Division, OR use Fibonacci Multiplication with m = 1024 */
uint32_t good_ptr_hash(void *ptr) {
    uint32_t addr32 = (uint32_t)((uintptr_t)ptr ^ ((uintptr_t)ptr >> 32));
    return (addr32 * 2654435769U) >> (32U - 10U); /* Mixes bits before extracting top 10 bits */
}
```

### 7.3 Pitfall 3: Storing Raw `char *` Key Pointers Without Deep-Copying

```c
/* BUG: Storing caller's transient buffer pointer directly */
node->key = caller_key; /* If caller_key is a local char buf[64], every node points to the same stack buffer! */

/* FIX: Deep-copy the string key onto the heap and free it on node removal */
node->key = my_strdup(caller_key);
```

### 7.4 Pitfall 4: Failing to Check for Duplicate Keys During Insertion

If `ht_put` blindly prepends a new `HashNode` at the head of `ht->buckets[index]` without first scanning the chain for an existing node with the same key, inserting `("total", 100)` followed by `("total", 250)` creates **two duplicate nodes** in the chain. Even worse, deleting `"total"` once leaves the older `"total"` node resurrected in the table!

```c
/* BUG: Blind head insertion creates duplicate keys and inflates ht->size */
HashNode *node = create_node(key, value);
node->next = ht->buckets[index];
ht->buckets[index] = node;
ht->size++;

/* FIX: Always scan bucket[index] first; update in-place if key already exists */
for (HashNode *curr = ht->buckets[index]; curr != NULL; curr = curr->next) {
    if (strcmp(curr->key, key) == 0) {
        curr->value = value; /* Updated in-place; no duplicate node allocated! */
        return true;
    }
}
```

### 7.5 Boundary Conditions Checklist

- [ ] **Empty String Key (`""`):** Does `hash_string_polynomial("", m)` safely return `0` without reading past the null terminator?
- [ ] **High-Bit Characters (`char` Signedness):** Are characters cast to `(unsigned char)str[i]` so UTF-8 bytes $\ge 128$ do not sign-extend into negative integers?
- [ ] **Duplicate Key Update (`ht_put` on existing key):** Does `ht_put` overwrite the value in-place *without* incrementing `ht->size` or leaking a duplicate node?
- [ ] **Deleting Head vs. Interior vs. Tail of Bucket Chain:** Does `ht_remove` properly distinguish `prev == NULL` (updating `ht->buckets[index]`) from `prev != NULL` (updating `prev->next`)?
- [ ] **Full Teardown (`ht_destroy`):** Are both `curr->key` and `curr` freed for every node in every bucket before freeing `ht->buckets` and `ht`?

---

## 8. Summary and Revision Checklist

### 8.1 Key Takeaways and Core Invariants

1. **Direct Addressing vs. Hashing:** Direct Addressing gives worst-case $O(1)$ operations using $\Theta(|U|)$ space, which is infeasible when $|U| \gg n$. Hash tables compress $U$ into $m = \Theta(n)$ buckets via $h : U \to \{0, \dots, m-1\}$, achieving $O(1 + \alpha)$ expected time in $\Theta(n + m)$ space.
2. **Inevitable Collisions:** By the Pigeonhole Principle ($|U| > m$) and the Birthday Paradox ($50\%$ collision probability at $n \approx 1.177\sqrt{m}$), collisions are unavoidable in general dynamic sets.
3. **Load Factor ($\alpha = n/m$):** Measures average chain length in Separate Chaining ($\alpha \ge 0$, can exceed $1$) and fraction of full slots in Open Addressing ($0 \le \alpha \le 1$).
4. **Division vs. Multiplication:** Division ($k \bmod m$) requires $m$ to be a prime not close to $2^p$ or $10^p$. Multiplication ($\lfloor m ((k A) \bmod 1) \rfloor$ with $A = (\sqrt{5}-1)/2$) works well with $m = 2^p$.
5. **String Hashing:** Naive character summation collides on all anagrams; Polynomial Rolling Hash via Horner's Rule (`hash = (hash * 31 + c) % m`) incorporates character order in $O(L)$ time.

### 8.2 Self-Test Revision Questions

1. **Mathematical Question:** Why is $m = 2^{10} = 1024$ a poor choice of table capacity for the Division Method $h(k) = k \bmod m$?
   *(Answer: Because $k \bmod 2^{10}$ is equivalent to `k & 0x3FF`, which depends exclusively on the lowest $10$ bits of $k$ and ignores the upper $22$ or $54$ bits completely).*
2. **Calculation Question:** Compute the hash index of key $k = 500$ in a table of capacity $m = 100$ using the Multiplication Method with $A = 0.6180339887$.
   *(Answer: $500 \times 0.6180339887 = 309.01699435$; fractional part is $0.01699435$; $\lfloor 100 \times 0.01699435 \rfloor = \lfloor 1.699435 \rfloor = \mathbf{1}$).*
3. **String Hashing Question:** Why do `"listen"` and `"silent"` collide in any table size $m$ under naive additive hashing, and how does Polynomial Rolling Hash prevent this?
   *(Answer: Both strings are anagrams with the exact same multiset of ASCII codes, so their sum is identical before taking $\bmod m$. Polynomial Rolling Hash multiplies each character $S[i]$ by a distinct positional weight $p^{L-1-i}$, breaking commutativity).*
4. **Invariants Question:** Why can a hash table **not** print its keys in sorted order in $O(n)$ time like a Binary Search Tree or a Trie?
   *(Answer: Because a good hash function intentionally scrambles keys uniformly across $[0, m-1]$ (Avalanche Effect) so that $k_1 < k_2$ has no correlation with $h(k_1) < h(k_2)$; sorting $n$ extracted keys requires $\Omega(n \log n)$ comparison time).*

### 8.3 Navigation Links

- **Previous Topic:** [03-Suffix-Tries-and-Suffix-Trees.md](./03-Suffix-Tries-and-Suffix-Trees.md) — Suffix Tries and Compressed Suffix Trees
- **Current File:** `04-Hash-Functions-and-Hash-Tables.md` (Unit 4 — Topic 4)
- **Next Topic:** [05-Collision-Handling-and-Rehashing.md](./05-Collision-Handling-and-Rehashing.md) — Collision Handling (Separate Chaining, Linear/Quadratic Probing, Double Hashing) and Rehashing
- **Unit 4 Review:** [06-Unit-4-Review.md](./06-Unit-4-Review.md) — Unit 4 Comprehensive Exam Preparation & Review
