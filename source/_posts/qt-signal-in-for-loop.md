---
title: "Best Practices Connecting Qt Signal in For Loop"
tags:
  - pyqt
categories:
  - tech summary
date: 2021-04-17
description: Improve your Qt programming skills with our best practices for connecting signals in a for loop. Our comprehensive tutorial provides practical examples and step-by-step guidance for creating efficient, scalable, and reliable code. Whether you're a beginner or an experienced Qt developer, our tutorial will help you master the techniques needed to optimize your signal handling and improve your programming workflow.
---

### Introduction

We often need to create ui elements on the fly, sometimes we do it in something like a for loop.
An example would be creating a series of `QPushButton` and connect them to a function through 
different argument values. An example is shown below:

```python
class Demo(QtWidgets.QWidget):
    def __init__(self, parent=None):
        super(Demo, self).__init__(parent)

        # initialization object
        layout = QtWidgets.QVBoxLayout()

        for index in range(6):
            pushbutton = QtWidgets.QPushButton('button {}'.format(index))
            pushbutton.clicked.connect(lambda: self.trigger(index))
            layout.addWidget(pushbutton)

        self.setLayout(layout)

    @staticmethod
    def trigger(index):
        print('button {} clicked'.format(index))
```

Here I created six `QPushButton` and when I click them it should output which button is being clicked. But if you
run this script and try to click each button it will always output "button 5 clicked" (aka, the last button).
It is safely to assume that the argument passed during the for loop always result in the last index.

### Explanation

Based on a kind response from stackoverflow: lambdas do **not** store the value of button when it is defined. 
The code describing the lambda function is parsed and compiled but not executed until you actually 
call the lambda. Therefore, when a button is clicked, the current value of that variable is used (the last index).

What's the solution?

### Lambda with solid variable

Passing solid variable to the lambda

```python
pushbutton.clicked.connect(lambda _, i=index: self.trigger(index=i))
```

Note that we created another temporary variable before index as 
the first argument passed in the lambda will always return as `False`.
Because Qt defines the signal `QAbstractButton.clicked` to take a single 
argument with a default value of `False`. Since your lambda is handling that signal, 
it gets called with `False`. 

### Partial approach

Use `functools.partial` also works

```python
from functools import partial
pushbutton.clicked.connect(partial(self.trigger, index))
```

Note that in some cases where wrappers are being used in `trigger` function, it could be trickier to use this as oppose to `lambda`

### Reference

[Stack Overflow - First lambda capture of local variable always False](https://stackoverflow.com/questions/27953895)

[Stack Overflow - Connecting multiples signal/slot in a for loop in pyqt](https://stackoverflow.com/questions/46300229)

