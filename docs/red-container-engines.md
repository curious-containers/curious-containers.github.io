---
title: "RED Container Engines"
layout: single
toc: true
sidebar:
  nav: "docs"
---

RED Container Engines are used to execute programs in a virtual environment like a docker container.
There are currently two engines available in RED. The first one is "docker".
To support nvidia GPUs there is a second engine called "nvidia-docker". If you want your application to use GPUs you have to use nvidia-docker.

To use docker or nvidia-docker you first have to build a docker image with `docker build`
(For more information see the [RED Beginners Guide](https://www.curious-containers.cc/red-beginners-guide.html#container-image) or
the [Docker build Documentation](https://docs.docker.com/engine/reference/commandline/build/)).


## Docker

Docker makes it possible to run the applications specified in the RED file in an isolated and secure environment.
To run your application inside a docker container you have to specify which docker image to use.

This docker image should meet the following conditions:
- The image must contain a Linux user with `uid:gid == 1000:1000` (usually the first user created after `root`)
- `ccagent --version` must be executable in the container as uid:gid 1000:1000
- Your application must be executable in the container as uid:gid 1000:1000
- For portability the image should be downloadable from a docker registry via its URL.  
  (For example via `docker pull <image-url>`)

Information about how to build such an image can be found in the [RED Beginners Guide](https://www.curious-containers.cc/red-beginners-guide.html#container-image).

Additional you can specify hardware limitations which are applied to the container.


### Example Configuration

```yaml
container:
  engine: "docker"
  settings:
    image:
      url: "example-image"
      auth:
        username: "username"
        password: "password"
        method: "BASIC"
    ram: 256
```


### Settings

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| image | dict | no | | The image |
| image.url | string | no | | The URL of the image |
| image.auth | dict | yes | | Authentication information |
| image.auth.username | string | no | | Username |
| image.auth.password | string | no | | Password |
| image.auth.method | enum: BASIC, DIGEST | yes | BASIC | Authentication method |
| ram | int | yes | | The RAM limitation for the container in MB |


## Nvidia-Docker

To enable support for nvidia GPUs you can use the nvidia-docker engine. The configuration for nvidia-docker is similar to the docker configuration. Additional there are hardware requirements for GPUs. If the executing computer can't fulfill the GPU requirements the execution will not be successful.


### Example Configuration

```yaml
container:
  engine: "nvidia-docker"
  settings:
    image:
      url: "example-image"
      auth:
        username: "username"
        password: "password"
        method: "BASIC"
    ram: 256
    gpus:
      - min_vram: 256
      - min_vram: 256
```


This configuration will try to pull the image "example-image", with the supplied authentication. The executing container will only use 256 MB RAM and two GPUs with at least 256 MB VRAM.

Also possible to allocate one ore more GPUs is the following configuration.

```yaml
container:
  image:
    # [...]
  settings:
    # [...]
    gpus:
      count: 1
```


### Settings

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| image | dict | no | | The image |
| image.url | string | no | | The URL of the image |
| image.auth | dict | yes | | Authentication information |
| image.auth.username | string | no | | Username |
| image.auth.password | string | no | | Password |
| image.auth.method | enum: BASIC, DIGEST | yes | BASIC | Authentication method |
| ram | int | yes | | The RAM limitation for the container in MB |
| gpus.min\_vram | int | yes | | The minimal VRAM that must be present in MB |
| gpus.count | int | yes | | The number of GPUs to allocate |
