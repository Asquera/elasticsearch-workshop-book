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
{"id": "6", "count": 20, "published_at": "2020-12-01"}
'
```

This adds a number of documents with the specified fields to the index.

The basic structure of a **term** query is

```json
{
  "query": {
    "term": {
      "id": {
        "value": "1"
      }
    }
  }
}
```

The field to search in is the `id` field, the search term to search for is `"1"`.
See the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-term-query.html) for more details.

The Search API endpoint for index `term_test` is available at `http://localhost:9200/term_test/_search`.


## Exercise

Build a **term** query to find documents.

✅ Write a query to find documents with id `5`

<details>
<summary>Possible solution</summary>

```bash
curl -X POST 'http://localhost:9200/term_test/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "term": {
      "id": {
        "value": "5"
      }
    }
  }
}'
```
</details>

✅ Write a query to find documents with published date `2020-12-01`

<details>
<summary>Possible solution</summary>

```bash
curl -X POST 'http://localhost:9200/term_test/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "term": {
      "published_at": {
        "value": "2020-12-01"
      }
    }
  }
}'
```
</details>

> There is a `terms` query that is similar to `term` that returns documents with one or more **exact** matches.

See the [Elasticsearch documentation on terms query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-terms-query.html).

✅ Write a **terms** query to find documents with counts `10` and `20`

<details>
<summary>Possible solution</summary>

```bash
curl -X POST 'http://localhost:9200/term_test/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "terms": {
      "count": ["10", "20"]
    }
  }
}'
```
</details>

> How would you query documents that must have `count` with value `10` **and** `published_date` set to `2020-12-01`?
