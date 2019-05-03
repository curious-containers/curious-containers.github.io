---
title: "RED Connector HTTP JSON"
permalink: /docs/red-connector-http-json
---

This connector can be used to receive JSON via HTTP and HTTPS connections. Only works with valid JSON files and sets the correct JSON content type, but is otherwise equivalent to the standard [RED Connector HTTP](/docs/red-connector-http) connector.

**Current CLI version**: 0.1

## Installation

RED connector HTTP JSON is included with RED connector HTTP. For installation instructions please refer to the [corresponding documentation](/docs/red-connector-http#installation).

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
  url: "http://example.com/data"
  method: "GET"
  auth:
    username: "username"
    password: "password"
    method: "BASIC"
  disableSSLVerification: False
```

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
  url: "http://example.com/data"
  method: "GET"
  auth:
    username: "username"
    password: "password"
    method: "BASIC"
  disableSSLVerification: False
```
