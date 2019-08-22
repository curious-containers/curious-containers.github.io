---
title: "RED Connector XNAT"
permalink: /docs/red-connector-xnat
---

This is a special purpose connector to exchange files with the [XNAT](https://www.xnat.org/) data management system. The complicated REST API of XNAT requires multiple subsequent HTTP requests (e.g. for session management), which are handled by this connector. The given access information is combined to form actual HTTP URLs.

**Current CLI version**: 1

## Installation

```bash
pip3 install --user --upgrade red-connector-xnat==1.0
```

## Inputs

### receive-file

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

## Outputs

### send-file

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| baseUrl | string | no | | XNAT base URL without any subsequent path, starting with http:// or https:// |
| project | string | no | | Project ID or label |
| subject | string | no | | Subject ID or label |
| session | string | no | | Session / Experiment ID or label |
| containerType | enum: scans, reconstructions, assessors | no | | Container Type |
| container | string | no | | Container ID or label |
| xsiType | string | yes | | Container xsiType, maybe required if container does not yet exist, raises exception if existing container does not match provided xsiType |
| resource | string | yes | OTHER | Resource ID or label |
| file | string | no | | File path |
| overwriteExistingFile | boolean | yes | False | Overwrite file if it already exists, otherwise raises exception if file exists |
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
  containerType: "assessors"
  container: "container"
  xsiType: "xnat:imageAssessorData"
  resource: "resource"
  file: "data.csv"
  overwriteExistingFile: False
  auth:
    username: "username"
    password: "password"
  disableSSLVerification: False
```
