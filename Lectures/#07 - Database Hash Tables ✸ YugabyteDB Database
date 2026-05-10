[00:00:00]  
### Summary of Lecture Introduction and Context  
- The lecture starts with an overview of the previous session which covered the **storage layer of database systems**, focusing on how data is stored on disk and buffered in memory.  
- The current and upcoming lectures focus on the **middle layer of the database system stack**, specifically the **access methods**—the APIs and data structures used to efficiently access, query, and manipulate data stored on disk and in memory.  
- The session introduces **hash tables** (unordered data structures) for this and the next week will cover **tree-based structures** (ordered data structures).  
- The importance of building custom data structures for databases rather than relying on standard libraries (like STL vectors or third-party libraries) is emphasized, because those do not integrate with the **buffer pool manager** and often rely on volatile OS memory rather than controllable, persistent storage.  
- The session will cover:  
  - Basic concepts behind hash tables  
  - Hash functions and their trade-offs  
  - Two static hashing schemes: **linear probing hashing** and **cuckoo hashing**  
  - Dynamic hashing schemes that support table resizing  
- The data structures discussed will be used throughout the database system for:  
  - Table indexes (primary and secondary keys)  
  - Internal metadata like page directories  
  - Core storage mechanisms (e.g., index-organized storage)  
  - Temporary storage during query execution (e.g., hash joins)  
- Concurrency and thread-safety aspects will be addressed later; for now, data structures are assumed to be **single-threaded**.  
- The lecture highlights a key database design principle: prioritizing **sequential I/O over random I/O** to improve disk access performance.

---

[00:06:51]  
### Core Concepts of Hash Tables  
- A **hash table** is an unordered associative array mapping arbitrary keys to arbitrary values.  
- The core mechanism:  
  - A **hash function** converts a key of arbitrary length into a fixed-size integer.  
  - This integer is used to compute an index (slot) in the hash table where the key-value pair should be stored or retrieved.  
- Space complexity is proportional to the expected number of keys \( n \).  
- Average time complexity for lookup is **O(1)**, assuming a good hash function and low collision rate; worst case is **O(n)** if many collisions occur.  
- Constants in runtime matter greatly in databases because even small efficiency improvements scale to significant savings at large volumes.  
- Unlike typical algorithm classes, database hash tables trade off some in-memory optimality to optimize for sequential I/O on disks.

---

[00:09:10]  
### Static Hashing: Simple Fixed-Size Hash Table  
- A **static hash table** assumes a fixed number of slots \( n \) allocated upfront.  
- The simplest scheme:  
  - Hash the key, take modulo \( n \) to find the slot.  
  - Each slot stores a pointer to the actual data or key-value pair.  
- Assumptions and limitations:  
  - Knowing the number of keys in advance is often unrealistic.  
  - Assumes keys are unique and no collisions occur or perfect hashing exists (rare in practice).  
  - Handling collisions is necessary since different keys may hash to the same slot.  
- **Perfect hashing** is theoretically collision-free but impractical for dynamic or unknown datasets.

---

[00:13:25]  
### Components of a Hash Table  
- A hash table consists of:  
  1. **Hash function** - maps keys to integers; trade-off between speed and collision rate.  
  2. **Hashing scheme** - method to resolve collisions and store keys that hash to the same slot.  
  3. **Storage space** - the fixed or dynamically allocated memory (slots/buckets) to store the data.  

---

[00:14:18]  
### Hash Functions in Practice  
- Modern systems usually use well-tested, off-the-shelf hash functions rather than custom ones.  
- Examples:  
  - **MurmurHash** (popular since 2008)  
  - **XXHash/XXHash3** (widely used for speed and low collision rate)  
  - **RapidHash** (newer and faster but not yet widely adopted)  
- Hash functions produce fast, fixed-length outputs (e.g., 32 or 64-bit integers).  
- Cryptographic properties and reversibility are not required since hashing is internal to the database.  
- Database systems do not generally worry about denial-of-service attacks that exploit hash collisions, as these tables are not exposed externally.

---

[00:20:00]  
### Static Hashing Schemes: Linear Probing and Cuckoo Hashing  
- **Linear Probing Hashing**  
  - Fixed-size hash table with open addressing.  
  - Upon collision (slot occupied), scan linearly to find the next free slot.  
  - Simple, commonly used, and often more performant than complex alternatives.  
  - Insertions, deletions, and lookups scan sequentially from the hashed slot until the target or free slot is found.  
  - Uses a **load factor** to determine fullness; once exceeded, the table must be resized by allocating a larger table and rehashing all keys.  
  - Deletions use **tombstones** (markers indicating a slot was occupied but now deleted) to avoid breaking linear scans during lookups.  
  - Fixed-length data can be stored inline; variable-length data requires pointers to external storage.  
  - Handling non-unique keys typically involves storing duplicates in the table itself or chaining (less common in linear probing).  

- **Cuckoo Hashing**  
  - Uses multiple (typically 2 or more) hash functions per key, creating multiple candidate slots for each key.  
  - Insertion may require **kicking out** existing keys (victims) to their alternate location, recursively.  
  - Lookups check all hash function slots; guaranteed O(1) lookup.  
  - More complex insertion logic and slightly slower than linear probing.  
  - Can get stuck in cycles requiring table resizing.  
  - Less widely used because linear probing is simpler and faster in practice.  

---

