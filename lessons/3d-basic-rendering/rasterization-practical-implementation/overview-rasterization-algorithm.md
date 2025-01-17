## Everything You Wanted to Know About the Rasterization Algorithm (But Were Afraid to Ask!)

The **rasterization rendering technique** is surely the most commonly used technique to render images of 3D scenes, and yet, is probably the least understood and **properly** documented technique of all (especially compared to ray-tracing).
The **rasterization rendering technique** is surely the most commonly used technique to render images of 3D scenes, and yet, is probably the least understood and **properly** documented technique of all (especially compared to ray-tracing).

Why this is so depends on different factors. First, it's a technique from the past. We don't mean to say the technique is obsolete - quite the contrary - but that most of the techniques that are used to produce an image with this algorithm were developed somewhere between the 1960s and the early 1980s. In the world of computer graphics this time is middle-ages, and the knowledge about the papers in which these techniques were developed tends to be lost. Rasterization is also the technique used by GPUs to produce 3D graphics. Hardware technology changed a lot since GPUs were first invented, but the fundamental techniques they implement to produce images haven't changed much since the early 1980s (the hardware changed, but the underlying pipeline by which an image is formed hasn't). In fact, these techniques are so fundamental and, consequently, so deeply integrated within the hardware architecture that no one pays attention to them anymore (only people designing GPUs can tell what they do, and this is far from being a trivial task. Designing a GPU and understanding the principle of the rasterization algorithm are two different things; thus explaining the latter should not be that hard!).
Why this is so depends on different factors. First, it's a technique from the past. We don't mean to say the technique is obsolete - quite the contrary - but that most of the techniques that are used to produce an image with this algorithm were developed somewhere between the 1960s and the early 1980s. In the world of computer graphics this time is middle-ages, and the knowledge about the papers in which these techniques were developed tends to be lost. Rasterization is also the technique used by GPUs to produce 3D graphics. Hardware technology changed a lot since GPUs were first invented, but the fundamental techniques they implement to produce images haven't changed much since the early 1980s (the hardware changed, but the underlying pipeline by which an image is formed hasn't). In fact, these techniques are so fundamental and, consequently, so deeply integrated within the hardware architecture that no one pays attention to them anymore (only people designing GPUs can tell what they do, and this is far from being a trivial task. Designing a GPU and understanding the principle of the rasterization algorithm are two different things; thus explaining the latter should not be that hard!).

Regardless, we thought it was urgent and important to correct this situation. With this lesson, we believe it to be the first resource that provides a clear and complete picture of the algorithm, as well as a full practical implementation of the technique. If you found answers in this lesson that you have been desperately looking for elsewhere, then please consider donating! This work is provided to you for free and requires many hours of hard work.
Regardless, we thought it was urgent and important to correct this situation. With this lesson, we believe it to be the first resource that provides a clear and complete picture of the algorithm, as well as a full practical implementation of the technique. If you found answers in this lesson that you have been desperately looking for elsewhere, then please consider donating! This work is provided to you for free and requires many hours of hard work.

## Introduction

