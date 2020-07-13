# Common Problems and their Solutions

This page collates some problems that were hit when transitioning to Python 3
and their respective solutions.

## Boost in a Python 3 Flavour of CY2019

Unfortunately Python 3.7.x cannot be added to the CY2019 version
of the VFX Reference Platform as-is.
Boost 1.66 fails to find Python correctly when building with Python support.
The easiest fix we (DNEG) found was to upgrade Boost to 1.70,
as this is the version of Boost defined in CY2020.
