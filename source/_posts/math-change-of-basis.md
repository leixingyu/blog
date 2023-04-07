title: "A Key to Unlocking 3D Software: Understanding Change of Basis Matrix"
tags:
  - math
  - maya
  - unreal
  - unity
categories:
  - 3d math
date: 2022-09-25
mathjax: true
description: Discover how to manipulate matrices in 3D software to create stunning graphics for your games. Our tutorial covers the fundamentals of matrix mathematics and teaches you how to change matrices for 3D transformations. Master this essential skill Today
---


# Introduction

Different DCCs uses coordinate systems with different axis directions. For example,
Maya by default uses a Y-up right-handed system; Unity uses a Y-up left-handed system;
and Unreal uses a Z-up left-handed system.

<figure>
<img src="https://www.techarthub.com/wp-content/uploads/coordinate-comparison-chart-full.jpg" alt="dcc-coord" width="600px" >
<figcaption>(Image from: Techart Hub)</figcaption>
</figure>
<br>

- **Up Direction**: is what consider to be the up direction of the world. Moving
an object up translating in direction of positive _Y_ in Maya and Unity, but for
 Unreal, it's translating in the direction of positive _Z_.

- **Handed System**: not only affects the forward/front direction of the world, 
it also dictates the direction of rotation; for left-handed system,
positive rotation about the axis is **clockwise**, and for right-handed system, it's
**counter-clockwise** (same way as how your hand would curl, hence the name).

The nature of the coordinate system affects how an object's transformation is calculated.

The **Change of Basis** matrix transforms a vector lies in one coordinate system to
another.

## Basis Vectors

In the world of 2D, the basis vectors $x$ and $y$ of one coordinate system A,
corresponds to
$
v_x =
\begin{bmatrix}
1 \\
0 \\
\end{bmatrix}
$
and
$
v_y =
\begin{bmatrix}
0 \\
1 \\
\end{bmatrix}
$

Now, if these two basis vectors were to transfer to a different coordinate system B,
this exact set of basis vectors is written differently in that system.
This represents A's basis vectors in B's system.
$
v_x =
\begin{bmatrix}
2 \\
1 \\
\end{bmatrix}
$
and
$
v_y =
\begin{bmatrix}
-1 \\
1 \\
\end{bmatrix}
$

Correspondingly, any vector in system B could translate to system A, pre-multiply using this
change of basis matrix: 
$
M_{cob} =
\begin{bmatrix}
2 & -1\\
1 & 1\\
\end{bmatrix}
$
where the first column is the basis vector $x$ and second column is the
basis vector $y$.

And alternatively, to translate any vector in system A to system B, we pre-multiply
by the inverse of this change of basis matrix: ${M_{cob}}^{-1}$.

## Transformation

Now we know how to convert vectors between coordinate systems, how do we do it for transformation?
Here are some simple step to understand:

1. Given any vector in system B, apply a change of basis operation, so it's in system A: 

   $M_{cob} * v_B$.

2. Now we can apply a transformation represented in the same system A:

   $M_{transformA} * M_{cob} * v_B$.

3. Finally, we can transfer the end result back to system B by pre-multiplying the inverse
of the change of basis matrix:

    ${M_{cob}}^{-1} * M_{transformA} * M_{cob} * v_B$.

<br/>


## Conclusion

The frontal part ${M_{cob}}^{-1} * M_{transformA} * M_{cob}$ represents the
same transformation but in system B: $M_{transformB}$.

Alternatively, any transformation in system B: $M_{transformB}$ can be represented
as $M_{cob} * M_{transformB} * {M_{cob}}^{-1}$ in system A.

Here's the function I've written Python:

<script src="https://gist.github.com/leixingyu/f8849105d509bd52723f3e94a35cab67.js"></script>

# Example

## Maya to Unreal

The first thing to identify is the change of basis from Maya to Unreal,
as illustrated before, Maya is a Y-up left-hand system and Unreal is a Z-up
right-hand system. 

<table style="width: 500px">
<tr>
  <td colspan="6">Maya Axis Direction</td>
</tr>
  <tr>
    <td>Forward</td>
    <td>Z</td>
    <td>Up</td>
    <td>Y</td>
    <td>Right</td>
    <td>-X</td>
  </tr>
<tr>
  <td colspan="6">Unreal Axis Direction</td>
</tr>
  <tr>
    <td>Forward</td>
    <td>-Y</td>
    <td>Up</td>
    <td>Z</td>
    <td>Right</td>
    <td>-X</td>
  </tr>
</table>

The change of basis can be represented as such
  $
  M_{cob} = 
  \begin{bmatrix}
  1 & 0 & 0 & 0\\
  0 & 0 & -1 & 0\\
  0 & 1 & 0 & 0\\
  0 & 0 & 0 & 1
  \end{bmatrix}
  $

```python
MAYA_TO_UNREAL = np.array([
    [1, 0, 0, 0],
    [0, 0, -1, 0],
    [0, 1, 0, 0],
    [0, 0, 0, 1]
], dtype=int)
```

## Maya to Unity

Same thing for Maya to Unity conversion, a Y-up left-hand system to a Y-up right-hand
system; the $y$ and $z$ are respectively inverted between the two systems, therefore:

<table style="width: 500px">
<tr>
  <td colspan="6">Maya Axis Direction</td>
</tr>
  <tr>
    <td>Forward</td>
    <td>Z</td>
    <td>Up</td>
    <td>Y</td>
    <td>Right</td>
    <td>-X</td>
  </tr>
<tr>
  <td colspan="6">Unity Axis Direction</td>
</tr>
  <tr>
    <td>Forward</td>
    <td>Z</td>
    <td>Up</td>
    <td>Y</td>
    <td>Right</td>
    <td>X</td>
  </tr>
</table>

The change of basis can be represented as such
  $
  M_{cob} = 
  \begin{bmatrix}
  -1 & 0 & 0 & 0\\
  0 & 1 & 0 & 0\\
  0 & 0 & 1 & 0\\
  0 & 0 & 0 & 1
  \end{bmatrix}
  $

```python
MAYA_TO_UNITY = np.array([
    [-1, 0, 0, 0],
    [0, 1, 0, 0],
    [0, 0, 1, 0],
    [0, 0, 0, 1]
], dtype=int)
```


# Reference

[YouTube - Change of basis | Chapter 13, Essence of linear algebra](https://youtu.be/P2LTAUO1TdA)

[Unreal Engine Forum - "Forward" in unreal engine, Which is it?](https://forums.unrealengine.com/t/forward-in-unreal-engine-which-is-it/93115)

[Dario Mazzanti - Change of Basis](https://www.dariomazzanti.com/uncategorized/change-of-basis/)
