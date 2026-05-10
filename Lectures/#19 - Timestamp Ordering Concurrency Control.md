[00:00:01]  
**Introduction and Administrative Announcements**  
- The session opens with casual remarks and announcements about upcoming course deadlines and events.  
- **Project 3** deadline is on **Saturday the 15th**, with office hours scheduled from 3:00 to 5:00 p.m. in Gates.  
- **Homework 5** deadline extended to the **23rd**.  
- Students are advised to post questions on Piazza or attend office hours for help.  
- A global latch can be used as a workaround if buffer pool managers or B+ trees from previous projects are not thread-safe.  
- Upcoming guest database talks are announced, including co-founders from Moon Cake and DBT, with some events providing pizza and info sessions on hiring.  
- Clarification is made that the Firebolt talk was rescheduled.

[00:03:12]  
**Recap of Two-Phase Locking (2PL) and Introduction to Optimistic Concurrency Control (OCC)**  
- Last class covered **Two-Phase Locking (2PL)**, a **pessimistic concurrency control protocol** ensuring serializability by acquiring locks in different modes before accessing data.  
- Most systems use **strict 2PL**, holding all locks until transaction commit.  
- 2PL assumes transactions will conflict and thus enforces locking to prevent conflicts upfront.  
- However, many transactions are fast and mostly non-conflicting (e.g., web page loads in milliseconds).  
- This motivates **Optimistic Protocols**, which assume conflicts are rare, allowing transactions to run without locks and validating conflicts only at commit time.  
- These protocols fall under **timestamp ordering concurrency protocols**, using timestamps to establish serializable transaction order.  

[00:05:30]  
**Timestamp Ordering and Maintaining Timestamps on Database Objects**  
- Instead of locks, timestamps track the ordering of transactions and database objects.  
- Each transaction is assigned a **unique, monotonically increasing timestamp** at commit time (not at start).  
- Database objects (tuples, pages, tables) maintain timestamps indicating last access or modification times.  
- These timestamps help enforce a serial order equivalent to executing transactions one after another.  
- Use of **UTC** for timestamps is recommended to avoid issues with daylight savings and clock rollback.  
- Timestamp sources include system clocks, atomic clocks, GPS satellites, or logical counters; each has pros and cons.  
- PostgreSQL uses 32-bit timestamps, which can wrap around, causing additional complexity.

[00:10:53]  
**Overview of the Optimistic Concurrency Control (OCC) Protocol**  
- OCC was introduced in 1981 at CMU by H. T. Kong, originally from a networking background.  
- OCC divides transaction execution into three phases:  
  1. **Read Phase:** Transaction reads and writes to a **private workspace** (in-memory copy or scratch area). Writes are buffered here; reads may be copied to ensure repeatable reads.  
  2. **Validation Phase:** On commit, the system validates whether the transaction can commit without violating serializability by checking conflicts with other transactions. The transaction receives its timestamp here.  
  3. **Write Phase:** If validation passes, the buffered writes are atomically applied to the global database, and timestamps are updated on modified objects.  
- The private workspace allows concurrent execution without immediate locking or blocking.  

[00:16:04]  
**Example Schedule of Two Transactions (T1 and T2) Under OCC**  
- T1 reads and writes key A; T2 reads key A.  
- Both transactions create private workspaces and copy data from the global database on reads.  
- T2 commits first, gets timestamp 1, no conflicts detected, so allowed to commit; no writes to apply since it only read.  
- T1 writes to A in its workspace, sets write timestamp to infinity (unknown yet), reads A again for repeatable reads.  
- T1 commits later, gets timestamp 2, validation passes (no conflicting reads/writes), applies changes with timestamp 2 to global database.  
- Logical commit order differs from physical start order, demonstrating flexibility and correctness of OCC.

[00:19:13]  
**Why Assign Timestamp at Validation Instead of Start?**  
- Assigning timestamps at commit (validation) rather than start allows flexible logical ordering, independent of physical start time, enhancing parallelism.  
- This accommodates scenarios where a transaction that started earlier may commit later and vice versa, maintaining serializability.  

[00:21:33]  
**Memory and Atomicity Considerations in OCC**  
- Concerns about memory overhead from copying large data sets to private workspaces arise; this is mitigated in multi-version concurrency control (MVCC) by copying only diffs rather than entire tuples.  
- Atomic commits require mechanisms such as global locks or sophisticated concurrency controls to apply changes atomically; earlier OCC implementations used a global latch due to single-core CPU limitations.  
- Modern systems use more advanced techniques for concurrency and atomicity.

