---
title: FAQ
permalink: /faq
layout: single
toc: true
toc_label: "Table of Contents"
---

If you have another **question** please open an [issue on Github](https://github.com/curious-containers/curious-containers/issues). Please not that we only accept issues in the [meta project](https://github.com/curious-containers/curious-containers).


## What is the purpose of the Curious Containers project?

The Curious Conatiners (CC) project allows for data-driven experiments to be executed, shared and reproduced. It is built around Docker container technologies, which enables the usage of local or distributed compute resources.


## What is unique about CC?

In the domain of distrubuted computing many alternative frameworks and schedulers exist, but if you share some of the concerns below, CC might be the right tool for you.

* **Reproducible Research**: We developed the RED file format to fully describe experiments. The components of an experiment (application, data, compute resources) are references (see [FAIR Principles](https://www.force11.org/fairprinciples)) to remote servers and are therefore loosely coupled and interchangeable.
* **Data Security**: The project is designed for biomedical applications, which involve the usage of sensitive data. CC enforces the separation of data storage and compute resources, where data is accessed via standard protocols supporting authentication. Data is never processed outside of temporary Docker containers.
* **Machine Learning Workloads**: We are improving CC's capablities in the realm of machine learning and other high performance workloads. CC ships with support for CUDA via [nvidia-docker](https://github.com/NVIDIA/nvidia-docker) and large data directories (e.g. [CAMELYON](https://camelyon17.grand-challenge.org/) image database) can be mounted via [FUSE](https://de.wikipedia.org/wiki/Filesystem_in_Userspace) based network connectors.


## What exactly is RED?

RED (Reproducible Experiment Description), is a YAML or JSON based file format, which allows for a full description of a data-driven experiment. The format is built upon the CLI specification of the Common Workflow Language (CWL) and supports the [FAIR Principles](https://www.force11.org/fairprinciples) for reproducible research. Take a look at the [RED Format](red-format.md) documentation for more details.


## You mentioned CWL: Is CC a workflow engine?

No, CC is not a workflow engine. The RED format only uses the CLI specification of the Common Workflow Language, but does not support any workflow specifications. Experiments supported by RED/CC must be self-contained, atomic entities and cannot reference other experiments. Multiple experiments based on the same application can be defined in a single RED file under the [batches](red-format.md#batches) keyword.


## How does CC compare to Kubernetes?

Kubernetes is an container orchestration tool, CC is not. An orchestration tool is useful for server deployments, where each container runs a service (webserver, database, etc.) for an indefinite time and the orchestrator provides service discovery, scaling and fault-tolerance. CC on the other hand is specifically designed for short-lived data processing applications, handling file inputs and outputs.


## Is CC fault-tolerant?

Yes, especially CC-Agency implements multiple failure recovery mechanisms. It is programmed to catch any error, either caused by data connectors and data-processing applications or by the underlying cluster infrastructure with faulty networks, docker-engines and configurations. The error history of applications and compute nodes is documented in a database to be inspected by the user. CC-Agency performs health checks to exclude problematic compute nodes from the processing pool or to automatically rejoin them. It can be configured to retry failed experiments a certain number of times.

## How to support the project?

If you are a user of Curious Container please get in touch and let us know. Since we do **not** track our users this is the only way for us to know that you exist and what kind of use cases you have.

If you are a developer and want to contribute code, please create an [issue](https://github.com/curious-containers/curious-containers/issues) for discussion before requesting a pull.