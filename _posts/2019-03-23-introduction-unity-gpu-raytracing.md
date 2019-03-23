---
layout: post
title: "Introduction to GPU Ray Tracing in Unity"
categories: graphics, unity, gpu, ray tracing
author: "Doruk Coşkun"
---

We are expected to implement a CPU ray tracer in my CENG795 Advanced Ray Tracing lecture written in C++. After an agreement with my lecturer, I have started to implement my ray tracer in Unity using Compute Shaders. The main reason behind this decision was to use the coding projects given in the lecture as a medium for deepening my knowledge and experience in Unity and shaders. 

I believe if I can manage to form the correlation between the physically based rendering concepts I learn during the semester with the quality of life tools presented in Unity's editor, I can create a working environment for myself where I can experiment on scenes I build and easily implement new concepts (with the help Unity editor provides) .

Mainly I can separate my ray tracer into two components:
- Editor
- Compute Shader

The editor includes the 3D scene (camera data, meshes, lights, materials), scripts which store these data in relative classes, sends these data to the compute shaders buffers. As I envision it, the editor will be further improved on with other scripts to create dynamic scenes and store the render as a video. If I can optimize my rendering times real-time gameplay can be provided.

Compute Shader is where the scene data sent from the Editor is processed and ray tracing concepts are applied. I believe as my understanding of GPU programming improves I will achieve better performance and I will structure my code better. One thing to note is that GPU programming introduces limitations one wouldn't face in CPU programming. Some limitations I have experienced so far include lack of recursive function calls (which are over extensively used in ray tracing) and lack of debugging options.

Ray tracing concepts which I will implement includes:
- Ray-object intersections
- Basic surface shading
- Hard shadows
- Mirror reflection
- Refraction
- Acceleration techniques (BVH)
- Depth of field
- Motion blur
- Glossy reflections
- Instancing, Model and view transformations
- Normal, displacement and bump mapping
- Spot, area, environment lights

As I progress I will introduce more concepts which I will implement.
