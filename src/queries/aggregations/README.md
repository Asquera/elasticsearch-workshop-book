# Aggregations

Elasticsearch provides aggregations (formerly known as *facets*) to summarize data as metrics or analytics. It is used to group data, perform calculations on them, e.g. sums, percentiles, histograms, etc. Aggregations are grouped into three categories.

* Bucket ggregations to group data into buckets
* Metrics ggregations to compute metrics over data
* Pipeline ggregations to calculate data on the output from previous aggregations

✅ Start Elasticsearch instance (see [Setup](./../introduction/setup.md))

The basic structure of an aggregation is:

```json
{
  "aggs": {
    "<label>": {
      "<type-of-aggregation>": { ... }
    }
  }
}
```

The aggregation is sent to the same Search API endpoint as the previous queries. Both `aggs` and `query` can complement each other, for example a search request can have a `query` block to filter and/or match specific documents while the aggregations in `aggs` use the resulting documents to group them or calculate metrics on them. The response format looks like (omitting some fields)

```json
{
  ...
  "aggregations": {
    "<label>": {
      "buckets": [
        {
          "key": "<some-key>",
          "doc_count": 3
        },
        {
          "key": "<next-key>",
          "doc_count": 2
        }
        ...
      ]
    }
  }
}
```

The entries in the `buckets` field are sorted by field `doc_count`, the number of documents that match the same term.


## Examples

* [Terms](./terms_aggs.md) (Bucket)
* [Filter](./filter_aggs.md) (Bucket)
* [Nested](./nested_aggs.md) (Bucket)
* [Stats](./stats_aggs.md) (Metric)
* [Percentiles](./percentiles_aggs.md) (Metric)
