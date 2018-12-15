# Container Images

Its part of Curious Containers' concept, that data is is only handled inside of a running container and never touches a file-system of the underlying infrastructures. Therefore agent implementations, like `ccagent cwl`, `ccagent red` or `ccagent connected`, must be installed in the container image. These agents are invoked by a compatible execution engine, like `faice agent cwl`, `faice agent red` or CC-Agency. The `ccagent` implementations are provided by the `cc-core` python package, which can be installed via `pip`.

The following sections show how to prepare a container image, for the two container engines `docker` and `nvidia-docker`.

| Table of Contents |
| --- |
| [Docker](#docker-engine) |
| [Nvidia-Docker](#nvidia-docker) |


## Docker

There are many ways to build a Docker engine, be it `FROM scratch` or from an existing base image. In our examples we chose `docker.io/debian:9.5-slim` as a base image, because it is relatively small and dependencies can easily be installed from the Debian package repositories via `apt-get install`.

Since `ccagent` is implemented in the Python package `cc-core`, a Python3 interpreter is always required. To make our lives easier, we also install `pip3`, a package manager for Python3. This allows us to install `cc-core` with a single command. Since Docker provides a clean environment by default, we have to set the `PATH` and `PYTHONPATH` environement variables explicitely. Please note, that your own application also needs to be located in a directory which is included in `PATH`.

```Dockerfile
FROM docker.io/debian:9.5-slim

RUN apt-get update \
&& apt-get install -y python3-pip \
&& useradd -ms /bin/bash cc

# install cc-core
USER cc

RUN pip3 install --no-input --user cc-core==5.4.0

ENV PATH="/home/cc/.local/bin:${PATH}"
ENV PYTHONPATH="/home/cc/.local/lib/python3.5/site-packages/"

# add commands here to install your application
```


## Nvidia-Docker

TODO
