# Nested Aggregation

Documents can contain nested properties by using the `nested` field type (see [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html)). The `nested` field type is useful when there is a relation between the main document and other related documents. For example a restaurant may contain the opening hours as inner objects or a blog post may contain comments.

Imagine having the following document for a restaurant.

```json
{
  "name": "Restaurant at the end of the universe",
  "opening_hours": [
    { "day": "monday", "time": "10-18" },
    { "day": "tuesday", "time": "10-19" },
    { "day": "wednesday", "time": "10-14" },
    { "day": "thursday", "time": "10-14" },
    { "day": "friday", "time": "09-16" },
  ]
}
```

The document describes a restaurant with `opening_hours` as a list of tuples of `day` and `time`.
When indexing this document structure **without** a `nested` field, the result is that these fields are stored as flat arrays.

A search request using this mapping structure cannot easily answer the question on which days the restaurant is open between `10-14` hours. The response will include the full document because when at least one entry in the `time` field match the value of `10-14`, but it's not necessarily clear how many / which days this applies to.

A mapping without the `nested` field type may look like the following mapping.

```json
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "opening_hours": {
        "properties": {
          "day": {
            "type": "keyword"
          },
          "time": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

An attempt to find all documents where `time` is `10-18` and `day` contains `monday` may look like:

```bash
curl -H 'Content-Type: application/json' -X GET 'http://localhost:9200/nested_aggs/_search?pretty' -d '{
  "query": {
    "bool": {
      "must": [
        { "term": { "opening_hours.day": { "value": "monday" } } },
        { "term": { "opening_hours.time": { "value": "10-18" } } }
      ]
    }
  }
}'
```

This query will return all documents were any entry of `opening_hours.day` contains `monday` and `opening_hours.time` contains the value `10-18`, but the specific combination is not expressed, the `term` queries are not related to the nested document structure. Therefore the response contains all documents were fields hold these values.

The `nested` field type groups these properties as inner documents, Elasticsearch indexes these inner documents separately. Once the index mapping makes use of the `nested` field a search request can be more precise by using a `nested` sub query.

<details>
<summary>Check the updated index mapping</summary>

The updated mapping with a `nested` field type may look as follows.

```json
{
  "mappings": {
    "properties": {
      "name": {
        "type": "text"
      },
      "opening_hours": {
        "tytpe": "nested",
        "properties": {
          "day": {
            "type": "keyword"
          },
          "time": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

This marks the `opening_hourss` field as a `nested` field.

</details>

For example to find all restaurants which are open on a `monday` between `10-18` using a bool based query with a `nested` sub query.

```bash
curl -H 'Content-Type: application/json' -X GET 'http://localhost:9200/nested_aggs/_search?pretty' -d '{
  "query": {
    "nested": {
      "path": "opening_hours",
      "query": {
        "bool": {
          "must": [
            { "term": { "opening_hours.day": { "value": "monday" } } },
            { "term": { "opening_hours.time": { "value": "10-18" } } }
          ]
        }
      }
    }
  }
}'
```

This time the query only returns the restaurants (documents) that match both values in the same tuple of `day` and `time`.


## Index Mapping

We have seen what the mapping may look like without the `nested` field type. Let's create a new index named `nested_aggs` with a similar mapping that defines a `nested` object `opening_hours` with `day` & `time`.

✅ Create new index `nested_aggs` with the following mapping.

```bash
curl -H 'Content-Type: application/json' -X PUT 'http://localhost:9200/nested_aggs' -d '{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard"
      },
      "opening_hours": {
        "type": "nested",
        "properties": {
          "day": {
            "type": "keyword"
          },
          "time": {
            "type": "keyword"
          }
        }
      }
    }
  }
}'
```

This defines an index mapping with the following fields

* `title` using the field type [text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html) with the `standard` analyzer.
* `opening_hours` field type [nested](https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html)
* `day` and `time` use field type [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html)


## Add Documents

Now that we have an index with a `nested` type, let's add a few documents.

✅ Bulk upload documents to index `nested_aggs`

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200/nested_aggs/_bulk' -d '
{"index":{"_index":"nested_aggs"}}
{"title":"Restaurant at the end of the universe", "opening_hours":[{"day":"monday","time":"10-18"},{"day":"wednesday","time":"10-18"}, {"day":"friday","time":"10-18"}]}
{"index":{"_index":"nested_aggs"}}
{"title":"Djimalaya", "opening_hours":[{"day":"monday","time": "10-18"}, {"day":"wednesday","time":"10-18"}, {"day":"friday","time":"10-18"}]}
{"index":{"_index":"nested_aggs"}}
{"title":"Kebap", "opening_hours":[{"day":"sunday","time": "10-14"}]}
{"index":{"_index":"nested_aggs"}}
{"title":"Curry 36", "opening_hours":[{"day":"monday","time": "10-22"},{"day":"tuesday","time": "10-18"},{"day":"thursday","time": "10-18"}]}
'
```


## Exercise

A nested aggregation is a bucket aggregation that calculates / summarizes data over nested documents.
It works on nested fields and requires a `path` to the nested object. See the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-nested-aggregation.html) on nested aggregations for more details.
The `nested` path tells Elasticsearch where the inner documents reside.

✅ Build a `nested` aggregation to group / count all restaurants by `opening_hours.day`.

<details>
<summary>Possible solution</summary>

This solution uses a `nested` aggregation.

```bash
curl -H 'Content-Type: application/json' -X GET 'http://localhost:9200/nested_aggs/_search?pretty' -d '{
  "size": 0,
  "aggs": {
    "open restaurants": {
      "nested": {
        "path": "opening_hours"
      },
      "aggs": {
        "by day": {
          "terms": {
            "field": "opening_hours.day"
          }
        }
      }
    }
  }
}'
```
</details>

✅ Build a `nested` query to find all restaurants that are open on a `monday` and group / count them by the opening hours.

<details>
<summary>Possible solution</summary>

This solution uses a bool based query with a `filter` clause to find all documents, then using a `terms` aggregation to group all restaurants by opening hours.

```bash
curl -H 'Content-Type: application/json' -X GET 'http://localhost:9200/nested_aggs/_search?pretty' -d '{
  "size": 0,
  "query": {
    "nested": {
      "path": "opening_hours",
      "query": {
        "bool": {
          "filter": {
            "term": {
              "opening_hours.day": {
                "value": "monday"
              }
            }
          }
        }
      }
    }
  },
  "aggs": {
    "open mondays": {
      "nested": {
        "path": "opening_hours"
      },
      "aggs": {
        "by time": {
          "terms": {
            "field": "opening_hours.time"
          }
        }
      }
    }
  }
}'
```
</details>
