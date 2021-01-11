# Completions & Suggesters

There are a number of special search queries to help users navigate the document space. These are mostly applied when the user enters text into search input field or a text box, where multiple suggestions appear shortly after.

* [Completion](./completion.md) - using `completion` field type & `ngram` based analyzer
* [Did You Mean Suggest](./did_you_mean.md) - using `suggest` query
* [Search As You Type](./search_as_you_type.md) - using `search_as_you_type` field type & custom `analyzer`
* [Context Suggester](./context_suggestions.md) - using `completion` field type with context information

> **❗️** Building a completion (or any text based) query may require balancing different expectations. In text based queries there are no absolute true answers. There may be subjectively good & bad results for a particular query. The goal of the developer / designer is to index the documents in a way that using a well written query matches most of the user expectations.


## Resources

* [Elasticsearch - Suggesters](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html)
* [Elasticsearch - Search As You Type](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-as-you-type.html)
* [Did You Mean: Elasticsearch suggestions?](https://dev.to/raoulmeyer/did-you-mean-elasticsearch-suggestions-5n1)
* [A detailed comparison between autocompletion strategies in ElasticSearch](https://medium.com/@mourjo_sen/a-detailed-comparison-between-autocompletion-strategies-in-elasticsearch-66cb9e9c62c4)
* [Elasticsearch: Building Autocomplete Functionality](https://hackernoon.com/elasticsearch-building-autocomplete-functionality-494fcf81a7cf)
