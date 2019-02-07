---
title: Publications
permalink: /publications
layout: single
toc: true
toc_label: "Table of Contents"
---

## Employing Docker Swarm on OpenStack for Biomedical Analysis (2016)

*Christoph Jansen, Michael Witt, Dagmar Krefting*

[DOI: 10.1007/978-3-319-42108-7_23](https://doi.org/10.1007/978-3-319-42108-7_23)

This paper describes a data processing platform based on XNAT as a data management system and a cluster of virtual machines managed by OpenStack. It goes into detail about the difficulties in scheduling biomedical experiments in the cluster using an early version of Docker and Docker Swarm. As a result of this research we developed the first version of Curious Containers with [CC-Server](https://github.com/curious-containers/cc-server) and [CC-Container-Worker](https://github.com/curious-containers/cc-container-worker) to schedule containerized experiments. At some point CC-Server dropped support for Docker Swarm, in favor of directly managing remote Docker machines for more control and better failure mangement. Both software components are now deprecated in favor of CC-Agency and CC-Core respectively.

```bibtex
@InProceedings{Jansen2016,
    author = {Jansen, Christoph and Witt, Michael and Krefting, Dagmar},
    editor = {Gervasi, Osvaldo and Murgante, Beniamino and Misra, Sanjay and Rocha, Ana Maria A.C. and Torre Carmelo M. and Taniar, David and Apduhan, Bernady O. and Stankova, Elena and Wang, Shangguang},
    title = {Employing Docker Swarm on OpenStack for Biomedical Analysis},
    booktitle = {Computational Science and Its Applications -- ICCSA 2016},
    year = {2016},
    publisher = {Springer International Publishing},
    address = {Cham},
    pages = {303--318},
    abstract = {Biomedical analysis, in particular image and biosignal analysis, often requires several methods applied to the same data. The data is typically of large volume, so data transfer can become a bottleneck in remote analysis. Furthermore, biomedical data may contain patient data, raising data protection issues. We propose a highly virtualized infrastructure, employing Docker Swarm technology as the computing infrastructure. An underlying Openstack based IaaS cloud provides additional security features for a flexible and efficient multi-tenant analysis platform. We introduce the prototype infrastructure along a sample use-case of multiple versions of a machine-learning method applied to feature sets extracted from multidimensional biosignal recordings from Sleep Apnea patients and healthy controls.},
    isbn = {978-3-319-42108-7}
}
```

## Towards Reproducible Research in a Biomedical Collaboration Platform Following the FAIR Guiding Principles (2017)

*Christoph Jansen, Maximilian Beier, Michael Witt, Sonja Frey, Dagmar Krefting*

[DOI: 10.1145/3147234.3148104](https://doi.org/10.1145/3147234.3148104)

In this paper we analyze our existing data processing platform and its individual software components in terms of the reproducibility of data-driven experiments guided by the [FAIR principles]((https://www.force11.org/fairprinciples)). It adresses the problem of reproducing these experiments in the scope of the platform, but also proposes a way to describe experiments in a way that they can be shared more easily and that enables other researchers to execute and derive experiments independently of the platform. The successor of the prototypical FAICE data format proposed in the paper is [RED](red-format.md) (Reproducible Experiment Description). The [FAICE](https://github.com/curious-containers/faice) (Fair Collaboration and Experiments) client software prototype is since deprecated and has been replaced by CC-FAICE.

```bibtex
@InProceedings{Jansen2017,
   author = {Jansen, Christoph and Beier, Maximilian and Witt, Michael and Frey, Sonja and Krefting, Dagmar},
   title = {Towards Reproducible Research in a Biomedical Collaboration Platform Following the FAIR Guiding Principles},
   booktitle = {Companion Proceedings of the10th International Conference on Utility and Cloud Computing},
   series = {UCC '17 Companion},
   year = {2017},
   isbn = {978-1-4503-5195-9},
   location = {Austin, Texas, USA},
   pages = {3--8},
   numpages = {6},
   url = {http://doi.acm.org/10.1145/3147234.3148104},
   doi = {10.1145/3147234.3148104},
   acmid = {3148104},
   publisher = {ACM},
   address = {New York, NY, USA},
   keywords = {cloud computing, docker, medical data, repeatability, reproducibility, xnat},
}
```
