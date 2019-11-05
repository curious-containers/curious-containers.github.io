---
title: "RED Connector FTP"
permalink: /docs/red-connector-ftp
---

This connector can be used for FTP connections.

**Current CLI version**: 1

# Installation

```bash
pip3 install --user --upgrade red-connector-ftp==1.0
```


# Inputs

## receive-file

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| url | string | no | | URL starting with ftp:// |


```yaml
command: "red-connector-ftp"
access:
  url: "ftp://example.com/files/data.csv"
```
