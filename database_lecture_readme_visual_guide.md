# Database Systems Lecture Notes – README

> Comprehensive visual notes and explanations from the introductory database systems lecture.  
> Source lecture transcript: fileciteturn0file0

---

# Table of Contents

1. Introduction to the Course
2. What is a Database?
3. What is a DBMS?
4. Problems with CSV/File-Based Storage
5. What is a Data System?
6. Data Models and Schema
7. Historical Evolution of Databases
8. Relational Model Fundamentals
9. Data Independence
10. Relational Terminology
11. Keys and Relationships
12. Constraints
13. Query Languages
14. Relational Algebra
15. Query Optimization
16. Key Takeaways

---

# 1. Introduction to the Course

The lecture introduces a course focused on:

- Database internals
- Database system design
- Relational data model
- Query processing
- Data systems architecture

The course emphasizes:

✅ Understanding how databases work internally  
✅ Building database components in C++  
✅ Learning relational algebra and SQL foundations  
✅ Understanding performance and scalability

---

# 2. What is a Database?

A **database** is a collection of related data representing real-world entities.

## Example

A music application may store:

- Artists
- Albums
- Songs
- Users
- Playlists

---

## Visual Diagram

```text
+----------------+
|    Database    |
+----------------+
        |
        +-------------------+
        |                   |
   +---------+         +---------+
   | Artists |         | Albums  |
   +---------+         +---------+
        |                   |
        +---------+---------+
                  |
             Relationships
```

---

# 3. What is a DBMS?

A **Database Management System (DBMS)** is software that manages databases.

Examples:

- PostgreSQL
- MySQL
- SQLite
- Oracle
- Redis
- ClickHouse

---

## Responsibilities of a DBMS

A DBMS:

- Stores data
- Retrieves data efficiently
- Handles concurrency
- Prevents corruption
- Provides recovery
- Optimizes queries
- Enforces constraints

---

## Visual Architecture

```text
+-------------------+
|   Application     |
+-------------------+
          |
          v
+-------------------+
|       DBMS        |
|-------------------|
| Query Engine      |
| Storage Engine    |
| Transaction Mgmt  |
| Recovery System   |
| Optimizer         |
+-------------------+
          |
          v
+-------------------+
|     Database      |
+-------------------+
```

---

# 4. Problems with CSV/File-Based Storage

The lecture strongly explains why storing application data directly in files is dangerous and inefficient.

## Example CSV Storage

```csv
artist_id,name
1,Taylor Swift
2,Drake
```

This looks simple initially but creates major problems.

---

# Major Problems

## 4.1 Linear Scans

Finding data requires scanning the entire file.

```text
Search Artist ID = 50000

[1][2][3][4]...[50000]
             ^
         finally found
```

Time complexity becomes very expensive for large datasets.

---

## 4.2 No Type Safety

Everything is stored as text.

```text
"123"
"abc"
"2025"
```

No validation exists.

---

## 4.3 No Relationships

A CSV cannot naturally enforce:

- Foreign keys
- Many-to-many relationships
- Referential integrity

---

## 4.4 Concurrency Problems

Two threads writing simultaneously:

```text
Thread A ---> write file
Thread B ---> write file

Possible Result:
CORRUPTED DATA
```

---

## 4.5 No Crash Recovery

If power fails during a write:

```text
WRITE STARTED
SYSTEM CRASHED
FILE HALF WRITTEN
```

Data corruption may occur.

---

## 4.6 No Scalability

Single-machine storage becomes a bottleneck.

---

## Core Conclusion

> Do not reinvent database functionality in application code.

Use mature DBMS systems instead.

---

# 5. What is a Data System?

A **data system** stores and processes application data efficiently.

Examples:

| System | Type |
|---|---|
| PostgreSQL | Relational DB |
| MongoDB | Document Store |
| Redis | Key-Value Store |
| ClickHouse | Analytical DB |

---

# 6. Data Models and Schema

## Data Model

Defines:

- How data is represented
- Relationships
- Constraints
- Allowed operations

---

## Schema

Defines the actual structure of a specific database.

Example:

```sql
CREATE TABLE Artists(
    id INT,
    name TEXT
);
```

---

# Architecture Analogy

```text
Data Model  -> Rules of construction
Schema      -> Blueprint of one building
```

---

# Types of Data Models

## Relational Model

```text
Tables + Rows + Columns
```

## Document Model

```json
{
  "name": "Kartik",
  "skills": ["C++", "SQL"]
}
```

## Key-Value Model

```text
key -> value
```

---

# 7. Historical Evolution of Databases

## Early Systems (1960s-1970s)

Before relational databases:

- Data navigation was manual
- Programmers traversed records directly
- Queries were tightly coupled to storage layout

---

