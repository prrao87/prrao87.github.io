+++ 
draft = true
date = 2023-08-26
title = "Embedded databases (1): The harmony of data model paradigms in the age of Arrow"
description = "A look at why these embedded in the relational, graph and vector paradigms"
tags = ["embedded-db", "vector-db", "graph-db"]
categories = ["databases"]
+++

## Embedded databases are here to stay

In the world of database systems, the client/server model has been by far the most widespread successful. Databases whose names you're likely very familiar with, like PostgreSQL, MySQL, MongoDB and many others, fall in this camp, whose foundations were laid in the early days of the world wide web. The client/server model was a natural fit because it allowed for a separation of concerns between the client (the web browser) and the server (the web server). In most applications in use even today, we think about applications using this architecture, where the client is responsible for rendering the user interface, while the server is responsible for storing and serving the data.

It's not a stretch to say that the world has progressed a *long* way since the early days of the web. As computing power has grown exponentially, modern embedded databases are able to do a lot more, *with a lot less*. Although the idea of embedded databases is not new (SQLite has been around since 2000! 🤯), there have been numerous research breakthroughs in the last decade, making it possible for vendors to build fast, lightweight, and easy to use alternatives to established solutions like Postgres or MySQL, especially for the massive analytical and ML workloads we're seeing today.

### The rise of enterprise data warehouses

Real-world data workloads can be broadly broken into two: **OLTP** (online transactional processing and) **OLAP** (online analytical processing). OLTP workloads involve a lot of small, fast transactions, like updating a user's profile, or adding a new item to a shopping cart. OLAP workloads, on the other hand, involve a lot of complex, long-running queries, like aggregating over a large dataset, or joining several large tables. Over the years, the client/server model has been a great fit for OLTP workloads, but has been less than ideal for OLAP workloads, mainly because their underlying storage is row-oriented. This has led to a host of OLAP data warehouses being built specifically to process heavy analytical workloads, such as Snowflake, Clickhouse and BigQuery, which are designed to use columnar storage.

However, a lot of these enterprise solutions are designed for "big" data (whatever that terms means for your organization). But there's a whole ocean of use cases *in between* single-CPU, in-memory analytics and large-scale, distributed analytics. It's in this middle ground where embedded databases shine ✨, because they're designed to be lightweight, easy to use, and are extremely performant, from small-scale to large-scale data analytics.

### Focus of this post

The aim of this post, and the upcoming series, is to first gain a birds-eye view of the embedded database landscape, and how they tie into the Apache Arrow ecosystem. From an OLAP perspective, three particular vendors are really exciting: **DuckDB** in the relational world, **KùzuDB** for graphs, and **LanceDB** for vectors. I'm focusing specifically on these three solutions, because they're all open-source, and importantly, **they embrace unorthodoxy** at their core. In this, and the upcoming series of posts, I want to explore what specifically makes them so interesting, fast and performant. As always, code and timing numbers to reproduce the results will be provided! 🔥

{{<figure src="embedded-db-breakdown.png">}}

