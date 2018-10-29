# RED Engines

RED Engines are used to execute programs in a virtual environment like a docker container.
There are currently two engines available in RED. The first one is "docker".
To support nvidia GPUs there is a second engine called "nvidia-docker". If you want your application to use GPUs you have to use nvidia-docker.

To use docker or nvidia-docker you first have to build a docker image with `docker build`
(For more information see the [RED Beginners Guide](https://www.curious-containers.cc/red-beginners-guide.html#container-image) or
the the [Docker build Documentation](https://docs.docker.com/engine/reference/commandline/build/)).

## Table of Contents

| Engine |
| --- |
| [Docker](#docker) |
| [Nvidia-Docker](#nvidia-docker) |

## Docker
Docker makes it possible to run the applications specified in the RED file in an isolated and secure environment.
To run your application inside a docker container you have to specify which docker image to use.

This docker image should meet the following conditions:
- Should contain a linux user with name "cc"
- You should be able to run `ccagent --version` inside a container using this image
- You should be able to run your application inside a container using this image
- The image should be accessible from where you want to start your container
  (For example via `docker pull <image-name>`)

Information about how to build such an image can be found in the [RED Beginners Guide](https://www.curious-containers.cc/red-beginners-guide.html#container-image).

Additional you can specify hardware limitations which are applied to the container.

### Example Configuration

```yaml
container:
  engine: "docker"
  settings:
    image:
      url: "grepwrap-image"
      auth:
        username: "username"
        password: "password"
        method: "BASIC"
    ram: 256
```

### Explanation

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| engine | string | no | | = "docker" |
| settings | dict | no | | Container information |
| settings.image | dict | no | | The image |
| settings.image.url | string | no | | The URL of the image |
| settings.image.auth | dict | yes | | Authentication information |
| settings.image.auth.username | string | no | | Username |
| settings.image.auth.password | string | no | | Password |
| settings.image.auth.method | enum: BASIC, DIGEST | yes | BASIC | Authentication method |
| settings.ram | int | yes | | The RAM limitation for the container in MB |

## Nvidia-Docker

To enable support for nvidia GPUs you can use the nvidia-docker engine. The configuration for nvidia-docker is similar to the docker configuration.
Additional there are hardware requirements for GPUs. If the executing computer can't fulfill the GPU requirements the execution will not be successful.

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

### Explanation

| Access | Type | Optional | Default | Description |
| --- | --- | --- | --- | --- |
| engine | string | no | | = "docker" |
| settings | dict | no | | Container information |
| settings.image | dict | no | | The image |
| settings.image.url | string | no | | The URL of the image |
| settings.image.auth | dict | yes | | Authentication information |
| settings.image.auth.username | string | no | | Username |
| settings.image.auth.password | string | no | | Password |
| settings.image.auth.method | enum: BASIC, DIGEST | yes | BASIC | Authentication method |
| settings.ram | int | yes | | The RAM limitation for the container in MB |
| settings.gpus.min\_vram | int | yes | | The minimal VRAM that must be present in MB |
| settings.gpus.count | int | yes | | The number of GPUs to allocate |
