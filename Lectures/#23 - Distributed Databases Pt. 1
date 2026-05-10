[00:00:07]  
**Course Logistics and Upcoming Deadlines**  
- Project 4b due September 7th.  
- Homework 6 to be released later today, due December 7th.  
- Final exam scheduled for December 11th; study guide and practice exam will be posted on the course platform (PATA).  
- The final exam is cumulative with emphasis on material post-midterm, including SQL, relational algebra, and buffer pool concepts.  
- Cheat sheet size for the final exam will be the same as for the midterm.  
- Students interested in being teaching assistants (TAs) for the next semester encouraged to sign up, whether they enjoyed or disliked the course codebase, as feedback is valuable.  

[00:02:22]  
**Upcoming Database Talks and Seminar Series**  
- Today: Presentation on XTDB, a time series database from the UK focusing on multi-dimensional time as a first-class concept in tuples.  
- Next week: Snowflake’s open source Apache Polaris catalog, an alternative to Apache Iceberg catalog after Snowflake couldn’t acquire the original Iceberg team.  
- End of semester: A talk on an Apache project called Flu (details *Not specified/Uncertain*).  
- Spring semester seminar series will continue with Zoom access for alumni and former students.  

[00:04:09]  
**Course Progress and Focus for Final Lectures**  
- Covered building a single-node database: transactional capabilities, fault tolerance, query planning, and execution.  
- Current focus: Introduction to **distributed database systems**, which add complexity beyond single-node systems.  
- Distributed databases involve splitting architecture across multiple nodes, raising challenges in communication, fault tolerance, and query execution.  
- Emphasis on applying all single-node database fundamentals to a multi-node environment.  
- Key challenges: communication speed and reliability, coordination across nodes (e.g., buffer pools), and heterogeneous hardware resources.  

[00:05:28]  
**Distinction Between Parallel and Distributed Database Systems**  
- **Parallel database systems**: multiple cores or CPUs within a single machine; communication is fast, reliable, and assumed to be synchronous (e.g., inter-process communication).  
- **Distributed database systems**: nodes are physically separated, possibly geographically; communication is much slower (milliseconds to seconds), potentially unreliable, and subject to network failures or delays such as garbage collection pauses.  
- Distributed systems require **consensus protocols** like Paxos and Raft to handle coordination and fault tolerance.  
- All transaction and query correctness principles still apply but require more complex coordination.  

[00:07:49]  
**Key High-Level Questions for Distributed Database Design**  
- How does an application locate data distributed across nodes?  
- How to partition data across nodes (per table, per row, fine-grained, etc.)?  
- How does query execution work when required data is on multiple nodes? Push query to data or pull data to query?  
- How is consistency and correctness guaranteed across distributed nodes?  
- How to handle fault tolerance and availability through replication?  

[00:10:27]  
**Lecture Plan for Distributed Database Systems**  
- Discuss system architectures (shared everything, shared nothing, shared disk, shared memory).  
- Partitioning schemes and data distribution strategies.  
- Replication strategies for fault tolerance and consistency.  
- Distributed transaction management.  
- Next class will cover distributed control and distributed OLAP query execution, including distributed joins.  

[00:11:43]  
**Distributed Database Architectures Explained**  

| Architecture      | CPU           | Memory         | Disk                    | Communication Method             | Notes                                                                                   |
|-------------------|---------------|----------------|-------------------------|--------------------------------|-----------------------------------------------------------------------------------------|
| Shared Everything | Single CPU    | Local memory   | Local disk              | None (single node)              | Traditional single-node DB architecture.                                               |
| Shared Nothing    | Multiple CPUs | Local memory   | Local disk per node     | Network (usually TCP/IP)        | Nodes own data; no direct access to other nodes’ memory/disk; communication via network.|
| Shared Disk       | Multiple CPUs | Local memory   | Shared global disk (e.g., S3) | Network + shared disk access    | Nodes share disk but have separate CPUs and memory; storage is centralized.             |
| Shared Memory     | Stateless CPUs| Shared memory  | Shared disk             | Shared memory access            | Theoretical/academic; rarely used outside HPC/scientific computing.                     |

