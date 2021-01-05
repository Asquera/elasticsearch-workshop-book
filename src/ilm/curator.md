# Elasticsearch Curator

The [Curator](https://www.elastic.co/guide/en/elasticsearch/client/curator/5.8/command-line.html) is a command line tool written in Python to run index specific tasks in Elasticsearch, with the focus on index life cycle management, e.g. delete indices, take snapshots, set aliases.

The Elasticsearch project now supports several new API endpoints that take on some of the functionality, that were covered by defining specific actions in the Curator, e.g. index life cycle management, log index rotation, etc. For example Elasticsearch now supports [Index Lifecycle Management](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/index-lifecycle-management.html) via lifecycle policies. For more information see how to [configre a lifecycle policy](https://www.elastic.co/guide/en/elasticsearch/reference/7.10/set-up-lifecycle-policy.html).

API endpoints marked with the **XPack** label require at least the [Basic license subscription](https://www.elastic.co/subscriptions). Other Elasticsearch distributions, most notably the [AWS Elasticsearch Service](https://aws.amazon.com/elasticsearch-service/faqs/) may not provide these API endpoints. They usually also lack some minor versions behind the official releases.


## Configuration

The Curator takes a configuration file that specifies the connection to the Elasticsearch cluster and an [actions file](https://www.elastic.co/guide/en/elasticsearch/client/curator/5.8/actions.html) to define tasks that are run in Elasticsearch. The curator is a CLI tool that can be executed via cron job or in other automated setups.

The `curator` command line tool takes a configuration file and an actions configuration. Once installed it can be executed in the terminal as follows:

```bash
curator --dry-run --config config.yml action.yml
```

The `curator` command takes the following options

* `--dry-run` (optional) runs the curator without executing the tasks, it only displays the actions it would execute
* `--config` specifies the path to the configuration file (`config.yml`)
* `action.yml` is the path to the actions file to execute

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

It specifies the connection to the Elasticsearch cluster, in this case it assumes there is an instance accessible at `elasticsearch:9200`.

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

Under the main element `actions` an indexed based list of actions are defined. The next few sections provide examples how these may look like.
