---
title:  "Health Bar Shader With Procedural Ticks"
description: "Health Bar Shader With Procedural Ticks"
categories:
  - Blog
header:
  image: /assets/images/hp_header.png
tags:
  - Shader
  - Health Bar
  - SDF
  #caption: "Photo credit: [**Unsplash**](https://unsplash.com)"
---
One request from my latest project is that we need a health bar on top of every character that represent it's hp.

After some experiments we found that it's not a good idea to put hp text inside a progress bar. Since there are lots of text UIs on the screen alreay.  

To give player a good understand of their current health. League of Legends and many other games use ruler-like line segments to represent the character's health point.
![image](/assets/images/lol_hp.png)
> A screenshot of League of Legends

Each thin 'tick' represent 100 health point and it will show a bigger tick every 1000 health point.
One thing to notice is the thin tick will disappear once the character's health reach 5000.

# Why Not Using UGUI?
Since health bars will update constantly during runtime. Using UGUI means lots of canvas rebuild. Usually you have 2 options. Either have a clean/separate canvases layout for all dynamic UIs and static UIs. Or just avoid put those frequent updated object into canvas. Here I choose the later one. 

# Progress Bar
The fill amount can easily drawn using UV of the mesh. Simply feed a float point between 0 and 1. then check it with uv.x.
{% highlight csharp %}
   // Property 
   _Fill ("Health", Range(0,1)) = 1
   // Declared
   half _Fill;
   
   // vertex/pixel shader
   half valueMask = fill > i.uv.x;
   half4 colorRgb = lerp(backgroundColor.rgb, fillColor.rgb, valueMask);

{% endhighlight %}


# Ticks on Bar - First Attempt
Next thing to do is to draw lines on the health bar. 
Here I draw one line every 300 health points. 

{% highlight csharp %}
   //Pixel shader
   ...
   ...
   //draw one line every 300 health points
   float lineNum = _MaxHp/300.0;
   
   //if d > 0, draw line
   half d = 1-step( _LineWidth , fmod(i.uv.x, 1.0 /lineNum));
{% endhighlight %}

![image](/assets/images/legacy_hp.png)

> A health bar that is currently 1000/1000 HP, Showing big tick every 300 HP.

## Ticks on Bar - Apply Anti-Aliasing

Everything looks fine before I hit the play button in unity editor and only get worse once I build the project into mobile devices.


![image](/assets/images/show_noAA.png)

> The ticks and outline on the health bar starts to flickering and has visible artifacts . Turns out it's because lacks of anti-aliasing.

To fix the artifact, instead of directly using UV's x value to draw the ticks. I use [2D Signed Distance Field from Ronja's tutorial](https://www.ronja-tutorials.com/post/034-2d-sdf-basics/) to draw line. By using SDF, we can apply local AA to each tick using fwidth() function!

{% highlight csharp %}
   //Pixel shader
   ...
   ...
   //_ValuePerGap = 300
   float lineNum = maxHp / _ValuePerGap;
   
   float distanceChange = fwidth(dist) * 0.5;
   float majorLineDistance = abs(frac(dist / lineDistance + 0.5) - 0.5) * lineDistance;
   
   float d = 1-smoothstep(lineThickness - distanceChange, lineThickness + distanceChange, majorLineDistance);

{% endhighlight %}

![image](/assets/images/hp_AA.png)
> The artifacts are greatly reduce!


## Conclusion
Since I didn't use instancing (because some transparent sorting issue), I get one extra draw call for each health bar. Though it didn't impact the performance much. It should work even better in URP with SRP batcher. 

SDF really open my eyes. we can draw many different shapes just by playing with SDF. Will spent more time study it.


#### References

 - [2D Signed Distance Field from Ronja's tutorial](https://www.ronja-tutorials.com/post/034-2d-sdf-basics/)
 - [使用Shader实现LOL血条分割线效果](https://zhuanlan.zhihu.com/p/386528412)
 - [Healthbars, SDFs & Lighting • Shaders for Game Devs [Part 2] from Freya Holmér](https://www.youtube.com/watch?v=mL8U8tIiRRg)