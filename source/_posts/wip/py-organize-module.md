---
title: How to Organize Custom Python Library
summary: 
tags:
  - python
  - style
categories:
  - learning log
date: 2021-08-14
hidden: true
---

I saw this post on stack overflow and thought it is something worth summarizing

How should we organize our own library?

### Method One

Stack everything inside one root package, each module contains many functions, one utility module containing
all the generic functions

```
root (package)
    animation.py  (module)
    audio.py      (module)
    modelling.py  (module)
    rigging.py    (module)
    library.py    (module)
    util.py       (module)
```

### Method Two

Have multiple separated packages, and split the code into smaller modules

```
root (package)
    animation (package)
        __init__.py  (module)
        core.py      (module)
        utils.py     (module)
        ui.py        (module
        bar.py       (module)

    audio (package)
        __init__.py  (module)
        core.py      (module)
        utils.py     (module)
        ui.py        (module
        foo.py       (module)

   rigging (package)
        __init__.py  (module)
        core.py      (module)
        utils.py     (module)
        ui.py        (module
        foo.py       (module)

   etc
```

The `__init__.py` inside of each package would import all the necessary functions

### Conclusion

Either are okay, but there are certain rules/recommendation to follow:

1. Functions in the same module are closely connected
2. Having fewer but bigger modules could be difficult to navigate
3. Try to make each module standalone
4. Having many separated modules makes import efficient, but the module should not be too small

### Reference

https://stackoverflow.com/questions/52082078/best-way-to-keep-python-modules-organised