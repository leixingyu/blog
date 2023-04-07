---
title: "A Simple Guide to Resizing Pixmap in PyQt"
tags:
  - pyqt
categories:
  - tech summary
date: 2021-08-22
description: Resize pixmap in PyQt with ease using our simple guide. Our step-by-step tutorial provides practical examples and best practices for resizing pixmap in PyQt and improving the user experience of your PyQt applications. Whether you're a beginner or an advanced user, our guide will help you master the techniques needed to resize pixmap in PyQt and enhance your skills.
---

### Introduction

During my attempts to create custom alignment push buttons, I encountered an issue with icon having
jagged looking (even if with low resolution).

I use a custom `paintEvent()` drawing `QPixmap`, and this happens when I use `scaled()` to resize my pixmap.

I noticed the icon does not have the jagged look with the built-in `setIcon()` and `setIconSize` in 
`QPushButton`. So I know there's some wrong with my approach.

### Example

To really show out the difference, I first reduce the resolution of my image.

```python
low_rez = QtCore.QSize(40, 40)
high_rez = QtCore.QSize(400, 400)
pixmap = QtGui.QPixmap(path)

pixmap = pixmap.scaled(low_rez)
```

I then increase the resolution back to normal. The default scale uses `FastTransformation`
```python
pixmap = pixmap.scaled(high_rez)
```

This is the result:

![pixmap-not-smooth](https://imgur.com/8KiYjlW.png)

### The Solution

I've searched many forums and people were all saying: enable the `SmoothTransformation`, I tried but didn't work.

Later on I found out that the Qt translation to Python has a mis-match keyword argument:
so instead of `transformMode=Qt.SmoothTransformation`, it should actually be `mode=Qt.SmoothTransformation`

So here's the solution:
```python
pixmap = pixmap.scaled(
    high_rez,
    aspectRatioMode=QtCore.Qt.KeepAspectRatio,
    mode=QtCore.Qt.SmoothTransformation
)
```

and result:

![pixmap-smooth](https://imgur.com/TIjyqkm.png)

### Extra

I also found post saying it might be some settings with the `QPainter`, but it is not the issue for me.
```python
painter = QtGui.QPainter(self)
painter.setRenderHint(QtGui.QPainter.Antialiasing, True)
painter.setRenderHint(QtGui.QPainter.SmoothPixmapTransform, True)
painter.drawPixmap(self.pixmap)
```


### Reference

[Qt Documentation - QPixmap Class](https://doc.qt.io/qt-5/qpixmap.html)

