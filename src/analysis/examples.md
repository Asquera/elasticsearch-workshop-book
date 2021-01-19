# Analyzer Examples

## Top Level Domain in URL

A custom analyzer can be used to analyze specific structured texts. This example shows how to determine the [top level domain](https://en.wikipedia.org/wiki/Top-level_domain) part of a given URL. For example having the URL "https://www.heise.**de**/preisvergleich" extracts the tld **"de"** from it.

For this example the [pattern_replace](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pattern-replace-charfilter.html) character filter and the [path_hierarchy](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pathhierarchy-tokenizer.html) tokenizer are used.

The idea is to strip all characters / tokens which are not part of the domain name, then only pick the last part of the domain which is the top level doman. This may or may not work for all valid URLs, but for the purpose of this example we assume that works well enough.

✅ Create a new index with the given settings

```bash
curl -X PUT 'http://localhost:9200/test_tld_analyzer' -H 'Content-Type: application/json' -d '{
  "settings": {
    "analysis": {
      "char_filter": {
        "remove_url_scheme_filter": {
          "type": "pattern_replace",
          "pattern": "http(s)://",
          "replacement": ""
        },
        "remove_url_path_filter": {
          "type": "pattern_replace",
          "pattern": "(.*)/(.*)",
          "replacement": "$1"
        }
      },
      "tokenizer": {
        "tld_tokenizer": {
          "type": "pattern",
          "pattern": "^.*\\.([^.]*)$",
          "group": 1
        }
      }
    }
  }
}'
```

This sets up a new index with the custom analyzer `tld_analyzer`. We use the [Analyze API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html) to check the generated tokens, therefore no mapping is provided.

Let's check the different parts of the analyzer. For this we will provide the URL `https://www.example.com/hello_world` for analysis. First let's see what the character filter `remove_url_scheme_filter` does. It uses the `pattern_replace` type that allows to provide a regular pattern in the `pattern` field.

✅ Analyze the URL with the char filter

```bash
curl -X GET 'http://localhost:9200/test_tld_analyzer/_analyze?pretty' -H 'Content-Type: application/json' -d '{
  "char_filter": ["remove_url_scheme_filter"],
  "text": "https://www.example.com/hello_world"
}'
```

If everything works fine, this outputs the token `www.example.com/hello_world`, removing the URL scheme `http://`.

Next let's check the next character filter `remove_url_path_filter`.

✅ Analyze the URL with both character filters

```bash
curl -X GET 'http://localhost:9200/test_tld_analyzer/_analyze?pretty' -H 'Content-Type: application/json' -d '{
  "char_filter": ["remove_url_scheme_filter", "remove_url_path_filter"],
  "text": "https://www.example.com/hello_world"
}'
```

This should output the token `www.example.com` if successful. The `remove_url_path_filter` char filter removes the path part of the URL.

Remember the analyzer processes the input text stream in the following order: character filters, tokenizer, token filters. Our analyzer needs to specify a tokenizer. The input stream at this point already processed URL scheme and path, the reamining part is the full domain. The tokenizer is of type [pattern](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pattern-tokenizer.html) that matches the input stream with the regex pattern and if it matches takes the pattern group as output token. The regex pattern should only emit the characters after the last `.` (dot) character.

✅ Analyze the URL with character filters and the tokenizer

```bash
curl -X GET 'http://localhost:9200/test_tld_analyzer/_analyze?pretty' -H 'Content-Type: application/json' -d '{
  "char_filter": ["remove_url_scheme_filter", "remove_url_path_filter"],
  "tokenizer": "tld_tokenizer",
  "text": "https://www.example.com/hello_world"
}'
```

If successful this outputs the following tokens `com`, the token we are looking for, the top level domain `com`.

The last remaining part may be to lower case all URLs to have one representation for the top level domain.

✅ Analyze the URL with all character filters, tokenizers and token filters

```bash
curl -X GET 'http://localhost:9200/test_tld_analyzer/_analyze?pretty' -H 'Content-Type: application/json' -d '{
  "char_filter": ["remove_url_scheme_filter", "remove_url_path_filter"],
  "tokenizer": "tld_tokenizer",
  "filter": ["lowercase"],
  "text": "https://www.example.COM/hello_world"
}'
```

Voila, this should output the top level domain `com` as single token.

> Of course this is a somewhat contrived example to demonstrate the analysis process for some structured text data. For this particular example it probably makes more sense to prepare the data differently and only have the final token stored in a keyword field of the index mapping.


## Exercise

### File Paths

In this exercise we will have a number of file paths as text input (ignoring Windows vs Linux path separators). A few file path examples

```text
/User/alice/old-documents/important.pdf
/User/alice/new-documents/photo.jpeg
/User/bob/dev-repos/elasticsearch-docker/README.md
```

✅ Specifiy a `path_hierarchy` tokenizer in the `settings` block of the index and test it with the Analyze API

See the [Elasticsearch reference](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-pathhierarchy-tokenizer.html) for more information.

Is the output what you would expect?

✅ Use the same `path_hierarchy` tokenizer and use the `reverse` field

How is the output of this tokenizer different from the previous one? What is the differenece?

<details>
<summary>Possible solution</summary>

This mapping uses the `path_hierarchy` tokenizer, create an index with the settings first.

```bash
curl -X PUT 'http://localhost:9200/test_file_paths' -H 'Content-Type: application/json' -d '{
  "settings": {
    "analysis": {
      "tokenizer": {
        "filepath_tokenizer": {
          "type": "path_hierarchy",
          "delimiter": "/"
        },
        "filepath_tokenizer_reversed": {
          "type": "path_hierarchy",
          "delimiter": "/",
          "reverse": true
        }
      }
    }
  }
}'
```

To check the tokenizer we are using the Analyze API, first the `filepath_tokenizer`.

```bash
curl -X POST 'http://localhost:9200/test_file_paths/_analyze?pretty=true' -H 'Content-Type: application/json' -d '{
  "tokenizer": "filepath_tokenizer",
  "text": "/User/bob/dev-repos/elasticsearch-docker/README.md"
}'
```

Then the `filepath_tokenizer_reversed` tokenizer.

```bash
curl -X POST 'http://localhost:9200/test_file_paths/_analyze?pretty=true' -H 'Content-Type: application/json' -d '{
  "tokenizer": "filepath_tokenizer_reversed",
  "text": "/User/bob/dev-repos/elasticsearch-docker/README.md"
}'
```
</details>

In order to provide reasonable results, an index mapping could index the same field containing file paths using multi fields with different analyzers.

<details>
<summary>Complete with Mapping</summary>

Given the above mapping from the previous exercise using `path_hierarchy` tokenizers, there are two analyzers with different directions.

```bash
curl -X PUT 'http://localhost:9200/test_file_paths' -H 'Content-Type: application/json' -d '{
  "settings": {
    "analysis": {
      "tokenizer": {
        "filepath_tokenizer": {
          "type": "path_hierarchy",
          "delimiter": "/"
        },
        "filepath_tokenizer_reversed": {
          "type": "path_hierarchy",
          "delimiter": "/",
          "reverse": true
        }
      },
      "analyzer": {
        "filepath_analyzer": {
          "tokenizer": "filepath_tokenizer",
          "filter": ["lowercase"]
        },
        "filepath_analyzer_reversed": {
          "tokenizer": "filepath_tokenizer_reversed",
          "filter": ["lowercase"]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "filepath": {
        "type": "text",
        "analyzer": "filepath_analyzer",
        "fields": {
          "reverse": {
            "type": "text",
            "analyzer": "filepath_analyzer_reversed"
          }
        }
      }
    }
  }
}'
```

Then we can check the generated tokens for fields `filepath` and `filepath.reversed` to see that the analyzers work as intended.

```bash
curl -X POST 'http://localhost:9200/test_file_paths/_analyze?pretty=true' -H 'Content-Type: application/json' -d '{
  "field": "filepath",
  "text": "/User/bob/dev-repos/elasticsearch-docker/README.md"
}'
```
</details>
