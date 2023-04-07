---
title: "Take Your Qt App to the Next Level with Custom Widgets and Dialogs (No .ui File Needed)"
tags:
  - pyqt
  - template
categories:
  - tech summary
date: 2021-02-11
description: Enhance your Qt app with custom widgets and dialogs using our tutorial – no .ui file needed. Our step-by-step guide provides practical examples and best practices for creating custom widgets and dialogs, and taking your app's user interface to the next level. Whether you're a beginner or an advanced user, our tutorial will help you master the techniques needed to make your Qt app stand out from the crowd.
---

### UI Module without `.ui` file

There is a different between inheriting from `QWidget` class vs `QMainWindow`

Inheriting from `QWidget`:

```python
class InheritQWidget(QtWidgets.QWidget):
    def __init__(self, parent=None):
        super(InheritQWidget, self).__init__(parent)

        # initialization object
        layout = QtWidgets.QGridLayout()
        listWidget = QtWidgets.QListWidget()
        #treeWidget = QtWidgets.QTreeWidget()

        # set
        # treeWidget.setParent(listWidget)
        layout.addWidget(listWidget)
        self.setLayout(layout)

        listWidget.addItem('item A')
        listWidget.addItem('item B')
```

Inheriting from `QMainWindow`

```python
class InheritQMainWindow(QtWidgets.QMainWindow):
    # Window inherits from QMainWindow the layout is already defined
    # to accommodate any toolbars or any other QMainWindow component
    # use the setCentralWidget() to accommodate this
    
    def __init__(self, parent=None):
        super(InheritQMainWindow, self).__init__(parent)

        # initialization object
        widget = QtWidgets.QWidget()
        layout = QtWidgets.QGridLayout()

        # set
        self.setCentralWidget(widget)
        widget.setLayout(layout)
        
        label = QtWidgets.QLabel('test')
        layout.addWidget(label, 0, 0)
```

### Custom Dialog

<script src="https://gist.github.com/leixingyu/2d14eec59143c2b3622a2bb56375ffc0.js"></script>

Sometimes you need a quick window to display some information,
but the built-in qt message boxes aren't suitable for the job.

---

### Custom Widget

You can choose to create a widget class, but also for saving time, you can create
a temporary custom widget

example:

<script src="https://gist.github.com/leixingyu/f0b339a3fb66c62c87b6ffadf7777df2.js"></script>

Couple of things worth noting:

1. correctly initialize the widget

    `self.customWidget = QtWidgets.QWidget()` will allow widget be child of the main window,
    thus allow widget to close when main window is closed
    
    so not `customWidget = QtWidgets.QWidget()` or `self.customWidget = QtWidgets.QWidget(self)`

2. use `getattr` in combination with `QStyle` and name of the built-in icon

3. use `win.setAttribute(QtCore.Qt.WA_DeleteOnClose)` to make sure the child widget is killed
after main window is closed, because default close only hides window objects
   
4. `self.customWidget.show()` is the core command to call it to display


### Custom `QMessageBox`

add custom buttons to the `QMessageBox` layout

```python
dialog = QtWidgets.QMessageBox()
dialog.setText("Overwrite?")
dialog.setIcon(QtWidgets.QMessageBox.Critical)

yes_btn = dialog.addButton("Yes!", QtWidgets.QMessageBox.YesRole)
no_btn = dialog.addButton("No", QtWidgets.QMessageBox.NoRole)
abort_btn = dialog.addButton("Abort", QtWidgets.QMessageBox.RejectRole)
dialog.exec_()

if dialog.clickedButton() == yes_btn:
    print("yes")
elif dialog.clickedButton() == no_btn:
    print("no")
elif dialog.clickedButton() == abort_btn:
    print("abort")
```

Don't use the return value of `QMessageBox::exec`, as it only makes sense
for standard buttons. Also don't rely on `buttonRole`
as multiple buttons could be sharing the same role.

### Reference

[Programiz - Python getattr()](https://www.programiz.com/python-programming/methods/built-in/getattr)

[GUIS - Q&A: Are there any built-in QIcons?](https://forum.learnpyqt.com/t/are-there-any-built-in-qicons/185/2)


