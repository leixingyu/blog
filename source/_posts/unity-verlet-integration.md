---
title: "Achieve Realistic Physics in Unity with Verlet Integration"
tags:
  - unity
  - c#
categories:
  - 3d math
date: 2020-11-19
mathjax: true
description: Discover how to achieve realistic physics in Unity with Verlet integration using our step-by-step tutorial. Our guide provides practical examples and best practices for creating lifelike simulations and dynamic game environments. Whether you're a beginner or an advanced user, our tutorial will help you master the techniques needed to create compelling physics-based experiences in Unity
---

## Introduction
When simulating a particle movement, we naturally think of Euler integration, where the position of the particle can be expressed as such: (free falling motion)

$$
x = 0.5*g*t^2
$$
This is an extremely easy use case. The position of the particle can be represented using a function of time. But in real life, a particle can be affected by multiple forces (and even unstabled force caused non-constant acceleration) and Euler integration also has inaccurate estimation when time step is very large.

In my understanding, the use of Verlet is good for complex particle movement. What it essentially does is relate force with position rather than velocity. The process first calculate the current particle position and the position one step back:

$$
velocity = X~current~-X~previous~;
$$

$$
X~previous~ = X~current~;
X~current~ = X~current~+velocity
$$

This works magically, here's some sample code to help understand it, and a [Demo](https://leixingyu.itch.io/rope-simulation).

```c#
// without other influence, the following updates the particle position
velocity = posNow - posOld;
posOld = posNow;
posNow += velocity;
```

## Calculate Force

What if we want to add gravity to the particle like the free falling above? Just add the acceleration to the current position, and the change will add-up on the next integration. The following also applies to forces like friction.
```c#
posNow += GRAVITY; // not sure if we should multipy Time.deltatime
```

## Constraint

Constraint are usually in the form of constraining particles to a specific distance, like spring. When mulitple constraints acts on single particle, one constraint will affect the other. To solve this, loop the constraint multiple time for each time step.

```c#
for (int count = 0; count < ITERATION; count++){
    Constriants();
}

// constraint on rope to make fixed distance between segments
void Constriants()
{
    for(int index = 0; index < segment-1; index++)
    {
        float distance = (currentSeg.posNow - nextSeg.posNow).magnitude;
        float error = Mathf.Abs(distance - ropeDist);
        Vector2 changeDir = Vector2.zero;
    
        if (distance > ropeDist)
            changeDir = (currentSeg.posNow - nextSeg.posNow).normalized;
        else if (distance < ropeDist)
            changeDir = (nextSeg.posNow - currentSeg.posNow).normalized;
    
        Vector2 changeAmount = changeDir * error;
        if (index == 0)
            nextSeg.posNow += changeAmount;
        else{
            currentSeg.posNow -= changeAmount * 0.5f;
            nextSeg.posNow += changeAmount * 0.5f;
        }
    }
}
```

## Collision

Penalty based system or projection collision reaction

```c#
// example for bounding box collision
Vector2 velocity = posNow - posOld;

if (posNow.x > screenWidth){
    posNow.x = screenWidth;
    posOld.x = posNow.x + velocity.x;
}
else if (posNow.x < 0){
    posNow.x = 0;
    posOld.x = posNow.x + velocity.x;
}
if (posNow.y > screenWidth){
    posNow.y = screenWidth;
    posOld.y = posNow.y + velocity.y;
}
else if (posNow.y < 0){
    posNow.y = 0;
    posOld.y = posNow.y + velocity.y;
}
```

## Resources

[Gamedev.net - A Verlet based approach for 2D game physics](https://www.gamedev.net/articles/programming/math-and-physics/a-verlet-based-approach-for-2d-game-physics-r2714/)

[Youtube - Coding Math: Episode 36 - Verlet Integration Part I](https://youtu.be/3HjO_RGIjCU)

[Youtube - 2D Rope Tutorial - Make Swinging Rope in Unity (Verlet Integration)](https://www.youtube.com/watch?v=FcnvwtyxLds)

[Wikipedia - Velocity Verlet](https://en.wikipedia.org/wiki/Verlet_integration#Velocity_Verlet)



