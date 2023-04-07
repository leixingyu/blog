---
title: CFX Cloth Pipeline
summary:
tags:
  - cfx
categories:
  - pipeline discussion
date: 2021-09-17 
hidden: true
---

## Asset Folder Structure

***
- Big binary data store in asset control system
- Smaller data: materials, tools, level/actor definition stored in Perforce

***

Assets in a project specific folder or a shared folder
- uni-directional for project specific folder
- bi-directional for shared folder

***
The use of symlink (symbolic link)

***
Folder example:

Network Drive:/proj-name/art/
  - assets-proj-name/
    - animals/
    - characters/
      - charA/
        - anim/
    - levels/
      - level-name1/
          - design/
          - lighting/
          - objects/
          - textures/
          - water/
          - work/
          - prototypes/
      - level-name2/
    - lighting/
        - gobos/
        - luts/
        - skies/
    - objects/
      - character-props/
      - weapons/
    - particles/
    - scenes/
    - sound/