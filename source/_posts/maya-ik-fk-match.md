---
title: "Automate IK FK Matching in Maya with 2 Simple Steps"
tags:
  - python
  - maya
categories:
  - 3d math
date: 2022-02-27 
mathjax: true
description: Learn how to create realistic and flexible character rigs for games and animations with our tutorial on IK/FK matching in Maya. Our step-by-step guide covers everything from setting up your rig to implementing IK/FK switching for smooth animation. Whether you're a seasoned pro or just starting out, our tips and tricks will help you take your rigging skills to the next level. Join our community of artists and animators and start creating dynamic characters today!
---

## Introduction

IK/FK matching refers to the ability to match any IK pose to FK pose and
vice versa. But how does it work? 

I'm going to summarize the basic approaches I learned along the way. 
One of them is math-based, and the other being Maya operational-based, 
which utilizes Maya's built-in constraint nodes,
this is more straight-forward and easy as it does the calculation for you.

Since we use controllers to drive joint movement in rigs,
the general idea of a IK/FK matching is to reverse this process,
and find the transformation of a controller based off the joint's transformation. 

>Note: 
There's no one easy way to deal with all custom rig setups, this is
due to operations like parenting, orienting joint and freezing transformation.
Transformation offsets occurs between joint and controller, even in world space.
I'll cover these more complex scenarios as references [here](#further-reading).

## FK Matching

<img src="https://i.imgur.com/q9kcckN.gif" alt="fk-match" width="800px">

FK is all about matching rotation, given that FK controller
constraints the corresponding joint's orientation.

Thus, in reverse, given the result joint's transform,
we should be able to find the FK controller's rotation. And 
knowing how one of them works can translate to all the other joints and FKs
(respectively, shoulder, elbow and wrist in arm and clavicle, knee and ankle in leg).

<a name="fk-script"></a>
### Script

1. obtain the **world space** rotation of the joint while
it is still in IK mode

    ```python
    # in IK mode
    rotation = cmds.xform(jnt, ro=1, ws=1, q=1)
    ```

2. switch to FK mode and apply the **world space** rotation onto the FK controller
    ```python
    # in FK mode
    def snap_fk_to_jnt(ctrl, target_rotation):
        cmds.xform(ctrl, ro=target_rotation, ws=1)
    ```

<a name="fk-operation"></a>
### Maya Operation

We can utilize Maya's constraint to match rotation.
Such as, in a three-chain setup, we could let IK joints to rotation constraint
the FK controllers without maintaining offset. But for a single-chain setup
with both FK and IK controllers sharing the joint,
doing so will cause cycle evaluation. 

To get around this, we use a temporary empty transform node
that is **relatively** parented under the joint. This temporary
node (the match node) will be used to constraint the FK controller, and once the
matching completes, we can safely delete the constraint and the match node itself.

## IK Matching

<img src="https://i.imgur.com/iJIrrML.gif" alt="ik-match" width="800px">

IK Matching matches IK controller
to the joint's transform in FK mode.
For IK handle, we need to match both the top joint's (wrist or ankle) translation and rotation value;
For IK pole vector, we only need to match the mid-joint's (elbow or knee) translation.

We already figured out how to acquire and match rotation in FK matching,
which can be directly used in matching IK handle, so we'll focus on how to
match translation below.

<a name="ik-script"></a>
### Script

1. obtain the **world space** translation of the joint while
it is still in FK mode

    ```python
    # in FK mode
    translation = cmds.xform(jnt, t=1, ws=1, q=1)
    ```

2. switch to IK mode and apply the **world space** translation onto the IK Handle
    ```python
    # in IK mode
    def snap_ik_to_jnt(ik, target_pos):
        cmds.xform(ik, t=target_pos, ws=1)
    ```

<a name="ik-operation"></a>
### Maya Operation

Similar to FK matching, we create a match node that inherits the
target joint's transformation in FK mode; 
Then we use it to parent constraint the IK handles (position constraint the IK pole)
without maintain offset. 
Once the IKs snaps to the target position, we then safely delete the constraint and the match node.

### Pole Vector: One Step Further

In the example above, IK pole vector will directly snap on top of the elbow or knee joint.
Doing so will achieve the IK matching effect, but we often want the pole vector to extend
a little further outwards, this gives more room for animators and avoids joint flipping.

