

## Exercise Description

This first exercise aims to project 4 vertices (that are making 2 triangles) onto an image plane and store the result in a PPM file. We will be provided with the 4 vertices' positions, the index array (the indices defining how these vertices are connected to form triangles), and the camera-to-world transformation matrix. We will also be provided with the horizontal field of view and the image dimensions.

Here is the info in code form:

```
float vertices[12] =

uint32_t index[6] = {};

float cam_to_world[16] = {};
float vertical_field_of_view = 45; // in degrees
float znear = 0.1;
float zfar = 100;

uint32_t width = 640, height = 480;
```

Here is the image you shall match:

![]{}

As we progress with the solution, links to lessons in which you can find specific information concerning a given technique or method will also be provided. As we go along, we will introduce you to some math libraries that have been quite popular in the CG community. Finally, we will see how some of the functions we need to solve this exercise are implemented in these libraries.

## Solution

When such an exercise is given to you, the process by which you project points onto a screen should come as an automatism. For example, on Stackoverflow, an OP [asked](https://gamedev.stackexchange.com/questions/201552/when-moving-the-camera-should-one-move-the-projection-matrix-or-move-the-world/204025#204025) "When moving the "camera" should one move the projection matrix or move the world?".

> I'm making my game engine as an exercise, and I need clarification on what I've read so far. For example, do I move the world or the camera's position when I want to move my camera? OpenGL tutorials and Scratchapixel state that you move the world, not the camera. But that confuses me. Wouldn't it be simpler to move the camera instead of, for example, applying millions of additions in worlds as big as Minecraft? What about VR headsets? Those have two cameras. How do they do it? Is there something I'm missing, or it just works like this?

This is a good starting point and opportunity to try explaining the process again. 

- When we create a 3D scene, we have objects whose vertices are defined in world space. Objects may be moved in the world using an `object-to-world` transformation matrix, but we will ignore this point. Instead, we will assume that all objects in the scene have their vertices defined in world space. Fine. Now the next step is to look at that scene through a camera. The camera will only look at a given part of the scene from a given point of view. To do so, we need to move the camera around into the scene like we do when we move our mobile phone to take a picture in the real world (or a DSLR camera if you have one). So far, so good. When you create the camera by default, it is centered at the world origin (and therefore has coordinates 0,0,0) and looks down the z-axis. That's what we need to do in principle, or to say it differently, how you can look at the process of framing a 3D scene with a virtual camera. We still need to explain how to render an image of the scene as seen through the camera. You can look at a 3D scene as some geometry whose vertices are defined in world space and a camera somehow located in that scene.

  Remember, Scratchapixel uses a right-hand coordinate system, with the y-axis looking up, the x-axis pointing to the right, and the z-axis toward you, the viewer.

  The transformation applied to the camera (translation, rotation, remember, no scale on a camera) can be baked in (for convenience) into a 4x4 matrix that we call the `camera-to-world` matrix. Why `camera`? Because it was in the default camera position originally. Why `world`? Because we moved it to a different position in the world.

  So what do we have at the end of that process? Objects' vertices in world space (their positions are defined with respect to the world's origin) and a `camera-to-world` matrix that defines the camera position and orientation into the scene with respect to that world origin. Great. 
  
