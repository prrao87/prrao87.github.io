+++ 
draft = true
date = 2023-09-04
title = "Embedded databases (2): Deep diving into KùzuDB, a modern, scalable & fast graph database"
description = "A benchmark study comparing the performance of KùzuDB vs. Neo4j on an artificial social network dataset"
tags = ["embedded-db", "graph-db", "neo4j"]
categories = ["databases"]
+++

## A benchmark study on graph databases: Kùzu and Neo4j

Embedded databases, which deviate from the well-known client/server architecture that we're used to, have been experiencing something of a renaissance lately. In the [first post](../embedded-db-1/), I gave a detailed overview on the embedded database landscape, breaking them down into domains. This second post focuses purely on graph databases -- in particular, [KùzuDB](https://github.com/kuzudb/kuzu), a modern graph database written in C++ that's emerging as a **very** viable option for users that want to run expensive, multi-hop queries on very large graphs.

Because it's hard to gauge performance in isolation, I'll be creating and running a benchmark study on two databases: Kùzu and [Neo4j](https://github.com/neo4j/neo4j), the most well-known (and potentially, most-used) graph database in the market. The study involves ingesting and querying an artificial social network dataset that's large enough to compare performance across either database, and hand-constructed in way that reveals interesting graph structures.

{{<notice note>}}
🚩 The aim of this post is **_NOT_** to state that one database is better or worse than the other. Please take this as a purely informative exercise, and come to your own conclusions by testing out such a workflow on your own data.
{{</notice>}}

### What makes Kùzu different?

The primary feature of Kùzu that stands out is, of course, its embedded architecture -- Kùzu is designed to run in-process, such that the database itself sits natively within the application code. In their blog post outlining their vision[^1], the makers of Kùzu very explicitly state their goal of replicating what DuckDB has done in the world of relational DBs, but for graph analytics applications.

However, there are other key design decisions of Kùzu that (in my opinion) make it particularly interesting and useful for graph practitioners.

1. First-class support for the labelled property graph (LPG) model, which was popularized by Neo4j and is now more or less the de-facto standard for graph databases in industry.
    - However, this doesn't mean that RDF graphs are not supported -- this is ongoing work and is part of the Kùzu roadmap[^2].
2. Choice of [openCypher](https://opencypher.org/) as its query language, which allows new users to leverage their existing knowledge of Cypher (if coming from Neo4j), and to easily port queries from Neo4j to Kùzu.
    - One of the main focuses of Kùzu is to achieve near-full feature parity with Neo4j's implementation of Cypher, and the current version of Kùzu is already quite close to this goal.
3. Built for vectorized execution of OLAP-type queries, which means that Kùzu is optimized for running expensive, multi-hop queries on very large graphs.
4. Can be used as the storage layer for machine learning on graphs, due to a high degree of interoperability with geometric deep learning frameworks like PyG and DGL, as well as graph analytics frameworks like NetworkX.

{{<figure src="../embedded-db-1/embedded-db-kuzudb.png" caption="Where KùzuDB sits in the graph analytics value chain">}}

## Dataset

Graph databases, and the graph data structure in general, are ideally suited to run queries on highly-connected data. If done in a relational database, this could involve expensive many-to-many joins on numerous tables, which can be very slow. In contrast, graph databases are designed to run such queries very efficiently, and are therefore well-suited for use cases like social networks, which is why this sort of dataset is constructed specifically for this study.

The dataset consists of a social network of 100,000 persons, with roughly 2.4 million edges between them, as well as their interests and the locations in which they live. To keep things simpler (and more in line with the anglicized fake names generated), only three countries and their cities exist -- United States, United Kingdom and Canada. The schema for this graph is shown below.

{{<figure src="kuzudb-graph-schema.png">}}

* `Person` node `FOLLOWS` another `Person` node
* `Person` node `LIVES_IN` a `City` node
* `Person` node `HAS_INTEREST` towards an `Interest` node
* `City` node is `CITY_IN` a `State` node
* `State` node is `STATE_IN` a `Country` node

To make the dataset more interesting, a small subset (~0.5%) of the `Person` nodes are considered "hubs", i.e., they are connected to up to 5% of all the other persons in the graph. This is to simulate the fact that in real-world social networks, there's a small fraction of people who are _very_ well-connected and have a large number of followers.

As a result, the generated graph between persons has some interesting structures, like cliques, trees and cycles, and is visualized below.

{{<figure src="person-person.png" caption="Cliques, trees and cycles in the graph of connected persons">}}

A more detailed explanation on the dataset generation is shown in the [GitHub repo](https://github.com/prrao87/kuzudb-study/tree/main/data). Note that the data is generated using a fixed random seed, so **it is fully reproducible**, and the number of artificial nodes and edges generated can be scaled up or down, in a fully reproducible manner, as needed.

## Queries

In order to run a reasonably sound benchmark, it's necessary to test a broad variety of queries on the dataset. This section walks through the queries that were chosen for this study, the rationale behind them, as well as the results obtained from either graph database. To make the results easier to read, the data is fetched from the database in question, materialized and then transformed to a Polars DataFrame for display.

---
## Benchmark

### Data ingestion

The data for the nodes and edges is stored as parquet, read and transformed to a list of dictionaries in Python, and ingested via the async API of the [Neo4j Python client](https://github.com/neo4j/neo4j-python-driver). Best practices as per the Neo4j docs for data ingestion via Python are followed, such as setting uniqueness constraints on nodes and relationships, and using `UNWIND` to ingest data in batches.



### Query benchmark


---

## Kùzu

### Data ingestion

### Query benchmark

## Observations

### Why was Kùzu that much faster than Neo4j?

### What are the current limitations of Kùzu?

* Support for in-built graph algorithms is still a work in progress (shortest path is available)
* 


### Thoughts on scalability

* Describe the extremely large graphs that Kùzu has been tested on (like ldbc).
* Describe the latency reduction due to the embedded design, and the fact that queries run as close to bare-metal as possible with the necessary optimizations in place.


## Code

All the code required to reproduce this study is available in my [GitHub repo](https://github.com/prrao87/kuzudb-study/tree/main). I'd love to hear from anybody who uses a similar approach on their own data, so please reach out if you do!

### Additional resources

* Kùzu [Slack](https://join.slack.com/t/kuzudb/shared_invite/zt-1w0thj6s7-0bLaU8Sb~4fDMKJ~oejG_g): A great place to discuss ideas and issues
* Kùzu [Blog](https://kuzudb.com/docusaurus/blog/): A great place to learn about the latest developments, including the deep technical details of their implementation


[^1]: What every competent graph database management system should do, [KùzuDB blog](https://kuzudb.com/docusaurus/blog/what-every-gdbms-should-do-and-vision/)
[^2]: RDF support roadmap, [KùzuDB GitHub](https://github.com/kuzudb/kuzu/issues/1570)