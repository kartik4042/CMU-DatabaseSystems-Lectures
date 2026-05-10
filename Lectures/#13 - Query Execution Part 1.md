[00:00:01]  
**Introduction and Course Updates**  
- Instructor shares recent travels visiting database companies including Snowflake, Supabase, PlanetScale, and Microsoft.  
- Mentions an upcoming discussion on a novel Microsoft approach using query optimizers and large language models (LLMs) for proving SQL query equivalence—highlighted as a "pure genius" idea to be covered next week.  
- Project 2 deadline approaching Sunday; project is harder than Project 1, students urged to start immediately.  
- AWS was down temporarily affecting access to course materials.  
- Announcements about upcoming tech talks from Columnar (Apache Arrow), Astronomer (Airflow), and SingleStore, including pizza and internship/job opportunities.  
- Recap of last class: join algorithms (nested loop, sort-merge, hash join), sorting, aggregation basics covered.  

---

[00:04:21]  
**Query Execution Overview**  
- With building blocks like sorting, aggregation, and joins in place, the focus shifts to **executing queries by combining components into a single system**.  
- Query plans in relational systems are DAGs (Directed Acyclic Graphs) of operators; often implemented as trees (subset of DAGs).  
- Data flows logically between operators up to the root, which produces final query output.  
- Introduces **pipelines**: sequences of operators where tuples flow continuously without stopping or waiting for full input, except at **pipeline breakers**—operators that must receive all input before producing output (e.g., hash join build phase, sort operator).  
- Pipelines optimize memory and CPU usage by processing tuples as far as possible without materializing intermediate results or writing to disk unnecessarily.  
- Distinction between **logical data flow** (conceptual pushing of tuples) and **physical data flow** (often implemented as pulling data).  

---

[00:08:57]  
**Query Processing Models: Control Flow and Data Flow**  
- Query processing models define how data moves between operators and how operators are invoked:  
  - **Control flow**: instructs operators when to run.  
  - **Data flow**: defines how data is passed between operators.  
- Output of operators is tuples, which may be passed in different forms:  
  - Early materialization: entire tuples passed up.  
  - Late materialization: minimal columns plus record IDs passed; allows fetching additional data later if needed.  
- Three basic query processing models:  
  1. **Iterative (Iterator/Volcano) Model**: passing one tuple at a time using next() calls.  
  2. **Materialization Model**: passing entire sets of tuples at once (all data).  
  3. **Vectorized (Batch) Model**: passing tuples in batches (vectors) of fixed size (e.g., 1024 tuples).  

---

[00:11:45]  
**Iterative Model (Volcano Model)**  
- Most common and simplest model used by many systems (Postgres, MySQL, SQLite).  
- Each operator implements an interface with `open()`, `next()`, and `close()` functions.  
- `next()` returns one tuple at a time or null if no more tuples.  
- Execution starts by calling `next()` on the root operator, which recursively calls `next()` on children.  
- Example: hash join build phase scans left table (R), builds hash table by calling `next()` repeatedly until no tuples left, then scans right table (S) to probe hash table.  
- Pipeline model allows streaming tuples up the operator tree without materializing intermediate results.  
- Advantages: simple to implement, easy to debug, well suited for OLTP workloads with small result sets.  
- Downsides: function call overhead can be expensive for large datasets; processing is strictly sequential in simple implementations.  

---

[00:21:09]  
**Materialization Model**  
- Passes entire sets of tuples from one operator to the next at once.  
- Operators output full buffers of tuples instead of single tuples.  
- Operator fusion is critical here to avoid passing unnecessary data and reduce overhead by combining operations like filtering and projection inside scan operators.  
- Developed in the 1990s for OLAP systems (example: MonetDB, precursor influences Snowflake).  
- Less common today but still used in some in-memory OLTP systems.  
- Issues: inefficient if only a small subset of tuples are needed (e.g., queries with LIMIT clauses), as it materializes large intermediate results unnecessarily.  

---

