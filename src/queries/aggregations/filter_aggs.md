# Filter & Filters Aggregations

The `filter` aggregation is another fairly simple bucket aggregation. It defines a filter similar to a [filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html) query as part of the `bool` query. See the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-filter-aggregation.html) on `filter` aggregation.

The following query shows the structure of a `filter` aggregation.

```json
{
  "aggs": {
    "<label>": {
      "filter": { ... },
      "aggs": {
        "<another-label>": { ... }
      }
    }
  }
}
```

The structure defines a single `filter` block inside the aggregation.

> The inner `aggs` block defines inner aggregations that get the output of the `filter` part as input.

It's similar to the following query using the bool-based `filter` query.

```json
{
  "query": {
    "bool": {
      "filter": { ... }
    }
  },
  "aggs": {
    "<another-label>": { ... }
  }
}
```

First the set of documents is filtered, then the aggregations are applied.

> **🔎** This particular example there is no real difference between using a bool based `filter` clause compared to using the `filter` aggregation. A good reason to use the former is, that the set of filtered documents applies to all aggregations. Using the `filter` aggregation from the example first determines the filtered documents before applying the inner aggregations on them. Other top level aggregations would therefore still see all documents. Depending on the type of search request using a bool based `filter` query first is probably the better option.


## Index Mapping

Let's build a new index to see how the `filter` aggregation works. It's the same mapping as on the [terms aggregation](./terms_aggs.md) page.

✅ Create a new index named `test_filter_aggs` with the following mapping.

```bash
curl -X PUT 'http://localhost:9200/test_filter_aggs' -H 'Content-Type: application/json' -d '{
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

We add a number of documents that define suitable `genre` and `tags` entries.

✅ Bulk upload documents to index `test_filter_aggs`

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200/test_filter_aggs/_bulk' -d '
{"index":{"_index":"test_filter_aggs"}}
{"title": "Star Trek Generations", "genre": "scifi", "tags": ["space", "captain", "scifi"]}
{"index":{"_index":"test_filter_aggs"}}
{"title": "Star Wars A New Hope", "genre": "scifi", "tags": ["space", "rebellion", "space opera"]}
{"index":{"_index":"test_filter_aggs"}}
{"title": "Stargate", "genre": "scifi", "tags": ["scifi", "portal", "military", "adventure", "space"]}
{"index":{"_index":"test_filter_aggs"}}
{"title": "Spiderman Homecoming", "genre": "superhero", "tags": ["comic", "superhero", "reboot"]}
{"index":{"_index":"test_filter_aggs"}}
{"title": "Spiderman Far From Home", "genre": "superhero", "tags": ["comic", "superhero", "sequel"]}
{"index":{"_index":"test_filter_aggs"}}
{"title": "Spiderman Into the Spiderverse", "genre": "animation", "tags": ["animation", "superhero", "multiverse"]}
{"index":{"_index":"test_filter_aggs"}}
{"title": "Rogue One: A Star Wars Story", "genre": "scifi", "tags": ["space", "rebellion", "death star"]}
{"index":{"_index":"test_filter_aggs"}}
{"title": "A Star is Born", "genre": "biopic", "tags": ["romance", "music", "singer"]}
{"index":{"_index":"test_filter_aggs"}}
{"title": "Star Trek Into Darkness", "genre": "scifi", "tags": ["space", "reboot", "starship"]}
{"index":{"_index":"test_filter_aggs"}}
{"title": "Third Star", "genre": "drama", "tags": ["friendship", "love"]}
{"index":{"_index":"test_filter_aggs"}}
{"title": "The Star Witness", "genre": "drama", "tags": ["prohibition", "murder", "duty"]}
'
```

## Exercise

The exercise is to build the equivalent `filter` aggregation to the following query based search request. The query finds all documentst that match *"scifi"* in the `genre` field and aggregates the result with a `terms` aggregation by `tags`.

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200/test_filter_aggs/_search?pretty' -d '{
  "query": {
    "bool": {
      "filter": {
        "term": {
          "genre": "scifi"
        }
      }
    }
  },
  "aggs": {
    "tags": {
      "terms": {
        "field": "tags"
      }
    }
  }
}'
```

✅ Build an equivalent search request that uses a `filter` aggregation instead of the `query` block from the search reqest above.

<details>
<summary>Possible solution</summary>

The query removes the `query` block and adds an equivalent `filter` aggregation.

```bash
curl -X POST 'http://localhost:9200/test_filter_aggs/_search?pretty' -H 'Content-Type: application/json' -d '{
  "size": 0,
  "aggs": {
    "genres": {
      "filter": { "term": { "genre": "scifi" } },
      "aggs": {
        "tags": {
          "terms": {
            "field": "tags"
          }
        }
      }
    }
  }
}'
```
</details>
