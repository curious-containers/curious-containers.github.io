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




## User Information

If you are a **user** and would like to connect to an existing installation of CC-Agency, please refer to the [REST API](cc-agency-api.md) documentation.

If you are new to RED and Curious Containers, please work through the [RED Beginner's Guide](#red-beginners-guide.md) first.

It is advised to test and validate your RED files and containers locally with [CC-FAICE](cc-faice.md).

In order to make your **resources** referenced in a RED file, like a Docker image and input/output file locations, available to a remote Docker cluster, they have to be **published** in an appropriate way. This means that Docker images have to be uploaded to a Docker registry, like the official [DockerHub](https://hub.docker.com/) or a private [registry](https://docs.docker.com/registry/), and that data exchange must be handled through a data management system (DMS) via the network. We provide [RED connector](#red-connectors.md) plugins for secure file exchange protocols and the [XNAT](https://www.xnat.org/) DMS. Of course, as demanded by the [FAIR princibles](https://www.force11.org/fairprinciples), common authentication mechanisms are supported for Docker registry and data connections.

As a positive side-effect, your experiments do not rely on any local files, which allows you to share and store your RED files in a **reproducible** way.

## Data Security

As a technical necessity, you are required to send your Docker image and data access credentials defined in your RED file to CC-Agency. It is therefore strongly advised to only use CC-Agency if you **trust the operator** with this sensitive data.

To improve data protection you should use **temporary credentials** whenever possible.

CC-Agency will store credentials temporarily in its database. As soon as the processing of a batch or experiment finishes, the values of **protected keys** under the `access` data in the connector sections and under the `auth` data in the engine sections are **overwritten** in the database. By default, only `password` is considered a protected key. You can protect other keys if they refer to string values, by prepending `_` (e.g. `_username`) in the RED data (see [Protected Keys](red-format.md#protected-keys)), before posting it to CC-Agency.

All data handling is done inside of Docker containers. The container file systems are deleted after processing, leaving no traces of your data on remote servers.