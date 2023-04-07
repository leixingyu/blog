---
title: "Unlock the Power of Unreal Command Line with Python: 3 Easy Methods"
tags:
  - unreal
  - python
  - batch
categories:
  - learning log
date: 2022-11-06
description: Unlock the power of Unreal Command Line with Python using our easy-to-follow tutorial. Learn three proven methods to automate tasks and enhance your Unreal Engine workflow with ease. Our step-by-step guide provides practical examples and best practices for leveraging the full potential of Python to simplify your development process. Whether you're a beginner or an advanced user, our tutorial will help you master the techniques needed to unlock the power of Unreal Command Line with Python.
---
  
## Introduction

The Unreal official documentation has two simple examples to run Python through
commandline, one being Commandlet: a headless mode without even opening the Editor,
the other one being the Full Editor mode which has Unreal Editor fully opened and loaded.

Do you know there's actually a "third" way?
And today, I'll discuss my experience and preferences of using all of them.


## Commandlet

```commandline
UnrealEditor.exe "C:\projects\MyProject.uproject" -run=pythonscript -script="c:\\my_script.py"
```

The "Commandlet" mode runs Unreal in headless mode without editor opened,
this means no asset and level loading (but we have access to the `unreal` python module). 

This is great if we want to write to the project (for example, importing/creating
assets), but it's not going to work well for reading from the project.

## Full Editor

```commandline
UnrealEditor.exe "C:\projects\MyProject.uproject" -ExecutePythonScript="c:\my_script.py"
```

Using "Full Editor" mode, we now have everything loaded in engine and are able to do
almost all operations, the trade-off being there's
a longer loading time, and also the editor will be opened.

Note that at this point, once the python script finishes executing, the editor
and the command prompt window will close automatically.

### UI

If we ever needs to create a simple UI that launches from the "Full Editor" mode,
we could do something like this in the python script.

```python
import sys

import unreal


if __name__ == "__main__":
    global app
    global window

    if not QtWidgets.QApplication.instance():
        app = QtWidgets.QApplication(sys.argv)

    window = TestWindow()
    unreal.parent_external_window_to_slate(
        window.winId().__init__(),
        unreal.SlateParentWindowSearchMethod.ACTIVE_WINDOW
    )
    window.show()
    sys.exit(app.exec_())
```

Note that we are using `sys.exit(app.exec_())` to prevent the UI being 
immediately destroyed. This does mean the Qt event loop would take over 
the main thread which renders Unreal editor non-interactable until
function calls hands over the process to Unreal.

However, I do find in rare cases 
(for example: when launching Movie Render Queue through commandline), 
either the UI still hangs the editor with the operation,
or (without the UI) the editor just exit immediately after the operation is being called.
Either of this will not let the render to finish.

This brings me to method number three that perfectly solves this issue. 

## Init Cmd

```commandline
UnrealEditor.exe "C:\projects\MyProject.uproject" -ExecCmds="py c:/my_script.py"
```

This method, although not present on Unreal Engine's wiki, it is mentioned 
in the FAQ by the official Epic Games account on Linear Content Creation.

> For Automation/Testing you can pass the following to keep the editor running.
> 
> –ExecCmds="py PathToPyFile"
> 
> Once you are done, you can call:
unreal.SystemLibrary.execute_console_command(None,"QUIT_EDITOR")

This is almost like running a specified startup python script in Unreal,
and the editor is fully interactable when the script is running. 

The editor exit automatically when calling "QUIT_EDITOR", if running a script with UI, we can hook up
to the `closeEvent()`.

```python
def closeEvent(self, event):
    QtWidgets.QMainWindow.closeEvent(event)
    unreal.SystemLibrary.execute_console_command(None, "QUIT_EDITOR")
```

to override `closeEvent()` in instance-level instead of class-level, see [here](https://www.xingyulei.com/post/qt-template/#unreal-engine)

## Logging

To generate output from the commandline we can add two arguments: `-stdout`
and `-FullStdOutLogOutput`.

```python
def call(python_script):
    command = [
        UNREAL_EXE,
        UPROJECT,
        "-ExecCmds=py {}".format(python_script),
        "-stdout",
        "-FullStdOutLogOutput"
    ]

    proc = subprocess.Popen(
        command,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,
    )

    return proc.communicate()
```

and we can filter out only the log generated from Python as such

```python
output = call('python_script.py')
for line in str(output).split(r'\r\n'):
    if 'LogPython' in line:
        print(line)
```

## Environment

Another thing to consider is that, running in batch mode can mess up our tools and start up scripts. 
due to it being a different environment. We might want to re-configure our environment 
which you can do like such:

```python
import os
import subprocess

env = os.environ.copy()
env['FOO'] = 'BAR'
# more env ...

proc = subprocess.Popen(
        command,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,
        env=env
    )
```


## Reference

[Unreal Doc - Scripting the Unreal Editor Using Python](https://docs.unrealengine.com/5.0/en-US/scripting-the-unreal-editor-using-python/)

[Unreal Forum - Unreal Engine Technical Guide to Linear Content Creation FAQs](https://dev.epicgames.com/community/learning/courses/Oex/unreal-engine-technical-guide-to-linear-content-creation-faqs/XvO9/unreal-engine-common-questions)

[Adamrehn Docs - UE4CLI](https://docs.adamrehn.com/ue4cli/descriptor-commands/run)
