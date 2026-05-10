[00:00:01]  
**Introduction and Recap of Previous Lecture**  
- The lecture begins with a brief recap of the last class focusing on concurrency control protocols in database systems, specifically the **ACID properties** with emphasis on **isolation**.  
- Two main forms of **serializability** were highlighted:  
  - **Conflict serializability**: Common in systems supporting serializable transactions. Verified via dependency graphs without cycles.  
  - **View serializability**: A more general form that allows some conflicts if they align with application semantics; however, no system supports this automatically due to its complexity.  
- The prior discussion assumed static schedules (all transaction operations known ahead), which is unrealistic in real systems where transactions are dynamic and unpredictable.  

[00:03:16]  
**Introduction to Lock-Based Concurrency Control**  
- To handle dynamic transactions, the lecture introduces **locks** as a mechanism to ensure **conflict serializability** during concurrent transaction execution.  
- Distinction between **locks** and **latches**:  
  - Locks protect higher-level database objects (tables, tuples, databases) and are held for the entire lifetime of a transaction (until commit or abort).  
  - Latches are short-term, lightweight synchronization primitives used inside data structures (e.g., indexes), held only briefly.  
- A **centralized lock manager** maintains metadata about locks held and requested by transactions, unlike latches embedded within data structures themselves.  
- Deadlock handling is necessary in locking systems, requiring detection or prevention mechanisms, unlike latches where careful programming avoids deadlocks.  

[00:06:19]  
**Basic Locking Example and Lock Manager Functionality**  
- Example with two transactions (T1 and T2) locking the same object A shows how the lock manager grants or denies locks, forcing transactions to wait if the lock is held exclusively by another.  
- Transactions request locks via the lock manager before reading or writing. Locks are released back to the manager when done or at commit/abort.  
- This basic locking mechanism allows interleaving of operations but requires careful protocols to avoid problems such as deadlocks and anomalies.  

[00:08:29]  
**Lock Types and Compatibility**  
- Locks have types analogous to latches but renamed:  
  - **Shared (S) locks**: Allow multiple concurrent holders (for read operations).  
  - **Exclusive (X) locks**: Only one holder allowed, typically for writes.  
- Compatibility matrix:  
  | Lock Type Held | Requesting Shared Lock | Requesting Exclusive Lock |  
  |----------------|------------------------|---------------------------|  
  | Shared         | Compatible             | Not Compatible            |  
  | Exclusive      | Not Compatible         | Not Compatible            |  
- Real systems have more complex lock types (e.g., intention locks, row exclusives) which extend this basic model but the essential principles remain the same.  

[00:10:15]  
**Lock Acquisition, Upgrades, and Lock Manager Metadata**  
- Transactions can **upgrade** a shared lock to exclusive if needed (e.g., reading then writing).  
- The lock manager maintains a hash table tracking:  
  - Which locks are held by which transactions.  
  - Queues of waiting transactions for each lock.  
- Lock acquisition and release are dynamic; transactions may block if locks are unavailable.  
- In practice, locks are not manually acquired or released by SQL users; systems handle this implicitly based on query semantics.  

[00:12:38]  
**Example: Unrepeatable Read and Limitations of Basic Locking**  
- Example shows that simply acquiring and releasing locks without protocol can lead to **unrepeatable reads**, where a transaction reads a different value for the same data item during its execution due to concurrent writes by other transactions.  
- This demonstrates that locks alone do not guarantee serializability unless used with a protocol defining when locks can be acquired and released.  

[00:13:13]  
**Two-Phase Locking (2PL) Protocol**  
- Introduced as the primary protocol to guarantee **conflict serializability** in dynamic transaction environments.  
- Two phases per transaction:  
  1. **Growing phase**: Transaction acquires all needed locks but does not release any.  
  2. **Shrinking phase**: After releasing the first lock, the transaction cannot acquire any new locks; it only releases locks.  
- Violating the 2PL protocol (e.g., releasing then reacquiring locks) can lead to anomalies such as unrepeatable reads.  
- In actual SQL systems, locks are acquired automatically based on query execution, not explicitly by users.  

