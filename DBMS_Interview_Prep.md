# DBMS — Interview Preparation

## 1. What is DBMS?
A Database Management System is software that allows creation, storage, retrieval, updating, and management of data in a structured way, providing controlled access, data integrity, and security — unlike a plain file system.

## 2. DBMS vs RDBMS
DBMS stores data as files with no relationships enforced between them. RDBMS (Relational DBMS) stores data in tables (relations) with rows and columns, enforces relationships via keys, and follows ACID properties and normalization rules (e.g., MySQL, PostgreSQL, Oracle).

## 3. Three-Level Architecture (ANSI-SPARC)
- **Internal (Physical) Level**: how data is physically stored on disk.
- **Conceptual (Logical) Level**: describes what data is stored and relationships (schema).
- **External (View) Level**: how individual users see the data (custom views).
This provides **data independence** — changes at one level don't affect others.

## 4. Data Independence
- **Logical Data Independence**: ability to change the conceptual schema without changing external schemas/applications.
- **Physical Data Independence**: ability to change the internal/physical storage without changing the conceptual schema.

## 5. Keys in DBMS
- **Primary Key**: uniquely identifies each record; cannot be NULL or duplicate.
- **Candidate Key**: a minimal set of attributes that can uniquely identify a record; a table can have multiple candidate keys, one becomes the primary key.
- **Super Key**: any set of attributes (including extra ones) that uniquely identifies a record.
- **Foreign Key**: an attribute in one table that references the primary key of another table, enforcing referential integrity.
- **Composite Key**: a primary key made of two or more columns.
- **Alternate Key**: candidate keys not chosen as the primary key.
- **Unique Key**: ensures all values are unique but allows one NULL (unlike primary key).

## 6. Normalization
The process of organizing data to reduce redundancy and avoid anomalies (insertion, update, deletion).
- **1NF**: eliminate repeating groups; each cell must have atomic (indivisible) values.
- **2NF**: 1NF + no partial dependency (non-key attributes fully depend on the whole primary key, relevant for composite keys).
- **3NF**: 2NF + no transitive dependency (non-key attributes depend only on the primary key, not on other non-key attributes).
- **BCNF (Boyce-Codd NF)**: stricter 3NF — for every functional dependency X→Y, X must be a super key.
- **4NF**: no multi-valued dependency.
- **5NF**: no join dependency; data can be reconstructed from smaller tables without redundancy.

## 7. Denormalization
Intentionally introducing redundancy into a normalized database to improve read performance, at the cost of write performance and data integrity — often used in data warehousing/reporting systems.

## 8. Functional Dependency
A constraint where one attribute (or set) determines another: X → Y means the value of X uniquely determines the value of Y.

## 9. ACID Properties
- **Atomicity**: a transaction is all-or-nothing; if any part fails, the whole transaction rolls back.
- **Consistency**: a transaction brings the database from one valid state to another, maintaining all defined rules/constraints.
- **Isolation**: concurrent transactions execute as if they were run serially, without interfering with each other.
- **Durability**: once a transaction is committed, changes persist even after a system crash.

## 10. Transaction States
Active → Partially Committed → Committed, or Active → Failed → Aborted.

## 11. Concurrency Control
Techniques to manage simultaneous transactions without conflicts:
- **Lock-based protocols**: shared (read) locks and exclusive (write) locks.
- **Two-Phase Locking (2PL)**: growing phase (acquire locks) and shrinking phase (release locks); guarantees serializability.
- **Timestamp-based protocols**: transactions ordered by timestamps.
- **Optimistic Concurrency Control**: assume no conflict, validate before commit.

## 12. Deadlock
A situation where two or more transactions wait indefinitely for locks held by each other. Handled via:
- **Prevention**: ordering resource requests, wait-die/wound-wait schemes.
- **Detection & Recovery**: wait-for graphs to detect cycles, then abort a transaction.

## 13. Isolation Levels
- **Read Uncommitted**: allows dirty reads.
- **Read Committed**: prevents dirty reads.
- **Repeatable Read**: prevents dirty and non-repeatable reads.
- **Serializable**: highest level, prevents dirty reads, non-repeatable reads, and phantom reads.

## 14. Concurrency Anomalies
- **Dirty Read**: reading uncommitted data from another transaction.
- **Non-Repeatable Read**: getting different values on re-reading the same row within a transaction because another transaction modified it.
- **Phantom Read**: new rows appear/disappear in repeated queries due to another transaction's insert/delete.

