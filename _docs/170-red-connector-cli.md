---
title: "Developing Custom Connectors"
permalink: /docs/red-connector-cli
---

This is the RED Connector CLI specification version 0.1, a pre-release of the upcomming stable version 1.

Take a look at the [RED format](https://www.curious-containers.cc/docs/red-format#inputs) documentation for an introduction to connectors.


## Subcommands

A RED Connector is a CLI tool with an arbitrary name for its executable. A connector can choose to implement a subset of functionality, which then requires certain subcommands to be present in the form of `connector-executable subcommand`. The following table lists the optional and required subcommands sorted by functionality.

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
red-connector-http receive access.json internal.json
```

access.json:
```json
{
	"url": "http://example.com/file.txt"
}
```

internal.json:
```json
{
	"path": "/tmp/myinputfile.txt"
}
```

This should result in a file created at `/tmp/myinputfile.txt` and with the content of `http://example.com/file.txt`.

`red-connector-http` is the name of the connector executable. `receive` is a subprogram, that is used to retrieve a file.
`access.json` and `internal.json` are json files describing where to access the data and where to store it.


## Receiving Files

To implement a connector called "my-connector" that is able to receive a file, this connector should at least implement the following command line calls.

```bash
# receive file
my-connector receive $access_json_file $internal_json_file

# validate access information
my-connector receive-validate $access_json_file

# remove file
my-connector receive-cleanup $internal_json_file
```

`my-connector` is the executable name of your connector. `receive`, `receive-validate` and `receive-cleanup` are subprograms of `my-connector` and they have to be written like this.
`$access_json_file` and `$internal_json_file` are paths to json files given as positional arguments.


### connector receive

The command line call `my-connector receive /path/to/access_file.json /path/to/internal_file.json` should result in a file whose path is defined in the `internal_file.json` file.
The `access_file.json` defines how to get access to the resource. The content of the access file is defined by the connector and can be different for different connectors.
The `ìnternal_file.json` defines where to store the file after downloading. It always contains a json object with one property `path`.

If it is not possible to access the data described in the access file, the connector should exit with a non zero exit code.
Additional the connector should print a human readable error message to stderr.


### connector receive-validate

The command line call `my-connector receive-validate /path/to/access_file.json` should exit with a non zero exit code, if the given access data is invalid.
The connector is not forced to check the accessibility of the described content.
Additional the connector should print a human readable error message to stderr in case of invalid access data.


### connector receive-cleanup

The command line call `my-connector receive-cleanup /path/to/internal_file.json` should remove the file defined in `internal_file.json`.
If the connector fails to remove the file, it should exit with a non zero exit code.


## Receiving Directories

To implement a connector that is able to receive a directory, this connector should at least implement the following command line calls.

```bash
# receive directory
my-connector receive-directory $access_json_file $internal_json_file --listing listing_file.json  # with listing
my-connector receive-directory $access_json_file $internal_json_file                              # without listing

# validate access information
my-connector receive-directory-validate $access_json_file

# remove directory
my-connector receive-directory-cleanup $internal_json_file
```

The `receive-directory` and `receive-directory-cleanup` calls have the same behaviour as the file receiving/removing functions, except that they create/remove a directory instead of a file.
The `receive-directory-validate` call works similar to the `receive-validate` function.
`receive-directory` and `receive-directory-validate` should accept the same access information.
`receive` and `receive-directory` can accept different access information. `receive-validate` and `receive-directory-validate` can also accept different access information.

A optional listing file in json format can be supplied. The listing has to be in cwl format (See [inputs: directories](/docs/red-format#inputs-directories)).
This listing can be ignored by a connector.


## Sending Files

To implement a connector that is able to send a file, this connector should at least implement the following command line calls.

```bash
# send file
my-connector send $access_json_file $internal_json_file

# validate access information
my-connector send-validate $access_json_file
```

The `send` call reads a file descibed in `$internal_json_file` and transfers it with the information in `$access_json_file`.
The `send-validate` call works similar to the `receive-validate` function.
