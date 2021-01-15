## Scoring

Elasticsearch ranks search results by calculating a score for each matching document that depends on its indexed data & the particular query. The scores define the order in which results are returned. The document with the highest score is returned first, the second highest scored is returned second, etc.

> **❗️** The *score* defines the order in which results are ranked. The scores themselves have no application level meaning. Given a particular query the calculated scores specify the relation between documents only. **Different** queries may produce **different** scores.

Each particular query generates its own range of scores that depends on a number of factors, e.g. sub queries, combinations, boost factors, indexed terms. Similar queries that search in separate fields, e.g. `title` vs `description` calculate scores on different sets of indexed terms. Combining these scores into a single list of results will most likely produce unexpected or wrong results, because their score ranges will be different.

> **🔎** For more information on scoring, see the [Elasticsearch Guide on Scoring](https://www.elastic.co/guide/en/elasticsearch/guide/2.x/scoring-theory.html).


> The **query context** in a query tries to answer question *How well does a document match this query clause*? For these queries the relevance score is calcualted.

> The **filter context** in a query answers the question *Does this document match this query clause*? It's a binary decision, either the document matches or not. For these filters no score is calculated or a default value of `1.0` is automatically applied.


## Useful Resources:

* [Query And Filter Context](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-filter-context.html) - Elasticsearch documentation
* [How Scoring Works In Elasticsearch](https://www.compose.com/articles/how-scoring-works-in-elasticsearch/)

