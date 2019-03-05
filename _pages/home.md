---
layout: single
permalink: /
sidebar:
  nav: "docs"
---

Welcome to the **Curious Containers** project and its **RED** file format for reproducible experiments.


## Computational Reproducibility

RED (Reproducible Experiment Description) is a JSON or YAML based file format to describe data-driven experiments. A RED file allows researchers to share or publish their computational experiments, such that others can reproduce the results or customize the experiments. A minimal RED file consists of an application's commandline interface (CLI) description in [Common Workflow Language](https://www.commonwl.org/v1.0/CommandLineTool.html) (CWL) syntax, a reference to a container image, as well as CLI arguments and input file references.

Curious Containers provides a reference implementation of RED in Python (`cc-core`). Experiments can be executed on a local Linux host using the [Docker](https://www.docker.com/) container runtime via the FAICE tool suite (`cc-faice`). For a more advanced usage, Curious Containers Agency (`cc-agency`) can distribute experiments in a Docker cluster across multiple hosts.

Together, RED and Curious Containers support the [FAIR principles](https://www.force11.org/fairprinciples) for reproducible research. If you are new to the project, we advise you to work through the [RED Beginner's Guide](docs/red-beginners-guide). Examples can be found on the [Tawian](https://somnonetz.github.io/tawian/) meta-platform.


## The Road to RED 10

In the past, Curious Containers was mainly a research project to develop ideas and try new concepts in the context of computational reproducibility. Our existing file formats and software components are considered BETA releases.

It is now time to stabilize the ecosystem as a major step for longterm reproducibility. We are therefore working towards RED 10, the first stable release of the RED file format. In addition, we will define a stable CLI for RED Connectors. RED 10 and the RED Connector CLI version 1 will be supported by all future releases of the Curious Containers software components in a backwards compatible way.


### Pre-Release

You can soon expect the RED 10 pre-release version called RED 7. A pre-release of the RED connector CLI will be published as version 0.1.


## Machine Learning Workloads

We are improving CC's capablities in the realm of machine learning and other high performance workloads. CC ships with support for CUDA via [nvidia-docker](https://github.com/NVIDIA/nvidia-docker). Large data directories (e.g. [CAMELYON](https://camelyon17.grand-challenge.org/) image database) can be mounted via [FUSE](https://de.wikipedia.org/wiki/Filesystem_in_Userspace) based network connectors.


## Issues

If you want to open an issue, please go to the [curious-containers](https://github.com/curious-containers/curious-containers/issues) meta project on Github. Issue trackers of every other repository are closed.


## Acknowledgements

The Curious Containers software is developed at [CBMI](https://cbmi.htw-berlin.de/) (HTW Berlin - University of Applied Sciences). The work is supported by the German Federal Ministry of Economic Affairs and Energy (ZIM project [BeCRF](https://www.htw-berlin.de/forschung/online-forschungskatalog/projekte/projekt/?eid=2170), grant number KF3470401BZ4), the German Federal Ministry of Education and Research (project [deep.TEACHING](https://www.deep-teaching.org/), grant number 01IS17056 and project deep.HEALTH, grant number 13FH770IX6) and HTW Berlin Booster.
