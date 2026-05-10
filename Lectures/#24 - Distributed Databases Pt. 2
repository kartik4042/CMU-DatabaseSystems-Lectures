[00:00:01] **Course Announcements and Exam Logistics**  
- Final exam is on Thursday, December 11th in the student center auditorium.  
- Homework 6 and Project 4 are both due Sunday.  
- Saturday office hours scheduled for December 6th, 3–5 PM at Gates 5th floor.  
- A practice final exam and study guide will be posted.  
- Two guest talks scheduled:  
  - Today at 4:30 PM: Snowflake team on Apache Polaris (Snowflake’s open-source reimplementation of Iceberg catalog API).  
  - Next week: Apache Flink team presenting their system.  
- Instructor encourages students interested in TA positions next semester to sign up.  
- Maximum project 4 bonus is 25 points (to be confirmed).  

[00:03:19] **Recap of Distributed Database Architectures**  
- Two main architectures:  
  - **Shared-Nothing:** Each node stores part of the data locally; nodes communicate via TCP/UDP.  
  - **Shared-Disk:** All nodes access a common external storage system (e.g., Amazon S3, GCS).  
- **Horizontal Partitioning (Sharding):** Data is split into disjoint subsets by keys, allowing queries to be routed to a single partition.  
- **Replication:** Two models: primary replica and multi-primary.  
- **Transaction Coordination:**  
  - Centralized coordinator (middleware or TP monitor).  
  - Decentralized model where nodes coordinate commits themselves.  
- The lecture separates focus into OLTP (online transaction processing) and OLAP (online analytical processing) systems:  
  - OLTP: Small, atomic transactions with low latency (target &lt;50 ms).  
  - OLAP: Long-running, complex queries involving joins and aggregations, latency tolerance in seconds to minutes.  
- Although OLTP and OLAP workloads differ, both require transaction coordination and consistency mechanisms.

[00:07:14] **Distributed Transactions and Atomic Commit Protocols**  
- Goal: A distributed database should appear as a single logical database to applications, preserving ACID properties even when data is partitioned or replicated across nodes.  
- Challenges include failure of nodes, message loss, and network delays during commit.  
- Assumption: All nodes are controlled and "well-behaved" (non-malicious). Byzantine fault tolerance (BFT) protocols are not used in typical distributed databases but are relevant for trustless environments like blockchains.  
- Blockchain systems run Byzantine fault tolerant consensus protocols (e.g., BFT) but are much slower and impractical for conventional transactional systems.  
- **Key advice:** Avoid MAPAP (Not specified what MAPAP stands for) and blockchain databases for normal transactional systems due to performance and complexity.  

[00:11:41] **Atomic Commit Protocols Overview**  
- Atomic commit protocols ensure all nodes agree on either committing or aborting a distributed transaction.  
- Also known as consensus protocols or distributed state machines, where transaction commit order defines system state.  
- The first widely used protocol: **Two-Phase Commit (2PC)**, developed by Jim Gray.  
- Variants: Three-phase commit (rarely used, overly complicated), four-phase commit (experimental Microsoft system).  
- 2PC involves two phases:  
  1. **Prepare Phase:** Coordinator sends prepare requests; participants vote "yes" or "no" to commit.  
  2. **Commit Phase:** If all vote "yes," coordinator sends commit; otherwise, abort is sent.  
- Nodes log all messages for durability and recovery.  
- If any participant votes "no," transaction aborts immediately without waiting for other votes.  
- Timeout and node failures cause aborts to guarantee progress (liveness).  
- Participants confirm commit or abort back to coordinator; acknowledgements logged to recover from crashes.  
- If a participant crashes during commit phase, the transaction is considered committed if majority agreed; faulty node recovery involves log replay.  
- If coordinator crashes, participants may abort transaction and elect a new coordinator; system blocks committing new transactions until resolved.  

[00:29:52] **Optimizations of Two-Phase Commit**  
- **Early Prepare Voting:** Participants piggyback commit intent with last query request to reduce messaging rounds; rare in practice.  
- **Early Acknowledgement:** Coordinator acknowledges commit to application after prepare phase (before full commit phase finishes) to reduce latency, assuming low failure probability.  
- Despite optimizations, 2PC blocks progress if any participant is unreachable until timeout.  

