# Rollover

An index rollover is an action that creates & switches to a new index when certain criteria are fulfilled, for example the index reached a specific index size, a maximum number of documents or a certain age. This is a good strategy when it's not necessary known upfront how much data will be stored in an index (e.g. logs) or how long the index will be relevant. This works best for indices that have write once documents, e.g. logs.

To make a rollover work it requires at least one alias set to the index that is currently written to. Indices for a rollover usually follow a naming scheme, for example by using a suffix to the index name, e.g. `logstash-000001` or similar.

The idea is that when specific criteria are met a new index is created and the previous index will become read only. The alias is then set to the new index, all write requests are then done with the new index.


## Setup

This example uses the Elasticsearch Curator to automate the rollover process. The Curator action uses the Rollover Index API (see [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/master/indices-rollover-index.html)).

The Docker Compose environment described in the [Setup](../introduction/setup.md#docker-compose-recommended) chapter, defines three Elasticsearch nodes in the `docker-compose.yml`.

```bash
docker-compose up elasticsearch01 curator cerebro
```

Once all containers started, check the Elasticsearch cluster via [Cerebro](http://localhost:9000/#/overview?host=http:%2F%2Felasticsearch01:9200).


## Index Mapping

To see how the rollover works, let's create a new index first named `rollover-000001`.

✅ Create a new index named `rollover-000001`.

```bash
curl -X PUT 'http://localhost:9200/rollover-000001' -H 'Content-Type: application/json' -d '{}'
```

In order to roll over the index internally an alias is used that points to the current rollver index. To create a new alias see [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-add-alias.html).

✅ Set alias named `rollover` that points to `rollover-000001`

```bash
curl -X PUT 'http://localhost:9200/rollover-000001/_alias/rollover' -H 'Content-Type: application/json' -d '{}'
```

This creates the alias `rollover`. Elasticsearch aliases act as index names, but provide a flexible mean to address different indices. An alias can point to multiple different indices at once. A good use case for an alias is to switch it from one index to another without the need to modify anything on the application level.

> Using an alias in application level code, e.g. a web service or API server, is recommended as it eases the maintainance effort. Often times an Elasticsearch index has to be updated, for example index mapping changes, then (re-)indexing documents to a new index is required. Using an alias allows the opportunity to create the new index, switching the alias to the new index without interrupting anything. From the perspective of a client the alias is the same as an index.

In order to see the effect, let's index a number of documents into this index by using the [Bulk API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-bulk.html) to index documents to index alias `rollover`:

✅ Bulk upload documents to index `rollover`

```bash
curl -H 'Content-Type: application/x-ndjson' -XPOST 'http://localhost:9200/rollover/_bulk' -d '
{"index":{"_index":"rollover"}}
{"title": "Harry Potter", "genre": "kids", "tags": ["action", "kids", "magic"]}
{"index":{"_index":"rollover"}}
{"title": "Star Trek", "genre": "action", "tags": ["space", "action", "better"]}
{"index":{"_index":"rollover"}}
{"title": "Star Wars", "genre": "action", "tags": ["space", "action", "good"]}
{"index":{"_index":"rollover"}}
{"title": "Star Lord", "genre": "kids", "tags": ["space", "action", "kids"]}
{"index":{"_index":"rollover"}}
{"title": "Spiderman", "genre": "action", "tags": ["action", "comic"]}
{"index":{"_index":"rollover"}}
{"title": "The Incredibles", "genre": "animation", "tags": ["kids", "spy"]}
{"index":{"_index":"rollover"}}
{"title": "The Favourite", "genre": "comedy", "tags": ["costume", "play"]}
{"index":{"_index":"rollover"}}
{"title": "Star Trek II", "genre": "action", "tags": ["space", "action", "better"]}
{"index":{"_index":"rollover"}}
{"title": "Black Panther", "genre": "action", "tags": ["comic", "action", "good"]}
{"index":{"_index":"rollover"}}
{"title": "Goonies", "genre": "kids", "tags": ["action", "kids"]}
'
```

This indexes a set of documents to index `rollover`.


## Curator

In this example we are using the Elasticsearch Curator with the `rollover` action (see [Curator documentation](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/rollover.html)) which uses the [Rollover Index API](https://www.elastic.co/guide/en/elasticsearch/reference/master/indices-rollover-index.html).

The `curator` container from the Docker Compose setup contains an `action` file with the following configuration.

```yaml
---
actions:
  1:
    action: rollover
    description: >-
      Rollover the index associated with alias 'rollover', with format "rollover-00000x" or
      using prefix-YYYY.MM.DD-1.
    options:
      # Alias name
      name: rollover
      conditions:
        max_age: 2d
        max_docs: 5
        max_size: 100mb
      extra_settings:
        # some index settings can be set here
        index.number_of_shards: 2
        index.number_of_replicas: 1
      timeout_override:
      continue_if_exception: False
      disable_action: False

```

This defines a `rollover` action that rolls over the index that the alias `rollover` points to when either of the following criteria are met

* the age of the index is older than 2 days
* the number of documents is more than 5
* or the maximum size of the index is 100 Mb

> The rollover criteria are only checked when the action is run with the Curator, or a similar request is sent to the Rollover Index API. Elasticsearch supports automated rollover with defining an Index Lifecycle Policy (see [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-index-lifecycle-management.html)) but this is only available via XPack.

Let's run the Curator to see the effect.

✅ Run the `curator` with the `rollover.yml` action file in dry mode

```bash
/usr/local/bin/curator --dry-run --config config.yml rollover.yml
```

There is no policy stored on the Elasticsearch node, therefore in order to make the rollover work the Curator needs to be run regularly in order to check the conditions and apply these changes.

To execute the Curator and apply the changes run.

✅ Run the `curator` to rollover index

```bash
/usr/local/bin/curator --config config.yml rollover.yml
```

This should roll over to a new index (due to number of documents), therefore the existing alias `rollover` should now point to a new index. To see the list of indices this one alias covers use the CAT API.

✅ Check the list of aliases

```bash
curl 'localhost:9200/_cat/aliases?v'
```

Alternatively you can check via Cerebro to which index the alias currently points to.

This write alias / index can be complemented with another alias (and action) to read from multiple indices. Assuming there are multiple indices (with rollover suffixes `-0000xx`).

* indices `rollover-000001`, `rollover-000002`, `rollover-000003`
* **write** alias `rollover` is set to index `rollover-000003` (our current alias)
* an additional **read** alias `rollover-search` (or any other name) can be set to all indices prefixed with `rollover-*`

> This is a good scenario for log based indices, where the write alias is used to write logs to the current index, while the read alias can be set to the list of indices to search in.
