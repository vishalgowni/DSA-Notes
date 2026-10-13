# Unit 4 — Topic 5: Collision Handling (Separate Chaining, Linear Probing, Quadratic Probing, Double Hashing) and Rehashing

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [Why Collision Handling Is the Heart of Hash Table Engineering](#11-why-collision-handling-is-the-heart-of-hash-table-engineering)
   - 1.2 [Two Fundamental Paradigms: Separate Chaining vs. Open Addressing](#12-two-fundamental-paradigms-separate-chaining-vs-open-addressing)
   - 1.3 [Learning Objectives](#13-learning-objectives)
2. [Definitions, Probing Taxonomy, and Mathematical Foundations](#2-definitions-probing-taxonomy-and-mathematical-foundations)
   - 2.1 [Formal Definition of Probe Sequences in Open Addressing](#21-formal-definition-of-probe-sequences-in-open-addressing)
   - 2.2 [Core Terminology Table](#22-core-terminology-table)
   - 2.3 [Primary Clustering vs. Secondary Clustering vs. Uniform Probing](#23-primary-clustering-vs-secondary-clustering-vs-uniform-probing)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Separate Chaining Structs (`ChainNode` and `ChainedHashTable`)](#31-separate-chaining-structs-chainnode-and-chainedhashtable)
   - 3.2 [Open Addressing Structs (`SlotState` Enum, `OpenSlot`, and `OpenHashTable`)](#32-open-addressing-structs-slotstate-enum-openslot-and-openhashtable)
   - 3.3 [Memory Layout Comparison: Pointer Chains vs. Flat Cache-Friendly Arrays](#33-memory-layout-comparison-pointer-chains-vs-flat-cache-friendly-arrays)
4. [Technique 1: Separate Chaining (Open Hashing)](#4-technique-1-separate-chaining-open-hashing)
   - 4.1 [Architecture and Operations (Insert, Search, Delete)](#41-architecture-and-operations-insert-search-delete)
   - 4.2 [Step-by-Step Worked Table-State Trace ($m = 7$)](#42-step-by-step-worked-table-state-trace-m--7)
   - 4.3 [Mathematical Complexity Analysis ($O(1 + \alpha)$)](#43-mathematical-complexity-analysis-o1--alpha)
5. [Technique 2: Linear Probing (Open Addressing)](#5-technique-2-linear-probing-open-addressing)
   - 5.1 [Probing Formula and Cyclic Wrap-Around](#51-probing-formula-and-cyclic-wrap-around)
   - 5.2 [The Deletion Problem and Lazy Deletion with Tombstones (`EMPTY`, `OCCUPIED`, `DELETED`)](#52-the-deletion-problem-and-lazy-deletion-with-tombstones-empty-occupied-deleted)
   - 5.3 [Primary Clustering and Knuth's Expected Probe Formulas](#53-primary-clustering-and-knuths-expected-probe-formulas)
   - 5.4 [Step-by-Step Worked Trace: Insert, Search, Delete, and Tombstone Reuse ($m = 11$)](#54-step-by-step-worked-trace-insert-search-delete-and-tombstone-reuse-m--11)
6. [Technique 3: Quadratic Probing](#6-technique-3-quadratic-probing)
   - 6.1 [Probing Formula and Elimination of Primary Clustering](#61-probing-formula-and-elimination-of-primary-clustering)
   - 6.2 [Secondary Clustering Explained](#62-secondary-clustering-explained)
   - 6.3 [The Prime Capacity & $\alpha \le 0.5$ Guarantee (Theorem & Proof)](#63-the-prime-capacity--alpha-le-05-guarantee-theorem--proof)
   - 6.4 [Step-by-Step Worked Table-State Trace ($m = 11$)](#64-step-by-step-worked-table-state-trace-m--11)
7. [Technique 4: Double Hashing](#7-technique-4-double-hashing)
   - 7.1 [Two-Function Probing Formula $h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod m$](#71-two-function-probing-formula-hk-i--h_1k--i-cdot-h_2k-bmod-m)
   - 7.2 [Two Mandatory Design Rules for $h_2(k)$: Non-Zero and Coprime to $m$](#72-two-mandatory-design-rules-for-h_2k-non-zero-and-coprime-to-m)
   - 7.3 [Why Double Hashing Eliminates Both Primary and Secondary Clustering](#73-why-double-hashing-eliminates-both-primary-and-secondary-clustering)
   - 7.4 [Step-by-Step Worked Table-State Trace ($m = 11, R = 7$)](#74-step-by-step-worked-table-state-trace-m--11-r--7)
8. [Technique 5: Rehashing and Dynamic Table Resizing](#8-technique-5-rehashing-and-dynamic-table-resizing)
   - 8.1 [Why and When to Rehash (Load Factor and Tombstone Thresholds)](#81-why-and-when-to-rehash-load-factor-and-tombstone-thresholds)
   - 8.2 [Why You Cannot Simply `realloc` or `memcpy` a Hash Table!](#82-why-you-cannot-simply-realloc-or-memcpy-a-hash-table)
   - 8.3 [Step-by-Step Rehashing Algorithm and Worked Trace ($m = 7 \to m' = 17$)](#83-step-by-step-rehashing-algorithm-and-worked-trace-m--7-to-m--17)
   - 8.4 [Amortized $O(1)$ Insertion Cost vs. $O(n + m')$ Single-Rehash Cost](#84-amortized-o1-insertion-cost-vs-on--m-single-rehash-cost)
9. [Complete C11 Implementation: All 5 Techniques Verified](#9-complete-c11-implementation-all-5-techniques-verified)
10. [Master Comparison Tables and Complexity Summary](#10-master-comparison-tables-and-complexity-summary)
    - 10.1 [Master Comparison: Chaining vs. Linear vs. Quadratic vs. Double Hashing](#101-master-comparison-chaining-vs-linear-vs-quadratic-vs-double-hashing)
    - 10.2 [Expected Probe Counts as a Function of Load Factor $\alpha$](#102-expected-probe-counts-as-a-function-of-load-factor-alpha)
11. [Common Errors, Pitfalls, and Edge Cases](#11-common-errors-pitfalls-and-edge-cases)
    - 11.1 [Pitfall 1: Setting a Deleted Slot to `EMPTY` Instead of `DELETED` (Tombstone)](#111-pitfall-1-setting-a-deleted-slot-to-empty-instead-of-deleted-tombstone)
    - 11.2 [Pitfall 2: Stopping at the First Tombstone During `insert` Without Checking for Duplicates](#112-pitfall-2-stopping-at-the-first-tombstone-during-insert-without-checking-for-duplicates)
    - 11.3 [Pitfall 3: Allowing $h_2(k) == 0$ in Double Hashing (Infinite Loop on $i \cdot 0$)](#113-pitfall-3-allowing-h_2k--0-in-double-hashing-infinite-loop-on-i-cdot-0)
    - 11.4 [Pitfall 4: Using `realloc` Without Recomputing $k \bmod m'$ During Rehashing](#114-pitfall-4-using-realloc-without-recomputing-k-bmod-m-during-rehashing)
    - 11.5 [Boundary Conditions Checklist](#115-boundary-conditions-checklist)
12. [Summary and Revision Checklist](#12-summary-and-revision-checklist)
    - 12.1 [Key Takeaways and Core Invariants](#121-key-takeaways-and-core-invariants)
    - 12.2 [Self-Test Revision Questions](#122-self-test-revision-questions)
    - 12.3 [Navigation Links](#123-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 Why Collision Handling Is the Heart of Hash Table Engineering

As established in Topic 4 via the **Pigeonhole Principle** ($|U| > m$) and the **Birthday Paradox** ($50\%$ collision probability at $n \approx 1.177\sqrt{m}$), even the best cryptographic or multiplicative hash function cannot prevent two distinct keys $k_1 \neq k_2$ from mapping to the same bucket index $h(k_1) = h(k_2)$.

When a newly inserted key $k_{\text{new}}$ hashes to a bucket $i = h(k_{\text{new}})$ that is **already occupied** by an existing key $k_{\text{old}}$, a **collision** occurs. How the hash table resolves this collision determines:
1. Whether the table can store more than $m$ elements ($\alpha > 1$).
2. How safely and cleanly keys can be deleted without breaking subsequent lookups.
3. Whether colliding keys clump together into performance-degrading clusters.
4. How CPU cache lines are utilized during high-throughput lookups.

### 1.2 Two Fundamental Paradigms: Separate Chaining vs. Open Addressing

All collision resolution strategies in computer science fall into one of **two fundamental architectural paradigms**:

```
     PARADIGM 1: SEPARATE CHAINING                      PARADIGM 2: OPEN ADDRESSING
        (Open Hashing / External Chains)                   (Closed Hashing / Probing Inside Array)

    Buckets T[0..m-1]     Heap Linked Lists            Flat Slot Array T[0..m-1] (No pointers!)
   +-----------------+                                +-----------------+
   | [0]: NULL       |                                | [0]:   22       | <-- Home of 22 (h=0)
   +-----------------+   +----+    +----+             +-----------------+
   | [1]: o----------+-->| 44 |--->| 22 |--->NULL     | [1]:   44       | <-- 44 (h=0) probed +1!
   +-----------------+   +----+    +----+             +-----------------+
   | [2]: NULL       |                                | [2]:  EMPTY     |
   +-----------------+   +----+                       +-----------------+
   | [3]: o----------+-->| 18 |--->NULL               | [3]:   18       | <-- Home of 18 (h=3)
   +-----------------+   +----+                       +-----------------+
```

1. **Separate Chaining (also called Open Hashing):**
   Each slot $T[i]$ in the hash table array acts as the head pointer of a dynamic **linked list** (chain) storing all keys that hash to index $i$. Colliding elements are stored *outside* the bucket array in dynamically allocated heap nodes.
2. **Open Addressing (also called Closed Hashing):**
   All key-value records are stored **directly inside the flat array $T[0 \dots m-1]$ itself**, with at most one element per slot ($n \le m$, so $0 \le \alpha \le 1$). When a collision occurs at home slot $h(k)$, the table systematically examines (**probes**) a deterministic sequence of alternative slots in the same array until an empty slot is found.
   - The three classic open addressing schemes are **Linear Probing**, **Quadratic Probing**, and **Double Hashing**.

### 1.3 Learning Objectives

After studying this note, you will be able to:
1. Implement and trace **Separate Chaining** and derive its expected successful ($1 + \alpha/2$) and unsuccessful ($1 + \alpha$) search complexities.
2. Implement and trace **Linear Probing**, explain **Primary Clustering**, and prove why naive deletion breaks lookups and requires **Tombstones (`DELETED` state)**.
3. Implement and trace **Quadratic Probing**, distinguish **Secondary Clustering** from Primary Clustering, and prove why a prime capacity $m$ with $\alpha \le 0.5$ guarantees finding an empty slot.
4. Implement and trace **Double Hashing** $h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod m$, enforcing $h_2(k) \neq 0$ and $\gcd(h_2(k), m) = 1$.
5. Implement **Dynamic Rehashing** to resize a hash table to the next prime $m' \approx 2m$, purge tombstones, and achieve $O(1)$ amortized insertion time.

---

## 2. Definitions, Probing Taxonomy, and Mathematical Foundations

### 2.1 Formal Definition of Probe Sequences in Open Addressing

In **Open Addressing**, every slot $T[j]$ holds either a single key-value pair or a state marker (`EMPTY` or `DELETED`). To insert or search for a key $k$, we extend the hash function to take a second argument—the **probe number (attempt index)** $i \in \{0, 1, 2, \dots, m-1\}$:

$$h : U \times \{0, 1, \dots, m-1\} \to \{0, 1, \dots, m-1\}$$

For any key $k \in U$, the ordered tuple of slots examined across attempts $i = 0, 1, \dots, m-1$:

$$\langle h(k, 0),\ h(k, 1),\ h(k, 2),\ \dots,\ h(k, m-1) \rangle$$

is called the **probe sequence** of key $k$.
- **Home Slot ($i = 0$):** $h(k, 0) = h'(k)$ is the initial hash index where key $k$ prefers to reside if no collision occurs.
- **Full-Permutation Ideal:** Ideally, for every key $k$, the probe sequence $\langle h(k, 0), \dots, h(k, m-1) \rangle$ should be a **permutation of $\{0, 1, \dots, m-1\}$** so that every slot in the table can be examined before declaring the table full.

### 2.2 Core Terminology Table

| Term | Symbol / Definition | Role in Collision Resolution |
| :--- | :--- | :--- |
| **Separate Chaining** | $T[i] \to \text{Linked List}$ | Resolves collisions by linking all synonyms $h(k) = i$ into a chain at bucket $i$. |
| **Open Addressing** | $h(k, i) \in \{0, \dots, m-1\}$ | Resolves collisions by probing alternative array indices inside $T[0 \dots m-1]$ without pointers. |
| **Probe Number** | $i \in \{0, 1, \dots, m-1\}$ | The number of collisions encountered so far during the current search or insertion ($i = 0$ is the first try). |
| **Primary Clustering** | Contiguous runs coalesce | Phenomenon in Linear Probing where any key landing in a contiguous block of occupied slots extends that block. |
| **Secondary Clustering** | Same home slot $\implies$ same path | Phenomenon in Quadratic Probing where keys with $h'(k_1) = h'(k_2)$ follow the exact same probe sequence. |
| **Tombstone (`DELETED`)** | Lazy deletion sentinel | Marker placed in a slot when an element is deleted in Open Addressing so probe chains do not break prematurely. |
| **Rehashing** | $m \to m' = \text{next\_prime}(2m)$ | Allocating a larger table and re-inserting all active keys using the new hash modulus $m'$. |

### 2.3 Primary Clustering vs. Secondary Clustering vs. Uniform Probing

How many distinct probe sequences can an open-addressed hash table of capacity $m$ generate?
1. **Ideal Uniform Hashing:** An idealized theoretical model in which each key $k$ is equally likely to have any of the **$m!$** permutations of $\{0, 1, \dots, m-1\}$ as its probe sequence.
2. **Linear Probing ($m$ distinct probe sequences):** Because step size is fixed at $+1$, the probe sequence is completely determined by the starting slot $h'(k) \in \{0, \dots, m-1\}$. Moreover, keys with *different* home slots ($h'(k_1) = 3$ and $h'(k_2) = 4$) merge into the same contiguous block (**Primary Clustering**).
3. **Quadratic Probing ($m$ distinct probe sequences):** The probe sequence is still determined solely by $h'(k) \in \{0, \dots, m-1\}$. Keys with different home slots jump over each other without merging into a single contiguous block (no Primary Clustering), but any two keys with the *same* home slot $h'(k_1) = h'(k_2)$ trace the exact same quadratic offsets $+1, +4, +9, \dots$ (**Secondary Clustering**).
4. **Double Hashing ($m^2$ or $m(m-1)$ distinct probe sequences):** Because the starting slot is chosen by $h_1(k) \in \{0, \dots, m-1\}$ and the step size is independently chosen by $h_2(k) \in \{1, \dots, m-1\}$, two keys with the same home slot $h_1(k_1) = h_1(k_2)$ jump by **different** step sizes $h_2(k_1) \neq h_2(k_2)$, eliminating both Primary and Secondary Clustering!

---

## 3. Representation and Memory Layout in C

### 3.1 Separate Chaining Structs (`ChainNode` and `ChainedHashTable`)

```c
typedef struct ChainNode {
    int               key;
    int               value;
    struct ChainNode *next;
} ChainNode;

typedef struct {
    ChainNode **buckets; /* Array of m pointers to ChainNode heads */
    int         m;       /* Table capacity (number of buckets)     */
    int         n;       /* Number of stored elements              */
} ChainedHashTable;
```

### 3.2 Open Addressing Structs (`SlotState` Enum, `OpenSlot`, and `OpenHashTable`)

In Open Addressing, we cannot use `NULL` pointers if keys and values are stored inline as integers/structs, and we cannot use a single boolean `is_occupied` because deletion requires **three distinct states**:

```c
typedef enum {
    SLOT_EMPTY = 0, /* Never occupied since table creation or last rehash */
    SLOT_OCCUPIED,  /* Currently holds a valid active (key, value) pair   */
    SLOT_DELETED    /* Tombstone: held a key that was later deleted       */
} SlotState;

typedef struct {
    int       key;
    int       value;
    SlotState state;
} OpenSlot;

typedef struct {
    OpenSlot *slots;      /* Contiguous flat array of m slots            */
    int       m;          /* Table capacity (chosen as a prime number)   */
    int       n;          /* Count of active SLOT_OCCUPIED entries       */
    int       tombstones; /* Count of SLOT_DELETED entries               */
    int       prime_r;    /* Secondary prime R < m for Double Hashing    */
} OpenHashTable;
```

### 3.3 Memory Layout Comparison: Pointer Chains vs. Flat Cache-Friendly Arrays

```
  SEPARATE CHAINING (Pointer Chasing Across Heap)
  buckets[i] ---> [Node A] --(cache miss)--> [Node B] --(cache miss)--> NULL

  OPEN ADDRESSING (Contiguous Flat Array in L1/L2 CPU Cache)
  slots: | [0: OCC] | [1: OCC] | [2: DEL] | [3: OCC] | [4: EMP] | ...
         \______________________________________________________/
           Adjacent linear probes sit in the SAME 64-byte CPU cache line!
```

Even though Linear Probing performs more mathematical probes than Double Hashing or Separate Chaining at moderate load factors ($\alpha \le 0.5$), Linear Probing is frequently **faster in wall-clock CPU time** on modern hardware because slots `idx`, `idx + 1`, `idx + 2` reside inside the **same 64-byte L1 cache line**, whereas Separate Chaining incurs a dynamic `malloc` on insertion and pointer-chasing RAM cache misses on traversal!

---

## 4. Technique 1: Separate Chaining (Open Hashing)

### 4.1 Architecture and Operations (Insert, Search, Delete)

- **Structure:** An array `buckets[0 .. m-1]` of $m$ head pointers, initialized to `NULL`.
- **Insert (`chain_insert(ht, key, value)`):**
  1. Compute $i = h(k) = k \bmod m$.
  2. Traverse `buckets[i]` to check whether `key` already exists; if found, update `curr->value = value` and return.
  3. Otherwise, allocate a new `ChainNode` and insert it at the **head** of `buckets[i]` in $O(1)$ time:
     ```c
     node->next = ht->buckets[i];
     ht->buckets[i] = node;
     ht->n++;
     ```
     *(Why insert at the head instead of the tail? Because head insertion takes $O(1)$ pointer assignments after the duplicate check and benefits from temporal locality—recently inserted items are near the front of the chain!)*
- **Search (`chain_search(ht, key)`):**
  Compute $i = h(k)$ and linearly scan the singly linked list `buckets[i]` until `curr->key == key` (found) or `curr == NULL` (not found).
- **Delete (`chain_delete(ht, key)`):**
  Compute $i = h(k)$, locate the node with `curr->key == key` while tracking `prev`, unlink `curr` from `buckets[i]`, call `free(curr)`, and decrement `ht->n--`. **No tombstones are ever needed in Separate Chaining!**

### 4.2 Step-by-Step Worked Table-State Trace ($m = 7$)

Suppose $m = 7$ with hash function $h(k) = k \bmod 7$, and we insert the 8 keys:

$$K = \langle 18,\ 41,\ 22,\ 44,\ 59,\ 32,\ 31,\ 73 \rangle$$

 using **head insertion** into each bucket chain:

1. **Compute Home Buckets ($k \bmod 7$):**
   - $h(18) = 18 \bmod 7 = \mathbf{4}$
   - $h(41) = 41 \bmod 7 = \mathbf{6}$
   - $h(22) = 22 \bmod 7 = \mathbf{1}$
   - $h(44) = 44 \bmod 7 = \mathbf{2}$
   - $h(59) = 59 \bmod 7 = \mathbf{3}$
   - $h(32) = 32 \bmod 7 = \mathbf{4}$ *(collides with $18$ at bucket 4 $\implies$ prepended before $18$)*
   - $h(31) = 31 \bmod 7 = \mathbf{3}$ *(collides with $59$ at bucket 3 $\implies$ prepended before $59$)*
   - $h(73) = 73 \bmod 7 = \mathbf{3}$ *(collides with $31, 59$ at bucket 3 $\implies$ prepended before $31$)*

2. **Final Resulting Separate Chaining Table ($m = 7, n = 8, \alpha = 8/7 \approx 1.14$):**

```
   Bucket Index
   +-------+
   |  [0]  | ---> NULL
   +-------+      +----+
   |  [1]  | ---> | 22 | ---> NULL
   +-------+      +----+
   |  [2]  | ---> | 44 | ---> NULL
   +-------+      +----+      +----+      +----+
   |  [3]  | ---> | 73 | ---> | 31 | ---> | 59 | ---> NULL
   +-------+      +----+      +----+      +----+
   |  [4]  | ---> | 32 | ---> | 18 | ---> NULL
   +-------+      +----+      +----+
   |  [5]  | ---> NULL
   +-------+      +----+
   |  [6]  | ---> | 41 | ---> NULL
   +-------+      +----+
```

Notice that $n = 8 > m = 7$ ($\alpha = 1.14 > 1.0$), yet Separate Chaining handles it effortlessly without table overflow!

### 4.3 Mathematical Complexity Analysis ($O(1 + \alpha)$)

Under the **Simple Uniform Hashing Assumption (SUHA)**, let $\alpha = n/m$ be the load factor:
1. **Unsuccessful Search (Key Not in Table):**
   Any searched key $k \notin K$ hashes to one of the $m$ buckets, which has expected length $E[N_{h(k)}] = \frac{n}{m} = \alpha$. Reaching `NULL` requires $O(1)$ time to compute $h(k)$ plus examining all $\alpha$ nodes on average:
   $$T_{\text{unsuccessful}} = \Theta(1 + \alpha) \quad \text{(Expected key comparisons: } \alpha\text{)}$$
2. **Successful Search (Key Present in Table):**
   Searching for a randomly chosen key $k \in K$ examines $k$ itself ($1$ comparison) plus the expected number of keys that were inserted into bucket $h(k)$ *after* $k$ (since head insertion places newer keys ahead of $k$). With $n - 1$ other keys each landing in bucket $h(k)$ with probability $1/m$, the expected number of comparisons is:
   $$E[\text{comparisons}] = 1 + \frac{n - 1}{2m} = 1 + \frac{\alpha}{2} - \frac{1}{2m} \approx 1 + \frac{\alpha}{2}$$
   $$T_{\text{successful}} = \Theta\left(1 + \frac{\alpha}{2}\right) = \Theta(1 + \alpha)$$
3. **Worst-Case Time:** If all $n$ keys hash to the exact same bucket, the chain has length $n$, so search, insertion (with duplicate check), and deletion take **$\Theta(n)$ worst-case time**.

---

## 5. Technique 2: Linear Probing (Open Addressing)

### 5.1 Probing Formula and Cyclic Wrap-Around

In **Linear Probing**, when a collision occurs at the home slot $h'(k) = k \bmod m$, we resolve it by stepping sequentially forward **one slot at a time** (wrapping around from index $m-1$ to index $0$ modulo $m$):

$$h(k, i) = (h'(k) + i) \bmod m \quad \text{for } i = 0, 1, 2, \dots, m - 1$$

Explicitly, the probe sequence for key $k$ is:

$$\langle h'(k),\ (h'(k) + 1) \bmod m,\ (h'(k) + 2) \bmod m,\ \dots,\ (h'(k) + m - 1) \bmod m \rangle$$

Because step size is $1$ and $\gcd(1, m) = 1$ for every positive integer $m$, Linear Probing is **guaranteed to visit all $m$ slots** before giving up if the table is not full.

### 5.2 The Deletion Problem and Lazy Deletion with Tombstones (`EMPTY`, `OCCUPIED`, `DELETED`)

Why can we **NOT** simply mark a slot back to `EMPTY` when deleting a key from an open-addressed hash table?

#### Visual Proof of the Naive Deletion Bug
Suppose $m = 7$, $h'(k) = k \bmod 7$, and we insert three colliding keys $10$, $17$, and $24$ (all have $h'(k) = 3$):
1. `Insert(10)`: $h(10, 0) = 3 \implies$ stored at `T[3]`.
2. `Insert(17)`: $h(17, 0) = 3$ (occupied by 10) $\to$ probes $h(17, 1) = 4 \implies$ stored at `T[4]`.
3. `Insert(24)`: $h(24, 0) = 3$ (occupied) $\to$ probes $4$ (occupied) $\to$ probes $h(24, 2) = 5 \implies$ stored at `T[5]`.

Now suppose we **delete key `17`** at slot `T[4]`, and afterwards **search for key `24`** (which is still in the table at `T[5]`!):

```
  CASE A (BUG): Naive Deletion Sets T[4] = EMPTY
  +---------+---------+---------+---------+---------+---------+---------+
  | [0]: .  | [1]: .  | [2]: .  | [3]: 10 | [4]: .  | [5]: 24 | [6]: .  |
  +---------+---------+---------+---------+---------+---------+---------+
                                    ^         ^         ^
                                  i = 0     i = 1     24 is stranded here!
                                (10 != 24)  (EMPTY! Search(24) stops early
                                             and falsely returns NOT FOUND!)

  CASE B (FIX): Lazy Deletion Marks T[4] = DELETED (Tombstone)
  +---------+---------+---------+---------+---------+---------+---------+
  | [0]: .  | [1]: .  | [2]: .  | [3]: 10 | [4]:DEL | [5]: 24 | [6]: .  |
  +---------+---------+---------+---------+---------+---------+---------+
                                    ^         ^         ^
                                  i = 0     i = 1     i = 2
                                (10 != 24)  (Skip DEL (24 == 24: FOUND!)
                                             & keep probing!)
```

#### Rules for the Three Slot States (`EMPTY`, `OCCUPIED`, `DELETED`)

| Operation | Action on `SLOT_OCCUPIED` | Action on `SLOT_DELETED` (Tombstone) | Action on `SLOT_EMPTY` |
| :--- | :--- | :--- | :--- |
| **`search(k)`** | If `slot.key == k`, return **FOUND**; else continue probing $i+1$. | **Keep probing $i+1$!** (Never stop a search at a tombstone). | Stop immediately and return **NOT FOUND**. |
| **`delete(k)`** | If `slot.key == k`, set `slot.state = SLOT_DELETED`, `n--`, `tombstones++`, and return. | Keep probing $i+1$. | Stop and return **NOT FOUND**. |
| **`insert(k, v)`** | If `slot.key == k`, update `slot.value = v` and return. Else keep probing $i+1$. | If `first_tombstone == -1`, record `first_tombstone = idx`. **Keep probing $i+1$** to ensure `k` does not exist further down! | Key `k` is not in table: insert at `first_tombstone` (if $\neq -1$) else at current `idx`. |

> [!IMPORTANT]
> **Subtlety in `insert(k, v)` with Tombstones:** When inserting `k`, if probe $i = 1$ hits a `DELETED` tombstone, you **must not** immediately write `k` into that tombstone and return—because `k` might *already* reside at probe $i = 2$! Instead, save the index of the `first_tombstone` encountered, continue probing until either `k` is found (update in-place) or `SLOT_EMPTY` is reached, and only then place the new key into `first_tombstone`.

### 5.3 Primary Clustering and Knuth's Expected Probe Formulas

Linear Probing suffers from a severe phenomenon known as **Primary Clustering**: contiguous blocks of occupied slots (clusters) act like "nets" that grow wider and merge together.

**Why Big Clusters Grow Faster Than Small Clusters:**
Suppose a table of capacity $m = 100$ has an isolated single occupied slot (`T[10]`) and a contiguous cluster of $L = 20$ occupied slots (`T[40 .. 59]`).
- For a new random key to land on `T[10]` or `T[11]` and create a cluster of size 2, its home hash must fall in $\{10, 11\}$—a probability of only **$2/100 = 2\%$**.
- For the new random key to extend the 20-element cluster `T[40 .. 59]` into `T[60]`, its home hash can fall on **any** of the 21 slots $\{40, 41, \dots, 59, 60\}$—a probability of **$21/100 = 21\%$**!

Thus, **the larger a cluster becomes, the higher the probability that the next insertion hits it and makes it even larger.**

Donald Knuth proved that under SUHA with load factor $\alpha = n/m < 1$, the expected number of probes in Linear Probing is:

$$E[\text{Probes}_{\text{successful}}] \approx \frac{1}{2}\left(1 + \frac{1}{1 - \alpha}\right)$$

$$E[\text{Probes}_{\text{unsuccessful / insert}}] \approx \frac{1}{2}\left(1 + \frac{1}{(1 - \alpha)^2}\right)$$

Notice the $(1 - \alpha)^2$ in the denominator of unsuccessful searches:
- At $\alpha = 0.50$: Unsuccessful search averages $\frac{1}{2}(1 + \frac{1}{0.25}) = \mathbf{2.5}$ probes.
- At $\alpha = 0.90$: Unsuccessful search averages $\frac{1}{2}(1 + \frac{1}{0.01}) = \mathbf{50.5}$ probes!

### 5.4 Step-by-Step Worked Trace: Insert, Search, Delete, and Tombstone Reuse ($m = 11$)

Let $m = 11$, $h'(k) = k \bmod 11$, and $h(k, i) = (h'(k) + i) \bmod 11$. We insert keys $K = \langle 18, 41, 22, 44, 59, 32, 31, 73 \rangle$:

| Step | Key $k$ | $h'(k) = k \bmod 11$ | Probe Sequence Examined | Slot Assigned | Table State `[0 .. 10]` After Operation |
| :---: | :---: | :---: | :--- | :---: | :--- |
| 1 | `18` | $18 \bmod 11 = 7$ | `7` (Empty) | **`[7]`** | `[ . , . , . , . , . , . , . , 18, . , . , . ]` |
| 2 | `41` | $41 \bmod 11 = 8$ | `8` (Empty) | **`[8]`** | `[ . , . , . , . , . , . , . , 18, 41, . , . ]` |
| 3 | `22` | $22 \bmod 11 = 0$ | `0` (Empty) | **`[0]`** | `[ 22, . , . , . , . , . , . , 18, 41, . , . ]` |
| 4 | `44` | $44 \bmod 11 = 0$ | `0` (hit 22) $\to$ `1` (Empty) | **`[1]`** | `[ 22, 44, . , . , . , . , . , 18, 41, . , . ]` |
| 5 | `59` | $59 \bmod 11 = 4$ | `4` (Empty) | **`[4]`** | `[ 22, 44, . , . , 59, . , . , 18, 41, . , . ]` |
| 6 | `32` | $32 \bmod 11 = 10$ | `10` (Empty) | **`[10]`** | `[ 22, 44, . , . , 59, . , . , 18, 41, . , 32]` |
| 7 | `31` | $31 \bmod 11 = 9$ | `9` (Empty) | **`[9]`** | `[ 22, 44, . , . , 59, . , . , 18, 41, 31, 32]` |
| 8 | `73` | $73 \bmod 11 = 7$ | `7`$\to$`8`$\to$`9`$\to$`10`$\to$`0`$\to$`1`$\to$`2` (Empty) | **`[2]`** | `[ 22, 44, 73, . , 59, . , . , 18, 41, 31, 32]` |

Look at Step 8 (`Insert(73)`): because the cluster `[7, 8, 9, 10]` merged across the wrap-around boundary with the cluster `[0, 1]`, inserting `73` required **7 probes** (`7, 8, 9, 10, 0, 1, 2`)!
- **Now trace `Delete(22)`:** Key `22` is at `[0]`. We set `T[0] = DEL`:
  `[ DEL, 44, 73, . , 59, . , . , 18, 41, 31, 32 ]`
- **Now trace `Search(73)`:** $h'(73) = 7$. Probes `7` (18) $\to$ `8` (41) $\to$ `9` (31) $\to$ `10` (32) $\to$ `0` (`DEL` — **skip tombstone and keep going!**) $\to$ `1` (44) $\to$ `2` (**73 == 73: FOUND** in 7 probes!).
- **Now trace `Insert(55)`:** $h'(55) = 55 \bmod 11 = 0$.
  - $i = 0$: `T[0]` is `DEL` $\implies$ record `first_tombstone = 0`, keep probing for duplicates!
  - $i = 1$: `T[1]` is `44 != 55`.
  - $i = 2$: `T[2]` is `73 != 55`.
  - $i = 3$: `T[3]` is `EMPTY` $\implies$ `55` is not in the table; write `55` into `T[first_tombstone] = T[0]`!
  Final state: `[ 55, 44, 73, . , 59, . , . , 18, 41, 31, 32 ]`.

---

## 6. Technique 3: Quadratic Probing

### 6.1 Probing Formula and Elimination of Primary Clustering

To break up the long contiguous clusters of Linear Probing, **Quadratic Probing** adds a quadratic polynomial function of the probe number $i$ to the initial hash $h'(k)$:

$$h(k, i) = (h'(k) + c_1 \cdot i + c_2 \cdot i^2) \bmod m \quad (c_2 \neq 0)$$

In standard textbooks and exams, $c_1 = 0$ and $c_2 = 1$, giving the canonical **Pure Quadratic Probing** rule:

$$h(k, i) = (h'(k) + i^2) \bmod m \quad \text{for } i = 0, 1, 2, \dots, m - 1$$

Thus, starting from home slot $h'(k)$, Quadratic Probing inspects offsets:

$$\langle h'(k) + 0,\ h'(k) + 1,\ h'(k) + 4,\ h'(k) + 9,\ h'(k) + 16,\ \dots \rangle \pmod m$$

Instead of crawling slowly through a cluster one slot at a time ($+1, +2, +3$), a colliding key **leaps quadratically** ($+1, +4, +9, +16$) right over the contiguous region, preventing adjacent home buckets from coalescing into a primary cluster!

### 6.2 Secondary Clustering Explained

Does Quadratic Probing eliminate *all* clustering? **No.**
Suppose two distinct keys $k_1 \neq k_2$ have the **exact same initial hash value**: $h'(k_1) = h'(k_2) = b$.
Then for every probe attempt $i = 0, 1, 2, \dots$:

$$h(k_1, i) = (b + i^2) \bmod m = h(k_2, i)$$

Because both keys start at $b$ and add the identical offsets $+1, +4, +9, +16$, **any two keys that collide on their very first probe ($i=0$) will collide on every subsequent probe too!** This milder phenomenon is called **Secondary Clustering**. (Unlike Primary Clustering, keys with *different* initial hashes $h'(k_1) \neq h'(k_2)$ do not merge into the same probe path).

### 6.3 The Prime Capacity & $\alpha \le 0.5$ Guarantee (Theorem & Proof)

Quadratic Probing has an important mathematical limitation that Linear Probing does not have: **Quadratic Probing does not visit all $m$ slots of the table!** In fact, squares modulo $m$ ($i^2 \bmod m$, known as *quadratic residues*) cover at most $\frac{m+1}{2}$ distinct offsets.

#### Counterexample When $\alpha > 0.5$ or $m$ Is Composite
Suppose $m = 7$ (prime), and $4$ slots are already occupied ($\alpha = 4/7 \approx 0.57 > 0.5$): slots `T[0]`, `T[1]`, `T[2]`, and `T[4]` are occupied, while slots `T[3]`, `T[5]`, and `T[6]` are **empty**.
Now try to insert a key $k$ with $h'(k) = 0$:
- $i = 0: (0 + 0^2) \bmod 7 = 0$ (Occupied)
- $i = 1: (0 + 1^2) \bmod 7 = 1$ (Occupied)
- $i = 2: (0 + 2^2) \bmod 7 = 4$ (Occupied)
- $i = 3: (0 + 3^2) \bmod 7 = 9 \bmod 7 = 2$ (Occupied)
- $i = 4: (0 + 4^2) \bmod 7 = 16 \bmod 7 = 2$ (Occupied)
- $i = 5: (0 + 5^2) \bmod 7 = 25 \bmod 7 = 4$ (Occupied)
- $i = 6: (0 + 6^2) \bmod 7 = 36 \bmod 7 = 1$ (Occupied)

Even though **3 slots (`[3]`, `[5]`, `[6]`) are completely empty**, $0 + i^2 \bmod 7$ cycles endlessly through $\{0, 1, 2, 4\}$ and **fails to insert $k$!**

To prevent this failure, we rely on the following fundamental theorem:

> [!IMPORTANT]
> **Quadratic Probing Guarantee Theorem:**
> If the table capacity $m$ is a **prime number** and the table is **at least half empty** ($\alpha = \frac{n}{m} \le 0.5$, i.e., fewer than $\lfloor m/2 \rfloor + 1 = \lceil m/2 \rceil$ slots are occupied), then Quadratic Probing $h(k, i) = (h'(k) + i^2) \bmod m$ is **guaranteed** to visit $\lceil m/2 \rceil$ strictly distinct slots during the first $\lceil m/2 \rceil$ probes ($0 \le i \le \lfloor m/2 \rfloor$), and therefore is **guaranteed to find an empty slot!**

#### Mathematical Proof (By Contradiction)
Let $m > 2$ be prime, and consider two distinct probe indices $0 \le i < j \le \lfloor m/2 \rfloor$.
Suppose for the sake of contradiction that probe $i$ and probe $j$ land on the **same slot**:

$$(h'(k) + i^2) \equiv (h'(k) + j^2) \pmod m$$

Subtracting $h'(k)$ from both sides and factoring the difference of squares gives:

$$j^2 - i^2 \equiv 0 \pmod m \implies (j - i)(j + i) \equiv 0 \pmod m$$

Because $m$ is a **prime number**, Euclid's Lemma states that if a prime $m$ divides a product $(j - i)(j + i)$, then $m$ must divide at least one of the factors: either $m \mid (j - i)$ or $m \mid (j + i)$.
- **Can $m$ divide $(j - i)$?** Since $0 \le i < j \le \lfloor m/2 \rfloor$, we have $1 \le j - i \le \lfloor m/2 \rfloor < m$. No integer strictly between $1$ and $m-1$ is divisible by $m$.
- **Can $m$ divide $(j + i)$?** Since $0 \le i < j \le \lfloor m/2 \rfloor = \frac{m-1}{2}$, we have:
  $$1 \le j + i \le \left(\frac{m-1}{2} - 1\right) + \frac{m-1}{2} = m - 2 < m$$
  Again, $j + i$ lies strictly between $1$ and $m-2$, so it cannot be divisible by $m$!

This contradiction proves that all $\lfloor m/2 \rfloor + 1 = \lceil m/2 \rceil$ probes for $i \in \{0, 1, \dots, \lfloor m/2 \rfloor\}$ inspect **strictly distinct slots**. Since fewer than $\lceil m/2 \rceil$ slots are occupied ($\alpha \le 0.5$), by the Pigeonhole Principle at least one of these $\lceil m/2 \rceil$ distinct slots **must be empty**! $\blacksquare$

### 6.4 Step-by-Step Worked Table-State Trace ($m = 11$)

Let $m = 11$ (prime), $h'(k) = k \bmod 11$, and $h(k, i) = (h'(k) + i^2) \bmod 11$. We insert the same 8 keys $K = \langle 18, 41, 22, 44, 59, 32, 31, 73 \rangle$:

| Step | Key $k$ | $h'(k) = k \bmod 11$ | Quadratic Probe Evaluations $(h'(k) + i^2) \bmod 11$ | Slot Assigned | Table State `[0 .. 10]` After Operation |
| :---: | :---: | :---: | :--- | :---: | :--- |
| 1 | `18` | $7$ | $i=0: (7+0)\bmod 11 = \mathbf{7}$ (Empty) | **`[7]`** | `[ . , . , . , . , . , . , . , 18, . , . , . ]` |
| 2 | `41` | $8$ | $i=0: (8+0)\bmod 11 = \mathbf{8}$ (Empty) | **`[8]`** | `[ . , . , . , . , . , . , . , 18, 41, . , . ]` |
| 3 | `22` | $0$ | $i=0: (0+0)\bmod 11 = \mathbf{0}$ (Empty) | **`[0]`** | `[ 22, . , . , . , . , . , . , 18, 41, . , . ]` |
| 4 | `44` | $0$ | $i=0: 0$ (hit 22) $\to i=1: (0+1^2)\bmod 11 = \mathbf{1}$ (Empty) | **`[1]`** | `[ 22, 44, . , . , . , . , . , 18, 41, . , . ]` |
| 5 | `59` | $4$ | $i=0: (4+0)\bmod 11 = \mathbf{4}$ (Empty) | **`[4]`** | `[ 22, 44, . , . , 59, . , . , 18, 41, . , . ]` |
| 6 | `32` | $10$ | $i=0: (10+0)\bmod 11 = \mathbf{10}$ (Empty) | **`[10]`** | `[ 22, 44, . , . , 59, . , . , 18, 41, . , 32]` |
| 7 | `31` | $9$ | $i=0: (9+0)\bmod 11 = \mathbf{9}$ (Empty) | **`[9]`** | `[ 22, 44, . , . , 59, . , . , 18, 41, 31, 32]` |
| 8 | `73` | $7$ | $i=0: 7$ (hit 18)<br>$i=1: (7+1)\bmod 11 = 8$ (hit 41)<br>$i=2: (7+4)\bmod 11 = 0$ (hit 22)<br>$i=3: (7+9)\bmod 11 = \mathbf{5}$ (Empty!) | **`[5]`** | `[ 22, 44, . , . , 59, 73, . , 18, 41, 31, 32]` |

Compare Step 8 (`Insert(73)`) in Quadratic Probing vs. Linear Probing:
- Linear Probing took **7 probes** (`7, 8, 9, 10, 0, 1, 2`).
- Quadratic Probing leaped over `9, 10, 1` and found slot **`[5]`** in only **4 probes** (`7, 8, 0, 5`)!

---

## 7. Technique 4: Double Hashing

### 7.1 Two-Function Probing Formula $h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod m$

**Double Hashing** is the gold-standard open addressing method. It uses **two independent hash functions** $h_1(k)$ and $h_2(k)$:
- **$h_1(k)$ (Primary Hash Function):** Determines the **starting home slot** $h_1(k) \in \{0, 1, \dots, m-1\}$.
- **$h_2(k)$ (Secondary / Step-Size Hash Function):** Determines the **key-dependent jump stride** $h_2(k) \in \{1, 2, \dots, m-1\}$.

The probe sequence for key $k$ on attempt $i \in \{0, 1, \dots, m-1\}$ is:

$$h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod m$$

### 7.2 Two Mandatory Design Rules for $h_2(k)$: Non-Zero and Coprime to $m$

Designing the secondary hash function $h_2(k)$ requires strict adherence to **two mathematical invariants**:

#### Rule 1: $h_2(k)$ Must NEVER Evaluate to $0$ ($h_2(k) \neq 0$ for all $k \in U$)
What happens if $h_2(k) = 0$ for some key $k$ that collides at $h_1(k)$?
Then for every probe $i = 0, 1, 2, \dots$:

$$h(k, i) = (h_1(k) + i \cdot 0) \bmod m = h_1(k)$$

Every single probe examines the exact same occupied home slot $h_1(k)$ forever—causing an **infinite loop** (or immediate false overflow)!
Therefore, $h_2(k)$ must have range $\subseteq \{1, 2, \dots, m-1\}$. Two standard textbook formulas guarantee $h_2(k) \ge 1$:
1. **Prime-Difference Formula (Most Common in Exams):** Choose a prime number $R < m$ (typically the largest prime smaller than $m$, e.g., $R = 7$ when $m = 11$):
   $$h_2(k) = R - (k \bmod R) \quad \in \{1, 2, \dots, R\}$$
2. **Mod-$(m-1)$ Plus One Formula:**
   $$h_2(k) = 1 + (k \bmod (m - 1)) \quad \in \{1, 2, \dots, m - 1\}$$

#### Rule 2: $h_2(k)$ and Table Capacity $m$ Must Be Relatively Prime ($\gcd(h_2(k), m) = 1$)
Suppose $m = 10$ (composite) and $h_2(k) = 4$, with $h_1(k) = 1$. Then $(1 + 4i) \bmod 10$ only visits slots $\{1, 5, 9, 3, 7, 1, 5, \dots\}$—cycling through only $\frac{m}{\gcd(4, 10)} = \frac{10}{2} = 5$ slots and missing all even slots!
By elementary number theory, the arithmetic progression $(h_1(k) + i \cdot d) \bmod m$ for $i = 0, 1, \dots, m-1$ is a **full permutation of all $m$ slots $\{0, 1, \dots, m-1\}$ if and only if $\gcd(d, m) = 1$**.
- **How to guarantee $\gcd(h_2(k), m) = 1$ for all $k$:**
  - **Design A (Prime $m$):** Choose $m$ to be a **prime number**. Since every integer $d \in \{1, 2, \dots, m-1\}$ is coprime to a prime $m$, both $h_2(k) = R - (k \bmod R)$ and $h_2(k) = 1 + (k \bmod (m-1))$ automatically satisfy $\gcd(h_2(k), m) = 1$!
  - **Design B (Power-of-Two $m = 2^p$):** Choose $m = 2^p$ and force $h_2(k)$ to always return an **odd integer** (e.g., `(hash2(k) | 1)`), since every odd integer is coprime to $2^p$.

### 7.3 Why Double Hashing Eliminates Both Primary and Secondary Clustering

Suppose keys $k_1 = 22$ and $k_2 = 44$ are inserted into a table of size $m = 11$ with $h_1(k) = k \bmod 11$ and $h_2(k) = 7 - (k \bmod 7)$:
- Both keys have the **exact same primary hash**: $h_1(22) = 22 \bmod 11 = 0$ and $h_1(44) = 44 \bmod 11 = 0$.
- In Linear Probing (`+1`) and Quadratic Probing (`+i^2`), `22` and `44` follow the **exact same probe sequence** from `0`.
- In Double Hashing:
  - $h_2(22) = 7 - (22 \bmod 7) = 7 - 1 = \mathbf{6}$ (probe path: $0 \to 6 \to 1 \to 7 \dots$)
  - $h_2(44) = 7 - (44 \bmod 7) = 7 - 2 = \mathbf{5}$ (probe path: $0 \to 5 \to 10 \to 4 \dots$)

Even though `22` and `44` collided at home slot `0`, their probe sequences immediately diverge on $i = 1$! Because each pair $(h_1(k), h_2(k))$ defines a distinct probe sequence, Double Hashing generates **$m(m-1) = \Theta(m^2)$ distinct probe sequences** and closely approximates theoretical Uniform Hashing:

$$E[\text{Probes}_{\text{unsuccessful / insert}}] \approx \frac{1}{1 - \alpha}, \qquad E[\text{Probes}_{\text{successful}}] \approx \frac{1}{\alpha} \ln\left(\frac{1}{1 - \alpha}\right)$$

### 7.4 Step-by-Step Worked Table-State Trace ($m = 11, R = 7$)

Let $m = 11$, $h_1(k) = k \bmod 11$, $h_2(k) = 7 - (k \bmod 7)$, and $h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod 11$. We insert the same 8 keys $K = \langle 18, 41, 22, 44, 59, 32, 31, 73 \rangle$:

| Step | Key $k$ | $h_1(k) = k \bmod 11$ | $h_2(k) = 7 - (k \bmod 7)$ | Double Hashing Probes $(h_1(k) + i\cdot h_2(k)) \bmod 11$ | Slot Assigned | Table State `[0 .. 10]` After Operation |
| :---: | :---: | :---: | :---: | :--- | :---: | :--- |
| 1 | `18` | $7$ | $7 - 4 = 3$ | $i=0: \mathbf{7}$ (Empty) | **`[7]`** | `[ . , . , . , . , . , . , . , 18, . , . , . ]` |
| 2 | `41` | $8$ | $7 - 6 = 1$ | $i=0: \mathbf{8}$ (Empty) | **`[8]`** | `[ . , . , . , . , . , . , . , 18, 41, . , . ]` |
| 3 | `22` | $0$ | $7 - 1 = 6$ | $i=0: \mathbf{0}$ (Empty) | **`[0]`** | `[ 22, . , . , . , . , . , . , 18, 41, . , . ]` |
| 4 | `44` | $0$ | $7 - 2 = 5$ | $i=0: 0$ (hit 22) $\to i=1: (0 + 1\cdot 5)\bmod 11 = \mathbf{5}$ (Empty) | **`[5]`** | `[ 22, . , . , . , . , 44, . , 18, 41, . , . ]` |
| 5 | `59` | $4$ | $7 - 3 = 4$ | $i=0: \mathbf{4}$ (Empty) | **`[4]`** | `[ 22, . , . , . , 59, 44, . , 18, 41, . , . ]` |
| 6 | `32` | $10$ | $7 - 4 = 3$ | $i=0: \mathbf{10}$ (Empty) | **`[10]`** | `[ 22, . , . , . , 59, 44, . , 18, 41, . , 32]` |
| 7 | `31` | $9$ | $7 - 3 = 4$ | $i=0: \mathbf{9}$ (Empty) | **`[9]`** | `[ 22, . , . , . , 59, 44, . , 18, 41, 31, 32]` |
| 8 | `73` | $7$ | $7 - 3 = 4$ | $i=0: 7$ (hit 18) $\to i=1: (7 + 1\cdot 4)\bmod 11 = 0$ (hit 22)<br>$\to i=2: (7 + 2\cdot 4)\bmod 11 = 15\bmod 11 = \mathbf{4}$ (hit 59)<br>$\to i=3: (7 + 3\cdot 4)\bmod 11 = 19\bmod 11 = \mathbf{8}$ (hit 41)<br>$\to i=4: (7 + 4\cdot 4)\bmod 11 = 23\bmod 11 = \mathbf{1}$ (Empty!) | **`[1]`** | `[ 22, 73, . , . , 59, 44, . , 18, 41, 31, 32]` |

Notice Step 4 (`Insert(44)`): instead of placing `44` next to `22` at slot `[1]`, Double Hashing jumped by $h_2(44) = 5$ directly to slot **`[5]`**!

---

## 8. Technique 5: Rehashing and Dynamic Table Resizing

### 8.1 Why and When to Rehash (Load Factor and Tombstone Thresholds)

As more elements $n$ are inserted into a hash table of fixed capacity $m$, the load factor $\alpha = n/m$ rises:
- In **Open Addressing**, expected probe counts grow rapidly as $\alpha \to 1$ (and in Quadratic Probing, insertions can fail once $\alpha > 0.5$). Moreover, repeated `insert`/`delete` cycles accumulate `SLOT_DELETED` **tombstones** that never terminate unsuccessful searches, degrading lookup performance even when $n$ is small!
- In **Separate Chaining**, if $n \gg m$, chains grow long and operations degrade from $O(1)$ toward $O(n/m)$.

**Rehashing** solves both problems by dynamically expanding the table whenever a threshold is crossed:
1. **Open Addressing Load Threshold:** Trigger rehash when $\alpha = \frac{n}{m} > \alpha_{\max}$ (typically $\alpha_{\max} = \mathbf{0.50}$ for Quadratic Probing and $\alpha_{\max} = \mathbf{0.70}$ to $\mathbf{0.75}$ for Linear Probing / Double Hashing), OR when $\frac{n + \text{tombstones}}{m} > 0.75$.
2. **Separate Chaining Load Threshold:** Trigger rehash when $\alpha = \frac{n}{m} > \mathbf{1.0}$ (or $0.75$ in Java's `HashMap`).

### 8.2 Why You Cannot Simply `realloc` or `memcpy` a Hash Table!

A very common exam and interview misconception is thinking that resizing a hash table from capacity $m$ to $m'$ is just like resizing a dynamic vector/stack via `realloc(ht->slots, new_m * sizeof(Slot))`.

> [!WARNING]
> **Why `realloc` / `memcpy` Alone Corrupts a Hash Table:**
> In a hash table of size $m$, a key $k$ is located at bucket $k \bmod m$. When the table capacity changes to $m'$, every subsequent `search(k)` will compute $k \bmod m'$!
> Because $k \bmod m \neq k \bmod m'$ in general, leaving keys at their old indices $0 \dots m-1$ makes them **invisible to future lookups**!

**Concrete Proof Example:**
- In a table of capacity $m = 7$, key $k = 10$ is stored at index $10 \bmod 7 = \mathbf{3}$.
- When resized to the next prime after $2 \times 7 = 14$, which is $m' = 17$, where will `search(10)` look? It will compute $10 \bmod 17 = \mathbf{10}$!
- If you merely `realloc` the array so `10` stays at index `3`, then `search(10)` looks at `T[10]` (which is `EMPTY`) and falsely reports that `10` is not in the table!

### 8.3 Step-by-Step Rehashing Algorithm and Worked Trace ($m = 7 \to m' = 17$)

#### Step-by-Step Rehashing Algorithm
1. **Determine New Prime Capacity $m'$:** Compute $m' = \text{next\_prime}(2m)$ (for example, $m = 7 \implies 2m = 14 \implies m' = 17$). If using Double Hashing, also update the secondary prime $R' = \text{prev\_prime}(m')$ (e.g., $R' = 13$).
2. **Allocate Fresh Table Array:** Allocate a brand-new array `new_slots[0 .. m'-1]` with all slots initialized to `SLOT_EMPTY` (or `NULL` pointers in Separate Chaining). Reset `ht->n = 0` and `ht->tombstones = 0`.
3. **Scan Old Table and Re-Insert Active Keys:** Iterate $j$ from $0$ to $m - 1$ over `old_slots`:
   - If `old_slots[j].state == SLOT_EMPTY` or `SLOT_DELETED`: **ignore and skip it!** *(Rehashing completely purges all tombstones!)*
   - If `old_slots[j].state == SLOT_OCCUPIED`: compute its **new** hash address $h_{m'}(k) = k \bmod m'$ and insert `(old_slots[j].key, old_slots[j].value)` into `new_slots`.
4. **Free Old Array:** Call `free(old_slots)`.

#### Worked Rehashing Trace ($m = 7 \to m' = 17$ with Linear Probing, $\alpha_{\max} = 0.70$)
Suppose we start with an empty Linear Probing table of capacity $m = 7$ ($\alpha_{\max} = 0.70$, so at most $\lfloor 0.70 \times 7 \rfloor = 4$ keys fit before rehashing) and insert $\langle 10, 24, 38, 17, 31, 45 \rangle$:

1. **Insert first 4 keys into $m = 7$ ($h(k) = k \bmod 7$):**
   - `Insert(10)`: $10 \bmod 7 = 3 \implies$ `T[3] = 10` ($\alpha = 1/7 = 0.14$)
   - `Insert(24)`: $24 \bmod 7 = 3$ (hit 10) $\to$ `T[4] = 24` ($\alpha = 2/7 = 0.29$)
   - `Insert(38)`: $38 \bmod 7 = 3$ (hit 10, 24) $\to$ `T[5] = 38` ($\alpha = 3/7 = 0.43$)
   - `Insert(17)`: $17 \bmod 7 = 3$ (hit 10, 24, 38) $\to$ `T[6] = 17` ($\alpha = 4/7 = 0.57 \le 0.70$)

   ```
   OLD TABLE BEFORE REHASH (m = 7, n = 4, alpha = 4/7 = 0.57):
   +--------+--------+--------+--------+--------+--------+--------+
   | [0]: . | [1]: . | [2]: . | [3]:10 | [4]:24 | [5]:38 | [6]:17 |
   +--------+--------+--------+--------+--------+--------+--------+
                                \_________________________________/
                                  4-element primary cluster!
   ```

2. **`Insert(31)` Triggers Rehash:**
   Adding a 5th key would raise $\alpha$ to $5/7 = 0.714 > 0.70$!
   - **Step A:** Choose $m' = \text{next\_prime}(2 \times 7) = \text{next\_prime}(14) = \mathbf{17}$.
   - **Step B:** Scan `old_slots[0 .. 6]` and re-insert each `OCCUPIED` key using $k \bmod 17$:
     - Key `10` (from old `[3]`): $10 \bmod 17 = \mathbf{10} \implies \text{New}[10] = 10$
     - Key `24` (from old `[4]`): $24 \bmod 17 = \mathbf{7} \implies \text{New}[7] = 24$
     - Key `38` (from old `[5]`): $38 \bmod 17 = \mathbf{4} \implies \text{New}[4] = 38$
     - Key `17` (from old `[6]`): $17 \bmod 17 = \mathbf{0} \implies \text{New}[0] = 17$
   - **Step C:** Now insert the triggering key `31`: $31 \bmod 17 = \mathbf{14} \implies \text{New}[14] = 31$ ($\alpha = 5/17 \approx 0.29$).
3. **`Insert(45)` into the Rehashed Table ($m' = 17$):**
   - $45 \bmod 17 = \mathbf{11} \implies \text{New}[11] = 45$ ($\alpha = 6/17 \approx 0.35$).

   ```
   NEW TABLE AFTER REHASH AND INSERT(31), INSERT(45) (m' = 17, n = 6, alpha = 0.35):
   Index: [0]  [1] [2] [3] [4]  [5] [6] [7]  [8] [9] [10] [11] [12] [13] [14] [15] [16]
   Value: 17    .   .   .  38    .   .  24    .   .   10   45    .    .   31    .    .
   ```
   Look at the transformation: the 4 keys `{10, 24, 38, 17}` that all collided at $k \bmod 7 = 3$ in the old table now sit at **four completely separate, collision-free home slots** (`[10]`, `[7]`, `[4]`, `[0]`) in the $m' = 17$ table!

### 8.4 Amortized $O(1)$ Insertion Cost vs. $O(n + m')$ Single-Rehash Cost

- **Single Rehash Cost:** Allocating $m' \approx 2m$ slots, scanning $m$ old slots, and re-inserting $n = \Theta(m)$ elements takes **$\Theta(n + m') = \Theta(n)$ worst-case time** for that single resize operation.
- **Amortized Cost Across $N$ Insertions:** Because the table capacity **doubles** ($m' \ge 2m$) on each rehash, a rehash of size $n$ only occurs after at least $n/2$ cheap $O(1)$ insertions have elapsed since the previous rehash. By the aggregate/accounting method:
  $$\text{Total Cost of } N \text{ Insertions} = N \cdot O(1) + \sum_{k=0}^{\lfloor \log_2 N \rfloor} O(2^k) = O(N) + O(2N) = O(N)$$
  Dividing by $N$ insertions yields **$O(1)$ amortized expected time per insertion**!

---

## 9. Complete C11 Implementation: All 5 Techniques Verified

```c
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <stdbool.h>

/* =========================================================================
 * 1. SEPARATE CHAINING HASH TABLE
 * ========================================================================= */

typedef struct ChainNode {
    int               key;
    int               value;
    struct ChainNode *next;
} ChainNode;

typedef struct {
    ChainNode **buckets;
    int         m;
    int         n;
} ChainedHashTable;

ChainedHashTable *chain_create(int m) {
    ChainedHashTable *ht = (ChainedHashTable *)malloc(sizeof(ChainedHashTable));
    ht->buckets = (ChainNode **)calloc((size_t)m, sizeof(ChainNode *));
    ht->m = m;
    ht->n = 0;
    return ht;
}

void chain_insert(ChainedHashTable *ht, int key, int value) {
    int idx = ((key % ht->m) + ht->m) % ht->m;
    for (ChainNode *curr = ht->buckets[idx]; curr != NULL; curr = curr->next) {
        if (curr->key == key) {
            curr->value = value;
            return;
        }
    }
    ChainNode *node = (ChainNode *)malloc(sizeof(ChainNode));
    node->key = key;
    node->value = value;
    node->next = ht->buckets[idx];
    ht->buckets[idx] = node;
    ht->n++;
}

void chain_print(const ChainedHashTable *ht) {
    printf("Separate Chaining Table (m = %d, n = %d, alpha = %.2f):\n",
           ht->m, ht->n, (double)ht->n / ht->m);
    for (int i = 0; i < ht->m; i++) {
        printf("  [%2d]: ", i);
        for (ChainNode *c = ht->buckets[i]; c != NULL; c = c->next) {
            printf("%d -> ", c->key);
        }
        printf("NULL\n");
    }
}

void chain_destroy(ChainedHashTable *ht) {
    for (int i = 0; i < ht->m; i++) {
        ChainNode *c = ht->buckets[i];
        while (c) {
            ChainNode *next = c->next;
            free(c);
            c = next;
        }
    }
    free(ht->buckets);
    free(ht);
}

/* =========================================================================
 * 2. OPEN ADDRESSING HASH TABLE (LINEAR, QUADRATIC, DOUBLE HASHING + REHASH)
 * ========================================================================= */

typedef enum {
    SLOT_EMPTY = 0,
    SLOT_OCCUPIED,
    SLOT_DELETED    /* Tombstone marker for lazy deletion */
} SlotState;

typedef enum {
    PROBE_LINEAR = 0,
    PROBE_QUADRATIC,
    PROBE_DOUBLE
} ProbeMode;

typedef struct {
    int       key;
    int       value;
    SlotState state;
} OpenSlot;

typedef struct {
    OpenSlot *slots;
    int       m;            /* Table capacity (prime)                  */
    int       n;            /* Active OCCUPIED count                   */
    int       tombstones;   /* Count of SLOT_DELETED tombstones        */
    int       prime_r;      /* Secondary prime R < m for Double Hash   */
    ProbeMode mode;
} OpenHashTable;

static bool is_prime(int x) {
    if (x < 2) return false;
    if (x % 2 == 0) return x == 2;
    for (int d = 3; d * d <= x; d += 2) {
        if (x % d == 0) return false;
    }
    return true;
}

static int next_prime(int start) {
    int p = (start < 2) ? 2 : start;
    while (!is_prime(p)) p++;
    return p;
}

static int prev_prime(int below) {
    for (int p = below - 1; p >= 2; p--) {
        if (is_prime(p)) return p;
    }
    return 2;
}

/* Primary hash h1(k) = k mod m */
static int h1_func(int key, int m) {
    int r = key % m;
    return (r < 0) ? (r + m) : r;
}

/* Secondary hash h2(k) = R - (k mod R), guaranteed in [1 .. R] (never 0!) */
static int h2_func(int key, int R) {
    int r = key % R;
    if (r < 0) r += R;
    return R - r;
}

/* Compute probe index h(k, i) for attempt i in [0 .. m-1] */
static int probe_index(const OpenHashTable *ht, int key, int i) {
    int base = h1_func(key, ht->m);
    switch (ht->mode) {
        case PROBE_LINEAR:
            return (base + i) % ht->m;
        case PROBE_QUADRATIC:
            return (base + i * i) % ht->m;
        case PROBE_DOUBLE: {
            int step = h2_func(key, ht->prime_r);
            return (base + i * step) % ht->m;
        }
    }
    return base;
}

OpenHashTable *open_create(int m, ProbeMode mode) {
    OpenHashTable *ht = (OpenHashTable *)malloc(sizeof(OpenHashTable));
    ht->m = m;
    ht->n = 0;
    ht->tombstones = 0;
    ht->prime_r = prev_prime(m);
    ht->mode = mode;
    ht->slots = (OpenSlot *)calloc((size_t)m, sizeof(OpenSlot));
    for (int i = 0; i < m; i++) {
        ht->slots[i].state = SLOT_EMPTY;
    }
    return ht;
}

/* Forward declaration for dynamic rehashing */
void open_rehash(OpenHashTable *ht);

/* Insert with Tombstone Reuse:
 * Must continue probing past DELETED slots to ensure key does not already exist,
 * while remembering the first DELETED slot seen to reuse if key is new.
 */
bool open_insert(OpenHashTable *ht, int key, int value, bool auto_rehash) {
    if (auto_rehash && ((double)(ht->n + 1) / ht->m > 0.70)) {
        open_rehash(ht);
    }

    int first_tombstone = -1;
    for (int i = 0; i < ht->m; i++) {
        int idx = probe_index(ht, key, i);
        if (ht->slots[idx].state == SLOT_OCCUPIED) {
            if (ht->slots[idx].key == key) {
                ht->slots[idx].value = value; /* Update existing key */
                return true;
            }
        } else if (ht->slots[idx].state == SLOT_DELETED) {
            if (first_tombstone == -1) {
                first_tombstone = idx; /* Remember first tombstone for reuse */
            }
        } else { /* SLOT_EMPTY reached: key is definitely not in the table */
            int target = (first_tombstone != -1) ? first_tombstone : idx;
            if (first_tombstone != -1) {
                ht->tombstones--;
            }
            ht->slots[target].key = key;
            ht->slots[target].value = value;
            ht->slots[target].state = SLOT_OCCUPIED;
            ht->n++;
            return true;
        }
    }

    /* If all m probes were OCCUPIED or DELETED, reuse first_tombstone if available */
    if (first_tombstone != -1) {
        ht->tombstones--;
        ht->slots[first_tombstone].key = key;
        ht->slots[first_tombstone].value = value;
        ht->slots[first_tombstone].state = SLOT_OCCUPIED;
        ht->n++;
        return true;
    }
    return false; /* Table is completely full */
}

/* Search: Skips DELETED tombstones; stops only on match, EMPTY, or m probes */
bool open_search(const OpenHashTable *ht, int key, int *out_idx, int *out_probes) {
    for (int i = 0; i < ht->m; i++) {
        int idx = probe_index(ht, key, i);
        if (ht->slots[idx].state == SLOT_EMPTY) {
            if (out_probes) *out_probes = i + 1;
            return false; /* Empty slot terminates search */
        }
        if (ht->slots[idx].state == SLOT_OCCUPIED && ht->slots[idx].key == key) {
            if (out_idx) *out_idx = idx;
            if (out_probes) *out_probes = i + 1;
            return true;
        }
        /* If state == SLOT_DELETED, keep probing! */
    }
    if (out_probes) *out_probes = ht->m;
    return false;
}

/* Lazy Deletion: Marks slot as SLOT_DELETED (Tombstone) instead of SLOT_EMPTY */
bool open_delete(OpenHashTable *ht, int key) {
    int idx = -1;
    if (!open_search(ht, key, &idx, NULL)) {
        return false;
    }
    ht->slots[idx].state = SLOT_DELETED;
    ht->n--;
    ht->tombstones++;
    return true;
}

/* Rehashing: Allocate m' = next_prime(2 * m), re-hash all OCCUPIED keys, purge tombstones */
void open_rehash(OpenHashTable *ht) {
    int old_m = ht->m;
    OpenSlot *old_slots = ht->slots;

    int new_m = next_prime(old_m * 2);
    printf("\n>>> [REHASH TRIGGERED] Load factor exceeded threshold! Resizing m: %d -> %d (R = %d) <<<\n",
           old_m, new_m, prev_prime(new_m));

    ht->m = new_m;
    ht->prime_r = prev_prime(new_m);
    ht->n = 0;
    ht->tombstones = 0;
    ht->slots = (OpenSlot *)calloc((size_t)new_m, sizeof(OpenSlot));
    for (int i = 0; i < new_m; i++) {
        ht->slots[i].state = SLOT_EMPTY;
    }

    /* Re-insert only OCCUPIED entries; DELETED tombstones are automatically purged */
    for (int i = 0; i < old_m; i++) {
        if (old_slots[i].state == SLOT_OCCUPIED) {
            open_insert(ht, old_slots[i].key, old_slots[i].value, false);
        }
    }
    free(old_slots);
}

void open_print(const OpenHashTable *ht, const char *title) {
    printf("%s (m = %d, n = %d, tombstones = %d, alpha = %.2f):\n  ",
           title, ht->m, ht->n, ht->tombstones, (double)ht->n / ht->m);
    for (int i = 0; i < ht->m; i++) {
        if (ht->slots[i].state == SLOT_EMPTY) {
            printf("[%2d:  .  ] ", i);
        } else if (ht->slots[i].state == SLOT_DELETED) {
            printf("[%2d: DEL ] ", i);
        } else {
            printf("[%2d: %3d ] ", i, ht->slots[i].key);
        }
        if ((i + 1) % 6 == 0 && i + 1 < ht->m) printf("\n  ");
    }
    printf("\n");
}

void open_destroy(OpenHashTable *ht) {
    free(ht->slots);
    free(ht);
}

int main(void) {
    int keys[] = {18, 41, 22, 44, 59, 32, 31, 73};
    int num_keys = 8;

    printf("=== 1. Separate Chaining Trace (m = 7) ===\n");
    ChainedHashTable *cht = chain_create(7);
    for (int i = 0; i < num_keys; i++) {
        chain_insert(cht, keys[i], i * 10);
    }
    chain_print(cht);
    chain_destroy(cht);

    printf("\n=== 2. Open Addressing Comparison on Same Key Set (m = 11) ===\n");
    OpenHashTable *lp = open_create(11, PROBE_LINEAR);
    OpenHashTable *qp = open_create(11, PROBE_QUADRATIC);
    OpenHashTable *dh = open_create(11, PROBE_DOUBLE);

    for (int i = 0; i < num_keys; i++) {
        open_insert(lp, keys[i], i, false);
        open_insert(qp, keys[i], i, false);
        open_insert(dh, keys[i], i, false);
    }
    open_print(lp, "Linear Probing   ");
    open_print(qp, "Quadratic Probing");
    open_print(dh, "Double Hashing   ");

    printf("\n=== 3. Tombstone Deletion & Reuse Trace in Linear Probing ===\n");
    /* In lp (m=11): 73(h=7) probed 7 -> 8 -> 9 -> 10 -> 0(22) -> 1(44) -> 2(73) */
    printf("Deleting key 22 at slot [0] (creates a DELETED tombstone)...\n");
    open_delete(lp, 22);
    open_print(lp, "After Delete(22) ");

    int found_idx = -1, probes = 0;
    if (open_search(lp, 73, &found_idx, &probes)) {
        printf("Search(73) succeeded at slot [%d] in %d probes (probed past Tombstone at [0]!)\n",
               found_idx, probes);
    }

    printf("Inserting key 55 (h1(55) = 0): reuses Tombstone at slot [0] after checking duplicates...\n");
    open_insert(lp, 55, 999, false);
    open_print(lp, "After Insert(55) ");

    printf("\n=== 4. Dynamic Rehashing Demonstration (Starting at m = 7, alpha_max = 0.70) ===\n");
    OpenHashTable *dyn = open_create(7, PROBE_LINEAR);
    int stream[] = {10, 24, 38, 17, 31, 45};
    for (int i = 0; i < 6; i++) {
        printf("Inserting %d...", stream[i]);
        open_insert(dyn, stream[i], i, true);
        printf(" (n = %d, m = %d, alpha = %.2f)\n", dyn->n, dyn->m, (double)dyn->n / dyn->m);
    }
    open_print(dyn, "Final Rehashed Table");

    open_destroy(lp);
    open_destroy(qp);
    open_destroy(dh);
    open_destroy(dyn);
    return 0;
}
```

#### Program Output:

```text
=== 1. Separate Chaining Trace (m = 7) ===
Separate Chaining Table (m = 7, n = 8, alpha = 1.14):
  [ 0]: NULL
  [ 1]: 22 -> NULL
  [ 2]: 44 -> NULL
  [ 3]: 73 -> 31 -> 59 -> NULL
  [ 4]: 32 -> 18 -> NULL
  [ 5]: NULL
  [ 6]: 41 -> NULL

=== 2. Open Addressing Comparison on Same Key Set (m = 11) ===
Linear Probing    (m = 11, n = 8, tombstones = 0, alpha = 0.73):
  [ 0:  22 ] [ 1:  44 ] [ 2:  73 ] [ 3:  .  ] [ 4:  59 ] [ 5:  .  ] 
  [ 6:  .  ] [ 7:  18 ] [ 8:  41 ] [ 9:  31 ] [10:  32 ] 
Quadratic Probing (m = 11, n = 8, tombstones = 0, alpha = 0.73):
  [ 0:  22 ] [ 1:  44 ] [ 2:  .  ] [ 3:  .  ] [ 4:  59 ] [ 5:  73 ] 
  [ 6:  .  ] [ 7:  18 ] [ 8:  41 ] [ 9:  31 ] [10:  32 ] 
Double Hashing    (m = 11, n = 8, tombstones = 0, alpha = 0.73):
  [ 0:  22 ] [ 1:  73 ] [ 2:  .  ] [ 3:  .  ] [ 4:  59 ] [ 5:  44 ] 
  [ 6:  .  ] [ 7:  18 ] [ 8:  41 ] [ 9:  31 ] [10:  32 ] 

=== 3. Tombstone Deletion & Reuse Trace in Linear Probing ===
Deleting key 22 at slot [0] (creates a DELETED tombstone)...
After Delete(22)  (m = 11, n = 7, tombstones = 1, alpha = 0.64):
  [ 0: DEL ] [ 1:  44 ] [ 2:  73 ] [ 3:  .  ] [ 4:  59 ] [ 5:  .  ] 
  [ 6:  .  ] [ 7:  18 ] [ 8:  41 ] [ 9:  31 ] [10:  32 ] 
Search(73) succeeded at slot [2] in 7 probes (probed past Tombstone at [0]!)
Inserting key 55 (h1(55) = 0): reuses Tombstone at slot [0] after checking duplicates...
After Insert(55)  (m = 11, n = 8, tombstones = 0, alpha = 0.73):
  [ 0:  55 ] [ 1:  44 ] [ 2:  73 ] [ 3:  .  ] [ 4:  59 ] [ 5:  .  ] 
  [ 6:  .  ] [ 7:  18 ] [ 8:  41 ] [ 9:  31 ] [10:  32 ] 

=== 4. Dynamic Rehashing Demonstration (Starting at m = 7, alpha_max = 0.70) ===
Inserting 10... (n = 1, m = 7, alpha = 0.14)
Inserting 24... (n = 2, m = 7, alpha = 0.29)
Inserting 38... (n = 3, m = 7, alpha = 0.43)
Inserting 17... (n = 4, m = 7, alpha = 0.57)
Inserting 31...
>>> [REHASH TRIGGERED] Load factor exceeded threshold! Resizing m: 7 -> 17 (R = 13) <<<
 (n = 5, m = 17, alpha = 0.29)
Inserting 45... (n = 6, m = 17, alpha = 0.35)
Final Rehashed Table (m = 17, n = 6, tombstones = 0, alpha = 0.35):
  [ 0:  17 ] [ 1:  .  ] [ 2:  .  ] [ 3:  .  ] [ 4:  38 ] [ 5:  .  ] 
  [ 6:  .  ] [ 7:  24 ] [ 8:  .  ] [ 9:  .  ] [10:  10 ] [11:  45 ] 
  [12:  .  ] [13:  .  ] [14:  31 ] [15:  .  ] [16:  .  ] 
```

---

## 10. Master Comparison Tables and Complexity Summary

### 10.1 Master Comparison: Chaining vs. Linear vs. Quadratic vs. Double Hashing

| Feature / Metric | Separate Chaining | Linear Probing | Quadratic Probing | Double Hashing |
| :--- | :--- | :--- | :--- | :--- |
| **Probing / Resolution Formula** | Linked list at $T[h(k)]$ | $(h'(k) + i) \bmod m$ | $(h'(k) + i^2) \bmod m$ | $(h_1(k) + i \cdot h_2(k)) \bmod m$ |
| **Storage Location** | External heap nodes | Inside flat array $T[0..m-1]$ | Inside flat array $T[0..m-1]$ | Inside flat array $T[0..m-1]$ |
| **Legal Load Factor $\alpha = n/m$** | **$\alpha \ge 0$ (can exceed $1$)** | $0 \le \alpha \le 1$ | **$0 \le \alpha \le 0.5$** (for guarantee) | $0 \le \alpha \le 1$ |
| **Recommended Rehash $\alpha_{\max}$** | $0.75 \text{ to } 1.0$ | $0.60 \text{ to } 0.70$ | **$0.50$** | $0.70 \text{ to } 0.80$ |
| **Clustering Vulnerability** | **None** (separate chains) | **Primary Clustering** (severe) | **Secondary Clustering** (mild) | **None** (neither primary nor secondary) |
| **Distinct Probe Sequences** | N/A | $m$ | $m$ | **$m(m - 1) = \Theta(m^2)$** |
| **Deletion Mechanism** | Standard `free(node)` | **Tombstone (`DELETED`)** | **Tombstone (`DELETED`)** | **Tombstone (`DELETED`)** |
| **CPU Cache Locality** | Poor (pointer chasing) | **Best** (adjacent slots in L1 line) | Moderate (small $i^2$ jumps) | Low (large $h_2(k)$ jumps) |
| **Hash Computation Cost** | 1 hash evaluation | 1 hash evaluation | 1 hash evaluation | **2 hash evaluations ($h_1, h_2$)** |

### 10.2 Expected Probe Counts as a Function of Load Factor $\alpha$

| Collision Scheme | Expected Probes (Unsuccessful Search / Insert) | Expected Probes (Successful Search) | At $\alpha = 0.50$ (Unsucc / Succ) | At $\alpha = 0.75$ (Unsucc / Succ) | At $\alpha = 0.90$ (Unsucc / Succ) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Separate Chaining** | $1 + \alpha$ | $1 + \dfrac{\alpha}{2}$ | $1.50$ / $1.25$ | $1.75$ / $1.38$ | $1.90$ / $1.45$ |
| **Linear Probing** | $\dfrac{1}{2}\left(1 + \dfrac{1}{(1-\alpha)^2}\right)$ | $\dfrac{1}{2}\left(1 + \dfrac{1}{1-\alpha}\right)$ | $2.50$ / $1.50$ | $8.50$ / $2.50$ | **$50.50$** / $5.50$ |
| **Double Hashing (Uniform)** | $\dfrac{1}{1 - \alpha}$ | $\dfrac{1}{\alpha}\ln\left(\dfrac{1}{1-\alpha}\right)$ | $2.00$ / $1.39$ | $4.00$ / $1.85$ | $10.00$ / $2.56$ |

---

## 11. Common Errors, Pitfalls, and Edge Cases

### 11.1 Pitfall 1: Setting a Deleted Slot to `EMPTY` Instead of `DELETED` (Tombstone)

```c
/* BUG: Setting deleted slot to SLOT_EMPTY breaks probe chains for keys probed past idx! */
void bad_open_delete(OpenHashTable *ht, int idx) {
    ht->slots[idx].state = SLOT_EMPTY; /* Any key that collided here is now unreachable! */
    ht->n--;
}

/* FIX: Mark slot as SLOT_DELETED (Tombstone) so search() continues probing past it */
void good_open_delete(OpenHashTable *ht, int idx) {
    ht->slots[idx].state = SLOT_DELETED;
    ht->n--;
    ht->tombstones++;
}
```

### 11.2 Pitfall 2: Stopping at the First Tombstone During `insert` Without Checking for Duplicates

If `open_insert` immediately overwrites the first `SLOT_DELETED` tombstone it sees and returns, and the key `k` already exists further along the probe chain, the table ends up containing **two copies of the same key `k`**!

```c
/* BUG: Immediately inserting at the first tombstone without checking rest of probe chain */
if (ht->slots[idx].state == SLOT_DELETED) {
    ht->slots[idx].key = key; /* BUG: What if 'key' is already stored at probe i + 1?! */
    ht->slots[idx].state = SLOT_OCCUPIED;
    return true;
}

/* FIX: Save first_tombstone = idx, continue probing until SLOT_EMPTY or duplicate match! */
if (ht->slots[idx].state == SLOT_DELETED && first_tombstone == -1) {
    first_tombstone = idx; /* Record index, then keep loop running! */
}
```

### 11.3 Pitfall 3: Allowing $h_2(k) == 0$ in Double Hashing (Infinite Loop on $i \cdot 0$)

```c
/* BUG: k % R can evaluate to 0 when k is a multiple of R! */
int bad_h2(int key, int R) {
    return key % R; /* If key = 14 and R = 7, returns 0 -> h(k, i) = h1(k) forever! */
}

/* FIX: Subtract (k % R) from R so result is strictly in [1 .. R] */
int good_h2(int key, int R) {
    int rem = key % R;
    if (rem < 0) rem += R;
    return R - rem; /* Strictly in {1, 2, ..., R} — never 0! */
}
```

### 11.4 Pitfall 4: Using `realloc` Without Recomputing $k \bmod m'$ During Rehashing

```c
/* BUG: Expanding array with realloc leaves keys at old (k % old_m) positions! */
ht->slots = realloc(ht->slots, new_m * sizeof(OpenSlot));
ht->m = new_m; /* Future searches compute k % new_m and fail to find existing keys! */

/* FIX: Allocate a fresh array of size new_m and re-insert every OCCUPIED key using k % new_m */
```

### 11.5 Boundary Conditions Checklist

- [ ] **Full Table in Open Addressing ($n = m$):** Does the probe loop terminate after `i < m` attempts instead of spinning in an infinite `while (true)` loop when all $m$ slots are `OCCUPIED` or `DELETED`?
- [ ] **All-Tombstone Table ($n = 0$, $\text{tombstones} = m$):** Does `open_search` terminate after $m$ probes when no `SLOT_EMPTY` exists anywhere in the table?
- [ ] **Quadratic Probing Load Limit ($\alpha \le 0.5$):** Is rehashing triggered before $\alpha$ exceeds $0.5$ when using Quadratic Probing?
- [ ] **Double Hashing Coprimality:** Is $m$ guaranteed to be prime, and does $h_2(k)$ return a value in $[1, m-1]$?
- [ ] **Tombstone Purging on Rehash:** Does `open_rehash` skip `SLOT_DELETED` entries and reset `ht->tombstones = 0`?

---

## 12. Summary and Revision Checklist

### 12.1 Key Takeaways and Core Invariants

1. **Separate Chaining vs. Open Addressing:** Separate Chaining stores colliding keys in external linked lists ($\alpha$ can exceed $1$, deletion is trivial `free()`, no clustering, but incurs pointer/heap overhead). Open Addressing stores all records inside a single flat array ($0 \le \alpha \le 1$, high cache locality, requires tombstones for deletion).
2. **Linear Probing ($h(k, i) = (h'(k) + i) \bmod m$):** Simplest and most cache-friendly, but suffers from **Primary Clustering** where contiguous runs of occupied slots coalesce and grow at rate $O((1-\alpha)^{-2})$.
3. **Tombstones (`SLOT_DELETED`):** In Open Addressing, deleting a key must mark its slot `DELETED` rather than `EMPTY`; searches skip `DELETED` slots, while insertions reuse the first `DELETED` slot after verifying the key does not appear later in the chain.
4. **Quadratic Probing ($h(k, i) = (h'(k) + i^2) \bmod m$):** Eliminates Primary Clustering (exhibiting only milder **Secondary Clustering** among keys with identical $h'(k)$). Guaranteed to find an empty slot if **$m$ is prime and $\alpha \le 0.5$**.
5. **Double Hashing ($h(k, i) = (h_1(k) + i \cdot h_2(k)) \bmod m$):** Uses a key-dependent step size $h_2(k) \neq 0$ with $\gcd(h_2(k), m) = 1$, eliminating both Primary and Secondary Clustering.
6. **Rehashing:** Resizing requires allocating a new prime table $m' \approx 2m$ and re-inserting every `OCCUPIED` key under $k \bmod m'$ (dropping all tombstones), running in $O(n + m')$ worst-case time and $O(1)$ amortized time per insertion.

### 12.2 Self-Test Revision Questions

1. **Trace Question:** Insert keys $\langle 10, 22, 31, 4, 15, 28, 17, 88, 59 \rangle$ into a hash table of capacity $m = 11$ with $h_1(k) = k \bmod 11$ using (a) Linear Probing, (b) Quadratic Probing ($h(k, i) = (h_1(k) + i^2) \bmod 11$), and (c) Double Hashing with $h_2(k) = 1 + (k \bmod 10)$.
   *(Answer:
   - Linear: `[0: 22, 1: 88, 2: ., 3: ., 4: 4, 5: 15, 6: 28, 7: 17, 8: 59, 9: 31, 10: 10]`
   - Quadratic: `[0: 22, 1: ., 2: 88, 3: 17, 4: 4, 5: ., 6: 28, 7: 59, 8: 15, 9: 31, 10: 10]`
   - Double: `[0: 22, 1: ., 2: 59, 3: 17, 4: 4, 5: 15, 6: 28, 7: 88, 8: ., 9: 31, 10: 10]`).*
2. **Conceptual Proof Question:** Why must $h_2(k)$ never equal $0$ in Double Hashing, and why must $\gcd(h_2(k), m) = 1$?
   *(Answer: If $h_2(k) = 0$, the step size is $0$, so every probe $i$ inspects $h_1(k)$ in an infinite loop. If $\gcd(h_2(k), m) = g > 1$, the probe sequence only visits $m/g < m$ distinct slots before cycling, potentially failing to find empty slots).*
3. **Tombstone Question:** During `open_insert(ht, key, value)`, why can't we stop probing immediately when we see the first `SLOT_DELETED` slot and place `key` there?
   *(Answer: Because `key` might already be stored at a later slot along the probe sequence! Stopping early at the tombstone without probing until `SLOT_EMPTY` or a key match would insert a duplicate copy of `key`).*

### 12.3 Navigation Links

- **Previous Topic:** [04-Hash-Functions-and-Hash-Tables.md](./04-Hash-Functions-and-Hash-Tables.md) — Hash Functions and Hash Tables
- **Current File:** `05-Collision-Handling-and-Rehashing.md` (Unit 4 — Topic 5)
- **Next File (Unit 4 Review):** [06-Unit-4-Review.md](./06-Unit-4-Review.md) — Unit 4 Comprehensive Exam Preparation & Review
