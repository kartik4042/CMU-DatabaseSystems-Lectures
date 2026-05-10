[00:00:00]  
**Introduction and Administrative Announcements**  
- The session begins with informal remarks and acknowledgment of DJ Cash.  
- Reminder about upcoming deadlines:  
  - **Homework 2 due the upcoming Sunday.**  
  - **Project 1 due the following Sunday.**  
- A Zoom recitation session for Project 1 is scheduled for the next night, focusing on higher-level concepts rather than basic programming help. It will be recorded and made available later.  
- Industry visits are planned for the next day at the Gates building, with an optimized schedule generated via a stable marriage algorithm to accommodate preferences.  
- A notable industry update: SingleStore was recently acquired by Vector Capital. SingleStore was formerly MemSQL, with Ashton Kutcher as an early investor. Alumni from SingleStore have moved on to other notable companies.

[00:04:27]  
**Recap of Previous Lecture: Storage Models Overview**  
- Three main storage models discussed previously:  
  1. **Tuple-oriented (row) storage with slotted pages** – traditional row storage.  
  2. **Log-structured storage** – appending updates, with background compaction (level/universal compaction).  
  3. **Index-organized storage** – B+ tree-based with data stored in leaf nodes rather than pointers to heap pages.  
- Each model has pros and cons, especially related to update-heavy workloads (OLTP).  
- However, **read-heavy analytical queries (OLAP)** perform poorly on these traditional storage models due to inefficient data access patterns.  
- The previous SingleStore talk introduced the motivation for **column stores**, which will be the focus of today’s lecture.

[00:06:13]  
**Database Workloads: OLTP, OLAP, HTAP, and AI**  
- **OLTP (Online Transaction Processing):**  
  - Ingestion of small amounts of data per transaction (e.g., posting a comment, buying an item).  
  - Queries are simple and often touch only a few tuples and attributes.  
  - Write-heavy but with many simple reads.  
- **OLAP (Online Analytical Processing):**  
  - Analytical queries over large volumes of data, aggregating and joining multiple tables.  
  - Read-heavy and queries tend to be complex (aggregations, window functions).  
- **HTAP (Hybrid Transactional/Analytical Processing):**  
  - Combines OLTP and OLAP in a single system, aiming for fast transactional updates and fast analytics on the same data.  
  - Challenging to implement perfectly; many systems compromise on either transactional or analytic performance.  
  - Examples include SingleStore, PingCAP TiDB.  
- **AI Workloads:**  
  - Viewed as an extension of OLAP, focusing on understanding "why" certain patterns occur, involving vector indexes and more complex analysis.  
  - Treated as part of the OLAP category for this discussion.  
- **Workload Complexity Quad Chart:**  
  - Axes: Write-heavy vs. Read-heavy and Simple vs. Complex queries.  
  - OLTP: Simple queries, write-heavy.  
  - OLAP: Complex queries, read-heavy.  
  - HTAP lies somewhere in the middle.

[00:13:40]  
**Wikipedia Schema Example for Workload Illustration**  
- Uses a simplified Wikipedia schema consisting of:  
  - **User account table** (login info).  
  - **Articles/pages table** (article content).  
  - **Revisions table** (versions of pages).  
- Demonstrates OLTP queries such as:  
  - Getting the latest revision for a page (index lookup by page ID).  
  - Updating a user’s last login timestamp.  
  - Inserting a new revision.  
- OLAP queries example:  
  - Counting logins from users with hostnames ending in ".gov" over time (complex scan and aggregation).  
- Highlights how OLTP queries access a small subset of data, often a single tuple, while OLAP queries scan large datasets.

[00:19:41]  
**Storage Models: Row Store (NSM) vs. Column Store (DSM) vs. Hybrid (PAX)**  

| Storage Model             | Description                                                                 | Best Use Cases                 | Key Characteristics                         |
|--------------------------|-----------------------------------------------------------------------------|-------------------------------|---------------------------------------------|
| **N-ary Storage Model (Row Store or NSM)** | Stores complete tuples contiguously on disk pages.                       | OLTP workloads                | Efficient for point queries and updates; reads full tuples.   |
| **Decomposition Storage Model (Column Store or DSM)** | Stores each attribute column-wise in separate pages.                    | OLAP workloads                | Efficient for queries accessing few columns; better compression. |
| **Hybrid Storage Model (PAX)**             | Partitions data into horizontal chunks; within each chunk data is stored column-wise but tuples are clustered together. | Mixed workloads (HTAP)         | Balances between row and column stores; reduces I/O and stitching overhead. |

- Row stores are common and efficient for OLTP because tuples are accessed or updated as a whole.  
- OLAP queries scan many tuples but usually only a few columns, so column stores avoid reading unnecessary data.  
- PAX (Partition Attributes Across) stores data in horizontal partitions (row groups) but stores columns within partitions to reduce overhead of stitching tuples.

