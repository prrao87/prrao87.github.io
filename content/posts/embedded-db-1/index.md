+++ 
draft = true
date = 2023-08-21
title = "Embedded databases (1): An introduction to DuckDB, KùzuDB and LanceDB"
description = "A look at why embedded DBs matter across the relational, graph and vector paradigms"
tags = ["embedded-db", "vector-db", "graph-db"]
categories = ["databases"]
+++

## The age of embedded (in-process) databases is here

In the world of computing and database systems, the client/server model has been by far the most widespread successful. Databases whose names you're likely very familiar with, like PostgreSQL, MongoDB, Elasticsearch, Neo4j, and many others, fall in this camp, whose concepts were laid in the early days of the world wide web. The client/server model is a natural fit for the web, because it allows for a separation of concerns between the client (the web browser) and the server (the web server). In most applications in use today, the client is responsible for rendering the user interface, while the server is responsible for storing and serving the data.

However, the world has progressed a *long* way since the early days of the web. Thanks to advancements in wireless technology, systems engineering, and a massive increase in the number of people connected to the internet, we have around 15 billion connected IoT devices today, projected to grow to ~30 billion devices by 2030[^1]. As edge devices become more powerful, they are increasingly capable of running complex software, including operating systems and databases. The key point is: workloads that were thought to be too expensive to run in-process, are becoming more and more possible, even for surprisingly large amounts of data. The age of embedded databases is here!

**The idea of embedded databases is not new**. However, there have been numerous research breakthroughs in database theory in the last decade, which have made it possible to build embedded databases that are fast, lightweight, and easy to use.

When it comes to OLAP (online analytical processing) workloads, three particular databases stand out in terms of their innovations in this space: DuckDB (relational), Kùzu (graph) and LanceDB (vector). In this, and upcoming series of posts, we will take a deeper look at each of these databases, and explore their unique features and capabilities and why they're making working with complex data such a joy!

{{<figure src="embedded-db-classes.png">}}


## What are embedded databases?

Embedded databases are in-process database management systems that are tightly integrated with the application layer. They are often referred to as serverless databases, because they do not follow a client/server architecture (as is prevalent in current large-scale software systems). Another characteristic of embedded databases is that they can perform computations in-memory, but they also efficiently spill data and compute to on-disk storage for larger-than-memory datasets, allowing them to scale to all sizes of data.

> 💡 The term "serverless" can mean different things depending on the context, but it seems apt to refer to embedded database architectures using this term, because there's no server!

It's this combination of features that allows embedded databases to be a powerful option for OLAP query workloads, i.e., when a user wants to run numerically-intensive analytical queries on the data.

## DuckDB

If you've worked with databases at some capacity, among the three databases we're looking at, DuckDB is probably the one you're most likely to have heard of. Why is it such a big deal?

DuckDB is a high-performance embedded relational database system (RDBMS) that can be queried via a rich SQL dialect. Its core is written in C++, and is designed to be fast, reliable and easy to use. It uses columnar storage, and is designed to support OLAP query workloads, which are typically characterized by complex, relatively long-running computations that process significant portions of the stored dataset -- for example, aggregations over entire tables, or joins between several large tables.

### Using DuckDB from Python

DuckDB has bindings for several programming languages, including Python, R, C++, Java, and Go. In this post, we'll look at how to use DuckDB from Python. Embedded databases are a joy to install and use, because they don't require any external dependencies. To install DuckDB, simply run:

```sh
pip install duckdb
```

Let's read in some data from a reasonably large file to get started.



[^1]: IoT connected devices worldwide, [Statista](https://www.statista.com/statistics/1183457/iot-connected-devices-worldwide/)


Summary
(https://tdwi.org/articles/2019/12/09/dwt-all-databases-in-devices.aspx#:~:text=Embedded%20databases%20are%20built%20into,servers%20accessed%20by%20client%20applications.)