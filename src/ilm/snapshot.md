# Snapshot

One important requirement in most production setups is to create backups of the data. Elasticsearch supports taking incremental snapshots of one or multiple indices. 

There are multiple options to store Elasticsearch snapshots in repositories. The repository described in this section is using the type `fs`, to store snapshots in a shared file system. Other repositories can be supported by installing the appropriate repository plugin, which are available for a number of cloud providers:

* S3 buckets
* Azure
* HDFS file system
* Google Cloud Storage

The list of plugins can be found in the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/plugins/7.10/repository.html).


## Setup

This example uses the Elasticsearch Curator to run the snapshot process. The Curator action uses the Create Snapshot API endpoint (see [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html)). It requires a bit of a setup first.

The Docker Compose environment described in the [Setup](../introduction/setup.md#docker-compose-recommended) chapter, defines three Elasticsearch nodes in the `docker-compose.yml`.

✅ Start the Docker containers

```bash
docker-compose up elasticsearch01 curator cerebro
```

Once all containers started, check the Elasticsearch cluster via [Cerebro](http://localhost:9000/#/overview?host=http:%2F%2Felasticsearch01:9200).

> The Elasticsearch containers have mounted the same shared volume to support the `fs` repository type.

First we create a new index with some data to showcase snapshot and restore.

✅ Create a new index named `logstash-2021.01.20`

```bash
curl -X PUT 'http://localhost:9200/logstash-2021.01.20' -H 'Content-Type: application/json' -d '{}'
```

✅ Bulk upload documents into index `logstash-2021.01.20`.

```bash
curl -H 'Content-Type: application/x-ndjson' -XPOST 'http://localhost:9200/logstash-2021.01.20/_bulk' -d '
{"index":{"_index":"logstash-2021.01.20"}}
{"message": "hello world 1"}
{"index":{"_index":"logstash-2021.01.20"}}
{"message": "hello world 2"}
{"index":{"_index":"logstash-2021.01.20"}}
{"message": "hello world 3"}
{"index":{"_index":"logstash-2021.01.20"}}
{"message": "hello world 4"}
{"index":{"_index":"logstash-2021.01.20"}}
{"message": "hello world 5"}
{"index":{"_index":"logstash-2021.01.20"}}
{"message": "hello world 6"}
{"index":{"_index":"logstash-2021.01.20"}}
{"message": "hello world 7"}
{"index":{"_index":"logstash-2021.01.20"}}
{"message": "hello world 8"}
{"index":{"_index":"logstash-2021.01.20"}}
{"message": "hello world 9"}
{"index":{"_index":"logstash-2021.01.20"}}
{"message": "hello world 10"}
'
```

This adds a number of documents we will use to take a snapshot with.


### Create repository

As a first step a snapshot repository needs to be registered. This is required to store the snapshots in.
The repository type `fs` is supported by Elasticsearch without installing any plugins.
To register a shared file system, the file system to the same location needs to be mounted on all master and data nodes.

> Elasticsearch requires the `path.repo` attribute in the configuration. This field needs to point to the same shared folder location.

This setting can be configured via `elasticsearch.yml` configuration file, for example

```yaml
# elasticsearch
path.repo:
  - /usr/share/elasticsearch/repos
```

The Docker containers in the Docker Compose setup have this configuration already set, they are using the same shared folder.

✅ Register a new file system repository named `logstash_backup_repository`

```bash
curl -X PUT "localhost:9200/_snapshot/logstash_backup_repository?pretty" -H 'Content-Type: application/json' -d '{
  "type": "fs",
  "settings": {
    "location": "/usr/share/elasticsearch/repos"
  }
}'
```

This registers a new repository of type `fs`, the snapshots are stored in the mounted shared folder at location `/usr/share/elasticsearch/repos`.

> **❗️** Using the Docker Compose setup may fail with an `access_denied_action`. Connect to the container and set the permission for the repository folder `chown elasticsearch /usr/share/elasticsearch/repos`.

> In a Kubernetes environment, the shared file system needs to be the same on all (data, master) nodes, this would be the same persistent volume mounted into each Elasticsearch pod.

[Cerebro](http://localhost:9000/#/repository?host=http:%2F%2Felasticsearch01:9200) and Kibana also provide views to the see the list of repositories and their snapshots.


### Create Snapshot

Once the repository is registered, Elasticsearch can create snapshots and store them in the repository.
To create a new snapshot of one or multiple indices the [Create Snapshot API](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html) is used.

✅ Create a new snapshot for all `logstash-*` prefix 

```bash
curl -H 'Content-Type: application/json' -X PUT "localhost:9200/_snapshot/logstash_backup_repository/logstash_snapshot_1?wait_for_completion=true&pretty" -d '{
  "indices": "logstash-*",
  "ignore_unavailable": true,
  "include_global_state": false
}'
```

This creates a new snapshot named `logstash_snapshot_1`. The `curl` command waits (the `wait_for_completion` query parameter) until the snapshot process is complete. The `indices` field inside the snapshot request can be a single index, multiple indices or using wildcards as in our case (`logstash-*`) to match all `logstash-` prefixed indcies.

> A snapshot name needs to be unique, an existing snapshot is not appended or overwritten.

One way to generate a new snapshot name is to use the Date related functions, see [Options for creating a Snapshot](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-take-snapshot.html#create-snapshot-options) in the Elasticsearch documentation.
For example to generate a new snapshot with on a daily basis is using snapshot name 

```bash
# without URL parameter encoding
curl -H 'Content-Type: application/json' -X PUT 'localhost:9200/_snapshot/logstash_backup_repository/<logstash-{now/d}>?wait_for_completion=true&pretty'
# with URL parameter encoding
curl -H 'Content-Type: application/json' -X PUT 'localhost:9200/_snapshot/logstash_backup_repository/%3Clogstash-%7Bnow%2Fd%7D%3E?wait_for_completion=true&pretty'
```

where path `/_snapshot/logstash_backup_repository/<logstash-{now/d}<` contains the snapshot name `<logstash-{now/d}>` (enclosed by brackets `<`, `>`) which is replaced with today's date, for example `logstash-2020.01.20`.


### Restore Snapshot

To restore a snapshot, for example to re-create an index from a backup or transferring indices to a new cluster the [Restore Snapshot API](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html) is used.

> When restoring an index from a snapshot **ensure** the target index is not present in the Elasticsearch cluster.

✅ Restore the previously snapshot `logstash_snapshot_1` from repository `logstash_backup_repository`.

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST "localhost:9200/_snapshot/logstash_backup_repository/logstash_snapshot_1/_restore?pretty" -d '{
  "indices": "logstash-*",
  "ignore_unavailable": true
}'
```

This restores all `logstash-` prefixed indices from snapshot `logstash_snapshot_1` in repository `logstash_backup_repository`.

> This may require to delete the `logstash-*` indices first.

This API endpoint also allows a few more settings, for example to rename an index when restoring it. It also has support to set new index settings, e.g. number of shards. For more information see the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshots-restore-snapshot.html#change-index-settings-during-restore).


### Delete Snapshot

To delete an existing snapshot use the [Delete Snapshot API](https://www.elastic.co/guide/en/elasticsearch/reference/current/delete-snapshots.html) endpoint. Snapshot deletion is useful to reduce the number of backups to keep.

✅ Delete the snapshot `logstash_snapshot_1` from repository `logstash_backup_repository`

```bash
curl -H 'Content-Type: application/x-ndjson' -X DELETE "localhost:9200/_snapshot/logstash_backup_repository/logstash_snapshot_1?pretty" -d '{}'
```

This deletes the snapshot `logstash_snapshot_1` from the repository `logstash_backup_repository`. Deleting snapshots is useful to only keep a certain number or a certain amount of indices as backup.


## Curator

To automate the creation of snapshots the Elasticsearch Curator can be used. The `curator` container contains an action file `snapshot.yml` in folder `/config`. This action file contains the task to create daily snapshots.

✅ Run the `curator` with the `snapshot.yml` action file in dry mode

```bash
/usr/local/bin/curator --dry-run --config config.yml snapshot.yml
```

To take regular snapshots a good use case is to run the curator with the actions file in a cron job.

✅ Run the `curator` to create a new snapsho

```bash
/usr/local/bin/curator --config config.yml snapshot.yml
```

If there is no current snapshot yet, one is created. Once successfully created we can use the Cerebro interface to check the created snapshots or use the [cat snapshots API](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-snapshots.html) to get a list of snapshots.

✅ List all available snapshots for repository `logstash_backup_repository` using the cat API

```bash
curl -X GET 'http://localhost:9200/_cat/snapshots/logstash_backup_repository?v'
```

This lists all available snapshots for the repository.


## References

* [Snapshot and Restore](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-restore.html)
* [Snapshot Lifecycle Management](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-lifecycle-management.html) - requires [Basic](https://www.elastic.co/subscriptions) license

