# Logstash

Logstash is a data pipeline management tool, it can dynamically unify data from multiple different input sources, transform and forward them to multiple destinations.
It is the **L** part of the ELK stack, the main component in log aggregation & indexing preferably in a highly distributed environment.


## Logstash Pipeline

Logstash supports the concept of pipelines, separate processes to fetch data from multiple sources, to filter / enricht or transform them to finally store or forward them to other systems, e.g. to Elasticsearch as in the ELK stack.

The basic Logstash configuration or pipeline consists of three parts

* **inputs**
  * see [list of input plugins](https://www.elastic.co/guide/en/logstash/current/input-plugins.html)
  * e.g. syslog, files, filebeat, Redis, Kafka, Postgres
* **filters**
  * to mutate, grok, drop, clone, tag data
* **outputs**
  * see [list of output plugins](https://www.elastic.co/guide/en/logstash/current/output-plugins.html)
  * Elasticsearch, databases, file, graphite, statsd

There are a lot of plugins available for most technologies that write or read data.
Logstash pipelines define separate processes that do not interact or interfere with each other.

For example see the following Logstash config

```logstash
input {
    redis {
        host      => "redis"
        port      => 6379
        data_type => "list"
        key       => "filebeat"
    }
}
filter {
}
output {
    elasticsearch {
        hosts => "elasticsearch01:9200"
        index => "logstash-%{+YYYY.MM.dd}"
}
```

This Logstasn configuration defines one input source `redis`, no filter and one output target `elasticsearch`. Logstash allows to configure:

* 1 or more input sources
* no or multiple filters
* 1 or more output targets

All the data collected from the input sources can be managed separately when using tags and filters, while the data can be forwarded to the same output targets or separately.


## Setup

This example uses the [Docker Compose Setup](../introduction/setup.md) to start a few containers. The setup already configures 2 separate pipelines.

✅ Start the associated containers to demonstrate Logstash pipelines

```bash
docker-compose up --build elasticsearch01 logindexer app redis cerebro
```

This starts the following containers

* `elasticsearch01` as the **output** target, logs are stored in an index
* `logindexer` the Logstash instance that waits for data from Redis, forwards them to Elasticsearch
* `app` mimicks an application that uses `filebeat` (see [filebeat documentation](https://www.elastic.co/guide/en/beats/filebeat/current/index.html)) to read logs from files inside the container
* `redis` is used as a message buffer that gets its data via the `app`
* `cerebro` (optional) to check that documents are indexed in Elasticsearch

This process looks as the following simplified chart

```
--------------     ------------     --------------     -----------------
|    app     | --> |  redis   | --> | logindexer | --> | elastiscearch |
| (filebeat) |     | (buffer) |     |            |     |   cluster     |
--------------     ------------     --------------     -----------------
```

✅ Connect to the `app` container (Centos based)

```bash
# Find docker id of app container
docker ps
# Use docker id to attach to running app container
docker exec -it <docker-id> /bin/bash
```

When successful this opens a shell in the `app` container. To check that Redis is reachable from within the `app` container we ping it.

✅ Send a "PING" message from within `app` container to `redis`

```bash
echo PING | nc redis 6379
```

See this [Stack Overflow thread](https://stackoverflow.com/questions/33243121/abuse-curl-to-communicate-with-redis) for more details on this command. When successful this should output simply

```bash
+PONG
```

The next step is to check the filebeat configuration within the `app` container is loaded.

✅ Check the filebeat configuration within the `app` container

```bash
cd /usr/share/filebeat
filebeat -c filebeat.yml -e -d "*"
```

This outputs if filebeat is running.
Now we should be able to generate logs. The `filebeat` configuration checks for new lines in the log file `/var/log/filebeat.log`. Therefore it should be sufficient to append new text lines to the file.

✅ Append a log text to the filebeat log

```bash
echo "Hello World" >> /var/log/filebeat.log
```

The string can be anything, for example JSON or any other typical log text.


## Exercise

**In preparation**
