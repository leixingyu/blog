title: "Transform Your Workflow: Learn How to Convert Matrices between DCCs"
tags:
  - unreal
  - unity
  - maya
  - math
categories:
  - learning log
date: 2022-10-22
mathjax: true
description: Learn the essential 3D software math skills for game development with our comprehensive guide. In this post, we cover the intricacies of converting and transforming matrices, providing step-by-step instructions and real-world examples in Unity, Maya and Unreal Engine
---

# Introduction

Have you noticed the different transform values when one asset is imported
to a different package?

When we do .fbx importing and exporting, the plugin does this conversion for
us. FBX is very reliable and hassle-free, but how does the math works?

> **Important:** Before continuing, I highly recommend reading these previous posts
> - [Euler Angles](https://www.xingyulei.com/post/math-euler-gimbal/)
> - [Change of Basis](https://www.xingyulei.com/post/math-change-of-basis/)

## Transformation Value

To demonstrate the different transform between DCCs,
here I have some _'axis'_ object placed in Maya, Unity and Unreal.

<img src="https://i.imgur.com/6z0QneT.jpg" alt="3in1" width="800px">

When inspecting the 'axis' located on the top-right, the transformation
values are seen as follow:

<table style="width: 800px">
  <tr>
    <th></th>
    <th>translation</th>
    <th>rotation</th>
    <th>scale</th>
  </tr>
  <tr>
    <td>Maya</td>
    <td>x: -1.31, y: 0.33, z: 0.19</td>
    <td>x: -335, y: -12, z: 98</td>
    <td>x: 1, y: 1, z: 1</td>
</tr>
  <tr>
    <td>Unity</td>
    <td>x: 1.31, y: 0.33, z: 0.19</td>
    <td>x: 7, y: -27, z: -102</td>
    <td>x: 1, y: 1, z: 1</td>
  </tr>
  <tr>
    <td>Unreal</td>
    <td>x: -1.31, y: 0.19, z: 0.33</td>
    <td>x: 149, y: 75, z: 123</td>
    <td>x: 1, y: 1, z: 1</td>
  </tr>
</table>
<br/>

Usually, for translation and scale there is a pattern to be found, usually
the operation involves swapping values between axes, and flipping them. For
rotation, however, the values don't seem to have connections between one
another, so how can we compute them?

The fundamental reason for all of this is that even though all DCCs use a 
Cartesian coordinate system, they have different **axis direction** and **rotation order**.

# Methods Overview

The method I use are all fundamental matrix calculation.

1. From the source package, retrieve translation, rotation and scale ($t_{source}, r_{source}, s_{source}$)
and compose a transformation matrix: $M_{source}$


2. Convert the transformation matrix to the target software coordinate system
using change of basis operation: $M_{target} = M_{cob} * M_{source} * {M_{cob}}^{-1}$


3. Decompose the transformation matrix: $M_{target}$ and re-order the
target translation, rotation and scale value such ($t_{target}, r_{target}, s_{target}$)

## Composition and De-composition

It is helpful to re-visit the transform matrix calculation, but we will later
use a math library to handle them.

- **Translation**:

  a translating along $t = <d_x, d_y, d_z>$ can be thought as multiplying by
  the translation matrix:
  $
  M_{translation} = 
  \begin{bmatrix}
  1 & 0 & 0 & d_x\\
  0 & 1 & 0 & d_y\\
  0 & 0 & 1 & d_z\\
  0 & 0 & 0 & 1
  \end{bmatrix}
  $

- **Scale**:

  a scaling upon $s = <\beta_x, \beta_y, \beta_z>$ can be thought as multiplying
  by the scale matrix:
  $
  M_{scale} = 
  \begin{bmatrix}
  \beta_x & 0 & 0 & 0\\
  0 & \beta_y & 0 & 0\\
  0 & 0 & \beta_z & 0\\
  0 & 0 & 0 & 1
  \end{bmatrix}
  $

- **Shear**: good to know, but not very commonly use;
$sh = <sh_x, sh_y, sh_z>$ represents shear along _x_, _y_ and _z_ axis;

  $
  M_{sh_x} = 
  \begin{bmatrix}
  1 & 0 & 0 & 0\\
  h_{yx} & 1 & 0 & 0\\
  h_{zx} & 0 & 1 & 0\\
  0 & 0 & 0 & 1 \\
  \end{bmatrix}
  $
  $
  M_{sh_y} = 
  \begin{bmatrix}
  1 & h_{xy} & 0 & 0\\
  0 & 1 & 0 & 0\\
  0 & h_{zy} & 1 & 0\\
  0 & 0 & 0 & 1
  \end{bmatrix}
  $
  $
  M_{sh_z} = 
  \begin{bmatrix}
  1 & 0 & h_{xz} & 0\\
  0 & 1 & h_{yz} & 0\\
  0 & 0 & 1 & 0\\
  0 & 0 & 0 & 1
  \end{bmatrix}
  $

  > <img src="https://i.stack.imgur.com/90xMH.jpg" alt="shear" width="400px">
  >
  > A **2D** shearing operation

- **Rotation**: a rotation can be thought as a combination of shearing and scaling,
a visual demonstration can be found [here](https://youtu.be/E3Phj6J287o).
  
  A rotation along _x_, _y_ and _z_ axis can then be represented as such:

  $
  M_{r_x} = 
  \begin{bmatrix}
  1 & 0 & 0 & 0\\
  0 & cos\theta & -sin\theta & 0\\
  0 & sin\theta & cos\theta & 0\\
  0 & 0 & 0 & 1
  \end{bmatrix}
  $
  $
  M_{r_y} = 
  \begin{bmatrix}
  cos\theta & 0 & sin\theta & 0\\
  0 & 1 & 0 & 0\\
  -sin\theta & 0 & cos\theta & 0\\
  0 & 0 & 0 & 1
  \end{bmatrix}
  $
  $
  M_{r_z} = 
  \begin{bmatrix}
  cos\theta & -sin\theta & 0 & 0\\
  sin\theta & cos\theta & 0 & 0\\
  0 & 0 & 1 & 0\\
  0 & 0 & 0 & 1
  \end{bmatrix}
  $

# Real Examples

I used `numpy.ndarray` to store matrix, and used a third-party library
for computing the composition and de-composition of transformation matrix.

<script src="https://gist.github.com/leixingyu/7496f9981c699a61ccf605c672898f9d.js"></script>

## Computation

Now let's put our code to the test:

Here are the transformation values I have in maya:

```python
maya_t = [-1.307, 0.331, 0.188]
maya_r = [-335, -12, 98]
maya_s = [1, 1, 1]
```

## Maya to Unity

From the previous post, I've computed the change of basis matrix for Unreal

```python
MAYA_TO_UNITY = np.array([
    [-1, 0, 0, 0],
    [0, 1, 0, 0],
    [0, 0, 1, 0],
    [0, 0, 0, 1]
], dtype=int)
```

Now followed by our methods:

```python
maya_matrix = compose_matrix(maya_t, maya_r, maya_s, order='sxyz')
unity_matrix = change_xform(maya_matrix, MAYA_TO_UNITY)
unity_t, unity_r, unity_s = decompose_matrix(unity_matrix, order='szxy')
```

Result:
```python
# transform in Unity
translation = [1.307, 0.331, 0.188] 
rotation = [7.3413883536944935, -26.641453221201548, -102.41010358310075] 
scale = [1.0, 1.0, 1.0]
```

**Important**: for rotation since the returned values corresponds to the first, second
and third rotation angle specified in the rotation order:
which is in 'zxy', we need to swap values to re-order it as 'xyz'.

## Maya to Unreal

Same goes for Unreal, the change of basis matrix is:

```python
MAYA_TO_UNREAL = np.array([
    [1, 0, 0, 0],
    [0, 0, -1, 0],
    [0, 1, 0, 0],
    [0, 0, 0, 1]
], dtype=int)
```

Then comes our conversion method:

```python
maya_matrix = compose_matrix(maya_t, maya_r, maya_s, order='sxyz')
unreal_matrix = change_xform(maya_matrix, MAYA_TO_UNREAL)
unreal_t, unreal_r, unreal_s = decompose_matrix(unreal_matrix, order='sxyz')
```

Result:
```python
# transform in Unreal
translation = [-1.307, 0.188, 0.331] 
rotation = [149.05728077227823, 75.61041143412538, 123.21509334477231] 
scale = [1.0, 1.0, 1.0]
```

**Caveat:** I need to inverse the value of translation _z_, the reason I haven't figured out yet,
but translation is hardly our concern since we just need to flip axes between Maya values;
The rotation value is the important one.



# Reference

[Transform3d API](https://matthew-brett.github.io/transforms3d/index.html)

[ThreeJS - 
Convert from one coordinate system to another?](https://discourse.threejs.org/t/convert-from-one-coordinate-system-to-another/13240)

[Tech Art Hub - A Practical Guide to Unreal Engine 4's Coordinate System](https://www.techarthub.com/a-practical-guide-to-unreal-engine-4s-coordinate-system/)
