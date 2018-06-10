# scylladb-manticore
Container set for parallel indexing in ScyllaDB and MantiCoreSearch

### Startup
```
# docker-compose up -d
```
-------

### Hosts
* scylladb
  * node1: 10.10.10.1
  * node1: 10.10.10.2
  * node1: 10.10.10.3
* manticore
  * node1: 10.10.10.50

#### Scylla CQL
```
# cqlsh 10.10.10.1
Connected to Test Cluster at 10.10.10.1:9042.
[cqlsh 5.0.1 | Cassandra 3.0.8 | CQL spec 3.3.1 | Native protocol v4]
Use HELP for help.
cqlsh>
```
###### nodetool status
```
Datacenter: DC1
===============
Status=Up/Down
|/ State=Normal/Leaving/Joining/Moving
--  Address     Load       Tokens       Owns (effective)  Host ID                               Rack
UN  10.10.10.2  360.14 KB  256          67.1%             262f5811-d264-4ce5-b023-644fbb8c8686  Rack1
UN  10.10.10.3  492.12 KB  256          69.2%             150712f9-1f43-491f-8e52-21854a894713  Rack1
UN  10.10.10.1  338.92 KB  256          63.7%             a9d6b757-5e5f-4012-891c-7de665087f1e  Rack1
```

### Manticore SQL

```
# mysql -h10.10.10.50 -P9306
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 2.6.4 37308c36@180503 release

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]>
```

--------

## Demo
#### Create CQL Keyspace & SQL Database
##### CQL
```
CREATE KEYSPACE myindex WITH REPLICATION = { 'class' : 'NetworkTopologyStrategy','DC1' : 3};

CREATE TABLE myindex.testrt (
   id UUID PRIMARY KEY,
   title text,
   content text,
   gid int);
```
##### SphinxQL
Using default table `testrt`

#### Insert Data
##### CQL
```
BEGIN BATCH
INSERT INTO myindex.testrt (id, title, content, gid) VALUES (1,'List of HP business laptops','Elitebook Probook',10) IF NOT EXISTS;
INSERT INTO myindex.testrt (id, title, content, gid) VALUES (2,'List of Dell business laptops','Latitude Precision Vostro',10) IF NOT EXISTS;
INSERT INTO myindex.testrt (id, title, content, gid) VALUES (3,'List of Dell gaming laptops','Inspirion Alienware',20) IF NOT EXISTS;
INSERT INTO myindex.testrt (id, title, content, gid) VALUES (4,'Lenovo laptops list','Yoga IdeaPad',30) IF NOT EXISTS;

APPLY BATCH;
```
##### SphinxQL
```
INSERT INTO testrt VALUES(1,'List of HP business laptops','Elitebook Probook',10);
INSERT INTO testrt VALUES(2,'List of Dell business laptops','Latitude Precision Vostro',10);
INSERT INTO testrt VALUES(3,'List of Dell gaming laptops','Inspirion Alienware',20);
INSERT INTO testrt VALUES(4,'Lenovo laptops list','Yoga IdeaPad',30);
INSERT INTO testrt VALUES(5,'List of ASUS ultrabooks and laptops','Zenbook Vivobook',30);
```
#### Query
##### SphinxQL
```
> SELECT * FROM testrt WHERE MATCH('list laptops');
+------+------+
| id   | gid  |
+------+------+
|    2 |   10 |
|    3 |   20 |
|    5 |   30 |
+------+------+
3 rows in set (0.00 sec)
```
