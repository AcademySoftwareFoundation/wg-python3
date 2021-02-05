# Python 3 Transition Guide

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
that the backport provides functionality from.

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
* `funcsigs; python_version<'3.3'`
* `futures; python_version=='2.7'`
* `mock; python_version<'3.3'`
* `ordereddict; python_version<'2.7'`
* `pathlib2; python_version<'3.6'`:
  The `pathlib` module was new in Python 3.4
  but additional features from the Python 3.6 version are also backported.
  See https://docs.python.org/3.6/library/pathlib.html.
* `typing; python_version<'3.5'`:
  This backports the typing module to older versions of Python.
  Use newer patch versions of Python 3.{5,6,7}.x to get backported features
  in those versions.
  See https://docs.python.org/3.7/library/typing.html.
* `unittest2; python_version<'3.5'`

## Avoid use of futurize stage 2 and its standard library backports

[futurize](https://python-future.org/overview.html#automatic-conversion-to-py2-3-compatible-code)
can be helpful for transitioning standalone projects and scripts to
Python 3, but `--futurize --stage2` adds dependencies on the
`future.standard_library` module that changes the behavior of the Python
standard library.  This makes it unsuitable for use in libraries or plugins
that have to co-exist alongside other python plugins in a host application.

It is preferrable to use `six` and the standard library instead of the
`future`, `past`, and `builtins` modules provided by `python-future`.

## Read the Conservative Python 3 Porting Guide

[The Conservative Python 3 Porting Guide](https://portingguide.readthedocs.io/en/latest/)
(CPPG) is an invaluable resource for porting to Python 3.
This document borrows heavily from the CPPG recommendations.

## Use pylint's py3k mode to catch compatibility issues

Pylint contains a "py3k" mode that enables Python 2 + 3 compatibility checks
and is extremely helpful when transitioning to Python 2 + 3 compatibility.
Use of `pylint --py3k` is strongly encouraged.

`pylint --py3k` helps ensure that compatibility does not regress without
needing to run the code, so it can be especially helpful for maintiaining
compatibility even when the project does not have tests.

## Use the six module for Python 2 + 3 compatible idioms

Familiarize yourself with the [six package](https://pypi.org/project/six/).
`six` contains helpful functions and symbols that allow you to write
compatible code that behaves consistently across Python versions.

Using `six` allows us to grep for `six.` and common symbols such as `PY2` in
the future when we're ready to drop support for Python 2.

As noted above, `six.PY2` should be used to special-case Python 2 code when
using `six` so that Python 3 (and newer) is on the forward-looking code path.

    if six.PY2:
        # Python 2 code to be removed in the future.
    else:
        # Python 3, 4, etc.

* Consider using `six.text_type` and `six.binary_type` to refer to
  `unicode` (python3 `str`) and `str` (python3 `bytes`) across versions.

* The builtin `long` type no longer exists in Python 3.
  Use `isinstance(value, six.integer_types)` to detect integers instead of the
  traditional `isinstance(value, (int, long))` Python 2 idiom.

* Use `six.string_types` to detect string types, `unicode`+`str` on Python 2
  and `str`-only on Python 3, using `isinstance(value, six.string_types)`.

* Use the `six.itervalues(dict)` and `six.iteritems(dict)` dict iterator
  functions to deal with the removal of `dict.iteritems()` and related
  methods.  If performance is not a concern then the cost of constructing a
  list using `dict.items()` on Python 2 may be an alternative to
  `six.iteritems(dict)` when updating code that uses `dict.iteritems()`.

*Note that DCCs might ship with a bundled version of `six`. The version
that they ship with might not be up to date, which means some functions might
not be available. Here is a list of the known DCCs that ship with `six` as of January 2021:*

| DCC     | Version             | six version                          | Notes                                        |
|---------|---------------------|--------------------------------------|----------------------------------------------|
| Maya    | 2020                | 1.10.0                               | Not distributed in earlier versions of Maya. |
| Nuke    | 12.\*               | 1.10.0                               | Not distributed in earlier versions of Nuke. |
| Houdini | 15.\*, 16.\*, 17.\* | 1.9.0                                |                                              |
|         | 18.\*               | 1.13.0                               |                                              |

## Consider use of python-modernize

The CPPG recommends the use of
[python-modernize](https://portingguide.readthedocs.io/en/latest/tools.html#automated-fixer-python-modernize)
for making code 2 + 3 compatible.  `python-modernize`, like `futurize`, is
an automated Python code fixer built on top of the `2to3` library.

Unlike `futurize`, `modernize` does not rely on monkey-patching of the
standard library and is safer to use in more circumstances.

`modernize` contains fixers that levarage the `six` module to achieve 2 + 3
compatibility and is most in tune with the recommendations set forth in this
document.

> :warning: **When modernizing code that uses maya.cmds:** At the time of
> writing, modernize will try to convert keyword arguments named "long"
> (eg. `maya.cmds.ls(long=False)`) to "int" (eg. `maya.cmds.ls(int=False)`).
> Although the underlying issue has been fixed,
> there has not been a release of the library with the fix included.
> Modernize 0.8.0 is required to use the library that contains the fix,
> and the fix itself is in the fissix library:
> https://github.com/jreese/fissix/commit/c1581648bc7a2bf35f08da99bfced69a5bc15920

## Be Mindful of Unicode Literals

One of the last steps in the porting process should be to enable
`from __future__ import unicode_literals` so that all string literals
in the file are treated as unicode text strings.

Enabling `from __future__ import unicode_literals` tends to
have the most dramatic effect on the behavior of Python 2 code.  Consider the
following example:

        def example():
            return 'hello world'

This example will return a different type when unicode literals are enabled,
and thus can have impact on externals callers of this function.

Now consider this example with a non-ASCII character:

        # -*- coding: utf-8 -*-
        def example():
            raise Exception('héllo world')

In Python 2, with unicode literals enabled, it will fail to convert the message
to str and will fail with  `<exception str() failed>`.

Consider enabling unicode literals separately as it tends to have a
larger impact and may be worth addressing after the rest of the simpler
future imports and fixes have been addressed.

## Additional Links

* The futurize project contains a
[Cheat Sheet](https://python-future.org/compatible_idioms.html)
for writing Python 2 + 3 compatible code with a list of compatible idioms.
Please note that some of the examples leverage the `future`, `builtins` and
`past` modules, and thus may not be appropriate for libraries or plugins.
