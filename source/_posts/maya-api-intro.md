---
title: "Maya Python API Tutorial Series (0. Introduction)"
tags:
  - maya
  - python
categories:
  - maya python api
date: 2019-08-18
description: Unlock the full potential of Maya with our expert guide to the Python API. Learn how to automate complex tasks, create custom tools, and streamline your workflow using the power of Python. Our comprehensive tutorial covers everything from the basics to advanced techniques.
---

### Overview

Different from Maya Command, Maya API offers more low level access to Maya's feature.
You can think of Maya in multiple layers:
1. Bottom Layer: System OS
2. Maya Core: The entire Maya program written in C++
3. Maya API: Designated API exposed to developers, which can access Maya Core
4. Maya Command: Helper Functions calling multiple Maya API
5. Maya GUI: User graphics interface

While Maya Command (MEL/Python) is most commonly used to write scripts, because it has fast prototyping and easy to learn. The downside is that it is often slow and can't offer low level access to Maya Core. 

Maya API on the other hand, is complicated, but offers faster speed and most flexibility. It comes with Python and C++. Python API has version 1.0 and 2.0.

[Difference Between Python API vs C++ API](http://help.autodesk.com/view/MAYAUL/2018/ENU/?guid=__files_Maya_Python_API_Differences_between_the_C_Maya_API_and_Maya_Python_API_htm)

### API1.0 vs 2.0

API 1.0 (Before 2012) has access to more features offered in C++
API 2.0 faster and more Pythonic, but some class such as OpenMayaFX, OpenMayaDeformerNode is not supported.

### General Format

<script src="https://gist.github.com/leixingyu/358f5a9c86278a43852148c87253e3d9.js"></script>

### Difference
1. **import module**
   - 1.0 has a separate `maya.OpenMayaMPx` along with `maya.OpenMaya`
   - 2.0 has `maya.api.OpenMaya` including the original MPx command
   
2. **maya_useNewAPI**
   - 2.0 use this function to declare using API 2.0
   
3. **cmd creator**
   - 1.0 uses a MPx pointer object to point to instance of the class
   - 2.0 director returns the instance of the class
   
4. **other syntax difference will be discussed later**

### Reference

[Chad Vernon - Maya API Programming](https://www.chadvernon.com/maya-api-programming/)


