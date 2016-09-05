---
layout: post
title: "Fast Marching Method"
date: 2013-07-09 14:27
comments: true
tags: [gsoc, inpainting, image-processing, algorithm]
keywords: "google summer of code, fast marching method, python, eikonal, inpaint, structural inpaint"
---
![Image Inpainting using FMM, as in Telea's Paper](/images/posts/inpaint.jpg "Image Inpainting using FMM, as in Telea's Paper")

Today I will throw some light on the different components in my [GSoC project](www.google-melange.com/gsoc/proposal/review/google/gsoc2013/chintak/1) and describe the algorithm for the first component. The project is divided into 3 parts – Image Decomposition, Structural Inpainting and Textural Synthesis. Traditionally, the algorithms used for inpainting are either intended for structural reconstruction or textural synthesis. Recently, new techniques are being developed which divide the image into structural and textural components. Independently apply different algorithms on each and finally merge the image. My mentor [Tony S. Yu](http://www.tonysyu.com/) advised me to start with Structural Inpainting so that there will be something workable for the community sooner. In the time to come, I will describe the algorithms regarding Image Decomposition and Textural Synthesis. This post outlines the algorithm for Structural Inpainting based on Fast Marching Method.

The following 3 papers are imperative to read for performing Image Inpainting using the Fast Marching Method:

- Telea, A., *An Image Inpainting Technique based on the Fast Marching Method*. Journal of Graphic Tools (2004).
- J. A. Sethian. *Level Set Methods and Fast Marching Methods*. Second edition. Cambridge, UK: Cambridge Univ. Press, 1999.
- J. A. Sethian. *A Fast Marching Level Set Method for Monotonically Advancing Fronts*. Proc. Nat. Acad. Sci. 93:4 (1996), 1591—1595.

The first paper by Telea is pretty exhaustive but in order to get more insight into Fast Marching Method, I read the other two papers as well. I’ll explain next in simple terms what this FMM is all about and what does it really do.

### WHAT IS IT?

FMM is used to track the evolution of a boundary (a curve in 2D or surface in 3D) moving in a direction normal to itself. In general, FMM is used in variety of field like arrival time problems in Control Theory, lithographic development calculations in Microchip manufacturing, etc. The point important to remember here is that, the direction of motion at every point in grid is taken to be normal to the boundary through it. We are not concerned with the tangential component of speed. (Velocity would be more apt to describe it)

### MATHEMATICALLY

Imagine you have a boundary, a curve in 2D or surface in 3D which divides the region into two parts. We are interested to find out how the boundary evolves as it propagates normal to itself with speed F. The time at which the boundary crosses a particular point in the region is denoted by T. At $ T = 0 $, the boundary is in its initial position. This is an initial value problem where our aim is to find how the boundary evolves over time. The order in which pixels are to be traversed, depending on the speed and direction of adjacent pixels are obtained by solving the [Eikonal equation](http://en.wikipedia.org/wiki/Eikonal_equation):

$$
F|\nabla T| = 1
$$

In the case of images, we assume that all the pixels have equal speed and set
it to 1, i.e. `F = 1`. Also numerically,

$$
|\nabla T| ≈ \mbox{max}( D^{-x} T , -D^{+x} T , 0 )^2 + \mbox{max}( D^{-y} T , -D^{+y} T , 0 )^2
$$

where, $ D^{-x} T( i , j ) = T( i , j ) - T( i-1 , j ) $ and $ D^{+x} T( i , j ) = T( i+1 , j ) - T( i , j ) $. Here, $ D^- $ stands for backward difference operator and $ D^+ $ stands for the forward difference operator.

The key to solve this is in observing that the value of $ T $ satisfying this is the smallest eikonal solution in the 4 quadrants of $ ( i , j ) $.

![Initial representation of FMM](/images/posts/fmm1.jpg)

### NOTES

- FMM always maintains a thin band between the known and unknown regions. This
  is crucial for passing intensity information from the neighborhood to the pixel being [inpainted](http://en.wikipedia.org/wiki/Inpainting).
- The order of traversal is from outside to the interior of the region to be in
  painted.

### ALGORITHM

Telea has adapted FMM to work with images for inpainting as outlined in his paper cited above. In the case of images we assume that all the pixels travel with the same speed = 1 and in **direction normal to the curve** through them. FMM computes the **distance/time map** which corresponds to the time taken by the boundary to reach the pixel. Some terminology that will be used below.

- `u` : Distance/time taken by the boundary to reach this pixel
- `flag` : Status of the pixel, either KNOWN, BAND or INSIDE
- KNOWN : Pixels whose u and intensity values are known
- BAND : Their u values are updated
- INSIDE : Intensity and u values unknown. Region to be inpainted.

The steps of the algorithm are as follows:

- Extract the pixel with the smallest `u` value in the BAND pixels
- Update its `flag` value as KNOWN
- March the boundary inwards by adding new points.
    - If they are either INSIDE or BAND, compute its `u` value using the `eikonal` function for all the 4 quadrants
    - If `flag` is INSIDE
        - Change it to BAND
        - Inpaint the pixel
    - Select the value and assign it as the `u` value of the pixel
    - Insert/update this new value in the heap

![Fast Marching Method output. Blue -> Red : increasing order of value. The transition of Blue to Red shows the increasing distance in the distance map. This shows the boundary propagation using FMM.](/images/posts/dist-time-map.jpg)

### INPAINT

This is a rather straight forward implementation which is based on the discrete gradient approximation: $ I(p) = I(q) + \nabla I(q) * (p - q) $.

The above value is summed over all pixels in the neighborhood of the pixel to be inpainted whose intensity values are known. A weight function is also used in order to pass the gradient values correctly. The design of the weight function comprises of 3 components:

- The directional component `dir(p, q)` ensures that the contribution of the pixels close to the normal direction $ N = \nabla T $
- The geometric distance component `dst(p, q)` decreases the contribution of the pixels geometrically farther from p.
- The level set distance component `lev(p, q)` ensures that pixels close to the contour through p contribute more than farther pixels.

### CODE

You can check out my [inpaint](https://github.com/chintak/scikit-image/tree/inpaint/skimage/inpaint) branch. I aim to submit a PR for this by the coming Sunday. Still couple of bug fixes remaining and unit tests and examples to be written. In my next post I’ll discuss my final code and explain all that you need to know to use it effectively!
