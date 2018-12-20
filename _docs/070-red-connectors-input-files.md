---
title: "RED Connectors: Input Files"
permalink: /docs/red-connectors-input-files
---

This document provides a list of available [input file](/docs/red-format#inputs) connectors and how to use them.


## HTTP

This connector can be used for HTTP and HTTPS connections.


### Usage

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| url | string | no | | URL starting with http:// or https:// |
| method | enum: GET, PUT, POST | no | | HTTP method  |
| auth | dict | yes | | Authentication information |
| auth.username | string | no | | Username |
| auth.password | string | no | | Password |
| auth.method | enum: BASIC, DIGEST | yes | BASIC | Authentication method |
| disableSSLVerification | boolean | yes | False | Disable verification of SSL cert |


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


### Installation

Not required, because this connector is included in [CC-Core](/docs/cc-core-cc-faice-cc-agency#cc-core).


## HTTP JSON

This connector can be used to receive JSON via HTTP and HTTPS connections. Only works with valid JSON files and sets the correct JSON content type, but is otherwise equivalent to the standard [HTTP](#http) connector.


### Usage

```yaml
pyModule: "cc_core.commons.connectors.http"
pyClass: "HttpJson"
# refer to HTTP connector for more details
```


### Installation

Not required, because this connector is included in [CC-Core](/docs/cc-core-cc-faice-cc-agency#cc-core).


## HTTP Mock Send

This connector can be used for HTTP and HTTPS connections. This connector is derived from the [HTTP](#http) connector and overrides the `send` method to not do anything. It is only useful for testing.


### Usage

```yaml
pyModule: "cc_core.commons.connectors.http"
pyClass: "HttpMockSend"
# refer to HTTP connector for more details
```


### Installation

Not required, because this connector is included in [CC-Core](/docs/cc-core-cc-faice-cc-agency#cc-core).


## SSH SFTP

This connector can be used with the SFTP protocol via SSH.


### Usage

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| host | string | no | | Remote host domain name or IP address |
| port | integer | yes | 22 | TCP port of SSH service on remote host |
| username | string | no | | Username |
| password | string | no | | Password |
| fileDir | string | no | | File directory on remote host |
| fileName | string | no | | File name on remote host |


```yaml
pyModule: "red_connector_ssh.sftp"
pyClass: "Sftp"
access:
  host: "example.com"
  port: 22
  username: "username"
  password: "password"
  fileDir: "/home/username/files"
  fileName: "data.csv"
```


### Installation

```bash
pip3 install --user --upgrade red-connector-ssh==0.2
```


## XNAT HTTP

This is a special purpose connector to exchange files with the [XNAT](https://www.xnat.org/) data management system. The complicated REST API of XNAT requires multiple subsequent HTTP requests (e.g. for session management), which are handled by this connector. The given access information is combined to form actual HTTP URLs.


### Usage: Option 1

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| baseUrl | string | no | | XNAT base URL without any subsequent path, starting with http:// or https:// |
| project | string | no | | Project ID or label |
| subject | string | no | | Subject ID or label |
| session | string | no | | Session / Experiment ID or label |
| containerType | enum: scans, reconstructions, assessors | no | | Container Type |
| container | string | no | | Container ID or label |
| resource | string | no  | | Resource ID or label |
| file | string | no | | File name |
| auth | dict | no | | Authentication information |
| auth.username | string | no | | Username |
| auth.password | string | no | | Password |
| disableSSLVerification | boolean | yes | False | Disable verification of SSL cert |


```yaml
pyModule: "red_connector_xnat.http"
pyClass: "Http"
access:
  baseUrl: "https://example.com/xnat"
  project: "project"
  subject: "subject"
  session: "session"
  containerType: "scans"
  container: "container"
  resource: "resource"
  file: "scan.dat"
  auth:
    username: "username"
    password: "password"
  disableSSLVerification: False
```


### Usage: Option 2

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| baseUrl | string | no | | XNAT base URL without any subsequent path, starting with http:// or https:// |
| project | string | no | | Project ID or label |
| subject | string | no | | Subject ID or label |
| session | string | no | | Session / Experiment ID or label |
| resource | string | no  | | Resource ID or label |
| file | string | no | | File name |
| auth | dict | no | | Authentication information |
| auth.username | string | no | | Username |
| auth.password | string | no | | Password |
| disableSSLVerification | boolean | yes | False | Disable verification of SSL cert |


```yaml
pyModule: "red_connector_xnat.http"
pyClass: "Http"
access:
  baseUrl: "https://example.com/xnat"
  project: "project"
  subject: "subject"
  session: "session"
  resource: "resource"
  file: "scan.dat"
  auth:
    username: "username"
    password: "password"
  disableSSLVerification: False
```

### Installation

```bash
pip3 install --user --upgrade red-connector-xnat==0.5
```
