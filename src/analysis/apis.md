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
