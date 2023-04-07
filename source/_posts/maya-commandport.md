---
title: "Streamline Your Maya Communication Workflow (1. Command Port)"
tags:
  - maya
  - python
  - socket
  - command port
categories:
  - maya communication
date: 2021-12-17
description: Learn how to enhance your Maya workflow by running the command port and utilizing socket communication. Our step-by-step tutorial covers everything from setup to execution, providing you with the knowledge and tools to streamline your workflow and improve collaboration.
---

## Maya Connector

The end result of this two-part blog is the [Maya Connector](https://github.com/leixingyu/mayaConnector) tool.

This external standalone tool is able to send command to maya and receive
real-time feedback from maya's script editor output.

![maya-connector](https://i.imgur.com/89SJibG.gif)

## Introduction

> If you are already familiar with maya command port and socket programming, skip to
> [Part 2](https://www.xingyulei.com/post/maya-streaming/) for Maya output streaming.

Although there are a lot of tools that can run internally in Maya which facilitate
the pipeline. The capability of communicating (monitor and control) with Maya externally is very handy.

Why? we sometime don't need to interact with Maya GUI directly or have way too
many maya instances to manage. examples are:

- sending cross-application remote command to active maya sessions or syncing between different maya sessions
- batch processing on either maya standalone sessions or active maya sessions

This requires two components which I will be covering:
1. command port: sending commands to maya to execute
2. output streaming: actively listening/receiving maya outputs (next blog)

These two parts can work independently, but are powerful as a whole. Also, references
in creating [maya standalone sessions](https://www.xingyulei.com/post/maya-batch-mode/)


## Using `cmds.commandPort()`

> [cmds.commandPort()](https://help.autodesk.com/cloudhelp/2018/ENU/Maya-Tech-Docs/CommandsPython/): 
> The command port comprises a socket to which a client program may connect

Traditional socket connection provides the basic communication between
client and server.
But Maya's built-in command port provides us a very convenient way of sending command for maya
to execute. 

## Procedures

1. Open command port

The port needs to be opened on Maya's (server) end first in order for client
to send command; ideally this would be achieved by adding a `cmds.commandPort()` open
during Maya startup.


```python
import maya.cmds as cmds
port = 5050
if not cmds.commandPort(":{}".format(port), query=True):
    cmds.commandPort(name=":{}".format(port))
```

2. Send command through an external application

the command sent can be either `MEL` or `Python` command which must be specified during
the opening the port determined by `sourceType` flag

```python
import socket


# the local host
HOST = '127.0.0.1' 
PORT = 5050
ADDR = (HOST, PORT)


def sendCommand():
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect(ADDR)
    command = 'polyCube()'
    client.send(command)
    client.close()
```

### TCP Return Message

We are using **TCP** for our communication protocol to guarantee command being sent
when the connection is established successfully.

> **Important:**
At this point, we are not attempting to use `client.recv(xxx)` for data returning
from Maya, unless it's a super simple single statement. 

Trying to use `client.recv(xxx)` will result in either `None` data returned or single
return as it is the behaviour of **TCP**. [See here](https://stackoverflow.com/questions/10434525/c-socket-programming-only-receiving-one-line-at-a-time)

Also sending command line by line is not reliable as there are no context between
lines for Maya server to understand. [Unless it's super simple series of commands](https://forums.cgsociety.org/t/telnet-or-socket-no-result-back-from-maya/1730817/2)

In order to receive output, we need a custom listening server to do that which is explained in [Part 2](https://www.xingyulei.com/post/maya-streaming).

### SourceType

We are opening a default command port which takes `MEL` input,
we don't need to specify `Python` as source type, as we can just use `python("[insert command here]")`
to wrap it into `MEL`, this can be blocks of independent codes or importing
and executing python files.

### Mixed Quotation symbol

Note that when doing a source type conversion with a string type command
, quotation symbol may cause conflicts.

the solution is to replace it with backslash before quotation symbol.

```python
command = 'python("' + command.replace(r'"', r'\"') + '")'
```


## Socket Programming Introduction

It is also very helpful to know the basics of socket programming in Python
to better understand how server and client communicates, and the difference 
between TCP and UDP, which will be used in the next part.

This [tutorial](https://youtu.be/3QiPPX-KeSc) is very handy, and the following 
are a basic template for setting up client and server using `socket` and `thread`

<script src="https://gist.github.com/leixingyu/f4b4d6ca7b84cf01af5311857219295e.js"></script>

<script src="https://gist.github.com/leixingyu/5e55f23113c0280ab5ab16595bda900d.js"></script>


## Reference

[Google Group - Receiving data from commandPort](https://groups.google.com/g/python_inside_maya/c/7AgWlldtvbE/m/zUTQlAcjBgAJ?pli=1)

[Stack Overflow - c socket programming, only receiving one line at a time](https://stackoverflow.com/questions/10434525)

[CG Talk - Telnet or Socket: no result back from Maya](https://forums.cgsociety.org/t/telnet-or-socket-no-result-back-from-maya/1730817/2)

[Youtube - Python Socket Programming Tutorial](https://youtu.be/3QiPPX-KeSc)

