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
`Document` | `doc_uuid` |
`Visitor` | `visitor_uuid`, `country`|

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

The Relationship can be illustrated as: Visitor __Thomas__ __viewed__ (_and specifically __downloaded___) Document __A Diary of Jane__. 

## 3. Creating Constraints & Indexes <a id="chapter-3"></a>

A __Constraint__ is a mechanism to control and ensure _data integrity_ in Neo4j. Constraints can be created on either Nodes or Relationships. There are basically three types of Constraints in Neo4j: 

1. Unique Node Property Constraints - to ensure that the Graph contains only a single Node with a specific Label and Property value.
2. Node Property Existence Constraints - to ensure that a certain Property of a specific Label exists within all Nodes in the Graph.  
3. Relationship Property Existence Constraints - to ensure that a certain Property exists within all Relationships of a specific structure.  
4. Node Keys - to ensure that, for a given Label and set of Properties, there exists all those Properties for that Label and that the combination of Property values is unique. Essentially, a combination of _existence_ and _uniqueness_.

We will create __Unique Node Property Constraints__ for our Graph as follows: 

On the `Document` Label for the Property `doc_uuid`: 
```
CREATE CONSTRAINT ON (d:Document) ASSERT d.doc_uuid IS UNIQUE
```
On the `Visitor` Label for the Property `visitor_uuid`:
```
CREATE CONSTRAINT ON (v:Visitor) ASSERT v.visitor_uuid IS UNIQUE
```

The main goal of this exercise is to _query_ the Graph to derive _insights_ about the data set. One way to improve the efficiency of retrieval of data is by using the concept of __Indexes__. The idea behind an Index here is the same as in __Relational__ and __NoSQL__ databases.

In Neo4j, an Index can be created on a _single_ property of a Label, these are known as __Single-Property Indexes__. An Index can also be created on _multiple_ properties of a Label, these are known as __Composite Indexes__. 

It is important to understand that, by creating a __Unique Node Property Constraint__ on a Property, Neo4j will alo create a __Single-Property Index__ on that Property. Hence, in our situation, the Indexes will be created on the `doc_uuid` Property for the `Document` Label and on the `visitor_uuid` Property for the `Visitor` Label.

