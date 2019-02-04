---
title: "Versioning"
permalink: /docs/versioning
---

All CC Python package versions consist of three distict numbers separated by `.` (e.g. `"3.2.1"`) with a `"${RED}.${CC}.${PACKAGE}"` scheme.


## RED Version

If you are working with a RED file in a specific version (e.g. `"3"`), you must install CC-Core with a matching RED version (`"3.X.X"`) in your container image.

```
pip3 install --user cc-core>=3,<4
```


## CC Version

If you are using CC-Core with a specific RED, CC and PACKAGE version (e.g. `"3.2.1"`) in your container image, you must use CC-FAICE or CC-Agency with matching RED and CC versions (`"3.2.X"`).

```
pip3 install --user cc-faice>=3.2,<3.3
pip3 install --user cc-agency>=3.2,<3.3
```


## PACKAGE Version

The PACKAGE version is only for maintenance releases of individual packages, which do not break compatibility.