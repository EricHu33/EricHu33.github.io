---
title:  "Blob Shadow System In Unity"
description: "Blob Shadow System In Unity"
categories:
  - Blog
header:
  image: /assets/images/hp_header.png
tags:
  - Unity
  - Shader
  - Blob
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

Option 1 is the easist way to do blob shadow. But the shadow is possible to penetrate into other surface when the surface is not an even surface. Unity projector is quite expensive as well and won't work in unity srp. 

In this blog I'll using a decal shader that can projected the shadow image on an uneven surface. While draw many shadow in a single draw through gpu instancing.

# Decal Shader
The first decal shader came into my mind is the sprays in counter-strike. [Here's a nice article talk about decal shader](https://www.ronja-tutorials.com/post/054-unlit-dynamic-decals/#naive-world-reconstruction). By resconstuct the world position and the decal's object position. We can achieve spray-like effect or projected shadow effect on an uneven surface. 
> 截圖  decal shader

# Stencil Buffer
In order to prevent the shadow image project on the character itself. I set a stencil value while render the character. Then, mask out the decal while the stencil value is equal to the one set by character's shader.
> 截圖 stencil buffer 效果
<div class="notice--info text-justify" markdown="1">

</div>
# GPU Instancing
We may have many dynamic object on the scene at the same time and require lots of draw calls. Seems quite expensive until gpu instancing came to the rescue once again.

To force enable instancing. I'll use the [Graphics.DrawMeshInstanced](https://docs.unity3d.com/ScriptReference/Graphics.DrawMeshInstanced.html) API. To update all shadow's instance position. we need prepare/update an array of Matrix4x4 that represent each instance's position, rotation, and scale.

<div class="notice--info " markdown="1">
For actuall usage of DrawMeshInstanced/DrawMeshInstancedIndirect, you can check the [This tutorial](https://toqoz.fyi/thousands-of-meshes.html) by Michael Palmos.
</div>

> 截圖  GPU Instancing 效果

# Follow the Owner Object
Assuming all blob shadows owner transform are dynamic, moving object. We will need to update the elements of Matrix4x4 array frequently .Instead of using a big for loop that iterate the array. I use a [IJobParallelForTransform](https://docs.unity3d.com/ScriptReference/Jobs.IJobParallelForTransform.html) job to access all shadow owner's transform position in parallel. This way we can calculate each shadow's Matrix4x4.TRS value parallelly into an native array. Then parse the native array into our managed matrix4x4 array after the job complete.  

# Find the Projected Point on Surface
To find the correct projected point on the ground. We need to cast a ray downward to the ground for each dynamic objects. Since we already use job system to calculate the matrix4x4 value. We can use [RaycastCommand](https://docs.unity3d.com/ScriptReference/RaycastCommand.html) to do all the ray casting which can naturally work with other jobs.

> 截圖  raycast

# Result
This is the final result of combining techniques I metioned above. We now have a system that draw many fake shadow at one time, with the ability to follow dynamic objects and projected on an uneven surface!

<iframe width="100%" height="400" src="https://user-images.githubusercontent.com/13420668/135239766-328378aa-ca6c-4a0f-8a41-46c0cd872049.mp4" frameborder="0" allowfullscreen></iframe>

## Conclusion



#### References