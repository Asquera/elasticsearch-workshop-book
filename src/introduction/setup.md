# Elasticsearch Setup

In order to follow the examples in this book it is recommended to run an Elasticsearch instance. Nearly all examples only require a single instance. There are multiple ways to set up an environment.

> All examples are tested with Elasticsearch 7.8.1. Using an earlier or newer version should most likely work fine.

## Docker Compose (recommended)

The recommended way is to use the same local setup as used in the presentations. The setup is available at the Github repository [Asquera/elasticsearch-docker](https://github.com/Asquera/elasticsearch-docker). It uses [Docker](https://www.docker.com/) & [Docker Compose](https://docs.docker.com/compose/) to start Elasticsearch (and other containers).

To set up the same environment:

* ✅ [Install Docker](https://docs.docker.com/get-docker/) for your operating system (if not present)
  * **Note** the Windows installation includes Compose
* ✅ in case the installation does not include **Compose**, [install Docker Compose](https://docs.docker.com/compose/install/)

Once installed, checkout the repository from Github:

✅ Use `git` to checkout the repository:

```bash
git clone https://github.com/Asquera/elasticsearch-docker.git
```

this project also includes a [Readme](https://github.com/Asquera/elasticsearch-docker/blob/main/Readme.md) with instructions on how to use the setup.

✅ Change to the `elasticsearch-docker` folder

```bash
cd elasticsearch-docker
```

✅ Start a single Elasticsearch instance by running the command:

```bash
docker-compose up --build elasticsearch01
```

This starts the Elasticsearch container named **elasticsearch01**, which is one of the containers in the setup. This may take a few minutes to initialize. The command also logs its output on stdout.

Once started, the Elasticsearch instance is accessible at [localhost:9200](http://localhost:9200). You may need to open another shell to access it or add the option `--detach` (or short `-d`) to the `docker-compose` command to start the instance in detached mode.

> **🔎** The repository contains a `docker-compose.yml` file that defines multiple other containers for different setups, e.g. ELK stack that is useful for a few of the examples.


## Docker

An alternative way to set up the environment with a single Elasticsearch Docker container requires only [Docker](https://www.docker.com/). 

✅ In case Docker is not installed [Install Docker](https://docs.docker.com/get-docker/) for your operating system.

Once installed the following command downloads the official Elasticsearch Docker image.

✅ Enter the following command into your terminal:

```shell
docker run -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.8.1
```

This downloads, unpacks and starts the official Elasticsearch Docker image (verion `7.8.1`). The command starts the Elasticsearch instance and forwards ports `9200` and `9300`. Once started the Elasticsearch instance is accessible at [localhost:9200](http://localhost:9200). You may need to open another shell to communicate with it.


## Docker Commands

Here are a few useful `docker` commands to start a container or to open a shell to a running container.

To start a Docker container from a specific image. The command

```bash
docker run postgres:13.1
```

downloads, unpacks and starts the Docker container for Postgres 13.1.

To list all running containers, for example to determine the **container id**.

```bash
docker ps
```

To connect to a running container to either run a command or open a shell

```bash
docker exec -it <container-id> /bin/bash
```

where `<container-id>` is the id of a running container. Once connected to a container you can navigate, edit files inside a container.

To stop a specific container run

```bash
docker stop <container-id>
```

To remove a stopped container, for example when different Docker Compose setups use same conflicting name.

```bash
docker rm <container-id>
```


## VM

There also ways to set up an Elasticsearch either locally (e.g. Virtual Box, vagrant) or remotely at a cloud provider. This may require manual installation of Elasticsearch, but the principle is similar.