[00:26:42]  
**Detailed Explanation of Row Store (NSM) Model**  
- Tuples stored contiguously in pages with a header and slot array indicating tuple offsets.  
- Efficient for OLTP queries like single tuple lookups or inserts.  
- OLAP queries scanning many tuples suffer because all columns are fetched even if only a few are needed, causing unnecessary I/O.

[00:28:11]  
**Detailed Explanation of Column Store (DSM) Model**  
- Stores data for each attribute contiguously in pages (column chunks).  
- Null bitmap is stored once per column, not per tuple, reducing overhead.  
- Queries accessing few columns only load relevant column pages, reducing I/O significantly.  
- Tuple reconstruction requires stitching column values together, which is costly for point queries or updates.  
- Fixed-length values simplify indexing and offset calculation.  
- Variable-length data (e.g., strings) are dictionary encoded into fixed-length integers for efficient storage and processing.

[00:39:13]  
**Hybrid Storage Model (PAX)**  
- Data is horizontally partitioned into row groups (e.g., 100MB or 1 million tuples).  
- Within each row group, data is stored in columnar format (column chunks).  
- This keeps data for a tuple close together within a row group, reducing tuple stitching cost compared to pure column stores.  
- PAX is the de facto approach in almost all modern column stores and file formats (e.g., Parquet, ORC).  
- Metadata is stored at the end of each file/row group, specifying offsets and compression details.

[00:47:12]  
**Compression Overview and Importance**  
- I/O is the primary bottleneck in database query performance.  
- Compression reduces the amount of data read from disk, improving query speed and buffer pool efficiency.  
- Trade-off: compression/decompression uses CPU cycles but saves significant I/O time.  
- Ideal compression scheme characteristics:  
  1. Produces fixed-length compressed values for efficient offset addressing.  
  2. Postpones decompression until absolutely necessary, allowing query processing on compressed data.  
  3. Is lossless (original data is perfectly reconstructible).  
- Variable-length data stored separately and compressed differently (e.g., PostgreSQL TOAST).

[00:51:27]  
**Compression Granularity Options**  

| Compression Level    | Description                                                                    | Commonality/Examples                         |
|---------------------|--------------------------------------------------------------------------------|---------------------------------------------|
| Block/Page Level     | Compress entire pages or blocks of data using general-purpose compression (gzip, snappy, zstd). | Common in many DBs (MySQL InnoDB uses this). |
| Tuple Level         | Compress entire tuples individually; rare and complex due to random access difficulty.                  | Rare; some Chinese system "Terra" uses.     |
| Attribute Level     | Compress individual attribute values, especially large ones like text (Postgres TOAST).                  | Common for large variable-length fields.    |
| Columnar Compression | Compress columns exploiting data homogeneity and patterns (dictionary encoding, RLE).                   | Most effective for column stores; widely used.|

- General-purpose compression algorithms are opaque to DB and require full decompression to access data.  
- Columnar compression schemes allow operating directly on compressed data.

[00:52:46]  
**MySQL InnoDB Compression Details**  
- InnoDB compresses pages before writing to disk; compressed pages have fixed sizes (e.g., 1KB, 4KB, 16KB).  
- Uses a modification log (mod log) to buffer updates without decompressing pages immediately.  
- When mod log fills, page is decompressed, updated, and recompressed.  
- Reads may require decompressing pages if data must be inspected.  
- Mod log acts similarly to a write-ahead log but at the page level, enabling efficient writes in compressed storage.

[01:00:18]  
**Operating on Compressed Data: Dictionary Encoding Example**  
- Database encodes values (e.g., strings) into fixed-length integer codes using a dictionary.  
- Queries rewrite predicates to operate on codes rather than original values, enabling fast integer comparisons.  
- For example, a query "WHERE name = 'Andy'" translates to "WHERE code = 10" after dictionary encoding.  
- This enables efficient scanning and filtering without decompressing full data.  
- Dictionary is order-preserving allowing range queries and prefix matches using encoded integers.

[01:02:05]  
**Common Compression Techniques in Column Stores**  

| Compression Technique | Description                                                                                  | Best Applied When                                        |
|-----------------------|----------------------------------------------------------------------------------------------|---------------------------------------------------------|
| **Run-Length Encoding (RLE)** | Stores consecutive repeated values as a single value + run length.                       | Data with many repeated values, especially after sorting. |
| **Bit-Packing**       | Stores values using minimal bits (e.g., 8 bits instead of 32) when domain is small.           | When data values are over-allocated in bit width.       |
| **Bitmap Encoding**   | Stores a bitmap per unique value indicating tuple positions.                                 | Low cardinality attributes (e.g., boolean flags).       |
| **Delta Encoding**    | Stores differences between consecutive values instead of absolute values.                    | Data with small incremental changes (e.g., timestamps). |
| **Dictionary Encoding**| Maps frequent values to small integer codes, storing codes instead of full values.           | All data with repeated values, especially variable-length data.|

