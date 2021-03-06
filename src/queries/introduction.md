# Introduction


## Mapping

In order to provide full text search and other queries the data needs to be indexed. An **index mapping** (see [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html)) defines how documents should be indexed. A mapping is comparable to a database schema as it defines which fields are analyzed & indexed.

> Only indexed fields are searchable.


## Create Index

The mapping needs to be specified at index creation time. Changes to this mapping later requires that the documents are either indexed into a new index or by using the [Reindex API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html) to index documents from one index to another.

To create a new index with a `mapping` the following command can be executed in your terminal.

✅ Create a new index

```bash
curl -H 'Content-Type: application/json' -X PUT 'http://localhost:9200/hello_world' -d '{
  "mappings": {
    "properties": {
      "title": {
        "type": "keyword"
      }
    }
  }
}'
```

This uses the [Create Index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html) to create a new index with a mapping. When successful a new index named `hello_world` is created. This index is empty and does not contain any documents yet. The only defined field for now is `title` (of type [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html)).

> **Note** the command assumes there is an Elasticsearch instance accessible at [localhost:9200](http://localhost:9200).

The examples of this chapter use the Index API to create a new index with settings & mapping.

> **🔎** To fetch the mapping of an existing index use the [Mapping API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-get-mapping.html).
>
> ✅ Return the mapping of index `hello_world`:
> ```bash
> curl http://localhost:9200/hello_world/_mapping?pretty
> ```


## Adding Documents

Once the index is created no documents are in it yet. In order to see that queries are working we first need to fill the index with some data.
Instead of adding each document one by one the [Bulk API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html) is used. This allows us to send multple documents in one HTTP request instead of multiple separate calls.

With the given index mapping the following payload can be send to the Elasticsearch index `hello_world`.

✅ Bulk upload documents to Elasticsearch index `hello_world`

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200/hello_world/_bulk' -d '
{"index":{"_index":"hello_world"}}
{"title":"elasticsearch"}
{"index":{"_index":"hello_world"}}
{"title":"mappings are great"}
{"index":{"_index":"hello_world"}}
{"title":"hello world"}
'
```

each pair of lines define actions by using **newline delimited JSON** (NDJSON), where the first line specifies to index the document, while the second line contains the document to index. Therefore the `Content-Type` in the HTTP request is set to `application/x-ndjson`. Elasticsearch offers a REST API, therefore it's a `POST` request.

> **🔎** To send documents from a file use **curl**'s `--data-binary` option.


## Search API

Once the index is created and contains a set of indexed documents, the [Search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html) is used to find documents that match a given query.
The simplest query is a [Match All Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-all-query.html) that returns all documents in an index.

The following command demonstrates how to search for documents in the `hello_world` index.

✅ Run the search query against the Search endpoint of index `hello_world` in your terminal

```bash
curl -H 'Content-Type: application/json' -X GET 'http://localhost:9200/hello_world/_search?pretty' -d '{
  "query": {
    "match_all": {}
  }
}'
```

This commands sends a query as JSON payload to the `/hello_world/_search` API endpoint of the index `hello_world` (**note** the content type is set to `application/json`). The `query` field specifies the query type, in this case a `match_all` query.

> **❗️** Elasticsearch accepts a JSON payload when sending a `GET` request. In case the used HTTP library or framework does not support this capability the same request can be send via `POST`.

The output of the search query looks similar to the following:

```json
{
  "took" : 29,
  "timed_out" : false,
  "_shards" : {
    "total" : 1,
    "successful" : 1,
    "skipped" : 0,
    "failed" : 0
  },
  "hits" : {
    "total" : {
      "value" : 3,
      "relation" : "eq"
    },
    "max_score" : 1.0,
    "hits" : [
      {
        "_index" : "hello_world",
        "_type" : "_doc",
        "_id" : "021L03YBpqOAGplM8qYe",
        "_score" : 1.0,
        "_source" : {
          "title" : "elasticsearch"
        }
      },
      {
        "_index" : "hello_world",
        "_type" : "_doc",
        "_id" : "1G1L03YBpqOAGplM8qYi",
        "_score" : 1.0,
        "_source" : {
          "title" : "mappings are great"
        }
      },
      {
        "_index" : "hello_world",
        "_type" : "_doc",
        "_id" : "1W1L03YBpqOAGplM8qYi",
        "_score" : 1.0,
        "_source" : {
          "title" : "hello world"
        }
      }
    ]
  }
}
```

The JSON response contains some meta information, such as how long the query took, how many shards were requested. The `hits` block contains number of matched documents and the list of documents.


## Delete Index

For the examples given in the book it's useful to delete an existing index, for example to create a new index with an updated mapping. Check the [Delete Index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-delete-index.html) in the Elasticsearch documentation for more details.

✅ Delete the Elasticsearch index `hello_world`

```bash
curl -X DELETE 'http://localhost:9200/hello_world'
```

If successful the deletion operation is acknowledged with:

```json
{"acknowledged":true}
```
