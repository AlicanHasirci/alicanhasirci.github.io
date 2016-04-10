---
layout: post
title: "Camera Transformation"
date: 2016-4-10 21:05:28
image: '/assets/img/'
description:
tags:
- matrices
- matrix
- computer
- graphics
- camera
- transformation
categories:
- computergraphics
twitter_text:
---

Lets start by talking about pinhole cameras. A pinhole camera is basicly a box with a film at one end and a hole in the other with a shutter or an obstacle. Removing the obstacle behind the hole exposes the film in the box giving you an upside down and backwards image of the of what are you pointing at on the film. The reason is obvious where the lights comes in from hole obliquely where something high ends up being low on the film.

{% include image.html url="https://upload.wikimedia.org/wikipedia/commons/thumb/3/3b/Pinhole-camera.svg/2000px-Pinhole-camera.svg.png" description="Pinhole Camera" %}

This is basicly what is being used in computer graphics, a pinhole model. To specify a camera though we need few things first. 

First thing first, we need to position for our camera. Looking at the image of pinhole camera we can right away say that our camera position is going to be the at point where pinhole stands called the camera point, we'll call it $$P$$ for future reference. 

Next, now that we have a point where our camera stands, we must know what we are point the camera to, so we need a vector that from our camera point that we'll call $$D$$. 

Another thing we will now be needing is an 'up' direction, right? Intuitively, when you are filming something with a camera you know that there is an up direction ergo you don't take pictures which looks like a whirlpool. We'll call our up direction $$U$$. 

Now that we've specified our up vector and direction vector we can take a cross product of these two($$D \times V$$) which will give us a third vector perpendicural to these two, this way we can establish an orthogonal frame for our camera.

Lets see, since we have a camera frame, next thing we'll be needing is an angle right? After all we must define our viewing angle which will be $$\alpha$$.

Ok then, we have a camera standing on a point $$P$$ that can see $$\alpha$$ angle, though this may seem enough we'll be needing two more things to make everything work. Near and far planes. These two are invented around 1982 for a flight simulation program as far as a remember where people working on it saw that a city really far far away seems like only a small pixel which was redundant even to render, so they came up with these two planes to clip away things that are really close and really far depending on your choice. Also calculations on graphics hardware are dependant to near and far, which we'll see soon.

Now we have a pyramid of things that we see in our hands called viewing pyramid(we'll since we clip away near and far it is actually a frustum), first thing is first, we have to translate this camera to the origin and align it's frame to our axis(and everyting with it). Now this part is important, everything we see on screen is a projection of 3D space on a 2D image which is our screen right? So we have to find a way to smash everything together into a plane so that we can display this image on screen. To do this we use an image space which is actually a cube with edges on $$-1,1$$ and our ultimate goal is to squeeze everything in our frustum to this cube, then smash then on to $$X,Y$$ plane on image space to project on our screen.

By working the math, you get a transformation matrix like this:
<div>
$$\begin{bmatrix}
\cot\frac{\alpha}{2} & 0 & 0 & 0 \\
0 & \cot\frac{\alpha}{2} & 0 & 0 \\
0 & 0 & \frac{f+n}{f-n} & \frac{2fn}{f-n} \\
0 & 0 & 1 & 0
\end{bmatrix}$$
</div>
You can test this by having some points on the viewing frustum and see where it ends on the image space. To summarize, while the $$\cot\frac{\alpha}{2}$$ parts the care of the 'squishing' behaviour to fit a frustum into a cube, the $$\frac{f+n}{f-n}$$ part takes the $$z$$ into with $$w$$ to scale the object according to it's distance to camera.

This may seem a bit overwhelming, surely you'll never write your own matrices, definately not a projection matrix since your GPU handles everyting for you, it is quite useful to know what goes under the hood especially when you are diving into shaders.

On the next post i'll be taking on the Clipping, where the GPU decides what to render and what to rule it, which is a vital part on rendering efficiency and may answer a lot of question if you have any on clipping or visibility problems.



