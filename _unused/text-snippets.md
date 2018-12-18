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