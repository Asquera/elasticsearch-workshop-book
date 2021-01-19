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


## References

* [Snapshot and Restore](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-restore.html)
* [Snapshot Lifecycle Management](https://www.elastic.co/guide/en/elasticsearch/reference/current/snapshot-lifecycle-management.html) - requires [Basic](https://www.elastic.co/subscriptions) license

