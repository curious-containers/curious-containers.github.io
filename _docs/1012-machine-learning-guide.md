---
title: "Machine Learning Guide"
permalink: /docs/machine-learning-guide
---


If you follow this guide, you learn how to create and reproduce a comprehensive machine learning experiment, that leverages the full potential of the CC framework.
Before you continue, make sure that you have understood the contents of the [RED Beginner's Guide](/docs/red-beginners-guide).
A basic understanding of machine learning methods is recommended.

This guide contains an experiment, that trains a Convolutional Neural Network (CNN) on the [PCAM](https://github.com/basveeling/pcam) dataset to classify tumor tissue in pathological image slides.


# Teaching Goals

The main teaching goals of this guide are:

* how to use a read-only SSHFS directory to mount a large training dataset located on a remote server
* how to use a writable SSHFS directory for live logging of the training process
* how to use batch processing for hyperparameter optimization of machine learning methods
* how to use Nvidia GPUs to accellerate the processing
* how to send experiments to the CC-Agency execution engine


# Prerequisites

Using an **Nvidia GPU** for training acceleration is **recommended** but not required.

The dataset used in this guide is located on the `avocado01.f4.htw-berlin.de` storage server, that is not available to the public.
You can still follow the guide using your **own SSH server**.


## Download Dataset to Storage Server

You can **skip this section**, if you have SSH access to `avocado01.f4.htw-berlin.de`.

Login to your SSH server, create a `PCAM` folder in your home directory, download the PCAM dataset using `curl` and extract the files using `gunzip`.

```bash
SSH_USERNAME=christoph
SSH_HOST=avocado01.f4.htw-berlin.de
ssh ${SSH_USERNAME}:${SSH_HOST}
mkdir PCAM
curl -fO https://zenodo.org/record/2546921/files/camelyonpatch_level_2_split_train_x.h5.gz
curl -fO https://zenodo.org/record/2546921/files/camelyonpatch_level_2_split_train_y.h5.gz
curl -fO https://zenodo.org/record/2546921/files/camelyonpatch_level_2_split_valid_x.h5.gz
curl -fO https://zenodo.org/record/2546921/files/camelyonpatch_level_2_split_valid_y.h5.gz
curl -fO https://zenodo.org/record/2546921/files/camelyonpatch_level_2_split_test_x.h5.gz
curl -fO https://zenodo.org/record/2546921/files/camelyonpatch_level_2_split_test_y.h5.gz
gunzip *.h5.gz
```

When following the tutorial, you have to **replace** all occurrences of `/data/ldap/histopathologic/original_read_only/PCAM_extracted` with `PCAM` and all occurrences of `avocado01.f4.htw-berlin.de` with your own SSH server.


## Create Log Directory

Create an empty `cnn-training/log` directory on in your SSH user's home directory, such that it can be mounted via SSHFS later.

```bash
SSH_USERNAME=christoph
SSH_HOST=avocado01.f4.htw-berlin.de
ssh ${SSH_USERNAME}:${SSH_HOST}
mkdir -p cnn-training/log
```


# Training Experiment

This part of the guide describes the setup of the experiment.


## Training Code

The following code uses a pre-release of tensorflow 2. A standard CNN architecture `NASNetMobile` is used, to learn the binary classification of tumor tiles.
Store this code in a file using `nano cnn-training.py`, or another editor of your choice, and make the file executable with `chmod u+x cnn-training.py`.

```python
#!/usr/bin/env python3

import os
import sys
import argparse
import random

import h5py
import numpy as np
import tensorflow as tf
from tensorflow.keras.applications.nasnet import NASNetMobile
from tensorflow.keras.optimizers import Adam
from tensorflow.keras.metrics import AUC


# Constants
WEIGHTS_FILE = 'weights.h5'
TRAIN_X_FILE = 'camelyonpatch_level_2_split_train_x.h5'
TRAIN_Y_FILE = 'camelyonpatch_level_2_split_train_y.h5'
VALID_X_FILE = 'camelyonpatch_level_2_split_train_x.h5'
VALID_Y_FILE = 'camelyonpatch_level_2_split_train_y.h5'
INPUT_SHAPE = (96, 96, 3)


# Arguments
parser = argparse.ArgumentParser()
parser.add_argument(
    dest='data_dir', type=str,
    help='Data: Path to read-only directory containing PCAM *.h5 files.'
)
parser.add_argument(
    '--learning-rate', type=float, default=0.0001,
    help='Training: Learning rate. Default: 0.0001'
)
parser.add_argument(
    '--batch-size', type=int, default=64,
    help='Training: Batch size. Default: 64'
)
parser.add_argument(
    '--num-epochs', type=int, default=5,
    help='Training: Number of epochs. Default: 5'
)
parser.add_argument(
    '--steps-per-epoch', type=int, default=None,
    help='Training: Steps per epoch. Default: data_size / batch_size'
)
parser.add_argument(
    '--log-dir', type=str, default=None,
    help='Debug: Path to writable directory for a log file to be created. Default: log to stdout / stderr'
)
parser.add_argument(
    '--log-file-name', type=str, default='training.log',
    help='Debug: Name of the log file, generated when --log-dir is set. Default: training.log'
)
args = parser.parse_args()


# Redirect output streams for logging
if args.log_dir:
    log_file = open(os.path.join(os.path.expanduser(args.log_dir), args.log_file_name), 'w')
    sys.stdout = log_file
    sys.stderr = log_file

print('GPU available:', tf.test.is_gpu_available())


# Model
model = NASNetMobile(weights=None, input_shape=INPUT_SHAPE, classes=2)
model.compile(
    loss='categorical_crossentropy',
    optimizer=Adam(args.learning_rate),
    metrics=['accuracy', AUC()]
)
model.summary()


# Input
def data_generator(x, y, batch_size=None):
    index = range(len(x))
    labels = np.array([[1, 0], [0, 1]])

    while True:
        index_sample = index
        if batch_size is not None:
            index_sample = sorted(random.sample(index, batch_size))

        x_data = x[index_sample] / 256.0
        y_data = y[index_sample]
        y_data = labels[y_data[:, 0, 0, 0]]
        yield x_data, y_data


data_dir = os.path.expanduser(args.data_dir)

train_x = h5py.File(os.path.join(data_dir, TRAIN_X_FILE), 'r', libver='latest', swmr=True)['x']
train_y = h5py.File(os.path.join(data_dir, TRAIN_Y_FILE), 'r', libver='latest', swmr=True)['y']
valid_x = h5py.File(os.path.join(data_dir, VALID_X_FILE), 'r', libver='latest', swmr=True)['x']
valid_y = h5py.File(os.path.join(data_dir, VALID_Y_FILE), 'r', libver='latest', swmr=True)['y']


# Training
data_size = len(train_x)
steps_per_epoch = data_size // args.batch_size

if args.steps_per_epoch:
    steps_per_epoch = args.steps_per_epoch

model.fit_generator(
    data_generator(train_x, train_y, batch_size=args.batch_size),
    steps_per_epoch=steps_per_epoch,
    epochs=args.num_epochs,
    validation_data=data_generator(valid_x, valid_y, batch_size=args.batch_size),
    validation_steps=1
)

# Output
model.save_weights(WEIGHTS_FILE)

if args.log_dir:
    sys.stdout.close()
```

The output of the script is a weights file named `weights.h5`, that is defined as a constant `WEIGHTS_FILE` and will be created in your current working directory.
These weights represent what the model has learned.

The training and validation data is contained in the PCAM directory.
Since this directory will be mounted in the container by CC, the local path inside the container is not known beforehand.
Therefore the path cannot be hard coded and is provided as a mandatory CLI argument at the first argument position.
You could, for example, run this program locally on your computer as `./cnn-training /path/to/PCAM --batch-size 32`.
This requires the PCAM data directory to be available locally as `/path/to/PCAM`.
For the sake of simplicity, the expected file names inside the directory are hard coded.

Additional training parameters, like `--steps-per-epoch` and `--learning-rate` are optional arguments.
Of course, in a real world application, many more parameters should be exposed via the CLI.

A special feature is the optional `--log-dir` argument.
If a path to an existing directory is specified, the stdout and stderr streams are redirected into a file that is created in this log directory.
The standard name for this file is `training.log`.
If multiple trainings are executed in parallel, this name can be changed to avoid conflicts using `--log-file-name`.
We will use this feature to mount a writable SSHFS network filesystem for logging, such that the training progress can be viewed live outside of the container.


## CWL

Now create a CLI description of the `cnn-training.py` program as `nano cnn-training.cwl.yml`.

```yaml
cwlVersion: "v1.0"
class: "CommandLineTool"
baseCommand: "cnn-training.py"
doc: "Train a CNN on PCAM data in HDF5 format."

inputs:
  data_dir:
    type: "Directory"
    inputBinding:
      position: 0
    doc: "Data: Path to read-only directory containing PCAM *.h5 files."
  learning_rate:
    type: "float?"
    inputBinding:
      prefix: "--learning-rate"
    doc: "Training: Learning rate. Default: 0.0001"
  batch_size:
    type: "int?"
    inputBinding:
      prefix: "--batch-size"
    doc: "Training: Batch size. Default: 64"
  num_epochs:
    type: "int?"
    inputBinding:
      prefix: "--num-epochs"
    doc: "Training: Number of epochs. Default: 5"
  steps_per_epoch:
    type: "int?"
    inputBinding:
      prefix: "--steps-per-epoch"
    doc: "Training: Steps per epoch. Default: data_size / batch_size"
  log_dir:
    type: "Directory?"
    inputBinding:
      prefix: "--log-dir"
    doc: "Debug: Path to writable directory for a log file to be created. Default: log to stdout / stderr"
  log_file_name:
    type: "string?"
    inputBinding:
      prefix: "--log-file-name"
    doc: "Debug: Name of the log file, generated when --log-dir is set. Default: training.log"

outputs:
  weights_file:
    type: "File"
    outputBinding:
      glob: "weights.h5"
    doc: "CNN model weights in HDF5 format."
```

Please note, that `data_dir` and `log_dir` are defined as directories.
This allows us to use RED connectors to download or mount these directories.
The `?` in type descriptions like `int?` denote optional arguments.
The output is defined as a `File` named `weights.h5`.
Since the file is mandatory, CC will report an error if the training code does not create it.


## Docker Image

Create the following Dockerfile using `nano Dockerfile`.

```docker
FROM docker.io/nvidia/cuda:10.0-cudnn7-runtime-ubuntu18.04

RUN apt-get update \
&& apt-get install -y python3-venv python3-pip sshfs \
&& useradd -ms /bin/bash cc

# switch user
USER cc

ENV PATH /home/cc/.local/bin:${PATH}

RUN mkdir -p /home/cc/.local/bin

# install connectors
RUN python3 -m venv /home/cc/.local/red \
&& . /home/cc/.local/red/bin/activate \
&& pip install wheel \
&& pip install red-connector-ssh==1.0 \
&& ln -s /home/cc/.local/red/bin/red-connector-* /home/cc/.local/bin

# install app
RUN pip3 install --user numpy h5py tensorflow-gpu==2.0.0b1

ADD --chown=cc:cc cnn-training.py /home/cc/.local/bin/cnn-training.py
```

This image installs the `red-connector-ssh`.
We plan to use the directory mounting feature of this connector.
Therefore `sshfs` must be installed as well.

The Python library dependencies for `cnn-training.py` are installed in the `cc` user's home directory via `pip3`.

You can build a new image and push it to the DockerHub registry using the following commands.

```bash
IMAGE=docker.io/curiouscontainers/cnn-training
docker build --tag ${IMAGE} .
docker run --rm -u 1000:1000 ${IMAGE} cnn-training.py --help
docker run --rm -u 1000:1000 ${IMAGE} red-connector-ssh --version
docker run --rm -u 1000:1000 ${IMAGE} sshfs --version
docker push ${IMAGE}
```

Of course, you won't be allowed to push the image to the `curiouscontainers` organization, but you can choose another image name for your own organization.
To follow this guide, you do not have to publish the image yourself, because it is already available under the given image URL.


## RED

The `batches` keyword in a RED file can be used to provide `inputs` / `outputs` pairs as a list.
This batch processing feature can be used to run multiple containers in different configurations.
Since the `inputs` / `outputs` configurations are all very similar, we will use a Python script to generate a RED file containing two batches.
This makes the process of creating a large RED file much easier.

Create the python script using `nano cnn-training.red.py` and make it executable `chmod u+x cnn-training.red.py`.

{% raw %}
```python
#!/usr/bin/env python3

import json

SSH_SERVER = 'avocado01.f4.htw-berlin.de'
SSH_AUTH = {'username': '{{ssh_username}}', 'password': '{{ssh_password}}'}
DATA_DIR = '/data/ldap/histopathologic/original_read_only/PCAM_extracted'
AGENCY_URL = 'https://agency.f4.htw-berlin.de/cc'
LEARNING_RATES = [0.0001, 0.0005]
STEPS_PER_EPOCH = 10


batches = []

for i, learning_rate in enumerate(LEARNING_RATES):
    batch = {
        'inputs': {
            'data_dir': {
                'class': 'Directory',
                'connector': {
                    'command': 'red-connector-ssh',
                    'mount': True,
                    'access': {
                        'host': SSH_SERVER,
                        'auth': SSH_AUTH,
                        'dirPath': DATA_DIR
                    }
                }
            },
            'learning_rate': learning_rate,
            'steps_per_epoch': STEPS_PER_EPOCH,
            'log_dir': {
                'class': 'Directory',
                'connector': {
                    'command': 'red-connector-ssh',
                    'mount': True,
                    'access': {
                        'host': SSH_SERVER,
                        'auth': SSH_AUTH,
                        'dirPath': 'cnn-training/log',
                        'writable': True
                    }
                }
            },
            'log_file_name': 'training_{}.log'.format(i)
        },
        'outputs': {
            'weights_file': {
                'class': 'File',
                'connector': {
                    'command': 'red-connector-ssh',
                    'access': {
                        'host': SSH_SERVER,
                        'auth': SSH_AUTH,
                        'filePath': 'weights_{}.h5'.format(i),
                    }
                }
            }
        }
    }
    batches.append(batch)

with open('cnn-training.cwl.json') as f:
    cli = json.load(f)

red = {
    'redVersion': '8',
    'cli': cli,
    'batches': batches,
    'container': {
        'engine': 'docker',
        'settings': {
            'image': {
                'url': 'docker.io/curiouscontainers/cnn',
            },
            'ram': 32000,
            'gpus': {
                'vendor': 'nvidia',
                'count': 1
            }
        }
    },
    'execution': {
        'engine': 'ccagency',
        'settings': {
            'access': {
              'url': AGENCY_URL,
              'auth': {
                  'username': '{{agency_username}}',
                  'password': '{{agency_password}}'
              }
            }
        }
    }
}

with open('cnn-training.red.json', 'w') as f:
    json.dump(red, f, indent=4)
```
{% endraw %}

This script uses the `json` module from the Python standard library to load a CWL file named `cnn-training.cwl.json`, because Python does not provide a built-in YAML module.
You may have noticed, that we created a CWL in YAML format earlier and we have to convert the file for further processing using the `faice convert format` tool.

```bash
faice convert format --format=json cnn-training.cwl.yml > cnn-training.cwl.json
```

You can read the `cnn-training.red.py` script to see, that the CWL information gets embedded in the red file under die `cli` keyword.
Furthermore the script loops over two learning rates, to create two distinct `inputs` / `outputs` pairs.
Both, `data_dir` and `log_dir` will use SSHFS, because `mount: True` is specified in the `connector` section, but only `log_dir` has the `writable: True` flag set.
Remember to change the `SSH_SERVER` constant, if you are not using `avocado01.f4.htw-berlin.de`.

Under the `execution` keyword CC-Agency is specified as engine.
The URL is set to `https://agency.f4.htw-berlin.de/cc`.
If you do not have an account for this instance of CC-Agency, you can change the URL to point to a self-hosted agency.

As an alternative, you can switch to CC-FAICE and execute the experiment locally using the following code snippet.


```python
red = {
    'execution': {
        'engine': 'ccfaice',
        'settings': {}
    }
}
```

If you want to use an Nvidia GPU in your system you must have the [docker-ce](https://docs.docker.com/install/) version of Docker and [Nvidia Container Toolkit](https://github.com/NVIDIA/nvidia-docker) installed, which is only possible on Linux.
If you do not have access to GPUs or your system does not fulfill the aforementioned requirements remove the `gpus` section from the RED file to run the program on a CPU. Using the CPU may slow down the processing.

You can now run `cnn-training.red.py` to create the `cnn-training.red.json` file.

```bash
./cnn-training.red.py
```

If you are using `engine: 'ccagency'`, use `faice exec` to submit the batches to CC-Agency.

```bash
faice exec cnn-training.red.json
```

If your are using `engine: 'ccfaice'` instead, you have to add the `--insecure` flag to set `SYS_ADMIN` capabilities in Docker.
These capabilities are required to use FUSE file systems like SSHFS in a Docker container.

```bash
faice exec --insecure cnn-training.red.json
```

You can watch the progress using the live log files.

```bash
SSH_USERNAME=christoph
SSH_HOST=avocado01.f4.htw-berlin.de

ssh ${SSH_USERNAME}@${SSH_HOST}
tail -f cnn-training/log/training_0.log
# tail -f cnn-training/log/training_1.log
```

In addition, you can check the status of your experiment's batches using the [CC-Agency API](/docs/cc-agency-api).
For example, the following Python snippet uses the external packages `requests` and `keyring` to get information about the last two registered batches.
The keyring part only works if you have used `faice exec` or the `keyring` CLI tool to store the variable values.


```python
#!/usr/bin/env python3

from pprint import pprint

import requests
import keyring


url = 'https://agency.f4.htw-berlin.de/cc'
auth = (
    keyring.get_password('red', 'agency_username'),
    keyring.get_password('red', 'agency_password')
)

r = requests.get(
    f'{url}/batches?limit=2',
    auth=auth
)
r.raise_for_status()
batches = r.json()
pprint(batches)
```