[00:24:39]  
**Validation Phase: The Core of OCC**  
- Validation ensures transaction commit order is equivalent to a serial order.  
- Two main validation strategies:  
  - **Forward Validation:** Checks conflicts against currently active transactions’ read/write sets.  
  - **Backward Validation:** Checks conflicts against transactions that have already committed.  
- Backward validation is more common and easier to implement.  

[00:27:21]  
**Forward Validation Detailed Conditions for Commit**  
For a transaction T1 trying to commit, with timestamp TS(T1), and another active transaction T2:  
- Condition 1: If TS(T1) &lt; TS(T2) and T2 has not started, T1 can commit safely.  
- Condition 2: If T1 completes write phase before T2 starts write phase and T1 does not modify anything T2 read, T1 can commit.  
- Condition 3: If the intersection of T1’s write set and T2’s read set is empty, no conflict exists, safe to commit.  
- If conflicts exist, transaction must abort to maintain serializability.

[00:30:20]  
**Example of Conflict Leading to Abort in OCC**  
- T1 writes to key A; T2 reads key A before T1 commits.  
- If T1 tries to commit after T2 read outdated data, T1 must abort because committing would violate serializability.  
- It is simpler for the transaction detecting conflict to abort itself rather than abort others, especially in concurrent systems.  

[00:34:06]  
**Contention and Trade-Offs Between OCC and 2PL**  
- OCC performs well under low contention; many read-only or non-conflicting transactions commit quickly.  
- Under high contention, OCC suffers from wasted work due to aborts at commit time; 2PL may perform better by blocking early.  
- Systems may attempt to dynamically switch between protocols based on workload, but this is complex and rarely implemented.  
- Under extreme contention, both protocols approach serial execution performance.  

[00:39:56]  
**Backward Validation Advantages**  
- Easier to implement and less interference with other running transactions since it involves only checking committed transactions.  
- Common in many modern OCC implementations and commercial systems including Amazon’s DSQL.

[00:41:02]  
**Challenges of Validation: Atomicity and Concurrency**  
- Serial validation (one transaction validating at a time) simplifies correctness but limits concurrency.  
- Parallel validation requires careful lock acquisition order on read/write sets to avoid deadlocks and race conditions.  
- Practical systems implement heuristics and locking protocols to manage concurrent validation safely.  

[00:46:25]  
**Summary of OCC Benefits and Drawbacks**  
- OCC is well-suited for workloads with low conflicts and many read-only transactions.  
- Memory overhead from copying entire tuples is a drawback; MVCC can reduce this with diffs.  
- High contention workloads favor pessimistic approaches like 2PL due to reduced wasted work.  
- Repeatable reads and other isolation guarantees are addressed in MVCC and extensions.  

[00:48:42]  
**Handling Inserts and Deletes: The Phantom Problem**  
- Until now, discussions focused on reads and updates; inserts and deletes complicate concurrency control due to **phantom reads**.  
- Phantom reads occur when range queries run twice and see different sets of tuples due to concurrent inserts or deletes.  
- Both 2PL and OCC can suffer from phantoms if not properly handled.  
- Example: T1 counts rows with status=paid, T2 inserts a new paid row and commits, then T1’s repeated count changes, violating serializability.  

[00:53:05]  
**Approaches to Phantom Problem**  
1. **Lock Everything:** Lock entire tables or pages to prevent concurrent modifications; simple but reduces concurrency drastically.  
2. **Re-execute Scans on Commit:** Re-run queries and verify results haven’t changed during transaction execution; expensive but precise.  
3. **Predicate Locking:** Lock logical predicates (conditions) rather than individual tuples; theoretical but complex and hard to implement.  
4. **Index Locking:** Use index structures to lock ranges and gaps efficiently; practical and commonly used approach.  

[00:55:35]  
**Details on Re-execution and Reverse Approach**  
- Some systems (e.g., Hecaton) re-execute queries at commit time to verify correctness.  
- Others (e.g., DynamoDB, Fauna) run “reconnaissance” or phantom queries before committing real transactions to check for conflicts.  
- Both approaches maintain serializability by ensuring read sets remain consistent.

[00:58:07]  
**Dirty Reads and Serializability in Conflicting Transactions**  
- If T1 updates many items and T2 reads a modified item before T1 commits, T2’s commit must wait to maintain serializability.  
- Dirty reads occur if T2 reads uncommitted data; strict serializability forbids this.  

