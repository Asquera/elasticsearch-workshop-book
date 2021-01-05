# Search Queries

Elasticsearch provides a full [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html) (Domain Specific Language) to define search queries. There is an exhaustive list of available query types.

> **Note**, it's not necessarily feasible to know all of the different query types, more importantly it's better to learn what query capabilities Elasticsearch has to offer and focus on your use cases.


## Mapping

In order to provide full text search and other queries the data needs to be indexed. An index mapping defines how documents should be indexed. A mapping is similar to a database schema as it defines which fields are analyzed / indexed. Only indexed fields are searchable.


### Create Index

The mapping needs to be given at index creation time. Changes to this mapping later requires that the documents are either indexed into a new index or by using [Reindex API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html) to index documents from one index to another.

To create a new index with with a `mapping` the following command can be executed in your terminal:

```bash
curl -X PUT 'http://localhost:9200/hello_world' -d '{
  "mappings": {
    "properties": {
      "title": {
        "type": "keyword"
      }
    }
  }
}'
```

This uses the [Index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html) to create a new index with a mapping. When successful a new index named `hello_world` is created. This index is empty and does not contain any documents yet. The only defined field for now is `title`.

> **Note** the command assumes there is an Elasticsearch instance accessible at [localhost:9200](http://localhost:9200).

The examples of this chapter use the Index API to create a new index with settings & mapping.


### Adding data

Once the index is created no documents are in it yet. In order to see that queries are working we first need to fill the index with some data.
Instead of adding each document one by one the [Bulk API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html) is used. This allows us to send multple documents in one HTTP request instead of multiple separate calls.

With the given index mapping the following payload can be send to the Elasticsearch index `hello_world`.

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

> **🔎** To send documents from a file use curl's `--data-binary` option.
