# Unit 3 — Topic 8: Graph Traversals — Breadth-First Search (BFS) and Depth-First Search (DFS)

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [What Is Graph Traversal?](#11-what-is-graph-traversal)
   - 1.2 [Why Graph Traversal Is Harder Than Tree Traversal (Cycles & Disconnection)](#12-why-graph-traversal-is-harder-than-tree-traversal-cycles--disconnection)
   - 1.3 [Relationship to Prior Concepts: Level-Order (Queue) vs. Preorder (Stack)](#13-relationship-to-prior-concepts-level-order-queue-vs-preorder-stack)
   - 1.4 [Learning Objectives](#14-learning-objectives)
2. [Definitions, State Models, and Core Terminology](#2-definitions-state-models-and-core-terminology)
   - 2.1 [The Boolean `visited[]` Array vs. The CLRS Three-Color Model (WHITE / GRAY / BLACK)](#21-the-boolean-visited-array-vs-the-clrs-three-color-model-white--gray--black)
   - 2.2 [Traversal Trees: Breadth-First Tree and Depth-First Forest](#22-traversal-trees-breadth-first-tree-and-depth-first-forest)
   - 2.3 [Core Traversal Terminology Table](#23-core-traversal-terminology-table)
   - 2.4 [Representative Benchmark Graph $G_1$ Used for All Traces](#24-representative-benchmark-graph-g_1-used-for-all-traces)
3. [Breadth-First Search (BFS)](#3-breadth-first-search-bfs)
   - 3.1 [Intuitive Explanation: Concentric Wavefront Exploration via a FIFO Queue](#31-intuitive-explanation-concentric-wavefront-exploration-via-a-fifo-queue)
   - 3.2 [Critical Invariant: Why Vertices MUST Be Marked `visited` When Enqueued (Not When Dequeued!)](#32-critical-invariant-why-vertices-must-be-marked-visited-when-enqueued-not-when-dequeued)
   - 3.3 [Step-by-Step BFS Algorithm and Pseudocode](#33-step-by-step-bfs-algorithm-and-pseudocode)
   - 3.4 [Complete Step-by-Step BFS Trace Table on Benchmark Graph $G_1$](#34-complete-step-by-step-bfs-trace-table-on-benchmark-graph-g_1)
   - 3.5 [Unweighted Shortest-Path Theorem and Path Reconstruction](#35-unweighted-shortest-path-theorem-and-path-reconstruction)
   - 3.6 [BFS Time and Space Complexity Derivation ($O(|V| + |E|)$ vs. $O(|V|^2)$)](#36-bfs-time-and-space-complexity-derivation-ov--e-vs-ov2)
4. [Depth-First Search (DFS)](#4-depth-first-search-dfs)
   - 4.1 [Intuitive Explanation: Deep Backtracking Exploration via a LIFO Stack](#41-intuitive-explanation-deep-backtracking-exploration-via-a-lifo-stack)
   - 4.2 [Recursive DFS vs. Iterative Explicit-Stack DFS (Order Differences & Reverse Push)](#42-recursive-dfs-vs-iterative-explicit-stack-dfs-order-differences--reverse-push)
   - 4.3 [Discovery ($d[u]$) and Finish ($f[u]$) Timestamps and the Parenthesis Theorem](#43-discovery-du-and-finish-fu-timestamps-and-the-parenthesis-theorem)
   - 4.4 [Classification of DFS Edges: Tree, Back, Forward, and Cross Edges](#44-classification-of-dfs-edges-tree-back-forward-and-cross-edges)
   - 4.5 [Complete Step-by-Step DFS Trace Tables (Recursive & Iterative) on $G_1$](#45-complete-step-by-step-dfs-trace-tables-recursive--iterative-on-g_1)
   - 4.6 [Handling Disconnected Graphs: Outer Loop (`dfsFull` / `bfsFull`) and Connected Components](#46-handling-disconnected-graphs-outer-loop-dfsfull--bfsfull-and-connected-components)
   - 4.7 [DFS Time and Space Complexity Derivation ($O(|V| + |E|)$ vs. $O(|V|^2)$)](#47-dfs-time-and-space-complexity-derivation-ov--e-vs-ov2)
5. [Complete C11 Implementation: BFS, Recursive DFS, and Iterative DFS](#5-complete-c11-implementation-bfs-recursive-dfs-and-iterative-dfs)
6. [Comparison Tables and Complexity Summary](#6-comparison-tables-and-complexity-summary)
   - 6.1 [Comprehensive Comparison: Breadth-First Search (BFS) vs. Depth-First Search (DFS)](#61-comprehensive-comparison-breadth-first-search-bfs-vs-depth-first-search-dfs)
   - 6.2 [Asymptotic Complexity Summary Table](#62-asymptotic-complexity-summary-table)
7. [Common Errors, Pitfalls, and Edge Cases](#7-common-errors-pitfalls-and-edge-cases)
   - 7.1 [Pitfall 1: Marking `visited` on Dequeue Instead of Enqueue in BFS](#71-pitfall-1-marking-visited-on-dequeue-instead-of-enqueue-in-bfs)
   - 7.2 [Pitfall 2: Calling `bfs(g, 0)` or `dfs(g, 0)` Only Once on a Disconnected Graph](#72-pitfall-2-calling-bfsg-0-or-dfsg-0-only-once-on-a-disconnected-graph)
   - 7.3 [Pitfall 3: False Cycle Detection in Undirected DFS (Not Ignoring the Immediate Parent)](#73-pitfall-3-false-cycle-detection-in-undirected-dfs-not-ignoring-the-immediate-parent)
   - 7.4 [Pitfall 4: Call-Stack Overflow in Recursive DFS on Deep Graphs ($|V| \ge 10^5$)](#74-pitfall-4-call-stack-overflow-in-recursive-dfs-on-deep-graphs-v-ge-105)
   - 7.5 [Boundary Conditions Checklist](#75-boundary-conditions-checklist)
8. [Summary and Revision Checklist](#8-summary-and-revision-checklist)
   - 8.1 [Key Takeaways and Core Invariants](#81-key-takeaways-and-core-invariants)
   - 8.2 [Self-Test Revision Questions](#82-self-test-revision-questions)
   - 8.3 [Navigation Links](#83-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 What Is Graph Traversal?

**Graph Traversal** (also called *Graph Search*) is the systematic process of visiting every reachable vertex and inspecting every incident edge of a graph $G = (V, E)$ starting from a given source vertex $s \in V$ (or across all vertices if the graph is disconnected), such that each vertex is processed **at most once**.

The two foundational graph traversal algorithms are:
1. **Breadth-First Search (BFS):** Explores the graph outward in **concentric layers (levels)** of increasing unweighted edge distance from the source $s$, governed by a **FIFO Queue**.
2. **Depth-First Search (DFS):** Explores as **deeply as possible** along a single path before backtracking to explore alternative branches, governed by a **LIFO Stack** (either the hardware call stack via recursion or an explicit stack).

```
     BREADTH-FIRST SEARCH (BFS: Wavefront)        DEPTH-FIRST SEARCH (DFS: Plunge & Backtrack)
     =====================================        ============================================
              Level 0:   (0)                                      (0) [1st]
                        /   \                                    /   \
              Level 1: (1)---(2)                       [2nd]   (1)---(2)   [3rd]
                       / \   /                                 / \   /
              Level 2:(3) (4)                          [5th] (3) (4) [4th]
                       |                                      |
              Level 3:(5)                              [6th] (5)

     BFS Order: 0 -> 1, 2 -> 3, 4 -> 5            DFS Order: 0 -> 1 -> 2 -> 4 -> (backtrack) -> 3 -> 5
```

### 1.2 Why Graph Traversal Is Harder Than Tree Traversal (Cycles & Disconnection)

In a binary tree or general rooted tree, standard traversals (`preorder`, `inorder`, `postorder`, `levelOrder`) require **no `visited[]` array** because:
1. A tree has a unique `root` from which **all** nodes are reachable.
2. A tree is strictly **acyclic** and hierarchical—a single downward path exists from the root to any node, so following `left`/`right` child pointers can never revisit a node.

In a general graph $G = (V, E)$, three new hazards arise:
1. **Cycles and Alternate Paths (Infinite Loops):** A cycle such as $0 \to 1 \to 2 \to 0$ will trap a naive tree traversal in an infinite loop unless we explicitly track which vertices have already been seen (`visited[v] == true`).
2. **Convergence (Diamond Paths):** Even in a Directed Acyclic Graph (DAG) with no cycles, two paths $0 \to 1 \to 4$ and $0 \to 2 \to 4$ can converge on vertex $4$. Without `visited[]`, vertex $4$ (and its entire subgraph) would be processed multiple times, causing exponential $O(2^{|V|})$ blowup.
3. **Disconnected Components:** Starting from a single vertex $s$ only visits the connected component containing $s$. To visit every vertex in a disconnected graph, an outer loop over all $u \in V$ is required.

### 1.3 Relationship to Prior Concepts: Level-Order (Queue) vs. Preorder (Stack)

- **BFS on a tree rooted at $r$** is identical to **Level-Order Tree Traversal** (using a FIFO Queue), augmented with a `visited[]` guard so cyclic edges are ignored.
- **DFS on a tree rooted at $r$** is identical to **Preorder / Postorder Tree Traversal** (using recursion or a LIFO Stack), augmented with a `visited[]` guard.

### 1.4 Learning Objectives

After completing this chapter, you will be able to:
- Implement and trace **Breadth-First Search (BFS)** using a FIFO queue and explain why marking vertices `visited` **at enqueue time** is mandatory.
- Prove and apply the **BFS Shortest-Path Property** in unweighted graphs and reconstruct shortest paths via the predecessor array $\pi[v]$.
- Implement and trace both **Recursive DFS** and **Iterative Stack-based DFS**, explaining why their traversal orders can differ and how pushing neighbors in reverse order reconciles them.
- Compute DFS discovery/finish timestamps ($d[u], f[u]$), classify edges into **Tree, Back, Forward, and Cross edges**, handle disconnected graphs, and derive the exact **$O(|V| + |E|)$** (Adjacency List) vs. **$O(|V|^2)$** (Adjacency Matrix) time complexities.

---

## 2. Definitions, State Models, and Core Terminology

### 2.1 The Boolean `visited[]` Array vs. The CLRS Three-Color Model (WHITE / GRAY / BLACK)

In simple implementations, a boolean array `bool visited[V]` suffices: `false` means unvisited, and `true` means discovered. However, many advanced graph algorithms (such as cycle detection in directed graphs) need to distinguish between a vertex that is **currently on the frontier / active call stack** versus one whose exploration is **completely finished**.

The classic textbook model (Cormen, Leiserson, Rivest, and Stein — *CLRS*) assigns each vertex $u \in V$ one of three colors:

| Color State | `visited[]` Equivalent | Meaning in BFS | Meaning in DFS |
| :--- | :--- | :--- | :--- |
| **WHITE (`0`)** | `visited[u] == false` | Undiscovered; not yet enqueued. | Undiscovered; `dfs(u)` not yet called. |
| **GRAY (`1`)** | `visited[u] == true` (active) | Discovered and **currently sitting inside the FIFO Queue** (frontier between explored and unexplored). | Discovered and **currently active on the recursion call stack** ($d[u]$ set, $f[u]$ not yet set). |
| **BLACK (`2`)** | `visited[u] == true` (done) | Dequeued and **all its incident edges have been inspected**. | **Finished**; `dfs(u)` has returned ($f[u]$ set). |

### 2.2 Traversal Trees: Breadth-First Tree and Depth-First Forest

Whenever traversal from vertex $u$ discovers an unvisited (WHITE) neighbor $v$ via edge $(u, v)$, we record $u$ as the **predecessor (or parent)** of $v$:

$$\pi[v] = u$$

The set of all vertices reachable from source $s$ together with the **tree edges** $\{(\pi[v], v) : v \ne s\}$ forms a spanning tree of the reachable component:
- In BFS, this is called the **Breadth-First Tree**, in which every tree path from $s$ to $v$ is a **shortest unweighted path** in $G$.
- In full-graph DFS across all components, these trees form a **Depth-First Forest** of one or more **Depth-First Trees**.

### 2.3 Core Traversal Terminology Table

| Symbol / Term | Meaning |
| :--- | :--- |
| $s \in V$ | Source (starting) vertex of the traversal. |
| $d[v]$ (in BFS) | Shortest unweighted path distance (minimum number of edges) from $s$ to $v$. |
| $\pi[v]$ (`parent[v]`) | Predecessor vertex that first discovered $v$ during BFS or DFS ($\pi[s] = -1$ or $\text{NIL}$). |
| $d[u]$ / $f[u]$ (in DFS) | **Discovery time** (when $u$ turns GRAY) and **Finish time** (when $u$ turns BLACK), integers in $1 \dots 2|V|$. |
| **Tree Edge** | An edge $(u, v)$ through which vertex $v$ is discovered for the first time ($v$ was WHITE). |
| **Back Edge** | An edge $(u, v)$ connecting $u$ to an **active ancestor** $v$ on the DFS stack ($v$ is GRAY). Indicates a **cycle**! |

### 2.4 Representative Benchmark Graph $G_1$ Used for All Traces

All step-by-step traces and C programs in this chapter use the following connected, undirected, unweighted graph $G_1 = (V, E)$ with $|V| = 6$ vertices and $|E| = 7$ edges (with adjacency lists sorted in ascending vertex order):

```
         BENCHMARK GRAPH G1 (|V| = 6, |E| = 7)           SORTED ADJACENCY LISTS OF G1
         =====================================           ============================
                 (0)-----------(1)                       head[0] -> 1 -> 2
                  |           / | \                      head[1] -> 0 -> 2 -> 3 -> 4
                  |         /   |   \                    head[2] -> 0 -> 1 -> 4
                  |       /     |    (3)---(5)           head[3] -> 1 -> 5
                  |     /       |                        head[4] -> 1 -> 2
                  |   /         |                        head[5] -> 3
                 (2)-----------(4)
```

---

## 3. Breadth-First Search (BFS)

### 3.1 Intuitive Explanation: Concentric Wavefront Exploration via a FIFO Queue

Imagine dropping a pebble at source vertex $s$ into a pond of connected channels. The ripple expands uniformly in all directions at the same speed:
- **Wave 0 ($d = 0$):** Visit the starting vertex $s$ itself.
- **Wave 1 ($d = 1$):** Visit all immediate neighbors of $s$ (vertices at distance $1$ edge away).
- **Wave 2 ($d = 2$):** Visit all unvisited neighbors of Wave 1 vertices (vertices at distance $2$ edges away).
- **Wave $k$ ($d = k$):** Visit all unvisited neighbors of Wave $k-1$ vertices before visiting *any* vertex at distance $k + 1$.

A **FIFO Queue** naturally enforces this concentric ordering: because Wave $k-1$ vertices were enqueued before Wave $k$ vertices, every Wave $k-1$ vertex is dequeued and expanded before the first Wave $k$ vertex reaches the front of the queue.

### 3.2 Critical Invariant: Why Vertices MUST Be Marked `visited` When Enqueued (Not When Dequeued!)

One of the most frequent implementation bugs in BFS is waiting until a vertex is **dequeued** to set `visited[v] = true`.

**Why is marking on dequeue wrong?**
Suppose vertex $0$ is dequeued in $G_1$. Its neighbors $1$ and $2$ are both unvisited, so both $1$ and $2$ are pushed into the queue: `Queue = [1, 2]`.
If we did *not* mark $2$ as `visited` when enqueuing it, then on the very next step when vertex $1$ is dequeued, $1$ inspects its neighbor $2$ (via edge $\{1, 2\}$). Because $2$ is still sitting in the queue waiting to be dequeued, `visited[2]` would still be `false`! Vertex $1$ would **enqueue a duplicate copy of $2$** (`Queue = [2, 2, 3, 4]`), corrupting `parent[2]` and `dist[2]`, and potentially overflowing a fixed-size queue of size $|V|$!

> **Golden Rule of BFS:** Always set `visited[v] = true` (turn $v$ GRAY) at the **exact instant $v$ is enqueued**. This guarantees that every vertex enters the FIFO queue **at most once**.

### 3.3 Step-by-Step BFS Algorithm and Pseudocode

```text
ALGORITHM: BFS(G, s)
Input:  Graph G = (V, E), starting source vertex s
Output: Shortest unweighted distances dist[0..V-1] and BFS tree parents parent[0..V-1]

1. for each vertex u in V do
2.     visited[u] <- false          // Color WHITE
3.     dist[u]    <- INFINITY       // Or -1 (unreachable sentinel)
4.     parent[u]  <- -1             // NIL parent
5. end for
6.
7. visited[s] <- true               // Mark source visited BEFORE / AT enqueue!
8. dist[s]    <- 0
9. Q <- empty FIFO Queue
10. ENQUEUE(Q, s)
11.
12. while not IS-EMPTY(Q) do
13.     u <- DEQUEUE(Q)
14.     PROCESS(u)                  // Print or record u in BFS order
15.     for each neighbor v of u do
16.         if visited[v] == false then
17.             visited[v] <- true  // CRITICAL: Mark visited upon ENQUEUE
18.             dist[v]    <- dist[u] + 1
19.             parent[v]  <- u
20.             ENQUEUE(Q, v)
21.         end if
22.     end for
23. end while
```

### 3.4 Complete Step-by-Step BFS Trace Table on Benchmark Graph $G_1$

Starting BFS at source $s = 0$ on $G_1$ (`head[0]={1,2}`, `head[1]={0,2,3,4}`, `head[2]={0,1,4}`, `head[3]={1,5}`, `head[4]={1,2}`, `head[5]={3}`):

| Step | Dequeued Vertex `u` | Neighbors of `u` Inspected | Newly Enqueued | Queue Contents (Front $\to$ Rear) | `visited[0..5]` (`T`/`F`) | `dist[0..5]` | `parent[0..5]` |
| :---: | :---: | :--- | :--- | :--- | :---: | :---: | :---: |
| **Init** | — | Enqueue source `0` | `0` | `[0]` | `[T, F, F, F, F, F]` | `[0, ∞, ∞, ∞, ∞, ∞]` | `[-1, -1, -1, -1, -1, -1]` |
| **1** | **`0`** | `1` (unvisited $\to$ enq), `2` (unvisited $\to$ enq) | `1, 2` | `[1, 2]` | `[T, T, T, F, F, F]` | `[0, 1, 1, ∞, ∞, ∞]` | `[-1, 0, 0, -1, -1, -1]` |
| **2** | **`1`** | `0` (vis), `2` (**already vis in Q!**), `3` (enq), `4` (enq) | `3, 4` | `[2, 3, 4]` | `[T, T, T, T, T, F]` | `[0, 1, 1, 2, 2, ∞]` | `[-1, 0, 0, 1, 1, -1]` |
| **3** | **`2`** | `0` (vis), `1` (vis), `4` (vis) | *none* | `[3, 4]` | `[T, T, T, T, T, F]` | `[0, 1, 1, 2, 2, ∞]` | `[-1, 0, 0, 1, 1, -1]` |
| **4** | **`3`** | `1` (vis), `5` (unvisited $\to$ enq) | `5` | `[4, 5]` | `[T, T, T, T, T, T]` | `[0, 1, 1, 2, 2, 3]` | `[-1, 0, 0, 1, 1, 3]` |
| **5** | **`4`** | `1` (vis), `2` (vis) | *none* | `[5]` | `[T, T, T, T, T, T]` | `[0, 1, 1, 2, 2, 3]` | `[-1, 0, 0, 1, 1, 3]` |
| **6** | **`5`** | `3` (vis) | *none* | `[]` *(Empty)* | `[T, T, T, T, T, T]` | `[0, 1, 1, 2, 2, 3]` | `[-1, 0, 0, 1, 1, 3]` |

**Final BFS Traversal Order:** `0, 1, 2, 3, 4, 5`

### 3.5 Unweighted Shortest-Path Theorem and Path Reconstruction

#### Theorem (BFS Shortest-Path Property)

In any unweighted directed or undirected graph $G = (V, E)$, Breadth-First Search starting from source $s$ discovers every reachable vertex $v$ at a distance value `dist[v]` equal to $\delta(s, v)$, the **minimum number of edges** on any path from $s$ to $v$. Furthermore, the predecessor chain backward from $v$ to $s$:

$$v \leftarrow \text{parent}[v] \leftarrow \text{parent}[\text{parent}[v]] \leftarrow \dots \leftarrow s$$

traces a shortest path from $s$ to $v$ in reverse.

- *Example in $G_1$:* For target $v = 5$, following `parent[]` gives $5 \leftarrow \text{parent}[5]=3 \leftarrow \text{parent}[3]=1 \leftarrow \text{parent}[1]=0$. Reversing yields the unique shortest path of length `dist[5] = 3`:
  $$0 \to 1 \to 3 \to 5$$

### 3.6 BFS Time and Space Complexity Derivation ($O(|V| + |E|)$ vs. $O(|V|^2)$)

Why do we write the time complexity of BFS as **$O(|V| + |E|)$** (rather than $O(|V| \cdot |E|)$)? Let us perform an exact aggregate accounting of work done:

1. **Initialization & Queue Operations ($\Theta(|V|)$):**
   - Initializing `visited[]`, `dist[]`, and `parent[]` takes $\Theta(|V|)$ time.
   - Because each vertex is marked `visited` upon enqueue and is never enqueued again, each vertex $u \in V$ is enqueued **at most once** and dequeued **at most once**. Since each `enqueue` and `dequeue` takes $O(1)$ time, total queue manipulation across the entire algorithm is $O(|V|)$.
2. **Scanning Incident Edges of Dequeued Vertices:**
   - **Case A — Adjacency List ($\Theta(|E|)$ total):** When vertex $u$ is dequeued, the inner `for` loop scans its linked list `head[u]` of length $\deg(u)$ (or $\deg^+(u)$). Because $u$ is dequeued at most once, its adjacency list is scanned **at most once**. By the Handshaking Lemma, the total work spent in the inner loop across the entire traversal is:
     $$\sum_{u \in V} O(\deg(u)) = O(2|E|) = O(|E|)$$
     Combining vertex initialization/queue work $O(|V|)$ with edge inspection $O(|E|)$ yields:
     $$T_{\text{BFS, List}} = O(|V| + |E|)$$
   - **Case B — Adjacency Matrix ($\Theta(|V|^2)$ total):** When vertex $u$ is dequeued, finding its neighbors requires scanning all $|V|$ entries of matrix row `adj[u][0..V-1]`, which costs $\Theta(|V|)$ per dequeued vertex. Across all $|V|$ vertices:
     $$T_{\text{BFS, Matrix}} = \Theta(|V|) + \sum_{u \in V} \Theta(|V|) = \Theta(|V|^2)$$
3. **Auxiliary Space Complexity:** $\Theta(|V|)$ for the `visited[]`, `dist[]`, `parent[]` arrays and the FIFO `queue[]` (which holds at most $|V|$ vertices at any instant).

---

## 4. Depth-First Search (DFS)

### 4.1 Intuitive Explanation: Deep Backtracking Exploration via a LIFO Stack

Whereas BFS explores broadly across all neighbors before going deeper, **Depth-First Search (DFS)** mimics solving a physical maze with a piece of chalk and a ball of string:
1. From the current vertex $u$, immediately move to the **first unvisited neighbor** $v$ and recursively explore from $v$.
2. Continue plunging deeper along a single path of unvisited vertices until reaching a **dead end**—a vertex whose neighbors have *all* already been visited.
3. **Backtrack** one step to the parent vertex that discovered the dead-end vertex, and try its next unvisited neighbor.

### 4.2 Recursive DFS vs. Iterative Explicit-Stack DFS (Order Differences & Reverse Push)

DFS can be written in two ways:
1. **Recursive DFS (Implicit OS Call Stack):**
   ```text
   ALGORITHM: DFS-RECURSIVE(G, u)
   1. visited[u] <- true
   2. PROCESS(u)
   3. for each neighbor v of u do
   4.     if visited[v] == false then
   5.         parent[v] <- u
   6.         DFS-RECURSIVE(G, v)
   7.     end if
   8. end for
   ```
2. **Iterative DFS (Explicit LIFO Stack):**
   ```text
   ALGORITHM: DFS-ITERATIVE(G, s)
   1. S <- empty LIFO Stack
   2. PUSH(S, s)
   3. while not IS-EMPTY(S) do
   4.     u <- POP(S)
   5.     if visited[u] == false then      // Check visited WHEN POPPED!
   6.         visited[u] <- true
   7.         PROCESS(u)
   8.         for each neighbor v of u (in REVERSE order) do
   9.             if visited[v] == false then
   10.                PUSH(S, v)
   11.            end if
   12.        end for
   13.    end if
   14. end while
   ```

#### Two Subtle Distinctions Between Recursive DFS and Iterative Stack DFS

1. **Why Neighbor Push Order Matters (Forward Push vs. Reverse Push):**
   - In **Recursive DFS** at vertex $0$ with neighbors $[1, 2]$, the loop inspects $1$ first and **immediately calls `dfs(1)`**, pausing the loop before $2$ is even looked at. Thus $1$ is visited before $2$.
   - In **Iterative DFS**, if we push neighbors of $0$ onto the LIFO stack in forward order (`PUSH(1)`, then `PUSH(2)`), then **$2$ sits on top of the stack** and gets popped *first*!
     - Forward-push order on $G_1$: `0, 2, 4, 1, 3, 5` (still a valid DFS, exploring rightmost neighbor first!).
     - Reverse-push order on $G_1$ (`PUSH(2)`, then `PUSH(1)` so $1$ is on top): `0, 1, 2, 4, 3, 5` (matches Recursive DFS **identically**!).
2. **When `visited[u]` Must Be Checked and Marked in Iterative DFS:**
   - Unlike BFS (which marks `visited` on enqueue), standard Iterative DFS marks `visited[u] = true` **when $u$ is popped from the stack** (Lines 5–6). Why? Because if vertex $2$ is pushed onto the stack when expanding $0$, and later vertex $1$ also has an edge to $2$, a true depth-first search from $1$ must visit $2$ immediately via $1 \to 2$ rather than leaving $2$ buried at the bottom of the stack! Consequently, $2$ may be pushed again by $1$, and the `if (!visited[u])` check upon popping discards the older, stale copy at the bottom of the stack.

### 4.3 Discovery ($d[u]$) and Finish ($f[u]$) Timestamps and the Parenthesis Theorem

By maintaining a global integer counter `timer` initialized to $0$ and incremented every time a vertex changes color:
- **Discovery time $d[u]$:** Recorded when $u$ is first entered (WHITE $\to$ GRAY).
- **Finish time $f[u]$:** Recorded when $u$'s adjacency list has been completely exhausted and `dfs(u)` is about to return (GRAY $\to$ BLACK).

Every vertex satisfies $1 \le d[u] < f[u] \le 2|V|$.
- **Parenthesis Theorem:** For any two vertices $u$ and $v$ in a DFS forest, the intervals $[d[u], f[u]]$ and $[d[v], f[v]]$ are either **completely disjoint** (neither is a descendant of the other) or **one is strictly contained inside the other** ($d[u] < d[v] < f[v] < f[u] \iff v$ is a proper descendant of $u$ in the DFS tree).

### 4.4 Classification of DFS Edges: Tree, Back, Forward, and Cross Edges

When DFS is at vertex $u$ (which is currently **GRAY**) and inspects edge $(u, v)$, the color of target vertex $v$ immediately classifies $(u, v)$ into one of four categories:

| Edge Type | Color of Target $v$ When Inspected | Timestamp Relation | Structural Meaning & Applications |
| :--- | :--- | :--- | :--- |
| **1. Tree Edge** | **WHITE** (Unvisited) | $d[u] < d[v] < f[v] < f[u]$ | $v$ is discovered for the first time via $(u, v)$; forms the DFS spanning forest. |
| **2. Back Edge** | **GRAY** (Active on call stack) | $d[v] \le d[u] < f[u] \le f[v]$ | Points from $u$ back to an **active ancestor** $v$. **Proves a cycle exists!** |
| **3. Forward Edge** | **BLACK** (Finished, with $d[u] < d[v]$) | $d[u] < d[v] < f[v] < f[u]$ | Non-tree edge pointing from $u$ to a finished proper descendant $v$ *(Digraphs only)*. |
| **4. Cross Edge** | **BLACK** (Finished, with $d[v] < d[u]$) | $d[v] < f[v] < d[u] < f[u]$ | Points between two vertices in separate branches or DFS trees *(Digraphs only)*. |

> **Important Exam Fact:** In an **undirected** graph, every edge is either a **Tree Edge** or a **Back Edge** (Forward and Cross edges can never occur in an undirected DFS). Furthermore, a directed or undirected graph has a cycle **if and only if** DFS encounters at least one **Back Edge** (excluding the trivial undirected edge back to the immediate parent $\pi[u]$).

### 4.5 Complete Step-by-Step DFS Trace Tables (Recursive & Iterative) on $G_1$

#### A. Recursive DFS Trace on $G_1$ Starting at $0$

```
  DFS TREE + BACK EDGES ON G1                    TIMESTAMP INTERVALS [d[u], f[u]]
  ===========================                    ================================
         (0) [1/12]                              0: [1 ........................ 12]
          |  \                                   1:   [2 .................... 11]
     Tree |   \ Back Edge (4->2, 2->0)           2:     [3 ........ 6]
          v    \                                 4:       [4 .. 5]
         (1) [2/11]                              3:             [7 ...... 10]
        /   \                                    5:               [8 .. 9]
  Tree v     v Tree
 [3/6](2)   (3) [7/10]
       |     |
  Tree v     v Tree
 [4/5](4)   (5) [8/9]
```

| Step / Timer | Action | Current Vertex | Active Call Stack (Bottom $\to$ Top) | `visited[0..5]` | Edge Classification / Notes |
| :---: | :--- | :---: | :--- | :---: | :--- |
| **t = 1** | Discover `0` | `0` | `[0]` | `[T, F, F, F, F, F]` | `d[0] = 1`; inspect `head[0]`: first neighbor is `1` (WHITE) |
| **t = 2** | Tree edge `0->1` | `1` | `[0, 1]` | `[T, T, F, F, F, F]` | `d[1] = 2`; `0` is parent; next neighbor of `1` is `2` (WHITE) |
| **t = 3** | Tree edge `1->2` | `2` | `[0, 1, 2]` | `[T, T, T, F, F, F]` | `d[2] = 3`; neighbor `0` is GRAY (`0 != parent[2]=1` $\implies$ **Back Edge `2->0`**); `1` is parent; next is `4` (WHITE) |
| **t = 4** | Tree edge `2->4` | `4` | `[0, 1, 2, 4]` | `[T, T, T, F, T, F]` | `d[4] = 4`; neighbor `1` is GRAY (`1 != parent[4]=2` $\implies$ **Back Edge `4->1`**); `2` is parent |
| **t = 5** | Finish `4` | `4` | `[0, 1, 2]` | `[T, T, T, F, T, F]` | `f[4] = 5`; backtrack from `4` to `2` |
| **t = 6** | Finish `2` | `2` | `[0, 1]` | `[T, T, T, F, T, F]` | `f[2] = 6`; backtrack from `2` to `1`; next unvisited of `1` is `3` |
| **t = 7** | Tree edge `1->3` | `3` | `[0, 1, 3]` | `[T, T, T, T, T, F]` | `d[3] = 7`; neighbor `1` is parent; next is `5` (WHITE) |
| **t = 8** | Tree edge `3->5` | `5` | `[0, 1, 3, 5]` | `[T, T, T, T, T, T]` | `d[5] = 8`; only neighbor `3` is parent (dead end) |
| **t = 9** | Finish `5` | `5` | `[0, 1, 3]` | `[T, T, T, T, T, T]` | `f[5] = 9`; backtrack to `3` |
| **t = 10** | Finish `3` | `3` | `[0, 1]` | `[T, T, T, T, T, T]` | `f[3] = 10`; backtrack to `1` (`4` already visited) |
| **t = 11** | Finish `1` | `1` | `[0]` | `[T, T, T, T, T, T]` | `f[1] = 11`; backtrack to `0` (`2` already visited) |
| **t = 12** | Finish `0` | `0` | `[]` *(Empty)* | `[T, T, T, T, T, T]` | `f[0] = 12`; traversal complete |

**Recursive DFS Order:** `0, 1, 2, 4, 3, 5`

#### B. Iterative Explicit-Stack DFS Trace on $G_1$ (Reverse Neighbor Push)

| Step | Popped Vertex `u` | Action Taken | Unvisited Neighbors Pushed (Reverse Order) | Explicit Stack Contents (Bottom $\to$ Top) | `visited[0..5]` |
| :---: | :---: | :--- | :--- | :--- | :---: |
| **Init** | — | Push start `0` | `0` | `[0]` | `[F, F, F, F, F, F]` |
| **1** | **`0`** | Visit `0` | Push `2`, then `1` | `[2, 1]` | `[T, F, F, F, F, F]` |
| **2** | **`1`** | Visit `1` | Push `4`, `3`, `2` | `[2, 4, 3, 2]` | `[T, T, F, F, F, F]` |
| **3** | **`2`** | Visit `2` | Push `4` | `[2, 4, 3, 4]` | `[T, T, T, F, F, F]` |
| **4** | **`4`** | Visit `4` | *none* (`1, 2` visited) | `[2, 4, 3]` | `[T, T, T, F, T, F]` |
| **5** | **`3`** | Visit `3` | Push `5` | `[2, 4, 5]` | `[T, T, T, T, T, F]` |
| **6** | **`5`** | Visit `5` | *none* (`3` visited) | `[2, 4]` | `[T, T, T, T, T, T]` |
| **7** | `4` | Skip (`visited[4]==T`) | — | `[2]` | `[T, T, T, T, T, T]` |
| **8** | `2` | Skip (`visited[2]==T`) | — | `[]` *(Empty)* | `[T, T, T, T, T, T]` |

**Iterative DFS Order (Reverse Neighbor Push):** `0, 1, 2, 4, 3, 5` (identical to Recursive DFS!).

### 4.6 Handling Disconnected Graphs: Outer Loop (`dfsFull` / `bfsFull`) and Connected Components

Calling `bfs(G, 0)` or `dfs(G, 0)` only visits vertices reachable from vertex `0`. If $G$ is disconnected (or is a directed graph where `0` cannot reach all vertices), unvisited vertices remain untouched.

To guarantee visiting **every** vertex in $G$ and to count the number of **Connected Components** in an undirected graph, wrap the traversal in an outer loop over all $u \in V$:

```text
ALGORITHM: DFS-FULL(G)
1. for u <- 0 to |V| - 1 do
2.     visited[u] <- false
3. end for
4. component_count <- 0
5. for u <- 0 to |V| - 1 do
6.     if visited[u] == false then
7.         component_count <- component_count + 1
8.         DFS-RECURSIVE(G, u)     // Or BFS(G, u)
9.     end if
10. end for
```

### 4.7 DFS Time and Space Complexity Derivation ($O(|V| + |E|)$ vs. $O(|V|^2)$)

1. **Vertex Work ($\Theta(|V|)$):** Because `DFS-RECURSIVE(G, u)` is called only when `visited[u] == false` and immediately sets `visited[u] = true`, `DFS-RECURSIVE` is invoked **exactly once per vertex** $u \in V$. The outer loop in `DFS-FULL` also runs $|V|$ times $\implies \Theta(|V|)$ vertex overhead.
2. **Edge Inspection Work:**
   - **Adjacency List ($\Theta(|E|)$):** Inside `DFS-RECURSIVE(G, u)`, the `for` loop iterates over `head[u]` of length $\deg(u)$ exactly once during the entire execution. By the Handshaking Lemma:
     $$\sum_{u \in V} \Theta(\deg(u)) = \Theta(|E|) \implies T_{\text{DFS, List}} = \Theta(|V| + |E|)$$
   - **Adjacency Matrix ($\Theta(|V|^2)$):** Inside `DFS-RECURSIVE(G, u)`, scanning row `u` checks all $|V|$ columns:
     $$\sum_{u \in V} \Theta(|V|) = \Theta(|V|^2) \implies T_{\text{DFS, Matrix}} = \Theta(|V|^2)$$
3. **Auxiliary Space Complexity:** $\Theta(|V|)$ in the worst case for the `visited[]` array and the recursion call stack (which reaches depth $|V|$ on a skewed line/path graph $0 - 1 - 2 - \dots - (|V|-1)$).

---

## 5. Complete C11 Implementation: BFS, Recursive DFS, and Iterative DFS

The following self-contained C11 program implements **BFS**, **Recursive DFS** (with discovery/finish timestamps), and **Iterative Stack DFS** (comparing both forward neighbor push and reverse neighbor push) on both the **Adjacency List** and **Adjacency Matrix** of benchmark graph $G_1$, followed by connected-component discovery on a disconnected graph.

```c
/**
 * @file dfs_and_bfs.c
 * @brief Complete C11 implementation of BFS, Recursive DFS, Iterative DFS,
 *        Shortest-Path Reconstruction, and Connected Components on both
 *        Adjacency List and Adjacency Matrix representations.
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_V 10
#define INF_DIST -1

/* =========================================================================
 * GRAPH REPRESENTATIONS (ADJACENCY LIST & ADJACENCY MATRIX)
 * ========================================================================= */
typedef struct AdjNode {
    int dest;
    struct AdjNode *next;
} AdjNode;

typedef struct {
    int V;
    int E;
    AdjNode *head[MAX_V];
    int mat[MAX_V][MAX_V];
} DualGraph;

void initDualGraph(DualGraph *g, int v) {
    g->V = v;
    g->E = 0;
    for (int i = 0; i < v; i++) {
        g->head[i] = NULL;
        for (int j = 0; j < v; j++) {
            g->mat[i][j] = 0;
        }
    }
}

static void insertSorted(AdjNode **head_ref, int dest) {
    AdjNode *new_node = (AdjNode *)malloc(sizeof(AdjNode));
    if (!new_node) {
        return;
    }
    new_node->dest = dest;
    new_node->next = NULL;

    if (*head_ref == NULL || (*head_ref)->dest >= dest) {
        new_node->next = *head_ref;
        *head_ref = new_node;
        return;
    }
    AdjNode *curr = *head_ref;
    while (curr->next != NULL && curr->next->dest < dest) {
        curr = curr->next;
    }
    new_node->next = curr->next;
    curr->next = new_node;
}

void addUndirectedEdge(DualGraph *g, int u, int v) {
    g->mat[u][v] = 1;
    g->mat[v][u] = 1;
    insertSorted(&g->head[u], v);
    insertSorted(&g->head[v], u);
    g->E++;
}

void freeDualGraph(DualGraph *g) {
    for (int i = 0; i < g->V; i++) {
        AdjNode *curr = g->head[i];
        while (curr != NULL) {
            AdjNode *tmp = curr;
            curr = curr->next;
            free(tmp);
        }
        g->head[i] = NULL;
    }
}

/* =========================================================================
 * 1. BREADTH-FIRST SEARCH (BFS) - ADJACENCY LIST & ADJACENCY MATRIX
 * ========================================================================= */
void printShortestPath(const int parent[], int start, int target) {
    int path[MAX_V];
    int len = 0;
    for (int curr = target; curr != -1; curr = parent[curr]) {
        path[len++] = curr;
    }
    printf("  Shortest Path (%d -> %d): ", start, target);
    for (int i = len - 1; i >= 0; i--) {
        printf("%d%s", path[i], (i > 0) ? " -> " : "\n");
    }
}

void bfsList(const DualGraph *g, int start) {
    bool visited[MAX_V] = {false};
    int dist[MAX_V];
    int parent[MAX_V];
    int queue[MAX_V];
    int front = 0, rear = 0;

    for (int i = 0; i < g->V; i++) {
        dist[i] = INF_DIST;
        parent[i] = -1;
    }

    /* CRITICAL INVARIANT: Mark visited WHEN ENQUEUED (not when dequeued!) */
    visited[start] = true;
    dist[start] = 0;
    queue[rear++] = start;

    printf("=== 1. BFS on Adjacency List (start = %d) ===\n  Order: ", start);
    while (front < rear) {
        int u = queue[front++];
        printf("%d ", u);

        for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
            int v = curr->dest;
            if (!visited[v]) {
                visited[v] = true;
                dist[v] = dist[u] + 1;
                parent[v] = u;
                queue[rear++] = v;
            }
        }
    }
    printf("\n  Vertex | Dist d[v] | Parent pi[v]\n");
    printf("  -------|-----------|-------------\n");
    for (int v = 0; v < g->V; v++) {
        printf("    %2d   |    %2d     |     %2d\n", v, dist[v], parent[v]);
    }
    printShortestPath(parent, start, 5);
    printf("\n");
}

void bfsMatrix(const DualGraph *g, int start) {
    bool visited[MAX_V] = {false};
    int queue[MAX_V];
    int front = 0, rear = 0;

    visited[start] = true;
    queue[rear++] = start;

    printf("=== 2. BFS on Adjacency Matrix (start = %d) ===\n  Order: ", start);
    while (front < rear) {
        int u = queue[front++];
        printf("%d ", u);

        for (int v = 0; v < g->V; v++) {
            if (g->mat[u][v] != 0 && !visited[v]) {
                visited[v] = true;
                queue[rear++] = v;
            }
        }
    }
    printf("\n\n");
}

/* =========================================================================
 * 2. DEPTH-FIRST SEARCH (DFS) - RECURSIVE & ITERATIVE
 * ========================================================================= */
static void dfsRecListHelper(const DualGraph *g, int u, bool visited[],
                             int disc[], int fin[], int *timer) {
    visited[u] = true;
    disc[u] = ++(*timer);
    printf("%d ", u);

    for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
        int v = curr->dest;
        if (!visited[v]) {
            dfsRecListHelper(g, v, visited, disc, fin, timer);
        }
    }
    fin[u] = ++(*timer);
}

void dfsRecursiveList(const DualGraph *g, int start) {
    bool visited[MAX_V] = {false};
    int disc[MAX_V] = {0};
    int fin[MAX_V] = {0};
    int timer = 0;

    printf("=== 3. Recursive DFS on Adjacency List (start = %d) ===\n  Order: ", start);
    dfsRecListHelper(g, start, visited, disc, fin, &timer);
    printf("\n  Vertex | Discovery d[v] | Finish f[v]\n");
    printf("  -------|----------------|------------\n");
    for (int v = 0; v < g->V; v++) {
        printf("    %2d   |       %2d       |     %2d\n", v, disc[v], fin[v]);
    }
    printf("\n");
}

static void dfsRecMatrixHelper(const DualGraph *g, int u, bool visited[]) {
    visited[u] = true;
    printf("%d ", u);
    for (int v = 0; v < g->V; v++) {
        if (g->mat[u][v] != 0 && !visited[v]) {
            dfsRecMatrixHelper(g, v, visited);
        }
    }
}

void dfsRecursiveMatrix(const DualGraph *g, int start) {
    bool visited[MAX_V] = {false};
    printf("=== 4. Recursive DFS on Adjacency Matrix (start = %d) ===\n  Order: ", start);
    dfsRecMatrixHelper(g, start, visited);
    printf("\n\n");
}

void dfsIterativeForwardPush(const DualGraph *g, int start) {
    bool visited[MAX_V] = {false};
    int stack[MAX_V * MAX_V];
    int top = -1;

    stack[++top] = start;
    printf("=== 5a. Iterative DFS (Forward Neighbor Push) ===\n  Order: ");
    while (top >= 0) {
        int u = stack[top--];
        if (visited[u]) {
            continue;
        }
        visited[u] = true;
        printf("%d ", u);

        for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
            if (!visited[curr->dest]) {
                stack[++top] = curr->dest;
            }
        }
    }
    printf("\n");
}

void dfsIterativeReversePush(const DualGraph *g, int start) {
    bool visited[MAX_V] = {false};
    int stack[MAX_V * MAX_V];
    int top = -1;

    stack[++top] = start;
    printf("=== 5b. Iterative DFS (Reverse Neighbor Push - Matches Recursive!) ===\n  Order: ");

    while (top >= 0) {
        int u = stack[top--];
        if (visited[u]) {
            continue;
        }
        visited[u] = true;
        printf("%d ", u);

        int neighbors[MAX_V];
        int count = 0;
        for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
            if (!visited[curr->dest]) {
                neighbors[count++] = curr->dest;
            }
        }
        for (int i = count - 1; i >= 0; i--) {
            stack[++top] = neighbors[i];
        }
    }
    printf("\n\n");
}

/* =========================================================================
 * 3. HANDLING DISCONNECTED GRAPHS (CONNECTED COMPONENTS)
 * ========================================================================= */
void dfsFullConnectedComponents(const DualGraph *g) {
    bool visited[MAX_V] = {false};
    int disc[MAX_V] = {0}, fin[MAX_V] = {0};
    int timer = 0;
    int comp_count = 0;

    printf("=== 6. Full-Graph DFS on Disconnected Graph (Connected Components) ===\n");
    for (int u = 0; u < g->V; u++) {
        if (!visited[u]) {
            comp_count++;
            printf("  Component %d: ", comp_count);
            dfsRecListHelper(g, u, visited, disc, fin, &timer);
            printf("\n");
        }
    }
    printf("  Total Connected Components = %d\n", comp_count);
}

int main(void) {
    /* Representative Connected Graph G1 (V = 6, E = 7)
     * Edges: {0,1}, {0,2}, {1,2}, {1,3}, {1,4}, {2,4}, {3,5}
     */
    DualGraph g1;
    initDualGraph(&g1, 6);
    const int edges[][2] = {
        {0, 1}, {0, 2}, {1, 2}, {1, 3}, {1, 4}, {2, 4}, {3, 5}
    };
    for (int i = 0; i < 7; i++) {
        addUndirectedEdge(&g1, edges[i][0], edges[i][1]);
    }

    bfsList(&g1, 0);
    bfsMatrix(&g1, 0);
    dfsRecursiveList(&g1, 0);
    dfsRecursiveMatrix(&g1, 0);
    dfsIterativeForwardPush(&g1, 0);
    dfsIterativeReversePush(&g1, 0);
    freeDualGraph(&g1);

    /* Disconnected Graph G_disc (V = 7, E = 5)
     * Component 1: {0, 1, 2, 3} with edges {0,1}, {1,2}, {2,3}
     * Component 2: {4, 5, 6} with edges {4,5}, {5,6}
     */
    DualGraph g_disc;
    initDualGraph(&g_disc, 7);
    addUndirectedEdge(&g_disc, 0, 1);
    addUndirectedEdge(&g_disc, 1, 2);
    addUndirectedEdge(&g_disc, 2, 3);
    addUndirectedEdge(&g_disc, 4, 5);
    addUndirectedEdge(&g_disc, 5, 6);

    dfsFullConnectedComponents(&g_disc);
    freeDualGraph(&g_disc);

    return 0;
}
```

#### Program Output:

```text
=== 1. BFS on Adjacency List (start = 0) ===
  Order: 0 1 2 3 4 5 
  Vertex | Dist d[v] | Parent pi[v]
  -------|-----------|-------------
     0   |     0     |     -1
     1   |     1     |      0
     2   |     1     |      0
     3   |     2     |      1
     4   |     2     |      1
     5   |     3     |      3
  Shortest Path (0 -> 5): 0 -> 1 -> 3 -> 5

=== 2. BFS on Adjacency Matrix (start = 0) ===
  Order: 0 1 2 3 4 5 

=== 3. Recursive DFS on Adjacency List (start = 0) ===
  Order: 0 1 2 4 3 5 
  Vertex | Discovery d[v] | Finish f[v]
  -------|----------------|------------
     0   |        1       |     12
     1   |        2       |     11
     2   |        3       |      6
     3   |        7       |     10
     4   |        4       |      5
     5   |        8       |      9

=== 4. Recursive DFS on Adjacency Matrix (start = 0) ===
  Order: 0 1 2 4 3 5 

=== 5a. Iterative DFS (Forward Neighbor Push) ===
  Order: 0 2 4 1 3 5 
=== 5b. Iterative DFS (Reverse Neighbor Push - Matches Recursive!) ===
  Order: 0 1 2 4 3 5 

=== 6. Full-Graph DFS on Disconnected Graph (Connected Components) ===
  Component 1: 0 1 2 3 
  Component 2: 4 5 6 
  Total Connected Components = 2
```

---

## 6. Comparison Tables and Complexity Summary

### 6.1 Comprehensive Comparison: Breadth-First Search (BFS) vs. Depth-First Search (DFS)

| Feature / Criterion | Breadth-First Search (BFS) | Depth-First Search (DFS) |
| :--- | :--- | :--- |
| **Underlying Data Structure** | **FIFO Queue** (`enqueue` / `dequeue`) | **LIFO Stack** (OS recursion call stack or explicit stack) |
| **Exploration Pattern** | **Level-by-level** concentric ripples (broad and shallow first) | **Path-by-path** plunge and backtrack (deep and narrow first) |
| **Tree Traversal Equivalent** | **Level-Order Traversal** | **Preorder / Postorder Traversal** |
| **When `visited` Is Marked** | **When enqueued** (mandatory to prevent duplicate queue entries) | **Upon entry** (recursive) or **when popped** (explicit stack) |
| **Unweighted Shortest Path?** | **Yes — Guaranteed** to find minimum-edge paths from source $s$ | **No** — May find a long, winding path before a 1-edge shortcut |
| **Space on Wide, Shallow Graph** (e.g., Star Graph $K_{1, n-1}$) | **High ($\Theta(V)$):** Queue holds all $n-1$ leaves simultaneously | **Low ($\Theta(1)$):** Call stack depth is at most $2$ |
| **Space on Deep, Narrow Graph** (e.g., Line Graph $P_n$) | **Low ($\Theta(1)$):** Queue holds at most $1$ or $2$ vertices at a time | **High ($\Theta(V)$):** Recursion stack reaches depth $n$ |
| **Primary Algorithmic Applications** | 1. Unweighted Shortest Path<br>2. Bipartite Graph Testing (2-Coloring)<br>3. Connected Components / Web Crawlers<br>4. Cheney's Copying Garbage Collector | 1. Cycle Detection (via Back Edges)<br>2. Topological Sorting of DAGs<br>3. Strongly Connected Components (Tarjan / Kosaraju)<br>4. Articulation Points, Bridges, Maze Solving |

### 6.2 Asymptotic Complexity Summary Table

Let $V = |V|$ be the number of vertices and $E = |E|$ be the number of edges.

| Traversal Algorithm | Time (Adjacency List) | Time (Adjacency Matrix) | Auxiliary Space (Worst Case) |
| :--- | :--- | :--- | :--- |
| **Breadth-First Search (BFS)** | $\Theta(V + E)$ | $\Theta(V^2)$ | $\Theta(V)$ (FIFO Queue + `visited[]`) |
| **Recursive Depth-First Search (DFS)** | $\Theta(V + E)$ | $\Theta(V^2)$ | $\Theta(V)$ (OS Call Stack + `visited[]`) |
| **Iterative Depth-First Search (DFS)** | $\Theta(V + E)$ | $\Theta(V^2)$ | $\Theta(V + E)$ or $\Theta(V)$ (Explicit Stack + `visited[]`) |

---

## 7. Common Errors, Pitfalls, and Edge Cases

### 7.1 Pitfall 1: Marking `visited` on Dequeue Instead of Enqueue in BFS

If `visited[v] = true` is delayed until `v` is dequeued, multiple vertices on the previous wave that share an edge to `v` will all see `visited[v] == false` and push duplicate copies of `v` into the queue.

```c
/* BUG: Marking visited when dequeued allows up to O(E) duplicate enqueues! */
while (front < rear) {
    int u = queue[front++];
    visited[u] = true; /* TOO LATE! u may already be in queue multiple times */
    for (AdjNode *c = g->head[u]; c != NULL; c = c->next) {
        if (!visited[c->dest]) {
            queue[rear++] = c->dest; /* Buffer overflow if queue size is MAX_V! */
        }
    }
}

/* FIX: Mark visited BEFORE / AT the moment of enqueue */
visited[start] = true;
queue[rear++] = start;
while (front < rear) {
    int u = queue[front++];
    for (AdjNode *c = g->head[u]; c != NULL; c = c->next) {
        if (!visited[c->dest]) {
            visited[c->dest] = true; /* Marked immediately! Enqueued at most once */
            queue[rear++] = c->dest;
        }
    }
}
```

### 7.2 Pitfall 2: Calling `bfs(g, 0)` or `dfs(g, 0)` Only Once on a Disconnected Graph

A single call from vertex `0` only explores the connected component containing `0`. Always wrap full-graph tasks (like cycle detection, topological sort, or component counting) in a `for (int u = 0; u < g->V; u++) if (!visited[u])` loop.

### 7.3 Pitfall 3: False Cycle Detection in Undirected DFS (Not Ignoring the Immediate Parent)

In an undirected graph, every edge $\{u, v\}$ appears in both `head[u]` and `head[v]`. When DFS moves from `u` to `v`, `u` is already `visited`. If `v` inspects `u` and immediately reports a back edge without checking `u != parent`, every single undirected edge will falsely trigger cycle detection!

```c
/* BUG: Treats the undirected tree edge (v -> parent) as a cycle! */
bool hasCycleUndirectedBug(const DualGraph *g, int u, bool visited[]) {
    visited[u] = true;
    for (AdjNode *c = g->head[u]; c != NULL; c = c->next) {
        if (!visited[c->dest]) {
            if (hasCycleUndirectedBug(g, c->dest, visited)) return true;
        } else {
            return true; /* FALSE POSITIVE on c->dest == parent! */
        }
    }
    return false;
}

/* FIX: Pass parent parameter and only flag a back edge when c->dest != parent */
bool hasCycleUndirectedFix(const DualGraph *g, int u, int parent, bool visited[]) {
    visited[u] = true;
    for (AdjNode *c = g->head[u]; c != NULL; c = c->next) {
        if (!visited[c->dest]) {
            if (hasCycleUndirectedFix(g, c->dest, u, visited)) return true;
        } else if (c->dest != parent) {
            return true; /* Genuine back edge to a non-parent ancestor! */
        }
    }
    return false;
}
```

### 7.4 Pitfall 4: Call-Stack Overflow in Recursive DFS on Deep Graphs ($|V| \ge 10^5$)

On a skewed graph (e.g., a long path of $10^5$ vertices), recursive DFS makes $10^5$ nested function calls, consuming several megabytes of stack frames and crashing with `SIGSEGV` (Stack Overflow) when the OS thread stack limit (typically $1\text{–}8\text{ MB}$) is exceeded. For large production graphs, use an **iterative explicit stack allocated on the heap** (`malloc`).

### 7.5 Boundary Conditions Checklist

- [ ] **Single-Vertex Graph ($|V| = 1, |E| = 0$):** Does BFS/DFS visit vertex `0` once and terminate cleanly?
- [ ] **Disconnected / Isolated Vertices:** Does the full traversal loop visit isolated vertices with $\deg(u) = 0$?
- [ ] **Self-Loops ($u \to u$) and Cycles ($u \leftrightarrow v$):** Does `if (!visited[v])` prevent infinite recursion on self-loops and 2-cycles?
- [ ] **Unreachable Target in Shortest Path:** Does `printShortestPath` check `dist[target] != INF_DIST` before attempting to trace `parent[target]`?

---

## 8. Summary and Revision Checklist

### 8.1 Key Takeaways and Core Invariants

1. **BFS = Queue + Level Order:** Explores vertices in non-decreasing order of unweighted distance $d[v]$ from source $s$. Always mark `visited[v] = true` **upon enqueue**.
2. **DFS = Stack + Backtracking:** Explores deeply along each branch before backtracking. Tracks discovery/finish timestamps $1 \le d[u] < f[u] \le 2|V|$ satisfying the **Parenthesis Theorem**.
3. **Iterative vs. Recursive DFS:** Pushing neighbors onto an explicit LIFO stack in **reverse order** makes Iterative DFS visit vertices in the exact same order as Recursive DFS.
4. **Time Complexity:** Both BFS and DFS run in **$\Theta(|V| + |E|)$** time on an Adjacency List and **$\Theta(|V|^2)$** time on an Adjacency Matrix.

### 8.2 Self-Test Revision Questions

1. **Trace Question:** In graph $G_1$, if we start BFS at vertex `3` instead of `0`, what is the BFS traversal order and what is `dist[0]`?
   *(Answer: Wave 0: `3`; Wave 1: `1, 5`; Wave 2: `0, 2, 4`. BFS Order: `3, 1, 5, 0, 2, 4`, and `dist[0] = 2` via path $3 \to 1 \to 0$).*
2. **Timestamp Question:** During a DFS of a directed graph, suppose `d[u] = 3, f[u] = 10` and `d[v] = 5, f[v] = 8`. If edge $(v, u)$ exists, what type of DFS edge is $(v, u)$?
   *(Answer: Since $d[u] < d[v] < f[v] < f[u]$, $u$ is an active ancestor of $v$ when $v$ inspects $(v, u)$. Thus $(v, u)$ is a **Back Edge**, proving the digraph contains a cycle! Conversely, $(u, v)$ would be a Tree or Forward edge).*
3. **Complexity Question:** Why is the time complexity of BFS on an Adjacency List written as $O(|V| + |E|)$ instead of simply $O(|E|)$?
   *(Answer: Because a disconnected graph can have $|V| = 10^6$ isolated vertices and $|E| = 0$ edges. Initializing `visited[]` and running `bfsFull` still requires $\Theta(|V|)$ time even when $|E| = 0$. Including both terms $O(|V| + |E|)$ is tight across both sparse/disconnected and dense regimes).*

### 8.3 Navigation Links

- **Previous Topic:** [07-Graph-Fundamentals-and-Representations.md](./07-Graph-Fundamentals-and-Representations.md) — Graph Fundamentals, Handshaking Lemma, Adjacency Matrix, and Adjacency List
- **Current File:** `08-DFS-and-BFS.md` (Unit 3 — Topic 8)
- **Next File (Unit 3 Exam Review):** [09-Unit-3-Review.md](./09-Unit-3-Review.md) — Unit 3 Comprehensive Review & Exam Preparation
