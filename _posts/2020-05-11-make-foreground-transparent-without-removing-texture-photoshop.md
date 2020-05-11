---
layout: post
title: "Making a foreground subject transparent without removing textures in Photoshop"
categories: Graphics
author: "Muhammad Taha"
---

Recently while working on a site, I needed some t-shirt mockups. The site had a product designer and I needed a lot of different colors for each product. 
So, I settled on using some css to color transparent parts of the mockup. [This is the PSD that we'll be working on as a sample](https://graphictwister.com/flat-t-shirt-mockup/).

Now, I could just open up its png in Photoshop and pick the t-shirt color range using magic tool, delete it and get a transparent shirt. 
But, it was also removing the texture. And if I was careful enough to exclude the texture from my selection, then the texture had a prior color of its own.. so basically the shirt itself got transparent but the texture on top of it had color so it didn't work out at all.
And besides, if you have a layered PSD file then why experiment with PNG anyway.

So, to get started:
1. Open the PSD file that you downloaded from the above link.
2. Hide layers to get the exact appearance you want. But don't hide the color layer (if exists). We need that. ![hide-useless-layers][hide-useless-layers]
3. Select layers that you want to make transparent. Please do NOT select Background layer as we need that. Right-click and Merge Layer. ![merge-layers][merge-layers]
4. Right-click on the newly-created layer and select Blending Options ![blending-options][blending-options]
5. Change the Knockout to Deep and the Opacity slider controls the transparency. Personally, I like to set it to anywhere from 30-50. ![knockout-effect][knockout-effect]

The final result should look like this: ![final][final]

[hide-useless-layers]: ../images/hide-useless-layers.png
[merge-layers]: ../images/merge-layers.png
[blending-options]: ../images/blending-options.png
[knockout-effect]: ../images/knockout-effect.png
[final]: ../images/final.png