## Navigational Database Model

```text
Customer
   |
   +--> Orders
            |
            +--> Products
```

Programmers manually traversed pointers.

This was:

❌ Complex  
❌ Error-prone  
❌ Hard to maintain

---

# Relational Revolution

The relational model introduced:

✅ Data independence  
✅ Declarative queries  
✅ Simpler application development

Key contributors:

- Ted Codd
- Jim Gray
- Mike Stonebraker
- Charles Bachman

---

# 8. Relational Model Fundamentals

The relational model is the foundation of modern databases.

---

# Core Principles

## 8.1 Data Stored as Relations

A relation = table.

```text
+----+---------+
| ID | Name    |
+----+---------+
| 1  | Alice   |
| 2  | Bob     |
+----+---------+
```

---

## 8.2 Relationships via Values

Relationships are created using values instead of physical pointers.

```text
Artist.id = Album.artist_id
```

---

## 8.3 Constraints Enforce Correctness

The DBMS prevents invalid data.

---

## 8.4 Declarative Queries

Users specify:

```text
WHAT they want
```

NOT:

```text
HOW to retrieve it
```

The DBMS decides execution strategy.

---

# Relational Query Example

## SQL

```sql
SELECT name
FROM Artists
WHERE id = 1;
```

The user does not specify:

- Index usage
- Scan method
- Join strategy
- Memory management

The DBMS optimizer handles all of that.

---

# 9. Data Independence

One of the most important database concepts.

---

# Three Schema Levels

```text
+-----------------------+
| External Schema       |
| (Views / APIs)        |
+-----------------------+
            |
+-----------------------+
| Logical Schema        |
| Tables / Relations    |
+-----------------------+
            |
+-----------------------+
| Physical Schema       |
| Files / Pages / Disk  |
+-----------------------+
```

---

## Physical Schema

How data is physically stored:

- Files
- Pages
- Blocks
- Indexes

---

## Logical Schema

Defines:

- Tables
- Columns
- Data types
- Relationships

---

## External Schema

Customized views for applications.

Example:

```sql
CREATE VIEW PublicUsers AS
SELECT name, email
FROM Users;
```

Sensitive fields can be hidden.

---

# Why Data Independence Matters

Applications continue working even if:

- Storage format changes
- Indexes change
- Files move
- Internal optimizations evolve

This is one of the biggest strengths of relational systems.

---

# 10. Relational Terminology

| Term | Meaning |
|---|---|
| Relation | Table |
| Tuple | Row |
| Attribute | Column |
| Domain | Allowed values |
| Null | Missing/unknown value |

---

# Example Relation

```text
Students

+----+---------+-----+
| ID | Name    | Age |
+----+---------+-----+
| 1  | Alice   | 21  |
| 2  | Bob     | 22  |
+----+---------+-----+
```

- Relation = Students
- Tuple = one row
- Attribute = ID/Name/Age
- Domain = integer/string constraints

---

# 11. Keys and Relationships

## Primary Key

Uniquely identifies a row.

```sql
PRIMARY KEY(id)
```

---

## Foreign Key

References another table.

```sql
artist_id REFERENCES Artists(id)
```

---

# Visual Relationship Diagram

```text
+-----------+        +-----------+
| Artists   |        | Albums    |
+-----------+        +-----------+
| id        |<------ | artist_id |
| name      |        | title     |
+-----------+        +-----------+
```

---

# Many-to-Many Relationships

Example:

- One artist → many albums
- One album → multiple artists

Requires a join table.

---

# Join Table Diagram

```text
+---------+
| Artists |
+---------+
      |
      |
+----------------+
| Artist_Album   |
+----------------+
      |
      |
+---------+
| Albums  |
+---------+
```

---

# Synthetic Keys

Sometimes natural values are not reliable.

Example:

```text
Artist names may duplicate.
```

So databases generate IDs:

```text
1
2
3
4
```

---

# 12. Constraints

Constraints enforce correctness.

---

# Common Constraints

## NOT NULL

```sql
name TEXT NOT NULL
```

---

## CHECK Constraint

```sql
CHECK(age > 0)
```

---

## UNIQUE Constraint

```sql
UNIQUE(email)
```

---

## FOREIGN KEY Constraint

Ensures referenced data exists.

---

# Constraint Enforcement Flow

```text
INSERT DATA
     |
     v
CHECK CONSTRAINTS
     |
     +---- valid ---> STORE
     |
     +---- invalid -> ERROR
```

---

# 13. Query Languages

Two major styles exist.

---

# Procedural Queries

Specify:

```text
HOW to retrieve data
```

Example:

- Relational algebra

---

# Declarative Queries

Specify:

```text
WHAT data is needed
```

Example:

- SQL

---

# Comparison

