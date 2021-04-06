# Common Problems and their Solutions

This page collates some problems that were hit when transitioning to Python 3
and their respective solutions.

## Boost in a Python 3 Flavour of CY2019

Unfortunately Python 3.7.x cannot be added to the CY2019 version
of the VFX Reference Platform as-is.
Boost 1.66 fails to find Python correctly when building with Python support.
The easiest fix we (DNEG) found was to upgrade Boost to 1.70,
as this is the version of Boost defined in CY2020.

## Using Boost::Python Shared Libraries in Boost 1.70 via CMake

This version of Boost is the first in the VFX Reference Platform to contain
CMake config files that get used in calls to `find_package(Boost)`
over the FindBoost module built into CMake.
However this introduces a change in behaviour where linking against Boost's
static libraries will be preferred unless
[`BUILD_SHARED_LIBS`](https://cmake.org/cmake/help/latest/variable/BUILD_SHARED_LIBS.html)
is set to `ON`.
Setting `Boost_USE_STATIC_LIBS=OFF` is supposed to be enough to turn this
behaviour off, however this version of Boost also contains a bug
such that static libraries will be used unless `BUILD_SHARED_LIBS` is set to
`ON`: https://lists.boost.org/Archives/boost/2019/06/246338.php

To rectify this problem, either apply the changes mentioned in the Boost mailing
list
(https://github.com/boostorg/boost_install/commit/160c7cb2b2c720e74463865ef0454d4c4cd9ae7c),
skip using the Boost config and use FindBoost instead
(`Boost_NO_BOOST_CMAKE=ON` and `find_package(Boost MODULE ...)`),
or set `BUILD_SHARED_LIBS=ON` in the libraries where it is appropriate.

```cmake
set(Boost_USE_STATIC_LIBS OFF)
# Boost 1.70.0 has a bug that requires BUILD_SHARED_LIBS to be set as well:
# https://lists.boost.org/Archives/boost/2019/06/246338.php
set(BUILD_SHARED_LIBS ON)
```

## Boost Python for CMake build system

As of Boost-1.67, Boost Python components require a Python version suffix,
the CMake file will need to explicitly specify the Python version, for example:

Prior 1.67:
```
find_package(Boost COMPONENTS python REQUIRED)
```

1.67 or newer:
```
find_package(Boost COMPONENTS python2.7 REQUIRED)
find_package(Boost COMPONENTS python3.7 REQUIRED)
```

Pixar's USD has a
[good example](https://github.com/PixarAnimationStudios/USD/blob/master/cmake/defaults/Packages.cmake#L42-L86)
how to find Python then programmatically find Boost Python.

