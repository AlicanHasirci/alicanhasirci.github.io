---
layout: post
title: "Depth Buffers and Raytracing"
date: 2016-05-01 17:05:28
image: '/assets/img/'
description:
tags:
- matrices
- matrix
- computer
- graphics
- depthbuffer
- raytracing
categories:
- computergraphics
- depthbuffer
- raytracing
twitter_text:
---
The Problem
-----------
Drawing something on the screen is all nice and dandy for now. Keeping it simple is always the way to go but when you are drawing many polygons on the screen you are facing some new problems and one of these that people back in the day had to figure out was overlapping. Lets elaborate on that, think about two polygons in our camera's view. As we have talked about it earlier their journey starts in our viewing frustum first, after getting transform with camera to the origin, then they will be transformed into where they will end up in a cube with all edges on $$-1$$ and $$1$$ then comes the clipping and by far things are pretty much straight forward, if we have few polygons we can just check their $$w$$ value that we got during viewing transform and draw the polygon that suits us best, well although it sounds like a good plan there is problem with that. What happens when a polygon partially overlaps with another one? This was the million dollar question for a long time and then the depth buffer(which is also referred to as z-buffer) solution came into rescue along with other solutions such as raytracing.

Depth Buffer
------------
A depth buffer is basically a buffer kept on GPU that stores a value(depending on GPU, may be 8-bit all the way to 32-bit, higher the better, objects that are really close to eachother may employ the same value on depth buffer which will cause a z-fighting where the polygons look interlaced and jittery) for each pixel. The depth is proportional to the distance between the screen plane and the fragment of polygon that has been drawn. Which means further it is, higher the depth value is. That being said, during every draw, while the fragments color is stored on frame buffer, if the depth value is to be stored on depth buffer, every other fragment from another overlapping polygon can check their depth value with the one that has been stored and if it's a smaller value override it, this is called a 'depth testing'. By doing this you may also discard the overlapped fragments(which is called 'depth culling') which will then save you from drawing unnecessary fragments. This behaviour resembles to 'stencil buffer' a lot and infact stencil buffer is an optional example to depth buffers.

Raytracing
----------
Another approach to this overlapping issue is raytracing. Remember the what depth buffer does, it check every pixels depth value in a buffer, raytracing does something similar to that. What it does is that for every pixel in the screen, it sends a ray to that direction and see what it hits. When you think about it, it is kind of ingenious and like how our viewing works like with a small difference. Take the camera as the eye, when we look at things, the way it works is that the light that bounces of the object is captured by our eye and then proccessed in our brain right? Raytracing works in a similar way but rather then getting the rays from outside, we send them to all our viewing frustum and see what it hits. The reason we do it this way is hardware restrictions. In nature light bounces off every surface acting as a light source itself which is proven to be impossible to do in realtime at a complex scene with multiple lights sources and multiple objects. 

Anyways lets get on to our subject, raytracing was thought to be ludacris and impossible to do back in 80s and 90s. Imagine you have a million pixels on screen which not that much at all, and send rays to all these pixels and testing them for each object on the screen. Although notoriously it was time consuming, the images that came out was beautiful and most importantly it sovled three of the most outstanding problems that was considered so hard that noone was even trying to solve. These were shadows, reflection and translucency. For every ray that is sent from camera and hit and object, by sending another ray to the light source and testing if something is inbetween, it was possible define a shadow and for translucency all you had to do was to create an other ray for refraction and the logic behind the reflaction was not that different. The logic behind it was nearly intuitional and the output was perfect, though raytracing is still is proven to be a hard labor for realtime usage it is getting more and more feasable and applications of it can be seen in various platforms already.