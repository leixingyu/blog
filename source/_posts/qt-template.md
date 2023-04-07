---
title: "(Hopefully) The Only Qt Template You'll Ever Need"
tags:
  - pyqt
  - template
categories:
  - tech summary
date: 2018-10-16
top: 999
description: Streamline your Qt development process with our comprehensive template that covers all the essentials. Our guide provides practical examples and best practices for creating a high-quality, reusable Qt project that meets your specific needs. Whether you're a beginner or an advanced user, our template will help you master the techniques needed to create Qt applications quickly and easily. Say goodbye to wasted time and resources, and hello to the ultimate Qt template.
---

# General

## Template

```python
class Window(QtWidgets.QMainWindow):

    def __init__(self, parent=None):
        super(Window, self).__init__(parent)

        self.setAttribute(QtCore.Qt.WA_DeleteOnClose)
        self.setWindowFlags(self.windowFlags() | QtCore.Qt.WindowStaysOnTopHint)
        _loadUi(UI_PATH, self)
```
        
## Import Qt Modules

if you are using `PyQt5` binding:

```python
from PyQt5 import QtCore, QtGui, QtWidgets
from PyQt5.uic import loadUi
```

if you are using `PySide2` binding, here's a wrapper I made which can
be found [here](#pyside2-ui-loading).

I'm using [Qt.py](https://github.com/mottosso/Qt.py) a Python 2 & 3 shim around all Qt bindings,
it just saves a lot of work.

```python
from Qt import QtCore, QtGui, QtWidgets
from Qt import _loadUi
```

## Using QApplication

Launch as standalone application using `QApplication`, or if a `QApplication` instance
exist, there's no need to instantiate it. use `QtWidgets.QApplication.instance()` to check

```python
if __name__ == '__main__':
    import sys
    if QtWidgets.QApplication.instance():
        window = ToolWindow()
        window.show()
    else:
        app = QtWidgets.QApplication(sys.argv)
        window = ToolWindow()
        window.show()
        sys.exit(app.exec_())
```

## Close Event

### Delete Widget when Closed

by default, closing the widget is hiding it not destroying the object

```python
{WindowObject}.setAttribute(QtCore.Qt.WA_DeleteOnClose)
```

### Unique Instance

Raise existing instance (for example: widgets that are hidden during `closeEvent`);
if no instance is available, create a new one.

```python
def show():
    global WINDOW

    exists = WINDOW is not None
    if not exists:
        WINDOW = ToolWindow()

    WINDOW.showNormal()
```

## Other

### Register Style Sheet
```python
def show():
    ...
    APP = QtWidgets.QApplication(sys.argv)
    with open(PATH_TO_STYLESHEET, 'r') as f:
        qss = f.read()
        APP.setStyleSheet(qss)
```

### Register QResources
`QtCore.QResource.registerResource(PATH_TO_RCC)`

# Maya

## Parent to Maya's Main Window

by default Qt window is **not** parented to Maya application

```python
from shiboken2 import wrapInstance


def get_maya_main_window():
    import maya.OpenMayaUI
    main_window_ptr = maya.OpenMayaUI.MQtUtil.mainWindow()
    return wrapInstance(long(main_window_ptr), QtWidgets.QMainWindow)


class ModuleUI(QtWidgets.QMainWindow):
    def __init__(self, parent=get_maya_main_window()):
        super(ModuleUI, self).__init__(parent)
        _loadUi(UI_PATH, self)
```

## Window Docking

### Dockable Window

<script src="https://gist.github.com/leixingyu/8fb523cfd38b47c9cb1476b5c41eafd7.js"></script>

The widget/window can now be docked into Maya's main window; 

### Auto-docking

The above example of auto-docking may not work for certain Maya builds

A workaround is to use the following method, but it only applies to `QWidget` type not `QMainWindow` type.

<script src="https://gist.github.com/leixingyu/83358dedd008f6a51e7f7b2b14d2fb34.js"></script>

## Instantiate Widget

When the UI has no parent (Maya main window), it will get instantly destroyed by the
garbage collector, unless you keep an instance:
```python
# in the show function
def show():
    window = ModuleUI()
    window.show()
    return window

# in maya script editor
win = moduleName.show()
```

or using global variable to store the instance

```python
def show():
    global window
    window = ModuleUI()
    window.show()

# in maya script editor
moduleName.show()
```

### Close Application in Maya

```python
def show():
    if cmds.window('xxx', q=1, exists=1):
        cmds.deleteUI('xxx')
    global window
    window = somethingWindow()
    window.setObjectName('xxx')
    window.show()
```

### Error in PyCharm

When this error occurs, it is due to MayaDevKit environment
MayaDevKit allows maya python command auto-completion, remove it from PyCharm

```python
app = QtWidgets.QApplication(sys.argv)
TypeError: 'NoneType' object is not callable
```

# Unreal Engine

```python
import unreal

def show():
    global app
    global window

    app = QtWidgets.QApplication.instance() or QtWidgets.QApplication(sys.argv)

    window = TestWindow()
    window.show()  # this needs to happen before parenting
    unreal.parent_external_window_to_slate(int(window.winId()))
```

## Raise Existing Instance

in the `show()` we need to return the window instance and assign it to a global variable:

```python
# main.py
def show():
    ...
    window.show()
    return window
```

in the parent python command call (this maybe an Unreal menu command)

```python
from test import main
global editor
editor = main.show()
```

## Override `closeEvent()`

To exit the Unreal Editor when Qt Window is closed:

```python
def closeEvent(self, event):
    QtWidgets.QMainWindow.closeEvent(event)
    unreal.SystemLibrary.execute_console_command(None, "QUIT_EDITOR")
```

To override at an instance level:

```python
import types

if __name__ == "__main__":
    def _close_event(self, event):
        QtWidgets.QMainWindow.closeEvent(event)
        unreal.SystemLibrary.execute_console_command(None, "QUIT_EDITOR")

    win = Window()
    win.closeEvent = types.MethodType(_close_event, win)
```

# Bonus:

## PySide2 UI Loading

```python
from PySide2 import QtCore, QtWidgets
from PySide2.QtUiTools import QUiLoader

class WrapperWindow(QtWidgets.QMainWindow):
    ui = ''
    
    def __init__(self, parent=None):
        super(WrapperWindow, self).__init__(parent)

        loader = QUiLoader()
        _file = QtCore.QFile(self.ui)
        _file.open(QtCore.QFile.ReadOnly)
        self.widget = loader.load(_file, parent)
        _file.close()

        self.setCentralWidget(self.widget)


class Window(WrapperWindow):
    ui = 'PATH_TO_UI_FILE'
    
    def __init__(self, parent=None):
        super(Window, self).__init__(parent)
        
        self.setCentralWidget(self.widget)

        # now we can access ui element like so
        self.widget.uiBtn.clicked.connect(self.someFunc)
```

# Reference

[Reddit - PyQt5 - Why do I get an Empty Window?](https://www.reddit.com/r/learnpython/comments/jyw8z1/pyqt5_why_do_i_get_an_empty_window/)

[Stack Overflow - pycharm use pyside2 TypeError: 'NoneType' object is not callable](https://stackoverflow.com/questions/58925453)

