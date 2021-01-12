# Context Suggestions

The last suggester described in this chapter is an extension of the `suggest` completion query type. 
It is a context based suggester that can be used to filter and/or boost suggestions by some criteria.

For example assume you want to suggest movies boosted by certain genres. In order to provide context based suggestions the `completion` field in the mapping is expanded with `contexts` information.

> It's **mandatory** to provide a context when indexing / querying a context enabled completion field.


## Index Mapping

To showcase how context based suggestions work, the following index mapping is used to set up a new index. See the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html#context-suggester) on context suggesters.

✅ Create a new index named `context_suggestions` with the following mapping.

```bash
curl -X PUT 'http://localhost:9200/context_suggestions' -H 'Content-Type: application/json' -d '{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "completion": {
            "type": "completion",
            "contexts": [
              {
                "name": "genre",
                "type": "category",
                "path": "genre"
              }
            ]
          }
        }
      },
      "genre": {
        "type": "keyword"
      },
      "year": {
        "type": "integer"
      }
    }
  }
}'
```

The mapping defines the following fields.

* `title` with field type [text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html)
* `title.completion` multi field with type [completion](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html#completion-suggester) with two contexts that point to fields `genre` and `year` of the same document
* `genre` with field type [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html)
* `year` with field type [integer](https://www.elastic.co/guide/en/elasticsearch/reference/current/number.html)

The interesting part here is that the `title.completion` field that defines two categories (`genre` & `year`) inside the `contexts` list.

> Additional context mappings increase the index size for the completion field.

> Only `keyword` or `text` fields can be parsed for context fields.


## Add Documents

Once the index is created add a few documents to learn more about context suggestions.

✅ Bulk upload documents to index `context_suggestions`

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200/context_suggestions/_bulk' -d '
{"index":{"_index":"context_suggestions"}}
{"title": "Star Trek Generations", "genre": "space action", "year": 1994}
{"index":{"_index":"context_suggestions"}}
{"title": "Star Wars A New Hope", "genre": "space action", "year": 1977}
{"index":{"_index":"context_suggestions"}}
{"title": "Stargate", "genre": "scifi", "year": 1994}
{"index":{"_index":"context_suggestions"}}
{"title": "Spiderman Homecoming", "genre": "superhero", "year": 2017}
{"index":{"_index":"context_suggestions"}}
{"title": "Spiderman Far From Home", "genre": "superhero", "year": 2019}
{"index":{"_index":"context_suggestions"}}
{"title": "Spiderman Into the Spiderverse", "genre": "animation", "year": 2018}
{"index":{"_index":"context_suggestions"}}
{"title": "Rogue One: A Star Wars Story", "genre": "space action", "year": 2016}
{"index":{"_index":"context_suggestions"}}
{"title": "A Star is Born", "genre": "biopic", "year": 2018}
{"index":{"_index":"context_suggestions"}}
{"title": "Star Trek Into Darkness", "genre": "action", "year": 2013}
{"index":{"_index":"context_suggestions"}}
{"title": "Third Star", "genre": "drama", "year": 2010}
{"index":{"_index":"context_suggestions"}}
{"title": "The Star Witness", "genre": "drama", "year": 1931}
'
```

## Exercise & Example

A context suggestion query is more complex than the previous completion query. The query requires a `contexts` field in the `suggest` block.

✅ Run the following `suggest` query with search term *"star"* to get a list of results

```bash
curl -X POST 'http://localhost:9200/context_suggestions/_search?pretty' -H 'Content-Type: application/json' -d '{
  "suggest": {
    "text": "star",
    "context_completion_genre": {
      "completion": {
        "field": "title.completion",
        "contexts": {
          "genre": ["action"]
        }
      }
    }
  }
}'
```

> Remember the `completion` field type indexes input text for a prefix based search, the text in `title` has to start with the given search term.

A context in the `contexts` field also supports boost factors. This provides a mean to rank values for a context differently. See the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html) on context suggesters.

✅ Build a query that searches for *"spider"* in genres `superhero` and `animation`, that boosts `animation` by a factor of `2.0`. Return the top 2 entries of this search.

<details>
<summary>Possible solution</summary>

Similar to the previous query there are now entries in the `contexts` field for `genre`. The entry for genre `animation` defines a boost factor of `2.0` which ranks these matches higher.

```bash
curl -X POST 'http://localhost:9200/context_suggestions/_search?pretty' -H 'Content-Type: application/json' -d '{
  "suggest": {
    "text": "spider",
    "context_completion_genre": {
      "completion": {
        "field": "title.completion",
        "size": 2,
        "contexts": {
          "genre": [
            { "context": "superhero" },
            { "context": "animation", "boost": 2.0 }
          ]
        }
      }
    }
  }
}'
```

</details>
