---
title: "RED Format"
permalink: /docs/red-format
---

A Reproducible Experiment Description (RED) contains all details about a data-driven experiment in YAML or JSON format. To process RED files or execute experiments use the [CC-FAICE](/docs/cc-faice) commandline tools.


The following listings show two possible YAML structures.

Option 1:

```yaml
redVersion: ...
cli: ...
container: ...
inputs: ...
outputs: ...    # optional for faice
execution: ...  # optional for faice, ccagency
```

Option 2:

```yaml
redVersion: ...
cli: ...
container: ...
batches: ...
execution: ...  # optional for faice, ccagency
```

If you want to know the exact jsonschema for RED supported by Curious Containers, you can install CC-FAICE and use its `faice schema` subcommands as follows.

```
pip3 install --user cc-faice
faice --version
faice schema list
faice schema show red
```

Read through the following tutorial sections to learn more about each part of a RED file.


# redVersion

The `redVersion` increases everytime the RED format changes. This means that a RED file in version `"3"` should be used with Curious Containers software packages in version `3.x.x`, higher software versions will not work. See [Versioning](/docs/versioning) for more details.


# cli

Curious Containers only works with applications providing a proper commandline interface (CLI). This CLI, with its positional and optional arguments, must be described in the Command Workflow Language (CWL) [commandline specification](https://www.commonwl.org/v1.0/CommandLineTool.html) syntax. Other CWL compatible tools require separate `.cwl` files, but a RED file has the CWL description embedded under the `cli` keyword.

The following listing shows the main keywords of an embedded CWL description.

 ```yaml
cli:
  cwlVersion: "v1.0"        # fixed
  class: "CommandLineTool"  # fixed
  baseCommand: ...
  doc: ...                  # optional
  inputs: ...
  outputs: ...
 ```

The `baseCommand` is an executable program, which is located in a `PATH` (environment variable) directory. Usually its a string like `"command"`, where `command` is the name of the program.

```yaml
cli:
  baseCommand: "command"
  ...
```

If you want to call a subcommand of a program, like `command subcommand`, you can specify a list.

```yaml
cli:
  baseCommand:
    - "command"
    - "subcommand"
```

Possible commandline arguments are defined under `cli.inputs`.

The arguments can be of type `string`, `int`, `long`, `float`, `double`, `boolean`, `File` or `Directory`, although for technical reasons CC does not distinguish between `int` and `long` or `float` and `double`. `File` and `Directory` must be valid paths, either absolute or relative. If an argument is optional, it is marked with `?`, like `File?`. If an argument can be repeated an arbitrary number of times, this can be indicated by list symbol, like `File[]`.

 Take the following program call as an example:
 
 ```bash
 command --optional-flag --optional-number=42 /required/file /required/dir
 ```

In this case the call uses two optional arguments, one is a boolean flag, the other takes an integer number. Both `/required/file` and `/required/dir` are mandatory positional arguments, at positions `0` and `1` respectively. The CWL syntax allows us to describe the CLI capabilities of the program without specifying concrete inputs. As can be seen in the listing below, the specific values like `42` or `/required/file` are not included in the CWL description. Please note, that we need to choose an arbitrary identifier, like `some_file`, for each of the arguments.

```yaml
cli:
  inputs:
    some_flag:
      type: "boolean?"
      inputBinding:
        prefix: "--optional-flag"
    some_number:
      type: "int?"
      inputBinding:
        prefix: "--optional-number="
        separate: False
    some_file:
      type: "File"
      inputBinding:
        position: 0
    some_dir:
      type: "Directory"
      inputBinding:
        position: 1
```

Running a command should result in one or more files being written to the filesystem. The corresponding file paths are defined under `cli.outputs`. Only outputs of type `File` are allowed.

We assume that the command produces a CSV table and a PDF plot file. The table file is called `table.csv` and is optional. The plot file is not optional, but its full name is not known beforehand, so we can use the glob pattern
`*.pdf`. **In the context of CC, any glob pattern must match exactly one file in the output directory.**

```yaml
cli:
  outputs:
    some_table:
      type: "File?"
      outputBinding:
        glob: "table.csv"
    some_plot:
      type: "File"
      outputBinding:
        glob: "*.pdf"
```


# inputs

To run an experiment, concrete inputs for the CLI program have to be provided.
This is done under the `inputs` keyword.
Inputs can be primitive types like `int` or `boolean` or remote Files and Directories.
Primitive types are just included in the RED file.
As can be seen in the listing below, the identifiers in `inputs` refer to the arbitrary identifiers in `cli.inputs` (e.g. `inputs.some_flag` refers to `cli.inputs.some_flag`).

```yml
cli:
  inputs:
    some_flag: ...
    some_number: ...
    some_file: ...
    some_dir: ...

inputs:
  some_flag: True
  some_number: 42
  some_file: ...
  some_dir: ...
```

To make files and directories available as input for an experiment, you have to use RED connectors, that support a variety of protocols like [SSH](/docs/red-connector-ssh), [HTTP](/docs/red-connector-http).
If the available connectors do not suit you, you can implement your own following the [RED Connector CLI 1](/docs/red-connector-cli-1) specification.
Input files are downloaded automatically into a running container. The download paths are provided to the programm command as CLI arguments (see section [cli](#cli)).

The `command` keyword refers to name of the connector executable that is made available to system via the `PATH` environment variable.
The information provided under `access` depends on the connector. Refer to the documentation of a specific connector for more information on how to specify `access` data correctly.

The following example uses the HTTP connector to download a single input file.

```yml
inputs:
  some_file:
    class: "File"
    connector:
      command: "red-connector-http"
      access:
        url: "https://raw.githubusercontent.com/curious-containers/red-guide-vagrant/master/in.txt"
```

## inputs: directories

In order to download an entire directory, some connectors like the HTTP connector require a directory `listing`. This listing defines the subfiles and subdirectories and is only allowed for directory connectors. Even for connectors which do not strictly require a listing it is recommended to include one, because it will be used to automatically check the directory contents for missing files and subdirectories.

```yml
inputs:
  some_dir:
    class: "Directory"
    connector:
      command: "red-connector-http"
      access:
        url: "https://raw.githubusercontent.com/curious-containers/cc-core/master/cc_core/"
    listing:
      - class: "File"
        basename: "version.py"
      - class: "Directory"
        basename: "agent"
        listing:
          - class: "File"
            basename: "__main__.py"
```

Please note, that not every connector provides functionality for input files and directories, but the HTTP and SSH connectors can be used in both cases.


# outputs

Outputs of an experiment can then be uploaded to remote servers using various connectors.

Again, the output identifiers in the `outputs` section refer to the arbitrary identifiers defined under `cli.outputs`.

```yaml
cli:
  outputs:
    some_table: ...
    some_plot: ...

outputs:
  some_table: ...
  some_plot: ...
```

If you are not interested in some of the outputs, you are not required to specify a connector for them. In this case, we are only interested in `some_table` and we specify an SSH connector.

```yaml
outputs:
  some_table:
    class: "File"
    connector:
      command: "red-connector-ssh"
      access:
        host: "example.com"
        port: 22
        auth:
          username: "username"
          password: "password"
        filePath: "/home/username/files/out.txt"
```


# batches

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
container: ...
execution: ...
```


# container

RED provides are generic way to include settings for container engines, such that CC or other tools can implement different engines. Curious Containers currently only supports Docker as a [RED Container Engine](/docs/red-container-engines).

Under the `container` keyword, you have to provide the `engine` name and the `settings` for the chosen engine.

```yaml
container:
  engine: "docker"
  settings: ...
```


# execution

Under the `execution` keyword you can specify an execution engine, which is capable of processing the given RED file. For example the URL and access information to a CC-Agency server can be given here. For supported execution engines take a look at the [RED Execution Engines](/docs/red-execution-engines) documentation.

```yaml
execution:
  engine: "ccagency"
  settings: ...
```

If you have specified an execution engine you can use the `faice exec` CLI tool to execute the experiment using the given engine.

