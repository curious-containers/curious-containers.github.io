# CC-Core

CC-Core is a Python package which, as the name says, provides core functionality to the Curious Containers ecosystem. The main purposes of this package are:

1. Providing JSON-schema definitions for the supported [CWL Command Line Tool Description](https://www.commonwl.org/v1.0/CommandLineTool.html) standard and the [RED format](red-format.md).
2. Implementing CLI programs, so-called [agents](#agents), to run data-driven experiments in CWL or RED format.
3. Being a software library for shared functionality in [CC-FAICE](cc-faice.md) and [CC-Agency](cc-agency.md).

## Installation

If you are installing CC-Core as a **dependency** of [CC-FAICE](cc-faice.md) or [CC-Agency](cc-agency.md), please refer to their respective installation documents, instead of following the instructions provided below.

If you are installing CC-Core in a Docker **container image** for it to be compatible with [CC-FAICE](cc-faice.md) or [CC-Agency](cc-agency.md), you can follow the instructions in the [RED Beginner's Guide](red-beginners-guide.md).

The instructions below are **not required** in most cases, but provide additional hints for special use cases.

### Local Installation

Install `python3-pip` as system package. The following instructions work for Fedora 28, but instructions for other Linux distribution should be similar.

```bash
sudo dnf install python3-pip
```

It is recommended to install a specific version of `cc-core`.

```bash
pip3 install --user --upgrade cc-core==5.2.0
```

Run CLI tool.

```bash
ccagent --version
ccagent --help
```

If this tool cannot be found, you should modify `PATH` (e.g. append `${HOME}/.local/bin`) or fall back to executing the tools as Python modules.

```bash
python3 -m cc_core.agent --version
```

### Docker Image Installation

Create a Dockerfile.

```Dockerfile
FROM docker.io/debian:9.3-slim

RUN apt-get update \
&& apt-get install -y python3-pip \
&& useradd -ms /bin/bash cc

USER cc

ENV PATH="/home/cc/.local/bin:${PATH}"
ENV PYTHONPATH="/home/cc/.local/lib/python3.5/site-packages/"

RUN pip3 install --no-input --user cc-core=5.2.0

ADD --chown=cc:cc . /opt/cc-core
```

Build with Docker.

```bash
docker build -t cc-core:5.2.0 .
```

Run a container based on the image.

```bash
docker run cc-core:5.2.0 ccagent --help
```

Use as a base image for other application images in another Dockerfile.

```
FROM cc-core:5.2.0

# add instructions to install your own application
```

## Agents

CC-Core implements three different agents, called `cwl`, `red` and `connected`, to execute experiments specified in CWL or RED format.

### CWL

The CLI tool `ccagent cwl` takes a CWL file and a corresponding job file as input. It is very similar to [cwltool](https://github.com/common-workflow-language/cwltool), the official CWL implementation. Please note, that the `ccagent cwl` does only support a subset of the full CWL standard, which means that every CWL file compatible with the agent is also compatible with `cwltool`, but not the other way round.

If CC-Core is installed in a Docker image, `ccagent cwl` can be invoked in a container by `faice agent cwl`.

Run the CLI tool.

```bash
ccagent cwl --help
```

### RED

The CLI tool `ccagent red` takes a RED file as input. It provides enhanced functionality over `ccagent cwl`, like downloading and uploading files via dedicated Python [connector](#red-connectors.md) modules.

If CC-Core is installed in a Docker image, `ccagent red` can be invoked in a container by `faice agent red`.

Run the CLI tool.

```bash
ccagent red --help
```

### Connected

The CLI tool `ccagent connected` serves a similar purpose as `ccagent red`, but is designed for usage in a distributed cluster of docker-engines. It is invoked by CC-Agency and communicates via REST callbacks to the [CC-Agency API](cc-agency-api.md).

Run the CLI tool.

```bash
ccagent connected --help
```
