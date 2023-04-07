---
title: "Maya Python API Tutorial Series (3. Custom Deformer)"
tags:
  - maya
  - python
categories:
  - maya python api
date: 2019-11-11
description: Unlock the full potential of Maya with our expert guide to the Python API. Learn how to automate complex tasks, create custom tools, and streamline your workflow using the power of Python. Our comprehensive tutorial covers everything from the basics to advanced techniques.
---

### API 1.0 custom deformer Example

<script src="https://gist.github.com/leixingyu/78352732cb52c6e797244755ade0bbfe.js"></script>

Note: `MPxDeformerNode` is only available in API 2.0

**Custom Attribute vs. Built-in Attribute**

In the last chapter, we know how to create custom numeric type attribute using `MFnNumericAttribute`.
Sometimes in our node, we want to access existing built-in attribute. 
We do so by using `OpenMayaMPx.cvar.MPxDeformerNode_(attributeName)` before Maya 2016,
we use `OpenMayaMPx.cvar.MPxGeometryFilter_(attributeName)` after 2016.

**Obtain Input Geometry**

In the sample code, we define our custom function `getDeformerInputGeom(self, dataBlock, geomIndex)`
to obtain the input mesh to the deformer node. We will discuss this later.

**Accessory Node**

Accessory node acts like a secondary driver node connected to our deformer so they can influence the deformation. In the sample code, our accessory node is a locator which when we connects its world matrix, it will change our mesh's deformation when translating.

**Custom Dependency Node vs. Custom Deformer Node**

**Registration:** 
In our previous chapter, we register our node using `registerNode()` with 
node type: `omMPx.MPxNode.kDependNode`, 
in deformer node, we use `omMPx.MPxNode.kDeformerNode` as our node type.

**Inheritance:** 
We now inherit our class from `omMPx.MPxDeformerNode` instead of `omMPxNode`
there's still `compute()` in `MPxDeformerNode` class, 
but we want to write our deformation algorithm in `deform()`.

**Accessory Node:**
`accessoryNodeSetup(self, dagModifier)` and `accessoryAttribute(self)` is override to allow us to control accessory node along with our deformer.

## Procedure

### Step 1: Declare attributes (Same as last chapter)

### Step 2: Initialize Node

**Node Creator**
```python
def nodeCreator():
    return mpx.asMPxPtr(MyDeformer())
```

Only API 1.0 is available.

**Node Initializer**

```python
def nodeInitializer():
    # 1: create reference to numericAttribute and matrixAttribute function sets
    numericAttrFn = om.MFnNumericAttribute()
    matrixAttrFn = om.MFnMatrixAttribute()

    # 2: create attribute using the function set
    MyDeformer.inNumAttr = numericAttrFn.create('num', 'n', om.MFnNumericData.kFloat, 0.0)
    numericAttrFn.setMin(-1.0)
    numericAttrFn.setMax(1.0)
    numericAttrFn.setReadable(False)
    
    MyDeformer.inMatAttr = numericAttrFn.create('matrix', 'm')
    matrixAttrFn.setStorable(False)
    matrixAttrFn.setConnectable(True)

    # 2.5: access built-in attribute using OpenMayaMpx.cvar.MPxGeometryFilter_outputGeom
    outputGeom = mpx.cvar.MPxGeometryFilter_outputGeom

    # 3: attach attribute
    MyDeformer.addAttribute(MyDeformer.inNumAttr)
    MyDeformer.addAttribute(MyDeformer.inMatAttr)

    # 4: add circuit (relationship in->out)
    MyDeformer.attributeAffects(MyDeformer.inNumAttr, ouputGeom)
    MyDeformer.attributeAffects(MyDeformer.inMatAttr, ouputGeom)

    # 5: make attribute paintable
    cmds.makePaintable(nodeName, 'weights', attrType='multiFloat', shapeMode='deformer')
```

we access the output Geometry attribute so we can later add relationship to it.

**RegisterNode**

```python
mplugin.registerNode(nodeName, nodeID, nodeCreator, nodeInitializer, om.MPxNode.kDeformNode)
```

**De-registerNode**

```python
mplugin.deregisterNode(nodeID)
```

### Step 3: Initialize Node （Deform Algorithm）

```python
class MyNode(om.MPxDeformNode):
    inNumAttr = om.MObject()
    inMatAttr = om.MObject()
    
	def __init__(self):
		om.MPxDeformNode.__init__(self)

    def deform(self, dataBlock, geomIterator, localToWorldMatrix, geomIndex):
        # step 1: access built-in attribute value using attribute name and attribute handle
        envelopeAttr = mpx.cvar.MPxGeometryFilter_envelope
        envelopeHandle = dataBlock.inputValue(envelopeAttr)
        envelopeValue = envelopeHandle.asFloat()

        # step 1.5: access custom attribute value
        inNumHandle = dataBlock.inputValue(MyDeformer.inNumAttr)
        inNumValue = inNumHandle.asFloat()

        # step 1.55: access custom translate value connected to an accessory node
        inMatHandle = dataBlock.inputValue(MyDeformer.inMatAttr)
        inMatValue = inNumHandle.asMatrix()
        transMatrix = om.MTransformationMatrix(inMatValue)  # matrix type
        translateValue = transMatrix.getTranslation(om.MSpace.kObject)  # vector type

        # step 2: access input mesh
        inputMesh = self.getDeformerInputGeom(dataBlock, geomIndex)

        # step 2.5: access mesh normals
        meshFn = om.MFnMesh(inputMesh)
        normalVectorArray = om.MFloatVectorArray()  # create float vector array to store normal vector
        meshFn.getVertexNormals(False, normalVectorArray, om.MSpace.kObject)  # (average normal or not?, the array to store, normal space)

        # step 3: iterate the mesh vertices and deform it
        newVertexPosArray = om.MPointArray()  # to store new vertices position
        while not geomIterator.isDone():
            vertexPos = geomIterator.position()
            vertexIndex = geomIterator.index()
            normalVector = om.MVector(normalVectorArray[vertexIndex])
            # built-in function weightValue(dataBlock, geomIndex, vertexIndex)
            weight = self.weightValue(dataBlock, geomIndex, vertexIndex)

            # vertexPos.x = vertexPos.x + [calculation of normalVector.x and translateValue[0]] * envelopeValue * weight
            newVertexPosArray.append(vertexPos)
            geomIterator.next()
        geomIterator.setAllPositions(newVertexPosArray)
```

