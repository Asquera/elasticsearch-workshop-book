# API endpoints

There are a huge number of API endpoints Elasticsearch provides (see [list of REST APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/rest-apis.html)) to search documents, set up index mappings, test analyzers or explain queries. Here is a list of useful API endpoints that are helpful when creating your own index mappings with analyzers.

These APIs assume there is an Elasticsearch instance running at [localhost:9200](http://localhost:9200).


## Search API

The [Search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-search.html) endpoint is available for every index, it's the main endpoint of all search related APIs (see [overview of Search APIs](https://www.elastic.co/guide/en/elasticsearch/reference/current/search.html)).
This endpoint is used whenever a search request is sent to an Elasticsearch Index.

For example assuming there is an index named `test`, a basic search request using the `/test/_search` endpoint looks as follows:

```bash
curl -X GET "localhost:9200/test/_search" -H 'Content-Type: application/json' -d '{}'
```

When successful the response contains the search results and meta data. Examples in the [Query chapter](../queries/index.md) use the Search API endpoint.


## Analyze API

The [Analyze API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html) can be used to test how Elasticsearch (or Lucene) analyzes text strings to return a list of tokens.

For example to test the output of the `standard` analyzer via **curl** the general `/_analyze` endpoint can be used.

✅ Run the following query against the Analyze API.

```bash
curl -X GET "localhost:9200/_analyze?pretty" -H 'Content-Type: application/json' -d '{
  "analyzer" : "standard",
  "text" : ["Bart Simpsons is a comic character"]
}'
```

The `/_analyze` API endpoint returns the following list of tokens:

```txt
["bart", "simpsons", "is", "a", "comic", "character"]
```

All tokens are lower cased.

> **💡 Tip:** the `?pretty` query argument in the URL above returns more human friendly formatted output.

There is more information returned, e.g. offsets, position, the complete output looks like:

```json
{
  "tokens" : [
    {
      "token" : "bart",
      "start_offset" : 0,
      "end_offset" : 4,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "simpsons",
      "start_offset" : 5,
      "end_offset" : 13,
      "type" : "<ALPHANUM>",
      "position" : 1
    },
    {
      "token" : "is",
      "start_offset" : 14,
      "end_offset" : 16,
      "type" : "<ALPHANUM>",
      "position" : 2
    },
    {
      "token" : "a",
      "start_offset" : 17,
      "end_offset" : 18,
      "type" : "<ALPHANUM>",
      "position" : 3
    },
    {
      "token" : "comic",
      "start_offset" : 19,
      "end_offset" : 24,
      "type" : "<ALPHANUM>",
      "position" : 4
    },
    {
      "token" : "character",
      "start_offset" : 25,
      "end_offset" : 34,
      "type" : "<ALPHANUM>",
      "position" : 5
    }
  ]
}
```


## Cat APIs

Elasticsearch offers a list of API endpoints to return **compact** and **aligned text** (CAT) formatted output. Here is a list of a few endpoints that help to get information on cluster, nodes, indices, aliases etc.

See [cat health API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-health.html) documentation.

✅ Display the health of the Elasticsearch cluster

```bash
curl -X GET 'http://localhost:9200/_cat/health?v'
```

<hr>

See [cat nodes API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-nodes.html) documentation.

✅ List all nodes in the Elasticsearch cluster

```bash
curl -X GET 'http://localhost:9200/_cat/nodes?v'
```

This command outputs the list of all available nodes with some basic metrics on heap, memory, cpu, load.

<hr>

See [cat indices API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-indices.html) documentation.

✅ List all indices from all nodes

```bash
curl -X GET 'http://localhost:9200/_cat/indices?v'
```

or to output information for a specific inddex

```bash
curl -X GET 'http://localhost:9200/_cat/indices/<index>?v'
```

<hr>

See [cat shards API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-shards.html)

✅ List all shards of all indices

```bash
curl -X GET 'http://localhost:9200/_cat/shards?v'
```
