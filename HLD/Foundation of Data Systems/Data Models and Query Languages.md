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

### Relation vs document db

| Document DB                                 | Relational DB                        |
|---------------------------------------------|--------------------------------------|
| Better due to schema flexibility            | Better in many to one relationships  |
| Performance is better due to locality       | Better in many to many relationships |
| close to data structure used by application | Better in terms of joins             |

#### Which data models lead to simpler application code

* If data have structure of **one-to-many** in application then go for document.
* Breaking document into relational table is complex it's called shredding.
  * Can be useful to bring data to point for relational querying, joins, indexing of any level .
  * In case of document we can't directly refer nested columns we have to use **access paths** as shown in below eg.

* JSON

```{
  "_id": ObjectId("..."),
  "userId": 251,
  "name": "John Doe",
  "profile": {
    "age": 30,
    "address": {
      "city": "New York",
      "zip": "10001"
    },
    "positions": [
      { "title": "Software Engineer", "location": "New York" },
      { "title": "Senior Software Engineer", "location": "San Francisco" }
    ]
  }
}
```

* Query will be

```db.users.find(
  { "userId": 251 },                          // Match user with userId: 251
  { 
    "name": 1,                                // Include "name"
    "profile.address.city": 1,                // Include city from address here 1 indicate include
    "profile.positions.1": 1,                 // Include second position (index 1) from positions
    "_id": 0                                  // Exclude "_id"  0 means exclude
  }
)
```

* Result

```{
  "name": "John Doe",
  "profile": {
    "address": {
      "city": "New York"
    },
    "positions": [ null, { "title": "Senior Software Engineer", "location": "San Francisco" } ] // excluded position value is set to null here
  }
}
```

* In **include‐mode:** all unspecified fields are dropped.
* In **exclude‐mode:** all unspecified fields are kept.
* It's just for _id we have to specify.
* Joins are needed or not depend on application and data for ag organisation structure its tree like we don't need
  * But if we need then we can keep the data in denormalized form or query again and again.
    * Which make system slow also push the responsibility on application code.
* For highly connected data document model won't work , relational is adjustable, graph is best.

#### Schema flexibility

* Schema validation is not enforced in JSON based document model or relational
* XML have optional validation in relational db
* But we can't say schemaless although at db level there is no validation but in application we have well-defined schema and db have to abide by.
  * It can be said it's **schema on read**(run time type checking) and not **schema on write**(compile time type checking).
* In case of schema changes in document model we just need to create new document with new field and keep the code in our application.
  * For eg earlier we were storing full name now we want to store separate first and last name
```
if (user && user.name && !user.first_name && !user.last_name) 
{
// Documents written before Apr 21, 2025 don't have first_name & last_name
user.first_name = user.name.split(" ")[0];
user.last_name = user.name.split(" ")[1];
}
```
* Whereas in case of SQL we will need to do migration using AlTER table and update command .
```
ALTER TABLE users ADD COLUMN first_name text;
UPDATE users SET first_name = split_part(name, ' ', 1); -- PostgreSQL
UPDATE users SET first_name = substring_index(name, ' ', 1); 
```
* Update command could be costly for huge tables
* If that's costly do it on read time similar to document

Use schema on read if there are many different object not possible to put in table. Or Schema is unknown decided by external system.

#### Data locality for queries

* 