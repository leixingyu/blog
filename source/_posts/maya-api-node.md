---
title: "Maya Python API Tutorial Series (2. Custom Node)"
tags:
  - maya
  - python
categories:
  - maya python api
date: 2019-10-10
description: Unlock the full potential of Maya with our expert guide to the Python API. Learn how to automate complex tasks, create custom tools, and streamline your workflow using the power of Python. Our comprehensive tutorial covers everything from the basics to advanced techniques.
---

### API 2.0 custom node example

<script src="https://gist.github.com/leixingyu/6789c67a3aea4258589c78878eeacc99.js"></script>

### Data block (MDataBlock)

Data block refers the entire container of a node. This container stores all the values for each attribute of the node.

The datablock of this node is provided in the `compute()` function.

### Plug (MPlug)

Plug is the outer plug of the node which provides the connections to other node's plug. The MPlug usually connects the plug from another node and the value of a certain attribute of this node.

The plug is also provided in the `compute()` function

### Data handle (MDataHandle)

Data handle usually stores an attribute's value. 

We can set a value to an attribute using `MDataHandle.setType(value)`, 
and get an attribute's value using `**`value = MDataHandle.asType()`**`.

## Procedure

### Step 1: Declare attributes (MObject) in the class

```python
class MyNode(om.MPxNode):
    inAttr = om.MObject()
    outAttr = om.MObject()
    
	def __init__(self):
		om.MPxNode.__init__(self)

	def compute(self, plug, dataBlock):
		pass
```

`inAttr` refers to the input attribute that we are creating,

`outAttr` refers to the output attribute that we are creating. 

Both are declared as `MObject()` type. 
We will later access them using `MyNode.inAttr` and `MyNode.outAttr`.

### Step 2: Initialize Node

**Node Creator**

```python
def nodeCreator():
    return MyNode()
```

Node Creator in API 2.0 directly returns an instance to the class, 
API 1.0 uses a pointer like in command plugin.

**Node Initializer**

```python
def nodeInitializer():
    '''1: create reference to attribute function set such as numericAttribute'''
    numericAttrFn = om.MFnNumericAttribute()

    ''' 2: create attribute using the function set'''
    MyNode.inAttr = numericAttrFn.create('in', 'i', om.MFnNumericData.kFloat, 1.0)
    numericAttrFn.readable = True
    numericAttrFn.writable = True
    MyNode.outAttr = numericAttrFn.create('out', 'o', om.MFnNumericData.kFloat)
    numericAttrFn.readable = True
    numericAttrFn.writable = False

    ''' 3: attach attribute'''
    MyNode.addAttribute(MyNode.inAttr)
    MyNode.addAttribute(MyNode.outAttr)

    ''' 4: add circuit (relationship in->out)'''
    MyNode.attributeAffects(MyNode.inAttr, MyNode.outAttr)
  ```

- `MFnNumericAttribute` provides the function sets to create an attribute for numeric type attribute.
   There's also `MFnMatrixAttribute` to create matrix type attribute and so on.
- Using the function set's `create()` function, we add parameters for attribute's 
  **long name, short name, data type, and default value (optional)**.
    - This returns a `MObject` type attribute which is attached to the function set. We store it in the attribute declared earlier in the class.
    - Set the attribute's property using function set, 
      such as **readable, writable, hidden, storable, connectable** 
      (In API1.0, use like this `numericAttrFn.setReadable(True)`)
- Using the `MyNode.addAttribute(MyNode.inAttr)` to attach an attribute to the node
- Using the `MyNode.attributeAffects(MyNode.inAttr, MyNode.outAttr)` to design the affect relationship between attributes, 
   in this case, change of `inAttr` will affect `outAttr.

**RegisterNode**

```python
mplugin.registerNode(nodeName, nodeID, nodeCreator, nodeInitializer, om.MPxNode.kDependNode, nodeClassify)
```

**Parameters:** name of the node, id of the node, node creator function, node initializer function, node type (DependNode or DeformNode...), node classification (utility, shading...)

**Example:**
- `nodeID = om.MTypeId(0x55555)`
- `nodeClassify = 'utility/general'`

**De-registerNode**

```python
mplugin.deregisterNode(nodeID)
```

### Step 3: Initialize Node

```python
class MyNode(om.MPxNode):
    inAttr = om.MObject()
    outAttr = om.MObject()
    
	def __init__(self):
		om.MPxNode.__init__(self)

    def compute(self, plug, dataBlock):
        if plug == MyNode.outAttr:
            # 1: get datablock handle (inputValue returns MDataHandle type object)
            inHandle = dataBlock.inputValue(MyNode.inAttr)
            outHandle = dataBlock.outputValue(MyNode.outAttr)

            # 2: extract input value from the handle
            inValue = inHandle.asFloat()

            # 3: create logic and set output value
            outValue = inValue * 2
            outHandle.setFloat(outValue)

            # 4: mark output plug as clean
            outHandle.setClean()
            
        else:
            return om.kUnknownParameter
```

1. When `MyNode.outAttr` is dirty (meaning it needs to recompute), 
we use `if plug == MyNode.outAttr:` to identify this certain plug. 
(this if statement will work, even if `plug` is `MPlug` type and `MyNode.outAttr` is `MObject` type)

2. We have already identified what input attribute is affecting this plug, in order to retrieve the value of this input, we need to attach a data handle on the data block specifying this certain input attribute we want to retrive.
Therefore, we have `handle = dataBlock.input/outputValue(MyNode.attr)`

3. Next, we use `value = handle.asFloat()` and `handle.setFloat(value)` to get and set the value from and to the attribute. (float type as example) 

4. Last, we mark the current plug as clean, by `setClean()` to the outHandle

### Reference

[Chad Vernon - Maya API Programming](https://www.chadvernon.com/maya-api-programming/)


