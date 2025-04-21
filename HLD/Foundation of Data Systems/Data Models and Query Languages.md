# Data models

* DM are important not only in the way how we write our software but also in a way how we think of solving a problem.
* Each **data layer** act as abstraction hiding complexity of lower layer to make life of engineers easy and work focussed.
* In this do we will discuss 3rd layer of hierarchy diagram.

[![Data Model Hierarchy](Images/Data%20Model%20Hierarchy.png)](Images/Data%20Model%20Hierarchy.png)

## Relational Model Versus Document Model

* We have relational database (data stored in rows), 
  * Used for transaction processing(banking , bookings) . Batch processing (invoices, payrolls)
* Then came in picture hierarchical db - parent child relation. 
  * Parent can have many child but not vice versa.
  * **Pros**
    * Easy to navigate, simple fast.
  * **Cons**
    * **Many-to-many** relation handling
* Network db: In which data is organised in record linked by pointers.
  * Data is stored in sets,
  * **Pros**
    * Handle **many-to-many** relation
  * **Cons**
    * Navigating via pointer make adhoc query difficult unlike SQL

Here relational db (SQL) won. It was also used beyond transaction & batch processing. Gaming Saas.

### NO-SQL

It was open source and helped explore below areas:
* Greater scalability, large datasets and very high write throughput.
* Dynamic data modeling.
* Query operation not supported by relational db.

### Specialized Operations and Why Relational Databases Struggle

| Specialized Operation                 | Why Relational Model Struggles                                                                                                                                        | Better Solution                                                               | Example                                                                                                       |
|:--------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------|
| **Recursive / Hierarchical Queries**  | Relational tables are flat (rows and columns) and lack native recursion. Recursively querying parent-child hierarchies is awkward and inefficient.                    | Graph Databases (e.g. Neo4j), Recursive CTE in SQL                            | 🔍 *Finding the management chain of an employee in an organization*                                           |
| **Graph Traversals**                  | Multi-hop joins (finding paths or relationships through several levels) are slow and complex in SQL.                                                                  | Graph Databases (e.g. Neo4j, Amazon Neptune)                                  | 🔍 *Finding mutual friends within 4 degrees of connection in a social network*                                |
| **Full-Text Search**                  | SQL `LIKE` is primitive, offering only basic pattern matching. No relevance ranking, synonym matching, or typo tolerance.                                             | Full-Text Search Engines (e.g. Elasticsearch, Solr)                           | 🔍 *Searching "travl" and still getting "travel" results ranked by relevance*                                 |
| **OLAP / Multi-Dimensional Analysis** | Performing multi-dimensional aggregations requires complex, heavy `GROUP BY` operations and subqueries.                                                               | OLAP Systems (e.g. Snowflake, BigQuery)                                       | 🔍 *Analyzing monthly sales by region, product category, and year over year growth*                           |
| **Temporal Queries**                  | Relational databases don’t natively support time-versioned records (what a record looked like at a point in time). Implementing this requires extra tables and logic. | Temporal Databases (e.g. bitemporal DBs, PostgreSQL with temporal extensions) | 🔍 *Finding what a customer’s subscription status was on 2021-08-01*                                          |
| **Semi-Structured / JSON Queries**    | Relational models are schema-first and rigid. Handling flexible, nested, or varied data structures like JSON is cumbersome and less performant.                       | Document Databases (e.g. MongoDB, Couchbase)                                  | 🔍 *Storing and querying product details where each product can have different attributes and specifications* |

We should opt for polyglot persistence.

### Object Relational Mismatch

* We generally follow **OOPs** for writing code. This have disconnect way we store data in tables.
* This disconnect to some extent is solved by **ORM** (hibernate) reducing boilerplate code.
* Let's say we have a resume the most basic schema is shown in  [![Resume Basic Schema](Images/Resume%20Basic%20Schema.png)](Images/Resume%20Basic%20Schema.png). In relational db format.
  * We can use database that store multi value info , like multiple jobs, multiple location.
  * We can also store in json or xml document store it in text.
  * Since resume is self-descriptive storing in JSON is better.
    * It seems using json will reduce mismatch it's not the case.
    * TODO: Problems faced
  * We can easily represent **one-to-many** info in tree like structure.
  * We generally store info in as region_id and region in one table and use region_id elsewhere. This prevents redundant writes of region in case of change.
    * Let's say we want to change region name or do localisation we will have to write at only one place. This is nothing but normalisation
  * Now if we see many people live in one region and many people work in one industry, **many-to-one** relationship.
    * For **one-to-many** relation in tree like structure we don't need joins.
    * Also document db doesn't support join so we have to write in application code which is bad choice because we have to query db multiple times.
    * Now let's say we introduce new futures like recommendation.
      * Recommendation will appear on user profile also any change in recommender profile it should be reflected. 
      * Thus, many user can have many recommender and recommender can recommend multiple users. This is **many-to-many**.
    * Also, if we have feature of showing links of school and organisation. This is also **many-to-many**.

### **Tree Structure**
[![Tree Representation Of Resume](Images/Tree%20Representation%20Of%20Resume.png)](Images/Tree%20Representation%20Of%20Resume.png).
      
This will lead us to use join this is similar to hierarchical model issue. Initially relational & network model were give solution

### Network model

* It uses pointer and to move from one record to another we have to follow entire path called **access path**.
* In case of **many-to-many** there are several path and these are stored in head. It's very difficult in case any changes are to be made.

### Relational model

* In these which parts of the query to execute in which order, and which indexes to use  is automatically decided by query optimiser. We don't need access path/
* Let's say initially we are trying to query from customer table some data with customer_id=1.
  * ```SELECT * FROM Orders WHERE CustomerID = 'C123'```
* Now entire table all the rows will be scanned to find the one matching above condition we don't need to change query.
* If we create below index. It will **index seek** like b-tree instead of scanning entire index.
  * ```CREATE INDEX idx_customerid ON Orders (CustomerID)```
* This is also because query is selective if it had < or > then it would have used **index scan** if the index exited on that column
* **B-tree indexes** are useful for equality queries or range queries(= or BETWEEN).
* **BitMap indexes** are useful for low cardinality column(gender or boolean).
* **Full text indexes** useful for LIKE queries
* In case of **one-to-many** document db are diff from relational, but otherwise they are same. Either we perform join using document reference or further query to get data.

TODO: Read about different indexes working

