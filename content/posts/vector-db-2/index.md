+++ 
draft = true
date = 2023-07-02
title = "Vector databases (Part 2): A primer on embeddings and vector search"
description = "How vector can databases be used to power downstream applications, like chatting with your data via LLMs"
tags = ["vector-db"]
categories = ["general", "databases"]
math = true
+++

# How vector databases power a host of downstream applications

## Why are vector DBs so much in demand?

This is the second post in a series on vector databases. In this post, I'll describe at a reasonably high level what vector databases are doing under the hood, without going into too much technical detail on the indexing algorithms used -- that's for another post! As mentioned in [part 1](../vector-db-1/) of this series, there's been a lot of marketing (and unfortunately, hype) related to vector databases in the first half of 2023, and if you're reading this, you're likely curious what vector databases are actually doing under the hood, and how search engines, and search functionality in general, are built on top of efficient vector storage.

But first of all, what can explain the frenzy of activity and investment in the vector DB space?

### The age of Large Language Models (LLMs)

In November 2022, an early demo of ChatGPT (which is OpenAI's interface to GPT 3.5 and above) was released, following which it quickly became the [fastest growing application in history](https://www.forbes.com/sites/cindygordon/2023/02/02/chatgpt-is-the-fastest-growing-ap-in-the-history-of-web-applications/), gaining a million users in just 5 days 🤯! Indeed, if you look at the ⭐️ history on GitHub for the major open-source vector database repos, it's clear that there are sharp spikes in the star counts for some of them _after_ the release of ChatGPT. However, GitHub stars are just one way of gauging popularity of a framework. As can be seen in the chart below, not all technologies experienced the same spike in interest, meaning that the rising tide of marketing by several newer vector database vendors can be attributed, at least in part, to this uptick in 2023.

{{< figure src="vector-db-stars.png" caption="Made with ❤️ by [star-history.com](https://star-history.com/)">}}

### The problem with relying on LLMs

LLMs are _generative_, meaning that they generate meaningful, coherent text in a sequential manner based on a user prompt. However, when it comes to search & retrieval, this cannot on its own guarantee quality, accuracy or relevancy of the result generated.

- LLMs store/memorize a compressed version of their training data, and although they do so quite well, they don't do so _perfectly_ (some information is always "lost" in a model's internal representation of the data)
- An LLM cannot know facts that occurred after its training was completed
- An LLM often _hallucinates_, i.e., fabricates non-existent information, such as pointing users to URLs that don't exist

