## Quick Review

In the previous chapter, we gave a high-level overview of the rasterization rendering technique. It can be decomposed into two main stages: first, the projection of the triangles' vertices onto the canvas, and second, the rasterization of the triangle itself. Rasterization means **"breaking apart" the triangle's shape into pixels**, or **raster element squares**; this is what pixels used to be called in the past. In this chapter, we will review the first stage. We have already described this method in the two previous lessons, thus we won't explain it here again. If you have any doubts about the principles behind perspective projection, check these lessons again. In this chapter, we will study a couple of new tricks related to projection that are going to be useful when we get to the lesson on the perspective projection matrix. We will learn about a new method to remap the coordinates of the projected vertices from screen space to Normal Device Coordinate (NDC) space. We will also learn more about the role of the z-coordinate in the rasterization algorithm and how it should be handled at the projection stage.

Keep in mind that, as already mentioned in the previous chapter, the goal of the rasterization rendering technique is to solve the visibility, or hidden surface, problem, which is to determine with parts of a 3D object are visible and which parts are hidden.

## Projection: What Are We Trying to Solve?

What we are trying to solve here at this stage of the rasterization algorithm? As explained in the previous chapter, the principle of rasterization is to figure out if pixels in the image overlap triangles. To do so, we first need to project triangles onto the image canvas and then convert their coordinates from screen space to raster space. This projection causes pixels and triangles to be defined in the same space, which means that it becomes possible to compare their respective coordinates (we can check the coordinates of a given pixel against the raster-space coordinates of a triangle's vertices).

The goal of this stage is thus to convert the vertices making up triangles from camera space to raster space.

## Projecting Vertices: Mind the Z-Coordinate!

In the previous two lessons, we mentioned that when we compute the raster coordinates of a 3D point. What we need in the end are its x- and y-coordinates (the position of the 3D point in the image). As a quick reminder, recall that these 2D coordinates are obtained by dividing the x and y coordinates of the 3D point, in camera space, by the point's respective z-coordinate (what we called the perspective divide), and then remapping the resulting 2D coordinates from screen space to NDC space, and then NDC space to raster space. Keep in mind that because the image plane is positioned at the near-clipping plane, we also need to multiply the x- and y-coordinate by the near-clipping plane. Again, we explained this process in great detail in the previous two lessons.

$$
\begin{array}{l}
Pscreen.x = \dfrac{ near * Pcamera.x }{ -Pcamera.z }\\
Pscreen.y = \dfrac{ near * Pcamera.y }{ -Pcamera.z }\\
\end{array}
$$

Note that so far, we have been considering points in screen space as 2D points (we didn't need to use the points' z-coordinate after the perspective divide). From now on, though, we will declare points in screen-space as 3D points and set their z-coordinate to the camera-space points' z-coordinate as follows:

$$
\begin{array}{l}
Pscreen.x = \dfrac{ near * Pcamera.x }{ -Pcamera.z }\\
Pscreen.y = \dfrac{ near * Pcamera.y }{ -Pcamera.z }\\
Pscreen.z = { -Pcamera.z }\\
\end{array}
$$

It is best at this point to set the projected point's z-coordinate to the inverse of the original point's z-coordinate, which, as you know by now, is negative. Dealing with positive z-coordinates will make everything simpler later on (but this is not mandatory).

![Figure 1: when two vertices in camera space have the same 2D raster coordinates, we can use the original vertices z-coordinate to find out which one is in front of the other (and thus which one is visible).](/images/rasterization/rasterizer-z.png?)

**Keeping track of the vertex z-coordinate in camera space is needed to solve the visibility problem**. Understanding why this is the case is easier if you look at Figure 1. Imagine two vertices, v1 and v2, which, when projected onto the canvas, have the same raster coordinates (as shown in Figure 1). If we project v1 before v2 then v2 will be visible in the image, when it should be v1 that is visible (v1 is clearly in front of v2). However, if we store the z-coordinate of the vertices along with their 2D raster coordinates, we can use these coordinates to define which point is closest to the camera independently of the order in which the vertices are projected (as shown in the code fragment below).

```
// project v2
Vec3f v2screen;
v2screen.x = near * v2camera.x / -v2camera.z;
v2screen.y = near * v2camera.y / -v2camera.z;
v2screen.z = -v2camera.z;

// project v1
Vec3f v1screen;
v1screen.x = near * v1camera.x / -v1camera.z;
v1screen.y = near * v1camera.y / -v1camera.z;
v1screen.z = -v1camera.z;

// If the two vertices have the same coordinates in the image, then compare their z-coordinate
if (v1screen.x == v2screen.x && v1screen.y == v2screen.y) {
    if (v1screen.z < v2screen.z){
        // if v1.z < v2.z, then store v1 in frame-buffer
        ....
    }
}
```

![Figure 2: the points on the surface of triangles that a pixel overlaps can be computed by interpolating the vertices making up these triangles. See chapter 4 for more details.](/images/rasterization/rasterizer-z2.png?)

What we want to render though are triangles, not vertices. So the question is, how does the method we just learned about apply to triangles? In short, we will use the triangle vertices' coordinates to find the position of the point on the triangle that the pixel overlaps (and thus it's z-coordinate). This idea is illustrated in Figure 2. If a pixel overlaps two or more triangles, we should be able to compute the position of the points on the triangles that the pixel overlap, and use the z-coordinates of these points as we did with the vertices to know which triangle is the closest to the camera. This method will be described in detail in [chapter 4 (The Depth Buffer. Finding the Depth Value of a Sample by Interpolation)](/lessons/3d-basic-rendering/rasterization-practical-implementation/visibility-problem-depth-buffer-depth-interpolation).

