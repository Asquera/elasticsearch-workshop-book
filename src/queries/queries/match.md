# Match Query

One of the standard queries to perform a full text search is the [match query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html). This query type returns documents that match the given input text.

> The input text in the search query is also analyzed. The `text` field type supports the `analyzer` field that is applied during **index time** and **search time**. A separate field `searcn_analyzer` can be used to set a different analyzer during **search time** but is rarely required to so.


## Index Mapping

To illustrate the match query let's create a new index with a mapping.

✅ Create a new index named `match_test` with the following mapping

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

This defines a mapping with the following fields:#

* `title` has field type [text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html)
* `tags` has field type [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html)

The **text** type also sets the [standard](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html) analyzer. The text input for the `title` field is analyzed using this analyzer. It ensures the indexed terms and the search query terms are analyzed the same way. Otherwise they would not necessarily match or would match on different terms.


## Add Documents

To illustrate the **match** query, we need a few documents in the index first.

✅ Bulk upload documents to index `match_test`

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200/match_test/_bulk' -d '
{"index":{"_index":"match_test"}}
{"title": "Harry Potter And The Philosophers Stone", "genre": "kids", "tags": ["action", "kids", "magic"]}
{"index":{"_index":"match_test"}}
{"title": "Star Trek", "genre": "action", "tags": ["space", "action", "better"]}
{"index":{"_index":"match_test"}}
{"title": "Star Wars", "genre": "action", "tags": ["space", "action", "good"]}
{"index":{"_index":"match_test"}}
{"title": "The Incredibles", "genre": "animation", "tags": ["kids", "spy"]}
{"index":{"_index":"match_test"}}
{"title": "Incredible Hulk", "genre": "action", "tags": ["action", "hero"]}
{"index":{"_index":"match_test"}}
{"title": "The Favourite", "genre": "comedy", "tags": ["costume", "play"]}
'
```

This adds a number of different documents with movie titles to the index.

The basic structure of a **match** query is

```json
{
  "query": {
    "match": {
      "<field>": {
        "query": "<search-term>"
      }
    }
  }
}
```

where `<field>` is the name of the field to search in (e.g. `title`) and the `<search-term>` is the term to search for.
See the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html).

The Search API endpoint for index `match_test` is available at `http://localhost:9200/match_test/_search`. To search for documents send a `GET` request with the query as JSON payload to this endpoint.

```bash
curl -X POST 'http://localhost:9200/match_test/_search' -H 'Content-Type: application/json' -d '{
  "query": {
    TODO!!
  }
}'
```

The JSON body needs to be filled with the query.


## Exercise

Build a **match** query to find documents.

✅ Write a query to find the text `star` in field `title`

<details>
<summary>A solution with <code>star</code> </summary>

```bash
curl -X POST 'http://localhost:9200/match_test/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "match": {
      "title": {
        "query": "star"
      }
    }
  }
}'
```
</details>

✅ Write a query to search for text `star trek` in field `title`

Check the search results for `star trek`, can you tell why there is also the document listed for `star wars`?

How could you solve this in a way, that the document with title `star wars` is not returned as a result?

<details>
<summary>Possible solution</summary>

Using `operator` field set to `AND` to require all terms to match.

```bash
curl -X POST 'http://localhost:9200/match_test/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "match": {
      "title": {
        "query": "star trek",
        "operator": "AND"
      }
    }
  }
}'
```
</details>

✅ Write a query that includes the [fuzziness](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query.html#query-dsl-match-query-fuzziness) property and search for `incredible`.

<details>
<summary>Possible solution</summary>

Using `fuzziness` with value `AUTO` allows a difference between search term and matched term.

```bash
curl -X POST 'http://localhost:9200/match_test/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "match": {
      "title": {
        "query": "incredible",
        "fuzziness": "AUTO"
      }
    }
  }
}'
```

The `fuzziness` setting is useful to match documents where the search term contains a typo or a transposition.
</details>

Check the results, are they what you would expect?
