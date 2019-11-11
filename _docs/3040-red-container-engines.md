---
title: "RED Container Engines"
permalink: /docs/red-container-engines
---

RED Container Engines are used to execute programs in a virtual environment like a docker container. Currently only Docker is supported as a container engine in Curious Containers.


# Docker

Docker makes it possible to run the applications specified in the RED file in an isolated and secure environment with optional resource limitations. To run your application inside a docker container you have to specify which docker image to use. Information about how to build such an image can be found in the [Container Images](/docs/container-images) documentation.


## Nvidia GPUs

If you are using the local RED execution engine `ccfaice` and want your application to use Nvidia GPUs for CUDA support, you have to install the proprietary Nvidia GPU driver and the [Nvidia Container Toolkit](https://github.com/NVIDIA/nvidia-docker) or its predecessor [Nvidia-Docker 2](https://github.com/NVIDIA/nvidia-docker).

It is not required to have CUDA installed on the host operating system, but it must be installed in the container image. See the [CUDA](/docs/container-images#cuda) section in the container images documentation for more information on building compatible images.


## Settings

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| image | dict | no | | The image |
| image.url | string | no | | The URL of the image |
| image.auth | dict | yes | | Authentication information |
| image.auth.username | string | no | | Username |
| image.auth.password | string | no | | Password |
| ram | int | CC-FAICE: yes; CC-Agency: no | | The RAM limitation for the container in MB |
| gpus | dict | yes | | Make Nvidia GPUs available in a Docker container |
| gpus.vendor | string | no | | Currently only "nvidia" allowed |
| gpus.count | int | if gpus.devices is specified | | The number of GPUs to allocate |
| gpus.devices | list | if gpus.count is specified | | A list of GPU devices with specified requirements |
| gpus.devices.${item}.vramMin | int | no | | The minimal VRAM that must be present in MB |


## Example Configuration

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


## Example Configurations with Nvidia GPUs

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
    gpus:
      vendor: "nvidia"
      devices:
        - vramMin: 256
        - vramMin: 256
```

The executing container will only use 256 MB RAM and two GPUs with at least 256 MB VRAM. A `vramMax` parameter does not exist, because GPUs are always allocated as a complete device.

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
    gpus:
      vendor: "nvidia"
      count: 1
```

This is an alternative way to specify the number of required GPUs, without specifying `vramMin` for each device.
