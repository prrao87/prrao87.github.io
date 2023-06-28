+++ 
draft = true
date = 2023-07-05
title = "Vector databases (Part 2): A brief primer on vector search"
description = "What are vector databases, embeddings and similarity in vector search?"
tags = ["vector-db"]
categories = ["general", "databases"]
math = true
+++

# Why are vector DBs so much in demand?

In this post, I'll do my best to describe what vector databases do under the hood, without going into too much technical detail (that's for another post!).

First of all, what can explain the sudden frenzy of activity and investment in this space?

### The age of LLMs

Large Language Models (LLMs) such as GPT-4 have taken the world by storm. In November 2022, an early demo of ChatGPT (which is OpenAI's interface to GPT-3.5+) was released, following which it quickly became the [fastest growing application in history](https://www.forbes.com/sites/cindygordon/2023/02/02/chatgpt-is-the-fastest-growing-ap-in-the-history-of-web-applications/) reaching a million users in just 5 days! My hypothesis is that the sheer hype around ChatGPT and what its capabilities were ties in closely with the flurry of VC funding that poured into the vector database ecosystem after its release.

One way to test this hypothesis is to take a look at the GitHub star history for seven open-source vector database systems.

{{< figure src="vector-db-stars.png" caption="Made with ❤️ by [star-history.com](https://star-history.com/) ">}}

After November 2022, we can see that a lot of vendors (Qdrant, Weaviate, Milvus, Chroma) saw interest in their GitHub repos spike. Some of the  reasons for this are explained below.

### The problem with relying on LLMs

It has become clear that, in production, one cannot rely on LLMs alone to provide data or facts to users. Putting away cost considerations for the moment, two reasons are listed below:

- An LLM cannot know facts that occurred after its training was completed
- An LLM often _hallucinates_, i.e., fabricates non-existent information, such as pointing users to URLs that don't exist

Vector databases help address both these problems, via their highly efficient underlying storage layer. Unlike traditional databases, vector DBs contain data structures that natively store data in vector form. As a result, we can now build applications with an LLM sitting on top of a vector storage layer that contains recent, up-to-date, factual data (well past the LLM's training date) to the end user.

Although vector databases existed well before the advent of LLMs, after the release of ChatGPT, the open-source community, as well as marketing teams at existing database vendors quickly realized their benefits in mainstream use cases like information retrieval and search. It is for this reason that I believe there's an absolute bonanza of VC funding and quite frankly, over-valuation going on today in the world of vector databases.

### What is a vector embedding?

At its core, the data being stored in a vector database is in the form of embeddings, which is essentially the contextual meaning of each and every word, image or number in the data represented by a list of floating point numbers. The dimension of the embedding depends on the number of data points $m$, and the embedding size of the vector $n$.

{{< figure src="vector-embedding.png" >}}

Intuitively, when we refer to an "embedding", we are talking about a compressed, low-dimensional representation of data that exists in higher dimensions (images or documents).

### How are embeddings generated?

The transformer revolution in NLP[^1] has provided practitioners ample means to generate these compressed representations, or embeddings.

- The most popular method is via the open source library `sentence-transformers`, available via [Hugging Face model hub](https://huggingface.co/models?library=sentence-transformers) or directly from [the source](https://www.sbert.net/)
- A less common and rather expensive way, is to use the [OpenAI embeddings API](https://platform.openai.com/docs/guides/embeddings)

It's important to keep in mind that the lower the dimensionality of the underlying vectors, the more compressed the representation, which can affect downstream task quality. The sentence transformers repo provides embedding models with a dimension $n$ in the range of 384, 512 and 768, and the models are completely free and open-source. OpenAI's embeddings, which require a paid API call to generate, can be considered higher quality due to a dimensionality of 1,536.

{{< notice note >}}
The choice of model used to generate embeddings is typically a trade-off between quality and cost. In most cases, the open-source embeddings from sentence transformers can be utilized as is for text that isn't too long (~300-400 words for text sequences).
{{< /notice >}}

### Storing the embeddings in vector databases

Because of the general-purpose nature of vectors in embedding space, vector databases are proving to be very useful in the realm of _similarity search_ for all forms of inputs (text, video/image, audio). This form of search considers semantics, where the input query sent by the user (typically in natural language) is translated into vector form, in the same embedding space as the data itself, so that results that are most similar to the input are returned. A visualization of this is shown below.

{{< figure src="embedding-pipeline.png">}}

### How is similarity computed?

The similarity of two vectors is computed using a distance metric, typically, via the dot product or cosine distance. Consider a simplified example where we vectorize the titles of wines in 2-D space, where the horizontal axis represents red wines and the vertical axis represents white wines. In this space, points that are closer together would represent wines that share similar words or concepts, and points that are further apart do not have that much in common. The cosine distance is defined as the cosine of the angle between the lines connecting the position of each wine in the embedding space to the origin.

$$cos \theta = \frac{a^T \cdot b}{|a| \cdot{ |b|}}$$

A visualization will make this more intuitive.

{{< figure src="wines-cosine.png">}}

On the left, the two wines (the Reserve White and the Toscana Red) have very little in common, both in terms of vocabulary and concepts, so they have a cosine distance approaching zero (they are orthogonal in the vector space). On the right, the two Zinfandels from Napa Valley have a lot more in common, so they have a much larger cosine similarity closer to 1. The limiting case here would be a cosine distance of 1; a sentence is always perfectly similar to itself.

Of course, in a real situation, the actual vector space is high dimensional (and have many more axes other than just the variety of wine) and cannot be visualized on a 2-D plane, but the same principle of cosine similarity applies.

### Scalable nearest neighbour search

Once the vector embeddings are generated and stored, when a user submits a search query, the goal of similarity search is to provide the top-k most similar vectors that match the vector-form of the input query. Once again, we can visualize this in a simplified 2-D vector space.

{{< figure src="knn.png">}}

The most naive way to do this is to compare the query vector with each and every vector in the database, with the so-called k-nearest neighbour method. However, this quickly becomes inefficient as we scale to millions (or billions) of vectors, as the search space keeps on increasing.

#### Approximate nearest neighbours

The algorithm at the heart of every vector database is called __Approximate Nearest Neighbour__ (ANN) search. Instead of performing an exhaustive search, an _approximate_ search is performed instead, where we lose some accuracy in the search result, but with a huge performance gain, allowing us to scale to searching billions of vectors in sublinear time.

#### Indexing

A key terminology in vector databases is _indexing_, which refers to the act of creating index data structures that efficiently look up the vectors by rapidly narrowing down on the search space. The deep learning models that encode the raw data as vectors typically produce vectors of high dimensionality (of the order of $10^2$ or $10^3$), and so, an ANN algorithm must attempt to generate a lower dimensional representation that captures the actual complexity of the data as efficiently as possible in time and space.

There are many vector indexing algorithms used for ANN search, and the details of them are out of the scope of this post. Stay tuned for more technical details in a future article!

- Locally Sensitive Hashing (LSH)
- Inverted File Index (IVF)
- Hierarchical Navigable Small World (HNSW) graphs
- Vamana

In general, the state-of-the-art in ANN indexing is achieved by newer algorithms like HNSW and Vamana, but not all database vendors offer these indexing options (yet).

### Putting it all together

To sum up: A vector database enables similarity search by scalably and rapidly retrieving the top-k approximate nearest neighbours for a given input query, and they achieve this by building efficient indexing structures under the hood. The choice of indexing algorithm impacts the quality of the similarity search, and this is a key area in which the different vector database vendors differ in their internals.

The following sequence of steps are performed to build a similarity search engine in production:

1. Transform raw data (text, images or audio) into vector embeddings using an appropriate embedding model (e.g., sentence transformers for text embeddings)
1. Build an ANN index using the generated vectors
1. Expose the vectors to a front end application via a search API layer
    - The API simply takes in the user query, transforms it into vector space using the same embedding model that was used to vectorize the data itself
1. Return the top-k approximate nearest neighbours that are closest to the given input query
