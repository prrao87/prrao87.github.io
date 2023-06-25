+++ 
draft = true
date = 2023-06-24
title = "A brief primer on vector databases"
description = "What makes vector databases like Qdrant, Weaviate, Milvus, Vespa, Chroma, Pinecone and LanceDB different from one another"
tags = ["vector-db"]
categories = ["general", "databases"]
math = true
+++

## A gold rush in the vector search landscape

There's been a lot of marketing (and unfortunately, hype) on vector databases lately, and if you're reading this, you're likely curious _why_ they exist and what makes them different from one another. In June 2023, I prepared a [tracking spreadsheet](https://docs.google.com/spreadsheets/d/1tUxif_MQYByprhFArZ2XqssvlMVfn5mKjsPjYTOO08o/edit?usp=sharing) to document key defining characteristics and milestones of the various vector databases out there, and I now have __eight__ purpose-built vector databases and __three__ incumbent (established) vendors in my list. There are likely many more that I've missed.

In this post, I'll do my best to describe what vector databases do under the hood, without going into too much technical detail. I'll also differentiate between numerous vendors, and explain how you can begin to decide which one's the right one for your needs.

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

On the left, the two wines (the Reserve White and the Toscana Red) have very little in common, both in terms of vocabulary and concepts, so they have a cosine distance approaching zero (they are orthogonal in the vector space). On the right, the two Zinfandels from Napa Valley have a lot more in common, so they have a much larger cosine similarity closer to 1. The limiting case here would be a cosine similarity of 1 -- a sentence is always perfectly similar to itself.

Of course, in a real situation, the actual vector space is high dimensional (and have many more axes other than just the variety of wine) and cannot be visualized on a 2-D plane, but the same principle of cosine similarity applies.

---

## Vector databases


[^1]: How the revolution of natural language processing is changing the way companies understand text, [TechCrunch](https://techcrunch.com/sponsor/nvidia/how-the-revolution-of-natural-language-processing-is-changing-the-way-companies-understand-text/)