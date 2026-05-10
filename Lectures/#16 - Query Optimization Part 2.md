[00:00:01]  
**Introduction and Course Logistics**  
- The class begins with administrative announcements regarding midterm exams, office hours, homework 4 due date, and project 3 recitation materials.  
- Recap of previous topics: the query optimizer's role in transforming SQL queries into executable physical plans.  
- Reminder: the query optimizer pipeline includes parsing, binding (resolving table and column names), logical plan transformations (without cost consideration), and cost-based optimization to select the most efficient physical plan.

[00:01:14]  
**Query Optimizer Overview and Cost-Based Search**  
- The query optimizer generates alternative query plans using transformation rules and evaluates them using a cost model.  
- Cost varies depending on system goals and environment; simplest cost models optimize I/O operations, more sophisticated ones include CPU and memory costs.  
- Quality of query plans depends on:  
  - Transformation rules (logical to logical or logical to physical plans)  
  - Search algorithms (exploring alternative plans)  
  - Cost model (estimating runtime/resource usage to compare plans)  
- Cost model values are internal, relative metrics unique to each database system, not directly comparable across systems.

[00:03:31]  
**Search Algorithms in Query Optimizers**  
- Two main approaches to enumerating query plans:  
  1. **Bottom-up (forward chaining):** Start from leaf nodes (table scans/access methods) and combine upwards to build join orders and physical plans. Uses memoization to avoid redundant computations.  
  2. **Top-down (branch and bound):** Start from desired final output and recursively explore ways to produce it by breaking down into sub-operations. Prunes search paths exceeding current best cost.  
- IBM System R’s early optimizer (1970s) used the top-down approach; many modern systems (PostgreSQL, DB2, DuckDB) adopt bottom-up or hybrid methods with enhancements.  
- Search algorithms use transformation rules such as predicate push-down before cost-based enumeration.

[00:05:49]  
**Bottom-Up Search and Example of Join Ordering**  
- Example query with three tables: artist, appears, album.  
- Bottom-up approach evaluates all table access methods (e.g., sequential scan vs. index scan).  
- Enumerates all possible join orders and join algorithms (hash join, merge join, nested loop join).  
- Dynamic programming (divide and conquer) is used to select the lowest-cost join order at each stage and prune suboptimal plans.  
- Search space explosion is a challenge; heuristics prune search space by:  
  - Restricting to left-deep join trees  
  - Eliminating Cartesian products  
  - Applying known schema patterns and heuristics  
- PostgreSQL switches to approximation methods (genetic algorithms) when queries involve more than 13 tables due to exponential growth in plan enumeration.

[00:10:30]  
**Handling Large Search Spaces and Approximation Algorithms**  
- Exact enumeration is infeasible for very large queries.  
- Approximation or randomized algorithms (e.g., genetic algorithms, simulated annealing) are used to find "good enough" plans quickly, sacrificing guaranteed optimality.  
- PostgreSQL’s genetic algorithm implementation is known to have bugs and is considered unreliable.  
- German research systems use hybrid approaches: exact search up to 100 tables, then approximation seeded with exact results beyond that.  
- These heuristics and approximations are critical for scalability in real-world systems.

[00:13:00]  
**Detailed Bottom-Up Algorithm Walkthrough**  
- For each pair of relations, different join algorithms are costed.  
- The plan with the minimal cost is retained for each subproblem (join subset).  
- This process repeats until the full join order with minimal cost is found.  
- Physical properties like data sorting are considered post hoc or via cost adjustments.  
- Heuristics avoid considering expensive or unlikely join types based on table sizes and statistics.

[00:16:12]  
**Top-Down Search (Cascades Framework)**  
- Top-down search is implemented in SQL Server, Greenplum, CockroachDB, and Databricks Catalyst optimizer.  
- Uses branch and bound search starting from the root with desired output and physical properties (e.g., sorted output).  
- Memorization tables prevent recomputation of costs for subplans.  
- Transformation rules are prioritized to ensure early plans have acceptable cost (e.g., hash joins before merge joins).  
- Physical property enforcement nodes (e.g., sort operators) are added to guarantee output format.  
- This approach can revisit subplans multiple times, making memorization critical for efficiency.

[00:23:10]  
**Cost Models: Purpose and Components**  
- Cost models estimate resource consumption to compare alternative query plans without executing them.  
- They provide relative cost scores, not absolute execution times.  
- Components include:  
  - **Physical cost:** I/O operations (disk read/write), CPU cycles, memory usage, network packets.  
  - **Logical cost:** Estimated number of tuples output by operators (cardinality estimates), which is consistent regardless of physical operator chosen.  
- Most systems prioritize I/O cost due to its dominating effect on query performance.  
- Sophisticated systems (DB2, Oracle) dynamically benchmark hardware to calibrate costs; PostgreSQL uses static “magic constants” configurable by users.

