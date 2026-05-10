[00:00:01]  
**Introduction & Course Logistics**  
- Instructor opens with casual discussion about student finances and crypto schemes, discouraging reliance on blockchain/crypto as a database solution.  
- Announces **midterm exam** details:  
  - Covers lectures 1 to 11 (excluding current lecture).  
  - Study guide and materials are posted online (Piazza).  
  - Exam format: Mostly multiple-choice with some fill-in-the-blank questions.  
  - Students may bring a **simple calculator** and **one double-sided handwritten note sheet**.  
  - Encouraged to handwrite notes to better process material.  
- Office hours moved to accommodate instructor’s travel: tomorrow at 4:15 PM.  
- Project 2 released with recitation scheduled after the midterm.  
- Upcoming guest lectures on advanced database topics:  
  - Motherdoc’s co-founder discussing single-node efficient databases.  
  - SpyroB team presenting Vortex (a Parquet replacement).  
  - Columnar team discussing Apache Arrow for in-memory data representation.  

---

[00:04:21]  
**Today’s Lecture Overview: Joins in Single Node Databases**  
- Focus on **joins**, the most expensive operation in single-node database query execution (e.g., DuckDB).  
- Emphasizes **divide and conquer** strategy for sorting, hashing, and aggregations learned previously.  
- Nested loop joins are naive and inefficient. More sophisticated algorithms (sorting, hashing) improve performance significantly.  
- Discussion on **normal forms**:  
  - Minimal emphasis in this course; real-world applications typically use **3rd normal form**, often handled automatically by ORMs.  
  - Normal forms justify splitting data into multiple tables, necessitating joins to reconstruct original data.  

---

[00:07:24]  
**Types of Joins & Terminology**  
- Focus on **binary equi-joins (inner joins)**—joining two tables on equality of join keys.  
- Other join types (left outer, full outer, anti-joins, theta joins) can be handled by variations of the same algorithms but are not primary focus.  
- Multi-way joins (joining more than two tables simultaneously) exist but are complex and often less performant; mostly absent from commercial systems except some specialized ones (e.g., Relational AI).  
- **Query plan basics**:  
  - Queries represented as trees or DAGs with data flowing from leaf nodes (tables/files) upwards through operators.  
  - Join order (which tables to join first) is determined later by query optimizer based on statistics. For now, focus is on join algorithms themselves.  
- General rule: always use the **smaller table as the outer table** in join algorithms to minimize cost.

---

[00:12:11]  
**Join Output & Materialization Strategies**  
- Output tuples of a join combine attributes from both tables based on matching join keys.  
- Two materialization approaches:  
  - **Early materialization:** pass all attributes of tuples upward immediately, avoiding repeated access to base tables.  
  - **Late materialization:** pass only record IDs and key columns; fetch other attributes only if needed later, beneficial for selective joins and column stores.  
- Late materialization reduces data movement when joins filter out many tuples, common in OLAP scenarios.

---

[00:16:43]  
**Cost Model for Join Algorithms**  
- Focus on **I/O cost** (disk I/O for reading and writing pages), ignoring CPU costs such as key comparisons or hash table lookups since disk I/O dominates.  
- Assume single-node environment, ignoring network costs.  
- Output cost ignored as it is similar across join algorithms.  
- Cost parameters:  
  | Symbol | Meaning                         |  
  |--------|--------------------------------|  
  | M      | Number of pages in table R      |  
  | m      | Number of tuples in table R     |  
  | N      | Number of pages in table S      |  
  | n      | Number of tuples in table S     |  

---

[00:18:53]  
**Nested Loop Join (NLJ)**  
- **Naive nested loop join:** For each tuple in outer table R, scan entire inner table S to find matches.  
- Extremely inefficient: quadratic I/O cost ~ M + m × N pages.  
- Example calculation: Joining tables of 1,000 and 500 pages can take over an hour due to excessive disk scans.  
- Always place **smaller table as outer table** to reduce cost, but improvement is limited.  
- **Block nested loop join:** Improve NLJ by reading blocks (pages) instead of tuples, scanning inner table once per block of outer table pages.  
- Cost reduces to: M + (M / (B-2)) × N, where B is number of buffer pages available.  
- Practical improvements can reduce join time from hours to seconds by leveraging buffer memory effectively.  
- NLJ is reasonable if entire data fits in memory, otherwise too slow.

---

[00:31:43]  
**Index Nested Loop Join**  
- Uses an index on inner table to speed up lookups instead of scanning all pages.  
- Cost depends on index structure (B+ tree, hash index) and data distribution; hard to model precisely.  
- SQL Server can auto-create temporary indexes to optimize queries.  
- Conceptually similar to hash join but with persistent index structures.

---

[00:34:41]  
**Sort Merge Join (SMJ)**  
- Two phases:  
  1. **Sort phase:** Sort both tables on join keys using external merge sort.  
  2. **Merge phase:** Use two cursors to scan sorted tables and merge matching tuples.  
- Handles duplicates and backtracking by tracking last matched values.  
- Advantages:  
  - Performs well if data already sorted or if output needs to be sorted (e.g., ORDER BY).  
  - Cost is sum of sorting plus a single scan merge:  
    - Sort cost depends on number of pages and buffers.  
    - Merge cost is roughly M + N pages read once.  
- Example cost: For given tables, sorting plus merge cost ~7,500 I/Os—still practical.  
- Worst-case scenario is when all join keys are identical, leading to large intermediate join sizes.

---

