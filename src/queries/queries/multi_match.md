# Multi Match Query

Another useful full text query is the [multi match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html). This query allows to search multiple fields at the same time. It provides a few more capabilities than a single match query. This query is a good start for more complex full text search.


## Index Mapping

To illustrate search over multiple fields, first we create a new index with a mapping.

✅ Create a new index named `multi_test` with the following mapping

```bash
curl -X PUT 'http://localhost:9200/multi_test' -H 'Content-Type: application/json' -d '{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard"
      },
      "description": {
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

This creates a new index named `multi_test` and an index mapping with three properties

* `title` and `description` are using field type [text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html) using the `standard` analyzer
* `tags` with field type [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html).


## Add Documents

Let's fill the empty index with some documents.

✅ Bulk upload documents to index `multi_test`

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200/multi_test/_bulk' -d '
{"index":{"_index":"multi_test"}}
{"title": "Harry Potter And The Philosophers Stone", "description": "First movie of many", "tags": ["action", "kids", "magic"]}
{"index":{"_index":"multi_test"}}
{"title": "Star Trek", "description": "This is not Star Wars", "tags": ["space", "action", "better"]}
{"index":{"_index":"multi_test"}}
{"title": "Star Wars", "description": "Its the least favourite one", "tags": ["space", "action", "good"]}
{"index":{"_index":"multi_test"}}
{"title": "Star Lord", "description": "A tale of two wars", "tags": ["space", "action", "good"]}
{"index":{"_index":"multi_test"}}
{"title": "The Incredibles", "description": "A good animated spy movie", "tags": ["kids", "spy"]}
{"index":{"_index":"multi_test"}}
{"title": "Incredible Hulk", "description": "Hulk is the star", "tags": ["action", "hero"]}
{"index":{"_index":"multi_test"}}
{"title": "The Favourite", "description": "Incredible cast is incredible", "tags": ["costume", "play"]}
'
```

After these documents were added, they can be searched.

> **🔎** There are a few ways to check if an index exists. Elasticsearch offers cat API endpoints that return machine readable output instead of JSON. One such API is the [cat indices API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-indices.html).
>
> ✅ Display the list of all indices
> ```bash
> curl -X GET http://localhost:9200/_cat/indices?v
> ```
> It lists a table with all indices, among the columns is the number of documents.

The basic structure of a **multi_match** query looks as follows:

```json
{
  "query": {
    "multi_match": {
      "query": "<search-term>",
      "fields": ["title"]
    }
  }
}
```

The `fields` property takes a comma separated list of fields to search in. In the given example the list contains only field `title`. The `<search-term>` is the term to search for.


## Exercise

Write **multi_match** queries to find a number of documents.
Check the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html) to learn what features you can use.

✅ Write a **multi_match** query to find documents whose text fields contain the term `incredible`

<details>
<summary>Possible Solution</summary>

```bash
curl -X POST 'http://localhost:9200/multi_test/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "multi_match": {
      "query": "incredible",
      "fields": ["title", "description"]
    }
  }
}'
```
</details>

✅ Write a **multi_match** query to find documents where the search term `star wars` is best found in one field.

<details>
<summary>Possible Solution</summary>

```bash
curl -X POST 'http://localhost:9200/multi_test/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "multi_match": {
      "query": "star wars",
      "fields": ["title", "description"],
      "type": "best_fields"
    }
  }
}'
```

> The `type` field allows a few different values. Compare the results between queries using `best_fields` and `most_fields`, what is the difference?

</details>

✅ Write a **multi_match** query to rank matches in `title` higher than in `description`.

<details>
<summary>Possible Solution</summary>

```bash
curl -X POST 'http://localhost:9200/multi_test/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "multi_match": {
      "query": "incredible",
      "fields": ["title^3", "description"]
    }
  }
}'
```

This query boosts a match in `title` by a factor of `3` compared to a match in `description`.

> Setting boost factors is a good way to rank documents according to the fields they match.
</details>
