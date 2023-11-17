+++ 
draft = true
date = 2023-11-15
title = "Embedded databases (3): LanceDB and the evolution of the modular data stack"
description = "A case study of LanceDB, an embedded vector DB for full-text, SQL and semantic search"
tags = ["embedded-db", "vector-db"]
categories = ["databases"]
+++

## An evolving hardware landscape

In September 2023, Wes McKinney, creator of the Pandas library, published an illuminating [blog post](https://wesmckinney.com/blog/looking-back-15-years/) highlighting his thoughts on 15 years of evolution in the data management space. In particular, he highlights the impact of modern hardware in pushing the industry toward a more modular, composable data stack. It's no wonder, then, that his friend, colleague and co-author of Pandas, [Chang She](https://github.com/changhiskhan) happens to be the founder of LanceDB, a developer-friendly embedded vector database that aligns very well with this vision, and is the focus of this post.

LanceDB is an open source, embedded and developer-friendly vector database. Some key features about LanceDB that make it extremely valuable are listed below, among many others listed on their GitHub repo.

* Incredibly lightweight (no DB servers to manage), because it is entirely in-process
* Extremely scalable from development to production
* Ability to perform full-text search, SQL search and vector search
* Multi-modal data support (images, text, video, audio, point-clouds, etc.)
* Zero-copy (via Arrow) with automatic versioning of data

The aim of this post is to demonstrate the full-text and vector search performance of LanceDB via serial and concurrent benchmarks, and to highlight the underlying components that make it possible.

### Towards "deconstructed databases"

The term *deconstructed database*[^1] was first coined in 2019 by Julien Le Dem, one of the original designers of the Parquet file format and Arrow specification. These database systems deviate from the well-known vertically-integrated systems that have dominated the DB landscape for decades. Instead, they are built from a collection of modular, reusable components, each of which can be developed and optimized by entirely separate groups of people, typically as open source projects.

In his blog[^2], Wes points out that today's compute hardware is radically different from what it was in 2013, when Parquet was conceived. In particular, the availability of blazing fast SSDs and NVMe drives have led to a shift in thinking toward designing systems that can store and query specialized data types like vectors and time series at scale.

## The composition of LanceDB

With these ideas in mind, we can attempt to deconstruct LanceDB. On the *surface*, it is a vector database written in Rust, but *underneath*, it's a collection of specialized modular components which are themselves independent components of the Rust 🦀 tooling ecosystem.

{{<figure src="lancedb-deconstructed.png" caption="The modular makeup of LanceDB">}}

The database is composed of the storage layer, the indexing layer (for full-text and vector indexes), and the query engine. LanceDB implements its own vector indexes on top of the underlying Lance data format, using either IVF-PQ or an upcoming DiskANN index. DataFusion, an embeddable query engine, is used to power the full-text/vector search queries via a SQL interface. Each of these components is described in more detail below.

### Lance

[Lance](https://github.com/lancedb/lance) is the persistent storage format used in LanceDB. It is optimized for machine learning datasets and workflows, and can be thought of as an alternative to Parquet that's highly optimized for ultra fast scans and random access on vectors. In addition, it supports multi-modal data (images, text, video, audio, point-clouds, etc.) in a zero-copy manner, thanks to its integration with the Arrow ecosystem.

### Arrow

It's impossible to overstate the impact that Apache Arrow[^2] has had on the consolidation of tooling in the analytical space, due to the way it connects disparate portions of the data stack via a unified in-memory interface. The following key features of Arrow[^2] are relevant:

* Column-oriented in-memory operation optimized for fast analytical processing performance
* Zero-copy, chunk-oriented data layer designed for moving and accessing large amounts of data from disparate storage layers
* Extensible type metadata for describing a wide variety of flat and nested data types occurring in real-world systems, with support for user-defined types

Lance is based on the Rust implementation of Arrow, [`arrow-rs`](https://github.com/apache/arrow-rs), and was itself rewritten from the ground up in Rust in 2023[^3].

### DataFusion

[DataFusion](https://github.com/apache/arrow-datafusion) is a fast, extensible embedded query engine that supports a SQL API, and can be used to query data stored in Arrow and Lance. In recent times, it has become the standard for building domain-specific query engines that are decoupled from the storage layer, and because it's written in Rust, performance is excellent despite not being natively integrated with the storage layer. Due of its extensibility, DataFusion is customized to power full-text/vector search via a unified SQL interface in LanceDB.

### Tantivy

[Tantivy](https://github.com/quickwit-oss/tantivy) a full-text search engine library written in Rust. It's very similar to Apache Lucene (which is written in Java), but Tantivy is faster and more memory efficient. Just like Lucene, Tantivy is not intended to be used on its own, but rather, as a component of a larger system such as LanceDB. Tantivy uses the BM25 algorithm for scoring documents, and supports boolean queries, phrase queries, fuzzy queries, and range queries. Incorporating Tantivy into LanceDB allows for full-text search queries to be performed on the data stored in Lance.

{{<notice note>}}
Based on the components listed above, we can see how LanceDB is a byproduct of multiple modular systems that come together to create a powerful vector database with data versioning and performance baked in, all while exposing  interfaces for full-text, SQL and semantic search queries.
{{</notice>}}

## Embedded vs. serverless

LanceDB is available in two forms, making it important to distinguish between them as it pertains to the database architecture:

* OSS LanceDB is *embedded*, meaning that the storage layer and the application layer are tightly coupled, and are typically on the same instance/machine
* LanceDB Cloud is *serverless*, meaning that the storage layer and the application layer are distinct from one another, and the application layer is scaled independently of the storage layer that may lie on a completely different instance.

## Benchmark: Full-text and vector search

With all this talk about performance and ease-of-use, it makes sense to describe how to run a benchmark to gauge it on your own data. In this example 

### Comparing LanceDB and existing options


### Serial benchmark


### Concurrent benchmark


## Discussion on performance


## Thoughts on the larger impact of modular data systems

Why is building on top of modular components such a big deal? Each module improves independently of the other, and the impact of each propagates up the stack in a nonlinear fashion.


## Code

### Additional resources



[^1]: How Apache Arrow Is Changing the Big Data Ecosystem, [thenewstack.io](https://thenewstack.io/how-apache-arrow-is-changing-the-big-data-ecosystem/)
[^2]: The Road to Composable Data Systems: Thoughts on the Last 15 Years and the Future, [wesmckinney.com](https://wesmckinney.com/blog/looking-back-15-years/)
[^3]: Please pardon our appearance during renovations, by Chang She, [LanceDB blog](https://blog.lancedb.com/please-pardon-our-appearance-during-renovations-da8c8f49b383)
