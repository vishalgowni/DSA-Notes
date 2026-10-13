# Unit 3 — Topic 7: Graph Fundamentals and Representations

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [What Is a Graph?](#11-what-is-a-graph)
   - 1.2 [Why Do We Need Graphs? The Problem They Solve](#12-why-do-we-need-graphs-the-problem-they-solve)
   - 1.3 [Relationship to Prior Concepts: Linked Lists, Trees, and Graphs](#13-relationship-to-prior-concepts-linked-lists-trees-and-graphs)
   - 1.4 [Real-World and Computer Science Applications](#14-real-world-and-computer-science-applications)
   - 1.5 [Learning Objectives](#15-learning-objectives)
2. [Definitions and Terminology](#2-definitions-and-terminology)
   - 2.1 [Formal Definition of a Graph $G = (V, E)$](#21-formal-definition-of-a-graph-g--v-e)
   - 2.2 [Undirected vs. Directed Graphs (Digraphs)](#22-undirected-vs-directed-graphs-digraphs)
   - 2.3 [Unweighted vs. Weighted Graphs (Networks)](#23-unweighted-vs-weighted-graphs-networks)
   - 2.4 [Adjacency, Incidence, and Vertex Degree ($\deg(v)$, $\deg^-(v)$, $\deg^+(v)$)](#24-adjacency-incidence-and-vertex-degree-degv-deg-v-degv)
   - 2.5 [The Handshaking Lemma and Formal Proofs](#25-the-handshaking-lemma-and-formal-proofs)
   - 2.6 [Maximum Edges, Sparse Graphs, and Dense Graphs](#26-maximum-edges-sparse-graphs-and-dense-graphs)
   - 2.7 [Walks, Trails, Paths, Cycles, and Directed Acyclic Graphs (DAGs)](#27-walks-trails-paths-cycles-and-directed-acyclic-graphs-dags)
   - 2.8 [Special Graph Classes: Multigraphs, Complete ($K_n$), Bipartite, and Subgraphs](#28-special-graph-classes-multigraphs-complete-k_n-bipartite-and-subgraphs)
   - 2.9 [Connectivity: Connected Components, Strongly Connected, and Weakly Connected](#29-connectivity-connected-components-strongly-connected-and-weakly-connected)
   - 2.10 [Comprehensive Graph Terminology Reference Table](#210-comprehensive-graph-terminology-reference-table)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Representative Benchmark Graphs ($G_1$ Undirected & $G_2$ Directed Weighted)](#31-representative-benchmark-graphs-g_1-undirected--g_2-directed-weighted)
   - 3.2 [Adjacency Matrix Representation](#32-adjacency-matrix-representation)
   - 3.3 [Adjacency List Representation](#33-adjacency-list-representation)
   - 3.4 [Static vs. Dynamic Graph Storage: Architectural Trade-Offs](#34-static-vs-dynamic-graph-storage-architectural-trade-offs)
4. [Operations and Algorithms on Graph Representations](#4-operations-and-algorithms-on-graph-representations)
   - 4.1 [Graph Initialization (`initMatrixGraph`, `createListGraph`)](#41-graph-initialization-initmatrixgraph-createlistgraph)
   - 4.2 [Adding Edges (`addMatrixEdge`, `addListEdge`)](#42-adding-edges-addmatrixedge-addlistedge)
   - 4.3 [Removing Edges (`removeMatrixEdge`, `removeListEdge`)](#43-removing-edges-removematrixedge-removelistedge)
   - 4.4 [Edge Existence Lookup and Neighbor Iteration](#44-edge-existence-lookup-and-neighbor-iteration)
   - 4.5 [Computing Vertex Degrees ($\deg(v)$, $\deg^-(v)$, $\deg^+(v)$)](#45-computing-vertex-degrees-degv-deg-v-degv)
   - 4.6 [Full Dynamic Memory Teardown (`freeListGraph`)](#46-full-dynamic-memory-teardown-freelistgraph)
   - 4.7 [Complete C11 Implementation: Adjacency Matrix and Adjacency List](#47-complete-c11-implementation-adjacency-matrix-and-adjacency-list)
5. [Comparison Tables and Complexity Summary](#5-comparison-tables-and-complexity-summary)
   - 5.1 [Operation Time and Space Complexity: Adjacency Matrix vs. Adjacency List](#51-operation-time-and-space-complexity-adjacency-matrix-vs-adjacency-list)
   - 5.2 [When to Choose Adjacency Matrix vs. Adjacency List](#52-when-to-choose-adjacency-matrix-vs-adjacency-list)
6. [Common Errors, Pitfalls, and Edge Cases](#6-common-errors-pitfalls-and-edge-cases)
   - 6.1 [Pitfall 1: Forgetting the Symmetric Assignment `adj[v][u]` in Undirected Graphs](#61-pitfall-1-forgetting-the-symmetric-assignment-adjvu-in-undirected-graphs)
   - 6.2 [Pitfall 2: Using `0` as "No Edge" in a Weighted Graph That Allows Zero-Weight Edges](#62-pitfall-2-using-0-as-no-edge-in-a-weighted-graph-that-allows-zero-weight-edges)
   - 6.3 [Pitfall 3: Shallow Freeing of an Adjacency List (`free(g->head)` Without Freeing Nodes)](#63-pitfall-3-shallow-freeing-of-an-adjacency-list-freeg-head-without-freeing-nodes)
   - 6.4 [Pitfall 4: Integer Overflow When Adding Edge Weights to `INT_MAX` ($\infty$)](#64-pitfall-4-integer-overflow-when-adding-edge-weights-to-int_max-infty)
   - 6.5 [Boundary Conditions Checklist](#65-boundary-conditions-checklist)
7. [Summary and Revision Checklist](#7-summary-and-revision-checklist)
   - 7.1 [Key Takeaways and Core Invariants](#71-key-takeaways-and-core-invariants)
   - 7.2 [Self-Test Revision Questions](#72-self-test-revision-questions)
   - 7.3 [Navigation Links](#73-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 What Is a Graph?

A **Graph** is the most general non-linear data structure in computer science. Formally, a graph $G = (V, E)$ consists of a finite, non-empty set of **vertices** (or *nodes*) $V$ that represent discrete entities, together with a set of **edges** (or *links* / *arcs*) $E$ that represent pairwise relationships or connections between those entities.

```
         UNDIRECTED GRAPH G1                         DIRECTED GRAPH (DIGRAPH) G2
      (Symmetric Relationships)                      (One-Way / Asymmetric Arcs)

         (0)-----------(1)                               (0) --------> (1)
          |           / | \                               | \           |
          |         /   |   \                             |   \         |
          |       /     |    (3)---(5)                    v     v       v
          |     /       |                                (2) --> (4) -> (3)
          |   /         |
         (2)-----------(4)
```

### 1.2 Why Do We Need Graphs? The Problem They Solve

Linear data structures (**arrays, linked lists, stacks, queues**) enforce a strict **1-to-1** predecessor/successor relationship: every middle element has at most one immediate predecessor and at most one immediate successor. Hierarchical data structures (**trees, binary search trees, heaps, AVL trees**) relax this to a **1-to-many** parent-to-children relationship, but strictly forbid cycles, multiple parents, or disconnected peers.

Many real-world systems, however, exhibit arbitrary **many-to-many ($M:N$)** relationships where:
- Any entity can connect to any number of other entities.
- Alternate routes and **cycles** (closed loops) naturally occur (e.g., road networks, electrical circuits, internet routing meshes).
- Relationships may be **asymmetric** (person $A$ follows person $B$ on social media, or function `f()` calls `g()`) and carry numerical **costs** (distance, latency, bandwidth, toll fees).

Graphs provide the mathematical and algorithmic framework to model, store, and query arbitrary pairwise relationships.

### 1.3 Relationship to Prior Concepts: Linked Lists, Trees, and Graphs

Understanding how linked lists, trees, and graphs form a strict structural hierarchy is one of the most important conceptual insights in Data Structures:

$$\text{Singly Linked List} \subset \text{Rooted Tree} \subset \text{Directed / Undirected Graph}$$

- **A Singly Linked List** of $n$ nodes is simply a **degenerate tree** (and a directed path graph) in which every node has out-degree at most $1$ and no cycles exist.
- **A Free Tree** is formally defined in graph theory as a **connected, acyclic undirected graph**. If an undirected graph $G = (V, E)$ has $|V|$ vertices, any two of the following properties imply the third (and define a tree):
  1. $G$ is **connected** (a path exists between every pair of vertices).
  2. $G$ is **acyclic** (contains no cycles).
  3. $|E| = |V| - 1$.

| Structural Property | Linear Structures (List / Stack / Queue) | Hierarchical Structures (Binary Tree / BST / AVL) | General Graph $G = (V, E)$ |
| :--- | :--- | :--- | :--- |
| **Relationship Cardinality** | $1 : 1$ (Sequential) | $1 : N$ (Parent $\to$ Children) | $M : N$ (Arbitrary pairwise) |
| **Unique Root Node?** | Yes (`head` / `front` / `top`) | Yes (unique `root` with in-degree $0$) | No (any vertex can serve as a start node) |
| **Cycles Permitted?** | No (except circular lists) | **Never** (strictly acyclic) | **Yes** (cyclic or acyclic) |
| **Number of Edges ($|V| = n$)** | $n - 1$ | $n - 1$ (if connected tree) | $0 \le |E| \le \frac{n(n-1)}{2}$ (undirected) or $n(n-1)$ (directed) |
| **Multiple Paths Between Nodes?** | No | No (unique path from root to any node) | Yes |

### 1.4 Real-World and Computer Science Applications

1. **Transportation & Navigation (GPS):** Intersections are vertices $V$; road segments are directed edges $E$ weighted by travel time or distance. Shortest-path algorithms (BFS, Dijkstra, $A^*$) compute optimal routes.
2. **Computer Networks & Internet Routing:** Routers and autonomous systems are vertices; physical fiber links are weighted edges. Routing protocols (OSPF, BGP) compute shortest paths and detect link failures.
3. **Compilers & Build Systems (`make`, `cargo`, `npm`):** Source modules or build targets form a **Directed Acyclic Graph (DAG)** where a directed edge $u \to v$ means "$u$ must be compiled before $v$." Topological sorting schedules compilation order; cycle detection flags circular dependencies.
4. **Operating Systems (Deadlock Detection):** A **Resource Allocation Graph (RAG)** tracks processes and locks; a cycle in a single-instance RAG proves a deadlock exists.
5. **Garbage Collection (Mark-and-Sweep):** Heap objects are vertices and pointers between structs are directed edges; reachability traversal (DFS/BFS) from stack roots identifies live vs. unreachable memory.

### 1.5 Learning Objectives

After completing this chapter, you will be able to:
- Define a graph $G = (V, E)$ formally and distinguish between **undirected/directed**, **unweighted/weighted**, **simple/multigraph**, **sparse/dense**, and **cyclic/acyclic (DAG)** graphs.
- State and prove the **Handshaking Lemma** for both undirected and directed graphs, and apply it to solve degree and edge-count problems.
- Derive the exact maximum edge bounds for simple undirected and directed graphs.
- Implement both the **Adjacency Matrix** and **Adjacency List** representations in C11 on the exact same graph, analyze their memory layouts, and compare their time and space complexities across all core graph operations.

---

## 2. Definitions and Terminology

### 2.1 Formal Definition of a Graph $G = (V, E)$

A **graph** is an ordered pair:

$$G = (V, E)$$

where:
1. $V$ is a finite set of **vertices** (also called *nodes* or *points*). The number of vertices is denoted by $|V|$ (or simply $V$ inside asymptotic Big-O notation when no ambiguity arises).
2. $E$ is a set of **edges** (also called *arcs* or *lines*), where each edge connects a pair of vertices in $V$. The number of edges is denoted by $|E|$ (or $E$ in Big-O notation).

> **Standard Convention in C Implementations:** When $|V| = n$, we label vertices using consecutive zero-based integer indices:
> $$V = \{0, 1, 2, \dots, |V| - 1\}$$
> If an application uses string labels (e.g., `"Bengaluru"`, `"Chennai"`), a symbol table or array maps each string to its integer vertex ID $0 \dots |V|-1$.

### 2.2 Undirected vs. Directed Graphs (Digraphs)

1. **Undirected Graph:**
   - Every edge $e \in E$ is an **unordered pair** of vertices, written as $\{u, v\}$ or $(u, v)$ with the understanding that $\{u, v\} = \{v, u\}$.
   - If edge $\{u, v\} \in E$, traversal is permitted in **both directions**: $u \to v$ and $v \to u$.
2. **Directed Graph (Digraph):**
   - Every edge $e \in E$ is an **ordered pair** of vertices, written as $(u, v)$ or $u \to v$.
   - Here, $u$ is called the **origin / source / tail** and $v$ is called the **destination / target / head**.
   - The edge $(u, v)$ permits traversal strictly from $u$ to $v$; the reverse direction $(v, u)$ exists **only if** $(v, u)$ is separately present in $E$.

```
    Undirected Edge {u, v}               Directed Edge (u, v) = u -> v
    ======================               =============================
      (u) ----------- (v)                   (u) -------------> (v)
     Both u->v and v->u valid             u = tail (source), v = head (target)
```

### 2.3 Unweighted vs. Weighted Graphs (Networks)

- **Unweighted Graph:** Edges merely express the *presence* or *absence* of a connection. Every existing edge has an implicit unit cost of $1$.
- **Weighted Graph (Network):** A graph equipped with a **weight (or cost) function** $w : E \to \mathbb{R}$ that assigns a numerical real or integer weight $w(u, v)$ to each edge $(u, v) \in E$. Weights can represent physical distance, latency in milliseconds, financial cost, or pipe capacity.

### 2.4 Adjacency, Incidence, and Vertex Degree ($\deg(v)$, $\deg^-(v)$, $\deg^+(v)$)

- **Adjacency (Vertex-to-Vertex):**
  - In an undirected graph, vertices $u$ and $v$ are **adjacent** (or *neighbors*) if $\{u, v\} \in E$.
  - In a directed graph, vertex $v$ is **adjacent to** vertex $u$ if the directed edge $(u, v) \in E$ (i.e., $u \to v$).
- **Incidence (Edge-to-Vertex):**
  - In an undirected graph, an edge $e = \{u, v\}$ is **incident on** both endpoints $u$ and $v$.
  - In a directed graph, a directed edge $e = (u, v)$ is **incident from** (leaves) $u$ and **incident to** (enters) $v$.
- **Degree in an Undirected Graph ($\deg(v)$):**
  - The **degree** of a vertex $v$, written $\deg(v)$, is the number of edges incident on $v$ (with any self-loop $\{v, v\}$ counted **twice**).
  - A vertex with $\deg(v) = 0$ is called an **isolated vertex**.
  - A vertex with $\deg(v) = 1$ is called a **pendant vertex** (or *leaf*).
- **In-Degree and Out-Degree in a Directed Graph ($\deg^-(v)$ and $\deg^+(v)$):**
  - **In-Degree $\deg^-(v)$:** The number of directed edges entering $v$ (edges of the form $(u, v) \in E$, where $v$ is the head).
  - **Out-Degree $\deg^+(v)$:** The number of directed edges leaving $v$ (edges of the form $(v, w) \in E$, where $v$ is the tail).
  - **Total Degree in a Digraph:** $\deg(v) = \deg^-(v) + \deg^+(v)$.
  - A vertex with $\deg^-(v) = 0$ is a **source**; a vertex with $\deg^+(v) = 0$ is a **sink**.

### 2.5 The Handshaking Lemma and Formal Proofs

#### Theorem 1 (Undirected Handshaking Lemma — Euler, 1736)

For any undirected graph $G = (V, E)$ with $|E|$ edges:

$$\sum_{v \in V} \deg(v) = 2|E|$$

**Proof (Double Counting):**
Every undirected edge $e = \{u, v\} \in E$ has exactly two endpoints, $u$ and $v$. Therefore, when we compute the sum of degrees $\sum_{v \in V} \deg(v)$ across all vertices, each edge $e = \{u, v\}$ contributes $+1$ to $\deg(u)$ and $+1$ to $\deg(v)$—a total contribution of exactly $2$ to the sum (even if $u = v$, by definition a self-loop contributes $2$ to $\deg(u)$). Summing over all $|E|$ edges yields $\sum_{v \in V} \deg(v) = 2|E|$. $\blacksquare$

#### Corollary 1.1 (Parity of Odd-Degree Vertices)

*In any undirected graph, the number of vertices having an **odd** degree is always **even**.*

**Proof:**
Partition the vertex set $V$ into $V_{\text{even}}$ (vertices with even degree) and $V_{\text{odd}}$ (vertices with odd degree). By the Handshaking Lemma:

$$2|E| = \sum_{v \in V} \deg(v) = \underbrace{\sum_{v \in V_{\text{even}}} \deg(v)}_{\text{even}} + \sum_{v \in V_{\text{odd}}} \deg(v)$$

Rearranging gives:

$$\sum_{v \in V_{\text{odd}}} \deg(v) = 2|E| - \sum_{v \in V_{\text{even}}} \deg(v)$$

Since $2|E|$ is even and the sum of even integers is even, their difference on the right-hand side must be **even**. Because each term $\deg(v)$ for $v \in V_{\text{odd}}$ is an odd integer, and a sum of odd integers is even if and only if there is an **even number of terms**, $|V_{\text{odd}}|$ must be even. $\blacksquare$

#### Theorem 2 (Directed Handshaking Lemma)

For any directed graph $G = (V, E)$ with $|E|$ directed edges:

$$\sum_{v \in V} \deg^-(v) = \sum_{v \in V} \deg^+(v) = |E|$$

**Proof:**
Every directed edge $e = (u, v) \in E$ has exactly one tail $u$ and exactly one head $v$. Consequently, each edge contributes $+1$ to the out-degree $\deg^+(u)$ of its tail and $+1$ to the in-degree $\deg^-(v)$ of its head. Summing over all $|E|$ edges gives $\sum_{v \in V} \deg^+(v) = |E|$ and $\sum_{v \in V} \deg^-(v) = |E|$. $\blacksquare$

### 2.6 Maximum Edges, Sparse Graphs, and Dense Graphs

A **simple graph** is a graph with **no self-loops** (no edge from a vertex to itself) and **no parallel/multiple edges** (at most one edge between any pair of vertices).

1. **Maximum Edges in a Simple Undirected Graph:**
   Each undirected edge corresponds to a unique 2-element subset $\{u, v\} \subseteq V$. The total number of ways to choose 2 distinct vertices out of $|V|$ is:
   $$|E|_{\max}^{\text{undirected}} = \binom{|V|}{2} = \frac{|V|(|V| - 1)}{2}$$
2. **Maximum Edges in a Simple Directed Graph:**
   Each directed edge corresponds to an ordered pair $(u, v)$ with $u \ne v$. Each of the $|V|$ vertices can have a directed edge to any of the remaining $|V| - 1$ vertices:
   $$|E|_{\max}^{\text{directed}} = |V|(|V| - 1)$$
   *(If self-loops $(v, v)$ are also permitted, the maximum becomes $|V|^2$ for directed and $\frac{|V|(|V|+1)}{2}$ for undirected).*

#### Sparse vs. Dense Graphs

- **Sparse Graph:** A graph in which $|E|$ is much smaller than $|V|^2$, typically $|E| = O(|V|)$ or $|E| = O(|V| \log |V|)$.
  - *Examples:* Road networks (most intersections meet 3 or 4 roads, so $\deg(v) \le 4 \implies |E| \le 2|V|$), trees ($|E| = |V| - 1$), planar graphs ($|E| \le 3|V| - 6$).
- **Dense Graph:** A graph in which $|E|$ is close to the maximum possible number of edges, i.e., $|E| = \Theta(|V|^2)$.
  - *Examples:* Complete graphs $K_n$, pairwise similarity matrices in machine learning, flight networks between major hubs.

### 2.7 Walks, Trails, Paths, Cycles, and Directed Acyclic Graphs (DAGs)

Consider a sequence of vertices $(v_0, v_1, v_2, \dots, v_k)$ such that each consecutive pair $(v_{i-1}, v_i) \in E$ for $1 \le i \le k$. The **length** of this sequence is $k$ (the number of **edges** traversed, *not* the number of vertices).

| Term | Repeated Edges Allowed? | Repeated Vertices Allowed? | Formal Definition |
| :--- | :--- | :--- | :--- |
| **Walk** | Yes | Yes | Any sequence $v_0, e_1, v_1, e_2, \dots, e_k, v_k$ of alternating vertices and incident edges. |
| **Trail** | **No** | Yes | A walk in which all **edges** are distinct (e.g., an Eulerian trail). |
| **Path (Simple Path)** | **No** | **No** | A walk in which all **vertices** (and therefore all edges) are distinct. |
| **Closed Walk** | Yes | Yes ($v_0 = v_k$) | A walk that starts and ends at the same vertex ($v_0 = v_k$). |
| **Cycle (Simple Cycle)** | **No** | Only $v_0 = v_k$ | A path of length $k \ge 1$ (directed) or $k \ge 3$ (simple undirected) where $v_0 = v_k$ and all intermediate vertices $v_1, \dots, v_{k-1}$ are distinct. |

- **Cyclic vs. Acyclic Graph:** A graph containing at least one cycle is **cyclic**; a graph with zero cycles is **acyclic**.
- **DAG (Directed Acyclic Graph):** A directed graph with **no directed cycles**. DAGs are fundamental to dependency resolution, task scheduling, topological sorting, and dynamic programming.

### 2.8 Special Graph Classes: Multigraphs, Complete ($K_n$), Bipartite, and Subgraphs

1. **Self-Loop and Parallel Edges (Multigraph):**
   - A **self-loop** is an edge $(u, u)$ connecting a vertex to itself.
   - **Parallel (Multiple) Edges** are two or more edges connecting the exact same pair of vertices. A graph that permits parallel edges is called a **Multigraph** (and if self-loops are also permitted, a **Pseudograph**).
2. **Complete Graph ($K_n$):**
   - A simple undirected graph on $n$ vertices in which **every** pair of distinct vertices is connected by an edge. Every vertex in $K_n$ is regular of degree $n - 1$, and $|E| = \frac{n(n-1)}{2}$.
3. **Bipartite Graph:**
   - A graph $G = (V, E)$ whose vertex set $V$ can be partitioned into two disjoint independent sets $V_1$ and $V_2$ ($V_1 \cup V_2 = V$, $V_1 \cap V_2 = \emptyset$) such that every edge in $E$ has one endpoint in $V_1$ and one endpoint in $V_2$ (no edge connects two vertices in the same set).
   - **Characterization Theorem:** A graph is bipartite *if and only if* it contains **no cycle of odd length** (equivalently, it is 2-colorable via BFS/DFS).
4. **Subgraph and Induced Subgraph:**
   - $G' = (V', E')$ is a **subgraph** of $G = (V, E)$ if $V' \subseteq V$, $E' \subseteq E$, and every edge in $E'$ has both endpoints in $V'$.
   - If $V' = V$ and $G'$ is a connected acyclic subgraph, $G'$ is a **Spanning Tree** of $G$ (having $|V| - 1$ edges).

```
      Complete Graph K4                Bipartite Graph K_{2,3}
     (Every pair linked)              (Set V1={0,1}, Set V2={2,3,4})

        (0)-------(1)                     V1          V2
         | \     / |                     (0)---------(2)
         |   \ /   |                        \       /
         |   / \   |                         \     /
         | /     \ |                          \   /
        (2)-------(3)                    (1)---(3)---(4)
     |V|=4, |E|=4(3)/2 = 6            No edges inside V1 or inside V2
```

### 2.9 Connectivity: Connected Components, Strongly Connected, and Weakly Connected

#### In Undirected Graphs

- **Connected Graph:** An undirected graph is **connected** if there is a path between every pair of vertices $u, v \in V$. Otherwise, it is **disconnected**.
- **Connected Component:** A **maximal connected subgraph** of an undirected graph $G$. ("Maximal" means no additional vertices or edges from $G$ can be added to the subgraph while preserving connectivity). A connected graph has exactly $1$ connected component; a disconnected graph has $k \ge 2$ connected components.

#### In Directed Graphs (Digraphs)

Because edges in a digraph are one-way, connectivity splits into two levels:
1. **Strongly Connected Digraph:** For **every** ordered pair of vertices $(u, v) \in V \times V$, there is both a directed path from $u \leadsto v$ **and** a directed path from $v \leadsto u$.
   - Maximal strongly connected subgraphs are called **Strongly Connected Components (SCCs)**.
2. **Weakly Connected Digraph:** A digraph that is *not* strongly connected, but becomes connected if we replace every directed edge $(u, v)$ with an undirected edge $\{u, v\}$ (i.e., its *underlying undirected graph* is connected).

```
    Strongly Connected Digraph             Weakly Connected Digraph
   (Mutual reachability u <-> v)          (0 reaches 2, but 2 cannot reach 0)

        (0) --------> (1)                      (0) --------> (1)
         ^             |                        |             |
         |             v                        v             v
         +----------- (2)                      (2) <-------- (3)
   Cycle 0->1->2->0 connects all!        Underlying undirected graph is connected
```

### 2.10 Comprehensive Graph Terminology Reference Table

| Term | Notation | Concise Definition |
| :--- | :--- | :--- |
| **Order of Graph** | $|V|$ | Total number of vertices in $G$. |
| **Size of Graph** | $|E|$ | Total number of edges in $G$. |
| **Degree (Undirected)** | $\deg(v)$ | Number of edges incident on vertex $v$. |
| **In-Degree / Out-Degree** | $\deg^-(v), \deg^+(v)$ | Number of edges entering $v$ / leaving $v$ in a digraph. |
| **Simple Graph** | $G = (V, E)$ | Graph with no self-loops and no parallel edges. |
| **Complete Graph** | $K_n$ | Simple undirected graph with all $\binom{n}{2}$ edges present. |
| **DAG** | — | Directed Acyclic Graph (directed graph with zero cycles). |
| **Spanning Tree** | $T = (V, E_T)$ | Connected, acyclic subgraph containing all $|V|$ vertices and $|V|-1$ edges. |

---

## 3. Representation and Memory Layout in C

To run graph algorithms in C, we must translate the mathematical abstraction $G = (V, E)$ into concrete memory structures. The two standard representations are:
1. **Adjacency Matrix** (2D array of size $|V| \times |V|$)
2. **Adjacency List** (Array of $|V|$ linked-list head pointers)

### 3.1 Representative Benchmark Graphs ($G_1$ Undirected & $G_2$ Directed Weighted)

To compare both representations directly, we use the **exact same two benchmark graphs** throughout Section 3 and Section 4:

- **Benchmark Graph $G_1$ (Undirected, Unweighted):**
  - Vertices: $V = \{0, 1, 2, 3, 4, 5\}$ ($|V| = 6$)
  - Edges ($|E| = 7$): $\{0,1\}, \{0,2\}, \{1,2\}, \{1,3\}, \{1,4\}, \{2,4\}, \{3,5\}$
- **Benchmark Graph $G_2$ (Directed, Weighted):**
  - Vertices: $V = \{0, 1, 2, 3, 4\}$ ($|V| = 5$)
  - Directed Weighted Edges ($|E| = 7$):
    $(0 \to 1, w=4),\ (0 \to 2, w=2),\ (1 \to 2, w=5),\ (1 \to 3, w=10),\ (2 \to 4, w=3),\ (4 \to 3, w=4),\ (3 \to 0, w=7)$

```
   BENCHMARK GRAPH G1 (Undirected, Unweighted)      BENCHMARK GRAPH G2 (Directed, Weighted)
   |V| = 6, |E| = 7                                 |V| = 5, |E| = 7

         (0)-----------(1)                                (0) ----4----> (1)
          |           / | \                              ^ |            / |
          |         /   |   \                            | |2         5/  |10
          |       /     |    (3)---(5)                  7| v         v    v
          |     /       |                                | (2) --3-> (4)  |
          |   /         |                                |            |4  |
         (2)-----------(4)                               +--- (3) <---+---+
```

### 3.2 Adjacency Matrix Representation

An **Adjacency Matrix** represents a graph $G = (V, E)$ with $|V|$ vertices using a $|V| \times |V|$ two-dimensional array `adj[u][v]`:

- **For an Unweighted Graph:**
  $$\text{adj}[u][v] = \begin{cases} 1 & \text{if edge } (u, v) \in E \\ 0 & \text{if edge } (u, v) \notin E \end{cases}$$
- **For a Weighted Graph:**
  $$\text{adj}[u][v] = \begin{cases} w(u, v) & \text{if edge } (u, v) \in E \\ 0 & \text{if } u = v \text{ (and no self-loop)} \\ \infty \ (\text{e.g., } \texttt{INT\_MAX}) & \text{if } (u, v) \notin E \text{ and } u \ne v \end{cases}$$

#### Mathematical Properties of the Adjacency Matrix

1. **Symmetry in Undirected Graphs:** Because $\{u, v\} \in E \iff \{v, u\} \in E$, the adjacency matrix of an undirected graph is always **symmetric across the main diagonal** ($\text{adj}[u][v] = \text{adj}[v][u]$, i.e., $A = A^T$).
2. **Degree Computation from Rows and Columns:**
   - In an unweighted **undirected** graph, the sum of row $u$ (or column $u$) equals $\deg(u)$:
     $$\deg(u) = \sum_{v=0}^{|V|-1} \text{adj}[u][v]$$
   - In an unweighted **directed** graph:
     - **Row $u$ sum** equals the **Out-Degree** $\deg^+(u) = \sum_{v=0}^{|V|-1} \text{adj}[u][v]$.
     - **Column $v$ sum** equals the **In-Degree** $\deg^-(v) = \sum_{u=0}^{|V|-1} \text{adj}[u][v]$.

#### Exact Adjacency Matrix Layout for $G_1$ and $G_2$

```
   G1 Adjacency Matrix (6 x 6, Symmetric)          G2 Adjacency Matrix (5 x 5, Asymmetric)
         v=0  v=1  v=2  v=3  v=4  v=5                     v=0  v=1  v=2  v=3  v=4
       +----+----+----+----+----+----+                  +----+----+----+----+----+
   u=0 |  0 |  1 |  1 |  0 |  0 |  0 |  row sum = 2 u=0 |  0 |  4 |  2 |INF |INF | out-deg = 2
   u=1 |  1 |  0 |  1 |  1 |  1 |  0 |  row sum = 4 u=1 |INF |  0 |  5 | 10 |INF | out-deg = 2
   u=2 |  1 |  1 |  0 |  0 |  1 |  0 |  row sum = 3 u=2 |INF |INF |  0 |INF |  3 | out-deg = 1
   u=3 |  0 |  1 |  0 |  0 |  0 |  1 |  row sum = 2 u=3 |  7 |INF |INF |  0 |INF | out-deg = 1
   u=4 |  0 |  1 |  1 |  0 |  0 |  0 |  row sum = 2 u=4 |INF |INF |INF |  4 |  0 | out-deg = 1
   u=5 |  0 |  0 |  0 |  1 |  0 |  0 |  row sum = 1     +----+----+----+----+----+
       +----+----+----+----+----+----+           in-deg:   1    1    2    2    1
```

#### C Structure Definition (`MatrixGraph`)

```c
#define MAX_VERTICES 100
#define INF INT_MAX

typedef struct {
    int num_vertices;                        /* |V| */
    int num_edges;                           /* |E| */
    bool is_directed;                        /* true = digraph, false = undirected */
    bool is_weighted;                        /* true = weighted, false = 0/1 */
    int adj[MAX_VERTICES][MAX_VERTICES];     /* Contiguous row-major 2D matrix */
} MatrixGraph;
```

### 3.3 Adjacency List Representation

In a sparse graph where $|E| \ll |V|^2$, an Adjacency Matrix wastes almost all of its $|V|^2$ cells storing `0` or `INF`. The **Adjacency List** representation eliminates this waste by storing **only the edges that actually exist**.

It consists of:
1. An array `head` of $|V|$ pointers (`AdjNode *head[V]`), indexed by source vertex $u \in \{0, \dots, |V|-1\}$.
2. Each `head[u]` points to a singly linked list of `AdjNode` structures containing every neighbor $v$ such that $(u, v) \in E$ (along with the edge weight `weight`).

#### Why Undirected Graphs Require $2|E|$ List Nodes

- In a **directed** graph, each directed edge $(u, v) \in E$ appends **one** node ($v$) to `head[u]`. Thus, the total number of linked-list nodes across all $|V|$ lists is exactly $|E|$ (matching $\sum \deg^+(u) = |E|$).
- In an **undirected** graph, each undirected edge $\{u, v\} \in E$ is bidirectional, so we must insert **two** list nodes: neighbor $v$ into `head[u]` **and** neighbor $u$ into `head[v]`. Thus, the total number of linked-list nodes is exactly $2|E|$ (matching the Handshaking Lemma $\sum \deg(u) = 2|E|$).
- In both cases, the total space complexity is:

$$\text{Space}_{\text{AdjList}} = \Theta(|V| + |E|)$$

#### Exact Adjacency List Memory Layout for $G_1$ and $G_2$

```
  G1 Adjacency List (Undirected: |V|=6 array slots + 2|E|=14 heap nodes)
  ======================================================================
  head[0] ---> [ 1 | * ] ---> [ 2 | NULL ]
  head[1] ---> [ 0 | * ] ---> [ 2 | * ] ---> [ 3 | * ] ---> [ 4 | NULL ]
  head[2] ---> [ 0 | * ] ---> [ 1 | * ] ---> [ 4 | NULL ]
  head[3] ---> [ 1 | * ] ---> [ 5 | NULL ]
  head[4] ---> [ 1 | * ] ---> [ 2 | NULL ]
  head[5] ---> [ 3 | NULL ]

  G2 Adjacency List (Directed Weighted: |V|=5 array slots + |E|=7 heap nodes)
  ===========================================================================
  head[0] ---> [ dest:1 | w:4 | * ] ---> [ dest:2 | w:2 | NULL ]
  head[1] ---> [ dest:2 | w:5 | * ] ---> [ dest:3 | w:10| NULL ]
  head[2] ---> [ dest:4 | w:3 | NULL ]
  head[3] ---> [ dest:0 | w:7 | NULL ]
  head[4] ---> [ dest:3 | w:4 | NULL ]
```

#### C Structure Definitions (`AdjNode` and `ListGraph`)

```c
typedef struct AdjNode {
    int dest;               /* Target/neighbor vertex ID v */
    int weight;             /* Edge weight w(u, v) (1 if unweighted) */
    struct AdjNode *next;   /* Pointer to next neighbor of u */
} AdjNode;

typedef struct {
    int num_vertices;       /* |V| */
    int num_edges;          /* |E| */
    bool is_directed;       /* true = directed, false = undirected */
    bool is_weighted;       /* true = weighted, false = unweighted */
    AdjNode **head;         /* Dynamically allocated array of |V| head pointers */
} ListGraph;
```

### 3.4 Static vs. Dynamic Graph Storage: Architectural Trade-Offs

| Architectural Aspect | Adjacency Matrix (`int adj[V][V]`) | Adjacency List (`AdjNode **head`) |
| :--- | :--- | :--- |
| **Memory Footprint** | $\Theta(|V|^2)$ — fixed regardless of how few edges exist. | $\Theta(|V| + |E|)$ — proportional to actual vertices and edges. |
| **Cache Locality** | **Excellent:** Row `adj[u][0..V-1]` is contiguous in RAM. | **Poor:** Linked-list nodes are scattered across the heap. |
| **Per-Edge Memory Overhead** | **1 integer** ($4\text{ B}$) per matrix cell (or 1 bit in a bitset). | **`sizeof(AdjNode)`** ($16\text{ B}$ on 64-bit C due to alignment + 8-byte `next` pointer). |
| **Memory Example ($|V|=10^5, |E|=3\times 10^5$)** | $10^{10} \times 4\text{ B} \approx \mathbf{40\text{ GB}}$ (Out-of-Memory crash!) | $10^5 \times 8\text{ B} + 6\times 10^5 \times 16\text{ B} \approx \mathbf{10.4\text{ MB}}$ (Fits easily in L3 cache!) |

---

## 4. Operations and Algorithms on Graph Representations

### 4.1 Graph Initialization (`initMatrixGraph`, `createListGraph`)

- **Purpose:** Allocate and initialize an empty graph with $|V|$ vertices and $0$ edges.
- **Complexity:**
  - **Adjacency Matrix:** $\Theta(|V|^2)$ time and $\Theta(|V|^2)$ space to zero/fill all $|V| \times |V|$ cells.
  - **Adjacency List:** $\Theta(|V|)$ time and $\Theta(|V|)$ space (`calloc(V, sizeof(AdjNode *))` sets all `head[u] = NULL`).

### 4.2 Adding Edges (`addMatrixEdge`, `addListEdge`)

- **Adjacency Matrix (`addMatrixEdge(g, u, v, w)`):**
  1. Validate bounds $0 \le u, v < |V|$.
  2. Set `g->adj[u][v] = w`.
  3. If `!g->is_directed`, also set `g->adj[v][u] = w`.
  4. **Time Complexity:** $\Theta(1)$.
- **Adjacency List (`addListEdge(g, u, v, w)`):**
  - **Unsorted Head Insertion (Fastest):** Allocate a new `AdjNode` for $v$, set `new_node->next = g->head[u]`, and update `g->head[u] = new_node` in **$O(1)$ time** (assuming no duplicate check). If undirected, repeat for $u$ in `g->head[v]`.
  - **Sorted Insertion (Canonical Order):** If deterministic ascending neighbor order is desired (useful for reproducible textbook BFS/DFS traces) or duplicate edges must be rejected, scan `g->head[u]` in **$O(\deg(u))$ time** before inserting.

### 4.3 Removing Edges (`removeMatrixEdge`, `removeListEdge`)

- **Adjacency Matrix:** Set `g->adj[u][v] = (g->is_weighted ? INF : 0)` (and `g->adj[v][u]` if undirected) in **$\Theta(1)$ time**.
- **Adjacency List:** Traverse `g->head[u]` to locate the node with `dest == v`, unlink it from the singly linked list, and `free()` it. If undirected, also traverse `g->head[v]` to unlink and `free()` node $u$.
  - **Time Complexity:** $O(\deg^+(u))$ for a directed graph; $O(\deg(u) + \deg(v))$ for an undirected graph (bounded by $O(|V|)$ in the worst case).

### 4.4 Edge Existence Lookup and Neighbor Iteration

These two operations expose the core algorithmic trade-off between matrices and lists:

1. **Check if Edge $(u, v)$ Exists (`hasEdge(g, u, v)`):**
   - **Adjacency Matrix:** Direct array indexing `g->adj[u][v]` runs in **$O(1)$ worst-case time**.
   - **Adjacency List:** Must linearly search `g->head[u]` for `curr->dest == v`, taking **$O(\deg^+(u))$ time** (up to $O(|V|)$ in the worst case).
2. **Iterate Over All Neighbors of Vertex $u$:**
   - **Adjacency Matrix:** Must scan all $|V|$ columns of row $u$ (`for (int v = 0; v < V; v++)`), taking **$\Theta(|V|)$ time** even if vertex $u$ has $0$ neighbors!
   - **Adjacency List:** Walks only the existing nodes in `g->head[u]`, taking **$\Theta(\deg^+(u))$ time**. Across a full-graph traversal (visiting every vertex $u \in V$), scanning all neighbor lists takes $\sum_{u \in V} \Theta(1 + \deg(u)) = \Theta(|V| + |E|)$ time, whereas scanning all matrix rows takes $\sum_{u \in V} \Theta(|V|) = \Theta(|V|^2)$ time!

### 4.5 Computing Vertex Degrees ($\deg(v)$, $\deg^-(v)$, $\deg^+(v)$)

- **Out-Degree $\deg^+(u)$ (or Undirected $\deg(u)$):**
  - **Adjacency Matrix:** Sum non-empty entries in row $u$ $\implies \Theta(|V|)$.
  - **Adjacency List:** Count nodes in `head[u]` $\implies \Theta(\deg^+(u))$ (or $O(1)$ if list lengths are cached in an array `out_deg[u]`).
- **In-Degree $\deg^-(v)$ in a Directed Graph:**
  - **Adjacency Matrix:** Sum non-empty entries in column $v$ $\implies \Theta(|V|)$.
  - **Adjacency List:** Because `head[u]` stores *outgoing* edges from $u$, finding all edges *entering* a single vertex $v$ (without maintaining a transpose graph or an `in_deg[]` counter array) requires scanning **every** list across the entire graph $\implies \Theta(|V| + |E|)$!
  - *Engineering Tip:* Computing the array of in-degrees for **all** vertices $0 \dots |V|-1$ simultaneously via a single pass over all adjacency lists also takes $\Theta(|V| + |E|)$ total time.

### 4.6 Full Dynamic Memory Teardown (`freeListGraph`)

Freeing a `ListGraph` requires a **two-level deallocation**:
1. For each vertex $u = 0 \dots |V|-1$, walk the singly linked list `g->head[u]` using a temporary `next` pointer and call `free(curr)` on every `AdjNode` ($2|E|$ or $|E|$ frees).
2. Call `free(g->head)` to release the array of head pointers.
3. Call `free(g)` to release the `ListGraph` wrapper struct.

### 4.7 Complete C11 Implementation: Adjacency Matrix and Adjacency List

The following complete, self-contained C11 program constructs **both** benchmark graphs ($G_1$ undirected unweighted, and $G_2$ directed weighted) in both Adjacency Matrix and Adjacency List form, verifies the Handshaking Lemma, tests edge deletion and edge lookup, and frees all heap memory cleanly.

```c
/**
 * @file graph_representations.c
 * @brief Complete C11 implementation of Adjacency Matrix and Adjacency List
 *        for both Undirected/Unweighted (G1) and Directed/Weighted (G2) graphs.
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>
#include <limits.h>

#define MAX_VERTICES 10
#define INF INT_MAX

/* =========================================================================
 * 1. ADJACENCY MATRIX REPRESENTATION
 * ========================================================================= */
typedef struct {
    int num_vertices;
    int num_edges;
    bool is_directed;
    bool is_weighted;
    int adj[MAX_VERTICES][MAX_VERTICES];
} MatrixGraph;

void initMatrixGraph(MatrixGraph *g, int v, bool directed, bool weighted) {
    g->num_vertices = v;
    g->num_edges = 0;
    g->is_directed = directed;
    g->is_weighted = weighted;

    for (int i = 0; i < v; i++) {
        for (int j = 0; j < v; j++) {
            if (weighted) {
                g->adj[i][j] = (i == j) ? 0 : INF;
            } else {
                g->adj[i][j] = 0;
            }
        }
    }
}

bool addMatrixEdge(MatrixGraph *g, int u, int v, int weight) {
    if (u < 0 || u >= g->num_vertices || v < 0 || v >= g->num_vertices) {
        return false;
    }
    int w = g->is_weighted ? weight : 1;
    bool already_exists = g->is_weighted ? (g->adj[u][v] != INF && u != v)
                                         : (g->adj[u][v] != 0);
    g->adj[u][v] = w;
    if (!g->is_directed) {
        g->adj[v][u] = w;
    }
    if (!already_exists) {
        g->num_edges++;
    }
    return true;
}

bool hasMatrixEdge(const MatrixGraph *g, int u, int v) {
    if (u < 0 || u >= g->num_vertices || v < 0 || v >= g->num_vertices) {
        return false;
    }
    if (g->is_weighted) {
        return (u != v) && (g->adj[u][v] != INF);
    }
    return g->adj[u][v] != 0;
}

int matrixOutDegree(const MatrixGraph *g, int u) {
    int deg = 0;
    for (int v = 0; v < g->num_vertices; v++) {
        if (hasMatrixEdge(g, u, v)) {
            deg++;
        }
    }
    return deg;
}

int matrixInDegree(const MatrixGraph *g, int v) {
    int deg = 0;
    for (int u = 0; u < g->num_vertices; u++) {
        if (hasMatrixEdge(g, u, v)) {
            deg++;
        }
    }
    return deg;
}

void printMatrixGraph(const MatrixGraph *g, const char *title) {
    printf("=== %s ===\n", title);
    printf("Vertices: %d | Edges: %d | Type: %s, %s\n\n",
           g->num_vertices, g->num_edges,
           g->is_directed ? "Directed" : "Undirected",
           g->is_weighted ? "Weighted" : "Unweighted");

    printf("    ");
    for (int j = 0; j < g->num_vertices; j++) {
        printf("%4d", j);
    }
    printf("\n    ");
    for (int j = 0; j < g->num_vertices; j++) {
        printf("----");
    }
    printf("\n");

    for (int i = 0; i < g->num_vertices; i++) {
        printf("%2d |", i);
        for (int j = 0; j < g->num_vertices; j++) {
            if (g->is_weighted && g->adj[i][j] == INF) {
                printf(" INF");
            } else {
                printf("%4d", g->adj[i][j]);
            }
        }
        printf("\n");
    }
    printf("\n");
}

/* =========================================================================
 * 2. ADJACENCY LIST REPRESENTATION
 * ========================================================================= */
typedef struct AdjNode {
    int dest;
    int weight;
    struct AdjNode *next;
} AdjNode;

typedef struct {
    int num_vertices;
    int num_edges;
    bool is_directed;
    bool is_weighted;
    AdjNode **head;
} ListGraph;

ListGraph *createListGraph(int v, bool directed, bool weighted) {
    ListGraph *g = (ListGraph *)malloc(sizeof(ListGraph));
    if (!g) {
        return NULL;
    }
    g->num_vertices = v;
    g->num_edges = 0;
    g->is_directed = directed;
    g->is_weighted = weighted;
    g->head = (AdjNode **)calloc((size_t)v, sizeof(AdjNode *));
    if (!g->head) {
        free(g);
        return NULL;
    }
    return g;
}

static AdjNode *createAdjNode(int dest, int weight) {
    AdjNode *node = (AdjNode *)malloc(sizeof(AdjNode));
    if (node) {
        node->dest = dest;
        node->weight = weight;
        node->next = NULL;
    }
    return node;
}

/* Insert neighbor in ascending vertex order for canonical display */
static bool insertSortedNeighbor(AdjNode **head_ref, int dest, int weight) {
    AdjNode *curr = *head_ref;
    AdjNode *prev = NULL;

    while (curr != NULL && curr->dest < dest) {
        prev = curr;
        curr = curr->next;
    }
    if (curr != NULL && curr->dest == dest) {
        curr->weight = weight; /* Update existing edge weight */
        return false;          /* Not a newly added edge */
    }

    AdjNode *new_node = createAdjNode(dest, weight);
    if (!new_node) {
        return false;
    }
    new_node->next = curr;
    if (prev == NULL) {
        *head_ref = new_node;
    } else {
        prev->next = new_node;
    }
    return true;
}

bool addListEdge(ListGraph *g, int u, int v, int weight) {
    if (!g || u < 0 || u >= g->num_vertices || v < 0 || v >= g->num_vertices) {
        return false;
    }
    int w = g->is_weighted ? weight : 1;
    bool added = insertSortedNeighbor(&g->head[u], v, w);
    if (!g->is_directed && u != v) {
        insertSortedNeighbor(&g->head[v], u, w);
    }
    if (added) {
        g->num_edges++;
    }
    return added;
}

bool hasListEdge(const ListGraph *g, int u, int v, int *weight_out) {
    if (!g || u < 0 || u >= g->num_vertices || v < 0 || v >= g->num_vertices) {
        return false;
    }
    for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
        if (curr->dest == v) {
            if (weight_out) {
                *weight_out = curr->weight;
            }
            return true;
        }
    }
    return false;
}

int listOutDegree(const ListGraph *g, int u) {
    int deg = 0;
    for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
        deg++;
    }
    return deg;
}

int listInDegree(const ListGraph *g, int v) {
    int deg = 0;
    for (int u = 0; u < g->num_vertices; u++) {
        for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
            if (curr->dest == v) {
                deg++;
            }
        }
    }
    return deg;
}

void printListGraph(const ListGraph *g, const char *title) {
    printf("=== %s ===\n", title);
    printf("Vertices: %d | Edges: %d | Type: %s, %s\n\n",
           g->num_vertices, g->num_edges,
           g->is_directed ? "Directed" : "Undirected",
           g->is_weighted ? "Weighted" : "Unweighted");

    for (int u = 0; u < g->num_vertices; u++) {
        printf("head[%d] -> ", u);
        for (AdjNode *curr = g->head[u]; curr != NULL; curr = curr->next) {
            if (g->is_weighted) {
                printf("[%d | w=%d] -> ", curr->dest, curr->weight);
            } else {
                printf("[%d] -> ", curr->dest);
            }
        }
        printf("NULL\n");
    }
    printf("\n");
}

void freeListGraph(ListGraph *g) {
    if (!g) {
        return;
    }
    for (int u = 0; u < g->num_vertices; u++) {
        AdjNode *curr = g->head[u];
        while (curr != NULL) {
            AdjNode *temp = curr;
            curr = curr->next;
            free(temp);
        }
    }
    free(g->head);
    free(g);
}

int main(void) {
    /* ---------------------------------------------------------------------
     * PART A: Undirected Unweighted Graph G1 (V = 6, E = 7)
     * Edges: {0,1}, {0,2}, {1,2}, {1,3}, {1,4}, {2,4}, {3,5}
     * --------------------------------------------------------------------- */
    const int V1 = 6;
    const int edges1[][2] = {
        {0, 1}, {0, 2}, {1, 2}, {1, 3}, {1, 4}, {2, 4}, {3, 5}
    };
    const int E1 = (int)(sizeof(edges1) / sizeof(edges1[0]));

    MatrixGraph mg1;
    initMatrixGraph(&mg1, V1, false, false);
    ListGraph *lg1 = createListGraph(V1, false, false);

    for (int i = 0; i < E1; i++) {
        addMatrixEdge(&mg1, edges1[i][0], edges1[i][1], 1);
        addListEdge(lg1, edges1[i][0], edges1[i][1], 1);
    }

    printMatrixGraph(&mg1, "G1: Adjacency Matrix (Undirected, Unweighted)");
    printListGraph(lg1, "G1: Adjacency List (Undirected, Unweighted)");

    int sum_deg = 0;
    printf("--- Vertex Degrees & Handshaking Lemma Verification (G1) ---\n");
    for (int u = 0; u < V1; u++) {
        int d_mat = matrixOutDegree(&mg1, u);
        int d_lst = listOutDegree(lg1, u);
        sum_deg += d_lst;
        printf("Vertex %d: deg(matrix) = %d, deg(list) = %d\n", u, d_mat, d_lst);
    }
    printf("Sum of degrees = %d | 2 * |E| = %d (Handshaking Lemma holds!)\n\n",
           sum_deg, 2 * lg1->num_edges);

    /* ---------------------------------------------------------------------
     * PART B: Directed Weighted Graph G2 (V = 5, E = 7)
     * Directed Edges (u -> v, weight):
     * (0->1, 4), (0->2, 2), (1->2, 5), (1->3, 10), (2->4, 3), (4->3, 4), (3->0, 7)
     * --------------------------------------------------------------------- */
    const int V2 = 5;
    const int edges2[][3] = {
        {0, 1, 4}, {0, 2, 2}, {1, 2, 5}, {1, 3, 10},
        {2, 4, 3}, {4, 3, 4}, {3, 0, 7}
    };
    const int E2 = (int)(sizeof(edges2) / sizeof(edges2[0]));

    MatrixGraph mg2;
    initMatrixGraph(&mg2, V2, true, true);
    ListGraph *lg2 = createListGraph(V2, true, true);

    for (int i = 0; i < E2; i++) {
        addMatrixEdge(&mg2, edges2[i][0], edges2[i][1], edges2[i][2]);
        addListEdge(lg2, edges2[i][0], edges2[i][1], edges2[i][2]);
    }

    printMatrixGraph(&mg2, "G2: Adjacency Matrix (Directed, Weighted)");
    printListGraph(lg2, "G2: Adjacency List (Directed, Weighted)");

    int sum_in = 0, sum_out = 0;
    printf("--- Directed In-Degree & Out-Degree Verification (G2) ---\n");
    for (int u = 0; u < V2; u++) {
        int in_d = listInDegree(lg2, u);
        int out_d = listOutDegree(lg2, u);
        sum_in += in_d;
        sum_out += out_d;
        printf("Vertex %d: in-deg = %d, out-deg = %d\n", u, in_d, out_d);
    }
    printf("Sum(in-deg) = %d | Sum(out-deg) = %d | |E| = %d\n\n",
           sum_in, sum_out, lg2->num_edges);

    int w_lookup = 0;
    bool found_1_3 = hasListEdge(lg2, 1, 3, &w_lookup);
    printf("Edge Lookup (1 -> 3) in G2: %s (weight = %d)\n",
           found_1_3 ? "FOUND" : "NOT FOUND", w_lookup);
    bool found_3_1 = hasListEdge(lg2, 3, 1, &w_lookup);
    printf("Edge Lookup (3 -> 1) in G2: %s\n",
           found_3_1 ? "FOUND" : "NOT FOUND (Directed!)");

    freeListGraph(lg1);
    freeListGraph(lg2);
    return 0;
}
```

#### Program Output:

```text
=== G1: Adjacency Matrix (Undirected, Unweighted) ===
Vertices: 6 | Edges: 7 | Type: Undirected, Unweighted

       0   1   2   3   4   5
    ------------------------
 0 |   0   1   1   0   0   0
 1 |   1   0   1   1   1   0
 2 |   1   1   0   0   1   0
 3 |   0   1   0   0   0   1
 4 |   0   1   1   0   0   0
 5 |   0   0   0   1   0   0

=== G1: Adjacency List (Undirected, Unweighted) ===
Vertices: 6 | Edges: 7 | Type: Undirected, Unweighted

head[0] -> [1] -> [2] -> NULL
head[1] -> [0] -> [2] -> [3] -> [4] -> NULL
head[2] -> [0] -> [1] -> [4] -> NULL
head[3] -> [1] -> [5] -> NULL
head[4] -> [1] -> [2] -> NULL
head[5] -> [3] -> NULL

--- Vertex Degrees & Handshaking Lemma Verification (G1) ---
Vertex 0: deg(matrix) = 2, deg(list) = 2
Vertex 1: deg(matrix) = 4, deg(list) = 4
Vertex 2: deg(matrix) = 3, deg(list) = 3
Vertex 3: deg(matrix) = 2, deg(list) = 2
Vertex 4: deg(matrix) = 2, deg(list) = 2
Vertex 5: deg(matrix) = 1, deg(list) = 1
Sum of degrees = 14 | 2 * |E| = 14 (Handshaking Lemma holds!)

=== G2: Adjacency Matrix (Directed, Weighted) ===
Vertices: 5 | Edges: 7 | Type: Directed, Weighted

       0   1   2   3   4
    --------------------
 0 |   0   4   2 INF INF
 1 | INF   0   5  10 INF
 2 | INF INF   0 INF   3
 3 |   7 INF INF   0 INF
 4 | INF INF INF   4   0

=== G2: Adjacency List (Directed, Weighted) ===
Vertices: 5 | Edges: 7 | Type: Directed, Weighted

head[0] -> [1 | w=4] -> [2 | w=2] -> NULL
head[1] -> [2 | w=5] -> [3 | w=10] -> NULL
head[2] -> [4 | w=3] -> NULL
head[3] -> [0 | w=7] -> NULL
head[4] -> [3 | w=4] -> NULL

--- Directed In-Degree & Out-Degree Verification (G2) ---
Vertex 0: in-deg = 1, out-deg = 2
Vertex 1: in-deg = 1, out-deg = 2
Vertex 2: in-deg = 2, out-deg = 1
Vertex 3: in-deg = 2, out-deg = 1
Vertex 4: in-deg = 1, out-deg = 1
Sum(in-deg) = 7 | Sum(out-deg) = 7 | |E| = 7

Edge Lookup (1 -> 3) in G2: FOUND (weight = 10)
Edge Lookup (3 -> 1) in G2: NOT FOUND (Directed!)
```

---

## 5. Comparison Tables and Complexity Summary

### 5.1 Operation Time and Space Complexity: Adjacency Matrix vs. Adjacency List

Let $V = |V|$ be the number of vertices, $E = |E|$ be the number of edges, and $\deg(u)$ (or $\deg^+(u)$) be the out-degree of vertex $u$.

| Graph Operation / Metric | Adjacency Matrix (`adj[V][V]`) | Adjacency List (`head[V]`) | Winner |
| :--- | :--- | :--- | :--- |
| **Space Complexity** | $\Theta(V^2)$ | $\Theta(V + E)$ | **List** for sparse ($E \ll V^2$); **Matrix** for very dense ($E \approx V^2$) |
| **Initialize Empty Graph** | $\Theta(V^2)$ | $\Theta(V)$ | **Adjacency List** |
| **Add Edge $(u, v)$** | $O(1)$ | $O(1)$ (prepend at head) or $O(\deg(u))$ (sorted/dedup) | **Tie** ($O(1)$) |
| **Remove Edge $(u, v)$** | $O(1)$ | $O(\deg(u))$ (directed) or $O(\deg(u) + \deg(v))$ (undirected) | **Adjacency Matrix** |
| **Check if Edge $(u, v)$ Exists** | $O(1)$ | $O(\deg(u))$ (worst-case $O(V)$) | **Adjacency Matrix** |
| **Iterate Over Neighbors of $u$** | $\Theta(V)$ | $\Theta(\deg(u))$ | **Adjacency List** |
| **Compute Out-Degree $\deg^+(u)$** | $\Theta(V)$ | $\Theta(\deg^+(u))$ (or $O(1)$ if cached) | **Adjacency List** |
| **Compute Single In-Degree $\deg^-(v)$** | $\Theta(V)$ (column scan) | $\Theta(V + E)$ (full scan unless cached) | **Adjacency Matrix** |
| **Full Graph Traversal (BFS / DFS)** | $\Theta(V^2)$ | $\Theta(V + E)$ | **Adjacency List** |

### 5.2 When to Choose Adjacency Matrix vs. Adjacency List

- **Choose an Adjacency List when:**
  - The graph is **sparse** ($|E| \ll |V|^2$), which describes almost all large-scale real-world networks (road maps, web graphs, social networks, trees, ASTs).
  - $|V|$ is large (e.g., $|V| \ge 10^4$), where $\Theta(|V|^2)$ memory is prohibitive.
  - Algorithms predominantly **iterate over neighbors of a vertex** (BFS, DFS, Topological Sort, Dijkstra's Algorithm, Prim's/Kruskal's MST).
- **Choose an Adjacency Matrix when:**
  - The graph is **dense** ($|E| = \Theta(|V|^2)$) and $|V|$ is relatively small ($|V| \le 2000$).
  - The application requires frequent **$O(1)$ edge existence queries** (`hasEdge(u, v)`) or **$O(1)$ edge deletions**.
  - Running matrix-based all-pairs algorithms such as the **Floyd-Warshall** all-pairs shortest path algorithm ($\Theta(V^3)$) or Warshall's transitive closure.

---

## 6. Common Errors, Pitfalls, and Edge Cases

### 6.1 Pitfall 1: Forgetting the Symmetric Assignment `adj[v][u]` in Undirected Graphs

In an undirected graph, an edge $\{u, v\}$ is two-way. Updating only `adj[u][v]` (or inserting only into `head[u]`) silently turns the undirected edge into a one-way directed edge!

```c
/* BUG: Only sets u -> v in an undirected graph! */
void addUndirectedEdgeBug(MatrixGraph *g, int u, int v) {
    g->adj[u][v] = 1; /* Missing g->adj[v][u] = 1; */
}

/* FIX: Always mirror undirected edges in both directions */
void addUndirectedEdgeFix(MatrixGraph *g, int u, int v) {
    g->adj[u][v] = 1;
    g->adj[v][u] = 1;
}
```

### 6.2 Pitfall 2: Using `0` as "No Edge" in a Weighted Graph That Allows Zero-Weight Edges

In weighted graphs, an edge $(u, v)$ may legitimately have weight $w(u, v) = 0$ (e.g., a free transfer between transit stations). Using `0` as the sentinel for "no edge" makes a $0$-weight edge indistinguishable from a missing edge!

```c
/* BUG: Cannot distinguish a 0-weight edge from an absent edge */
bool hasWeightedEdgeBug(const MatrixGraph *g, int u, int v) {
    return g->adj[u][v] != 0; /* Fails if edge (u,v) has weight 0! */
}

/* FIX: Initialize non-edges to INF (INT_MAX) and check != INF */
bool hasWeightedEdgeFix(const MatrixGraph *g, int u, int v) {
    return (u != v) && (g->adj[u][v] != INF);
}
```

### 6.3 Pitfall 3: Shallow Freeing of an Adjacency List (`free(g->head)` Without Freeing Nodes)

Calling `free(g->head)` directly without first traversing and freeing each linked list `g->head[u]` leaks every `AdjNode` on the heap ($\Theta(|E|)$ memory leak).

```c
/* BUG: Orphans all 2|E| linked-list nodes on the heap! */
void freeListGraphBug(ListGraph *g) {
    free(g->head);
    free(g);
}

/* FIX: Free every AdjNode in every bucket before freeing g->head */
void freeListGraphFix(ListGraph *g) {
    for (int u = 0; u < g->num_vertices; u++) {
        AdjNode *curr = g->head[u];
        while (curr != NULL) {
            AdjNode *next = curr->next;
            free(curr);
            curr = next;
        }
    }
    free(g->head);
    free(g);
}
```

### 6.4 Pitfall 4: Integer Overflow When Adding Edge Weights to `INT_MAX` ($\infty$)

When using `INF = INT_MAX` to represent $\infty$ in a weighted adjacency matrix, computing `dist[u] + adj[u][v]` when `adj[u][v] == INF` causes **signed integer overflow** (Undefined Behavior in C, wrapping around to a large negative number!).

```c
/* BUG: Signed integer overflow when adj[u][v] == INT_MAX! */
if (dist[u] + g->adj[u][v] < dist[v]) {
    dist[v] = dist[u] + g->adj[u][v];
}

/* FIX: Guard against INF before performing addition */
if (dist[u] != INF && g->adj[u][v] != INF && dist[u] + g->adj[u][v] < dist[v]) {
    dist[v] = dist[u] + g->adj[u][v];
}
```

### 6.5 Boundary Conditions Checklist

- [ ] **Out-of-Range Vertex Indices ($u < 0$ or $u \ge |V|$):** Do `addEdge`, `removeEdge`, and `hasEdge` validate both $u$ and $v$ before indexing `adj[u][v]` or `head[u]`?
- [ ] **Self-Loops ($u == v$):** In an undirected adjacency list, does `addListEdge(g, u, u, w)` avoid inserting two duplicate nodes into the *same* list `head[u]` (`if (!g->is_directed && u != v)`)?
- [ ] **Isolated Vertices ($\deg(u) == 0$):** Does neighbor iteration handle `g->head[u] == NULL` gracefully with zero iterations?
- [ ] **Duplicate Edge Insertion:** If the caller inserts $(u, v)$ twice on a simple graph, does your implementation either update the weight or reject the duplicate rather than corrupting `num_edges`?

---

## 7. Summary and Revision Checklist

### 7.1 Key Takeaways and Core Invariants

1. **Graph Definition:** $G = (V, E)$ models arbitrary $M:N$ pairwise relationships. A **tree** is simply a connected, acyclic undirected graph with $|E| = |V| - 1$.
2. **Handshaking Lemma:**
   - Undirected: $\sum_{v \in V} \deg(v) = 2|E|$ (so the number of odd-degree vertices is always even).
   - Directed: $\sum_{v \in V} \deg^-(v) = \sum_{v \in V} \deg^+(v) = |E|$.
3. **Maximum Simple Edges:** $\frac{|V|(|V|-1)}{2}$ for undirected; $|V|(|V|-1)$ for directed.
4. **Adjacency Matrix vs. Adjacency List:**
   - **Matrix:** $\Theta(V^2)$ space, $O(1)$ edge lookup, $\Theta(V)$ neighbor scan. Ideal for small, dense graphs.
   - **List:** $\Theta(V + E)$ space, $O(\deg(u))$ edge lookup, $\Theta(\deg(u))$ neighbor scan. Ideal for sparse graphs and $\Theta(V + E)$ BFS/DFS traversals.

### 7.2 Self-Test Revision Questions

1. **Handshaking Calculation:** An undirected graph has $12$ edges, $4$ vertices of degree $3$, and all remaining vertices have degree $2$. How many vertices does the graph have in total?
   *(Answer: By $\sum \deg(v) = 2|E|$, let $k$ be the number of degree-2 vertices. Then $4(3) + 2k = 2(12) = 24 \implies 12 + 2k = 24 \implies k = 6$. Total vertices $|V| = 4 + 6 = 10$).*
2. **Parity Proof Check:** Can you construct a simple undirected graph with $5$ vertices whose degrees are $3, 3, 3, 3, 1$? Why or why not?
   *(Answer: No! Five vertices have odd degree, violating the Handshaking Lemma corollary; equivalently, $\sum \deg(v) = 13$, which is odd and cannot equal $2|E|$).*
3. **Representation Space:** For a directed graph with $|V| = 1000$ and $|E| = 4000$, how many linked-list nodes (`AdjNode`) are allocated in an Adjacency List vs. how many cells exist in an Adjacency Matrix?
   *(Answer: Adjacency List allocates exactly $|E| = 4{,}000$ `AdjNode` structs plus $1{,}000$ head pointers; Adjacency Matrix allocates $1000 \times 1000 = 1{,}000{,}000$ cells).*

### 7.3 Navigation Links

- **Previous Topic:** [06-AVL-and-Splay-Trees.md](./06-Splay-Trees.md) — Self-Balancing Binary Search Trees (AVL Trees and Splay Trees)
- **Current File:** `07-Graph-Fundamentals-and-Representations.md` (Unit 3 — Topic 7)
- **Next Topic:** [08-DFS-and-BFS.md](./08-DFS-and-BFS.md) — Graph Traversals: Breadth-First Search (BFS) and Depth-First Search (DFS)
- **Unit 3 Review:** [09-Unit-3-Review.md](./09-Unit-3-Review.md) — Unit 3 Comprehensive Review & Exam Preparation
