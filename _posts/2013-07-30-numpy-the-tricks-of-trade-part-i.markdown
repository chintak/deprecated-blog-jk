---
layout: post
title: "Numpy: The Tricks of Trade (Part I)"
date: 2013-07-30 16:24
comments: true
tags: [gsoc, python, numpy, tricks]
keywords: "numpy, indexing, fancy indexing, tricks"
---
This post aims to highlights some of the basic features of NumPy which gives it an edge over Python in-built containers for computational purposes. The next post will talk about some advanced and more fascinating features of NumPy.

[NumPy](http://www.numpy.org/) is an extension package for performing efficient manipulations in multi-dimensional data. The numpy array object is extremely efficient at handling such arithmetics and provides vast support for basic and advanced manipulations.

### CREATING A NUMPY ARRAY

{% highlight python lineanchors %}
# Creating a NumPy array
>>> import numpy as np
>>> an_array = np.array([[1,2,3], [4,5,6]], dtype=np.uint8)
>>> an_array
array([[1, 2, 3],
       [4, 5, 6]], dtype=uint8)
{% endhighlight %}

### SOME COMMONLY USED ATTRIBUTES AND METHODS OF A NUMPY ARRAY OBJECT
{% highlight python lineanchors %}
# Some Attributes
    >>> an_array
    array([[1, 2, 3],
           [4, 5, 6]], dtype=uint8)
    >>> an_array.shape
    (2, 3)
    >>> an_array.ndim
    2
    >>> an_array.T # Also as_array.transpose()
    array([[1, 4],
           [2, 5],
           [3, 6]], dtype=uint8)
    >>> an_array.max()
    6
    >>> an_array.argmax(axis=0) # Returns index of max element along the given axis
    array([1, 1, 1])
    >>> an_array.argmax(axis=1)
    array([2, 2])
    >>> an_array.mean()
    3.5
    >>> an_array.astype(dtype=np.float) # Convert to another data type
    array([[ 1., 2., 3.],
           [ 4., 5., 6.]])
    >>> an_array.strides # Number of steps to move in the memory to reach the next element of row or column
    (3, 1)
{% endhighlight %}

There are a whole lot more of such attributes and methods already
in-built for the most basic day-to-day operations on a numpy array
structure. The last attribute stated here, strides provides an idea of
the memory layout underneath and manipulations with these strides can
provide with some really powerful techniques by avoiding unnecessary
copies of the data. For example, the strides of a numpy array and its
transpose are simply swapped. So, the returned array is actually the
same data and not a copy of the original data. These kind of
techniques make numpy array very efficient and gentle on the
memory. More on this in the next post under
`numpy.lib.stride_tricks.as_strided` function.

{% highlight python lineanchors %}
# More about Transpose of an array
    >>> an_array.T.strides
    (1, 3)
    >>> array_trans = an_array.T
    >>> array_trans[1, 1] = 0
    >>> array_trans
    array([[1, 4],
           [2, 0],
           [3, 6]], dtype=uint8)
    >>> an_array
    array([[1, 2, 3],
           [4, 0, 6]], dtype=uint8)
{% endhighlight %}

### INDEXING

NumPy supports the Python-like slicing and indexing. Slices of numpy arrays are views (pointers to data as in C/C++) to the original data and any modification is reflected in the orginal data as well.

{% highlight python lineanchors %}
# Basics of Indexing
    # Indices start from 0, ..., (n-1) if there are 'n' elements
    >>> a = np.arange(10); a # Function to generate a sequence of numbers
    array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
    >>> a[2:5]
    array([2, 3, 4])
    >>> a[::1]
    array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
    >>> a[::2]  # Increments in the index
    array([0, 2, 4, 6, 8])
    # Indices start with -1 from the opposite direction: -n, ..., -1
    >>> a[3:-3]
    array([3, 4, 5, 6])
{% endhighlight %}

### FANCY INDEXING

NumPy also supports Fancy Indexing: indexing using boolean or integer arrays.

{% highlight python lineanchors %}
# Fancy Indexing
    # `np.random.random_integers` function to generate random integers
    # First 2 arguments refer to the range in which the random int are to be
    # generated and the 3rd arg is for the size of the output
    >>> mask = np.random.random_integers(0, 1, (3,3))
    >>> mask
    array([[0, 0, 0],
           [0, 0, 1],
           [1, 0, 0]])
    >>> arr = np.random.random_integers(10, 20, (3,3))
    >>> arr
    array([[13, 10, 18],
           [11, 14, 15],
           [10, 17, 11]])
    >>> arr[mask == 1]
    array([15, 10])
{% endhighlight %}

### REFERENCES

- [SciPy GitHub Lectures](http://scipy-lectures.github.io/)
- Van Der Walt, Stefan, S. Chris Colbert, and Gael Varoquaux. *“The NumPy array: a structure for efficient numerical computation.”* Computing in Science & Engineering 13.2 (2011): 22-30.

### FURTHER READING

[NumPy: The tricks of the trade (Part II)](/2013/07/numpy-the-tricks-of-the-trade-part-ii/)
