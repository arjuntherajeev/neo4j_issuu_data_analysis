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

This is illustrated as: `UNWIND value.items AS item`

At this stage, it is pertinent to know that `item` 

As described in [Chapter 2](#chapter-2), we have 2 Nodes and 1 Relationship within the Graph.
`
WITH "https://raw.githubusercontent.com/arjuntherajeev/neo4j_issuu_data_analysis/master/issuu_cw2.json" AS url
CALL apoc.load.json(url) YIELD value
UNWIND value.items AS item
WITH item
WHERE NOT item.env_doc_id IS NULL
MERGE (document:Document {doc_uuid:item.env_doc_id})
MERGE (visitor:Visitor {visitor_uuid:item.visitor_uuid})
MERGE (visitor)-[:VIEWED{activity:item.event_type}]->(document)
`
