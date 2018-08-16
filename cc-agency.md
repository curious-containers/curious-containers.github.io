# CC Agency

## Install for Development

Install the latest version of Docker and docker-compose.

```bash
sudo dnf install git python3-pip python3-venv uwsgi uwsgi-plugin-python3
git clone https://github.com/curious-containers/cc-core.git
git clone https://github.com/curious-containers/cc-agency.git
cd cc-agency
pip3 install --user --upgrade poetry
poetry install
```

Run the following software components in separate terminals.

### Terminal 1 - MongoDB

```bash
docker-compose -f compose/docker-compose.yml up --build
```

The created database is stored in the local filesystem (see `compose/docker-compose.yml` for details) and will persist if the container is deleted.

A MongoDB admin user account is created automatically by a `mongo-seed` container defined in `compose/docker-compose.yml`. The credentials are read from `compose/cc-agency.yml`.

### Terminal 2 - Users

```bash
poetry run python3 -m cc_agency.tools create-broker-user -c compose/cc-agency.yml
```

You can create as many CC-Agency Broker (REST API component) user accounts as you need. Accounts are stored in MongoDB.

### Terminal 3 - CC-Agency Controller

```bash
poetry run python3 -m cc_agency.controller -c compose/cc-agency.yml
```

### Terminal 4 - CC-Agency Broker

```bash
PYTHONPATH=../cc-core:${PYTHONPATH} poetry run uwsgi --ini uwsgi.ini
```