[00:26:34]  
**Vectorized (Batch) Model**  
- Hybrid approach passing fixed-size batches (vectors) of tuples between operators (common batch sizes: 1024 or 2048 tuples).  
- Most modern OLAP systems (DuckDB, Firebolt, ClickHouse, Snowflake) use this model.  
- Enables efficient CPU usage by leveraging SIMD (Single Instruction, Multiple Data) instructions and GPU acceleration.  
- Operator implementations maintain output buffers and emit batches only when full, improving throughput and cache locality.  
- Introduced around 2006-2007 as an improvement over iterative and materialization models for columnar OLAP workloads.  
- Vectorized model combined with push-based execution (see below) is prevalent in high-performance analytical databases.  

---

[00:32:21]  
**Pull-Based vs Push-Based Execution Models**  
- **Pull-Based Model**: (traditional iterator approach) execution starts at the root operator and "pulls" data from children by calling `next()`. Data flows upward.  
- **Push-Based Model**: execution starts at the leaf nodes and "pushes" data upwards through operators without relying on `next()` calls.  
- Push-based allows scheduling of discrete pipelines as tasks, enabling more global optimization and reordering of execution.  
- Push-based approach can write intermediate results to buffers accessible by dependent pipelines, with scheduler managing execution order based on data dependencies.  
- Most systems use pull-based for simplicity and ease of implementing LIMIT and control flow.  
- Push-based used in some high-performance OLAP systems (e.g., Hyper, Umbra, DuckDB switched to push-based).  
- Push/pull distinction orthogonal to the choice of iterative, materialization, or vectorized processing models.  

---

[00:39:31]  
**Support for Multiple Processing Models**  
- Theoretically possible for a database engine to support multiple processing models (iterator, materialization, vectorized) within the same system, but engineering complexity and maintenance overhead make this rare.  
- Some systems provide multiple engines (row store with iterator model and column store with vectorized model) under the same product (e.g., Oracle, SQL Server, DB2).  
- Vectorized model with batch size one is essentially iterator model but less efficient.  

---

[00:42:14]  
**Access Methods for Leaf Operators**  
- Access methods define how data is retrieved at the leaf nodes of the query plan.  
- Three basic types:  
  1. **Sequential Scan**: fallback method; scans all pages sequentially from disk or memory, emitting tuples.  
  2. **Index Scan**: uses indexes (B+ tree, skip lists, tries) to locate subsets of tuples matching predicates efficiently.  
  3. **Multi-Index Scan**: combines multiple indexes when predicates involve multiple columns; results are intersected or unioned as appropriate.  
- Sequential scan can be optimized using compression, prefetching, buffer management, clustering, sorting, parallel scanning, and materialized views.  
- Materialized views and result caching—rarely automatic; require manual or incremental maintenance.  

---

[00:47:32]  
**Data Skipping Techniques**  
- Aim to avoid reading irrelevant data blocks.  
- Two approaches:  
  1. **Approximate Queries**: allow approximate answers with statistical guarantees (used for queries not requiring exact precision).  
  2. **Zone Maps (More Common)**: store min, max, null counts, distinct counts per data block (e.g., file, page).  
- Zone maps enable skipping blocks with no matching tuples for a predicate (e.g., skip block if max value &lt; predicate lower bound).  
- Supported by most modern columnar file formats (Parquet, ORC, Apache Arrow).  

---

[00:51:53]  
**Index Scan Selection and Multi-Index Scans**  
- Challenge: selecting the best index when multiple indexes exist for predicates involving multiple columns.  
- Selection depends on predicate selectivity and index type (hash index vs B+ tree).  
- Example: if fewer students under age 30 than in CS department, use age index first, then filter by department.  
- Multi-index scans retrieve candidate tuples from multiple indexes and combine results via intersection (AND) or union (OR).  
- PostgreSQL calls this a bitmap scan, MySQL calls it index merge; all implement similar logic.  
- After index retrieval of record IDs, tuples are fetched from the table for any remaining filters.  

---

