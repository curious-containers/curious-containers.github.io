---
title: "RED Beginner's Guide"
permalink: /docs/red-beginners-guide
---

This tutorial explains how to create a reproducible data-driven experiment and how to document it in a Reproducible Experiment Description (RED). RED is based on the Common Workflow Language (CWL), that is demonstrated as well.


## Prerequisites

Curious Containers is best supported on Linux distributions and all experiments run as CLI tools in Linux containers using Docker.

From the Curious Container 8 release onwards, CC-FAICE supports Mac using [Docker for Mac](https://docs.docker.com/docker-for-mac/). Please note, that Docker for Mac internally uses a virtual machine to run Linux containers.


### Option 1: Linux Setup

If you are using a Linux distribution, please ensure that the following packages are installed.

* nano (or another text editor)
* python3
* python3-pip
* python3-venv
* git
* docker

On Ubuntu 18.04:

```bash
sudo groupadd docker  # to avoid reboot
sudo apt-get update
sudo apt-get install nano python3 python3-pip python3-venv git docker.io
```

On Fedora 30:

```bash
sudo groupadd docker  # to avoid reboot
sudo dnf install nano python3 python3-pip python3-venv git moby-engine
```

Use `docker info`, to verify that the Docker daemon is running and that your user is allowed to connect.


### Option 2: Mac Setup

1. [Install Docker for Mac](https://docs.docker.com/docker-for-mac/install/).
    * Use `docker info`, to verify that the Docker daemon is running and that your user is allowed to connect.
2. [Install Brew](https://brew.sh/index_de).
3. Install required packages via `brew`.

```bash
brew install nano python git
```


### Option 3: Windows Setup

Windows support is planned for the upcoming Curious Container 8.1 release. As of now, please skip to Option 4 or use a different virtualization technology, like [Hyper-V](https://docs.microsoft.com/en-us/windows-server/virtualization/hyper-v/supported-ubuntu-virtual-machines-on-hyper-v) or [Windows Subsystem for Linux 2 (WSL2)](https://devblogs.microsoft.com/commandline/wsl-2-is-now-available-in-windows-insiders/).


### Option 4: Vagrant VM Setup

If you don't have access to a Linux system or just don't want to install Docker by hand, you can setup a provisioned vagrant VM to follow the tutorial.

First install Git, Vagrant and Virtualbox, then follow the instructions below.

```bash
git clone https://github.com/curious-containers/red-guide-vagrant.git
cd red-guide-vagrant
vagrant up
vagrant ssh
```


## Install CWL and RED tools

CWLTool and CC-FAICE are tools used in the course of this guide. They are both implemented in Python3 and should be installed under separate virtual environments (venv) to avoid conflicts.


### CWLTool

CWLTool is the reference implementation of CWL and not associated with the Curious Containers project. Install the tool as follows.

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


### CC-FAICE

CC-FAICE is the reference implementation of RED and part of the Curious Containers project. Installation is equivalent to CWLTool.

```bash
# create installation directory
mkdir -p ~/.local/red-guide

# create venv
python3 -m venv ~/.local/red-guide/cc-faice

# activate venv
. ~/.local/red-guide/cc-faice/bin/activate

# install packages
pip install wheel
pip install cc-faice==8.*

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


## Sample Application

Lets first create our own small CLI application with Python3. It's called `grepwrap`.

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

The program is a wrapper for `grep`. It stores results to `out.txt` and has a simplified interface. Use `grepwrap --help` to show all CLI arguments.


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

Use `cat out.txt` to check the programs output.

In this case the command `grepwrap -B 1 QU in.txt` is an **experiment** based on the program `grepwrap`, which has a defined **CLI** and has `python3` and `grep` as **dependencies**. It is executed with `in.txt` as **input** file, as well as `-B 1` and `QU` as **input** arguments. It produces a single file `out.txt` as **output**.

The next steps of this guide, will demonstrate the formalization of the experiment, which allows for persistent storage, enables distribution and improves reproducibility. In order to do so, we need to describe the **CLI**, **dependencies**, **inputs** and **outputs**.


## Container Image

The next step is to explicitely document the runtime environment with all required dependencies of `grepwrap`. Container technologies are useful to create this kind reproducible and distributable environment.

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
&& pip install red-connector-http==1.0 \
&& ln -s /home/cc/.local/red/bin/red-connector-* /home/cc/.local/bin

# install app
ADD --chown=cc:cc grepwrap /home/cc/.local/bin/grepwrap
```

As can be seen in the Dockerfile, we extend a slim Debian image from the official [DockerHub](https://hub.docker.com/) registry. To improve reproducibility, you should always add a very specific tag like `9.5-slim` or an [image digest](https://docs.docker.com/engine/reference/commandline/images/#list-image-digests).

As a first step, `python3-venv` is installed from Debian repositories which is used to create a Python virtual environment for the connectors. Then a new user `cc` is created. The name of this user is not relevant, but since it is the first user created in this image, user id and group id `1000` will be assigned. This is important, because CC-FAICE will always start a container with `uid:gid` set to `1000:1000`. This behavior is equivalent to the CWL reference implementation CWLTool. As a next step the Dockerfile switches to the `cc` user and installs the application. In this case the `grepwrap` script is added to the image. As a last step we install `red-connector-http`, that we want to use to transfer data into and out of the Docker container.

Use the Docker client to build the image and name it `grepwrap`.

```bash
docker build --tag grepwrap .
```

Use `docker image list` to check if the new image exists.

To check if the container image is configured correctly, try running the installed commands in a container based on the new image.

```bash
docker run --rm -u 1000:1000 grepwrap whoami  # should print cc
docker run --rm -u 1000:1000 grepwrap grepwrap --help
docker run --rm -u 1000:1000 grepwrap red-connector-http --version
```


## CWL

The Common Workflow Language (CWL) provides a [syntax](http://www.commonwl.org/v1.0/CommandLineTool.html) for describing a commandline tool's interface (CLI). Curious Containers and the RED format build upon this CLI description syntax, but only support a subset of the CWL specification. In other words, every CWL description compatible with RED is also compatible with the CWL standard (e.g. with [cwltool](https://github.com/common-workflow-language/cwltool), a CWL reference implementation) but not the other way round.

The supported CWL subset is specified as a jsonschema description in the `cc-core` Python package. Use the following `faice` command to show the jsonschema.

```bash
faice schema show cwl
```

You can use `faice schema --help` and `faice schema show --help` to learn more about these subcommands. The `faice schema list` command prints all available schemas.

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


## RED

The CWL `job.yml` has been used to reference input files in the local file system. To achieve reproducibility accross different computers, all input files should be accessed via network protocols instead of local filesystem paths.

Unfortunately, the CWL `location` keyword in a job file can only hold a single URI (e.g. `http://example.com`), which is a limiting factor when connecting to a non-standard API is required (e.g. the REST API of [XNAT](https://www.xnat.org/) 1.6.5 is not stateless and requires explicit session deletion). RED execution engines like CC-FAICE therefore use dedicated connector programs provided by the user as part of the container image. If you go back to [Container Image](#container-image) section, you can see that `red-connector-http` is used in this guide, but other connector implementations for various network protocols exist.

Create a new file and insert the following RED data with `nano grepwrap.red.yml`.

```yaml
redVersion: "8"
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

container:
  engine: "docker"
  settings:
    image:
      url: "grepwrap"

inputs:
  query_term: "QU"
  text_file:
    class: "File"
    connector:
      command: "red-connector-http"
      access:
        url: "https://raw.githubusercontent.com/curious-containers/red-guide-vagrant/master/in.txt"
  before_context: 1
```

This minimal RED file contains four sections:

* `redVersion`: specifies the RED format version
* `cli`: contains the application's CLI description in CWL format, without a `requirements` section
* `container`: container engine settings to replace the `requirements.DockerRequirement` section of CWL
* `inputs`: is similar to a CWL job description, but requires RED connectors


The RED inputs format is very similar to a CWL job. Note that the `connector` keyword replaces CWL's `location`. Each connector requires the `command` and `access` keywords. The information contained in `access` is validated by the connector itself and therefore varies for different connector implementations. Refer to the RED documentation for more information.

Learn more about the container engine description by showing the corresponding jsonschema with `faice schema show red-engine-container-docker` (also see [RED Container Engines](/docs/red-container-engines)).

Use the `faice agent red` commandline tool to execute the experiment.

```bash
faice agent red --disable-pull grepwrap.red.yml
```

The output file will be moved to the `outputs` directory. Use `cat outputs/out.txt` to check the programs output.


### Push Image to Container Registry

In order to have a fully portable experiment, the `grepwrap` Docker image must be pushed to a [Docker registry](https://docs.docker.com/registry/). This allows you to reference the image using a URL in the RED file. You can connect to a private registry or create a free account on [DockerHub](https://hub.docker.com/). Please note, that the free DockerHub account will only allow publicly accessible images.

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

Then open the RED file with `nano grepwrap.red.yml` and change the image URL.

```yaml
container:
  engine: "docker"
  settings:
    image:
      url: "docker.io/curiouscontainers/grepwrap"
```

If you run `faice agent red` again, you won't need the `--disable-pull` flag anymore.

```bash
faice agent red grepwrap.red.yml
```


### Specify RED Execution Engine

Since the experiment has now been tested with `faice agent red`, the RED execution engine of CC-FAICE, we can specify it in the optional `execution` section of the RED document. Open the file and append the following RED data with `nano grepwrap.red.yml`.

```yaml
execution:
  engine: "ccfaice"
  settings: {}
```

Please note, that the `settings` dictionary must be empty. For other RED execution engines, like `ccagency`, various settings are possible in this section.

Now, that the RED execution engine is specified, we can invoke `faice exec` to run the experiment. This tool will read the `execution` section and automatically hand the RED file to specified engine.

```bash
faice exec grepwrap.red.yml
```
