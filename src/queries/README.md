# Search Queries

Elasticsearch provides a full [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html) (Domain Specific Language) to define search queries. There is an exhaustive list of available query types. 

> **Note**, it's not necessarily feasible to know all of the different query types, more importantly it's better to learn what query capabilities Elasticsearch has to offer and focus on your use cases.


✅ Start Elasticsearch instance (see [Setup](./../introduction/setup.md)).


## Mapping

In order to provide full text search and other queries the data needs to be indexed. An index [mapping](https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping.html) defines how documents should be indexed. A mapping is comparable to a database schema as it defines which fields are analyzed / indexed. **Only indexed fields are searchable.**


### Create Index

The mapping needs to be given at index creation time. Changes to this mapping later requires that the documents are either indexed into a new index or by using [Reindex API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html) to index documents from one index to another.

To create a new index with a `mapping` the following command can be executed in your terminal.

✅ Create a new index

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

This uses the [Index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-index_.html) to create a new index with a mapping. When successful a new index named `hello_world` is created. This index is empty and does not contain any documents yet. The only defined field for now is `title` (of type [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html)).

> **Note** the command assumes there is an Elasticsearch instance accessible at [localhost:9200](http://localhost:9200).

The examples of this chapter use the Index API to create a new index with settings & mapping.

> **🔎** To fetch the mapping of an existing index use the [Mapping API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-get-mapping.html).
>
> ✅ Return the mapping of index `hello_world`:
> ```bash
> curl http://localhost:9200/hello_world/_mapping?pretty
> ```


### Adding data

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

> **🔎** To send documents from a file use curl's `--data-binary` option.


## Search API

Once the index is created and contains a set of indexed documents, the [Search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html) is used to find documents that match a given query.

The simplest query is a [Match All Query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-all-query.html) that simply returns all documents in an index.

The following command demonstrates how to search for documents in the `hello_world` index. Enter the command in your terminal:

✅ Run the search query against the Search endpoint of index `hello_world`

```bash
curl -H 'Content-Type: application/json' -X GET http://localhost:9200/hello_world/_search -d '{
  "query": {
    "match_all": {}
  }
}'
```

This commands sends a query as JSON payload to the `/hello_world/_search` API endpoint of the index `hello_world` (**note** the content type is set to `application/json`). The `query` field specifies the query type, in this case a `match_all` query.

> **❗️** Elasticsearch accepts a JSON payload when sending a `GET` request. In case the used HTTP library or framework does not provide this capability the same request can be send via `POST`.


