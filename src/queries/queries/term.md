# Term Query

One of the [term-level queries](https://www.elastic.co/guide/en/elasticsearch/reference/current/term-level-queries.html) is the [term query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html). This query type returns document that **exactly** match a term in a provided field.

> A term query is not necessarily applicable to `text` based fields. Analyzers store modified terms of the text. This may result in different matches.


## Index Mapping

✅ Create a new index named `term_test` with the following mapping.

```bash
curl -X PUT 'http://localhost:9200/term_test' -H 'Content-Type: application/json' -d '{
  "mappings": {
    "properties": {
      "id": {
        "type": "keyword"
      },
      "count": {
        "type": "integer"
      },
      "published_at": {
        "type": "date"
      }
    }
  }
}'
```

This defines a mapping with the following fields

* `id` with field type [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html#keyword-field-type)
* `content` with numeric field type [integer](https://www.elastic.co/guide/en/elasticsearch/reference/current/number.html)
* `published_at` with field type [date](https://www.elastic.co/guide/en/elasticsearch/reference/current/date.html)


## Add Documents

✅ Bulk upload documents to index `term_test`

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200/term_test/_bulk' -d '
{"index":{"_index":"term_test"}}
{"id": "1", "count": 10, "published_at": "2020-12-01"}
{"index":{"_index":"term_test"}}
{"id": "2", "count": 20, "published_at": "2020-12-05"}
{"index":{"_index":"term_test"}}
{"id": "3", "count": 1, "published_at": "2020-11-01"}
{"index":{"_index":"term_test"}}
{"id": "4", "count": 4, "published_at": "2020-12-24"}
{"index":{"_index":"term_test"}}
{"id": "5", "count": 42, "published_at": "2020-03-05"}
{"index":{"_index":"term_test"}}
{"id": "6", "count": 50, "published_at": "2020-06-13"}
'
```
