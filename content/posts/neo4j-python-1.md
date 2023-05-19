---
type: "post"
title: "Neo4j for Pythonistas: Part 1"
date: 2023-05-01T21:29:09-04:00
draft: true
showTableOfContents: true
tags:
- python
- neo4j
- async
- pydantic
- knowledge-graphs
- data-engineering
---

## Using Pydantic and async Python to build a graph Neo4j

Neo4j has been among the world's [most popular graph databases](https://www.endava.com/en/blog/Engineering/2021/Following-the-patterns-the-rise-of-neo4j-and-graph-databases) for a while now, and I really enjoy working with it, and graphs in general. A quick Google search reveals *lot* of blog posts and tutorials that show how to bulk-load data into Neo4j via Cypher, Neo4j's query language, or [APOC](https://neo4j.com/labs/apoc/) (Awesome Procedures on Cypher). If you're a Python engineer like me (because, ugh, Java 😖), and are looking to use Python all the way to ingest large amounts of data into Neo4j in the most efficient way possible, read on!

But first, let's take a step back, and ask ourselves, when APOC and Cypher can do the job, why involve Python at all?

* You may already have other data-handling workflows in Python, so switching to another language like Java or bash scripts for Neo4j can be a chore.
* Using a data validation framework like [Pydantic](https://docs.pydantic.dev/latest/) allows you to increase the robustness of the ETL process, while staying within the world of Python and not have to glue together tools written in multiple languages.
* Most database Python clients these days implement asynchronous clients to communicate with databases in a non-blocking manner, and [Neo4j is no exception](https://neo4j.com/docs/python-manual/current/concurrency/).
* Python's `asyncio` API has evolved a lot since its early days, and it's actually now quite easy to use.

Combining data validation via Pydantic with async data ingestion can make for fast, efficient and readable Python code that can be maintained by future engineers without having to deal with a mess of glue code. With that in mind, let's get started with an example!

## The data

We'll be working with [this wine reviews dataset from Kaggle](https://www.kaggle.com/datasets/zynicide/wine-reviews). It consists of 130k wine reviews from the Wine Enthusiast magazine, including the variety, location, winery, price, description, and some other metadata for each wine. Refer to the Kaggle source for more detailed information on the data and how it was scraped. The original data was made available as a single JSON file. For the purposes of this blog post, the data was converted to newline-delimited JSON (`.jsonl`) format where each line of the file contains a valid JSON object.

An example JSON line is shown below.

```json
{
    "points": "90",
    "title": "Castello San Donato in Perano 2009 Riserva  (Chianti Classico)",
    "description": "Made from a blend of 85% Sangiovese and 15% Merlot, this ripe wine delivers soft plum, black currants, clove and cracked pepper sensations accented with coffee and espresso notes. A backbone of firm tannins give structure. Drink now through 2019.",
    "taster_name": "Kerin O'Keefe",
    "taster_twitter_handle": "@kerinokeefe",
    "price": 30,
    "designation": "Riserva",
    "variety": "Red Blend",
    "region_1": "Chianti Classico",
    "region_2": null,
    "province": "Tuscany",
    "country": "Italy",
    "winery": "Castello San Donato in Perano",
    "id": 40825
}
```

## The graph data model

The first step prior to data import is conceptualizing a data model. This primarily depends on the kinds of questions we're trying to answer. In this dataset, each JSON blob contains information about one particular wine, but, conceptually, how are the wines themselves connected to each other?

Inspecting the JSON blob above, we can see that there is not only a rating and price information, but also location information about each wine (country, province, region, etc.). A wine is also tasted by a reviewer, who is a person in the real world with a Twitter handle.

### Where graphs really shine

The reason that graph databases are _really_ useful in the real world is best illustrated with an example. Imagine you're building a query API for wine enthusiasts that follow wine reviewers on Twitter. Suppose these end users are interested the question: "Which wine varieties have been most tasted by my favourite reviewer, Roger Voss, and what countries are those wines from"?

This seems like a reasonable enough question. But the problem is, the data we have is in the form of individual JSON blobs (as shown above), where each wine reviewer's name and Twitter handle is associated with a *single* wine, and the reviewer's name appears many, many times over the entire dataset. A conventional NoSQL data store (like MongoDB) would require an aggregation query [built through a pipeline](https://www.mongodb.com/docs/manual/core/aggregation-pipeline/) to answer this question, where it would first calculate the number of wines tasted by a person, apply the necessary grouping clauses to gather the wine variety and the country of origin, and finally return the answer. Not only is this approach computationally expensive as the dataset gets larger and larger, it's also *exceptionally unintuitive* to us as human beings to reason about.

Our minds tend to see the real world as a collection of concepts and entities, and graphs, due to their inherent "connected" nature, make it very easy to reason about the data at hand. With this in mind, the following graph model makes sense.

![](/images/neo4j-python-1/data_model.png)

We can view each JSON blob as a `Wine` node, that connects to a `Person` node, where the `Person` is a taster. In addition, each wine is also connected to a `Country` node and a `Province` node, with each type of location indicating where the wine originates. In addition, a `Province` can be located in a `Country`, but not the other way around. The above data model encapsulates all of these real-world relationships nicely, capturing both the *nature* and the *direction* of the relationship, and as an added bonus, makes querying a lot easier as well.

If the data is modelled this way, the following Cypher query in Neo4j answers the original question in just a few lines of code:


```sql
-- Which wine varieties have been most tasted by my favourite reviewer, Roger Voss,
-- and what countries are those wines from
MATCH (wine:Wine)-[:TASTED_BY]->(taster:Person {tasterName: "Roger Voss"})
WITH wine, taster
MATCH (wine)-[r:IS_FROM_COUNTRY]->(country:Country)
RETURN
  taster.tasterName AS tasterName,
  wine.variety AS wineVariety,
  count(r) AS winesTasted,
  country.countryName AS country
ORDER BY winesTasted DESC, tasterName LIMIT 5
```

The result looks something like below.

```
╒════════════╤══════════════════════════╤═══════════╤══════════╕
│tasterName  │wineVariety               │winesTasted│country   │
╞════════════╪══════════════════════════╪═══════════╪══════════╡
│"Roger Voss"│"Bordeaux-style Red Blend"│4702       │"France"  │
├────────────┼──────────────────────────┼───────────┼──────────┤
│"Roger Voss"│"Chardonnay"              │2724       │"France"  │
├────────────┼──────────────────────────┼───────────┼──────────┤
│"Roger Voss"│"Portuguese Red"          │2462       │"Portugal"│
├────────────┼──────────────────────────┼───────────┼──────────┤
│"Roger Voss"│"Pinot Noir"              │1827       │"France"  │
├────────────┼──────────────────────────┼───────────┼──────────┤
│"Roger Voss"│"Rosé"                    │1555       │"France"  │
└────────────┴──────────────────────────┴───────────┴──────────┘
```

We can see that Roger Voss is a hugely prolific wine taster! And a [quick Google search reveals the same](https://www.winemag.com/2013/09/19/wes-roger-voss-wins-prestigious-bordeaux-award/) -- Between 2009-2013, Roger had tasted over 3,000 Bordeaux-style red wines! And our dataset in this blog post is from 2017, by which time he managed to taste over 4,700 of them! 🤯


## Data ingestion into Neo4j

With that bit of background out of the way, let's look at how to build the graph using Python. And we won't just be ingesting the data into Neo4j as is. As mentioned before, the following ETL best practices are baked into this process:

- __Data quality__: The data is validated via [Pydantic](https://docs.pydantic.dev/latest/) to ensure that the types align with what we expect during querying
- __Performance__: The Cypher queries that actually perform the merging operations are written with efficiency, quality and scalability in mind

### Option 1: Sync loader




### Option 2: Async loader