[00:32:12] **Consensus Protocols: Paxos, Viewstamped Replication, and Raft**  
- 2PC requires unanimous agreement, blocking if any node fails.  
- Paxos, Viewstamped Replication (1988), Zab (Zookeeper Atomic Broadcast), and Raft are consensus protocols that require only a majority quorum to proceed.  
- These protocols offer non-blocking commit as long as majority of nodes are available.  
- Paxos is notoriously difficult to understand and implement; Leslie Lamport's original 1989 paper is written as an allegory, not an implementation guide.  
- Raft simplifies Paxos and is widely used due to easier implementation and open source availability.  
- 2PC can be seen as a special case of Paxos where all nodes must agree.  
- Paxos roles: proposers, acceptors, and learners (observers).  
- Commit proceeds if majority accept. Failed nodes catch up by reading logs on recovery.  
- If any participant votes abort, the transaction aborts (except in some leader election scenarios).  

[00:39:26] **Integrity Constraints and Commit Protocols**  
- Integrity checks (e.g., uniqueness, not null) must be done before commit.  
- If constraints are checked during commit, the protocol devolves into 2PC requiring unanimous agreement.  
- Systems commonly check constraints during transaction execution or validation phase before commit.  
- Replicas (in multi-primary or sharded systems) must mirror state precisely and follow logs for consistency.  
- Sharding does not inherently increase risk of integrity violations; constraints still apply locally.  

[00:42:34] **Handling Failures and Recovery**  
- Logs (write-ahead logs) record state transitions and messages to enable recovery after crashes.  
- After crash, nodes consult logs and coordinator to determine transaction state (committed, aborted, or unknown).  
- Coordinator crashes generally lead to transaction abort and leader re-election.  
- Participant crashes during commit phase assumed to vote abort to avoid inconsistent states.  
- Recovery involves blocking new commits until uncertain transactions are resolved.  

[00:47:48] **Summary of Consensus Protocol Differences**  

| Protocol         | Agreement Requirement         | Blocking Behavior                 | Fault Tolerance              | Usage Notes                                         |
|------------------|------------------------------|---------------------------------|-----------------------------|----------------------------------------------------|
| Two-Phase Commit | Unanimous (all nodes vote yes) | Blocks until all respond or timeout | Cannot commit if any node down | Early distributed DBs; simple but blocking          |
| Paxos            | Majority quorum              | Non-blocking if majority alive   | Can tolerate minority failures | Complex, used in modern distributed systems         |
| Raft             | Majority quorum              | Non-blocking if majority alive   | Similar to Paxos             | Easier to implement, widely used (e.g., etcd)       |

- Raft leader election favors node with most up-to-date logs.  
- Leader periodically renewed to detect failures and avoid split leadership.  
- Exponential backoff used to resolve leadership contention.  

[00:48:54] **Distributed System Challenges: CAP Theorem**  
- Proposed by Eric Brewer, CAP theorem states distributed systems can only satisfy two of three properties simultaneously:  
  - **Consistency (C):** All nodes see the same data at the same time.  
  - **Availability (A):** Every request receives a response (not necessarily the latest data).  
  - **Partition Tolerance (P):** System continues to operate despite network partitions.  
- Network partitions force a tradeoff between consistency and availability.  
- Traditional relational and distributed SQL systems prioritize **consistency over availability**, rejecting writes when partitions occur.  
- Many NoSQL systems prioritize **availability over consistency**, allowing writes on partitioned nodes and reconciling later (eventual consistency).  

[00:50:56] **Split-Brain Problem and Replication**  
- Example: Primary and replica in two data centers. Network partition causes both to believe they are primary and accept writes independently.  
- When partition heals, reconciling divergent writes is complex.  
- Solutions:  
  - Stop writes during partition (sacrifices availability).  
  - Allow writes and merge conflicts later (complex, eventual consistency).  
- Techniques for reconciliation include:  
  - Last writer wins (based on timestamps).  
  - Vector clocks (track multiple versions; complex).  
