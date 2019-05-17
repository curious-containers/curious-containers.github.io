---
title: "RED Container Engines"
permalink: /docs/red-container-engines
---

RED Container Engines are used to execute programs in a virtual environment like a docker container.

There are currently two engines available in Curious Containers, `docker` and `nvidia-docker`.

If you want your application to use Nvidia GPUs for CUDA support you have to use `nvidia-docker`.


## Docker

Docker makes it possible to run the applications specified in the RED file in an isolated and secure environment.

To run your application inside a docker container you have to specify which docker image to use. Information about how to build such an image can be found in the [Container Images](/docs/container-images) documentation.

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
| ram | int | yes | | The RAM limitation for the container in MB |


## Nvidia-Docker

To enable support for nvidia GPUs you can use the `nvidia-docker` engine. The configuration for `nvidia-docker` is similar to the `docker` configuration. Additional there are hardware requirements for GPUs. If the executing computer can't fulfill the GPU requirements the execution will not be successful.

This engine requires CUDA to be installed in the container images. See the [Nvidia Docker](/docs/container-images#nvidia-docker) section in the container images documentation for more information on building compatible images.


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
    ram: 256
    gpus:
      - minVram: 256
      - minVram: 256
```


This configuration will try to pull the image `example-image`, with the supplied authentication. The executing container will only use 256 MB RAM and two GPUs with at least 256 MB VRAM. A maxVram parameter does not exist, because GPUs are always allocated as a complete device.

As an alternative, the `count` of required GPUs can be given, without specifying `minVram` parameters:

```yaml
container:
  engine: "nvidia-docker"
  settings:
    # ...
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
| ram | int | yes | | The RAM limitation for the container in MB |
| gpus.minVram | int | yes | | The minimal VRAM that must be present in MB |
| gpus.count | int | yes | | The number of GPUs to allocate |
