# CC Agency

## Development Setup

The following instructions have been tested on Fedora 28 with Python 3.6. Instructions for other Linux distributions should be similar.

### Install Dependencies

First install the **latest** version of `docker-ce` from the official external Docker repository (see [instructions](https://docs.docker.com/install/linux/docker-ce/fedora/)).

Then install required distribution packages and Python modules.

```bash
sudo dnf install git python3-pip python3-venv uwsgi uwsgi-plugin-python3 docker-compose
git clone https://github.com/curious-containers/cc-core.git
git clone https://github.com/curious-containers/cc-agency.git
cd cc-agency
pip3 install --user --upgrade poetry
poetry install
```

Curious Containers uses [Poetry](https://poetry.eustace.io/) for dependency management and deployment.

### Run Components

Run the following software components in separate terminals.

#### Terminal 1 - MongoDB

Run MongoDB in a Docker container.

```bash
docker-compose -f dev/docker-compose.yml up
```

The created database is stored in the local filesystem (see `dev/docker-compose.yml` for details) and will persist if the container is deleted.

A MongoDB admin user account is created automatically by a `mongo-seed` container defined in `dev/docker-compose.yml`. The credentials are read from `dev/cc-agency.yml`.

#### Terminal 2 - CC-Agency Controller

You can only run one instance of CC-Agency Controller at a time. It provides the central scheduling component, which connectes to a cluster of docker-engines.

```bash
PYTHONPATH=../cc-core poetry run python3 -m cc_agency.controller -c dev/cc-agency.yml
```

#### Terminal 3 - CC-Agency Broker

CC-Agency Broker provides a REST API, to schedule RED experiments, receive agent callbacks and to query information. It informs the Controller about changes via a ZMQ socket. Edit `dev/uwsgi.ini` to increase the number of Broker processes or threads.

```bash
PYTHONPATH=../cc-core poetry run uwsgi --ini dev/uwsgi.ini
```

### Create Users

Create users to authenticate with the CC-Agency Broker REST API, with or without admin privileges. Admin privileges can change the behaviour of certain API endpoints.

```bash
PYTHONPATH=../cc-core poetry run python3 -m cc_agency.tools create-broker-user -c dev/cc-agency.yml
```

### Create CC-Core Image

The conf file `dev/cc-agency.yml` references a `cc-core` Docker image. You can build it locally.

```bash
docker build -t cc-core ../cc-core
```

Run a container based on this image, to check if everything is working.

```
docker run -u 1000:1000 cc-core ccagent --help
```

### Reset Database

If you need to reset the database during development, run the following command and specify the collections to be dropped.

```bash
COLLECTIONS="experiments batches users tokens block_entries callback_tokens"
PYTHONPATH=../cc-core poetry run python3 -m cc_agency.tools drop-db-collections -c dev/cc-agency.yml ${COLLECTIONS}
```
