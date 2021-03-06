# Index Deletion

One common task in a log-based Elasticsearch infrastructure is to keep logs for a certain amount of time, e.g. 30 days. The Curator can be used to automatically delete older indices. 

> In case the XPack functionality is available, check out the [index lifecycle management](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html) documentation of Elasticsearch on how to use index lifecycle policies.

## Configuration

Set up the `config.yml` to communicate with the Elasticsearch cluster. Check the [example `config.yml`](curator.html#configuration) on how to set up the configuration file.

## Action

The Curator provides a `delete_indices` action to delete indices, see [Curator documentation](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/delete_indices.html) for more details. The basic structure of this action is:

```yaml
action: delete_indices
description: "Delete selected indices"
options:
  timeout_override: 300
  continue_if_exception: False
filters:
- filtertype: ...
```

One important property is the [filters](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/filters.html) list that can take a number of `filtertypes` fields. Check the [age](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/filtertype_age.html) and [pattern](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/filtertype_pattern.html) filters.

The `pattern` filter type allows to find indices that match the given string pattern, e.g. `logstash-`. The `age` filter type matches the indices based on their age. The age can be specified in multiple ways, e.g. name based, creation based, etc.

> Specifying multiple filter types requires that indices match all of them. In this way indices can be filtered out.


## Exercise

Follow these steps to run the excercise

✅ Start Elasticsearch

To make this example visible, first we create at least 3 daily indices with a date based suffix, e.g. `logstash-2021.01.05`, `logstash-2021.01.04`, using the [Create Index API](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-create-index.html).

✅ Create three `logstash-*` prefixed indices with daily dates

```bash
# curl -X PUT 'http://localhost:9200/<logstash-{now/d-2d}>'
curl -X PUT 'http://localhost:9200/%3Clogstash-%7Bnow%2Fd-2d%7D%3E'
# curl -X PUT 'http://localhost:9200/<logstash-{now/d-1d}>'
curl -X PUT 'http://localhost:9200/%3Clogstash-%7Bnow%2Fd-1d%7D%3E'
# curl -X PUT 'http://localhost:9200/<logstash-{now/d}>'
curl -X PUT 'http://localhost:9200/%3Clogstash-%7Bnow%2Fd%7D%3E'
```

> Elasticsearch supports Date math for index names. For example the index name `/<logstash-{now/d-2d}>` references an index named `logstash-2021.01.17` from two days ago (assuming it's 2021.01.19 when used). The pair of `<`, `>` marks the template part of the index, while the function `{now/d-2d}` creates the `2021.01.19` and subtracts two days from it. The index name has to be URL encoded to conform to the URL format.

✅ Create or check the `config.yml` (see [Configuration section](curator.html#configuration))

Check the [delete_indices](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/delete_indices.html) action reference in the Curator documentation to see how an `action` configuration looks.

✅ Create an actions file named `delete_indices.yml` that deletes `logstash-*` prefixed indices that are older than `2` days

Once both config and action files are created, let's run the curator in dry mode

✅ Run the `curator` on the command line `--dry-mode`

```bash
curator --dry-run --config config.yml delete_indices.yml
```

Check the output of this command, what does it do?

✅ Run `curator` command to delete older indices

```bash
curator --config config.yml delete_indices.yml
```

When successful, there should only be two indices starting with the same suffix, e.g. `logstash-*`, older indices should have been deleted.

> Depending on your chosen values in the **age** filter type no indices may have been deleted. What is the difference between `source: creation_date` and `source: timestring`?

Here is a version of the `action.yml` to delete older indices.

<details>
<summary>Possible Solution</summary>

```yaml
# delete_indices.yml
---
actions:
  1:
    action: delete_indices
    description: >-
      Delete indices older than 2 days (based on timestring pattern), for logstash-
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
      unit_count: 2
```
</details>

Mulitple action files can be used separately. This way separate indices or index groups can be managed, for example different applications, users etc.