To extend the pole vector outwards, we need to acquire the directional vector
from mid-point of the root and top joint of the rig chain to the mid-joint.

<img src="https://i.imgur.com/pIPPm2t.png" width="500px" alt="ik-pole">

It is easier to achieve this through script:

1. convert positional value to vector as Maya doesn't support operation
on position (represented as `list[3]`)

    ```python
    root_vec = Vector3(root_pos)
    mid_vec = Vector3(mid_pos)
    top_vec = Vector3(top_pos)
    
    # alternatively, use maya's built-in MVector
    from maya.api import OpenMaya as om
    vec = om.MVector(pos[0], pos[1], pos[2])
    ```

3. find the mid-point between the root joint and the top joint

    ```python
    mid_point = (root_vec + top_vec) * 0.5
    ```

4. find the directional vector from mid-point to the mid-joint

    ```python
    pole_dir = mid_pos - mid_point
    ```

5. the pole vector's target position can be found by extending this directional
vector beyond the mid-joint position with an arbitrary multiplier

    ```python
    pole_pos = mid_pos + (pole_dir * multiplier)
    ```

## Further Reading

The above matching logics aren't bulletproof. What???!!

<img src="https://i.imgur.com/qgvTH55.gif" width="600px" alt="wrong">

Yeah, because not long after releasing my [Snap tool](https://github.com/leixingyu/snapTool),
Squirrel Daph found the tool not behaving correctly and
kindly sent me the rig for further testing. 

After some troubleshooting, I found that because I built the tool around my auto-generated rig,
so there are many things I didn't account for.
The following sections aim to overcome some of my Snap tool's previous limitations.

### Transformation Matrix

Transformation matrix are commonly used in Computer Graphics,
it can be used to represent an object's transformation in 3D world; It
is an extremely useful knowledge to have in terms of rigging.

Maya has many built-in ways get the matrix representation of a node's transformation.

```python
def get_matrix(node, is_world=1):
    return om.MMatrix(
        cmds.xform(node, q=1, matrix=1, ws=is_world, os=not is_world)
    )
```

Converting the Maya matrix representation `MMatrix` to `MTransformationMatrix`,
gives access to more built-in functions including [decomposition](#decomposing-matrix).

```python
mxform_matrix = om.MTransformationMatrix(mmatrix)
```

Or construct by using function set `MFnTransform`:

```python
def get_transform_matrix(dag_node):
    """
    Get the local transformation matrix of a given dag node

    :param dag_node: om.MObject. input maya dag node
    :return: om.MTransformationMatrix. local transformation matrix
    """
    fn_transform = om.MFnTransform(dag_node)
    return fn_transform.transformation()
```

The differences between `om.MMatrix` and `om.MTransformationMatrix`:
- `om.MMatrix` constructed by `cmds.xform` has access to both local and world
transformation matrix, but unable to track rotation greater than 360 degrees.
- `om.MTransformationMatrix` using `MFnTransform` only gets local transformation
matrix, but it stores rotation information greater than 360 degrees.

### Translation Offset

When there's world space offset between IK controller and joint, 
we would want to account for the offset when calculating the IK result position.

```python
jnt_pos = Vector(cmds.xform(jnt, ws=1, q=1, t=1))
ik_pos = Vector(cmds.xform(ik, ws=1, q=1, t=1))
offset = jnt_pos - ik_pos

ik_target_pos = jnt_target_pos - offset
```

### Rotation Offset 

Rotational offset can be caused by axis in-alignment during constraints,
we'll need to use matrix multiplication to find it instead of simple vector math.

<img src="https://i.imgur.com/czn3XY1.gif" width="1000px" alt="axis-align">

Example of rotation offset between joint and controller

#### Pre-multiplying Matrix

> Important: 
> - the order of matrix multiplication matters
> - to get a unit matrix, instead of dividing matrix, we
> multiply its inverse matrix

Joint transformation is the result of controller constraints, in which 
the constraint matrix serves as a pre-multiplication matrix.

$$
M_{joint} = M_{const} * M_{ctrl}
$$

We are then able to get the constraint matrix by post-multiplying the inverse
of controller matrix.

$$
{M_{joint}} * M_{ctrl}^{-1} = M_{const} * M_{ctrl} * M_{ctrl}^{-1}
$$

$$
M_{const} = {M_{joint}} * M_{ctrl}^{-1}
$$

In the function below, the result matrix argument would be the joint matrix,
and source matrix argument would be the controller matrix.

```python
def get_pre_mult_matrix(result_mat, source_mat):
    """
    Get pre-multiplying matrix
    
    :param result_mat: om.MMatrix. the result matrix after multiplication
    :param source_mat: om.MMatrix. the source matrix used in pre multiplying
    :return: om.MMatrix. matrix used in pre multiplication
    """
    return result_mat * source_mat.inverse()
```

Similar process, we are able to find the target controller's transform based off
the rotational offset and the target joint transform.

$$
M_{ctrl} = M_{const}^{-1} * {M_{joint}} 
$$

#### Implementation

Bringing this back to the context of our IK matching process:

We know the **current controller** rotation and **current joint** rotation, this will give us the
constraint matrix (rotational offset),

We also know the **target joint** rotation in world space,
Therefore, we can convert this **target joint** rotation to **target controller** rotation in world space.

Now through script:

```python
# step 1: obtain the constraint matrix
current_jnt_rot = matrix.get_matrix(jnt)
current_fk_rot = matrix.get_matrix(fk)
cons_mat = matrix.get_pre_mult_matrix(current_jnt_rot, current_fk_rot)

# step 2: get controller's target matrix 
target_ctrl_mat = om.MTransformationMatrix(
  cons_mat.inverse() * target_jnt_mat
)
```

We can later [extract the rotation data](#rotation) from the transformation matrix.

### Model Matrix

Model matrix converts transformation matrix from world space to object space,
this has no direct usage in the IK/FK matching, as `cmds.xform()` already
does the work for us. But it's good to cover as it is very relevant.

#### Post-multiplying Matrix

Model matrix is a post-multiplying matrix:

$$
M_{world} = M_{local} * M_{model}
$$

We now pre-multiply the inverse of the local transform matrix:

$$
M_{local}^{-1} * M_{world} = M_{local}^{-1} * M_{local} * M_{model}
$$

$$
M_{model} = M_{local}^{-1} *{M_{world}}
$$

So similarly we can create the following function, with the result matrix argument 
being the world transform matrix,
and source matrix argument being the local transform matrix.

```python
def get_post_mult_matrix(result_mat, source_mat):
    """
    Get post multiplying matrix

    :param result_mat: om.MMatrix. the result matrix after multiplication
    :param source_mat: om.MMatrix. the source matrix used in post multiplying
    :return: om.MMatrix. matrix used in post multiplication
    """
    return source_mat.inverse() * result_mat
```

### Decomposing Matrix

Decomposing a transformation matrix (`om.MTransformationMatrix`) refers to
acquiring translation, rotation and scale values as individual `list[3]` type.
These values can now be applied using `cmds.xform()` either in local space or
in world space depending on the input transformation matrix.

#### Translation

```python
translation = matrix.translation(om.MSpace.kWorld)
```

#### Scale

```python
scale = matrix.scale(om.MSpace.kWorld)
```

#### Rotation

Extract rotation in axis angles in default **xyz** rotation order.

```python
import math

rot_vec = matrix.rotation().asVector()
rotation = [math.degrees(angle) for angle in (rot_vec.x, rot_vec.y, rot_vec.z)]
```

## References

[Tech Artists Org - Convert world space coordinates to object space coordinates in Maya?](http://discourse.techart.online/t/convert-world-space-coordinates-to-object-space-coordinates-in-maya/)

[Stack Exchange - Given this transformation matrix, how do I decompose it into translation, rotation and scale matrices?](https://math.stackexchange.com/questions/237369/)

[AK Eric - Find Euler rotation values of Maya matrix](https://www.akeric.com/blog/?p=1067)

[Maya Help - MFnTransform Class Reference](https://help.autodesk.com/view/MAYAUL/2018/ENU/?guid=__cpp_ref_class_m_fn_transform_html)

[Maya Help - MMatrix Class Reference](https://help.autodesk.com/view/MAYAUL/2018/ENU/?guid=__cpp_ref_class_m_matrix_html)

[Maya Help - MTransformationMatrix Class Reference](https://help.autodesk.com/view/MAYAUL/2018/ENU/?guid=__cpp_ref_class_m_transformation_matrix_html)


