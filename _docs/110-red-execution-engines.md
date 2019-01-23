---
title: "RED Execution Engines"
permalink: /docs/red-execution-engines
---

A RED execution engine is a software, which executes an experiment based on a given RED file. Two execution engines are
implemented by the Curious Containers project: the `faice agent red` command included in the CC-FAICE package and
CC-Agency.

Both execution engines implement the `docker` and `nvidia-docker` [Container Engines](/docs/red-container-engines).

Use `faice exec` CLI tool to automatically invoke the engine specified in your RED file for execution.


## CC-FAICE

Set `execution.engine` as `ccfaice`in your RED file, then use `faice exec red.yml`. This is equivalent to the following
command.

```bash
faice agent red --debug --outputs red.yml
```

If you want more control over `faice agent red` you should invoke it directly.


### Example Configuration

```yaml
execution:
  engine: "ccfaice"
  settings: {}
```


### Settings

Not available.


## CC-Agency

Set `execution.engine` as `ccagency`, which allows `faice exec` to contact the CC-Agency Broker REST interface
(`${url}/red`) to register the experiment on your behalf.


### Example Configuration

```yaml
execution:
  engine: "ccagency"
  settings:
    access:
      url: "https://example.com/cc"
      auth:
        username: "username"
        password: "password"
    retryIfFailed: True,
    batchConcurrencyLimit: 8
    
```


### Settings

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| access | dict | yes | | REST interface access to register experiment |
| access.url | string | no | | The base URL of the REST interface |
| access.auth | dict | yes | | Authentication information |
| access.auth.username | string | no | | Username |
| access.auth.password | string | no | | Password |
| retryIfFailed | boolean | yes | False | Retry each batch once if the execution failed |
| batchConcurrencyLimit | integer | yes | 64 | Limit concurrently executed batches of given experiment |