- To access a value from an attribute, we use `handle = dataBlock.input/outputValue(MyNode.attr)`


- if we have a custom attribute `inNumAttr:` 
  - `inNumHandle = dataBlock.inputValue(MyDeformer.inNumAttr)`
  - `inNumValue = inNumHandle.asFloat()`
    

- if we have a built-in attribte `envelope`:
  - we first get our attribute name `envelope`
  - `envelopeAttr = mpx.cvar.MPxGeometryFilter_envelope`
  - `envelopeHandle = dataBlock.inputValue(envelopeAttr)`
  - `envelopeValue = envelopeHandle.asFloat()`


- To get normal for individual vertices on our input mesh, 
  we first need to obtain our input mesh using our own function: 
  `getDeformerInputGeom(self, dataBlock, geomIndex)`. 
  And using mesh function set MeshFn's `getVertexNormals()` we store the
  normal vector in `om.MFloatVectorArray()` type array.


- To deform our mesh: we use the geometry iterator to perform iteration on
  each mesh vertex and re-calculate its position. 
  We combine the use of `geoIterator.position()` and 
  `geomIterator.setPosition(point)` or `geomIterator.setAllPositions(pointArray)`.


- To access weight value on each vertex, we use built-in
  function `weightValue(dataBlock, geomIndex, vertexIndex)`. 
  In which, `geomIndex` is provided in `deform()` and `vertexIndex` is from `geomIterator`.

### Step 3.1: Get in-Mesh

```python
    def getDeformerInputGeom(self, dataBlock, geomIndex):
        inputAttr = mpx.cvar.MPxGeometryFilter_input
        inputHandle = dataBlock.outputArrayValue(inputAttr)  # use outputArray instead of inputArray to avoid re-computation
        inputHandle.jumpToElement(geomIndex)
        inputElementHandle = inputHandle.outputValue()

        inputGeomAttr = mpx.cvar.MPxGeometryFilter_inputGeom
        inputGeomHandle = inputElementHandle.child(inputGeomAttr)  # this is different from how we usually get handler
        inputGeomMesh = inputGeomHandle.asMesh()

        return inputGeomMesh
```

At this point, I can't fully interpret the meaning of this segment.

### Step 4: Accessory Node

```python
    def accessoryNodeSetup(self, dagModifier):
        # step1: create the accessory node using the supplied dagModifier
        locator = dagModifier.createNode('locator')

        # step2: access accessory node's attribute(can't use mplug type, has to be mobject type)
        # access dependency node function set
        dependNodeFn = om.MFnDependencyNode(locator)
        matrixPlug = dependNodeFn.findPlug('worldMatrix')  # this returns mplug type attribute, we need mobject type attribute
        matrixAttr = matrixPlug.attribute()

        # step3: connect mobject type(required) together
        # param: accessory node(mobject), accessory attr(mobject), deformer node(mobject: using self.thisMObject()), deformer attr(mobject)
        mConnectStatus = dagModifier.connect(locator, matrixAttr, self.thisMObject(), MyDeformer.inMatAttr)

        # now the accessory node's worldMatrix is driving to the custom in-matrix of the deformer node
        return mConnectStatus

    def accessoryAttribute(self):
        # returns the deformer node attribute connected
        return MyDeformer.inMatAttr
```

The `dagModifer` is supplied in the accessory node. We use dagModifier's connect function to
connect the accessory node's attribute to our deformer node's attribute. 
In this case, we have accessory's attribute: `worldMatrix` 
(a built-in attribute obtained from `MFnDependencyNode.findPlug())` 
and our custom defined `MyDeformer.inMatAttr`.

One thing to note is that, the `.connect()` only takes MObject which we cannot supply `MPlug` type
object `matrixPlug = ...findPlug('attributeName')`, we perform an additional step
`matrixAttr = matrixPlug.attribute()` to get the MObject type attribute.

Now we supply `.connect()` with parameters: an accessory node (MObject type), 
accessory node's attribute (MObject type), deformer node (MObject type) 
and deformer node's attribute (MObject type) as follows: 
`mConnectStatus = dagModifier.connect(locator, matrixAttr, self.thisMObject(), MyDeformer.inMatAttr)`


### Reference

[Chad Vernon - Maya API Programming](https://www.chadvernon.com/maya-api-programming/)

