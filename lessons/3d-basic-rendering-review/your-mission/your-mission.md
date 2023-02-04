

## Your Missions

It is now time to put in use the knowledge we have been through in the basic 3D rendering section. This will be an extremely valuable exercise to assess your knowledge of 3D programming. It will either expose you to the fact that some concepts are still very blurry, or it will comfort you that you are ready to move on.

The idea of this section is to give you some concrete, relatively small, and simple exercises and challenges for you to do. A short description of the exercises/challenges can be found below. So keep reading. The solutions will be given in the following lessons -- one per challenge. We will provide you with a visual result to match each exercise.

These small exercises do not come from anywhere. They are based on our experience programming graphics applications (on a modest scale). At some point, you need to check that your code produces correct results; over the years, we've developed strategies to do so. This series of challenges is based on that (valuable) experience and is designed to show you the mindset required to debug/check 3D graphics applications. Checking (does it do the right thing?) is as important as debugging (a crash?) here. It will save you time if you have to do the same.  

### What Will You Learn?

You will learn new things in this series. Nothing fundamental but likely to be extremely useful in practice. We will introduce small variants to some of the techniques studied in the basic section and put into practice things such as matrix multiplications, etc. There's no value in repeating what we already spent a great deal going into. However, we will re-iterate the principles in more direct ways. This may give you (or us) a second chance to finally get it right.

We also plan to introduce you to math libraries commonly used in our industry. Through this exercise, you will, of course, first get acquainted with these libraries. You didn't even know these libraries existed. You may have looked at them, but the code looked like old Egyptian to you. But now that you have acquired some knowledge about 3D programming, the time has come for you to realize that you can read these hieroglyphs and even understand what they mean and what they do. And that's a great confidence booster. Furthermore, studying other people's code is the primary way you will learn and progress. If you are not doing this already, this will give you a taste of the practice. It also helps you decide which coding style you prefer best. This seems anecdotal, but it's not.

In this series, we will look at Imath, glm, glh (relatively old projects used in OpenGL programs -- but there's a lot of good in looking at older code bases for reasons we will explain when we get there) and some code provided by Autodesk with the 3D software Maya. It would be good to look at the source code of Blender, but we will leave that to another lesson.

The concepts from the basic 3D rendering section are worth a year of study. However, look at these lessons as either an exam or a way of getting back into work after a summer break. _These are probably things any tech company would expect you to be able to do with eyes wide shut if you were to apply for a position as a "CG" programmer_. If you are about to apply for a job or if you are a tech company, you may find some inspiration for testing your knowledge and designing some coding tests in this content.

That's your missions. Our mission is to consolidate your knowledge and be sure you have solid foundations and a good learning methodology.

{{
## Exercise 1: given 4 vertices defining 2 triangles, project the vertices on the camera screen and store the result in a PPM file.

In this exercise, we will give you the vertices' position and some additional info, such as the image dimensions (in pixels), the cameras transformations matrices (the `world-to-camera` and `camera-to-world` matrices as an array of 16 floats), and other information about the camera (its field of view, etc.). Your mission is to build a perspective projection matrix to project the vertices onto the screen and store the result in a PPM file. 

Once you get the first result, you will be asked to recreate the `world-to-camera` and `camera-to-world` matrices from scratch. You will only be given the rotation and translation values for the camera transformation. Since you will be given the final coefficients for the matrix, it will be easier for you to check that your code does the right thing.

There is a trap in this lesson. Your output is very likely to mismatch the reference. Our second challenge aims at checking that everything is correct with the code from our first challenge. But then, if our two programs produce a result that matches why don't they match themselves with the reference? Mystery? Why is what you will need to find out (and of course, you will need to fix the issue too).
}}

{{
## Exercise 2: render the two triangles from exercise 1 using ray tracing.

In the second and last challenge, you will have to render an image of the two above triangles using ray tracing (and store the result in an image file). 

Note that the two images of the two challenges should match. But they might be different from the reference image. We will need to find the problem and adjust whatever is necessary to make the result match the reference.
}}

Good luck!