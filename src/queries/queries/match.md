# Match Query


-One of the standard queries to perform a full text search is the [match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html). This query type returns documents that match  given text. The text in the query is analyzed before matching.


## Index Mapping

To illustrate the match query let's create a new index with a mapping.

✅ Create a new index named `match_test` with mapping

```bash
curl -X PUT 'http://localhost:9200/match_test' -H 'Content-Type: application/json' -d '{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard"
      },
      "tags": {
        "type": "keyword"
      }
    }
  }
}'
```

-This defines a mapping with two properties, `title` and `tags`. The former has field type [text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html), the latter has the [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html) type. The **text** type also sets the [standard](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html) nalyzer. The text input for the `title` field is analyzed using this analyzer.


## Add Data

To illustrate the **match** query, we need a few documents in the index.

✅ Bulk upload documents to index `match_test`

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200
```