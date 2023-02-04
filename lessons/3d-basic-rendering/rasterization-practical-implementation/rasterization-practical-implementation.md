## Improving the Rasterization Algorithm

![Figure 1: jagged edges pixel artifacts can be reduced using anti-aliasing.](/images/rasterization/jaggies.png)

![Figure 2: when one sample is used, the triangle is missed, but by using sub-pixels, we can detect that the pixel overlaps the triangle at least partially. The pixel color is equal to the sum of the sub-pixel colors divided by the total number of sub-pixels, or samples (in this example, 16 samples are used).](/images/rasterization/antialiasing3.png?)
![Figure 2: when one sample is used, the triangle is missed, but by using sub-pixels, we can detect that the pixel overlaps the triangle at least partially. The pixel color is equal to the sum of the sub-pixel colors divided by the total number of sub-pixels, or samples (in this example, 16 samples are used).](/images/rasterization/antialiasing3.png?)

![Figure 3: pixels can't properly capture the shape of continuous surfaces.](/images/rasterization/antialiasing1.png?)

PROOFREADER NOTE: Not sure what you mean by "fight aliasing". I searched up this term on google, and it doesn't seem there is a type of aliasing called "fight". I'm assuming you meant to write "using sub-pixels to improve/fight aliasing".
PROOFREADER NOTE: Not sure what you mean by "fight aliasing". I searched up this term on google, and it doesn't seem there is a type of aliasing called "fight". I'm assuming you meant to write "using sub-pixels to improve/fight aliasing".
![Figure 4: using sub-pixels to improve fight aliasing.](/images/rasterization/antialiasing2.png?)

![Figure 5: anti-aliasing helps with smoothing jagged edges.](/images/rasterization/antialiasing4.png?)

![Figure 6: 1 sample per pixel, or 1 pps (top), vs. 4 samples per pixel (bottom).](/images/rasterization/antialiasing.png?)
![Figure 6: 1 sample per pixel, or 1 pps (top), vs. 4 samples per pixel (bottom).](/images/rasterization/antialiasing.png?)

![Figure 7: if the 4 corners of an 8x8 pixel grid overlap the triangle, then all remaining pixels of the grids cover the triangle too.](/images/rasterization/rasterization-optimization.png?)
![Figure 7: if the 4 corners of an 8x8 pixel grid overlap the triangle, then all remaining pixels of the grids cover the triangle too.](/images/rasterization/rasterization-optimization.png?)

### Aliasing and Anti-Aliasing

All the techniques we presented in the previous chapters are the foundation of the rasterization algorithm, though we have only implemented these techniques in a very basic way. The GPU rendering pipeline and other rasterization-based production renderers use the same concepts, but they used highly optimized versions of these algorithms. Presenting all the different tricks that are used to speed up the algorithm goes way beyond the scope of an introduction. We will just review some of them quickly here, but we plan to devote a lesson to this topic in the future.
All the techniques we presented in the previous chapters are the foundation of the rasterization algorithm, though we have only implemented these techniques in a very basic way. The GPU rendering pipeline and other rasterization-based production renderers use the same concepts, but they used highly optimized versions of these algorithms. Presenting all the different tricks that are used to speed up the algorithm goes way beyond the scope of an introduction. We will just review some of them quickly here, but we plan to devote a lesson to this topic in the future.

First, let's consider one basic problem with 3D rendering. If you zoom up on the image of the triangle that we rendered in the previous chapter, you will notice that the edges of the triangle is irregular (in fact, this is not specific to the edge of the triangle - you can also see that the checkerboard pattern is irregular on the edge of the squares). The staircase steps that you can easily see in Figure 1 are called **jaggies**. These jagged edges, or stair-stepped edges (depending on what you prefer to call them), are not an artifact. This is just the result of the fact that the triangle is broken down into pixels. What we do with the rasterization process is break down a continuous surface (the triangle) into discrete elements (the pixels), a process that we already mentioned in the [Introduction to rendering](lessons/3d-basic-rendering/rendering-3d-scene-overview). The problem is similar to trying to represent a continuous curve or surface with Lego bricks - you simply can't as you will always see the bricks (Figure 2). The solution to this problem in rendering is called **anti-aliasing** (also denoted AA). Rather than rendering only 1 sample per pixel (checking if the pixel overlaps the triangle by testing if the point in the center of the pixel covers the triangle), we split the pixel into sub-pixels and repeat the coverage test for each sub-pixel. Of course, each sub-pixel is nothing more than another brick, thus this the problem is not solved entirely. Nonetheless, it allows for capturing the edges of objects with slightly more precision. Pixels are, most of the time, divided into an N by N number of sub-pixels, where N is generally a power of 2 (2, 4, 8, etc.), though it can technically take on any value greater or equal to 1 (1, 2, 3, 4, 5, etc.). There are different ways of addressing this aliasing issue. The method we described belongs to the category of sampling-based anti-aliasing methods.
First, let's consider one basic problem with 3D rendering. If you zoom up on the image of the triangle that we rendered in the previous chapter, you will notice that the edges of the triangle is irregular (in fact, this is not specific to the edge of the triangle - you can also see that the checkerboard pattern is irregular on the edge of the squares). The staircase steps that you can easily see in Figure 1 are called **jaggies**. These jagged edges, or stair-stepped edges (depending on what you prefer to call them), are not an artifact. This is just the result of the fact that the triangle is broken down into pixels. What we do with the rasterization process is break down a continuous surface (the triangle) into discrete elements (the pixels), a process that we already mentioned in the [Introduction to rendering](lessons/3d-basic-rendering/rendering-3d-scene-overview). The problem is similar to trying to represent a continuous curve or surface with Lego bricks - you simply can't as you will always see the bricks (Figure 2). The solution to this problem in rendering is called **anti-aliasing** (also denoted AA). Rather than rendering only 1 sample per pixel (checking if the pixel overlaps the triangle by testing if the point in the center of the pixel covers the triangle), we split the pixel into sub-pixels and repeat the coverage test for each sub-pixel. Of course, each sub-pixel is nothing more than another brick, thus this the problem is not solved entirely. Nonetheless, it allows for capturing the edges of objects with slightly more precision. Pixels are, most of the time, divided into an N by N number of sub-pixels, where N is generally a power of 2 (2, 4, 8, etc.), though it can technically take on any value greater or equal to 1 (1, 2, 3, 4, 5, etc.). There are different ways of addressing this aliasing issue. The method we described belongs to the category of sampling-based anti-aliasing methods.

The final pixel color is computed as the sum of all the sub-pixel's colors divided by the total number of sub-pixels. As with pixels, if a sub-pixel, or sample, covers a triangle, it then takes on the color of that triangle, otherwise it takes on the background color. Let's take an example. Imagine that the triangle is white, and the background is black. If only 2 or 4 samples overlap the triangle, the final pixel color will be equal to (0+0+1+1)/4=0.5. The pixel won't be completely white, but it won't be completely black either. Thus, rather than having a "binary" transition between the edge of the triangle and the background, that transition is more gradual, which reduces the stair-stepped pixels artifact. This is what we call **anti-aliasing**. To fully understand anti-aliasing you need to study signal processing theory, which again is a very large and pretty complex topic on its own.
The final pixel color is computed as the sum of all the sub-pixel's colors divided by the total number of sub-pixels. As with pixels, if a sub-pixel, or sample, covers a triangle, it then takes on the color of that triangle, otherwise it takes on the background color. Let's take an example. Imagine that the triangle is white, and the background is black. If only 2 or 4 samples overlap the triangle, the final pixel color will be equal to (0+0+1+1)/4=0.5. The pixel won't be completely white, but it won't be completely black either. Thus, rather than having a "binary" transition between the edge of the triangle and the background, that transition is more gradual, which reduces the stair-stepped pixels artifact. This is what we call **anti-aliasing**. To fully understand anti-aliasing you need to study signal processing theory, which again is a very large and pretty complex topic on its own.

The reason why it is best to choose N as a power of 2 is because most processors these days can run several instructions in parallel and the number of instructions that run in parallel is also a power of 2. You can look on the web for things such as SSE instruction sets, which are specific to CPUs, but GPUs use the same concept. SSE is a feature that is available on most modern CPUs which can be used to run 4 or 8 floating point calculations at the same (in one cycle). All this means is that, for the price of 1 floating point operation, you get 3 or 7 for free. This can, in theory, speed up your rendering by 4 or 8 times (you can never reach this level of performance, though, because you need to pay a small penalty for setting these instructions up). You can use SSE instructions, for example, to render 2x2 sup-pixels for the cost of computing 1 pixel, and as a result, get smoother edges for "free" (the stair-stepped edges are less visible).
The reason why it is best to choose N as a power of 2 is because most processors these days can run several instructions in parallel and the number of instructions that run in parallel is also a power of 2. You can look on the web for things such as SSE instruction sets, which are specific to CPUs, but GPUs use the same concept. SSE is a feature that is available on most modern CPUs which can be used to run 4 or 8 floating point calculations at the same (in one cycle). All this means is that, for the price of 1 floating point operation, you get 3 or 7 for free. This can, in theory, speed up your rendering by 4 or 8 times (you can never reach this level of performance, though, because you need to pay a small penalty for setting these instructions up). You can use SSE instructions, for example, to render 2x2 sup-pixels for the cost of computing 1 pixel, and as a result, get smoother edges for "free" (the stair-stepped edges are less visible).

### Rendering Blocks of Pixels

Another common technique to accelerate rasterization is to render blocks of pixels. Rather than testing all pixels contained in the block, we first start to test pixels at the corners of the block. GPU algorithms can use blocks of 8x8 pixels. The technique used is more elaborate and based on the concept of tiles, but we won't detail it here. If all four corners of that 8x8 grid cover the triangle, then the other pixels of the grid also cover the rectangle (as shown in Figure 7). In that case, there is no need to test all the other pixels which saves time - they can just be filled up with the triangle's colors. If vertex attributes need to be interpolated across the pixels block, this is also straightforward because, if you have computed them at the block's corners, then all you need to do is linearly interpolate them in both directions (horizontally and vertically). This optimization only works when triangles are close to the screen and thus large in screen space - small triangles don't benefit from this technique.
Another common technique to accelerate rasterization is to render blocks of pixels. Rather than testing all pixels contained in the block, we first start to test pixels at the corners of the block. GPU algorithms can use blocks of 8x8 pixels. The technique used is more elaborate and based on the concept of tiles, but we won't detail it here. If all four corners of that 8x8 grid cover the triangle, then the other pixels of the grid also cover the rectangle (as shown in Figure 7). In that case, there is no need to test all the other pixels which saves time - they can just be filled up with the triangle's colors. If vertex attributes need to be interpolated across the pixels block, this is also straightforward because, if you have computed them at the block's corners, then all you need to do is linearly interpolate them in both directions (horizontally and vertically). This optimization only works when triangles are close to the screen and thus large in screen space - small triangles don't benefit from this technique.

### Optimizing the Edge Function

The edge function can be optimized too. Let's have a look at the function implementation again:
The edge function can be optimized too. Let's have a look at the function implementation again:

```
int orient2d(const Point2D& a, const Point2D& b, const Point2D& c)
{
    return (b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x - a.x);
}
```

Recall that `a` and `b` in this function are the triangle vertices and `c` is the pixel coordinate (in raster space). One interesting thing to note is that this function is going to be called for each pixel contained in the triangle bounding box, though while we iterate over multiple pixels only `c` changes, and the variables `a` and `b` stay the same. Suppose we evaluate the equation once and get a result `w0`:
Recall that `a` and `b` in this function are the triangle vertices and `c` is the pixel coordinate (in raster space). One interesting thing to note is that this function is going to be called for each pixel contained in the triangle bounding box, though while we iterate over multiple pixels only `c` changes, and the variables `a` and `b` stay the same. Suppose we evaluate the equation once and get a result `w0`:

```
w0 = (b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x - a.x);
```

Then `c.x` is incremented by an amount `s` (the per pixel step). The new value of `w0` is going to be:
Then `c.x` is incremented by an amount `s` (the per pixel step). The new value of `w0` is going to be:

```
w0_new = (b.x - a.x) * (c.y - a.y) - (b.y - a.y) * (c.x + s - a.x);
```

Subtracting the first equation from the second, we get:

```
w0_new - w0 = -(b.y - a.y) * s;
```

The term `-(b.y - a.y) * s` is a constant value for a given triangle, because `s` is the same amount each time (one pixel), and `a` and `b`, as already mentioned, are constant too. We can calculate it once and store it in a variable (call it `w0_step`) and then the calculation reduces to:
The term `-(b.y - a.y) * s` is a constant value for a given triangle, because `s` is the same amount each time (one pixel), and `a` and `b`, as already mentioned, are constant too. We can calculate it once and store it in a variable (call it `w0_step`) and then the calculation reduces to:

```
w0_new = w0 + w0step;
```

You can do this for `w1` and `w2`, and also follow a similar process for the `c.y` step.
You can do this for `w1` and `w2`, and also follow a similar process for the `c.y` step.

The edge function uses 2 mults and 5 subs, but with this trick it can be reduced to a simple addition (of course you need to compute the initial values first). This technique is well documented on the internet. We won't be using it in this lesson, but we will study it in more detail and implement it in another lesson devoted to advanced rasterization techniques.
The edge function uses 2 mults and 5 subs, but with this trick it can be reduced to a simple addition (of course you need to compute the initial values first). This technique is well documented on the internet. We won't be using it in this lesson, but we will study it in more detail and implement it in another lesson devoted to advanced rasterization techniques.

### Fixed Point Coordinates

![Figure 8: fixed point coordinates.](/images/rasterization/subpixel-precision.png?)

Finally, to conclude this section, we will briefly talk about the technique that consists of converting vertex coordinates, which are initially defined in floating-point format, to fixed-point format just before the rasterization stage. Fixed-point is the fancy word (in fact the correct technical term) for integer. When vertex coordinates are converted from NDC to raster space, they are also converted from floating point numbers to fixed point numbers. Why do we do that? There is no easy and quick answer to this question. To be brief, let's just say that GPUs use fixed point arithmetic because, from a computing point of view, manipulating integers is easier and faster than manipulating floats or doubles (it only requires logical bit operations). Again, this is just a very brief explanation. The conversion from floating point to integer coordinates and how the rasterization process is implemented using integer coordinates is a large and complex topic that is not documented on the internet (you will find very little information about this topic, which is very strange considering that this very process is central to the way modern GPUs work).
Finally, to conclude this section, we will briefly talk about the technique that consists of converting vertex coordinates, which are initially defined in floating-point format, to fixed-point format just before the rasterization stage. Fixed-point is the fancy word (in fact the correct technical term) for integer. When vertex coordinates are converted from NDC to raster space, they are also converted from floating point numbers to fixed point numbers. Why do we do that? There is no easy and quick answer to this question. To be brief, let's just say that GPUs use fixed point arithmetic because, from a computing point of view, manipulating integers is easier and faster than manipulating floats or doubles (it only requires logical bit operations). Again, this is just a very brief explanation. The conversion from floating point to integer coordinates and how the rasterization process is implemented using integer coordinates is a large and complex topic that is not documented on the internet (you will find very little information about this topic, which is very strange considering that this very process is central to the way modern GPUs work).

The conversion step involves rounding off the vertex coordinates to the nearest integer. After doing this conversion, though, you sort of snap the vertex coordinates to the nearest pixel corner coordinates. This is not so much an issue when you render a still image, but it creates visual artifacts with animation (vertices are snapped to different pixels from frame to frame). The workaround is to convert the number to the smallest integer value, but also reserve some bits to encode the **sub-pixel** position of the vertex (the fractional part of the vertex position). GPUs typically use 4 bits to encode sub-pixel precision (you can search through graphics API documentation for the term sub-pixel precision). In other words, on a 32 bits integer, 1-bit is used to encode the number's sign, 27 bits are used to encode the vertex integer position, and 4 bits are used to encode the fractional position of the vertex within the pixel. This means that the vertex position is "snapped" to the closest corner of a 16x16 sub-pixel grid, as shown in Figure 8 (with 4 bits, you can represent any integer number in the range [1:15]). Somehow, the vertex position is still snapped to some grid corner, but snapping in this case is less of a problem than when the vertex is snapped to the pixel coordinates. This conversion process leads to many other issues, one of which is **integer overflow** (overflow occurs when the result of an arithmetic operation produces a number that is greater than the number that can be encoded with the current available bits). This overflow may happen because integers cover a smaller range of values than floats. Things get sort of complex when anti-aliasing is thrown into the mix - it would take a lesson on its own to explore the topic in detail.
The conversion step involves rounding off the vertex coordinates to the nearest integer. After doing this conversion, though, you sort of snap the vertex coordinates to the nearest pixel corner coordinates. This is not so much an issue when you render a still image, but it creates visual artifacts with animation (vertices are snapped to different pixels from frame to frame). The workaround is to convert the number to the smallest integer value, but also reserve some bits to encode the **sub-pixel** position of the vertex (the fractional part of the vertex position). GPUs typically use 4 bits to encode sub-pixel precision (you can search through graphics API documentation for the term sub-pixel precision). In other words, on a 32 bits integer, 1-bit is used to encode the number's sign, 27 bits are used to encode the vertex integer position, and 4 bits are used to encode the fractional position of the vertex within the pixel. This means that the vertex position is "snapped" to the closest corner of a 16x16 sub-pixel grid, as shown in Figure 8 (with 4 bits, you can represent any integer number in the range [1:15]). Somehow, the vertex position is still snapped to some grid corner, but snapping in this case is less of a problem than when the vertex is snapped to the pixel coordinates. This conversion process leads to many other issues, one of which is **integer overflow** (overflow occurs when the result of an arithmetic operation produces a number that is greater than the number that can be encoded with the current available bits). This overflow may happen because integers cover a smaller range of values than floats. Things get sort of complex when anti-aliasing is thrown into the mix - it would take a lesson on its own to explore the topic in detail.

Fixed point coordinates allow us to speed up the rasterization process and the edge function even more. This is one of the reasons for converting vertex coordinates to fixed-point values. These optimization techniques will be presented in another lesson.
Fixed point coordinates allow us to speed up the rasterization process and the edge function even more. This is one of the reasons for converting vertex coordinates to fixed-point values. These optimization techniques will be presented in another lesson.

## Notes Regarding our Implementation of the Rasterization Algorithm

Finally, we are going to quickly review the code provided in the source code chapter. Here is a description of its main components:

<details>
Use this code for learning purposes only, it is not efficient at all. Many simple optimizations could be made, but we wrote it for clarity, not efficiency. Our goal is not to write production code, but code that helps people to learn how things work in principle. Optimizations are left as an option for you, which could make a good exercise.
Use this code for learning purposes only, it is not efficient at all. Many simple optimizations could be made, but we wrote it for clarity, not efficiency. Our goal is not to write production code, but code that helps people to learn how things work in principle. Optimizations are left as an option for you, which could make a good exercise.
</details>

<details>
The object that is being rendered by the program is stored in an include file. This is fine for this small program, but this would never be done in a professional application. Geometry files can be very large and including this data in the program means that the size of the program will become very large as well (which is not a good thing, as compile time is going to increase a lot). Though, again for this simple test, and because the object we are using is quite light, this is not an issue. Though, more generally, if you are unsure about the way we represent the geometry of a 3D object in a program, check the lesson on this topic in the [Modeling and Geometry section](lessons/modeling-geometry). The only information we will be using in this program are the triangle vertices' positions (in world space) and their texture or st coordinates.
The object that is being rendered by the program is stored in an include file. This is fine for this small program, but this would never be done in a professional application. Geometry files can be very large and including this data in the program means that the size of the program will become very large as well (which is not a good thing, as compile time is going to increase a lot). Though, again for this simple test, and because the object we are using is quite light, this is not an issue. Though, more generally, if you are unsure about the way we represent the geometry of a 3D object in a program, check the lesson on this topic in the [Modeling and Geometry section](lessons/modeling-geometry). The only information we will be using in this program are the triangle vertices' positions (in world space) and their texture or st coordinates.
</details>

- We will use the function `computeScreenCoordinates` to compute the screen coordinates using the method detailed in the lesson devoted to the [pinhole camera model](lessons/3d-basic-rendering/3d-viewing-pinhole-camera). This is needed to be sure that our render output can be compared to a render in Maya which is also using a physically-based camera model.
- We will use the function `computeScreenCoordinates` to compute the screen coordinates using the method detailed in the lesson devoted to the [pinhole camera model](lessons/3d-basic-rendering/3d-viewing-pinhole-camera). This is needed to be sure that our render output can be compared to a render in Maya which is also using a physically-based camera model.

  ```
  float t, b, l, r; 
  
  computeScreenCoordinates( 
      filmApertureWidth, filmApertureHeight, 
      imageWidth, imageHeight, 
      kOverscan, 
      nearClippingPLane, 
      focalLength, 
      t, b, l, r); 
  ```

- The function `convertToRaster()` is where we convert the triangle vertices' coordinates from camera space to raster space. The function is similar to the function `computePixelCoordinates()` that we implemented in the previous lesson. Remember that, in the second chapter of this lesson, we learned about a method to convert coordinates from screen space to NDC space (keep in mind that on GPUs, coordinates in NDC space are in the range [-1,1]). We will use the same remapping method in this function. The other important thing to remember is that, up to the previous lesson, projected points were 2D points. From now on, they will need to be 3D points. In the x- and y-coordinates of these points, we will store the coordinates of the projected point in screen space. In the z-coordinate of the point, we will store the vertex's camera space z-coordinate. The z-coordinate is needed to solve the visibility problem, as explained in [chapter four](lessons/3d-basic-rendering/rasterization-practical-implementation/visibility-problem-depth-buffer-depth-interpolation) of this lesson.
- The function `convertToRaster()` is where we convert the triangle vertices' coordinates from camera space to raster space. The function is similar to the function `computePixelCoordinates()` that we implemented in the previous lesson. Remember that, in the second chapter of this lesson, we learned about a method to convert coordinates from screen space to NDC space (keep in mind that on GPUs, coordinates in NDC space are in the range [-1,1]). We will use the same remapping method in this function. The other important thing to remember is that, up to the previous lesson, projected points were 2D points. From now on, they will need to be 3D points. In the x- and y-coordinates of these points, we will store the coordinates of the projected point in screen space. In the z-coordinate of the point, we will store the vertex's camera space z-coordinate. The z-coordinate is needed to solve the visibility problem, as explained in [chapter four](lessons/3d-basic-rendering/rasterization-practical-implementation/visibility-problem-depth-buffer-depth-interpolation) of this lesson.

  ```
  convertToRaster(v0, worldToCamera, l, r, t, b, nearClippingPLane, imageWidth, imageHeight, v0Raster); 
  convertToRaster(v1, worldToCamera, l, r, t, b, nearClippingPLane, imageWidth, imageHeight, v1Raster); 
  convertToRaster(v2, worldToCamera, l, r, t, b, nearClippingPLane, imageWidth, imageHeight, v2Raster);
  ```

- Don't forget that all vertex attributes associated with triangle vertices need to be "pre-divided" by the vertices' z-coordinates (this is needed for perspective-correct interpolation). This is usually done just before the triangle gets rendered (just before the loop that iterates over the pixels), though be careful because, in the code, the vertex z-coordinate is set to its reciprocal (to speed up the computation of the sample depth). Thus, rather than using a division, we will use multiplication instead.
- Don't forget that all vertex attributes associated with triangle vertices need to be "pre-divided" by the vertices' z-coordinates (this is needed for perspective-correct interpolation). This is usually done just before the triangle gets rendered (just before the loop that iterates over the pixels), though be careful because, in the code, the vertex z-coordinate is set to its reciprocal (to speed up the computation of the sample depth). Thus, rather than using a division, we will use multiplication instead.

  ```
  // precompute reciprocal of the vertex z-coordinate
  v0Raster.z = 1 / v0Raster.z, 
  v1Raster.z = 1 / v1Raster.z, 
  v2Raster.z = 1 / v2Raster.z; 
  
  Vec2f st0 = st[stindices[i * 3]]; 
  Vec2f st1 = st[stindices[i * 3 + 1]]; 
  Vec2f st2 = st[stindices[i * 3 + 2]]; 
  
  // This is needed for perspective correct interpolation
  st0 *= v0Raster.z, st1 *= v1Raster.z, st2 *= v2Raster.z;
  ```

- The function contains two loops: the first to iterate over all the triangles in the scene (the outer loop), and the second to iterate over all pixels contained in the bounding box overlapping the triangle that is being rendered (the inner loop). Note that some variables in the inner loop are constant and can thus be pre-computed. This is the case for the inverse of the triangle vertices' z-coordinates, which are linearly interpolated for each pixel that covers the triangle (as well as the vertex attributes that need to be divided by their respective z-coordinates).
- The function contains two loops: the first to iterate over all the triangles in the scene (the outer loop), and the second to iterate over all pixels contained in the bounding box overlapping the triangle that is being rendered (the inner loop). Note that some variables in the inner loop are constant and can thus be pre-computed. This is the case for the inverse of the triangle vertices' z-coordinates, which are linearly interpolated for each pixel that covers the triangle (as well as the vertex attributes that need to be divided by their respective z-coordinates).

  ```
  // Outer loop. Loop over triangles
  for (uint32_t i = 0; i < ntris; ++i) {
      ...
      // Inner loop. Loop over pixels
      for (uint32_t y = y0; y <= y1; ++y) { 
          for (uint32_t x = x0; x <= x1; ++x) {
              ...
          }
      }
  }
  ```

- The rest of the program is straightforward. Each pixel is tested for coverage using the edge function technique. If a pixel covers a triangle, we compute the pixel barycentric coordinates. We then use these coordinates to compute the depth of the sample. If the pixel passes the depth buffer test, we then update the buffer with the new depth value and update the frame-buffer with the triangle color.
- The rest of the program is straightforward. Each pixel is tested for coverage using the edge function technique. If a pixel covers a triangle, we compute the pixel barycentric coordinates. We then use these coordinates to compute the depth of the sample. If the pixel passes the depth buffer test, we then update the buffer with the new depth value and update the frame-buffer with the triangle color.

  ```
  Vec3f pixelSample(x + 0.5, y + 0.5, 0); 
  float w0 = edgeFunction(v1Raster, v2Raster, pixelSample); 
  float w1 = edgeFunction(v2Raster, v0Raster, pixelSample); 
  float w2 = edgeFunction(v0Raster, v1Raster, pixelSample); 
  if (w0 >= 0 && w1 >= 0 && w2 >= 0) { 
      w0 /= area; 
      w1 /= area; 
      w2 /= area; 
      // linearly interpolate sample depth
      float oneOverZ = v0Raster.z * w0 + v1Raster.z * w1 + v2Raster.z * w2; 
      float z = 1 / oneOverZ; 
      // do we pass the depth buffer test?
      if (z < depthBuffer[y * imageWidth + x]) { 
          depthBuffer[y * imageWidth + x] = z;
          // update frame buffer
          ...
      }
  }
  ```

- **Shading wise:** to make our image more visually interesting, we will be using several shading techniques. The model we are using has only one vertex attribute: st or texture coordinates. The texture coordinates can be used to create a checkerboard pattern, which is then combined with a simple shading trick called facing ratio. The facing ratio takes the dot product between the triangle normal (which we can compute with a simple cross product between any two edges of the triangle) and the view direction. The view direction is simply the vector defined by the point P on the triangle that is being shaded and the camera position E. Since all points at this stage of the program are defined in camera space, the camera position (or the eye position) is simply E=(0,0,0). Thus, the view direction can be computed as -P, which then needs to be normalized. The dot product can be negative, so we need to clamp it (we only want positive values).
- **Shading wise:** to make our image more visually interesting, we will be using several shading techniques. The model we are using has only one vertex attribute: st or texture coordinates. The texture coordinates can be used to create a checkerboard pattern, which is then combined with a simple shading trick called facing ratio. The facing ratio takes the dot product between the triangle normal (which we can compute with a simple cross product between any two edges of the triangle) and the view direction. The view direction is simply the vector defined by the point P on the triangle that is being shaded and the camera position E. Since all points at this stage of the program are defined in camera space, the camera position (or the eye position) is simply E=(0,0,0). Thus, the view direction can be computed as -P, which then needs to be normalized. The dot product can be negative, so we need to clamp it (we only want positive values).

  ```
  Vec3f n = (v1Cam - v0Cam).crossProduct(v2Cam - v0Cam); 
  n.normalize(); 
  Vec3f viewDirection = -pt; 
  viewDirection.normalize(); 
  // facing ratio 
  float nDotView =  std::max(0.f, n.dotProduct(viewDirection));
  ```
  

  Note that, for this technique to work, we also need to find the coordinates of P. The position of the point on the triangle that the pixel overlaps can be computed like any vertex attribute. We need to take the vertices in camera space, divide them by their respective z-coordinates, interpolate them with the barycentric coordinates, and multiply the result by the sample depth (which is also, coincidentally, P's (TODO: Check that this "P's" word makes sense) z-coordinate):

  ```
  // Get triangle vertices in camera space.
  Vec3f v0Cam, v1Cam, v2Cam; 
  worldToCamera.multVecMatrix(v0, v0Cam); 
  worldToCamera.multVecMatrix(v1, v1Cam); 
  worldToCamera.multVecMatrix(v2, v2Cam); 
  
  // Divide them by the respective z-coordinate as with any other vertex attribute and interpolate using 
  // barycentric coordinates.
  float px = (v0Cam.x/-v0Cam.z) * w0 + (v1Cam.x/-v1Cam.z) * w1 + (v2Cam.x/-v2Cam.z) * w2; 
  float py = (v0Cam.y/-v0Cam.z) * w0 + (v1Cam.y/-v1Cam.z) * w1 + (v2Cam.y/-v2Cam.z) * w2; 
   
  // P in camera space
  Vec3f pt(px * z, py * z, -z);
  ```
  
  Don't worry if you don't understand these shading techniques very well, they will be studied as soon as we get to the lessons on shading.
  Don't worry if you don't understand these shading techniques very well, they will be studied as soon as we get to the lessons on shading.

- Finally, the content of the frame buffer is stored in a PPM file.

On the left, you can see a render of the object in Maya. On the right is our render. As you can see, the results are identical in terms of the objects' position and shading. Maya uses a technique called stochastic sampling that has an effect to make the aliasing artifact slightly less visible than in our render, but the difference is subtle.
On the left, you can see a render of the object in Maya. On the right is our render. As you can see, the results are identical in terms of the objects' position and shading. Maya uses a technique called stochastic sampling that has an effect to make the aliasing artifact slightly less visible than in our render, but the difference is subtle.

![](/images/rasterization/cowresult.png?)

As you can see, there is nothing magic about rendering. When you know the rules, you can reproduce images produced by professional applications.

![](/images/rasterization/cowresult1.png?)

![](/images/rasterization/cowresult2.png?)

As a bonus, we exported the position, in world space, of the point on the triangle that each pixel in the image overlaps. We then displayed all these points in a 3D viewer, and you can see the results below. Not surprisingly, points only show up on parts of the object that are directly visible to the camera. This shows that the depth-buffer technique works as expected. Every triangle on the back of the object that is hidden by another triangle is not rendered. The second image shows a close-up of the same point set (right).
As a bonus, we exported the position, in world space, of the point on the triangle that each pixel in the image overlaps. We then displayed all these points in a 3D viewer, and you can see the results below. Not surprisingly, points only show up on parts of the object that are directly visible to the camera. This shows that the depth-buffer technique works as expected. Every triangle on the back of the object that is hidden by another triangle is not rendered. The second image shows a close-up of the same point set (right).

## Conclusion

The main advantage of the rasterization algorithm is simplicity and speed. The main drawback is that it is only useful to solve the visibility problem. Remember that rendering involves two steps: visibility and shading. This algorithm is of no use when it comes to shading.
The main advantage of the rasterization algorithm is simplicity and speed. The main drawback is that it is only useful to solve the visibility problem. Remember that rendering involves two steps: visibility and shading. This algorithm is of no use when it comes to shading.

In this lesson, we have told you everything you needed to know about the rasterization algorithm. Everything else you can learn about is not so much about the technique itself, but more about optimizing it (how to run it in parallel, how to make it efficient, how to run it on the GPU, etc.).
In this lesson, we have told you everything you needed to know about the rasterization algorithm. Everything else you can learn about is not so much about the technique itself, but more about optimizing it (how to run it in parallel, how to make it efficient, how to run it on the GPU, etc.).

We hope to have provided one of the best introductions to the rasterization technique. If you appreciated this work and learned something while reading this lesson, please consider donating.

## Exercises

- Implement anti-aliasing. Divide the pixel into 4 sub-pixels. Generate a sample in the middle of each sample, and run the coverage test for each sample. Sum up the color of the samples and divide by 4, and set the pixel color to that result.
- Implement back-face culling. Don't render the triangle if the dot product of its normal and the view direction is lower than a certain threshold. Remember that if two vectors point in opposite directions, their dot product is negative. Also, remember that the dot product of two vectors is the cosine of the angle between the two vectors. Express the threshold value as an angle in degrees.
- Implement anti-aliasing. Divide the pixel into 4 sub-pixels. Generate a sample in the middle of each sample, and run the coverage test for each sample. Sum up the color of the samples and divide by 4, and set the pixel color to that result.
- Implement back-face culling. Don't render the triangle if the dot product of its normal and the view direction is lower than a certain threshold. Remember that if two vectors point in opposite directions, their dot product is negative. Also, remember that the dot product of two vectors is the cosine of the angle between the two vectors. Express the threshold value as an angle in degrees.
- Write the content of the z-buffer to the PPM file (you will need to remap the values of the depth buffer to the range [0,255]).

## Reference

- A Parallel Algorithm for Polygon Rasterization. Juan Pineda. Siggraph 1988.
- A Subdivision Algorithm for Computer Display of Curved Surfaces. Edwin Earl Catmull. Thesis 1974.
- Fundamentals of Texture Mapping and Image Wrapping. Paul S. Heckbert. Thesis 1989.