+++ 
draft = true
date = 2023-07-29
title = "Vector databases (Part 4): Practical considerations"
description = "Understanding the trade-offs involved in working with and choosing a vector DB solution"
tags = ["vector-db"]
categories = ["databases"]
+++

## Choosing the right vector DB solution for your use case

Welcome back! In the [previous post](../vector-db-3/) in this series, we looked at the different types of vector indexes and their trade-offs. However, indexing is just a small portion of the entire pipeline when it comes to vector databases. Recall that in [part 2](../vector-db-2/#putting-it-all-together), we described what a vector database *is*. The key components that one needs to consider when deciding the system to use are described below:

* Application layer, and where it sits
* Data layer, and where it sits in relation to the *database* and the application layer
* Indexing strategy
* Storage layer and distributed capabilities

Each of these components, as always, involve trade-offs! I've broken them down into nine categories (I'm sure there are more that I've missed, but this is a start). Without further ado, let's dive in.

### 1. On-prem vs. cloud hosting

A lot of vendors sell "cloud-native" as though it's the best thing since sliced bread. However, in deciding what solution works best for your organization and use case, it's important to consider the breakdown of hosting options in the first place. It's now possible to have a mix-and-match of the following combinations:

* On-prem (self-hosted) + embedded
* Cloud-native (managed) + embedded
* Cloud-native (managed) + client-server

{{< figure src="vector-db-hosting-2.png">}}

It's clear that the most crowded space is the client-server + cloud-native model, that's been tried and tested in various other database paradigms over many years. The embedded/serverless hosting model, described below, is particularly interesting for this reason.

#### Embedded vs. client-server models

Ever since DuckDB successfully monetized its open-source embedded SQL DB offering through MotherDuck[^1], it's served as inspiration to others in the database space. In the world of vector DBs, one vendor stands out on this front: **LanceDB**[^2], which follows an embedded-first, serverless model. Its competitor, Chroma, also offers an embedded option, but it's purely in-memory and on a single machine, due to which they appear to be moving toward the tried-and-tested client-server model hosted on the cloud for scalability reasons[^3].

For organizations that hold a lot of data spread across on-premise and cloud environments, embedded/serverless vector DBs offer a lot of freedom and flexibility to  developers when connecting the data layers to the application layer (where users interact with the data), **especially when privacy and security are a concern**.

Despite a lot of cloud hosted options offering secure endpoints in which to deploy their offerings, there are still a lot of organizations that are hesitant to move their data to the cloud. This is especially true for organizations that have a lot of legacy systems that are not cloud-native. In these cases, it's often easier to deploy a vector database on-prem, and then connect it to the application layer. This is especially true for organizations that have a lot of legacy systems that are not cloud-native.

#### Cost considerations

Databases that offer cloud-native, managed solutions typically charge based on the amount of data stored and the number of queries made. This is a great model for organizations that have a lot of data, but don't want to pay for the infrastructure to transport and store it. However, for organizations that have a lot of data, but already have large in-house data infrastructure teams ready to support on-prem or embedded hosting, sending data to the cloud, both for indexing and for querying, makes almost no sense. While there's no "critical mass" in terms of an organization's size where this distinction can be clearly made, as an engineer or a technical leader, it's important to know that cloud-native + client-server databases **are not the only option** in 2023.



### 2. Purpose-built vendor vs. incumbent vendor

### 3. External embedding pipeline vs. built-in hosting of embedding models

### 4. Indexing speed vs. query speed

### 5. Recall vs. latency

### 6. In-memory vs. on-disk

### 7. Sparse vs. dense vector storage

### 8. Full-text search vs. vector search hybrid strategy

### 9. Filtering strategy


[^1]: Teach your DuckDB to fly, [MotherDuck](https://motherduck.com/)
[^2]: Developer-friendly, serverless vector database, [LanceDB](https://lancedb.com/)
[^3]: Deployment, [Chroma](https://docs.trychroma.com/deployment)