- **Shared Everything** corresponds to single node systems.  
- **Shared Nothing** is the classic distributed approach with separate resources and network communication.  
- **Shared Disk** is common in cloud data lakes/serverless systems using distributed object stores like S3, Azure Blob Storage, or Google Cloud Storage.  
- **Shared Memory** is mostly theoretical and not common in commercial DBMSs.  

[00:12:41]  
**Shared Nothing Architecture Details**  
- Each node has exclusive CPU, memory, and disk.  
- Inter-node communication is via network protocols (TCP/IP mostly, sometimes UDP or RDMA).  
- Best performance for distributed queries localized to single nodes due to local data access.  
- Challenges scaling out due to data movement when adding/removing nodes.  
- Example: 1986 paper by Mike Stonebraker advocating shared nothing architectures.  

[00:17:57]  
**Shared Disk Architecture Details**  
- Nodes have local CPU and memory with local disk used mainly for caching.  
- Primary database storage on shared distributed file system (e.g., S3).  
- No need to physically move data when scaling nodes, only logical reassignments in metadata/catalog.  
- Widely adopted in modern cloud data warehouses and lakehouse systems (Snowflake, Databricks, Firebolt, Yellowbrick).  
- Tradeoff: higher latency due to network I/O for disk access compared to local disk in shared nothing.  
- Caching at nodes helps mitigate some latency but consistency and synchronization of cache is a challenge.  

[00:32:36]  
**Query Execution Strategies: Push vs. Pull**  

| Strategy       | Description                                                                                         | Pros                              | Cons                                  |
|----------------|-------------------------------------------------------------------------------------------------|----------------------------------|--------------------------------------|
| Push Query     | Send computation (query fragments) to the node where data resides and execute there               | Reduces data transfer over network | Requires compute resources on data node |
| Pull Data      | Transfer data to query node and run query locally                                                | Simplifies query execution logic  | Potentially large data transfer costs  |

- These methods can be combined.  
- Example: S3 Select allows pushing simple predicates (filters) down to the storage layer to minimize data transferred.  
- S3 Select deprecated for new accounts but still useful for existing users; limitations include lack of complex computations like joins.  
- Trade-offs include latency, cost, and loss of control over storage layer operations.  

[00:41:25]  
**Partitioning and Data Distribution**  
- Partitioning splits the database across nodes based on selected columns (partition keys).  
- Types of partitioning:  
  - Table partitioning: whole tables assigned to nodes (coarse-grained).  
  - Horizontal partitioning (sharding): rows partitioned by key, spread across nodes (fine-grained).  
  - Predicate partitioning: data split by specified WHERE clauses (rare).  
  - Round-robin partitioning: assigning data evenly but arbitrarily.  
- Most common: **horizontal partitioning** using hash or range partitioning.  

[00:53:23]  
**Hash Partitioning Explained**  
- Hash partitioning computes hash(key) mod number_of_partitions to assign data.  
- Works well if queries filter on partition key.  
- Problems:  
  - Range queries are inefficient because data is scattered.  
  - Adding/removing nodes changes partition count, causing many data relocations.  

[00:56:30]  
**Solutions to Partitioning Challenges: Consistent Hashing and Rendezvous Hashing**  

| Technique            | Description & Benefits                                                                                           |
|----------------------|-----------------------------------------------------------------------------------------------------------------|
| Consistent Hashing   | Maps nodes and keys on a hash ring [0,1]. Adding/removing nodes only moves data for affected partitions, minimizing reshuffling. Used by Amazon DynamoDB, Cassandra, Couchbase. May require virtual nodes for load balancing. |
| Rendezvous Hashing   | For each key, compute hash with each node identifier; assign to node with highest hash value. Efficient, stable, better load distribution. Often outperforms consistent hashing in lookup speed and balance. |

- Both techniques reduce rebalancing overhead when cluster membership changes.  
- Support replication by assigning copies to successive nodes on the hash ring.  

[01:05:10]  
**Replication Strategies for Fault Tolerance**  