Vector databases help address many of these problems, via their highly efficient underlying storage layer that can be made easily accessible to an LLM. Unlike traditional databases, vector DBs utilize data structures that natively store data in vector form. As a result, we can now build applications with an LLM sitting on top of a vector storage layer that contains recent, up-to-date, factual data (well past the LLM's training date) to the end user.

Although vector database implementations (e.g., Vespa, Weaviate) existed well before the advent of LLMs, after the release of ChatGPT, the open-source community, as well as marketing teams at existing database vendors quickly realized their benefits in mainstream use cases like information retrieval and search. It's mainly for this reason that there's been an absolute bonanza of VC funding going on in the world of vector databases.

{{<notice note>}}
All this being said, there are genuine technical reasons why efficient vector storage is critical to a world with LLMs in it, so the rest of this post will focus on these aspects.
{{</notice>}}

## What is a vector embedding?

The data being stored in a vector database is in the form of embeddings, which are essentially lists of numbers (floats) that encode the contextual meaning of the data, which could be images, audio or text. Intuitively, when we refer to an "embedding", we are talking about a **compressed**, low-dimensional representation of data (images, text, audio) that actually exists in higher dimensions.

The dimension of the embedding depends on the number of data points $m$, and the embedding size of the vector $n$.

{{< figure src="vector-embedding.png" >}}

To store the entire data in the embedding space, the vectors that represent each data point are stacked together in an efficient manner, with optimizations in place to perform lookups at scale.


## How are embeddings generated?

The transformer revolution in NLP[^1] has provided practitioners ample means to generate these compressed representations, or embeddings.

- The most popular method is via the open source library `sentence-transformers`, available via [Hugging Face model hub](https://huggingface.co/models?library=sentence-transformers) or directly from [the source repo](https://www.sbert.net/)
- A less common and rather expensive way, is to use one of many paid API services:
   - [OpenAI embeddings API](https://platform.openai.com/docs/guides/embeddings)
   - [Cohere embeddings API](https://cohere.com/embed)

It's important to keep in mind that the lower the dimensionality of the underlying vectors, the more compressed the representation, which can affect downstream task quality. Sentence Transformers (sbert) provides embedding models with a dimension $n$ in the range of 384, 512 and 768, and the models are completely free and open-source. OpenAI and Cohere's embeddings, which require a paid API call to generate, can be considered higher quality due to a dimensionality of a few thousand. One reason it makes sense to use a paid API to generate embeddings is if your data is multilingual (Cohere is known to possess high-quality [multilingual embedding models](https://docs.cohere.com/docs/multilingual-language-models#model-performance) that perform better than open source variants).

{{< notice note >}}
The choice of model used to generate embeddings is typically a trade-off between quality and cost. In most cases, for textual data in English, the open-source embeddings from sentence transformers can be utilized as is for text that isn't too long (~300-400 words for text sequences). It's possible to deal with text that's longer than the context length of sentence transformer models, but that's for another post!
{{< /notice >}}

## Storing the embeddings in vector databases

Because of their amenability to operating in embedding space, vector databases are proving to be very useful in the realm of _semantic or similarity-based search_ for all forms of inputs (text, video/image, audio). This form of search considers semantics, where the input query sent by the user (typically in natural language) is translated into vector form, in the same embedding space as the data itself, so that results that are most similar to the input are returned. A visualization of this is shown below.

{{< figure src="embedding-pipeline.png">}}

## How is similarity computed?

The various vector databases offer different metrics to compute similarity, but for text, the following two metrics are most commonly used:
- __Dot product__: This produces a non-normalized value of an arbitrary magnitude
- __Cosine distance__: This produces a normalized value (between -1 and 1)

### An example of measuring cosine distance

Consider a simplified example where we vectorize the titles of red and white wines in 2-D space, where the horizontal axis represents red wines and the vertical axis represents white wines. In this space, points that are closer together would represent wines that share similar words or concepts, and points that are further apart do not have that much in common. The cosine distance is defined as the cosine of the angle between the lines connecting the position of each wine in the embedding space to the origin.

$$cos \theta = \frac{a^T \cdot b}{|a| \cdot{ |b|}}$$

A visualization will make this more intuitive.

{{< figure src="wines-cosine.png">}}

On the left, the two wines (the Reserve White and the Toscana Red) have very little in common, both in terms of vocabulary and concepts, so they have a cosine distance approaching zero (they are orthogonal in the vector space). On the right, the two Zinfandels from Napa Valley have a lot more in common, so they have a much larger cosine similarity closer to 1. The limiting case here would be a cosine distance of 1; a sentence is always perfectly similar to itself.

Of course, in a real situation, the actual vector space is high dimensional (and have many more axes other than just the variety of wine) and cannot be visualized on a 2-D plane, but the same principle of cosine similarity applies.

## Scalable nearest neighbour search

Once the vector embeddings are generated and stored, when a user submits a search query, the goal of similarity search is to provide the top-k most similar vectors that match the vector-form of the input query. Once again, we can visualize this in a simplified 2-D vector space.

{{< figure src="knn.png">}}

The most naive way to do this is to compare the query vector with each and every vector in the database, with the so-called k-nearest neighbour method. However, **this quickly becomes expensive** as we scale to millions (or billions) of vectors, as the number of comparisons required keeps increasing linearly with the data.

### Approximate nearest neighbours (ANN)

The reason why vector databases are so powerful is that they make search highly efficient (regardless of the size of the dataset), via a class of algorithms called __Approximate Nearest Neighbour__ (ANN) search algorithms. Instead of performing an exhaustive search, an approximate search is performed for nearest neighbours, resulting in some loss of accuracy in the search result, but a massive performance gain.

## Indexing

Data is stored in a vector database via _indexing_, which refers to the act of creating index data structures that efficiently look up the vectors by rapidly narrowing down on the search space. The deep learning models that encode the raw data as vectors typically produce vectors of high dimensionality (of the order of $10^2$ or $10^3$), and so, an ANN algorithm must attempt to generate a lower dimensional representation that captures the actual complexity of the data as efficiently as possible in time and space.

There are many vector indexing algorithms powering ANN search, and the details of them are out of the scope of this post. Stay tuned for a more technical description in part 3 of this series!

- Locally Sensitive Hashing (LSH)
- Inverted File Index (IVF)
- Hierarchical Navigable Small World (HNSW) graphs
- Vamana (utilized in the DiskANN implementation)

In general, the state-of-the-art in ANN indexing is achieved by newer algorithms like HNSW and Vamana, but not all database vendors offer newer indexing options like Vamana (yet).

## Putting it all together

To sum up what we covered so far: A vector database enables similarity search by scalably and rapidly retrieving the top-k approximate nearest neighbours for a given input query, and they achieve this by building efficient indexing structures under the hood. The choice of indexing algorithm impacts the quality of the similarity search, and this is a key area in which the different vector database vendors differ in their internals.

The following sequence of steps are performed to build a similarity search engine in production:

1. Transform raw data (text, images or audio) into vector embeddings using an appropriate embedding model (e.g., sentence transformers for text embeddings)
1. Build an ANN index using the generated vectors
1. Expose the vectors to a front end application via a search API layer
    - The API simply takes in the user query, transforms it into vector space using the same embedding model that was used to vectorize the data itself
1. Return the top-k approximate nearest neighbours that are closest to the given input query

---

# What can we do best with vector databases?




[^1]: Revolution in NLP is changing the way companies understand text, [TechCrunch](https://techcrunch.com/sponsor/nvidia/how-the-revolution-of-natural-language-processing-is-changing-the-way-companies-understand-text/)