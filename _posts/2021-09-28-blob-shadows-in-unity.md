---
title:  "Blob Shadow System In Unity"
description: "Blob Shadow System In Unity"
categories:
  - Blog
header:
  image: https://user-images.githubusercontent.com/13420668/135382134-651d6fa9-ef4e-4a31-b394-ecdb647221f5.png
tags:
  - Unity
  - Shader
  - Blob
  - Fake Shadow
  - Shadows
  - Instancing
  - Job System
  #caption: "Photo credit: [**Unsplash**](https://unsplash.com)"
---
Shadows is essential for video games. It's also quite expensive to compute and draw a pretty shadow for your character.
Developers had been using fake shadow since the dawn of video games. AKA Blob Shadows.

According to [arm's document](https://developer.arm.com/documentation/102109/0100/Fake-as-much-as-possible) - 
Here are some ways to implement fake shadows:
> 1. Use a 3D mesh, plane, or quad, that is placed under the character and applying a blurred texture to it.
> 2. Use a Unity Built-in Render Pipeline feature to apply dynamic blob shadow using projector. This method is more expensive than using a quad under a character, and is not available in the Universal Render Pipeline that is recommended for mobile.
> 3. Write custom shaders to create more sophisticated blob shadows.

Option 1 is the easiest way to do blob shadow. But the shadow is possible to penetrate into other surface when the surface is not an even surface. Unity projector is quite expensive as well and won't work in unity srp. 

In this blog I'll using a decal shader that can projected the shadow image on an uneven surface. While draw many shadow in a single draw through gpu instancing.

# Decal Shader
The first decal shader came into my mind is the sprays in counter-strike. [Here's a nice article talk about decal shader](https://www.ronja-tutorials.com/post/054-unlit-dynamic-decals/#naive-world-reconstruction). By resconstuct the world position and the object position inside the decal mesh(box). We can achieve spray-like effect or projected shadow effect on an uneven surface. 

<img src="https://user-images.githubusercontent.com/13420668/135374637-3a47c525-6ac7-41f5-8447-c90ae3b259c1.jpg" alt="drawing" width="45%"/> <img src="https://user-images.githubusercontent.com/13420668/135374923-256fe028-ae64-4c07-825e-e7a96f76b752.jpg" alt="drawing" width="45%"/>
> A example of deferred decals in unity’s example project. Each decal is implemented as a box here, and affects any geometry inside the box volume.

# Stencil Buffer
In order to prevent the shadow image project on the character itself. I set a stencil value while render the character. Then, mask out the decal while the stencil value is equal to the one set by character's shader.

<img src="https://user-images.githubusercontent.com/13420668/135374947-7bea1220-6437-4b7b-9d02-b6c05ba656cc.png" alt="drawing" width="45%"/>Without stencil
<img src="https://user-images.githubusercontent.com/13420668/135374949-35e28ee8-3e42-4b6a-a762-63484d929bdc.png" alt="drawing" width="45%"/>Compare stencil buffer using NotEqual

<div class="notice--info text-justify" markdown="1">
For stencil operation you can check [ShaderLab command: Stencil](https://docs.unity3d.com/Manual/SL-Stencil.html)
</div>

# GPU Instancing
We may have many dynamic object on the scene at the same time and require lots of draw calls. Seems quite expensive until gpu instancing came to the rescue once again.

To force enable instancing. I'll use the [Graphics.DrawMeshInstanced](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstanced.html) API. To update all shadow's instance position. we need prepare/update an array of Matrix4x4 that represent each instance's position, rotation, and scale.

![image (4)](https://user-images.githubusercontent.com/13420668/135374954-3f805722-e6fc-4741-8fe4-efd9f51ee5ee.png)
> Draw all decal shaders in 1 set pass call

<div class="notice--info " markdown="1">
For actuall usage of DrawMeshInstanced/DrawMeshInstancedIndirect, you can check the [This tutorial](https://toqoz.fyi/thousands-of-meshes.html) by Michael Palmos.
</div>

# Follow the Owner Object
Assuming all blob shadows owner transform are dynamic, moving object. We will need to update the elements of Matrix4x4 array frequently. Instead of using a big for loop that iterate the array. I use a [IJobParallelForTransform](https://docs.unity3d.com/ScriptReference/Jobs.IJobParallelForTransform.html) job to access all shadow owner's transform position in parallel. This way we can calculate each shadow's Matrix4x4.TRS value parallelly into an native array. Then parse the native array into our managed matrix4x4 array after the job complete.

<div class="notice--info " markdown="1">
The Unity C# Job System lets you write simple and safe multithreaded code that interacts with the Unity Engine for enhanced game performance. Check officail manual [here](https://docs.unity3d.com/Manual/JobSystem.html)
</div>

# Find the Projected Point on Surface
To find the correct projected point on the ground. We need to cast a ray downward to the ground for each dynamic objects. Since we already use job system to calculate the matrix4x4 value. We can use [RaycastCommand](https://docs.unity3d.com/ScriptReference/RaycastCommand.html) to do all the ray casting which can naturally work with other jobs.

![image (1)](https://user-images.githubusercontent.com/13420668/135379258-09d463a4-5546-4780-be43-1fc2692e009c.png)
> cast a ray downward to find the desired projected point for decal shadows

# Demo
This is the final result of combining techniques I mentioned above. We now have a system that draw many fake shadow at one time, with the ability to follow dynamic objects and projected on an uneven surface!

<iframe width="100%" height="400" src="https://user-images.githubusercontent.com/13420668/135380490-6a89be72-4c6e-4fe6-a27f-f5e8a1104260.mp4
" frameborder="0" allowfullscreen></iframe>

## Conclusion
Blob shadows has a long histroy and different implementation throghout the development of games. The technique itself is old. But still very useful specially for mobile game. I'm looking forward to study more way to draw a shadow.

#### References

 - [Job System Tutorial](hhttps://www.jacksondunstan.com/articles/4796)
 - [Improve Performance with C# Job System and Burst Compiler in Unity](https://realerichu.medium.com/improve-performance-with-c-job-system-and-burst-compiler-in-unity-eecd2a69dbc8)
 - [Unlit Dynamic Decals/Projection](https://www.ronja-tutorials.com/post/054-unlit-dynamic-decals/)
 - [UnityURPUnlitScreenSpaceDecalShader](https://github.com/ColinLeung-NiloCat/UnityURPUnlitScreenSpaceDecalShader)