---
title: Qt Model View List Model
summary: A Qt5 version of the pyqt model view tutorial series part.1
tags:
  - pyqt
categories:
  - model view programming
date: 2021-02-07
hidden: true
---

The following example demonstrates how to create custom list model for 
displaying list-like data

```python
from Qt import QtGui, QtCore, QtWidgets
import sys

class PaletteListModel(QtCore.QAbstractListModel):
    def __init__(self, colors=[], parent=None):
        QtCore.QAbstractListModel.__init__(self, parent)
        self.__colors = colors

    def headerData(self, section, orientation, role):
        # orientation: indicates horizontal or vertical header
        # section:     indicates which index on the header
        if role == QtCore.Qt.DisplayRole:
            if orientation == QtCore.Qt.Horizontal:
                return 'Palette'
            else:
                return 'Color {}'.format(section+1)

    def rowCount(self, parent):
        # parent: are for tree view with hierarchical structure
        return len(self.__colors)

    def data(self, index, role):
        # display data for each index, of each data role
        if role == QtCore.Qt.EditRole:
            return self.__colors[index.row()].name()

        if role == QtCore.Qt.ToolTipRole:
            return "Hex code: "+self.__colors[index.row()].name()

        if role == QtCore.Qt.DecorationRole:
            row = index.row()
            value = self.__colors[row]

            pixmap = QtGui.QPixmap(26, 26)
            pixmap.fill(value)
            icon = QtGui.QIcon(pixmap)
            return icon

        if role == QtCore.Qt.DisplayRole:
            row = index.row()
            value = self.__colors[row]
            return value.name()

    def flags(self, index):
        return QtCore.Qt.ItemIsEditable | QtCore.Qt.ItemIsEnabled | QtCore.Qt.ItemIsSelectable

    def setData(self, index, value, role=QtCore.Qt.EditRole):
        # set data for each index of the value, data role is set to edit role default
        if role == QtCore.Qt.EditRole:
            row = index.row()
            color = QtGui.QColor(value)

            if color.isValid():
                self.__colors[row] = color
                # have to emit and dataChanged signal to sync with display
                self.dataChanged.emit(index, index)
                return True
        return False

    def insertRows(self, position, rows, parent=QtCore.QModelIndex()):
        self.beginInsertRows(parent, position, position+rows-1)

        for i in range(rows):
            self.__colors.insert(position, QtGui.QColor("#000000"))

        self.endInsertRows()
        return True

    def removeRows(self, position, rows, parent=QtCore.QModelIndex()):
        self.beginRemoveRows(parent, position, position+rows-1)

        for i in range(rows):
            value = self.__colors[position]
            self.__colors.remove(value)

        self.endRemoveRows()
        return True

if __name__ == '__main__':
    app = QtWidgets.QApplication(sys.argv)

    red = QtGui.QColor(255, 0, 0)
    green = QtGui.QColor(0, 255, 0)
    blue = QtGui.QColor(0, 0, 255)

    model = PaletteListModel([red, green, blue])
    model.insertRows(0, 5)

    listView = QtWidgets.QListView()
    listView.show()
    listView.setModel(model)

    sys.exit(app.exec_())
```

### Note

1. Since `parent` is used  for hierarchical structure, it is not set or set to null index
in this example.

2. `index` is a `QModelIndex` instance to locate data in model

3. In `setData()`, make sure to add `self.dataChanged.emit(index, index)` for syncing data change

4. For inserting and removing rows
    1. always starts with `insertRows/removeRows` and
    close with `endInsertRows/endRemoveRows` so that view and model is in-sync
    2. no parent in list view so pass a null index: `QModelIndex()`
    3. essentially, it is modifying the `self.__colors` which is displayed in `data()`

### Source
[https://www.youtube.com/watch?v=mCHVI8OXDxw&list=PLG4y4w32mF3qmweFe59P_8INlVNOm_IHo](https://www.youtube.com/watch?v=mCHVI8OXDxw&list=PLG4y4w32mF3qmweFe59P_8INlVNOm_IHo)

[https://doc.qt.io/qtforpython-5.12/PySide2/QtCore/QModelIndex.html#PySide2.QtCore.QModelIndex](https://doc.qt.io/qtforpython-5.12/PySide2/QtCore/QModelIndex.html#PySide2.QtCore.QModelIndex)