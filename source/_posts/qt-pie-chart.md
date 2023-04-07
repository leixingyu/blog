---
title: "Create Stunning Pie Charts in Qt : A Step-by-Step Tutorial (with Examples)"
tags:
  - pyqt
  - template
categories:
  - tech summary
date: 2022-04-27
description: Compare and contrast the best techniques for creating stunning pie charts in Qt with our comprehensive tutorial. Our step-by-step guide provides practical examples and best practices for designing visually appealing and informative charts. Whether you're a beginner or an advanced user, our tutorial will help you master the techniques needed to create stunning pie charts in Qt.
---

## Introduction

It's time to learn new stuff again. This time, I want to implement a
data visualization component in my tool: pie charts. I had some experience
with `matplotlib` before, but I'm excited to find out that Qt
has an add-on module `QtCharts` that can integrate with Qt application.

Here are my learning results: on the left side, I created a static and simplistic design
and on the right side, a slightly more flashy, animated design.

In this blog, I will break down how they are created. They are also available
in my [guiUtil](https://github.com/leixingyu/guiUtil/blob/master/template/pieChart.py) with full code.

<table>
    <tr>
        <td>
            <img src="https://i.imgur.com/Gz2ykTF.png" alt="static" width="600px">
            <br>
            <a href="#example-1">Example #1</a>
        </td>
        <td>
            <img src="https://i.imgur.com/fR6H2PO.gif" alt="animate" width="500px">
            <br>
            <a href="#example-2">Example #2</a>
        </td>
    </tr>
</table>

## Installing `QtCharts`

`QtCharts` was first introduced as an add-on module in Qt in version 5.7,
so the [Qt.py](https://github.com/mottosso/Qt.py) for Python Qt binding doesn't
support `QtCharts` as it isn't available in PyQt and PySide.

1. check your PySide2 or PyQt5 version:`QtCore.qVersion()`. (I'm using PyQt5 version 5.15.2)

2. install the corresponding version of [PyQtChart](https://pypi.org/project/PyQtChart/) using 
    ```commandline
    pip install PyQtChart
    ```

3. finally, test import `from PyQt5 import QtCharts`

## The Basics

### Classes

[QChart](https://doc.qt.io/qt-5/qchart.html):
`QChart` refers to the main diagram, in our case the pie chart is a `QChart` object.

[QPieSeries](https://doc.qt.io/qt-5/qpieseries.html):
To draw a pie chart, we'll need one or more `QPieSeries` added to our `QChart` object,
to form the circular shape. To build other types of chart, we'll want to use
Series such as `QBoxPlotSeries`, `QCandlestickSeries`, 
`QXYSeries`, `QAreaSeries` or `QAbstractBarSeries`.

[QPieSlice](https://doc.qt.io/qt-5/qpieslice.html):
a `QPieSlice` object represents a slice inside a `QPieSeries` object.

![explain](https://i.imgur.com/vxQHPEz.jpg)

### Components

There are other components that the examples will cover.

**Value** reflects the size/span of each slice, and is added one by one in run-time
to the `QPieSeries`. We're able to retrieve useful information such
as percentage and span angle once all the data is added.

**Label** is a crucial component within the scope of the `QPieSlice`. 
Label can be displayed in different ways (Inside the slice: _LabelInsideHorizontal, LabelInsideTangential, LabelInsideNormal_
or outside the slice: _LabelOutside_ with labelArm).

**Legend** in default, is attached to `QChart`, connecting with the labels of
all the `QPieSlice`. We can separate it by detaching it or setting each individual
markers (items) of the legend.

We also defined an immutable data structure to represent the data fed
into the `QChart`, which we'll take a look shortly.

## Example 1

```python

class MySimpleChart(QtChart.QChart):

    def __init__(self, datas, parent=None):
        super(MySimpleChart, self).__init__(parent)
        self._datas = datas

        self.outer = QtChart.QPieSeries()
        self.set_outer_series()
        self.addSeries(self.outer)

    def set_outer_series(self):
        slices = list()
        for data in self._datas:
            slice_ = QtChart.QPieSlice(data.name, data.value)
            slice_.setLabelVisible()
            slice_.setColor(data.primary_color)
            slice_.setLabelBrush(data.primary_color)

            slices.append(slice_)
            self.outer.append(slice_)

        # label styling
        for slice_ in slices:
            label = "<p align='center' style='color:{}'>{}%</p>".format(round(slice_.percentage()*100, 2))
            slice_.setLabel(label)
```

We start off by subclassing `QChart`, we then would want to add a `QPieSeries`
as a container for inserting our slices (generated from our data).

A simplified process:

```python
pie_chart = QChart()
series = QPieSeries()
slice_ = QPieSlice(label, value)

series.append(slice_)
pie_chart.addSeries(series)
```
note: _slice_ is a Python built-in name so I'm against using it as variable name


### Labels and Legend

As you can see from the example above, we have some flexibility with 
label formatting (`QLabel` methods and HTML formatting), 
we can also do legend formatting to some extent. Such as alignment and marker shapes.

```python
self.legend().setAlignment(QtCore.Qt.AlignRight)
self.legend().setMarkerShape(QtChart.QLegend.MarkerShapeCircle)
```

we can even separate legend vs. slice label, so they could display different
content
(but it may be bad being practice to unlink the two)

```python
for index, marker in enumerate(self.legend().markers()):
   marker.setLabel(self._datas[index].name)
```

### Other

Adding variation can be easily done by shifting angles of all the slices.

```python
offset = 40
self.outer.setPieStartAngle(offset)
self.outer.setPieEndAngle(offset+360)
```

If the outside labels are cramped together, we can create additional spacing
by extending the label arm:
```python
slice_.setLabelArmLengthFactor(0.4)
```

## Example 2

```python
class MyChart(QtChart.QChart):

    def __init__(self, datas, parent=None):
        super(MyChart, self).__init__(parent)
        self._datas = datas

        self.legend().hide()
        self.setAnimationOptions(QtChart.QChart.SeriesAnimations)

        self.outer = QtChart.QPieSeries()
        self.inner = QtChart.QPieSeries()
        self.outer.setHoleSize(0.35)
        self.inner.setPieSize(0.35)
        self.inner.setHoleSize(0.3)

        self.set_outer_series()
        self.set_inner_series()

        self.addSeries(self.outer)
        self.addSeries(self.inner)

    def set_outer_series(self):
        slices = list()
        for data in self._datas:
            slice_ = QtChart.QPieSlice(data.name, data.value)
            slice_.setLabelVisible()
            slice_.setColor(data.primary_color)
            slice_.setLabelBrush(data.primary_color)

            slices.append(slice_)
            self.outer.append(slice_)

        # label styling
        for slice_ in slices:
            color = 'black'
            if slice_.percentage() > 0.1:
                slice_.setLabelPosition(QtChart.QPieSlice.LabelInsideHorizontal)
                color = 'white'

            label = "<p align='center' style='color:{}'>{}<br>{}%</p>".format(
                color,
                slice_.label(),
                round(slice_.percentage()*100, 2)
                )
            slice_.setLabel(label)

    def set_inner_series(self):
        for data in self._datas:
            slice_ = self.inner.append(data.name, data.value)
            slice_.setColor(data.secondary_color)
            slice_.setBorderColor(data.secondary_color)

```

### Inner and Outer Series

To create a circular/loop shape pie chart, we need to create a hole
with `setHoleSize()`. And the inner loop and outer loop are two separate series
adjacent to each other.

In the example, I used a light color for inner loop (50% blend with white)
and used solely the outer loop to display labels and values. Also, in order for
labels to be displayed properly, I added a condition so that label will
be displayed outside when the angle span is less than a threshold.

### Exploding Animation

`QPieSlice` has a built-in `hovered` signal for us to achieve the
exploding effect when mouse hovering over.

```python
    def set_outer_series(self):
       ...
       slice_.hovered.connect(partial(self.explode, slice_))

    def explode(self, slice_, is_hovered):
        if is_hovered:
            start = slice_.startAngle()
            end = slice_.startAngle()+slice_.angleSpan()
            self.inner.setPieStartAngle(end)
            self.inner.setPieEndAngle(start+360)
        else:
            self.inner.setPieStartAngle(0)
            self.inner.setPieEndAngle(360)

        slice_.setExplodeDistanceFactor(0.1)
        slice_.setExploded(is_hovered)
```

The outer loop explosion can be set using `slice_.setExploded()`; The inner loop shifting
is done by offsetting the pie start and end angle.

Also, make sure we have set `QChart.SeriesAnimations` on the `QChart`
object.

## Bonus

### Custom Data Class

```python
from collections import namedtuple

Data = namedtuple('Data', ['name', 'value', 'primary_color', 'secondary_color'])

node = Data('Node', 333, QtGui.QColor("#82d3e5"), QtGui.QColor("#cfeef5"))
connection = Data('Connection', 105, QtGui.QColor("#fd635c"), QtGui.QColor("#fdc4c1"))
other = Data('Other', 20, QtGui.QColor("#feb543"), QtGui.QColor("#ffe3b8"))
```

A helper data structure for adding items to our pie chart.

Generally, only the label/name and the value are required. 
I also assigned specific color values to achieve certain palette, 
but they can be randomly generated in the `QChart` class as well.


### Putting it Together

The last thing we need to do is add a main body and
instantiate a pie chart with some random data. We do this using a `QChartView` container.

> All the code is available at [guiUtil](https://github.com/leixingyu/guiUtil/blob/master/template/pieChart.py).

```python
import sys


class ChartView(QtWidgets.QMainWindow):

    def __init__(self, parent=None):
        super(ChartView, self).__init__(parent)
        self.setFixedSize(QtCore.QSize(700, 400))

        datas = [node, connection, other]
        chart = MySimpleChart(datas)

        chart_view = QtChart.QChartView(chart)
        chart_view.setRenderHint(QtGui.QPainter.Antialiasing)
        self.setCentralWidget(chart_view)

if __name__ == '__main__':
    global win
    app = QtWidgets.QApplication(sys.argv)
    win = ChartView()
    win.show()
    sys.exit(app.exec_())
```

## Reference

[Qt Doc - Nested Donut Example](https://doc.qt.io/qt-5/qtcharts-nesteddonuts-example.html)

[Qt Doc - QLegend](https://doc.qt.io/qt-5/qlegend.html)

[Qt Doc - QChart](https://doc.qt.io/qt-5/qchart.html)

[Qt Doc - QPieSeries](https://doc.qt.io/qt-5/qpieseries.html)

[Qt Doc - QPieSlice](https://doc.qt.io/qt-5/qpieslice.html)

[Stack Overflow - Attach colors of my choosing to each slice of QPieSeries](https://stackoverflow.com/questions/56727499)

