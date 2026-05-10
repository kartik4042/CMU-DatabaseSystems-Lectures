[00:00:01] **Course Introduction and Project Deadlines**  
- The class begins with a schedule update:  
  - Project 3 due Sunday.  
  - Special office hours Saturday at 3 PM.  
  - Homework 5 extended to the 23rd.  
  - Project 4 released earlier this week, due December 7th.  
  - Recitation schedule to be announced before Thanksgiving.  
- Emphasis on finishing Project 3, passing all tests, merging latest public repo code, then starting Project 4.  
- Questions about join algorithms (naive vs block nested loop join) were addressed as flexible choices.

[00:02:30] **Recap of Optimistic Concurrency Control (OCC) and Transition to Multi-Versioning**  
- OCC assumes conflicts are rare, allowing transactions to proceed without locks, validating only at commit time.  
- Transactions work on private copies of tuples, not the shared database.  
- Upon commit, changes are validated and then applied to the global database.  
- Multi-version concurrency control (MVCC) differs by storing multiple versions directly in the global database, not private workspaces.  
- Transactions see different versions according to visibility rules, which may be invisible to others concurrently.  
- The “phantom read” problem was highlighted: reads over ranges may see inconsistent tuples due to concurrent inserts/deletes, requiring mechanisms like index locks or table locks to prevent anomalies.

[00:04:20] **Introduction to Multi-Version Concurrency Control (MVCC)**  
- MVCC is not a locking or timestamp ordering protocol by itself but a versioning enhancement usable with 2PL, OCC, or other protocols.  
- It enables better parallelism by maintaining multiple physical versions of logical tuples identified by primary keys.  
- Updates never overwrite existing tuples; instead, new physical versions are created for each update.  
- Transactions determine the visible version using metadata stored with each tuple.  
- Over time, old versions become obsolete and require garbage collection to reclaim storage space.  
- MVCC enables “time travel” queries, allowing queries on historical states of the database using timestamps, important for auditing and financial applications.  
- Commercial systems often charge extra for time travel features.  
- Historical context: early implementations date to the 1980s (e.g., Oracle RDB from DEC).

[00:08:23] **Key Properties of MVCC**  
- **Writers do not block readers:** Readers always access the visible version without waiting for writers.  
- **Readers do not block writers:** Writers can create new versions without waiting for readers.  
- Contrast with Two-Phase Locking (2PL), where shared and exclusive locks conflict, causing contention.  
- Index management is complicated under MVCC.

[00:09:45] **Historical Background and Systems Implementing MVCC**  
- MVCC idea dates back to a 1978 MIT dissertation, initially in OS/file systems context.  
- Early database implementations include DEC’s RDB VMS and Interbase by Jim Starkey (co-founder of NDB).  
- Interbase evolved into Firebird (open source) and variants like Red DB.  
- The Firefox browser name reflects a naming conflict with Firebird database.  
- Oracle RDB (from DEC lineage) exists but is distinct from Oracle’s flagship RDBMS product.

[00:12:23] **MVCC Versioning Mechanism Explained via Timestamps**  
- Each tuple stores two timestamps: **begin timestamp** and **end timestamp** representing visibility intervals.  
- Transactions get a start timestamp at begin.  
- A tuple version is visible to a transaction if the transaction’s timestamp falls within the tuple’s begin and end timestamps.  
- The newest version has an infinite/null end timestamp.  
- When a transaction creates a new version (e.g., T2 writes a new A1 at timestamp 2), it sets the begin timestamp to 2 and updates the previous version’s end timestamp to 2, marking its visibility interval.  
- The **end timestamp is essential** to determine visibility boundaries and to know when old versions can be garbage collected.  

[00:16:28] **Transaction Status Table and Repeatable Reads**  
- A **transaction status table** globally tracks active, committed, and aborted transactions with their timestamps.  
- This helps determine which versions are visible or obsolete.  
- Example: T1 and T2 active, T1 reading a consistent snapshot (repeatable read).  
- Once transactions commit, their versions become visible or obsolete, enabling safe garbage collection.

[00:18:07] **Write Conflicts and Aborts Under MVCC**  
- If a transaction tries to write a new version while the previous version’s creating transaction is still active (not committed), a conflict occurs.  
- The second transaction must abort to prevent lost updates.  
- Physically, the database sees all versions but logically ignores uncommitted versions for reads.  
- This behavior is called **Snapshot Isolation**.

[00:22:06] **Snapshot Isolation and Its Guarantees**  
- Snapshot Isolation provides a **consistent snapshot** of data as of the transaction start time, only including committed transactions before that time.  
- Prevents torn writes: partial visibility of multi-tuple updates is avoided.  
- The **“first-writer-wins”** rule applies: if two transactions write the same tuple, the first to commit wins; the other aborts.  
- However, Snapshot Isolation is **not serializable** and can suffer from anomalies like **write skew**.

[00:24:31] **Write Skew Anomaly Example**  
- Illustrated with an example of two concurrent transactions flipping colors of marbles:  
  - Both transactions see the same initial snapshot (2 black, 2 white).  
  - One flips whites to black, the other flips blacks to white simultaneously.  
  - Both commit successfully, resulting in a state impossible under serial execution (2 black, 2 white remain).  
