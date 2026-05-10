[00:00:00]  
**Introduction and Course Context**  
- The speaker begins from an emergency room in London and uses the waiting time to teach a class on databases, focusing on building database systems rather than just querying them.  
- Recap: Previous class covered SQL, relational algebra, and relational models at the logical application level.  
- Current focus: How to build the underlying software systems that execute SQL queries and store data.  
- Course outline:  
  - Start with single-node database systems.  
  - Progress to multi-node, scalable databases.  
  - Emphasis on layered architecture with well-defined APIs between layers, starting from disk management up through query execution.  
- Upcoming classes:  
  - This class: Storage (disk and file organization).  
  - Next class: Buffer management (memory management of disk pages).  
  - Later: Query execution, concurrency, recovery, and scaling.

[00:02:49]  
**Database System Architecture Overview**  
- Database systems are composed of multiple layers:  
  - **Disk Manager**: Reads/writes raw files on disk.  
  - **Buffer Manager**: Loads pages from disk into memory and manages them.  
  - **Access Methods**: Interfaces for accessing different data structures.  
  - **Query Planner/Optimizer**: Converts SQL queries into physical execution plans.  
- The lecture series will build understanding from the bottom (disk) up to the top (query execution).

[00:04:07]  
**Focus on Disk-Based Storage Architecture**  
- The database system assumes **primary storage is non-volatile disk**, e.g., SSDs, spinning disks, or networked storage.  
- Data must be moved from disk (non-volatile) into memory (volatile, e.g., DRAM) before it can be manipulated.  
- Writes to disk are delayed and batched for performance and durability.  
- This **classic von Neumann architecture** is the foundation for the semester's material.

[00:06:36]  
**Storage Hierarchy and Access Characteristics**  
- Storage hierarchy from fastest/smallest to slowest/largest:  
  - CPU registers (fastest, smallest)  
  - CPU caches  
  - DRAM (volatile memory)  
  - Non-volatile storage (SSD, HDD, network storage, tape)  
- Key distinction:  
  - **Volatile storage** (e.g., DRAM) supports **random byte-level access** but loses data on power loss.  
  - **Non-volatile storage** supports **block-level access** (smallest unit called a block or page, usually 4KB) and retains data after power loss.  
  - To access a single byte on disk, the entire block must be read into memory.  
- For the course, everything below the volatile/DRAM line is collectively called "disk," regardless of specific hardware.

[00:10:21]  
**Relative Latencies of Storage Accesses**  
- Access times differ drastically between storage types:  
  | Storage Type        | Access Time (nanoseconds) | Equivalent time if nanoseconds = seconds |  
  |---------------------|---------------------------|------------------------------------------|  
  | L1 CPU Cache        | ~1-4 ns                   | 1-4 seconds (human scale)                 |  
  | DRAM                | ~100 ns                   | 100 seconds (over a minute)                |  
  | SSD                 | ~16,000-50,000 ns         | Hours to days                              |  
  | Spinning Disk       | Much higher (ms range)     | Years to decades (very slow)               |  
- Jim Gray’s analogy: Treating nanoseconds as seconds highlights the enormous cost of disk IO compared to memory.  
- **Minimizing disk IO and optimizing for sequential access** are critical for performance.

[00:12:37]  
**Sequential vs Random IO**  
- Sequential IO (reading contiguous blocks) is significantly faster than random IO (jumping around blocks).  
- Example latency differences:  
  | Access Type       | Typical Latency Range     | Notes                                  |  
  |-------------------|--------------------------|---------------------------------------|  
  | Random IO (SSD)   | 80–100 microseconds       | Slower due to seek time                |  
  | Sequential IO      | 10–100 nanoseconds        | Much faster, can read multiple blocks  |  
- Databases try to maximize sequential IO to reduce latency and improve throughput.  
- Example: MySQL uses a double write buffer to batch sequential writes before asynchronously doing random writes.

[00:15:06]  
**Goal of the Database System**  
- Build a system that can manage databases **larger than available memory** by efficiently moving data between disk and memory.  
- Provide the **illusion of large memory** by caching, reusing pages, and scheduling IO carefully to avoid stalls.  
- Challenges include:  
  - Handling queries concurrently.  
  - Minimizing disk reads/writes.  
  - Managing memory efficiently.  
- If working set size exceeds memory capacity, performance degrades significantly.

[00:17:06]  
**High-Level View of Disk-Based Database Storage**  
- Databases are stored as collections of files on disk.  
- Some systems (e.g., SQLite, DuckDB) store the entire database in a single file; others use multiple files.  
- Files are standard OS files (e.g., opened with `fopen`), with no special meaning to the OS.  
- Files are broken into **fixed-length pages** (blocks), managed internally by the database.  
- A **page directory** (metadata) tracks pages and their locations within files.  
- In-memory, a **buffer pool** caches these pages as frames to serve queries efficiently.

