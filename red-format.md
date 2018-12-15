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
| [container](#container) |
| [execution](#execution) |


The following listings show two possible YAML structures.

Option 1:

```yaml
redVersion: ...
cli: ...
inputs: ...
outputs: ...     # optional for ccagent, faice
container: ...  # optional for ccagent
execution: ...   # optional for ccagent, faice, ccagency
```

Option 2:

```yaml
redVersion: ...
cli: ...
batches: ...
container: ...  # optional for ccagent
execution: ...   # optional for ccagent, faice, ccagency
```

If you want to know the exact jsonschema for RED supported by Curious Containers, you can install CC-FAICE and use its `faice schema` subcommands as follows.

```
pip3 install --user cc-faice
faice --version
faice schema list
faice schema show red
```

Read through the following tutorial sections to learn more about each part of a RED file.


## redVersion

The `redVersion` increases everytime the RED format changes, even if the new version is backwards compatible. This means that a RED file in version `"3"` should be used with Curious Containers software packages in version `3.x.x`, higher software versions are not guaranteed to work. See [Versions](versions.md) for more details.


## cli

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


## inputs

To run an experiment, concrete inputs for the CLI program have to be provided. This is done under the `inputs` keyword. Inputs can be primitive types like `int` or `boolean` or remote Files and Directories. Primitive types are just included in the RED file. As can be seen in the listing below, the indentifiers in `inputs` refer to the arbitrary identifiers in `cli.inputs` (e.g. `inputs.some_flag` refers to `cli.inputs.some_flag`).

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

If you want to use files or directories as input for an experiment in a RED file, you have to use so-called RED connectors. RED connectors for [input-files](red-connectors-input-files.md) and [input-directories](red-connectors-input-directories.md) exist, supporting a variety of protocols. If the available connectors do not suit you, you can implement your own in Python and easily integrate them with the [CC plugin API for connectors](developing-custom-connectors.md).

Input-connectors are executed before the actual program to download files and directories to the container file-system. The download paths are provided to the programm command TODOas CLI arguments (see section [cli](#cli)).

A connector is implemented as Python class included in a Python module. The location of the module must be included in the `PYTHONPATH` evironment variable. The can be achieved by installing the module via `pip` or by adding the path manually. Module and class must be specified under the `pyModule` and `pyClass` keywords respectively. The information provided under `access` depends on the connector.

You can now specify a connector which fetches this file via HTTP to make it accessible for your program. Details about the specific connectors can be found in the [RED Connectors: Input Files](#red-connectors-input-files.md) documentation.

```yml
inputs:
  some_file:
    class: File
    connector:
      pyModule: "cc_core.commons.connectors.http"
      pyClass: "Http"
      access:
        url: "https://raw.githubusercontent.com/curious-containers/red-guide-vagrant/master/in.txt"
        method: "GET"
```

In order to download an entire directory, some connectors like the HTTP connector require a directory `listing`. This listing defines the subfiles and subdirectories and is only allowed for directory connectors. Even for connectors which do not strictly require a listing it is recommended to include one, because it can be used to check the directory contents for missing files and subdirectories.

```yml
inputs:
  some_dir:
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

Please note, that not every connector provides functionality for files and directories, but the HTTP connector can be used in both cases. For information about these connectors take a look at the [RED Connectors: Input Directories](red-connectors-input-directories.md) documentation.


## outputs

Outputs of a data-processing application must be written to files. These files can then be uploaded to remote servers using various connectors. CC-Core includes a HTTP connector, but setting up an appropriate HTTP server, which can receive the file can be complicated. Since most Servers have SSH already configured, using the SSH SFTP connector is more convenient. For more information on how to install and use a different connector take a look at the Red Connectors for [Output Files](red-connectors-output-files.md) documentation.

Again, the output identifiers in the `outputs` section refer to the identifiers defnined under `cli.outputs`.

```yaml
cli:
  outputs:
    some_table: ...
    some_plot: ...

outputs:
  some_table: ...
  some_plot: ...
```

If you are not interested in some of the outputs, you are not required to specify a connector for them. In this case, we are only interested in `some_table` and we specify a SSH SFTP connector.

```yaml
outputs:
  some_table:
    class: File
    connector:
      pyModule: "red_connector_ssh.sftp"
      pyClass: "Sftp"
      access: ...
```


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
container: ...
execution: ...
```


## container

RED provides are generic way to include settings for container engines, such that CC or other tools can implement different engines. Curious Containers is built around Docker and its supported implementations can be found in the [RED Container Engines](red-container-engines.md) documenation.

Under the `container` keyword, you have to provide the `engine` name and the `settings` for the chosen engine.

```yaml
container:
  engine: "docker"
  settings: ...
```

When using the `faice agent red` CLI tool or when sending an experiment to CC-Agency, a container engine must be specified.


## execution

Under the `execution` keyword you can specify an execution engine, which is capable of processing the given RED file. For example the URL and access information to a CC-Agency server can be given here. For supported execution engines take a look at the [RED Execution Engines](red-execution-engines.md) documentation.

```yaml
execution:
  engine: "ccagency"
  settings: ...
```

If you have specified an execution engine you can use the `faice exec` CLI tool to execute the experiment using the given engine.