- 2PL would prevent this by locking reads and writes.  
- Snapshot Isolation allows this anomaly due to non-blocking readers and writers.

[00:28:43] **MVCC Design Considerations and Protocol Choices**  
- MVCC can be layered on top of various concurrency protocols (2PL, OCC, timestamp ordering).  
- Important design decisions include:  
  - Which concurrency protocol to use.  
  - How to store versions physically.  
  - Garbage collection strategies.  
  - Index management.  
  - Handling deletes.

[00:30:05] **MVCC with Two-Phase Locking (2PL) Example**  
- With 2PL layered on MVCC, exclusive locks are acquired on the **logical tuple**, not physical versions.  
- Writers block other writers but readers can still read older versions without locking conflicts.  
- Locks are released after commit, allowing queued writers to proceed.  
- This approach preserves serializability with MVCC benefits.

[00:33:15] **Postgres MVCC Demonstration**  
- Postgres stores versions with xmin/xmax transaction IDs for visibility.  
- Two concurrent transactions reading and updating tuples show:  
  - Read-committed isolation:  
    - A transaction sees the latest committed version at its start.  
    - Uncommitted changes by other transactions are invisible.  
  - Writers block each other on the logical tuple exclusive lock.  
  - Commit releases locks allowing subsequent writers to proceed.  
- Postgres stores multiple versions in the same page, which is inefficient but historically inherited.

[00:38:13] **MySQL MVCC Behavior**  
- MySQL uses repeatable read by default, so transactions see the snapshot at their start time, even if newer commits occur.  
- MySQL stores versions differently (row-based, rollback segments), leading to different visibility rules compared to Postgres.  
- Demonstrations show consistent snapshot isolation behavior.

[00:41:14] **Postgres Tuple Metadata Storage**  
- Postgres stores transaction IDs in tuple headers: `xmin` (creating transaction), `xmax` (deleting/invalidating transaction).  
- CTID stores physical location (page and slot).  
- Version visibility depends on these metadata and status in transaction status table.

[00:44:57] **Deadlock Detection in Postgres**  
- Postgres detects deadlocks via timeout-based detection.  
- Example: two transactions waiting on each other’s locks get one aborted after timeout, resolving deadlock.  
- Deadlock detection is vital for serializable isolation level.

[00:46:42] **Postgres Does Not Implement True Read Uncommitted**  
- Postgres treats read uncommitted as read committed internally to prevent dirty reads.  
- Oracle behaves oppositely, offering weaker isolation than requested.  
- MVCC implementations typically do not provide true read uncommitted due to complexity and limited use cases.

[00:49:34] **Reads and Locks in MVCC**  
- Reads are done on logical objects using primary key-level locks (not physical record IDs) because physical locations may move.  
- Reading scans all versions physically but logically filters visible versions based on timestamps and transaction status.  
- Writes require acquiring exclusive locks on logical tuples.

[00:51:44] **Version Storage Techniques**  
- Three main strategies for storing multiple versions:  

| Storage Type    | Description                                                                                  | Example Systems                 | Notes                                      |
|-----------------|----------------------------------------------------------------------------------------------|--------------------------------|--------------------------------------------|
| Append-Only     | New versions appended as full copies to same table space                                     | Postgres                       | Simple but inefficient, requires scanning version chains; Postgres approach. |
| Time Travel     | Old versions stored separately from main table, enabling efficient historical queries        | SQL Server, SAP HANA (formerly) | Separate tables for old versions; easier GC but more complex schema.            |
| Delta Storage   | Store only changes (deltas) for modified columns; overwrite main tuple with latest version   | MySQL, Oracle                  | Most efficient; reduces storage and lookup overhead; current best practice.    |

- Append-only is easiest but least efficient, especially for wide tuples.  
- Delta storage is preferred in modern systems for performance and storage efficiency.  
- Postgres uses append-only; MySQL and Oracle use delta storage.

[00:55:19] **Append-Only Version Chain Management**  
- Version chains are linked lists of tuple versions maintained in the tuple header.  
- Two ordering approaches:  
  - Oldest to newest: index points to oldest version, newer versions linked forward. Requires scanning to find latest visible version.  
  - Newest to oldest: index points to newest version, easier to find latest version but requires updating indexes on inserts.  
- Version chain length impacts performance; longer chains slow reads.

[00:58:14] **Time Travel Storage**  
- Old versions moved to separate “time travel” tables, leaving main table with only latest version.  
- Facilitates garbage collection without locking main table.  
- SAP HANA experimented with version placement but settled on time travel storage.

[00:59:13] **Delta Storage Explained**  
- Only changed columns stored as deltas in a rollback or delta segment.  
- Main table always contains latest version.  
- Old versions reconstructed by applying deltas to latest version in memory.  
- Efficient for large tuples with few modified columns.  
- Allows efficient garbage collection by dropping entire delta segments.

