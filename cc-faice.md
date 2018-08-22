# CC-FAICE

FAICE (Fair Collaboration and Experiments) is a CLI tool suite, providing a lot of functionality to users. The main functions are:

* Implementing meta [agents](#agents) to launch corresponding [CC-Core agents](cc-core.md#agents) inside of Docker containers, which will then run the actual experiments defined in CWL or RED format.
* Additional tools to convert, validate and export RED files.

## Installation

Install `python3-pip` as system package. The following instructions work for Fedora 28, but instructions for other Linux distribution should be similar.

```bash
sudo dnf install python3-pip
```

It is recommended to install a specific version of `cc-faice`. This will automatically install the latest compatible version of `cc-core`.

```bash
pip3 install --user --upgrade cc-faice==5.2.0
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
