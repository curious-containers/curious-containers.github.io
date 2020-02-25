---
title: "Release"
permalink: /docs/release
toc: false
---

The documentation refers to release version `9`.

If you need an older version of this documentation go to the [curious-containers.github.io](https://github.com/curious-containers/curious-containers.github.io) software repository.

# Versioning

All CC Python package versions consist of three distict numbers separated by `.` (e.g. `"3.2.1"`).
The first number refers to the supported RED version.

For **users** only this RED version is relevant.
For example, if `redVersion: "9"` is set in a RED file, install `cc-faice` 9.x.y as follows.

```bash
pip3 install --user --upgrade cc-faice==9.*
```

If you want to send an experiment to CC-Agency it must also match the RED version.
A way to retrieve the version number of CC-Agency can be found in the [API documentation](/docs/cc-agency-api#get-version).
