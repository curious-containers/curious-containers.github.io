---
title: "RED Connector HTTP"
permalink: /docs/red-connector-http
---

This connector can be used for HTTP and HTTPS connections.

**Current CLI version**: 1

## Installation

```bash
pip3 install --user --upgrade red-connector-http==1.0
```

Additionally, if you would like to use [mount-dir](#mount-dir) functionality, the HTTPDirFS CLI tools must be installed [from source](https://github.com/fangfufu/httpdirfs).

## Inputs

### receive-file

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


### receive-dir

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

#### Listing

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

They will be stored as:

* `${GENERATED_DIR}/mydata/hello.txt`
* `${GENERATED_DIR}/mydata/foo/world.json`
* `${GENERATED_DIR}/mydata/foo/bar/table.csv`

### mount-dir

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| url | string | no | | URL starting with http:// or https:// |
| auth | dict | yes | | Authentication information |
| auth.username | string | no | | Username |
| auth.password | string | no | | Password |


```yaml
command: "red-connector-httpdirfs"
mount: true
access:
  url: "http://example.com/files"
  auth:
    username: "username"
    password: "password"
```

#### Listing

Optional. Only used for verification.

## Outputs

### send-file

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| url | string | no | | URL starting with http:// or https:// |
| method | enum: GET, PUT, POST | yes | POST | HTTP method  |
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
