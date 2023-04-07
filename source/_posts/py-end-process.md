---
title: "Simplify Windows Process Management with Python: Find PIDs and End Processes"
tags:
  - python
  - windows
  - batch
categories:
  - tech summary
date: 2021-10-25
description: Learn how to find PIDs and end processes with ease, automate routine tasks, and streamline your workflow. Our step-by-step guide provides practical examples and best practices for optimizing your system's performance. Whether you're a beginner or an advanced user, our tutorial will help you master the techniques needed to simplify Windows process management with Python.
---

  
### Introduction

Interacting with Windows shell to end process is very common, 
there are many ways to do so, like through the traditional batch script

but to gain more flexibility, using python is probably a better idea.

- `os.system` is not the most elegant way to use, and it is meant to be replaced by `subprocess`
- `subprocess` comes with Python standard library and allows you to spawn new processes, connect to their input/output/error pipes, and obtain their return codes
- `psutil` (python system and process utilities) is a **cross-platform** library for retrieving information on running processes and system utilization.
However, it is a third-party library

---

### Bare Minimum

the bare minimum command to kill process utilizes window's `taskkill`; 
which doesn't matter if you use `os.system` or `subprocess`

```python
import os

PROCESS = 'notepad.exe'
STATUS = 'running'  # running or not responding
CMD = r'taskkill /fi "IMAGENAME eq {}" /fi "STATUS eq {}" '.format(PROCESS, STATUS)

os.system(CMD)
```

### Using `os.system`

Now consider a more flexible case where you want to gather information about the processes like its PID,
and then proceed on ending the process. One of the downside of window shell command is that the output
can't be passed on to other command, the output is just text. Therefore, we 
output the text to a csv file which we will later process.

```python
import csv
import os
import signal
import subprocess

PROCESS = 'notepad.exe'
STATUS = 'running'  # running or not responding
TMP = r'{}/Desktop/tmp.txt'.format(os.environ['userprofile'])
CMD = r'tasklist /fi "IMAGENAME eq {}" /fi "STATUS eq {}" /fo "csv" > "{}"'.format(PROCESS, STATUS, TMP)

# output as csv format
os.system(CMD)

with open(TMP, 'r') as temp:
	reader = csv.reader(temp)
	header = next(reader)
	pids = [int(row[1]) for row in reader]

# kill process
for pid in pids:
	os.kill(pid, signal.SIGTERM) # or signal.SIGKILL 
	print('killed process with pid: {}'.format(pid))

if os.path.exists(TMP):
  os.remove(TMP)
```


### Using `subprocess`

With `subprocess`, we no longer need to create a temp file to store the output.

<script src="https://gist.github.com/leixingyu/293918b0f2bda5112c696f3cccecceec.js"></script>

### Using `psutil`

```python
import psutil

PROGRAM = r'maya.exe'

def findProcess(name):    
    procs = list()
    #Iterate over the all the running process
    for proc in psutil.process_iter():
        try:
            if proc.name() == name and proc.status() == psutil.STATUS_RUNNING:
            	pid = proc.pid            	
                procs.append(pid)
        except:
            pass      
    return procs

processes = findProcess(PROGRAM)
```

we can find process start time by using

```python
import time
startTime = time.strftime('%Y-%m-%d %H:%M:%S', time.localtime(proc.create_time()))
```

to kill process, either `kill()` or `terminate()` will work
respectfully, `SIGKILL` or `SIGTERM`

```python
p = psutil.Process(PID)
p.terminate()
p.kill()
p.wait
```

<a name="find-open-port"></a>
### Bonus: Find Open Port (for socket connection)

```python
process = psutil.Process(pid=PID)

connections = process.connections(kind='tcp4')
for c in [x for x in connections if x.status == psutil.CONN_LISTEN]:
    # gets the port number
    print('port opened: {}'.format(c.laddr[-1]))
```

<a name="find-win-title"></a>
### Bonus: Find Main Window Title

`ctypes` is a foreign function library for python, resulting a not-pythonic function

<script src="https://gist.github.com/leixingyu/7d85c0c1439b0cf9b423ba5c0d5ef184.js"></script>

### Reference

[Microsoft Doc - tasklist](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/tasklist)

[ThisPointer - Python : Check if a process is running by name and find it’s Process ID (PID)](https://thispointer.com/python-check-if-a-process-is-running-by-name-and-find-its-process-id-pid/)

[Johannes Sasongko - Win32 Python: Getting all window titles](https://sjohannes.wordpress.com/2012/03/23/win32-python-getting-all-window-titles/)

[Stack Overflow - Obtain Active window using Python](https://stackoverflow.com/questions/10266281)

[Microsoft Docs - winuser.h header](https://docs.microsoft.com/en-us/windows/win32/api/winuser/)