[00:19:07]  
**Page Fetching Workflow**  
- When a query requests a page (e.g., page #2):  
  1. The system loads the page directory into memory.  
  2. The directory indicates where page #2 is stored.  
  3. The page is read from disk into the buffer pool in memory.  
  4. A pointer to the page in memory is returned to the execution engine.  
- The buffer pool guarantees the page is not replaced while in use.  
- The system does not interpret page contents at this low level; higher layers interpret bytes.

[00:21:40]  
**Two Core System Questions**  
1. **How to represent the database as files on disk?**  
2. **How to move pages between disk and memory efficiently?**  
- This lecture focuses on the first question (file and page organization).  
- Next lecture will focus on buffer management and memory handling.

[00:22:11]  
**Files and File Formats**  
- Files are proprietary to each database system; formats are not standardized or interchangeable between systems.  
- Open file formats like Parquet exist but are covered in advanced topics.  
- Databases generally run on standard off-the-shelf file systems (e.g., ext4, NTFS).  
- Some enterprise systems use custom file systems layered on raw block devices for performance (e.g., Oracle ASM), but this is rare due to complexity.

[00:24:44]  
**Storage Manager (Disk Manager)**  
- Responsible for reading/writing pages to disk files.  
- Tracks free space in files and manages page allocation.  
- Does **not** handle replication or redundancy; this is handled by lower-level storage (RAID) or higher-level distributed system layers.  
- Pages are fixed-size blocks that contain all data, metadata, catalog info, and indexes.  
- Pages are usually dedicated to one object (table or index) to avoid mixing types.

[00:28:15]  
**Page Identifiers and Metadata**  
- Each page has a unique **Page ID** to identify and locate it within the database.  
- Page directories map these IDs to physical file offsets.  
- In case of crash, recovery mechanisms rely on page metadata and directory synchronization.  
- Three notions of pages:  
  - Hardware page (typically 4KB atomic unit of IO).  
  - OS page (usually 4KB, sometimes huge pages).  
  - Database page (varies by system, often 4KB-16KB).  
- Database page size affects performance and is chosen based on workload characteristics (read-heavy vs write-heavy).

| Database System | Typical Page Size | Notes                          |  
|-----------------|-------------------|--------------------------------|  
| SQLite          | 4 KB (can be 512 bytes for embedded) | Small embedded footprint |  
| Oracle          | 4 KB (default)    | Enterprise-grade               |  
| PostgreSQL      | 8 KB              | Common default                 |  
| MySQL           | Up to 16 KB       | Adjustable                    |  
| SQL Server      | 8 KB              | Standard                      |  

[00:32:31]  
**Page Size Considerations**  
- Larger pages benefit **read-heavy** workloads by enabling more sequential reads and reducing IO operations.  
- Smaller pages benefit **write-heavy** workloads by reducing the amount of data written for small changes (due to block-level writes).  
- Course will start with 4-16 KB pages and explore trade-offs later.

[00:34:03]  
**Page Organization Methods**  
- **Heap File**: Unordered collection of pages; simplest and most common.  
- **Tree File**: Pages organized in a tree structure for efficient traversal (used in indexes).  
- **ISAM (Indexed Sequential Access Method)**: Older method combining index and sequential files; largely obsolete.  
- **Hashing**: Used for direct access based on hash values; less common for primary storage.  
- This lecture assumes heap file organization.

[00:35:25]  
**Heap File API Requirements**  
- Create pages.  
- Access pages by page ID.  
- Iterate over all pages for scanning.  
- Track free space within pages for storing tuples efficiently.

[00:36:53]  
**Page Addressing**  
- Simple arithmetic: page offset = base file offset + (page number × page size)  
- Page directory metadata handles mapping pages to files or network locations in multi-file or distributed systems.  
- Synchronization of page directory with page data on disk is important but can be loose for performance vs recovery trade-offs.

[00:40:22]  
**Page Structure**  
- Each page contains a **header** with metadata:  
  - Page size  
  - Software version  
  - Transaction visibility info (covered later)  
  - Compression and encoding info  
  - Checksums to detect corruption  
- Pages store tuples (records), catalogs, indexes, etc.  
- Pages are typically self-contained with metadata for crash recovery.

[00:42:33]  
**Tuple Storage Model: Row-Oriented**  
- Tuples are stored as contiguous rows within a single page.  
- Each tuple must fit entirely on one page.  
- This class focuses on classic row-store layout; column-store and other layouts introduced later.  
- Other models (log-structured storage, index-organized tables) exist but are out of scope for now.

[00:44:03]  
**Tuple Storage Challenges and Slotted Page Design**  
- Simple approach: fixed-length tuples stored sequentially in page; deletion causes fragmentation and complex management.  
- Problems with variable-length data and tuple movement: updating physical locations requires index updates.  
- **Slotted Page Architecture**:  
  - Page header at the start.  
  - Slot directory (array) holding offsets to tuples, growing from the header downward.  
  - Tuple data stored contiguously from the bottom of the page upward.  
  - Allows moving tuples within a page without changing external references, only updating slot pointers.  
- Advantage: localized compaction and deletion without affecting indexes.

[00:48:47]  
**Memory vs Disk Manipulation**  
- Manipulating tuple organization inside a page in memory is cheap compared to disk IO.  
- After loading a page into memory, reorganization such as compaction can be done freely to optimize storage.

[00:49:50]  
**Record Identifiers (Record IDs, Tuple IDs, Row IDs)**  
- Record IDs uniquely identify tuples and encode physical location: file ID, page ID, slot number.  
- Used by indexes and other components to reference tuples.  
- Usually not stored explicitly but derived at runtime.  
- Examples:  
  - Ingres: 4-byte tuple ID  
  - PostgreSQL: 6-byte tuple ID (ctid)  
  - SQLite: stores RowID as primary key even if user does not specify one.  
- Record IDs may change due to tuple movement; applications should not rely on them.

[00:52:38]  
**Tuple Internal Structure**  
- A tuple is a byte array with a header and attribute data.  
- Tuple header contains:  
  - Null bitmap (which columns are null).  
  - Transaction visibility info (covered post-midterm).  
  - Other metadata but does **not** include schema (schema is at page/table level).  
- Attributes stored in the order defined in schema, with padding for alignment.

[00:54:38]  
**Example Tuple Layout**  
- Example: Table with 5 columns (e.g., 3 integers, 1 double, 1 float).  
- Tuple stored as continuous bytes with header followed by attribute bytes in defined order.  
- Accessing an attribute involves pointer arithmetic and type casting at runtime.

[00:56:11]  
**Alignment and Padding Issues**  
- Attributes must be aligned for efficient access (usually 64-bit alignment).  
- Misaligned data that crosses word boundaries causes performance issues or hardware faults.  
- Solutions:  
  - Pad data with unused bytes (waste storage but simplest).  
  - Reorder columns physically to align data better (rarely done in practice).  
- Most systems prefer padding to ensure correctness and performance.

[00:58:44]  
**Data Types and Representation**  
- Integer and floating point use standard hardware representations (e.g., 32-bit int, IEEE 754 floats).  
- Text and binary data may be stored inline or via pointers to overflow pages.  
- Endianness is usually managed by the DB system to ensure portability.  
- Dates/timestamps often stored as integers representing time since epoch, with time zone info managed separately.

[01:02:14]  
**Floating Point vs Fixed Precision Numerics**  
- Floating point (IEEE 754) is fast but imprecise due to rounding errors.  
- Fixed precision (numeric/decimal) types store exact decimal values with metadata about scale and precision.  
- Fixed precision is critical for financial, scientific, and other accuracy-sensitive applications.  
- Example: PostgreSQL’s numeric type stores decimal data as a byte array with detailed metadata and implements arithmetic in software, which is slower but precise.

[01:07:12]  
**Handling NULL Values**  
- Common method: store a bitmap in tuple header indicating which columns are NULL.  
- Alternative methods:  
  - Use special reserved values to indicate NULL (reduces value domain, common in column stores).  
  - Store a per-attribute NULL flag (rare and inefficient).  
- The bitmap approach is standard in row stores.

[01:09:35]  
**Large Attribute Storage: Overflow Pages**  
- Tuples must fit on a single page, but some attributes may be larger than page size.  
- Solution: store large attribute values in **overflow pages**, referenced by pointers (record IDs) from the main tuple.  
- Databases differ on thresholds for overflow storage (e.g., PostgreSQL uses 2KB threshold).  
- Techniques to optimize overflow:  
  - Compression (e.g., Snappy, gzip).  
  - Prefix storage (store a prefix inline to avoid fetching overflow pages when possible).  
  - Chaining overflow pages for very large values.  
- External storage (outside DB) also possible for very large objects (e.g., Oracle BFiles, SQL Server filestreams).

[01:14:53]  
**Summary and Next Steps**  
- Database system storage consists of files broken into pages, with metadata managing page locations and contents.  
- Tuple storage inside pages uses slotted page architecture enabling flexible and efficient management.  
- Data types and alignment issues influence internal tuple layout.  
- Overflow pages and external storage handle large attributes.  
- Next class will cover buffer pool management: bringing pages into memory, managing them safely, and writing back to disk.  
- Future lectures will extend into execution engine and other components.

---

**Key Takeaways:**  
- **Database storage is fundamentally about managing files broken into fixed-size pages on disk.**  
- **Pages are cached in memory via a buffer pool to hide disk latency.**  
- **Slotted pages separate tuple metadata from data, enabling efficient insertion, deletion, and compaction.**  
- **Record IDs uniquely identify tuples but may change, so apps should not rely on them.**  
- **Tuple layout requires careful handling of alignment, null values, and large attribute storage.**  
- **Disk IO latency dominates performance; systems optimize for sequential IO and minimize random IO.**  
- **The layered architecture allows modular design and replacement of components without affecting others.**

This lecture lays the fundamental groundwork for building database storage engines, balancing theoretical concepts with practical system design considerations.