[00:15:58]  
**2PL Example and Ensuring Serializability**  
- Example revisited with proper 2PL: T1 holds exclusive lock on A until it finishes, blocking T2 from acquiring the lock and ensuring serializable execution order.  
- Once T1 releases the lock (enters shrinking phase), T2 can acquire the lock and proceed.  
- This avoids anomalies by preventing interleaved access that breaks serializability.  

[00:17:45]  
**Cascading Aborts Problem and Its Impact**  
- Even with 2PL guaranteeing serializability, if transactions release locks early (before commit), **cascading aborts** can occur:  
  - T2 reads data written by T1 before T1 commits.  
  - If T1 aborts, T2 must also abort to maintain correctness, potentially causing a chain reaction.  
- Cascading aborts degrade performance and waste computational resources but do not violate correctness.  
- Metadata is needed to track which transactions depend on others to enforce cascading aborts.  

[00:27:23]  
**Strong Strict Two-Phase Locking (SS2PL)**  
- To prevent cascading aborts, most systems implement **Strong Strict 2PL**, where:  
  - All locks (shared and exclusive) are held **until commit or abort**.  
  - Shrinking phase happens only at the very end of the transaction.  
- This prevents other transactions from reading uncommitted data, eliminating dirty reads and cascading aborts.  
- A less strict variant, **Strict 2PL**, allows releasing exclusive locks before commit but holds shared locks until commit, which still risks cascading aborts.  
- SS2PL is the standard for serializable transaction execution in modern DBMS.  

[00:31:13]  
**Example: Bank Account Transfer with Different Locking Protocols**  
- Scenario: Transfer $100 between accounts and compute their sum concurrently.  
- Without 2PL, anomalies occur (missing $100 in the sum).  
- With normal 2PL, some anomalies persist due to early lock releases.  
- With SS2PL, full serializability and correctness are ensured as locks are held until commit, preventing concurrent inconsistent reads.  

[00:35:33]  
**Deadlocks and Their Detection/Prevention**  
- **Deadlocks** occur when transactions wait indefinitely for locks held by each other (cycles in wait-for graph).  
- Two primary approaches:  
  - **Deadlock detection**: Periodically check the wait-for graph for cycles and abort one transaction to break deadlock.  
  - **Deadlock prevention**: Use protocols to prevent cycles by ordering lock acquisition or aborting transactions before deadlock forms.  
- Detection involves maintaining a **wait-for graph** in the lock manager, representing lock dependencies.  
- Detection frequency is a tradeoff between resource use and responsiveness.  

[00:40:55]  
**Deadlock Resolution and Victim Selection**  
- Upon detecting a deadlock, the system aborts one transaction (victim) to break the cycle.  
- Criteria for victim selection include:  
  - Age of transaction (newest often chosen to minimize starvation).  
  - Amount of work done (minimize wasted effort by aborting transaction with less work).  
  - Number of locks held.  
  - Impact on cascading aborts (if allowed).  
- High-end systems use sophisticated heuristics; simpler systems may pick victims arbitrarily (e.g., newest transaction).  

[00:44:08]  
**Savepoints and Partial Rollbacks**  
- Instead of aborting entire transactions on deadlock, **savepoints** allow rolling back to an intermediate point within a transaction, reducing wasted work.  
- Savepoints are application-defined checkpoints; rollback to savepoint undoes partial work without aborting the entire transaction.  
- Useful in fine-grained lock acquisition scenarios (e.g., updating multiple records sequentially).  

[00:46:38]  
**Deadlock Prevention Protocols: Wait-Die and Wound-Wait**  
- Both use transaction timestamps to enforce strict ordering and prevent cycles:  
  - **Wait-Die**: Older transaction waits for younger; younger aborts if it tries to wait for older.  
  - **Wound-Wait**: Younger transaction waits for older; older aborts (wounds) younger if it tries to wait for younger.  
- By restricting wait directions according to timestamps, cycles (deadlocks) cannot form.  
- Systems choose one protocol based on workload characteristics; some allow runtime switching.  

[00:51:12]  
**Lock Granularity and Hierarchical Locking**  
- Locking every single tuple individually is costly and not scalable for large transactions.  
- Introduce **lock granularity hierarchy**:  
  - Database → Table → Page → Tuple → Attribute  
