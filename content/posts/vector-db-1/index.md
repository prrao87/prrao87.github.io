+++ 
draft = true
date = 2023-06-24
title = "Vector databases (Part 1): A visual comparison"
description = "What makes vector databases like Qdrant, Weaviate, Milvus, Vespa, Chroma, Pinecone and LanceDB different from one another"
tags = ["vector-db"]
categories = ["general", "databases"]
math = true
+++

# A gold rush in the database landscape

There's been a lot of marketing (and unfortunately, hype) related to vector databases in the first half of 2023, and if you're reading this, you're likely curious _why_ so many kinds exist and what makes them different from one another. On paper, vector databases all do the same thing (they perform semantic search), so how does one begin to form an informed opinion on each of them? 🤔

In this post, I'll do my best to highlight the differences between the various vector databases out there, as visually as possible. I'll also highlight specific dimensions on which I'm performing the comparison, to offer the most holistic view I can.

## So many options! 🤯

I've been researching the different vector database options for the last several months, and I find the following issues:

1. Each database vendor upsells their own capabilities while downselling their competitors, so it's easy to get a biased view, depending on the source you get the information from
1. There are **many** vector DB vendors out there, and it requires a lot of reading across multiple sources to connect the various dots and understand the landscape, and what underlying technologies exist
1. When it comes to vector search, there are **many** trade-offs to consider:
  * Hybrid, or keyword search?: Often, a hybrid search method (keyword + vector) yields the best results, and each vector database vendor, having realized this, offers their own hybrid search solutions
  * On-premise, or cloud-native?: A lot of vendors upsell "cloud-native" as though infrastructure is the world's biggest pain point, but on-premise deployments can be much more economical over the long term
1. Each vendor monetizes their technology stack very differently, so it helps to know the above trade-offs

To consolidate things and keep them a bit more organized, I've prepared a [running spreadsheet](https://docs.google.com/spreadsheets/d/1tUxif_MQYByprhFArZ2XqssvlMVfn5mKjsPjYTOO08o/edit?usp=sharing) of the different vendors to document key defining characteristics and milestones for each vendor, and I now have **eight** purpose-built vector databases and **three** incumbent (established) vendors in my list. There are likely many more that I've missed, so do let me know in the comments!

## Comparing the various vector databases

