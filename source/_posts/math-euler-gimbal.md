title: "Euler Angles and Gimbal Lock: Things You Need to Know"
tags:
  - maya
  - math
categories:
  - 3d math
date: 2022-09-17
mathjax: true
description: Unlock the power of 3D game development with our in-depth guide to Euler angles and gimbal lock. Learn how to avoid common pitfalls and achieve smooth, realistic animations using essential 3D software math. Our comprehensive tutorial is perfect for game developers of all levels.
---

# Introduction

When talking about rotation, most of us would picture gimbal movement: an object
rotate about a set of three axes. This is the same principle of the Euler angles
presentation of rotation. It being the most popular representation for rotation,
is used across many DCCs, it is intuitive and easy for us to understand. 

![gimbal](https://upload.wikimedia.org/wikipedia/commons/thumb/d/d5/Gyroscope_operation.gif/220px-Gyroscope_operation.gif)

While an Euler angles are usually written as a list of three values representing
rotation degrees in _x_, _y_, _z_ axis. 

Scientifically it is actually denoted as
$\alpha$, $\beta$, $\gamma$ with radian as rotation unit where

- $\alpha$ is rotation about the first axis
- $\beta$ is rotation about the second axis
- $\gamma$ is rotation about the third axis


# Rotation Order

There are two major issue with Euler angles.

## Ambiguity

The target orientation is rotation order dependent, meaning even with the 
same rotation $\alpha$, $\beta$, $\gamma$ values, 
the results differs using different order of rotation.

A _zyx_ order refers to rotation about axis z, y, x in that order (think
of it as x axis being a child of y axis being a child of z axis). 
Therefore, to represent a target rotation, aside from providing rotation values
$\alpha$, $\beta$, $\gamma$, we also need to provide 
the rotation order and [Rotation Type](#rotation-type).

There are a total of 12 possible rotation orders per rotation type/group.

## Gimbal Problem

In short, the Gimbal problem is due to euler angle decomposing a (linear) rotational movement
as a product of more than one axis rotations.

Although any target orientation can be achieved using Euler rotation, the rotational
movements are sometimes undesired. This is due to a problem in Animation field known as
Gimbal Lock.

<table style="width: 900px">
  <tr>
    <td><img src="https://upload.wikimedia.org/wikipedia/commons/4/49/Gimbal_Lock_Plane.gif" alt="gimbal-lock" width="300px"></td>
    <td>Gimbal lock happens when two of the three axes are rotated to a parallel
configuration, 'locking' the system into a two-dimensional space.</td>
  </tr>
</table>

<br>

In order to "break free" of this configuration,
the system needs to input rotation along two or more axes simultaneously.

In the context of keyframe Animation, interpolation along more than one axis means the 
rotational movement will not appear linear, but in an unpredictable trajectory,
which is bad. (See video illustration [here](https://youtu.be/zc8b2Jo7mno?t=130))

### Test in Maya

You can test out the gimbal problem in Maya yourself.

Going into rotation tools and select _"Axis Orientation"_ to _"Gimbal"_, you can
now interact with the object using gimbal rotation. This rotation is what
actually get recorded by Maya and presented in the channel box.

If you go back to the _"Object"_ mode rotation, even though your rotation gizmo is
now unlocked, the gimbal problem don't just go away.
Maya is still recording the gimbal rotation in the background.
The _"Object"_ mode rotation is just an interaction tool.

<table style="width: 600px">
  <tr>
    <td><img src="https://i.imgur.com/wVrvepx.png" alt="gimbal" width="300px"></td>
    <td><img src="https://i.imgur.com/jiUWIWa.png" alt="object" width="300px"></td>
  </tr>
  <tr>
    <td>Gimbal Mode</td>
    <td>Object Mode</td>
  </tr>
</table>


## Quaternion

> Rotation lies in non-Euclidean space.

Luckily there is a more advanced representation for rotation: Quaternion, which
game engines uses internally. This is due to it being data compact (only stores four numbers) and
computational efficient (rotational movement using Euler angles usually need to be converted to rotation
matrix, which involves a lot of _sin_ and _cos_ calculation).

The bigger advantage comes from it
producing linear and predictable results (the shortest path along source and
target rotation). It is, however, very hard for users to comprehend.

A quaternion consists of a scaler part and a vector part:

  $q = [s, (x, y, z)]$ where $s, x, y, z \in R$

  or.
  $q = s + ix + ij + kz$ where $i^2 = j^2 = k^2 = ijk = -1$


# Rotation Type   

## Intrinsic and Extrinsic Rotations

**Intrinsic rotations** are elemental rotations that occur about the axes of 
a coordinate system XYZ attached to a moving body. (i.e. rotation about axis
in the current coordinate, like object space)

**Extrinsic rotations** are elemental rotations that occur 
about the axes of the fixed coordinate system xyz. (i.e. rotation about axis
in the original coordinate, like world space)

![intrinsic-vs-extrinsic-rotation](https://img-blog.csdnimg.cn/fbc97c759b134aa3b87b1d3fa6bbc567.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBASGVyZW5lXw==,size_20,color_FFFFFF,t_70,g_se,x_16)

(Fig.1: Intrinsic rotations; Fig.2: Extrinsic rotations)

## Example

Rotation calculation usually happens along with other transform type using matrix.
Here I made a matrix composition and decomposition function in Python, utilizing
a third party math library [transforms3d](http://matthew-brett.github.io/transforms3d/index.html).

As you can see, since the rotation matrix varies depending on rotation order and type,
this is reflected as one of the argument. 

<script src="https://gist.github.com/leixingyu/7496f9981c699a61ccf605c672898f9d.js"></script>

> The first character is 'r' (rotating == intrinsic), 
> or 's' (static == extrinsic).
> 
> The next three characters give the axis ('x', 'y' or 'z')
> about which to perform the rotation, 
> in the order in which the rotations will be performed.
> 
> For example the string 'szyx' specifies that the angles should be interpreted
> relative to extrinsic (static) coordinate axes, and be performed in the order:
> rotation about z axis; rotation about y axis; rotation about x axis.

## Conversion

Any extrinsic rotation can be represented in intrinsic rotation with the same
angle values but inverted order or elemental rotation.

So a $\alpha$, $\beta$, $\gamma$ with extrinsic __zyx__ order is equivalent to
the same angle with intrinsic __xyz__ order.

# Reference

[Stack Exchange - Confusion about order of rotations for Euler Angles](https://math.stackexchange.com/questions/2943074/)

[Unreal Engine Forum - What is the rotation order for Components or Bones?](https://forums.unrealengine.com/t/what-is-the-rotation-order-for-components-or-bones/286910/5)

[Stack Exchange - Proof of the extrinsic to intrinsic rotation transform](https://math.stackexchange.com/questions/1137745)

[YouTube - Euler (Gimbal Lock) Explained](https://youtu.be/zc8b2Jo7mno)

[TD Matt - Rotation Orders](http://td-matt.blogspot.com/2010/11/rotation-orders.html)

[CSDN - Understanding Euler Angles](https://blog.csdn.net/weixin_42165913/article/details/120167295)
