# Versions

All Curious Containers Python package versions consists of three distict numbers separated by `.` (e.g. 3.2.1).

The scheme is X.Y.Z:

* X: RED version
* Y: CC version
* Z: PACKAGE version

## RED Version

If you are working with a RED file in RED version 3, you must use Curious Containers packages where X is 3 (e.g. 3.2.1).

* `cc-core >= 3, < 4`

## CC Version

If you are using `cc-core == 3.2.1`, you must use other Curious Containers packages where X is 3 and Y is 2 (e.g. 3.2.2).

* `cc-faice >= 3.2, < 3.3`
* `cc-agency >= 3.2, < 3.3`

Use `pip3 install --user --upgrade cc-faice`, which will automatically install the latest compatible version of `cc-core` as a dependency of `cc-faice`.

## PACKAGE Version

The PACKAGE version Z is only for maintenance releases of individual packages, which do not break compatibility.
