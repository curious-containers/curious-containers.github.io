# RED Format

A Reproducible Experiment Description (RED) contains all details about a data-driven experiment in YAML or JSON format. To process RED files or execute experiments use the [CC-FAICE](cc-faice.md) commandline tools.

The table below contains the top level dictionary keys of a RED file.

| Table of Contents |
| --- |
| [redVersion](#redversion) |
| [cli](#cli) |
| [inputs](#inputs) |
| [outputs](#outputs) |
| [batches](#batches) |
| [containers](#containers) |
| [execution](#execution) |


The following listings show two possible YAML structures.

Option 1:

```yaml
redVersion: ...
cli: ...
inputs: ...
outputs: ...     # optional for ccagent, faice
containers: ...  # optional for ccagent
execution: ...   # optional for ccagent, faice, ccagency
```

Option 2:

```yaml
redVersion: ...
cli: ...
batches: ...
containers: ...  # optional for ccagent
execution: ...   # optional for ccagent, faice, ccagency
```

Read through the following tutorial sections to learn more about each part of a RED file.


## redVersion

The `redVersion` increases everytime the RED format changes, even if the new version is backwards compatible. This means that a RED file in version `5` should be used with Curious Containers software packages in version `5.x.x`, higher software versions are not guaranteed to work. See [Versions](versions.md) for more details. 


## cli




## inputs


## outputs


## batches

With the `batches` keyword multiple `inputs` and `outputs` can be specified as a list. Each batch is processed independently in its own Docker container.

When the `batches` keyword is used, the `inputs` and `outputs` keywords cannot appear in the top level section of the RED file, as shown below.

```yaml
redVersion: ...
cli: ...
batches:
  - inputs: ...
    outputs: ...
  - inputs: ...
    outputs: ...
  ...
containers: ...
execution: ...
```

## containers


## execution
