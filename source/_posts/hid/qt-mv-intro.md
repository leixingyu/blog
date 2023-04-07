---
title: Qt Model View List
summary: A Qt5 version of the pyqt model view tutorial series part.2
tags:
  - pyqt
categories:
  - model view programming
date: 2021-02-06
hidden: true
---

The following example demonstrated how same data is accessed through a Model
and shared across Views

```python
from Qt import QtGui, QtCore, QtWidgets
import sys

if __name__ == '__main__':
    app = QtWidgets.QApplication(sys.argv)
    
    # no QString needed in Qt5, use python string instead
    data = ['one', 'two', 'three']

    listView = QtWidgets.QListView()
    listView.show()

    model = QtCore.QStringListModel(data)
    listView.setModel(model)

    combobox = QtWidgets.QComboBox()
    combobox.setModel(model)
    combobox.show()

    sys.exit(app.exec_())
```

In Qt 5, the `QtCore.QStringList()` is replaced by python's built-in `string` type

```python
    # qt 4
    data = QtCore.QStringList()
    data << "one" << "two" << "three" << "four" << "five"
    
    # qt 5
    data = ['one', 'two', 'three', 'four', 'five']
```

### Source

[https://www.youtube.com/watch?v=mCHVI8OXDxw&list=PLG4y4w32mF3qmweFe59P_8INlVNOm_IHo](https://www.youtube.com/watch?v=mCHVI8OXDxw&list=PLG4y4w32mF3qmweFe59P_8INlVNOm_IHo)

[https://stackoverflow.com/questions/27757678/importerror-cannot-import-name-qstringlist-in-pyqt5](https://stackoverflow.com/questions/27757678/importerror-cannot-import-name-qstringlist-in-pyqt5)