[00:57:15]  
**Modifying Queries: Insert, Update, Delete, Upsert, Merge**  
- Update and delete operations reuse scan operators to identify tuples matching predicates.  
- Output tuples are then modified or removed accordingly.  
- Insert may materialize tuples inside the insert operator or receive tuples from upstream operators.  
- Upserts and merges combine scan, update, and insert operations.  
- Truncate operations drop and recreate tables (simple).  
- Important to handle cases where modifying tuples may cause physical location changes, leading to repeated processing.  

---

[00:59:38]  
**Halloween Problem**  
- Occurs when updating tuples during a scan causes their physical location to change, risking the same logical tuple being processed multiple times.  
- Example: increasing salaries may move tuples in an index, causing them to be seen again during the scan.  
- Discovered in IBM’s System R project in the 1970s on Halloween (hence the name).  
- Solution: track already modified or processed tuples to avoid repeated updates/deletes.  
- Requires careful bookkeeping in scan and modification operators.  
- Students will address this in Project 3 and upcoming lectures on concurrency control.  

---

[01:04:03]  
**Expression Evaluation Using Expression Trees**  
- Predicates (WHERE clauses, join conditions) represented as expression trees associated with operators in the query plan.  
- Expressions evaluated per tuple during scan or join operations to determine if tuples satisfy predicates.  
- Example: conjunction of join condition (R.ID = S.ID) and filter (S.value &gt; 100) represented as tree nodes.  
- Supports prepared statements by parameterizing constants in expressions for repeated query execution with different parameters.  
- Execution context maintains current tuple, parameters, and schema offsets for efficient evaluation.  
- Tree traversal for evaluation is depth-first, recursively extracting attribute values, constants, and applying operators.  

---

[01:08:18]  
**Compiling Expression Trees for Performance**  
- Interpreting expression trees per tuple is slow due to overhead of function calls and indirection.  
- Compiling expressions into machine code (JIT compilation) can flatten trees into direct code, improving CPU efficiency.  
- Vectorized execution combined with compiled expressions can leverage SIMD instructions for batch processing.  
- PostgreSQL added JIT compilation for WHERE clauses about five years ago; example shows runtime reduction from ~4 seconds to ~2.7 seconds at the cost of upfront compilation.  
- Trade-off managed by cost model estimating if JIT is worthwhile.  
- DuckDB (columnar, vectorized, push-based) executes similar queries in under 1 second without JIT, showing architectural performance advantages.  

---

[01:15:24]  
**Additional Expression Optimizations**  
- **Constant Folding**: precompute constant expressions once to avoid repeated evaluation.  
- **Common Subexpression Elimination**: reuse results of duplicate expressions appearing multiple times in a query.  
- These optimizations reduce redundant computation and improve query speed without changes to SQL queries.  
- Despite compiler backends (LLVM, GCC) performing similar optimizations, database systems implement these independently to ensure efficiency, especially for partial query compilation or interpretation.  
- Some advanced systems generate assembly and gradually optimize with compiler pipelines, replacing interpreted code dynamically during query execution.  

---

[01:18:52]  
**Summary and Final Remarks**  
- The same SQL query can be executed via different physical execution strategies depending on workload and system design.  
- OLAP workloads benefit from vectorized, push-based execution models; OLTP workloads favor index scans and iterative or materialization models.  
- Expression trees provide a clean abstraction for predicates but can be optimized via compilation and rewriting.  
- Next class will cover parallel execution and combining results from multiple workers.  
- The lecture concludes with a recap and encouragement to understand trade-offs between processing and execution models based on target use cases.  

---

**Key Takeaways:**  
- **Pipelines and pipeline breakers** are critical for efficient query execution without unnecessary materialization.  
- **Iterative, materialization, and vectorized models** represent fundamental ways to move data between operators, each with unique trade-offs.  
- **Pull-based (iterator) vs push-based execution** models influence scheduling and data flow control.  
- **Access methods (sequential scan, index scan, multi-index scan)** underpin efficient data retrieval.  
- The **Halloween problem** is an important anomaly to handle in update/delete queries.  
- **Expression trees** are central to predicate evaluation and can be accelerated by JIT compilation and expression optimizations.  
- Modern OLAP databases almost universally use **vectorized, push-based execution** for high performance.  

**End of Summary**
