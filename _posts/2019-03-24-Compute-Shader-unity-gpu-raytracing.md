---
layout: post
title: "Compute Shader Basics of Unity GPU Ray Tracer"
categories: compute shader, graphics, unity, gpu, ray tracing
author: "Doruk Coşkun"
---

Now that we have taken a look into Editor layout, we can move on to the most crucial part of my Unity ray tracer, the compute shader. You can check my [GitHub repository](https://github.com/Doruk-Coskun/Unity-Ray-Tracer) for the whole project.

## Primitive Structs

First of all, I set up the structs I will use in my ray tracer code. As I progress and implement new features these primitive structures will probably expand and change.

## Ray Tracing Logic


```csharp
Ray ray = CreateCameraRay(uv);

float3 result = float3(0.0f, 0.0f, 0.0f);

for (uint i = 0; i < _MaxRecursionDepth; i++) 
{
    RayHit hit = Trace(ray);
    result += ray.energy * Shade(ray, hit);

    if (!any(ray.energy))
    {
        break;
    }
}

Result[id.xy] = float4(result, 1);
```


Since compute shaders do not support recursive function calls we have to change the way we think. In the ray tracer kernel, we create the camera ray according to the camera position and projection we acquired and the pixel position. `Trace(Ray)` determines whether the ray hit an object and stores its data in a `RayHit`. When `Shade` function returns, the color value of the closest intersection is added to the result. `Shade` function also updates the ray we gave to it as a parameter. Rather than making recursive `Shade` calls we implement an iteration of `Trace` and `Shade` calls. Rays direction and energy is updated and then used in the second iteration. This continues until we reach the max recursion depth. Right of the bat, we can notice that when the ray hits an object which doesn't generate a new ray (such as only diffuse surfaces, or rays which don't intersect with any surfaces) other cycles of the iteration are pointless. Jeremy Cowles in his [blog](https://medium.com/@jcowles/gpu-ray-tracing-in-one-weekend-3e7d874b3b0f) discusses this issue and introduces the ray scheduler. I might consider a similar approach in the following weeks.


```csharp
float R = CalculateFresnelEquation(ray, hit);

bool outside = dot(ray.direction, hit.normal) < 0; 
float3 bias = 0.001 * hit.normal; 

float3 refractDir = normalize(CalculateRefractedRay(ray, hit));

// Calculate and add refracted color then return reflected ray to main.
if (R > 0.5) 
{   
    Ray refractedRay = CreateRay(hit.position, refractDir);
    refractedRay.origin = outside? hit.position - bias : hit.position + bias;
    refractedRay.energy = (1 - R) * ray.energy;

    RayHit refractedHit = Trace(refractedRay);
    refractedRay.energy *= outside? pow(_MaterialList[hit.materialID].transparency, length(abs(refractedHit.position - hit.position))) : 1;
    color += refractedRay.energy * ShadeOnce(refractedRay, refractedHit);

    ray.origin = outside? hit.position + bias : hit.position - bias;
    ray.direction = outside? reflect(ray.direction, hit.normal) : reflect(ray.direction, -hit.normal);
    ray.energy *= R;
    ray.energy *= outside? 1 : pow(_MaterialList[hit.materialID].transparency, 2);
}
else
// Calculate and add reflected color then return refracted ray to main.
{
    Ray reflectedRay = CreateRay(hit.position, reflect(ray.direction, hit.normal));
    reflectedRay.origin = outside? hit.position + bias : hit.position - bias;
    reflectedRay.direction = outside? reflect(ray.direction, hit.normal) : reflect(ray.direction, -hit.normal);
    reflectedRay.energy =  R * ray.energy;

    RayHit reflectedHit = Trace(reflectedRay);
    reflectedRay.energy *= outside? 1 : pow(_MaterialList[hit.materialID].transparency, length(abs(reflectedHit.position - hit.position)));
    color += reflectedRay.energy * ShadeOnce(reflectedRay, reflectedHit);

    ray.origin = outside? hit.position - bias : hit.position + bias;
    ray.direction = refractDir;
    ray.energy *= (1 - R);
    ray.energy *= outside? pow(_MaterialList[hit.materialID].transparency, 2) : 1;
    ray.isInObject = !ray.isInObject;
}
```


`Shade(Ray, RayHit)` function calculates ambient, diffuse and specular (Blinn-Phong) shading. If the `RayHit` material is mirror-like, the initial ray is updated and used in the next iteration. So far so good but when we introduce refraction to the equation we face yet another limitation. When a ray hits a transparent surface, it both reflects and refracts. Color values of these rays are added according to the Fresnel equation. Since we don't have recursive calls, we can not just call `Shade` function for these rays again. Thus I have decided to compute one of the resulting rays using `ShadeOnce` function which is basically a depth one version of `Shade` function. Ray with bigger Fresnel value has a higher contribution to the pixel color so it replaces the initial ray and is computed in the next iteration. Ray with smaller Fresnel value is calculated using `ShadeOnce` function and added to the return color value of the `Shade` function. By using this method I managed to get good enough results. One bug I faced and couldn't solve yet is: mirror reflection calculations in `ShadeOnce` produces artifacts on transparent objects. I will tackle this issue in the following weeks.

Beer's Law is introduced to compute the attenuation of the ray's energy inside a transparent object.

## Some Small Details

There are some small yet worth mentioning points I want to talk about.

- As of now, I don't precompute and store the normal values of the triangles. Thus each ray computes the same triangle normals over and over again. As the triangle count increases it may introduce some performance issues.

- While dealing with transparent objects I check whether the ray is inside or outside of the object. I determine the origin of my ray according to this.

- There are more than I like if-else statements in my code. The performance cost of too many conditionals are not negligible and I believe I need to refactor my code to optimize further.

Here you can compare my result (former) to the referance image (latter)

![my-result](/assets/screen-shots/me_CornellBox_glass.png) 


![reference-image](/assets/screen-shots/cornellbox_glass.png)
