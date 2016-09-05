---
layout: post
title: "GSoC 2013: Mid-term results"
date: 2013-08-19 16:24
comments: true
tags: gsoc
keywords: "google summer of code, mid-term, scikit-image"
---

So, I'm back on campus now. College started from 1st August. I'm pretty happy, this is the last semester on-campus and I just have 3 electives, that's about 10 hours of course-work weekly. I thought I would have been able to peacefully allot about 6-8 hrs daily to [GSoC](http://code.google.com/soc/), but the registrations, placement seminars and work in the Department I co-founded on campus last semester [http://cte.bits-goa.com](http://cte.bits-goa.com), kept me pretty busy. I'm finally done with them and I can now get back to GSoC work in full swing (need to allot 8 hrs now to cope up).

First things first, on 2nd August the results for GSoC mid-term were declared and fortunately I have passed! Thanks a lot Stefan and Tony. 

![Thanks to "mammy" on http://www.create-a-kit.com.au](http://www.create-a-kit.com.au/images/products/happy_party_time.gif)

### UPDATES

Just a quick update about the progress of my project:

1. Work on **[Fast Marching Method implementation](http://en.wikipedia.org/wiki/Fast_marching_method)** ([PR #663](https://github.com/scikit-image/scikit-image/pull/663)) is over and now I'm waiting for my mentor to merge it. (He's out of town at the moment, but hopefully we should have it merged by this weekend) With this I would have **completed my first module on Structural Inpainting**, out of the three modules I had initially proposed in my [project](www.google-melange.com/gsoc/proposal/review/google/gsoc2013/chintak/1). As soon as it is merged I will be following up with another blog post which will be detailing about how you can use it for your purpose. I've tried to add enough documentations, so as to make it exhaustive for people accustomed to [NumPy](http://www.numpy.org/) and scikit-image to use, but in case if you are beginner in either, then the post is targeted to you.

2. I was working simultaneously on the FMM PR review and also implementation of my next module on [Texture Synthesis](http://en.wikipedia.org/wiki/Texture_synthesis). **Efros and Leung's algorithm** has proved to be state-of-the-art algorithm for Texture Synthesis. I thought it would be nice to include something like this in scikit-image library. In the last week, I completed this implementation ([PR #670](https://github.com/scikit-image/scikit-image/pull/670)) and am **awaiting review** from the mentors and the community in general. In face, much of the last two post on [NumPy tricks](/2013/07/numpy-the-tricks-of-the-trade-part-ii/) was what I had learned during this implementation. This produces better results as compared to FMM, however is slower (by an order of magnitude). My next post will be outlining the Efros' algorithm which I have implemented.

3. I am **currently working** on a variant of Efros' algorithm, **Criminisi et al. algorithm** which makes it more efficient and capable for handling linear structures along with texture synthesis. If I am able to achieve the results as stated by their paper, I think I may not have to implement my 3rd module altogether! Since this algorithm would anyway provide the results of combined structural inpainting and textural synthesis on a giving image which is what my third module sought to do. However, I don't want to jump the gun as of now, since as it turns out this implementation is pretty tricky and has its own subtleties which can give you plausible results yet not correct.

So once again wrapping up, I have *completed work on FMM PR*, *awaiting review on Texture Synthesis PR* and **currently working on the amazing Criminisi's** algorithm.

### REFERENCES

- A. Efros and T. Leung. *Texture synthesis by non-parametric sampling*. In Proc. Int. Conf. Computer Vision, pages 1033–1038, Kerkyra, Greece, September 1999.
- Criminisi, Antonio; Perez, P.; Toyama, K., "Region filling and object removal by exemplar-based image inpainting," *Image Processing, IEEE Transactions on* , vol.13, no.9, pp.1200,1212, Sept. 2004. doi: 10.1109/TIP.2004.833105.
