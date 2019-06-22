---
layout: post
title: "Acceleration Structures"
categories: raytracing
author: "Doruk Coşkun"
---

![Bunny1](/assets/screen-shots/Bunny1.png)

In this post, I want to talk about my work in optimizing my ray tracer. It has been a challenging journey, but I feel the pride to announce that I have managed to achieve real-time ray tracing.

The first step in optimizing my ray tracer was implementing BVH acceleration structures. Parsing scene data in the Unity's editor and making the rendering in the GPU posed a challenge. I decided to first:

* Construct a BVH tree of each mesh in the CPU according to the mesh data acquired by my `SceneParser`.

* Transform those BVH trees into a single array of `LinearBVHNodes` and store that array as a `StructuredBuffer` in the GPU.

* Trace the flattened `LinearBVHNode` array in the GPU.

Since I was aiming for a real-time ray tracer, I wanted to construct the BVH trees once before the rendering began. The construction time of my BVH trees was not crucial, but the trees had to be efficient during rendering.

During my searches, I have encountered many different solutions for the construction of the BVH trees. One of them suggested precomputing the BVH trees and storing them as `ScriptableObject`'s of Unity. This way, recalculation of the scenes was not necessary.

I have created `BVHAcc` script which handled the construction of the BVH trees. Initially, I have used EqualCounts algorithm suggested in the ![Physically Based Rendering](http://www.pbr-book.org/3ed-2018/contents.html) book of Peter Sherley. The algorithm divided the primitives (in our case, the triangles) into two equal count lists. It determined the split axis according to the volume of the members of the primitives list and divided the list into two equal counted primitivers list by the longest axis.

Further, into the development, I have also started working on implementing Surface Area Heuristics BVH tree construction. I have also provided simple UI dropdown to switch between different construction algorithms.



I have also implemented a simple Unity script to visualize my BVH tree nodes and manually check the correctness of my BVH trees. 

![DrawGizmos1](/assets/screen-shots/DrawGizmos1.png)

![DrawGizmos2](/assets/screen-shots/DrawGizmos2.png)

I have added private int FlattenBVHTree(BVHBuildNode node, ref int offset) function to store my BVH tree as an array of LinearBVHNode's. Each LinearBVHNode saved the bounds of its bounding volume, primitive offset if it is a leaf node (hence, pointing to a triangle) and second children offset if its an inner node. I have constructed the array in a way that next LinearBVHNode was the first child of the previous node. 

After flattening the BVH trees and appending them to one another, I have stored the BVH node offset of each mesh in MeshData class. I have sent this array of LinearBVHNodes as a StructedBuffer to the GPU.

Tracing the BVH array in the GPU was by far the hardest part. In an infinite loop, I have started to intersect rays with bounding boxes of the nodes. If an inner node was hit, its second child was stored in an index array to be checked later. When no more BVH nodes were hit and the `uint nodesToVisit[64];`  array is empty, then the while terminates. Recorded `RayHit` is used for light calculations.

# Other acceleration methods

I have used the fastest ray-axis aligned bounding box implementation method on the internet, and you can check the detailed explanation in the Roman Wiche's blog.

I have used the least branching ray-triangle intersection algorithm I could find.

I have stripe aligned my StructuredBuffers by using 32 bytes sized LinearBVHNode's. You can check this ![article by Evan Hart](https://developer.nvidia.com/content/understanding-structured-buffer-performance) to gain further information about increasing memory access performance of StructuredBuffers.

And finally you can check the results here. I have used the Stanford Bunny in my demo scenes. I have decimated polygon count down to 696 polygons using Blender and rendered in 600x600 resolution.

![Bunny2](/assets/screen-shots/Bunny2.png)
