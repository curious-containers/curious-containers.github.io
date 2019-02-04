---
title: "RED Connectors: Input Directories"
permalink: /docs/red-connectors-input-directories
---

This document provides a list of available [input directory](/docs/red-format#inputs) connectors and how to use them.

**Listings:** Some of the connectors strictly require a directory listing. Take a look at the [inputs: directories](/docs/red-format#inputs-directories) section of the RED format documentation first.

**Mount Privileges:** Some of the connectors mount directories via [FUSE](https://de.wikipedia.org/wiki/Filesystem_in_Userspace). They require `mount: true` to be set in the `connector` section. This feature has security implications, because a Docker container needs to be started with additional sysadmin capabilities to allow mounting.

## HTTP

This connector can be used for HTTP and HTTPS connections.


### Listing

This connector requires a listing. The connector will automatically combine the provided base URL with the directories and files in the listing to form new URLs.

Example:

```yaml
inputs:
  mydata:
    class: "Directory"
    connector:
      command: "red-connector-http"
      access:
        url: "http://example.com/files"
    listing:
      - class: 'File'
        basename: 'hello.txt'
      - class: 'Directory'
        basename: 'foo'
        listing:
          - class: 'File'
            basename: 'world.json'
          - class: 'Directory'
            basename: 'bar'
            listing:
              - class: 'File'
                basename: 'table.csv'
```

This will download files from the following URLs:

* `http://example.com/files/hello.txt`
* `http://example.com/files/foo/world.json`
* `http://example.com/files/foo/bar/table.csv`

The will be stored as:

* `${GENERATED_DIR}/mydata/hello.txt`
* `${GENERATED_DIR}/mydata/foo/world.json`
* `${GENERATED_DIR}/mydata/foo/bar/table.csv`


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
pip3 install --user --upgrade red-connector-http==0.3
```


## SSH

This connector uses SSH SFTP and SSH SCP for file transfers.


**Important Security Information:**

In order to access the requested data, this connector creates a ssh connection to the host.
To make this work the connector requires a password to connect to the host.

Be aware that anyone who has access to this login information could potentially connect to the host.
Make sure you trust the executor of your RED file.


### Listing

Optional.


### Access

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| host | string | no | | Remote host domain name or IP address |
| port | integer | yes | 22 | TCP port of SSH service on remote host |
| username | string | no | | Username |
| password | string | no | | Password |
| dirName | string | no | | Directory on remote host |


```yaml
command: "red-connector-ssh"
access:
  host: "example.com"
  port: 22
  username: "username"
  password: "password"
  dirName: "/home/username/files"
```


### Installation

```bash
pip3 install --user --upgrade red-connector-ssh==0.5
```

## SSHFS

*Requires mount privileges.*

This connector uses the FUSE filesystem SSHFS to mount a remote directory via SSH.


### Listing

Optional.

### Access

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| host | string | no | | Remote host domain name or IP address |
| port | integer | yes | 22 | TCP port of SSH service on remote host |
| username | string | no | | Username |
| password | string | no | | Password |
| dirName | string | no | | Directory on remote host |


```yaml
command: "red-connector-sshfs"
mount: true
access:
  host: "example.com"
  port: 22
  username: "username"
  password: "password"
  dirName: "/home/username/files"
```

### Installation

```bash
pip3 install --user --upgrade red-connector-sshfs==0.1
```
