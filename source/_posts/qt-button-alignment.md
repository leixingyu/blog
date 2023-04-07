---
title: "Create Beautiful Qt Button Interfaces: Left-align Icon and Center-align Text"
tags:
  - pyqt
  - template
categories:
  - tech summary
date: 2021-08-21
description: Create beautiful Qt button interfaces with our comprehensive tutorial on left-aligning icons and center-aligning text. Our step-by-step guide provides practical examples and best practices for designing visually stunning interfaces that enhance user experience. Whether you're a beginner or an advanced user, our tutorial will help you master the techniques needed to create beautiful Qt button interfaces that are both functional and aesthetically pleasing.
---

### Introduction

I recently needed to display a series of buttons for my shelf tool, the problem I'm having with this is that
although every button has an icon and text label, they are displayed as different width. Using `center-align`
made it look not uniform.

Should I go with `left-align`? Well, there are buttons with relatively longer label and some with shorter ones,
so it doesn't look nice either with empty spaces on the right side.

So the solution is obvious, separate the alignment of the icon and the label: the icon stays `left-align` to
give a clear sign of broader, and the label would be `center-align` to make the width look uniform.

![three-alignment-side-by-side](https://i.imgur.com/dMOZpkH.jpg)

(Left: default center align, Center: left align, Right: custom align)

In the following section, I will demonstrate three methods of achieving this custom alignment effect:

![push-button-alignment](https://imgur.com/3G6fmVN.png)

### Overriding QPushButton `paintEvent()`

This method subclass from `QPushButton` and override the `paintEvent()` and `sizeHint()`
to extend how a button is drawn;

The alignment of content of the button is default to `center-aligned`, but
we make the pixmap to be drawn on the left-side (5px margin against the left border)

With this method, we no longer use `QIcon`, we use `QPixmap` instead; that is why
we created a custom `setPixmap()` method to our `MyButton` to give user access
to the pixmap being drawn.

<script src="https://gist.github.com/leixingyu/009a594f3d94643d39b9eee5ac6cf118.js"></script>

### Custom layout inside pushbutton

Here is another interesting approach: 
this method defaults its style to `left-aligned`, but it only contains the icon.

What about the push button label(text)?

It is actually a `QLabel` placed in the `QPushButton` layout, and being vertically
`center-aligned`; To modify the push button text, use `setText()` to the label 
inside the button layout, instead of the button.

<script src="https://gist.github.com/leixingyu/78b83e758e74cb56f9da5f076e5c4d9d.js"></script>

### Use `QProxyStyle`

I haven't personally test it because my Qt python binding doesn't have QProxyStyle included

but it's worth putting it here in case someone is able to try it.

```python
class ProxyStyle(QtWidgets.QProxyStyle):
    def drawControl(self, element, option, painter, widget=None):
        if element == QtWidgets.QStyle.CE_PushButtonLabel:
            icon = QtGui.QIcon(option.icon)
            option.icon = QtGui.QIcon()
        super(ProxyStyle, self).drawControl(element, option, painter, widget)
        if element == QtWidgets.QStyle.CE_PushButtonLabel:
            if not icon.isNull():
                iconSpacing = 4
                mode = (
                    QtGui.QIcon.Normal
                    if option.state & QtWidgets.QStyle.State_Enabled
                    else QtGui.QIcon.Disabled
                )
                if (
                    mode == QtGui.QIcon.Normal
                    and option.state & QtWidgets.QStyle.State_HasFocus
                ):
                    mode = QtGui.QIcon.Active
                state = QtGui.QIcon.Off
                if option.state & QtWidgets.QStyle.State_On:
                    state = QtGui.QIcon.On
                window = widget.window().windowHandle() if widget is not None else None
                pixmap = icon.pixmap(window, option.iconSize, mode, state)
                pixmapWidth = pixmap.width() / pixmap.devicePixelRatio()
                pixmapHeight = pixmap.height() / pixmap.devicePixelRatio()
                iconRect = QtCore.QRect(
                    QtCore.QPoint(), QtCore.QSize(pixmapWidth, pixmapHeight)
                )
                iconRect.moveCenter(option.rect.center())
                iconRect.moveLeft(option.rect.left() + iconSpacing)
                iconRect = self.visualRect(option.direction, option.rect, iconRect)
                iconRect.translate(
                    self.proxy().pixelMetric(
                        QtWidgets.QStyle.PM_ButtonShiftHorizontal, option, widget
                    ),
                    self.proxy().pixelMetric(
                        QtWidgets.QStyle.PM_ButtonShiftVertical, option, widget
                    ),
                )
                painter.drawPixmap(iconRect, pixmap)


if __name__ == "__main__":
    app = QtWidgets.QApplication(sys.argv)
    app.setStyle('fusion')
    proxy_style = ProxyStyle(app.style())
    app.setStyle(proxy_style)

    w = QtWidgets.QWidget()
    lay = QtWidgets.QVBoxLayout(w)
    icons = [
        app.style().standardIcon(standardIcon)
        for standardIcon in (
            QtWidgets.QStyle.SP_MediaPlay,
            QtWidgets.QStyle.SP_MediaPause,
            QtWidgets.QStyle.SP_MediaSeekBackward,
            QtWidgets.QStyle.SP_MediaSeekForward,
        )
    ]
    for text, icon in zip("Play Pause Backward Forward".split(), (icons)):
        button = QtWidgets.QPushButton(text)
        button.setIcon(icon)
        lay.addWidget(button)
    w.show()
    sys.exit(app.exec_())
```


### Reference

[Stack Overflow - QPushButton icon aligned left with text centered](https://stackoverflow.com/questions/44091339)

[Stack Overflow - qpushbutton icon left alignment text center alignment](https://stackoverflow.com/questions/56129402)

