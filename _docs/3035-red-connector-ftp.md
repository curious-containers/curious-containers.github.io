---
title: "RED Connector FTP"
permalink: /docs/red-connector-ftp
---

This connector can be used for FTP connections.

**Current CLI version**: 1

# Installation

```bash
pip3 install --user --upgrade red-connector-ftp==1.1
```


# Inputs

## receive-file

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| url | string | no | | URL starting with ftp:// |
| auth.username | string | yes | anonymous | Username |
| auth.password | string | yes | | Password |


```yaml
command: "red-connector-ftp"
access:
  url: "ftp://example.com/files/data.csv"
  auth:
    username: "username"
    password: "password"
```

## receive-dir

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| url | string | no | | URL starting with ftp:// |
| auth.username | string | yes | anonymous | Username |
| auth.password | string | yes | | Password |


```yaml
command: "red-connector-ftp"
access:
  url: "ftp://example.com/files"
  auth:
    username: "username"
    password: "password"
```

### Listing

Optional. If listing exists, only the specified subdirectories and files are being transferred.

# Outputs

## send-file

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| url | string | no | | URL starting with ftp:// |
| auth.username | string | yes | anonymous | Username |
| auth.password | string | yes | | Password |


```yaml
command: "red-connector-ftp"
access:
  url: "ftp://example.com/files/data.csv"
  auth:
    username: "username"
    password: "password"
```

## send-dir

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| url | string | no | | URL starting with ftp:// |
| auth.username | string | yes | anonymous | Username |
| auth.password | string | yes | | Password |


```yaml
command: "red-connector-ftp"
access:
  url: "ftp://example.com/files"
  auth:
    username: "username"
    password: "password"
```

### Listing

Optional. If listing exists, only the specified subdirectories and files are being transfered.