[00:58:38]  
**Predicate Locking Concept**  
- Locks on predicates correspond to regions in a multidimensional space representing query conditions.  
- Transactions acquire locks on predicate regions; overlapping predicates represent conflicts.  
- Exact predicate locking is complex and computationally expensive, especially with many columns and complex predicates.  

[01:00:08]  
**Index Locking and Gap Locks**  
- Index locking is a practical approximation of predicate locking using existing B+ tree indexes.  
- Locks can be taken on exact keys or on **gap locks**, which cover ranges between keys to prevent phantom inserts.  
- Gap locks cover intervals and prevent inserts into these gaps while held.  
- Locks are tracked in lock tables separately from the physical data structures (locks vs. latches).  

[01:06:24]  
**Hierarchical Locking in Indexes**  
- Locks can be hierarchical: intention locks on pages and exclusive locks on key ranges or gaps within pages.  
- This allows concurrent transactions to lock disjoint keys or ranges safely.  
- This mechanism prevents phantom reads by controlling inserts into locked ranges.

[01:10:37]  
**Impact of Missing Indexes**  
- Without indexes, locking ranges becomes expensive and may require full table scans or full table locks, reducing concurrency and performance.  
- Indexes are critical for efficient phantom prevention and serializability enforcement.  

[01:12:32]  
**Isolation Levels Beyond Serializability**  
- Most databases do **not provide serializability by default** due to cost and complexity.  
- Oracle, SQL Server, MySQL, PostgreSQL default to lower isolation levels like **repeatable read** or **read committed**.  
- Isolation levels define which anomalies (dirty reads, non-repeatable reads, phantoms) can occur.  
- The SQL standard defines four main levels:  
  | Isolation Level    | Guarantees                                  | Possible Anomalies                  |  
  |--------------------|---------------------------------------------|-----------------------------------|  
  | Serializable       | Full serializability; no anomalies          | None                              |  
  | Repeatable Read    | No dirty reads, no non-repeatable reads     | Phantoms possible                  |  
  | Read Committed     | No dirty reads                               | Non-repeatable reads, phantoms possible |  
  | Read Uncommitted   | Dirty reads possible                         | All anomalies possible            |  

[01:16:09]  
**Implementation of Isolation Levels in 2PL and OCC**  
- Serializable requires strict 2PL plus phantom protection (index locks, predicate locks, or re-execution).  
- Repeatable read omits phantom protection but holds read locks until commit.  
- Read committed releases read locks immediately, allowing non-repeatable reads and phantoms.  
- OCC similarly can be tuned by relaxing validation or locking requirements.  

[01:18:34]  
**Isolation Level Defaults and Industry Practice**  
- Survey of production databases shows most use **read committed** or **repeatable read** by default.  
- Serializable is rare due to performance overhead.  
- Snapshot Isolation (SI), a multi-version concurrency technique, is also common but has anomalies distinct from 2PL-based serializability.  

[01:20:52]  
**Examples of Systems Supporting Serializable Isolation**  
- Ingres, KroDB, Spanner, and some distributed databases provide serializability by default or optionally.  
- Google Spanner implements **strict serializability** (external consistency), meaning transactions commit in the order they are received, using synchronized clocks.  

[01:22:33]  
**Additional Isolation Levels and Cursor Stability**  
- Other isolation levels such as **cursor stability** exist, e.g., IBM DB2’s approach holding locks on data being scanned by cursors.  
- These provide consistency within a scan but may allow anomalies between scans.

[01:23:32]  
**Final Summary and Takeaways**  
- Two main concurrency control categories: **pessimistic (2PL)** and **optimistic (OCC)**, each with trade-offs.  
- No one protocol is best under all workloads; choice depends on contention and transaction characteristics.  
- Multi-version concurrency control (MVCC), to be discussed next, combines these ideas and reduces overhead by maintaining multiple versions of data with timestamp metadata.  
- Real systems often combine locking, timestamps, and versioning to achieve practical performance and correctness.

---

### Key Concepts and Definitions

