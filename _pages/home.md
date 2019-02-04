---
layout: single
permalink: /
sidebar:
  nav: "docs"
---

Welcome to the **Curious Containers** project and its **RED** file format for reproducible experiments.

RED (Reproducible Experiment Description) is a JSON or YAML based file format to describe data-driven experiments. A RED file allows researchers to share or publish their computational experiments, such that others can reproduce the results or customize the experiments. A minimal RED file consists of an application's commandline interface (CLI) description in [Common Workflow Language](https://www.commonwl.org/v1.0/CommandLineTool.html) (CWL) syntax, as well as CLI arguments and input file references.

Curious Containers provides a reference implementation of RED in Python (`cc-core`). Experiments can be executed on a local Linux host using the [Docker](https://www.docker.com/) container runtime via the FAICE tool suite (`cc-faice`). For a more advanced usage, Curious Containers Agency (`cc-agency`) can distribute experiments in a Docker cluster across multiple hosts.

Together, RED and Curious Containers support the [FAIR principles](https://www.force11.org/fairprinciples) for reproducible research. If you are new to the project, we advise you to work through the [RED Beginner's Guide](docs/red-beginners-guide). Examples can be found on the [Tawian](https://somnonetz.github.io/tawian/) meta-platform.


## Acknowledgements

The Curious Containers software is developed at [CBMI](https://cbmi.htw-berlin.de/) (HTW Berlin - University of Applied Sciences). The work is supported by the German Federal Ministry of Economic Affairs and Energy (ZIM project [BeCRF](https://www.htw-berlin.de/forschung/online-forschungskatalog/projekte/projekt/?eid=2170), grant number KF3470401BZ4), the German Federal Ministry of Education and Research (project [deep.TEACHING](https://www.deep-teaching.org/), grant number 01IS17056 and project deep.HEALTH, grant number 13FH770IX6) and HTW Berlin Booster.