# Hot / Warm Setup

In some setups the most recent data (e.g. logs) are more often queried / accessed / written than older data that is mostly read. One example for a hot / warm setup is when using daily Logstash logs.

References:

* See Elastic blog post https://www.elastic.co/blog/hot-warm-architecture-in-elasticsearch-5-x
* https://medium.com/imaginelearning/a-production-elasticsearch-curator-example-ea08a41131f3
* https://cinhtau.net/2017/06/14/hot-warm-architecture/
* Hot Warm Cold Architecture with Elasticsearch using ILM policy: https://ptran32.github.io/2020-08-08-hot-warm-cold-elasticsearch/
* How to enable shard allocation awareness: https://www.elastic.co/guide/en/elasticsearch/reference/7.x/modules-cluster.html#enabling-awareness
* Index Lifecycle Policies in Elasticsearch: https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-index-lifecycle-management.html


## Setup

This requires to use [Rack / Shard Allocation Awareness](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/modules-cluster.html#shard-allocation-awareness).

Use the `docker-compose.yml` from the `examples/curator` folder, that defines a 3 nodes Elasticsearch cluster. From the folder `examples/curator` start the Docker setup with:

```bash
docker-compose up
```

Once all containers started, check the Elasticsearch cluster via [Cerebro](http://localhost:9000/#/overview?host=http:%2F%2Felasticsearch01:9200).


## Settings

Each container includes specific rack attributes that are used by Elasticsearch to distribute shards onto different nodes. The attribute `node.attr.data` is a custom named attribute (in this case `data`) we can use it to attach labels / values to a node. This configuration setting is best configured in the `elasticsearch.yml`. The `docker-compose.yml` in the folder `examples/curator` already assigned these settings to all Elasticsearch nodes, one node has the `hot` attribute, all others are set to `warm`.

**Note** in a bigger cluster more than one node should have an assigned attribute `hot`, especially if the indices have replicas or otherwise replica shards are not assigned to any node.

**Note** the attribute values `hot` and `warm` have no inherent meaning to Elasticsearch

One way to create indices on `hot` nodes is to specify the index routing allocation parameter in the index settings. For example to create a new index (`hot-warm-index`) on a hot node add the `index.routing.allocation.require.data` property to the index settings.

For example to create a new index named `hot-warm-index` on a `hot` node run the following command:

```bash
curl -X PUT 'http://localhost:9200/hot-warm-index' -H 'Content-Type: application/json' -d '{
  "settings": {
    "index.routing.allocation.require.data": "hot",
    "index.number_of_shards": 2,
    "index.number_of_replicas": 0
  }
}'
```

This creates a new index with two primary shards. Both shards should be located on the same `hot` node.

Alternatively this allocation setting can be given in an [index template](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-templates.html), all new indices that use this template automatically are created on the hot node. Once the index & its shards are not considered `hot` anymore, the index can be adjusted by moving them over to `warm` nodes, e.g. daily log indices after 1 or several days.

Use the [Update Index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-update-settings.html) to change the allocation routing setting from `hot` to `warm`. This is an index setting that can be modified while the index already exists instead of its creation time.

To set the allocation attribute from `hot` to `warm`, update the settings of the index `hot-warm-index` with the following command:

```bash
curl -X PUT 'http://localhost:9200/hot-warm-index/_settings' -H 'Content-Type: application/json' -d '{
  "index.routing.allocation.require.data": "warm"
}'
```

It may take a few minutes until the shards move from the `hot` node to a `warm` node due to allocation delay settings. All shards of this index moved to nodes with the attribute `warm`.

The same applies for any other phase that might be introduced, e.g. `cold`. A typical setup with log data may contain the following phases: hot, warm, delete. The delete phase marks the end of the index, it is then deleted, e.g. after 30 days or any other interval. A [delete_index](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/delete_indices.html) action with the Elasticsearch Curator could handle this case.


## Curator

To automate the process of transferring shards from `hot` to `warm` or to delete existing indices, the Elasticsearch Curator CLI can be used. Use the Docker setup in folder `examples/curator`. You can find the [Readme here](../Readme.md).

Start the containers by running the command:

```bash
docker-compose up --build
```

Then start the Curator container by

```bash
docker-compose run --rm curator /bin/ash
```

This starts the container, connects to it and provides a shell. Inside the container there is the `/config` folder with the ES configuration file `config.yml`. The action file `index_rotate.yml` can also be found there and is used in this example.

To check the output of the Curator in a dry run, execute the following command in the curator container:

```bash
curator --dry-run --config config.yml index_rotate.yml
```

This executes the curator in dry mode, it only displays the actions it would take, but does not apply them yet.

In order to see the curator in action, create a couple of new logstash indices with prefix `logstash-` and a date. The current action configuration in `index_rotate.yml` keeps one daily `logstash-*` prefixed index on the `hot` node, thereafter these indices are moved to the `warm` nodes. There is another defined action (`2`) that uses the force merge action to merge the segments of indices older than 2 days. This optimizes the number of segments for each index:

First create a few indices using the index pattern `logstash-YYYY.MM.DD`. **Note**, try to choose some current dates to see the effect.

For example execute the following commands to create indices `logstash-2020.11.24`, `logstash-2020.11.25`, `logstash-2020.11.26`:

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

Run the curator command again, it should not apply anything yet. Once the changes look good, run the curator again without dry run option.

```bash
/usr/local/bin/curator --config config.yml index_rotate.yml
```

This should move indices older than one day from `hot` to `warm`, moved indices also merge their segments to one segment.
