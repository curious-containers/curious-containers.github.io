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
  ...
  baseCommand: "command"
  ...
```

If you want to call a subcommand of a program, like `command subcommand`, you can specify a list.

```yaml
cli:
  ...
  baseCommand:
    - "command"
    - "subcommand"
  ...
```

Possible commandline arguments are defined under `cli.inputs`.

The arguments can be of type `string`, `int`, `long`, `float`, `double`, `boolean`, `File` or `Directory`, although for technical reasons CC does not distinguish between `int` and `long` or `float` and `double`. `File` and `Directory` must be valid paths, either absolute or relative. If an argument is optional, it is marked with `?`, like `File?`. If an argument can be repeated an arbitrary number of times, this can be indicated by list symbol, like `File[]`.

For example `command --optional-bool-flag --optional-int-number=32 /path/to/file /path/to/dir` can be expressed as follows.

```yaml
cli:
  ...
  inputs:
    bool_flag:
      type: "boolean?"
      inputBinding:
        prefix: "--optional-bool-flag"
    int_number:
      type: "int?"
      inputBinding:
        prefix: "--optional-int-number="
        separate: False
    some_file:
      type: "File"
      inputBinding:
        position: 0
    some_dir:
      type: "Directory"
      inputBinding:
        position: 1
  ...
```

Running a command should result in one or more files being written to the filesystem. The corresponding file paths are defined under `cli.outputs`.

We assume that the command produces a CSV table and a PDF plot file. The table file is called `table.csv` and is optional. The plot file is not optional, but its full name is not known beforehand, so we can use the glob pattern
`*.pdf`. The glob pattern must match exactly one file.


```yaml
cli:
  ...
  outputs:
    table:
      type: "File?"
      outputBinding:
        glob: "table.csv"
    plot:
      type: "File"
      outputBinding:
        glob: "*.pdf"
  ...
```


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
