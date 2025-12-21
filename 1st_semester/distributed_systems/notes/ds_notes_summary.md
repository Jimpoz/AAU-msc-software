# Distributed Systems Course Notes Summary

## 1. Introduction & Models
A distributed system involves networked computers coordinating via message passing.
* **Concurrency Issues:** Deadlocks, livelocks, non-determinism.
* **Failure Models:**
    * **Crash-stop/Crash-silent:** Process halts.
    * **Byzantine:** Arbitrary/malicious behavior.
    * **System Failure:** Combined process failures + link assumptions.
* **Link Models:** Perfect, Fair-loss, Stubborn, Logged-perfect, Authenticated.

---

## 2. Time in Distributed Systems
Time is critical for scheduling, failure detection, and ordering.

### Physical Clocks (NTP)
synchronization protocol to correct clock drift (measured in ppm).
* **Round-trip delay ($\delta$):**
    $$\delta = (T_4 - T_1) - (T_3 - T_2)$$
* **Clock offset ($\theta$):**
    $$\theta = \left(T_3 + \frac{\delta}{2}\right) - T_4$$
* **Correction:**
    * $|\theta| < 125ms$: **Slew** (gradual adjust).
    * $125ms \le |\theta| < 1000s$: **Jump** (immediate set).
    * $|\theta| \ge 1000s$: **Ignore**.

### Logical Clocks
Used to track the **Happens-Before Relation ($\rightarrow$)**.
1.  **Lamport Clocks:** Single integer.
    * Update rule: $t = \max(t, t_{msg}) + 1$
    * Condition: $a \rightarrow b \implies L(a) < L(b)$.
2.  **Vector Clocks:** Array of size $N$.
    * Update rule: $Vi[j] = \max(Vi[j], V'[j])$
    * Delivery condition (Causal):
        1.  $Vm[i] = Vj[i] + 1$
        2.  $Vm[k] \le Vj[k]$ for all $k \neq i$

---

## 3. Multicast
Guarantees: Integrity, Validity, Agreement.
* **Ordering:** FIFO, Causal, Total Order (Atomic).
* **ISIS Algorithm (Total Order):** Sender collects proposed timestamps from group, picks max, and finalizes.

---

## 4. Consensus
Agreement on a single value (or sequence).
* **Byzantine Generals Problem:**
    * **Impossibility:** Consensus is impossible if $n \le 3f$ (or $f \ge n/3$).
    * **Requirement:** $n > 3f$ (e.g., $n=4$ for $f=1$).
* **Synchronous Crash Failures:**
    * Requires **$f + 1$ rounds** of communication in worst case.
* **Paxos (Asynchronous):**
    * Roles: Proposers, Acceptors, Learners.
    * Resilient to $(n-1)/2$ crash failures.
    * **Majority Rule:** $Promise$ and $Accept$ phases require majority quorum.

---

## 5. Distributed Mutual Exclusion
Ensuring exclusive access to a Critical Section (CS).

| Algorithm | Type | Message Complexity | Synch Delay | Notes |
| :--- | :--- | :--- | :--- | :--- |
| **Centralized** | Coordinator | **3** | 2 delays | SPOF |
| **Token Ring** | Logical Ring | **1 to $\infty$** | 1 to $N$ | Token loss issues |
| **Ricart & Agrawala** | Timestamped | **$2(n-1)$** | 1 delay | $N-1$ Requests + $N-1$ Replies |
| **Maekawa** | Voting Set (Quorum) | **$3\sqrt{n}$** | 2 delays | Quorum size $\approx \sqrt{n}$ |

* **Maekawa Voting Set ($V_i$):**
    * $p_i \in V_i$
    * $\forall i,j : V_i \cap V_j \neq \emptyset$

---

## 6. Replication
* **CAP Theorem:** Cannot simultaneously guarantee **C**onsistency, **A**vailability, and **P**artition Tolerance.
* **Consistency Models:**
    * **Linearizability:** Real-time ordering preserved.
    * **Sequential:** Process order preserved.
    * **Eventual:** Convergence over time.
* **Leader Election:**
    * **Bully Algorithm:** Best case $N-2$ messages; Worst case $O(N^2)$.
    * **Chang & Roberts (Ring):** Worst case $3N - 1$.

---

## 7. Internet of Things (IoT)
Evolution: Embedded Systems $\rightarrow$ WSN $\rightarrow$ CPS $\rightarrow$ IoT.

### Protocols
* **IEEE 802.15.4:** PHY/MAC layer. Superframe = $15ms \cdot 2^n$ ($0 \le n \le 14$).
* **6LoWPAN:** IPv6 over 802.15.4. Compressed headers (40 bytes $\to$ ~4 bytes).

### Zigbee Address Assignment
Distributed addressing in a tree topology (defined by $Cm, Rm, Lm$).
* **Cskip(d):** Address block size at depth $d$.
* **N-th Child Router ($1 \le n \le Rm$):**
    $$A_{parent} + (n - 1) \times Cskip(d) + 1$$
* **N-th End Device:**
    $$A_{parent} + Rm \times Cskip(d) + n$$

### Routing
* **AODV:** On-demand, uses routing tables.
* **Pastry (P2P/IoT):** Prefix routing. Target ID matched by longest prefix. Hops $\approx \log_{16} N$.

---

## 8. Cryptography & Blockchain
Decentralized, immutable ledger.

### Hashing
* **Properties:** Pre-image resistance, Second pre-image resistance, Collision resistance.
* **Chained Hashes:** To verify data chunks $c_1 \dots c_k$:
    $$h_{1}=H(c_{1}), \quad h_{i}=H(c_{i}||h_{i-1})$$

### Blockchain Mechanics
* **Merkle Tree:** Binary tree of hashes. Root hash validates entire block content. Cost to find root $\approx 2N$ hash evaluations.
* **Proof of Work (PoW):**
    * Find nonce such that $H(header + nonce)$ starts with $n$ zeros.
    * Protects against Sybil attacks and double spending.

---

## 9. Distributed Algorithms (Graph)
* **Symmetry Breaking:** Nodes pick random ID from range $\{1, \dots, r\}$. Prob of collision $1/r$.
* **Luby’s MIS Algorithm (Maximal Independent Set):**
    * Comparison round: Choose random value $\underline{val} = rand(1, n^5)$.
    * If $\underline{val} >$ all neighbors, node joins MIS.
* **Coloring:** $color = \min\_allowed(forbidden)$.