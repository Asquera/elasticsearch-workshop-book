# Mapping

The mapping defines how the document is structured, the fields it contains, how it stores and indexes the data. It's similar to a database schema in that it defines the fields of a document.

For each field in the index mapping (either explicitely or implicitely) a field type is specified. The field type defines how the input is stored / indexed, e.g. `text`, `numeric` values, `boolean`, `date` etc.


## Field Types

Elasticsearch supports a number of different field data types. The full list can be seen in the [Elasticsearch reference](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html).

Common field types are:

* `binary`
* `boolean`
* `keywords`
* numeric types: `long`, `double`, `integer`, `byte`, etc.
* dates such as `date` or `date_nanos`
* `text`, also `search_as_you_type`, `completion`
* object & relational types such as `object`, `nested`, `flattened`
* spatial types such as `geo_point`, `geo_shape`, `point`, `shape`
* special types such as `ip`, `range`
* and more

> Arrays do not require a specific field type, a field of type `keyword` will accept a single entry, e.g. `"hello"` or an array of inputs, e.g. `["hello", "world"]`.

There is support to index the same field multiple times, for example to support different use cases, using `multi fields` (see [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-fields.html)).


## Create Mapping

Each index in Elasticsearch has a mapping, either explicitely given in when creating a new index or implicitely when Elasticsearch finds a new field for which no current mapping entry exists.

> Whenever Elasticsearch detects a new document field for which no mapping entry exists, it tries to determine a "good" fit for its field type.

In order to see how the mapping works, lets create a new index.

✅ Start Elasticsearch instance

To create a new with an index mapping the [Create Index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html) endpoint is used. This API endpoint is also used in most of the exercises / examples in this book. The JSON payload is sent as a string to the endpoint via a REST request.

✅ Create a new Elasticsearch with the following mapping

```bash
curl -X PUT "http://localhost:9200/twitter" -H 'Content-Type: application/json' -d '{
  "mappings": {
    "properties": {
      "user_id": {
        "type": "long"
      },
      "content": {
        "type": "text",
        "analyzer": "standard"
      },
      "tweeted_at": {
        "type": "date"
      }
    }
  }
}'
```

This creates a new index named `twitter` and defines the following fields inside the `mappings` block.

* `user_id` with field type [integer](https://www.elastic.co/guide/en/elasticsearch/reference/current/number.html)
* `content` with field type [text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html) using one of Elasticsearch's built in analyzers, the [standard](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-analyzer.html) analyzer
* `tweeted_at` with field type [date](https://www.elastic.co/guide/en/elasticsearch/reference/current/date.html)

> A document needs to match this mapping in order to get indexed successfully. Otherwise it will be rejected.

The document may contain other fields which are not currently declared in the mapping. Adding new fields to the mapping can be made via [Update Mapping API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-put-mapping.html) endpoint. This only works if the field is not already declared.

> Most mapping changes (except additions) require a new index with the new mapping. It's recommended to re-index the documents from the old to the new index to apply the mapping changes. The [Re-Index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html) endpoint can be used for this if only the mapping changed, but not the documents.

Let's add a new document to the index using the [Index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html).

✅ Upload the following document to index `twitter`

```bash
curl -X POST "http://localhost:9200/twitter/_doc?pretty&refresh=true" -H 'Content-Type: application/json' -d '{
  "user_id": 1,
  "content": "Hello to the workshop",
  "tweeted_at": "2021-01-20T10:11:15"
}'
```

This successfully indexes, analyzes and stores the document with its fields `user_id`, `content` and `tweeted_at`.
The response output may look like:

```json
{
  "_index" : "twitter",
  "_type" : "_doc",
  "_id" : "PMKvG3cBn-w2fsl7JlpX",
  "_version" : 1,
  "result" : "created",
  "forced_refresh" : true,
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 1,
  "_primary_term" : 1
}
```

Among other properties, the response contains the `_id` field, the document id of the new document. We can fetch a document by its `_id` using the [Get API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-get.html).

✅ Get the uploaded document by its doc id

```bash
curl -X GET 'http://localhost:9200/twitter/_doc/PMKvG3cBn-w2fsl7JlpX?pretty' -H 'Content-Type: application/json'
```

This returns the previous document.

> Analyzed and indexed data are stored in the Elasticsearch index, the document itself is stored on disk, accessible by its document id.

Now let's see what happens when Elasticsearch detects an unknown field

✅ Upload the following document to index `twitter` containing extra field

```bash
curl -X POST "http://localhost:9200/twitter/_doc?pretty&refresh=true" -H 'Content-Type: application/json' -d '{
  "user_id": 1,
  "content": "Hello to the workshop",
  "tweeted_at": "2021-01-20T10:11:15",
  "image_link": "www.example.com/funny.gif"
}'
```

This document gets indexed successfully. Let's see the index mapping via [Get Mapping API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-get-mapping.html).

✅ Get index mapping of index `twitter`

```bash
curl -X GET 'http://localhost:9200/twitter/_mapping?pretty' -H 'Content-Type: application/json'
```

This time the mapping contains 4 fields, the extra field `image_link` from the last document with field types.

```json
{
  "twitter" : {
    "mappings" : {
      "properties" : {
        ...
        "image_link" : {
          "type" : "text",
          "fields" : {
            "keyword" : {
              "type" : "keyword",
              "ignore_above" : 256
            }
          }
        }
      }
    }
  }
}
```

The field `image_link` defines two types, one using field type `text` while the multi field `image_link.keyword` uses field type `keyword`. This is the default assigned field type for unknown text field that Elasticsearch sets.

> Elasticsearch tries to find "best" suitable field type for the detected input.

Last but not least let's try to add another document that does not conform to the current mapping.

✅ Upload document with the following properties to index `twitter`

```bash
curl -X POST "http://localhost:9200/twitter/_doc?pretty&refresh=true" -H 'Content-Type: application/json' -d '{
  "user_id": "1",
  "content": "Hello to the workshop",
  "tweeted_at": "yesterday"
}'
```

This document gets rejected, Elasticsearch responds with an error message that provides the reason it rejected the document.

```
(excerpt)

"failed to parse date field [yesterday] with format [strict_date_optional_time||epoch_millis]"
```