## 15. Indexing
A data structure (commonly B-Tree or B+ Tree) that improves the speed of data retrieval at the cost of extra storage and slower writes.
- **Primary Index**: on the primary key, usually on a sorted file.
- **Secondary Index**: on non-key attributes for faster lookup.
- **Clustered Index**: determines the physical order of data in the table (only one per table).
- **Non-Clustered Index**: a separate structure with pointers to actual data rows (multiple allowed).

## 16. Joins
- **Inner Join**: returns matching rows from both tables.
- **Left (Outer) Join**: all rows from the left table + matched rows from the right (NULLs if no match).
- **Right (Outer) Join**: all rows from the right table + matched rows from the left.
- **Full Outer Join**: all rows from both tables, matched where possible.
- **Self Join**: a table joined with itself.
- **Cross Join**: Cartesian product of two tables.

## 17. SQL Command Categories
- **DDL** (Data Definition Language): CREATE, ALTER, DROP, TRUNCATE.
- **DML** (Data Manipulation Language): SELECT, INSERT, UPDATE, DELETE.
- **DCL** (Data Control Language): GRANT, REVOKE.
- **TCL** (Transaction Control Language): COMMIT, ROLLBACK, SAVEPOINT.

## 18. DELETE vs TRUNCATE vs DROP
- **DELETE**: removes rows (with optional WHERE), can be rolled back, triggers fire, slower (logged row by row).
- **TRUNCATE**: removes all rows, cannot use WHERE, minimal logging, faster, resets identity counters, generally cannot be rolled back (varies by DB).
- **DROP**: removes the entire table structure along with data.

## 19. Views
A virtual table based on the result of a SQL query; does not store data physically (usually) but simplifies complex queries, provides security by restricting column/row access.

## 20. Stored Procedures and Triggers
- **Stored Procedure**: a precompiled set of SQL statements stored in the database, callable with parameters, improves performance and reusability.
- **Trigger**: a procedure that automatically executes in response to certain events (INSERT, UPDATE, DELETE) on a table.

## 21. ER Model
Entity-Relationship model represents entities (objects), attributes (properties), and relationships (associations) graphically, later converted into relational tables.
- **Entity**: a real-world object (e.g., Student).
- **Attribute**: property of an entity (e.g., Name, ID).
- **Relationship**: association between entities (e.g., Enrolls).
- **Cardinality**: defines the numeric relationship between entities — 1:1, 1:N, M:N.

