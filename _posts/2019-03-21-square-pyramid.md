---
layout: post
title: "My 3D Low Poly Passion Project"
categories: gamedev
author: "Doruk Coşkun"
---

For [Gateway 2018](https://www.facebook.com/GATEWay.METU/), a game convention held in METU Informatics Institute, I have prepared Rise of the Square Pyramid, my first passionate 3D game made in Unity.

![ROTSP-poste2](/assets/screen-shots/ROTSP-poster2.png)

After years of fooling around in Unity, I have decided to bring my desire to create a 3D game to life for [GateWay](https://www.facebook.com/GATEWay.METU/) game convention. Months before the convention I have started my project. During this period I wanted to focus on creating beautiful graphics and an immersive story. I have also decided to use this project as an excuse for myself to delve deeper into the graphics side of game development. One of the key points I strived for was achieving this visual fidelity with a constant 60 FPS in MacBook retina display (2880x1800). So optimization and performance were crucial.

![ROTSP](/assets/screen-shots/RiseOfTheSquarePyramid.png)

## Deserts
I adore deserts. Their incomprehensible size raises strong feelings of fear and respect in me. I have been to Gobi desert in Mongolia and I was feeling exactly like that while I was horse riding over the dunes.

When I started my project, I had an image in my mind. A vast low poly desert going as far as the eye could see... A midday sun shining on the golden sands. Low poly transparent clouds slowly moving in the same direction... Crystal clear blue sky looking amazing with high-resolution skybox texture... Mountain range looks detailed enough but optimized with Level Of Detail (LOD)... I wanted my game to be a journey, making the player share my feelings.

In this blog post, I want to give an overview of things I worked on and the reasons behind my design decisions. 

## Every successful project starts in...
...the Project Settings of course!

![ROTSP-project-settings](/assets/screen-shots/ROTSP-project-settings.png)

For a more accurate color calculation in the graphics pipeline, I have set the Color Space to Linear. Chose the Ultra Quality setting. Later on in the development tweaked the shadow settings for a better performance/quality tradeoff. Choosing a rendering path took some time for me to decide. I planned on making my game in an open area (in a desert environment) with a single directional light but later in the level, I wanted the player to enter a cave which was lighted by multiple glowing crystals. Forward rendering path would be a more suitable choice for the former and Deferred for the latter level design but in the end, I have decided to go with Forward rendering path as the majority of the game was in an open area with a single light source.

## Lighting Strategy 
From the get-go, I wanted to get my hands dirty with Realtime Global Illumination (RGI). I didn't prefer Mixed Lighting with baked shadows because I planned to use a dynamic directional light to simulate a day cycle. Unfortunately, I had to give up on this idea since creating a day/night cycle meant I couldn't use a set skybox. After my extensive search on dynamic skyboxes, I have found advanced methods to simulate a day/night cycle but for that time it was over my head. So I settled on a fixed directional light and focused my efforts on other tasks.

![ROTSP-light-probes](/assets/screen-shots/ROTSP-light-probes.png)

In my scene, I had areas which contain comparatively smaller objects. Making these objects Lightmap Static and including them in RGI precomputations would increase the RGI precomputation time unnecessarily. That's why I used Light Probes for the indirect light calculations of these objects.

## Shadows

![ROTSP-sun-set](/assets/screen-shots/ROTSP-sun-set.png)

Initially, I wanted to create a desert scene with a sunset. Over many experiments and iterations, I have decided that a midday sun would reflect the spirit of the game better. While I was dealing with different times of the day one topic kept coming to my attention: shadows. My desert scene was more than 2000x2000 unit scale with at least a couple of hundred meshes. Undoubtedly shadows played a huge role in this case. They created performance issues but even more notable than that, they produced light artifacts. The light was leaking through the meshes and the meshes were forming unrealistic shadows. Taking advantage of my lighting settings, I disabled Receive Shadows and Cast Shadows settings of insignificant objects. For important objects, I set Two Sided Cast shadows which eliminated light leaking artifacts. I also tweaked directional light Bias settings to find a sweet spot between realism and shadow artifacts.

![ROTSP-lights](/assets/screen-shots/ROTSP-lights.png)

## Camera and Post Processing Effects
I enabled Occlusion Culling, MSAA, and HDR camera settings and added Post Processing Stack on my main camera for post-processing effects. 

Initially, I added ambient occlusion to my game. When I realized it was causing a GPU bottleneck after my Profiler analysis, I decided its performance cost was outweighing its visual contributions and turned it off. Apart from SSAO, I'm also using Bloom for golden sands effect, Color Grading and small amounts of Grain effects. Using Color grading I created a warm, a little bit saturated lower contrast effect.

## Optimizations
I used 3 Leveled LOD objects with smooth transitions to lower my vertex count for distant objects. 

I pre-created my sand storm particle systems and coded a controller script to loop them at random positions around the player. This way I was able to reuse my particle systems without destroying and instantiating them again. This prevented random FPS drops during the gameplay but still, when the particle system covered the majority of the camera frustum it caused FPS drops. To tackle this problem, I tweaked my particle system settings so that I could fake the same visual effect with lower particle count. This fixed the FPS drop issues.

After extensive Profiler and Frame Debugger analyses, I realized the need for reducing the Dynamic Batch count. 

![ROTSP-scene-window](/assets/screen-shots/ROTSP-scene-window.png)

Since I have an enormous scene (2000x2000 units with hundreds of meshes) RGI precomputation time was unacceptabily high. This raised the need to optimize the UV charts and Clustering of my objects. I have already mentioned that I disabled GI for small objects. For the terrain and mountain objects, I have tweaked Lightmap Parameters and RealtimeUV settings. I balanced the visual consequences of my optimizations and precomputation time. As a result without sacrificing too much from visual fidelity, I managed to reduce precomputation time from 14 hours to 2.5 minutes.

![ROTSP-optimization](/assets/screen-shots/ROTSP-optimization.png)

## Gameplay, Sounds & Narrative
I have mentioned I wanted my game to feel like a journey. Visual storytelling was one of the main points of the game. Hand in hand with the story, the narrative was given as a monologue which I have voice acted. I have used subtle sand storm sounds for the ambient and used sounds for walking on the sand. Unfortunately gameplay wise, the game was plain. The character could only walk and look around. Yet game kept teasing the player with many planned but cut out game mechanics. My main focus was on creating an immersive world and beautiful visuals and as a result, other aspects of the game suffered. One could say my project is a trailer of a much bigger game and to be honest I never intended it to be more than that in the limited time I had.

## Story & Character
Square Pyramid from Pyramids Dynasty has awakened from his pyramid tomb after centuries-long sleep. The marvelous pyramids built in the Golden Age of Pyramids have been eroded by thousands year long ignorance and even worse there is not a single triangle-based Shape left. Square Pyramid, who once was a powerful and worshipped Triangle-Caster, now left without any powers and he has to adapt to this new hexahedron world. He is on a quest for answers. Where has his kin gone? What happened to the Pyramids Dynasty and why was he awaken? He had to find many triangles but all he could see was squares...
Square Pyramid is a Triangle-Caster and he is from loyal Pyramid blood. He is smart, calm and a rational character. Though you should care your words around him, especially if you are not from the honorable Triangle family.

“A Pyramid?! (Said as an insult) I will wipe your abominable shape from our perfect Cube world!”

## Screen Shots

![ROTSP-gameplay](/assets/screen-shots/ROTSP-gameplay.png)
