---
title: "Make Your Qt Button More Dynamic with Double-Click and Hover Effects"
tags:
  - pyqt
  - template
categories:
  - tech summary
date: 2021-08-08
description: Make your Qt button more dynamic with our comprehensive tutorial on double-click and hover effects. Our step-by-step guide provides practical examples and best practices for adding interactivity and visual feedback to your button widgets. Whether you're a beginner or an advanced user, our tutorial will help you master the techniques needed to make your Qt button more engaging and intuitive.
---

## Introduction

I want to showcase some examples that I used to create push button with 
custom behaviours.

![double](https://i.imgur.com/e2LbhNJ.gif)

(Double click button)


![hover](https://i.imgur.com/FCzg1MQ.gif)

(Button hover effect)

## Double Click

### Problem with `MouseButtonDblClick`

Hey, I thought this would be easy, since Qt offers a built-in event
type: `QEvent.MouseButtonDblClick`.
But the issue is it couldn't distinguish a single click vs. a double click.
Which means, the single click event will also be invoked when double-clicked.

### Solution using timeout

Subclass `QPushButton` and override `eventFilter()`

<script src="https://gist.github.com/leixingyu/0a38a3d77a848f51cfb3588a3ac19627.js"></script>

### Using `eventFilter()`

`eventFilter()` takes three argument, a `QObject` that the filter is installed
on, a `QObject` that is being watched, and a reference to a `QEvent` type object
to be filtered.

Be sure to add `self.installEventFilter(self)` so to override event filtering.
Now it's only the matter of adding condition logic to filter out different
event types such as `QEvent.MouseButtonPress` and `QEvent.MouseButtonDblClick`
with `event.type()`

We can also filter which button is being used using `event.button()`.

### timeout

Using a built-in timer from Qt `QTimer()`, we are able to fire off timer
events between our clicks.

A `timeout` signal is fired after a certain interval we defined using `setInterval()`.
the `timeout()` method is then used to determine what custom signal to emit.

### Custom Signal

Note that I created three custom signals for three different clicking type
I want to register.

```python
right_clicked = QtCore.Signal()
left_clicked = QtCore.Signal()
double_clicked = QtCore.Signal()
```

This enhanced the usability of the click behaviour, meaning that this extended
`QPushbutton` can tell whether the user did a single left or right click or a
double click.

In my example above, I wrapped all the click events in the class itself.
But in the main application, we can also instantiate `MyButton` and connect click event
to method of our choice.

```python
btn = MyButton('Button1')
btn.doubled_clicked.connect(func1)
btn.right_clicked.connect(func2)
```

## Hover Effect

As a bonus, I want to include a "Button" with custom hover effect, but it is
technically a `QLabel` object with extended functionality, which is being used
in my [SnapTool](https://github.com/leixingyu/snapTool).

```python
class HoverBtn(QtWidgets.QLabel):

    clicked = QtCore.Signal(QtCore.QObject)

    def __init__(self, parent):
        super(HoverBtn, self).__init__(parent)

        style = """
        QFrame{
            border-radius: 25px;
            border-width: 2px;
            border-style: solid;
            border-color: rgb(20, 20, 20);
            background-color: rgb(170, 170, 170);
        }
        """
        self.installEventFilter(self)
        self.setStyleSheet(style)

    def eventFilter(self, obj, event):
        if event.type() == QtCore.QEvent.Enter:
            self.set_outline(1)
            return True
        elif event.type() == QtCore.QEvent.Leave:
            self.set_outline(0)
            return True

        if event.type() == QtCore.QEvent.MouseButtonPress \
                and event.button() == QtCore.Qt.LeftButton:
            self.clicked.emit(self)
            return True

        return False

    def set_outline(self, status):
        """
        Update widget stylesheet when highlight
        """
        style = self.styleSheet()

        border_pattern = r'border-color\: rgb\(\d+, \d+, \d+\)'
        dark = 'border-color: rgb(20, 20, 20)'
        light = 'border-color: rgb(21, 255, 9)'

        if status == 1:
            style = re.sub(border_pattern, light, style)
        elif status == 0:
            style = re.sub(border_pattern, dark, style)

        self.setStyleSheet(style)
```

The hover effect is achieved by swapping stylesheet properties, and the rest
of the event filtering is very similar to the previous double click button.

### Reference

[Qt Centre - Double Click Capturing](https://www.qtcentre.org/threads/7858-Double-Click-Capturing)

[Stack Overflow - Pyqt Mouse MouseButtonDblClick event](https://stackoverflow.com/questions/19247436)

