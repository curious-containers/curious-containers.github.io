When development of the Curious Containers (CC) software was started as a research project with a simple goal in mind: to run biomedical analysis workloads in a distributed compute environment [[Jansen2016](publications.md#employing-docker-swarm-on-openStack-for-biomedical-analysis-2016)]. Medical data contains sensitive information, which requires strict data management and protection. In contrast to existing distributed computing environments like Hadoop, we wanted the data to reside in a data management system (DMS) specifically designed for medical data (e.g. XNAT). The data should only be copied to a compute node in a server cluster for the time of processing and be deleted afterwards. As an efficient alternative to Virtual Machines, we chose Docker containers as a controlled environment, which can be whiped off a compute node entirely after processing.


The following steps describe the basic mechanics of CC:

1. Build a container image containing all software artifacts.
2. Create an experiment description, reference the container image and data sources/destinations.
3. Run a container from the experiment description. The main process **in the container** is an agent software, which handles
    * **download** of input data from a data source into the running container,
    * **execution** of the data processing application,
    * **upload** of the output data to a destination server.
4. Stop the container and delete its file system.


These mechanics were implemented in the now deprecated software componentents [CC-Server](https://github.com/curious-containers/cc-server) and [CC-Container-Worker](https://github.com/curious-containers/cc-container-worker).






If you want to use files or directories as input for an experiment in a red file, you have to use connectors.
There are *input* connectors and *output* connectors.
If an input connector is specified in a red file, this connector will be executed before the actual program of the red file.
This will fetch some files or directories to the local filesystem to make this data available to the executed program.

Let's say you have a little program like `cat`, which simply prints the content of a file to `stdout` and
you have a file, which is online available like [this file](https://raw.githubusercontent.com/curious-containers/vagrant-quickstart/master/in.txt).
You can now specify a connector which fetches this file via http to make it accessible for your cat program.


### Input Files

The following red file specifies an experiment in which the `cat` program is used to print the content of a file:

```yml
redVersion: "5"
cli:
  cwlVersion: "v1.0"
  class: "CommandLineTool"
  baseCommand: "cat"
  doc: "Prints the content of a given file."

  inputs:
    myinputfile:
      type: File
      inputBinding:
        position: 1
  outputs: {}

inputs:
  myinputfile:
    class: File
    connector:
      pyModule: "cc_core.commons.connectors.http"
      pyClass: "Http"
      access:
        url: "https://raw.githubusercontent.com/curious-containers/vagrant-quickstart/master/in.txt"
        method: "GET"
```

As you can see we only specified a single input `myinputfile`, which is the file we are going to print.
To match this input we specify a connector with the same input key. We specify that this should fetch a file and not a directory with `class: File`.
After this we specify the connector to fetch the input file. The `pyModule: "cc_core.commons.connectors.http"` defines the python module from where to import the connector class `pyClass: Http`.
Every connector needs to know how to fetch the file, but different connectors with different protocols need different information. For example a connector using the ssh protocol doens't need a
method like the connector using HTTP. A list of the required information for each connector can be found below.


### Input Directories

The following experiment uses the `tree` command to print the directory structure.

```yml
redVersion: "5"
cli:
  cwlVersion: "v1.0"
  class: "CommandLineTool"
  baseCommand: "tree"
  doc: "Simple Test Script"

  inputs:
    myinputdirectory:
      type: Directory
      inputBinding:
        position: 1
  outputs: {}

inputs:
  myinputdirectory:
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

The first part of the red file is similar to the red file above. Instead of `type: File` we now have a directory with `type: Directory` as input. We again use the HTTP connector,
which can be used for files and folders.  Next to the `connector` field there is now a `listing` field. This listing defines the subfiles and subdirectories and is only allowed
for directory connectors.

In this listing we have a subfile with name `version.py`. We can define as many subfiles as we want.
Subdirectories are also possible. These can again have a listing field, where again subfiles and subdirectories can be specified.
If a listing field is present for a connector, the connector should only fetch the files, which are present in the listing. If no listing is present the hole directory should be fetched.
Some connectors like the HTTP-Connector require a listing field.