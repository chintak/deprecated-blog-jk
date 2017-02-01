---
layout: post
title: "Texture Synthesis - Efros and Leung's Algo"
date: 2013-08-31 10:18
comments: true
tags: [algorithm, gsoc, image-processing, inpainting]
keywords: "google summer of code, texture synthesis, filling in, efros, leung, non parametric, inpaint"
---

![Inpainting using Texture Synthesis. 1. Input image, black portion is to be reconstructed. 2. Reconstructed image, notice no "blurring" effect.](/images/posts/efros_results/color.jpg)

Today I will throw some light on Texture Synthesis. Last time I had posted the outline of [Fast Marching Method](/2013/07/fast-marching-method/) which is essentially a method for structural inpainting, i.e. it can reconstruct solid objects but not so good at reconstructing textures which inherently have a repeating pattern. Efros and Leung's algorithm tries to perform texture synthesis and does a fairly good job at reconstructing structures.

### ALGORITHM

The algorithm is fairly straightforward and pretty intuitive to understand. It goes as follows:

- Given a binary `mask` with `True` values representing the pixels to be inpainted (we'll call this the region of interest or ROI).
- Iterative: Generate the boundary pixels of the ROI - we can do this by using morphological operator erosion and subtracting this from the original mask.
    - For each boundary pixel, generate a template of shape `(window, window)` with the centre at the boudnary pixel. (Perform **Template Matching** with Sum of Squared Difference as the metric)
        - Compute the Sum of Squared Difference between the template and all similarly sized patches in the image (*Here we neglest the padded region and all patches which have a pixel in the unknown (ROI) region*).
        - Find the centre pixel of the patch which gives the smallest SSD.
        - Update the intensity value of the boundary pixel with the value found above.
    - Repeat for all boundary pixels.
- Repeat untill all pixels are inpainted.

### RESULTS

Fast Marching Method and other diffusion based methods suffer from blurring. Whereas, with this algorithm no blurring is observed. However, since template matching with all the patches in the image is an expensive operation the speed is relatively very slow as compared to Fast Marching Method.

![Texture Synthesis of a rough wall. The reconstructed portion "blends" into the backgroung and hence gives a plausible result.](/images/posts/efros_results/wall.jpg)

![Texture Synthesis of a brick structure. If we observe carefully, in the right reconstructed portion, the "+" is not perfectly reconstructed. This could be attributed to the fact that in this algorithm we update pixel by pixel.](/images/posts/efros_results/brick.jpg)

### NOTE

There this has mainly three drawbacks:

- The algorithm follows "**onion peel**" ordering, in the sense we go from outermost portion to the innermost. In case of repeating texture patterns ot does a pretty descent job but may not so much for structures.
- In this algorithm we update pixel-by-pixel. Due to this we may **not observe perfect structural reconstruction** with this algorithm, as noted in the image above.
- The computation is painstakingly **slow**. For example, a 500 pixel block takes upto 30 seconds to execute whereas this same block takes about 0.06 seconds (results for texture synthesis are however much better for Efros' algorithm).

### CODE

I have completed with this implementation and you can find the code here: [PR #670](https://github.com/scikit-image/scikit-image/pull/670). Or in case if this is merged, then you can find it in the [`skimage.filter`](https://github.com/scikit-image/scikit-image/tree/master/skimage/filter) module.

### UP NEXT

One of my mentor [Emmanuelle Gouillart](https://github.com/emmanuelle) suggested narrowing the SSD computation to the 10 nearest mean and variance matches of query template. Over the next few days, I'll try and see if we can use a dictionary to index the mean and variance of all the Valid sample patches in the image. Once we have the top 10 matches, we compute their SSD and choose the sample patch with the least SSD. This should help us speed up the implementation quite a bit. I'll post my findings once I'm done.

In my next post I will be describing about my third module implementation, i.e. Criminisi et al. algorithm which is mainly an improvement to Efros' algorithm. They provide add an ordering scheme for inpainting pixels with more information surrounding them or with edges hitting them directly. The algorithm updates patch by patch which enhances the speed considerably. Hence, is able to to provide better results for both texture as well as structural portions.

### REFERENCES

- A. Efros and T. Leung. "*Texture Synthesis by Non-Parametric Sampling*". In Proceedings of International Conference on Computer Vision, pages 1033-1038, Kerkyra, Greece, September 1999. http://graphics.cs.cmu.edu/people/efros/research/EfrosLeung.html
- Criminisi, Antonio; Perez, P.; Toyama, K., “*Region filling and object removal by exemplar-based image inpainting*”, Image Processing, IEEE Transactions on , vol.13, no.9, pp.1200,1212, Sept. 2004. doi: 10.1109/TIP.2004.833105.