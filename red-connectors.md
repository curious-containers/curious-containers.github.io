# RED Connectors

RED connectors are Python modules, which implement download and upload functions to transfer data. You can install and use existing connectors or implement your own custom connectors for any protocol or connection type you need.

In a RED file you can refer to arbitrary installed connectors, which will be dynamically imported and executed as long as they correctly implement their function interfaces.

The following documentation refers to the official RED connectors.

## Table of Contents

| Connector |
| --- |
| [Introduction](#introduction) |
| [HTTP](#http) |
| [HTTP JSON](#http-json) |
| [HTTP Mock Send](#http-mock-send) |
| [SSH SFTP](#ssh-sftp) |
| [XNAT HTTP](#xnat-http) |
| [Implementing own Connectors](#implement-own-connectors) |

## Introduction

If you want to use files or directories as input for an experiment in a red file, you have to use connectors.
There are *input* connectors and *output* connectors.
If an input connector is specified in a red file, this connector will be executed before the actual program of the red file.
This will fetch some files or directories to the local filesystem to make this data available to the executed program.

Let's say you have a little program like `cat`, which simply prints the content of a file to `stdout` and
you have a file, which is online available like [this file](https://raw.githubusercontent.com/curious-containers/vagrant-quickstart/master/in.txt).
You can now specify a connector which fetches this file via http to make it accessable for your cat program.

### Input Files
The following red file specifies an experiment in which the `cat` program is used to print the content of a file:

```yml
redVersion: "5"
cli:
  cwlVersion: "v1.0"
  class: "CommandLineTool"
  baseCommand: "cat"
  doc: "Prints the content of a given file."

  inputs:
    myinputfile:
      type: File
      inputBinding:
        position: 1
  outputs: {}

inputs:
  myinputfile:
    class: File
    connector:
      pyModule: "cc_core.commons.connectors.http"
      pyClass: "Http"
      access:
        url: "https://raw.githubusercontent.com/curious-containers/vagrant-quickstart/master/in.txt"
        method: "GET"
```

As you can see we only specified a single input `myinputfile`, which is the file we are going to print.
To match this input we specify a connector with the same input key. We specify that this should fetch a file and not a directory with `class: File`.
After this we specify the connector to fetch the input file. The `pyModule: "cc_core.commons.connectors.http"` defines the python module from where to import the connector class `pyClass: Http`.
Every connector needs to know how to fetch the file, but different connectors with different protocols need different information. For example a connector using the ssh protocol doens't need a
method like the connector using HTTP. A list of the required information for each connector can be found below.

### Input Directories
The following experiment uses the `tree` command to print the directory structure.

```yml
redVersion: "5"
cli:
  cwlVersion: "v1.0"
  class: "CommandLineTool"
  baseCommand: "tree"
  doc: "Simple Test Script"

  inputs:
    myinputdirectory:
      type: Directory
      inputBinding:
        position: 1
  outputs: {}

inputs:
  myinputdirectory:
    class: 'Directory'
    connector:
      pyModule: "cc_core.commons.connectors.http"
      pyClass: "Http"
      access:
        url: "https://raw.githubusercontent.com/curious-containers/cc-core/master/cc_core/"
        method: "GET"
    listing:
      - class: 'File'
        basename: 'version.py'
      - class: 'Directory'
        basename: 'agent'
        listing:
          - class: 'File'
            basename: '__main__.py'
```

The first part of the red file is similar to the red file above. Instead of `type: File` we now have directory with `type: Directory` as input. We again use the HTTP connector.
Next to the `connector` field there is now a `listing` field. This listing defines the subfiles and subdirectories and is only allowed for directory connectors.

In this listing we have a subfile with name `version.py`. We can define as many subfiles as we want.
Also possible are subdirectories. These can have a listing field again, where again subfiles and subdirectories can be specified.
If a listing field is present for a connector, the connector should only fetch the files, which are specified in this listing. If no listing is present the hole directory should be fetched.

## HTTP

This connector can be used for HTTP and HTTPS connections.

### Installation

Not required, because this connector is included in [CC-Core](cc-core.md).

### Download and Upload

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| url | string | no | | URL starting with http:// or https:// |
| method | enum: GET, PUT, POST | no | | HTTP method  |
| auth | dict | yes | | Authentication information |
| auth.username | string | no | | Username |
| auth.password | string | no | | Password |
| auth.method | enum: BASIC, DIGEST | yes | BASIC | Authentication method |
| disableSSLVerification | boolean | yes | False | Disable verification of SSL cert |


```yaml
pyModule: "cc_core.commons.connectors.http"
pyClass: "Http"
access:
  url: "http://example.com/files/data.csv"
  method: "GET"
  auth:
    username: "username"
    password: "password"
    method: "BASIC"
  disableSSLVerification: False
```


## HTTP JSON

This connector can be used to send and receive JSON via HTTP and HTTPS connections. Only works with valid JSON files and sets the correct JSON content type, but is otherwise equivalent to the standard [HTTP](#http) connector.

### Installation

Not required, because this connector is included in [CC-Core](cc-core.md).

### Download and Upload

```yaml
pyModule: "cc_core.commons.connectors.http"
pyClass: "HttpJson"
# refer to HTTP connector for more details
```


## HTTP Mock Send

This connector can be used for HTTP and HTTPS connections. This connector is derived from the [HTTP](#http) connector and overrides the `send` method to not do anything. It is only useful for testing.

### Installation

Not required, because this connector is included in [CC-Core](cc-core.md).

### Download and Upload

```yaml
pyModule: "cc_core.commons.connectors.http"
pyClass: "HttpMockSend"
# refer to HTTP connector for more details
```


## SSH SFTP

This connector can be used with the SFTP protocol via SSH.

### Installation

```bash
pip3 install --user --upgrade red-connector-ssh==0.2
```

### Download and Upload

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| host | string | no | | Remote host domain name or IP address |
| port | integer | yes | 22 | TCP port of SSH service on remote host |
| username | string | no | | Username |
| password | string | no | | Password |
| fileDir | string | no | | File directory on remote host |
| fileName | string | no | | File name on remote host |


```yaml
pyModule: "red_connector_ssh.sftp"
pyClass: "Sftp"
access:
  host: "example.com"
  port: 22
  username: "username"
  password: "password"
  fileDir: "/home/username/files"
  fileName: "data.csv"
```

### Source

[github.com/curious-containers/red-connector-ssh](https://github.com/curious-containers/red-connector-ssh)


## XNAT HTTP

This is a special purpose connector to exchange files with the [XNAT](https://www.xnat.org/) data management system. The complicated REST API of XNAT requires multiple subsequent HTTP requests (e.g. for session management), which are handled by this connector. The given access information is combined to form actual HTTP URLs.

```bash
pip3 install --user --upgrade red-connector-xnat==0.5
```

### Download

#### Option 1:

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| baseUrl | string | no | | XNAT base URL without any subsequent path, starting with http:// or https:// |
| project | string | no | | Project ID or label |
| subject | string | no | | Subject ID or label |
| session | string | no | | Session / Experiment ID or label |
| containerType | enum: scans, reconstructions, assessors | no | | Container Type |
| container | string | no | | Container ID or label |
| resource | string | no  | | Resource ID or label |
| file | string | no | | File name |
| auth | dict | no | | Authentication information |
| auth.username | string | no | | Username |
| auth.password | string | no | | Password |
| disableSSLVerification | boolean | yes | False | Disable verification of SSL cert |


```yaml
pyModule: "red_connector_xnat.http"
pyClass: "Http"
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

#### Option 2:

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| baseUrl | string | no | | XNAT base URL without any subsequent path, starting with http:// or https:// |
| project | string | no | | Project ID or label |
| subject | string | no | | Subject ID or label |
| session | string | no | | Session / Experiment ID or label |
| resource | string | no  | | Resource ID or label |
| file | string | no | | File name |
| auth | dict | no | | Authentication information |
| auth.username | string | no | | Username |
| auth.password | string | no | | Password |
| disableSSLVerification | boolean | yes | False | Disable verification of SSL cert |


```yaml
pyModule: "red_connector_xnat.http"
pyClass: "Http"
access:
  baseUrl: "https://example.com/xnat"
  project: "project"
  subject: "subject"
  session: "session"
  resource: "resource"
  file: "scan.dat"
  auth:
    username: "username"
    password: "password"
  disableSSLVerification: False
```

### Upload

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| baseUrl | string | no | | XNAT base URL without any subsequent path, starting with http:// or https:// |
| project | string | no | | Project ID or label |
| subject | string | no | | Subject ID or label |
| session | string | no | | Session / Experiment ID or label |
| containerType | enum: scans, reconstructions, assessors | no | | Container Type |
| container | string | no | | Container ID or label |
| xsiType | string | yes | | Container xsiType, maybe required if container does not yet exist, raises exception if existing container does not match provided xsiType |
| resource | string | no  | | Resource ID or label |
| file | string | no | | File name |
| overwriteExistingFile | boolean | yes | False | Overwrite file if it already exists, otherwise raises exception if file exists |
| auth | dict | no | | Authentication information |
| auth.username | string | no | | Username |
| auth.password | string | no | | Password |
| disableSSLVerification | boolean | yes | False | Disable verification of SSL cert |


```yaml
pyModule: "red_connector_xnat.http"
pyClass: "Http"
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

### Source

[github.com/curious-containers/red-connector-xnat](https://github.com/curious-containers/red-connector-xnat)
