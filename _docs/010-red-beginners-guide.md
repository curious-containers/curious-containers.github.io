---
title: "RED Beginner's Guide"
permalink: /docs/red-beginners-guide
---

This tutorial explains how to create a reproducible data-driven experiment and how to document it in a Reproducible Experiment Description (RED).


## Prerequisites

This tutorial requires a **Linux** distribution, where nano (or another text editor), python3-pip, git and [Docker](https://www.docker.com/) are installed.

If a Linux distribution is not already installed on your computer, use [Vagrant](https://www.vagrantup.com/) to create a Virtual Machine (VM) with your preferred operating system (see [Vagrant VM Setup](#vagrant-vm-setup-optional))


### Vagrant VM Setup (Optional)

If you don't have access to a Linux system or just don't want to install Docker by hand, you can setup a provisioned vagrant VM to follow the tutorial. This is entirely optional.

First install Git, Vagrant and Virtualbox, then follow the instructions below.

```bash
git clone https://github.com/curious-containers/red-guide-vagrant.git
cd red-guide-vagrant
vagrant up
vagrant ssh
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


Set the executable flag for `grepwrap` and add the current directory to the `PATH` environment variable.

```bash
chmod u+x grepwrap
export PATH=$(pwd):${PATH}
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
grepwrap -B 1 QU in.txt
```


Use `cat out.txt` to check the programs output.

In this case the command `grepwrap -B 1 QU in.txt` is an **experiment** based on the program `grepwrap`, which has a defined **CLI** and has `python3` and `grep` as **dependencies**. It is executed with `in.txt` as **input** file, as well as `-B 1` and `QU` as **input** arguments. It produces a single file `out.txt` as **output**.

The next steps of this guide, will demonstrate the formalization of the experiment, which allows for persistent storage, enables distribution and improves reproducibility. In order to do so, we need to describe the **CLI**, **dependencies**, **inputs** and **outputs**.


## Install CC-Faice and CC-Core

Install the current version of `cc-faice`, which will also install a compatible version of `cc-core` as a dependency.

```bash
pip3 install --user cc-faice==6.0.0
```


* `cc-core` provides the `ccagent` commandline tool
* `cc-faice` provides the `faice` commandline tool

Both tools are located in Python's script directory, which should be included in the `PATH` environment variable. The following code prints their version numbers.

```bash
ccagent --version
faice --version
```


The `--help` argument shows available subtools and CLIs.

```bash
ccagent --help
faice --help
```


If these tools cannot be found, you should modify `PATH` (e.g. append `${HOME}/.local/bin`) or fall back to executing the tools as Python modules.

```bash
python3 -m cc_core.agent --version
python3 -m cc_faice --version
```


Please note that `cc-core` and `cc-faice` are compatible if the first two numbers of their versions match, as described in the [Versioning](/docs/versioning) documentation.


## CWL (ccagent)

The Common Workflow Language (CWL) provides a [syntax](http://www.commonwl.org/v1.0/CommandLineTool.html) for describing a commandline tool's interface (CLI). Curious Containers and the RED format build upon this CLI description syntax, but only support a subset of the CWL specification. In other words, every CWL description compatible with RED is also compatible with the CWL standard (e.g. with [cwltool](https://github.com/common-workflow-language/cwltool), a CWL reference implementation) but not the other way round.

The supported CWL subset is specified as a jsonschema description in the `cc-core` Python package. Use the following `faice` command to show the jsonschema.

```bash
faice schema show cwl
```

You can use `faice schema --help` and `faice schema show --help` to learn more about these subcommands. The `faice schema list` command prints all available schemas.

Create a new file and insert the following CWL description with `nano grepwrap-cli.cwl`. Then save and close the file.

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
```


CWL uses job files to describe inputs. Create a new file and insert the following job with `nano job.yml`. Then save and close the file.


```yaml
query_term: "QU"
text_file:
  class: "File"
  path: "in.txt"
before_context: 1
```


Use the `ccagent cwl` commandline tool to execute the experiment. This is equivalent to `cwltool ./grepwrap-cli.cwl ./job.yml`.

```bash
ccagent cwl ./grepwrap-cli.cwl ./job.yml
```

To get debug information with detailed exceptions use `--debug`.

```bash
ccagent cwl --debug ./grepwrap-cli.cwl ./job.yml > debug.yml
```

The resulting files will be moved to the current working directory.



## RED (ccagent)

The CWL `job.yml` has been used to reference input files in the local file system. To achieve reproducibility accross different computers, which is the goal of RED and FAICE, all input files should be downloadable from remote hosts and all output files should be uploadable to remote hosts.

Although the CWL specification also supports remote input files via the `location` keyword in a job file, it lacks the possibility to send output files to remote hosts. In addition the `location` value can only be a single string containing a URI (e.g. `http://example.com`), which is a limiting factor when connecting to a non-standard API is required (e.g. the REST API of [XNAT](https://www.xnat.org/) 1.6.5 is not stateless and requires explicit session deletion).

Create a new file and insert the following RED data with `nano red.yml`.

```yaml
redVersion: "6"
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
        url: "https://raw.githubusercontent.com/curious-containers/vagrant-quickstart/master/in.txt"
  before_context: 1
```


This minimal RED file contains three sections:

* `redVersion`: specifies the RED format version
* `cli`: contains the application's CLI description in CWL format
* `inputs`: is similar to a CWL job description, but references RED connectors


The RED inputs format is very similar to a CWL job. Note that connectors only work with files, and that the `connector` keyword replaces `path` and `location`. Each connector requires the `command` and `access` keywords for a connector. The information contained in `access` is validated by the connector itself and therefore varies for different connector implementations.

In order to use the connector it must be installed and its command must be executable. The Curious Containers provides various connectors. The HTTP connector can be installed as follows.

```bash
pip3 install --user --upgrade red-connector-http==0.2
```

This package provides three different connector, which should now be executable.

```bash
red-connector-http --version
red-connector-http-json --version
red-connector-http-mock-send --version
```


Use the `ccagent red` commandline tool to execute the experiment.

```bash
ccagent red ./red.yml
```

The resulting files will be moved to the current working directory.

The RED format also allows for connector descriptions for output files. Open the existing RED file and append the following `outputs` section with `nano red.yml`.

```yaml
outputs:
  out_file:
    class: "File"
    connector:
      command: "red-connector-http"
      access:
        url: "http://localhost:5000/server-out.txt"
```


Usually an external host with a static IP / domain name and a proper Authorization configuration should be chosen for this. This improves reproducibility, because all destinations of the original experiment results are well documented.

For the purpose of this guide, we temporarily start a local HTTP server on TCP PORT 5000 to receive the output file.

```bash
# start server as background job
faice file-server &
```

Use the `ccagent red` commandline tool with the `--outputs` flag to execute the experiment. Use `--outputs` to enable the specified connectors. Otherwise the `outputs` section of your RED file will be ignored and resulting files will be moved to the current working directory.

```bash
ccagent red --outputs ./red.yml
```

The `faice file-server` is programmed to use the file name specified in the URL. Use `cat server-out.txt`
to check the programs output.

You can stop the file-server as follows.

```bash
# terminate the last background job
kill %%
```


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

# install connectors
RUN python3 -m venv /home/cc/.local/red \
&& . /home/cc/.local/red/bin/activate \
&& pip install wheel \
&& pip install red-connector-http==0.2

ENV PATH="/home/cc/.local/red/bin:${PATH}"

# install app
ADD --chown=cc:cc grepwrap /home/cc/.local/bin/grepwrap

ENV PATH="/home/cc/.local/bin:${PATH}"
```


As can be seen in the Dockerfile, we extend a slim Debian image from the official [DockerHub](https://hub.docker.com/) registry. To improve reproducibility, you should always add a very specific tag like `9.5-slim` or an [image digest](https://docs.docker.com/engine/reference/commandline/images/#list-image-digests).

As a first step, `python3-venv` is installed from Debian repositories which is used to create a Python virtual environment for the connectors. Then a new user `cc` is created. The name of this user is not relevant, but since it is the first user created in this image, user id and group id `1000` will be assigned. This is important, because `faice` will always start a container with `uid:gid` set to `1000:1000`. This behavior is equivalent to the CWL reference implementation `cwltool`. As a next step the Dockerfile switches to the `cc` user and installs the application. In this case the `grepwrap` script is added to the image. As a last step we install `red-connector-http`, which we want to use in our given experiment.

Use the Docker client to build the image and name it `grepwrap-image`.

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

You should consider pushing the image to a registry like [DockerHub](https://hub.docker.com/) and reference it by its full URL. This ensures reproducibility across hosts. With RED it is also possible to use private Docker registries where authorization is required. For the sake of this guide, we will only reference the image by its local name `grepwrap-image`.


## CWL (faice agent)

The `faice agent` commandline tool implements two agents similar to `ccagent` with the major difference that `faice agent` only works with containers.

The first agent `faice agent cwl` implements a syntax equivalent to `ccagent cwl` and `cwltool`. Compared with the CWL reference implementation `cwltool`, it does not run an application (e.g. `grepwrap`) directly, but instead invokes `ccagent cwl`, which then handles the application execution in the container.

In order to execute the application with local input files, we can use the `job.yml` file created earlier. Only the CWL file needs an additional `dockerPull` entry.

Create a new file and insert the following CWL description with `nano grepwrap-cli-docker.cwl`.

```yaml
cwlVersion: "v1.0"
class: "CommandLineTool"
baseCommand: "grepwrap"
doc: "Search for query terms in text files."

requirements:
  DockerRequirement:
    dockerPull: "grepwrap"

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
```


Please note, that only the `requirements` section is different here. These CWL and job files are fully compatible with the CWL specification.

Use the `faice agent cwl` commandline tool to execute the experiment.

```bash
faice agent cwl --disable-pull ./grepwrap-cli-docker.cwl ./job.yml
```

The `--disable-pull` flag is required, because we are referencing a local container image and not a URI pointing to a registry.

A directory called `outputs` was created, which is temporarily mounted as Docker volume to exchange files between the container and the host. It contains the result files.


## RED (faice agent)

The second agent implementation `faice agent red` works with RED files to execute experiments in containers. Its syntax is equivalent to `ccagent red` and it utilizes the `ccagent red` installation in the container image.

A compatible RED file is very similar to the RED file used `ccagent red`, but requires an additional `container` section.

Create a new file and insert the following RED data with `nano red-docker.yml`.

```yaml
redVersion: "6"
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
        url: "https://raw.githubusercontent.com/curious-containers/vagrant-quickstart/master/in.txt"
  before_context: 1
```

Learn more about the container engine description by showing the corresponding jsonschema with `faice schema show red-engine-container-docker` (also see [RED Container Engines](/docs/red-container-engines)).

Use the `faice agent red` commandline tool to execute the experiment.

```bash
faice agent red --disable-pull ./red-docker.yml
```

A directory called `outputs` was created, which is temporarily mounted as Docker volume to exchange files between the container and the host. It contains the result files.

Again connector descriptions for output files can be included in the RED file. Open the existing file and append the following `outputs` section with `nano red-docker.yml`.

```yaml
outputs:
  out_file:
    class: "File"
    connector:
      command: "red-connector-http"
      access:
        url: "http://172.17.0.1:5000/server-out.txt"
```


Please note, that in this case we are running the experiment in a container. In order to send output files from a container to the `faice file-server` running on the host, we use the standard Docker bridge IP `172.17.0.1`. Use `ifconfig` to check if another IP has been assigned to the Docker bridge on your system.

Before the experiment can be executed, the file server needs to be started.

```bash
# start server as background job
faice file-server &
```


Use the `faice agent red` commandline tool to execute the experiment. Again, use `--outputs` to enable the RED output connectors.

```bash
faice agent red --disable-pull --outputs ./red-docker.yml
```

Use `cat server-out.txt` to check the programs output. Since we used the `--outputs` flag, no `outputs` directory was created for data exchange between the container and the host.

You can stop the file-server as follows.

```bash
# terminate the last background job
kill %%
```