| Replication Model | Description                                                                                         | Pros                                           | Cons                                             |
|-------------------|-------------------------------------------------------------------------------------------------|------------------------------------------------|--------------------------------------------------|
| Primary-Replica   | One primary node handles writes; replicas asynchronously or synchronously replicate data updates. | Simpler conflict management, leader election for failover. | Potential stale reads on replicas, leader bottleneck. |
| Multi-Primary     | Multiple nodes can accept writes; must resolve conflicts (e.g., via consensus or conflict resolution). | Higher availability; no single point of write failure. | Complex conflict resolution, coordination overhead. |

- Primary-replica is most common in enterprise systems (e.g., PostgreSQL clusters).  
- Leader election protocols (Paxos, Raft) used to handle failover and elect new primaries.  
- In multi-primary, conflicts may arise if concurrent writes happen on different nodes; transaction ordering and conflict resolution are critical.  

[01:11:21]  
**Propagation Timing in Replication**  
- **Synchronous propagation**: Primary waits for replicas to acknowledge writes before confirming commit to client (strong consistency).  
- **Asynchronous propagation**: Primary commits immediately, propagates updates later (higher performance, eventual consistency).  

Trade-offs:  
- Synchronous offers durability guarantees but higher latency.  
- Asynchronous improves throughput but risks data loss if primary fails before replicas catch up.  

[01:15:46]  
**Distributed Transaction Coordination**  

| Coordination Model     | Description                                                                                                   |
|-----------------------|---------------------------------------------------------------------------------------------------------------|
| Centralized Coordinator | A global transaction manager handles lock requests, transaction commits, and coordination across partitions. Uses two-phase commit (2PC) protocols. |
| Decentralized Coordination | Nodes coordinate among themselves, potentially using leader election for commit decisions, without a single global coordinator. |

- Centralized approach resembles traditional TP monitors from the 80s and 90s (e.g., BEA, Transarc).  
- Middleware-based centralized coordination abstracts multiple nodes behind a single logical database interface (used by Google pre-Spanner, Facebook MySQL clusters).  
- Decentralized coordination is more scalable but more complex to implement and reason about.  

[01:22:17]  
**Challenges in Distributed Transactions**  
- Transactions touching multiple partitions require distributed concurrency control and commit protocols.  
- Issues like deadlocks, latency, and failure recovery become more complex in distributed environments.  
- Network latency varies widely between local racks and geographically distributed data centers, affecting performance.  
- Next class will delve deeper into distributed OLTP and OLAP query execution strategies, including multi-node joins.  

[01:23:15]  
**Federated Databases**  
- Middleware providing a unified query interface over heterogeneous database systems (e.g., Trino, Presto, PostgreSQL Foreign Data Wrappers).  
- Challenges include supporting efficient predicate pushdown and minimizing data movement.  
- Tend to be less common and less performant due to heterogeneity and lowest-common-denominator limitations.  

[01:24:10]  
**Summary of Parallel vs. Distributed Databases**  
- Parallel databases run on single machines with many CPUs and shared storage (network-attached storage, NAS).  
- Distributed databases span multiple machines and data centers.  
- Differences impact architectural choices for concurrency control, storage, and communication.  

---

### **Key Insights and Conclusions**  
- Distributed database systems extend single-node principles into a multi-node environment with significant challenges around communication, consistency, and fault tolerance.  
- Architectures vary from shared everything (single node) to shared nothing (classic distributed) and shared disk (modern cloud data lakes), each with trade-offs in scalability, performance, and manageability.  
- Partitioning is a fundamental technique to distribute data, with hash and range partitioning dominating; sophisticated hashing schemes (consistent, rendezvous) mitigate data movement during scaling.  
- Replication enables fault tolerance with trade-offs between consistency and performance, managed via synchronous or asynchronous propagation.  
- Distributed transaction coordination can be centralized or decentralized, with established protocols like two-phase commit and consensus algorithms crucial for correctness.  
- Modern cloud data warehouses favor shared disk architectures leveraging distributed object stores (e.g., S3) and open file formats like Parquet to simplify data management and scalability.  
- Middleware and federated databases provide abstraction layers for heterogeneous systems but face inherent performance challenges.  

This lecture lays the groundwork for understanding distributed database systems’ architecture and design, setting the stage for more detailed treatments of distributed query execution and transaction management in subsequent sessions.