- Acquiring a coarse-grained lock (e.g., on a table) implicitly locks all finer-grained objects beneath it, reducing lock manager overhead.  
- Tradeoff: Coarse locks reduce concurrency; fine locks increase overhead but improve parallelism.  
- Examples:  
  - Early MongoDB used only database-level locks.  
  - Most modern systems support tuple-level locks for fine concurrency.  
  - Attribute-level locks are rare (e.g., YugabyteDB).  

[00:55:28]  
**Intention Locks to Support Lock Hierarchies**  
- Simple shared/exclusive locks insufficient to efficiently manage hierarchical locking.  
- **Intention locks** provide hints about lower-level locks a transaction intends to acquire, enabling compatibility checks at higher levels without exhaustive traversal.  
- Types of intention locks:  
  | Lock Type                 | Meaning                                         | Compatibility                          |  
  |---------------------------|------------------------------------------------|---------------------------------------|  
  | Intention Shared (IS)      | Intends to acquire shared locks below           | Compatible with all except Exclusive  |  
  | Intention Exclusive (IX)   | Intends to acquire exclusive locks below        | Compatible with IS but not Exclusive  |  
  | Shared Intention Exclusive (SIX) | Holds shared lock at current level, exclusive below | Blocks Exclusive and IS               |  
- Lock acquisition proceeds top-down: acquire appropriate intention locks at higher levels, then explicit locks at leaf nodes.  

[00:58:13]  
**Hierarchical Locking Examples**  
- Reading a tuple: acquire IS lock at table level and shared lock at tuple level.  
- Updating a tuple: acquire IX lock at table level and exclusive lock at tuple level.  
- Multiple transactions can run concurrently if their lock modes and intentions are compatible.  
- Examples show how concurrent readers and writers can coexist with intention locks preventing unnecessary blocking.  

[01:03:50]  
**Schema Changes and Locking**  
- For schema modifications (e.g., adding/dropping columns), exclusive locks at table level are acquired to block concurrent access, ensuring safe metadata changes.  

[01:04:24]  
**Lock Acquisition in SQL and Lock Hints**  
- SQL queries do not manually acquire locks; the DBMS infers lock modes based on query semantics.  
- However, SQL allows **lock hints** such as `FOR UPDATE` to explicitly request exclusive locks during reads when an update is imminent, avoiding costly lock upgrades later.  
- Variants exist (e.g., `FOR SHARE`, `LOCK IN SHARE MODE`) for finer control depending on the DBMS.  

[01:07:14]  
**Skip Locked Hint for Queues and Workloads**  
- `SKIP LOCKED` is a hint supported by some DBMS (e.g., PostgreSQL) allowing reads to ignore locked rows, useful for work queues where tasks held by other workers are skipped to avoid waiting.  
- This can lead to non-repeatable and inconsistent reads but is acceptable for some workloads prioritizing throughput over strict consistency.  

[01:09:04]  
**Summary of Two-Phase Locking Usage and Variants**  
- Variants of 2PL (basic, strict, strong strict) are widely used in all major DBMSs.  
- Locking protocols ensure correctness (serializability) but may affect performance due to blocking and deadlocks.  
- Next lectures will cover **optimistic concurrency control** and **multiversion concurrency control**.  

[01:10:28]  
**Guest Speaker: Ben from Firebolt – Industry Perspective on Query Planning and Cardinality Estimates**  
- Firebolt is an analytical, scale-out database optimized for large-scale, mission-critical real-time applications with high concurrency and low latency.  
- Workloads: Typically very **predictable and repetitive query patterns**, e.g., dashboards queried repeatedly by many users.  
- Predictable query plans and consistent performance are more important than finding the absolute optimal plan.  
- Query performance unpredictability is mainly caused by:  
  - Changes in ingestion or query patterns (*Not specified/Uncertain, usually stable*).  
  - Load spikes, handled by autoscaling and queuing systems.  
  - Overengineering query optimizers causing unpredictable plans.  

