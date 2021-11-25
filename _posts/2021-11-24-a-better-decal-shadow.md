---
title:  "A Better Decal Shadow in Unity"
description: "A Better Decal Shadow in Unity"
categories:
  - Blog
header:
  image: https://user-images.githubusercontent.com/13420668/143266244-4c1127eb-ab93-4d47-82cb-638af0d7c27d.png
tags:
  - Unity
  - Shader
  - Decal Shadows
  - URP
  - GrabPass
  #caption: "Photo credit: [**Unsplash**](https://unsplash.com)"
---
Recently I've encounter an interesting problem :

The renderer for characters in the scene needs to receive real time shadows. BUT must not receive it's own shadow in order to maintained the overall art style. While still casting shadow to other surface in the same time.

So, after some quick research I decide to use a decal shader to create the fake shadow. This way I can leave the **Cast Shadows** option in renderer as **off** and still have character's shadow on the ground.

Instead of use simple blob(or circle) image. I setup an camera that positioned & rotated exact as light source. Create a render texture as target. Then pass the render texture to the decal shader.

![the  camera output](https://user-images.githubusercontent.com/13420668/143265352-ab3ce562-1c79-4420-89fb-8e15f6710105.png)

Now I have a fake shadow that looks same shape as my character.
![enter image description here](https://user-images.githubusercontent.com/13420668/143266244-4c1127eb-ab93-4d47-82cb-638af0d7c27d.png)

Let's change the decal's color into pure shadow color and it will be done. right?
Well, yes and no.

![enter image description here](https://user-images.githubusercontent.com/13420668/143266971-ba2edfea-4b0e-44af-9faa-a74f62d6b8fb.png)

Everything looks good until the fake shadow intersect with other real time shadow in the scene.


![enter image description here](https://user-images.githubusercontent.com/13420668/143364574-87f6edd3-308e-49d2-9563-b03fda14187c.png)
<div  class="notice--info "  markdown="1">
The overlapping part become so noticeable even with alpha blend transparent.
</div>

The main issue is that the real time shadows's color on different surface will differed base on the surface it cast on. The floor shadow that cast on darker grid will darker than the shadow cast on lighter grid. The shadow cast on the right side of the above screenshot will be tint to a more brownish shadow than other part of the screen.

Right now, my decal shadow can only use a single color as a whole. This create the noticeable color difference when it intersect with real time shadows.

## FIX
The goal is to make the decal shadow looks same as other real time shadow in the scene. In order to do that I use a custom renderer feature to grab the whole screen in to a texture before render the decal.

![enter image description here](https://user-images.githubusercontent.com/13420668/143274662-0c4c9560-8649-4bba-9689-309a1b46fce3.png)

<div  class="notice--info "  markdown="1">
I use unity's Universal Render Pipeline to specific when to grab the whole screen. For built-in RP,  You can use GrabPass or command buffer to achieve the same result.
</div>

Pass the screen texture into decal shader. sample the texture with screen position.
We now have the surface color for each pixel the decal projected on.
Calculate the surface color as it is shadowed, take into account the lightmap and environment color in lighting setting as well.


    #ifdef LIGHTMAP_ON
        lightMapUV = lightMapUV * unity_LightmapST.xy + unity_LightmapST.zw;
        bakedGI =  SAMPLE_GI(lightMapUV, half3(0,0,0), i.normal);
    #else
        bakedGI =  SampleSH(float3(0,1,0));
        half4 final = grabColor * lerp(float4(bakedGI, 1), 0, 1-shadowAttenutation);
    #endif

![enter image description here](https://user-images.githubusercontent.com/13420668/143275200-cef8e53e-c03f-47db-88e1-06f946672143.png)
<div  class="notice--info "  markdown="1">
The decal shadow now display different color based on the surface it cast on.
</div>
To prevent shadow the surface twice on the overlapped area, check if the pixel has real time shadow through sampling shadowmap. If the pixel is shadowed by shadowmap. Don't draw the decal on it.

![enter image description here](https://user-images.githubusercontent.com/13420668/143277539-0d4b241d-2cb2-4d8e-928f-818e77710752.png)![enter image description here](https://user-images.githubusercontent.com/13420668/143276119-f0221780-b029-4159-997b-6d8bd7475b15.png)

    //you can either clip it or use shadowAttenutation as alpha
    return  half4(finalColor.rgb, shadowAttenutation);

## DEMO
The color is not completely same as real time shadow since I pass some hard coded parameter when calculating shadow color(ex : world normal) .
But right now it's good enough to trick player's eye in most situation.
<iframe  width="100%"  height="500"  src="https://user-images.githubusercontent.com/13420668/143366385-47927e43-3a48-4051-bfe2-2dc271791659.mp4"  frameborder="0"  allowfullscreen></iframe>

## REFRENCES

- [how-to-take-a-multiply-shadow-and-make-it-blend-with-shadows](https://forum.unity.com/threads/does-anyone-know-how-to-take-a-multiply-shadow-and-make-it-blend-with-shadows-instead-projectors.974520/)

- [URP Custom Grab Pass Renderer Feature](https://gist.github.com/EricHu33/d3dfb936b5001fac79a407c7aca0885e)