---
title: "RED Connector FTP Archive"
permalink: /docs/red-connector-ftp-archive
---

This connector can be used to receive an archive file via an FTP connection and then extract it into a directory. This connector is a variant of [RED Connector FTP](/docs/red-connector-ftp).

**Current CLI version**: 1

# Installation

RED connector FTP Archive is included with RED connector FTP. For installation instructions please refer to the [corresponding documentation](/docs/red-connector-ftp#installation).

# Inputs

## receive-dir

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| url | string | no | | URL starting with ftp:// |
| archiveFormat | enum: zip, tar, gztar, bztar, xztar | no | | Archive format supported by Python 3 shutil |

```yaml
command: "red-connector-ftp"
access:
  url: "ftp://example.com/files/data.tar.gz"
  archiveFormat: "gztar"
```

### Listing

Optional. Only used for verification.
