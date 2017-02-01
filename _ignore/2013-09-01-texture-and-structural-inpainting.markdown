---
layout: post
title: "Texture &amp; Structural Inpainting - Criminisi's Exemplar Method"
date: 2013-09-01 16:01
comments: true
tags: [algorithm, gsoc, image-processing, inpainting]
keywords: "google summer of code, texture synthesis, filling in, criminisi, exemplar based, efros leung, inpaint"
---
![Inpainting using Exemplar Based Method. Notice no "blurring" effect and perfect edge reconstruction.](/images/posts/criminisi_res/checkerboard.jpg)

Today I will be writing about Criminisi's algorithm as outlined in "Region Filling and Object Removal by Exemplar-Based Image Inpainting" paper. They propose two main improvements to Efros and Leung's algorithm (in case you don't know about Efros' Texture Synthesis algorithm check the [previous post]({% post_url 2013-08-31-texture-synthesis-efros-and-leung-algo %})).

### ALGORITHM

The changes are as follows:

1. An ordering scheme to determine which pixel out of the outermost boundary pixels should be inpainted first. For doing this consider, a template of shape `(window, window)` surrounding any boundary pixel **`p`**. And let **`q`** refer to a pixel within this template. We compute the following two terms:

    - **Confidence** term: Amount of reliable information in the surrounding region of the pixel in question. Already known pixels have a confidence of `1` and the confidence of term for any unknown pixel is computed according to the equation below. (`(window, window)` is the size of the template considered about the boundary pixel)

    $$
    C(p) = \dfrac{\Sigma_{\forall~q~\in~(template~\bigcap~known)} C(q)}{window^2}
    $$

    - **Data** term: This term represents the amount of image intensity change in the direction of the boundary in a given template about the boundary pixel. That is, we first select the max gradient in the known region of the template as the gradient at the boundary pixel. Take a dot product between the "gradient" rotated by 90 degree and the normal unit vector to the boundary.

    $$
    D(p) = \dfrac{|\nabla I_{p}^{\perp} \cdot n_p|} {|\nabla I_{p}^{\perp}||n_p|}
    $$

    **Priority** term: $$ P(p) = C(p) \times D(p) $$ We use this term to decide the order in which the pixels are to be inpainted.

2. **Patch based** inpainting instead of updating pixel-by-pixel. This provides in a considerable speed gain as compared ot the Efros' algorithm. Due to the ordering scheme the patch based updating method does not intrduce any false or incorrect features into the region, which is the reason why we don't modify Efros' algorithm directly to update patches instead of pixels at a time.

So a quick recap of the complete algorithm is as follows:

- Loop: Generate the boundary pixels of the region to be inpainted
    - Loop: Compute the priority of each pixel
        - Generate a template of `(window, window)`, center: boundary pixel
        - confidence_term: avg amount of reliable information in template
        - data_term: strength of the isophote hitting this boundary pixel
        - ``priority = data_term * confidence_term``
    - Repeat for all boundary pixels and chose the pixel with max priority
    - Template matching of the pixel with max priority
        - Generate a template of (window, window) around this pixel
        - Compute the Sum of Squared Difference (SSD) between template and
          similar sized patches across the image
        - Find the pixel with smallest SSD, such that patch isn't where
          template is located (False positive)
        - Update the intensity value of the unknown region of template as
          the corresponding value from matched patch
- Repeat until all pixels are inpainted

### RESULTS

The inpainting performed by this algorithm provides a much better result as compared to [Efros' algorithm](/2013/08/texture-synthesis-efros-and-leung-algo/) and also [Fast Marching Method](/2013/07/fast-marching-method/).

![Inpainting using Exemplar Based Method. Notice that the reconstruction looks more natural in this case.](/images/posts/criminisi_res/wall.jpg)

![Inpainting using Exemplar Based Method. It is able to reconstruct the central crossing correctly without any distortion as observed in <a href=/images/posts/efros_results/brick.jpg>Efros' algorithm Result</a>.](/images/posts/criminisi_res/brick.jpg)

![Inpainting using Exemplar Based Method. It is able to reconstruct the straight lines correctly.](/images/posts/criminisi_res/brick2.jpg)

### CODE

The implementation for this algorithm can be found here: [PR #706](https://github.com/scikit-image/scikit-image/pull/706). In case if it is already merged, kindly check out the [`skimage.filter`](https://github.com/scikit-image/scikit-image/tree/master/skimage/filter) module.

### REFERENCES

- Criminisi, Antonio; Perez, P.; Toyama, K., “*Region filling and object removal by exemplar-based image inpainting*”, Image Processing, IEEE Transactions on , vol.13, no.9, pp.1200,1212, Sept. 2004. doi: 10.1109/TIP.2004.833105.
- A. Efros and T. Leung. "*Texture Synthesis by Non-Parametric Sampling*". In Proceedings of International Conference on Computer Vision, pages 1033-1038, Kerkyra, Greece, September 1999.
  [http://graphics.cs.cmu.edu/people/efros/research/EfrosLeung.html](http://graphics.cs.cmu.edu/people/efros/research/EfrosLeung.html)
