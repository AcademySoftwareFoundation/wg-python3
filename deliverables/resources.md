# Resources

The documents collates from useful resources for use both when
transitioning code to Python 3 and when planning a transition strategy.

When adding a new resource to this list,
please also include a short description of why the resource is useful.

* A guide for porting code from Python 2 to 3 in a way that supports both
  (https://portingguide.readthedocs.io/en/latest/index.html):
  This guide's biggest use is in outlining many things that changed between Python 2 and 3,
  how to change code to adapt to those changes,
  and even how common those changes come up in transitions.
  The guide also gives a very basic strategy for transitioning a code base to Python 3.
  I have found this guide most useful in preparing teaching material for developers.

* A keynote by Instagram at PyCon (https://youtu.be/66XoCk79kjM?t=768):
  The keynote describes Instagram's high level strategy for transitioning to Python 3.
  This was very useful in coming up with our own strategy.
  They also talk about some of the pitfalls they hit when transitioning.

* Cheat Sheet for Python 2 and 3 compatible code
  (http://python-future.org/compatible_idioms.html)
  This is a sub page of the Python `future` website.
  It provides examples of the differences,
  and comes with options we can consider when
  the code must be compatible for both Python 2 and 3.