As described in [the goals](#the-goal), we are comparing 8 purpose-built vector database offerings, and 3 existing ones that add vector search as an extra feature.

### Location of headquarters

Database vendor | Headquartered in
----------------|-----------------
Weaviate        |  🇳🇱 Amsterdam
Qdrant          |  🇩🇪 Berlin
Pinecone        |  🇺🇸 San Francisco
Milvus/Zilliz   |  🇨🇳 / 🇺🇸 San Francisco
Chroma          |  🇺🇸 San Francisco
LanceDB         |  🇺🇸 San Francisco
Vespa           |  🇺🇸 Redwood City
Vald            |  🇯🇵 Tokyo

There's clearly a LOT of activity in the Bay Area, California when it comes to vector databases!

### Timeline

How long has each vector database been around?

{{< figure src="vector-db-timeline.png" >}}

Vespa was among the earliest vendors to incorporate vector similarity search alongside the then-dominant BM25 keyword-based search algorithm. Weaviate soon followed with an open-source, dedicated vector search database offering in the end of 2018, and by 2019, we began to see more competition in the space with Milvus (also open-source) being released. Not that Zilliz, shown in the timeline, is not listed in the primary list because it is a commercial cloud-based offering built on top of Milvus. In 2021, three new vendors entered the foray: Vald, Qdrant and Pinecone. Incumbents like Elasticsearch, Redis & PostgreSQL were conspicuously absent until this point and began offering vector search much later than one might think -- only in 2022 and beyond.

### Source code availability

Among all the options listed, **only one** is completely closed-source: Pinecone. Although Zilliz is a commercial offering, it's built on top of Milvus (which is open source) by the same parent company. All the other options are at least source-available in terms of the code base, with the exact license determining the permissibility of commercial deployment of the database itself.

{{< figure src="vector-db-source-available.png" >}}

{{< notice tip >}}
Tracking issues, PRs and releases in the open source GitHub repos for each database in question provides a pretty decent idea of what's being prioritized and addressed in the roadmap. Also, the number of GitHub stars ⭐️ is a good indicator of community interest (though in some cases, hype).

> As a developer, I always prefer working with source-available databases for these reasons.
{{< /notice >}}

### Choice of programming language

Fast, responsive and scalable databases are typically written these days in modern languages like Golang or Rust. Among the purpose-built vendors, only Vespa is written in Java. Chroma is currently a Python/TypeScript wrapper on top of Clickhouse, an OLAP database built in C++, and an open source vector index, [HNSWLib](https://github.com/nmslib/hnswlib).

{{< figure src="vector-db-lang.png" >}}


Interestingly, both Pinecone[^2] and Lance[^3], the underlying storage format for LanceDB, **were rewritten from the ground up** in Rust, even though they were originally written in C++. Clearly, the database community is slowly but surely embracing Rust 🦀!

### Hosting methods

The typical hosting methods provided by database vendors include self-hosted (on-premise) and managed/cloud-native hosting options, which both follow the client-server architecture. The third, more recent option, is embedded mode, where the data storage is tightly coupled with the application code in a serverless manner. Currently, only Chroma and LanceDB provide this hosting mode.

{{< figure src="vector-db-hosting.png" >}}

The circled portion in the diagram is particularly interesting for two reasons:

* Chroma is attempting to build an all-in-one offering that has embedded mode (default), a managed cloud-native server[^4] that follows a client-server architecture, and a cloud-hosted distributed system that enables serverless computing on the cloud
* LanceDB, the youngest vector database out there at the time of writing, has an ambitious goal of offering multimodal AI database in embedded mode (default), with a fully managed cloud offering[^5] that offers a distributed serverless computing environment, very similar to Chroma, but fully based on the [Lance data format](https://lancedb.github.io/lance/), a modern, columnar format for ML under the hood.

### Indexing methods

Most vendors use a hybrid vector search methodology, that combines keyword and vector search in various ways. However, the underlying vector index used by each database can differ quite significantly.

{{< figure src="vector-db-indexes.png" >}}

The vast majority of database vendors opt for their custom implementation of the HNSW (Hierarchical Navigable Small-World graphs) index. Vald is the only vendor that provides a purely tree-based index, NGT. Persisting the indexes to disk is fast becoming an important, and numerous vendors in this list (Qdrant, Weaviate, Milvus and LanceDB) already provide the means to index larger-than-memory datasets using a variety of algorithms. The 2019 NeurIPS paper on DiskANN[^6] showed that it has the potential to be the fastest on-disk indexing algorithm for searching on billions of points, and so, databases like Milvus, Weaviate and LanceDB, that have already implemented their own versions of DiskANN (or are actively doing so), are ones to watch out for in this area.


## Choosing between the options


## Conclusions




[^1]: How the revolution of natural language processing is changing the way companies understand text, [TechCrunch](https://techcrunch.com/sponsor/nvidia/how-the-revolution-of-natural-language-processing-is-changing-the-way-companies-understand-text/)
[^2]: Rewriting a high performance vector database in Rust, [Pinecone blog](https://www.pinecone.io/learn/rust-rewrite/)
[^3]: Rewriting Lance in Rust, [LanceDB blog](https://blog.lancedb.com/please-pardon-our-appearance-during-renovations-da8c8f49b383)
[^4]: Chroma [Roadmap](https://docs.trychroma.com/roadmap)
[^5]: LanceDB Cloud (serverless), [coming soon](https://lancedb.com/)
[^6]: DiskANN: Fast Accurate Billion-point Nearest
Neighbor Search on a Single Node, [NeurIPS 2019](https://suhasjs.github.io/files/diskann_neurips19.pdf)