[00:48:10]  
**Hash Join (HJ)**  
- Considered the most efficient and common join algorithm when no indexes exist.  
- Uses a hash function on join keys to partition data and quickly find matches.  
- Two phases:  
  1. **Build phase:** Scan smaller (outer) table, build in-memory hash table on join keys.  
  2. **Probe phase:** Scan larger (inner) table, hash join keys, probe hash table for matches.  
- Typically uses linear probing hash tables for speed and simplicity.  
- Hash join avoids nested loops and large scans by hashing data into buckets.  
- Similar to index nested loop join but builds temporary hash table dynamically rather than relying on existing index.  
- **Bloom filter optimization:**  
  - Builds a bloom filter during build phase to quickly exclude non-matching tuples during probe phase.  
  - Significantly reduces expensive hash table lookups for selective joins, potentially doubling performance.  

---

[00:54:47]  
**Grace Hash Join / Partitioned Hash Join**  
- Handles data sets too large to fit entirely in memory.  
- Partition both tables using the same hash function into smaller buckets that fit into memory.  
- Join each pair of corresponding partitions independently using in-memory hash join.  
- If partitions still too large, recursively repartition (recursive partitioning) until manageable size.  
- Recursive partitioning may fail for pathological cases where all join keys are identical, then fallback to block nested loop join.  
- Each partition processed independently, ensuring correctness since same hash function guarantees matching keys land in same partition.  

---

[00:57:03]  
**Historical Context & Specialized Hardware**  
- Grace Hash Join originated from 1980s “Grace” database machine research.  
- Historically, specialized hardware was built for database operations (sorting, joins).  
- Modern trend: explore GPUs for acceleration of database workloads.  
- Specialized hardware faces challenges from rapid CPU improvements and cloud commoditization, limiting adoption.  
- Some companies (Oracle Exadata, Teradata, Yellow Brick) still sell database appliances with specialized hardware tailored for high-performance workloads.

---

[01:07:52]  
**Hybrid Hash Join**  
- Extension of Grace Hash Join to handle skewed data distributions.  
- Keeps “hot” partitions (large/skewed buckets) in memory and spills others to disk.  
- Performs in-memory join on hot partition, disk-based join on others.  
- Balances memory use and I/O by avoiding heavy recursive partitioning on skewed data.  
- Implementation complexity and tuning required; some systems adopt, others prefer simpler buffer management.

---

[01:10:19]  
**Hash Join Implementation Details & Query Optimization**  
- Hash table sizing is critical: too small causes spills and recursive partitioning; too large wastes memory.  
- Estimating join input and output sizes is difficult and often inaccurate, especially with multiple joins.  
- Over-allocation of memory is common to avoid spills but reduces overall memory efficiency.  
- Most database systems implement multiple join algorithms and choose one at query optimization phase based on cost estimates.  
- Examples: PostgreSQL, DuckDB, Oracle, DB2 support nested loop, sort-merge, and hash joins.  
- MySQL added hash join support only recently (2019) focusing historically on OLTP workloads.  

---

[01:11:49]  
**Summary of Join Algorithms**  

| Algorithm             | Key Idea                                       | Strengths                              | Weaknesses                         | Typical Usage                      |  
|-----------------------|------------------------------------------------|---------------------------------------|-----------------------------------|----------------------------------|  
| Nested Loop Join (NLJ) | Naive nested loops over tuples                   | Simple, fast in-memory                 | Quadratic disk I/O cost            | Small datasets, in-memory joins   |  
| Block Nested Loop Join | Process outer table in blocks/pages              | Reduces iterations, better disk usage | Still high disk I/O on large data | Starting implementation           |  
| Index Nested Loop Join | Uses existing index on inner table               | Efficient if index exists              | Depends on index structure & data | When indexes available            |  
| Sort Merge Join (SMJ)  | Sort both tables then merge sorted runs          | Good if data sorted or output sorted  | Expensive sorting phase           | Analytical queries, sorted output |  
| Hash Join (HJ)         | Build hash table for smaller table, probe larger | Fastest generally, scalable            | Requires memory for hash table    | Most common in modern systems     |  
| Grace/Partition Hash   | Partition large tables into memory-sized buckets | Handles data larger than memory        | Recursive partitioning complexity | Large datasets with skew handling |  
| Hybrid Hash Join       | Keep hot partitions in-memory, spill others      | Efficient skew handling                | Complex implementation             | Skewed data distributions         |  

- **Hash join is generally preferable**, except when data is already sorted or output sorting required.  
- All major database systems implement multiple join algorithms and dynamically select best method during query optimization.

---

[01:16:54]  
**Closing Notes & Exam Reminder**  
- Midterm exam reminders: bring notes and student ID, no unusual items allowed.  
- Instructor will be absent; PhD students will proctor exam.  
- Encourages students to prepare well and avoid last-minute cramming.  

---

**Overall Key Insights:**  
- Joins are critical and expensive operations in database systems; efficient join algorithms are essential for performance.  
- Nested loop joins are simple but inefficient for large data; block nested loop and index nested loop improve but have limitations.  
- Sort merge joins are useful when sorting is already needed or data is pre-sorted.  
- Hash joins dominate modern systems due to efficiency, with optimizations like bloom filters and partitioning to handle large or skewed data.  
- Query optimizers decide which join algorithm to use based on cost models primarily driven by disk I/O considerations.  
- Real-world systems implement multiple join methods and sophisticated optimization to adapt to workload and data characteristics.  
- Understanding join algorithms at a conceptual level is foundational to database query performance tuning and system design.
