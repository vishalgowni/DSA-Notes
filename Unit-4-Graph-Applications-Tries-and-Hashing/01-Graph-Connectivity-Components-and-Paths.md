# Unit 4 — Topic 1: Graph Connectivity, Connected Components, and Path Finding

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [From Raw Graph Traversals to Structural Graph Queries](#11-from-raw-graph-traversals-to-structural-graph-queries)
   - 1.2 [Why Do We Need Connectivity, Components, and Path Finding?](#12-why-do-we-need-connectivity-components-and-path-finding)
   - 1.3 [Relationship to Prior Concepts: BFS and DFS as Building Blocks](#13-relationship-to-prior-concepts-bfs-and-dfs-as-building-blocks)
   - 1.4 [Real-World Engineering and Computer Science Applications](#14-real-world-engineering-and-computer-science-applications)
   - 1.5 [Learning Objectives](#15-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Walks, Trails, Simple Paths, Reachability, and Distance](#21-walks-trails-simple-paths-reachability-and-distance)
   - 2.2 [Undirected Connectivity and Connected Components](#22-undirected-connectivity-and-connected-components)
   - 2.3 [Directed Graph Connectivity: Weak vs. Strong Connectivity and Transpose Graph $G^T$](#23-directed-graph-connectivity-weak-vs-strong-connectivity-and-transpose-graph-gt)
   - 2.4 [Core Terminology Table](#24-core-terminology-table)
   - 2.5 [Mathematical Properties, Invariants, and Formulas](#25-mathematical-properties-invariants-and-formulas)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Adjacency List Graph and Traversal State Arrays (`visited`, `component_id`, `parent`, `dist`)](#31-adjacency-list-graph-and-traversal-state-arrays-visited-component_id-parent-dist)
   - 3.2 [How the `parent[]` Predecessor Array Encodes an Inverted Tree](#32-how-the-parent-predecessor-array-encodes-an-inverted-tree)
   - 3.3 [ASCII Memory Layout Diagram](#33-ascii-memory-layout-diagram)
   - 3.4 [Static vs. Dynamic Storage Trade-Offs for Auxiliary Arrays](#34-static-vs-dynamic-storage-trade-offs-for-auxiliary-arrays)
4. [Operations and Algorithms](#4-operations-and-algorithms)
   - 4.1 [Testing Graph Connectivity (Undirected and Directed Strong Connectivity)](#41-testing-graph-connectivity-undirected-and-directed-strong-connectivity)
   - 4.2 [Counting and Labelling Connected Components](#42-counting-and-labelling-connected-components)
   - 4.3 [Finding Paths in a Network ($s$ to $t$) and Predecessor Reconstruction](#43-finding-paths-in-a-network-s-to-t-and-predecessor-reconstruction)
   - 4.4 [DFS-Based Reachability & Path Finding (And Why DFS Fails to Guarantee Shortest Paths)](#44-dfs-based-reachability--path-finding-and-why-dfs-fails-to-guarantee-shortest-paths)
   - 4.5 [BFS-Based Shortest Path in Unweighted Graphs (With Mathematical Proof)](#45-bfs-based-shortest-path-in-unweighted-graphs-with-mathematical-proof)
   - 4.6 [Complete C11 Implementation: Connectivity, Components, DFS Path, and BFS Shortest Path](#46-complete-c11-implementation-connectivity-components-dfs-path-and-bfs-shortest-path)
5. [Comparison Tables and Complexity Summary](#5-comparison-tables-and-complexity-summary)
   - 5.1 [Time and Space Complexity Table (Adjacency List vs. Adjacency Matrix)](#51-time-and-space-complexity-table-adjacency-list-vs-adjacency-matrix)
   - 5.2 [Algorithmic Comparison: DFS Path Finding vs. BFS Shortest Path vs. Weighted Shortest Paths](#52-algorithmic-comparison-bfs-shortest-path-vs-dfs-path-finding-vs-weighted-shortest-paths)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Pitfall 1: Assuming DFS Finds the Shortest Path in an Unweighted Graph](#61-pitfall-1-assuming-dfs-finds-the-shortest-path-in-an-unweighted-graph)
   - 6.2 [Pitfall 2: Marking Vertices `visited` Upon Dequeue Instead of Enqueue in BFS](#62-pitfall-2-marking-vertices-visited-upon-dequeue-instead-of-enqueue-in-bfs)
   - 6.3 [Pitfall 3: Printing the `parent[]` Chain Without Reversing It](#63-pitfall-3-printing-the-parent-chain-without-reversing-it)
   - 6.4 [Pitfall 4: Testing Only Single-Pass Reachability from Vertex `0` on a Directed Graph](#64-pitfall-4-testing-only-single-pass-reachability-from-vertex-0-on-a-directed-graph)
   - 6.5 [Boundary Conditions Checklist](#65-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)
   - 7.1 [Key Takeaways and Core Invariants](#71-key-takeaways-and-core-invariants)
   - 7.2 [Self-Test Revision Questions](#72-self-test-revision-questions)
   - 7.3 [Navigation Links](#73-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 From Raw Graph Traversals to Structural Graph Queries

In Unit 3, we studied how to represent graphs in memory using **Adjacency Matrices** and **Adjacency Lists**, and how to systematically visit vertices using **Breadth-First Search (BFS)** and **Depth-First Search (DFS)**. While a raw traversal simply outputs the sequence in which vertices are visited, real-world applications rarely run BFS or DFS just to print vertex numbers. Instead, BFS and DFS serve as **algorithmic engines** to answer fundamental structural and routing questions about a network $G = (V, E)$:

1. **Global Connectivity Query:** Can every node in the network communicate with every other node, or has a link failure fractured the network into isolated islands?
2. **Component Decomposition Query:** If the graph is disconnected, how many isolated sub-networks (**connected components**) exist, and which vertices belong to which component?
3. **Reachability and Path Reconstruction Query:** Given a source vertex $s$ and a destination vertex $t$, does a path from $s$ to $t$ exist? If so, what is the exact sequence of intermediate vertices along that path?
4. **Minimum-Hop Shortest Path Query:** Among all possible paths between $s$ and $t$ in an unweighted network, which path uses the **minimum number of edges (hops)**?

```
   RAW TRAVERSAL (Unit 3)                  STRUCTURAL GRAPH APPLICATIONS (Unit 4)
  +----------------------+                +-----------------------------------------------+
  |  BFS(G, s) / DFS(G,s)|  +visited_cnt  | 1. Connectivity Test (visited_count == V ?)   |
  |                      | -------------> | 2. Connected Components (component_id[v] = c) |
  |  Visits reachable    |  +parent[v]    | 3. Path Reconstruction (backtrack parent[])   |
  |  vertices in O(V+E)  |  +dist[v]      | 4. Unweighted Shortest Path (BFS level order) |
  +----------------------+                +-----------------------------------------------+
```

### 1.2 Why Do We Need Connectivity, Components, and Path Finding?

Without efficient $O(V + E)$ algorithms for connectivity and path finding, fundamental operations in computer networks, operating systems, compilers, and image processing would become computationally intractable:

- **Network Partition Detection:** When a fiber-optic cable is severed or a router crashes, a cloud data center must determine within milliseconds whether the topology remains connected or has split into disconnected partitions (the "split-brain" problem in distributed consensus).
- **Cluster Identification:** In a graph with millions of nodes, an outer loop over vertices combined with BFS/DFS partitions the entire vertex set into disjoint **connected components** in linear $O(V + E)$ time—assigning an integer label `component_id[v]` so that any subsequent same-component query `component_id[u] == component_id[v]` answers reachability in **$O(1)$ time**!
- **Optimal Unweighted Routing:** Exhaustively enumerating all simple paths between $s$ and $t$ to find the shortest one takes $O(V!)$ time in the worst case (since a complete graph has exponentially many simple paths). By augmenting BFS with a predecessor array `parent[]` and a hop-distance array `dist[]`, we find the guaranteed minimum-hop path in linear **$O(V + E)$ time**.

### 1.3 Relationship to Prior Concepts: BFS and DFS as Building Blocks

| Graph Application Problem | Underlying Traversal | Auxiliary State Maintained | Time Complexity (Adj. List) |
| :--- | :--- | :--- | :--- |
| **Undirected Connectivity Test** | Single BFS or DFS from vertex `0` | `visited[V]`, `visited_count` | $O(V + E)$ |
| **Directed Strong Connectivity Test** | Two-Pass DFS/BFS on $G$ and $G^T$ | `visited[V]`, Transpose Graph $G^T$ | $O(V + E)$ |
| **Connected Component Labelling** | Outer loop $0 \dots V-1$ + BFS/DFS | `component_id[V]`, `comp_count` | $O(V + E)$ |
| **Any Valid Path ($s \leadsto t$)** | Single DFS or BFS from $s$ | `visited[V]`, `parent[V]` | $O(V + E)$ |
| **Shortest Unweighted Path ($s \leadsto t$)** | **Strictly BFS** from $s$ | `visited[V]`, `parent[V]`, `dist[V]` | $O(V + E)$ |

### 1.4 Real-World Engineering and Computer Science Applications

1. **Computer Networking (RIP & Layer-2/Layer-3 Routing):** The Routing Information Protocol (RIP) uses hop count as its metric; finding a route with the fewest router hops is directly an unweighted shortest-path problem solved by BFS.
2. **Computer Vision & Medical Image Segmentation:** In binary images, foreground pixels adjacent horizontally or vertically form an implicit grid graph. **Connected-Component Labelling (CCL)** groups contiguous pixels into distinct objects (e.g., counting cells on a microscope slide or isolating characters in Optical Character Recognition).
3. **Compiler Dead-Code Elimination & Garbage Collection:** A program's control-flow graph (CFG) or heap reference graph is a directed graph. Reachability from the entry block (`main`) or root set identifies live code/objects; unreachable components are dead code or garbage memory to be reclaimed.
4. **Social Network Analysis ("Degrees of Separation"):** Computingwhether two users are connected within 3 hops ("3rd-degree connection" on LinkedIn) and displaying the shortest chain of mutual acquaintances uses BFS with a `parent[]` array.

### 1.5 Learning Objectives

By the end of this topic, you will be able to:
- Test whether an undirected graph is connected using a single $O(V + E)$ traversal and distinguish **Weak Connectivity** from **Strong Connectivity** in directed graphs using $G$ and its transpose $G^T$.
- Count and label all connected components of a disconnected graph using an outer vertex loop and prove why its total time complexity remains $O(V + E)$.
- Maintain `parent[]` and `dist[]` arrays during graph traversal and reconstruct the forward path $s \to \dots \to t$ by backtracking from $t$ to $s$.
- Demonstrate with a concrete counterexample why **DFS does NOT guarantee the shortest path**, and prove mathematically why **BFS guarantees the shortest path** in unweighted graphs.
- Implement all four algorithms in clean, memory-safe C11 code.

---

## 2. Definitions and Terminology

### 2.1 Walks, Trails, Simple Paths, Reachability, and Distance

Let $G = (V, E)$ be an undirected or directed graph with $|V| = V$ vertices and $|E| = E$ edges.

1. **Walk:** A finite alternating sequence of vertices and edges $W = \langle v_0, e_1, v_1, e_2, \dots, e_k, v_k \rangle$ such that each edge $e_i = (v_{i-1}, v_i) \in E$ for $1 \le i \le k$. The **length** of a walk is the number of edges $k$ it contains (a single vertex $\langle v_0 \rangle$ is a trivial walk of length $0$).Vertices and edges may be repeated in a walk.
2. **Trail:** A walk in which **all edges are distinct** (no edge is traversed more than once), though vertices may be repeated.
3. **Simple Path (or simply Path):** A walk $P = \langle v_0, v_1, \dots, v_k \rangle$ in which **all vertices are distinct** ($v_i \neq v_j$ for all $0 \le i < j \le k$), which automatically implies all edges are distinct.
4. **Reachability ($u \leadsto v$):** A vertex $v$ is **reachable** from a vertex $u$ (denoted $u \leadsto v$) if and only if there exists a path from $u$ to $v$ in $G$. Every vertex is trivially reachable from itself ($u \leadsto u$) via a path of length $0$.
5. **Unweighted Shortest-Path Distance $d(u, v)$:** The **geodesic distance** (or **hop distance**) $d(u, v)$ from $u$ to $v$ is the minimum length (number of edges) among all paths from $u$ to $v$. If $v$ is not reachable from $u$, we write $d(u, v) = \infty$ (represented in C as `-1` or `INT_MAX`).

### 2.2 Undirected Connectivity and Connected Components

- **Connected Undirected Graph:** An undirected graph $G = (V, E)$ is **connected** if and only if for every pair of vertices $u, v \in V$, $v$ is reachable from $u$ ($u \leadsto v$).
- **Connected Component:** A **connected component** of an undirected graph $G$ is a **maximal connected subgraph** $C = (V_C, E_C)$ of $G$. Here, *maximal* means that no vertex $w \in V \setminus V_C$ can be added to $V_C$ while keeping the subgraph connected (i.e., there are zero edges between $V_C$ and $V \setminus V_C$).

```
      CONNECTED UNDIRECTED GRAPH                   DISCONNECTED UNDIRECTED GRAPH (k = 3 Components)
          (1 Component: k = 1)
                                                  Component C1          Component C2       Component C3
             (0)---(1)                              (0)---(1)            (3)---(4)             (7)
              | \   |                                 \   /               |     |          (Isolated
              |  \  |                                  (2)               (6)---(5)           Vertex)
             (2)---(3)
     Every pair (u, v) has a path.               No edge crosses between C1, C2, and C3.
```

### 2.3 Directed Graph Connectivity: Weak vs. Strong Connectivity and Transpose Graph $G^T$

In a directed graph (digraph), edge $(u, v)$ permits traversal from $u$ to $v$ but **not** from $v$ to $u$. Therefore, $u \leadsto v$ does not imply $v \leadsto u$, giving rise to two distinct notions of connectivity:

1. **Strongly Connected Digraph:** A directed graph $G = (V, E)$ is **strongly connected** if and only if for **every ordered pair** of vertices $(u, v) \in V \times V$, both $u \leadsto v$ **and** $v \leadsto u$ hold.
2. **Weakly Connected Digraph:** A directed graph $G = (V, E)$ is **weakly connected** if and only if its **underlying undirected graph** $G_{undir}$ (obtained by replacing every directed edge $(u, v)$ with an undirected edge $\{u, v\}$) is connected. Every strongly connected graph is weakly connected, but the converse is false.
3. **Strongly Connected Component (SCC):** A maximal subgraph $S \subseteq G$ of a directed graph such that every pair of vertices within $S$ is mutually reachable ($u \leadsto v$ and $v \leadsto u$).
4. **Transpose (Reverse) Graph $G^T$:** Given a directed graph $G = (V, E)$, its **transpose graph** is $G^T = (V, E^T)$, where every edge's direction is reversed:
   $$E^T = \{ (v, u) \mid (u, v) \in E \}$$
   Crucially, a path $u \leadsto 0$ exists in $G$ **if and only if** a path $0 \leadsto u$ exists in $G^T$!

```
   STRONGLY CONNECTED DIGRAPH        WEAKLY CONNECTED (NOT STRONG)       TRANSPOSE GRAPH G^T OF DIGRAPH
                                                                         (All edge arrows reversed)
        (0) ------> (1)               (0) -> (1) -> (2) -> (3)
         ^           |                                                        (0) <------ (1)
         |           v                0 reaches 1, 2, 3,                       |           ^
        (3) <------ (2)               but 3 cannot reach 0, 1, or 2!           v           |
   Directed cycle: every vertex       Underlying undirected graph             (3) ------> (2)
   can reach every other vertex.      is a single connected chain.
```

### 2.4 Core Terminology Table

| Term | Formal Definition | Algorithmic Significance |
| :--- | :--- | :--- |
| **Simple Path** | Sequence of distinct vertices $\langle v_0, v_1, \dots, v_k \rangle$ with $(v_{i-1}, v_i) \in E$ | Contains no cycles; maximum possible length is $V - 1$ edges |
| **Hop Distance $d(s, t)$** | Minimum number of edges in any path from $s$ to $t$ | Computed exactly by BFS in $O(V + E)$ time on unweighted graphs |
| **Connected Component** | Equivalence class of vertices under undirected reachability | Partitioned and labeled $1 \dots k$ by an outer loop + BFS/DFS |
| **Predecessor (`parent[v]`)** | Vertex $u$ that first discovered vertex $v$ during traversal | Forms an inverted tree rooted at source $s$ (`parent[s] = -1`) |
| **Breadth-First Tree** | Subgraph $T_{BFS} = (V_R, \{(\text{parent}[v], v) : v \in V_R \setminus \{s\}\})$ | Every tree path from $s$ to $v$ in $T_{BFS}$ is a **shortest path** in $G$ |
| **Transpose Graph $G^T$** | Directed graph with all edges reversed: $(v, u) \in E^T \iff (u, v) \in E$ | Allows testing whether all vertices can reach $s$ via a single traversal from $s$ in $G^T$ |

### 2.5 Mathematical Properties, Invariants, and Formulas

1. **Equivalence Relation of Undirected Reachability:**
   On any undirected graph $G = (V, E)$, define the binary relation $\sim$ on $V$ by $u \sim v \iff u \leadsto v$. Then $\sim$ is an **equivalence relation**:
   - **Reflexive:** $u \sim u$ (via the trivial $0$-edge path $\langle u \rangle$).
   - **Symmetric:** $u \sim v \implies v \sim u$ (since every undirected edge can be traversed in reverse: $\langle v_k, v_{k-1}, \dots, v_0 \rangle$).
   - **Transitive:** $(u \sim v \land v \sim w) \implies u \sim w$ (concatenating the walk $u \leadsto v$ and $v \leadsto w$).
   Because $\sim$ is an equivalence relation, it uniquely **partitions** the vertex set $V$ into $k$ pairwise disjoint equivalence classes—the **connected components** $V = V_1 \sqcup V_2 \sqcup \dots \sqcup V_k$:
   $$V_i \cap V_j = \emptyset \quad (\forall i \neq j), \qquad \sum_{i=1}^{k} |V_i| = V$$

2. **Edge Count Bounds for a Graph with $k$ Connected Components:**
   Every connected component $C_i$ with $|V_i|$ vertices requires at least $|V_i| - 1$ edges (a spanning tree) and can have at most $\binom{|V_i|}{2}$ simple edges. Summing across all $k$ components yields:
   $$V - k \le E \le \frac{(V - k)(V - k + 1)}{2}$$
   - **Corollary 1:** A connected undirected graph ($k = 1$) must satisfy $E \ge V - 1$. If $E < V - 1$, the graph is **guaranteed to be disconnected** (having at least $k \ge V - E$ connected components).
   - **Corollary 2:** Any simple undirected graph with $E > \frac{(V - 1)(V - 2)}{2}$ is **guaranteed to be connected** ($k = 1$).

3. **Shortest-Path Triangle Inequality and Bellman Optimality Equation (Unweighted):**
   For any source $s \in V$ and any edge $(u, v) \in E$:
   $$d(s, v) \le d(s, u) + 1$$
   Furthermore, for every reachable vertex $v \neq s$, the shortest-path distance satisfies:
   $$d(s, v) = \min_{(u, v) \in E} \bigl\{ d(s, u) + 1 \bigr\}$$

---

## 3. Representation and Memory Layout in C

### 3.1 Adjacency List Graph and Traversal State Arrays (`visited`, `component_id`, `parent`, `dist`)

To perform connectivity checks, component labelling, and path reconstruction in $O(V + E)$ time, we pair an **Adjacency List** representation of $G$ with vertex-indexed auxiliary arrays of length $V$:

```c
#include <stdbool.h>

/* Singly linked node in a vertex's adjacency list */
typedef struct AdjNode {
    int vertex;             /* Destination vertex ID in {0 .. V-1} */
    struct AdjNode *next;   /* Pointer to next neighbor in list    */
} AdjNode;

/* Graph wrapper structure */
typedef struct Graph {
    int num_vertices;       /* Total number of vertices V          */
    bool is_directed;       /* false = undirected, true = directed */
    AdjNode **head;         /* Array of V pointers to AdjNode      */
} Graph;
```

Alongside `Graph *g`, our algorithms allocate or accept the following $O(V)$ auxiliary arrays:

| Array Declaration | Element Type | Initial Sentinel Value | Semantic Invariant After Traversal |
| :--- | :--- | :--- | :--- |
| `bool visited[V]` | `bool` | `false` (`0`) | `visited[v] == true` iff $v$ is reachable from source $s$ |
| `int component_id[V]` | `int` | `0` (unassigned) | `component_id[v] == c` ($\in \{1 \dots k\}$) identifies $v$'s connected component |
| `int parent[V]` | `int` | `-1` (no parent) | `parent[v] == u` means edge $(u, v)$ discovered $v$; `parent[s] == -1` |
| `int dist[V]` | `int` | `-1` (unreachable) | `dist[v] == d(s, v)` minimum hops from $s$ to $v$; `dist[s] == 0` |

### 3.2 How the `parent[]` Predecessor Array Encodes an Inverted Tree

Why do we store `parent[v]` (a single integer per vertex) instead of storing a list of children per vertex?
- In a tree, a parent vertex $u$ may have **many** children (up to $V - 1$), requiring dynamic lists if pointers point downward from parent to child.
- However, in any spanning tree or BFS shortest-path tree rooted at $s$, **every vertex $v \neq s$ has at most ONE unique parent $u$**!
- Therefore, a flat integer array `int parent[V]` represents the entire Breadth-First Tree or Depth-First Tree in exactly $V \times \text{sizeof(int)}$ bytes with zero dynamic allocation per edge.

```
     BFS SHORTEST-PATH TREE ROOTED AT s = 0             INVERTED POINTER VIEW (parent[] Array)
     (Edges point downward during discovery)            (Each node points upward to its unique parent)

                    (0)  dist = 0                                      (0) ---> -1 (Root Sentinel)
                   /   \                                              ^   ^
                  v     v                                            /     \
       dist = 1 (1)     (2) dist = 1                               (1)     (2)
                / \                                                ^ ^
               v   v                                              /   \
    dist = 2 (3)   (4) dist = 2                                 (3)   (4)
              |                                                  ^
              v                                                  |
    dist = 3 (5)                                                (5)

     Forward Path 0 -> 1 -> 3 -> 5 is recovered by walking UPWARD: 5 -> parent[5]=3 -> parent[3]=1 -> parent[1]=0!
```

### 3.3 ASCII Memory Layout Diagram

Below is the exact memory layout of a 6-vertex network graph after running `findShortestPathBFS(g, 0, 5, parent, dist)`:

```
  GRAPH STRUCT (Heap)              ADJACENCY LISTS (Heap)
  +-------------------+            head[0] --> [1|*] --> [2|NULL]
  | num_vertices = 6  |            head[1] --> [0|*] --> [3|*] --> [4|NULL]
  | is_directed  = 0  |            head[2] --> [0|*] --> [4|NULL]
  | head         -----+--------->  head[3] --> [1|*] --> [5|NULL]
  +-------------------+            head[4] --> [1|*] --> [2|*] --> [5|NULL]
                                   head[5] --> [3|*] --> [4|NULL]

  AUXILIARY STATE ARRAYS (Contiguous O(V) Arrays in Memory)
  Index v:        [  0  |  1  |  2  |  3  |  4  |  5  ]
                  +-----+-----+-----+-----+-----+-----+
  visited[v]:     |  1  |  1  |  1  |  1  |  1  |  1  |   (1 byte each)
                  +-----+-----+-----+-----+-----+-----+
  dist[v]:        |  0  |  1  |  1  |  2  |  2  |  3  |   (4 bytes each: hop count from s=0)
                  +-----+-----+-----+-----+-----+-----+
  parent[v]:      | -1  |  0  |  0  |  1  |  1  |  3  |   (4 bytes each: predecessor vertex)
                  +-----+-----+-----+-----+-----+-----+
                     ^     |           |           |
                     +-----+-----------+-----------+
                     Backtracking chain from t=5: 5 -> parent[5]=3 -> parent[3]=1 -> parent[1]=0 -> -1
```

### 3.4 Static vs. Dynamic Storage Trade-Offs for Auxiliary Arrays

| Storage Strategy | Declaration Style | Advantages | Disadvantages |
| :--- | :--- | :--- | :--- |
| **Stack / Caller-Provided Fixed Array** | `int parent[MAX_V];` | Zero heap allocation overhead; automatic cleanup on return; cache-friendly | Fixed upper bound `MAX_V`; large `MAX_V` (e.g., $10^7$) risks stack overflow |
| **Heap-Allocated (`calloc`/`malloc`)** | `int *parent = malloc(V * sizeof(int));` | Scales to arbitrary runtime graph sizes $V$; `calloc` zero-initializes `visited[]` in $O(V)$ | Caller or function must explicitly `free()` to avoid memory leaks |

---

## 4. Operations and Algorithms

### 4.1 Testing Graph Connectivity (Undirected and Directed Strong Connectivity)

#### 1. Purpose and Preconditions
- **Undirected Connectivity:** Determine whether an undirected graph $G = (V, E)$ is a single connected piece ($k = 1$).
- **Directed Strong Connectivity:** Determine whether every vertex $u \in V$ can reach every vertex $v \in V$ in a directed graph $G = (V, E)$.
- **Preconditions:** `g` is a valid `Graph` pointer (`V >= 0`). By convention, an empty graph ($V = 0$) or a single-vertex graph ($V = 1$) is vacuously connected (`true`).

#### 2. Step-by-Step Algorithm
- **Case A — Undirected Graph (`isUndirectedConnected`):**
  1. If $V \le 1$, return `true`.
  2. Allocate `visited[0 .. V-1]` initialized to `false` and set `visited_count = 0`.
  3. Run a single DFS (or BFS) starting from vertex `0`. Each time an unvisited vertex $u$ is marked `visited[u] = true`, increment `visited_count`.
  4. Free `visited[]` and return `(visited_count == V)`.
  - *Why is a single traversal from vertex `0` sufficient?* Because undirected reachability is symmetric and transitive: if `0` can reach every vertex $v \in V$, then for any two vertices $x, y \in V$, $x$ can reach `0` (by symmetry of $0 \leadsto x$) and `0` can reach $y$, so $x \leadsto 0 \leadsto y$.

- **Case B — Directed Graph Strong Connectivity (`isDirectedStronglyConnected` — Kosaraju Two-Pass Idea):**
  1. If $V \le 1$, return `true`.
  2. **Pass 1 (Forward Reachability):** Run DFS/BFS from vertex `0` on $G$. Count `count_forward`. If `count_forward < V`, some vertex is unreachable from `0`, so return `false` immediately.
  3. **Transpose Construction:** Build the transpose graph $G^T$ in $O(V + E)$ time by iterating over every adjacency list of $G$ and inserting reversed edge $(v, u)$ into $G^T$ for each $(u, v) \in E$.
  4. **Pass 2 (Backward Reachability):** Reset `visited[]` to `false` and run DFS/BFS from vertex `0` on $G^T$, counting `count_transpose`.
  5. Free $G^T$ and `visited[]`. Return `(count_transpose == V)`.
  - *Why are two passes necessary and sufficient?* For any pair $(x, y) \in V \times V$: Pass 2 guarantees that `0` reaches $x$ in $G^T$, which means $x \leadsto 0$ in $G$. Pass 1 guarantees that $0 \leadsto y$ in $G$. Concatenating the two paths yields $x \leadsto 0 \leadsto y$ in $G$!

#### 3. Pseudocode

```text
ALGORITHM IsUndirectedConnected(G)
  Input:  Undirected graph G = (V, E)
  Output: true if G is connected, false otherwise

  if V <= 1 then return true
  visited[0 .. V-1] <- {false}
  visited_count <- 0

  DFS_Reach(G, 0, visited, visited_count)
  return (visited_count == V)

ALGORITHM IsDirectedStronglyConnected(G)
  Input:  Directed graph G = (V, E)
  Output: true if G is strongly connected, false otherwise

  if V <= 1 then return true
  visited[0 .. V-1] <- {false}
  count_forward <- 0
  DFS_Reach(G, 0, visited, count_forward)
  if count_forward < V then return false

  G_T <- ConstructTranspose(G)   // Reverse every directed edge (u -> v) to (v -> u)
  visited[0 .. V-1] <- {false}
  count_transpose <- 0
  DFS_Reach(G_T, 0, visited, count_transpose)
  return (count_transpose == V)
```

#### 4. ASCII Trace: Why Single-Pass Fails on a Directed Chain vs. Two-Pass Success

```
  DIRECTED CHAIN G:   (0) ---> (1) ---> (2) ---> (3)
  (Weakly connected, NOT strongly connected)

  - Pass 1 from vertex 0 in G:
    0 -> 1 -> 2 -> 3.  visited_count = 4 == V.  (A naive 1-pass check would falsely say "Connected"!)

  - Construct Transpose G^T:  (0) <--- (1) <--- (2) <--- (3)

  - Pass 2 from vertex 0 in G^T:
    Vertex 0 has NO outgoing edges in G^T!
    visited = {0}, count_transpose = 1 != 4  ===> Returns FALSE (Correctly detects 1,2,3 cannot reach 0!).
```

---

### 4.2 Counting and Labelling Connected Components

#### 1. Purpose and Preconditions
Partition an undirected graph $G = (V, E)$ into its $k$ maximal connected components, returning the total count $k$ and populating `component_id[0 .. V-1]` such that `component_id[u] == component_id[v]` if and only if $u$ and $v$ belong to the same connected component.

#### 2. Step-by-Step Algorithm
1. Initialize `component_id[v] = 0` for all $v \in \{0, \dots, V - 1\}$ (where `0` serves as both the "unvisited" flag and the "unlabelled" sentinel, avoiding the need for a separate `visited[]` array!).
2. Initialize `comp_count = 0`.
3. Loop `v` from `0` to `V - 1`:
   - If `component_id[v] == 0` (meaning vertex `v` was not reached by any previous component's traversal):
     - Increment `comp_count` (`comp_count++`).
     - Invoke `dfsLabelComponent(G, v, comp_count, component_id)` (or BFS), which sets `component_id[u] = comp_count` for `v` and every vertex `u` reachable from `v`.
4. Return `comp_count`.

#### 3. Pseudocode

```text
ALGORITHM FindConnectedComponents(G, component_id[0 .. V-1])
  Input:  Undirected graph G = (V, E)
  Output: Total number of connected components k, and populated component_id[] array

  for v <- 0 to V - 1 do
      component_id[v] <- 0          // 0 indicates unvisited

  comp_count <- 0
  for v <- 0 to V - 1 do
      if component_id[v] == 0 then
          comp_count <- comp_count + 1
          DFS_Label(G, v, comp_count, component_id)

  return comp_count

PROCEDURE DFS_Label(G, u, comp_id, component_id[])
  component_id[u] <- comp_id
  for each neighbor w in Adj[u] do
      if component_id[w] == 0 then
          DFS_Label(G, w, comp_id, component_id)
```

#### 4. Step-by-Step ASCII Diagram and Trace on an 8-Vertex Disconnected Graph

Consider the 8-vertex undirected graph $G$ below with $V = 8$ and $E = 7$:

```
      Component 1 (Triangle)          Component 2 (4-Cycle)           Component 3 (Isolated)
           (0)---(1)                       (3)---(4)                           (7)
            \     /                         |     |
              (2)                          (6)---(5)
```

Let us trace the outer loop `for (v = 0; v < 8; v++)` and the state of `component_id[0..7]`:

| Outer Loop `v` | Condition `component_id[v] == 0`? | Action Taken | `component_id[0 .. 7]` After Iteration |
| :---: | :---: | :--- | :--- |
| **Init** | — | Set all entries to `0`, `comp_count = 0` | `[ 0, 0, 0, 0, 0, 0, 0, 0 ]` |
| **`v = 0`** | **True** (`0 == 0`) | `comp_count = 1`; `DFS_Label(G, 0, 1)` visits `0 -> 1 -> 2` | `[ 1, 1, 1, 0, 0, 0, 0, 0 ]` |
| **`v = 1`** | False (`1 != 0`) | Skip (already labeled in Component 1) | `[ 1, 1, 1, 0, 0, 0, 0, 0 ]` |
| **`v = 2`** | False (`1 != 0`) | Skip (already labeled in Component 1) | `[ 1, 1, 1, 0, 0, 0, 0, 0 ]` |
| **`v = 3`** | **True** (`0 == 0`) | `comp_count = 2`; `DFS_Label(G, 3, 2)` visits `3 -> 4 -> 5 -> 6` | `[ 1, 1, 1, 2, 2, 2, 2, 0 ]` |
| **`v = 4`** | False (`2 != 0`) | Skip (already labeled in Component 2) | `[ 1, 1, 1, 2, 2, 2, 2, 0 ]` |
| **`v = 5`** | False (`2 != 0`) | Skip (already labeled in Component 2) | `[ 1, 1, 1, 2, 2, 2, 2, 0 ]` |
| **`v = 6`** | False (`2 != 0`) | Skip (already labeled in Component 2) | `[ 1, 1, 1, 2, 2, 2, 2, 0 ]` |
| **`v = 7`** | **True** (`0 == 0`) | `comp_count = 3`; `DFS_Label(G, 7, 3)` visits `7` (no neighbors) | `[ 1, 1, 1, 2, 2, 2, 2, 3 ]` |

- **Aggregate Complexity Analysis (Why is it $O(V + E)$ and NOT $O(V(V + E))$?):**
  A common student misconception is to multiply $V$ iterations of the outer `for` loop by $O(V + E)$ per DFS call to get $O(V(V + E))$. That bound is overly pessimistic because `DFS_Label(G, u, ...)` is **only invoked when `component_id[u] == 0`**, and its first line immediately sets `component_id[u] = comp_id` ($\neq 0$). Consequently:
  - Every vertex $u \in V$ enters `DFS_Label` **exactly once** across the entire algorithm: $\sum_{u \in V} 1 = V$.
  - Each vertex $u$'s adjacency list `Adj[u]` is scanned **at most once**: $\sum_{u \in V} \text{deg}(u) = 2E$.
  - The outer loop performs $V$ `if` checks.
  - Total Time Complexity: $O(V) + O(V + 2E) = \mathbf{O(V + E)}$.

---

### 4.3 Finding Paths in a Network ($s$ to $t$) and Predecessor Reconstruction

#### 1. Purpose and Preconditions
Given a populated predecessor array `parent[0 .. V-1]` generated by a traversal starting at source $s$ (with `parent[s] = -1` and `parent[v] = -1` for unreachable vertices), extract and print the forward sequence of vertices $\langle s = v_0, v_1, \dots, v_k = t \rangle$ from $s$ to $t$.

#### 2. Step-by-Step Algorithm (`reconstructPath`)
1. **Unreachability Check:** If $s \neq t$ and `parent[t] == -1`, then $t$ was never discovered during the traversal from $s$. Report that no path exists from $s$ to $t$ and return.
2. **Backward Trace ($t \leadsto s$):** Initialize an empty array `path[]` and `len = 0`. Start at `curr = t`. While `curr != -1`:
   - Append `curr` to `path[len++]`.
   - If `curr == s`, break (we have reached the source).
   - Advance `curr = parent[curr]`.
3. **Source Verification:** Verify that `path[len - 1] == s` (guards against malformed `parent[]` arrays).
4. **In-Place Reversal:** Since `path[]` currently holds $\langle t, \text{parent}[t], \dots, s \rangle$ in reverse order, swap `path[i]` with `path[len - 1 - i]` for $0 \le i < \lfloor \text{len}/2 \rfloor$.
5. Output `path[0 .. len-1]`, which now reads $\langle s, \dots, t \rangle$ with `len - 1` edges.

#### 3. ASCII Diagram of Backward Trace + Array Reversal

```
  Suppose s = 0, t = 5, and parent[] = [ -1, 0, 0, 1, 1, 3 ]

  STEP 1: Follow parent[] backward from curr = t = 5 until curr == s == 0:
          curr = 5  --> path[0] = 5,  curr = parent[5] = 3
          curr = 3  --> path[1] = 3,  curr = parent[3] = 1
          curr = 1  --> path[2] = 1,  curr = parent[1] = 0
          curr = 0  --> path[3] = 0,  curr == s (STOP!)

          path[] before reversal (Backwards):   [  5  |  3  |  1  |  0  ]  (len = 4)
                                                   ^     ^     ^     ^
                                                   |     +--+--+     |
                                                   +--------+--------+  (Swap symmetric pairs)

  STEP 2: Reverse path[] in-place:              [  0  |  1  |  3  |  5  ]
          Output Forward Path (len - 1 = 3 edges):  0 -> 1 -> 3 -> 5
```

---

### 4.4 DFS-Based Reachability & Path Finding (And Why DFS Fails to Guarantee Shortest Paths)

#### 1. Purpose and Behavior
A Depth-First Search starting at $s$ that records `parent[v] = u` whenever edge $(u, v)$ discovers unvisited $v$ (and halts early when $u == t$) is guaranteed to find **a valid simple path** from $s$ to $t$ if one exists, in $O(V + E)$ time.

However, **DFS does NOT guarantee the shortest path!** Why? Because DFS plunges as deeply as possible along the **first** neighbor in `Adj[s]` before it ever inspects the **second** neighbor of `s`. Even if the second neighbor of $s$ is $t$ itself (a 1-edge direct shortcut!), DFS will blindly follow a winding $(V - 1)$-edge path through the first neighbor if that branch happens to reach $t$.

#### 2. Concrete Counterexample: DFS Finds a 4-Edge Path When a 1-Edge Path Exists!

Consider the following 5-vertex undirected graph where we want a path from $s = 0$ to $t = 4$. Suppose edges are added in the order `(0,1), (1,2), (2,3), (3,4), (0,4)`, so `Adj[0] = [1, 4]`:

```
              COUNTEREXAMPLE GRAPH (s = 0, t = 4)

                  (1) ------- (2) ------- (3)
                  /                         \
                 / (Explored 1st by DFS!)    \
               (0) ------------------------- (4)
                    Direct 1-hop edge (0, 4)

  Adjacency Lists:
    Adj[0]: 1 -> 4
    Adj[1]: 0 -> 2
    Adj[2]: 1 -> 3
    Adj[3]: 2 -> 4
    Adj[4]: 3 -> 0
```

- **DFS Trace from $s = 0$ to $t = 4$:**
  1. Start at `u = 0` (`visited[0] = true`). First neighbor in `Adj[0]` is `1`.
  2. Set `parent[1] = 0`, recurse `dfs(1)` (`visited[1] = true`). Neighbor `0` is visited; next neighbor is `2`.
  3. Set `parent[2] = 1`, recurse `dfs(2)` (`visited[2] = true`). Neighbor `3` is unvisited.
  4. Set `parent[3] = 2`, recurse `dfs(3)` (`visited[3] = true`). Neighbor `4` is unvisited.
  5. Set `parent[4] = 3`, recurse `dfs(4)`. Now `u == t == 4` $\implies$ **Target found!**
  - **DFS Result:** Reconstructs path `0 -> 1 -> 2 -> 3 -> 4` (**4 edges**), completely ignoring the direct 1-edge link `0 -> 4`!
- **BFS Trace on the Exact Same Graph:**
  1. Dequeue `0`. Inspect **all** neighbors of `0` (`1` and `4`) before moving to distance-2 vertices!
  2. Mark `visited[1] = true, dist[1] = 1, parent[1] = 0`, enqueue `1`.
  3. Mark `visited[4] = true, dist[4] = 1, parent[4] = 0`, enqueue `4`.
  - **BFS Result:** Reconstructs path `0 -> 4` (**1 edge**, optimal shortest path!).

---

### 4.5 BFS-Based Shortest Path in Unweighted Graphs (With Mathematical Proof)

#### 1. Step-by-Step Algorithm (`findShortestPathBFS`)
1. Initialize `visited[0 .. V-1] = false`, `parent[0 .. V-1] = -1`, and `dist[0 .. V-1] = -1`.
2. Initialize an empty FIFO Queue $Q$.
3. **Seed Source $s$:** Set `visited[s] = true`, `dist[s] = 0`, `parent[s] = -1`, and `enqueue(Q, s)`.
4. **Level-by-Level Exploration Loop:** While $Q$ is not empty:
   - Dequeue the front vertex `u = dequeue(Q)`.
   - (Optional early exit: if `u == t`, we may stop if we only need the path to `t`, because `dist[t]` and `parent[t]` were already locked in when `t` was first enqueued).
   - For each neighbor `v` in `Adj[u]`:
     - If `!visited[v]`:
       - Set `visited[v] = true` **immediately** (before enqueueing!).
       - Set `dist[v] = dist[u] + 1`.
       - Set `parent[v] = u`.
       - Call `enqueue(Q, v)`.

#### 2. Pseudocode

```text
ALGORITHM FindShortestPathBFS(G, s, t, parent[0..V-1], dist[0..V-1])
  Input:  Graph G = (V, E), source vertex s, target vertex t
  Output: Populated dist[] and parent[] arrays; returns true if t is reachable

  for v <- 0 to V - 1 do
      visited[v] <- false
      parent[v]  <- -1
      dist[v]    <- -1

  Q <- CreateEmptyQueue(V)
  visited[s] <- true
  dist[s]    <- 0
  Enqueue(Q, s)

  while not IsEmpty(Q) do
      u <- Dequeue(Q)
      for each neighbor v in Adj[u] do
          if not visited[v] then
              visited[v] <- true
              dist[v]    <- dist[u] + 1
              parent[v]  <- u
              Enqueue(Q, v)

  return visited[t]
```

#### 3. Mathematical Proof: Why BFS Guarantees Shortest Paths in Unweighted Graphs

Why does discovering a vertex $v$ for the first time during BFS guarantee that $\text{dist}[v] = d(s, v)$ (the true minimum number of edges from $s$ to $v$)?

- **Queue Monotonicity Invariant:** At every point during the execution of BFS, if the FIFO queue $Q$ contains vertices $\langle v_1, v_2, \dots, v_r \rangle$ ordered from `front` ($v_1$) to `rear` ($v_r$), then:
  1. **Non-Decreasing Order:** $\text{dist}[v_1] \le \text{dist}[v_2] \le \dots \le \text{dist}[v_r]$.
  2. **Two-Layer Bound:** $\text{dist}[v_r] \le \text{dist}[v_1] + 1$ (all vertices in $Q$ belong to at most two consecutive distance levels $k$ and $k+1$).

- **Proof of Invariant by Induction on Queue Operations:**
  - *Base Case:* Initially $Q = \langle s \rangle$ with $\text{dist}[s] = 0$. Both properties hold trivially.
  - *Inductive Step:*
    - **Dequeue($v_1$):** Removing $v_1$ leaves $\langle v_2, \dots, v_r \rangle$. Since $\text{dist}[v_2] \ge \text{dist}[v_1]$, we have $\text{dist}[v_r] \le \text{dist}[v_1] + 1 \le \text{dist}[v_2] + 1$, preserving both properties.
    - **Enqueue($w$) when exploring $u = v_1$:** Any newly discovered neighbor $w$ of $v_1$ is assigned $\text{dist}[w] = \text{dist}[v_1] + 1$ and placed at the rear of $Q$. By the induction hypothesis, the previous rear element $v_r$ satisfied $\text{dist}[v_r] \le \text{dist}[v_1] + 1 = \text{dist}[w]$, so the queue remains sorted in non-decreasing order. Moreover, the current front of the queue has distance at least $\text{dist}[v_1]$, so $\text{dist}[w] = \text{dist}[v_1] + 1$ exceeds the front's distance by at most $1$.

- **Theorem (Shortest-Path Correctness):** For every vertex $v \in V$, BFS sets $\text{dist}[v] = d(s, v)$, and the path formed by following `parent[]` from $v$ to $s$ is a shortest path of length $d(s, v)$.
  - *Proof by Induction on $k = d(s, v)$:*
    - For $k = 0$, $v = s$ is initialized with $\text{dist}[s] = 0 = d(s, s)$.
    - Assume every vertex at true distance $\le k$ is discovered in non-decreasing distance order and assigned $\text{dist}[u] = d(s, u)$. Let $v$ be any vertex with true shortest-path distance $d(s, v) = k + 1$, and let $\langle s, \dots, u, v \rangle$ be a shortest path to $v$, where $d(s, u) = k$ and $(u, v) \in E$.
    - Because the queue is processed in strictly non-decreasing order of distance, $v$ cannot be discovered by any vertex at distance $< k$ (otherwise $d(s, v) \le k$, a contradiction). By the time $u$ (at distance $k$) is dequeued and scanned, either $v$ was already discovered by an earlier vertex $u'$ also at distance $k$ (giving $\text{dist}[v] = k + 1$), or $u$ discovers $v$ now and sets $\text{dist}[v] = \text{dist}[u] + 1 = k + 1 = d(s, v)$ and $\text{parent}[v] = u$. $\blacksquare$

#### 4. Step-by-Step Level-by-Level BFS Trace on a 7-Node Network

Consider our 7-vertex network graph ($s = 0$, $t = 5$, and isolated node $6$):

```
         Level 0          Level 1          Level 2          Level 3        Unreachable
                       +--- (1) ------------ (3) ------------ (5)             (6)
                      /      |                                 |           (Isolated)
                    (0)      |                                 |
                      \      |                                 |
                       +--- (2) ------------ (4) --------------+
```

| Step | Action | Queue $Q$ (`front .. rear`) | `visited[0..6]` | `dist[0..6]` | `parent[0..6]` |
| :---: | :--- | :--- | :--- | :--- | :--- |
| **0** | Init & Enqueue `0` | `[0]` | `[1,0,0,0,0,0,0]` | `[0,-1,-1,-1,-1,-1,-1]` | `[-1,-1,-1,-1,-1,-1,-1]` |
| **1** | Dequeue `0`; Enqueue `1, 2` | `[1, 2]` | `[1,1,1,0,0,0,0]` | `[0, 1, 1,-1,-1,-1,-1]` | `[-1, 0, 0,-1,-1,-1,-1]` |
| **2** | Dequeue `1`; Enqueue `3, 4` | `[2, 3, 4]` | `[1,1,1,1,1,0,0]` | `[0, 1, 1, 2, 2,-1,-1]` | `[-1, 0, 0, 1, 1,-1,-1]` |
| **3** | Dequeue `2` (`0, 4` already visited) | `[3, 4]` | `[1,1,1,1,1,0,0]` | `[0, 1, 1, 2, 2,-1,-1]` | `[-1, 0, 0, 1, 1,-1,-1]` |
| **4** | Dequeue `3`; Enqueue `5` | `[4, 5]` | `[1,1,1,1,1,1,0]` | `[0, 1, 1, 2, 2, 3,-1]` | `[-1, 0, 0, 1, 1, 3,-1]` |
| **5** | Dequeue `4` (`1, 2, 5` visited) | `[5]` | `[1,1,1,1,1,1,0]` | `[0, 1, 1, 2, 2, 3,-1]` | `[-1, 0, 0, 1, 1, 3,-1]` |
| **6** | Dequeue `5` (`3, 4` visited) | `[]` (Empty) | `[1,1,1,1,1,1,0]` | `[0, 1, 1, 2, 2, 3,-1]` | `[-1, 0, 0, 1, 1, 3,-1]` |

---

### 4.6 Complete C11 Implementation: Connectivity, Components, DFS Path, and BFS Shortest Path

```c
/**
 * @file graph_connectivity_and_paths.c
 * @brief Complete C11 implementation of Undirected/Directed Connectivity Testing,
 *        Connected Component Labelling, DFS Path Finding, and BFS Shortest Path.
 *        Compile: gcc -std=c11 -Wall -Wextra graph_connectivity_and_paths.c -o graph_paths
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

/* ============================================================================
 * 1. GRAPH DATA STRUCTURE (ADJACENCY LIST) & FIFO QUEUE
 * ============================================================================ */

typedef struct AdjNode {
    int vertex;
    struct AdjNode *next;
} AdjNode;

typedef struct Graph {
    int num_vertices;
    bool is_directed;
    AdjNode **head;
} Graph;

typedef struct Queue {
    int *items;
    int front;
    int rear;
    int capacity;
} Queue;

static Queue *createQueue(int capacity) {
    Queue *q = (Queue *)malloc(sizeof(Queue));
    if (!q) {
        perror("Failed to allocate Queue");
        exit(EXIT_FAILURE);
    }
    q->items = (int *)malloc((size_t)capacity * sizeof(int));
    if (!q->items) {
        perror("Failed to allocate Queue items");
        free(q);
        exit(EXIT_FAILURE);
    }
    q->front = 0;
    q->rear = -1;
    q->capacity = capacity;
    return q;
}

static bool isQueueEmpty(const Queue *q) {
    return q->rear < q->front;
}

static void enqueue(Queue *q, int value) {
    if (q->rear + 1 >= q->capacity) {
        fprintf(stderr, "Error: Queue overflow\n");
        return;
    }
    q->items[++q->rear] = value;
}

static int dequeue(Queue *q) {
    return q->items[q->front++];
}

static void freeQueue(Queue *q) {
    if (q) {
        free(q->items);
        free(q);
    }
}

static AdjNode *createAdjNode(int v) {
    AdjNode *node = (AdjNode *)malloc(sizeof(AdjNode));
    if (!node) {
        perror("Failed to allocate AdjNode");
        exit(EXIT_FAILURE);
    }
    node->vertex = v;
    node->next = NULL;
    return node;
}

Graph *createGraph(int vertices, bool is_directed) {
    Graph *g = (Graph *)malloc(sizeof(Graph));
    if (!g) {
        perror("Failed to allocate Graph");
        exit(EXIT_FAILURE);
    }
    g->num_vertices = vertices;
    g->is_directed = is_directed;
    g->head = (AdjNode **)calloc((size_t)vertices, sizeof(AdjNode *));
    if (!g->head) {
        perror("Failed to allocate adjacency list heads");
        free(g);
        exit(EXIT_FAILURE);
    }
    return g;
}

/* Append edge to the tail of the adjacency list to preserve insertion order */
static void appendDirectedEdge(Graph *g, int u, int v) {
    AdjNode *new_node = createAdjNode(v);
    if (g->head[u] == NULL) {
        g->head[u] = new_node;
    } else {
        AdjNode *curr = g->head[u];
        while (curr->next != NULL) {
            curr = curr->next;
        }
        curr->next = new_node;
    }
}

void addEdge(Graph *g, int u, int v) {
    if (!g || u < 0 || u >= g->num_vertices || v < 0 || v >= g->num_vertices) {
        return;
    }
    appendDirectedEdge(g, u, v);
    if (!g->is_directed) {
        appendDirectedEdge(g, v, u);
    }
}

void freeGraph(Graph *g) {
    if (!g) return;
    for (int i = 0; i < g->num_vertices; i++) {
        AdjNode *curr = g->head[i];
        while (curr) {
            AdjNode *temp = curr;
            curr = curr->next;
            free(temp);
        }
    }
    free(g->head);
    free(g);
}

/* ============================================================================
 * 2. CONNECTIVITY TESTING (UNDIRECTED & DIRECTED STRONG CONNECTIVITY)
 * ============================================================================ */

static void dfsReach(const Graph *g, int u, bool visited[], int *visited_count) {
    visited[u] = true;
    (*visited_count)++;
    for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
        int v = curr->vertex;
        if (!visited[v]) {
            dfsReach(g, v, visited, visited_count);
        }
    }
}

bool isUndirectedConnected(const Graph *g) {
    if (!g || g->num_vertices <= 1) return true;

    bool *visited = (bool *)calloc((size_t)g->num_vertices, sizeof(bool));
    int visited_count = 0;

    dfsReach(g, 0, visited, &visited_count);

    free(visited);
    return visited_count == g->num_vertices;
}

static Graph *createTransposeGraph(const Graph *g) {
    Graph *gt = createGraph(g->num_vertices, true);
    for (int u = 0; u < g->num_vertices; u++) {
        for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
            int v = curr->vertex;
            appendDirectedEdge(gt, v, u); /* Reverse (u -> v) to (v -> u) */
        }
    }
    return gt;
}

bool isDirectedStronglyConnected(const Graph *g) {
    if (!g || g->num_vertices <= 1) return true;

    int V = g->num_vertices;
    bool *visited = (bool *)calloc((size_t)V, sizeof(bool));
    int count_forward = 0;

    /* Pass 1: Can vertex 0 reach all vertices in G? */
    dfsReach(g, 0, visited, &count_forward);
    if (count_forward < V) {
        free(visited);
        return false;
    }

    /* Pass 2: Can all vertices reach vertex 0 in G (i.e., 0 reaches all in G^T)? */
    Graph *gt = createTransposeGraph(g);
    for (int i = 0; i < V; i++) visited[i] = false;
    int count_transpose = 0;

    dfsReach(gt, 0, visited, &count_transpose);

    freeGraph(gt);
    free(visited);
    return count_transpose == V;
}

/* ============================================================================
 * 3. COUNTING AND LABELLING CONNECTED COMPONENTS
 * ============================================================================ */

static void dfsLabelComponent(const Graph *g, int u, int comp_id, int component_id[]) {
    component_id[u] = comp_id;
    for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
        int v = curr->vertex;
        if (component_id[v] == 0) {
            dfsLabelComponent(g, v, comp_id, component_id);
        }
    }
}

int findConnectedComponents(const Graph *g, int component_id[]) {
    int V = g->num_vertices;
    for (int v = 0; v < V; v++) {
        component_id[v] = 0; /* 0 means unvisited / unlabelled */
    }

    int comp_count = 0;
    for (int v = 0; v < V; v++) {
        if (component_id[v] == 0) {
            comp_count++;
            dfsLabelComponent(g, v, comp_count, component_id);
        }
    }
    return comp_count;
}

void printConnectedComponents(const Graph *g) {
    int V = g->num_vertices;
    int *component_id = (int *)malloc((size_t)V * sizeof(int));
    int total_comps = findConnectedComponents(g, component_id);

    printf("Total Connected Components Found: %d\n", total_comps);
    printf("Vertex Component ID Array: [ ");
    for (int v = 0; v < V; v++) {
        printf("%d:%d ", v, component_id[v]);
    }
    printf("]\n");

    for (int c = 1; c <= total_comps; c++) {
        printf("  Component C%d: { ", c);
        bool first = true;
        for (int v = 0; v < V; v++) {
            if (component_id[v] == c) {
                if (!first) printf(", ");
                printf("%d", v);
                first = false;
            }
        }
        printf(" }\n");
    }
    free(component_id);
}

/* ============================================================================
 * 4. PATH FINDING: DFS PATH vs. BFS SHORTEST PATH & RECONSTRUCTION
 * ============================================================================ */

void printReconstructedPath(int s, int t, const int parent[]) {
    if (s != t && parent[t] == -1) {
        printf("  No path exists from %d to %d.\n", s, t);
        return;
    }

    /* Count vertices along the predecessor chain from t back to s */
    int capacity = 64;
    int *path = (int *)malloc((size_t)capacity * sizeof(int));
    int len = 0;

    int curr = t;
    while (curr != -1) {
        if (len >= capacity) {
            capacity *= 2;
            path = (int *)realloc(path, (size_t)capacity * sizeof(int));
        }
        path[len++] = curr;
        if (curr == s) break;
        curr = parent[curr];
    }

    if (path[len - 1] != s) {
        printf("  No path exists from %d to %d.\n", s, t);
        free(path);
        return;
    }

    /* Reverse the array in-place so path reads s -> ... -> t */
    for (int i = 0; i < len / 2; i++) {
        int temp = path[i];
        path[i] = path[len - 1 - i];
        path[len - 1 - i] = temp;
    }

    printf("  Path (%d edges): ", len - 1);
    for (int i = 0; i < len; i++) {
        printf("%d%s", path[i], (i < len - 1) ? " -> " : "\n");
    }
    free(path);
}

static bool dfsPathUtil(const Graph *g, int u, int t, bool visited[], int parent[]) {
    visited[u] = true;
    if (u == t) {
        return true; /* Target reached! */
    }
    for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
        int v = curr->vertex;
        if (!visited[v]) {
            parent[v] = u;
            if (dfsPathUtil(g, v, t, visited, parent)) {
                return true;
            }
        }
    }
    return false;
}

bool findPathDFS(const Graph *g, int s, int t, int parent[]) {
    int V = g->num_vertices;
    bool *visited = (bool *)calloc((size_t)V, sizeof(bool));
    for (int i = 0; i < V; i++) {
        parent[i] = -1;
    }
    bool found = dfsPathUtil(g, s, t, visited, parent);
    free(visited);
    return found;
}

bool findShortestPathBFS(const Graph *g, int s, int t, int parent[], int dist[]) {
    int V = g->num_vertices;
    bool *visited = (bool *)calloc((size_t)V, sizeof(bool));
    for (int i = 0; i < V; i++) {
        parent[i] = -1;
        dist[i] = -1;
    }

    Queue *q = createQueue(V);
    visited[s] = true;
    dist[s] = 0;
    enqueue(q, s);

    bool found = false;
    while (!isQueueEmpty(q)) {
        int u = dequeue(q);
        if (u == t) {
            found = true;
            /* Note: If we only need the shortest path to t, we can break here,
             * or continue to compute dist[] to all reachable vertices. */
        }
        for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
            int v = curr->vertex;
            if (!visited[v]) {
                visited[v] = true;
                dist[v] = dist[u] + 1;
                parent[v] = u;
                enqueue(q, v);
            }
        }
    }

    freeQueue(q);
    free(visited);
    return found;
}

/* ============================================================================
 * 5. MAIN DRIVER DEMONSTRATION
 * ============================================================================ */

int main(void) {
    printf("=== 1. UNDIRECTED & DIRECTED CONNECTIVITY TESTING ===\n");
    Graph *undir_g = createGraph(8, false);
    /* Component 1: {0, 1, 2} */
    addEdge(undir_g, 0, 1);
    addEdge(undir_g, 1, 2);
    addEdge(undir_g, 2, 0);
    /* Component 2: {3, 4, 5, 6} */
    addEdge(undir_g, 3, 4);
    addEdge(undir_g, 4, 5);
    addEdge(undir_g, 5, 6);
    addEdge(undir_g, 3, 6);
    /* Component 3: {7} (isolated vertex) */

    printf("Undirected 8-vertex graph connected? %s\n",
           isUndirectedConnected(undir_g) ? "YES" : "NO");

    /* Directed graph strong connectivity check */
    Graph *dir_sc = createGraph(4, true);
    addEdge(dir_sc, 0, 1);
    addEdge(dir_sc, 1, 2);
    addEdge(dir_sc, 2, 3);
    addEdge(dir_sc, 3, 0); /* Forms a directed cycle 0->1->2->3->0 */
    printf("Directed 4-cycle strongly connected? %s\n",
           isDirectedStronglyConnected(dir_sc) ? "YES" : "NO");

    Graph *dir_weak = createGraph(4, true);
    addEdge(dir_weak, 0, 1);
    addEdge(dir_weak, 1, 2);
    addEdge(dir_weak, 2, 3); /* 0 reaches all, but 3 cannot reach 0 */
    printf("Directed chain (0->1->2->3) strongly connected? %s\n\n",
           isDirectedStronglyConnected(dir_weak) ? "YES" : "NO");

    printf("=== 2. COUNTING AND LABELLING CONNECTED COMPONENTS ===\n");
    printConnectedComponents(undir_g);
    printf("\n");

    printf("=== 3. DFS PATH vs. BFS SHORTEST PATH (COUNTEREXAMPLE) ===\n");
    /*
     * Counterexample Graph (5 vertices):
     *   0 connects to 1 first, and also directly to 4.
     *   Path A (explored first by DFS): 0 -> 1 -> 2 -> 3 -> 4 (4 edges)
     *   Path B (direct edge):           0 -> 4                (1 edge)
     */
    Graph *counter_g = createGraph(5, false);
    addEdge(counter_g, 0, 1);
    addEdge(counter_g, 1, 2);
    addEdge(counter_g, 2, 3);
    addEdge(counter_g, 3, 4);
    addEdge(counter_g, 0, 4); /* Direct shortcut edge added after (0,1) */

    int parent[8];
    int dist[8];

    findPathDFS(counter_g, 0, 4, parent);
    printf("[DFS Path Finder from s=0 to t=4]:\n");
    printReconstructedPath(0, 4, parent);

    findShortestPathBFS(counter_g, 0, 4, parent, dist);
    printf("[BFS Shortest Path from s=0 to t=4 (dist[4] = %d)]:\n", dist[4]);
    printReconstructedPath(0, 4, parent);
    printf("\n");

    printf("=== 4. MULTI-HOP NETWORK ROUTING & UNREACHABLE TARGET ===\n");
    /*
     * 7-vertex communication network:
     *   0 -- 1 -- 3 -- 5
     *   |    |         |
     *   2 -- 4 --------+
     *   Vertex 6 is isolated (offline node)
     */
    Graph *net_g = createGraph(7, false);
    addEdge(net_g, 0, 1);
    addEdge(net_g, 0, 2);
    addEdge(net_g, 1, 3);
    addEdge(net_g, 1, 4);
    addEdge(net_g, 2, 4);
    addEdge(net_g, 3, 5);
    addEdge(net_g, 4, 5);

    findShortestPathBFS(net_g, 0, 5, parent, dist);
    printf("BFS from source s=0 across 7-node network:\n");
    printf("  Vertex:   ");
    for (int v = 0; v < 7; v++) printf("%3d ", v);
    printf("\n  dist[]:   ");
    for (int v = 0; v < 7; v++) printf("%3d ", dist[v]);
    printf("\n  parent[]: ");
    for (int v = 0; v < 7; v++) printf("%3d ", parent[v]);
    printf("\n");

    printf("Shortest route from 0 to 5:\n");
    printReconstructedPath(0, 5, parent);
    printf("Route check from 0 to isolated node 6:\n");
    printReconstructedPath(0, 6, parent);

    freeGraph(undir_g);
    freeGraph(dir_sc);
    freeGraph(dir_weak);
    freeGraph(counter_g);
    freeGraph(net_g);
    return 0;
}
```

#### Program Output:

```text
=== 1. UNDIRECTED & DIRECTED CONNECTIVITY TESTING ===
Undirected 8-vertex graph connected? NO
Directed 4-cycle strongly connected? YES
Directed chain (0->1->2->3) strongly connected? NO

=== 2. COUNTING AND LABELLING CONNECTED COMPONENTS ===
Total Connected Components Found: 3
Vertex Component ID Array: [ 0:1 1:1 2:1 3:2 4:2 5:2 6:2 7:3 ]
  Component C1: { 0, 1, 2 }
  Component C2: { 3, 4, 5, 6 }
  Component C3: { 7 }

=== 3. DFS PATH vs. BFS SHORTEST PATH (COUNTEREXAMPLE) ===
[DFS Path Finder from s=0 to t=4]:
  Path (4 edges): 0 -> 1 -> 2 -> 3 -> 4
[BFS Shortest Path from s=0 to t=4 (dist[4] = 1)]:
  Path (1 edges): 0 -> 4

=== 4. MULTI-HOP NETWORK ROUTING & UNREACHABLE TARGET ===
BFS from source s=0 across 7-node network:
  Vertex:     0   1   2   3   4   5   6 
  dist[]:     0   1   1   2   2   3  -1 
  parent[]:  -1   0   0   1   1   3  -1 
Shortest route from 0 to 5:
  Path (3 edges): 0 -> 1 -> 3 -> 5
Route check from 0 to isolated node 6:
  No path exists from 0 to 6.
```

---

## 5. Comparison Tables and Complexity Summary

### 5.1 Time and Space Complexity Table (Adjacency List vs. Adjacency Matrix)

Let $V = |V|$ be the number of vertices, $E = |E|$ be the number of edges, and $L = d(s, t) \le V - 1$ be the length of the reconstructed path.

| Operation / Algorithm | Time (Adjacency List) | Time (Adjacency Matrix) | Auxiliary Space | Notes |
| :--- | :---: | :---: | :---: | :--- |
| **Undirected Connectivity Test** | $O(V + E)$ | $O(V^2)$ | $O(V)$ | Single DFS/BFS from vertex `0` |
| **Directed Strong Connectivity (2-Pass)** | $O(V + E)$ | $O(V^2)$ | $O(V + E)$ | Builds $G^T$ + 2 traversals from `0` |
| **Connected Components (`findConnectedComponents`)** | $O(V + E)$ | $O(V^2)$ | $O(V)$ | Outer loop $V$ + aggregate traversal |
| **Same-Component Query (`comp_id[u] == comp_id[v]`)** | $O(1)$ | $O(1)$ | $O(1)$ | After $O(V + E)$ preprocessing |
| **DFS Path Finding ($s \leadsto t$)** | $O(V + E)$ | $O(V^2)$ | $O(V)$ | Finds *a* simple path; **not shortest** |
| **BFS Shortest Path ($s \leadsto t$)** | $O(V + E)$ | $O(V^2)$ | $O(V)$ | Guarantees minimum-hop path $d(s, t)$ |
| **Path Reconstruction (`printReconstructedPath`)** | $O(L) \subseteq O(V)$ | $O(L) \subseteq O(V)$ | $O(L) \subseteq O(V)$ | Backtracks `parent[]` and reverses |

### 5.2 Algorithmic Comparison: BFS Shortest Path vs. DFS Path Finding vs. Weighted Shortest Paths

| Feature / Property | DFS Path Finding | BFS Shortest Path | Dijkstra's Algorithm | Bellman-Ford Algorithm |
| :--- | :--- | :--- | :--- | :--- |
| **Supported Graph Type** | Unweighted / Any | **Unweighted** (or uniform weight $w = c$) | Weighted ($w(u, v) \ge 0$) | Weighted (allows $w(u, v) < 0$, no negative cycles) |
| **Guarantees Shortest Path?** | **NO** | **YES** (minimum edges) | **YES** (minimum total weight) | **YES** (minimum total weight) |
| **Data Structure Used** | Recursion Stack / LIFO Stack | **FIFO Queue** | Min-Priority Queue (Binary Heap) | Edge list relaxation ($V - 1$ passes) |
| **Time Complexity (Adj. List)** | $O(V + E)$ | **$O(V + E)$** | $O((V + E) \log V)$ | $O(V \cdot E)$ |
| **Auxiliary Space** | $O(V)$ | $O(V)$ | $O(V)$ | $O(V)$ |

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Pitfall 1: Assuming DFS Finds the Shortest Path in an Unweighted Graph

```c
/* BUG: Using DFS when the problem asks for the MINIMUM number of hops/steps */
bool getMinimumHops(const Graph *g, int s, int t, int parent[]) {
    /* DFS explores deeply along the first neighbor of s; if that branch
     * eventually reaches t via V-1 edges, DFS returns it even if edge (s, t) exists! */
    return findPathDFS(g, s, t, parent);
}

/* FIX: Always use BFS with a FIFO Queue for unweighted shortest-path queries */
bool getMinimumHops(const Graph *g, int s, int t, int parent[], int dist[]) {
    return findShortestPathBFS(g, s, t, parent, dist);
}
```

### 6.2 Pitfall 2: Marking Vertices `visited` Upon Dequeue Instead of Enqueue in BFS

If you delay setting `visited[v] = true` until `v` is **dequeued**, multiple vertices in the current level can enqueue the same unvisited neighbor `v` multiple times, overflowing the queue of capacity $V$ and overwriting `parent[v]` and `dist[v]` with a later (potentially deeper!) discovery!

```c
/* BUG: Marking visited[u] = true when DEQUEUEING u */
while (!isQueueEmpty(q)) {
    int u = dequeue(q);
    visited[u] = true; /* TOO LATE! u may already be in q multiple times! */
    for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
        int v = curr->vertex;
        if (!visited[v]) {
            parent[v] = u;          /* Overwrites earlier parent[v]! */
            dist[v] = dist[u] + 1;  /* Overwrites earlier dist[v]!   */
            enqueue(q, v);          /* Enqueues duplicate v!         */
        }
    }
}

/* FIX: Mark visited[v] = true IMMEDIATELY when ENQUEUEING v (and for s before the loop) */
visited[s] = true;
dist[s] = 0;
enqueue(q, s);
while (!isQueueEmpty(q)) {
    int u = dequeue(q);
    for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
        int v = curr->vertex;
        if (!visited[v]) {
            visited[v] = true;      /* Lock in v immediately upon discovery! */
            dist[v] = dist[u] + 1;
            parent[v] = u;
            enqueue(q, v);
        }
    }
}
```

### 6.3 Pitfall 3: Printing the `parent[]` Chain Without Reversing It

```c
/* BUG: Printing curr directly while walking parent[] from t to s */
int curr = t;
while (curr != -1) {
    printf("%d ", curr); /* Prints t -> ... -> s BACKWARDS! */
    curr = parent[curr];
}

/* FIX: Store the chain in an array (or push onto a stack) and reverse before printing */
int len = 0;
for (int curr = t; curr != -1; curr = parent[curr]) {
    path[len++] = curr;
}
for (int i = len - 1; i >= 0; i--) {
    printf("%d%s", path[i], (i > 0) ? " -> " : "\n");
}
```

### 6.4 Pitfall 4: Testing Only Single-Pass Reachability from Vertex `0` on a Directed Graph

```c
/* BUG: Checking only whether vertex 0 can reach all vertices in a DIRECTED graph */
bool isStronglyConnectedBuggy(const Graph *g) {
    bool *visited = calloc(g->num_vertices, sizeof(bool));
    int count = 0;
    dfsReach(g, 0, visited, &count);
    free(visited);
    /* In 0 -> 1 -> 2, count == 3, but 2 cannot reach 0! */
    return count == g->num_vertices;
}

/* FIX: Also verify that vertex 0 can reach all vertices in the Transpose Graph G^T */
```

### 6.5 Boundary Conditions Checklist

- [ ] **Trivial Path ($s == t$):** Does `findShortestPathBFS(g, s, s, ...)` report `dist[s] = 0` and reconstruct the 0-edge path `s` (even though `parent[s] == -1`)?
- [ ] **Unreachable Target (`dist[t] == -1`):** Does path reconstruction check `if (s != t && parent[t] == -1)` before walking `parent[]`?
- [ ] **Isolated Vertices ($\text{deg}(v) == 0$):** Does `findConnectedComponents` assign each isolated vertex its own unique component ID?
- [ ] **Empty or Single-Vertex Graph ($V \le 1$):** Do connectivity functions handle $V \le 1$ without indexing `head[0]` on a 0-vertex graph?
- [ ] **Disconnected Cycles:** Does the component counter still discover a cycle `{3, 4, 5, 6}` that is disconnected from vertex `0`? (Yes, via the outer loop over $v \in \{0 \dots V-1\}$).

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways and Core Invariants

1. **Undirected Connectivity & Components:** Undirected reachability is an equivalence relation that partitions $V$ into $k$ disjoint connected components. A single BFS/DFS from vertex `0` tests connectivity ($k = 1$), while an outer loop over all $v \in \{0 \dots V-1\}$ labels all $k$ components in aggregate **$O(V + E)$ time**.
2. **Directed Strong Connectivity:** Requires mutual reachability ($u \leadsto v$ and $v \leadsto u$). Testing that vertex `0` reaches all vertices in $G$ **and** reaches all vertices in $G^T$ verifies strong connectivity in **$O(V + E)$ time**.
3. **Predecessor Tree (`parent[]`):** Setting `parent[v] = u` when $u$ first discovers $v$ encodes a tree rooted at $s$ (`parent[s] = -1`). Walking `curr = parent[curr]` from $t$ to $s$ traces the path in reverse.
4. **DFS vs. BFS for Path Finding:** DFS finds *some* simple path in $O(V + E)$ time but can return a path of $V - 1$ edges even when a 1-edge direct link exists. **BFS explores vertices in strictly non-decreasing order of hop distance $d(s, v)$**, guaranteeing that the first time $t$ is discovered, `dist[t]` and `parent[]` yield a **shortest unweighted path**.

### 7.2 Self-Test Revision Questions

1. **Complexity Question:** In `findConnectedComponents`, there is an outer `for` loop that runs $V$ times, and inside it a call to `dfsLabelComponent`. Why is the worst-case time complexity $O(V + E)$ instead of $O(V(V + E))$?
   *(Answer: Because `dfsLabelComponent(G, v, ...)` is guarded by `if (component_id[v] == 0)` and immediately marks every reached vertex with a non-zero `comp_id`. Thus, each vertex is visited once and each adjacency list is scanned once across the entire execution).*
2. **Proof/Trace Question:** Suppose an undirected graph has $V = 10$ vertices and $E = 7$ edges. What is the minimum possible number of connected components $k$ in this graph? Can this graph ever be connected?
   *(Answer: By the inequality $E \ge V - k$, we have $k \ge V - E = 10 - 7 = 3$. The graph must have at least $3$ connected components and can never be connected, since a connected 10-vertex graph requires at least $V - 1 = 9$ edges).*
3. **Algorithm Design Question:** Given a directed graph $G = (V, E)$ and a target destination vertex $t$, how can you find the shortest unweighted distance from **every** vertex $u \in V$ **to** $t$ in a single $O(V + E)$ BFS pass instead of running $V$ separate BFS passes?
   *(Answer: Construct the transpose graph $G^T$ by reversing all directed edges, and run a single BFS starting from source $t$ in $G^T$. Since a path $u \leadsto t$ of length $k$ in $G$ corresponds to a path $t \leadsto u$ of length $k$ in $G^T$, `dist[u]` in $G^T$ equals $d_G(u, t)$ for all $u \in V$).*

### 7.3 Navigation Links

- **Previous Topic (Unit 3):** [Unit 3 — Graph Traversals (BFS and DFS)](../Unit-3-Tree-Applications-and-Graphs/08-DFS-and-BFS.md)
- **Current File:** `01-Graph-Connectivity-Components-and-Paths.md` (Unit 4 — Topic 1)
- **Next Topic:** [02-Trie-Trees.md](./02-Trie-Trees.md) — Trie Trees (Prefix Trees / Digital Search Trees)
- **Further Ahead:** [03-Suffix-Tries-and-Suffix-Trees.md](./03-Suffix-Tries-and-Suffix-Trees.md) — Suffix Tries and Compressed Suffix Trees
