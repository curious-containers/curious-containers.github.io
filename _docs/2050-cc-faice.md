---
title: "CC-FAICE"
permalink: /docs/cc-faice
---


CC-FAICE (Curious Containers - FAIR Collaboration and Experiments) is a CLI tool suite, providing a lot of functionality to users. The main functions are:

* Providing an agent to run experiments in RED format.
* Additional tools to convert, validate and export RED files.

# Installation

Install `python3-pip` as system package. The following instructions work for Fedora 28, but instructions for other Linux distribution should be similar.

```bash
sudo dnf install python3-pip
```

It is recommended to install a specific version of `cc-faice`. This will automatically install the latest compatible version of `cc-core`.

```bash
pip3 install --user --upgrade cc-faice==8.*
```

Run CLI tool.

```bash
faice --version
faice --help
```

If this tool cannot be found, you should modify `PATH` (e.g. append `${HOME}/.local/bin`) or fall back to executing the tools as Python modules.

```bash
python3 -m cc_faice --version
```

# Usage

The `--help` flag shows a list of tools, that can have nested subcommands. Each subcommand has its own help section.

```bash
faice --help
faice exec --help
```
