# CC-Agency

CC-Agency is an advanced server software, which is able to connect to a distributed cluster of docker-engines and schedules experiments defined in RED format for parallel execution.

It implements two major software components: CC-Agency Broker provides a restful web API, to register experiments and to query information. CC-Agency Controller is a background process, which connects to a Docker cluster and schedules the experiments for execution.

CC-Agency persists the state of experiments in MongoDB and is very fault tolerant, when it comes to failing containers, docker-engines or network connections.

## User Information

If you are a **user** and would like to connect to an existing installation of CC-Agency, please refer to the [REST API](cc-agency-api.md) documentation.

If you are new to RED and Curious Containers, please work through the [RED Beginner's Guide](#red-beginners-guide.md) first.

It is advised to test and validate your RED files and containers locally with [CC-FAICE](cc-faice.md).

In order to make your **resources** referenced in a RED file, like a Docker image and input/output file locations, available to a remote Docker cluster, they have to be **published** in an appropriate way. This means that Docker images have to be uploaded to a Docker registry, like the official [DockerHub](https://hub.docker.com/) or a private [registry](https://docs.docker.com/registry/), and that data exchange must be handled through a data management system (DMS) via the network. We provide [RED connector](#red-connectors.md) plugins for secure file exchange protocols and the [XNAT](https://www.xnat.org/) DMS. Of course, as demanded by the [FAIR princibles](https://www.force11.org/fairprinciples), common authentication mechanisms are supported for Docker registry and data connections.

As a positive side-effect, your experiments do not rely on any local files, which allows you to share and store your RED files in a **reproducible** way.

## Data Security

As a technical necessity, you are required to send your Docker image and data access credentials defined in your RED file to CC-Agency. It is therefore strongly advised to only use CC-Agency if you **trust the operator** with this sensitive data.

To improve data protection you should use **temporary credentials** whenever possible.

CC-Agency will store credentials temporarily in its database. As soon as the processing of a batch or experiment finishes, the values of **protected keys** under the `access` data in the connector sections and under the `auth` data in the engine sections are **overwritten** in the database. By default, only `password` is considered a protected key. You can protect other keys if they refer to string values, by prepending `_` (e.g. `_username`) in the RED data (see [Protected Keys](red-format.md#protected-keys)), before posting it to CC-Agency.

All data handling is done inside of Docker containers. The container file systems are deleted after processing, leaving no traces of your data on remote servers.

## Installation

The following instructions have been tested on Ubuntu 18.04 with Python 3.6. Instructions for other Linux distributions should be similar.

### System Packages

*As admin user.*

```bash
sudo apt-get update
sudo apt-get install python3-pip uwsgi uwsgi-plugin-python3
sudo apt-get install apache2 libapache2-mpm-itk libapache2-mod-proxy-uwsgi
```

### MongoDB

*As admin user.*

Install MongoDB 4.0. Instructions can be found in the
[MongoDB documentation](https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/).

After the installation of the required system packages, enable and start the service.

```bash
sudo systemctl enable mongod
sudo systemctl start mongod
```

### System User

*As admin user.*

Create a new system user called `cc`. CC-Agency will run under the privileges of this user.

```bash
sudo useradd -ms /bin/bash cc
```

### Python Packages

*As cc user.*

Install Python packages for user `cc`.

```bash
pip3 install --user --upgrade cc-agency==5.3.4
source ~/.profile
```

Run CLI tool.

```bash
ccagency --help
```

If this tool cannot be found, you should modify `PATH` (e.g. append `${HOME}/.local/bin`).

### Configuration

*As cc user.*

Create configuration file `~/.config/cc-agency.yml`. Copy the following content, but choose a new strong `mongo.password` and save it in the file. The parameter `broker.external_url` should match the domain name of your server, as we will later define in the Apache2 site configuration.

```yaml
broker:
  external_url: "https://example.com/cc"
  auth:
    num_login_attempts: 3
    block_for_seconds: 30
    tokens_valid_for_seconds: 86400  # 24 h

controller:
  external_url: "tcp://127.0.0.1:6001"
  bind_host: "127.0.0.1"
  bind_port: 6001
  docker:
    core_image:
      url: "docker.io/curiouscontainers/cc-core:5.3.1"
      disable_pull: False
    nodes: {}
  scheduling:
    strategy: 'spread'
    attempts_to_fail: 3

mongo:
  db: "ccagency"
  username: "ccadmin"
  password: "SECRET"
```

Create `~/.config/cc-agency-broker.ini` for uwsgi.

```ini
[uwsgi]
plugins = python3
socket = /home/cc/.cache/cc-agency-broker.sock
wsgi-file = /home/cc/.local/lib/python3.6/site-packages/cc_agency/broker/app.py
uid = cc
gid = cc
processes = 4
lazy-apps = True
```

### MongoDB User

*As cc user.*

Use the `ccagency` CLI tool, to create a new MongoDB user as specified in the `cc-agency.yml` configuration file.

```bash
ccagency create-db-user
```

### Broker User

*As cc user.*

Run the interactive `ccagency` CLI tool, to create at least one CC-Agency Broker user. Users created with this script can authenticate with the Broker REST API.

```bash
ccagency create-broker-user
```

Additional users can be added at all times.

### Systemd Units

*As admin user.*

Create Systemd unit file `/etc/systemd/system/ccagency-controller.service` for CC-Agency Controller.

```ini
[Unit]
Description=CC-Agency Controller
Documentation=https://www.curious-containers.cc/
Requires=mongod.service
After=mongod.service

[Service]
Type=simple
User=cc
Group=cc
ExecStart=/home/cc/.local/bin/ccagency-controller
Restart=no

[Install]
WantedBy=multi-user.target
```

Enable and start `ccagency-controller` service.

```bash
sudo systemctl enable ccagency-controller
sudo systemctl start ccagency-controller
```

Create Systemd unit file `/etc/systemd/system/ccagency-broker.service` for CC-Agency Broker.

```ini
[Unit]
Description=CC-Agency Broker
Documentation=https://www.curious-containers.cc/
Requires=ccagency-controller.service mongod.service apache2.service
After=ccagency-controller.service mongod.service apache2.service

[Service]
Type=simple
User=cc
Group=cc
ExecStart=/usr/bin/uwsgi /home/cc/.config/cc-agency-broker.ini
Restart=no

[Install]
WantedBy=multi-user.target
```

Enable and start `ccagency-broker` service.

```bash
sudo systemctl enable ccagency-broker
sudo systemctl start ccagency-broker
```

Create apache2 site `/etc/apache2/sites-available/ccagency-broker.conf`. Change `SSLCertificateFile` and `SSLCertificateKeyFile` paths or switch to an unencrypted configuration, which is not recommended. The server name should match the domain of your server. The Broker will listen on `https://example.com/cc`, as also defined under `broker.external_url` in the `cc-agency.yml` file.

```apache
Listen 443

<VirtualHost *:443>
    ServerName example.com

    SSLEngine on
    SSLCertificateFile /opt/ssl/cert.pem
    SSLCertificateKeyFile /opt/ssl.key.pem

    AssignUserId cc cc

    ProxyRequests Off
    ProxyPass /cc unix:/home/cc/.cache/cc-agency-broker.sock|uwsgi://ccagency-broker/
</VirtualHost>
```

Enable Apache2 mods and site.

```bash
sudo a2enmod ssl
sudo a2enmod mpm_itk
sudo a2enmod proxy_uwsgi

sudo a2ensite ccagency-broker

sudo systemctl restart apache2
```

### Docker Cluster

*As cc user.*

Edit `~/.config/cc-agency.yml` and add the individual docker-machines in the `controller.docker.nodes` dictionary. If you are connecting to remote machines, the docker-engines must listen on a TCP port (see [Docker documentation](https://docs.docker.com/edge/engine/reference/commandline/dockerd/#daemon-socket-option)). Using TLS is optional, but recommended. If you are not using TLS, remove the corresponding `tls` sections from the config.

```yaml
controller:
  docker:
    nodes:
      node1:
        base_url: "tcp://192.168.0.100:2376"
        tls:
          verify: "/home/cc/.docker/machine/machines/node1/ca.pem"
          client_cert:
            - "/home/cc/.docker/machine/machines/node1/cert.pem"
            - "/home/cc/.docker/machine/machines/node1/key.pem"
          assert_hostname: False
      node2:
        base_url: "tcp://192.168.0.101:2375"
      node3:
        base_url: "unix://var/run/docker.sock"
```

*As admin user.*

Restart CC-Agency Controller:

```bash
sudo systemctl restart ccagency-controller
```

### Status and Logging

*As admin user.*

Use the following commands to inspect the status and log files of the configured software components.

```bash
# MongoDB
sudo systemctl status mongod
sudo journalctl -u mongod

# Apache2
sudo systemctl status apache2
sudo journalctl -u apache2
sudo less /var/log/apache2/error.log
sudo less /var/log/apache2/access.log

# CC-Agency Controller
sudo systemctl status ccagency-controller
sudo journalctl -u ccagency-controller

# CC-Agency Broker
sudo systemctl status ccagency-broker
sudo journalctl -u ccagency-broker
```

### Source

[github.com/curious-containers/cc-agency](https://github.com/curious-containers/cc-agency)
