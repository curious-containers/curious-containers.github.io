---
title: "Developing Custom Connectors"
permalink: /docs/developing-custom-connectors
---

This text explains how to implement custom connectors. See [here](https://www.curious-containers.cc/docs/red-connectors-input-files) for an introduction to RED connectors.

A RED connector is a command line program, that can be used to receive files/directories or to send files.
To make a connector work with cc-core and faice it has to implement a command line interface defined in the next sections.

A RED connector takes access information as input, defining from where to receive data or where to send it.


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
my-connector receive-validate access_file.json

# remove file
my-connector receive-cleanup internal.json
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
my-connector receive-directory access_file.json internal_file.json --listing listing_file.json  # with listing
my-connector receive-directory access_file.json internal_file.json                              # without listing

# validate access information
my-connector receive-directory-validate access_file.json

# remove directory
my-connector receive-directory-cleanup internal.json
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
my-connector send access_file.json internal_file.json

# validate access information
my-connector send-validate access_file.json
```

The `send` call reads a file descibed in `internal_file.json` and transfers it with the information in `access_file.json`.
The `send-validate` call works similar to the `receive-validate` function.
