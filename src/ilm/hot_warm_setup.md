# Hot / Warm Setup

In some setups the most recent data (e.g. logs) are more often queried / accessed / written than older data that is mostly read. One example for a hot / warm setup is when using daily Logstash logs.


## Setup

A hot / warm architecture requires the use of [Rack / Shard Allocation Awareness](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/modules-cluster.html#shard-allocation-awareness).

The Docker Compose environment described in the [Setup](../introduction/setup.md#docker-compose-recommended) chapter, defines three Elasticsearch nodes in the `docker-compose.yml`.

```bash
docker-compose up elasticsearch01 elasticsearch02 elasticsearch03 cerebro
```

Once all containers started, check the Elasticsearch cluster via [Cerebro](http://localhost:9000/#/overview?host=http:%2F%2Felasticsearch01:9200).


## Settings

Each container includes specific rack attributes that are used by Elasticsearch to distribute shards onto different nodes. The attribute `node.attr.data` is a custom named attribute (in this case `data`) we can use to attach labels / values to a node. This configuration setting is best configured in the `elasticsearch.yml`. The `docker-compose.yml` of the Docker Compose environment already assigned these settings to all Elasticsearch nodes.

* container `elasticsearch01` has value `hot`
* containers `elasticsearch02` and `elasticsearch03` have value `warm`

> in a bigger cluster more than one node should have an assigned attribute `hot`, especially if the indices have replicas or otherwise replica shards are not assigned to any node.
>
> The node values `hot` and `warm` have no inherent meaning to Elasticsearch.

One way to create indices on `hot` nodes is to specify the index routing allocation parameter in the index settings, see the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/modules-cluster.html). For example to create a new index (`hot-warm-index`) on a `hot` node add the `index.routing.allocation.require.data` property to the index settings.

✅ To create a new index named `hot-warm-index` on a `hot` node run the following command:

```bash
curl -X PUT 'http://localhost:9200/hot-warm-index' -H 'Content-Type: application/json' -d '{
  "settings": {
    "index.routing.allocation.require.data": "hot",
    "index.number_of_shards": 2,
    "index.number_of_replicas": 0
  }
}'
```

This creates a new index with two primary shards without a replica. Both shards should be located on the same `hot` node.

Alternatively this allocation setting can be given in an [index template](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-templates.html), all new indices that use this template automatically are created on the hot node. Once the index and its shards are not considered `hot` anymore, the index can be adjusted by moving them over to `warm` nodes, e.g. daily log indices after a few days.

Use the [Update Index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-update-settings.html) to change the allocation routing setting from `hot` to `warm`. This is an index setting that can be modified while the index already exists instead of its creation time.

To set the allocation attribute from `hot` to `warm`, update the settings of the index `hot-warm-index` with the following command:

✅ Update the allocation routing setting to `warm` for index `hot-warm-index`.

```bash
curl -X PUT 'http://localhost:9200/hot-warm-index/_settings' -H 'Content-Type: application/json' -d '{
  "index.routing.allocation.require.data": "warm"
}'
```

It may take a few minutes until the shards move from the `hot` node to a `warm` node due to allocation delay settings. All shards of this index should then have been moved to nodes with the attribute `warm`.

The same applies for any other phase that might be introduced, e.g. `cold`. A typical setup with log data may contain the following phases: `hot`, `warm`, `delete`. The `delete` phase then marks the end of the index, it is then deleted, e.g. after 30 days or any other interval. A [delete_index](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/delete_indices.html) action with the Elasticsearch Curator could handle this case.


## Curator

To automate the process of transferring shards from `hot` to `warm` or to delete existing indices, the Elasticsearch Curator CLI can be used. See the [Curator](./curator.md) chapter on how to set it up.

✅ Start the Curator and connect to it

```bash
docker-compose run curator /bin/ash
```

Inside the container there is the `/config` folder with the ES configuration file `config.yml`. The action file `index_rotate.yml` can also be found there and is used in this example.

To check the output of the Curator in a dry run, execute the following command in the curator container:

✅ Run curator in *dry mode*

```bash
curator --dry-run --config config.yml index_rotate.yml
```

This outputs the actions the Curator would process, but does not apply them yet.

In order to see the curator in action, create a couple of new logstash indices with prefix `logstash-` and a date. The current action configuration in `index_rotate.yml` keeps one daily `logstash-*` prefixed index on the `hot` node, thereafter these indices are moved to the `warm` nodes. There is another defined action (`2`) that uses the force merge action to merge the segments of indices older than 2 days. This optimizes the number of segments for each index:

For example execute the following commands to create indices `logstash-2020.11.24`, `logstash-2020.11.25`, `logstash-2020.11.26`:

> Create a few indices using the index pattern `logstash-YYYY.MM.DD`. **Note**, try to choose some current dates to see the effect.

✅ Create the following `logstash-*` prefixed indices

```bash
curl -X PUT 'http://localhost:9200/logstash-2020.11.26' -H 'Content-Type: application/json' -d '{
  "settings": {
    "index.routing.allocation.require.data": "hot",
    "index.number_of_shards": 2
  }
}'
curl -X PUT 'http://localhost:9200/logstash-2020.11.25' -H 'Content-Type: application/json' -d '{
  "settings": {
    "index.routing.allocation.require.data": "hot",
    "index.number_of_shards": 2
  }
}'
curl -X PUT 'http://localhost:9200/logstash-2020.11.24' -H 'Content-Type: application/json' -d '{
  "settings": {
    "index.routing.allocation.require.data": "hot",
    "index.number_of_shards": 2
  }
}'
```

Check the output of the Curator in *dry mode* again, it should not apply anything yet. Once the changes look good, run the curator again without dry run option.

✅ Run the Curator to apply all changes

```bash
/usr/local/bin/curator --config config.yml index_rotate.yml
```

This should move indices older than one day from `hot` to `warm`, moved indices also merge their segments to one segment.


## Resources:

* [Elasticsearch: How to enable shard allocation awareness](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-cluster.html#enabling-awareness)
* [Elasticsearch: Index Lifecycle Policies](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-index-lifecycle-management.html)
* [Hot Warm Architecture in Elastic Blog](https://www.elastic.co/blog/hot-warm-architecture-in-elasticsearch-5-x)
* [A Production Elasticsearch Curator Example](https://medium.com/imaginelearning/a-production-elasticsearch-curator-example-ea08a41131f3)
* [Elasticsearch Hot Warm Architecture](https://cinhtau.net/2017/06/14/hot-warm-architecture/)
* [Hot Warm Cold Architecture with Elasticsearch using ILM policy](https://ptran32.github.io/2020-08-08-hot-warm-cold-elasticsearch/)
