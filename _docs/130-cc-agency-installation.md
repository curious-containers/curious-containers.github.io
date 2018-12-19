---
title: "CC-Agency: Installation"
permalink: /docs/cc-agency-installation
---

The following instructions have been tested on Ubuntu 18.04 with Python 3.6. Instructions for other Linux distributions should be similar.

### System Packages

*As admin user.*

```bash
sudo apt-get update
sudo apt-get install python3-pip python3-venv uwsgi uwsgi-plugin-python3
sudo apt-get install apache2 libapache2-mod-proxy-uwsgi
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

If this tool cannot be found, you should modify `PATH` (e.g. append `${HOME}/.local/bin`).

### UWSGI Configuration

*As admin user.*

```bash
sudo mkdir -p /opt/ccagency/privileged /opt/ccagency/unprivileged
sudo chown www-data:www-data /opt/ccagency/unprivileged
```

Create `/opt/ccagency/privileged/ccagency-broker.ini` for uwsgi. The `wsgi-file` path may vary for your version of Python 3.

```ini
[uwsgi]
plugins = python3
chown-socket = www-data:www-data
socket = /opt/ccagency/unprivileged/ccagency-broker.sock
wsgi-file = /home/cc/ccagency-venv/lib/python3.5/site-packages/cc_agency/broker/app.py
uid = cc
gid = cc
processes = 4
lazy-apps = True

if-env = VIRTUAL_ENV
virtualenv = %(_)
endif =
```

### Python Packages

*As cc user.*

Install Python packages for user `cc`.

```bash
python3 -m venv ~/ccagency-venv
source ~/ccagency-venv/bin/activate
pip install wheel
pip install cc-agency
```

Run CLI tool.

```bash
ccagency --help
```


### CC-Agency Configuration

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
      url: "docker.io/curiouscontainers/cc-core:5.4.0"
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
ExecStart=/home/cc/ccagency-venv/bin/ccagency-controller
Restart=no
Environment=PYTHONUNBUFFERED=1

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
ExecStart=/usr/bin/uwsgi /opt/ccagency/privileged/ccagency-broker.ini
Restart=no
Environment=VIRTUAL_ENV=/home/cc/ccagency-venv

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

    ProxyRequests Off
    ProxyPass /cc unix:/opt/ccagency/unprivileged/ccagency-broker.sock|uwsgi://ccagency-broker/
</VirtualHost>
```

Enable Apache2 mods and site.

```bash
sudo a2enmod ssl
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