[00:27:44]  
**Challenges in Cost Estimation**  
- Cost depends on current data distributions which may change over time, causing cost models to become stale.  
- PostgreSQL’s cost model is simple, uses fixed weights, and is brittle; more advanced systems incorporate adaptive and hardware-aware costing.  
- Distributed systems add complexity with network costs and heterogeneous storage tiers, which PostgreSQL does not capture.

[00:32:26]  
**Statistics Collection for Cost Estimation**  
- Statistics are metadata summaries of data distributions used by the optimizer to estimate cardinalities and selectivities.  
- Stored as special internal tables (e.g., PostgreSQL’s `pg_statistic`), ensuring durability and transactional consistency.  
- Collection can be triggered manually or automatically based on data changes (e.g., when 10-20% of a table changes).  
- Statistics include:  
  - Number of distinct values per column  
  - Most common values and their frequencies  
  - Histograms representing value distributions  
- Some systems maintain correlated statistics on multiple columns to improve estimation accuracy when attributes are dependent.

[00:39:31]  
**Types of Statistical Data Structures Used**  
| Type        | Description                                                                                 | Usage & Prevalence                             |
|-------------|---------------------------------------------------------------------------------------------|------------------------------------------------|
| Histograms  | Buckets counting frequencies of values or ranges within a column                           | Most common; used in PostgreSQL, Oracle, etc. |
| Sketches    | Probabilistic data structures (e.g., Count-Min Sketch, HyperLogLog) for approximate counts | Increasingly popular; used in some modern systems |
| Sampling    | Maintaining subsets of data to estimate distribution characteristics                        | Rare; used in SQL Server, some German systems  |
| ML Models   | Machine learning approaches to learn data distributions                                    | Research stage; not yet production-ready       |

[00:41:36]  
**Histograms and Their Variants**  
- Storing exact counts for all unique values in a column is impractical for large datasets.  
- **Equi-width histograms:** Buckets have equal width (range), counts aggregated per bucket.  
- **Equi-depth histograms:** Buckets have equal counts, ranges vary; better represent skewed data with heavy hitters.  
- **N-bias histograms:** Store exact counts for top N most frequent values; all others lumped into a catch-all bucket.  
- Histograms approximate selectivity by assuming uniform distribution within buckets, which introduces estimation errors.

[00:46:54]  
**Sketches and Sampling for Statistics**  
- Sketches provide efficient approximate answers for frequency and distinct count queries, and are mergeable.  
- Sampling involves creating small representative subsets to estimate statistics; requires careful handling to avoid query interference.  
- SQL Server uses sampling as part of its optimizer workflow, running `ANALYZE` on demand to update statistics.  
- Sampling and sketches can supplement or partially replace histograms for large or dynamic datasets.

[00:50:16]  
**Cardinality and Selectivity Estimation**  
- Key tasks: estimate how many tuples satisfy predicates, how many tuples join operators output, and distinct values for group-by operations.  
- Cardinality estimates underpin cost models and guide plan selection.  
- Formulas use metadata: total tuples (|R|), distinct values for attribute A (V(A)), and predicate characteristics.  
- Example: For equality predicate `A = a`, selectivity = frequency of `a` / total tuples.  
- In histograms, bucket-based approximations divide bucket counts by number of values per bucket assuming uniformity.

[00:54:21]  
**Assumptions and Limitations in Selectivity Estimation**  
- Uniform distribution assumption: values within a bucket occur with equal probability.  
- Independence assumption: predicates on different attributes are independent, allowing selectivities to be multiplied for conjunctions.  
- These assumptions often fail in real data, leading to significant estimation errors.  
- Systems mitigate errors by tracking heavy hitters and using correlated statistics where possible.

[01:01:01]  
**Problem of Correlated Attributes**  
- Example: Query on `make = Honda AND model = Accord`.  
- Assuming independence, selectivity = selectivity(make) * selectivity(model) = 0.001 (very low).  
- Reality: Accord models only exist for Honda, so selectivity = selectivity(model) = 0.01 (higher).  
- Independence assumption causes underestimation of result size by an order of magnitude.  
- Microsoft SQL Server applies heuristics to reduce impact of such errors by weighting predicates.

[01:03:26]  
**Join Size Estimation and the Containment Principle**  
- Join cardinality estimation is challenging due to complex relationships between tables (1-N, N-M joins).  
- Containment principle: assume every value in one relation matches at least one value in the other, simplifying estimation.  
- Formula approximates join size as:  
  \|R\| * \|S\| / max(V(R.attr), V(S.attr))  
- Without this assumption, cardinality estimation becomes intractable.  
- Under- or overestimation here propagates errors upward in query plans, worsening with multiple joins.

