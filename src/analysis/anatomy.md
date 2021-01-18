# Anatomy of an Analyzer

Elasticsearch provides a long list of built-in analyzers (see [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html)) which can be used to index fields without further configuration.

The function of an analyzer is to analyze the incoming text, split it, transform it and index it.

In case the existing analyzers are insufficient a custom analyzer can be defined. This `custom` analyzer consists of the following components:

* `0` or more character filters ([reference](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-charfilters.html))
* one tokenizer ([reference](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html))
* `0` or more token filters ([reference](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenfilters.html))

## Example

Let's see how a custom analyzer is built and check what indexed text looks like.

✅ Start Elasticsearch instance

✅ Create a new index with the following mapping

```bash
curl -X PUT 'http://localhost:9200/test_custom_analyzer' -H 'Content-Type: application/json' -d '{
  "settings": {
    "analysis": {
      "analyzer": {
        "custom_analyzer": {
          "type": "custom",
          "char_filter": [
            "html_strip"
          ],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "asciifolding"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "custom_analyzer"
      }
    }
  }
}'
```

The `settings` block contains an analyzer named `custom_analyzer` that is defined as follows.

* its `type` is set to `custom` to tell Elasticsearch that it's not a built-in analyzer
* list of character filters in `char_filter` contains the entry `html_strip` (see [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-htmlstrip-charfilter.html)), which strips HTML characters from the input stream
* the analyzer uses the `standard` tokenizer (see [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-standard-tokenizer.html)) that splits the input text into tokens, e.g. to split text at word boundaries such as whitespace
* list of token filters in the `filter` block contains `lowercase` (see [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lowercase-tokenfilter.html)) and `asciifolding` (see [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-asciifolding-tokenfilter.html)) filters. The `lowercase` token filter changes the text into lower case text, while the `asciifolding` token filter converts characters that are not in Basic Latin Unicode block (first 127 ASCII chars) into their equivalent if it exists.

An analyzer processes input text as a stream of characters, given the following text `<html>Hello <b>World</b></html>` the process looks as follows.

Our input text is provided as a stream of characters

```text
"<","h","t","m","l",">","H","e","l","l","o"," ","<","b",">",
"W","o","r","l","d","<","/","b",">","<","/","h","t","m","l",">"
```

All character filters are applied, in this case the `html_strip` character filter receives the stream of characters, strips all HTML elements from the input string and returns the transformed character stream

```text
"H","e","l","l","o"," ","W","o","r","l","d"
```

<details>
<summary>See Analyze API</summary>

Check the character filters with the [Analyze API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html).

✅ Run the following analyze step with char filters only

```bash
curl -X GET "localhost:9200/test_custom_analyzer/_analyze?pretty" -H 'Content-Type: application/json' -d '{
  "char_filter" : ["html_strip"],
  "text" : "<html>Hello <b>World</b></html>"
}'
```

this outputs the following result:

```json
{
  "tokens" : [
    {
      "token" : "\nHello World\n",
      "start_offset" : 0,
      "end_offset" : 31,
      "type" : "word",
      "position" : 0
    }
  ]
}
```
</details>

> The list of given character filters is processed by the order in which they are given. The output stream from one filter is the input stream for the next until all character filters are processed.

The tokenizer receives the stream of characters, breaks it into individual tokensand returns the list of generated tokens. In this case the input stream is split at word boundaries (whitspaces).

```text
"Hello", "World"
```

<details>
<summary>See Analyze API</summary>

Check the tokenizer using the [Analyze API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html).

✅ Run the following analyze step to show tokens with tokenizer and char filters only

```bash
curl -X GET "localhost:9200/test_custom_analyzer/_analyze?pretty" -H 'Content-Type: application/json' -d '{
  "char_filter" : ["html_strip"],
  "tokenizer": "standard",
  "text" : "<html>Hello <b>World</b></html>"
}'
```

this outputs the following list of tokens:

```json
{
  "tokens" : [
    {
      "token" : "Hello",
      "start_offset" : 6,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "World",
      "start_offset" : 15,
      "end_offset" : 24,
      "type" : "<ALPHANUM>",
      "position" : 1
    }
  ]
}
```
</details>

The last processing step is to process the token stream with the token filters, in this case `lowercase` and `asciifolding`. The `lowercase` filter lowercases all incoming tokens, producing:

```text
"hello", "world"
```

The `asciifolding` token filter does not change any of the tokens, because all characters are given in Basic Latin Unicode. This token filter would drop all diacritical marks, e.g. transform 

<details>
<summary>See Analyze API</summary>

Check the `custom_analyzer` using the [Analyze API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html).

✅ Run the analyze step for the `custom_analyzer` including char filters, tokenizer, token filters.

```bash
curl -X GET "localhost:9200/test_custom_analyzer/_analyze?pretty" -H 'Content-Type: application/json' -d '{
  "analyzer" : "custom_analyzer",
  "text" : "<html>Hello <b>World</b></html>"
}'
```

this outputs the following list of tokens:

```json
{
  "tokens" : [
    {
      "token" : "hello",
      "start_offset" : 6,
      "end_offset" : 11,
      "type" : "<ALPHANUM>",
      "position" : 0
    },
    {
      "token" : "world",
      "start_offset" : 15,
      "end_offset" : 24,
      "type" : "<ALPHANUM>",
      "position" : 1
    }
  ]
}
```
</details>

> The output contains start, end offsets and positions. These reference the generated tokens with the original text input.
