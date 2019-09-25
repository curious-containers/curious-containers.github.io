---
title: "Machine Learning Guide"
permalink: /docs/machine-learning-guide
---

**This guide is currently under construction**

If you follow this guide, you learn how to create and reproduce a comprehensive machine learning experiment, that leverages the full potential of the CC framework.
Before you continue, make sure that you have understood the contents of the [RED Beginner's Guide](/docs/red-beginners-guide).

This guide contains two experiments.
In the first experiment, a Convolutional Neural Network (CNN) is trained on the PCAM dataset to classify tumor tissue in pathological image slides.
The second experiment uses the trained model in an inference task.


## Prerequisites

Using an **Nvidia GPU** for training acceleration is **recommended** but not required.

The dataset used in this guide is located on the `avocado01.f4.htw-berlin.de` storage server, that is not available to the public.
You can still follow the guide using your **own SSH server**.


### Download Dataset to Storage Server

You can skip this section, if you have SSH access to `avocado01.f4.htw-berlin.de`.

