# Terms Aggregation

One of the simpler aggregations is the `terms` bucket aggregation (see [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-terms-aggregation.html)).
It groups documents by terms. All documents that match the same term, e.g. the same `genre`, are grouped into the same bucket.

> Terms aggregations work well on the `keyword` field type or other non-text fields.

> **❗️** It's possible but not recommended to build a terms aggregation for `text` field types, but this requires to enable [fielddata](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html#fielddata-mapping-param).

One use case of a `terms` aggregation is to group all relevant documents by a one or multiple criteria. For example in a web shop an aggregation could be used to count all products by category, color, size, price range etc. These types of aggregation often play well in combination with filters the user may want to enable, e.g. products with color red. In this sense the aggregation also helps to navigate the search space. Another use case may be to find the most popular movies by counting ratings.


## Index Mapping

Let's build a new index to test how terms aggregations work.

✅ Create a new index named `test_terms_aggs` with the following mapping.

```bash
curl -X PUT 'http://localhost:9200/test_terms_aggs' -H 'Content-Type: application/json' -d '{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard"
      },
      "genre": {
        "type": "keyword"
      },
      "tags": {
        "type": "keyword"
      }
    }
  }
}'
```

This defines a mapping with the following fields:

* `title` using the field type [text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html) with the `standard` analyzer.
* `genre` and `tags` using field type [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html)


## Add Documents

Let's add a few documents that define suitable `genre` and `tags` entries.

✅ Bulk upload documents to index `test_terms_aggs`

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200/test_terms_aggs/_bulk' -d '
{"index":{"_index":"test_terms_aggs"}}
{"title": "Star Trek Generations", "genre": "scifi", "tags": ["space", "captain", "scifi"]}
{"index":{"_index":"test_terms_aggs"}}
{"title": "Star Wars A New Hope", "genre": "scifi", "tags": ["space", "rebellion", "space opera"]}
{"index":{"_index":"test_terms_aggs"}}
{"title": "Stargate", "genre": "scifi", "tags": ["scifi", "portal", "military", "adventure", "space"]}
{"index":{"_index":"test_terms_aggs"}}
{"title": "Spiderman Homecoming", "genre": "superhero", "tags": ["comic", "superhero", "reboot"]}
{"index":{"_index":"test_terms_aggs"}}
{"title": "Spiderman Far From Home", "genre": "superhero", "tags": ["comic", "superhero", "sequel"]}
{"index":{"_index":"test_terms_aggs"}}
{"title": "Spiderman Into the Spiderverse", "genre": "animation", "tags": ["animation", "superhero", "multiverse"]}
{"index":{"_index":"test_terms_aggs"}}
{"title": "Rogue One: A Star Wars Story", "genre": "space action", "tags": ["space", "rebellion", "death star"]}
{"index":{"_index":"test_terms_aggs"}}
{"title": "A Star is Born", "genre": "biopic", "tags": ["romance", "music", "singer"]}
{"index":{"_index":"test_terms_aggs"}}
{"title": "Star Trek Into Darkness", "genre": "action", "tags": ["space", "reboot", "starship"]}
{"index":{"_index":"test_terms_aggs"}}
{"title": "Third Star", "genre": "drama", "tags": ["friendship", "love"]}
{"index":{"_index":"test_terms_aggs"}}
{"title": "The Star Witness", "genre": "drama", "tags": ["prohibition", "murder", "duty"]}
'
```


## Exercise

See the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-terms-aggregation.html) on `terms` aggregation.


✅ Build a search query with a `terms` aggregation to count all documents by `genre`.

<details>
<summary>Possible solution</summary>

The following query uses the term aggregation named *"genres"* to build buckets for distinct entries of `genre` for all documents.

```bash
curl -X POST 'http://localhost:9200/test_terms_aggs/_search?pretty' -H 'Content-Type: application/json' -d '{
  "aggs": {
    "genres": {
      "terms": {
        "field": "genre"
      }
    }
  }
}'
```

</details>

What can you observe from the JSON response?

✅ Build a query that finds documents that have the text *"space"* in the field `tag` and has a `terms` aggregation that ranks them by `genre`. The documents themselves are not required to be part of the response.

Check the [Elasticsearch reference](https://www.elastic.co/guide/en/elasticsearch/reference/master/search-search.html) on the Search API for more information.

> There are multiple solutions, it's recommended to use a `query` block to match the documents first.
> It may be useful to start with the `query` block first to check that the right documents are matched.

<details>
<summary>A solution</summary>

The following search request uses a `query` block with `term` match to find all documents that contain the text *"space"* in the `tag` document field.

```bash
curl -X POST 'http://localhost:9200/test_terms_aggs/_search?pretty' -H 'Content-Type: application/json' -d '{
  "size": 0,
  "query": {
    "term": {
      "tags": "space"
    }
  },
  "aggs": {
    "genres": {
      "terms": {
        "field": "genre"
      }
    }
  }
}'
```

> The `size` field (with value `0`) in the search request signals Elasticsearch to not return any documents.

</details>
