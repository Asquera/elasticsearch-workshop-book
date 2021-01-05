# Index Lifecycle Management

Since Elasticsearch 7.x a number of new APIs are available to manage the index lifecycle ([ILM](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html)). These are specific tasks to help automate the lifecycle of indices, for example to delete older indices in a log based infrastructure, to automatically create snapshots or to rollover indices.

> **❗️** Please note some of the features are only available in Elasticsearch [XPack](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-xpack.html) that requires at least a **Basic** license. Depending on your use case XPack might not be available, e.g. AWS Elasticsearch Service.

In previous versions of Elasticsearch (6.x and older) one way to achieve similar index lifecycle tasks was to use the [Elasticsearch Curator](https://www.elastic.co/guide/en/elasticsearch/client/curator/5.8/index.html).

The [Curator](./curator.md) is still available and therefore can be used to manage Elasticsearch indices. The examples in this chapter use the Curator.