## 22. Aggregate Functions
COUNT, SUM, AVG, MIN, MAX — operate on a set of values to return a single summary value; often used with GROUP BY and filtered with HAVING (not WHERE, since WHERE can't filter aggregated results).

## 23. WHERE vs HAVING
WHERE filters rows before grouping; HAVING filters groups after the GROUP BY aggregation.

## 24. Referential Integrity
Ensures that a foreign key value always points to an existing, valid primary key value in the referenced table (or is NULL).

## 25. Schedule and Serializability
A **schedule** is a sequence of operations from multiple transactions. It is **serializable** if its outcome is equivalent to some serial (one-at-a-time) execution of those transactions — ensuring correctness under concurrency.

## 26. CAP Theorem (relevant for distributed DBMS/NoSQL)
In a distributed system, you can guarantee at most two of: **Consistency**, **Availability**, **Partition Tolerance** — not all three simultaneously.

## 27. SQL vs NoSQL
SQL databases are relational, schema-based, and ACID-compliant (MySQL, PostgreSQL). NoSQL databases are non-relational, schema-flexible, and optimized for scalability and specific data models — document (MongoDB), key-value (Redis), column-family (Cassandra), graph (Neo4j).

---

## 28. Normalization Worked Through One Table
Start with an unnormalised table:

| StudentID | Name | Courses | DeptID | DeptName | HOD |
|---|---|---|---|---|---|
| 1 | Praneeth | DBMS, OS | D1 | AMCS | Dr. X |

**1NF** — every cell must be atomic and there must be no repeating groups. Split `Courses`:

| StudentID | Name | Course | DeptID | DeptName | HOD |
|---|---|---|---|---|---|
| 1 | Praneeth | DBMS | D1 | AMCS | Dr. X |
| 1 | Praneeth | OS | D1 | AMCS | Dr. X |

**2NF** — must be in 1NF and have no *partial dependency* (a non-key attribute depending on only part of a composite key). The key here is (StudentID, Course), but Name and DeptID depend on StudentID alone. Split:
- `Student(StudentID, Name, DeptID, DeptName, HOD)`
- `Enrollment(StudentID, Course)`

**3NF** — must be in 2NF and have no *transitive dependency* (a non-key attribute depending on another non-key attribute). `DeptName` and `HOD` depend on `DeptID`, not on `StudentID`. Split again:
- `Student(StudentID, Name, DeptID)`
- `Department(DeptID, DeptName, HOD)`
- `Enrollment(StudentID, Course)`

**BCNF** — for every functional dependency X → Y, X must be a superkey. It is a stricter 3NF that also handles the case where a *prime* attribute depends on a non-prime one. Most 3NF tables are already in BCNF.

**4NF** removes multi-valued dependencies; **5NF** removes join dependencies. In interviews, 3NF/BCNF is where you should be fluent.

**One-line summary to memorise**: "The key, the whole key, and nothing but the key, so help me Codd." 1NF = atomic values, 2NF = no partial dependency on the key, 3NF = no transitive dependency.

## 29. Anomalies Normalization Prevents
- **Insertion anomaly** — you cannot record a new department until at least one student joins it.
- **Update anomaly** — changing the HOD requires updating every student row, and missing one leaves the data inconsistent.
- **Deletion anomaly** — deleting the last student in a department also deletes the only record of that department.

## 30. SQL Query Patterns You Must Be Able to Write
```sql
-- Second highest salary
SELECT MAX(salary) FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

-- Nth highest salary (window function)
SELECT DISTINCT salary FROM (
  SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk FROM employees
) t WHERE rnk = 3;

-- Find duplicates
SELECT email, COUNT(*) FROM users GROUP BY email HAVING COUNT(*) > 1;

-- Delete duplicates keeping the lowest id
DELETE FROM users a USING users b WHERE a.id > b.id AND a.email = b.email;

-- Employees earning more than their department average
SELECT e.name, e.salary, e.dept_id FROM employees e
WHERE e.salary > (SELECT AVG(salary) FROM employees WHERE dept_id = e.dept_id);

-- Departments with more than 5 employees
SELECT d.name, COUNT(*) AS headcount
FROM departments d JOIN employees e ON e.dept_id = d.id
GROUP BY d.name HAVING COUNT(*) > 5 ORDER BY headcount DESC;

-- Employees with no manager (self join / outer join)
SELECT e.name FROM employees e LEFT JOIN employees m ON e.manager_id = m.id
WHERE m.id IS NULL;

-- Running total (window function)
SELECT order_date, amount,
       SUM(amount) OVER (ORDER BY order_date) AS running_total
FROM orders;

-- Top 3 per group
SELECT * FROM (
  SELECT *, ROW_NUMBER() OVER (PARTITION BY dept_id ORDER BY salary DESC) AS rn
  FROM employees
) t WHERE rn <= 3;
```

**Logical order of evaluation** (different from the written order, and a favourite question):
`FROM` → `JOIN` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `DISTINCT` → `ORDER BY` → `LIMIT`.
That is exactly why you cannot use a `SELECT` alias in `WHERE` but can in `ORDER BY`.

## 31. Set Operations and Subqueries
- `UNION` removes duplicates, `UNION ALL` keeps them and is faster. `INTERSECT` returns common rows, `EXCEPT`/`MINUS` returns rows in the first but not the second.
- **Correlated subquery** references the outer query and is evaluated once per outer row — potentially slow. A **non-correlated** subquery runs once.
- `EXISTS` stops at the first match, so it is usually faster than `IN` for large subquery results. `NOT IN` is dangerous with NULLs — if the subquery returns any NULL, `NOT IN` yields no rows at all. Use `NOT EXISTS` instead.
- **CTE** (`WITH x AS (...)`) makes complex queries readable and can be recursive for hierarchies like an org chart.

## 32. NULL Semantics (a common trap)
- NULL means "unknown", not zero and not an empty string.
- Any arithmetic or comparison with NULL yields NULL, so `WHERE col = NULL` never matches. Use `IS NULL` / `IS NOT NULL`.
- `COUNT(*)` counts rows including NULLs; `COUNT(col)` skips NULLs.
- Aggregates like `SUM` and `AVG` ignore NULLs, so `AVG` over 3 values where one is NULL divides by 2.
- `COALESCE(a, b, c)` returns the first non-NULL. `NULLIF(a, b)` returns NULL if a = b.
- A `UNIQUE` constraint typically permits multiple NULLs, because two unknowns are not provably equal.

## 33. Indexing in More Depth
- Default index structure is a **B+ tree**: balanced, all data in leaves, leaves linked for range scans, O(log n) lookup. **Hash indexes** are O(1) for equality only and useless for ranges. **Bitmap indexes** suit low-cardinality columns in warehouses. **Full-text indexes** for search.
- **Clustered index** determines the physical row order — one per table (the primary key in InnoDB). **Non-clustered / secondary index** is a separate structure holding the key plus a pointer to the row.
- **Composite index (a, b, c)** follows the **leftmost prefix rule**: it can serve queries filtering on (a), (a,b), or (a,b,c) — but not (b) or (c) alone.
- **Covering index** — an index that contains every column the query needs, so the table itself is never touched.
- **When an index is NOT used**: wrapping the column in a function (`WHERE YEAR(date) = 2024`), a leading wildcard (`LIKE '%abc'`), low selectivity (a gender column), or an implicit type conversion.
- **Cost**: every index slows down INSERT, UPDATE, and DELETE, and consumes storage. Index the columns you filter, join, and sort on — not everything.
- **`EXPLAIN` / `EXPLAIN ANALYZE`** shows the query plan: whether it did a sequential scan or an index scan, the join order, and the estimated versus actual row counts. Knowing to say "I would run EXPLAIN first" is a strong answer to any optimisation question.

## 34. Query Optimisation Checklist
1. Run `EXPLAIN` and look for full table scans on large tables.
2. Add or fix indexes on WHERE, JOIN, ORDER BY, and GROUP BY columns.
3. `SELECT` only the columns you need — never `SELECT *` in production code.
4. Avoid functions on indexed columns in the WHERE clause.
5. Filter as early as possible; push conditions into the WHERE rather than the HAVING when they do not involve aggregates.
6. Replace correlated subqueries with joins or window functions where possible.
7. Use `LIMIT` and keyset pagination (`WHERE id > last_seen`) instead of large `OFFSET`s.
8. Batch inserts instead of row-by-row.
9. Consider denormalising or adding a materialised view for expensive read-heavy aggregations.
10. Update table statistics so the optimiser makes good choices.

## 35. MongoDB Essentials (on my resume)
- **Document database**: data is stored as BSON documents in collections. No fixed schema, though schema validation can be enforced.
- **Mapping**: database → database, table → collection, row → document, column → field, join → `$lookup`, primary key → `_id` (an ObjectId by default).
- **CRUD**: `insertOne`/`insertMany`, `find({...})`, `updateOne({filter}, {$set: {...}})`, `deleteOne`.
- **Aggregation pipeline** stages: `$match` (filter — put it first so it can use an index), `$group` (aggregate with accumulators like `$sum`, `$avg`), `$project` (reshape), `$sort`, `$limit`, `$unwind` (flatten an array into one document per element), `$lookup` (left outer join to another collection).
- **Embedding vs referencing**: embed when the child is always read with the parent and is bounded in size; reference when the data is large, shared, or grows unboundedly. The 16 MB document limit forces referencing eventually.
- **Indexes** work like SQL, including compound and multikey (on array fields).
- **Replica set** gives high availability with automatic failover; **sharding** gives horizontal scale by partitioning on a shard key.
- **Transactions** across multiple documents are supported since 4.0, but single-document operations are atomic by default.
- MongoDB is normally **CP** in CAP terms with the default write concern.

## 36. Oracle-Specific Points (on my resume)
- Oracle is object-relational: it supports **object types** (`CREATE TYPE`), **VARRAYs**, **nested tables**, `REF` types, and type inheritance — features that PostgreSQL and MySQL do not fully replicate. My ClassRoom Code project talks to a real Oracle server for exactly this reason.
- **PL/SQL** is Oracle's procedural extension: blocks of `DECLARE / BEGIN / EXCEPTION / END`, cursors, procedures, functions, packages, and triggers.
- **Sequences** (`seq.NEXTVAL`) and **synonyms** are Oracle idioms; identity columns arrived later.
- **DDL commits implicitly** in Oracle — a `CREATE TABLE` ends the current transaction, so you cannot roll it back. This bit my project: per-student schemas were needed rather than a shared login.
- `DUAL` is a one-row dummy table used for expression evaluation (`SELECT SYSDATE FROM DUAL`).
- Unquoted identifiers are **upper-cased** in Oracle and **lower-cased** in PostgreSQL — which is why my platform compares result-set column names case-insensitively.

## 37. MySQL-Specific Points
- **Storage engines**: InnoDB (default — transactional, ACID, row-level locking, foreign keys, clustered primary key) versus MyISAM (older, table-level locking, no transactions, faster for read-only workloads).
- `AUTO_INCREMENT` for surrogate keys.
- Default isolation level is **REPEATABLE READ**, unlike PostgreSQL and Oracle which default to READ COMMITTED.
- InnoDB uses **MVCC** so readers do not block writers.

## 38. Transactions in Practice
```sql
BEGIN;
  UPDATE accounts SET balance = balance - 500 WHERE id = 1;
  UPDATE accounts SET balance = balance + 500 WHERE id = 2;
COMMIT;   -- or ROLLBACK on failure
```
- **SAVEPOINT** marks a point you can partially roll back to.
- **Write-ahead logging (WAL)** is how durability and atomicity are actually implemented: the change is written to the log and flushed to disk *before* the data pages are, so a crash can be recovered by redoing committed transactions and undoing uncommitted ones.
- **Two-phase commit** coordinates a transaction across multiple databases: a prepare phase where every participant votes, then a commit or abort phase.

---

# Practice Questions with Answers (DBMS)

### Conceptual / Definition-based

**1. What is a DBMS and how is it better than a file system?**
A DBMS is software that stores, retrieves, and manages data with a defined structure, and provides querying, concurrency control, integrity enforcement, security, and recovery. A plain file system gives none of that: it has data redundancy and inconsistency, no easy querying, no concurrent access control, no atomicity across multiple updates, no integrity constraints, and no built-in backup and recovery. A DBMS also gives **data independence** — you can change the storage layout without rewriting applications.

**2. Explain the three-level ANSI-SPARC architecture.**
**Internal/physical level** — how data is actually stored: files, indexes, compression. **Conceptual/logical level** — the whole database's logical structure: tables, columns, relationships, constraints. **External/view level** — what individual users or applications see, possibly a subset or a transformation. The separation gives **logical data independence** (changing the conceptual schema does not break external views) and **physical data independence** (changing storage does not break the conceptual schema).

**3. What are the different types of keys?**
**Super key** — any set of attributes that uniquely identifies a row. **Candidate key** — a minimal super key (no attribute can be removed). **Primary key** — the candidate key chosen as the identifier; unique and NOT NULL. **Alternate key** — the candidate keys not chosen. **Composite key** — a primary key made of more than one column. **Foreign key** — a column referencing the primary key of another table, enforcing referential integrity. **Surrogate key** — a system-generated meaningless identifier (auto-increment or UUID) used instead of natural data.

**4. What is normalization and why do we do it?**
Organising tables so that redundancy and dependency are minimised, by decomposing them according to functional dependencies. The goal is to eliminate insertion, update, and deletion anomalies and to keep the data consistent. See sections 28 and 29 for the worked example and the anomalies.

**5. Explain the ACID properties.**
**Atomicity** — a transaction is all or nothing; a partial transfer that debits without crediting can never be observed. **Consistency** — a transaction moves the database from one valid state to another, preserving every constraint. **Isolation** — concurrent transactions do not see each other's intermediate state; the result is as if they ran in some serial order. **Durability** — once committed, the change survives a crash, guaranteed by write-ahead logging and flushing to non-volatile storage.

**6. What is a transaction and what states does it go through?**
A logical unit of work — a sequence of operations that must succeed or fail as a whole. States: **Active** (executing) → **Partially committed** (last statement done, changes still in buffers) → **Committed** (durable) or **Failed** (an error occurred) → **Aborted** (rolled back, either restarted or killed) → **Terminated**.

**7. What is an ER model, and what are its components?**
A conceptual design notation. **Entities** (rectangles) are the things you store, **attributes** (ovals) describe them, and **relationships** (diamonds) connect them. Attribute types: simple, composite (address → street, city), multi-valued (phone numbers), derived (age from date of birth), and key attributes. **Cardinality**: 1:1, 1:N, M:N. A **weak entity** has no key of its own and depends on an owner entity (drawn with a double rectangle). Converting to tables: each entity becomes a table, 1:N puts the foreign key on the "many" side, and M:N needs a junction table.

### Comparison-based

**8. DBMS vs RDBMS.**
A DBMS stores data but may not enforce relationships between records (older hierarchical and network systems, or simple file-based managers). An RDBMS stores data in tables with rows and columns, enforces relationships through primary and foreign keys, supports SQL, follows Codd's rules, and provides full ACID transactions. Every RDBMS is a DBMS; the reverse is not true.

**9. SQL vs NoSQL.**
SQL databases have a fixed schema, are relational, scale vertically (bigger machine), guarantee ACID, and use SQL — good for financial data, anything with complex relationships and strict integrity. NoSQL databases are schema-flexible, come in several models (document, key-value, column-family, graph), scale horizontally by sharding, usually offer eventual consistency (BASE), and are good for large volumes of semi-structured data, rapidly changing schemas, and very high write throughput. My ClassRoom Code project uses PostgreSQL for the core relational data and supports MongoDB as a lab engine, so I have used both deliberately.

**10. DELETE vs TRUNCATE vs DROP.**
`DELETE` is DML — removes rows matching a WHERE clause, fires triggers, logs each row, can be rolled back, and does not reset AUTO_INCREMENT. `TRUNCATE` is DDL — removes all rows by deallocating pages, is much faster, does not fire row triggers, resets the identity counter, and generally cannot be rolled back. `DROP` is DDL — removes the entire table, its structure, indexes, and constraints.

**11. WHERE vs HAVING.**
`WHERE` filters individual rows *before* grouping and cannot contain aggregate functions. `HAVING` filters groups *after* `GROUP BY` and is where aggregate conditions go. `WHERE salary > 50000` is row-level; `HAVING COUNT(*) > 5` is group-level. Filtering in `WHERE` is cheaper because fewer rows reach the grouping stage.

**12. Primary key vs unique key.**
A primary key is unique, cannot be NULL, and there is exactly one per table; it usually creates the clustered index. A unique key is also unique but permits NULLs (typically more than one), and a table can have many of them. Both create an index.

**13. Clustered vs non-clustered index.**
A clustered index defines the physical ordering of the rows, so there can be only one, and looking up by it lands directly on the data. A non-clustered index is a separate structure holding the indexed key plus a pointer back to the row, so a lookup costs an extra step; a table can have many.

**14. Explain all the join types.**
`INNER JOIN` — only rows matching in both. `LEFT (OUTER) JOIN` — all rows from the left, NULLs where the right has no match. `RIGHT JOIN` — the mirror image. `FULL OUTER JOIN` — all rows from both, NULLs where either side is missing. `CROSS JOIN` — the Cartesian product, every combination. `SELF JOIN` — a table joined to itself, used for hierarchies like employee-manager. `NATURAL JOIN` joins on all identically named columns automatically, which is convenient and dangerous.

**15. View vs materialized view.**
A view is a stored query — it holds no data, is computed fresh on every access, and always reflects current data. A materialised view stores the actual result on disk, so reads are fast but the data is stale until refreshed. Views are used to simplify complex queries, restrict column or row access for security, and provide a stable interface over a changing schema.

**16. Stored procedure vs function vs trigger.**
A **stored procedure** is a named block of SQL invoked explicitly with `CALL`, can return zero or many values via OUT parameters, and can modify data. A **function** must return a value and is designed to be used inside expressions, typically with restrictions on side effects. A **trigger** is not called explicitly at all — it fires automatically BEFORE or AFTER an INSERT, UPDATE, or DELETE, and is used for auditing, enforcing complex constraints, and maintaining derived data.

### Scenario/Application-based

**17. How would you design the schema for a college classroom platform?**
This is my actual ClassRoom Code schema, so I can answer it concretely. `users` (id, google_sub, email, name, role) with a unique index on `lower(email)` so case does not create duplicates. `courses`, then two junction tables `course_teachers` and `course_enrollments`, each with a composite primary key of (course_id, user_id) — because a course is co-taught and a student takes many courses, both M:N. `worksheets` belongs to a course with a status of draft or published and an optional deadline. `questions` belongs to a worksheet with a position for ordering. `test_cases` belongs to a question. `submissions` has a `UNIQUE (question_id, student_id)` constraint because only one current submission is kept per student per question. `feedback` has a `UNIQUE` foreign key to submission — a 1:1 relationship. I used `ON DELETE CASCADE` where the child is meaningless without the parent (a test case without its question) and `ON DELETE SET NULL` where it is not (a course whose creator's account is removed). I used UUID primary keys rather than sequential integers so ids are not guessable in URLs, and `CHECK` constraints on enum-like columns (`role IN ('student','teacher','admin')`) so bad data cannot enter even if the application has a bug.

**18. A query is slow. How do you fix it?**
See section 34. Say the order out loud: measure with `EXPLAIN ANALYZE` first, then index, then rewrite, then consider schema or caching changes. Never guess.

**19. When would you deliberately denormalise?**
When reads vastly outnumber writes and the joins are expensive — a reporting dashboard, a product listing that needs the category name on every row, or a counter like `comment_count` that would otherwise require a COUNT over millions of rows. The cost is that you now have to keep the duplicated data in sync, usually with a trigger or application logic, and you have reintroduced the update anomaly on purpose.

**20. How do you handle a many-to-many relationship?**
With a junction (bridge/associative) table holding foreign keys to both sides, with a composite primary key over the pair to prevent duplicates. Any attributes of the relationship itself — an enrolment date, a grade — belong on the junction table, not on either entity.

**21. What is referential integrity and how do you maintain it?**
The rule that a foreign key value must either be NULL or match an existing primary key in the referenced table. It is enforced by declaring the foreign key constraint, and its behaviour on parent deletion is chosen with `ON DELETE`: `CASCADE` (delete the children too), `SET NULL`, `SET DEFAULT`, `RESTRICT`/`NO ACTION` (refuse the delete). The same options apply `ON UPDATE`.

### Concurrency / Transaction-based

**22. What concurrency problems can occur without isolation?**
**Dirty read** — reading data written by an uncommitted transaction that then rolls back. **Non-repeatable read** — reading the same row twice in one transaction and getting different values because another transaction committed an update in between. **Phantom read** — re-running the same range query and finding new rows that another transaction inserted. **Lost update** — two transactions read the same value, both compute a new one, and the second overwrites the first.

**23. Explain the four isolation levels.**
**READ UNCOMMITTED** — allows dirty reads; effectively no isolation. **READ COMMITTED** — only committed data is read, preventing dirty reads, but non-repeatable and phantom reads remain (the PostgreSQL and Oracle default). **REPEATABLE READ** — a row read twice returns the same value, preventing non-repeatable reads; phantoms may still occur in the standard, though MySQL InnoDB blocks them with gap locks. **SERIALIZABLE** — full isolation, the result is equivalent to some serial order; safest and slowest. The trade-off is always correctness against concurrency.

**24. What are the concurrency control techniques?**
**Lock-based**: shared (read) and exclusive (write) locks, managed by **two-phase locking (2PL)** — a growing phase where locks are only acquired and a shrinking phase where they are only released. Strict 2PL holds all exclusive locks until commit, which guarantees recoverability. **Timestamp ordering**: each transaction gets a timestamp and conflicting operations are ordered by it, aborting anything out of order. **Optimistic concurrency control**: run without locking, then validate at commit and abort if there was a conflict — good when conflicts are rare. **MVCC**: writers create new versions rather than overwriting, so readers see a consistent snapshot and never block writers — used by PostgreSQL, Oracle, and InnoDB.

**25. What is a deadlock in a database and how is it handled?**
Two transactions each hold a lock the other needs. Databases usually **detect** it by finding a cycle in the wait-for graph and then abort the cheaper transaction as a victim, which the application should retry. Prevention strategies: acquire locks in a consistent global order, keep transactions short, use a lock timeout, or use the wait-die / wound-wait timestamp schemes.

**26. What is serializability?**
A schedule is serializable if it produces the same result as *some* serial execution of the same transactions. **Conflict serializability** is checked by building a precedence graph over conflicting operation pairs (read-write, write-read, write-write on the same item) and testing for a cycle — no cycle means it is conflict serializable. **View serializability** is a weaker, more general condition but is NP-hard to test, so systems use conflict serializability in practice.

**27. Explain the CAP theorem.**
In a distributed data store you can guarantee at most two of **Consistency** (every read sees the latest write), **Availability** (every request gets a response), and **Partition tolerance** (the system keeps working despite dropped messages between nodes). Since network partitions are unavoidable in a real distributed system, the real choice is between CP (refuse requests to stay consistent — MongoDB, HBase) and AP (answer with possibly stale data — Cassandra, DynamoDB). A single-node RDBMS is CA only because it is not distributed.

### Joins / Query-based

**28. Write a query for the second-highest salary.** See section 30 — both the subquery form and the `DENSE_RANK()` window function form. Mention that `DENSE_RANK` handles ties correctly and generalises to the Nth value.

**29. What is the difference between UNION and JOIN?**
A JOIN combines **columns** from two tables based on a matching condition, producing wider rows. A UNION combines **rows** from two result sets that already have the same number and types of columns, producing a longer result.

**30. What is a self join and when do you need it?**
A table joined to itself with different aliases, used when rows in a table relate to other rows in the same table — an employee and their manager, a category and its parent, a chain of referrals. `SELECT e.name, m.name FROM employees e LEFT JOIN employees m ON e.manager_id = m.id;`

**31. What do aggregate functions do with NULL?** They ignore it. See section 32 for the full NULL trap list — this is a favourite gotcha.

### Miscellaneous

**32. What are the SQL command categories?**
**DDL** (CREATE, ALTER, DROP, TRUNCATE, RENAME) — defines structure, auto-commits. **DML** (SELECT, INSERT, UPDATE, DELETE) — manipulates data. **DCL** (GRANT, REVOKE) — permissions. **TCL** (COMMIT, ROLLBACK, SAVEPOINT) — transaction control. Some texts put SELECT in its own **DQL** category.

**33. What is SQL injection and how do you prevent it?**
An attacker supplies input that becomes part of the SQL statement, for example a password field containing `' OR '1'='1`, so the query's logic changes. Prevention, in order of importance: **parameterised queries / prepared statements** so user input is always data and never code; an ORM or query builder that parameterises by default; least-privilege database accounts so a compromised query cannot drop tables; input validation and allow-listing; and never building SQL by string concatenation. Escaping alone is not sufficient. This is also OWASP A03 — see my cybersecurity notes.

**34. What is a schema, an instance, and a catalog?**
The **schema** is the design — the structure, defined once and rarely changed. The **instance** is the actual data in the database at a moment in time, changing constantly. The **catalog / data dictionary** is the database's own metadata about its schemas, tables, columns, and constraints, queryable through `information_schema` or Oracle's `USER_TABLES` views.

**35. What is a cursor?**
A pointer that lets procedural code iterate a result set one row at a time. **Implicit** cursors are created automatically for single-row statements; **explicit** cursors are declared, opened, fetched from in a loop, and closed. They are useful when per-row processing genuinely cannot be expressed as a set operation, but they are slow — SQL is set-based, and a cursor loop over a million rows is almost always the wrong answer.

**36. What is data warehousing and OLTP vs OLAP?**
**OLTP** is transaction processing: many short read-write transactions, highly normalised, current data, optimised for insert/update latency — this is your application database. **OLAP** is analytical processing: few long read-heavy queries scanning huge volumes, denormalised into star or snowflake schemas with fact and dimension tables, historical data, optimised for aggregation. A **data warehouse** is the OLAP store, populated from OLTP systems by an **ETL** (extract, transform, load) pipeline.

---

# Rapid-Fire One-Liners

- **Cardinality** — the number of distinct values in a column; high cardinality makes an index useful.
- **Degree** — the number of attributes (columns) in a relation. **Tuple** — a row.
- **Domain** — the set of permitted values for an attribute.
- **Entity integrity** — the primary key cannot be NULL. **Domain integrity** — values must be of the declared type and satisfy CHECK constraints.
- **Constraint types** — NOT NULL, UNIQUE, PRIMARY KEY, FOREIGN KEY, CHECK, DEFAULT.
- **Trigger timing** — BEFORE, AFTER, or INSTEAD OF (on views), and FOR EACH ROW or FOR EACH STATEMENT.
- **Sharding** — splitting rows across machines by a key. **Partitioning** — splitting a table within one database, by range, list, or hash.
- **Replication** — copying data to other nodes; master-slave for read scaling, master-master for write availability.
- **Connection pool** — reusing a fixed set of open connections, because opening one is expensive.
- **ORM** — maps objects to tables; convenient but hides the generated SQL, which causes the **N+1 query problem** (one query for a list plus one per item, fixed by eager loading or a join).
- **BASE** — Basically Available, Soft state, Eventually consistent; the NoSQL counterpart to ACID.
- **Idempotent write** — applying it twice has the same effect as once, which matters for retries.

---

# Linking DBMS to My Projects

- **ClassRoom Code** is a full DBMS answer on its own. It uses PostgreSQL with hand-written SQL migrations, UUID primary keys, composite keys on junction tables, `CHECK` constraints for enums, `ON DELETE CASCADE` and `SET NULL` chosen per relationship, partial and functional indexes (`CREATE UNIQUE INDEX users_email_lower_idx ON users (lower(email))`), and JSONB columns for semi-structured data like starter code and run results. It also runs an **SQL and MongoDB lab engine**: student queries execute against a freshly seeded database per run — a real isolation problem — and the result sets are compared while deliberately ignoring column-name case (because Oracle upper-cases and PostgreSQL lower-cases identifiers), column order (because MongoDB's `$project` does not preserve field order), and row order unless the question is specifically about `ORDER BY`. That last point is a direct application of "a query without ORDER BY has no defined row order."
- It supports **four engines behind one interface** — SQLite, PostgreSQL, Oracle, and MongoDB — which forced me to learn where the standard ends and vendor behaviour begins. The Oracle implicit-DDL-commit problem is a concrete war story.
- **ModelAuth** stores its experiment streams as **JSONL** rather than in a database, which is a reasonable answer to "when would you not use a DBMS?" — append-only, write-once, read-sequentially scientific data with no concurrent writers and no relational queries.