| Term                          | Definition                                                                                   | Notes                                                                                                   |
|-------------------------------|----------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------|
| Two-Phase Locking (2PL)        | Pessimistic concurrency control with lock acquisition before access and release at commit.  | Strict 2PL holds all locks until commit, ensuring serializability.                                      |
| Optimistic Concurrency Control (OCC) | Allows transactions to run without locks, validates conflicts at commit time.              | Uses timestamps and validation phases; best for low contention workloads.                               |
| Timestamp Ordering             | Protocol assigning unique increasing timestamps to transactions to enforce serial order.    | Timestamps assigned at commit time in OCC; physical start order may differ from logical commit order.   |
| Phantom Read                  | Anomalous phenomenon where new rows appear or disappear between repeated query executions.   | Caused by inserts/deletes within a transaction's range queries; phantom prevention requires special locks. |
| Predicate Locking             | Locking based on query predicates rather than individual tuples.                             | Theoretical and complex; approximated by index locking in practice.                                     |
| Gap Lock                     | Lock on a range (gap) between keys in an index to prevent phantom inserts.                   | Used to lock nonexistent keys/ranges efficiently.                                                      |
| Isolation Levels              | Levels of transaction isolation controlling visibility of concurrent transactions’ effects. | Includes Serializable, Repeatable Read, Read Committed, Read Uncommitted; affects anomalies possible.   |
| Snapshot Isolation (SI)        | Multi-version concurrency control providing a consistent snapshot view to each transaction. | Different anomaly profile; not true serializability.                                                   |
| Serializability              | The correctness criterion requiring transaction schedules to be equivalent to some serial order. | The strongest isolation level, ensuring no anomalies occur.                                            |

---

### Summary Table: Transaction Phases in OCC

| Phase         | Description                                                                                  | Key Actions                                   |
|---------------|----------------------------------------------------------------------------------------------|-----------------------------------------------|
| Read Phase    | Transaction reads and writes data in a private workspace; no global changes yet.              | Reads copied for repeatable reads; writes buffered. |
| Validation Phase | Conflict detection and serializability check; timestamp assigned here.                      | Checks for conflicts with other transactions’ read/write sets. |
| Write Phase   | If validation passes, buffered writes are atomically applied to the global database.          | Updates applied with assigned timestamp.     |

---

### Summary Table: Isolation Levels and Anomalies

| Isolation Level     | Dirty Reads | Non-Repeatable Reads | Phantoms   | Implementation Note                       |
|---------------------|-------------|---------------------|------------|-------------------------------------------|
| Serializable        | No          | No                  | No         | Full locking or OCC with phantom prevention |
| Repeatable Read     | No          | No                  | Possible   | Locks held until commit; no phantom locks  |
| Read Committed      | No          | Possible            | Possible   | Locks released immediately after read      |
| Read Uncommitted    | Possible    | Possible            | Possible   | No locks, lowest isolation                  |

---

### Timeline Table: Transaction Example (T1 and T2)

| Timestamp | Transaction Event                         | Description                                            |
|-----------|-----------------------------------------|--------------------------------------------------------|
| Start T1  | T1 begins, creates private workspace    | Reads object A, copies to workspace                    |
| Start T2  | T2 begins, creates private workspace    | Reads object A, copies to workspace                    |
| Commit T2 | T2 enters validation, gets timestamp 1  | No conflicts, commits successfully (read-only)        |
| T1 modifies A | T1 writes new value to A in workspace | Write timestamp set to infinity (unknown yet)         |
| T1 reads A  | T1 reads modified A from workspace     | Ensures repeatable reads                               |
| Commit T1 | T1 enters validation, gets timestamp 2  | Validation passes, applies changes with timestamp 2   |

---

### Frequently Asked Questions (FAQ)

**Q:** Why assign timestamps at validation rather than transaction start?  
**A:** Assigning timestamps at validation allows flexible logical ordering, improving concurrency and avoiding unnecessary serialization constraints.

**Q:** How does OCC handle memory overhead from copying data?  
**A:** OCC can incur overhead copying entire tuples; MVCC optimizes by copying only changes (diffs), reducing memory and CPU usage.

**Q:** What is the phantom problem and why is it hard?  
**A:** Phantoms occur when inserts/deletes affect range queries between reads in a transaction, violating serializability. Preventing phantoms requires locking or validation strategies on ranges or predicates, which is complex and costly.

**Q:** Which validation approach is more common, forward or backward?  
**A:** Backward validation is more common due to simpler implementation and less interference with concurrent transactions.

**Q:** Why don't most databases default to serializable isolation?  
**A:** Serializable isolation is expensive and can degrade performance; most systems default to lower isolation levels balancing correctness and efficiency.

**Q:** How does index locking prevent phantoms?  
**A:** By locking key ranges and gaps in indexes, the system prevents inserts or deletes in those ranges during a transaction, thus avoiding phantom anomalies.

---

**Overall, the lecture thoroughly explains the foundations and trade-offs of concurrency control in databases, focusing on timestamp-based optimistic concurrency control, its phases, validation mechanisms, and challenges such as phantom reads. It also situates these concepts within practical database systems and their default isolation levels, preparing for future discussions on multi-version concurrency control.**
