+++
date = '2025-07-30T23:51:01-03:00'
draft = true
title = 'Prerendered Backgrounds in Godot'
+++
<Screenshot of demo>

### Terminology 
- Background: The prerendered static part of the screen during gameplay
- 3D object: "Real" object that the player can interact and fully rendered in real time
- Background object: Things in the prerendered background that can be drawn on top of 3D objects to give the illusion of being another 3D object.

## Some context
Back in the early 2000s, one of my friends brought his PS1 with a copy of Resident Evil. That was my first contact with the series. Me and my family were astonished on how good the first cutscene looked (the one where the zombie is eating some guy's face). 

<Screenshot of RE1>

Some years later i found out the trick to make the game look that good in such a weak system was to render a 3D static scene in a computer application, like Softimage 3D or 3D studio, put it in the game, put 3d objects on top of that and make the in game camera match the one used in the 3D render. Ever since I knew that I wanted to recreate it somehow on my end.

The part of getting 3d objects on top of a PNG is easy. My first attempt was rendering a simple room and matching the blender camera values with those of a Raylib camera (sadly, I don't have any screenshots or the code of this progotype). It worked fine, but as soon as you want to add background objects in the prerender that could be on top of the 3d objects, it gets a little bit more complicated. You need to somehow tell the engine "see this specific pixel of the background? It's supposed to be in front of the 3d object right now, so paint it after drawing the 3d object". In this attempt, i had two renders: the "albedo", which is what you end up seeing in game, and the "depth", which is a black and white image that tells how far things are in the image. 

<Both images of albedo and depth>

In theory, you could check the color of the depth render to determine the order the pixels of the albedo should be drawn. This had two problems i was never able to resolve:
- it involves writing shaders. They should be almost perfect for this situation since they take care of pixel logic instantly. I gave it a very long try but I can't wrap my mind around shader programming, so I gave up. 
- the depth map is not that precise. After trying the shader way, i tried to make a simple logic of "if a depth pixel is darker than X, then draw it again on top of everything" just to make a proof of concept of what it would look like. It was not that good. The edges of the image looked really "misty" and it wasn't a clear cut, because of the way blender renders the depth map.

I put the project on hold for a while and tried doing it again, but this time in Godot. I found a video by <author>, and I learned you can import .gltf files into it, and include every relevant thing to the game. What he does in the second method of the video is render the image and project the render on top of every object. While this works, this has the following problems:
- excess of polygons. After all, the point of this is to lower resource usage.
- the projection is messed up along slanted faces. If you're gonna do projections on polygons, your best bet is to make them as perpendicular to the camera angle as possible

## My method 
While this method is not as efficient as they were on old systems, it should run on a toaster, it looks convincing and it's pretty easy to import into your game. It consists of strategically placed planes and projecting parts of the background render to those. So in essence, the background is just another 3D static object that gives the illusion of being a more complex thing.

<Gif showing the effect>

First, I model the complete room to be rendered in Blender. There's not much science for this, besides 3D modeling knowledge.
< Screenshots of the room>

Then I make a low poly version of the room that will serve as collision data. This will help both for character navigation and to place the background objects planes.
< Screenshot of colision data>

Now we get to the interesting part. We know thanks to the collision model where the player (or any object for that matter) WON'T be. We can take advantage of that. Since no 3D object will be present inside that area, we can place a plane inside it and project the render to it. 
<Screenshot of the plane inside the collision area>

This way, if something should appear behind the background object, the plane simply occludes it. Because the plane is just another 3d real time object, no special tricks are needed.
<Gif showing this from two angles>