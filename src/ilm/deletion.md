# Index Deletion

One common task in a log-based Elasticsearch infrastructure is to keep logs for a certain amount of time, e.g. 30 days. The Curator can be used to automatically delete older indices. 

> **❗️** in case the XPack functionality is available, check out the [index lifecycle management](https://www.elastic.co/guide/en/elasticsearch/reference/current/index-lifecycle-management.html) documentation of Elasticsearch on how to use index lifecycle policies.

## Configuration

Set up the `config.yml` to communicate with the Elasticsearch cluster. Check the [example `config.yml`](curator.html#configuration) on how to the configuration file needs to be defined.


## Action

The Curator provides a [delete_indices](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/delete_indices.html) action to delete indices. The basic structure of this action is:

```yaml
action: delete_indices
description: "Delete selected indices"
options:
  timeout_override: 300
  continue_if_exception: False
filters:
- filtertype: ...
```

The important property is the [filters](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/filters.html) list that can take a number of `filtertypes` fields. Check the [age](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/filtertype_age.html) and [pattern](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/filtertype_pattern.html) filters.

The `pattern` filter type allows to find indices that match the given string pattern, e.g. `logstash-`. The `age` filter type matches the indices based on their age. The age can be specified in multiple ways, e.g. name based, creation based, etc.

> Specifying multiple filter types requires that indices match all of them. In this way indices can be filtered out.


## Exercise

Follow these steps to run the excercise

* ✅ start Elasticsearch
* ✅ create at least 3 daily indices with a date based suffix, e.g. `logstash-2021.01.05`, `logstash-2021.01.04`, etc., in order to see the effect it's recommended to use current dates
* ✅ create a `config.yml` (see [Configuration section](curator.html#configuration))
* ✅ create a `action.yml` that uses the [delete_indices](https://www.elastic.co/guide/en/elasticsearch/client/curator/current/delete_indices.html) action to delete indices older than `2` days
* ✅ run `curator` command line tool with `--dry-mode`, check the output for actions
* ✅ run `curator` command to delete older indices

When successful, there should only be two indices starting with the same suffix, e.g. `logstash-*`, older indices should have been deleted.

> **🔎** depending on your chosen values in the **age** filter type no indices may have been deleted. What is the difference between `source: creation_date` and `source: timestring`?


## Solution

Here is a version of the `action.yml`.

<details>
<summary>Click Me</summary>

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
