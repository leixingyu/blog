---
title: "Learn Qt Custom Sorting with Our Easy Guide"
tags:
  - pyqt
categories:
  - tech summary
date: 2021-01-23
description: Master custom sorting in Qt with our easy-to-follow guide. Our tutorial provides step-by-step instructions and practical examples for customizing sorting behavior in your Qt applications. Whether you're a beginner or an experienced developer, our guide will help you understand the fundamentals of Qt sorting and apply them to your own projects. Start optimizing your application's performance and user experience today with our comprehensive guide to custom sorting in Qt.
---

### Sorting Overview

Sorting happens a lot in qt viewports like list, table and tree. Using 
convenience class such as item-based widget provides limited sorting options.

One common thing may occur when you are sorting items is the widget treats their
value as *string* instead of *int*, which will result incorrect ordering

```python
list = ["3", "1", "2", "20", "92", "89", "40", "10", "11"]

if __name__ == '__main__':
    app = QtWidgets.QApplication(sys.argv)
    mywidget = QtWidgets.QTableWidget()
    mywidget.insertColumn(0)

    for index in range(len(list)):
        mywidget.insertRow(index)
        value = list[index]
        item = QtWidgets.QTableWidgetItem(value)
        mywidget.setItem(index, 0, item)

    mywidget.sortItems(0, QtCore.Qt.AscendingOrder)
    mywidget.show()
    sys.exit(app.exec_())
```

This will sort the list of items based on their string, so the order will be:
```
- 1
- 10
- 11
- 2
```
instead of 
```
- 1
- 2
- ...
```

So naturally, you would need to specify the item value as integer type, 
but you cannot instantiate item with integer, but `setData()` will work
```python
# intead of 
item = QtWidgets.QTableWidgetItem(int(value))
# do this
item.setData(QtCore.Qt.ItemDataRole, int(value))
```

Now everything will sort by numeric order

### Custom Sorting (Operator override)

What if you need more than numeric value for your data, for example, when displaying frame number, you would like to include frame range
too. Like frame 1, frame 2, frame 3-7, frame 8, frame 9-14. It looks like using string
is the only option.

This time you need to override value compare operator for these value, that is making
your own item class like the following example

<script src="https://gist.github.com/leixingyu/0eea1cb8e325a8d52cc3a64953faf467.js"></script>

the `__lt__` is the less than operator (`<`)

Everything should behave correctly

### Custom Sorting (QSortFilterProxyModel Class)

https://doc-snapshots.qt.io/qtforpython-5.15/PySide2/QtCore/QSortFilterProxyModel.html

### Reference

[Stack Overflow - Is it possible to sort numbers in a QTreeWidget column?](https://stackoverflow.com/questions/363200)


