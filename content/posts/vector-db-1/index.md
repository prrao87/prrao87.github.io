+++ 
draft = false
date = 2023-06-28
title = "Vector databases (Part 1): What makes each one different?"
description = "What makes vector databases like Qdrant, Weaviate, Milvus, Vespa, Vald, Chroma, Pinecone and LanceDB different from one another"
tags = ["vector-db"]
categories = ["general", "databases"]
math = true
+++

# A gold rush in the database landscape

There's been a lot of marketing (and unfortunately, hype) related to vector databases in the first half of 2023, and if you're reading this, you're likely curious _why_ so many kinds exist and what makes them different from one another. On paper, vector databases all do the same thing (they enable a host of applications that require semantic search), so how does one begin to form an informed opinion on each of them? 🤔

In this post, I'll do my best to highlight the differences between the various vector databases out there, as visually as possible. I'll also highlight specific dimensions on which I'm performing the comparison, to offer a more holistic view.

## So many options! 🤯

Over the last several months, I've been researching different vector databases and their internals, as well as interacting with them via their Python APIs, and I've come across the following common issues:

1. Each database vendor upsells their own capabilities (naturally) while downselling their competitors', so it's easy to get a biased viewpoint, depending on where you look
2. There are **many** vector DB vendors out there, and it requires a lot of reading across multiple sources to connect the various dots and understand the landscape, and what underlying technologies exist
3. When it comes to vector search, there are **many** trade-offs to consider:
    * __Hybrid, or keyword search?__ Often, a hybrid of keyword + vector search yields the best results, and each vector database vendor, having realized this, offers their own hybrid search solutions
    * __On-premise, or cloud-native?__ A lot of vendors upsell "cloud-native" as though infrastructure is the world's biggest pain point, but on-premise deployments can be much more economical, and thus effective, over the long term
    * __Open source, or fully managed?__ Most vendors build on top of a source-available or open source code base that showcases their underlying methodology, and then monetize the deployment and infrastructure portions of the pipeline (through fully managed SaaS). It's still possible to self-host a lot of them, but that comes with additional manpower and in-house skill requirements.

To consolidate things and keep them a bit more organized, I've prepared a [running spreadsheet](https://docs.google.com/spreadsheets/d/1tUxif_MQYByprhFArZ2XqssvlMVfn5mKjsPjYTOO08o/edit?usp=sharing) of the different vector DB vendors to document key defining characteristics and milestones. There are likely many more that I've missed, so do let me know in the comments!

## Comparing the various vector databases

As of June 2023, I'm looking at 8 purpose-built vector database offerings, and 3 existing ones that add vector search as an extra feature.

### Location of headquarters and funding

Keeping aside the incumbents (i.e., established vendors) we can start by tracking funding milestones[^1] for each vector database startup.

Company         | Headquartered in | Funding
----------------|------------------|-----------------
Weaviate        |  🇳🇱 Amsterdam           |  $68M Series B
Qdrant          |  🇩🇪 Berlin              |  $11M Seed
Pinecone        |  🇺🇸 San Francisco       |  $138M Series B
Milvus/Zilliz   |  🇨🇳 / 🇺🇸 San Francisco  |  $113M Series B
Chroma          |  🇺🇸 San Francisco       |  $20M Seed
LanceDB         |  🇺🇸 San Francisco       |  Venture
Vespa           |  🇺🇸 Redwood City        |  Yahoo
Vald            |  🇯🇵 Tokyo               |  Yahoo Japan

There's clearly a LOT of activity in the Bay Area, California when it comes to vector databases! Also, there's a large amount of variation in the funding and valuation, and it's clear that there exists no correlation between the capabilities of a database and the amount it's funded for.

### Choice of programming language