Rasterization and ray tracing try to solve the visibility problem, or hidden surface problem, but in a different order (the visibility problem was introduced in the lesson [Rendering an Image of a 3D Scene, an Overview](http://www.scratchapixel.com/lessons/3d-basic-rendering/rendering-3d-scene-overview/visibility-problem)). Both algorithms use techniques from geometry to solve the visibility problem. In this lesson, we will describe briefly how the rasterization algorithm works (you can write rasterisation if you prefer UK English to US English). Understanding the principle is quite simple, but implementing it requires the use of a series of techniques, notably from the field of geometry, that you will also find explained in this lesson.
Rasterization and ray tracing try to solve the visibility problem, or hidden surface problem, but in a different order (the visibility problem was introduced in the lesson [Rendering an Image of a 3D Scene, an Overview](http://www.scratchapixel.com/lessons/3d-basic-rendering/rendering-3d-scene-overview/visibility-problem)). Both algorithms use techniques from geometry to solve the visibility problem. In this lesson, we will describe briefly how the rasterization algorithm works (you can write rasterisation if you prefer UK English to US English). Understanding the principle is quite simple, but implementing it requires the use of a series of techniques, notably from the field of geometry, that you will also find explained in this lesson.

The program we will develop in this lesson is to demonstrate how rasterization works in practice. This program is important because we will use it again in the next lessons to additionally implement the ray-tracing algorithm. Having both algorithms implemented in the same program will allow us to more easily compare performance, as well as the output produced by the two rendering techniques (they should both produce the same result; at least before shading is applied). This process is a great way to better understand the pros and cons of both algorithms.
The program we will develop in this lesson is to demonstrate how rasterization works in practice. This program is important because we will use it again in the next lessons to additionally implement the ray-tracing algorithm. Having both algorithms implemented in the same program will allow us to more easily compare performance, as well as the output produced by the two rendering techniques (they should both produce the same result; at least before shading is applied). This process is a great way to better understand the pros and cons of both algorithms.

## The Rasterization Algorithm

There are not one, but multiple, rasterization algorithms. To get straight to the point, let's say that all these different algorithms are based upon the same overall principle. In other words, all these algorithms are just variants of the same idea. It is this idea, or principle, that we will refer to when we speak of rasterization in this lesson.
There are not one, but multiple, rasterization algorithms. To get straight to the point, let's say that all these different algorithms are based upon the same overall principle. In other words, all these algorithms are just variants of the same idea. It is this idea, or principle, that we will refer to when we speak of rasterization in this lesson.

What is this idea? In the previous lessons, we already talked about the difference between rasterization and ray-tracing. We also suggested that the rendering process can essentially be decomposed into two main tasks: visibility, and shading. Rasterization is essentially a method to solve the **visibility problem**. Visibility consists of being able to tell which parts of 3D objects are visible to the camera. Some parts of these objects can be hidden because they are either outside the camera's visible area, or occluded by other objects.
What is this idea? In the previous lessons, we already talked about the difference between rasterization and ray-tracing. We also suggested that the rendering process can essentially be decomposed into two main tasks: visibility, and shading. Rasterization is essentially a method to solve the **visibility problem**. Visibility consists of being able to tell which parts of 3D objects are visible to the camera. Some parts of these objects can be hidden because they are either outside the camera's visible area, or occluded by other objects.

![Figure 1: in ray tracing, we trace a ray passing through the center of each pixel in the image and then test if this ray intersects any geometry in the scene. If an intersection is found, we set the pixel color to the color of the object the ray intersected. Because a ray may intersect several objects, we need to keep track of the closest intersection distance.](/images/rasterization/raytracing-raster.png?)
![Figure 1: in ray tracing, we trace a ray passing through the center of each pixel in the image and then test if this ray intersects any geometry in the scene. If an intersection is found, we set the pixel color to the color of the object the ray intersected. Because a ray may intersect several objects, we need to keep track of the closest intersection distance.](/images/rasterization/raytracing-raster.png?)

Solving this problem can be done in essentially two ways. One way is to trace a ray through each pixel in the image and, for each ray, find the distance between the camera and the object the ray intersects (if any). The object visible through the pixel we traced the ray from is the object with the smallest intersection distance (generally denoted t). This is the technique used in ray tracing. Note that in this particular case, you render the image by looping over all pixels in the image, tracing a ray for each pixel, and then finding out if the ray intersects any of the objects in the scene. In other words, the algorithm requires two main loops. The outer loop iterates over the pixels in the image, and the inner loop iterates over the objects in the scene:
Solving this problem can be done in essentially two ways. One way is to trace a ray through each pixel in the image and, for each ray, find the distance between the camera and the object the ray intersects (if any). The object visible through the pixel we traced the ray from is the object with the smallest intersection distance (generally denoted t). This is the technique used in ray tracing. Note that in this particular case, you render the image by looping over all pixels in the image, tracing a ray for each pixel, and then finding out if the ray intersects any of the objects in the scene. In other words, the algorithm requires two main loops. The outer loop iterates over the pixels in the image, and the inner loop iterates over the objects in the scene:

```
for (each pixel in the image) { 
    Ray R = computeRayPassingThroughPixel(x,y); 
    float tClosest = INFINITY; 
    float tClosest = INFINITY; 
    Triangle triangleClosest = NULL; 
    for (each triangle in the scene) { 
        float tHit; 
        if (intersect(R, object, tHit)) { 
             if (tHit < tClosest) { 
                 triangleClosest = triangle;
                 tClosest = tHit;
        float tHit; 
        if (intersect(R, object, tHit)) { 
             if (tHit < tClosest) { 
                 triangleClosest = triangle;
                 tClosest = tHit;
             } 
        } 
    } 
    if (triangleClosest) { 
        imageAtPixel(x,y) = triangleColorAtHitPoint(triangle, tClosest); 
        imageAtPixel(x,y) = triangleColorAtHitPoint(triangle, tClosest); 
    } 
} 
```

Note that, in this example, the objects are considered to be made of triangles (and triangles only). Rather than iterating through objects, we just consider the objects as a pool of triangles and iterate through triangles instead. For reasons we have already explained in the previous lessons, the triangle is often used as the basic rendering primitive; both in ray tracing, and in rasterization (GPUs require the geometry to be triangulated).
Note that, in this example, the objects are considered to be made of triangles (and triangles only). Rather than iterating through objects, we just consider the objects as a pool of triangles and iterate through triangles instead. For reasons we have already explained in the previous lessons, the triangle is often used as the basic rendering primitive; both in ray tracing, and in rasterization (GPUs require the geometry to be triangulated).

Ray tracing is the first possible approach to solve the visibility problem. We say the technique is **image-centric** because we shoot rays from the camera into the scene (we start from the image), as opposed to the other way around, which is the approach we will be using in rasterization.
Ray tracing is the first possible approach to solve the visibility problem. We say the technique is **image-centric** because we shoot rays from the camera into the scene (we start from the image), as opposed to the other way around, which is the approach we will be using in rasterization.

![Figure 2: rasterization can roughly be decomposed into two steps. We first project the 3D vertices, which make up triangles, onto the screen using perspective projection. Then, we loop over all pixels in the image and test whether they lie within the 2D triangles. If they do, we fill the pixel with the triangle's color.](/images/rasterization/raytracing-raster2.png?)
![Figure 2: rasterization can roughly be decomposed into two steps. We first project the 3D vertices, which make up triangles, onto the screen using perspective projection. Then, we loop over all pixels in the image and test whether they lie within the 2D triangles. If they do, we fill the pixel with the triangle's color.](/images/rasterization/raytracing-raster2.png?)

Rasterization takes the opposite approach. To solve for visibility, we actually "projects" triangles onto the screen; in other words, we go from a 3D representation to a 2D representation of the triangle by using perspective projection. This can easily be done by projecting the vertices, which make up the triangle, onto the screen (using perspective projection as we just explained). The next step in the algorithm is to use some chosen technique to fill up all of the pixels in the image that are covered by the 2D triangle. These two steps are illustrated in Figure 2. From a technical point of view, they are very simple to perform. The projection steps only require a perspective divide and a remapping of the resulting coordinates from image space to raster space; a process we already covered in previous lessons. Finding out which pixels in the image are covered by the projected 2D triangles is also very simple, and will be described later.
Rasterization takes the opposite approach. To solve for visibility, we actually "projects" triangles onto the screen; in other words, we go from a 3D representation to a 2D representation of the triangle by using perspective projection. This can easily be done by projecting the vertices, which make up the triangle, onto the screen (using perspective projection as we just explained). The next step in the algorithm is to use some chosen technique to fill up all of the pixels in the image that are covered by the 2D triangle. These two steps are illustrated in Figure 2. From a technical point of view, they are very simple to perform. The projection steps only require a perspective divide and a remapping of the resulting coordinates from image space to raster space; a process we already covered in previous lessons. Finding out which pixels in the image are covered by the projected 2D triangles is also very simple, and will be described later.

What does the rasterization algorithm look like compared to the ray tracing approach? The rasterization algorithm firstly iterates over all of the triangles in the scene (in the outer loop). Then, in the inner loop, the algorithm iterates over all of the pixels in the image and, for each pixel, finds out if the current pixel is "contained" within the "projected image" of the current triangle (Figure 2). In other words, the inner and outer loops of the two algorithms are swapped.
What does the rasterization algorithm look like compared to the ray tracing approach? The rasterization algorithm firstly iterates over all of the triangles in the scene (in the outer loop). Then, in the inner loop, the algorithm iterates over all of the pixels in the image and, for each pixel, finds out if the current pixel is "contained" within the "projected image" of the current triangle (Figure 2). In other words, the inner and outer loops of the two algorithms are swapped.

```
// rasterization algorithm
for (each triangle in scene) { 
    // STEP 1: project vertices of the triangle using perspective projection
    Vec2f v0 = perspectiveProject(triangle[i].v0); 
    Vec2f v1 = perspectiveProject(triangle[i].v1); 
    Vec2f v2 = perspectiveProject(triangle[i].v2); 
    for (each pixel in image) { 
        // STEP 2: is this pixel contained in the projected image of the triangle?
        if (pixelContainedIn2DTriangle(v0, v1, v2, x, y)) { 
            image(x,y) = triangle[i].color; 
        } 
    } 
}
```

This algorithm is **object-centric**, because we start from the geometry and move our way back to the image, as opposed to the approach used in ray tracing where we start from the image and move our way back to the scene.
This algorithm is **object-centric**, because we start from the geometry and move our way back to the image, as opposed to the approach used in ray tracing where we start from the image and move our way back to the scene.

Both algorithms are simple in principle, but differ slightly in their complexity when it comes to implementation and finding solutions to the different problems they solve. In ray tracing, generating the rays is simple, but finding the intersection between the ray and geometry can be difficult (depending on the type of geometry you deal with) and potentially computationally expensive. Let's ignore ray tracing for now though. In the rasterization algorithm, we need to project vertices onto the screen, which is simple and fast. We will see that the second step, which requires finding out if a pixel is contained within the 2D representation of a triangle, has an equally simple geometric solution. In other words, computing an image using the rasterization approach relies on two very simple and fast techniques: (1) the perspective process, and (2) finding out if a pixel lies within a 2D triangle. Rasterization is a good example of an "elegant" algorithm. The two techniques it relies on have simple solutions; they are also easy to implement and produce predictable results. For all these reasons, the rasterization algorithm is very well suited for the GPU and is the rendering technique used by GPUs to generate images of 3D objects (it can also be run in parallel easily).
Both algorithms are simple in principle, but differ slightly in their complexity when it comes to implementation and finding solutions to the different problems they solve. In ray tracing, generating the rays is simple, but finding the intersection between the ray and geometry can be difficult (depending on the type of geometry you deal with) and potentially computationally expensive. Let's ignore ray tracing for now though. In the rasterization algorithm, we need to project vertices onto the screen, which is simple and fast. We will see that the second step, which requires finding out if a pixel is contained within the 2D representation of a triangle, has an equally simple geometric solution. In other words, computing an image using the rasterization approach relies on two very simple and fast techniques: (1) the perspective process, and (2) finding out if a pixel lies within a 2D triangle. Rasterization is a good example of an "elegant" algorithm. The two techniques it relies on have simple solutions; they are also easy to implement and produce predictable results. For all these reasons, the rasterization algorithm is very well suited for the GPU and is the rendering technique used by GPUs to generate images of 3D objects (it can also be run in parallel easily).

In summary:

- Converting geometry to triangles makes the visiblity problem simpler. If all primitives are converted to the triangle primitive, we can write fast and efficient code to project triangles onto the screen and check if pixels lie within these 2D triangles.
- Rasterization is **object-centric**. We project geometry onto the screen and determine their visibilities by looping over all pixels in the image.
- It relies mainly on two techniques: (1) projecting vertices onto the screen, and (2) finding out if a given pixel lies within a 2D triangle.
- The rendering pipeline which runs on GPUs is based on the rasterization algorithm.
- Converting geometry to triangles makes the visiblity problem simpler. If all primitives are converted to the triangle primitive, we can write fast and efficient code to project triangles onto the screen and check if pixels lie within these 2D triangles.
- Rasterization is **object-centric**. We project geometry onto the screen and determine their visibilities by looping over all pixels in the image.
- It relies mainly on two techniques: (1) projecting vertices onto the screen, and (2) finding out if a given pixel lies within a 2D triangle.
- The rendering pipeline which runs on GPUs is based on the rasterization algorithm.

> The fast rendering of 3D Z-buffered linearly interpolated polygons is a problem that is fundamental to state-of-the-art workstations. In general, the problem consists of two parts: 1) the 3D transformation, projection, and light calculation of the vertices, and 2) the rasterization of the polygon into a frame buffer. (A Parallel Algorithm for Polygon Rasterization, Juan Pineda - 1988)

The term **rasterization** comes from the fact that polygons (triangles in this case) are decomposed into pixels, and an image made of pixels is called a raster image. Technically, this process is referred to as the **rasterization of triangles into an image buffer or frame buffer**.
The term **rasterization** comes from the fact that polygons (triangles in this case) are decomposed into pixels, and an image made of pixels is called a raster image. Technically, this process is referred to as the **rasterization of triangles into an image buffer or frame buffer**.

> Rasterization is the process of determining which pixels are inside a triangle, and nothing more. (Michael Abrash in Rasterization on Larrabee)

Hopefully, at this point in the lesson, you have understood how the image of a 3D scene (made of triangles) is generated using the rasterization approach. Of course, what we described so far is the simplest form of the algorithm. First, it can be optimized greatly, but furthermore, we haven't explained yet what happens when two triangles projected onto the screen overlap the same pixels in the image. When that happens, how do we define which one of these two (or more) triangles is visible to the camera? We will now answer these two questions.
Hopefully, at this point in the lesson, you have understood how the image of a 3D scene (made of triangles) is generated using the rasterization approach. Of course, what we described so far is the simplest form of the algorithm. First, it can be optimized greatly, but furthermore, we haven't explained yet what happens when two triangles projected onto the screen overlap the same pixels in the image. When that happens, how do we define which one of these two (or more) triangles is visible to the camera? We will now answer these two questions.

<details>
What happens if my geometry is not made of triangles? Can I still use the rasterization algorithm? The easiest solution to this problem is to triangulate the geometry. Modern GPUs only render triangles (as well as lines and points) thus you are required to triangulate the geometry anyway. Rendering 3D geometry raises a series of problems that can be more easily resolved with triangles. You will understand why as we progress in the lesson.
</details>

## Optimizing: 2D Triangles Bounding Box

![Figure 3: to avoid iterating over all pixels in the image, we can iterate over all pixels contained in the bounding box of the 2D triangle instead.](/images/rasterization/raytracing-raster3.png?)

The problem with the naive implementation of the rasterization algorithm we gave so far is that it requires the inner loop to iterate over all pixels in the image, even though only a small number of these pixels may be contained within the triangle (as shown in Figure 3). Of course, this depends on the size of the triangle on the screen. Because we are not interested in rendering one triangle, but an object made up of potentially a few hundred to a few million triangles, it is unlikely that, in a typical production scenario, these triangles will be very large in the image.
The problem with the naive implementation of the rasterization algorithm we gave so far is that it requires the inner loop to iterate over all pixels in the image, even though only a small number of these pixels may be contained within the triangle (as shown in Figure 3). Of course, this depends on the size of the triangle on the screen. Because we are not interested in rendering one triangle, but an object made up of potentially a few hundred to a few million triangles, it is unlikely that, in a typical production scenario, these triangles will be very large in the image.

![Figure 4: once the bounding box around the triangle is computed, we can loop over all pixels contained in the bounding box and test if they overlap the 2D triangle.](/images/rasterization/raytracing-raster4.png?)

There are different ways of minimizing the number of tested pixels, but the most common one consists of computing the 2D bounding box of the projected triangle and iterating over the pixels contained in that 2D bounding box, rather than iterating over the pixels of the entire image. While some of these pixels might still lie outside the triangle, at least on average it can considerably improve the performance of the algorithm. This idea is illustrated in Figure 3.
There are different ways of minimizing the number of tested pixels, but the most common one consists of computing the 2D bounding box of the projected triangle and iterating over the pixels contained in that 2D bounding box, rather than iterating over the pixels of the entire image. While some of these pixels might still lie outside the triangle, at least on average it can considerably improve the performance of the algorithm. This idea is illustrated in Figure 3.

Computing the 2D bounding box of a triangle is very simple: we just need to find the minimum and maximum x- and y-coordinates of the three vertices making up the triangle in raster space. This is illustrated in the following pseudo code:
Computing the 2D bounding box of a triangle is very simple: we just need to find the minimum and maximum x- and y-coordinates of the three vertices making up the triangle in raster space. This is illustrated in the following pseudo code:

```
// define the bounding box min and max values
Vec2f bbmin = [INFINITY, INFINITY], bbmax = [-INFINITY, -INFINITY]; 
// define the bounding box min and max values
Vec2f bbmin = [INFINITY, INFINITY], bbmax = [-INFINITY, -INFINITY]; 
Vec2f vproj[3]; 
for (int i = 0; i < 3; ++i) { 
    // convert the vertices of the current triangle to raster space
    // convert the vertices of the current triangle to raster space
    vproj[i] = projectAndConvertToNDC(triangle[i].v[i]); 
    // coordinates are in raster space, but are still floats, not integers
    // coordinates are in raster space, but are still floats, not integers
    vproj[i].x *= imageWidth; 
    vproj[i].y *= imageHeight; 
    // record the minimum and maximum x- and y-coordinates into the bounding box
    if (vproj[i].x < bbmin.x) bbmin.x = vproj[i].x; 
    if (vproj[i].y < bbmin.y) bbmin.y = vproj[i].y; 
    if (vproj[i].x > bbmax.x) bbmax.x = vproj[i].x; 
    if (vproj[i].y > bbmax.y) bbmax.y = vproj[i].y; 
    // record the minimum and maximum x- and y-coordinates into the bounding box
    if (vproj[i].x < bbmin.x) bbmin.x = vproj[i].x; 
    if (vproj[i].y < bbmin.y) bbmin.y = vproj[i].y; 
    if (vproj[i].x > bbmax.x) bbmax.x = vproj[i].x; 
    if (vproj[i].y > bbmax.y) bbmax.y = vproj[i].y; 
} 
```

Once we have calculated the 2D bounding box of the triangle (in raster space), we need to loop over the pixels defined by that box. You need to be very careful about how you convert the triangle coordinates to raster space, which in our code are defined as floats rather than integers. First, note that one or two vertices may be projected outside the boundaries of the image canvas. Thus, their raster coordinates may be lower than 0 or greater than the image size. We solve this problem by clamping the pixel coordinates to the range [0, Image Width - 1] for the x coordinate, and [0, Image Height - 1] for the y coordinate. Furthermore, we will need to round off the minimum and maximum coordinates of the bounding box to the nearest integer value (note that this works fine when we iterate over the pixels in the loop, because we initialize the x and y variables to `xmim` or `ymin` respectively, and break from the loop when either the x or y variable is lower or equal to `xmax` or `ymax` respectively). All these tests need to be applied before using the final fixed point (or integer) bounding box coordinates in the loop. Here is the pseudo-code:
Once we have calculated the 2D bounding box of the triangle (in raster space), we need to loop over the pixels defined by that box. You need to be very careful about how you convert the triangle coordinates to raster space, which in our code are defined as floats rather than integers. First, note that one or two vertices may be projected outside the boundaries of the image canvas. Thus, their raster coordinates may be lower than 0 or greater than the image size. We solve this problem by clamping the pixel coordinates to the range [0, Image Width - 1] for the x coordinate, and [0, Image Height - 1] for the y coordinate. Furthermore, we will need to round off the minimum and maximum coordinates of the bounding box to the nearest integer value (note that this works fine when we iterate over the pixels in the loop, because we initialize the x and y variables to `xmim` or `ymin` respectively, and break from the loop when either the x or y variable is lower or equal to `xmax` or `ymax` respectively). All these tests need to be applied before using the final fixed point (or integer) bounding box coordinates in the loop. Here is the pseudo-code:

```
... 
uint xmin = std::max(0, std:min(imageWidth - 1, std::floor(bbmin.x))); 
uint ymin = std::max(0, std:min(imageHeight - 1, std::floor(bbmin.y))); 
uint xmax = std::max(0, std:min(imageWidth - 1, std::floor(bbmax.x))); 
uint ymax = std::max(0, std:min(imageHeight - 1, std::floor(bbmax.y))); 
for (y = ymin; y <= ymax; ++y) { 
uint xmin = std::max(0, std:min(imageWidth - 1, std::floor(bbmin.x))); 
uint ymin = std::max(0, std:min(imageHeight - 1, std::floor(bbmin.y))); 
uint xmax = std::max(0, std:min(imageWidth - 1, std::floor(bbmax.x))); 
uint ymax = std::max(0, std:min(imageHeight - 1, std::floor(bbmax.y))); 
for (y = ymin; y <= ymax; ++y) { 
    for (x = xmin; x <= xmax; ++x) { 
        // check if current pixel lies in triangle
        // check if current pixel lies in triangle
        if (pixelContainedIn2DTriangle(v0, v1, v2, x, y)) { 
            image(x,y) = triangle[i].color; 
        } 
    } 
} 
```

<details>
Note that production rasterizers use more efficient methods than looping over the pixels contained in the bounding box of the triangle. As mentioned, many of the pixels do not overlap the triangle, and testing if these pixel samples overlap the triangle is a waste. We won't study these more optimized methods in this lesson.
</details>

<details>
If you already studied this algorithm or studied how GPUs render images, you may have heard or read that the coordinates of projected vertices are sometimes converted from floating point to **fixed point numbers** (in other words integers). The reason behind this conversion is that basic operations such as multiplication, division, addition, etc. on fixed point numbers can be done very quickly (compared to the time it takes to do the same operations with floating point numbers). This used to be the case in the past and GPUs are still designed to work with integers at the rasterization stage of the rendering pipeline. However modern CPUs generally have FPUs (floating-point units), so if your program runs on the CPU, there is probably little to no advantage to using fixed point numbers (it actually might even run slower).
If you already studied this algorithm or studied how GPUs render images, you may have heard or read that the coordinates of projected vertices are sometimes converted from floating point to **fixed point numbers** (in other words integers). The reason behind this conversion is that basic operations such as multiplication, division, addition, etc. on fixed point numbers can be done very quickly (compared to the time it takes to do the same operations with floating point numbers). This used to be the case in the past and GPUs are still designed to work with integers at the rasterization stage of the rendering pipeline. However modern CPUs generally have FPUs (floating-point units), so if your program runs on the CPU, there is probably little to no advantage to using fixed point numbers (it actually might even run slower).
</details>

## The Image-Buffer (or Frame-Buffer)
## The Image-Buffer (or Frame-Buffer)

Our goal is to produce an image of the scene. We have two ways of visualizing the result of our program; (1) by displaying the rendered image directly on the screen, or (2) saving the image to disk, and using a program, such as Photoshop, to preview the image later on. In both cases, we somehow need to store the image that is being rendered while it's being rendered. For that purpose, we use what we call in Computer Graphics (CG) an image-buffer, or **frame-buffer**. It is nothing else than a two-dimensional array of colors that has the same size as the image. Before the rendering process starts, the frame-buffer is created and the pixels are all set to black. At render time, when the triangles are rasterized, if a given pixel overlaps a given triangle, then we store the color of that triangle in the frame-buffer at that pixel location. When all triangles have been rasterized, the frame-buffer will contain the image of the scene. All that is left to do is either display the content of the buffer on the screen, or save its content to a file. In this lesson, we will choose the latter option.
Our goal is to produce an image of the scene. We have two ways of visualizing the result of our program; (1) by displaying the rendered image directly on the screen, or (2) saving the image to disk, and using a program, such as Photoshop, to preview the image later on. In both cases, we somehow need to store the image that is being rendered while it's being rendered. For that purpose, we use what we call in Computer Graphics (CG) an image-buffer, or **frame-buffer**. It is nothing else than a two-dimensional array of colors that has the same size as the image. Before the rendering process starts, the frame-buffer is created and the pixels are all set to black. At render time, when the triangles are rasterized, if a given pixel overlaps a given triangle, then we store the color of that triangle in the frame-buffer at that pixel location. When all triangles have been rasterized, the frame-buffer will contain the image of the scene. All that is left to do is either display the content of the buffer on the screen, or save its content to a file. In this lesson, we will choose the latter option.

<details>
In programming, there is no solution to display images on the screen that is cross-platform (which is a shame). For this reason, it is better to store the content of the image in a file and use a cross-platform application, such as Photoshop or another image editing tool, to view the image. Of course, the software you will be using to view the image needs to support the image format that the image is saved in. In this lesson, we will use the very simple PPM image file format.
In programming, there is no solution to display images on the screen that is cross-platform (which is a shame). For this reason, it is better to store the content of the image in a file and use a cross-platform application, such as Photoshop or another image editing tool, to view the image. Of course, the software you will be using to view the image needs to support the image format that the image is saved in. In this lesson, we will use the very simple PPM image file format.
</details>

## When Two Triangles Overlap the Same Pixel: The Depth Buffer (or Z-Buffer)

Keep in mind that the goal of the rasterization algorithm is to solve the visibility problem. To display 3D objects, it is necessary to determine which surfaces are visible. In the early days of computer graphics, two methods were used to solve the "hidden surface" (visibility) problem: the **Newell algorithm** and the **z-buffer**. We only mention the Newell algorithm for historical reasons, but we won't study it in this lesson because it is not used anymore. We will only study the z-buffer method, which is used by GPUs.
Keep in mind that the goal of the rasterization algorithm is to solve the visibility problem. To display 3D objects, it is necessary to determine which surfaces are visible. In the early days of computer graphics, two methods were used to solve the "hidden surface" (visibility) problem: the **Newell algorithm** and the **z-buffer**. We only mention the Newell algorithm for historical reasons, but we won't study it in this lesson because it is not used anymore. We will only study the z-buffer method, which is used by GPUs.

![Figure 5: when a pixel overlaps two triangles, we set the pixel color to the color of the triangle with the smallest distance to the camera pixel.](/images/rasterization/raytracing-raster5.png?)
![Figure 5: when a pixel overlaps two triangles, we set the pixel color to the color of the triangle with the smallest distance to the camera pixel.](/images/rasterization/raytracing-raster5.png?)

There is one last thing that we need to do to get a basic rasterizer working - we need to account for the fact that more than one triangle may overlap the same pixel in the image (as shown in Figure 5). When this happens, how do we decide which triangle is visible? The solution to this problem is very simple. We will use what we call a **z-buffer**, which is also called a **depth buffer**. You may have heard or read about these two terms often. A z-buffer is nothing more than a two-dimensional array that has the same dimension as the image, however, rather than being an array of colors, it is simply an array of floating point numbers. Before we start rendering the image, we initialize each pixel in this array to a very large number. When a pixel overlaps a triangle, we read the value stored in the z-buffer at that pixel location. As you might have guessed, this array is used to store the distance from the camera to the nearest triangle which overlaps any pixel in the image. Since this value is initially set to infinity (or a very large number), the first time we find that a given pixel X overlaps a triangle T1, the distance from the camera to that triangle will be lower than the value stored in the z-buffer. After this comparison, we replace the value stored in the z-buffer for that pixel with the distance between the camera and T1. Next, when the same pixel X is tested, and we find that it overlaps another triangle T2, we then compare the distance between the camera and this new triangle to the distance stored in the z-buffer (which at this point, will store the distance between the camera and the first triangle T1). If the distance between the camera and the second triangle is lower than the distance between the camera and the first triangle, then T2 is visible and T1 is hidden by T2. In this case, we update the value in the z-buffer with the distance between the camera and T2. Otherwise, T1 is visible and T2 is hidden by T1, so the z-buffer doesn't need to be updated since the first triangle T1 is still the closest triangle we found for that pixel so far. As you can see **the z-buffer is used to store the distance of each pixel to the nearest object in the scene** (we don't really use the distance, but we will give the details further in the lesson). In Figure 5, we can see that the red triangle is behind the green triangle along the z-axis in 3D space. If we were to render the red triangle first, and the green triangle second, then for a pixel that would overlap both triangles, we would have to write into the z-buffer 3 different values at that pixel location. First, we a very large number (that happens when the z-buffer is initialized), and then the distance to the red triangle, and finally the distance to the green triangle.
There is one last thing that we need to do to get a basic rasterizer working - we need to account for the fact that more than one triangle may overlap the same pixel in the image (as shown in Figure 5). When this happens, how do we decide which triangle is visible? The solution to this problem is very simple. We will use what we call a **z-buffer**, which is also called a **depth buffer**. You may have heard or read about these two terms often. A z-buffer is nothing more than a two-dimensional array that has the same dimension as the image, however, rather than being an array of colors, it is simply an array of floating point numbers. Before we start rendering the image, we initialize each pixel in this array to a very large number. When a pixel overlaps a triangle, we read the value stored in the z-buffer at that pixel location. As you might have guessed, this array is used to store the distance from the camera to the nearest triangle which overlaps any pixel in the image. Since this value is initially set to infinity (or a very large number), the first time we find that a given pixel X overlaps a triangle T1, the distance from the camera to that triangle will be lower than the value stored in the z-buffer. After this comparison, we replace the value stored in the z-buffer for that pixel with the distance between the camera and T1. Next, when the same pixel X is tested, and we find that it overlaps another triangle T2, we then compare the distance between the camera and this new triangle to the distance stored in the z-buffer (which at this point, will store the distance between the camera and the first triangle T1). If the distance between the camera and the second triangle is lower than the distance between the camera and the first triangle, then T2 is visible and T1 is hidden by T2. In this case, we update the value in the z-buffer with the distance between the camera and T2. Otherwise, T1 is visible and T2 is hidden by T1, so the z-buffer doesn't need to be updated since the first triangle T1 is still the closest triangle we found for that pixel so far. As you can see **the z-buffer is used to store the distance of each pixel to the nearest object in the scene** (we don't really use the distance, but we will give the details further in the lesson). In Figure 5, we can see that the red triangle is behind the green triangle along the z-axis in 3D space. If we were to render the red triangle first, and the green triangle second, then for a pixel that would overlap both triangles, we would have to write into the z-buffer 3 different values at that pixel location. First, we a very large number (that happens when the z-buffer is initialized), and then the distance to the red triangle, and finally the distance to the green triangle.

You may wonder how we find the distance from the camera to the triangle. Let's first look at an implementation of this algorithm in pseudo-code and we will come back to this point later (for now let's just assume that the function pixelContainedIn2DTriangle computes the distance for us):
You may wonder how we find the distance from the camera to the triangle. Let's first look at an implementation of this algorithm in pseudo-code and we will come back to this point later (for now let's just assume that the function pixelContainedIn2DTriangle computes the distance for us):

