---
title: "RED Execution Engines"
permalink: /docs/red-execution-engines
---

A RED execution engine is a software, that executes an experiment based on a given RED file.
Two execution engines are implemented by the Curious Containers project:
The CC-FAICE toolsuite includes a local execution engine and CC-Agency implements a remote execution engine.
Both execution engines implement the [Docker container engine](/docs/red-container-engines).

The RED client `faice exec` is also part of the CC-FAICE toolsuite and is used to send the experiment to one of the two execution engines.


# CC-FAICE

Set `execution.engine` as `ccfaice` in your RED file `red.yml` and invoke the RED client.

```bash
faice exec red.yml
```

## Example Configuration

```yaml
execution:
  engine: "ccfaice"
  settings: {}
```


## Settings

Not available. Insert empty dict, for `ccfaice` engine settings.


# CC-Agency

Set `execution.engine` as `ccagency` in your RED file `red.yml` and invoke the RED client.

```bash
faice exec red.yml
```

The RED client will contact the CC-Agency Broker REST interface (`${url}/red`) to register the experiment on your behalf.


## Example Configuration

```yaml
execution:
  engine: "ccagency"
  settings:
    access:
      url: "https://example.com/cc"
      auth:
        username: "username"
        password: "password"
    retryIfFailed: True
    batchConcurrencyLimit: 8
```


## Settings

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| access | dict | yes | | REST interface access to register experiment |
| access.url | string | no | | The base URL of the REST interface |
| access.auth | dict | yes | | Authentication information |
| access.auth.username | string | no | | Username |
| access.auth.password | string | no | | Password |
| retryIfFailed | boolean | yes | False | Retry each batch once if the execution failed |
| batchConcurrencyLimit | integer | yes | 64 | Limit concurrently executed batches of given experiment |
