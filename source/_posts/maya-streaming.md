---
title: "Streamline Your Maya Communication (2. Streaming Output)"
tags:
  - maya
  - python
  - socket
  - threading
categories:
  - maya communication
date: 2022-1-9
description: Optimize your Maya workflow with our comprehensive tutorial on console callback and socket communication. Our step-by-step guide provides practical examples and best practices to help you streamline your production pipeline and increase productivity. Whether you're a beginner or an advanced user, our tutorial will help you master the techniques needed to improve your Maya skills
---

### Maya Connector

I've combined the two parts together and created the end result: [Maya Connector](https://github.com/leixingyu/mayaConnector) tool.

The tool will utilize command port to send open streaming command,
which establishes the connection and enables the callback.
The output is being sent back to the tool and the output is updated in the GUI.

![maya-connector](https://i.imgur.com/89SJibG.gif)


### Introduction

> Make sure to see [Part 1](https://www.xingyulei.com/post/maya-commandport/) for more context in socket programming and maya command port

Like mentioned in the previous blog, this chapter is independent for enabling 
output streaming, but would work even better with command port to send command and get active
response from Maya.

This kind of log streaming feature can be helpful not only for monitoring 
but for catching errors like when Maya crashes, or to connect with an IDE similar to
PyCharm's plugin [MayaCharm](https://github.com/cmcpasserby/MayaCharm) and Sublime's plugin 
[MayaSublime](https://github.com/justinfx/MayaSublime/)

### Listening Server Separation

I've been looking at Maya's command port to see if the return message would be
sufficient for debugging log, as command port already is a server.

But it turns out that this approach is not recommended by many, as there's little 
control on the return message as the TCP communication returns one message per
command, more explained in the previous blog.

The correct way suggested, and widely used is to stream Maya script output back
to the external application.

![connection graph](https://i.imgur.com/MtvlyMe.png)

As shown in the graph, two separate communications are established instead of utilizing
just the command port.

1. external application (client) send command to Maya's command port (server) where Maya
would process normally. This is achieved through TCP to ensure no loss in sending command.
2. Maya's `MCommandMessage` callback triggers (client) send the latest output result to external application's listening
port (server). This is achieved through UDP to keep the connection fast and easy, but may
result in lost in feedbacks (which isn't that important)

### Procedures

to establish listening server, need to use a different port number than the command port number
if we are also connecting to it.

```python
import socket


SERVER = '127.0.0.1'
PORT = 5051
ADDR = (SERVER, PORT)

server = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server.bind(ADDR)

while True:
    data = server.recvfrom(1024)
    message = data[0]
    address = data[1]

    print(message)
```

On the maya client side, there are two parts:
1. setup `MCommendMessage` callback
2. stream the data to the listening server

```python
import socket

import maya.OpenMaya as om

SERVER = '127.0.0.1'
PORT = 5051

if 'STREAM_CALLBACK' in globals():
    try:
        om.MMessage.removeCallback(STREAM_CALLBACK)
    except RuntimeError:
        pass

STREAM_CALLBACK = None


def openStream(addr=(SERVER, PORT)):
    global STREAM_CALLBACK
    print("Enable Streaming ScriptEditor at ({}:{})\n".format(addr[0], addr[1]))
    STREAM_CALLBACK = om.MCommandMessage.addCommandOutputCallback(streamToConsole, addr)


def closeStream():
    global STREAM_CALLBACK
    print("Disable Streaming ScriptEditor\n")
    om.MMessage.removeCallback(STREAM_CALLBACK)
    STREAM_CALLBACK = None
```

borrowed some code from [MayaSublime]() for stringIO

```python
def streamToConsole(msg, mtype, addr):
    buf = StringIO()
    buf.seek(0)
    buf.truncate()
    buf.write(msg)
    buf.seek(0)
    # start with trying to send 8kb packets
    bufsize = 8*1024
    # loop until the buffer is empty
    while True:
        while bufsize > 0:
            # save our position in case we error and need to roll back
            pos = buf.tell()

            part = buf.read(bufsize)
            if not part:
                # buffer is empty. Nothing else to send
                return
            try:
                client.sendto(part, addr)

            except Exception as e:
                if e.errno == errno.EMSGSIZE:
                    # we have hit a message size limit. 
                    # scale down and try the packet again
                    bufsize /= 2
                    buf.seek(pos)
                    continue
                # some other error
                raise
            # message sent without error
            break
```

#### Message Type

Command message type support:

```python
if mtype == om.MCommandMessage.kWarning:
    buf.write('# Warning: ')
    buf.write(msg)
    buf.write(' #\n')
elif mtype == om.MCommandMessage.kError:
    buf.write('// Error: ')
    buf.write(msg)
    buf.write(' //\n')
elif mtype == om.MCommandMessage.kResult:
    buf.write('# Result: ')
    buf.write(msg)
    buf.write(' #\n')
else:
    buf.write(msg)
```

#### Logging GUI

Like a logger, instead of sending output string to standard output, I created
a `QPlainTextEdit` and use `insertPlainText(string)` to display it.
We opened a thread to receive data, so that we still have control over the main tool, and
can send command to maya if wanted.

**Note**: we shouldn't update the GUI in the listening thread, instead we emit
a custom signal and pass the return message as argument,
we then create a custom slot to handle GUI update.

so instead of doing:
```python
while True:
    data = server.recvfrom(1024)
    message = data[0]
    self.ui_text_edit.insertPlainText(message)
```


we should be doing:
```python
while True:
    data = server.recvfrom(1024)
    message = data[0]
    self.message_received.emit(message)
```

#### Clean up

During application close, we need to do the following clean up: 
1. send command to Maya to remove the callback 
2. close the tool's UDP listening server
3. end the thread to the listening server


### References

[Maya Help - OpenMaya.MCommandMessage Class Reference](https://help.autodesk.com/view/MAYAUL/2016/ENU/?guid=__py_ref_class_open_maya_1_1_m_command_message_html)

[Google Groups - Extracting data from Output Window](https://groups.google.com/g/python_inside_maya/c/pp_E7rCs7d0)

[Github - MayaCharm](https://github.com/cmcpasserby/MayaCharm)

[Github - MayaSublime](https://github.com/justinfx/MayaSublime/)


