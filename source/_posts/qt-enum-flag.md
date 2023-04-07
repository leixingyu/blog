---
title: "A Complete Comparison: Understanding QEnum and QFlags in Qt"
tags:
  - pyqt
categories:
  - learning log
date: 2021-12-06
description: Get a complete understanding of QEnum and QFlags in Qt with our comprehensive comparison tutorial. Our step-by-step guide provides practical examples and best practices for selecting the right type of enumeration for your application, and improving your development efficiency. Whether you're a beginner or an experienced Qt developer, our tutorial will help you master the differences between QEnum and QFlags and choose the right enumeration type for your application
---

### Introduction

I often take Qt [namespace](https://doc.qt.io/qt-5/qt.html) for granted, it became
a natural habit of setting a Qt parameter using Qt namespace, for instance:
setting alignment for a `QStandardItem`

```python
item = QtGui.QStandardItem('test')
item.setTextAlignment(QtCore.Qt.AlignRight)
type(QtCore.Qt.AlignRight)
```

Here, the `Qt.AlignRight` is an `AlignmentFlag` Enum type object which has a 
value of `0x0002` or `2`, which creates the behavior of aligning with the right edge.

Now, let's try parsing the alignment of the `QStandardItem` again, using:

```python
align = item.textAlignment()
type(align)
```

this returns a `PyQt5.QtCore.Alignment` object, which we don't really
know the value of. And also note that it is not a `AlignmentFlag` object like 
previously.

So, What's the difference between these two, and how should I retrieve the namespace
value?

### `QEnum` and `QFlags`

QFlags is used to store combinations of Enum, which provides type checking safety.
thus, `Alignment` QFlags type is simply a typedef for `AlignmentFlag` QEnum.


- `Qt.AlignmentFlag` is QEnum type; `AlignmentFlag` being the enum name
- `Qt.Alignment` is QFlags type; `Alignment` being the type name
- there's also non-flag enums, which have the same type and enum name

### Example

Declaration of both object types

```python
# <class 'PyQt5.QtCore.AlignmentFlag'>
align_flag = QtCore.Qt.AlignRight
align_flag_value = 2

# <class 'PyQt5.QtCore.Alignment'>
align = QtCore.Qt.Alignment(align_flag)
# or <class 'int'>
align = QtCore.Qt.Alignment(align_flag_value)
```

As you can see, `setAlignment()` takes a `Qt.Alignment parameter`, which means that any
combination of `Qt.AlignmentFlag` values, or `int`, is legal.

```python
item = QtGui.QStandardItem('test')

# Alignment type is compatible with either int or AlignmentFlag(Enum)
# <class 'PyQt5.QtCore.AlignmentFlag'>
item.setTextAlignment(QtCore.Qt.AlignRight)
# or <class 'PyQt5.QtCore.Alignment'>
item.setTextAlignment(align)
# or <class 'int'>
item.setTextAlignment(2)
```

The return value is QFlags type, but it can be cast to an integer type to
reveal its Enum value

```python
# <class 'PyQt5.QtCore.Alignment'>
align = pathItem.textAlignment()

# to get the Alignment value, convert the QFlags to int
int(align)
```

### Parse Namespace and Value Mapping

Still not sure what the Enum value means? 

you can either check the docs, or 
use this to print out a mapping of the namespace and its corresponding value

```python
def enum_mapping(cls, enum):
    mapping = dict()
    for key in dir(cls):
        value = getattr(cls, key)
        if isinstance(value, enum):
            mapping[key] = value
    return mapping

enum = enum_mapping(QtCore.Qt, QtCore.Qt.AlignmentFlag)
# enum = enum_mapping(QtWidgets, QtWidgets.QStyle)

for item in sorted(enum.items(), key=str):
    print('%s: %s' % item)
```

### Reference

[GitHub - qutebrowser](https://github.com/qutebrowser/qutebrowser/blob/v1.0.3/qutebrowser/utils/debug.py#L95-L131)

[Qt Doc - QFlags Class Reference](https://het.as.utexas.edu/HET/Software/html/qflags.html)

[Qt Doc - QEnum/QFlag](https://doc.qt.io/qtforpython-5/PySide2/QtCore/QEnum.html)


