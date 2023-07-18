+++ 
draft = true
date = 2023-07-13
title = "Vector databases (Part 3): A deep dive into vector indexing"
description = "Understanding Flat, Annoy, IVF, HNSW and Vamana vector indexes in vector databases"
tags = ["vector-db"]
categories = ["databases"]
math = true
+++

# What are vector indexes and what do they do?

This is my third post in a series on vector databases. [Part 1](../vector-db-1/) covered the differences between various popular DB vendors in the market, while [Part 2](../vector-db-2/) focused on the basics of what vector databases are and what they do. This post is for those of you who want to go a little deeper into the technical details of what's happening under the hood 😉. My reasons for writing this post (and this series, in general) are that it's actually quite hard to gain a bird's eye view of the vector DB landscape, unless you're willing to put in a lot of time reading through the host of blog posts and marketing material out there. 

It's definitely possible to get *much* more detailed explanations elsewhere, but the information is scattered across numerous sources and it takes time and effort to absorb it all. Well, I've put in the work! My hope is that if you're reading this series, it helps you collate the knowledge scattered across the internet, and that you'll walk away with a deeper knowledge of what makes vector databases tick, and, hopefully use some of those lessons in your next project.

## Why should you care about an index?

Now that you know [what a vector database *is*](../vector-db-2/#putting-it-all-together), it's useful to take a step back and wonder, how does it all scale so wonderfully to millions, billions, or even trillions of vectors[^1]? The goal of vector databases is to provide a fast, efficient means to store data and query them via semantic (similarity-based) search on all forms of data, including text, images and audio. As such, it's important to differentiate between the search algorithm, and the underlying index on which the search algorithm operates.



As with most things, there is a tradeoff between accuracy (precision/recall) and performance, and this is where understanding the various indexing algorithms out there helps.





[^1]: Trillion-scale similarity, [Milvus blog](https://milvus.io/blog/Milvus-Was-Built-for-Massive-Scale-Think-Trillion-Vector-Similarity-Search.md)