---
layout: post
title: "Editor layout of Unity GPU Ray Tracer"
categories: editor, graphics, unity, gpu, ray tracing
author: "Doruk Coşkun"
---

In this post, I will go over the components and scripts used in the editor. You can check my [GitHub repo](https://github.com/Doruk-Coskun/Unity-Ray-Tracer) for the whole project.

## Scene Layout and Hierarchy Window

![Editor-hierarchy-layout](/assets/screen-shots/Editor-Hierarchy-Layout.png)

As it can be seen from the scene hierarchy window, our scene consists of Scene Geometry, Scene Cameras, and Scene Lights. 

SceneParser object contains the SceneParser script which is responsible for parsing the scene and storing the data in a custom Scene object. Scene object data is then used while setting the buffers of our Compute Shader. SceneParser script will be further explained.

### Object Components

![Editor-Mesh-Inspector](/assets/screen-shots/Editor-Mesh-Inspector.png)

Using the Transform component of each gameobject, their transform, rotation, and scale can easily be set. 

For Scene Geometry objects, Mesh property of the Mesh Renderer component is used to get the vertex positions of the object. In the case of spheres, their mathematical representation is used. The radius of the spheres is taken as half of their x scale. This way, cheaper yet more accurate representation of the spheres is achieved. Although, this representation may introduce problems while implementing ovoid objects. Objects also have a RayTracingMaterial. Ambient, diffuse, specular, Phong exponent, mirror reflectance, transparency, and the refraction index of the object can be set here. xyz components represent RGB color reflectance coefficients respectively.

Currently, only directional light and point lights are supported. Direction (directional light only), intensity, and color of the lights can be set in the Inspector.

![Editor-Camera-Inspector](/assets/screen-shots/Editor-Camera-Inspector.png)

Camera object has Transform and Camera components. Field of view and projection of the camera can be set here. This object also contains the RayTracingMaster script. This script is responsible for creating the render texture and the compute shader. It also sets the compute shaders buffers and finally displays the render texture. For camera objects to work, they have to be the child of SceneCameras gameobject.

![Editor-SceneParser-Inspector](/assets/screen-shots/Editor-SceneParser-Inspector.png)

SceneParser object can be used the play with various render settings. Camera No slider can be used to change the active camera. Using Generate Scene from XML option, an XML file can be parsed to generate the scene in the editor mode. Some example XML can be found in the _SceneXMLs folder. GenerateScene Scene Editor option lets you update the scene during play mode as you desire, or you can enable Realtime Update. This feature is one of the main reasons I wanted to implement a ray tracer in Unity. We can easily create dynamic scenes and see how they play out in the play mode.

PNGConverter script saves the first rendered frame as a PNG into Resources folder. I wish to work on this script in the future and be able to take videos of our render.

One thing to note here is Inspector Conditional Hide code I have used is acquired from Brecht Lecluyes. You can also find them [here](https://github.com/SebLague/Procedural-Planets/tree/master/Procedural%20Planet%20Hide%20Editor), used by Sebastian Lague. SebLague has an amazing procedural planet generation tutorial which you can check out.

### Scripts

SceneParser script locates scene objects and stores their data in a SceneData object. For SceneParser to work properly, scene objects have to be correctly categorized as it was in the Hierarchy window. 

There are some important things to note related to my SceneParser script and the way I parse the data: To set up my camera rays in the compute shader I use CameraToWorldMatrix and CameraInverseProjectionMatrix. This, unfortunately, arrises problems I can't specify at the moment and requires a rework. Another problem I faced was while acquiring the vertex data of the meshes. Since vertex data in Mesh is stored in object space, they have to be rotated, scaled and translated according to the Transform component of the object.

SceneData class is organized in the way it is so that RayTracingMaster script can set the GPU structured buffers easier. Lists of Structs can be set to ComputeBuffers using ```ComputeBuffer.SetData(int struct_count, int struct_size_as_float)``` function. These buffers will be later used in compute shader for our calculations.

