# Search As You Type

Elasticsearch offers a new field type (`search_as_you_type`) since version 7.2 for text like fields that is optimized for *search as you type* completion queries. This field automatically generates a few sub fields to index terms using different analyzers. In previous Elasticsearch versions custom `text` analyzers (using edge n-gram tokenizers) had to be used to support the same type of completion search.

> This type of field is used in auto completion queries. A user starts to type into a search text box and a number of suggested search terms are displayed. The use case for this type of query is navigation, the user can learn what the system (e.g. a website or webshop) has to offer.


## Index Mapping

For example the following index mapping sets a text field with the `search-as-you-type` field type (see [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-as-you-type.html)).

✅ Create a new index named `search_as_you_type` with the following mapping.

```bash
curl -X PUT 'http://localhost:9200/search_as_you_type' -H 'Content-Type: application/json' -d '{
  "mappings": {
    "properties": {
      "title": {
        "type": "search_as_you_type",
        "fields": {
          "text": {
            "type": "text",
            "analyzer": "standard"
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

This mapping defines the following fields.

* `title` with field type [search_as_you_type](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-as-you-type.html)
* `title.text` with field type [text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html)
* `genre` with field type [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html)

> **❗️** Using the field `search_as_you_type` may generate a lot of indexed terms, depending on the length of the text in the document, therefore it's recommended to apply to only those fields which profit the most from an auto completion, e.g. `keyword`, `genre`, `title`, `tag`.


### 🔎 Explanation

> **🔎** The field type `search_as_you_type` generates a number of fields that can be used in queries to refine the completion search. In the example above other than the field `title` there are also multi fields `title._2gram`, `title._3gram` and `title._index_prefix` (by default).

Let's see how these fields analyze & index a given input text, given the movie title *"Harry Potter and the Goblet of Fire"*.

✅ Display the indexed terms for the `title` field using the Analyze API

```bash
curl -H 'Content-Type: application/json' -XPOST 'http://localhost:9200/search_as_you_type/_analyze?pretty' -d '{
  "field": "title",
  "text": "Harry Potter and the Goblet of Fire"
}'
```

This generates the following terms (without offsets or position)

```
"harry"
"potter"
"and"
"the"
"goblet"
"of"
"fire"
```

This is the normal output equivalent to the `standard` text analyer. Now let's check what the indexed terms look like for field `title._2gram`.

✅ Display the indexed terms for the `title._2gram` field using the Analyze API

```bash
curl -H 'Content-Type: application/json' -XPOST 'http://localhost:9200/search_as_you_type/_analyze?pretty' -d '{
  "field": "title._2gram",
  "text": "Harry Potter and the Goblet of Fire"
}'
```

The response returns now the following indexed terms

```
"harry potter"
"potter and"
"and the"
"the goblet"
"goblet of"
"of fire"
```

Analyzing the field `title._3gram` with the same input text results in:

```
"harry potter and"
"potter and the"
"and the goblet"
"the goblet of"
"goblet of fire"
```

Both generated fields `title._2gram` and `title._3gram` are using shingles internally to generate tuples of neighboring terms. The main purpose of these two fields is to match phrases. The [shingle token filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-shingle-tokenfilter.html) is part of the analyzer to index these tuples.

The last generated field is the `title._index_prefix` multi field.

✅ Display the indexed terms for the `title._index_prefix` field using the Analyze API

```bash
curl -H 'Content-Type: application/json' -XPOST 'http://localhost:9200/search_as_you_type/_analyze?pretty' -d '{
  "field": "title._index_prefix",
  "text": "Harry Potter and the Goblet of Fire"
}'
```

This generates a lot (85) of indexed tokens:

```
"h"
"ha"
"har"
"harr"
"harry"
"harry "
"harry p"
"harry po"
"harry pot"
"harry pott"
"harry potte"
"harry potter"
"harry potter "
"harry potter a"
"harry potter an"
"harry potter and"
"p"
"po"
"pot"
[...]
"of fire"
"of fire "
"f"
"fi"
"fir"
"fire"
"fire "
"fire  "
```

The analyzer of the `title._index_prefix` field wraps the analyzer of the `title._3gram` with an edge ngram token filter. An [edge n-gram token filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-edgengram-tokenfilter.html) generates a set of tokens that start at the word bounday to store differently sized prefixes. The main purpose is to find prefixes that match words. A prefix matches well against the given input of the user.

For example assume that the user is searching for the word *potter*. After writing the letter `p` a search as you type query could search in `title` for matches with this letter. In the given example this will match the title string *"harry potter and the goblet of fire"*, because the second word starts with letter `p`. On the other hand it will not match for example the `title` *"star wars"*.


## Add Documents

To learn how this query works let's add a few documents first.

✅ Bulk upload documents to index `search_as_you_type`

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200/search_as_you_type/_bulk' -d '
{"index":{"_index":"search_as_you_type"}}
{"title": "Star Trek Generations", "genre": "space action"}
{"index":{"_index":"search_as_you_type"}}
{"title": "Star Wars A New Hope", "genre": "space action"}
{"index":{"_index":"search_as_you_type"}}
{"title": "Spiderman Homecoming", "genre": "superhero"}
{"index":{"_index":"search_as_you_type"}}
{"title": "Spiderman Far From Home", "genre": "superhero"}
{"index":{"_index":"search_as_you_type"}}
{"title": "Rogue One: A Star Wars Story", "genre": "space action"}
{"index":{"_index":"search_as_you_type"}}
{"title": "A Star is Born", "genre": "biopic"}
{"index":{"_index":"search_as_you_type"}}
{"title": "Star Trek Into Darkness", "genre": "action"}
{"index":{"_index":"search_as_you_type"}}
{"title": "Third Star", "genre": "drama"}
{"index":{"_index":"search_as_you_type"}}
{"title": "Lego Star Wars", "genre": "kids"}
{"index":{"_index":"search_as_you_type"}}
{"title": "The Star Witness", "genre": "drama"}
'
```


## Exercise

Build a search query that finds documents with the following search terms. See the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-as-you-type.html) for more information.

✅ Build a query that searches in all title fields (`title`, `title._2gram`, `title._3gram`, `title._index_prefix`) to find results for input `star`

> There are multiple ways to build such a query, for example one is to use the [multi-match](./../queries/multi_match.md) query type, another is to use a [bool](./../queries/bool.md) query.

See how the results look if you search for `s` then `st`, `sta`, `star`. Are the results what you expect?

<details>
<summary>A solution using mulit-match</summary>

The following query uses the `multi-match` query type to search in all these fields.

```bash
curl -X POST 'http://localhost:9200/search_as_you_type/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "multi_match": {
      "query": "s",
      "type": "bool_prefix",
      "fields": [
        "title",
        "title._2gram",
        "title._3gram",
        "title._index_prefix"
      ]
    }
  }
}'
```
</details>

✅ Use the same query to search for `star w`

* Are the results what you would expect?