[01:01:09] **Garbage Collection (GC) in MVCC**  
- Old versions no longer visible to any active transaction can be safely removed.  
- Visibility determined by comparing version timestamps with active transactions’ timestamps in the transaction status table.  
- GC must also clean up aborted transaction versions.  
- Three GC approaches:  
  - Tuple-level: scanning tables for obsolete versions, run by background workers or cooperatively during normal transactions.  
  - Cooperative cleaning: transactions clean up old versions encountered during reads.  
  - Transaction-level tracking: track invalidated versions during transactions to know exactly which versions to reclaim after commit.

[01:04:55] **Postgres Vacuuming Optimization**  
- Postgres maintains bitmap of modified pages to avoid scanning entire table during vacuum.  
- Only modified pages are cleaned, reducing IO and buffer pollution.  
- Cooperative cleaning trades off read-only queries becoming writes due to updating version chains during cleanup.

[01:07:31] **Transaction-Level Garbage Collection**  
- Tracking versions invalidated by each transaction helps targeted cleanup.  
- Upon commit, vacuum knows exactly which versions can be removed.  
- Centralized approach complements tuple-level scanning.

[01:09:02] **Index Management Under MVCC**  
- **Primary key indexes:** point to head of version chain, easier to maintain.  
- Updating primary key involves delete + insert treatment.  
- **Secondary indexes:** challenging because updates to head of version chain require updating all secondary indexes pointing to the tuple.  
- Two strategies:  
  - Store physical record ID in secondary indexes and update on version changes (expensive).  
  - Store primary key as a logical pointer in secondary indexes, then perform secondary lookup in primary index to find actual version chain head (used by MySQL).  
- Logical pointers reduce index update overhead but add indirection.

[01:12:55] **Secondary Indexes and Version Chain Updates**  
- Updating versions changes head of version chain; physical pointers in secondary indexes must be updated unless logical indirection used.  
- Logical indirection decouples secondary indexes from physical version chain changes.  
- Trade-off: extra lookup cost vs. index update cost.

[01:16:59] **Handling Duplicate Keys and Non-Unique Index Entries**  
- MVCC requires supporting multiple versions with same logical key for different snapshots.  
- Index keys include version metadata to ensure physical uniqueness.  
- Garbage collection also cleans obsolete index entries pointing to removed versions.

[01:20:06] **Handling Deletes in MVCC**  
- Deletion is logical, marking latest version as deleted via a flag or tombstone tuple at the end of the version chain.  
- Garbage collection physically removes deleted versions and entire chains once obsolete.  
- Proper visibility rules ensure deleted tuples are not visible to active transactions.

[01:20:58] **Summary of MVCC Implementations Across Systems**  

| System          | Storage Type       | Notes                                               |
|-----------------|--------------------|-----------------------------------------------------|
| Postgres        | Append-Only        | Stores multiple versions in same table pages; inefficient but historically entrenched. |
| SQL Server      | Time Travel        | Old versions stored separately; better GC management. |
| SAP HANA        | Time Travel (legacy)| Flipped strategies; now discontinued time travel.   |
| MySQL           | Delta Storage      | Efficient delta storage; uses logical pointers in secondary indexes. |
| Oracle          | Delta Storage      | Similar to MySQL; efficient modern implementation.  |

- Uber’s migration experience: switched from Postgres to MySQL due to Postgres’s inefficient secondary index MVCC implementation affecting performance and scalability.

[01:22:17] **Conclusion and Next Steps**  
- MVCC is a foundational technique in modern database systems enabling high concurrency and snapshot isolation.  
- Understanding MVCC impacts all aspects of database internals: storage layout, locking, indexing, garbage collection, and isolation guarantees.  
- Upcoming lectures will cover **logging and recovery**, essential for durability and correctness, followed by **distributed databases**, which introduce further complexity.

---

**Key Insights:**  
- MVCC enables non-blocking reads and writes by maintaining multiple versions of tuples with visibility rules based on timestamps and transaction status.  
- Snapshot Isolation is a practical but not fully serializable isolation level, susceptible to write skew anomalies.  
- Storage strategies (append-only, time travel, delta) have significant performance and complexity trade-offs; delta storage is the modern best practice.  
- Index management, especially for secondary indexes, is complex in MVCC systems due to version chain updates and visibility.  
- Garbage collection mechanisms are critical to reclaim storage and maintain performance in MVCC-enabled databases.  
- Real-world systems (Postgres, MySQL, Oracle, SQL Server) implement MVCC differently, influencing their scalability and suitability for workloads.

**Terminology:**  
- **MVCC:** Multi-Version Concurrency Control  
- **OCC:** Optimistic Concurrency Control  
- **2PL:** Two-Phase Locking  
- **Snapshot Isolation:** Isolation level providing consistent snapshot visibility at transaction start  
- **Version Chain:** Linked list of tuple versions for a logical tuple  
- **Garbage Collection:** Removing obsolete tuple versions no longer visible to active transactions  
- **Logical vs Physical Pointers:** Logical keys point abstractly to tuples; physical pointers reference exact storage locations  
- **Tombstone:** Marker indicating logical deletion of a tuple version

This comprehensive coverage grounds the viewer in the fundamental mechanisms, challenges, and design choices of MVCC in modern database systems.
