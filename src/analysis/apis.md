# API endpoints

There are a number of API endpoints Elasticsearch provides to set up index mappings, test analyzers or explain queries. Here is a list of useful API endpoints that are helpful when creating your own index mappings with analyzers.

These APIs assume there is an Elasticsearch instance running at [localhost:9200](http://localhost:9200).

## Analyze API

The [Analyze API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html) can be used to test how Elasticsearch (or Lucene) analyzes text strings into tokens.

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