- **Now comes the time to render the 3D scene as viewed through the camera**. To do so, we will apply the `world-to-camera` matrix to the scene's vertices. The `world-to-camera` is the `camera-to-world` inverse matrix. You can look at the process as if we were moving the entire scene, the objects and the camera included, to put the camera back into its default position and orientation. Note that in the process, as the objects' vertices are moving with the camera, what you see through the camera doesn't change. For example, if you hold an object in front of your mobile phone and move around, as the object moves with the camera, it will not move in the frame. This is illustrated in the short video below.

  Yes, sure, it may seem like a lot of calculations, but there is a reason why we do so. That is because projecting points onto the camera's image plane becomes very simple when we are in that configuration. It is reduced to simply dividing the vertices' x and y coordinates by their respective z coordinates. This is known as the **z or perspective divide** -- a fundamental process in creating CG images. Again projecting a point onto the camera screen is reduced to two simple divisions. And how do we move "the entire world's geometry" into camera space? By transforming the objects' vertices in world space with the `world-to-camera` matrix, of course (the inverse of the `camera-to-world` matrix).
  
  Now, there's a reason why we use a perspective projection matrix rather than simply dividing the points' x and y coordinates by their respective z-coordinate. The perspective projection not only does the z-divide for us but also encodes in the projection process the camera field of view, the image aspect ratio (and additionally remaps the depth of the point with respect to the camera's to the [0,1] or [-1,1] range depending on the preferred convention).

<iframe class="video" src="/images/projecting-3Dpoints-review/proj-review-world-to-cam-example.mp4" frameborder="0" allowfullscreen></iframe>

!!!
Note that all real-time APIs are working that way. We (developers) make them work that way because modern real-time APIs all have a programmable pipeline. So it's our responsibility to calculate the `world_to_camera` and the perspective projection matrix and pass them on to what's typically called a **vertex shader** whose primary function is precisely to project the vertices making up the models on the camera image plane. These matrices are often combined in a matrix commonly referred to (in code and the literature) as the `model_view` matrix. This matrix generally encodes the object-to-world matrix (this transforms the object's vertices from object space to world space), the world-to-camera matrix, and finally, the perspective (or orthographic) matrix. So three matrices in one. Here is an example of such a vertex shader in the WGLS format (where again `model_view_matrix` is the result of the projection matrix multiplied by the `world_to_camera` matrix multiplied by the `object_to_world` matrix - the model part in the matrix):

```
struct Uniforms {
	object_to_world : mat4x4<f32>,
	world_to_camera : mat4x4<f32>,
	projection : mat4x4<f32>,
}

@binding(0) @group(0) var<uniform> uniforms : Uniforms;

@vertex fn main(@location(0) pos: vec3<f32>)
	-> @builtin(position) {
	return uniforms.projection * uniforms.world_to_camera * object_to_world * vec4<f32>(pos, 1.0); 	
})");
```

It doesn't matter if your scene has zillions of polygons like Minecraft because GPUs are optimized to make that process light speed fast. The entire point of GPUs is that they are designed to run such operations (transforming a vertex in the image plane of the frame buffer) in parallel on a gazillion of processors. They have been designed so that making this projection step could be done as efficiently as possible, and while it may seem inefficient to process potentially millions of vertices, it succeeds in rendering 3D scenes, including potentially complex ones, in real-time; this is exactly how it works when you play Minecraft or look at some 3D scenes in VR (at 60 fps in stereo).
!!!

In summary, when you want to project a point onto the image plane of a camera, you need to apply the following steps:

1. Transform the object's vertices into camera space using the `world-to-camera` matrix.
2. Project the vertices in camera space onto the screen of the image plane using the perspective projection matrix.

To solve our exercise, we will use the following tools:

1. We only have the `camera-to-world` matrix. So we will need some code to compute the matrix inverse. This will give us the `world-to-camera` matrix. Finding the inverse of a matrix was explained in the lesson [Matrix Inverse: Gauss-Jordan Method](/lessons/mathematics-physics-for-computer-graphics/matrix-inverse/matrix-inverse)
2. We will need some code to transform points by matrices. To find out to transform points and vectors with matrices, please refer to the lesson on [Geometry](/lessons/mathematics-physics-for-computer-graphics/geometry/transforming-points-and-vectors).
3. We will need some code to build a perspective projection matrix. This function will take, as input parameters, the camera field of view, the image aspect ratio, and the znear and zfar clipping planes. The lesson [The Perspective and Orthographic Projection Matrix](/lessons/3d-basic-rendering/perspective-and-orthographic-projection-matrix/opengl-perspective-projection-matrix) provides detailed information on this topic.

Let's build these elements step by step.

## Calculating the Inverse of a Matrix

As examples of libraries that implement a matrix transformation, we will limit ourselves, in this case, to **Imath**. Imath is a library initially developed by ILM (Industrial Light and Magic), a California-based VFX studio responsible for some of the most iconic visual effects of all times. It has designed and produced visual effects for films such as Terminator, Jurassic Park, and the Star Wars franchise.

Imath was the foundation upon which ILM later released other open-source projects. Namely OpenEXR. Both Imath and OpenEXR are not managed by the [Academy Software Foundation](https://www.aswf.io/) and can be found on GitHub at this [link](https://github.com/AcademySoftwareFoundation/imath). Imath is quite popular in the CG industry, particularly within VFX and animation studios' software departments, and can often be used or referenced by other projects such as OpenVDB or USD.

As its name suggests and as mentioned, Imath focuses on providing a rich library of math functions and tools, including vector, point, and matrix manipulation, among the most important. One particularity of the library is that it supports the type `half`, real numbers that use half as many bytes as standard floats. This was devised to support float images (exr files) that were not as heavy as if their pixels had been stored using floats.

Imath supports the two common methods for inverting matrices: the Gauss-Jordan method (the only method currently explained on Scratchapixel) and the determinant-based method (that we will present in the future revision of the lesson on matrix inversion).

Let's look at the code of the matrix inversion method using the Gauss-Jordan method provided by Imath:

```
template <class T>
inline Matrix44<T>
Matrix44<T>::gjInverse (bool singExc) const
{
    int      i, j, k;
    Matrix44 s;
    Matrix44 t (*this);

    // Forward elimination

    for (i = 0; i < 3; i++)
    {
        int pivot = i;

        T pivotsize = t.x[i][i];

        if (pivotsize < 0) pivotsize = -pivotsize;

        for (j = i + 1; j < 4; j++)
        {
            T tmp = t.x[j][i];

            if (tmp < 0) tmp = -tmp;

            if (tmp > pivotsize)
            {
                pivot     = j;
                pivotsize = tmp;
            }
        }

        if (pivotsize == 0)
        {
            if (singExc)
                throw std::invalid_argument ("Cannot invert singular matrix.");

            return Matrix44 ();
        }

        if (pivot != i)
        {
            for (j = 0; j < 4; j++)
            {
                T tmp;

                tmp           = t.x[i][j];
                t.x[i][j]     = t.x[pivot][j];
                t.x[pivot][j] = tmp;

                tmp           = s.x[i][j];
                s.x[i][j]     = s.x[pivot][j];
                s.x[pivot][j] = tmp;
            }
        }

        for (j = i + 1; j < 4; j++)
        {
            T f = t.x[j][i] / t.x[i][i];

            for (k = 0; k < 4; k++)
            {
                t.x[j][k] -= f * t.x[i][k];
                s.x[j][k] -= f * s.x[i][k];
            }
        }
    }

    // Backward substitution

    for (i = 3; i >= 0; --i)
    {
        T f;

        if ((f = t.x[i][i]) == 0)
        {
            if (singExc)
                throw std::invalid_argument ("Cannot invert singular matrix.");

            return Matrix44 ();
        }

        for (j = 0; j < 4; j++)
        {
            t.x[i][j] /= f;
            s.x[i][j] /= f;
        }

        for (j = 0; j < i; j++)
        {
            f = t.x[j][i];

            for (k = 0; k < 4; k++)
            {
                t.x[j][k] -= f * t.x[i][k];
                s.x[j][k] -= f * s.x[i][k];
            }
        }
    }

    return s;
}
```

If you follow the steps, you will hopefully recognize everything we have described in the lesson on matrix inversion:
- We start with the matrix we which to invert and a matrix set to identity.
- Step 1: we swap rows if necessary to get the largest possible value for the pivot for the processed row.
- Step 2: we eliminate all elements in the matrix below the diagonal (forward elimination).
- Step 3: we set the elements along the diagonal to 1.
- Step 4: we eliminate all numbers above the diagonal (backward substitution).

The only difference with our implementation is that in step 4, Imath processes the column in reverse order, starting at index 3 and going down to 0. There is a slight implementation detail here. Because the index in the loop is decreasing and the loops stop with the index lower than 0 exactly, the index can't be of type `uint32_t`. It needs to be of type `int`. Using an `uint32_t` and going from 0 to 3 would probably be a better choice. This is a small example of the difference that may exist between various implementations and make one implementation better than others. Besides that detail, the Imath code is rather easy to read. So let's write a simple `Matrix44f` class and use this function.

For reference, you can also look at how Imath implements points and vectors.

```
class Matrix44f
{
public:
    float matrix[4][4];
};

int main()
{
	Matrix44f cam_to_world({});
	Matrix44f world_to_cam = cam_to_world.Inverse();
	std::cerr << world_to_cam << std::endl;

	// Project vertices
	// TODO
	
	return 0;
}
```

We now have the right matrix to transform the vertices from world to camera space.

```
int main()
{
    ...
	// Project vertices
	Point3f vertices[4] = {{}, {}, {}, {}};
	for (uint32_t i = 0; i < 4; ++i) {
	    Point3f vertex_camera = vertices[i] * world_to_camera;
        ...
	}
	...
}

```

## Building a Perspective Projection Matrix

We have already learned about perspective projection matrices. For this part of the exercise, we will look at another open-source library called **GLM**. The acronym stands for OpenGL Mathematics (as it was initially developed to be compatible with the OpenGL shading language GLSL). GLM is also commonly used in the industry but may be more commonly encountered in the personal projects of developers/engineers and in the game industry (many 3D-related projects on GitHub are using it). Here is a [link](https://github.com/g-truc/glm) to the library on GitHub. 

Similarly to Imath, GLM provides everything you need to manipulate a point, vector, and matrices. Still, it also provides a wide range of functions, including creating projection matrices.

The reason why GLM is popular is that it is quite exhaustive. For example, when it comes to projection matrices, GLM provides the possibility between several different options:
- Whether you want a matrix working with left-hand rule coordinate.
- Whether you want a matrix working with a right-hand rule coordinate.
- A matrix that remaps depth values from 0 to 1.
- A matrix that remaps depth values in the range -1 to 1.

GLM provides a function for each possible combination of these options. 

- `perspectiveRH_ZO`
- `perspectiveRH_NO`
- `perspectiveLH_ZO`
- `perspectiveLH_NO`

Where `LH` and `RH` stand for left-hand and right-hand, and `ZO` and `N0` stand for zero to one and a negative one (to one, that is).

These functions can be found in `/glm/ext/matrix_clip_space.inl` (the `.inl` extension here defines inline functions. The file can be included in a header to get inline functions and in a .c file to get regular function definitions). For example, here is the function as defined in the GLM library that generates a perspective projection matrix for a **right-hand coordinate system** (the convention used by Scratchapixel) in which the depth values are remapped from 0 to 1:

```
template<typename T>
GLM_FUNC_QUALIFIER mat<4, 4, T, defaultp> perspectiveRH_ZO(T fovy, T aspect, T zNear, T zFar)
{
	assert(abs(aspect - std::numeric_limits<T>::epsilon()) > static_cast<T>(0));

	T const tanHalfFovy = tan(fovy / static_cast<T>(2));

	mat<4, 4, T, defaultp> Result(static_cast<T>(0));
	Result[0][0] = static_cast<T>(1) / (aspect * tanHalfFovy);
	Result[1][1] = static_cast<T>(1) / (tanHalfFovy);
	Result[2][2] = zFar / (zNear - zFar);
	Result[2][3] = - static_cast<T>(1);
	Result[3][2] = -(zFar * zNear) / (zFar - zNear);
	return Result;
}
```

As you can see, it's best to know about things such as the right and left-hand coordinate systems. The remapping between 0 to 1 or -1 to 1 is a matter of preference. There are small differences related to precision, etc., but we will get into these details later. This won't change the visual result.

This looks a bit different than the code provided in the lesson on the [Perspective Projection Matrix](/lessons/3d-basic-rendering/perspective-and-orthographic-projection-matrix/opengl-perspective-projection-matrix), but that's only because our code was decomposing the construction of the matrix in two steps: in step 1, we calculated the camera frustum coordinates (top, bottom, left, right as shown in the figure below) that we then used in step 2 to calculate the matrix. The function provided by GLM combines these steps into one. When you do so, and if you assume that the left/right and top/bottom coordinates of the frustum are symmetrical (left and right have the same absolute value. Top and bottom also), which is almost always the case, you will realize that some terms cancel each other out and that you are left with the function as it in GLM. To prove that point, let's look at the code from another library called GLH. GLH was very popular in the OpenGL times (you can [download the library from Source Forge](https://sourceforge.net/projects/glhlib/)). Not that OpenGL is not popular anymore, but it is superseded by modern graphics APIs such as DirectX, Vulkan, or Metal. So yes, OpenGL is now something of the past. But a lot of code was written for OpenGL, and looking at GLH (the ancestor of GLM if you wish) can be super instructive.

```
void glhFrustumf(sreal *matrix, sreal left, sreal right, sreal bottom, sreal top, sreal znear, sreal zfar)
{
	sreal matrix2[16], temp, temp2, temp3, temp4, resultMatrix[16];
	temp=2.0*znear;
	temp2=right-left;
	temp3=top-bottom;
	temp4=zfar-znear;
	matrix2[0]=temp/temp2;
	matrix2[1]=0.0;
	matrix2[2]=0.0;
	matrix2[3]=0.0;
	matrix2[4]=0.0;
	matrix2[5]=temp/temp3;
	matrix2[6]=0.0;
	matrix2[7]=0.0;
	matrix2[8]=(right+left)/temp2;
	matrix2[9]=(top+bottom)/temp3;
	matrix2[10]=(-zfar-znear)/temp4;
	matrix2[11]=-1.0;
	matrix2[12]=0.0;
	matrix2[13]=0.0;
	matrix2[14]=(-temp*zfar)/temp4;
	matrix2[15]=0.0;
	MultiplyMatrices4by4OpenGL_FLOAT(resultMatrix, matrix, matrix2);

	memcpy(matrix, resultMatrix, 16*sizeof(sreal));
}

void glhPerspectivef(sreal *matrix, sreal fovyInDegrees, sreal aspectRatio, sreal znear, sreal zfar)
{
	sreal ymax, xmax;
	ymax=znear*tanf(fovyInDegrees*M_PI/360.0);
	//ymin=-ymax;
	//xmin=-ymax*aspectRatio;
	xmax=ymax*aspectRatio;
	glhFrustumf(matrix, -xmax, xmax, -ymax, ymax, znear, zfar);
}
```

In the good old times, you'd construct a perspective projection matrix by calling the `glPerspective` function (here called `glhPerspective`). As you can see, it takes a matrix as an input variable, a field of view (in degrees), an image aspect ratio, and a near and far clipping plane. Here, we are in familiar territory. Because we are given the vertical field of view, we will calculate the top coordinate first (here called `ymax`) using the near and vertical field of view. `ymin` is `-ymax`. Multiplying `ymax` and `ymin` by the image aspect ratio gives us `xmas` and `xmin`, respectively. We then pass left, right, bottom, and top (`-xmas`, `xmas`, `-ymax` and `ymax` respectively) to `glhFrustumf` that will use the frustum coordinates to set our final perspective matrix. Not that because `xmin = -xmas` and `yin = -ymax`, `matrix2[8]=(right+left)/temp2;` and `matrix2[9]=(top+bottom)/temp3;` are equal to 0 (the numerator equals 0 in both cases). We let you do the rest of the work, but you will see that if you move the code from `glhPerspetivef` into 
`glhFrustumf` and do the simplifications we spoke about. You will end with `perspectiveRH_ZO`.

![Figure xx: The bottom/left and top/right camera frustum's coorddinates](/images/perspective-matrix/projectionOpenGL2.png)

<details>
**If you don't know what the frustum is** and what these left/right, top/bottom coordinates relate to, check the lesson [The Perspective and Orthographic Projection Matrix](/lessons/3d-basic-rendering/perspective-and-orthographic-projection-matrix/).
</details>

This shows two things:

- Our code works. All code from other libraries does the same thing. In the end. Hopefully.
- It's really good to know about existing libraries, as they help us learn and check our code. Therefore the more you will know about libraries, the better.

Great. So we have our `world-to-camera` matrix, our projection matrix. What are we missing to conclude the mission? Some code for point-matrix and matrix-matrix multiplication. Let's go.

## Points and Matrix Multiplications

Let's look at the Imath code for this. You can also search in the GLM library where this is done, but it's more easily done in Imath.

For the point matrix multiplication, the Imath code looks like this:

```
template <class T>
template <class S>
IMATH_HOSTDEVICE inline void
Matrix44<T>::multVecMatrix (const Vec3<S>& src, Vec3<S>& dst) const
    IMATH_NOEXCEPT
{
    S a, b, c, w;

    a = src.x * x[0][0] + src.y * x[1][0] + src.z * x[2][0] + x[3][0];
    b = src.x * x[0][1] + src.y * x[1][1] + src.z * x[2][1] + x[3][1];
    c = src.x * x[0][2] + src.y * x[1][2] + src.z * x[2][2] + x[3][2];
    w = src.x * x[0][3] + src.y * x[1][3] + src.z * x[2][3] + x[3][3];

    dst.x = a / w;
    dst.y = b / w;
    dst.z = c / w;
}
```

Imath vector class is called `Vec3` and be specialized so that its elements x, y, z can either be floats or doubles (hence the template). Note that this method calculates `w` and add the translation part of the matrix (the elements `x[3][0]`, `x[3][1]`, and `x[3][2]`) to the respective elements of the `Vec3` instance. You should immediately realize that this function is indeed for points. Remember though from the lesson on Geometry that `w` is never defined explicitely. Yet, when it comes to transform points by matrices, we need to calculate its value (it is changed by the projection matrix as we will see later). Remember that it also needs to be normalized. As `w` is discarded at the end of the process, we don't need to explicitely set it to 1, though we need to divide the other vector elements by `w`.

Also looking at the code you should immediately notice that Imath uses the **column-major convention**. This is important since if you decide to use it for your own project, you need to be aware of the convention used by Imath of course. Scratchapixel too uses the column-major convention. The vector is on the left of the matrix defined as row of elements. So are the elements of the matrix. In this convention the vector is on the left, the matrix on the right (of course), and we need to multiply for each elements in the transformed point, the elements of the vector by the elements in the matrix' row. For example if you calculate `pt.x` where `pt` is `p` transformed by matrix `m`, then:

$$$pt.x = p.x * m[0][0] + p.y * m[1][0] + p.z * m[2][0] + \color{magenta}{m[3][0]}$$

Where the last element in the @@\mmagenta@@ color represents the translation part. For `pt.y`:

$$$pt.y = p.x * m[0][1] + p.y * m[1][1] + p.z * m[2][1] + \color{magenta}{m[3][1]}$$

And so on. If you are not clear with the concept of **row** vs. **column-major** vectors and matrices, please read the lesson on Geometry once more.

For the sake of completeness, compare the above fuction with the following which is used to transform vectors (aka directions). Note that translations values are ignored as well as the calculation for `w` and the `w` divide:

```
template <class T>
template <class S>
IMATH_HOSTDEVICE inline void
Matrix44<T>::multDirMatrix (const Vec3<S>& src, Vec3<S>& dst) const
    IMATH_NOEXCEPT
{
    S a, b, c;

    a = src.x * x[0][0] + src.y * x[1][0] + src.z * x[2][0];
    b = src.x * x[0][1] + src.y * x[1][1] + src.z * x[2][1];
    c = src.x * x[0][2] + src.y * x[1][2] + src.z * x[2][2];

    dst.x = a;
    dst.y = b;
    dst.z = c;
}
```

Let's move to the matrix-matrix multiplication. Here is the code from Imath:

```
template <class T>
IMATH_HOSTDEVICE inline IMATH_CONSTEXPR14 Matrix44<T>
Matrix44<T>::multiply (const Matrix44& a, const Matrix44& b) IMATH_NOEXCEPT
{
    const auto a00 = a.x[0][0];
    const auto a01 = a.x[0][1];
    const auto a02 = a.x[0][2];
    const auto a03 = a.x[0][3];

    const auto c00 =
        a00 * b.x[0][0] + a01 * b.x[1][0] + a02 * b.x[2][0] + a03 * b.x[3][0];
    const auto c01 =
        a00 * b.x[0][1] + a01 * b.x[1][1] + a02 * b.x[2][1] + a03 * b.x[3][1];
    const auto c02 =
        a00 * b.x[0][2] + a01 * b.x[1][2] + a02 * b.x[2][2] + a03 * b.x[3][2];
    const auto c03 =
        a00 * b.x[0][3] + a01 * b.x[1][3] + a02 * b.x[2][3] + a03 * b.x[3][3];

    const auto a10 = a.x[1][0];
    const auto a11 = a.x[1][1];
    const auto a12 = a.x[1][2];
    const auto a13 = a.x[1][3];

    const auto c10 =
        a10 * b.x[0][0] + a11 * b.x[1][0] + a12 * b.x[2][0] + a13 * b.x[3][0];
    const auto c11 =
        a10 * b.x[0][1] + a11 * b.x[1][1] + a12 * b.x[2][1] + a13 * b.x[3][1];
    const auto c12 =
        a10 * b.x[0][2] + a11 * b.x[1][2] + a12 * b.x[2][2] + a13 * b.x[3][2];
    const auto c13 =
        a10 * b.x[0][3] + a11 * b.x[1][3] + a12 * b.x[2][3] + a13 * b.x[3][3];

    const auto a20 = a.x[2][0];
    const auto a21 = a.x[2][1];
    const auto a22 = a.x[2][2];
    const auto a23 = a.x[2][3];

    const auto c20 =
        a20 * b.x[0][0] + a21 * b.x[1][0] + a22 * b.x[2][0] + a23 * b.x[3][0];
    const auto c21 =
        a20 * b.x[0][1] + a21 * b.x[1][1] + a22 * b.x[2][1] + a23 * b.x[3][1];
    const auto c22 =
        a20 * b.x[0][2] + a21 * b.x[1][2] + a22 * b.x[2][2] + a23 * b.x[3][2];
    const auto c23 =
        a20 * b.x[0][3] + a21 * b.x[1][3] + a22 * b.x[2][3] + a23 * b.x[3][3];

    const auto a30 = a.x[3][0];
    const auto a31 = a.x[3][1];
    const auto a32 = a.x[3][2];
    const auto a33 = a.x[3][3];

    const auto c30 =
        a30 * b.x[0][0] + a31 * b.x[1][0] + a32 * b.x[2][0] + a33 * b.x[3][0];
    const auto c31 =
        a30 * b.x[0][1] + a31 * b.x[1][1] + a32 * b.x[2][1] + a33 * b.x[3][1];
    const auto c32 =
        a30 * b.x[0][2] + a31 * b.x[1][2] + a32 * b.x[2][2] + a33 * b.x[3][2];
    const auto c33 =
        a30 * b.x[0][3] + a31 * b.x[1][3] + a32 * b.x[2][3] + a33 * b.x[3][3];
    return Matrix44 (
        c00,
        c01,
        c02,
        c03,
        c10,
        c11,
        c12,
        c13,
        c20,
        c21,
        c22,
        c23,
        c30,
        c31,
        c32,
        c33);
}
```

Remember from the Geometry lesson that for a matrix-matrix multiplication you need to multiply the elements of each row in the left-hand matrix by each column in the right-hand matrix (assuming you are using a column-major order convention). This can be expressed as follows:

```
inline
Matrix4 Matrix4::operator*(const Matrix4& rhs) const
{
    Matrix4 mult;
    const Matrix4& lhs = *this;
    for (uint32_t j = 0; j < 4; ++j) {
		for (uint32_t i = 0; i < 4; ++i) {
            mult[j][i] = lhs[j][0] * rhs[0][i] + lhs[j][1] * rhs[1][i] + 
                lhs[j][2] * rhs[2][i] + lhs[j][3] * rhs[3][i];
        }
    }

    return mult;
}
```

The code from Imath does the same thing as this function, but the two for-loops are unfolded. The unfolded version is preferred in production code because while less compact, by carefully arranging the term, you can avoid a few memeory fetches which should make the function faster to execute. In practice, it's hard to see the different nowardays, as when you write `for` loops, especially simple ones like the ones we have in matrix-multiplication, modern compilers are likely to do an excellent job at optimzing the code.

Congratulation. You have all the tools you need. Let's proceed to the final step. Let's project the points.

## Processing Vertices and Converting Them to Image Space

xx

## Storing the Result Into an Image File

xx