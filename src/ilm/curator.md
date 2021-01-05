# Elasticsearch Curator

The [Curator](https://www.elastic.co/guide/en/elasticsearch/client/curator/5.8/command-line.html) is a command line tool written in Python to run index specific tasks in Elasticsearch, with the focus on index life cycle management, e.g. delete indices, take snapshots, set aliases.

The Elasticsearch project now supports several new API endpoints that take on some of the functionality, that were covered by defining specific actions in the Curator, e.g. index life cycle management, log index rotation, etc. For example Elasticsearch now supports [Index Lifecycle Management](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/index-lifecycle-management.html) via lifecycle policies. For more information see how to [configre a lifecycle policy](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/set-up-lifecycle-policy.html).

Most if not all of these new API endpoints that Elasticsearch offers require at least the Basic license of Elasticsearch, grouped under the `X-Pack` label. API endpoints marked with this label are accessible with the [Basic license subscription](https://www.elastic.co/subscriptions). Other Elasticsearch distributions, most notably the [AWS Elasticsearch Service](https://aws.amazon.com/elasticsearch-service/faqs/) may not provide these API endpoints. They usually also lack some minor versions behind the official releases.


## Configuration

The curator takes a configuration file that specifies the connection to the Elasticsearch cluster and an [actions file](https://www.elastic.co/guide/en/elasticsearch/client/curator/5.8/actions.html) to define tasks that are run in Elasticsearch. The curator is a tool that can be executed via cron job or in other automated setups.

The `curator` CLI takes a configuration file and an actions configuration, it can be run as:

```bash
curator --dry-run --config config.yml action.yml
```

The `--dry-run` option runs the curator without executing the tasks. It only displays the actions it would execute. The `config.yml` is the configuration file, the file `action.yml` defines the actions to be executed.

An example of the [config file](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/configfile.html) is given here.

```yml
---
client:
  hosts:
    - elasticsearch01
  port: 9200
  url_prefix:
  use_ssl: False
  certificate:
  client_cert:
  client_key:
  ssl_no_validate: False
  http_auth:
  timeout: 30
  master_only: False

logging:
  loglevel: INFO
  logfile:
  logformat: default
  blacklist: ['elasticsearch', 'urllib3']
```

The [action file](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/actionfile.html) is used to define a list of actions / tasks that are executed in the Elasticsearch Cluster. The curator communicates via HTTP with Elasticsearch using its different API endpoints.

The structure of the action file is:

```yml
---
actions:
  1:
    action: <action>
    description: >-
      A longer description of the action

  2:
    action: <action>
    ..
```

For example the following action file describes a task to delete all `logstash-` prefixed indices older than 30 days.

```yml
# delete_indices.yml
---
actions:
  1:
    action: delete_indices
    description: >-
      Delete indices older than 30 days (based on index name), for logstash-
      prefixed indices. Ignore the error if the filter does not result in an actionable list
      of indices (ignore_empty_list) and exit cleanly.
    options:
      ignore_empty_list: True
      timeout_override:
      continue_if_exception: True
      disable_action: False
    filters:
    - filtertype: pattern
      kind: prefix
      value: logstash-
    - filtertype: age
      source: name
      direction: older
      timestring: '%Y.%m.%d'
      unit: days
      unit_count: 30
```

Put both files into the same folder (named `config.yml` and `delete_indices.yml` respectively), then run the curator:

```bash
curator --config config.yml delete_indices.yml
```

## Docker Setuplife

This folder contains a [Docker](https://docs.docker.com/get-docker/) setup with a[docker-compose.yml](https://docs.docker.com/compose/install/) that can be used to run the different Curator examples. The setup consists of the following containers:

* elasticsearch01 ([localhost:9200](http://localhost:9200))
* elasticsearch02 ([localhost:9201](http://localhost:9201))
* elasticsearch03 ([localhost:9202](http://localhost:9202))
* cerebro ([localhost:9000](http://localhost:9000/#/rest?host=http:%2F%2Felasticsearch01:9200))
* curator

To run the examples start the containers:

```bash
docker-compose up --build
```

Once all containers run, start the curator container and open a shell in it

```bash
docker-compose run --rm curator /bin/ash
```

Once the container started, add the appropriate action configuration file to the container. Inside the container create a new action configuration file and insert / paste the content of the example you want to test, for example the content of the `delete_indices.yml` above:

For example

```bash
touch delete_indices.yml
vi delete_indices.yml
```

Then copy the content into it, save the file and you should be good to go to test the curator.
