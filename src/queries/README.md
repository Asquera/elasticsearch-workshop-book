# Search

Elasticsearch provides a full [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html) (Domain Specific Language) to define search queries. There is an exhaustive list of available query types. 

> It's not necessarily feasible to know all of the different query types, more importantly it's better to learn what query capabilities Elasticsearch has to offer and focus on your use cases.

> **❗️** Building a text based query (e.g. auto completion, multi match) may require balancing different expectations. In text based queries there are no absolute true answers. There may be subjectively good & bad results for a particular query. The goal of the developer / designer is to index the documents in a way that using a well written query matches most of the user expectations.

✅ Start Elasticsearch instance (see [Setup](./../introduction/setup.md)).

This chapter is divided into the following sections.

* [Queries](./queries/README.md)
* [Suggestions](./suggestions/README.md)
* [Aggregations](./aggregations/README.md)


## Scoring

Elasticsearch ranks search results by calculating a score for each matching document that depends on its indexed data & the particular query. The scores define the order in which results are returned. The document with the highest score is returned first, the second highest scored is returned second, etc.

> **❗️** The *score* defines the order in which results are ranked. The scores themselves have no application level meaning. Given a particular query the calculated scores specify the relation between documents only. **Different** queries may produce **different** scores.

Each particular query generates its own range of scores that depends on a number of factors, e.g. sub queries, combinations, boost factors, indexed terms. Similar queries that search in separate fields, e.g. `title` vs `description` calculate scores on different sets of indexed terms. Combining these scores into a single list of results will most likely produce unexpected or wrong results, because their score ranges will be different.

> **🔎** For more information on scoring, see the [Elasticsearch Guide on Scoring](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/scoring-theory.html).


Useful Resources:

* [How Scoring Works In Elasticsearch](https://www.compose.com/articles/how-scoring-works-in-elasticsearch/)
