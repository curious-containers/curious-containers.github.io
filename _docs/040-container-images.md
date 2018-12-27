---
title: "Container Images"
permalink: /docs/container-images
---

Its part of Curious Containers' concept, that data is is only handled inside of a running container and never touches a file-system of the underlying infrastructures. Therefore agent implementations, like `ccagent cwl`, `ccagent red` or `ccagent connected`, must be installed in the container image. These agents are invoked by a compatible execution engine, like `faice agent cwl`, `faice agent red` or CC-Agency. The `ccagent` implementations are provided by the `cc-core` python package, which can be installed via `pip`.

The following sections show how to prepare a container image, for the two container engines `docker` and `nvidia-docker`.


## Docker

There are many ways to build a Docker engine, be it `FROM scratch` or from an existing base image. In our examples we chose `docker.io/debian:9.5-slim` as a base image, because it is relatively small and dependencies can easily be installed from the Debian package repositories via `apt-get install`.

Since `ccagent` is implemented in the Python package `cc-core`, a Python3 interpreter is always required. To make our lives easier, we also install `pip3`, a package manager for Python3. This allows us to install `cc-core` with a single command. Since Docker provides a clean environment by default, we have to set the `PATH` and `PYTHONPATH` environement variables explicitely. Please note, that your own application also needs to be located in a directory which is included in `PATH`.

Another requirement is, that `ccagent` and the application are executed as user with uid:gid set to `1000:1000`. The debian base image does not yet include another user besides `root`. We can therefore create the first user called `cc`, wich will by default be assigned the uid:gid pair `1000:1000`.

The Dockerfile below demonstrate the correct `cc-core` setup with two additional connector packages. Please note, that this Dockerfile does not include an application to be executed by `ccagent`. For a more complete example we advise you to work through the [RED Beginner's Guide](/docs/red-beginners-guide).

```docker
FROM docker.io/debian:9.5-slim

RUN apt-get update \
&& apt-get install -y python3-pip \
&& useradd -ms /bin/bash cc

# install cc-core
USER cc

RUN pip3 install --no-input --user cc-core==6.0.0

ENV PATH="/home/cc/.local/bin:${PATH}"
ENV PYTHONPATH="/home/cc/.local/lib/python3.5/site-packages/"

# install additional connectors if you need them
RUN pip3 install --no-input --user red-connector-ssh red-connector-xnat

# add commands here to install your application
```

After adding your own application to the Dockerfile, create the image using the `docker build` command. The `.` at the end of the command indicates, that `docker build` will pick up your Dockerfile from the current working directory.

```bash
docker build -t docker.io/myorganization/myimage .
```

As you can see, we tagged the image with a URL, pointing to a location in the `docker.io` registry, also known as [DockerHub](https://hub.docker.com/). If you want to share your experiment with others or to execute it in a compute cluster via CC-Agency, you have to push your locally created image to a Docker registry. You can either use a public or paid organization on DockerHub or setup a private Docker registry on your own server.

Login to your registry and push the image.

```bash
docker login docker.io
docker push docker.io/myorganization/myimage
```

In your RED file you can now reference your image under `container.settings.image.url`. If read access to your image is restricted via user credentials you can provide them as `container.settings.image.auth.username` and `container.settings.image.auth.password` respectively. See the [RED Container Engines](/docs/red-container-engines) documentation for more information.


## Nvidia-Docker

In order to use Cuda applications on Nvidia GPUs, you have to switch from `container.engine: "docker"` to `container.engine: "nvidia-docker"` in your RED file. For more details take a look at the [nvidia-docker container engine](/docs/red-container-engines#nvidia-docker) documentation.

The easiest way to make Cuda available to your containerized application is to start your Dockerfile with `FROM nvidia/cuda:9.0-base`. Of course the underlying host's driver and hardware must support the desired cuda version (e.g. 9.0).

See the following Dockerfile listing for an example.

```Dockerfile
FROM nvidia/cuda:9.0-base

TODO
```

For a complete list of available Cuda base images take a look at the their [DockerHub site](https://hub.docker.com/r/nvidia/cuda).
