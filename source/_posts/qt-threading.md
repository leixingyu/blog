---
title: "Say Goodbye to Freezing with Qt Multithreading: Learn with Examples"
tags:
  - pyqt
  - template
  - threading
categories:
  - tech summary
date: 2022-05-15
description: Say goodbye to freezing with Qt multithreading using our comprehensive tutorial. Learn how to use multithreading to create responsive and efficient GUI applications, avoid freezing and lag, and improve user experience. Our step-by-step guide provides practical examples and best practices for implementing multithreading in your Qt projects. Whether you're a beginner or an advanced user, our tutorial will help you master the techniques needed to create high-quality and responsive GUI applications with Qt.
---

## Introduction

It is inevitable that some of our tasks in the program will take long time to run,
leaving the user staring at a frozen screen, whether it is reading/writing a large file,
searching database or syncing assets. In a production environment, 
these seconds can really add up for the entire team; can you imagine 
how many working hours are wasted when every user need to wait for, 
let's say, a library tool to query the entire asset database and finally displaying them
every single time. 

We can surely improve the query, but that's usually not where the bottleneck is.
The speed of querying is usually sufficient for human brain to process. What
we want is not to be slow down by un-interactive GUI.
So, why not have assets to fill little by little in a continuous stream, or
just query their name first, and use a separate thread to load in rest of the information
like date created, author and thumbnail.

Now, let's get straight to the point. I'm going to show three basic examples
of different methods to deal with frozen GUI, each has its own use cases.


## Example Issue

```python
import time


class Window(QtWidgets.QMainWindow):
    def __init__(self, parent=None):
        QtWidgets.QMainWindow.__init__(self, parent)

        # ui setup: please ignore
        widget = QtWidgets.QWidget()
        layout = QtWidgets.QGridLayout()

        button = QtWidgets.QPushButton('run task!')
        button.clicked.connect(self.run_long_task)
        edit = QtWidgets.QLineEdit()

        layout.addWidget(edit, 0, 0)
        layout.addWidget(button, 1, 0)
        widget.setLayout(layout)
        self.setCentralWidget(widget)

    def run_long_task(self):
        time.sleep(2)


if __name__ == '__main__':
    app = QtWidgets.QApplication(sys.argv)
    win = Window()
    win.show()
    sys.exit(app.exec_())
```

Here I created a simple example to show how a long-running task
can block controls in our Qt Gui.

The `run_long_task()` is simply a `time.sleep()` that will halt for 2 seconds
util returning control back to the main event loop.

After evoking the `run_long_task()` method, we can 
no longer interact with the line edit from our UI, until the task has finished.

![no-work](https://i.imgur.com/VZBrqsw.gif)|
:---:|
(Clicking and typing has no effect while task is running)|


## `processEvent()`

Using `QApplication.processEvent()` can achieve a **semi-interactive** GUI
during a long-running task. Here's to show what I meant.

Let's add a progress bar to track our task status.

```python
class Window(QtWidgets.QMainWindow):
    def __init__(self, parent=None):
        QtWidgets.QMainWindow.__init__(self, parent)

        # ui setup: add progress bar
        self.ui_progress = QtWidgets.QProgressBar()
        self.statusBar().addPermanentWidget(self.ui_progress)
        ...
    

def update_status(status_bar, msg):
    status_bar.showMessage(msg, 2000)


def update_progress(progress_bar, value):
    progress_bar.setValue(value)

    if value >= 100:
        progress_bar.setVisible(False)
    elif progress_bar.isHidden():
        progress_bar.setVisible(True)
```

And now, by evoking the
static method `processEvent()` during the long-running task, we enforce
Qt to handle normal events like updating GUI, and respond to our user input,
before handing the control back to the task.

```python
    def run_long_task(self):
        for i in range(1, 11):
            sleep(0.5)
            update_status(self.statusBar(), str(i))
            update_progress(self.ui_progress, i*10)
            
            QtCore.QCoreApplication.processEvents()
```

Now, as you can see, our main GUI can update the progress bar and messages,
and even allow us to sort of interact with the line edit, but the interaction is by no
means smooth.

![processEvent](https://i.imgur.com/J3a5hdT.gif)|
|:---:|
(Somewhat interactive, yet choppy)|

Before we create a real multi-threading solution, first, 
let's establish a signal slot workflow that Qt provides: instead of
updating the progress bar and status bar by calling its method directly,
we emit a signal to handle that.

```python
def __init__(self):
    ...
    self.progressed.connect(lambda value: update_progress(self.ui_progress, value))
    self.messaged.connect(lambda msg: update_status(self.statusBar(), msg))

def run_long_task(self):
    for i in range(1, 11):
        sleep(0.5)
        self.progressed.emit(int(i*10))
        self.messaged.emit(str(i))
```


### QThread

<script src="https://gist.github.com/leixingyu/9601e352eba22124a0f97e1fc574a262.js"></script>

Instead of directly subclassing `QThread()`, it is recommended by many to
create a `QObject()` and attach it to a `QThread()`. The `QObject()` we created,
known as the worker, will be running our long-running task, and emits the
update signals.

It is important to keep a note that, the `QThread()` that houses our worker needs to live in the main
event loop (i.e. kept in the main application as `self.__thread`). 
If not, it will be collected by _gc_ and the thread will exit pre-maturely.

`thread.worker = worker` is also essential if the worker doesn't live in the main
event loop (i.e. `self.__worker`).

Now as you can see, we can freely interact with the line edit while the long-running
task is handled in the background.

|![qthread](https://i.imgur.com/CXxVPC4.gif)|
| :---: |
|(Very smooth typing)|

### QThreadPool and QRunnable

A natural progression would be handling multiple long-running tasks,
luckily, we have `QThreadPool` for that.

In the following example, we have **three** long-running task in the background,
while we still have free control of the main GUI.


|![qthreadpool](https://i.imgur.com/ru7UWl1.gif)|
|:---:|
(Multiple threads and smooth typing)|


<script src="https://gist.github.com/leixingyu/f5fc3cc1ef03db1f254dab2a23b85bc7.js"></script>

- `QThreadPool` manages `QRunnable`, which doesn't have built-in signals, thus we
need to attach signals externally by creating a `QObject` to store them.

- `QThreadPool` also deletes the `QRunnable` instances automatically when it finishes by default.

- if max thread is exceeded, the process is queued until a thread is available.

## Reference

[PythonGUIs - Multithreading PyQt5 applications with QThreadPool](https://www.pythonguis.com/tutorials/multithreading-pyqt-applications-qthreadpool/)

[Maya's Programming & Electronics Blog - How To Really, Truly Use QThreads; The Full Explanation](https://mayaposch.wordpress.com/2011/11/01/how-to-really-truly-use-qthreads-the-full-explanation/)

[Stack Overflow - Background thread with QThread in PyQt](https://stackoverflow.com/questions/6783194)

[Qt doc - Process Events](https://doc.qt.io/qt-5/qcoreapplication.html#processEvents)

