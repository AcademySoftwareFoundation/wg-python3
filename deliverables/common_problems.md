# Common Problems and their Solutions

This page collates some problems that were hit when transitioning to Python 3
and their respective solutions.

## Boost in a Python 3 Flavour of CY2019

Unfortunately Python 3.7.x cannot be added to the CY2019 version
of the VFX Reference Platform as-is.
Boost 1.66 fails to find Python correctly when building with Python support.
The easiest fix we (DNEG) found was to upgrade Boost to 1.70,
as this is the version of Boost defined in CY2020.

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

