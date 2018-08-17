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

```bash
docker-compose -f compose/docker-compose.yml up
```

The created database is stored in the local filesystem (see `compose/docker-compose.yml` for details) and will persist if the container is deleted.

A MongoDB admin user account is created automatically by a `mongo-seed` container defined in `compose/docker-compose.yml`. The credentials are read from `compose/cc-agency.yml`.

#### Terminal 2 - Create Users

```bash
PYTHONPATH=../cc-core poetry run python3 -m cc_agency.tools create-broker-user -c compose/cc-agency.yml
```

You can create as many CC-Agency Broker (REST API component) user accounts as you need. Accounts are stored in MongoDB. Users can be added at all times, even while Controller and Broker are running.

#### Terminal 3 - CC-Agency Controller

```bash
PYTHONPATH=../cc-core poetry run python3 -m cc_agency.controller -c compose/cc-agency.yml
```

#### Terminal 4 - CC-Agency Broker

```bash
PYTHONPATH=../cc-core poetry run uwsgi --ini uwsgi-dev.ini
```