| Procedural | Declarative |
|---|---|
| Explicit execution steps | Desired result only |
| More control | Easier to write |
| Harder to optimize manually | DBMS optimizer handles execution |

---

# 14. Relational Algebra

Relational algebra is the mathematical foundation of SQL.

---

# Seven Core Operators

| Operator | Symbol | Purpose |
|---|---|---|
| Select | σ | Filter rows |
| Project | π | Choose columns |
| Union | ∪ | Combine relations |
| Intersection | ∩ | Common tuples |
| Difference | − | Subtract tuples |
| Cartesian Product | × | All combinations |
| Join | ⋈ | Combine related tables |

---

# 14.1 Select Operator

Filters rows.

```text
σ(A = 2)(R)
```

Equivalent SQL:

```sql
SELECT * FROM R WHERE A = 2;
```

---

# Visual Example

```text
R
+---+---+
| A | B |
+---+---+
| 1 | x |
| 2 | y |
| 2 | z |
+---+---+

σ(A=2)(R)

+---+---+
| 2 | y |
| 2 | z |
+---+---+
```

---

# 14.2 Project Operator

Chooses columns.

```text
π(B)(R)
```

Equivalent SQL:

```sql
SELECT B FROM R;
```

---

# Visual Example

```text
R
+---+---+
| A | B |
+---+---+
| 1 | x |
| 2 | y |
+---+---+

π(B)(R)

+---+
| B |
+---+
| x |
| y |
+---+
```

---

# 14.3 Union

Combines rows from two relations.

```text
R ∪ S
```

Both relations must have compatible schemas.

---

# 14.4 Cartesian Product

Produces all possible combinations.

```text
R × S
```

---

# Example

```text
R = {1,2}
S = {A,B}

R × S =
(1,A)
(1,B)
(2,A)
(2,B)
```

---

# 14.5 Join Operator

Most important operator.

Combines related rows.

```text
Artists ⋈ Albums
```

---

# Join Visualization

```text
Artists
+----+-------+
| id | name  |
+----+-------+
| 1  | Drake |
+----+-------+

Albums
+-----------+---------+
| artist_id | title   |
+-----------+---------+
| 1         | Views   |
+-----------+---------+

JOIN RESULT

+-------+-------+
| Drake | Views |
+-------+-------+
```

---

# 15. Query Optimization

The lecture emphasizes:

> Different execution strategies produce vastly different performance.

---

# Example

## Bad Plan

```text
JOIN huge tables first
THEN filter rows
```

Very expensive.

---

## Better Plan

```text
FILTER rows first
THEN join smaller datasets
```

Much faster.

---

# Query Optimization Diagram

```text
          SQL Query
               |
               v
       +---------------+
       | Query Planner |
       +---------------+
               |
               v
       +---------------+
       | Best Plan     |
       +---------------+
               |
               v
         Query Result
```

---

# Why SQL is Powerful

The user writes:

```sql
SELECT * FROM Orders WHERE price > 100;
```

The DBMS internally decides:

- Index scan
- Sequential scan
- Hash join
- Merge join
- Parallel execution
- Memory allocation

This abstraction is one of the greatest strengths of relational systems.

---

# 16. Key Takeaways

## Database vs DBMS

```text
Database = Data
DBMS      = Software managing the data
```

---

## Relational Model Dominates Modern Systems

Because it provides:

- Simplicity
- Flexibility
- Data independence
- Declarative querying
- Strong correctness guarantees

---

## Relational Algebra is the Foundation of SQL

Understanding relational algebra helps understand:

- Query execution
- Optimization
- SQL internals
- Database engines

---

## Mature DBMS Systems Matter

Databases solve extremely hard problems:

- Concurrency
- Recovery
- Consistency
- Scaling
- Optimization
- Durability

Building these correctly is difficult.

---

# Final Mental Model

```text
                Applications
                      |
                      v
              +---------------+
              |      SQL      |
              +---------------+
                      |
                      v
              +---------------+
              |     DBMS      |
              |---------------|
              | Optimizer     |
              | Transactions  |
              | Storage       |
              | Recovery      |
              +---------------+
                      |
                      v
                Physical Data
```

---

# Suggested Next Topics

After this lecture, important follow-up concepts include:

1. SQL fundamentals
2. Relational algebra deeply
3. Indexing
4. Transactions and ACID
5. Query execution engines
6. Storage systems
7. Concurrency control
8. Recovery mechanisms
9. Distributed databases
10. Modern cloud data systems

---

# End Notes

This lecture establishes the foundation for understanding:

- Why databases exist
- Why relational systems dominate
- How DBMSs abstract complexity
- Why query optimization matters
- How modern data systems evolved historically

It sets the stage for deeper exploration into database internals and system design.

