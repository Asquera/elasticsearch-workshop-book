# Pagination

Elasticsearch returns the top 10 matching results for a search request by default. In most situations that is not sufficient, for example the user wants to browse the products on a product page.
One way to fetch more documents is to use the `from` & `size` parameters in the search query, see the [Search API documentation](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/search-search.html) for more information.

> **❗️** The default limit of documents to fetch by using `from` and `size` is `10.000` (defined by the [index.max_result_window](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/index-modules.html#index-max-result-window) setting). Increasing this limit may have an impact on search performance.

Using `size` and `from` parameter provides an easy way to implement pagination of search results, e.g. products.
The search query can look as follows:

```json
{
  "from": 90,
  "size": 10,
  "query": {
    "match_all": {}
  }
}
```

This query will fetch documents `91` to `100`, skipping the first 90 documents.

**🔎** Elasticsearch has to fetch a total of `90 + 10` documents in order to determine which are the matches from `91 - 100`. The first `90` documents are then omitted. During the *query* phase of the search, all requested shards forward the request to Lucene, each Lucene segment tries to determine `90` documents. When merging the results from all Lucene segments over the shard level back to the index level a lot of documents have to be fetched and are then filtered out.

> **❗️** The deeper the pagination parameters are the more documents have to be considered in total. The result is a degrading performance and longer response times.


## Search After

Since Elasticsearch 7.10 the recommended way to fetch deep pages of documents is to use the `search_after` feature of Elasticsearch (see [Search After documentation](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/paginate-search-results.html#search-after)).

Elasticsearch recommends the `search_after` parameter instead of using the [Scroll API](https://www.elastic.co/guide/en/elasticsearch/reference/current/scroll-api.html) to get the next batch of search results.

Using `search_after` requires multiple search requests with the same `query` and `sort` values.
The response then contains a `sort` field with values that are used for subsequent queries which acts as a pointer into the search results.

> **❗️** There is a *point in time* feature to keep search results consistent, otherwise an index refesh may return already seen documents or skip them during subsequent searches. This feature is part of [XPack](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-xpack.html) and may not be available.


## Example

Let's create an index with a number of documents to illustrate the `search_after` feature, without the *point in time* feature.

✅ Create a new index named `test_search_after` with the following mapping.

```bash
curl -X PUT 'http://localhost:9200/test_search_after' -H 'Content-Type: application/json' -d '{
  "mappings": {
    "properties": {
      "id": {
        "type": "integer"
      },
      "title": {
        "type": "text"
      }
    }
  }
}'
```

✅ Bulk upload documents to index `test_search_after`

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200/test_search_after/_bulk' -d '
{"index":{"_index":"test_search_after"}}
{"id": 1, "title": "Star Trek Generations"}
{"index":{"_index":"test_search_after"}}
{"id": 2, "title": "Star Wars A New Hope"}
{"index":{"_index":"test_search_after"}}
{"id": 3, "title": "Stargate"}
{"index":{"_index":"test_search_after"}}
{"id": 4, "title": "Spiderman Homecoming"}
{"index":{"_index":"test_search_after"}}
{"id": 5, "title": "Spiderman Far From Home"}
{"index":{"_index":"test_search_after"}}
{"id": 6, "title": "Spiderman Into the Spiderverse"}
{"index":{"_index":"test_search_after"}}
{"id": 7, "title": "Rogue One: A Star Wars Story"}
{"index":{"_index":"test_search_after"}}
{"id": 8, "title": "A Star is Born"}
{"index":{"_index":"test_search_after"}}
{"id": 9, "title": "Star Trek Into Darkness"}
{"index":{"_index":"test_search_after"}}
{"id": 10, "title": "Third Star"}
{"index":{"_index":"test_search_after"}}
{"id": 11, "title": "The Star Witness"}
'
```

This adds a number of documents to our index.

## Exercise

To get the first page of results using `search_after`, use the `sort` argument in the search request. To make this work this requires a sortable field in the document. In the indexed documents there is an `id` field that can be sorted.

Check the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/paginate-search-results.html#search-after) on `search_after`.

> Without a `sort` parameter all matched documents are ranked by their score. Documents can be sorted by different fields or by running a short script instead. Check the [Elasticsearch documentation on sorting search results](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/sort-search-results.html) for more details.

✅ Build a search request to get the first `5` documents of all documents, specify the `id` property in the `sort` block.

<details>
<summary>Possible solution</summary>

For the `id` field we know that the first value is `1`, therefore providing `0` is a good start.

```bash
curl -H 'Content-Type: application/x-ndjson' -X GET 'http://localhost:9200/test_search_after/_search?pretty' -d '{
  "size": 5,
  "query": {
    "match_all": {}
  },
  "sort": {
    "id": "asc"
  },
  "search_after": [0]
}'
```
</details>

✅ Check the response for `sort` values.

What is the next step to get the next `5` documents, what would you do?

What are common use cases for this feature?

<details>
<summary>A few ideas</summary>

* scrolling through logs by a `timestamp` field
* pagination for larger product catalogs by `product_id` or something else

> Some sortable fields may return results which seem odd, e.g. `_doc_id`, even though they work.

</details>