```
// a z-buffer is just a 2D array of floats
// a z-buffer is just a 2D array of floats
float buffer = new float [imageWidth * imageHeight]; 
// initialize the distance for each pixel to a very large number
for (uint32_t i = 0; i < imageWidth * imageHeight; ++i) {
for (uint32_t i = 0; i < imageWidth * imageHeight; ++i) {
    buffer[i] = INFINITY; 
}
}
 
for (each triangle in scene) { 
    // project vertices
    ... 
    // compute bbox of the projected triangle
    ... 
    for (y = ymin; y <= ymin; ++y) { 
        for (x = xmin; x <= xmax; ++x) { 
            // check if current pixel lies in triangle
            // check if current pixel lies in triangle
            float z;  //distance from the camera to the triangle 
            if (pixelContainedIn2DTriangle(v0, v1, v2, x, y, z)) { 
                // If the distance to that triangle is lower than the distance stored in the
                // z-buffer, update the z-buffer, and update the image at pixel location (x,y)
                // z-buffer, update the z-buffer, and update the image at pixel location (x,y)
                // with the color of that triangle
                if (z < zbuffer(x,y)) { 
                    zbuffer(x,y) = z; 
                    image(x,y) = triangle[i].color; 
                } 
            } 
        }
        }
    } 
} 
```

## What's Next?

This is only a very high-level description of the algorithm (Figure 6), but this should hopefully give you an idea of what we will need in the program to produce an image. We will need:
This is only a very high-level description of the algorithm (Figure 6), but this should hopefully give you an idea of what we will need in the program to produce an image. We will need:

- A frame-buffer (a 2D array of colors),
- A frame-buffer (a 2D array of colors),
- A depth-buffer (a 2D array of floats),
- Triangles (the geometry making up the scene),
- A function to project vertices of the triangles onto the canvas,
- A function to rasterize the projected triangles,
- Some code to save the content of the frame-buffer to disk.
- Some code to save the content of the frame-buffer to disk.

![Figure 6: schematic view of the rasterization algorithm.](/images/rasterization/rasterization-schema.png?)

In the next chapter, we will see how coordinates are converted from camera to raster space. The method is identical to the one we studied and presented in the previous lesson, however, we will present a few more tricks along the way. In chapter three, we will learn how to rasterize triangles. In chapter four, we will study in detail how the z-buffer algorithm works. As usual, we will conclude this lesson with a practical example.

PROOFREADER NOTE: There is no practical example given at the end of this lesson (this markdown document), despite being promised in the previous paragraph.