Fast, responsive and scalable databases are typically written these days in modern languages like Golang or Rust. Among the purpose-built vendors, the only one that is built in Java is Vespa. Chroma is currently a Python/TypeScript wrapper on top of Clickhouse, an OLAP database built in C++, and an open source vector index, [HNSWLib](https://github.com/nmslib/hnswlib).

{{< figure src="vector-db-lang.png" >}}


Interestingly, both Pinecone[^2] and Lance[^3], the underlying storage format for LanceDB, **were rewritten from the ground up** in Rust, even though they were originally written in C++. Clearly, the database community is slowly embracing Rust 🦀💪!

### Timeline

How long has each vector database been around?

{{< figure src="vector-db-timeline.png" >}}

Vespa was among the earliest vendors to incorporate vector similarity search alongside the then-dominant BM25 keyword-based search algorithm. Weaviate soon followed with an open-source, dedicated vector search database offering in the end of 2018, and by 2019, we began to see more competition in the space with Milvus (also open-source) being released. Note that Zilliz, shown in the timeline, is not listed in the primary list because it is a commercial cloud-based offering built on top of Milvus. In 2021, three new vendors entered the foray: Vald, Qdrant and Pinecone. Incumbents like Elasticsearch, Redis & PostgreSQL were conspicuously absent until this point and began offering vector search much later than one might think -- only in 2022 and beyond.

### Source code availability

Among all the options listed, **only one** is completely closed-source: Pinecone. Zilliz is a closed-source fully-managed commercial solution too, but it's built entirely on top of Milvus and can be viewed as Milvus's parent entity. All the other options are at least source-available in terms of the code base, with the exact license determining the permissibility of the code and how it's deployed.

{{< figure src="vector-db-source-available.png" >}}

{{< notice tip >}}
As a developer, I've found that tracking issues, PRs and releases in the open source GitHub repos for each database in question provides a pretty decent idea of what's being prioritized and addressed in the roadmap. Also, the number of GitHub stars ⭐️ is a good indicator of community interest in the project.

> My developer mind is always uncomfortable with the opaque nature of closed-source products, and as such, I always prefer working with source-available products for the above reasons.
{{< /notice >}}

### Hosting methods

The typical hosting methods provided by database vendors include self-hosted (on-premise) and managed/cloud-native, both of which follow the client-server architecture. The third, more recent option, is _embedded_ mode, where the database itself is tightly coupled with the application code in a serverless manner. Currently, only Chroma and LanceDB are available as embedded databases.

{{< figure src="vector-db-hosting.png" >}}

The circled portion in the diagram is particularly interesting for two reasons:

* Chroma is attempting to build an all-in-one offering that has embedded mode (default), a managed cloud-native server[^4] that follows a client-server architecture, and a cloud-hosted distributed system that enables serverless computing on the cloud
* LanceDB, the youngest vector database out there at the time of writing, has an ambitious goal of offering multimodal AI database in embedded mode (default), with a fully managed cloud offering[^5] that offers a distributed serverless computing environment, very similar to Chroma, but fully based on the [Lance data format](https://lancedb.github.io/lance/), a modern, columnar format for ML.

### Indexing methods

Most vendors use a hybrid vector search methodology, that combines keyword and vector search in various ways. However, the underlying vector index used by each database can differ quite significantly.

{{< figure src="vector-db-indexes.png" >}}

The vast majority of database vendors opt for their custom implementation of HNSW (Hierarchical Navigable Small-World graphs). Persisting the vector index to disk is fast becoming an important objective, so as to handle larger-than-memory datasets. The 2019 NeurIPS paper on DiskANN[^6] showed that it has the potential to be the fastest on-disk indexing algorithm for searching on billions of data points, and so, databases like Milvus, Weaviate and LanceDB, that have already implemented their own versions of DiskANN (or are actively doing so), are ones to watch out for in this area.

## Key features

In this section, I'll list some of the key features and the pros and cons of each database. A lot of this is my informed opinion, obtained through various means (blogs, podcasts, research papers, conversations with users, and my own code).

### Pinecone

* __Pros__: Very easy to get up and running (no hosting burden, it's fully cloud-native), and doesn't expect the user to know anything about vectorization or vector indexes. As per their documentation (which is also quite good), it just _works_.
* __Cons__: Fully proprietary, and it's impossible to know what goes on under the hood and what's on their roadmap without being able to follow their progress on GitHub. Also, [certain users' experience](https://twitter.com/garywupx/status/1631238355634495488) showcase the danger of relying on a fully external, third-party hosting service and the complete lack of control from the developer's standpoint on how the database is set up and run. The cost implications in the long run also can be significant, especially with the sheer number of self-hosted alternatives out there.

* __My take__: In 2020-21, when vector databases were very much under the radar, Pinecone was much ahead of the curve and offered convenience features to developers in a way that other vendors didn't. Fast forward to 2023, and frankly, there's little that Pinecone offers now that other vendors don't, and most of the other vendors at least offer a self-hosted, managed or embedded mode, not to mention that the source code for their algorithms and their underlying technology is transparent to the end user.

### Weaviate

* __Pros__: _Amazing_ [documentation](https://weaviate.io/developers/weaviate) (one of the best out there, including technical details and ongoing experiments). Weaviate really seems to be focused on building the best developer experience possible, and it's very easy to get up and running via Docker. In terms of querying, it produces fast, sub-millisecond search results, while offering both keyword and vector search functionality.
* __Cons__: Because Weaviate built in Golang, scalability is achieved through Kubernetes, and this approach (in a similar way to Milvus) is known require a fair amount of infrastructure resources when the data gets really large. The cost implications for Weaviate's fully-managed offering over the long term are unknown, and it may make sense to compare its performance with other Rust-based alternatives like Qdrant and LanceDB (though time will tell which approach scales better in the most cost-effective manner).
* __My take__: Weaviate has a great user community, and the development team are actively showcasing extreme scalability (hundreds of billions of vectors), so it does seem that their target market is large-scale enterprises that have mountains of data on which they aim to do vector search. Offering keyword search alongside vector search, and a powerful hybrid search in general, allow it to generalize to a variety of use cases, directly competing with document databases like Elasticsearch. Another interesting area Weaviate is actively focusing on involves [data science and machine learning](https://www.youtube.com/watch?v=IRWHa57T-zk) via vector databases, taking it outside the realm of traditional search & retrieval applications.

### Qdrant

* __Pros__: Although newer than Weaviate, Qdrant also has great documentation that helps developers get up and running via Docker with ease. Built entirely in Rust, it offers APIs that developers can tap into via its Rust, Python and Golang clients, which are the most popular languages for backend devs these days. Due to the underlying power of Rust, its resource utilization seems lower than alternatives built in Golang (at least in my experience). Scalability is currently achieved through partitioning and the Raft consensus protocol, which are standard practices in the database space.
* __Cons__: Being a relatively newer tool than its competitors, Qdrant has been playing catch-up with alternatives like Weaviate and Milvus, in areas like user interfaces for querying, though [that gap is rapidly reducing](https://qdrant.tech/articles/qdrant-1.3.x/#qdrant-web-user-interface) with each new release.
* __My take__: I think Qdrant stands poised to become the go-to, first-choice vector search backend for a lot of companies that want to minimize infrastructure costs and leverage the power of a modern programming language, Rust. At the time of writing, hybrid search is not yet available, but as per their roadmap, it's being actively worked on. Also, Qdrant is continually publishing updates on how they are optimizing their HNSW implementation, both in-memory and on-disk, which will greatly help with its search accuracy & scalability goals over the long term. It's clear that Qdrant's user community is rapidly growing (interestingly, faster than Weaviate's), as per [its GitHub star history!](https://star-history.com/#weaviate/weaviate&qdrant/qdrant&Date). Perhaps the world is all hyped-up about Rust 🦀? Either way, it's a LOT of fun building on top of Qdrant, at least in in my view 😀.

### Milvus/Zilliz

* __Pros__: Very mature database with a host of algorithms, due to its lengthy presence in the vector DB ecosystem. Offers [a **lot** of options](https://milvus.io/docs/index.md) for vector indexing and built from the ground up in Golang to be extremely scalable. As of 2023, it's the only major vendor to offer a [working DiskANN implementation](https://milvus.io/blog/2021-09-24-diskann.md), which is said to be the most efficient on-disk vector index.
* __Cons__: To my eyes, Milvus seems to be a solution that throws the kitchen sink _and_ the refrigerator at the scalability problem -- it achieves a high degree of scalability through a combination of proxies, load balancers, message brokers, Kafka and Kubernetes, which makes the overall system really complex and resource-intensive. The client APIs (e.g., Python) are also not as readable or intuitive as newer databases like Weaviate and Qdrant, who tend to focus a lot more on developer experience.
* __My take__: It's clear that Milvus was built with the idea of massive scalability for streaming data to a vector index, and in many cases, when the size of the data isn't too large, Milvus can seem to be overkill. For more static and infrequent large-scale situations, alternatives like Qdrant or Weaviate may be cheaper and faster to get up and running in production.

### Chroma

* __Pros__: Offers a convenient JavaScript API for front end developers to quickly get a vector store up and running. it was the first vector database in the market to offer embedded mode by default, where the database itself and application layers are tightly integrated.
* __Cons__: Unlike the other purpose-built vendors, Chroma is largely a Python/TypeScript wrapper around an existing OLAP database (Clickhouse), and an existing open source vector search implementation ([hnswlib](https://github.com/nmslib/hnswlib)). A server implementation is in progress, and a lot regarding their road map is fluid and subject to change, depending on the state of the competitive market.
* __My take__: The vector DB market is rapidly evolving, and Chroma seems to be inclined[^7] to adopt a "wait and watch" philosophy, in terms of favouring a serverless/embedded model vs. a full-fledged, cloud-native SaaS solution based on a client-server architecture. Based on their JavaScript-friendly API and the fact that they're still working on their own underlying storage layer while leveraging existing databases and frameworks in the interim, the key market focus seems to be front-end developers who can put LLM and vector search directly into the hands of non-technical users. From a long term perspective, we've never yet seen an embedded database architecture be successfully commercialized in the vector search space, so its evolution (alongside LanceDB, described below) will be interesting to watch!!


### LanceDB

* __Pros__: Is designed to perform distributed indexing and search natively on multi-modal data (images, audio, text), built on top of the [Lance data format](https://lancedb.github.io/lance/), an innovative and new columnar data format for ML. Just like Chroma, LanceDB uses an embedded, serverless architecture, and is built from the ground up in Rust, so along with Qdrant, this is the only other major vector database vendor to leverage the speed 🔥, memory safety and relatively low resource utilization of Rust 🦀.
* __Cons__: LanceDB is a **very** young database, so a lot of features are under active development, and prioritization of features will be a challenge in the coming year or so due to a small engineering team.
* __My take__: I think among all the vector databases out there, LanceDB differentiates itself the most from the rest. This is mainly because it innovates on the data storage layer (using Lance, a new, faster columnar format than parquet, that's designed for very efficient lookups), and on the infrastructure layer -- by using a serverless architecture. As a result, a lot of infrastructure complexity is reduced, greatly increasing the developer's freedom and ability to build semantic search applications directly connected to data lakes in a distributed manner.

### Vespa

* __Pros__: Provides the most enterprise-ready hybrid search capabilities, combining the tried-and-tested power of keyword search and a custom vector search on top of HNSW. Although other vendors like Weaviate also provide keyword and vector search, Vespa was among the first to market with this offering, which has given them ample time to optimize their offering.
* __Cons__: The developer experience in Java-based databases (simply due to the bulky nature of Java packages), including setup and teardown, is nowhere near as smooth as in more modern alternatives that are written in performance-oriented languages like Go or Rust.
* __My take__: I believe the world is slowly but surely moving away from Java-based databases. Most new databases built these days are written in languages like Go or Rust, and those options could be the better choice for leaner, younger teams in terms of resource utilization, memory consumption and scalability.

### Vald

* __Pros__: Designed to handle multi-modal data storage via a highly distributed architecture, along with useful features like index backup. Uses a very fast ANN search algorithm, NGT (Neighbourhood Graph & Tree), which is among the fastest ANN algorithms when used in conjunction with a highly distributed vector index.
* __Cons__: Seems to have much less overall traction and usage than other vendors, and the documentation doesn't clearly describe what vector index is used ("distributed index" is rather vague). It also seems entirely funded by one entity, Yahoo! Japan, with little information on any other major users.
* __My take__: I think Vald is a much more niche vendor than others, catering primarily to Yahoo! Japan's search requirements, and with a much smaller user community overall, at least on the basis of their GitHub stars. Part of this could be the fact that it's based in Japan, and not marketed as heavily as the other more well-placed vendors in the EU and the Bay Area.

### Elasticsearch, Redis and pgvector

* __Pros__: If you're already using an existing data store like Elasticsearch, Redis or PostgreSQL, it's pretty straightfoward to utilize their vector indexing and search offering without having to resort to a new technology.
* __Cons__: Existing databases do not necessarily store or index the data in the most optimal fashion, as they are designed to be general purpose, and as a result, for data involving million-scale vector search and beyond, performance suffers. Redis VSS (Vector Search Store) is fast primarily because it's purely in-memory, but the moment the data becomes larger than memory, it becomes necessary to think of alternative solutions.
* __My take__: I think purpose-built and specialized vector databases will slowly out-compete established databases in areas that require semantic search, mainly because they are innovating on the most critical component when it comes to vector search -- the storage layer. Indexing methods like HNSW and ANN algorithms are well-documented in the literature and most database vendors can roll out their own implementations, but purpose-built vector databases have the benefit of being optimized to the task at hand (not to mention that they're written in modern programming languages like Go and Rust), and for reasons of scalability and performance, will most likely win out in this space in the long run.

## Conclusions: The trillion-scale problem

It's hard to imagine any other time in history when any one kind of database has captured this much of the public's attention, not to mention the VC ecosystem. One key use case that vector database vendors (like Milvus[^8], Weaviate[^9]) are trying to solve is how to achieve trillion-scale vector search with the lowest latency possible. This is an incredibly difficult task, and with the amount of data coming via streams or batch processing these days, it makes sense that purpose-built vector databases that optimize for storage and querying performance under the hood are the most primed to break this barrier in the near future.

Whew, this was a lot to digest in one post! In the next posts, I'll summarize the underlying search and indexing algorithms in vector databases and go a bit deeper into the technical details.

[^1]: Source of funding numbers: [objectbox.io](https://objectbox.io/vector-database/)
[^2]: Rewriting a high performance vector database in Rust, [Pinecone blog](https://www.pinecone.io/learn/rust-rewrite/)
[^3]: Rewriting Lance in Rust, [LanceDB blog](https://blog.lancedb.com/please-pardon-our-appearance-during-renovations-da8c8f49b383)
[^4]: Chroma [Roadmap](https://docs.trychroma.com/roadmap)
[^5]: LanceDB Cloud (serverless), [coming soon](https://lancedb.com/)
[^6]: DiskANN: Fast Accurate Billion-point Nearest
Neighbor Search on a Single Node, [NeurIPS 2019](https://suhasjs.github.io/files/diskann_neurips19.pdf)
[^7]: Chroma: Open source embedding database, [Software Engineering Daily Podcast](https://softwareengineeringdaily.com/2023/04/20/open-source-embedding-database/)
[^8]:Milvus Was Built for Massive-Scale (Think Trillion) Vector Similarity Search, [Milvus blog](https://milvus.io/blog/Milvus-Was-Built-for-Massive-Scale-Think-Trillion-Vector-Similarity-Search.md)
[^9]: ANN algorithms: Vamana vs. HNSW, [Weaviate blog](https://weaviate.io/blog/ann-algorithms-vamana-vs-hnsw#large-scale)