The bottom half of the image above is particularly interesting: [Arrow](https://arrow.apache.org/), a language-agnostic development ecosystem for in-memory data, is the common denominator between all three of these databases. Arrow is a columnar, in-memory data format that's designed to be a fast, efficient, columnar memory format for flat and hierarchical data, organized for efficient analytic operations on modern CPUs and GPUs.

In this and upcoming posts, I'll describe how these three databases, despite having radically different internals and data modelling paradigms, are able to play well together, and how they can be used to build a powerful, flexible, and fast analytical stack, that can be used for a wide variety of use cases, for nearly all sizes of data.

## What are embedded databases?

Embedded databases are **in-process** database management systems that are tightly integrated with the application layer. The term "in-process" is important because the database computation runs within the same underlying process as the application (which could be written in any language, like Python, R, JavaScript, C++). In cases like RocksDB, a key-value embedded database, the application it runs inside could *itself be another database*[^5]!

In the database community, embedded databases are often referred to as *serverless* databases, because they do not require a client/server architecture (as is prevalent in most large-scale software systems), and this is the definition of serverless I'll be using in this series. Another characteristic of embedded databases is that they are designed from the ground up to clearly separate storage from compute -- data that's larger than memory can be stored and queried on-disk, allowing them to scale to all sizes of data.

{{<notice info>}}

💡 The term "serverless" can mean different things depending on the context (especially if you're coming from a cloud or microservices background), so I'll not be using the terms "embedded" and "serverless" interchangeably.

{{</notice>}}

### A breakdown of the landscape

The three solutions I'll be focusing on are but part of a larger landscape of embedded DBs, which are out of the scope of this series. However, it makes sense to first gain a birds-eye view at the breakdown of the embedded landscape.

{{<figure src="embedded-db-landscape.png" caption="Embedded databases organized by data model paradigm">}}

### Where are embedded databases useful?

In the last few years, the term [edge computing](https://en.wikipedia.org/wiki/Edge_computing) is becoming more and more common. Thanks to advancements in wireless technology, systems engineering, and a massive increase in the number of people connected to the internet, we have around 15 billion IoT devices today, projected to grow to ~30 billion devices by 2030[^1]. As these edge devices become more powerful, they are increasingly capable of running complex software, including operating systems and databases. The key point is: workloads that were thought to be too expensive to run in-process, are becoming more and more possible, even for surprisingly large amounts of data. 

The reason embedded database have a role to play in empowering ML/AI applications of the future, especially on the edge, is largely to do with the following:

* **Privacy**: IoT and mobile devices collect data *on the edge*, i.e., on the device itself. These devices typically have low(ish) computational resources, with a lot of their compute being dedicated to performing application-specific tasks. The data on these devices is often sensitive and could contain private information about the user, so sending it via a network to a remote server to process it for analytics or ML may not be the most secure option. Embedded databases, by their very nature of being tightly integrated with the application, could allow for the data to be stored, queried and analyzed on the device itself.

* Security: Client/server databases often include expensive, complex security features, such as authentication, authorization, encryption, and disaster recovery. Embedded databases, due to the fact that they're tightly connected to the application, can be designed to be lightweight and secure, with only the necessary security features required for the application, greatly reducing the attack surface.

* Latency: Sending data over a network to a remote server for processing can be slow, especially if large amounts of data are being sent at regular intervals on a mobile network. Embedded databases allow for data to be stored and queried on the device itself, massively reducing the latency of the overall system.

* Cost: A lot of the embedded databases available today (like the three mentioned in this post) are open-source and extremely lightweight, while also being extremely performant. For analytical workloads and rapid deployment of applications, embedded databases are a great option, because they don't require any external dependencies, increasing the speed with which teams can take applications to production while reducing the overall cost of the application. In addition, a lot of complex ETL workflows that may have been required to move data from the edge to a remote server (while maintaining synchronization between the two locations) can be avoided, further reducing the cost of the overall system.

All this being said, embedded databases offer a TON of value even in the conventional sense, outside of edge computing, when you have huge amounts of data stored on the cloud. As mentioned before, databases like DuckDB, KùzuDB and LanceDB, allow you to do *more, with less*.

---

## DuckDB

DuckDB is a high-performance embedded relational database system (RDBMS) that can be queried via a rich SQL dialect that's very similar to Postgres. Its core is written in C++, and it's designed to be fast, ACID-compliant, and easy to use. It's designed to support large-scale OLAP query workloads, which are typically characterized by complex, relatively long-running queries that process significant portions of the stored data -- for example, aggregations over entire tables, or joins between several large tables.

### An interesting take on the "big data" narrative

In early 2023, the makers of DuckDB posted a blog with a click-baity title, "*Big data is dead*" [^2]. It was very smartly (and aptly) countered in another article that came shortly after, "*Big data is dead ... Long live big data!*"[^3]. Both articles are very well-written, and contain a lot of insights from folks who have spent years in the world of big data, so it's recommended you give them both a read.

* The thesis: [Big data is dead](https://motherduck.com/blog/big-data-is-dead/), by Jordan Tigani, a founding engineer at Google BigQuery, now founder and CEO of MotherDuck, DuckDB's commercial cloud offering.
* The antithesis: [Big data is dead, long live big data!](https://ponder.io/big-data-is-dead-long-live-big-data/), by Aditya Parameswaran, Associate Professor at UC Berkeley, and co-founder of Ponder, a data science startup.

The first blog post makes the (rather strong) claim that the age of big data is over, and that because a large fraction of organizations don't have Google-scale data, they are thus "overpaying" to enterprise data warehouse vendors for features or scale that they don't need, and a lot of their workloads could instead be run on single nodes.

While this is largely true, the second post rightly counters this viewpoint (while also acknowledging that DuckDB is an excellent solution), by stating that although a large fraction of organizations don't *yet* have big data, they will have big data *in the future*, because the relationship between having data and the ability to grow it (by better leveraging insights from existing data), is nonlinear. Organizations will *eventually* reach humongous scales of data, provided they have the right tools to organically arrive at that scale, and it all starts with having multiple tools that allow them to process data at all scales, from small to large.

I'm very much on board with both camps! Everything has to be taken in context, for each individual use case. 😎

### Key features of DuckDB

All jokes aside, DuckDB is a *serious* database, with a host of unique and amazing features that are straight from the world of academic database research, making it stand out from the crowd. As they mention in their blog[^4], DuckDB stands on the shoulders of giants. A few of their key features for blazing fast OLAP query performance are listed below.

* Like other OLAP data warehouses, DuckDB uses columnar storage, which is a great fit for analytical workloads, because it allows for fast, efficient scans over large amounts of data
* Uses vectorized query execution, which is a technique that allows for processing large amounts of data in batches, which is a great fit for modern CPUs
* Utilizes recent advances in query optimization, with ideas from dynamic programming to unnesting arbitrary subqueries
* Uses concurrent execution via threads, which allows for faster execution of queries, and is a great fit for modern multi-threaded CPUs
* Utilizes a dialect of SQL that is very similar to Postgres (unlike Clickhouse, which deviates quite far from PostgreSQL)
* **Possibly the most important**: DuckDB has the potential to become a universal data connector[^5], due to the sheer number of data formats it's able to natively read data from
  * A lot of ETL processes involve expensive transformations that reshape data from one form to another
  * DuckDB natively reads from formats like CSV, JSON (including nested JSON), Parquet, and has scanners to directly read from Postgres and SQLite databases
  * It also offers connectors that allow users to directly read Parquet data from S3, GCS and Azure storage


Because DuckDB also natively supports the Arrow in-memory format, it's trivial to transform data from relational tables to JSON, and back, and also to transform the data from the relational model to the graph model, as will be shown in a future post on KùzuDB.

## KùzuDB




## LanceDB


---

## Will embedded databases be commercially successful?

The million-dollar question now is how successful embedded databases will be, in terms of monetizing their offerings. The client/server architecture has been around for a *very* long time, and has been proven to be a successful commercial model for numerous databases, and are the norm in large-scale production use cases. The embedded architecture is still relatively new, and several vendors are building them out as open-source technology, and are still figuring out their monetization strategies. However, the universal approach seems to be similar:

* Build a sound open-source offering
* Gain a sizeable user community
* Showcase blazing fast performance 🚀 and ease of use
* Offer a managed cloud service that builds on top of the open-source version, with added convenience features
  * From a pricing perspective, it would be essential to be on par with (or lower than) tried-and-tested client/server databases

{{<notice note>}}
One can only speculate, and time will tell how successful the embedded database monetization strategy will be! But my view is that there is already ample opportunity in both small and large organizations to use embedded databases for a wide variety of use cases, and the three vendors I've mentioned in this post, in my view, seem very well-positioned to be commercially successful in the long run.
{{</notice>}}

### DuckDB + MotherDuck

In 2023, DuckDB made large strides on the monetization front, with its [MotherDuck](https://motherduck.com/company/) commercial cloud offering (currently in beta). It offers the following convenience features on top of its open-source offering[^4], designed specifically for serverless deployment on the cloud.

* Convenient persistent storage via a managed service
* Hybrid execution mode, allowing seamlessly combining querying from in-memory, on-disk, or on-cloud, in a fully distributed fashion
* Secrets management to handle data stored on the cloud
* A notebook-like SQL IDE (similar to DataBricks Spark notebooks) to make data scientists and analysts more productive
* Sharing databases with teammates

### KùzuDB

KùzuDB is still in its early days, but is the furthest ahead among graph DB vendors the quest to provide a scalable and easy-to-use embedded graph DB.

* [Kùzu](https://kuzudb.com/) is a powerful, open-source, ACID-compliant graph database **ready for production**, with great support for the openCypher query language
  * My experiments with it show it to be blazing fast 🔥, and it's able to handle large-scale graph queries with ease: a separate blog to come on this soon!
* As they mention in their blog[^6], Kùzu is aiming to emulate in the graph database world what DuckDB has done in the SQL world, and gain widespread adoption through a sound core that's open-source and scalable
* As the use cases for graph data structures & algorithms proliferate into ML/AI applications, my take is that an as-yet unannounced commercial offering from KùzuDB stands to be a serious competitor to Neo4j in the graph DB space 😎

### LanceDB + LanceDB Cloud

LanceDB is making waves in the world of vector DBs, and although it's still in its early days, it's coming out to be the primary option for vector querying and search directly on cloud storage[^7], in a similar way to DuckDB. I've written more about its unique features in my [other post](../vector-db-3/#conclusions) on vector DBs, specifically focusing on the power of its upcoming DiskANN implementation.

Due to the unique nature of [Lance](https://github.com/lancedb/lance), a new, columnar data format optimized for vector compute (and on top of which LanceDB is built), the management of vector storage and versioning on the cloud is an interesting area that LanceDB is looking to revolutionize. Some of the convenience features on [LanceDB Cloud](https://lancedb.com/), currently in beta, are described below.

* Managed service to store and version Lance datasets in the cloud
* Auto re-indexing
* Caching layer
* Dashboard to more conveniently manage your data


## Conclusions

This was just an initial foray into the fascinating world of embedded databases. As a developer interested in all data model paradigms, I can only speculate on the long-term commercial outcomes of these solutions, but, in the upcoming posts in this series, I'll go deeper into writing code for each of the three solutions I've mentioned here. Specifically, I'll showcase code examples on timing the performance of each DB vs. traditional (client/server) databases in this space. Hope this was interesting, and more to come soon! 🚀


[^1]: IoT connected devices worldwide, [Statista](https://www.statista.com/statistics/1183457/iot-connected-devices-worldwide/)
[^2]: *Big data is dead* by Jordan Tigani, [MotherDuck blog](https://motherduck.com/blog/big-data-is-dead/)
[^3]: B*ig data is dead... Long live big data!* by Aditya Parameswaran, [ponder.io](https://ponder.io/big-data-is-dead-long-live-big-data/)
[^4]: Standing on the shoulders of giants, [DuckDB docs](https://duckdb.org/why_duckdb.html#standing-on-the-shoulders-of-giants)
[^5]: Why we built Rill with DuckDB, [Rill blog](https://www.rilldata.com/blog/why-we-built-rill-with-duckdb)
[^4]: MotherDuck, [DuckDB](https://kestra.io/blogs/2023-07-28-duckdb-vs-motherduck)
[^5]: Building on top of RocksDB, [Cockroach Labs blog](https://www.cockroachlabs.com/blog/cockroachdb-on-rocksd/)
[^6]: What every competent graph database system should do, [KùzuDB blog](https://kuzudb.com/docusaurus/blog/what-every-gdbms-should-do-and-vision/)
[^7]: *S3 backed full-text search with Tantivy* by Rob Meng, [LanceDB blog](https://blog.lancedb.com/s3-backed-full-text-search-with-tantivy-part-1-ac653017068b)
