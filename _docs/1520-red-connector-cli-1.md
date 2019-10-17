---
title: "RED Connector CLI 1 Specification"
permalink: /docs/red-connector-cli-1
---

This is the RED Connector CLI specification version `1`.
The specification takes effect with the release of RED 8 and will be supported in all future releases.
Therefore, existing container images that contain connectors following this spec will not require updates to work with upcoming RED and CC releases.

Take a look at the [RED format](https://www.curious-containers.cc/docs/red-format#inputs) documentation for an introduction to connectors.

# Connector Functionality

A connector can implement all or a subset of the following functionality:

* `receive-file`: Download an input file via the network into the container filesystem before the `baseCommand` of the actual experiment program is executed.
* `receive-dir`: Download an input directory via the network into the container filesystem before the `baseCommand` of the actual experiment program is executed.
* `mount-dir`: Mount an input directory via a FUSE network filesystem into the container filesystem before the `baseCommand` of the actual experiment program is executed.
* `send-file`: Upload an output file from the container filesystem via the network after the `baseCommand` has terminated.
* `send-directory`: Upload an output directory from the container filesystem via the network after the `baseCommand` has terminated.


# Connector Validation

If a connector implements a certain functionality, an implementation of a validation subcommand must be provided for this functionality.
The validation subcommands will be exectuted in the experiment container before any other program.
This allows connector implementation to catch potential connection problems or configuration errors done by the user before any lengthy data transmission or processing happens.


# Connector Subcommands

A RED Connector is a CLI tool with an arbitrary name for its executable.
A connector can choose to implement a subset of functionality, which then requires certain subcommands to be present in the form of `example-connector subcommand`.

The following table lists the optional and required subcommands sorted by functionality.

| Subcommand | Required | Functionality |
| --- | --- | --- |
| cli-version | yes | Report CLI version |
| receive-file | no | Download input file to container |
| receive-file-validate | if receive-file | |
| receive-dir | no | Download input directory to container |
| receive-dir-validate | if receive-dir | |
| mount-dir | no | Mount input directory in container |
| mount-dir-validate | if mount-dir | |
| umount-dir | if mount-dir | Unmount directory before container exits |
| send-file | no | Upload output file from container |
| send-file-validate | if send-file | |
| send-dir | no | Upload output directory from container |
| send-dir-validate | if send-dir | |


## Return Codes and Errors

If the validation, data transfer or directory mounting works as expected, these subcommands must terminate with return code `0`.
If an error occurs, the commands must terminate with a return code other than `0`.
This allows the execution engine to catch errors and to treat the experiment as `failed`.

In addition, the connector should print a human readable error message to stderr in case of invalid access data.


## Access Data

The information that a connector implementation requires to access remote network locations, is referred to as access data.
Since each connector is different and arbitrary connectors can be provided as part of a container image, this access data is not specified as part of RED.
Instead, abitrary access data can be embedded in the connector sections of a RED file and can only be [validated](/docs/red-connector-cli-1#connector-validation) by a connector.

The execution engine extracts the individual access data for a certain input or output connector from the RED file, stores it in a temporary file in the container file system using the JSON format.
The path to this access file is provided as a CLI argument when the connector subcommand is executed.


## Directory Listing

RED supports directory listings as defined in [CWL 1.0 - 5.1.5.1 Directory](https://www.commonwl.org/v1.0/CommandLineTool.html#Directory).
In the RED format it is optional for the user to specify a listing.
If a listing is given, the RED execution engine can use this listing to verify if the contents described in a listing are available in an input or output directory and set the experiment status to `failed` if the contents do not match.

In addition to the functionality implemented by the RED execution engine, listings can be used by a connector implementation.
Therefore, if a listing is specified for a given input or output, the RED execution engine will copy the contents into a temporary file in container file system using the JSON format.
The path to this JSON file is handed to the `receive-dir`, `receive-dir-validate`, `send-dir` or `send-dir-validate` connector subcommands via an optional `--listing` CLI argument.
Therefore these connector subcommands must be prepared to receive this argument, even if they do not use the listing data.

A specific connector functionality might require a listing being specified by the user. In this case, the respective validation subcommand, `receive-dir-validate` or `send-dir-validate` should terminate with an exit code other than `0` if the `--listing` argument is not not provided.

For the `mount-dir` and `mount-dir-validate` subcommands no `--listing` argument will be provided, because mounting a directory should not rely on this information being present.
The RED execution engine can still validate the contents of the directory after the `mount-dir` subcommand has been executed.


## Directory Mounting

RED allows mounting of input directories via FUSE file systems.
If the user sets the `mount: true` option for an input directory, the RED execution engine will automatically execute the `mount-dir-validate` and `mount-dir` subcommands of the specified connector, instead of `receive-dir` and `receive-dir-validate`.
The RED execution engine will enable the FUSE device and the required admin capabilities for the Docker container if at least one input directory with the `mount: true` option is specified.

If a connector provides the `mount-dir` functionality, the `mount-dir-validate`, `mount-dir` and `umount-dir` subcommands must be implemented.
The `umount-dir` subcommand will be executed before the container exits and allows the connector to gracefully disconnect the mounted network directory.


## Example

This example shows the execution of the `receive-file-validate` and `receive-file` subcommands.

```bash
rec-connector-http receive-file-validate /path/to/access.json
red-connector-http receive-file /path/to/access.json /tmp/myinputfile.txt
```

Example content of `access.json`:

```json
{
	"url": "http://example.com/file.txt"
}
```

This should result in a file created at `/tmp/myinputfile.txt` and with the content of `http://example.com/file.txt`.

`red-connector-http` is the name of the connector executable.
`receive-file-validate` is a subcommand that is called early during container runtime to allow the connector to validate `access.json` contents and check for potential connectivity problems.
`receive-file` is a subcommand, that downloads a file.
`access.json` is a JSON file with information on how to access the data.
The content of an access file is extracted from the corresponding `connector.access` section of a RED file.


## Subcommand Interfaces

This section shows the variations of commandline interface calls that a connector must accept if the corresponding functionality is implemented.

The name of the connector executable used in these examples is `example connector`.

### cli-version

Connectors must print their CLI version, if this subcommand is executed.

```bash
example-connector cli-version
```

A connector implementing the RED Connector CLI version 1 must print the following line when the `cli-verison` subcommand is executed.

```
1
```


### receive-file 

```bash
example-connector receive-file /path/to/access.json /path/to/store/input/file
```


### receive-file-validate

```bash
example-connector receive-file-validate /path/to/access.json
```


### receive-dir-validate

```bash
# Option 1
example-connector receive-dir-validate /path/to/access.json

# Option 2
example-connector receive-dir-validate /path/to/access.json --listing /path/to/listing.json

# Option 3
example-connector receive-dir-validate --listing /path/to/listing.json /path/to/access.json
```


### receive-dir

```bash
# Option 1
example-connector receive-dir /path/to/access.json /path/to/input/directory

# Option 2
example-connector receive-dir /path/to/access.json /path/to/input/directory --listing /path/to/listing.json

# Option 3
example-connector receive-dir --listing /path/to/listing.json /path/to/access.json /path/to/input/directory
```


### mount-dir-validate

```bash
example-connector mount-dir-validate /path/to/access.json
```


### mount-dir

```bash
example-connector mount-dir /path/to/access.json /path/to/input/directory
```


### umount-dir

```bash
example-connector umount-dir /path/to/input/directory
```


### send-file-validate

```bash
example-connector send-file-validate /path/to/access.json
```


### send-file

```bash
example-connector send-file /path/to/access.json /path/to/output/file
```


### send-dir-validate

```bash
# Option 1
example-connector send-dir-validate /path/to/access.json

# Option 2
example-connector send-dir-validate /path/to/access.json --listing /path/to/listing.json

# Option 3
example-connector --listing /path/to/listing.json send-dir-validate /path/to/access.json
```


### send-dir

```bash
# Option 1
example-connector send-dir /path/to/access.json /path/to/output/directory

# Option 2
example-connector send-dir /path/to/access.json /path/to/output/directory --listing /path/to/listing.json

# Option 3
example-connector send-dir --listing /path/to/listing.json /path/to/access.json /path/to/output/directory
```
