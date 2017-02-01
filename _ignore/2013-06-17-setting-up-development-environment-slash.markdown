---
layout: post
title: "Setting up Development Environment!"
date: 2013-06-17 14:14
comments: true
keywords: "setup, scikit-image, numpy, cython, scipy, matplotlib, mac osx, Sublime Text 2"
categories:
- scikit-image
- setup
- numpy
- cython
- scipy
---

It’s been quite a while since I last posted. I am extremely extremely happy to
be one of the selected students to do GSoC this year. A lot of credit goes to
the scikit-image community, Tony Yu (my GSoC mentor) and Stefan for being
patient with me and reviewing my proposal. Once again, thanks a lot !! :D  I
will be working on the scikit-image: Image Inpainting for Image Restoration. I
hope that by the end of the summers I can provide scikit-image with structural
and textural inpainting functions. I encourage folks interested in Image
Processing to check out scikit-image and preferably start contributing on
[GitHub](https://github.com/scikit-image/scikit-image). The code is very well
documented hence, easy to understand and kick start development.

This is just an introductory post in which I am briefly going to describe the procedure for setting up scikit-image.

### MINIMUM DEPENDENCIES

- [Python](http://www.python.org/download/): version 2.5 or more.
- [NumPy](http://numpy.scipy.org/): version 1.6 or more.
- [SciPy](http://scipy.org): version 0.10 or more.
- [Cython](http://www.cython.org/): version 0.15 or more.

Additionally, you may want to install:

[Matplotlib](http://matplotlib.org/downloads.html): version 1.0 or more. This
is needed to generate the examples in the documentation.
You may check out Dependencies for other Optional installations like *FreeImage* and *PyQt4*.

### INSTALL

This is the easiest part. Just download the scikit-image.zip. Alternately, you
can also clone the repo to your local directory. Browse to the scikit-image
folder after unzipping it and simply run the following to install skimage
globally:

    $ python setup.py install

### EDITOR

The code written in scikit-image as is most of the open source Python code is
written following the PEP8 guidelines. Hence, you want to be able to configure
your editor so as to let it take care of it automatically. You are free to
choose whichever editor you are comfortable with but I personally prefer
[Sublime Text 2](http://www.sublimetext.com/2). You’ll most likely want to get
a plugin for Flake8 or Pyflake.

In case of Sublime Text 2, a very useful tool for easily installing and
automatically updating the additional packages is Package Control. Once
Package Control is installed, press `Shift + Command + P` to open the Command
Palette. Browse below and click on “Package Control: Install Package“. And
install the `Python Flake8Lint` package. This ensures that the code you
write won’t have any PEP8 issues. Believe me guys, I faced a lot of issues due
to PEP8 issues. 42 comments for my first PR! You’ll be better off letting your
editor take care of it.
