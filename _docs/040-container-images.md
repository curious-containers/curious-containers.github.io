---
title: "Container Images"
permalink: /docs/container-images
---

The following sections show how to prepare a container image, for the two container engines `docker` and `nvidia-docker`.


## Docker

There are many ways to build a Docker image. In our examples we chose `docker.io/debian:9.5-slim` as a base image, because it is relatively small and dependencies can easily be installed from the Debian package repositories via `apt-get install`. A `/bin/sh` shell, as it is preconfigured in most images, is required.

For the image to be used effectively, RED connector commandline application have to be installed. In order to use any of the standard connectors provided by the Curious Containers project, a Python3 interpreter must be installed as well. Since Docker provides a clean environment by default, we have to set the `PATH` environment variables explicitely. Please note, that your own application also needs to be located in a directory which is included in `PATH`.

Another requirement is, that the application and the RED connectors are executed as user with uid:gid set to `1000:1000`. The debian base image does not have such a user configured. We can therefore create the first user called `cc`, wich will by default be assigned the uid:gid pair `1000:1000`. Please note, that the user's name does not matter.

The Dockerfile below does not include an application. For a more complete example we advise you to work through the [RED Beginner's Guide](/docs/red-beginners-guide).

```docker
FROM docker.io/debian:9.5-slim

RUN apt-get update \
&& apt-get install -y python3-venv \
&& useradd -ms /bin/bash cc

# switch user
USER cc

# install connectors
RUN python3 -m venv /home/cc/.local/red \
&& . /home/cc/.local/red/bin/activate \
&& pip install wheel \
&& pip install red-connector-http==0.3 red-connector-ssh==0.5

ENV PATH="/home/cc/.local/red/bin:${PATH}"

# add commands here to install your application
# ...
```

After adding your own application to the Dockerfile, create the image using the `docker build` command. The `.` at the end of the command indicates, that `docker build` will pick up your Dockerfile from the current working directory.

```bash
docker build -t docker.io/myorganization/myimage .
```

To check your image, you can run a container based on the image with uid:gid `1000:1000`.

```bash
docker run --rm -u 1000:1000 docker.io/myorganization/myimage red-connector-http --version
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

# [...]
```

For a complete list of available Cuda base images take a look at the their [DockerHub site](https://hub.docker.com/r/nvidia/cuda).
