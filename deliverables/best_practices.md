# Best Practices

This document collates some best practices and advice to consider
when transitioning to Python 3.

## Checking for the Python Version

When code needs to decide whether to do something based on the version of Python,
it is best to set a constant that denotes the version of Python
rather than doing comparisons against ``sys.version_info`` all over code
so that this transition code is easier to find and remove later.

Bad:

```python
if sys.version_info[0] == 2:
    # Do something Python 2 specific
    ...
```

Good:

```python
PY2 = sys.version_info[0] == 2

...

if PY2:
    # Do something Python 2 specific
```

The name of these constants match the name used in ``six`` as well
so a single search through code will find both custom checks and checks using ``six``.

### Python 4

Furthermore, think about what your check will do if and when Python 4 comes around.
Opt to check for Python 2 rather than checking for Python 3.

Bad:

```python
if PY3:
    # Do something Python 3 specific
    ...
else:
    # This will run in Python 2 and 4 :(
    ...
```

Good:

```python
if PY2:
    # Do something Python 2 specific
    ...
else:
    # This will run in Python 3 and 4
    ...
```

## Avoid using backport modules in Python 3

Backport modules provided a way for parts of the future standard library
to be accessible in an older version of Python.
It is preferable to use the standard library versions where possible,
because they will be supported for longer.
Some backport modules will refuse to install in the newer versions of Python,
but some end up breaking in other ways at run time.

We can avoid installation of these modules in the newer versions of Python
by using [PEP-508](https://www.python.org/dev/peps/pep-0508/) environment markers
to not install the backport modules when in a version of Python
for the backport provides functionality from.

Code will also need to guard against usage of the backport module
to maintain compatibility with Python 2.
For example:

```python
import sys

GE_PY3_3 = sys.version_info[:2] >= (3, 3)
if GE_PY3_3:
    from functools import lru_cache
else:
    from backports.functools_lru_cache import lru_cache
```

The following lists known backport packages
and the version of Python that the backport provides functionality for
in the syntax that can be used in a setup.py file.
Some backports provide functionality from a range of Python versions.
Always choose the version of Python that is correct for your usage.

* `backports_abc; python_version<'3.3'`
* `backports.functools_lru_cache; python_version<'3.3'`
* `backports.shutil_which; python_version<'3.3'`
* `backports.ssl_match_hostname; python_version<'3.5'`
* `backports.tempfile; python_version<'3.5'`:
  The provided `TemporaryDirectory` class is available in Python 3.4,
  but the Python 3.5 `mkdtemp` signature is also backported.
  See https://docs.python.org/3.5/library/tempfile.html.
* `backports.weakref; python_version<'3.6'`:
  `finalize` was new in Python 3.4,
  however some additional features are backported from
  the Python 3.6 implementation.
  See https://docs.python.org/3.6/library/weakref.html#weakref.finalize.
* `configparser; python_version<'3.5'`:
  Many of the new features come from Python 3.2,
  however additional features are backported from Python 3.6.
  See https://docs.python.org/3.6/library/configparser.html.
* `contextlib2; python_version<'3.6'`:
  `nullcontext` from Python 3.7 is also included.
* `enum34; python_version<'3.4'`
* `futures; python_version=='2.7'`
* `ordereddict; python_version<'2.7'`
* `pathlib2; python_version<'3.6'`:
  The `pathlib` module was new in Python 3.4
  but additional features from the Python 3.6 version are also backported.
  See https://docs.python.org/3.6/library/pathlib.html.
* `typing; python_version<'3.8'`:
  The `typing` module was new in Python 3.5
  but additional features from the Python 3.8 version are also backported.
  See https://docs.python.org/3.8/library/typing.html.
