---
title: "RED Connector SSH"
permalink: /docs/red-connector-ssh
---

RED Connector SSH is currently the only complete connector implementation for Curious Containers.

**Current CLI version**: 1


## Important Security Information

In order to access the requested data, this connector creates a ssh connection to the host.
To make this work the connector requires a password or a valid private key to connect to the host.

Be aware that anyone who has access to this login information could potentially connect to the host.
Make sure you trust the executor of your RED file.

# Installation

```bash
pip3 install --user --upgrade red-connector-ssh==1.1
```

Additionally, if you would like to use [mount-dir](#mount-dir) functionality, the SSHFS CLI tools must be installed.

```bash
# on Debian / Ubuntu
apt-get install sshfs
```

```bash
# on Fedora
dnf install sshfs
```

# Inputs

## receive-file

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| host | string | no | | Remote host domain name or IP address |
| port | integer | yes | 22 | TCP port of SSH service on remote host |
| auth.username | string | no | | Username |
| auth.password | string | yes, if auth.privateKey | | Password |
| auth.privateKey | string | yes, if auth.password | | SSH Private Key |
| auth.passphrase | string | yes | | Passphrase for SSH Private Key |
| filePath | string | no | | File path on remote host |


If the privateKey is encrypted, the passphrase must be given. If the private key is given, please make sure to copy all lines of the private key, seperated by "\n".


```yaml
command: "red-connector-ssh"
access:
  host: "example.com"
  port: 22
  auth:
    username: "username"
    password: "password"
  filePath: "/home/username/files/data.csv"
```

## receive-dir

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| host | string | no | | Remote host domain name or IP address |
| port | integer | yes | 22 | TCP port of SSH service on remote host |
| auth.username | string | no | | Username |
| auth.password | string | yes, if auth.privateKey | | Password |
| auth.privateKey | string | yes, if auth.password | | SSH Private Key |
| auth.passphrase | string | yes | | Passphrase for SSH Private Key |
| dirPath | string | no | | Directory path on remote host |


If the privateKey is encrypted, the passphrase must be given. If the private key is given, please make sure to copy all lines of the private key, seperated by "\n".


```yaml
command: "red-connector-ssh"
access:
  host: "example.com"
  port: 22
  auth:
    username: "username"
    password: "password"
  dirPath: "/home/username/files"
```

### Listing

Optional. If listing exists, only the specified subdirectories and files are being transferred.

## mount-dir

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| host | string | no | | Remote host domain name or IP address |
| port | integer | yes | 22 | TCP port of SSH service on remote host |
| auth.username | string | no | | Username |
| auth.password | string | yes, if auth.privateKey | | Password |
| auth.privateKey | string | yes, if auth.password | | SSH Private Key |
| auth.passphrase | string | yes | | Passphrase for SSH Private Key |
| dirPath | string | no | | Directory path on remote host |
| writable | boolean | yes | false | Enable write access to mounted directory |


```yaml
command: "red-connector-ssh"
mount: true
access:
  host: "example.com"
  port: 22
  auth:
    username: "username"
    password: "password"
  dirPath: "/home/username/files"
  writable: false
```

### Listing

Optional. Only used for verification.

# Outputs

## send-file

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| host | string | no | | Remote host domain name or IP address |
| port | integer | yes | 22 | TCP port of SSH service on remote host |
| auth.username | string | no | | Username |
| auth.password | string | yes, if auth.privateKey | | Password |
| auth.privateKey | string | yes, if auth.password | | SSH Private Key |
| auth.passphrase | string | yes | | Passphrase for SSH Private Key |
| filePath | string | no | | File path on remote host |


If the privateKey is encrypted, the passphrase must be given. If the private key is given, please make sure to copy all lines of the private key, seperated by "\n".


```yaml
command: "red-connector-ssh"
access:
  host: "example.com"
  port: 22
  auth:
    username: "username"
    password: "password"
  filePath: "/home/username/files/data.csv"
```

## send-dir

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| host | string | no | | Remote host domain name or IP address |
| port | integer | yes | 22 | TCP port of SSH service on remote host |
| auth.username | string | no | | Username |
| auth.password | string | yes, if auth.privateKey | | Password |
| auth.privateKey | string | yes, if auth.password | | SSH Private Key |
| auth.passphrase | string | yes | | Passphrase for SSH Private Key |
| dirPath | string | no | | File path on remote host |


If the privateKey is encrypted, the passphrase must be given. If the private key is given, please make sure to copy all lines of the private key, seperated by "\n".


```yaml
command: "red-connector-ssh"
access:
  host: "example.com"
  port: 22
  auth:
    username: "username"
    password: "password"
  dirPath: "/home/username/files"
```

### Listing

Optional. If listing exists, only the specified subdirectories and files are being transfered.
