---
title: "Source Code Setup"
permalink: /docs/source-code-setup
---

The following instructions have been tested on Fedora 28 with Python 3.6. Instructions for other Linux distributions should be similar.

All software components have dependencies defined in pyproject.toml and pyproject.lock files (see [PEP 518](https://www.python.org/dev/peps/pep-0518/)). [Poetry](https://poetry.eustace.io/) is used to work with these files, to automatically create virtual environments and to build and deploy the resulting Python packages.


# Development Setup

## System Packages

Curious Containers requires a Linux distribution of your choice and the **latest** stable version of Docker to be installed. It is recommended to install Docker-CE from the official external Docker repository (see [instructions](https://docs.docker.com/install/linux/docker-ce/fedora/)), because Docker packages shipped by Linux distributions are often outdated.

In addition to Docker-CE, install the following system packages.

```bash
sudo dnf install git python3-pip python3-venv               # general dev dependencies
sudo dnf install uwsgi uwsgi-plugin-python3 docker-compose  # ccagency dev dependencies
```


## Python Virtual Environments

```bash
mkdir -p ~/.cache/cc/
python3 -m venv ~/.cache/cc/poetry
python3 -m venv ~/.cache/cc/dev
```


## Poetry

Install [Poetry](https://github.com/sdispater/poetry), which will handle all the Python dependencies of each package.

```bash
source ~/.cache/cc/poetry/bin/activate
curl -sSL https://raw.githubusercontent.com/sdispater/poetry/master/get-poetry.py | python
deactivate
source ~/.poetry/env
```


## Python Packages

The `cc-core`, `cc-faice` and `cc-agency` packages have complementary dependencies, that can be installed into a single venv.
Shared dependencies of `cc-faice` and `cc-agency` are defined in `cc-core`.

```bash
git clone https://github.com/curious-containers/curious-containers.git
cd curious-containers

# activate venv
source ~/.cache/cc/dev/bin/activate

# add source code folders to PYTHONPATH
export PYTHONPATH=$(pwd)/cc-core:$(pwd)/cc-faice:$(pwd)/cc-agency:${PYTHONPATH}

# optional: add PYTHONPATH change to .bashrc to make it permanent
echo export PYTHONPATH=$(pwd)/cc-core:$(pwd)/cc-faice:$(pwd)/cc-agency:'${PYTHONPATH}' >> ~/.bashrc

# install cc-core dependencies
cd cc-core
poetry install
cd ..

# install cc-faice dependencies
cd cc-faice
poetry install
cd ..

# install cc-agency dependencies
cd cc-agency
poetry install
cd ..
```

Check if commandline tools work as expected.

```bash
faice --help
ccagency --help
```


# Running CC-Agency

Run the following components in separate terminals.


## Terminal 1 - MongoDB

Run MongoDB in a Docker container. Make sure that

```bash
cd cc-agency
docker-compose -f dev/docker-compose.yml up
```

The created database is stored in the local filesystem (see `dev/docker-compose.yml` for details) and will persist if the container is deleted.

A MongoDB admin user account is created automatically by a `mongo-seed` container defined in `dev/docker-compose.yml`. The credentials are read from `dev/cc-agency.yml`.


## Terminal 2 - CC-Agency Trustee

You can only run one process/thread of CC-Agency Trustee at a time. It provides a central in-memory secrets storage for experiments. If you restart this service all secrets will be lost and unfinished experiments will fail.

```bash
source ~/.cache/cc/dev/bin/activate
export PYTHONPATH=$(pwd)/cc-core:$(pwd)/cc-faice:$(pwd)/cc-agency:${PYTHONPATH}

cd cc-agency
uwsgi --ini dev/uwsgi-trustee.ini
```


## Terminal 3 - CC-Agency Controller

You can only run one process/thread of CC-Agency Controller at a time. It provides the central scheduling component, which connectes to a cluster of docker-engines.

```bash
source ~/.cache/cc/dev/bin/activate
export PYTHONPATH=$(pwd)/cc-core:$(pwd)/cc-faice:$(pwd)/cc-agency:${PYTHONPATH}

cd cc-agency
ccagency-controller -c dev/cc-agency.yml
```


## Terminal 4 - CC-Agency Broker

CC-Agency Broker provides a REST API, to schedule RED experiments, receive agent callbacks and to query information. It informs the Controller about changes via a ZMQ socket. Edit `dev/uwsgi.ini` to increase the number of Broker processes or threads.

```bash
source ~/.cache/cc/dev/bin/activate
export PYTHONPATH=$(pwd)/cc-core:$(pwd)/cc-faice:$(pwd)/cc-agency:${PYTHONPATH}

cd cc-agency
uwsgi --ini dev/uwsgi-broker.ini
```

## Create Users

Create users to authenticate with the CC-Agency Broker REST API, with or without admin privileges. Admin privileges can change the behaviour of certain API endpoints.

```bash
source ~/.cache/cc/dev/bin/activate
export PYTHONPATH=$(pwd)/cc-core:$(pwd)/cc-faice:$(pwd)/cc-agency:${PYTHONPATH}

cd cc-agency
ccagency create-broker-user -c dev/cc-agency.yml
```

## Reset Database

If you need to reset the database during development, run the following command and specify the collections to be dropped.

```bash
source ~/.cache/cc/dev/bin/activate
export PYTHONPATH=$(pwd)/cc-core:$(pwd)/cc-faice:$(pwd)/cc-agency:${PYTHONPATH}

cd cc-agency
COLLECTIONS="experiments batches users tokens block_entries callback_tokens"
ccagency drop-db-collections -c dev/cc-agency.yml ${COLLECTIONS}
```
