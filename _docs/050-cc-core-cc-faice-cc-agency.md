---
title: "CC-Core, CC-FAICE, CC-Agency"
permalink: /docs/cc-core-cc-faice-cc-agency
---

The following section introduce the software components of Curious Containers.

## CC-Core

CC-Core is a Python package which, as the name says, provides core functionality to the Curious Containers ecosystem. The main purposes of this package are:

1. Providing JSON-schema definitions for the supported [CWL Command Line Tool Description](https://www.commonwl.org/v1.0/CommandLineTool.html) standard and the [RED format](/docs/red-format).
2. Implementing CLI programs, so-called agents, to run data-driven experiments in CWL or RED format.
3. Being a software library for shared functionality in CC-FAICE and CC-Agency.

### Installation

If you are installing CC-FAICE, CC-Core will be automatically installed as a package dependency with the highest compatible version.

A manual installation of CC-Core is only required in container images. Take a look at the [Container Image](/docs/container-image) documentation for more information.

## CC-FAICE

FAICE (Fair Collaboration and Experiments) is a CLI tool suite, providing a lot of functionality to users. The main functions are:

* Implementing meta agents to launch corresponding CC-Core agents inside of Docker containers, which will then run the actual experiments defined in CWL or RED format.
* Additional tools to convert, validate and export RED files.

### Installation

Install `python3-pip` as system package. The following instructions work for Fedora 28, but instructions for other Linux distribution should be similar.

```bash
sudo dnf install python3-pip
```

It is recommended to install a specific version of `cc-faice`. This will automatically install the latest compatible version of `cc-core`.

```bash
pip3 install --user --upgrade cc-faice==5.4.0
```

Run CLI tool.

```bash
faice --version
faice --help
```

If this tool cannot be found, you should modify `PATH` (e.g. append `${HOME}/.local/bin`) or fall back to executing the tools as Python modules.

```bash
python3 -m cc_faice --version
```

### Agents

CC-FAICE implements two meta agents, `cwl` and `red`, which refer to the corresponding CC-Core agents. While the CC-Core agents execute an experiment directly, the CC-FAICE agents only work with Docker containers, where CC-Core has to be installed in the Docker image beforehand. A CC-FAICE meta agent will launch the corresponding CC-Core agent via Docker, which will then take over control to download input files, run the experiment and upload output files within the container.

Use the following CLI tools.

```bash
faice agent cwl --help
faice agent red --help
```

## CC-Agency

CC-Agency is an advanced server software, which is able to connect to a distributed cluster of docker-engines and schedules experiments defined in RED format for parallel execution.

It implements two major software components: CC-Agency Broker provides a restful web API for users, to register experiments and to query information. CC-Agency Controller is a background process, which connects to a Docker cluster and schedules the experiments for execution.

CC-Agency persists the state of experiments in a database and is very fault tolerant, when it comes to failing containers, docker-engines or network connections.

### Usage

If you are a user and would like to connect to an existing installation of CC-Agency, please refer to the [REST API](/docs/cc-agency-api) documentation.

It is advised to test and validate your RED files and containers locally with [CC-FAICE](#cc-faice).

In order to make your resources referenced in a RED file, like a Docker image and input/output file locations, available to a remote Docker cluster, they have to be **published** in an appropriate way. This means that Docker images have to be uploaded to a Docker registry, like the official [DockerHub](https://hub.docker.com/) or a private [registry](https://docs.docker.com/registry/), and that data exchange must be handled through a data management system (DMS) via the network. We provide RED connector plugins for secure file exchange protocols and the [XNAT](https://www.xnat.org/) DMS. Of course, as demanded by the [FAIR princibles](https://www.force11.org/fairprinciples), common authentication mechanisms are supported for Docker registry and data connections.

As a positive side-effect, your experiments do not rely on any local files, which allows you to share and store your RED files in a **reproducible** way.

### Data Security

As a technical necessity, you are required to send your Docker image and data access credentials defined in your RED file to CC-Agency. It is therefore strongly advised to only use CC-Agency if you **trust the operator** with this sensitive data.

To improve data protection you should use **temporary credentials** whenever possible.

CC-Agency will store credentials temporarily in its database. As soon as the processing of a batch or experiment finishes, the values of **protected keys** under the `access` data in the connector sections and under the `auth` data in the engine sections are **overwritten** in the database. By default, only `password` is considered a protected key. You can protect other keys if they refer to string values, by prepending `_` (e.g. `_username`) in the RED data (see [Protected Keys](/docs/red-format-protecting-credentials#protected-keys)), before posting it to CC-Agency.

All data handling is done inside of Docker containers. The container file systems are deleted after processing, leaving no traces of your data on remote servers.