[00:47:02]  
### Dynamic Hashing: Growing and Shrinking Hash Tables  
- Static hashing assumes fixed size; dynamic hashing schemes allow tables to grow and shrink incrementally.  
- Two key dynamic hashing schemes:  
  1. **Extendable Hashing**  
     - Buckets are split when full, redistributing keys locally instead of rehashing the entire table.  
     - Uses a **global bit counter** and **local bit counters** to determine how many bits of the hash are used to index buckets.  
     - Multiple entries in the bucket pointer array can point to the same physical bucket until splits occur.  
     - Avoids long bucket chains but may require multiple rounds of splitting for large growth.  
     - Not widely used in mainstream DBMS but found in some embedded or older systems.  
  2. **Linear Hashing**  
     - Splits buckets sequentially according to a **split pointer** that advances as the table grows.  
     - Uses two hash functions or mod values to reassign keys during splits.  
     - Allows incremental resizing without rehashing the entire table at once.  
     - Used in systems like PostgreSQL’s dynamic hash tables and Berkeley DB.  
- Both schemes optimize for online resizing with minimal disruption to ongoing queries and insertions.

---

[01:07:55]  
### Guest Lecture: Karthik (Co-founder of YugabyteDB)  
- **YugabyteDB** is a distributed, cloud-native database built on PostgreSQL codebase, fully open source under Apache 2.0.  
- Focus areas:  
  - Distributed transactions  
  - Multi-API support (Postgres-compatible and Cassandra-compatible APIs)  
  - Cloud-native deployment and scalability  
  - Enterprise-grade resilience, availability, and geo-distributed multi-region support  
  - Support for AI workloads and modern applications  
- Market positioning:  
  - Balances **Postgres compatibility** with **cloud-native innovation**.  
  - Built to leverage Postgres ecosystem while addressing modern distributed application needs.  
- Example customer: **National Payment Corporation of India (NPCI)**, which runs all mobile payments in India on YugabyteDB, highlighting the critical importance of scalability and uptime.  
- Architectural approach:  
  - Retain PostgreSQL query processing engine (“green box”) as stateless and run it on any node.  
  - Replace PostgreSQL storage engine (“blue box”) with a distributed transactional storage layer that handles replication and distribution transparently.  
  - This enables seamless Postgres compatibility with distributed execution.  
- Challenges and solutions:  
  - Network latency and data distribution impact performance compared to single-node Postgres.  
  - They optimize by pushing down query operations closer to data nodes to reduce data transfer and latency.  
  - Work is ongoing to improve performance and feature parity with Postgres, including cost-based query optimization in a distributed environment.  
- Additional features under development include connection pooling, vector indexes, and new indexing methods like BM25 for search relevance.  
- The guest lecture reflects real-world application of database principles discussed earlier, including data structures and system design trade-offs.

---

### Key Insights and Conclusions  
- **Custom-built data structures in databases are essential** for integrating with buffer managers and storage layers, enabling efficient disk I/O and crash resiliency.  
- **Hash tables in databases differ from textbook versions** by emphasizing controlled memory usage, sequential I/O, and integration with persistent storage rather than ephemeral OS memory.  
- **Static hashing schemes like linear probing are simple and performant**, but require resizing on load or collision thresholds.  
- **Dynamic hashing schemes like extendable and linear hashing enable incremental resizing**, improving practical usability for growing datasets.  
- **Hash function choice matters** for speed and collision characteristics but is usually outsourced to proven libraries.  
- The **handling of deletions in open addressing schemes requires tombstones** to maintain correct lookup semantics.  
- **Distributed databases like YugabyteDB leverage these concepts and extend them** to cloud-native, scalable systems that maintain compatibility with existing relational models like Postgres.  
- Performance trade-offs exist between single-node and distributed systems, but pushing computation closer to data and adaptive query planning help bridge the gap.  
- The lecture and guest talk together provide a comprehensive view of both foundational data structures and modern distributed database architecture.

---

### Glossary Table

| Term                   | Definition                                                                 |
|------------------------|-----------------------------------------------------------------------------|
| Buffer Pool Manager    | Component managing memory pages cached from disk for efficient access.      |
| Hash Function          | A function mapping arbitrary keys to fixed-size integers for indexing.      |
| Linear Probing         | Collision resolution by scanning sequentially for next free slot.           |
| Cuckoo Hashing         | Uses multiple hash functions and eviction to resolve collisions.             |
| Tombstone              | Marker indicating a deleted entry that preserves lookup correctness.        |
| Extendable Hashing     | Dynamic hashing scheme splitting buckets incrementally with bit counters.   |
| Linear Hashing         | Dynamic hashing scheme that splits buckets sequentially using a split pointer.|
| Postgres Query Engine  | The component of Postgres responsible for parsing, planning, and executing queries.|
| YugabyteDB             | Distributed, cloud-native database built on Postgres with multi-region support.|

---

### Summary Timeline Table

| Timestamp  | Topic/Event                                  | Key Points                                                  |
|------------|----------------------------------------------|-------------------------------------------------------------|
| 00:00:00   | Lecture Introduction                         | Overview of storage layers, buffer pools, and access methods |
| 00:06:51   | Hash Table Fundamentals                      | Definition, complexity, and importance of constants          |
| 00:09:10   | Static Hash Table Design                      | Fixed-size arrays, assumptions, and collision challenges     |
| 00:14:18   | Hash Functions                              | Modern hash functions and trade-offs                         |
| 00:20:00   | Static Hashing Schemes                      | Linear probing and cuckoo hashing detailed                   |
| 00:47:02   | Dynamic Hashing Schemes                     | Extendable and linear hashing for incremental resizing       |
| 01:07:55   | Guest Lecture: YugabyteDB                    | Distributed Postgres-compatible database design and use cases|

---

This completes a detailed, professional summary strictly grounded in the provided transcript, covering foundational concepts, practical implementations, and modern applications of hash tables and database architecture as discussed in the lecture and guest talk.
