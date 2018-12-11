# Source Code Setup

The following instructions have been tested on Fedora 28 with Python 3.6. Instructions for other Linux distributions should be similar.

All software components have dependencies defined in pyproject.toml and pyproject.lock files (see [PEP 518](https://www.python.org/dev/peps/pep-0518/)). [Poetry](https://poetry.eustace.io/) is used to work with these files, to automatically create virtual environments and to build and deploy the resulting Python packages.


| Table of Contents |
| --- |
| [System Dependencies](#system-dependencies) |
| [Python Dependencies](#python-dependencies) |
| [Git Repositories](#git-repositories) |
| [CC-Core](#cc-core) |
| [CC-FAICE](#cc-faice) |
| [CC-Agency](#cc-agency) |


## System Dependencies

Curious Containers requires a Linux distribution of your choice and the **latest** stable version of Docker to be installed. It is recommended to install Docker-CE from the official external Docker repository (see [instructions](https://docs.docker.com/install/linux/docker-ce/fedora/)), because Docker packages shipped by Linux distributions are often outdated.

In addition to Docker-CE, install the following system packages.

```bash
sudo dnf install git python3-pip python3-venv 
```

## Python Dependencies

Install [Poetry](https://github.com/sdispater/poetry), which will handle all the Python dependencies of each package.

```bash
curl -sSL https://raw.githubusercontent.com/sdispater/poetry/master/get-poetry.py | python
```

## Git Repositories

Clone the git repostories of the software components and place them in a folder next to each other.

```bash
git clone https://github.com/curious-containers/cc-core.git
git clone https://github.com/curious-containers/cc-faice.git
git clone https://github.com/curious-containers/cc-agency.git
```

## CC-Core

Use `poetry` to install Python dependencies in a virtual environment.

```bash
cd cc-core
poetry install
```

Run CLI modules.

```bash
poetry run ccagent --help
```

Most CLI modules in the Curious Containers ecosystem provide subcommands, which have their own `--help` flags for detailed usage information.

```bash
poetry run ccagent red --help
```

## CC-FAICE

Use `poetry` to install Python dependencies in a virtual environment.

```bash
cd cc-faice
poetry install
```

Run CLI modules.

```bash
PYTHONPATH=../cc-core poetry run faice --help
```

## CC-Agency

Install additional system packages.

```bash
sudo dnf install uwsgi uwsgi-plugin-python3 docker-compose
```

Use `poetry` to install Python dependencies in a virtual environment.

```bash
cd cc-agency
poetry install
```

Run the following components in separate terminals.

### Terminal 1 - MongoDB

Run MongoDB in a Docker container.

```bash
docker-compose -f dev/docker-compose.yml up
```

The created database is stored in the local filesystem (see `dev/docker-compose.yml` for details) and will persist if the container is deleted.

A MongoDB admin user account is created automatically by a `mongo-seed` container defined in `dev/docker-compose.yml`. The credentials are read from `dev/cc-agency.yml`.

### Terminal 2 - CC-Agency Controller

You can only run one instance of CC-Agency Controller at a time. It provides the central scheduling component, which connectes to a cluster of docker-engines.

```bash
PYTHONPATH=../cc-core poetry run ccagency-controller -c dev/cc-agency.yml
```

### Terminal 3 - CC-Agency Broker

CC-Agency Broker provides a REST API, to schedule RED experiments, receive agent callbacks and to query information. It informs the Controller about changes via a ZMQ socket. Edit `dev/uwsgi.ini` to increase the number of Broker processes or threads.

```bash
PYTHONPATH=../cc-core poetry run uwsgi --ini dev/uwsgi.ini
```

### Create Users

Create users to authenticate with the CC-Agency Broker REST API, with or without admin privileges. Admin privileges can change the behaviour of certain API endpoints.

```bash
PYTHONPATH=../cc-core poetry run ccagency create-broker-user -c dev/cc-agency.yml
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
PYTHONPATH=../cc-core poetry run ccagency drop-db-collections -c dev/cc-agency.yml ${COLLECTIONS}
```
