In this chapter, we will learn about atmospheric scattering. We recommend reading the lesson on volume rendering and subsurface scattering. They share with this topic almost the same concepts. Atmospheric scattering can be seen as an extension of volume rendering.

## Introduction

For centuries, the sky has been a subject of fascination for many artists who tried to depict its colors with as much accuracy as possible. Many generations of physicists and mathematicians before the 19th-20th century also probably got obsessed with trying to figure out what could cause the sky to appear orange at sunset and sunrise and blue during the day. Historically, the discovery of atmospheric scattering is attributed to **Rayleigh** (also known as John Strutt), a Nobel prize English physicist whose main body of work was produced in the last 19th century (Rayleigh succeeded **Maxwell** at Cambridge University who is known for his work on electromagnetism. A large portion of the computer graphics research relates to Maxwell's work. Rayleigh also studied black bodies with **Max Planck**). Atmospheric scattering also has an effect on an object's appearance. This effect is known as **aerial perspective** and was first observed, studied, and reproduced by Leonardo Da Vinci in his paintings (in fact this technique can be found in paintings prior to Da Vinci's work). As the distance between an object and an observer increases, the colors of the object is replaced with the colors of the atmosphere (the sky is usually blue during the day and red-orange at sunrise and sunset). This is an important visual clue as we, humans, are used to evaluate the distance of objects in a landscape by comparing how blue they appear in relation to each other. In a (digital) painting, aerial perspective can greatly help add depth to an image (the brain can more easily process that an object is further away than another), as illustrated in the following reproduction of a Da Vinci painting (Madonna of the Rocks).

![](/images/atmospheric%20scattering/as-vinci.png?)

In more recent history, that is, the history of computer graphics, **Nishita** wrote a seminal paper in 1993 entitled "**Display of the Earth Taking into Account Atmospheric Scattering**" in which he describes an algorithm to simulate the colors of the sky. Interestingly enough, his paper and the following one he wrote in 1996 on the same topic ("**Display Method of the Sky Color Taking into Account Multiple Scattering**") was not entirely about atmospheric simulation but more about the realistic rendering of the Earth (including the rendering of ocean surfaces, clouds, and continents) seen from outer space.

![Figure 1: top, an actual photograph of the Earth from outer space. Bottom, a simulation using Nishita's model. These images have been copied from the paper "Display of the Earth taking into account Atmospheric Scattering" written by Nishita in 1993 (c) Siggraph.](/images/atmospheric%20scattering/as-nishita1.png?)

This reminds us that in these years, the development of computer graphics techniques was driven by manufacturing industries to simulate their products accurately or to provide a 3D virtual training environment rather than by the entertainment industries (films and games). The technique described by Nishita to simulate the sky hasn't changed much since his time. Most of the research in this area focused on implementing his algorithms on the GPU without any noticeable improvement to the simulation quality (the focus was more on speed and real-time simulation). Another sky model exists, though, which was described by **Preetham & al.** in a paper published at Siggraph in 1999 (**"A Practical Analytic Model for Daylight**"). As its title suggests, this is an analytical model which provides an accurate simulation of the sky colors; however, it is more restrictive than the technique proposed by Nishita (for instance, the model only works for an observer located on the ground). It is worth mentioning a paper published by **Jensen & al.** at Siggraph in 2011 on the simulation of a night sky ("**A Physically-Based Nightsky Model**"). This chapter will propose an implementation of the Nishita model. In the following chapters, we will study the other algorithms.

In the next paragraph, we will describe the various phenomena which, once put together, are responsible for the colors of the sky. We will then show how Nishita's algorithm simulates these phenomena. Once we have the model implemented in a C++ program, we will show that we can easily extend the technique to simulate aerial perspective and that by varying the model's parameters, we can also create alien skies.

## Atmospheric Model

An atmosphere is a layer of gas surrounding a planet. This atmosphere stays in place because of the planet's gravity. What defines this atmosphere is mainly its thickness and composition (the different elements it is made of). The earth's atmosphere thickness is about 100 km (delimited by the Kármán line) and is made of oxygen, nitrogen, argon (a gas discovered by Rayleigh), carbon dioxide, and water vapor. However, in the simulation, we will use a thickness of 60km (we will only consider scattering in the first two layers of the Earth's atmosphere, the troposphere and the stratosphere). If you read the lesson on volume rendering (we invite you to do it now before you continue if you haven't yet), you will know that photons are scattered when they collide with particles. When light travels through a volume (the atmosphere) in a specific direction, photons are deflected in other directions when they collide with particles. This phenomenon is called scattering. How frequently particles scatter light depends on the particles' properties (mainly their size) and their density in the volume (how many particles/molecules are in one unit of volume). We now know the size of the molecules that the Earth's atmosphere is made of and their concentration/density. We will use these scientific measurements for our atmospheric scattering simulation. One more critical aspect of the atmosphere surrounding the planet is that the density of these particles decreases with height. In other words, we find many more molecules per unit cube of air at sea level than 2 km above the sea. This is an important fact as the amount of scattering depends on the density of the molecules, as we just mentioned. As light travels in the atmosphere from the upper layer to the ground, we will need to sample the atmosphere density along the ray at regular intervals and compute the amount of scattering based on the atmosphere density at these sample positions (we perform a numerical integration which we explain in detail further down). Most papers (including Nishita's) assume that the atmospheric density decreases exponentially with height. In other words, it can be modeled with a simple equation of the form:

$$density(h)=density(0) e^{-\frac{h}{H}}.$$

Where density(0) is the air density at sea level, \(h\) is the current height (altitude), where we measure the atmospheric density. \(H\) is the thickness of the atmosphere if its density were uniform (in scientific papers, \(H\) is called the **scale height** and relates to the altitude by which the atmosphere's density decreases by a factor of \(e\). \(H\) depends on temperature). Some papers claim that this model is a good approximation. However, some others claim it to be incorrect and prefer to model the sky density as a series of concentric layers, each determined by their thickness and density (XX please quote paper here XX).

<detais>
![](/images/atmospheric%20scattering/as-nishita2.png?)

Modeling the sky densities as a series of layers (called spherical shells in Nishita's paper) also presents an advantage from a computational point of view. As we will perform a numerical integration along the path of the light towards the viewer, samples taken at a position in the atmosphere where the density is high are more critical than samples taken where the density is low. Performing a "constant step" integration along the ray would result in poor convergence. Today, computers are so fast that we can brute force this computation by taking regular steps but in Nishita's time, optimizing the algorithm was necessary. He proposed a model where the atmosphere is represented as a series of spherical shells, separated by small intervals at low altitudes and long intervals at high altitudes. However, in the diagram published in the paper, the thickness of the layers very much has an exponential variation. This is not surprising as the radius of each shell is precisely determined in his model by the density distribution of air molecules (and the formula he uses to compute this density is very similar to the one we have written above).
</details>

![Figure 2: aerosols present in the atmosphere are responsible for haze.](/images/atmospheric%20scattering/as-aerosol.png?)

![Figure 3: a simple graphical representation of the atmospheric model we will use in this lesson. It is defined by the planet's radius (Re) and the radius of the atmosphere (Ra). The atmosphere comprises aerosols (mainly present at low altitudes) and air molecules. The density of particles decreases exponentially with altitude. This diagram is not to scale](/images/atmospheric%20scattering/as-atmosmodel.png?)


The atmosphere is made of tiny particles (air molecules) mixed up at low altitudes with bigger particles called **aerosols**. These particles can be anything from dust or sand raised by wind or be there because of air pollution. They certainly impact the atmosphere's appearance significantly as they don't scatter light the same way air molecules do. The scattering of light by air molecules is called **Rayleigh scattering**, and the scattering of light by aerosols is called **Mie scattering**. In short, Rayleigh scattering (the scattering of light by air molecules) is responsible for the blue color of the sky (and its red-orange colors at sunrise and sunset). In contrast, Mie scattering (the scattering of light by aerosols) is usually responsible for the white-grey haze that you typically see above polluted cities (haze obscures the clarity of the sky, see figure 2).

The model would be incomplete without mentioning that it will require the user to specify the planet's radius and the sky. These numbers will compute the altitude of the sample positions taken along the view and light rays. For Earth, we will use Re = 6360 km (e for Earth) and Ra = 6420 km (a for atmosphere). All distances and parameters from our model related to distance should be expressed in the same unit system (either km, m, etc.).

<div class="question">For atmospheric models, this can be tricky as the size of the planets can be large. For such dimensions, the kilometer is usually a more suitable choice. However, scattering coefficients are very small and are more easily expressed in meters or millimeters (even nanometers for light wavelengths). Choosing a good unit is also driven by floating precision limitations (that is, if the numbers are too big or too small, the floating representation of these numbers by the computer can become inaccurate, which would result in calculation errors). Input parameters to the model can be expressed in different units and internally converted by the program.</div>

![Figure 4: the sun is so far away from the Earth, that each light ray reaching the Earth's atmosphere can be considered as being parallel to each other.](/images/atmospheric%20scattering/as-sun.png?)

Finally, we will finish the description of our model by noting that the sun is the primary illumination source of our sky. The sun is so far from the Earth that we can assume that all the light rays reaching the atmosphere are parallel. We will further show how this observation can simplify the implementation of our model.

## Rayleigh Scattering

Rayleigh discovered light scattering by air molecules in the late 19th century. He showed, in particular, that this phenomenon has a strong wavelength dependency and that air molecules scatter blue light more than green and red light. This form of scattering and the equation he came up with to compute the scattering coefficients for volumes made of molecules, such as the one we find in the atmosphere, only apply to particles whose sizes are much smaller than the wavelengths making up visible light (the particle should be at least one-tenth smaller than the scattered wavelength). Wavelengths for visible light vary from 380 to 780 nm; 440, 550, and 680 are the peaks for blue, green, and red light, respectively. We will be using these values for the rest of this lesson. The Rayleigh scattering equation provides the scattering coefficients of a volume for which we know the molecular density. In the literature on atmospheric scattering, the scattering coefficients are denoted by the greek letter β (beta).

$$\beta_R^s (h, \lambda) = \frac{8\pi^3(n^2-1)^2}{3N\lambda^4} e^{-\frac{h}{H_R}}$$

The superscript _S_ stands for scattering and the subscript \(R\) for Rayleigh (to differentiate these coefficients from the Mie scattering coefficients). In this equation, \(h\) is the altitude, \(\lambda\) (lambda) is the wavelength, \(N\) is the molecular density at sea level, n is the index of refraction of air, and \(H_R\) is the scale height. In our simulation, we will use \(H_R\) = 8km (see scale height on the web for more accurate measurements if needed). As you can see with this equation, light with a short wavelength (such as blue light) will give a higher value for \(\beta\); than light with a long wavelength (red) which explains why the sky appears blue during the day. As light from the sun travels through the atmosphere, more blue light is scattered toward the observer than green and red light. Why does it appear red-orange at sunrise and sunset, then? In that particular configuration, the light from the sun has to travel a much longer portion of the atmosphere to reach the observer's position than it does when the sun is above your head (the **zenith** position). That distance being considerably longer, most of the blue light has been scattered away before it can reach the observer's location, while some red light, which is not scattered as often as blue light, remains. Hence the red-orange appearance of the sky at sunrise and sunset (which sometimes can either be purple or slightly green).

![Figure 5: when the sun is at its zenith, sunlight travels a short distance before it reaches the eye. At sunset (or sunrise), sunlight travels a greater distance before it reaches the observer's eyes. In the first case, blue light is scattered toward the eye, and the sky appears blue. In the second case, most of the blue light has been scattered away before it can have a chance to reach the eye. Only red-green light reaches the observer's position, which explains why the sky looks red-orange when the sun is at the horizon.](/images/atmospheric%20scattering/as-figure2.png?)

We could use some measurements for \(N\), \(n\), etc. to compute a value for \(\beta\), but we will use precomputed values that correspond to the scattering coefficients of the sky at sea level (\(33.1 \mathrm{e^{-6}} \mathrm{m^{-1}}\), \(13.5 \mathrm{e^{-6}} \mathrm{m^{-1}}\) and \(5.8 \mathrm{e^{-6}} \mathrm{m^{-1}}\) for wavelengths 440, 550 and 680 respectively. Note how the scattering coefficients for blue light are greater than those for green and red light). By applying the equation's exponential part (the right term), we can adjust these coefficients for any given altitude \(h\).

<details>
One of the main problems with Nishita's papers and the other papers we mention in this lesson is that they either don't give the values used for N, n, etc., to produce the images shown in the papers. When measurements are used, authors of the papers don't usually cite their sources. Finding scientific data for molecular density at sea levels is not apparent, and if you have any helpful link or information on this topic that you could provide us with, we would like to hear from you. The sea level scattering coefficients we use in this lesson can be found in two papers: _Efficient Rendering of Atmospheric Phenomena_, by Riley et al. and _Precomputed Atmospheric Scattering_ by Bruneton et al.
</details>

We won't go into much detail about air density, but interesting information can be found on the internet. In this lesson, all we need are average scattering coefficients at sea level that work well for recreating the colors of the sky, but learning how to compute these coefficients will be of interest to any curious reader who wishes to have more control over the atmospheric model.

Read the lesson on Volume Rendering. You will know that the physical models used to render participating media are the **scattering coefficients**, the **absorption coefficients**, and the **phase function**, which describes how much and in which directions light is scattered when it collides with particles. In this lesson, we only gave values (and explained how to compute) scattering coefficients for the Earth's atmosphere. For atmospheric scattering, it is usually admitted that absorption is negligible. In other words, we will assume that atmosphere doesn't absorb light. Remember from the lesson on Volume Rendering that the **extinction coefficient** that we need in the physical model used to render a participating media is the sum of the absorption and scattering coefficients; therefore (since the absorption coefficient is zero), we can write for the extinction coefficient:

$$\beta_R^e = \beta_R^s$$

The Raleigh phase function looks like this:

$$P_R(\mu)=\frac{3}{16\pi}(1+\mu^2).$$

Where \(\mu\) is the cosine of the angle between the light and the view directions (see figure 8).

## Mie Scattering

Mie scattering is similar to the Rayleigh equation but applies to particles whose size is greater than the scattered wavelength. It is the case of aerosols which we find in the low altitudes of the Earth's atmosphere. If we were to apply the Rayleigh equation to aerosols, we would not get a convincing image. The Mie equation to render the scattering coefficients looks like this:

$$\beta_M^s(h,\lambda)=\beta_M^s(0,\lambda) e^{-\frac{h}{H_M}}$$

where the subscript \(M\) holds for Mie. Note that there is a specific scale height value \(H_M\) for the Mie scattering, which is usually set to 1.2 km. Unlike Rayleigh scattering, we don't need an equation to compute the Mie scattering coefficients. We will use values from measurements made at sea level instead. For Mie scattering we will use \(\beta_S = 210 \mathrm{e^{-5}} \mathrm{m^{-1}}\). Aerosols, too, have their density decreasing exponentially with altitude. As for the Rayleigh scattering, we will simulate this effect by including an exponential term on the right inside of the equation (modulated by the scale height factor \(H_M\)). Mie extinction coefficient is about 1.1 times the value of its scattering coefficient. And the Mie phase function equation is:

$$P_M(\mu)=\frac{3}{8\pi}\frac{(1-g^2)(1+\mu^2)}{(2+g^2)(1+g^2-2g\mu)^{\frac{3}{2}}}$$

The Mie phase function includes a term \(g\) (the Rayleigh phase function doesn't) which controls the anisotropy of the medium. Aerosols exhibit a strong forward directivity. Some papers use \(g\)=0.76 (which we will also be used as our default value).

## The Concept of Optical Depth

Before we look at a practical implementation of the algorithm in C++, we will put all the pieces of the puzzle together. First, the sky is nothing else than a spherically shaped volume surrounding a solid sphere. Because it is nothing more than a volume, to render the sky, we can use the ray-marching algorithm, which is the most common technique to render the participating medium. We studied that algorithm in detail in the lesson on Volume Rendering. When we render a volume, the observer (or the camera) can either be inside or outside.

![Figure 6: the camera can either be inside the atmosphere or outside. When it is inside, we are only interested in the intersection point between the camera ray and the atmosphere. When the camera is outside, it can intersect the atmosphere in two points.](/images/atmospheric%20scattering/as-camera.png?)

When the camera is inside, we must find where the viewing ray exits the volume. When it is outside, we need to find the points where the viewing ray enters and exits the volume. Because the sky is a sphere, we will use a ray-sphere intersection routine to compute these points analytically. We have presented a couple of techniques to compute the intersection of a ray and a sphere in the basic section (Ray-Quadratic Shapes Intersection). We know if the camera is inside or outside the atmosphere by testing its altitude using the ground as a reference. If this altitude is greater than the atmosphere thickness, the camera is outside the atmosphere, and the viewing ray might intersect the atmosphere in two places.

![Figure 7: single scattering is responsible for the sky color. A viewer rarely looks at the sun (which is dangerous). However, when we are looking away from the sun, the atmosphere has a color that is the result of blue light from the sunlight being scattered in the direction of the observer's eyes.](/images/atmospheric%20scattering/as-scattering.png?)

For now, we will assume that the camera is on the ground (or one meter above the ground), but the algorithm will work for any arbitrary camera position (we will render an image of the sky from outer space at the end of this lesson). Let's assume that we have a viewing ray (corresponding to one pixel in the frame) and that we know where this ray intersects with the upper limit of the atmosphere. The problem we need to solve at this point is to find out how much light is traveling along this ray in the viewer's direction. As we can see in the following figure, it is unlikely that our camera or observer will be directly looking at the sun (which is dangerous for your eyes).

![Figure 8: to compute the sky color, we first trace a ray from Pc to Pa and then integrate the amount of light coming from the sun (with direction L) that is reflected along that ray (V).](/images/atmospheric%20scattering/as-figure1.png?)

In all logic, if you were looking in the direction of the sky, you shouldn't see anything because you are looking at outer space, which is empty (space between celestial bodies). However, the fact is that you see a blue color. This is caused by light from the sun entering the atmosphere and deflected by air molecules in the viewing direction. In other words, there's no direct light coming from the sun traveling along the viewing direction (unless the viewing direction is pointing directly at the sun), but as the light coming from the sun is scattered by the atmosphere, some of that light ends up traveling in the direction of your eye. This phenomenon is called **single scattering** (explained in the lesson on Volume Rendering).

What do we know so far? We know that to render the sky color for one pixel in the frame, we will first cast a viewing ray (\(V\)) from the point \(Pc\) (the camera position) in the direction of interest. We will compute the point \(Pa\) where this ray intersects with the atmosphere. Finally, we need to compute the amount of light traveling along this ray due to single scattering. This value can be calculated with the volume rendering equation which we studied in the lesson on Volume Rendering:

$$L(P_c, P_a)=\int_{P_c}^{P_a} T(P_c,X)L_{sun} (X)ds$$

Where \(T\) is the transmittance between point \(Pc\) and \(X\) (the sample position along the viewing direction) and \(L\) is the amount of light in the volume at \(X\). All it says is that the total amount of light reaching the observer at \(Pc\) equals all the sunlight scattered along the viewing direction \(V\). This quantity can be obtained by summing up the light at various positions (\(X\)) along \(V\). This technique is called numerical integration. The more samples we take along the ray, the better the result, but the longer it takes to compute. The transmittance term (\(T\)) in the equation accounts for the fact that light scattered along \(V\) in the direction of the viewer at each sample position (\(X\)) is also attenuated as it travels from \(X\) to \(Pc\).

Recall from the lesson on volume rendering that light is attenuated as it travels from one point to another in the volume due to absorption and out-scattering. Let's call the amount of light we have at \(Pb\), \(Lb\), and the amount of light arriving at \(Pa\) from \(Pb\), \(La\). In the presence of a participating medium, \(La\) (light received from \(Pb\)) will be lower than \(Lb\). Transmittance represents the ratio \(La\) over \(Lb\), and \(T\) is therefore in the range zero to one. In equation form, transmittance looks like this:

$$T(P_a,P_b)=\frac{L_a}{L_b}=exp(-\sum_{P_a}^{P_b} \beta_e(h)ds)$$

In plain English, this equation means that we need to measure the extinction coefficients \(\beta_e\) of the atmosphere at various sample positions along the path ray Pa-Pb, sum these values up, multiply them by the length segment \(ds\) (that is the distance Pa-Pb divided by the number of samples used), take the negative of this value and feed it to an exponential function.

![Figure 9: as light (Lb) travels from Pb to Pa, it gets attenuated because of out-scattering and absorption. The result is La. Transmittance is the amount of light received at Pa from Pb after it was attenuated while traveling through the atmosphere (that is T=La/Lb).](/images/atmospheric%20scattering/as-transmittance.png?)

Going back to what we said about Rayleigh scattering, you will remember that \(\beta_s\) (from which we will compute \(\beta_e\)) can be calculated using the scattering coefficients at sea level modulated by the exponential of the altitude \(h\) over the scale height (\(H\)).

$$\beta_s(h) = \beta_s(0)exp(-\frac {h}{H})$$

For Rayleigh scattering, absorption can be ignored, and we can write:

$$\beta_e(h)=\beta_s(h)+\beta_a(h)=\beta_s(h)+0=\beta_s(h)$$

You can move \(\beta_e(0)\) out of the integral (because it is a constant), and you can re-write the transmittance equation as:

$$T(P_a, P_b)=exp(-\beta_e(0) \sum_{P_a}^{P_b} exp(-\frac{h}{H})ds)$$

The sum inside the exponential function can be seen as the average density value between \(Pa\) and \(Pb\). The equation itself is usually referred to in the literature as the **optical depth** of the atmosphere between points Pa-Pb.

## Adding the Sunlight

Finally, we will use the rendering equation presented in the previous paragraph to compute the sky color. Let's write it again:

$$L(P_c,P_a)=\int_{P_c}^{P_a} T(P_c, X)L_{sun}(X)ds$$

We have explained how to render the transmittance. Let's now look at the term Lsun(X) and explain how to evaluate it. Lsun(X) corresponds to the amount of sunlight scattered at the sample position \(X\) along the viewing direction. To evaluate it, first, we must compute the light arriving at \(X\). Taking the sunlight intensity directly is not enough. Indeed, if light enters the atmosphere at \(Ps\), it will also be attenuated as it travels to \(X\). We, therefore, need to compute this attenuation using a similar equation to the one we have been using to calculate the attenuation of light traveling along the viewing ray from \(Pa\) to \(Pc\) (equation 1):

$$L_{sun}(X)=Sun \: Intensity * T(X,P_s)$$

**Equation 1**

Technically for each sample position \(X\) along the viewing ray, we will need to cast a ray in the sunlight direction (\(L\)) and find where this light ray intersects with the atmosphere (\(Ps\)). We will then use the same numerical integration technique we used for the viewing ray to evaluate the transmittance (or optical depth) term in equation 1\. The light ray will be cut into segments, and the atmosphere's density at the center of each light segment will be evaluated.

One of the last elements in building an algorithm to compute the sky color is accounting for the amount of light scattered in the viewing direction based on the light direction and the view direction. This is the role of the phase function. Rayleigh and Mie scattering have their phase function, which we gave further up. If you need a refresher on the phase function, read the Volume Rendering lesson. In short, the phase function is a function that describes how much light coming from direction \(L\) is scattered in direction \(V\). The Mie phase function contains an additional term \(g\), called the **mean cosine** (among many other possible names), which defines if the light is mainly scattered along the forward direction (\(L\)) or the backward direction (\(-L\)). For forward scattering, \(g\) is in the range [0:1], and for backward scattering, \(g\) is in the range [-1:0]. When \(g\) equals zero, light is equally scattered in all directions, and we say that the scattering is isotropic. We will set \(g\) for Mie scattering to 0.76.

$$L_{sun}(X)=Sun \: Intensity*T(X,P_s)*P(V,L)$$

Finally, we must consider that for Rayleigh scattering, mainly blue light is scattered along the view direction. To reflect this, we will multiply the result of the previous equation by the scattering coefficients (\(\beta_s\)), which gives the whole equation for the light part of the equation. However, recall that \(\beta_s\) changes with the altitude (its value is a function of height) where \(h\) is the height of \(X\) about the ground (sea level more precisely):

$$
\begin{array}{l}
L_{sun}(X)=Sun \: Intensity*T(X,P_s)*P(V,L)*\beta_S(h)\\
L(X)=\int_{4\pi} Light \: Intensity * T(X, P_s)*P(V,L)*\beta_S(h)
\end{array}
$$

<details>
For the Earth's atmosphere, the sun is the only light source in the sky; however, to write a generic form of the previous equation, we should consider that light might come from many directions. In a scientific paper, you will usually see this equation as an integral over \(4\pi\), which describes a sphere of incoming directions. This more generic equation would also account for sunlight reflected by the ground (multiple scattering), which we will ignore in this chapter.
</details>

## Computing the Sky Color

Putting all the elements together, we have (equation 2):

$$
\begin{array}{l}
Sky \: Color(P_c, P_a)=\int_{P_c}^{P_a}T(P_c,X)L_{sun}(X)ds
\end{array}
$$

**Equation 2**

With (equation 3):

$$L_{sun}(X)=Sun \: Intensity * T(X,P_s)*P(V,L)*\beta_S(h)$$

**Equation 3**

If we replace 3 in 2, we get:

$$
\begin{array}{l}
Sky \: Color(P_c, P_a) = \\
\int_{P_c}^{P_a} T(P_c, X) * Sun \: Intensity * P(V,L) * T(X,P_s)*\beta_s(h)ds
\end{array}
$$

The result of the phase function is a constant, as well as the sunlight intensity. We can therefore move these two terms out of the integral (equation 4):

$$
\begin{array}{l}
Sky \: Color(P_c, P_a) = \\
Sun \: Intensity * P(V,L)\int_{P_c}^{P_a} T(P_c, X) * T(X,P_s)*\beta_s(h)ds
\end{array}
$$

**Equation 4**

This is the final equation for rendering the color of the sky for a particular view and light direction. As such, it's not so complicated. It's more computationally demanding since we compute an integral and do many calls to the exponential function (through the transmittance term). Plus, remember that the sky color results from Rayleigh and Mie scattering. Therefore we need to compute this equation twice for each type of scattering:

$$
\begin{array}{l}
Sky \: Color(P_c, P_a) = \\
Sky \: Color_{Rayleigh}(P_c,P_a) +Sky \: Color_{Mie}(P_c,P_a)
\end{array}
$$

Remember from calculus that the multiplication of two exponential functions is equal to one exponential function, which argument is the sum of the arguments from the first two functions:

$$e^ae^b=e^{a+b}$$

We can use this property to our advantage (we compute one exponential instead of two) and re-write the multiplication of the two transmittance terms in equation 4 with:

$$
\begin{array}{l}
T(P_c,X)=e^{ -\beta_{e0} } \\
T(X,P_s)=e^{ -\beta_{e1} } \\
T(P_c,X)*T(X,P_s)=e^{ -\beta_{e0} }*e^{ -\beta_{e1} }=e^{ -(\beta_{e0} + \beta_{e1}) }
\end{array}
$$

**Equation 5**

Let's now have a look at the implementation of the atmospheric model in a C++ program.

## Implementation (C++)

We now have all the elements needed to implement a version of Nishita's algorithm in a C++ program. As usual, the program will be written so the algorithm can easily be understood. Some optimizations will be suggested at this paragraph's end to make the program run faster. However, real-time is not the goal here; images of the sky can still be created with this program in a matter of seconds.

First, we will create an Atmosphere class that we will use to specify all the parameters of our system: the radius of the planet and the atmosphere (Re, Ra), the Rayleigh and Mie scattering coefficients at sea level, the Rayleigh and Mie scale height (Hr and Hm), the sun direction, the sun intensity, and the mean cosine. All distances are expressed in meters (as well as the scattering coefficients).

```
class Atmosphere 
{ 
public: 
    Atmosphere( 
        Vec3f sd = Vec3f(0, 1, 0), 
        float er = 6360e3, float ar = 6420e3, 
        float hr = 7994, float hm = 1200) : 
        sunDirection(sd), 
        earthRadius(er), 
        atmosphereRadius(ar), 
        Hr(hr), 
        Hm(hm) 
    {} 
 
    Vec3f computeIncidentLight(const Vec3f& orig, const Vec3f& dir, float tmin, float tmax) const; 
 
    Vec3f sunDirection;      //The sun direction (normalized) 
    float earthRadius;       //In the paper this is usually Rg or Re (radius ground, eart) 
    float atmosphereRadius;  //In the paper this is usually R or Ra (radius atmosphere) 
    float Hr;                //Thickness of the atmosphere if density was uniform (Hr) 
    float Hm;                //Same as above but for Mie scattering (Hm) 
 
    static const Vec3f betaR; 
    static const Vec3f betaM; 
}; 
 
const Vec3f Atmosphere::betaR(3.8e-6f, 13.5e-6f, 33.1e-6f); 
const Vec3f Atmosphere::betaM(21e-6f); 
```

We will render the sky as if it had been captured with a fisheye lens. The camera looks straight up and captures a 360 degrees view of the sky. To create an animation of the sky rendered for different positions of the sun, we will render a series of frames. In the first frame, the sun is at the zenith (center frame). In the last frame, the sun is slightly under the horizon.

```
void renderSkydome(const Vec3f& sunDir, const char *filename) 
{ 
    Atmosphere atmosphere(sunDir); 
    auto t0 = std::chrono::high_resolution_clock::now(); 
 
    const unsigned width = 512, height = 512; 
    Vec3f *image = new Vec3f[width * height], *p = image; 
    memset(image, 0x0, sizeof(Vec3f) * width * height); 
    for (unsigned j = 0; j < height; ++j) { 
        float y = 2.f * (j + 0.5f) / float(height - 1) - 1.f; 
        for (unsigned i = 0; i < width; ++i, ++p) { 
            float x = 2.f * (i + 0.5f) / float(width - 1) - 1.f; 
            float z2 = x * x + y * y; 
            if (z2 <= 1) { 
                float phi = std::atan2(y, x); 
                float theta = std::acos(1 - z2); 
                Vec3f dir(sin(theta) * cos(phi), cos(theta), sin(theta) * sin(phi)); 
                // 1 meter above sea level
                *p = atmosphere.computeIncidentLight(Vec3f(0, atmosphere.earthRadius + 1, 0), dir, 0, kInfinity); 
            } 
        } 
        fprintf(stderr, "\b\b\b\b\%3d%c", (int)(100 * j / (width - 1)), '%'); 
    } 
 
    std::cout << "\b\b\b\b" << ((std::chrono::duration<float>)(std::chrono::high_resolution_clock::now() - t0)).count() << " seconds" << std::endl; 
    // Save result to a PPM image (keep these flags if you compile under Windows)
    std::ofstream ofs(filename, std::ios::out | std::ios::binary); 
    ofs << "P6\n" << width << " " << height << "\n255\n"; 
    p = image; 
    for (unsigned j = 0; j < height; ++j) { 
        for (unsigned i = 0; i < width; ++i, ++p) { 
#if 1 
            // Apply tone mapping function
            (*p)[0] = (*p)[0] < 1.413f ? pow((*p)[0] * 0.38317f, 1.0f / 2.2f) : 1.0f - exp(-(*p)[0]); 
            (*p)[1] = (*p)[1] < 1.413f ? pow((*p)[1] * 0.38317f, 1.0f / 2.2f) : 1.0f - exp(-(*p)[1]); 
            (*p)[2] = (*p)[2] < 1.413f ? pow((*p)[2] * 0.38317f, 1.0f / 2.2f) : 1.0f - exp(-(*p)[2]); 
#endif 
            ofs << (unsigned char)(std::min(1.f, (*p)[0]) * 255) 
                << (unsigned char)(std::min(1.f, (*p)[1]) * 255) 
                << (unsigned char)(std::min(1.f, (*p)[2]) * 255); 
        } 
    } 
    ofs.close(); 
    delete[] image; 
} 
 
int main() 
{ 
#if 1 
    // Render a sequence of images (sunrise to sunset)
    unsigned nangles = 128; 
    for (unsigned i = 0; i < nangles; ++i) { 
        char filename[1024]; 
        sprintf(filename, "./skydome.%04d.ppm", i); 
        float angle = i / float(nangles - 1) * M_PI * 0.6; 
        fprintf(stderr, "Rendering image %d, angle = %0.2f\n", i, angle * 180 / M_PI); 
        renderSkydome(Vec3f(0, cos(angle), -sin(angle)), filename); 
    } 
#else 
    ... 
#endif 
 
    return 0; 
}
```

Finally, here is the function used to compute equation 4 (the color of the sky for a particular camera ray). The first thing we do is find the intersection point of the camera ray with the atmosphere (line 4). Then we compute the value of the Rayleigh and Mie phase function (using the sun and camera ray direction. Lines 14 and 16). The first loop (line 17) creates samples along the camera ray. Note that the sample position (X in the equations) is the segment middle point (line 19). From there, we can compute the sample (X) height (line 19). We calculate `exp(-h/H)` multiplied by `ds` for Rayleigh and Mie scattering (using Hr and Hm). These values are accumulated (lines 23 and 24) to compute the optical depth at `X`. We will also use them later to scale the scattering coefficients \(\beta_s(h)\) in equation 4 (lines 42 and 43). Then we compute the light coming from the sun at X (lines 31 to 38). We cast a ray in the direction of the sun (rays are parallel) and find the intersection with the atmosphere. This ray is cut into segments, and we evaluate the density in the middle of the segment (lines 35 and 36). Accumulating these values gives us the optical depth of the light ray. Note that we test if each light sample is above or below the ground. If it is below the ground, the light ray is in the shadow of the earth; we can then safely discard the contribution of this ray (lines 34 and 39). Note that the Mie extinction coefficient is about 1.1 times the value of the Mie scattering coefficient (line 40). Finally, using the trick from equation 5, we can compute the accumulated transmission of the light and the camera ray by accumulating their optical depth in one single exponential call (lines 40 and 41). At the end of the function, we return the final color of the sky for this particular ray, which is the sum of the Rayleigh and Mie scattering transmittance multiplied by their respective phase function and scattering coefficients. This sum is also multiplied by the sun's intensity (line 48).

<details>
At this point, the sun's intensity is just a magic number (we used the value 20). But in a future revision of this lesson, we will learn how to use actual physical data).
</details>

```
Vec3f Atmosphere::computeIncidentLight(const Vec3f& orig, const Vec3f& dir, float tmin, float tmax) const 
{ 
    float t0, t1; 
    if (!raySphereIntersect(orig, dir, atmosphereRadius, t0, t1) || t1 < 0) return 0; 
    if (t0 > tmin && t0 > 0) tmin = t0; 
    if (t1 < tmax) tmax = t1; 
    uint32_t numSamples = 16; 
    uint32_t numSamplesLight = 8; 
    float segmentLength = (tmax - tmin) / numSamples; 
    float tCurrent = tmin; 
    Vec3f sumR(0), sumM(0);  //mie and rayleigh contribution 
    float opticalDepthR = 0, opticalDepthM = 0; 
    float mu = dot(dir, sunDirection);  //mu in the paper which is the cosine of the angle between the sun direction and the ray direction 
    float phaseR = 3.f / (16.f * M_PI) * (1 + mu * mu); 
    float g = 0.76f; 
    float phaseM = 3.f / (8.f * M_PI) * ((1.f - g * g) * (1.f + mu * mu)) / ((2.f + g * g) * pow(1.f + g * g - 2.f * g * mu, 1.5f)); 
    for (uint32_t i = 0; i < numSamples; ++i) { 
        Vec3f samplePosition = orig + (tCurrent + segmentLength * 0.5f) * dir; 
        float height = samplePosition.length() - earthRadius; 
        // compute optical depth for light
        float hr = exp(-height / Hr) * segmentLength; 
        float hm = exp(-height / Hm) * segmentLength; 
        opticalDepthR += hr; 
        opticalDepthM += hm; 
        // light optical depth
        float t0Light, t1Light; 
        raySphereIntersect(samplePosition, sunDirection, atmosphereRadius, t0Light, t1Light); 
        float segmentLengthLight = t1Light / numSamplesLight, tCurrentLight = 0; 
        float opticalDepthLightR = 0, opticalDepthLightM = 0; 
        uint32_t j; 
        for (j = 0; j < numSamplesLight; ++j) { 
            Vec3f samplePositionLight = samplePosition + (tCurrentLight + segmentLengthLight * 0.5f) * sunDirection; 
            float heightLight = samplePositionLight.length() - earthRadius; 
            if (heightLight < 0) break; 
            opticalDepthLightR += exp(-heightLight / Hr) * segmentLengthLight; 
            opticalDepthLightM += exp(-heightLight / Hm) * segmentLengthLight; 
            tCurrentLight += segmentLengthLight; 
        } 
        if (j == numSamplesLight) { 
            Vec3f tau = betaR * (opticalDepthR + opticalDepthLightR) + betaM * 1.1f * (opticalDepthM + opticalDepthLightM); 
            Vec3f attenuation(exp(-tau.x), exp(-tau.y), exp(-tau.z)); 
            sumR += attenuation * hr; 
            sumM += attenuation * hm; 
        } 
        tCurrent += segmentLength; 
    } 
 
    // We use a magic number here for the intensity of the sun (20). We will make it more
    // scientific in a future revision of this lesson/code
    return (sumR * betaR * phaseR + sumM * betaM * phaseM) * 20; 
}
```

![](/images/atmospheric%20scattering/as-anim.gif)

Each frame only takes a few seconds to compute on a 2.5GHz processor (this time depends on the number of views and light samples you use). A few features can be added to this program. Some papers run a tone mapping on the resulting image before saving it. This can help reduce the contrast between the brightness of the sky around the sun (very bright and white) and the rest of the sky. In addition, you can save the resulting image to a floating point image format (HDR, EXR) as values can easily be greater than one depending on your sun intensity (remapping and clipping the values is not a great choice). This function can easily be added to any existing renderer (see the paragraph on Aerial Perspective for more details about this). In the images, the color of the sky turns red-orange when the sun is above or slightly below the horizon. You can also comment on the contribution of the Mie scattering to see the contribution of the Rayleigh scattering independently (and vice versa). However, by just looking at the resulting images and remembering what you have learned about these two scattering models, you can easily guess that Mie scattering is responsible for the main white halo around the sun and the brightening/desaturation of the sky at the horizon. In contrast, Rayleigh scattering is responsible for the blue/red/orange colors of the sky.

![](/images/atmospheric%20scattering/as-comp.png?)

An interesting project could consist of passing a date and time to the program, then computing the sun's position based on this data, rendering a frame with this sun's direction, and comparing the result to a photograph of the real sky taken at the same time on the same day. How closely would they match? The sun's position is related to its declination and the solar hour angle (in other words, the time of the day).

<details>
![](/images/atmospheric%20scattering/as-symmetry.png?)

Optimization: Nishita observed that the sky is symmetrical about the axis defined by the sun's position and the center of the Earth. Based on this observation, the paper proposes a technique to bake the optical depth in a 2D table which you can access using \(\mu\) (the cosine of the angle between the view and the light direction) and h (the height of the sample point). It would be best if you created a table for each scattering model (Rayleigh and Mie). These tables can be computed beforehand, saved into a file, and reused each time the program is run. Using precomputed values for the optical depth rather than computing it on the fly (which requires a few expensive exponential calls) would speed the render considerably. If speed is essential for your application, this optimization is the first one you could implement (see Nishita's paper for details).
</details>

## Light Shafts

![](/images/atmospheric%20scattering/as-lightshafts.png?)

![Figure 10: light shafts appear when objects in the scene shadow some regions of the volume. In this example, some areas of the atmosphere are shadowed by the mountains.](/images/atmospheric%20scattering/as-lightshafts2.png?)

Atmospheric effects can sometimes create spectacular visual effects. Light shafts usually make these effects more dramatic. If light traveled freely through the volume, the volume would appear uniformly lit. But if we were to place some objects in the scene, areas of the volumes shadowed by these objects would appear darker than areas that are not occluded. Light shafts are beams of light corresponding to regions of the volume illuminated by a light source (the sun). But they are only visible because regions of this volume are shadowed by objects in the scene (in the case of the Earth's atmosphere, mainly mountains and clouds).

## Simulating Aerial Perspective

Computing aerial perspective doesn't require many changes to our code. As you can see in the following figure, the color of the mountain is affected by the presence of the blue atmosphere. Because the distance from the eye to the ground is shorter for ray A than for B, the contribution of the atmosphere is less noticeable in ray A's color than in ray B's color.

![](/images/atmospheric%20scattering/as-aerialpersp.png?)

First, you should render the color of the geometry (the terrain) for the ray. Then it would be best if you rendered the transmittance (aka the opacity) and the color of the atmosphere (using the eye position and the intersection point of the ray with the geometry for Pc and Pa) and composite the two colors together using the following alpha blending formula:

```
Ray Color = Object Color * (1 - Transmittance) + Atmosphere Color
```

![](/images/atmospheric%20scattering/as-aerial1.png?)

![](/images/atmospheric%20scattering/as-aerial2.png?)

In this lesson, though, to keep the code simple, we won't be rendering the ground color, which will be black by default. If you wish to understand in detail how this compositing operation can be done, check the lesson on volume rendering. Here are a couple of results we got with two different positions for the sun. The camera is so far away that the Earth is visible in the bottom half of the frame. We first find if the primary ray intersects with the Earth. If it does, we then compute the color of the atmosphere from the eye position to this intersection point.

These images can be computed with a very basic ray tracer. If the camera ray hits the Earth, we must update the ray `tmin` and `max` variables (lines 36-39) before we call the function that computes the atmosphere color.

```
void renderSkydome(const Vec3f& sunDir, const char *filename) 
{ 
    Atmosphere atmosphere(sunDir); 
    ... 
#if 1 
    // Render fisheye
    ... 
#else 
    // Render from a normal camera
    const unsigned width = 640, height = 480; 
    Vec3f *image = new Vec3f[width * height], *p = image; 
    memset(image, 0x0, sizeof(Vec3f) * width * height); 
    float aspectRatio = width / float(height); 
    float fov = 65; 
    float angle = std::tan(fov * M_PI / 180 * 0.5f); 
    unsigned numPixelSamples = 4; 
    Vec3f orig(0, atmosphere.earthRadius + 1000, 30000);  //camera position 
    std::default_random_engine generator; 
    std::uniform_real_distribution<float> distribution(0, 1);  //to generate random floats in the range [0:1] 
    for (unsigned y = 0; y < height; ++y) { 
        for (unsigned x = 0; x < width; ++x, ++p) { 
            for (unsigned m = 0; m < numPixelSamples; ++m) { 
                for (unsigned n = 0; n < numPixelSamples; ++n) { 
                    float rayx = (2 * (x + (m + distribution(generator)) / numPixelSamples) / float(width) - 1) * aspectRatio * angle; 
                    float rayy = (1 - (y + (n + distribution(generator)) / numPixelSamples) / float(height) * 2) * angle; 
                    Vec3f dir(rayx, rayy, -1); 
                    normalize(dir); 
                    // Does the ray intersect the planetory body? (the intersection test is against the Earth here
                    // not against the atmosphere). If the ray intersects the Earth body and that the intersection
                    // is ahead of us, then the ray intersects the planet in 2 points, t0 and t1. But we
                    // only want to comupute the atmosphere between t=0 and t=t0 (where the ray hits
                    // the Earth first). If the viewing ray doesn't hit the Earth, or course, the ray
                    // is then bounded to the range [0:INF]. In the method computeIncidentLight() we then
                    // compute where this primary ray intersects the atmosphere, and we limit the max t range 
                    // of the ray to the point where it leaves the atmosphere.
                    float t0, t1, tMax = kInfinity; 
                    if (raySphereIntersect(orig, dir, atmosphere.earthRadius, t0, t1) && t1 > 0) 
                        tMax = std::max(0.f, t0); 
                    // The *viewing or camera ray* is bounded to the range [0:tMax]
                    *p += atmosphere.computeIncidentLight(orig, dir, 0, tMax); 
                } 
            } 
            *p *= 1.f / (numPixelSamples * numPixelSamples); 
        } 
        fprintf(stderr, "\b\b\b\b%3d%c", (int)(100 * y / (width - 1)), '%'); 
    } 
#endif 
    ... 
}
```

## Alien Skies

By changing the parameters of the Atmosphere model, it's easy to create skies that have very different looks from our Earthly sky. For instance, one could imagine re-creating the planet Mars's atmosphere or creating an imaginary sky. The scattering coefficients and the thickness of the atmosphere are the most apparent parameters you can play with to change its look. Increasing the contribution of Mie scattering quickly turns the atmosphere into a very foggy-hazy sky, combined with light shafts, which can create expressive/moody images.

## What about Multiple Scattering?

The color of the sky is the result of light being scattered once or multiple times in the atmosphere toward the viewer. However, in the literature, it is emphasized that single scattering predominates. Therefore rendering images of the sky by ignoring multiple scattering still gives very plausible results. Most models in the literature ignore or do not provide a technique to take multiple scattering into account. Bruneton suggests that the amount of light reflected by the ground is large enough to influence the sky color. He proposes a model in which light scattered by the ground is considered (in this model, Earth is assumed to be a perfectly spherical shape).

## Conclusion

The primary few ideas you need to remember from this chapter is that the sky can be rendered as a massive volume in the shape of a sphere (surrounding another larger sphere representing the Earth). Absorption and scattering coefficients and a phase function can define any volume (as learned in the lesson on Volume Rendering). The sky's appearance is the combination of Rayleigh and Mie scattering. Rayleigh scattering is responsible for the blue color of the sky (and its red-orange color at sunrise and sunset) and is caused by light being scattered by air molecules whose sizes are much smaller than the light wavelength. Air molecules scatter blue light more than green and blue light. Mie scattering is responsible for the hazy whitish look of the atmosphere. It is caused by sunlight scattered by molecules more significant than the light wavelength. These molecules are known as aerosols. Aerosols scatter light of all wavelengths equally.

## Ideas of Improvements/Exercises

If you are looking to improve/extend the code, here is a list of ideas:

- Render the surface of the Earth (land and ocean) using procedural texturing or mip-mapping.
- Add clouds to the sky model.
- Precompute the optical depth between the sun and an arbitrary point along the ray (use this technique to speed up the program - for more details on the method, check Nishita's paper).
- Add multiple scattering.
- Compare this model with other Sky models (check the paper "A Framework for the Experimental Comparison of Solar and Skydive Illumination").

## References

_Display of The Earth Taking into Account Atmospheric Scattering_, Nishita et al., Siggraph 1993.

_Display Method of the Sky Color Taking into Account Multiple Scattering_, Nishita et al., Siggraph 1996.

_Precomputed Atmospheric Scattering_, Eric Bruneton, and Fabrice Neyret, Eurographics 2008.

_A Practical Analytic Model for Daylight_, A. J. Preetham, et al. Siggraph 1999.

_A Framework for the Experimental Comparison of Solar and Skydive Illumination_, Joseph T. Kider Jr et al., Cornell University 2014.