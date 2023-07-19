+++ 
draft = true
date = 2023-07-18
title = "Vector databases (Part 3): A deep dive into vector indexes"
description = "Understanding Flat, Annoy, IVF, HNSW and Vamana vector indexes in vector databases"
tags = ["vector-db"]
categories = ["databases"]
math = true
+++

# Organizing vector indexes

This is my third post in a series on vector databases. [Part 1](../vector-db-1/) covered the differences between various popular vector DB vendors in the market, while [Part 2](../vector-db-2/) focused on the basics of what vector DBs are and what they do. If you've been following the developments in the world of vector DBs of late, you'll agree that it's actually quite hard to gain a bigger picture understanding of the landscape and the underlying trade-offs, unless you're willing to put in a *lot* of time and effort reading through a host of blog posts, papers and marketing material out there. My hope is that this series of blog posts helps you collate the vast amount of knowledge scattered across these sources and make an informed decision on which vector DB, and its underlying index, to use for your use case.

Assuming that it's amply clear to you [what a vector database *is*](../vector-db-2/#putting-it-all-together), it's worth taking a step back to wonder, how does it all scale so wonderfully to millions, billions, or even trillions of vectors[^1]? The primary aim of a vector database is to provide a fast and efficient means to store and *semantically* query data, in a way that the `Vector` data type is a first-class citizen. The similarity between two vectors is gauged by distance metrics like cosine distance or the dot product. When working with vector databases, it's important to distinguish between the *search algorithm*, and the underlying *index* on which the Approximate Nearest Neighbour (ANN) search algorithm operates.

As in most situations, choosing a vector index involves a tradeoff between accuracy (precision/recall) and throughput, and this is where understanding the various indexing algorithms out there helps you make a more informed decision. Having scoured the literature, I find that vector indexing methods can be organized in two levels: by their data structures, and by their level of compression.

## Level 1: Data structures

At the top level, vector indexes can be organized by their underlying data structures as follows.

{{< figure src="vector-db-indexing-types.png" >}}

### Hash-based index

Hash-based indexes like LSH (Locally Sensitive Hashing) transform higher dimensional data into lower-dimensional hash codes that aim to keep the original similarity as far as possible. The main focus of this indexing method is to design hash functions that generate better hash codes, which can then be looked up very efficiently to find similar neighbours. The main advantage of hash-based indexes is that they are very fast and scale to huge amounts of data, but the downside is that they are not very accurate.

### Tree-based index

Tree-based index structures allow for rapid searches in high-dimensional spaces, via binary search trees. The tree is constructed in a way that similar data points are more likely to end up in the same subtree, making it much faster to discover approximate nearest neighbours. **Annoy** (Approximate Nearest Neighbours Oh Yeah) was such a method developed at Spotify, and was designed specifically to handle music recommendations on their vast collection of songs. The downside of tree-based indexes is that they perform reasonably well only for low-dimensional data, and are not very accurate for high-dimensional data because they cannot adequately capture the complexity of the data.

### Graph-based index

Graph-based indexes like **HNSW** (Hierarchical Navigable Small World) are based on the idea that data points in vector space organized as a graph, where the nodes represent the data values, and edges connecting the nodes represent the similarity between the data points. The graph is constructed in a way that similar data points are more likely to be connected by edges, and the ANN search algorithm is designed to traverse the graph in an efficient manner. The main advantage of graph-based indexes is that they are very efficient in finding approximate nearest neighbours in diverse directions, and can exist almost entirely in memory, making them a great compromise between accuracy and speed.

One other index, NGT[^2] (Neighbourhood Graphs and Trees), developed by Yahoo! Japan Corporation, performs two constructions during indexing: one that transforms a dense kNN graph into a bidirectional graph, and another that incrementally constructs a navigable small world graph. The difference from HNSW is the use of range search, a variant of greedy search during graph construction. Because both of these methods have a high outward degree, to avoid combinatorial explosion, the seed vertex from which a search originates uses another range search on a tree-like structure (Vantage-point, or 'VP' trees) to make the search more efficient. This makes NGT a hybrid graph-tree index.

### Inverted file index

Inverted file index (IVF) is an indexing method that divides the vector space into a number of tessellated (Voronoi) cells, which reduces the search space in the same way that clustering does -- in order to find the nearest neighbours, the ANN algorithm must simply locate the centroid of the nearest Voronoi cell, and then search only within that cell. The benefit of IVF is that it helps design ANN algorithms that rapidly narrow down on the similarity region of interest, but the disadvantage of IVF in its raw form is that the quantization step involved in tessellating the vector space can be slow for very large amounts of data, which is why it is commonly combined with quantization methods like product quantization (PQ) to improve performance, described below.

## Level 2: Compression

The second level on which indexes are broken down is their compression level: a "flat" index is one that stores vectors in their unmodified form. When a query vector is received, it is exhaustively compared against each and every vector in the database, as shown in the simplified example below in 3-D vector space. In essence, using a true flat index would be like doing a kNN search, where the returned results are exact matches with the $k$ nearest neighbouring vectors. As you can imagine, the time required to return results using a flat index would increase linearly with the size of the data, making it impractical when used on more than a few hundred thousand vectors.

{{< figure src="vector-db-flat-index.png" caption="Flat index for kNN (exhaustive) search: Image inspired from [Pinecone blog](https://www.pinecone.io/learn/series/faiss/vector-indexes/)" >}}


The solution to improve search efficiency, at the cost of some accuracy in the retrieval, is compression. This process is called *quantization*, where the underlying vectors in the index are broken into chunks made up of fewer bytes (typically via converting floats to integers) to reduce memory consumption and computational cost during search.

{{< figure src="vector-db-indexing-compression.png" >}}

### Flat composite indexes

The default index on which an ANN search operates can be termed as "Flat". Unlike a flat index, an existing index like IVF or HNSW is called "flat" when it directly calculates the distance between the query vector and all the vectors in the database. To differentiate this from the quantized variants, the term "flat" is typically associated with these indexes, for example, IVF-Flat.

### Quantized composite indexes

In contrast, a quantized index is a form of composite index that combines an existing index (IVF, HNSW or Vamana) with compression methods like quantization to reduce the memory footprint of the index. The most popular quantization method is product quantization (PQ), which is a lossy compression technique[^3] that breaks up the original vector into smaller chunks called subvectors, and then stores the subvectors in a way that allows for fast reconstruction of the original vector, massively reducing memory requirements. PQ is not used its own, and instead relies on an existing index to first rapidly narrow down the search space, following which it can be applied to improve search speeds. Examples of these types of composite indexes include IVF-PQ and HNSW-PQ.

For those curious to go deeper, here's the original paper on product quantization for vector search: [*Product quantization for nearest neighbor search*](https://inria.hal.science/inria-00514462v2/document), Jégou & al., PAMI 2011


---
# Popular indexes

In this section, it makes sense to go through some popular vector indexes in use in existing databases today.

## IVF-PQ

## HNSW

## Vamana

---

# ANN toolkits


## FAISS

## Hnswlib

## DiskANN


# Takeaways: What makes a vector database?




[^1]: Trillion-scale similarity, [Milvus blog](https://milvus.io/blog/Milvus-Was-Built-for-Massive-Scale-Think-Trillion-Vector-Similarity-Search.md)
[^2]: A Comprehensive Survey and Experimental Comparison of Graph-Based Approximate Nearest Neighbor Search, [arxiv.org](https://arxiv.org/pdf/2101.12631.pdf)
[^3]: Research foundations of FAISS, [docs](https://github.com/facebookresearch/faiss/wiki#research-foundations-of-faiss)
