---
title: "RED Connectors: Input Files"
permalink: /docs/red-connectors-input-files
---

This document provides a list of available [input file](/docs/red-format#inputs) connectors and how to use them.


## HTTP

This connector can be used for HTTP and HTTPS connections.


### Access

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| url | string | no | | URL starting with http:// or https:// |
| method | enum: GET, PUT, POST | yes | GET | HTTP method  |
| auth | dict | yes | | Authentication information |
| auth.username | string | no | | Username |
| auth.password | string | no | | Password |
| auth.method | enum: BASIC, DIGEST | yes | BASIC | Authentication method |
| disableSSLVerification | boolean | yes | False | Disable verification of SSL cert |


```yaml
command: "red-connector-http"
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

```bash
pip3 install --user --upgrade red-connector-http==0.2
```


## HTTP JSON

This connector can be used to receive JSON via HTTP and HTTPS connections. Only works with valid JSON files and sets the correct JSON content type, but is otherwise equivalent to the standard [HTTP](#http) connector.


### Access

```yaml
command: "red-connector-http-json"
access: ...
# refer to HTTP connector for more details
```


### Installation

```bash
pip3 install --user --upgrade red-connector-http==0.2
```


## HTTP Mock Send

This connector can be used for HTTP and HTTPS connections. This connector is derived from the [HTTP](#http) connector and overrides the `send` method to not do anything. It is only useful for testing.


### Access

```yaml
command: "red-connector-http-mock-send"
access: ...
# refer to HTTP connector for more details
```


### Installation

```bash
pip3 install --user --upgrade red-connector-http==0.2
```


## SSH

This connector uses SSH SFTP and SSH SCP for file transfers.


**Important Security Information:**

In order to access the requested data, this connector creates a ssh connection to the host.
To make this work the connector requires a password or a valid private key to connect to the host.

Be aware that anyone who has access to this login information could potentially connect to the host.
Make sure you trust the executor of your RED file.


### Access

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| host | string | no | | Remote host domain name or IP address |
| port | integer | yes | 22 | TCP port of SSH service on remote host |
| username | string | no | | Username |
| password | string | yes | | Password |
| privateKey | string | yes | | SSH Private Key |
| passphrase | string | yes | | Passphrase for SSH Private Key |
| fileDir | string | no | | File directory on remote host |
| fileName | string | no | | File name on remote host |


If the private key is encrypted, the passphrase must be given.
If the private key is given, please make sure to copy all lines of the private key, seperated by "\n".


```yaml
command: "red-connector-ssh"
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
pip3 install --user --upgrade red-connector-ssh==0.4
```


## XNAT HTTP

This is a special purpose connector to exchange files with the [XNAT](https://www.xnat.org/) data management system. The complicated REST API of XNAT requires multiple subsequent HTTP requests (e.g. for session management), which are handled by this connector. The given access information is combined to form actual HTTP URLs.


### Access

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| baseUrl | string | no | | XNAT base URL without any subsequent path, starting with http:// or https:// |
| project | string | no | | Project ID or label |
| subject | string | yes, except if session is set | | Subject ID or label |
| session | string | yes, except if containerType is set | | Session / Experiment ID or label |
| containerType | enum: scans, reconstructions, assessors | yes, except if container is set | | Container Type |
| container | string | yes | | Container ID or label |
| resource | string | no  | | Resource ID or label |
| file | string | no | | File path |
| auth | dict | no | | Authentication information |
| auth.username | string | no | | Username |
| auth.password | string | no | | Password |
| disableSSLVerification | boolean | yes | False | Disable verification of SSL cert |


```yaml
command: "red-connector-xnat-http"
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


### Installation

```bash
pip3 install --user --upgrade red-connector-xnat==0.8
```
