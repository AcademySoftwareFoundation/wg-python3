Python 2+3 Coding Guidelines
============================

When writing Python code, please follow these coding conventions to help
maintain code clarity, consistency, and compatibility with Python 2+3.

.. contents:: Table of Contents
   :backlinks: entry
   :local:


Python File Conventions
-----------------------

Naming of Files
^^^^^^^^^^^^^^^

Python packages and modules are part of a shared global top-level namespace.
Thus, it is important to use a unique and descriptive top-level package namespace.

For example, "gui", "core", and similar generic names are discouraged as a top-level name.

Never name packages the same as a builtin Python module.

Prefer all-lowercase names for package and module names.  Avoid underscores and try to
follow the same style as the Python standard library itself, which generally strictly
follows the `PEP-8 naming guide <https://www.python.org/dev/peps/pep-0008/#naming-conventions>`_.

Modules vs. Packages
~~~~~~~~~~~~~~~~~~~~

When in doubt, create a top-level package namespace.  If you are going to have more than
one Python source file in your library then you should create a top-level package for
the modules.  A package is a directory with an empty ``__init__.py``, as compared to a
module which is exposed as a single ``.py`` file. [#f1]_

Having code in ``__init__.py`` files is strongly discouraged.  There are specific cases
where it is okay (for example, when exposing a wrapped C++ library) but generally we
should avoid side-effects from importing modules, and generally from keeping code in
``__init__.py`` files.

When possible, do not execute code in packages or modules.  Define functions only.
Do not run code on import unless absolutely necessary.

Sometime it is useful to write a module which can also be executed as a script
for testing or other purposes.  When doing so place the executable "script" parts
of the code under a ``__main__`` block:

.. code-block:: python

       if __name__ == '__main__':
           sys.exit(main())

.. rubric:: Footnotes

.. [#f1] Python 3.3 introduced `implicit namespace packages
   <https://www.python.org/dev/peps/pep-0420/>`_ which eliminated the
   requirement for empty ``__init__.py`` files.  This feature cannot be used
   in Python 2+3 compatible projects and is thus discouraged outside of
   standalone Python 3-only projects with no external dependents.

Placement of Files in Projects
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you are a pure-Python project then it is preferrable to keep the python package at the
top-level of the project.  The source tree layout should exactly match the installation
layout.  This allows running the code directly without needing to go through an installation step.

.. code-block:: none

   source-root/
       +---- bin/
       +---- project/
           +---- __init__.py
           +---- module.py
       +---- share/
           +---- project/
               +---- resources/
       +---- [other files]


* ``bin``: standalone executable scripts

If your project has a mix of Python and C++ where Python is being wrapped from C++ libraries
then it might make more sense to place the Python code adjacent to the C++ library code.

.. code-block:: none

   source-root/
       +---- bin/
       +---- src/
           +---- project/          # C++ library
               +---- python/       # Python wrapping
       +---- tests/
       +---- [other files]

Shebang lines
~~~~~~~~~~~~~

Start executable python scripts using ``#!/usr/bin/env python``.
This ensures that the Python executable in the environment will be used.

Use "python3" if your code is Python 3-only, otherwise assume that you are writing
Python 2+3 compatible code.

Python Code Naming Conventions
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Prefixes and Capitalization
~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1

   * - Type
     - Public
     - Private / Protected
   * - Function
     - ``function_name(...)``
     - ``_function_name(...)``
   * - Class
     - ``ClassName``
     - ``_ClassName``
   * - Variable
     - ``variable_name``
     - ``_variable_name``
   * - Constant
     - ``CONSTANT``
     - ``_CONSTANT``


These are conventions that should be taken into account when writing Python code.  While developers
should aim to keep these conventions, there will be exceptions.  When encountering exceptions, be
sure to add comments to document why an exception was made.

Source Formatting
-----------------

Indent using four spaces only.  Avoid tabs.

Python Implementation conventions
---------------------------------

Imports
^^^^^^^

Always use package namespaces in code.  Prefer importing the package or module name
rather than importing pieces (e.g. classes, functions) from within the module.
Each project should keep its own project namespace.


* Avoid ``from ... import *``

* There are times when ``import realname as alias`` is needed to avoid naming conflicts.

* Import standard library modules first, followed by external packages, and
  project imports last.

* Prefer keeping each section alphabetized to minimize merge conflicts.

* Use only a single package import per line.

* Avoid imports at function scope unless there is a strong reason to do so.
  Add comments when breaking this rule.

* Enable absolute imports, and always use absolute imports when importing
  external modules.  Always use relative imports within your package.

.. code-block:: python

       #### package/example.py
       from __future__ import absolute_import
       import sys

       from . import util


       def example():
           return sys.exit(util.function())


       #### package/util.py
       def function():
           return 42


Relative imports are preferred inside library packages and modules because it makes
the code relocatable and minimizes hard-coding the package name everywhere.
If the package is renamed, none of the module code will need to change.

Exceptions
^^^^^^^^^^

Avoid blind try/catch statements.  Specify which exceptions are expected to be caught.

.. code-block:: python

       # Bad: overly-broad exception
       try:
           foo = bar['baz']
       except:
           pass

       # Good: catches only the excepted KeyError
       try:
           foo = bar['baz']
       except KeyError:
           pass


Do not use exceptions for control-flow.  Exceptions should be used for
exceptional circumstances where the called code has no way to recover and
nothing useful to return.

Prefer `built-in exception types <https://docs.python.org/3/library/exceptions.html>`_
when it makes sense.  For example, if a function gets passed an invalid value
then it makes sense to raise ``ValueError``.

Derive from ``Exception`` when defining custom exceptions, and prefer using
a common base class for families of custom exceptions.

Multiple Inheritance
^^^^^^^^^^^^^^^^^^^^

Avoid multiple inheritance whenever possible.  Mixins are sometimes useful,
but also complicated and can be implemented using other approaches.

Strings
^^^^^^^

For Python 2+3 compatibility it is encouraged to enable Python 3-style unicode
string literals.  NOTE: this can cause problems with code that is not
unicode-aware, or when calling into wrapped C++ libraries, but should
generally be preferred for new code.

.. code-block:: python

       # Needed for Python 2
       from __future__ import unicode_literals


Features to Avoid
^^^^^^^^^^^^^^^^^


* ``os.system()``: use ``subprocess`` whenever possible.  ``os.system(...)`` goes
  through a shell, and thus you need to worry about shell quoting,
  shell meta-characters, whitespace, and other details.

* ``subprocess.Popen(..., shell=True)``: should be avoided for the same reasons
  as above.  It is better to pass the command as a list so that arbitrary
  arguments are robustly handled.

* ``getopt``: use ``argparse`` when parsing command-line flags.

* ``import user``: should not appear in programs.  This module is not intended
  to be used by programs or libraries.

* ``sys.path``: avoid modifying ``sys.path`` whenever possible.

* Avoid disabling pylint, and if you have to, add comments explaining why.

* Avoid overuse of environment variables.  For example, prefer using
  ``pwd.getpwuid(os.getuid()).pw_name`` to get the current username over the
  ``$USER`` environment variable.  This is preferred because ``$USER`` is not
  defined in certain contexts (for example, when running under cron).

  Environment variables are a great way to allow for testing, overriding defaults,
  and enabling specific behavior, though a configuration file should be preferred
  when possible.


Python 3 Compatibility
----------------------

Python3 will be coming in the future.  Prefer writing Python 2+3 compatible
code whenever possible by avoiding features that no longer exist in Python3.

Future Imports
^^^^^^^^^^^^^^

In general, try to write Python 2+3 compatible code.  This means that all Python source files
should generally begin with this line:

.. code-block:: python

    from __future__ import absolute_import, division, print_function, unicode_literals

This enables the following Python3 behavior when in Python 2:


* ``absolute_import``: Imports become absolute by default, which allows for
  safe use of relative imports within packages.

* ``division``: Division will return floats for fractional results even when
  both operands are integers.  Truncating integer division can be performed using
  the ``//`` double-slash division operator.

  .. code-block:: python

       2 / 3   # returns 0.666... (float)
       2 // 3  # returns 0  (int)

* ``print_function``: Disable the use of ``print`` statements.
  ``print`` can only be called as a function.

* ``unicode_literals``: String literals will be unicode strings instead of byte
  strings.


Unicode Strings
^^^^^^^^^^^^^^^

Unicode literals are preferred because unicode-aware applications and
libraries should use unicode text exclusively in their internals, and only
expose bytes when interfacing with files, IO, etc.  This is a Python
implementation pattern that provides a practical approach for correctly
handling non-ascii unicode data in an application.

See `unicode sandwich <https://nedbatchelder.com/text/unipain/unipain.html#35>`_
for more details.


Python 3 Porting Recommendations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

On the use of ``futurize``
~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``futurize`` tool is helpful for standalone projects, but it adds
dependencies on a ``future.standard_library`` module that changes the behavior
of the stdlib.  We do not recommend the use of ``futurize --stage2``,
especially for library code, though the futurize "stage1" fixes are generally
safe and recommended.

For maximally-compatible Python 2+3 code, the following futurize invocation
is recommended.

.. sourcecode:: sh

    futurize -1 -a -u -p -w <filename.py>
    # AKA
    futurize --stage1 --all-imports --unicode-literals --print-function --write


On the use of the ``six`` module
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Use of the `six module <https://pypi.org/project/six/>`_ module is recommended
when you need to write code that special-cases Python 2 or Python3 behavior.
In addition to functionality, we'll be able to grep for ``six.`` in the future
when we're ready to drop compatibility with Python 2.

Prefer using ``six.PY2`` to special-case Python 2 code so that Python3 code is
considered the forward-looking code path.

.. sourcecode:: python

    if six.PY2:
        # Python 2 behavior.
    else:
        # Python 3, 4, etc.

The ``six`` module provides various useful symbols, eg. ``six.text_type``
and ``six.binary_type`` that can be used to refer to the ``unicode/str``
and ``bytes`` types in a portable manner.

In Python 3, the ``long`` type went away.  In Python 2 it was common to detect
integer types using ``isinstance(value, (int, long))`` but this will no longer
work in Python3.  ``six`` allows you to check for
``isinstance(value, six.integer_types)`` and it supply ``(int, long)`` on
Python 2 and ``int`` only on Python 3.

``six`` contains many helpful functions and utilities that allow you to write
compatible code that works across versions.  Take time to familiarize yourself
with ``six``, particularly the dict iterator methods such as ``six.itervalues``,
``six.iteritems``, and friends.

Use the ``pylint --py3k`` compatibility checker
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Run ``pylint --py3k`` over your project to get get a quick birds-eye view of
all of the potential compatibility problems in your Python code.  Use pylint
in py3k mode to detect issues so that they can be eliminated.

You may need to silence pylint in a few edge cases, but most code should be
able to be made ``pylint --py3k`` compatible.


Use the print function
~~~~~~~~~~~~~~~~~~~~~~

`print <https://python-future.org/compatible_idioms.html#print>`_
is now a function instead of a statement.
For most cases, using ``print(foo)`` instead of ``print foo`` is sufficient.
`PEP-3105 <https://www.python.org/dev/peps/pep-3105/>`_.

.. sourcecode:: python

    # If you have old code that uses a print statement like this:
    print >> sys.stderr 'abc', 'xyz',  # Omits a trailng newline
    print 'aaa', 'bbb'                 # Prints "aaa bbbb"

    # Change it to use a print function like this:
    from __future__ import print_function
    print('abc', 'xyz', file=sys.stderr, end='')
    print('aaa', 'bbb')

This is automatically fixed by futurize stage1.


Use relative imports for intra-package imports
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`Implicit relative imports
<https://python-future.org/compatible_idioms.html#imports-relative-to-a-package>`_
are no longer valid in Python 3 as they are ambiguous
and possibly confusing. If you need to import something from the same package
you will need to update it to use ``.`` or ``..``.
`PEP-328 <https://www.python.org/dev/peps/pep-0328/#rationale-for-absolute-imports>`_.

.. sourcecode:: python

    # For example, in mypkg/mymodule.py we previously could import
    # mypkg/relativemoudle.py as follows.  # This will no longer work in Python 3.
    import relativemodule

    # The old relative syntax is ambiguous and best avoided.
    # In Python 3 these become absolute imports.  Use relative imports for
    # all internal modules
    from __future__ import absolute_import
    from . import relativemodule


Use new-style divison
~~~~~~~~~~~~~~~~~~~~~

The `divison <https://python-future.org/compatible_idioms.html#division>`_
operator (``/``) changed in Python 3.  In Python 2, expressions such
as ``1 / 3`` would perform truncating integer division and returns 0.
In Python 3, this performs floating-point divison and returns 0.333.
`PEP-238 <https://www.python.org/dev/peps/pep-0238/>`_.

Update code to use explcit integer division where needed.

.. sourcecode:: python

    # This snippet of old code should be updated to perform integer division
    # eplicitly.
    result = some_value / 2

    # New-style Python code uses the `//` division for integer division.
    result = some_value // 2


Use unicode literals
~~~~~~~~~~~~~~~~~~~~

Be mindful of `Python string types and literals
<https://python-future.org/compatible_idioms.html#strings-and-bytes>`_.

All strings are unicode literals in python 3. String handling is a scenario
where we will want to be extra careful.

There are two options here. You can define individual strings using the 'u'
prefix to indicate that it is unicode, or you can globally change all string
literals in a particular module or file to unicode using by using
``from __future__ import unicode_literals``.

Not everything supports unicode strings so this one should be used with some
caution and updated "outside in".  Updating scripts or terminal libraries
with few or no dependencies that can be easily tested is preferred to minimize
the surface area of a potential change.

The complications around updating a core library to start emitting unicode
strings is that all of its dependents need to be prepared to properly handle
receiving unicode strings.

There are notable cases (eg. when interfacing with C++ python bindings) where
using unicode strings can lead to an error, thus changing the return type of a
library with many dependents should be done carefully.

For new code, standalone scripts, and other python code that does not have
these concerns, the use of unicode literals is strongly recommended.

.. sourcecode:: python

   from __future__ import unicode_literals

   def main():
       print('Hello unicode string')

Be mindful of string encodings.  When reading from files containing text it is
safe to assume inputs are utf-8 encoded unless otherwise specified.  When
writing files, use utf-8.

Use helper functions to make it easy to uniformly encode/decode text that may
or may not already be unicode or byte strings.  See the ``dtk.utils.decode()``
and ``dtk.utils.encode()`` for examples of functions that will robustly encode
and decode strings of any type into unicode text and bytes, respectively.


Use new-style exception handling
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Exceptions are no longer valid as statements when `raising exceptions
<https://python-future.org/compatible_idioms.html#raising-exceptions>`_.
`PEP-317 <https://www.python.org/dev/peps/pep-0317/>`_.

.. sourcecode:: python

    # This is no longer valid
    raise ValueError, 'invalid syntax'

    # Write this instead
    raise ValueError('py2+3 syntax')


Use of the ``as`` keyword is now required when catching exceptions instead of
comma (``,``)..  `PEP-3110 <https://www.python.org/dev/peps/pep-3110/>`_.

.. sourcecode:: sh

    # In the past, this was okay
    try:
        ...
    except Valueerror, e:
        ...

    # Using "as ..." is the compatible syntax.
    try:
        ...
    except ValueError as e:
        ...

    # Bad: uses "," in the except clause
    try:
        foo = value['key']
    except (KeyError, ValueError), e:
        ...

    # Good: uses "as" in the except clause
    try:
        foo = value['key']
    except (KeyError, ValueError) as e:
        ...

Use the new octal syntax
~~~~~~~~~~~~~~~~~~~~~~~~

The ambiguous syntax for
`octal numbers <https://python-future.org/compatible_idioms.html#octal-constants>`_
has been replaced with a precise ``0o###`` notation.

.. sourcecode:: python

    # Old-style octal syntax
    chmod = 0644

    # New-style 0o### syntax
    chmod = 0o644


Use the ``repr(..)`` function instead of backticks
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`backtick repr syntax <https://python-future.org/compatible_idioms.html#backtick-repr>`_
is not suported in Python 3 and has been replaced with the ``repr(...)`` function.

.. sourcecode:: text

    # Backtick syntax is no longer supported.
    x_repr = `x`

.. sourcecode:: python

    # Use repr(...) instead.
    x_repr = repr(x)


Use the new ``dict`` idioms
~~~~~~~~~~~~~~~~~~~~~~~~~~~

``dict.iterkeys`` and ``dict.iteritems`` no longer exists in Python 3,
and the ``keys()`` and ``values()`` methods now return
`view objects <https://docs.python.org/3/library/stdtypes.html#dictionary-view-objects>`_.

Use the ``iteritems``, ``iterkeys``, and ``itervalues`` methods from the
``six`` module if you need to preserve the use of iterators.  Use the
``items()`, ``keys()``, and ``values()`` methods instead if the iterator vs.
list performance tradeoff is not a concern.

Use the following built-in idioms as replacements whenever possible.

.. sourcecode:: python

    # This:
    for x in my_dict.iterkeys():
        ...
    keys = my_dict.keys()

    # Becomes:
    for x in my_dict:
        ...
    keys = list(my_dict)

Use ``foo in bar`` to check for dict existence rather than ``bar.has_key(foo)``.
``has_key`` has been removed from Python 3 dictionaries. You should use ``in``
to  check membership.

.. sourcecode:: python

    # This:
    if my_dict.has_key(k):
        ...

    # Becomes:
    if k in my_dict:
        ...


Use ``functools`` for ``reduce``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The `reduce <https://python-future.org/compatible_idioms.html#reduce>`_
function, which was formerly a builtin symbol that was always
in scope, must now be imported from the ``functools`` module.

.. sourcecode:: python

    # This:
    reduce(lambda x, y: x+y, [1, 2, 3, 4, 5])

    # Becomes:
    from functools import reduce

    reduce(lambda x, y: x+y, [1, 2, 3, 4, 5])


Use ``codes.open`` instead of ``file`` and ``open``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``file`` does not exist in Python 3 so all uses of ``file`` will need to be
replaced with ``open()`` to read bytes, or ``codecs.open()`` to read unicode
text.  Prefer ``codecs.open()`` whenever possible.  It's okay to assume that
``utf-8`` is the text encoding to use.

.. sourcecode:: python

    # Replace this usage:
    f = file(path)

    # With this instead:
    with codec.open(path, mode, encoding='utf8') as f:
        ...

When reading/writing files, prefer using ``codecs.open`` to handle unicode
decoding when reading, and encoding when writing.  Note: this is a change in
behavior compared to the regular ``open()`` because the result of ``f.read()`` returns
unicode text and ``f.write()`` expects unicode text when using ``codecs.open()``.

Keep using regular ``open()`` if you require bytes.

.. sourcecode:: python

    # This:
    with open(path, mode) as f:
        ...

    # Becomes:
    with codec.open(path, mode, encoding='utf8') as f:
        ...

Note: ``dtk.utils.write(path, content)`` can be used to write utf-8 encoded
files and will accept either bytes or unicode text as input.

If you want to use bytes, the ``'rt'`` and ``'rb'`` modes can be used with
`io.open <https://python-future.org/compatible_idioms.html#file>`_ as well.


Use the ``exec`` function
~~~~~~~~~~~~~~~~~~~~~~~~~

`exec <https://python-future.org/compatible_idioms.html#exec>`_
is no longer a statement in Python 3.
`PEP-3100 <https://www.python.org/dev/peps/pep-3100/>`_.

.. sourcecode:: python

    # Python 2
    exec "x = 10"

    # Python 2+3
    exec("x = 10")


Update use of ``execfile`` to account for its removal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`execfile <https://python-future.org/compatible_idioms.html#execfile>`_
has been removed in python 3, you can replicate the functionality
with open, compile and exec.
`PEP-3100 <https://www.python.org/dev/peps/pep-3100/>`_.

.. sourcecode:: python

    # This:
    execfile("file.py")

    # Becomes:
    exec(compile(open('file.py').read()))


Update use of ``apply`` to use function call syntax
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`apply <https://python-future.org/compatible_idioms.html#apply>`_
has been removed in Python 3.
`PEP-3100 <https://www.python.org/dev/peps/pep-3100/>`_.

.. sourcecode:: python

    # This:
    apply(f, args, kwargs)

    # Becomes:
    f(*args, **kwargs)


Update use of ``reload``
~~~~~~~~~~~~~~~~~~~~~~~~

``reload`` moved to the to ``imp`` module in Python 3.
`PEP-3100 <https://www.python.org/dev/peps/pep-3100/>`_.

.. sourcecode:: python

    # This:
    reload(module)

    # Becomes:
    from imp import reload

    reload(module)


Update use of ``filter`` and ``map``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`filter and map <https://python-future.org/compatible_idioms.html#map>`_
changed behavior and now return iterator-like objects in Python 3.

Most uses of filter and map can be can be represented using a list
comprehension, but this should be left at the discretion of the implementer to
decide if a list comprehension is appropriate.

For cases when we need ton continue using ``filter`` or ``map``, you can
force the original list-returning behavior by wrapping ``list(...)`` around
the expression.  `PEP-3100 <https://www.python.org/dev/peps/pep-3100/>`_.


.. sourcecode:: python

    # This:
    result = map(do_something, my_list)
    result = filter(is_something, my_list)

    # Becomes:
    result = [do_something(x) for x in my_list]
    result = [x for x in my_list if is_something(x)]
    # or
    result = list(map(do_something, my_list))
    result = list(filter(is_something, my_list))


Update use of ``zip`` when lists are expected
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Like ``filter`` and ``map``, ``zip`` will be returning iterator-like objects
in Python 3.  When code expects a ``list`` to be returned from ``zip``, rather
than simply looping over the result, then it is simplest to just wrap the
``zip`` expression with ``list``.  ``zip`` expressions are not easily replaced
with list comprehensions.  `PEP-3100 <https://www.python.org/dev/peps/pep-3100/>`_.

.. sourcecode:: python

    # This:
    result = zip(list1, list2)

    # Becomes:
    result = list(zip(list1, list2))


Update custom iterators that implement the ``next()`` iterator protocol
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`next <https://python-future.org/compatible_idioms.html#custom-iterators>`_
has become ``__next__`` in class method definitions in Python 3.
There is an easy Python 2+3 solution for this.
`PEP-3114 <https://www.python.org/dev/peps/pep-3114/>`_.

.. sourcecode:: python

    # This:
    def next(self):
        ...

    # Becomes
    def next(self):
        ...

    __next__ = next

    # Futurize also changes this:
    obj.next()
    # To:
    next(obj)


Update use of ``xrange`` to account for its removal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There is no ``xrange`` in Python 3, and ``range`` will function like ``xrange``
did back in Python 2.  ``range`` will no longer return a list in Python 3 so
like ``zip/map/filter``, if you are expecting a list it will be best to wrap it
explicitly.  `PEP-3100 <https://www.python.org/dev/peps/pep-3100/>`_.

.. sourcecode:: python

    # This:
    my_list = range(8)

    # Becomes:
    my_list = list(range(8))


Update use of ``sys.exitfunc`` in favor of ``atexit``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``sys.exitfunc`` has been deprecated since Python 2.4 and has been replaced with
`atexit.register <https://docs.python.org/3/library/atexit.html#atexit.register>`_.

Be aware that ``atexit`` does also provide an unregister function that allows
removal of an exit function.  `PEP-3100 <https://www.python.org/dev/peps/pep-3100/>`_.

.. sourcecode:: python

    # This:
    sys.exitfunc(exit_func)  # exit_func must be parameterless

    # Becomes:
    atexit.register(exit_func)  # register() supports *args and **kwargs


Update use of function attribute names
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Some of the function attributes have been renamed in Python 3. The new names
have been aliased in Python 2 for compatibility and only the new names should
be used.

.. sourcecode:: python

    # This:
    my_func.func_defaults
    my_func.func_dict
    my_func.func_closure
    my_func.func_globals
    my_func.func_code
    my_func.func_name
    my_func.func_doc

    # Becomes:
    my_func.__defaults__
    my_func.__doc__
    my_func.__name__
    my_func.__dict__
    my_func.__closure__
    my_func.__globals__
    my_func.__code__


Update use of the ``intern`` function
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The builtin function ``intern`` has moved to ``sys.intern`` in Python 3.

.. sourcecode:: python

    # This:
    interned_str = intern(some_str)

    # Becomes:
    import sys
    interned_str = sys.intern(some_str)


Update use of method attribute names
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Similarly with how some of the function attributes have been removed, some of
the method attributes have also been removed.

.. sourcecode:: python

    # This:
    ExampleClass.method.im_func
    ExampleClass.method.im_class
    ExampleClass.method.im_self

    # Becomes:
    ExampleClass.method.__func__
    ExampleClass.method.__class__
    ExampleClass.method.__self__

Update use of the ``<>`` not-equals operator
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``<>`` operator for not-equal has been removed in Python 3.
`PEP-3100 <https://www.python.org/dev/peps/pep-3100/>`_.

.. sourcecode:: python

    # This:
    1 <> 2

    # Becomes:
    1 != 2

Update list comprehensions that omit parentheses
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

List comprehensions syntax has become more strict and will require parentheses
in some situation on Python 3.

.. sourcecode:: python

    # This:
    [x for x in 1, 2, 3]

    # Becomes:
    [x for x in (1, 2, 3)]


Update use of ``sys.maxint`` to use ``sys.maxsize`` instead
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``sys.maxint`` has been removed in Python 3.  Use ``sys.maxsize`` instead.

.. sourcecode:: python

    # This:
    sys.maxint

    # Becomes:
    sys.maxsize


Update use of ``StandardError`` to account for its removal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``StandardError`` has been removed in Python 3.  Use Exception instead.
`PEP-3100 <https://www.python.org/dev/peps/pep-3100/>`_.

.. sourcecode:: python

    # This:
    raise StandardError("some error string")

    # Becomes:
    raise Exception("some error string")

Prefer creating custom exception types, or reusing a relevant built-in
exception, rather than throwing ``Exception`` directly, though.


Update use of ``sys.exc_value``, ``sys.exc_type``, and ``sys.exc_traceback``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The deprecated ``sys.exc_value``, ``sys.exc_type``, and ``sys.exc_traceback``
have been removed in Python 3.  Use ``sys.exc_info()`` instead.
`PEP-3100 <https://www.python.org/dev/peps/pep-3100/>`_.

.. sourcecode:: python

    # This:
    sys.exc_value, sys.exc_type, sys.exc_traceback

    # Becomes:
    exc_type, exc_value, exc_traceback = sys.exc_info()


Update use of tuple parameters to account for their removal
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Tuple parameters have been removed in python 3. The futurize stage1 fixes will
replace it with a xxx_todo_changeme variable that you will need to update.
`PEP-3113 <https://www.python.org/dev/peps/pep-3113/>`_.

.. sourcecode:: python

    # This:
    def func(a, (b, c), d):
        ...

    # Becomes:
    def func(a, b_c, d):
        b, c = b_c
        ...
