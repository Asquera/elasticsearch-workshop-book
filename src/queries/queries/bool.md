# Bool Query

The `bool` query is a compound query that allows boolean combinations of other queries. It contains a list of boolean clauses that are useful in a number of search queries. It's often a good use case for full text search as a template.

See the [bool query documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html).

The basic structure of a bool query is

```json
{
  "query": {
    "bool": {
      "must": {..},
      "filter": {..},
      "must_not": {..},
      "should": {..}
    }
  }
}
```

These different boolean clauses are used as follows

* `must` - the query **must** appear in matching documents, contributes to the score
* `filter` - the query **must** appear in matching documents, the score for this query will be ignored
* `should` - the query **should** appear in the matching documents
* `must_not` - the query **must not** appear in the matching documents, 

> At least one of these clauses need to be specified inside a `bool` query.

> The bool query works on a "more matches is better" approach. Each score from the `must` and `should` clauses will contribute to the final score.

Each boolean clause can either take a single JSON object or a JSON array of multiple sub queries.
For example specifying a single `term` query in the `must` clause.

```json
{
  "bool": {
    "must": { "term": { "id": 5 } }
  }
}
```

For a list of multiple queries in the `should` clause a JSON array can be specified as well.

```json
{
  "bool": {
    "should": [
      { "term": { "id": 5 } },
      { "term": { "count": 5 } },
    ]
  }
}
```

In this case if any of the `term` queries match, the document is returned. If a document matches both it scores higher.


## Index Mapping

We are re-using the index mapping from the [term query](./term.md) page and store it in a new index.

✅ Create a new index named `bool_test` with the following mapping.

```bash
curl -X PUT 'http://localhost:9200/bool_test' -H 'Content-Type: application/json' -d '{
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
      },
      "color": {
        "type": "keyword"
      }
    }
  }
}'
```

This defines a mapping with the following fields

* `id` and `color` with field type [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html#keyword-field-type)
* `content` with numeric field type [integer](https://www.elastic.co/guide/en/elasticsearch/reference/current/number.html)
* `published_at` with field type [date](https://www.elastic.co/guide/en/elasticsearch/reference/current/date.html)


## Add Documents

We add a number of documents to the index.

✅ Bulk upload documents to index `bool_test`

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200/bool_test/_bulk' -d '
{"index":{"_index":"bool_test"}}
{"id": "1", "count": 10, "published_at": "2020-12-01", "color": ["yellow", "blue"]}
{"index":{"_index":"bool_test"}}
{"id": "2", "count": 30, "published_at": "2020-12-01", "color": ["green", "blue"]}
{"index":{"_index":"bool_test"}}
{"id": "3", "count": 20, "published_at": "2020-12-05", "color": ["red"]}
{"index":{"_index":"bool_test"}}
{"id": "4", "count": 10, "published_at": "2020-11-01", "color": ["blue"]}
{"index":{"_index":"bool_test"}}
{"id": "5", "count": 40, "published_at": "2020-12-24", "color": ["green", "blue"]}
{"index":{"_index":"bool_test"}}
{"id": "6", "count": 42, "published_at": "2020-03-05", "color": ["green"]}
{"index":{"_index":"bool_test"}}
{"id": "7", "count": 10, "published_at": "2020-12-01", "color": ["red"]}
{"index":{"_index":"bool_test"}}
{"id": "9", "count": 20, "published_at": "2020-12-01", "color": ["yellow", "red"]}
'
```

These documents have similar fields as the example in the [term query](./term.md) page.


## Exercise

Build a **term** query to find documents.

✅ Write a **bool** query to find documents that **must** have a `published_at` with value `2020-12-01` and **should** have a `count` of `10` or `20` or a `color` with value `blue`.

> There are multiple ways to solve this!

<details>
<summary>Possible solution</summary>

This query uses `must` and multiple `should` clauses.

```bash
curl -X POST 'http://localhost:9200/bool_test/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "bool": {
      "must": {
        "term": { "published_at": "2020-12-01" }
      },
      "should": [
        { "term": { "count": 10 } },
        { "term": { "count": 20 } },
        { "term": { "color": "blue" } }
      ]
    }
  }
}'
```
</details>

✅ Modify your previous **bool** query to require at least 2 of the **should** queries to match.

> Check the [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-bool-query.html).

<details>
<summary>Possible solution</summary>

The query uses a `must` clause and multiple `should` clauses.

```bash
curl -X POST 'http://localhost:9200/bool_test/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "bool": {
      "must": {
        "term": { "published_at": "2020-12-01" }
      },
      "should": [
        { "term": { "count": 10 } },
        { "term": { "count": 20 } },
        { "term": { "color": "yellow" } }
      ],
      "minimum_should_match": "2"
    }
  }
}'
```

> The `minimum_should_match` field defines how many clauses in the `should` block needs to match.

</details>
