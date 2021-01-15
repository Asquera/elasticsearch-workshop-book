# Percentiles Aggregation

One multi-value metrics aggregation is the `percentiles` aggregation (see the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-percentile-aggregation.html)).
This aggregation is useful to show a distribution of values that do not follow a normal distribution. It's useful when the distribution follows a power law distribution. A practical example is finding the slowest response times of a web service, in particular to find response times above 95%, 99% etc. For this type of distribution the normal statistics do not offer useful insights, an average response time of `50ms` may hide the fact that there are outliers above `2.000ms`.


## Index Mapping

Let's build a new index with the following mapping that could be used to store logs from a web service.

✅ Create a new index named `percentiles_aggs` with the following mapping.

```bash
curl -X PUT 'http://localhost:9200/percentiles_aggs' -H 'Content-Type: application/json' -d '{
  "mappings": {
    "properties": {
      "response_time_in_ms": {
        "type": "integer"
      },
      "status_code": {
        "type": "keyword"
      },
      "url": {
        "type": "text"
      }
    }
  }
}'
```

The mapping contains the following fields:

* `url` using the field type [text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html)
* `status_code` using field type [keyword](https://www.elastic.co/guide/en/elasticsearch/reference/current/keyword.html)
* `response_time_in_ms` using field type [integer](https://www.elastic.co/guide/en/elasticsearch/reference/current/number.html)


## Add Documents

Let's add a few documents that represent web service responses (same as in the [stats aggregation](./stats_aggs.md) example).

✅ Bulk upload documents to index `percentiles_aggs`

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200/percentiles_aggs/_bulk' -d '
{"index":{"_index":"percentiles_aggs"}}
{"url": "http://example.com/1", "status_code": 200, "response_time_in_ms": 50 }
{"index":{"_index":"percentiles_aggs"}}
{"url": "http://example.com/1", "status_code": 200, "response_time_in_ms": 25 }
{"index":{"_index":"percentiles_aggs"}}
{"url": "http://example.com/1", "status_code": 200, "response_time_in_ms": 30 }
{"index":{"_index":"percentiles_aggs"}}
{"url": "http://example.com/1", "status_code": 200, "response_time_in_ms": 100 }
{"index":{"_index":"percentiles_aggs"}}
{"url": "http://example.com/1", "status_code": 200, "response_time_in_ms": 5 }
{"index":{"_index":"percentiles_aggs"}}
{"url": "http://example.com/1", "status_code": 200, "response_time_in_ms": 15 }
{"index":{"_index":"percentiles_aggs"}}
{"url": "http://example.com/1", "status_code": 200, "response_time_in_ms": 18 }
{"index":{"_index":"percentiles_aggs"}}
{"url": "http://example.com/1", "status_code": 200, "response_time_in_ms": 25 }
{"index":{"_index":"percentiles_aggs"}}
{"url": "http://example.com/redirect", "status_code": 302, "response_time_in_ms": 25 }
{"index":{"_index":"percentiles_aggs"}}
{"url": "http://example.com/2", "status_code": 201, "response_time_in_ms": 25 }
{"index":{"_index":"percentiles_aggs"}}
{"url": "http://example.com/2", "status_code": 201, "response_time_in_ms": 35 }
{"index":{"_index":"percentiles_aggs"}}
{"url": "http://example.com/2", "status_code": 201, "response_time_in_ms": 12 }
{"index":{"_index":"percentiles_aggs"}}
{"url": "http://example.com/2", "status_code": 500 }
{"index":{"_index":"percentiles_aggs"}}
{"url": "http://example.com/2", "status_code": 500 }
'
```


## Exercise

See the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/search-aggregations-metrics-percentile-aggregation.html) on `percentiles` aggregation.

✅ Build a `percentiles` aggregation query on field `response_time_in_ms`.

<details>
<summary>A solution</summary>

The following query uses a `percentiles` aggregation named `response_times`.

```bash
curl -X POST 'http://localhost:9200/stats_aggs/_search?pretty' -H 'Content-Type: application/json' -d '{
  "size": 0,
  "aggs": {
    "response_times": {
      "percentiles": {
        "field": "response_time_in_ms"
      }
    }
  }
}'
```

The output looks like this:

```json
{
  ...
  "aggregations" : {
    "response_times" : {
      "values" : {
        "1.0" : 5.0,
        "5.0" : 5.700000000000001,
        "25.0" : 16.5,
        "50.0" : 25.0,
        "75.0" : 32.5,
        "95.0" : 94.99999999999993,
        "99.0" : 100.0
      }
    }
  }
}
```

</details>

How do you interpret these values?

> The aggregation supports the field `percents` to provide a list of float values the distribution is grouped into, e.g. `[25, 50, 95, 99, 99.9]`.

