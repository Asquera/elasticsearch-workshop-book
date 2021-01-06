# Queries

This chapter describes the different query types used in search, completion, suggestions and aggregations. To follow these examples only a single Elasticsearch instance is required.

✅ Start Elasticsearch instance (see [Setup](./../introduction/setup.md))

## Query Types

There is an extensive list of query types, see [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html).

For the more common search queries a number of exercises can be found in this chapter

* [Match](./match.md)
* [Multi Match](./multi_match.md)
* [Term](./term.md.md)
* [Bool](./bool.md)
* [Query String](./query_string.md)


## Completion Queries & Suggesters

There are also a number of special search queries to help users navigate the document space. These are mostly applied when the user enters text into search input field or a text box, where multiple suggestions appear shortly after.

* [Completion](./completion.md) - using `completion` field type & `ngram` based analyzer
* [Did You Mean Suggest](./did_you_mean.md) - using `suggest` query
* [Search As You Type](./search_as_you_type.md) - using `search_as_you_type` field type & custom `analyzer`
* [Context Suggester](./context_suggestions.md) - using `completion` field type with context information


## Aggregations

* [Terms](./terms_aggs.md)
* [Filter](./filter_aggs.md)
* [Nested](./nested_aggs.md)