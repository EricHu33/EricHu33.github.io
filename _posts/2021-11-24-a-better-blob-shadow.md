---
title:  "A Better Blob Shadow in Unity"
description: "A Better Blob Shadow in Unity"
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

The overlapping part become so noticeable even with alpha blend transparent.
 ![enter image description here](https://user-images.githubusercontent.com/13420668/143268670-e4e16254-e4f5-4490-89eb-f741f27159ce.png)

The main issue is that the real time shadows's color on different surface will differed base on the surface it cast on. The floor shadow that cast on darker grid will darker than the shadow cast on lighter grid. The shadow cast on the right side of the above screenshot will be tint to a more brownish shadow than other part of the screen.

Right now, my decal shadow can only use a single color as a whole. This create the noticeable color difference when it intersect with real time shadows.

## Fix
The goal is to make the decal shadow looks same as other real time shadow in the scene. In order to do that I use a custom renderer feature to grab the whole screen in to a texture before render the decal.
![enter image description here](https://user-images.githubusercontent.com/13420668/143274662-0c4c9560-8649-4bba-9689-309a1b46fce3.png)

<div  class="notice--info "  markdown="1">
I use unity's Universal Render Pipeline to specific when to grab the whole screen. For built-in RP,  You can use GrabPass or command buffer to achieve the same result.
</div>

Pass the screen texture into decal shader. sample the texture with screen position.
We now have the surface color for each pixel the decal projected on.
Calculate the surface color as it is shadowed, take into account the lightmap and environment color in lighting setting as well.

The decal shadow now display different color based on the surface it cast on.
![enter image description here](https://user-images.githubusercontent.com/13420668/143275200-cef8e53e-c03f-47db-88e1-06f946672143.png)

To prevent shadow the surface twice on the overlapped area, check if the pixel has real time shadow through sampling shadowmap. If the pixel is shadowed by shadowmap. Don't draw the decal on it.
![enter image description here](https://user-images.githubusercontent.com/13420668/143277539-0d4b241d-2cb2-4d8e-928f-818e77710752.png)
![enter image description here](https://user-images.githubusercontent.com/13420668/143276119-f0221780-b029-4159-997b-6d8bd7475b15.png)

# Demo