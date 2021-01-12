# Completion Suggest

Elasticsearch provides a few specific data types to support suggesters. One of the newer field types is the `completion` type. Typically document fields that contain text for which the auto complete feature should be used, are good candidates for this type.
See the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html#completion-suggester).

In earlier versions of Elasticsearch a custom text `analyzer` was used to implement the auto complete feature. This is still available.

> **❗️** Using the `completion` field type in the index mapping is not working with the [highlighting](https://www.elastic.co/guide/en/elasticsearch/reference/master/highlighting.html) feature. This means auto completions **do not** highlight the matched part of the text.

## Index Mapping

Let's create a new index with a mapping that supports auto completion.

✅ Create new index `completion_test` with the following mapping

```bash
curl -X PUT 'http://localhost:9200/completion_test' -H 'Content-Type: application/json' -d '{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "completion": {
            "type": "completion"
          }
        }
      },
      "genre": {
        "type": "keyword"
      }
    }
  }
}'
```

This creates a new index & mapping with a **multi field** (see [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-fields.html)).

* `title` with field type [text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html)
* `title.completion` is a multi field entry using field type [completion](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html#completion-suggester)


## Add Documents

Add a couple of documents to the index.

✅ Bulk upload documents to index `completion`

```bash
curl -H 'Content-Type: application/x-ndjson' -XPOST 'http://localhost:9200/completion_test/_bulk' -d '
{"index":{"_index":"completion_test"}}
{"title": "Harry Potter"}
{"index":{"_index":"completion_test"}}
{"title": "Star Trek"}
{"index":{"_index":"completion_test"}}
{"title": "Star Wars"}
{"index":{"_index":"completion_test"}}
{"title": "Star Lord"}
{"index":{"_index":"completion_test"}}
{"title": "The Incredibles"}
{"index":{"_index":"completion_test"}}
{"title": "The Favourite"}
{"index":{"_index":"completion_test"}}
{"title": "A Star is Born"}
'
```

## Example

The following query returns suggestions that are not necessarily part of the documents.

✅ Run the following search query to find documents

Can you guess which documents should match?

```bash
curl -H 'Content-Type: application/json' -X POST 'http://localhost:9200/completion_test/_search?pretty&human' -d '{
  "_source": ["title"],
  "suggest": {
    "text": "star",
    "title-suggest": {
      "completion": {
        "field": "title.completion"
      }
    }
  }
}'
```

> Try different search inputs, what works, what is not possible? What happens when you search for 'incredible'?

<details>
<summary>Explanation</summary>

There are a few constraints when using the `completion` field type.
For example it's a prefix based search that requires the match to start with the same search term

A search term of "star" will find "star wars" or "star trek" but not "a star is born". For shorter texts this is often a good choise, but to suggest terms the text field to not begin with, other implementations have to be used. One way to alleviate this issue is to index the text multiple time with decreasing number of terms.

For example given the text *Harry Potter and The Philosopher's Stone*, it can be indexed as:

```
Harry Potter and The Philosopher's Stone
Potter and The Philosopher's Stone
and The Philosopher's Stone
The Philosopher's Stone
Philosopher's Stone
Stone
```

Indexing term this way allows to search for inner terms.

As mentioned earlier one feature that is currently not supported with the `completion` field type is the `highlight` feature.
On the other hand using the `completion` field type allows to remove duplicates from the search results. Imagine having multiple documents with the same title, e.g. "Harry Potter", this allows to return only unique suggestions.
</details>
