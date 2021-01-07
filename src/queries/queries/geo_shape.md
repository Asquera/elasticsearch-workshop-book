# Geo Shape Queries

Elasticsearch supports geo based queries. The mapping supports two geo based field types, [geo_point](https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-point.html) and [geo_shape](https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-shape.html).

* `geo_point` is a pair of two coordinates, can be expressed in different ways, e.g. "41.12,-71.34" (latitude, longitude)
* `geo_shape` can be an arbitrary geo shape, e.g. rectangles or polygons

Elasticsearch provides support for the [WKT format](https://en.wikipedia.org/wiki/Well-known_text_representation_of_geometry) as input.


There are also a number of geo queries that can be used to find documents, see [geo queries documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-queries.html)

* `geo_bounding_box` - find documents with geo points inside a specified rectangle
* `geo_distance` - find documents within the specified distance of a central point
* `geo_polyon` - find documents with geo points within the specified polygon
* `geo_shape` - find documents with either `geo_point` & `geo_shape` field types


## Index Mapping

For this example we set up a list of restaurants around a specific area (Berlin) using the `geo_point` field type.

✅ Create a new index named `geo_test` with the following mapping.

```bash
curl -X PUT 'http://localhost:9200/geo_test' -H 'Content-Type: application/json' -d '{
  "mappings": {
    "properties": {
      "title": {
        "type": "text",
        "analyzer": "standard"
      },
      "position": {
        "type": "geo_point"
      },
      "review": {
        "type": "integer"
      }
    }
  }
}'
```

This sets ups a mapping with the following properties

* `title` with field type [text](https://www.elastic.co/guide/en/elasticsearch/reference/current/text.html)
* `position` with field type [geo_point](https://www.elastic.co/guide/en/elasticsearch/reference/current/geo-point.html)
* `review` with field type [integer](https://www.elastic.co/guide/en/elasticsearch/reference/current/number.html)


## Add Documents

Let's add a few documents that represent restaurants with locations.

✅ Bulk upload documents to index `geo_test`

```bash
curl -H 'Content-Type: application/x-ndjson' -X POST 'http://localhost:9200/geo_test/_bulk' -d '
{"index":{"_index":"geo_test"}}
{"title": "tak tak polish deli", "position": "52.5364964,13.3686393", "review": [4, 5, 3]}
{"index":{"_index":"geo_test"}}
{"title": "#84 Vietnamese Vegan Kitchen", "position": "52.5364964,13.3686393", "review": [5]}
{"index":{"_index":"geo_test"}}
{"title": "Brammibals Donuts", "position": "52.4936728,13.4231551", "review": [5]}
{"index":{"_index":"geo_test"}}
{"title": "ZOLA Pizza", "position": "52.4958723,13.421415", "review": [5, 3]}
{"index":{"_index":"geo_test"}}
{"title": "Cocola Ramen", "position": "52.4960332,13.4155789", "review": [5, 4]}
{"index":{"_index":"geo_test"}}
{"title": "Djimalaya Hummus & Grill", "position": "52.5323549,13.3968597", "review": [5, 2, 5]}
{"index":{"_index":"geo_test"}}
{"title": "Distrikt Coffee", "position": "52.5272252,13.3880857", "review": [5, 3, 2]}
{"index":{"_index":"geo_test"}}
{"title": "19grams", "position": "52.5331751,13.3779764", "review": [5, 3, 2]}
{"index":{"_index":"geo_test"}}
{"title": "Yumcha Heroes", "position": "52.5311998,13.4005831", "review": [5, 3, 4]}
'
```


## Exercise

To give some ideas, here are some landmarks in Berlin as reference points:

* Brandenburger Tor - `52.5156952,13.377515`
* Park am Nordbahnhof - `52.5331779,13.3821109`
* Museum Island - `52.5168903,13.3977636`
* Berlin Fernsehturm (tv tower) - `52.5208274,13.4072431`

✅ Write a **geo_distance** query that finds all documents in 3 kilometers distance of the Fernsehturm

See the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-geo-distance-query.html) for details.

<details>
<summary>Possible solution</summary>

```bash
curl -X POST 'http://localhost:9200/geo_test/_search?pretty' -H 'Content-Type: application/json' -d '{
  "query": {
    "bool": {
      "filter": {
        "geo_distance": {
          "distance": "3km",
          "position": {
            "lat": 52.5208274,
            "lon": 13.4072431
          }
        }
      }
    }
  }
}'
```
</details>

> What happens when you use a different reference / central point or change the distance?

✅ Write a **geo_bounding_box** query that finds all documents within a rectangle