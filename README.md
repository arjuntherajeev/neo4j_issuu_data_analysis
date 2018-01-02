# Getting Started with Data Analysis using Neo4j 

+ Author: Arjun Rajeev Nedungadi (@arjuntherajeev)
+ Field: Data Analysis/Graph Databases
+ Topic: Importing & Performing Queries on a JSON Data Set from [issuu.com](https://issuu.com) using Neo4j

## Why are we doing this?

Data Analysis is the phenomenon of dissecting, structuring and understanding data. In a nutshell - we want to find meaning from our data. In this tutorial, we aim to _analyze_ a data set from [issuu.com](https://issuu.com). The goal is to find answers to a variety of simple and complex questions. 

There are a plethora of tools, techniques and methods available to pursue Data Analysis. We will use Neo4j - a Graph Database, to represent and visualize the data. It uses a query language called __Cypher__ which allows us to build queries and find all the answers that we seek. By the end of this tutorial, we will be able to import a JSON data set to Neo4j and comfortably perform queries on our data.

## Recommended Reading 

Readers are encouraged to read and possess a basic understanding about Neo4j and Graph Databases prior to starting this tutorial.
I previously published a beginner's guide to Neo4j - [__Getting Started with Neo4j__](https://github.com/arjuntherajeev/neo4jStarterLesson) which introduces the concepts of Graph Databases and elaborates on how to perform basic CRUD (Create-Read-Update-Delete) operations using Neo4j. 


## Getting Set Up 

The first thing we need to do is to [download](https://neo4j.com/download/community-edition/) and [install](https://neo4j.com/docs/operations-manual/current/installation/) Neo4j. 
We will use the Neo4j __Web Interface__ which provides a _user-friendly_ UI to visualise the Graph and run queries. The Web Interface can be accessed via any Web Browser on `localhost` or `127.0.0.1` on (default) port `7474`.

The next step is to acquire and understand the data set!

## 1. The Data Set <a id="chapter-1"></a>

The data set that we will analyze comes from [issuu.com](https://issuu.com) - an online repository for magazines, catalogs, newspapers, and other publications. They published the [__Issuu Research Dataset__](http://labs.issuu.com/dataset_spec.html) with a treasure of data about documents and visitors. The data set is completely anonymised and provides an insight into the usage of the website. 

The data is available in the `JSON` format. It can be downloaded/accessed from this GitHub repository. There are two flavors of this file: 
1. A small version - [`issuu_sample.json`](https://github.com/arjuntherajeev/neo4j_issuu_data_analysis/blob/master/issuu_sample.json) (This version of the data set has `4` entries). 
2. A large version - [`issuu_cw2.json`](https://github.com/arjuntherajeev/neo4j_issuu_data_analysis/blob/master/issuu_cw2.json) (This version of the data set has `10,000` entries). 

Both these data sets have been slightly modified for the purpose of this tutorial. To summarize the modification - All `JSON` entries are now stored in an array and referenced by a key called `items`.

The data set is vast and the detailed specification is available [here](http://labs.issuu.com/dataset_spec.html). However, we are interested in the following attributes: 

Attribute | Purpose | 
--- | --- |
`env_doc_id` | Uniquely identify each Document |
`visitor_uuid` | Uniquely identify each Visitor |
`visitor_country` | Two-letter code to identify Visitor's country |
`event_type` | Type of action accomplished by Visitor on the Document |

## 2. Understanding the Graph <a id="chapter-2"></a>

Now that we have selected our data set and cherry-picked the necessary attributes, the next step is to formulate the data as a _graph_.
To create the graph in Neo4j, we need to identify the following elements:
+ Nodes 
+ Relationships 
+ Properties 

From our data set, we can identify __2__ Nodes with the following properties: 

Node | Properties | 
--- | --- |
`Document` | `uuid` |
`Visitor` | `uuid`, `country`|

> `uuid` stands for Universally Unique IDentifier.

__Tip: A Node can also be thought as a _Class_ in Object-Oriented Programming!__

What about the Relationships? 
Yes! We can create 1 Relationship between the `Document` and `Visitor` Nodes. 

Relationship | Properties | 
--- | --- |
`Visitor` __viewed__ `Document` | `type` |

The relationship `viewed` is _generic_ in nature. The `type` property indicates the _specific_ type of event that was accomplished.

As an example, if we consider a Visitor __Thomas__ and a Document __A Diary of Jane__, then the Relationship can be illustrated as: Thomas __viewed__ A Diary of Jane. However, the _type_ of viewership could be any one the following: 
+ `impression`
+ `click`
+ `read`
+ `download`
+ `share`
+ `pageread`
+ `pagereadtime`
+ `continuation_load`

Creating a Relationship for each _type_ is not good practice and might create confusion within the Graph. Hence, we add the _type_ as a property. Now, the Relationship can be illustrated as: Visitor __Thomas__ __viewed__ (_and specifically __downloaded___) Document __A Diary of Jane__. 

## 3. Importing the JSON Data Set <a id="chapter-3"></a>

Let's now get our hands dirty! 

Assuming that Neo4j is started (with an appropriate `Database Location` selected), we should first see an empty Graph. This means that there are __no__ Nodes (and Relationships). Our goal is to populate the Graph with the data from the `JSON` data set by defining its skeleton (Nodes and Relationships). 

To accomplish this, we will use the concept of _user defined procedures_ in Neo4j. These procedures are specific functionalities which can be _re-used_ to manipulate the Graph. We will specifically use a procedure from the __APOC__ library which is a collection of 200+ commonly used procedures. 

__Tip: More information about user defined procedures and the APOC library (installation, usage, examples) is elaborated on a Neo4j blog article titled - [APOC: An Introduction to User-Defined Procedures and APOC](https://neo4j.com/blog/intro-user-defined-procedures-apoc/).__

We are interested in a specific procedure - `apoc.load.json` which will allow us to load data from a `JSON` document. This procedure will return a singular map if the result is a `JSON` object or a stream of maps if the result is an array. 

The following code snippet illustrates the use of the `apoc.load.json` procedure with our _large_ data set - [`issuu_cw2.json`](https://github.com/arjuntherajeev/neo4j_issuu_data_analysis/blob/master/issuu_cw2.json):
```
WITH "/path/to/issuu_cw2.json" AS url
CALL apoc.load.json(url) 
YIELD value
```

Now, `value` contains our _data_ which we need to utilize to create the Nodes and Relationships. However, we first need to _parse_ this bulk `JSON` data. 

To do this, we can use the `UNWIND` keyword in Neo4j. In a nutshell, `UNWIND` will expand a list into a sequence of rows (More information about `UNWIND` is available [here](https://neo4j.com/docs/developer-manual/current/cypher/clauses/unwind/)). At this stage, we are required to possess an understanding of the _structure_ of the JSON data. As mentioned in [Chapter 1](#chapter-1), we can access the entries using the key `items`. 

This is illustrated as: 
```
UNWIND value.items AS item
```

At this stage, we have access to a single row via `item`. Now, we can use the API to access the values! 

In Neo4j, the `WITH` clause allows using the output of a sub-query with following sub-query parts (More information about `WITH` is available [here](http://neo4j.com/docs/developer-manual/current/cypher/clauses/with/)).

From our data, we are concerned about Documents from the [issuu.com](https://issuu.com) "Reader" software. We uniquely identify these Documents using the `env_doc_id` attribute. However, it is to be noted that not all Documents have the `env_doc_id` attribute. Thus, we are required to explicity select those entries which possess the attribute. The following code snippet illustrates this: 

```
WITH item
WHERE NOT item.env_doc_id IS NULL
```

> Notice how we access the value using `item.env_doc_id`. This style of retrieving the value makes working with `JSON` data on Neo4j a smooth experience!

Now that we have access to the values of _each_ entry, it is time to create the Nodes and Relationships. This is accomplished using the `MERGE` keyword in Neo4j. It is _crucial_ to know the difference between `CREATE` and `MERGE`. As an example: 

If we execute the following statements (in order):
```
CREATE (d:Document {id:1}) RETURN (d) 
CREATE (d:Document {id:1}) RETURN (d)
```
This will result in __2 Nodes__ being created! A way to control this is by using `MERGE` which will create the Node __only__ if it does not exist. This can illustrated as follows:
```
MERGE (d:Document {id:1}) RETURN (d)
```

This same principle can be applied for Relationships as well! 

Hence, the final __Cypher__ query consisting of all the above components will look like:
```
WITH "https://raw.githubusercontent.com/arjuntherajeev/neo4j_issuu_data_analysis/master/issuu_cw2.json" AS url
CALL apoc.load.json(url) 
YIELD value
UNWIND value.items AS item
WITH item
WHERE NOT item.env_doc_id IS NULL
MERGE (document:Document {doc_uuid:item.env_doc_id})
MERGE (visitor:Visitor {visitor_uuid:item.visitor_uuid, visitor_country:item.visitor_country})
MERGE (visitor)-[:VIEWED{type:item.event_type}]->(document)
```

If we run this query verbatim on Neo4j, the output should be (similar to):
```
Added 2294 labels, created 2294 nodes, set 5752 properties, created 2171 relationships, statement executed in 12230 ms.
```

__Reminder: Make sure APOC is correctly installed as described [here](https://neo4j.com/blog/intro-user-defined-procedures-apoc/). This is ensure that the `apoc.load.json` procedure is available for use!__

To check whether the Graph was populated successfully, we can run the __Cypher__ query: `MATCH (n) RETURN (n) LIMIT 25` which will only display the top 25 results. 

The output can be visualized as follows:
<img src="https://raw.githubusercontent.com/arjuntherajeev/neo4j_issuu_data_analysis/master/graph_image.svg?sanitize=true">

## 4. Creating Indexes <a id="chapter-4"></a>

Now that we have populated the Graph with Nodes and Relationships, it is time to think about retrieval! 

The main idea is to _query_ the Graph to derive _insights_ about the data set. One way to improve the efficiency of retrieval of data is by using the concept of __Indexes__. The idea behind an Index here is the same as in __Relational__ and __NoSQL__ databases.

We can view existing Indexes in Neo4j using the query:
```
CALL db.indexes
```
It should return: 
```
(no rows)
```

In Neo4j, an Index can be created on a _single_ property of a Label, these are known as __Single-Property Indexes__. An Index can also be created on _multiple_ properties of a Label, these are known as __Composite Indexes__. 

In our Graph, we have __2__ Labels: `Document` and `Visitor`. We will create __Single-Property Indexes__ on `Document` and `Visitor` as follows:

For the `Document` Label, we would like to create an Index on the __property__ `doc_uuid` which is the unique identifier for each Document Node in the Graph. 
```
CREATE INDEX ON :Document(doc_uuid)
```
It should return: 
```
Added 1 index, statement executed in 135 ms.
```
For the `Visitor` Label, we would like to create an Index on the __property__ `visitor_uuid` which is the unique identifier for each Visitor Node in the Graph. 
```
CREATE INDEX ON :Visitor(visitor_uuid)
```
It should return: 
```
Added 1 index, statement executed in 86 ms.
```

We can confirm the existence of the Indexes using `CALL db.indexes` which should return: 
```
╒══════════════════════════════╤══════╤═══════════════════╕
│description                   │state │type               │
╞══════════════════════════════╪══════╪═══════════════════╡
│INDEX ON :Document(doc_uuid)  │online│node_label_property│
├──────────────────────────────┼──────┼───────────────────┤
│INDEX ON :Visitor(visitor_uuid│online│node_label_property│
│)                             │      │                   │
└──────────────────────────────┴──────┴───────────────────┘
```

## 5. Let's Query <a id="chapter-5"></a>

Finally, we can derive insights from our data! 

We need to ask our Graph _questions_. These questions need to be translated to __Cypher__ queries which will return the appropriate results. Let us answer some basic and advanced questions about the data set:

__Note: For the purpose of this tutorial, we will only display the top 10 results for queries with a large number of rows. This is achieved by using the `LIMIT 10` constraint.__

### Query 1. Find the number of visitors from each country and display them in the _descending_ order of count. 
```
MATCH (v:Visitor) 
RETURN v.visitor_country AS Country, count(v) AS Count 
ORDER BY count(v) DESC 
LIMIT 10
```
__Result:__
```
╒═══════╤═════╕
│Country│Count│
╞═══════╪═════╡
│US     │312  │
├───────┼─────┤
│BR     │143  │
├───────┼─────┤
│MX     │136  │
├───────┼─────┤
│PE     │47   │
├───────┼─────┤
│CA     │46   │
├───────┼─────┤
│ES     │43   │
├───────┼─────┤
│GB     │36   │
├───────┼─────┤
│AR     │35   │
├───────┼─────┤
│FR     │34   │
├───────┼─────┤
│CO     │32   │
└───────┴─────┘
```
__Discussion:__

This query simply performs an internal _group by_ operation where Visitor Nodes are grouped based on the `visitor_country` property. The count is computed using the `count()` aggregate function. We sort the results in the descending order using the `ORDER BY <column> DESC` clause in Neo4j.

### Query 2. For a given document, find the number of visitors from each country. (Example Document UUID = `140228101942-d4c9bd33cc299cc53d584ca1a4bf15d9`)
```
MATCH (d:Document)<-[:VIEWED]-(v:Visitor)
WHERE d.doc_uuid='140228101942-d4c9bd33cc299cc53d584ca1a4bf15d9'
RETURN v.visitor_country AS Country, count(v.visitor_country) AS Count 
ORDER BY count(v.visitor_country) DESC
```
__Result:__
```
╒═══════╤═════╕
│Country│Count│
╞═══════╪═════╡
│GY     │15   │
├───────┼─────┤
│CA     │12   │
├───────┼─────┤
│US     │11   │
├───────┼─────┤
│CW     │1    │
├───────┼─────┤
│BB     │1    │
└───────┴─────┘
```
__Discussion:__

This query is very similar to the one above. Here, we perform an internal _group by_ operation to group the Visitor Nodes based on the `visitor_country` property. However, this query differs from the previous one in the sense that we want to _filter_ the counts for a particular __Document UUID__. 

In order to achieve this filteration, we need to utilise the Relationship within the Graph. Hence, we first `MATCH`, filter using the `WHERE` clause and then return the desired values. 

__Tip: The relationship given here: `MATCH (d:Document)<-[:VIEWED]-(v:Visitor)` can also be written as `MATCH (v:Visitor)-[:VIEWED]->(d:Document)`.__

### Query 3. Find the number of occurrences for each _type_ of viewership activity. 
```
MATCH (d:Document)<-[r:VIEWED]-(v:Visitor)
RETURN r.type AS Type, count(d.doc_uuid) AS Count
ORDER BY Count ASC
```
__Result:__
```
╒════════════╤═════╕
│Type        │Count│
╞════════════╪═════╡
│click       │1    │
├────────────┼─────┤
│read        │63   │
├────────────┼─────┤
│pageread    │369  │
├────────────┼─────┤
│pagereadtime│779  │
├────────────┼─────┤
│impression  │959  │
└────────────┴─────┘
```
__Discussion:__

This query also performs an internal _group by_ operation on the Relationship property `type`. An interesting aspect of this query is the `ORDER BY Count ASC`. Previously, we followed  the style of using `ORDER BY count(d.doc_uuid) ASC`. However, once we add a _column name_ such as `Count`, we can use that in subsequent parts of the query. 

Hence, `ORDER BY count(d.doc_uuid) ASC` can also be written as `ORDER BY Count ASC`.

### Query 4. Find the visitors for each document and display the top 3 in the _descending_ order of number of visitors.
```
MATCH (d:Document)<-[r:VIEWED]-(v:Visitor)
RETURN DISTINCT d.doc_uuid AS DocUUID, collect(v.visitor_uuid) AS Collect, count(v.visitor_uuid) AS Count
ORDER BY Count DESC
LIMIT 3
```
__Result:__
```

### Query 5. For a given document, find other documents _like_ the one given. (This is also known as the _also-likes functionality_).
```

```
