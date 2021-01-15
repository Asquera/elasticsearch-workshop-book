# Search

Elasticsearch provides a full [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html) (Domain Specific Language) to define search queries. There is an exhaustive list of available query types. 

> It's not necessarily feasible to know all of the different query types, more importantly it's better to learn what query capabilities Elasticsearch has to offer and focus on your use cases.

> **❗️** Building a text based query (e.g. auto completion, multi match) may require balancing different expectations. In text based queries there are no absolute true answers. There may be subjectively good & bad results for a particular query. The goal of the developer / designer is to index the documents in a way that using a well written query matches most of the user expectations.

✅ Start Elasticsearch instance (see [Setup](./../introduction/setup.md)).

This chapter is divided into the following sections.

* [Introduction](./introduction.md) - an introduction into how to set up an index with mapping and documents
* [Scoring](./scoring.md) - how scoring documents works in Elasticsearch
* [Queries](./queries/README.md) - a list of search query examples
* [Suggestions](./suggestions/README.md) - a list of suggestion search queries
* [Aggregations](./aggregations/README.md) - a list of aggregations to calculate / summarize over documents