[01:07:04]  
**Error Propagation in Cardinality Estimation**  
- Cardinality estimates at base tables may be reasonable, but errors amplify through joins.  
- Underestimation leads to choosing inefficient join algorithms (e.g., nested loop instead of hash join), degrading performance.  
- Adaptive query processing (AQP) techniques allow runtime plan adjustments if cardinalities differ significantly from estimates, but these are rare and complex.  
- Systems may abort queries and reoptimize when errors are detected, trading off optimization time against execution efficiency.

[01:13:41]  
**Consequences of Estimation Errors**  
- Overestimation: allocating excessive memory or resources; potentially inefficient plans but usually safe.  
- Underestimation: causing plan failures or excessive runtime due to inappropriate join algorithms or resource allocation.  
- PostgreSQL suffers from reallocation problems when hash tables grow beyond estimates, impacting performance.

[01:14:25]  
**Empirical Evaluation of Cardinality Estimation Accuracy**  
- A 2015 German research paper benchmarked cardinality estimation errors across major database systems using IMDb benchmark.  
- Observations:  
  - All systems tend to **underestimate join cardinalities**, with error increasing as number of joins grows.  
  - SQL Server performed better due to sampling-based statistics.  
  - Oracle showed significant estimation errors despite being a commercial high-end system (may have improved since).  
  - PostgreSQL’s simplistic cost model leads to poor estimates compared to others.  
  - German systems showed improvement over time by incorporating better statistics and sampling.  
- Even decades of development and investment have not eliminated cardinality estimation errors in production systems.

[01:19:08]  
**Summary and Outlook**  
- Query optimization is complex and remains an open research challenge due to:  
  - Massive plan search space  
  - Imperfect cost models relying on approximated statistics  
  - Changing data distributions and correlations  
- Cost models provide relative plan costs but suffer inaccuracies from assumptions like uniformity and independence.  
- Statistical metadata (histograms, sketches, sampling) enable estimations but have limitations and overheads.  
- Advanced topics like adaptive query processing, correlated statistics, and ML-based models are emerging but not widespread in production.  
- The next lecture will cover **transactions**, addressing correctness and concurrency, another challenging aspect of database systems.

---

**Key Concepts and Terms**  
| Term                     | Definition                                                                                      |
|--------------------------|------------------------------------------------------------------------------------------------|
| Query Optimizer          | Component that generates and selects an efficient physical execution plan from a SQL query.    |
| Transformation Rules     | Rules to rewrite logical/physical plans to equivalent alternatives for optimization.            |
| Search Algorithms        | Methods (bottom-up or top-down) to enumerate and evaluate alternative query plans.             |
| Cost Model               | Internal metric to estimate resource consumption of query plans for comparison, not absolute.  |
| Memoization              | Technique to cache results of sub-plans to avoid redundant computations during optimization.   |
| Histograms               | Statistical summaries that bucket column values and frequencies for selectivity estimation.    |
| Sketches                 | Probabilistic data structures approximating frequency and distinct counts (e.g., Count-Min).   |
| Sampling                 | Using a subset of data to estimate statistics for the full dataset.                            |
| Cardinality Estimation   | Predicting the number of tuples produced by query operators, key to cost estimation.           |
| Containment Principle    | Assumption that all join attribute values in one relation appear in the other, simplifying joins. |
| Adaptive Query Processing| Runtime techniques to adjust query plans based on actual observed data during execution.       |

---

**Summary Table: Cost Model Components**

| Component       | Description                                   | Example Factors                        | Notes                                   |
|-----------------|-----------------------------------------------|-------------------------------------|-----------------------------------------|
| Physical Cost   | Resource usage during query execution         | Disk I/O, CPU cycles, memory, network | Dominated by I/O in most systems         |
| Logical Cost    | Estimated output size of each operator        | Number of tuples output per operator  | Independent of physical operator choice  |
| Statistics     | Metadata summaries used for cardinality estimation | Histograms, sketches, sampling        | Collection/update frequency varies       |
| Assumptions    | Simplifications for tractability               | Uniform distribution, predicate independence | Cause estimation errors                   |

---

**References and Further Reading**  
- Microsoft’s open-source book on SQL Server query optimizer (recommended for deep dive).  
- Research papers on Cascades framework and join ordering algorithms.  
- 2015 German benchmark paper on cardinality estimation error in database systems.  

---

**Conclusion**  
This lecture provides a comprehensive overview of the **query optimization process**, focusing on the **search algorithms**, **cost modeling**, and **statistical cardinality estimation** that underpin cost-based query optimization. It highlights the inherent complexity and approximation involved in generating efficient execution plans and sets the stage for upcoming topics on transactions and concurrency.
