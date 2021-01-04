# Elasticsearch Setup

In order to follow the examples it is recommended to run an Elasticsearch instance. Most examples only require a single instance. There are multiple ways to set up an environment.


## Docker Compose (recommended)

The recommended way is to use the same environment as used in the presentations, available at the Github repository [Asquera/elasticsearch-docker](https://github.com/Asquera/elasticsearch-docker). This setup uses [Docker](https://www.docker.com/) & [Docker Compose](https://docs.docker.com/compose/) to start Elasticsearch (and other containers).

To set up the same environment:

* [Install Docker](https://docs.docker.com/get-docker/) for your operating system
  * **Note** the Windows installation includes Compose
* in case the installation does not include **Compose**, [install Docker Compose](https://docs.docker.com/compose/install/)

Once installed, checkout the repository from Github:

```bash
git clone https://github.com/Asquera/elasticsearch-docker.git
```

this project also includes a [Readme](https://github.com/Asquera/elasticsearch-docker/blob/main/Readme.md) with instructions on how to use the setup. To start a single Elasticsearch instance run:

```bash
docker-compose up --build elasticsearch01
```

this starts the Elasticsearch container named **elasticsearch01**. This may take a few minutes, there is also log output. Once started, the Elasticsearch instance is accessible at [localhost:9200](http://localhost:9200). You may need to open another shell to access it or add the option `--detach` (or short `-d`) to start the instance in detached mode.


## Docker

An alternative way to set up the environment with a single Elasticsearch Docker container. It only requires [Docker](https://www.docker.com/). In case Docker is not installed, [Get Docker](https://docs.docker.com/get-docker/) first for your operating system. Once installed the following command downloads the official Elasticsearch Docker image. Enter the following command into your terminal:

```shell
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.8.1
```

this downloads, unpacks and starts the Docker image with Elasticsearch (verion `7.8.1`). This starts the Elasticsearch instance, forwards ports `9200` and `9300`. Once started the Elasticsearch instance is accessible at [localhost:9200](http://localhost:9200). You may need to open another shell to access it.


## VM

There also ways to set up an Elasticsearch either locally (e.g. Virtual Box, vagrant) or remotely at a cloud provider. This may require manual installation of Elasticsearch, but the principle is similar.
