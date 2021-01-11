# Did You Mean

Another suggestion type is the *did you mean* feature, most notably known by using Google's search. Similar to the other suggestion features it's useful to help navigate the user to more relevant search results. This feature is often used to show alternatives, where the user may have entered a misspelled text to search for. For example instead of showing an empty list of search results, the user may see the suggestion and click on it to execute the same search again, with the adjusted search term.


## Index Mapping

Let's create a new index with a mapping that provides did you mean suggestions for the `title` field.

✅ Create new index `did_you_mean` with the following mapping

```bash
curl -X PUT 'http://localhost:9200/did_you_mean' -H 'Content-Type: application/json' -d '{
  "settings": {
    "index": {
      "analysis": {
        "filter": {
          "shingle_filter": {
            "type": "shingle",
            "min_shingle_size": 2,
            "max_shingle_size": 3
          }
        },
        "analyzer": {
          "did_you_mean_analyzer": {
            "type": "custom",
            "tokenizer": "whitespace",
            "filter": [
              "lowercase",
              "shingle_filter"
            ]
          }
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "did_you_mean": {
            "type": "text",
            "analyzer": "did_you_mean_analyzer"
          }
        }
      }
    }
  }
}'
```

This creates a new index named `did_you_mean` with a mapping using a **multi field** (see [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/multi-fields.html)).

* `title` with field type [text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html)
* `title.did_you_mean` is a multi field using a [custom analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-custom-analyzer.html) to index input text.

The custom analyzer named `did_you_mean_analyzer` indexes terms as follows. For the example title text *"Star Trek Into Darkness"* it generates the following tokens.

✅ Run the analyze query to see the generated tokens.

```bash
curl -X GET "localhost:9200/did_you_mean/_analyze?pretty" -H 'Content-Type: application/json' -d '{
  "field" : "title.did_you_mean",
  "text" : ["Star Trek Into Darkness"]
}'
```

This returns the following list of tokens:

```
"star"
"star trek"
"star trek into"
"trek"
"trek into"
"trek into darkness"
"into"
"into darkness"
"darkness"
```

The analyzer indexes single words or groups of 2 or 3 words.


## Add Documents

Let's fill the index with some documents first.

✅ Bulk upload documents to index `did_you_mean`

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200/did_you_mean/_bulk' -d '
{"index":{"_index":"did_you_mean"}}
{"title": "Star Trek Generations"}
{"index":{"_index":"did_you_mean"}}
{"title": "Star Wars A New Hope"}
{"index":{"_index":"did_you_mean"}}
{"title": "Spiderman Homecoming"}
{"index":{"_index":"did_you_mean"}}
{"title": "Spiderman Far From Home"}
{"index":{"_index":"did_you_mean"}}
{"title": "Rogue One: A Star Wars Story"}
{"index":{"_index":"did_you_mean"}}
{"title": "A Star is Born"}
{"index":{"_index":"did_you_mean"}}
{"title": "Star Trek Into Darkness"}
{"index":{"_index":"did_you_mean"}}
{"title": "Third Star"}
{"index":{"_index":"did_you_mean"}}
{"title": "Lego Star Wars"}
{"index":{"_index":"did_you_mean"}}
{"title": "The Star Witness"}
'
```


## Query & Exercise

The basic `suggest` query looks as follows. See the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html) on suggesters to learn more.

✅ Run the next query with `starr` and then `starr warrs`.

```bash
curl -H 'Content-Type: application/json' -X POST 'http://localhost:9200/did_you_mean/_search?pretty&human' -d '{
  "suggest": {
    "text": "starr",
    "did_you_mean": {
      "phrase": {
        "field": "title.did_you_mean",
        "size": 4,
        "max_errors": 2
      }
    }
  }
}'
```

Both queries (`starr` and `starr warrs`) should find matches.

> Do the results match what you would expect?

Some suggestions do not make much sense, for example when searching for input text *"starr warrs"* the returned results contain entries which do not exist this way in the documents.

✅ Build a query that uses the `collate` property to remove non existing suggestions.

<details>
<summary>A solution using collate</summary>

The following search request uses the `suggest` query with the `collate` field. It also shows how to use the `highlight` property to add pre and post tags to the match in the document field.

```bash
curl -X POST 'http://localhost:9200/did_you_mean/_search?pretty' -H 'Content-Type: application/json' -d '{
  "suggest": {
    "text": "starr warrs",
    "did_you_mean": {
      "phrase": {
        "field": "title.did_you_mean",
        "size": 4,
        "max_errors": 2,
        "highlight": {
          "pre_tag": "<em>",
          "post_tag": "</em>"
        },
        "collate": {
          "query": {
            "source": {
              "match_phrase": {
                "{{field_name}}": "{{suggestion}}"
              }
            }
          },
          "params": { "field_name": "title.did_you_mean" },
          "prune": false
        }
      }
    }
  }
}'
```

The `collate` block defines a `match_phrase` query to check our suggestion in field `title.did_you_mean`. The block makes use of template variables `field_name` and `suggestion`, the former is defined as a parameter in `params`, the latter is automatically provided by Elasticsearch.
</details>
