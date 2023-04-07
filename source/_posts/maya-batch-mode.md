---
title: "Save Time and Boost Efficiency: Learn to Run Maya in Batch Mode"
tags:
  - maya
  - python
categories:
  - tech summary
date: 2019-05-09
description: Learn how to run Maya in batch mode for animation and game development with our step-by-step tutorial. Streamline your workflow and save time with this efficient method for rendering large scenes and complex animations. Join our community of artists and game developers today and take your projects to the next level with Maya!
---

### Import Standalone Module

```python
import maya.standalone
maya.standalone.initialize(name='python')
```

Sometimes, import error could occur
```python
ImportError: No module named maya.standalone
```

**Solution**
  - check your python interpreter, it should match the maya python2 version. I changed my project interpreter in my pycharm setting. 
  - setup system environment variable, like the following
  - if error still occurs, please check your maya directory

```python
import sys
import os


MAYA_LOCATION = "C:/Program Files/Autodesk/Maya2018"
PYTHON_LOCATION = MAYA_LOCATION + "/Python/Lib/site-packages"

os.environ["MAYA_LOCATION"] = MAYA_LOCATION
os.environ["PYTHONPATH"] = PYTHON_LOCATION

sys.path.append(MAYA_LOCATION)
sys.path.append(PYTHON_LOCATION)
sys.path.append(MAYA_LOCATION+"/bin")
sys.path.append(MAYA_LOCATION+"/lib")
sys.path.append(MAYA_LOCATION+"/Python")
sys.path.append(MAYA_LOCATION+"/Python/DLLs")
sys.path.append(MAYA_LOCATION+"/Python/Lib")
sys.path.append(MAYA_LOCATION+"/Python/Lib/plat-win")
sys.path.append(MAYA_LOCATION+"/Python/Lib/lib-tk")
print('\n'.join(sys.path))
```

### Add Functions

Let's add a test function to our standalone script.

```python
def test():
    import maya.cmds as cmds
    # full path to your Maya file to OPEN
    maya_file_to_open = r"C:\Users\Lei\Desktop\test.ma"
    # Open your file
    cmds.file(maya_file_to_open, o=True)
    # full path to your Maya file to IMPORT
    maya_file_to_import = r"C:\Users\Lei\Desktop\import.ma"
    # Import the file. the variable "nodes" will hold the names of all nodes imported, just in case.
    cmds.file(maya_file_to_import, i=True, type="mayaAscii")
    render = r"C:\Users\Lei\Desktop\te.ma"
    cmds.file(rename=render)
    cmds.file(force=True, save=True, options='v=1;p=17', type='mayaBinary')
    print('a')
```
This test function opens a maya file on my desktop and import a maya file and save out as another file.

### Execute from `mayapy`

**Execute from Command Line**

- to do so, make sure to add mayapy to your system environment variable.

- in system variable: add `C:\Program Files\Autodesk\Maya2018\bin` to `Path`

- open command prompt, and enter `mayapy [directory/firstStandalone.py]`

**Execute using `subprocess`**

you can do it without going to command line, but by executing another `.py` using subprocess

```python
import subprocess

command = 'mayapy [directory/firstStandalone.py]'
process = subprocess.Popen(command, shell=True, stdout=subprocess.PIPE)
process.wait()

print(process.returncode)  # this return the message from cmd
```

### Standalone Script with Arguments

Sometimes, we would want to pass arguments to our standalone script

get argument as variable using `sys.argv`

```python
import sys

file_path = sys.argv[1]
file_name = sys.argv[2]
```

Example from Toadstorm Nerdblog:

```python
import subprocess


maya_path = 'directory/mayapy.exe'  # mayapy or full path to mayapy.exe
script_path = 'directory/firstStandalone.py'


def add_layer(file_names,layer_name):
    for file_name in file_names:
        command = r'mayapy {} {} {}'.format(
            script_path, file_name, layer_name
        )
        
        process = subprocess.Popen(
            command,
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE
        )
        process.wait()
        print(process.returncode)  # this return the message from cmd
        
        
if __name__ == '__main__':
    # define a list of filenames to iterate through
    files = ['file1', 'file2', 'file3']
    render_layer = 'a new render layer'
     
    # run procedure, assuming you've already defined it
    add_layer(files, render_layer)
```

### Reference

[Tech-Artist Org - Import maya.standalone problem](http://discourse.techart.online/t/import-maya-standalone-problem/1437)

[Stack Overflow - use external python script to open maya and run another script inside maya](https://stackoverflow.com/questions/27437733)

[Stack Overflow - How to use an external interpreter for Maya?](https://stackoverflow.com/questions/17500157)

[Toadstorm Nerdblog - Python in Maya Standalone](https://www.toadstorm.com/blog/?p=136)


