---
title: "RED Connector HTTP JSON"
permalink: /docs/red-connector-http-archive
---

This connector can be used to receive an archive file via HTTP and HTTPS connections and then extract it into a directory. This connector is a variant of [RED Connector HTTP](/docs/red-connector-http) connector.

**Current CLI version**: 0.1

## Installation

RED connector HTTP Archive is included with RED connector HTTP. For installation instructions please refer to the [corresponding documentation](/docs/red-connector-http#installation).

## Inputs

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
| archiveFormat | enum: zip, tar, gztar, bztar, xztar | no | | Archive format supported by Python 3 shutil |

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

Optional. Only used for verification.
