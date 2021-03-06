---
title: "RED Beginner's Guide"
permalink: /docs/red-beginners-guide
---

This tutorial explains how to create a reproducible data-driven experiment and how to document it in a Reproducible Experiment Description (RED). RED is based on the Common Workflow Language (CWL), that is demonstrated as well.


# Prerequisites

Curious Containers is best supported on Linux distributions and all experiments run as CLI tools in Linux containers using Docker.

From the Curious Containers 8 release onwards, CC-FAICE supports Mac using [Docker for Mac](https://docs.docker.com/docker-for-mac/). From the Curious Containers 9 release onwards, CC-FAICE supports Windows using [Docker for Windows](https://docs.docker.com/docker-for-windows/). Both, Docker for Mac and Docker for Windows, internally use a virtual machines to run Linux containers.

The last section of this guide, [Upload Output to a Remote Destination](#upload-output-to-a-remote-destination), requires you to have write access to an arbitrary SSH server.


## Option 1: Linux Setup

If you are using a Linux distribution, please ensure that the packages `nano` (or another text editor), `python3`, `python3-pip`, `python3-venv`, `git` and a docker-engine are installed.

On Ubuntu 18.04:

```bash
sudo groupadd docker
sudo usermod -aG docker $(whoami)  # before docker install to avoid reboot
sudo apt-get update
sudo apt-get install nano python3 python3-pip python3-venv docker.io
```

On Fedora 30:

```bash
sudo groupadd docker
sudo usermod -aG docker $(whoami)  # before docker install to avoid reboot
sudo dnf install nano python3 python3-pip python3-venv moby-engine
```

Use `docker info`, to verify that the Docker daemon is running and that your user is allowed to connect.

If you plan on using the Nvidia GPU of your system later, you should install the [docker-ce](https://docs.docker.com/install/) version of Docker.
The `docker.io` or `moby-engine` versions from your Linux distribution's package repositories will not work with [Nvidia Container Toolkit](https://github.com/NVIDIA/nvidia-docker).


## Option 2: Mac Setup

1. [Install Docker for Mac](https://docs.docker.com/docker-for-mac/install/).
    * Use `docker info`, to verify that the Docker daemon is running and that your user is allowed to connect.
2. [Install Brew](https://brew.sh/index_de).
3. Install required packages via `brew`.

```bash
brew install nano python
```


## Option 3: Windows Setup

Please note, that while CC-FAICE runs on Windows, this guide is written for Bash on Linux or Mac.
CMD and Powershell on Windows require a different syntax.
Therefore you can only follow along this guide, if you are able to translate the code samples.

1. [Install Docker for Windows](https://docs.docker.com/docker-for-windows/install/).
    * Use `docker info`, to verify that the Docker daemon is running and that your user is allowed to connect.
2. [Install Miniconda3](https://docs.conda.io/en/latest/miniconda.html).

Open "Anaconda Powershell Prompt" from the Start Menu and install `cc-faice` via pip.

```
pip install cc-faice==9.*
faice --version
```

### Troubleshooting

If you are using `faice exec` to run an experiment and get an error message related to "npipe" support, then the Python package "pywin32" is not installed properly.
Follow the [setup instructions of pywin32](https://github.com/mhammond/pywin32#installing-via-pip) and run the post-install script in an admin shell.
The functionality is required to connect to Docker for Windows.
As an alternative, configure Docker for Windows to expose a TCP port on localhost and configure your `DOCKER_HOST` environment variable to point to the exposed Docker service before running `faice exec`.


## Option 4: Vagrant VM Setup

If you don't have access to a Linux system or just don't want to install Docker by hand, you can setup a provisioned vagrant VM to follow the tutorial.

First install Git, Vagrant and Virtualbox, then follow the instructions below.

```bash
git clone https://github.com/curious-containers/red-guide-vagrant.git
cd red-guide-vagrant
vagrant up
vagrant ssh
```


# Install CWL and RED tools

cwltool and CC-FAICE are tools used in the course of this guide.
They are both implemented in Python3 and should be installed under separate virtual environments (venv) to avoid conflicts.


## cwltool

cwltool is the reference implementation of CWL and not associated with the Curious Containers project.
Install the tool as follows.

```bash
# create installation directory
mkdir -p ~/.local/red-guide

# create venv
python3 -m venv ~/.local/red-guide/cwltool

# activate venv
. ~/.local/red-guide/cwltool/bin/activate

# install packages
pip install wheel
pip install cwltool

# deactivate venv
deactivate

# append venv bin directory to PATH
export PATH=${PATH}:${HOME}/.local/red-guide/cwltool/bin
```

Consider making the `PATH` change permanent by appending the line to your `~/.bashrc` file.

```bash
echo 'export PATH=${PATH}:${HOME}/.local/red-guide/cwltool/bin' >> ~/.bashrc
```

The `cwltool` command should now be available.

```bash
cwltool --version
cwltool --help
```


## CC-FAICE

CC-FAICE is the reference implementation of RED and part of the Curious Containers project.
Installation is equivalent to cwltool.

```bash
# create installation directory
mkdir -p ~/.local/red-guide

# create venv
python3 -m venv ~/.local/red-guide/cc-faice

# activate venv
. ~/.local/red-guide/cc-faice/bin/activate

# install packages
pip install wheel
pip install cc-faice==9.*

# deactivate venv
deactivate

# append venv bin directory to PATH
export PATH=${PATH}:${HOME}/.local/red-guide/cc-faice/bin
```

Consider making the `PATH` change permanent by appending the last line to your `~/.bashrc` or `~/.profile` file.

```bash
echo 'export PATH=${PATH}:${HOME}/.local/red-guide/cc-faice/bin' >> ~/.bashrc
```

The `faice` command should now be available.

```bash
faice --version
faice --help
```


# Sample Application

Lets first create our own small CLI application with Python3.
It's called `grepwrap`.

Create a new file and insert the Python3 code below with `nano grepwrap`. Then save and close the file.

```python
#!/usr/bin/env python3

from argparse import ArgumentParser
from subprocess import call

OUTPUT_FILE = 'out.txt'

parser = ArgumentParser(description='Search for query terms in text files.')
parser.add_argument(
   'query_term', action='store', type=str, metavar='QUERY_TERM',
   help='Search for QUERY_TERM in TEXT_FILE.'
)
parser.add_argument(
   'text_file', action='store', type=str, metavar='TEXT_FILE',
   help='TEXT_FILE containing plain text.'
)
parser.add_argument(
   '-A', '--after-context', action='store', type=int, metavar='NUM',
   help='Print NUM lines of trailing context after matching lines.'
)
parser.add_argument(
   '-B', '--before-context', action='store', type=int, metavar='NUM',
   help='Print NUM lines of leading context before matching lines.'
)
args = parser.parse_args()

command = 'grep {} {}'.format(args.query_term, args.text_file)

if args.after_context:
   command = '{} -A {}'.format(command, args.after_context)

if args.before_context:
   command = '{} -B {}'.format(command, args.before_context)

command = '{} > {}'.format(command, OUTPUT_FILE)

exit(call(command, shell=True))
```


Set the executable flag for `grepwrap`.

```bash
chmod u+x grepwrap
```

The program is a wrapper for `grep`. It stores results to `out.txt` and has a simplified interface. Use `./grepwrap --help` to show all CLI arguments.


Create a new file with sample data by inserting the text below with `nano in.txt`. Then save and close the file.

```text
FOO
BAR
BAZ
QUX
QUUX
```

Then execute `grepwrap` as follows.

```bash
./grepwrap -B 1 QU in.txt
```

In this case the command `./grepwrap -B 1 QU in.txt` is an **experiment** based on the program `grepwrap`, which has a defined **CLI** and has `python3` and `grep` as **dependencies**.
It is executed with `in.txt` as **input** file, as well as `-B 1` and `QU` as **input** arguments.
It produces a single file `out.txt` as **output**. Use `cat out.txt` to check the programs output.

You should add the directory, that contains the executble, to your `PATH` variable.
This way, you can run the program without having to specify the path to the executable.

```bash
export PATH=$(pwd):${PATH}
grepwrap -B 1 QU in.txt
```

The next steps of this guide, will demonstrate the formalization of the experiment, which allows for persistent storage, enables distribution and improves reproducibility. In order to do so, we need to describe the **CLI**, **dependencies**, **inputs** and **outputs**.


# Container Image

The next step is to explicitely document the runtime environment with all required dependencies of `grepwrap`.
Container technologies are useful to create this kind reproducible and distributable environment.

Create a new Dockerfile and insert the following description with `nano Dockerfile`.

```docker
FROM docker.io/debian:9.5-slim

RUN apt-get update \
&& apt-get install -y python3-venv \
&& useradd -ms /bin/bash cc

# switch user
USER cc

ENV PATH /home/cc/.local/bin:${PATH}

RUN mkdir -p /home/cc/.local/bin

# install connectors
RUN python3 -m venv /home/cc/.local/red \
&& . /home/cc/.local/red/bin/activate \
&& pip install wheel \
&& pip install red-connector-http==1.0 red-connector-ssh==1.2 \
&& ln -s /home/cc/.local/red/bin/red-connector-* /home/cc/.local/bin

# install app
ADD --chown=cc:cc grepwrap /home/cc/.local/bin/grepwrap
```

As can be seen in the Dockerfile, we extend a slim Debian image from the official [DockerHub](https://hub.docker.com/) registry.
To improve reproducibility, you should always add a very specific tag like `9.5-slim` or an [image digest](https://docs.docker.com/engine/reference/commandline/images/#list-image-digests).

As a first step, `python3-venv` is installed from Debian repositories which is used to create a Python virtual environment for the connectors.
Then a new unprivileged user `cc` is created. The name of this user is not relevant.
CC-FAICE will always start a container as the user last set by the `USER` keyword in a Dockerfile.
As a next step the RED connectors in a virtual environment using `pip`.
The `red-connector-http` and `red-connector-ssh` programs will be used to use to transfer data into and out of the Docker container when using a CC exectuion engine.
As a last step the `grepwrap` application is added to the image.
Please note, that the `ENV` command sets the `PATH` variable, such that `grepwrap` and the connectors are executable from any working directory.


Use the Docker client to build the image and name it `grepwrap`.

```bash
docker build --tag grepwrap .
```

Use `docker image list` to check if the new image exists.

To check if the container image is configured correctly, try running the installed commands in a container based on the new image.

```bash
docker run --rm grepwrap whoami  # should print cc
docker run --rm grepwrap grepwrap --help
docker run --rm grepwrap red-connector-http --version
docker run --rm grepwrap red-connector-ssh --version
```


# CWL

The Common Workflow Language (CWL) provides a [syntax](http://www.commonwl.org/v1.0/CommandLineTool.html) for describing a command line interface (CLI).
Curious Containers and the RED format build upon this CLI description syntax, but only support a subset of the CWL specification.
In other words, every CWL description compatible with RED is also compatible with the CWL standard (e.g. with [cwltool](https://github.com/common-workflow-language/cwltool), a CWL reference implementation) but not the other way round.

The supported CWL subset is specified as a part of the [RED JSON Schema](/docs/json-schema).
Use the following `faice` command to show the jsonschema.
The relevant section of the schema is `definitions.cli`.

```bash
faice schema show red
```

You can use `faice schema --help` and `faice schema show --help` to learn more about these subcommands.
The `faice schema list` command prints all available schemas.

Create a new file and insert the following CWL description with `nano grepwrap.cwl.yml`. Then save and close the file.

```yaml
cwlVersion: "v1.0"
class: "CommandLineTool"
baseCommand: "grepwrap"
doc: "Search for query terms in text files."

inputs:
  query_term:
    type: "string"
    inputBinding:
      position: 0
    doc: "Search for QUERY_TERM in TEXT_FILE."
  text_file:
    type: "File"
    inputBinding:
      position: 1
    doc: "TEXT_FILE containing plain text."
  after_context:
    type: "int?"
    inputBinding:
      prefix: "-A"
    doc: "Print NUM lines of trailing context after matching lines."
  before_context:
    type: "int?"
    inputBinding:
      prefix: "-B"
    doc: "Print NUM lines of leading context before matching lines."

outputs:
  out_file:
    type: "File"
    outputBinding:
      glob: "out.txt"
    doc: "Query results."

requirements:
  DockerRequirement:
    dockerPull: "grepwrap"
```


CWL uses job files to describe inputs. Create a new file and insert the following job with `nano job.yml`. Then save and close the file.


```yaml
query_term: "QU"
text_file:
  class: "File"
  location: "in.txt"
before_context: 1
```


Use `cwltool` to execute the experiment. The `--disable-pull` flag is used, because the Docker image is only available locally and cannot be pulled from a registry.

```bash
cwltool --disable-pull ./grepwrap.cwl.yml ./job.yml
```

The resulting files will be moved to the current working directory. Use `cat out.txt` to check the programs output.


## Push Image to Container Registry

In order to make the experiment portable, the `grepwrap` Docker image must be pushed to a [Docker registry](https://docs.docker.com/registry/).
This allows you to reference the image using a URL.
You can connect to a private registry or create a free account on [DockerHub](https://hub.docker.com/).
Please note, that the free DockerHub account will only allow publicly accessible images.

The following commands can be used to publish an image.
In this case, the image has already been pushed to the `curouscontainers` organization on DockerHub and it is not required to push the image yourself in order to follow the tutorial.
If you want to push the image to your own registry or [organization](https://docs.docker.com/docker-hub/orgs/), change the variable values accordingly.

```bash
REGISTRY=docker.io
ORGANIZATION=curiouscontainers
IMAGE=grepwrap
IMAGE_URL=${REGISTRY}/${ORGANIZATION}/${IMAGE}

docker login ${REGISTRY}

# rename image to full URL
docker tag ${IMAGE} ${IMAGE_URL}

# push the image to the registry
docker push ${IMAGE_URL}
```

You can now use the `${IMAGE_URL}` to refer to your image in the CWL file.

```yaml
requirements:
  DockerRequirement:
    dockerPull: "docker.io/curiouscontainers/grepwrap"
```

This allows you to run cwltool without the `--disable-pull` flag.

```bash
cwltool ./grepwrap.cwl.yml ./job.yml
```

# RED

The CWL `job.yml` has been used to reference input files in the local file system. To achieve reproducibility accross different computers, all input files should be accessed via network protocols instead of local filesystem paths.

Unfortunately, the CWL `location` keyword in a job file can only hold a single URI (e.g. `http://example.com`), which is a limiting factor when connecting to a non-standard API is required (e.g. the REST API of [XNAT](https://www.xnat.org/) 1.6.5 is not stateless and requires explicit session deletion). RED Execution Engines like CC-FAICE therefore use dedicated connector programs provided by the user as part of the container image. If you go back to [Container Image](#container-image) section, you can see that `red-connector-http` is used in this guide, but other connector implementations for various network protocols exist.

Create a new file and insert the following RED data with `nano grepwrap.red.yml`.

```yaml
redVersion: "9"
cli:
  cwlVersion: "v1.0"
  class: "CommandLineTool"
  baseCommand: "grepwrap"
  doc: "Search for query terms in text files."

  inputs:
    query_term:
      type: "string"
      inputBinding:
        position: 0
      doc: "Search for QUERY_TERM in TEXT_FILE."
    text_file:
      type: "File"
      inputBinding:
        position: 1
      doc: "TEXT_FILE containing plain text."
    after_context:
      type: "int?"
      inputBinding:
        prefix: "-A"
      doc: "Print NUM lines of trailing context after matching lines."
    before_context:
      type: "int?"
      inputBinding:
        prefix: "-B"
      doc: "Print NUM lines of leading context before matching lines."

  outputs:
    out_file:
      type: "File"
      outputBinding:
        glob: "out.txt"
      doc: "Query results."

inputs:
  query_term: "QU"
  text_file:
    class: "File"
    connector:
      command: "red-connector-http"
      access:
        url: "https://raw.githubusercontent.com/curious-containers/red-guide-vagrant/master/red-beginners-guide/in.txt"
  before_context: 1

container:
  engine: "docker"
  settings:
    image:
      url: "docker.io/curiouscontainers/grepwrap"

execution:
  engine: "ccfaice"
  settings: {}
```

This RED file contains five sections:

* `redVersion`: specifies the RED format version
* `cli`: contains the application's CLI description in CWL format, without a `requirements` section
* `inputs`: is similar to a CWL job description, but requires RED connectors
* `container`: container engine settings to replace the `requirements.DockerRequirement` section of CWL
* `execution`: set the RED Execution Engine to be `ccfaice`.


The RED inputs format is very similar to a CWL job. Note that the `connector` keyword replaces CWL's `location`.
Each connector requires the `command` and `access` keywords.
The information contained in `access` is validated by the connector itself and therefore varies for different connector implementations.
Curious Containers cannot access files from local file paths, because it would the defeat the purpose of a portable experiment.
Therefore the `in.txt` was pushed to GitHub, where it can be accessed from any computer using an HTTP URL.

Use the `faice exec` is a RED client, that reads the information in the `execution` section of the RED file and hands the experiment to the specified RED Execution Engine `ccfaice`.
`ccfaice` is a built-in RED Execution Engine, that will run the experiment with your local Docker configuration

```bash
faice exec grepwrap.red.yml
```

The output file will be automatically copied from the container filesystem to the `outputs` directory in your current working directory. Use `cat outputs/out.txt` to check the programs output.


## Upload Output to a Remote Destination

As demonstrated in this guide, the RED Execution Engine of CC-FAICE will copy the `out.txt` file to the local filesystem for the user to inspect.
This is a convenience feature of CC-FAICE, that is not available in other execution engines like CC-Agency.
Instead, output files and directories should be uploaded to a remote server location using connectors.

Since CC is a framework and not a tightly integrated research platform, you must have access to a storage server.
In this section, the non-public storage server `avocado01.f4.htw-berlin.de` is used.
Accessing `avocado01.f4.htw-berlin.de` requires you to be in the HTW Berlin university network or to use the [HTW Berlin VPN](https://anleitungen.rz.htw-berlin.de/de/vpn/).
If you do not have access to this server, you have to replace the output connector `access` information in the RED file to fit your **own SSH server**.

Append the following section to the RED file using `nano grepwrap.red.yml`.

{% raw %}
```yaml
outputs:
  out_file:
    class: "File"
    connector:
      command: "red-connector-ssh"
      access:
        host: "avocado01.f4.htw-berlin.de"
        auth:
          username: "{{ssh_username}}"
          password: "{{ssh_password}}"
        filePath: "out.txt"
```

Please note, that `{{ssh_username}}` and `{{ssh_password}}` are [variables](/docs/red-format-protecting-credentials).
This is a powerful feature of RED, that allows you to share or publish these files, even if the configuration requires authentication credentials.
The RED client `faice exec` will interactively ask you to fill in this information on the command-line.
You have the option to store these values in a keyring service, if one is installed on your system.
{% endraw %}

Please note, that CC will **not** use any SSH private keys that are stored on your system.
If you want to use key authentication, the SSH private key and passphrase must be specified in the RED file according to the  [red-connector-ssh](/docs/red-connector-ssh#send-file) docs.

The name `outputs.out_file` refers to the arbitrary name, that is specified under `cli.outputs.out_file`.
While the information under `cli.outputs.out_file` tells the connector where the file is located in the container filesytem, the information under `outputs.out_file` tells the connector the desired upload destination.
There can only be a single output connector per output file.

Again, use the RED client `faice exec` to start the experiment.
The RED client will hand the experiment to the builtin RED Execution Engine `ccfaice`.
`ccfaice` will read the `outputs` section and use the connectors, instead of copying the outputs to your local filesystem.

```bash
faice exec grepwrap.red.yml
```

Since the specified `filePath` is relative, the file will be uploaded to the SSH user's home directory. It can be downloaded via `scp`.

```bash
SSH_USERNAME=christoph
SSH_HOST=avocado01.f4.htw-berlin.de

scp ${SSH_USERNAME}@${SSH_HOST}:out.txt .
```
