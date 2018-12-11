# FAQ

If you have another **question** please open an [issue on Github](https://github.com/curious-containers/curious-containers.github.io/issues).

| Table of Contents |
| --- |
| [What is the purpose of the Curious Containers project?](#what-is-the-purpose-of-the-curious-containers-project) |
| [What is unique about CC?](#what-is-unique-about-cc) |
| [What exactly is RED?](#what-exactly-is-red) |
| [You mentioned CWL: Is CC a workflow engine?](#you-mentioned-cwl-is-cc-a-workflow-engine) |


## What is the purpose of the Curious Containers project?

The Curious Conatiners (CC) project allows for data-driven experiments to be executed, shared and reproduced. It is built around Docker container technologies, which enables the usage of local or distributed compute resources.


## What is unique about CC?

In the domain of distrubuted computing many alternative frameworks and schedulers exist, but if you share some of the concerns below, CC might be the right tool for you.

* **Reproducible Research**: We developed the RED file format to fully describe experiments. The components of an experiment (application, data, compute resources) are references (see [FAIR Principles](https://www.force11.org/fairprinciples)) to remote servers and are therefore loosely coupled and interchangeable.
* **Data Security**: The project is designed for biomedical applications, which involve the usage of sensitive data. CC enforces the separation of data storage and compute resources, where data is accessed via standard protocols supporting authentication. Data is never processed outside of temporary Docker containers.
* **Cuda and Machine Learning**: We are improving CC's capablities in the realm of machine learning and other high performance workloads. CC already ships with support for Cuda via [nvidia-docker](https://github.com/NVIDIA/nvidia-docker). Connectors for mounting huge data directories (like the [CAMELYON](https://camelyon17.grand-challenge.org/) image database) will be added soon.


## What exactly is RED?

RED (Reproducible Experiment Description), is a YAML or JSON based file format, which allows for a full description of a data-driven experiment. The format is built upon the CLI specification of the Common Workflow Language (CWL) and supports the [FAIR Principles](https://www.force11.org/fairprinciples) for reproducible research. Take a look at the [RED Format](red-format.md) documentation for more details.


## You mentioned CWL: Is CC a workflow engine?

No, CC is not a workflow engine. The RED format only uses the CLI specification of the Common Workflow Language, but does not support any workflow specifications. Experiments supported by RED/CC must be self-contained, atomic entities and cannot reference other experiments.