- Vector clocks are rarely used due to complexity; last writer wins is common but can cause data loss or inconsistency.  
- Application-level workarounds (e.g., Facebook storing posts in local cookies to mask delays) are common for latency-sensitive systems.

[00:55:58] **Latency vs. Consistency Tradeoffs (PACELC Theorem)**  
- PACELC (2010) extends CAP by incorporating latency:  
  - When Partitioned (P), tradeoff between Availability (A) and Consistency (C).  
  - Else (E), tradeoff between Latency (L) and Consistency (C).  
- Systems must decide how long to wait for acknowledgements from replicas before responding to clients.  
- Waiting for all replicas increases latency but ensures consistency; shorter waits reduce latency but risk inconsistency.  
- Different systems have different timeout settings (e.g., Spanner ~10s, CockroachDB ~5min).  
- Application requirements dictate appropriate balance:  
  - Financial applications demand strong consistency.  
  - Social media tolerates eventual consistency and latency.  

[01:00:15] **Distributed OLAP and Joins**  
- Joins are the most expensive operations in distributed analytical queries, heavily influenced by data partitioning and storage.  
- Naïve approach: gather all data on one node and join, losing parallelism and scalability.  
- Efficient distributed joins aim to minimize data movement and preserve correctness.  
- Three basic join scenarios based on data distribution:  

| Scenario                       | Description                                                                                         | Join Execution Strategy                       |
|-------------------------------|-------------------------------------------------------------------------------------------------|----------------------------------------------|
| Partitioned + Replicated       | One table partitioned on join key; the other replicated on all nodes (small table replication)    | Local joins on partitioned data; union results |
| Both Partitioned on Join Key   | Both tables partitioned on join key with matching partitions/ranges                               | Local joins independently on each node       |
| Unpartitioned or Mispartitioned| Neither table partitioned on join key or partitioning does not match join keys                    | Data reshuffling (broadcast or repartition)  |

- **Broadcast Join:** Small table copied to all nodes; local join performed.  
- **Shuffle Join:** Large tables repartitioned on join key by redistributing data across nodes before join.  
- Broadcast and shuffle joins involve substantial network I/O and are major performance bottlenecks.

[01:08:41] **Semi-Join Optimization**  
- When joining large fact tables with smaller dimension tables, semi-join reduces data movement by sending minimal filtering information (e.g., keys or bloom filters) instead of full data.  
- Semi-join example:  
  - Extract relevant keys from dimension table.  
  - Send keys to fact table nodes to filter records before sending data for join.  
- Supports predicate pushdown and projection to minimize unnecessary data transfer.  
- Some systems expose semi-join as an operator, though not standardized in SQL.  
- Bloom filters are used to compactly represent filter sets and reduce network overhead.

[01:11:47] **Shuffle Phase and Query Execution Pipelines**  
- Shuffle phase redistributes data according to join keys or grouping keys, enabling local computation of joins or aggregations.  
- Shuffle is a pipeline breaker and important for query optimization and parallelism.  
- Query execution modeled as stages:  
  1. Data read and initial processing (e.g., scans, projections).  
  2. Shuffle phase redistributes data across nodes based on partition keys.  
  3. Subsequent stages operate on locally partitioned data.  
- Shuffle allows dynamic scaling of worker nodes for downstream stages.  
- Specialized hardware (e.g., FPGA-based SmartNICs) can accelerate shuffle in large-scale systems like Google BigQuery.  
- Shuffle abstracts data movement, simplifying operator development by making joins and aggregations partition-aware without complex operators.

[01:17:10] **Final Summary and Takeaways**  
- Strong consistency in distributed transactional systems is challenging but necessary to prevent application-level consistency errors.  
- Consensus protocols such as two-phase commit, Paxos, and Raft enable atomic commit across distributed nodes.  
- Raft is currently the most popular consensus protocol due to ease of implementation and availability of open-source tools.  
- Distributed joins require careful data partitioning and movement strategies to optimize performance.  
- Minimizing network data transfer and avoiding false negatives in joins is critical for correctness and efficiency.  
- Next class will review final exam topics and briefly cover advanced analytical query optimization techniques like vectorization and parallel hash joins (not on final exam).