[01:16:22]  
**Firebolt's Query Planner Architecture**  
- Postgres-compliant SQL dialect but with a completely custom C++ planner implementation.  
- Uses over 170 rule-based optimizations (filter pushdown, redundant join removal, expression simplification).  
- Cost-based join reordering uses dynamic programming and join graphs.  
- Full support for **Iceberg tables**, a modern table format for analytical workloads.  

[01:18:20]  
**Avoiding Reliance on Cardinality Estimates**  
- Firebolt prioritizes **predictable query plans** over perfect but potentially unstable plans.  
- Users can override planner decisions (fix join orders, aggregation strategies) for production stability.  
- Cardinality estimates are only used for join ordering, where they have large impact (up to 1000x).  
- Elsewhere, cardinality estimates are avoided as they often degrade predictability.  

[01:19:18]  
**Challenges with Cardinality Estimates on Iceberg Tables**  
- Iceberg metadata often provides only basic statistics (e.g., row counts).  
- Example join with tables of vastly different sizes (5 billion vs. 70,000 rows) shows how naive cardinality formulas can misestimate selectivity, causing poor plan choices and "plan flapping."  

[01:20:55]  
**Propagating Cardinality Bounds to Reduce Risk**  
- Firebolt propagates **guaranteed lower and upper bounds** through the query plan to capture uncertainty.  
- These bounds help in join ordering by minimizing risk: ensure the smaller table is on the build side for hash joins, even if exact cardinalities are uncertain.  
- This approach improves plan stability and reliability for production workloads.  

[01:22:24]  
**Closing Remarks and Contact Invitation**  
- The guest encourages questions and offers further discussion on Firebolt’s query planner innovations.  
- Highlights the importance of balancing optimization power with predictable, stable query performance in real-world systems.  

---

### **Key Concepts and Definitions**

| Term                        | Definition                                                                                         |
|-----------------------------|--------------------------------------------------------------------------------------------------|
| Conflict Serializability     | A schedule of transactions is conflict serializable if its dependency graph has no cycles.       |
| View Serializability         | More general serializability allowing some conflicts based on application semantics.             |
| Lock                        | A synchronization mechanism protecting database objects during transactions, held until commit. |
| Latch                       | Lightweight, short-term synchronization primitive within data structures.                        |
| Two-Phase Locking (2PL)      | Protocol with growing (lock acquiring) and shrinking (lock releasing) phases to ensure serializability. |
| Strong Strict 2PL (SS2PL)    | Variant of 2PL where all locks are held until commit/abort to prevent cascading aborts.          |
| Deadlock                    | Cycle of transactions waiting for each other's locks, causing indefinite blocking.              |
| Intention Locks              | Locks used to indicate intention to acquire lower-level locks in a hierarchy.                    |
| Wait-Die / Wound-Wait        | Deadlock prevention protocols using transaction timestamps to avoid cycles in wait-for graph.   |
| Savepoint                   | Checkpoint within a transaction allowing partial rollback.                                      |
| Cardinality Estimate          | Database optimizer’s prediction of the number of rows produced by a query or operation.         |
| Plan Flapping                | Unstable query plans caused by fluctuating cardinality estimates or statistics.                  |

---

### **Summary**

This lecture provided a comprehensive overview of **lock-based concurrency control** in database systems, focusing on the **two-phase locking protocol (2PL)** and its variants that guarantee **serializability** for dynamic transaction workloads. Key issues such as **deadlocks**, **cascading aborts**, and **lock granularity** were addressed, along with practical mechanisms like **intention locks** and **lock hierarchies** to improve scalability and concurrency.

A detailed explanation of **deadlock detection and prevention** mechanisms highlighted the complexity of managing concurrent transactions efficiently, including victim selection strategies and the use of **savepoints** for partial rollbacks.

The lecture concluded with a guest industry perspective from a Firebolt engineer focused on the challenges of **query optimization** in large-scale analytical databases. The talk emphasized the tradeoff between optimizing for perfect query plans versus maintaining **predictable, stable query performance**, especially in the face of uncertain or incomplete **cardinality estimates**, and presented innovative approaches such as propagating cardinality bounds to minimize risk in join ordering.

Overall, the session bridged theoretical concurrency protocols with real-world system design and optimization challenges, offering a deep understanding of how databases maintain correctness and performance amid complex workloads.
