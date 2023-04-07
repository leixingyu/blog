---
title: "Create Professional-Quality Cinematics with Unreal Engine: An Easy-to-Follow Guide"
tags:
  - unreal
  - pipeline
categories:
  - unreal cinematic
date: 2022-07-16
description: Unlock the full potential of Unreal Engine and create professional-quality cinematics with our easy-to-follow guide. Our step-by-step tutorial provides practical examples and best practices for enhancing your storytelling and creating immersive experiences. Whether you're a beginner or an advanced user, our guide will help you master the techniques needed to take your cinematics to the next level.
---

## Introduction

In this blog, I'm going to introduce the different components
of a cinematic, how they are represented. This hopefully will give you an
overview of how cinematic works in Unreal.

## Glossary

- **Skeleton**: a hierarchy that is used to define Bones (sometimes called joints),
In Unreal, skeletons are used to store/driven by associated animation data.

- **Static Mesh**: a piece of geometry that consists of a set of polygons 
that can be cached in video memory and rendered out.

- **Skeletal Mesh**: in short is a collection of a skeleton hierarchy, and mesh(es) skinned to
that skeleton. A set of polygons composed to make up the surface of 
the Skeletal Mesh, and a hierarchical set of interconnected bones 
which can be used to animate the vertices of the polygons

- **Level**: is all or part of your game's "world". Levels contain everything a 
player can see and interact with, like environments, usable objects, 
other characters, and so on.

- **Actor**: an Actor is any object that can be placed into a Level, 
such as a camera, Static Mesh, or player start location. 
Actors support 3D transformations such as translation, rotation, and scaling. 
They can be created and destroyed.

- **Binding**: a track in sequencer that binds a specific actor in the level, enables
the level sequence to modify or animate its property when the level sequence is active.

## What makes a Sequencer

> Similar to other film and animation editing software, Sequencer requires you
> to add tracks in order to reference elements in your sequence.
>
> Any character, prop, camera, effect, or other viewport element can be 
> referenced and manipulated in Sequencer

![overview](https://i.imgur.com/UI68i63.png)

### Level Sequence

A Level Sequence asset is a container for cinematic data, which includes
tracks, animation and binding.

### Level Sequence Track

If you used any video editing software, then tracks shouldn't be new to you.
Tracks are like layers, and the _Level Sequence_ is the whole canvas;

_Tracks_ usually are used to reference different actors in the level, 
and then animated its properties. So in most time, tracks don't overlay each
other like layers in Photoshop do, they're more like controlling 
different color channels that compose the final image.

### Level Sequence Section Asset

_Sections_ are the components that laid out in a _Track_, sections can be treated
as individual animation clips of which make up the whole track.

Some common section are _Camera Cuts_ section, _Shots_ section and _Animation_ section.

## Basic Cinematic Component

We'll break down a cinematic as the following components and tackle them
one by one.

- Camera (Camera Actor)
- Background (Static Mesh Actor)
- Foreground & Character (Skeletal Mesh Actor)
- Lighting (Light Actor)
- FX (Visual Effects Actor)
- Audio (Sound Wave Asset)
- Other


## Actor to Sequence (Object Binding)

The level sequence operates within levels and operates on actors in the level.
It can bind to an existing actor and animates its properties. It can also spawn an actor
as needed.

[Spawnables vs. Possessables](https://docs.unrealengine.com/5.0/en-US/spawn-temporary-actors-in-unreal-engine-cinematics/)

### Static Mesh
![static](https://i.imgur.com/NGIIEAp.png)

This is the simplest scenario, a static mesh actor is bound to the level sequence
and the transform are keyed.

### Skeletal Mesh

![skm](https://i.imgur.com/Rh2kTsR.png)

When it comes to skeletal mesh, another component is exposed: Animation.

This sub-track takes Unreal [Animation Sequence](https://docs.unrealengine.com/5.0/en-US/animation-sequences-in-unreal-engine/) assets, which consists the 
skeleton animation, this effectively drives the skeleton within the skeletal
mesh actor.

### Camera Binding and Camera Cut

![camera](https://i.imgur.com/exrhTgo.png)

In order to set up camera view properly, in-addition to the camera binding,
we also need a _Camera Cuts Track_. Whatever is on this track is what the viewport
is capturing, it allows you to switch and blend between different cameras.

(In short, the Camera Binding tracks drives the Camera Actor, the Camera Cut track decides
what takes over the viewport)

### Audio

![audio](https://i.imgur.com/PNHsLUU.png)

Audio can be created using by possessing an audio/sound actor, or alternatively
it can be created by adding Unreal [Sound Wave](https://docs.unrealengine.com/5.0/en-US/audio-files-in-unreal-engine/) asset. I would recommend
the latter as it allows you to scrub in the timeline and still have the 
correct audio playback.

> **Note:** Set Sound Wave Asset's '_Sound Asset Compression Type_' to '_PCM_' so
> it plays correctly when scrubbing in timeline.

### More

There are of course different actor types in the level that the level sequence
can possess/bind to. Such as _Light Actors, Gameplay Actors, Visual Effects
Actors_ and more.


## Sub-sequence and Shot

There are other two types of concept I'd like to touch upon:
_Subsequences_ and _Shots_.

They are usually not directly bound to an actor, but they are containers for
a level sequence that does. They can be very useful to organize and composite
the cinematic level sequence structure.

![subsequence](https://i.imgur.com/hrVKhin.png)

[Subsequences](https://docs.unrealengine.com/5.0/en-US/cinematic-subscequences-track-in-unreal-engine/) 
represent an entire Level Sequence.
Having subsequences allows multiple level sequence to be added into a "parent" level sequence,
These subsequences can be considered as the partition of the "parent" and can be worked on as different
sub-scenes by different artists.



![shot](https://docs.unrealengine.com/4.27/Images/AnimatingObjects/Sequencer/Overview/TracksShot/Thumbnails.png)

(Image from Unreal)

Similar to Subsequence, each _Shot Section_ represents a Level Sequence,
it allows multiple level sequences be assembled and re-arranged freely on the
_Shots Track_ as Shot _Section_ to form the full sequence/cinematic. This
is to get rendering in Unreal to work correctly, and able to set per-shot
preset in Movie Render Queue.

> **Note:** avoid using the "lock" property in any of the clip as it sometimes causes
> un-wanted behaviours.

### High Quality Render with Motion Blur

When implementing a setup with a full sequence length animation clip 
(skeletal mesh .fbx not being broken down into shots). It is important to follow
the following to ensure there's no Motion Blur produce during camera cuts.

- Camera: separate camera actor for each shot.

- Animation: leave the animation clip as 'Keep State' when finished if possible.

- Subsequence: a good practice is to not extend sub-sequence range over the 
shot active range; Unreal still calculates motion in sub-sequence extending beyond 
the range of the shot active range such as Motion Blur, which can produce
unwanted result during cuts between shots.

- Render Preset
  - Camera Setting has to be set to 'Frame Close'
  - Anti-aliasing Setting can be toggled on for better quality

For more information on setting up a sequencer with correct first frame, see 
[Epic's post on render warm-ups](https://dev.epicgames.com/community/learning/tutorials/l4OR/unreal-engine-movie-render-queue-warmup-and-first-frame-issues)

## References

[Unreal Engine Documentation - Cinematics and Sequencer](https://docs.unrealengine.com/5.0/en-US/cinematics-and-movie-making-in-unreal-engine/)
