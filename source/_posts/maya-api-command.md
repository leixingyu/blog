---
title: "Maya Python API Tutorial Series (1. Command Plugin with Flags)"
tags:
  - maya
  - python
categories:
  - maya python api
date: 2019-09-29
description: Unlock the full potential of Maya with our expert guide to the Python API. Learn how to automate complex tasks, create custom tools, and streamline your workflow using the power of Python. Our comprehensive tutorial covers everything from the basics to advanced techniques.
---


## "Flags" vs "Argument"

Take this as an example: 

```cmds.group('circle1', 'sphere1', name='group1')```

- `circle1` and `sphere1` are arguments
- `name` is the flag and `group1` is the value

Another example:

```cmds.polyCube(sx=10, axis=[0, 0, 1])```
- **no argument** is specified
- `sx` is the flag's short name,  `subdivisionX` is the flag's long name
- `[0, 0, 1]` is axis flag's value, each individual number is called parameters


## Procedure

### Step 1: Declare flag name outside the class

```python
firstFlagShortName = '-f'
firstFlagLongName = '-first'
secondFlagShortName = '-s'
secondFlagLongName = '-second'
# more flags ...
```

### Step 2:  Add flag and argument in syntax creator outside of class

(this syntax creator will be further included in the plugin initialize function)

```python
def syntaxCreator():
    """
    create a OpenMaya.MSyntax object to store flags and argument
    """
    syntax = om.MSyntax()
    
    # add flags with short name, long name, and value type
    syntax.addFlag(firstFlagShortName, firstFlagLongName, om.MSyntax.kDouble)
    syntax.addFlag(secondFlagShortName, secondFlagLongName, (om.MSyntax.kDouble, om.MSyntax.kDouble, om.MSyntax.kDouble))
    
    # add more flags ...
    
    # add argument using MSyntax.addArg() function
    
    # add argument is not discussed, refer to document later
    
    return syntax
```

### Step 3:  Parsing flags, called inside the class's `doIt` function

```python
    def parseArguments(self, args):
        """ 
        instantiate MArgParser object, self.syntax() refers to the syntax created in step 2
        """
        argData = om.MArgParser(self.syntax(), args)

        # check if certain flags are set
        if argData.isFlagSet(firstFlagShortName):
            firstValue = argData.flagArgumentString(firstFlagShortName, 0)
        if argData.isFlagSet(secondFlagShortName):
            secondParam0 = argData.flagArgumentInt(secondFlagShortName, 0)
            secondParam1 = argData.flagArgumentInt(secondFlagShortName, 1)
            secondParam2 = argData.flagArgumentInt(secondFlagShortName, 2)
            
        # parse more flags ...
```

### Reference

[Chad Vernon - Maya API Programming](https://www.chadvernon.com/maya-api-programming/)

