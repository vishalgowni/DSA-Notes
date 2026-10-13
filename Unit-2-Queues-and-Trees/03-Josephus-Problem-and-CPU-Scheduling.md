# Unit 2 — Topic 3: Queue Applications — The Josephus Problem and OS CPU Scheduling

---

## Table of Contents

1. [Introduction and Learning Objectives](#1-introduction-and-learning-objectives)
   - 1.1 [From Queue Mechanics to Systems and Algorithmic Modelling](#11-from-queue-mechanics-to-systems-and-algorithmic-modelling)
   - 1.2 [Overview of the Two Syllabus Case Studies](#12-overview-of-the-two-syllabus-case-studies)
   - 1.3 [Learning Objectives](#13-learning-objectives)
2. [Definitions, Mathematical Formulations, and Terminology](#2-definitions-mathematical-formulations-and-terminology)
   - 2.1 [Case Study 1: Formal Definition of the Josephus Problem $J(n, k)$](#21-case-study-1-formal-definition-of-the-josephus-problem-jn-k)
   - 2.2 [Mathematical Recurrence Relation for $J(n, k)$ (0-Indexed and 1-Indexed)](#22-mathematical-recurrence-relation-for-jn-k-0-indexed-and-1-indexed)
   - 2.3 [Case Study 2: OS Scheduling Queues vs. CPU Scheduling Policies](#23-case-study-2-os-scheduling-queues-vs-cpu-scheduling-policies)
   - 2.4 [CPU Scheduling Metrics and Mathematical Formulas](#24-cpu-scheduling-metrics-and-mathematical-formulas)
   - 2.5 [Core Terminology Table](#25-core-terminology-table)
3. [Representation and Memory Layout in C](#3-representation-and-memory-layout-in-c)
   - 3.1 [Modeling Circular Rotation Using a FIFO Queue](#31-modeling-circular-rotation-using-a-fifo-queue)
   - 3.2 [Process Control Block (`Process`) and Ready Queue Representation](#32-process-control-block-process-and-ready-queue-representation)
4. [Case Study 1: The Josephus Problem — Algorithms, Trace, and C11 Implementation](#4-case-study-1-the-josephus-problem--algorithms-trace-and-c11-implementation)
   - 4.1 [Queue-Based Simulation Algorithm ($O(n \cdot k)$) and Recurrence Algorithm ($O(n)$)](#41-queue-based-simulation-algorithm-on-cdot-k-and-recurrence-algorithm-on)
   - 4.2 [Complete Step-by-Step Worked Trace for $n = 7, k = 3$](#42-complete-step-by-step-worked-trace-for-n--7-k--3)
   - 4.3 [Recurrence Verification Table for $n = 7, k = 3$](#43-recurrence-verification-table-for-n--7-k--3)
   - 4.4 [Complete C11 Program: Josephus Queue Simulation & $O(n)$ Mathematical Solver](#44-complete-c11-program-josephus-queue-simulation--on-mathematical-solver)
5. [Case Study 2: CPU Scheduling — Algorithms, Worked Examples, and C11 Implementation](#5-case-study-2-cpu-scheduling--algorithms-worked-examples-and-c11-implementation)
   - 5.1 [First-Come, First-Served (FCFS) Scheduling and the Convoy Effect](#51-first-come-first-served-fcfs-scheduling-and-the-convoy-effect)
   - 5.2 [Worked Example 1: FCFS Scheduling with Gantt Chart and Metrics Table](#52-worked-example-1-fcfs-scheduling-with-gantt-chart-and-metrics-table)
   - 5.3 [Round-Robin (RR) Scheduling with Time Quantum $q$](#53-round-robin-rr-scheduling-with-time-quantum-q)
   - 5.4 [Worked Example 2: Round-Robin Scheduling ($q = 2$) with Ready-Queue Trace](#54-worked-example-2-round-robin-scheduling-q--2-with-ready-queue-trace)
   - 5.5 [Complete C11 Program: FCFS and Round-Robin CPU Scheduler Simulation](#55-complete-c11-program-fcfs-and-round-robin-cpu-scheduler-simulation)
6. [Comparison Tables and Complexity Summary](#6-comparison-tables-and-complexity-summary)
   - 6.1 [Josephus Problem Algorithmic Approaches Compared](#61-josephus-problem-algorithmic-approaches-compared)
   - 6.2 [CPU Scheduling Policies Compared (FCFS vs. Round-Robin vs. Priority)](#62-cpu-scheduling-policies-compared-fcfs-vs-round-robin-vs-priority)
7. [Common Errors, Pitfalls, and Edge Cases](#7-common-errors-pitfalls-and-edge-cases)
   - 7.1 [Pitfall 1: Rotating $k$ Times Instead of $k - 1$ Times in Josephus Simulation](#71-pitfall-1-rotating-k-times-instead-of-k---1-times-in-josephus-simulation)
   - 7.2 [Pitfall 2: Mixing 0-Indexed and 1-Indexed Josephus Recurrences](#72-pitfall-2-mixing-0-indexed-and-1-indexed-josephus-recurrences)
   - 7.3 [Pitfall 3: Enqueuing Preempted Process Before Newly Arrived Processes in Round-Robin](#73-pitfall-3-enqueuing-preempted-process-before-newly-arrived-processes-in-round-robin)
   - 7.4 [Pitfall 4: Failing to Advance Simulation Clock During CPU Idle Gaps](#74-pitfall-4-failing-to-advance-simulation-clock-during-cpu-idle-gaps)
   - 7.5 [Boundary Conditions Checklist](#75-boundary-conditions-checklist)
8. [Summary and Revision Checklist](#8-summary-and-revision-checklist)
   - 8.1 [Key Takeaways and Essential Formulas](#81-key-takeaways-and-essential-formulas)
   - 8.2 [Self-Test Revision Questions](#82-self-test-revision-questions)
   - 8.3 [Navigation Links](#83-navigation-links)

---

## 1. Introduction and Learning Objectives

### 1.1 From Queue Mechanics to Systems and Algorithmic Modelling

In [01-Queue-Fundamentals.md](./01-Queue-Fundamentals.md) and [02-Circular-Priority-and-Deque.md](./02-Circular-Priority-and-Deque.md), we mastered the internal representation and $O(1)$ operations of Linear, Circular, Priority, and Double-Ended Queues. A fundamental skill in Data Structures and Algorithms is recognizing when a seemingly complex cyclic or temporal scheduling problem reduces directly to a sequence of queue operations:

$$\text{Cyclic Rotation by } 1 \text{ step} \iff \text{enqueue}\big(Q,\ \text{dequeue}(Q)\big)$$

Whenever an item at the `front` of a queue is given a turn (or skipped) and remains active for a future turn, removing it from `front` and immediately re-inserting it at `rear` advances the cyclic state in $O(1)$ time while preserving relative arrival order among all participants.

### 1.2 Overview of the Two Syllabus Case Studies

1. **Case Study 1 — The Josephus Problem ($J(n, k)$):**
   A classic combinatorial and algorithmic puzzle in which $n$ people stand in a circle and every $k$-th person is eliminated until only a single survivor remains. It illustrates the duality between **discrete event simulation using a Queue** ($O(n \cdot k)$) and **closed-form dynamic programming / mathematical recurrence** ($O(n)$).
2. **Case Study 2 — Operating System CPU Scheduling (FCFS & Round-Robin):**
   The core mechanism by which an Operating System kernel multiplexes a single CPU core across multiple competing processes using a **Ready Queue**, quantifying fairness and responsiveness through **Completion Time (CT)**, **Turnaround Time (TAT)**, **Waiting Time (WT)**, and **Response Time (RT)**.

### 1.3 Learning Objectives

After studying this note, you will be able to:
1. Model the **Josephus Problem** using a FIFO/Circular Queue ($k - 1$ `dequeue`+`enqueue` rotations followed by $1$ permanent `dequeue`), trace the exact elimination sequence for $n = 7, k = 3$, and derive the $O(n)$ recurrence relation $J(n, k) = (J(n-1, k) + k) \bmod n$.
2. Clearly distinguish between **OS Scheduling Queues** (Ready Queue, I/O Waiting Queue, Multi-Level Queues) and **CPU Scheduling Policies** (FCFS, Round-Robin, Priority).
3. Compute **CT, TAT, WT, and RT**, draw accurate **Gantt charts**, and trace the exact **Ready Queue state** at every arrival and preemption in **FCFS** and **Round-Robin** scheduling.
4. Write complete, compilable C11 simulations for both the Josephus Problem and Round-Robin CPU Scheduling.

---

## 2. Definitions, Mathematical Formulations, and Terminology

### 2.1 Case Study 1: Formal Definition of the Josephus Problem $J(n, k)$

Named after the 1st-century historian Flavius Josephus, the **Josephus Problem** is stated as follows:
- Let $n \ge 1$ people, labeled $1, 2, \dots, n$ in clockwise order, stand in a circle.
- Counting begins at person $1$ and proceeds clockwise around the circle of **currently remaining** people.
- In each round, $k - 1$ people are skipped ($k \ge 1$), and the **$k$-th person is eliminated** and removed from the circle.
- Counting for the next round resumes immediately with the person standing clockwise after the just-eliminated person.
- The process repeats for $n - 1$ elimination rounds until **exactly $1$ survivor** remains. The problem asks for the initial position $J(n, k)$ of this survivor (as well as the full elimination sequence).

```
             Initial Circle for n = 7, k = 3 (Counting starts at 1)

                                    (1)
                                (7)     (2)
                               (6)       (3)  <-- 1st eliminated (count: 1, 2, 3!)
                                  (5) (4)
                                       ^
                                       |-- Next count starts at (4)
```

### 2.2 Mathematical Recurrence Relation for $J(n, k)$ (0-Indexed and 1-Indexed)

Why can we find the survivor in $O(n)$ time without simulating all $n \cdot k$ steps?
Suppose the $n$ people are **0-indexed**: $0, 1, 2, \dots, n - 1$.
1. **Base Case ($n = 1$):** When only $1$ person (index `0`) is in the circle, they are the survivor:
   $$J_0(1, k) = 0$$
2. **Inductive Step ($n > 1$):**
   - In the very first round, counting $k$ positions starting from index $0$ eliminates the person at index:
     $$e_1 = (k - 1) \bmod n$$
   - Exactly $n - 1$ people remain, and counting for the next round starts at the person immediately following $e_1$, which is index $k \bmod n$.
   - If we relabel the remaining $n - 1$ people so that the new starting person (old index $k \bmod n$) gets new index $0'$, the next person gets $1'$, and so on up to $(n - 2)'$, the mapping from a new index $x' \in \{0, \dots, n-2\}$ back to its original index $x \in \{0, \dots, n-1\}$ is a cyclic shift by $+k$ modulo $n$:
     $$x = (x' + k) \bmod n$$
   - By the inductive hypothesis, the survivor among $n - 1$ people is at new index $x' = J_0(n - 1, k)$. Mapping back to the original $n$-person circle yields the **0-indexed Josephus Recurrence**:
     $$J_0(n, k) = \big(J_0(n - 1, k) + k\big) \bmod n \qquad \text{for } n \ge 2$$

3. **1-Indexed Formula ($1, 2, \dots, n$):**
   Since 1-indexed labels simply add $1$ to 0-indexed labels ($J_1(n, k) = J_0(n, k) + 1$):
   $$J_1(1, k) = 1, \qquad J_1(n, k) = \Big(\big(J_1(n - 1, k) + k - 1\big) \bmod n\Big) + 1$$

### 2.3 Case Study 2: OS Scheduling Queues vs. CPU Scheduling Policies

Students frequently confuse the **queues** maintained by an operating system with the **scheduling policy** used to select from those queues. Keep this distinction sharp:

```
                      OPERATING SYSTEM PROCESS QUEUEING MODEL

   New Arrivals               +--------------------+
  -------------> [Job Queue]  |   READY QUEUE      |        +---------+   Terminated
                 (Admission)  | +----+----+----+   |------->|   CPU   |------------->
                              | | P3 | P2 | P1 |   |Dispatch+---------+
                              | +----+----+----+   |         |       |
                              +--------------------+         |       |
                                 ^              ^            |       | Time Slice
                                 | I/O Complete |------------|       | Expired
                                 |              |  (Preemption / RR) +---------------+
                          +------+-------+      +------------------------------------+
                          | I/O Device   |
                          | Wait Queue   |<------------------------------------------+
                          +--------------+               I/O Request
```

#### A. OS Scheduling Queues (Where Processes Wait)
1. **Job Queue:** Holds all processes entering the system awaiting main memory allocation.
2. **Ready Queue:** Holds all processes residing in main memory that are **ready and waiting** to execute on a CPU core.
3. **I/O (Device) Waiting Queues:** Separate queues per hardware device (disk, NIC, terminal) holding processes blocked on an I/O completion interrupt.
4. **Multi-Level Queue (MLQ) / Multi-Level Feedback Queue (MLFQ):** Partitions the Ready Queue into multiple separate queues by workload class (e.g., Q0: Interactive/Foreground using Round-Robin; Q1: Batch/Background using FCFS).

#### B. CPU Scheduling Policies (How the Ready Queue Is Ordered and Served)
1. **First-Come, First-Served (FCFS):** **Non-preemptive** policy backed by a standard **FIFO Queue**. Once a process gets the CPU, it runs until completion (or voluntary I/O block).
2. **Round-Robin (RR):** **Preemptive** policy designed for time-sharing systems, backed by a **Circular FIFO Queue** and a fixed hardware timer slice called the **Time Quantum ($q$)**. Each process at the `front` of the Ready Queue runs for at most $q$ time units; if it has not finished, the timer interrupts it, and the OS re-enqueues it at the `rear` of the Ready Queue.
3. **Priority Scheduling:** Backed by a **Priority Queue** where the process with the highest priority is dispatched next.

### 2.4 CPU Scheduling Metrics and Mathematical Formulas

For each process $P_i$ ($i = 1, 2, \dots, n$), let:
- $\text{AT}_i$: **Arrival Time** — the timestamp when $P_i$ enters the Ready Queue.
- $\text{BT}_i$: **Burst Time** — total CPU execution time required by $P_i$.
- $\text{Start}_i$: **First CPU Start Time** — the timestamp when $P_i$ gets the CPU for the **very first time**.
- $\text{CT}_i$: **Completion Time** — the timestamp when $P_i$ finishes its entire CPU burst.

From these four quantities, we define the three universal performance metrics:

1. **Turnaround Time ($\text{TAT}_i$):** Total elapsed time from arrival until completion:
   $$\text{TAT}_i = \text{CT}_i - \text{AT}_i$$

2. **Waiting Time ($\text{WT}_i$):** Total time spent waiting inside the Ready Queue (excluding actual CPU execution time):
   $$\text{WT}_i = \text{TAT}_i - \text{BT}_i = (\text{CT}_i - \text{AT}_i) - \text{BT}_i$$

3. **Response Time ($\text{RT}_i$):** Elapsed time from arrival until the process **first** begins executing on the CPU:
   $$\text{RT}_i = \text{Start}_i - \text{AT}_i$$
   *(Note: In non-preemptive policies like FCFS, a process never relinquishes the CPU once started, so $\text{RT}_i = \text{WT}_i$. In preemptive policies like Round-Robin, a process may wait again after its first time slice, so $\text{RT}_i \le \text{WT}_i$!)*

4. **System Averages:**
   $$\overline{\text{TAT}} = \frac{1}{n}\sum_{i=1}^{n} \text{TAT}_i, \qquad \overline{\text{WT}} = \frac{1}{n}\sum_{i=1}^{n} \text{WT}_i, \qquad \overline{\text{RT}} = \frac{1}{n}\sum_{i=1}^{n} \text{RT}_i$$

### 2.5 Core Terminology Table

| Term | Symbol | Definition |
| :--- | :---: | :--- |
| **Josephus Survivor** | $J(n, k)$ | Position of the last remaining person among $n$ when every $k$-th is eliminated. |
| **Preemptive Scheduling** | — | OS can forcibly interrupt a running process (e.g., via timer interrupt) and move it back to the Ready Queue. |
| **Non-Preemptive Scheduling** | — | Running process retains the CPU until it voluntarily terminates or blocks for I/O. |
| **Time Quantum (Time Slice)** | $q$ | Maximum contiguous CPU time units a process may execute per turn in Round-Robin. |
| **Context Switch** | — | Kernel operation saving the current process's CPU registers/PCB and loading the next process's state. |
| **Convoy Effect** | — | Phenomenon in FCFS where many short processes wait behind one long CPU-bound process, inflating $\overline{\text{WT}}$. |

---

## 3. Representation and Memory Layout in C

### 3.1 Modeling Circular Rotation Using a FIFO Queue

Why does a standard FIFO Queue (implemented via a Circular Array or Linked List) naturally model the circle of people in the Josephus Problem?
- Imagine the circle unrolled so that the **next person to be counted** is always at the **`front`** of the queue, and the person just behind the starting point is at the **`rear`**.
- **Skipping a person (`1` through `k - 1`):** The person at `front` survives this count and passes the turn to the next person. In queue terms, we simply `dequeue` them from `front` and immediately `enqueue` them at `rear`!
- **Eliminating the $k$-th person:** After $k - 1$ rotations, the $k$-th person is now sitting right at `front`. We `dequeue` them permanently without re-enqueuing!

```
  One Round of Josephus (k = 3) Using a FIFO Queue:

  Initial Queue:   front -> [ 1 | 2 | 3 | 4 | 5 | 6 | 7 ] <- rear
  Rotation 1/2:    dequeue(1), enqueue(1) -> [ 2 | 3 | 4 | 5 | 6 | 7 | 1 ]
  Rotation 2/2:    dequeue(2), enqueue(2) -> [ 3 | 4 | 5 | 6 | 7 | 1 | 2 ]
  Elimination (3): dequeue(3) PERMANENTLY -> [ 4 | 5 | 6 | 7 | 1 | 2 ] (Next round starts at 4!)
```

### 3.2 Process Control Block (`Process`) and Ready Queue Representation

```c
typedef struct {
    int pid;             /* Process Identifier (e.g., 1 for P1)          */
    int arrival_time;    /* AT: Timestamp when process enters system     */
    int burst_time;      /* BT: Total required CPU time                  */
    int remaining_time;  /* Remaining CPU time (initialized to BT)       */
    int start_time;      /* First time scheduled on CPU (-1 if not yet)  */
    int completion_time; /* CT: Timestamp when remaining_time reaches 0  */
    int turnaround_time; /* TAT = CT - AT                                */
    int waiting_time;    /* WT  = TAT - BT                               */
    int response_time;   /* RT  = start_time - AT                        */
} Process;
```

---

## 4. Case Study 1: The Josephus Problem — Algorithms, Trace, and C11 Implementation

### 4.1 Queue-Based Simulation Algorithm ($O(n \cdot k)$) and Recurrence Algorithm ($O(n)$)

```text
ALGORITHM JosephusQueueSimulation(n, k)
Input:  Number of people n >= 1, step count k >= 1
Output: Prints elimination sequence and returns 1-indexed survivor position
1. Initialize CircularQueue Q of capacity n
2. FOR person <- 1 TO n DO
3.     Enqueue(Q, person)
4. END FOR
5. FOR round <- 1 TO n - 1 DO
6.     /* Rotate k - 1 people from front to rear */
7.     FOR step <- 1 TO k - 1 DO
8.         skipped_person <- Dequeue(Q)
9.         Enqueue(Q, skipped_person)
10.    END FOR
11.    /* Permanently eliminate the k-th person at front */
12.    eliminated_person <- Dequeue(Q)
13.    PRINT "Round ", round, ": Eliminated ", eliminated_person
14. END FOR
15. survivor <- Dequeue(Q)
16. RETURN survivor

ALGORITHM JosephusMathematicalRecurrence(n, k)
Input:  n >= 1, k >= 1
Output: 1-indexed survivor J_1(n, k) in O(n) time and O(1) space
1. survivor_0 <- 0                     /* Base case: J_0(1, k) = 0 */
2. FOR i <- 2 TO n DO
3.     survivor_0 <- (survivor_0 + k) MOD i
4. END FOR
5. RETURN survivor_0 + 1               /* Convert 0-indexed to 1-indexed */
```

### 4.2 Complete Step-by-Step Worked Trace for $n = 7, k = 3$

We initialize the queue with $n = 7$ people: `front -> [1, 2, 3, 4, 5, 6, 7] <- rear`.
In every round, we perform $k - 1 = 2$ rotations (`dequeue` + `enqueue`), followed by $1$ permanent `dequeue`:

| Round | Queue Before Round (`front -> rear`) | Rotation 1 (`dequeue` $\to$ `enqueue`) | Rotation 2 (`dequeue` $\to$ `enqueue`) | **$k$-th Eliminated (`dequeue`)** | Queue After Round (`front -> rear`) |
| :---: | :--- | :--- | :--- | :---: | :--- |
| **1** | `[1, 2, 3, 4, 5, 6, 7]` | Move `1` to rear $\to$ `[2, 3, 4, 5, 6, 7, 1]` | Move `2` to rear $\to$ `[3, 4, 5, 6, 7, 1, 2]` | **Eliminate `3`** | `[4, 5, 6, 7, 1, 2]` (size = 6) |
| **2** | `[4, 5, 6, 7, 1, 2]` | Move `4` to rear $\to$ `[5, 6, 7, 1, 2, 4]` | Move `5` to rear $\to$ `[6, 7, 1, 2, 4, 5]` | **Eliminate `6`** | `[7, 1, 2, 4, 5]` (size = 5) |
| **3** | `[7, 1, 2, 4, 5]` | Move `7` to rear $\to$ `[1, 2, 4, 5, 7]` | Move `1` to rear $\to$ `[2, 4, 5, 7, 1]` | **Eliminate `2`** | `[4, 5, 7, 1]` (size = 4) |
| **4** | `[4, 5, 7, 1]` | Move `4` to rear $\to$ `[5, 7, 1, 4]` | Move `5` to rear $\to$ `[7, 1, 4, 5]` | **Eliminate `7`** | `[1, 4, 5]` (size = 3) |
| **5** | `[1, 4, 5]` | Move `1` to rear $\to$ `[4, 5, 1]` | Move `4` to rear $\to$ `[5, 1, 4]` | **Eliminate `5`** | `[1, 4]` (size = 2) |
| **6** | `[1, 4]` | Move `1` to rear $\to$ `[4, 1]` | Move `4` to rear $\to$ `[1, 4]` | **Eliminate `1`** | **`[4]` (size = 1 — SURVIVOR!)** |

- **Exact Elimination Order:** **`3, 6, 2, 7, 5, 1`**
- **Final Survivor:** **`4`**

### 4.3 Recurrence Verification Table for $n = 7, k = 3$

Let us verify that our $O(n)$ recurrence formula produces the exact same survivor $J_1(7, 3) = 4$:

| Subproblem Size $i$ | 0-Indexed Formula: $J_0(i, 3) = (J_0(i-1, 3) + 3) \bmod i$ | 1-Indexed Survivor: $J_1(i, 3) = J_0(i, 3) + 1$ |
| :---: | :--- | :---: |
| $i = 1$ | $J_0(1, 3) = 0$ (Base case) | $0 + 1 = 1$ |
| $i = 2$ | $(0 + 3) \bmod 2 = 3 \bmod 2 = 1$ | $1 + 1 = 2$ |
| $i = 3$ | $(1 + 3) \bmod 3 = 4 \bmod 3 = 1$ | $1 + 1 = 2$ |
| $i = 4$ | $(1 + 3) \bmod 4 = 4 \bmod 4 = 0$ | $0 + 1 = 1$ |
| $i = 5$ | $(0 + 3) \bmod 5 = 3 \bmod 5 = 3$ | $3 + 1 = 4$ |
| $i = 6$ | $(3 + 3) \bmod 6 = 6 \bmod 6 = 0$ | $0 + 1 = 1$ |
| **$i = 7$** | **$(0 + 3) \bmod 7 = 3 \bmod 7 = 3$** | **$3 + 1 = 4$ (Verified!)** |

### 4.4 Complete C11 Program: Josephus Queue Simulation & $O(n)$ Mathematical Solver

```c
/*
 * File: josephus_simulation.c
 * Standard: C11
 * Description: Solves the Josephus Problem J(n, k) using (1) an explicit
 *              Circular Queue simulation tracing every round and (2) the
 *              O(n) dynamic programming recurrence relation.
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_PEOPLE 64

typedef struct {
    int data[MAX_PEOPLE];
    int front;
    int rear;
    int count;
} JosQueue;

void jqInit(JosQueue *q) {
    q->front = 0;
    q->rear  = -1;
    q->count = 0;
}

bool jqEnqueue(JosQueue *q, int person) {
    if (q->count == MAX_PEOPLE) return false;
    q->rear = (q->rear + 1) % MAX_PEOPLE;
    q->data[q->rear] = person;
    q->count++;
    return true;
}

bool jqDequeue(JosQueue *q, int *out_person) {
    if (q->count == 0) return false;
    *out_person = q->data[q->front];
    q->front = (q->front + 1) % MAX_PEOPLE;
    q->count--;
    return true;
}

void jqPrintState(const JosQueue *q) {
    printf("[");
    for (int i = 0; i < q->count; i++) {
        int idx = (q->front + i) % MAX_PEOPLE;
        printf("%d%s", q->data[idx], (i < q->count - 1) ? ", " : "");
    }
    printf("]");
}

/* Method 1: O(n * k) Queue Simulation */
int josephusQueueSimulate(int n, int k, bool verbose) {
    JosQueue q;
    jqInit(&q);
    for (int p = 1; p <= n; p++) {
        jqEnqueue(&q, p);
    }

    if (verbose) {
        printf("  Initial Queue (n=%d, k=%d): ", n, k);
        jqPrintState(&q);
        printf("\n");
    }

    for (int round = 1; round <= n - 1; round++) {
        /* Perform k - 1 rotations: dequeue from front, enqueue at rear */
        for (int step = 1; step <= k - 1; step++) {
            int skipped = 0;
            jqDequeue(&q, &skipped);
            jqEnqueue(&q, skipped);
        }
        /* Permanently eliminate the k-th person */
        int eliminated = 0;
        jqDequeue(&q, &eliminated);

        if (verbose) {
            printf("  Round %d: Eliminated person %d | Remaining Queue: ", round, eliminated);
            jqPrintState(&q);
            printf("\n");
        }
    }

    int survivor = 0;
    jqDequeue(&q, &survivor);
    return survivor;
}

/* Method 2: O(n) Mathematical Recurrence Relation */
int josephusRecurrence1Indexed(int n, int k) {
    int survivor_0 = 0; /* Base case: J_0(1, k) = 0 */
    for (int i = 2; i <= n; i++) {
        survivor_0 = (survivor_0 + k) % i;
    }
    return survivor_0 + 1; /* Convert to 1-indexed */
}

int main(void) {
    printf("=== CASE STUDY 1: JOSEPHUS PROBLEM (n = 7, k = 3) ===\n");
    int surv_sim  = josephusQueueSimulate(7, 3, true);
    int surv_math = josephusRecurrence1Indexed(7, 3);
    printf("\n  -> Queue Simulation Survivor : %d\n", surv_sim);
    printf("  -> O(n) Recurrence Survivor  : %d\n", surv_math);

    printf("\n=== CROSS-VERIFICATION ACROSS MULTIPLE (n, k) PAIRS ===\n");
    printf("  +------+------+------------------+-------------------+\n");
    printf("  |  n   |  k   | Queue Simulation | O(n) Recurrence   |\n");
    printf("  +------+------+------------------+-------------------+\n");
    int test_cases[][2] = {{5, 2}, {7, 3}, {10, 3}, {14, 2}, {41, 3}};
    for (size_t i = 0; i < sizeof(test_cases) / sizeof(test_cases[0]); i++) {
        int n = test_cases[i][0];
        int k = test_cases[i][1];
        printf("  | %4d | %4d |       %4d       |       %4d        |\n",
               n, k, josephusQueueSimulate(n, k, false), josephusRecurrence1Indexed(n, k));
    }
    printf("  +------+------+------------------+-------------------+\n");
    return 0;
}
```

#### Program Output:

```text
=== CASE STUDY 1: JOSEPHUS PROBLEM (n = 7, k = 3) ===
  Initial Queue (n=7, k=3): [1, 2, 3, 4, 5, 6, 7]
  Round 1: Eliminated person 3 | Remaining Queue: [4, 5, 6, 7, 1, 2]
  Round 2: Eliminated person 6 | Remaining Queue: [7, 1, 2, 4, 5]
  Round 3: Eliminated person 2 | Remaining Queue: [4, 5, 7, 1]
  Round 4: Eliminated person 7 | Remaining Queue: [1, 4, 5]
  Round 5: Eliminated person 5 | Remaining Queue: [1, 4]
  Round 6: Eliminated person 1 | Remaining Queue: [4]

  -> Queue Simulation Survivor : 4
  -> O(n) Recurrence Survivor  : 4

=== CROSS-VERIFICATION ACROSS MULTIPLE (n, k) PAIRS ===
  +------+------+------------------+-------------------+
  |  n   |  k   | Queue Simulation | O(n) Recurrence   |
  +------+------+------------------+-------------------+
  |    5 |    2 |          3       |          3        |
  |    7 |    3 |          4       |          4        |
  |   10 |    3 |          4       |          4        |
  |   14 |    2 |         13       |         13        |
  |   41 |    3 |         31       |         31        |
  +------+------+------------------+-------------------+
```

---

## 5. Case Study 2: CPU Scheduling — Algorithms, Worked Examples, and C11 Implementation

### 5.1 First-Come, First-Served (FCFS) Scheduling and the Convoy Effect

In **First-Come, First-Served (FCFS)** scheduling:
- Processes are inserted into the `rear` of a standard FIFO Ready Queue in increasing order of their **Arrival Time ($\text{AT}_i$)** (ties broken by lower Process ID).
- The CPU dispatcher dequeues the process at `front` and runs it **non-preemptively** for its entire Burst Time ($\text{BT}_i$).
- **The Convoy Effect:** If a CPU-intensive process with a huge burst time (e.g., $\text{BT}_1 = 24\text{ ms}$) arrives just before several short interactive processes ($\text{BT}_2 = 3\text{ ms}, \text{BT}_3 = 3\text{ ms}$), all short processes remain trapped in the Ready Queue behind $P_1$, causing high average waiting and response times.

### 5.2 Worked Example 1: FCFS Scheduling with Gantt Chart and Metrics Table

Consider $n = 4$ processes scheduled under **FCFS**:

| Process | Arrival Time ($\text{AT}$) | Burst Time ($\text{BT}$) |
| :---: | :---: | :---: |
| $P_1$ | $0$ | $5$ |
| $P_2$ | $1$ | $3$ |
| $P_3$ | $2$ | $8$ |
| $P_4$ | $3$ | $6$ |

#### ASCII Gantt Chart (FCFS)

```
  +---------------+---------+------------------------+------------------+
  |      P1       |   P2    |           P3           |        P4        |
  +---------------+---------+------------------------+------------------+
  0               5         8                       16                 22
```

#### Per-Process Metrics Calculation Table (FCFS)

| Process | $\text{AT}$ | $\text{BT}$ | First Start | $\text{CT}$ | $\text{TAT} = \text{CT} - \text{AT}$ | $\text{WT} = \text{TAT} - \text{BT}$ | $\text{RT} = \text{Start} - \text{AT}$ |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **$P_1$** | $0$ | $5$ | $0$ | $5$ | $5 - 0 = 5$ | $5 - 5 = 0$ | $0 - 0 = 0$ |
| **$P_2$** | $1$ | $3$ | $5$ | $8$ | $8 - 1 = 7$ | $7 - 3 = 4$ | $5 - 1 = 4$ |
| **$P_3$** | $2$ | $8$ | $8$ | $16$ | $16 - 2 = 14$ | $14 - 8 = 6$ | $8 - 2 = 6$ |
| **$P_4$** | $3$ | $6$ | $16$ | $22$ | $22 - 3 = 19$ | $19 - 6 = 13$ | $16 - 3 = 13$ |
| **Total / Avg** | — | $\sum = 22$ | — | — | $\overline{\text{TAT}} = \frac{45}{4} = \mathbf{11.25}$ | $\overline{\text{WT}} = \frac{23}{4} = \mathbf{5.75}$ | $\overline{\text{RT}} = \frac{23}{4} = \mathbf{5.75}$ |

### 5.3 Round-Robin (RR) Scheduling with Time Quantum $q$

**Round-Robin (RR)** eliminates the Convoy Effect and guarantees bounded **Response Time** by adding a hardware timer interrupt with **Time Quantum $q$**:
1. The scheduler dequeues process $P_i$ from the `front` of the **Circular Ready Queue**.
2. $P_i$ executes on the CPU for $\Delta t = \min(\text{remaining\_time}_i,\ q)$ time units, advancing the clock from $t$ to $t' = t + \Delta t$.
3. **CRITICAL ARRIVAL-BEFORE-PREEMPTION RULE:**
   Any new processes that arrived during the interval $(t,\ t']$ (i.e., with $\text{AT} \le t'$) are enqueued into the `rear` of the Ready Queue **first** (in arrival order).
4. Next, if $P_i$ still has $\text{remaining\_time}_i > 0$ at time $t'$, $P_i$ is preempted and re-enqueued at the `rear` of the Ready Queue (behind any newly arrived processes). If $\text{remaining\_time}_i == 0$, $P_i$ records $\text{CT}_i = t'$ and terminates.

#### Choosing the Time Quantum $q$
- **If $q \to \infty$ (larger than maximum burst time):** Round-Robin degenerates into **FCFS** (zero preemptions).
- **If $q \to 0$ (excessively small):** Processes experience rapid context switches; **context-switch overhead** dominates useful CPU work. In practice, $q$ is chosen around $10\text{--}100\text{ ms}$ so $\sim 80\%$ of interactive CPU bursts complete within a single quantum.

### 5.4 Worked Example 2: Round-Robin Scheduling ($q = 2$) with Ready-Queue Trace

Consider the **exact same 4 processes** ($P_1: \text{AT}=0, \text{BT}=5$; $P_2: \text{AT}=1, \text{BT}=3$; $P_3: \text{AT}=2, \text{BT}=8$; $P_4: \text{AT}=3, \text{BT}=6$) scheduled under **Round-Robin with Time Quantum $q = 2$**:

#### Step-by-Step Ready Queue & Preemption Trace ($q = 2$)

| Time Slice | Running Process | Remaining $\text{BT}$ | New Arrivals During Slice | Preempted / Completed Action | Ready Queue State at End of Slice (`front -> rear`) |
| :---: | :---: | :---: | :--- | :--- | :--- |
| **$t = 0$ (Init)** | — | $P_1:5$ | $P_1$ arrives at $t=0$ | Enqueue $P_1$ | `[P1]` |
| **$0 \to 2$** | **$P_1$** | $5 \to 3$ | $P_2 (\text{AT}=1), P_3 (\text{AT}=2)$ arrive | Enqueue $P_2, P_3$, then re-enqueue preempted $P_1$ | `[P2, P3, P1]` |
| **$2 \to 4$** | **$P_2$** | $3 \to 1$ | $P_4 (\text{AT}=3)$ arrives | Enqueue $P_4$, then re-enqueue preempted $P_2$ | `[P3, P1, P4, P2]` |
| **$4 \to 6$** | **$P_3$** | $8 \to 6$ | None | Re-enqueue preempted $P_3$ | `[P1, P4, P2, P3]` |
| **$6 \to 8$** | **$P_1$** | $3 \to 1$ | None | Re-enqueue preempted $P_1$ | `[P4, P2, P3, P1]` |
| **$8 \to 10$** | **$P_4$** | $6 \to 4$ | None | Re-enqueue preempted $P_4$ | `[P2, P3, P1, P4]` |
| **$10 \to 11$** | **$P_2$** | $1 \to 0$ | None | **$P_2$ Completes ($\text{CT}_2 = 11$)** (used only 1 unit!) | `[P3, P1, P4]` |
| **$11 \to 13$** | **$P_3$** | $6 \to 4$ | None | Re-enqueue preempted $P_3$ | `[P1, P4, P3]` |
| **$13 \to 14$** | **$P_1$** | $1 \to 0$ | None | **$P_1$ Completes ($\text{CT}_1 = 14$)** (used only 1 unit!) | `[P4, P3]` |
| **$14 \to 16$** | **$P_4$** | $4 \to 2$ | None | Re-enqueue preempted $P_4$ | `[P3, P4]` |
| **$16 \to 18$** | **$P_3$** | $4 \to 2$ | None | Re-enqueue preempted $P_3$ | `[P4, P3]` |
| **$18 \to 20$** | **$P_4$** | $2 \to 0$ | None | **$P_4$ Completes ($\text{CT}_4 = 20$)** | `[P3]` |
| **$20 \to 22$** | **$P_3$** | $2 \to 0$ | None | **$P_3$ Completes ($\text{CT}_3 = 22$)** | `[]` (Empty — Done!) |

#### ASCII Gantt Chart (Round-Robin, $q = 2$)

```
  +----+----+----+----+----+----+----+----+----+----+----+----+
  | P1 | P2 | P3 | P1 | P4 | P2 | P3 | P1 | P4 | P3 | P4 | P3 |
  +----+----+----+----+----+----+----+----+----+----+----+----+
  0    2    4    6    8   10   11   13   14   16   18   20   22
```

#### Per-Process Metrics Calculation Table (Round-Robin, $q = 2$)

| Process | $\text{AT}$ | $\text{BT}$ | First Start | $\text{CT}$ | $\text{TAT} = \text{CT} - \text{AT}$ | $\text{WT} = \text{TAT} - \text{BT}$ | $\text{RT} = \text{Start} - \text{AT}$ |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **$P_1$** | $0$ | $5$ | $0$ | $14$ | $14 - 0 = 14$ | $14 - 5 = 9$ | $0 - 0 = 0$ |
| **$P_2$** | $1$ | $3$ | $2$ | $11$ | $11 - 1 = 10$ | $10 - 3 = 7$ | $2 - 1 = 1$ |
| **$P_3$** | $2$ | $8$ | $4$ | $22$ | $22 - 2 = 20$ | $20 - 8 = 12$ | $4 - 2 = 2$ |
| **$P_4$** | $3$ | $6$ | $8$ | $20$ | $20 - 3 = 17$ | $17 - 6 = 11$ | $8 - 3 = 5$ |
| **Total / Avg** | — | $\sum = 22$ | — | — | $\overline{\text{TAT}} = \frac{61}{4} = \mathbf{15.25}$ | $\overline{\text{WT}} = \frac{39}{4} = \mathbf{9.75}$ | $\overline{\text{RT}} = \frac{8}{4} = \mathbf{2.00}$ |

> **Key Engineering Insight (FCFS vs. Round-Robin):**
> Notice that Round-Robin reduced **Average Response Time ($\overline{\text{RT}}$)** from **$5.75$** down to **$2.00$** (every interactive process got its first CPU turn almost immediately!), at the cost of a higher Average Turnaround Time ($15.25$ vs. $11.25$) because long jobs are multiplexed concurrently rather than finishing one at a time.

### 5.5 Complete C11 Program: FCFS and Round-Robin CPU Scheduler Simulation

```c
/*
 * File: cpu_scheduling_simulation.c
 * Standard: C11
 * Description: Complete C11 simulation of FCFS and Round-Robin (RR) CPU
 *              Scheduling using a Circular Ready Queue, outputting the
 *              Gantt chart, per-process CT/TAT/WT/RT table, and averages.
 */

#include <stdio.h>
#include <stdlib.h>
#include <stdbool.h>

#define MAX_PROCS 16
#define MAX_SLICES 64

typedef struct {
    int pid;
    int at;          /* Arrival Time    */
    int bt;          /* Burst Time      */
    int rem_bt;      /* Remaining Burst */
    int start_time;  /* First CPU Start */
    int ct;          /* Completion Time */
    int tat;         /* Turnaround Time */
    int wt;          /* Waiting Time    */
    int rt;          /* Response Time   */
} Process;

typedef struct {
    int pid;         /* 0 indicates IDLE */
    int start_t;
    int end_t;
} GanttSlice;

/* Circular Queue of Process Array Indices */
typedef struct {
    int data[MAX_PROCS];
    int front;
    int rear;
    int count;
} ReadyQueue;

void rqInit(ReadyQueue *rq) {
    rq->front = 0;
    rq->rear  = -1;
    rq->count = 0;
}

bool rqIsEmpty(const ReadyQueue *rq) { return (rq->count == 0); }

bool rqEnqueue(ReadyQueue *rq, int proc_idx) {
    if (rq->count == MAX_PROCS) return false;
    rq->rear = (rq->rear + 1) % MAX_PROCS;
    rq->data[rq->rear] = proc_idx;
    rq->count++;
    return true;
}

bool rqDequeue(ReadyQueue *rq, int *out_proc_idx) {
    if (rq->count == 0) return false;
    *out_proc_idx = rq->data[rq->front];
    rq->front = (rq->front + 1) % MAX_PROCS;
    rq->count--;
    return true;
}

void printGanttAndMetrics(const char *title, const Process procs[], int n,
                          const GanttSlice slices[], int num_slices) {
    printf("=== %s ===\n\n", title);
    printf("  ASCII Gantt Chart:\n  ");
    for (int i = 0; i < num_slices; i++) printf("+------");
    printf("+\n  ");
    for (int i = 0; i < num_slices; i++) {
        if (slices[i].pid == 0) printf("| IDLE ");
        else                    printf("|  P%-2d ", slices[i].pid);
    }
    printf("|\n  ");
    for (int i = 0; i < num_slices; i++) printf("+------");
    printf("+\n  %-7d", slices[0].start_t);
    for (int i = 0; i < num_slices; i++) printf("%-7d", slices[i].end_t);
    printf("\n\n");

    double sum_tat = 0.0, sum_wt = 0.0, sum_rt = 0.0;
    printf("  +-----+----+----+-------+----+-----+----+----+\n");
    printf("  | PID | AT | BT | Start | CT | TAT | WT | RT |\n");
    printf("  +-----+----+----+-------+----+-----+----+----+\n");
    for (int i = 0; i < n; i++) {
        printf("  | P%-2d | %2d | %2d |  %3d  | %2d | %3d | %2d | %2d |\n",
               procs[i].pid, procs[i].at, procs[i].bt, procs[i].start_time,
               procs[i].ct, procs[i].tat, procs[i].wt, procs[i].rt);
        sum_tat += procs[i].tat;
        sum_wt  += procs[i].wt;
        sum_rt  += procs[i].rt;
    }
    printf("  +-----+----+----+-------+----+-----+----+----+\n");
    printf("  Average Turnaround Time (TAT) = %.2f\n", sum_tat / n);
    printf("  Average Waiting Time    (WT)  = %.2f\n", sum_wt  / n);
    printf("  Average Response Time   (RT)  = %.2f\n\n", sum_rt  / n);
}

/* Simulate Round-Robin (or FCFS when quantum is very large) */
void simulateScheduler(const char *title, Process procs[], int n, int quantum) {
    ReadyQueue rq;
    rqInit(&rq);
    bool in_queue[MAX_PROCS] = {false};
    GanttSlice slices[MAX_SLICES];
    int num_slices = 0;

    for (int i = 0; i < n; i++) {
        procs[i].rem_bt     = procs[i].bt;
        procs[i].start_time = -1;
    }

    int current_time = 0;
    int completed    = 0;

    /* Enqueue processes arriving at time 0 */
    for (int i = 0; i < n; i++) {
        if (procs[i].at <= current_time && !in_queue[i]) {
            rqEnqueue(&rq, i);
            in_queue[i] = true;
        }
    }

    while (completed < n) {
        if (rqIsEmpty(&rq)) {
            /* CPU Idle Gap: advance clock to next arrival */
            int next_at = 1000000;
            for (int i = 0; i < n; i++) {
                if (procs[i].rem_bt > 0 && procs[i].at < next_at) {
                    next_at = procs[i].at;
                }
            }
            slices[num_slices++] = (GanttSlice){0, current_time, next_at};
            current_time = next_at;
            for (int i = 0; i < n; i++) {
                if (procs[i].at <= current_time && !in_queue[i] && procs[i].rem_bt > 0) {
                    rqEnqueue(&rq, i);
                    in_queue[i] = true;
                }
            }
            continue;
        }

        int idx = 0;
        rqDequeue(&rq, &idx);

        if (procs[idx].start_time == -1) {
            procs[idx].start_time = current_time;
        }

        int run_time = (procs[idx].rem_bt < quantum) ? procs[idx].rem_bt : quantum;
        int start_t  = current_time;
        current_time += run_time;
        procs[idx].rem_bt -= run_time;

        slices[num_slices++] = (GanttSlice){procs[idx].pid, start_t, current_time};

        /* 1. Enqueue any newly arrived processes up to current_time FIRST */
        for (int i = 0; i < n; i++) {
            if (procs[i].at <= current_time && !in_queue[i] && procs[i].rem_bt > 0) {
                rqEnqueue(&rq, i);
                in_queue[i] = true;
            }
        }

        /* 2. If current process still has remaining burst, re-enqueue at rear */
        if (procs[idx].rem_bt > 0) {
            rqEnqueue(&rq, idx);
        } else {
            procs[idx].ct  = current_time;
            procs[idx].tat = procs[idx].ct - procs[idx].at;
            procs[idx].wt  = procs[idx].tat - procs[idx].bt;
            procs[idx].rt  = procs[idx].start_time - procs[idx].at;
            completed++;
        }
    }

    printGanttAndMetrics(title, procs, n, slices, num_slices);
}

int main(void) {
    Process base_procs[4] = {
        {.pid = 1, .at = 0, .bt = 5},
        {.pid = 2, .at = 1, .bt = 3},
        {.pid = 3, .at = 2, .bt = 8},
        {.pid = 4, .at = 3, .bt = 6}
    };

    /* 1. FCFS Simulation (equivalent to non-preemptive / infinite quantum) */
    simulateScheduler("FIRST-COME, FIRST-SERVED (FCFS) SCHEDULING", base_procs, 4, 1000000);

    /* 2. Round-Robin Simulation with Time Quantum q = 2 */
    simulateScheduler("ROUND-ROBIN (RR) SCHEDULING (TIME QUANTUM q = 2)", base_procs, 4, 2);

    return 0;
}
```

#### Program Output:

```text
=== FIRST-COME, FIRST-SERVED (FCFS) SCHEDULING ===

  ASCII Gantt Chart:
  +------+------+------+------+
  |  P1  |  P2  |  P3  |  P4  |
  +------+------+------+------+
  0      5      8      16     22     

  +-----+----+----+-------+----+-----+----+----+
  | PID | AT | BT | Start | CT | TAT | WT | RT |
  +-----+----+----+-------+----+-----+----+----+
  | P1  |  0 |  5 |    0  |  5 |   5 |  0 |  0 |
  | P2  |  1 |  3 |    5  |  8 |   7 |  4 |  4 |
  | P3  |  2 |  8 |    8  | 16 |  14 |  6 |  6 |
  | P4  |  3 |  6 |   16  | 22 |  19 | 13 | 13 |
  +-----+----+----+-------+----+-----+----+----+
  Average Turnaround Time (TAT) = 11.25
  Average Waiting Time    (WT)  = 5.75
  Average Response Time   (RT)  = 5.75

=== ROUND-ROBIN (RR) SCHEDULING (TIME QUANTUM q = 2) ===

  ASCII Gantt Chart:
  +------+------+------+------+------+------+------+------+------+------+------+------+
  |  P1  |  P2  |  P3  |  P1  |  P4  |  P2  |  P3  |  P1  |  P4  |  P3  |  P4  |  P3  |
  +------+------+------+------+------+------+------+------+------+------+------+------+
  0      2      4      6      8      10     11     13     14     16     18     20     22     

  +-----+----+----+-------+----+-----+----+----+
  | PID | AT | BT | Start | CT | TAT | WT | RT |
  +-----+----+----+-------+----+-----+----+----+
  | P1  |  0 |  5 |    0  | 14 |  14 |  9 |  0 |
  | P2  |  1 |  3 |    2  | 11 |  10 |  7 |  1 |
  | P3  |  2 |  8 |    4  | 22 |  20 | 12 |  2 |
  | P4  |  3 |  6 |    8  | 20 |  17 | 11 |  5 |
  +-----+----+----+-------+----+-----+----+----+
  Average Turnaround Time (TAT) = 15.25
  Average Waiting Time    (WT)  = 9.75
  Average Response Time   (RT)  = 2.00
```

---

## 6. Comparison Tables and Complexity Summary

### 6.1 Josephus Problem Algorithmic Approaches Compared

Let $n$ be the number of people in the circle and $k$ be the step count.

| Approach | Time Complexity | Auxiliary Space | Produces Full Elimination Order? | Handles Huge $n$ (e.g., $n = 10^7$)? |
| :--- | :---: | :---: | :---: | :---: |
| **FIFO / Circular Queue Simulation** | $O(n \cdot k)$ | $O(n)$ | **Yes** (exact sequence of all $n-1$ eliminations) | Slow if $k$ is large |
| **Circular Singly Linked List** | $O(n \cdot k)$ | $O(n)$ | **Yes** | Slow if $k$ is large |
| **Mathematical Recurrence $J(n, k)$** | **$O(n)$** | **$O(1)$** (iterative) | **No** (computes final survivor directly) | **Yes** (independent of $k$) |

### 6.2 CPU Scheduling Policies Compared (FCFS vs. Round-Robin vs. Priority)

| Feature | First-Come, First-Served (FCFS) | Round-Robin (RR, quantum $q$) | Priority Scheduling |
| :--- | :--- | :--- | :--- |
| **Underlying Queue Structure** | Simple / Circular FIFO Queue | Circular FIFO Queue + Timer | Priority Queue (Ordered List / Heap) |
| **Preemptive?** | **No** (Non-preemptive) | **Yes** (Time-sliced every $q$ units) | Preemptive or Non-Preemptive |
| **Starvation Possible?** | **No** (every job eventually reaches `front`) | **No** (bounded wait $\le (n-1)q$ per turn) | **Yes** (low-priority jobs can starve; solved by **Aging**) |
| **Response Time ($\text{RT}$)** | Poor when long jobs arrive first (**Convoy Effect**) | **Excellent** (ideal for interactive time-sharing) | Excellent for high-priority tasks |
| **Context Switch Overhead** | Minimal (1 dispatch per process) | Medium-to-High (depends on $1/q$) | Low-to-Medium |

---

## 7. Common Errors, Pitfalls, and Edge Cases

### 7.1 Pitfall 1: Rotating $k$ Times Instead of $k - 1$ Times in Josephus Simulation

Because the $k$-th person is eliminated by a `dequeue` call, only **$k - 1$** people should be rotated (`dequeue` + `enqueue`) before that elimination `dequeue`. Rotating $k$ times eliminates the $(k + 1)$-th person!

```c
/* BUG: Rotating k times skips k people and eliminates the (k + 1)-th person! */
for (int step = 1; step <= k; step++) {
    int p;
    jqDequeue(&q, &p);
    jqEnqueue(&q, p);
}
jqDequeue(&q, &eliminated);

/* FIX: Rotate strictly (k - 1) times, then dequeue the k-th person permanently */
for (int step = 1; step <= k - 1; step++) {
    int p;
    jqDequeue(&q, &p);
    jqEnqueue(&q, p);
}
jqDequeue(&q, &eliminated);
```

### 7.2 Pitfall 2: Mixing 0-Indexed and 1-Indexed Josephus Recurrences

```c
/* BUG: Initializing 1-indexed base case survivor = 1 but applying 0-indexed modulo! */
int badJosephus(int n, int k) {
    int s = 1;
    for (int i = 2; i <= n; i++) {
        s = (s + k) % i; /* Produces 0 when (s + k) is a multiple of i! */
    }
    return s;
}

/* FIX: Compute 0-indexed J_0(n, k) cleanly, then add +1 at the very end */
int goodJosephus(int n, int k) {
    int s0 = 0; /* J_0(1, k) = 0 */
    for (int i = 2; i <= n; i++) {
        s0 = (s0 + k) % i;
    }
    return s0 + 1;
}
```

### 7.3 Pitfall 3: Enqueuing Preempted Process Before Newly Arrived Processes in Round-Robin

In standard OS textbooks (Silberschatz/Galvin) and competitive/exam problems, when a running process $P_{\text{run}}$ finishes its time quantum at time $t'$, any process $P_{\text{new}}$ that arrived during $(t, t']$ is already waiting in the Ready Queue at time $t'$ and **must be enqueued before $P_{\text{run}}$ is re-enqueued**. Reversing this order alters the entire Gantt chart!

```c
/* BUG: Re-enqueuing preempted process BEFORE checking new arrivals up to current_time */
if (procs[idx].rem_bt > 0) {
    rqEnqueue(&rq, idx);
}
for (int i = 0; i < n; i++) {
    if (procs[i].at <= current_time && !in_queue[i]) {
        rqEnqueue(&rq, i);
        in_queue[i] = true;
    }
}

/* FIX: Always enqueue new arrivals with (at <= current_time) FIRST, then re-enqueue preempted process */
for (int i = 0; i < n; i++) {
    if (procs[i].at <= current_time && !in_queue[i] && procs[i].rem_bt > 0) {
        rqEnqueue(&rq, i);
        in_queue[i] = true;
    }
}
if (procs[idx].rem_bt > 0) {
    rqEnqueue(&rq, idx);
}
```

### 7.4 Pitfall 4: Failing to Advance Simulation Clock During CPU Idle Gaps

If $P_1$ finishes at $t = 3$ and $P_2$ does not arrive until $\text{AT}_2 = 6$, the Ready Queue is empty at $t = 3$ even though `completed < n`. If your loop does not advance `current_time` to the next unscheduled arrival time, the simulation enters an infinite loop!

### 7.5 Boundary Conditions Checklist

- [ ] **Josephus $k = 1$:** Does `k - 1 = 0` rotations eliminate $1, 2, \dots, n-1$ in order and leave survivor $n$?
- [ ] **Josephus $n = 1$:** Does the simulation perform $n - 1 = 0$ elimination rounds and immediately return survivor `1`?
- [ ] **Round-Robin Remaining Burst $< q$:** When a process has $\text{rem\_bt} = 1$ and $q = 4$, does the clock advance by $\min(1, 4) = 1$ unit rather than $4$ units?
- [ ] **Response Time Recording:** Is `start_time` recorded **only on the first dispatch** (`if (start_time == -1)`) rather than overwritten on subsequent Round-Robin turns?

---

## 8. Summary and Revision Checklist

### 8.1 Key Takeaways and Essential Formulas

1. **Josephus Queue Rotation:** Each elimination round consists of **$k - 1$** `enqueue(dequeue())` rotations followed by **$1$** permanent `dequeue()`, running in $O(n \cdot k)$ total time.
2. **Josephus Recurrence ($O(n)$):**
   $$J_0(1, k) = 0, \quad J_0(n, k) = (J_0(n - 1, k) + k) \bmod n, \quad J_1(n, k) = J_0(n, k) + 1$$
3. **CPU Scheduling Identities:**
   $$\text{TAT} = \text{CT} - \text{AT}, \qquad \text{WT} = \text{TAT} - \text{BT}, \qquad \text{RT} = \text{First CPU Start Time} - \text{AT}$$
4. **FCFS vs. Round-Robin:** FCFS is non-preemptive ($\text{RT} = \text{WT}$, vulnerable to the **Convoy Effect**); Round-Robin uses a Circular Ready Queue with time quantum $q$ to guarantee fast interactive **Response Time ($\text{RT}$)**.

### 8.2 Self-Test Revision Questions

1. **Trace Question:** Use the Josephus queue rotation method to find the elimination order and survivor for $n = 5, k = 2$. Verify your answer with the recurrence relation $J_1(5, 2)$.
   *(Answer: Elimination order: `2, 4, 1, 5`; Survivor: `3`.)*
2. **Scheduling Question:** Why is $\text{Response Time (RT)}$ always equal to $\text{Waiting Time (WT)}$ in FCFS scheduling, whereas $\text{RT} \le \text{WT}$ in Round-Robin scheduling?
   *(Answer: In non-preemptive FCFS, a process runs to completion once started, so all of its waiting occurs before its first start time. In preemptive Round-Robin, a process can be interrupted after its first time slice and accumulate additional waiting time in the Ready Queue).*
3. **Edge-Case Question:** In Round-Robin scheduling with $q = 3$, if $P_1$ runs from $t = 0$ to $t = 3$ (with remaining burst $2$) and $P_2$ arrives at $t = 3$, which process runs during $t = 3 \to 6$?
   *(Answer: $P_2$ runs next because arrivals up to $t = 3$ enter the Ready Queue before the preempted process $P_1$ is re-enqueued).*

### 8.3 Navigation Links

- **Topic 1:** [01-Queue-Fundamentals.md](./01-Queue-Fundamentals.md) — Queue ADT, Simple Array Queue, and Linked-List Queue
- **Topic 2:** [02-Circular-Priority-and-Deque.md](./02-Circular-Priority-and-Deque.md) — Circular Queues, Priority Queues, and Double-Ended Queues (Deques)
- **Current File:** `03-Josephus-Problem-and-CPU-Scheduling.md` (Unit 2 — Topic 3)
