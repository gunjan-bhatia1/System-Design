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

### [Specialized Operations and Why Relational Databases Struggle](../Appendix/Specialized%20Operations%20and%20Why%20Relational%20Databases%20Struggle.md)
* There are certain e.g. like graph traversal, full text search

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

### Relational vs document today

| Document DB                                 | Relational DB                        |
|---------------------------------------------|--------------------------------------|
| Better due to schema flexibility            | Better in many to one relationships  |
| Performance is better due to locality       | Better in many to many relationships |
| close to data structure used by application | Better in terms of joins             |

#### Which data models lead to simpler application code

* If data have structure of **one-to-many** in application then go for document.
* Breaking document into relational table is complex it's called shredding.
  * Can be useful to bring data to point for relational querying, joins, indexing of any level .
  * In case of document we can't directly refer nested columns we have to use **access paths** as shown in below e.g.

* JSON

```{
  "_id": ObjectId("..."),
  "userId": 251,
  "name": "Tia",
  "profile": {
    "age": 26,
    "address": {
      "city": "Panipat",
      "zip": "1001"
    },
    "positions": [
      { "title": "Software Engineer", "location": "Remote" },
      { "title": "Senior Software Engineer", "location": "Gurgaon" }
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
  "name": "Tia",
  "profile": {
    "address": {
      "city": "Karnal"
    },
    "positions": [ null, { "title": "Senior Software Engineer", "location": "Gurgaon" } ] // excluded position value is set to null here
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
  * For e.g. earlier we were storing full name now we want to store separate first and last name

```kotlin
if (user != null && user.name != null && user.firstName == null && user.lastName == null) {
    val parts = user.name.split(" ")
    user.firstName = parts.getOrNull(0)
    user.lastName = parts.getOrNull(1) ?: ""
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

* A document is usually stored as one block json or xml(such as MONGODB BSON).
* So if application require accessing entire document again and again there is performance advantage to storage locality.
* It only applies if we need large part of data. If small parts are needed then loading entire document isn't sensible
* Any writes  entire document needs to be rewritten.
* Only if encoded size isn't being changed can be easily performed.
* It's generally recommended to keep the doc size small and not write that increase size of document

* For e.g. changing city from Panipat to Gurgaon won't increase the size. Inplace update
```
db.users.updateOne(
  { _id: "123" },
  { $set: { city: "Gurgaon" } }
)
```
* But if we change city from Panipat to Hyderabad the size will increase. Relocation stress as well XD.
* Column family concept in big data (cassandra and HBase) also use concept of locality

MongoDB will mark the old space as "deleted" and move the updated document to a new location. This causes document moves and rewrites, which is slower and can lead to fragmentation.

#### Convergence of document and relational databases (Hybrid is the future)

* Using XML documents give similar experience as documents. Going forward support for JSON also came in  e.g. PostgresSQL
* Document DB have Rethink DB and mongo drive which support joins also automatically reference at client side. But this will increase query time.

## Query Languages for Data

* There are two types:
  * Declarative : We just tell what to do i.e. condition to be met, how it is to be grouped, sorted & aggregated
    * Rest is taken care by query optimizer
    * Based on relational algebra,  σ is the selection operator, returning based on condition
      ```sharks = σ family = “Sharks” (animals)```
```
SELECT * FROM animals WHERE family = 'Sharks';
```
  * Imperative : give line by line instruction on how to do.
    * We can literally evaluate how it will happen similar  java any other programming language.

```
for (int i = 0; i < animals.size(); i++) {
  if (animals.get(i).family.equals("Sharks")) {
    sharks.add(animals.get(i));
  }
}
```
  * Declarative add abstraction on implementation. Even if data structure is changed no impact on query but on imperative code it will be impacted.
  * Modern CPU work faster by parallelizing by adding cores.
  * With imperative code it's not possible because order of inst execution is to be followed which is not the case with declarative code. Declarative focus on how and not what.
  * Databases, big data tools (like Spark), and query engines take advantage of this

**Example**

```
SELECT customer_id, SUM(order_amount)
FROM orders
GROUP BY customer_id
```
* Spark can split orders into multiple partitions.
* Each partition can compute partial sums in parallel.
* Final sums are merged.
* Declarative queries are used on web using XSL instead of CSS -> Not need to go in detail (TODO: for future)

### Map-Reduce

* Programming lang for processing bulk of data
* Used by NoSQL(e.g. MongoDB & CouchDB) for performing read only query in documents. 
* Mix of imperative + declarative.
* Map (transforming from one collection to other). Reduce (accumulating collection).
* **Real word e.g.**
  * **map** → convert prices from USD to INR 
  * **filter** → keep only the orders above ₹1000 
  * **reduce** → sum up the total value

* **SQL**
```
SELECT SUM(price_usd * 83) AS total_in_inr
FROM orders
WHERE price_usd * 83 > 1000;
```

* **MongoDb query using mapReduce**
```db.orders.mapReduce(
  function map() {
    var price_in_inr = this.price_usd * 83;
    if (price_in_inr > 1000) {
      emit(null, price_in_inr);  //I don't want to group by anything specific. Just aggregate all these values. that's why key is null
    }
  },
  function reduce(key, values) {
    return Array.sum(values);  // Sum all values for the given key (null in our case)
  },
  {
    out: { inline: 1 }  // This will return the result inline (instead of creating a new collection)
  }
)
```
* Map reduce function work only given input it is  disconnected from external system.
* On failure, it can be safely executed and could be paralleled.
* Pure function (same input same output) no random behavior.
* Using this we can do **transformation, calculation, call library function.**
* Monopoly of executing distributed query isn't with mapReduce there are several other will see later.
* It's hard to write 2 coordinated map & reduce function then one single query
  * To improve this mongo query lang called **aggregation pipeline** .

```
db.orders.aggregate([
  {
    $project: {
      price_in_inr: { $multiply: ["$price_usd", 83] }
    }
  },
  {
    $match: { price_in_inr: { $gt: 1000 } }
  },
  {
    $group: {
      _id: null,
      total_in_inr: { $sum: "$price_in_inr" }
    }
  }
])
```

## Graph-Like Data Models

For complex **many-to-many** relation
* Graph consist of two thing vertices and edges.
* **Social Graph:** Vertices are people , edge are connections.
* **Web Graph:** Vertices are pages , edges are links
* **Rail networks:** Vertices are junctions, edges are rail track.
* 