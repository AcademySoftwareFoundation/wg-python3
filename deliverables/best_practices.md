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
