---
layout: post
title: "Exploring Databases"
date: 2020-09-28 20:13:22 GMT
tags: database keyvalue redis cassandra hbase mongodb mysql graphdb neo4j acid base
---

A requirement of almost any program is to persist data and retrieve data. That’s why we have Databases.
A Database is usually designed to store and manages a large amount of data. A database has to be accurate, with all sorts of internal checks, giving integrity to the data it manages. 
Since they are a solution to a pervasive problem, it is easy to see why they have been developed since the early days of CS, and why we have different flavors, for different needs. Let's review them. 

## Key-Value
The most simple approach is a hash pairing *keys* and *values*. 
Fast and easy to use, they are popular to build caches. 
Because they hold data in memory, there is a limitation to the amount of data at their disposal, but at the same time, by avoiding round trips to slow second memory, they are super fast. 
They are also limited in the interface. No fancy queries, JOINs, or anything like that. Just read and write. 
Let's see an example in Redis
```
# redis-cli
> 127.0.0.1:6379> SET maurice_moss reynholm
OK
> 127.0.0.1:6379> GET maurice_moss
"reynholm"
```
Best for: Reduce data latency. Usually deployed on top of some other database used to persist data. 
Popular alternatives: [Redis](https://redis.io), [MemCache](https://memcached.org)

## Wide Column
We can stretch the value part of a key-value DB, to store a set of ordered rows, and then we have a wide column DB. That way we can group data together and associate it with the same key. 
These databases don't have a schema, and they can easily handle unstructured data
I know, I know, Cassandra does have a schema. That's true. It is also true that it was developed schemaless. Schemas were added later. 
We can interact with them with some languages (like CQL), that usually are similar to the most popular SQL, but limited (still no fancy operations like JOINs)
Because of its nature, they are easy to replicate and scale-up. And no, the reason they are easy to replicate is not that they are NoSQL, it is because they relax on the [ACID](https://en.wikipedia.org/wiki/ACID) requirements. You see, read scaling is not that hard. Bottlenecks appear only when introducing JOINs and that kind of operations, which can be opt-out even in RDBMS. The problem is to scale up writes. If you want to speed up writes, then you will need to relax on *atomicity* by shorten the time tables are locked (like MongoDB), *consistency* which let's scale-up in a cluster of nodes (like Cassandra) or *durability* holding everything on memory and avoiding round trips to disk (as we saw already, Redis). 
In fact, these types of databases are popular in applications where writing is much more frequent than reading. 
Let's imaging a system that persist readings from a vast array of wheather stations: 

```
cqlsh> CREATE KEYSPACE IF NOT EXISTS mycassandra WITH REPLICATION = {'class': 'SimpleStrategy', 'replication_factor': 1};
cqlsh> USE mycassandra;
cqlsh:mycassandra> CREATE TABLE IF NOT EXISTS wheather (temp float, pressure int, humidity float, location varchar, time timestamp, PRIMARY KEY(location));
cqlsh:mycassandra> INSERT INTO wheather (temp, pressure, humidity, location, time) VALUES (23, 1016, 90, 'Buenos Aires', toTimestamp(now()));
cqlsh:mycassandra> INSERT INTO wheather (temp, pressure, humidity, location, time) VALUES (18, 1030, 72, 'Lisbon', toTimestamp(now()));
cqlsh:mycassandra> SELECT * FROM wheather;

 location     | humidity | pressure | temp | time
--------------+----------+----------+------+---------------------------------
       Lisbon |       72 |     1030 |   18 | 2020-09-24 22:25:44.563000+0000
 Buenos Aires |       90 |     1016 |   23 | 2020-09-24 22:24:36.110000+0000

(2 rows)
```
Best for: Backing IoT
Popular alternatives: [Apache Cassandra](https://cassandra.apache.org), [Apache HBase](https://hbase.apache.org), [Cloud Bigtable](https://console.cloud.google.com/marketplace/details/google-cloud-platform/cloud-bigtable)

## Document DB
They are based on documents, where each document is a container of *key-value* pairs. They are unstructured and don't require a schema. 
Documents are group together in collections, and fields within collections can be indexed. 
Collections can be organized in hierarchies, allowing some kind of relational modeling. 
Still no JOINs. 
Denormalization is encouraged, because of this, write operations could be a little slower, but, as we saw earlier, they relax on ACID requirements to achieve better performance. 

```
root@7747c048549d:/# mongo
MongoDB shell version v4.4.1
> use reynholm_employees;
switched to db reynholm_employees
> db.it.save({first: "Maurice", last: "Moss"});
WriteResult({ "nInserted" : 1 })
> db.it.save({first: "Roy", last: "Trenneman"});
WriteResult({ "nInserted" : 1 })
> db.it.save({first: "Jen", last: "Barber"});
WriteResult({ "nInserted" : 1 })
> db.it.find({first: "Maurice"});
{ "_id" : ObjectId("5f6d2a4ced7dc6a9061ed522"), "first" : "Maurice", "last" : "Moss" }
> 
```

Best for: They are very popular in IoT and content management. They are also great to start if not sure about how data is structured. 
Popular alternatives: [MongoDB](https://www.mongodb.com), [Apache CouchDB](https://couchdb.apache.org)

## RDBMs
Very popular, and one of the older paradigms. 
They are a collection of multiple data sets organized in tables with a well-defined relationship between them. 
Each table is a relation, each table record (row), contains a unique data instance defined for a corresponding column category.
One or more data or record characteristics relate to one or many records to form a functional dependency (normalization). 
*One to One: One table record relates to another record in another table. 
*One to Many: One table record relates to many records in another table(s).
*Many to One: More than one table record relates to a record in a different table. 
*Many to Many: More than one record relates to other records in different tables.
We can interact with them with SQL (Structured Query Language) languages. 
Normalization requires a schema, which can be tricky if the data structure is not known in advance. The flip side is that we finally get to play with JOINs

```
mysql> CREATE TABLE orders (
    ->   order_id INT AUTO_INCREMENT PRIMARY KEY,
    ->   timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP
    -> );
Query OK, 0 rows affected (0.02 sec)

mysql> CREATE TABLE details (
    ->   product_id INT AUTO_INCREMENT PRIMARY KEY,
    ->   name VARCHAR(100),
    ->   qty INT,
    ->   order_id INT
    -> );
Query OK, 0 rows affected (0.02 sec)

mysql> INSERT INTO orders (order_id) VALUES (NULL);
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO orders (order_id) VALUES (NULL);
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO details VALUES (NULL, 'Apricots', 4, 1);
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO details VALUES (NULL, 'Bananas', 2, 1);
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO details VALUES (NULL, 'Eggfruit', 1, 2);
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO details VALUES (NULL, 'Blueberries', 3, 2);
Query OK, 1 row affected (0.00 sec)

mysql> SELECT o.order_id, o.timestamp, d.name, d.qty FROM orders o INNER JOIN details d ON o.order_id = d.order_id;
+----------+---------------------+-------------+------+
| order_id | timestamp           | name        | qty  |
+----------+---------------------+-------------+------+
|        1 | 2020-09-25 12:32:02 | Apricots    |    4 |
|        1 | 2020-09-25 12:32:02 | Bananas     |    2 |
|        2 | 2020-09-25 12:32:06 | Eggfruit    |    1 |
|        2 | 2020-09-25 12:32:06 | Blueberries |    3 |
+----------+---------------------+-------------+------+
4 rows in set (0.00 sec)
```

Best for: Perhaps the most popular family of DBs, and essentials when data integrity is a must (financial).
Popular alternatives: [MySQL](https://www.mysql.com), [PostgreSQL](https://www.postgresql.org)

## Graph
In graph DB, the relationships between elements are first-class citizens, they are treated exactly the same as the elements. 
From a mathematical point of view, the relations are edges of a graph where the elements are nodes. 
Edges are always directed.
It is far more efficient to traverse the data. We can specify edges, or move across the entire graph. Because the graph is already built, there is no need to compute JOINs and the performance is thus greatly improved. 
[neo4j](https://neo4j.com) is perhaps the most popular graph database out there. In the sandbox it provides, there is a database with movies, actors, and directors. To compute the [Tom Hanks number two](https://simple.wikipedia.org/wiki/Bacon_number) we can do something like 

```
MATCH (a:Person{name: "Tom Hanks"})-[:ACTED_IN]->(m:Movie)<-[:ACTED_IN]-(b:Person) 
MATCH (b:Person)-[:ACTED_IN]->(n:Movie)<-[:ACTED_IN]-(c:Person)
WHERE c <> a AND NOT (a)-[ACTED_IN]->()<-[:ACTED_IN]-(c)
-> RETURN c.name
```

Best for: Anything that can be expressed as a graph. Very popular with engine recommendations, and fraud detection. 
Popular alternatives: [MongoDB](https://www.mongodb.com), [Apache CouchDB](https://couchdb.apache.org)