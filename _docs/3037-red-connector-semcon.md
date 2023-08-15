---
title: "RED Connector Semcon"
permalink: /docs/red-connector-semcon
---

This connector can be used for connections to [Semantic Containers](https://www.ownyourdata.eu/en/semcon/).

**Current CLI version**: 1

# Installation

```bash
pip3 install --user --upgrade red-connector-semcon==3.0
```


# Inputs

## receive-file

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| url | string | no | | URL starting with http:// or https:// |
| auth.username | string | yes | | Username |
| auth.password | string | no, if auth.username | | Password |
| auth.scope | string | yes | read | OAuth Scope |
| resource.resourceType | string | no | | Type of the resource ('id', 'dri', 'schema_dri', 'table') |
| resource.resourceValue | string | no | | Name of the resource |
| resource.key | string | yes | | JSON key |
| resource.format | string | yes | | Output format ('plain', 'full', 'meta', 'validation') |
| disableSSLVerification | boolean | yes | False | Disable verification of SSL cert |


```yaml
command: "red-connector-semcon"
access:
  url: "http://example.com/api/data"
  auth:
    username: "username"
    password: "password"
    scope: "read"
  resource:
    resourceType: "id"
    resourceValue: "1"
    key: "foo"
    format: "plain"
  disableSSLVerification: false
```

# Outputs

## send-file

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| url | string | no | | URL starting with http:// or https:// |
| auth.username | string | yes | | Username |
| auth.password | string | no, if auth.username | | Password |
| auth.scope | string | yes | read | OAuth Scope |
| resource.dri | string | yes | | Name of the dri |
| resource.schemaDri | string | yes | | Name of the schema |
| resource.tableName | string | yes | | Name of the table |
| resource.key | string | yes | | JSON key |
| disableSSLVerification | boolean | yes | False | Disable verification of SSL cert |


```yaml
command: "red-connector-semcon"
access:
  url: "http://example.com/api/data"
  auth:
    username: "username"
    password: "password"
    scope: "write"
  resource:
    dri: "0"
    schemaDri: "bar"
    tableName: "default"
    key: "foo"
  disableSSLVerification: false
```