Now that we created the Constraints, we can view existing Indexes in Neo4j using the query:
```
CALL db.indexes
```
It should return: 
```
╒═══════════════════════════════╤══════╤════════════════════╕
│description                    │state │type                │
╞═══════════════════════════════╪══════╪════════════════════╡
│INDEX ON :Document(doc_uuid)   │online│node_unique_property│
├───────────────────────────────┼──────┼────────────────────┤
│INDEX ON :Visitor(visitor_uuid)│online│node_unique_property│
└───────────────────────────────┴──────┴────────────────────┘
```
__Tip: More information about Constraints and their effect on Indexes is elaborated [here](http://neo4j.com/docs/developer-manual/current/cypher/schema/constraints/).__

## 4. Importing the JSON Data Set <a id="chapter-4"></a>

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
CREATE (d:Document {doc_uuid:1}) RETURN (d) 
CREATE (d:Document {doc_uuid:1}) RETURN (d)
```
This will result in __2 Nodes__ being created! A way to control this is by using `MERGE` which will create the Node __only__ if it does not exist. This can illustrated as follows:
```
MERGE (d:Document {doc_uuid:1}) RETURN (d)
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
Added 2293 labels, created 2293 nodes, set 5749 properties, created 2170 relationships, statement executed in 15523 ms.
```

__Reminder: Make sure APOC is correctly installed as described [here](https://neo4j.com/blog/intro-user-defined-procedures-apoc/). This is to ensure that the `apoc.load.json` procedure is available for use!__

To check whether the Graph was populated successfully, we can run the __Cypher__ query: `MATCH (n) RETURN (n) LIMIT 200` which will only display the top 200 results. 

The output can be visualized as follows:
<img src="https://raw.githubusercontent.com/arjuntherajeev/neo4j_issuu_data_analysis/master/graph_image_large.svg?sanitize=true">

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
│MX     │135  │
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
│read        │62   │
├────────────┼─────┤
│pageread    │369  │
├────────────┼─────┤
│pagereadtime│779  │
├────────────┼─────┤
│impression  │959  │
└────────────┴─────┘
```
__Discussion:__

This query also performs an internal _group by_ operation on the Relationship property `type`. An interesting aspect of this query is the `ORDER BY Count ASC`. Previously, we followed the style of using `ORDER BY count(d.doc_uuid) ASC`. However, once we add a _column name_ such as `Count`, we can use that in subsequent parts of the query. 

Hence, `ORDER BY count(d.doc_uuid) ASC` can also be written as `ORDER BY Count ASC`.

### Query 4. Find the visitors for each document and display the top 3 in the _descending_ order of number of visitors.
```
MATCH (d:Document)<-[r:VIEWED]-(v:Visitor)
RETURN DISTINCT d.doc_uuid AS DocUUID, collect(DISTINCT v.visitor_uuid) AS Collect, count(DISTINCT v.visitor_uuid) AS Count
ORDER BY Count DESC
LIMIT 3
```
__Result:__
```
╒══════════════════════════════╤══════════════════════════════╤═════╕
│DocUUID                       │Collect                       │Count│
╞══════════════════════════════╪══════════════════════════════╪═════╡
│140224101516-e5c074c3404177518│[923f25aa749f67f6, 4f4bd7a35b2│26   │
│bab9d7a65fb578e               │0bd1f, 19f5285fef7c1f00, f102d│     │
│                              │9d4fc4bacdc, 3819fc022d225057,│     │
│                              │ abefb3fe7784f8d3, e5d957682bc│     │
│                              │8273b, 4d49271019c7ed96, 6b2d3│     │
│                              │cca6c1f8595, 78e1a8af51d44194,│     │
│                              │ b0ba142cdbf01b11, 0930437b533│     │
│                              │a0031, d6b90f07f29781e0, 7bd81│     │
│                              │3cddec2f1b7, e1bfcb29e0f3664a,│     │
│                              │ 3db0cb8f357dcc71, 6d87bcdc5fa│     │
│                              │5865a, 6170372b90397fb3, 3ec46│     │
│                              │5aa8f36302b, 43dd7a8b2fafe059,│     │
│                              │ 797846998c5624ca, ee14da6b126│     │
│                              │3a51e, e3392e4a18d3370e, 23f8a│     │
│                              │503291a948d, 502ddaaa898e57c4,│     │
│                              │ 6fd040328d2ad46f]            │     │
├──────────────────────────────┼──────────────────────────────┼─────┤
│140228202800-6ef39a241f35301a9│[43b59d36985d8223, 355361a3510│25   │
│a42cd0ed21e5fb0               │94143, 51fd872df55686a5, 4e2a7│     │
│                              │5f30e6b4ce7, 6229cca3564cb1d1,│     │
│                              │ 52873ed85700e41f, 2f63e0cca69│     │
│                              │0da91, febc786c33113a8e, 47d26│     │
│                              │08ec1f9127b, ca8079a4aaff28cb,│     │
│                              │ e8fa4a9e63248deb, b6169f1bebb│     │
│                              │be3ad, b3ded380cc8fdd24, 17db8│     │
│                              │6d260543ddd, b2a24f14bb5c9ea3,│     │
│                              │ fabc9339a406616d, 32563acf872│     │
│                              │f5449, 280bd96790ade2d4, 2f21e│     │
│                              │e71e0c6a2ce, 458999cbf4307f34,│     │
│                              │ 55ac6c3ce6325228, 6d3b99b2041│     │
│                              │af286, 36a12501ee94d15c, d2d6e│     │
│                              │7d1a25ee0b0, 13ca53a93b1594bf]│     │
├──────────────────────────────┼──────────────────────────────┼─────┤
│140228101942-d4c9bd33cc299cc53│[ba76461cdd66d337, fa3810e505f│24   │
│d584ca1a4bf15d9               │4f792, d5ed3cfc4a454fe9, 2d415│     │
│                              │36695cc4814, 67c698e88b4fbdcc,│     │
│                              │ 78deb8ffdb03d406, ee42ba15ed8│     │
│                              │618eb, 688eb0dcd6ad8c86, c97c3│     │
│                              │83d774deae0, 9b2cb60327cb7736,│     │
│                              │ 06d465bfb51b0736, a96854d2178│     │
│                              │0c1f9, d1c98b02398e9677, 14355│     │
│                              │42d699350d9, a8cf3c4f1449cc5d,│     │
│                              │ b1cdbeca3a556b72, 08c069dc405│     │
│                              │cad2e, 5b2baf0329663564, 0d477│     │
│                              │95fb1ddba9d, 667283570b5cedfe,│     │
│                              │ 6c661964d1d13c61, f5986e1cb02│     │
│                              │378e4, 0e8ddc2d2a60e14f, 6823c│     │
│                              │573efad29f6]                  │     │
└──────────────────────────────┴──────────────────────────────┴─────┘
```
__Discussion:__

This query utilizes the `collect()` aggregate function which groups multiple records into a _list_. An important consideration made here is the use of the `DISTINCT` operator to ensure that _duplicate_ values are omitted from the output. Finally, we display the _top 3_ using the `LIMIT 3` constraint. 

### Query 5. For a given document, find recommendations of other documents _like_ it. 

__Example 1: Document UUID = `130205102930-4a65860329964f3790e39d482ff86bc7`__
```
MATCH (d:Document)<-[:VIEWED]-(v:Visitor)
WHERE d.doc_uuid = '130205102930-4a65860329964f3790e39d482ff86bc7'
WITH collect(v) as visitor_col, d.doc_uuid as uuid
UNWIND visitor_col as vis
MATCH (vis)-[:VIEWED]->(d1:Document)
WHERE d1.doc_uuid <> uuid
RETURN uuid, collect(DISTINCT vis.visitor_uuid), (collect(DISTINCT d1.doc_uuid))
```
__Result:__
```
╒══════════════════════════════╤══════════════════════════════╤══════════════════════════════╕
│uuid                          │collect(DISTINCT vis.visitor_u│(collect(DISTINCT d1.doc_uuid)│
│                              │uid)                          │)                             │
╞══════════════════════════════╪══════════════════════════════╪══════════════════════════════╡
│130205102930-4a65860329964f379│[4f9e21d836e90dcf]            │[131010151705-60d7ba5cbcd6a255│
│0e39d482ff86bc7               │                              │652ca9d7a27f76d3, 140122153540│
│                              │                              │-94ee7b54cb95f635349a63a82835e│
│                              │                              │92b, 130205150342-921840826cfb│
│                              │                              │4f988682db05aaa32d2e, 13020514│
│                              │                              │4459-7a7448cd74dd45458feb1307d│
│                              │                              │907fddb, 130226165227-be6ae627│
│                              │                              │fd5f4be0b9a0705b43c7286a, 1305│
│                              │                              │20152036-0fc36dda7e7546bdb9430│
│                              │                              │26e50690814, 140121155329-9856│
│                              │                              │0c64f9e276da0f0cf433fb50fa63, │
│                              │                              │130416135921-659f9c05a9ea4ffc9│
│                              │                              │3739ffd8a04908b]              │
└──────────────────────────────┴──────────────────────────────┴──────────────────────────────┘
```
__Example 2: Document UUID = `130323125939-5f4318404cda4025a2463c66435ad7c8`__
```
MATCH (d:Document)<-[:VIEWED]-(v:Visitor)
WHERE d.doc_uuid = '130323125939-5f4318404cda4025a2463c66435ad7c8'
WITH collect(v) as visitor_col, d.doc_uuid as uuid
UNWIND visitor_col as vis
MATCH (vis)-[:VIEWED]->(d1:Document)
WHERE d1.doc_uuid <> uuid
RETURN uuid, collect(DISTINCT vis.visitor_uuid), (collect(DISTINCT d1.doc_uuid))
```
__Result:__
```
╒══════════════════════════════╤══════════════════════════════╤══════════════════════════════╕
│uuid                          │collect(DISTINCT vis.visitor_u│(collect(DISTINCT d1.doc_uuid)│
│                              │uid)                          │)                             │
╞══════════════════════════════╪══════════════════════════════╪══════════════════════════════╡
│130323125939-5f4318404cda4025a│[823babba91f61957, 6206d63a841│[130428213605-d16c4651f5624855│
│2463c66435ad7c8               │bb27f, aaa4eaf77abab0b2, 8b82e│9b242f75ad4884f5, 130901012449│
│                              │1a60343f2d5]                  │-98b43cc6fd8e4d1ccbcac441b9fb2│
│                              │                              │8cd, 131204101549-619a95597b70│
│                              │                              │2aed1369d851c891fd60, 13060917│
│                              │                              │1715-7c9c218afa6cb11f91babc13f│
│                              │                              │6c8394a]                      │
└──────────────────────────────┴──────────────────────────────┴──────────────────────────────┘
```
__Discussion:__
This query

## Summary <a id="chapter-6"></a>

There you go, ladies & gentlemen! 

In this tutorial, we saw an example of performing Data Analysis using Neo4j. We examined the [__Issuu Research Dataset__](http://labs.issuu.com/dataset_spec.html) and elaborated on its structure, format and fields. Next, we formulated the _model/schema_ of our desired Graph by choosing appropriate Nodes, Relationships and Properties. Further, we created Constraints and Indexes in Neo4j to ensure uniqueness and improve the performance of _querying_. After this, we discussed how to import the raw `JSON` data set, parse it and populate our Graph by following the previously determined schema. Lastly, we saw some sample __Cypher__ queries which helped us derive insight from our vast data set! 

## References <a id="chapter-7"></a>

+ [Cypher Query Language](https://neo4j.com/developer/cypher-query-language/)
+ [Neo4j Documentation](https://neo4j.com/docs/)
+ [APOC: An Introduction to User-Defined Procedures and APOC](https://neo4j.com/blog/intro-user-defined-procedures-apoc/)
+ [Issuu Research Dataset](http://labs.issuu.com/dataset_spec.html)

## What's Next? <a id="chapter-8"></a>

The possibilities are endless! If you enjoyed this tutorial, then you can try to derive insights using another data set of your choice! While we chose to construct a rather simple Graph, you can make it much more complex and detailed. Further to this, you can also explore and experiment with the various [APOC user defined procedures](https://neo4j-contrib.github.io/neo4j-apoc-procedures/) on the Graph! 

## Note <a id="chapter-9"></a>

If you have any other interesting Cypher queries that might be useful to readers, please feel free to create a new __Issue__ and include a short _description_ and the _query_ itself. 

