---
title: "Versioning"
permalink: /docs/versioning
---

All CC Python package versions consist of three distict numbers separated by `.` (e.g. `"3.2.1"`).
The first number refers to the supported RED version.


# Users

For users, only the **RED** version is relevant.
For example, if `redVersion: "8"` is set in a RED file, install `cc-faice` 8.X.X as follows.

```bash
pip3 install --user --upgrade cc-faice==8.*
```

If you want to send an experiment to CC-Agency it must match the **RED** version.
A way to retrieve the version number of CC-Agency can be found in the [API documentation](/docs/cc-agency-api#get-version).
