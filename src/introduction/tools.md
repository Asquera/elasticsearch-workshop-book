# Tools

Elasticsearch offers its functionality via API endpoints over HTTP. Therefore any HTTP client can be used to communicate with it.


## curl

The most common way to communicate with Elasticsearch is using the command line tool [curl](https://curl.se/). The examples in the official Elasticsearch documentation provide curl commands to copy.

For example to check that Elasticsearch is running at [localhost:9200](http://localhost:9200) use the following command in your terminal:

```bash
curl localhost:9200
```

it should output name, version and some details, similar to the following output:

```txt
{
  "name" : "8226182e008a",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "j-gk46dIT5yoAe_8l4sMrA",
  "version" : {
    "number" : "7.8.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "b5ca9c58fb664ca8bf9e4057fc229b3396bf3a89",
    "build_date" : "2020-07-21T16:40:44.668009Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```


## Graphical clients

All communication is done via HTTP therefore any graphical client can be used as well. The following tools provide a graphical interface, allow to store previous queries etc.

* [Postman App](https://www.postman.com/downloads/) - fully fledged API platform client
* [Insomnia](https://insomnia.rest/) - a REST client


## Cerebro

Another useful tool to check the state of Elasticsearch is [Cerebro](https://github.com/lmenezes/cerebro) (the successor to **head** and **kopf**). It provides a web interface to see the cluster state, Elasticsearch instances, indexes. It also provides some basic management capabilities, for example to set aliases, create / restore snapshots. One of the views is a basic query interface.
