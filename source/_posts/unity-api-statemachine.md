---
title: "Create Dynamic Animation Behaviors with Unity's State Machine Scripting API"
tags:
  - unity
  - c#
categories:
  - tech summary
date: 2021-08-29
description: Unlock the full potential of Unity's state machine scripting API and create dynamic animation behaviors with our comprehensive tutorial. Our guide provides practical examples and best practices for creating complex and engaging animations that respond to user input, events, and variables. Whether you're a beginner or an advanced user, our step-by-step tutorial will help you master the techniques needed to take your Unity animation skills to the next level.
---

## Introduction

This post is not about state machine basic, rather how we would setup/access state machine parameters.
There is a higher level `Animator` class to access a part of the animator features, but we need to use 
Unity's `Animations` package in order to dive deeper.

```c#
using UnityEditor.Animations;
```

There's a lot of parts composing Unity's Animator: Layer, Parameter, State machine (and everything inside it
including states of different kind, transition between states), there are of course multiple parameters inside a single state.

The examples in this post is only about accessing states and state machines, as the rest is translatable.

We start from the root: Base layer, if you think about it, a layer is essentially a state machine.
Inside this root state machine are sets of states (including blend tree/state/sub-state machine)

```c#
// getting the base layer state machine
AnimatorStateMachine rootStateMachine = controller.layers[0].stateMachine;
```

![base layer state machine](https://i.imgur.com/HuSyYGs.png)

(Base Layer)

Next up, a very confusing part of the scripting process are the definition of 
`AnimatorStateMachine` vs. `ChildAnimatorStateMachine` and `AnimatorState` vs. `ChildAnimatorState`.
I really wish Unity provides internal ways to cast these to each other. 

To summarize: 
- `ChildAnimatorStateMachine[]` and `ChildAnimatorState[]` is used in conjunction with Arrays as `AnimatorStateMachine.states` and 
`AnimatorStateMachine.stateMachines` will return the previous mentioned type.
  
- Instead of casting, they have an internal property `ChildAnimatorStateMachine.stateMachine` and `ChildAnimatorState.state` which 
respectively returns `AnimatorStateMachine` and `AnimatorState` type object

## States

All states (state or. state with blendtree inside)

```c#
ChildAnimatorState[] childStates = rootStateMachine.states;

foreach(var childState in childStates)
{
    AnimatorState state = childState.state;
    print(state.name);

    // state with blendtree
    try
    {
        var blendTree = (BlendTree)state.motion;

        foreach (var child in blendTree.children)
        {
            print("\t" + child.motion.name);
        }
    }
    // simple state
    catch
    {
        print("\t" + state.motion.name);
    }
}
```

![simple state](https://i.imgur.com/QbLtx2O.png)

(Simple state only contains a single clip)

![state with blendtree](https://i.imgur.com/vQMjJ5z.png)

(BlendTree nested in a state)

## Sub-state Machine

```c#
ChildAnimatorStateMachine[] childSubStateMachines = rootStateMachine.stateMachines;

foreach(var childSubStateMachine in childSubStateMachines){
    
    // Convert `ChildAnimatorStateMachine` to `AnimatorStateMachine`
    AnimatorStateMachine subStateMachine = childSubStateMachine.stateMachine;
    print(subStateMachine.name);

    // all states in idle sub state machine
    ChildAnimatorState[] states = subStateMachine.states;

    foreach (var childState in states)
    {
        print("\t" + childState.state.name);
    }
}
```

![sub state machine](https://i.imgur.com/j6IoXLr.png)

(Essentially, another sub layer of state machine)

## Reference

[Unity Manual - State Machine Basics](https://docs.unity3d.com/Manual/StateMachineBasics.html)

[Unity Scripting API - AnimatorStateMachine](https://docs.unity3d.com/ScriptReference/Animations.AnimatorStateMachine.html)

[Unity Scripting API - Animator.Play](https://docs.unity3d.com/ScriptReference/Animator.Play.html)