## Screen Space is Three-Dimensional

![Figure 3: the process of perspective projection. Notice here that the screen space is three-dimensional, not two-dimensional.](/images/rasterization/screen-space-3D.png?)

In summary, to go from camera space to screen space (which is the process during which the perspective divide is happening), we need to:

- Perform the perspective divide: that is, dividing the point's camera space x- and y-coordinates by the point's z-coordinate. 

  $$
  \begin{array}{l}
  Pscreen.x = \dfrac{ near * Pcamera.x }{ -Pcamera.z }\\
  Pscreen.y = \dfrac{ near * Pcamera.y }{ -Pcamera.z }\\
  \end{array}
  $$

- Also set the projected point's z-coordinate to the original point's z-coordinate (the point in camera space).

  $$
  Pscreen.z = { -Pcamera.z }
  $$

Practically, this means that our projected point is not a 2D point anymore, but a 3D point. Or, to say it differently, that screen space is not two-dimensional, but three-dimensional. In his thesis [Ed-Catmull](http://en.wikipedia.org/wiki/Edwin_Catmull) writes:

Screen-space is also three-dimensional, but the objects have undergone a perspective distortion so that an orthogonal projection of the object onto the x-y plane would result in the expected perspective image. (Ed-Catmull's Thesis, 1974).

![Figure 4: we can form an image of an object in screen space by projecting lines orthogonal (or perpendicular) to the x-y image plane.](/images/rasterization/screen-space-3D2.png?)

You should now be able to understand the previous quote. The process is illustrated in Figure 3. First, the geometry vertices are defined in camera space (top image). Then, each vertex undergoes a perspective divide. Lastly, the point's x- and y-coordinates are divided by their z-coordinate, and we set the resulting projected point's z-coordinate to the inverse of the original point's z-coordinate. This infers a change of direction in the z-axis of the screen space coordinate system. As you can see, the z-axis is now pointing inward, rather than outward (middle image in Figure 3). The most important thing to notice is that the resulting object is a deformed version of the original object. What Ed-Catmull means when he writes "an orthogonal projection of the object onto the x-y plane would result in the expected perspective image", is that, once the object is in screen space, tracing lines perpendicular to the x-y image plane from the object to the canvas gives a perspective representation of that object (as shown in Figure 4). This means that the image creation process can be seen as **a perspective projection followed by an orthographic projection**. Don't worry if you don't understand clearly the difference between perspective and orthographic projection, as it is the topic of the next lesson. Try to remember this observation, as it will come in handy later.

## Remapping Screen Space Coordinates to NDC Space

In the previous two lessons, we explained that once in screen space the x- and y-coordinates of the projected points need to be remapped to NDC space. In previous lessons, we also explained that in NDC space, points on the canvas had their x- and y-coordinates contained in the range [0,1]. On the GPU, coordinates in NDC space are contained in the range [-1,1]. We could have kept the convention [0,1], but rasterization requires us to conform to the GPU coordinate bounds of [-1,1].

You may wonder why we didn't use the [-1,1] convention in the first place. Firstly, the term "normalize" should always suggest that the value is being normalized in the range [0,1]. Secondly, it is good to be aware that several rendering systems use different conventions with respect to the concept of NDC space. The RenderMan specifications, for example, define NDC space as a space defined over the range [0,1].

Once the points have been converted from camera space to screen space, the next step is to remap them from the range [l,r] and [b,t], for the x- and y-coordinates respectively, to the range [-1,1]. The terms l, r, b, and t relate to the left, right, bottom, and top coordinates of the canvas. By re-arranging the terms, we can easily find an equation that performs the remapping we want:

$$l < x < r$$

Where x here is the x-coordinate of a 3D point in screen space (remember that from now on, we will assume that points in screen space are three-dimensional - as explained above). If we subtract l from all terms, we get:

$$0 < x - l < r - l$$

By dividing all terms by (r-l), we get:

TODO add note about the fact that r > l, so you are dividing by a non-negative number.

$$
\begin{array}{l}
0 < \dfrac {(x - l)}{(r - l)} < \dfrac {(r - l)}{(r - l)} \\
0 < \dfrac {(x - l)}{(r  -l)} < 1 \\
\end{array}
$$

We can now further develop the term in the middle of the equation:

$$0 < \dfrac {x}{(r  -l)} - \dfrac {l}{(r  -l)}< 1$$

And multiply all terms by 2:

$$0 < 2 * \dfrac {x}{(r  -l)} - 2 * \dfrac {l}{(r  -l)}< 2$$

We now subtract 1 from all terms:

$$-1 < 2 * \dfrac {x}{(r  -l)} - 2 * \dfrac {l}{(r-l)} - 1 < 1$$

If we further develop the terms and regroup them, we finally get:

$$
\begin{array}{l}
-1 < 2 * \dfrac {x}{(r  -l)} - 2 * \dfrac {l}{(r-l)} - \dfrac {(r-l)}{(r-l)} < 1 \\
-1 < 2 * \dfrac {x}{(r  -l)} + \dfrac {-2*l+l-r}{(r-l)} < 1 \\
-1 < 2 * \dfrac {x}{(r  -l)} + \dfrac {-l-r}{(r-l)} < 1 \\
-1 < \textcolor{red}{\dfrac {2x}{(r  -l)}} \textcolor{green}{- \dfrac {r + l}{(r-l)}} < 1 \\
\end{array}
$$

The last equation is very important because the red and green terms of the equation will become the coefficients of the perspective projection matrix. We will study this matrix in the next lesson. For now, we will just apply this equation **to remap the x-coordinate of a point in screen space to NDC space** (any point that lies on the canvas has its coordinates contained in the range [-1,1] when defined in NDC space). If we apply the same reasoning to the y-coordinate we get:

$$-1 < \textcolor{red}{\dfrac {2y}{(t  - b)}} \textcolor{green}{- \dfrac {t + b}{(t-b)}} < 1$$

## Putting Things Together

At the end of this lesson, we now can perform the first stage of the rasterization algorithm, which you can decompose into two steps:

- (1) Convert a point in camera space to screen space. We project a point onto the canvas, but keep in mind that we also need to store the original point's z-coordinate. The point in screen-space is tree-dimensional and the z-coordinate will be useful to solve the visibility problem later on. 

  $$
  \begin{array}{l}
  Pscreen.x = \dfrac{ near * Pcamera.x }{ -Pcamera.z }\\
  Pscreen.y = \dfrac{ near * Pcamera.y }{ -Pcamera.z }\\
  Pscreen.z = { -Pcamera.z }\\
  \end{array}
  $$

- (2) We then convert the x- and y-coordinates of these points in screen space to NDC space using the following formulas: 

  $$
  \begin{array}{l}
  -1 < \textcolor{red}{\dfrac {2x}{(r  -l)}} \textcolor{green}{- \dfrac {r + l}{(r-l)}} < 1\\
  -1 < \textcolor{red}{\dfrac {2y}{(t  - b)}} \textcolor{green}{- \dfrac {t + b}{(t-b)}} < 1
  \end{array}
  $$

  Where l, r, b, t denote the left, right, bottom, and top coordinates of the canvas.

From here, it is extremely simple to convert the coordinates to raster space. We remap the x- and y-coordinates in NDC space to the range [0,1] and multiply the resulting number by the image width and image height respectively (don't forget that, in raster space, the y-axis goes down, while in NDC space it goes up. We need to change the direction of the y-axis during this remapping process). In code, we get:

```
float nearClippingPlane = 0.1;
// point in camera space
Vec3f pCamera;
worldToCamera.multVecMatrix(pWorld, pCamera);
// convert to screen space
Vec2f pScreen;
pScreen.x = nearClippingPlane * pCamera.x / -pCamera.z;
pScreen.y = nearClippingPlane * pCamera.y / -pCamera.z;
// now convert point from screen space to NDC space (in range [-1,1])
Vec2f pNDC;
pNDC.x = 2 * pScreen.x / (r - l) - (r + l) / (r - l);
pNDC.y = 2 * pScreen.y / (t - b) - (t + b) / (t - b);
// convert to raster space and set the point's z-coordinate to -pCamera.z
Vec3f pRaster;
pRaster.x = (pNDC.x + 1) / 2 * imageWidth;
// in raster space, y is down, so invert direction
pRaster.y = (-pNDC.y + 1) / 2 * imageHeight;
// store the point's camera space z-coordinate (as a positive value)
pRaster.z = -pCamera.z;
```

Note that the coordinates of points or vertices in raster space are still defined as floating point numbers here and not integers (which is the case for pixel coordinates).

## What's Next?

We have now projected the triangle onto the canvas and converted these projected vertices to raster space. Both the vertices of the triangle and the pixel now live in the same coordinate system. We are now ready to loop over all pixels in the image and use a given technique to find out if they overlap a triangle. This is the topic of the next chapter.