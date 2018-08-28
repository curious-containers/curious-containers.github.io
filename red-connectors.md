# RED Connectors

RED connectors are Python modules, which implement download and upload functions to transfer data. You can install and use existing connectors or implement your own custom connectors for any protocol or connection type you need.

In a RED file you can refer to arbitrary installed connectors, which will be dynamically imported and executed as long as they correctly implement their function interfaces.

The following documentation refers to the official RED connectors.

## Table of Contents

| Connector |
| --- |
| [HTTP](#http) |
| [SSH SFTP](#ssh-sftp) |
| [XNAT HTTP](#xnat-http) |


## HTTP

This connector can be used for HTTP and HTTPS connections.

### Installation

Not required, because this connector is included in CC-Core.

### Download and Upload

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| url | string | no | | URL starting with http:// or https:// |
| method | enum GET, PUT, POST | no | | HTTP method  |
| auth | dict | yes | | Authentication information |
| auth.username | string | no | | Username |
| auth.password | string | no | | Password |
| auth.method | enum BASIC, DIGEST | yes | BASIC | Authentication method |
| disableSSLVerification | boolean | yes | True | Disable verification of SSL cert |


```yaml
pyModule: "cc_core.commons.connectors.http"
pyClass: "Http"
access:
  url: "http://example.com/files/data.csv"
  method: "GET"
  auth:
    username: "username"
    password: "password"
    method: "BASIC"
  disableSSLVerification: False
```


## SSH SFTP

This connector can be used with the SFTP protocol via SSH.

### Installation

```bash
pip3 install --user --upgrade red-connector-ssh==0.2
```

### Download and Upload

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| host | string | no | | Remote host domain name or IP address |
| port | integer | yes | 22 | TCP port of SSH service on remote host |
| username | string | no | | Username |
| password | string | no | | Password |
| fileDir | string | no | | File directory on remote host |
| fileName | string | no | | File name on remote host |


```yaml
pyModule: "red_connectors_ssh.sftp"
pyClass: "Sftp"
access:
  host: "example.com"
  port: 22
  username: "username"
  password: "password"
  fileDir: "/home/username/files"
  fileName: "data.csv"
```


## XNAT HTTP

```bash
pip3 install --user --upgrade red-connector-xnat==0.3
```

### Download

TODO

### Upload

TODO
