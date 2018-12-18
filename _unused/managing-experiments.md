# Managing Experiments

Working with data-driven experiments requires you to work on three levels, namely *artifacts*, *description* and *environment*.

First of all you have software *artifacts*, most importantly your own CLI application. In addition, `cc-core` and required RED connectors are artifacts you need to provide. To easily distribute all artifacts, we install them in a RED compatible [container image](container-image.md). The container image is considered a derived artifact, which we can easily distribute across computers using a container image registry.

Using a single fiel in [RED format](red-format.md), we can provide a complete *description* of the experiment. This description refers to the artifacts. We reference the image using its registry URL. We can describe the commandline interface of the application using the CWL syntax and we can refer to RED connectors installed inside the image to access data.

To run an experiment in a compute *environment*, we can use execution engines like `faice agent red` or CC-Agency. We only have to provide them the RED file. Please note, that the [version](version.md) of the `cc-core` package installed in the image must be compatible with the version of CC-FAICE or CC-Agency.

The following ASCII diagram shows all the different parts of an experiment.

```
ARTIFACTS ░  cc-core ────────╮
░░░░░░░░░░░  cc-connectors ──┤
░░░░░░░░░░░  │ application ──┴─> container image
             │ │                 │
             │ │                 │
DESCRIPTION  │ │                 ╰─> container (registry access) ──╮
░░░░░░░░░░░  │ ╰───────────────────> cli (cwl) ────────────────────┤
░░░░░░░░░░░  │                   ╭─> inputs (data access) ─────────┤
░░░░░░░░░░░  ╰───────────────────┴─> outputs (data access) ────────┴─> red.yml ────╮
                                                                                   │
                                                                                   │
ENVIRONMENT                                                            cc-faice/   │
░░░░░░░░░░░                                                            cc-agency ──┴─> exec experiment
```

The following sections introduce CC-FAICE and CC-Agency as tools for running and managing experiments.

| Table of Contents |
| --- |
| [CC-FAICE](#cc-faice) |
| [CC-Agency](#cc-agency) |


## CC-FAICE

TODO


## CC-Agency

TODO
