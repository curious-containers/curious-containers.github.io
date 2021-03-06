---
title: "Container Images"
permalink: /docs/container-images
---

The following sections show how to prepare a Docker container image for RED.


# Docker

There are many ways to build a Docker image.
In our examples we chose `docker.io/debian:9.5-slim` as a base image, because it is relatively small and dependencies can easily be installed from the Debian package repositories via `apt-get install`. A POSIX shell (`/bin/sh`), as it is preconfigured in most images, and a Python3 interpreter (version >= 3.4), must be installed.

For the image to be used effectively, RED connector commandline applications have to be installed.
In order to use any of the standard connectors provided by the Curious Containers project, a Python3 interpreter must be installed as well. Since Docker provides a clean environment by default, we have to set the `PATH` environment variables explicitely.
Please note, that your own application also needs to be located in a directory which is included in `PATH`.

Another requirement is, that the application and the RED connectors are executed as the user last set by the `USER` keyword in a Dockerfile.
Please note, that the user's name does not matter.

The Dockerfile below does not include an application.
For a more complete example we advise you to work through the [RED Beginner's Guide](/docs/red-beginners-guide).

```docker
FROM docker.io/debian:9.5-slim

RUN apt-get update \
&& apt-get install -y sshfs python3-venv \
&& useradd -ms /bin/bash cc

# switch user
USER cc

# install connectors
ENV PATH /home/cc/.local/bin:${PATH}

RUN python3 -m venv /home/cc/.local/red \
&& . /home/cc/.local/red/bin/activate \
&& pip install wheel \
&& pip install red-connector-http==1.0 red-connector-ssh==1.2 \
&& mkdir -p /home/cc/.local/bin \
&& ln -s /home/cc/.local/red/bin/red-connector-* /home/cc/.local/bin

# add commands here to install your application
# ...
```

After adding your own application to the Dockerfile, create the image using the `docker build` command. The `.` at the end of the command indicates, that `docker build` will pick up your Dockerfile from the current working directory.

```bash
docker build -t docker.io/myorganization/myimage .
```

To check your image, you can run a container based on the image.

```bash
docker run --rm docker.io/myorganization/myimage red-connector-http --version
```

As you can see, we tagged the image with a URL, pointing to a location in the `docker.io` registry, also known as [DockerHub](https://hub.docker.com/). If you want to share your experiment with others or to execute it in a compute cluster via CC-Agency, you have to push your locally created image to a Docker registry. You can either use a public or paid organization on DockerHub or setup a private Docker registry on your own server.

Login to your registry and push the image.

```bash
docker login docker.io
docker push docker.io/myorganization/myimage
```

In your RED file you can now reference your image under `container.settings.image.url`. If read access to your image is restricted via user credentials you can provide them as `container.settings.image.auth.username` and `container.settings.image.auth.password` respectively. See the [RED Container Engines](/docs/red-container-engines) documentation for more information.


## CUDA

In order to use CUDA applications on Nvidia GPUs, you have to specify GPUs in your RED file. For more details take a look at the [Docker container engine](/docs/red-container-engines#docker) documentation.

The easiest way to make CUDA available to your containerized application is to start your Dockerfile with `FROM docker.io/nvidia/cuda:10.1-cudnn7-runtime-ubuntu18.04`. Of course the underlying host's driver and hardware must support the desired cuda version (e.g. 10.1).

See the following Dockerfile listing for an example.

```docker
FROM docker.io/nvidia/cuda:10.1-cudnn7-runtime-ubuntu18.04

# ...
```

For a complete list of available CUDA base images take a look at the their [DockerHub site](https://hub.docker.com/r/nvidia/cuda).
