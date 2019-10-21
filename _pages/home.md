---
layout: single
permalink: /
sidebar:
  nav: "docs"
---

Welcome to the **Curious Containers** framework for reproducible research.


## Computational Reproducibility

Data, code and environment are the crucial components required to successfully reproduce a **computational experiment**.
Usually these are scripts or executables, that have been setup to work on a researcher's computer and that load data from the local filesystem.
In this situation, it is difficult for the experiment to be **stored** for future use, **shared** with coworkers, **referenced** in a publication or **distributed** in a compute cluster for parallel batch processing.

Curious Containers (CC) is a framework for computational reproducibility, that allows researchers to define each component of an experiment as a network resource.
These resources are referenced by unique identifiers in the Reproducible Experiment Description (RED) file format.
An execution engine provided by CC interprets the RED file and combines all resources to run the experiment.
This concept allows resources to be interchangeable and to have individual access restriction based on the chosen network transmission and authentication protocol.

If you are new to the project, we advise you to work through the [RED Beginner's Guide](docs/red-beginners-guide).
As an introduction (in german), watch the following [talk at the deRSE 2019](https://www.de-rse.org/de/conf2019/talk/7LLTCN/) conference for research software engineering ([PDF Slides](https://www.de-rse.org/de/conf2019/talk/7LLTCN/slides.pdf)).

{% raw %}
<iframe width="560" height="315" scrolling="no" src="//av.tib.eu/player/42497" frameborder="0" allowfullscreen></iframe>
{% endraw %}


## Issues and Feedback

If you have problems or want to give feedback, please open an issue in the [curious-containers](https://github.com/curious-containers/curious-containers/issues) meta project on Github. Issue trackers of every other repository are closed.


## How it works

CC employs [Docker](https://www.docker.com/) to install an experiment runtime environment, including code and dependencies, in a container image.
Using a container registry this image becomes a network resource, that can be deployed on any Linux computer with Docker installed.

In contrast to existing research platforms, CC as a framework does not provide a tightly integrated storage solution.
Instead, it connects to any data storage, that is already present in a research institution.
For this to work, CC provides *connector* programs for common storage interfaces like SSH, HTTP or [XNAT](http://xnat.org/), that are shipped as part of an experiment's container image.
These connectors download input data into the running container, upload results to a defined destination or mount directories via [FUSE](https://de.wikipedia.org/wiki/Filesystem_in_Userspace) network filesystems.
If a connector for a certain storage solution does not yet exist, the user can provide a custom connector by implementing an [interface specification](/docs/red-connector-cli-1).

CC implements two execution engines that run experiments defined in RED files.
*CC-FAICE* is simple to install on a local computer and can run one experiment at a time.
*CC-Agency* is a server side execution engine, that connects to docker-engines in a compute cluster for parallel execution, tracks the execution state of scheduled experiments in a database and provides a REST web interface.

CC and RED support the [FAIR principles](https://www.force11.org/fairprinciples) for reproducible research, that require all experiment resources to be findable, accessible, interoperable and reusable.
Therefore, the command-line interface (CLI) description of the experiment's script or executable, that is embedded in a RED file, follows the CommandLine Description of the [Common Workflow Language](https://www.commonwl.org/v1.0/CommandLineTool.html) (CWL).
This enables portability between CC execution engines and a CWL runtime.


### Machine Learning Workloads

CC explicitely supports machine learning and other high performance workloads.
Available NVIDIA graphics processing units are accessible in Docker containers using [Nvidia Container Toolkit](https://github.com/NVIDIA/nvidia-docker) (or its predecessor nvidia-docker).
Large training data directories, that are multiple terabytes in size, can be mounted via [FUSE](https://de.wikipedia.org/wiki/Filesystem_in_Userspace).


## The Road to RED 10

In the past, Curious Containers was mainly a research project to develop ideas and try new concepts in the context of computational reproducibility. Our existing file formats and software components are considered BETA releases.

It is now time to stabilize the ecosystem as a major step towards long-term reproducibility. RED 10 will be the first stable release, such that experiments defined in the RED 10 format will be supported in all future Curious Containers software releases.


### RED 10 Pre-Releases

We have released RED 8, that includes the RED Connector CLI 1 specification. Therefore container images and their installed RED connectors, that are prepared to work with RED 8, will be compatible with all future RED versions.

With RED 9, we will move from the CWL 1.0 to the CWL 1.1 standard. RED 9 will be tested extensively without many additional changes, hopefully leading to a rock solid RED 10 release.


## Acknowledgements

The Curious Containers software is developed at [CBMI](https://cbmi.htw-berlin.de/) (HTW Berlin - University of Applied Sciences). The work is supported by the German Federal Ministry of Economic Affairs and Energy (ZIM project [BeCRF](https://www.htw-berlin.de/forschung/online-forschungskatalog/projekte/projekt/?eid=2170), grant number KF3470401BZ4), the German Federal Ministry of Education and Research (project [deep.TEACHING](https://www.deep-teaching.org/), grant number 01IS17056 and project deep.HEALTH, grant number 13FH770IX6) and HTW Berlin Booster.