- Techniques are often **composed multiplicatively**, e.g., RLE + Delta Encoding for better compression.  
- Sorting data columns improves RLE effectiveness by grouping identical values.

[01:03:04]  
**Run-Length Encoding (RLE) Details**  
- Ideal when sorted data has long runs of identical values.  
- Example: Boolean "is_dead" column with many repeated "yes" or "no" values compressed to run-length triples (value, offset, length).  
- RLE performs poorly on highly alternating data but sorting can mitigate this.  
- Sorting one column affects others; trade-offs exist in choosing sort keys.

[01:05:45]  
**Bit-Packing Details**  
- Reduces storage by using only as many bits as necessary (e.g., 8 bits instead of 32).  
- Handles outliers with a "patch table" for exceptional large values (known as "mostly encoding" in Amazon Redshift).  
- Patch table stores exceptions and is accessed via special sentinel values in the main data.

[01:09:32]  
**Bitmap Encoding Details**  
- Maintains one bit vector per distinct value indicating presence at each tuple offset.  
- Efficient for very low cardinality columns (e.g., boolean flags).  
- Enables extremely fast bitwise operations (AND, OR) for queries.  
- Not suitable for high-cardinality columns due to space explosion.

[01:13:48]  
**Delta Encoding Details**  
- Stores the difference between consecutive or base values rather than absolute values.  
- Effective for time series or slowly varying data (e.g., temperature, timestamps).  
- Combined with RLE for additional compression gains.  
- Variants include frame-of-reference encoding using global minimum values.

[01:16:11]  
**Dictionary Encoding Details**  
- Replaces repeated values with fixed-length integer codes.  
- Dictionary stores mapping between codes and original values.  
- Supports fast query evaluation by rewriting predicates using dictionary codes.  
- Dictionary is sorted to preserve order, enabling range queries without decompressing data.  
- Deletion in dictionary is straightforward; insertion is complex and may require recompression.  
- OLAP workloads benefit from mostly immutable data easing dictionary maintenance.

[01:20:37]  
**Conclusion and Next Steps**  
- The lecture concludes with a brief mention that the next class will cover handling OLTP and OLAP workloads together.  
- A guest speaker from Yugabyte is scheduled.  
- Homework and project recitations and industry talks occur soon.

---

### **Key Insights**  
- **Database workload types (OLTP, OLAP, HTAP) dictate storage and compression strategies.**  
- **Row stores (NSM) excel at OLTP but perform poorly for analytical queries due to I/O overhead loading unnecessary columns.**  
- **Column stores (DSM) optimize OLAP by storing data column-wise, enabling selective data access and better compression.**  
- **Hybrid models like PAX combine the benefits of row and column stores by grouping tuples horizontally but storing columns contiguously within groups.**  
- **Compression is critical to reduce I/O cost and improve performance; it must be lossless and often exploits data regularities.**  
- **Common compression schemes include run-length encoding, bit-packing, bitmap encoding, delta encoding, and dictionary encoding, often used in combination.**  
- **Dictionary encoding enables operating on compressed data by rewriting queries to use compressed codes for efficient comparison and filtering.**  
- **Trade-offs exist between compression effectiveness, query performance, and update complexity, especially for mutable OLTP data.**

---

### **Summary Table: Storage Models and Compression Techniques**

| Aspect                     | Row Store (NSM)                  | Column Store (DSM)              | Hybrid (PAX)                  |
|----------------------------|---------------------------------|---------------------------------|------------------------------|
| Data Layout                | Tuples stored contiguously      | Columns stored contiguously      | Horizontal partitions storing columns contiguously within partitions |
| Best For                  | OLTP (write-heavy, point queries)| OLAP (read-heavy, column-access)| Mixed workloads (HTAP)       |
| Query Performance          | Fast for point queries and updates| Fast for analytical queries accessing few columns | Balanced performance          |
| Compression                | Limited (general block-level)  | Extensive (dictionary, RLE, delta, bitmap) | Columnar compression within partitions |
| Tuple Reconstruction Cost | Low                           | High (stitching columns)         | Medium                       |

| Compression Technique      | Description                    | Strengths                        | Limitations                   |
|----------------------------|-------------------------------|---------------------------------|------------------------------|
| Run-Length Encoding (RLE) | Compresses repeated values    | Very effective on sorted data   | Ineffective on random data    |
| Bit-Packing               | Uses minimal bits per value    | Saves space on low-range values | Requires patching for outliers|
| Bitmap Encoding           | Bit vector per unique value    | Fast bitwise operations         | Not suitable for high cardinality columns |
| Delta Encoding            | Stores differences between values | Great for time series/smooth data | Needs sorted or ordered data  |
| Dictionary Encoding       | Maps values to integer codes   | Enables fast comparison on compressed data | Complex insertion/deletion    |

---

This detailed summary captures all major points, technical explanations, examples, and nuances as presented in the original transcript, ensuring clarity and a professional tone suitable for an academic or technical audience.
