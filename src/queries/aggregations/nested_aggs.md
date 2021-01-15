# Nested Aggregation

Documents can contain nested properties by using the `nested` field type (see [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/nested.html)). The `nested` field type is useful when there is a relation between the main document and inner documents. For example a restaurant may contain the opening hours as an inner object or a blog post may contain comments.

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
When indexing this document structure **without** a `nested` field, the result is that these fields are stored as flat arrays separately.

A search request using this mapping structure cannot easily answer the question on which days the restaurant is open between `10-14` hours.
For example a mapping without the `nested` field type may look like the following mapping.

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

An attempt to find all documents where `time` is `10-18` **and** `day` contains `monday` may look like:

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

With the given index mapping above, this query will return all documents where any entry of `opening_hours.day` contains value `monday` and where `opening_hours.time` contains the value `10-18`. This does not reduce the results to these documents only that match both values for the same entry of `opening_hours`.

> **🔎** The `nested` field type groups properties as inner documents. Elastiscearch indexes these documents separately.

Once the index mapping makes use of the `nested` field the search request can search for inner documents by using a `nested` sub query.

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

This time the query only returns the restaurants (documents) that match values in the same entry of `opening_hours`.


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
{"title":"Restaurant at the end of the universe", "opening_hours":[{"day":"monday","time":"10-18"},{"day":"wednesday","time":"10-15"}, {"day":"friday","time":"10-15"}]}
{"index":{"_index":"nested_aggs"}}
{"title":"Djimalaya", "opening_hours":[{"day":"monday","time": "10-18"}, {"day":"wednesday","time":"10-18"}, {"day":"friday","time":"10-18"}]}
{"index":{"_index":"nested_aggs"}}
{"title":"Kebap", "opening_hours":[{"day":"sunday","time": "10-14"}]}
{"index":{"_index":"nested_aggs"}}
{"title":"Curry 36", "opening_hours":[{"day":"monday","time": "10-22"},{"day":"tuesday","time": "10-14"},{"day":"thursday","time": "10-18"}]}
'
```


## Exercise

A nested aggregation is a bucket aggregation that calculates / summarizes data on nested documents.
It works on nested fields and requires a `path` to the nested object. See the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-bucket-nested-aggregation.html) on nested aggregations for more details.
The `nested` path tells Elasticsearch where the inner documents reside.

✅ Build a `nested` aggregation to group / count all restaurants by their `opening_hours.day`.

<details>
<summary>Possible solution</summary>

This solution uses a `nested` aggregation.

```bash
curl -H 'Content-Type: application/json' -X GET 'http://localhost:9200/nested_aggs/_search?pretty' -d '{
  "size": 0,
  "aggs": {
    "open restaurants": {
      "nested": { "path": "opening_hours" },
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

✅ Build a `nested` aggregation to find all restaurants that are open on a `monday` and group the available opening hours for these restaurants.

> Building this aggregation combines a few aggregations. Instead of using `query` block it is recommended to use a `nested` aggregation with an inner `filter` aggregation. The main difference between the `query` block and the aggregations is that the query block returns the full documents, while the `filter` aggregation can be used to work on the inner `opening_hours` entries.

<details>
<summary>Possible solution</summary>

This query below uses a `nested` aggregation with an inner `filter` aggregation to find all documents that have `opening_hours.day` set to `monday`, groups all hits using a `terms` aggregation. The result is the list of opening hours slots of all restaurants open on a Monday.

```bash
curl -H 'Content-Type: application/json' -X GET 'http://localhost:9200/nested_aggs/_search?pretty' -d '{
  "size": 0,
  "aggs": {
    "restaurants": {
      "nested": {
        "path": "opening_hours"
      },
      "aggs": {
        "monday": {
          "filter": {
            "term": {
              "opening_hours.day": "monday"
            }
          },
          "aggs": {
            "open hours": {
              "terms": {
                "field": "opening_hours.time"
              }
            }
          }
        }
      }
    }
  }
}'
```
</details>

Do you see the results you would expect?
