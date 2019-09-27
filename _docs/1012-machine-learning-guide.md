---
title: "Machine Learning Guide"
permalink: /docs/machine-learning-guide
---

**This guide is currently under construction**

If you follow this guide, you learn how to create and reproduce a comprehensive machine learning experiment, that leverages the full potential of the CC framework.
Before you continue, make sure that you have understood the contents of the [RED Beginner's Guide](/docs/red-beginners-guide).
A basic understanding of machine learning methods is recommended.

This guide contains two experiments.
In the first experiment, a Convolutional Neural Network (CNN) is trained on the [PCAM](https://github.com/basveeling/pcam) dataset to classify tumor tissue in pathological image slides.
The second experiment uses the trained model in an inference task.


## Teaching Goals

The main teaching goals of this guide are:

* how to use a read-only SSHFS directory to mount a large training dataset located on a remote server
* how to use a writable SSHFS directory for live logging of the training process
* how to use batch processing for hyperparameter optimization of machine learning methods
* how to use Nvidia GPUs to accellerate the processing
* how to send experiments to the CC-Agency execution engine


## Prerequisites

Using an **Nvidia GPU** for training acceleration is **recommended** but not required.

The dataset used in this guide is located on the `avocado01.f4.htw-berlin.de` storage server, that is not available to the public.
You can still follow the guide using your **own SSH server**.


### Download Dataset to Storage Server

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


### Create Log Directory

Create an empty `cnn-training/log` directory on in your SSH user's home directory, such that it can be mounted via SSHFS later.

```bash
SSH_USERNAME=christoph
SSH_HOST=avocado01.f4.htw-berlin.de
ssh ${SSH_USERNAME}:${SSH_HOST}
mkdir -p cnn-training/log
```


## Training Experiment



