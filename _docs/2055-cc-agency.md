---
title: "CC-Agency"
permalink: /docs/cc-agency
---

# CC-Agency

CC-Agency is an advanced server software, which is able to connect to a distributed cluster of docker-engines and schedules experiments defined in RED format for parallel execution.

It implements three major software components: CC-Agency Broker provides a restful web API for users, to register experiments and to query information. CC-Agency Controller is a background process, which connects to a Docker cluster and schedules the experiments for execution. CC-Agency Trustee is a service to temporarily store credentials in memory, until the correponding experiment finished in one of the states *succeeded*, *failed* or *cancelled*.

CC-Agency persists the state of experiments in a database and is very fault tolerant, when it comes to failing containers, docker-engines or network connections.


## Usage

If you are a user and would like to connect to an existing installation of CC-Agency, please refer to the [REST API](/docs/cc-agency-api) documentation. It is recommended to specify a CC-Agency instance under the [execution](/docs/red-format#execution) keyword of your RED file. You can then use the `faice exec` commandline tool to send the experiment to CC-Agency. The tool will automatically validate your RED file and asks you for missing [variable values](/docs/red-format-protecting-credentials) if they are not found in your keyring.

In order to make your resources referenced in a RED file, like a Docker image and input/output file locations, available to a remote Docker cluster, they have to be **published** in an appropriate way. This means that Docker images have to be uploaded to a Docker registry, like the official [DockerHub](https://hub.docker.com/) or a private [registry](https://docs.docker.com/registry/), and that data exchange must be handled through a data management system (DMS) via the network. We provide RED connector plugins for secure file exchange protocols and the [XNAT](https://www.xnat.org/) DMS. Of course, as demanded by the [FAIR princibles](https://www.force11.org/fairprinciples), common authentication mechanisms are supported for Docker registry and data connections.

As a positive side-effect, your experiments do not rely on any local files, which allows you to share and store your RED files in a **reproducible** way.


## Data Security

As a technical necessity, you are required to send your Docker image and data access credentials defined in your RED file to CC-Agency. It is therefore strongly advised to only use CC-Agency if you **trust the operator** with this sensitive data.

To improve data protection you should use **temporary credentials** whenever possible. CC-Agency will store credentials temporarily in memory. As soon as the processing of a batch or experiment finishes, these credentials will be removed.

All data handling is done inside of Docker containers. The container file systems are deleted after processing, leaving no traces of your data on remote servers.