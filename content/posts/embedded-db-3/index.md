+++ 
draft = true
date = 2023-11-19
title = "Embedded databases (3): LanceDB and the modular data stack"
description = "A case study of LanceDB, an embedded vector DB for full-text, SQL and semantic search"
tags = ["embedded-db", "vector-db"]
categories = ["databases"]
+++

## An evolving hardware landscape

In September 2023, Wes McKinney, creator of the Pandas library, published an illuminating [blog post](https://wesmckinney.com/blog/looking-back-15-years/) describing his thoughts on 15 years of evolution in data management systems. In particular, he highlights the impact of modern hardware in pushing the industry towards a more modular, composable data stack. It's no wonder, then, that his friend, colleague and co-author of Pandas, [Chang She](https://github.com/changhiskhan) happens to be the founder of LanceDB, a developer-friendly embedded vector database that aligns very well with this vision, and is the focus of the third post in [this series](https://thedataquarry.com/tags/embedded-db/).

LanceDB is an open source, embedded and developer-friendly vector database. Some of its key features (among many others) that make it different from a number of existing solutions are listed below:

* Incredibly lightweight (no DB servers to manage), because it is entirely in-process
* Extremely scalable from development to production
* Ability to perform full-text search (FTS), SQL search and vector search
* Multi-modal data support (images, text, video, audio, point-clouds, etc.)
* Zero-copy (via Arrow) with automatic versioning of data

The aim of this post is to understand the internals of LanceDB, and to showcase its performance on full-text and vector search via serial and concurrent workflows.

### Towards "deconstructed databases"

The term *deconstructed database*[^1] was first coined in 2019 by Julien Le Dem, one of the original designers of the Parquet file format and Arrow specification. These database systems deviate from the well-known vertically-integrated systems that have dominated the DB landscape for decades. Instead, they are built from a collection of modular, reusable components, each of which can be developed and optimized by entirely separate groups of people, typically as open source projects.

In his blog[^2], Wes points out that today's compute hardware is radically different from what it was in 2013, when Parquet was conceived. In particular, the availability of blazing fast SSDs and NVMe drives have led to a shift in thinking toward designing systems that can store and query specialized data types like vectors and time series at scale.

## The composition of LanceDB

With these ideas in mind, we can attempt to deconstruct LanceDB. On the *surface*, it is a vector database written in Rust, but *underneath*, it's a collection of specialized modular components which are themselves independent components of the Rust 🦀 tooling ecosystem.

{{<figure src="lancedb-components.png" caption="The modular components of LanceDB">}}

LanceDB implements its own vector indexes on top of the underlying [Lance](https://lancedb.github.io/lance/) data format, using either an IVF-PQ or an upcoming DiskANN index. Tantivy is an open source full-text search engine that is incorporated to allow keyword-based search via BM25. DataFusion, an embeddable SQL query engine, is used to power the full-text/vector search queries via a SQL interface. The Apache arrow format is used to allow a smooth transition between in-memory and on-disk data storage, and also for smooth interoperability with other data formats like DuckDB, Pandas or Polars. The characteristics of these components is described in more detail below.

### Lance

[Lance](https://github.com/lancedb/lance) is the persistent storage format used in LanceDB. It is optimized for machine learning datasets and workflows, and can be thought of as an alternative to Parquet that's highly optimized for ultra fast scans and random access on vectors. In addition, it supports multi-modal data (images, text, video, audio, point-clouds, etc.) in a zero-copy manner, thanks to its integration with the Arrow ecosystem.

### Arrow

It's impossible to overstate the impact that Apache Arrow[^2] has had on the consolidation of analytical tooling in the industry, due to the way it connects disparate portions of the data stack via a language-agnostic, columnar, in-memory design. The following key features of Arrow[^2] are relevant:

* Column-oriented in-memory operation optimized for fast analytical processing
* [Zero-copy](https://en.wikipedia.org/wiki/Zero-copy), chunk-oriented data layer designed for moving and accessing large amounts of data from disparate storage layers
* Extensible type metadata for describing a wide variety of flat and nested data types occurring in real-world systems, with support for user-defined types

Lance is based on the Rust implementation of Arrow, [`arrow-rs`](https://github.com/apache/arrow-rs), and was itself rewritten from the ground up in Rust in 2023[^3].

### DataFusion

[DataFusion](https://github.com/apache/arrow-datafusion) is a fast, extensible embedded query engine that supports a SQL API, and can be used to query data stored in Arrow and Lance. In recent times, it has become the standard for building domain-specific query engines that are decoupled from the storage layer, and because it's written in Rust, performance is excellent despite not being natively integrated with the storage layer. Due to its extensibility, DataFusion has been re-purposed in LanceDB for full-text/vector search via a unified SQL interface.

### Tantivy

[Tantivy](https://github.com/quickwit-oss/tantivy) a full-text search engine library written in Rust. It's very similar to Apache Lucene (which is written in Java), but Tantivy is faster and more memory efficient. Just like Lucene, Tantivy is not intended to be user-facing, but rather, is used as the basis of a downstream system such as LanceDB. Tantivy uses the BM25 algorithm for scoring documents, and supports boolean queries, phrase queries, fuzzy queries, and range queries. Incorporating Tantivy into LanceDB allows for full-text search queries that run directly on a Lance dataset.

{{<notice info>}}
Based on the components listed above, we can see how LanceDB is a byproduct of multiple modular systems that come together to create a powerful vector database system with in-built data versioning and fast performance, while exposing interfaces for full-text, SQL and semantic search queries.
{{</notice>}}

## Embedded vs. serverless

It is common to see the terms "embedded" and "serverless" used interchangeably, but for a database, they are not the same architecturally. LanceDB is available to users in two flavours:

* Open source LanceDB is *embedded*, meaning that the storage layer and the application layer are tightly coupled, and are typically on the same instance/machine
* LanceDB Cloud (a commercial product) is *serverless*, meaning that storage and compute are clearly separated. This is done by connecting the application client to a remote database via a network  connection, with the application layer and storage layer being scaled independently of one another.

## Benchmark: Full-text and vector search

Studying the performance of any tool in isolation doesn't make much sense, so for the sake of comparison, an Elasticsearch workflow is provided in this repo. Elasticsearch is a popular full-text and vector search engine based on Lucene (which Tantivy is inspired by), so this makes it a meaningful comparison for a benchmark.

{{< notice info >}}
🚩The goal in this section is to provide the framework for a preliminary benchmark comparing the performance of LanceDB and Elasticsearch via their Python clients, as they stand today. It is expected that the numbers shown below for LanceDB will continually improve as new versions are released, as each underlying system improves.

As always, any benchmark should be performed on your own data and hardware to get a sense for your use case.
{{< /notice >}}

### Dataset

The dataset used is the same one in prior blogs, i.e., a wine reviews dataset from Kaggle. It consists of 129,971 wine reviews from the Wine Enthusiast magazine, made available in newline-delimited JSON as shown below. Refer to the [Kaggle source](https://www.kaggle.com/datasets/zynicide/wine-reviews) for more detailed information on the dataset and how it was scraped.

An example JSON line containing a wine review and its metadata is shown below.

```json
{
    "id": 40825,
    "points": "90",
    "title": "Castello San Donato in Perano 2009 Riserva  (Chianti Classico)",
    "description": "Made from a blend of 85% Sangiovese and 15% Merlot, this ripe wine delivers soft plum, black currants, clove and cracked pepper sensations accented with coffee and espresso notes. A backbone of firm tannins give structure. Drink now through 2019.",
    "taster_name": "Kerin O'Keefe",
    "taster_twitter_handle": "@kerinokeefe",
    "price": "30.0",
    "designation": "Riserva",
    "variety": "Red Blend",
    "region_1": "null",
    "region_2": null,
    "province": "Tuscany",
    "country": "Italy",
    "winery": "Castello San Donato in Perano"
}
```

### Full-text search (FTS) queries

Tantivy supports Lucene query syntax, in which the `+` symbol represents the `AND` boolean operator. The following ten keyword-based queries are used to search the `description` field of the dataset. For example, the first query below searches for documents that contain both the words `apple` and `pear` in the `description` field.

```txt
+apple +pear
"tropical fruit"
+citrus +almond
+orange +grapefruit
+full +bodied
"citrus acidity"
+blueberry +mint
+beef +lamb
+shellfish +seafood
+vegetable +fish
```

### Vector search queries

Ten vector search queries are defined as shown below. The first query below searches for documents that are similar to the vector representation of the words 'vanilla' and 'smoky'. The second query searches for documents whose vector representations are similar to 'dessert' and 'sweetness'. You get the idea!

```txt
vanilla and a hint of smokiness
rich and sweet dessert wine with balanced tartness
cherry and plum aromas
right balance of citrus acidity
grassy aroma with apple and tropical fruit
bitter with a dry aftertaste
sweet with a hint of chocolate and berry flavor
acidic on the palate with oak aromas
balanced tannins and dry and fruity composition
peppery undertones that pairs with steak or barbecued meat
```


### Serial benchmark

The serial benchmark involves sequentially running up to 10,000 randomly sampled queries in a Python for loop. This isn't representative of a realistic use case in production, but is useful to understand the throughput potential of the underlying search engines for either system.

We first randomly sample the FTS and vector search queries, with repetition, to generate a list of queries to run sequentially.

```python
import random

# Generate a list of 10,000 randomly sampled queries from the existing list of queries
random_choice_queries = [random.choice(queries) for _ in range(10_000)]
```

These are run in a for loop for the serial benchmark:

```python
# args.search is either "fts" or "vector"
for query in random_choice_queries:
    if args.search == "fts":
        _ = fts_search(tbl, query)
    else:
        _ = vector_search(MODEL, tbl, query)
```

For reference, the full code is [here](https://github.com/prrao87/lancedb-study/blob/main/lancedb/benchmark_serial.py).



### Concurrent benchmark


## Results


## The nonlinear impact of foundation systems

Why is building on top of modular components such a big deal? Each module improves independently of the other, and the impact of each propagates up the stack in a nonlinear fashion.


## Future developments in the Lance ecosystem


## Code

### Additional resources



[^1]: How Apache Arrow Is Changing the Big Data Ecosystem, [thenewstack.io](https://thenewstack.io/how-apache-arrow-is-changing-the-big-data-ecosystem/)
[^2]: The Road to Composable Data Systems: Thoughts on the Last 15 Years and the Future, [wesmckinney.com](https://wesmckinney.com/blog/looking-back-15-years/)
[^3]: Please pardon our appearance during renovations, by Chang She, [LanceDB blog](https://blog.lancedb.com/please-pardon-our-appearance-during-renovations-da8c8f49b383)
