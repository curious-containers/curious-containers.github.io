---
title: "RED Connector CLI 1 Specification"
permalink: /docs/red-connector-cli-1
---

This is the RED Connector CLI specification version `0.1`, a pre-release of the upcomming stable version `1`.

Take a look at the [RED format](https://www.curious-containers.cc/docs/red-format#inputs) documentation for an introduction to connectors.


## Subcommands

A RED Connector is a CLI tool with an arbitrary name for its executable. A connector can choose to implement a subset of functionality, which then requires certain subcommands to be present in the form of `example-connector subcommand`. The following table lists the optional and required subcommands sorted by functionality.

| Subcommand | Required | Functionality |
| --- | --- | --- |
| cli-version | yes | Report CLI version |
| receive-file | no | Download input file to container |
| receive-file-validate | if receive-file | |
| send-file | no | Upload output file from container |
| send-file-validate | if send-file | |
| receive-dir | no | Download input directory to container |
| receive-dir-validate | if receive-dir | |
| send-dir | no | Upload output directory from container |
| send-dir-validate | if send-dir | |
| mount-dir | no | Mount input directory in container |
| mount-dir-validate | if mount-dir | |
| umount-dir | if mount-dir | Unmount directory before container exits |


## Example

This example shows a connector cli and a typical execution of it.

Connector command line call:

```bash
red-connector-http receive access.json /tmp/myinputfile.txt
```

access.json:
```json
{
	"url": "http://example.com/file.txt"
}
```

This should result in a file created at `/tmp/myinputfile.txt` and with the content of `http://example.com/file.txt`.

`red-connector-http` is the name of the connector executable. `receive-file` is a subprogram, that is used to retrieve a file. `access.json` is a json file with information on how to access the data. The content of an access file is provided by the corresponding `connector.access` section of a RED file. The expected access information is not part of the RED specification, but instead specified by the connector implementation.


## Inputs: Receiving Files

To implement a connector called `example-connector` that is able to receive a file, this connector should at least implement the following command line calls.

```bash
# receive file
example-connector receive-file /path/to/access.json /path/to/store/input/file

# validate access information
example-connector receive-file-validate /path/to/access.json
```

`example-connector` is the executable name of your connector. `receive-file` and `receive-file-validate` are subprograms of `example-connector`. `/path/to/access.json` and `/path/to/store/input/file` are positional arguments.


### receive-file

The command line call `example-connector receive-file /path/to/access.json /path/to/store/input/file` should result in a new file stored under `/path/to/store/input/file`.
The `access.json` file defines how to get access to the resource. The content of the access file is defined by the connector and can be different for different connectors.

If it is not possible to access the data described in the access file, the connector should exit with a non zero exit code.
Additional the connector should print a human readable error message to stderr.


### receive-file-validate

The command line call `example-connector receive-file-validate /path/to/store/input/file` should exit with a non-zero exit code, if the given access data is invalid.
The connector is not forced to check the accessibility of the described content.
Additional the connector should print a human readable error message to stderr in case of invalid access data.


## Inputs: Receiving Directories

To implement a connector that is able to receive a directory, this connector should at least implement the following command line calls.

```bash
# receive directory, when a listing is provided in RED connector section
example-connector receive-dir /path/to/access.json /path/to/store/input/directory --listing /path/to/listing.json

# receive directory, when no listing is provided in RED connector section
example-connector receive-dir /path/to/access.json /path/to/store/input/directory

# validate access information, when a listing is provided in RED connector section
example-connector receive-dir-validate /path/to/access.json --listing /path/to/listing.json

# validate access information, when no listing is provided in RED connector section
example-connector receive-dir-validate /path/to/access.json
```

The `receive-dir` call has a similar behaviour as the `receive-file` call, except that it creates a directory instead of a file.
The `receive-dir-validate` call works similar to the `receive-file-validate` call.
`receive-dir` and `receive-dir-validate` should accept the same access information.

A optional listing file in json format can be supplied. The listing has to be in CWL format (See [inputs: directories](/docs/red-format#inputs-directories)). If the connector does not require a listing, it can be ignored, but the connector should be prepared accept the --listing option. If a listing is required by the connector but not provided in the RED connector section, the recive-dir-validate call should fail with a non-zero exit code.


## Outputs: Sending Files

To implement a connector that is able to send a file, this connector should at least implement the following command line calls.

```bash
# send file
example-connector send-file /path/to/access.json /path/to/output/file

# validate access information
example-connector send-file-validate /path/to/access.json
```

The `send` call transfers a file located under the given `/path/to/output/file` to the remote target as specifified in `/path/to/access.json`.
The `send-file-validate` call works similar to the `receive-file-validate` call.

TODO
