---
title: "Supercharge Your Python Skills with These Essential Import Tips"
tags:
  - python
  - style
categories:
  - learning log
date: 2019-03-16
description: Unlock the full potential of Python with our essential import tips. Learn how to import modules, packages, and third-party libraries with ease, avoid common errors, and optimize your code for better performance. Our step-by-step guide provides practical examples and best practices for mastering the art of importing in Python. Whether you're a beginner or an advanced user, our tutorial will help you supercharge your Python skills and take your programming to the next level.
---

### `from ... import` vs. `import`

Always avoid wildcard imports like such:

```python
from my_module import *  # don't use this
```

Instead, use regular import like:
```python
import my_module  # use this
```

Why?
- using wildcard import will pollute namespaces
- Using wildcard import will **not** import names with a leading underscore (unless the module defines an `__all__` list)
- *PEP8* recommend using regular import


### What does `__init__.py` do?

1. `__init__.py` is used to specify a package, when import is trying to find the modules,
But it is not required: meaning a package without `__init__.py`,
The system can still find the modules after configuring appropriate `PYTHONPATH` using
`sys.path.append`.

2. `__init__.py` is executed after importing the package, I've seen
sub-directory being imported by appending as environment variable within `__init__.py`

<a name='init-example'></a>
#### Example:
so instead of using `import project.foo.bar` for the following structure:

```
project/
    __init__.py
    foo/
        __init__.py
        bar/
            b.py
```

inside the `__init__.py`, we could do a `sys.path.append(PATH_TO_BAR)`

so with this file structure, you can just do `import project`

they could even add the import statement for you in the `__init__.py`, although it is not transparent.

```
project/
    __init__.py
    foo/
        bar/
            b.py
```

Note: Whatever gets appended last overrides the previous
env variable, so import to the same name module will find the latest append

### Dot Notation (`.`) in Import

```
parent/
    __init__.py
    file.py
    one/
        __init__.py
        anotherfile.py
    two/
        __init__.py
    three/
        __init__.py
```

Each dot in your import will refer to something inside the package, could be another package
or a module. But it can't be a class.

Import python modules could look like:
`import parent.file` or `import parent.one.anotherfile`

From ... import classes or functions look like this:
`from parent.file import class`
which gives you direct access to the class namespace, but not the example above.

### Import Order

Based on PEP8, imports should be grouped by the following order:

1. Standard library import
2. Related third-party import
3. Local application/library specific import

**What is Standard Library Imports?**

Standard library are installed automatically by Python installer, full documentation link
is here: https://docs.python.org/3/library/

**What is the order after grouping?**

There is no specific rules, but based on common preferences, use alphabetical order, with
import first and from ... import after

```python
import abc
import def
import x
from g import gg
from x import xx
```


### Intra-package

In a structure like this, how would you do import from another directory?
say from `module-x.py` import `module-a`

```
top-package/
    __init__.py
    sub-package-a/
        __init__.py
        module-x.py
        module-y.py
    sub-package-b/
        __init__.py
        module-a.py
        module-b.py
```

Here's some examples doing relative imports in `module-x`

```python
import module-y
from . import module-y
from .module-y import classA
from .. import sub-package-b
from ..subpackage-b import module-a
from ..subpackage-b.module-a import classB
```

### Import an import

It is a common practice in `C#` to use import module or static class to stores all the global variables
used for settings, or even all the modules. In Python it would be something like:
`constant.py`

```python
import module_a
import module_b
import module_c

GLOBAL_VAR_MAX = 50
GLOBAL_VAR_MIN = 10
GLOBAL_VAR_TIMEOUT = 2000

GLOBAL_NAME = r'random name'

# or even
class Constant(object):
    gravity = 9.8
    is_true = True
```

With this setup, all the module in the same project would just import the `constant` module and have access to
all the imports and variable. I thought this was a neat way to make code cleaner by getting rid of all the duplicated imports
that might happen.

there are also some voices against it:
- based on the style guide: Constants are usually defined on a *module* level
- also, suggestions have mentioned to refrain from using *class* as it could be instantiated which makes no sense.
- unless there's a valid reason for all those modules to be collected under a common name. If not, then they should
be kept separately. This is due to documentation, as other people open your file, they don't get information on
what is getting imported (what is needed)


#### Same module import multiple times

So if multiple files are importing the same module separately, does python optimize the import?

Yes, python modules are considered as singletons, no matter how many times you import them they get initialized only once.
unless reload is being called

### Reference

[Stack Overflow - constants in Python: at the root of the module or in a namespace inside the module?](https://stackoverflow.com/questions/5027400)

[Stack Overflow - in python, do you need to import modules in each split file?](https://stackoverflow.com/questions/40419582)

[Stack Overflow - Python: Importing an "import file"](https://stackoverflow.com/questions/6206204)

[Stack Overflow - Does python optimize modules when they are imported multiple times?](https://stackoverflow.com/questions/296036)

[Stack Overflow - Why can I import successfully without __init__.py?](https://stackoverflow.com/questions/37974843)

[Stack Overflow - relative path not working even with __init__.py](https://stackoverflow.com/questions/9427037)


