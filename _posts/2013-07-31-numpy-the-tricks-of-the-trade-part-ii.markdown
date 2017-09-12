---
layout: post
title: 'NumPy: The tricks of the trade (Part II)'
date: 2013-07-31 16:24
comments: true
tags: [gsoc, numpy, python, tricks]
keywords: "numpy, tricks, advanced numpy, vectorization, vector function, broadcasting, indexing, as_strided, ogrid, newaxis, einsum"
---

This post will describe about the more advanced and fun stuff about [NumPy](http://www.numpy.org/). For basics, refer to Part I. I will be talking about the following topics in this post:

- Vectorization
- Broadcasting
    - What is it and how does it work ?
    - `np.newaxis`
    - `np.ogrid`
    - Creating a grid using broadcasting
- Making use of the `numpy.lib.stride_tricks.as_strided`
    - Underlying numpy implementation
    - Advantages and caution
    - `np.einsum`
- Example

### Vectorization

First let's revisit how we would do any arithmetic operation on all the elements of list ([Python](http://www.python.org/) in-built container). Looping through all the elements is probably the only way to go about it. The neatest syntax is using [list comprehensions](http://en.wikipedia.org/wiki/List_comprehension).

{% highlight python lineanchors %}
# List Comprehension
>>> L = [1,2,3,4]
>>> [i * 2 for i in L]  # list comprehension
[2, 4, 6, 8]
{% endhighlight %}

Now imagine doing this for a multi-dimensional list of data and think about the readability. Not so cool, right? This is what vectorization takes care of for NumPy arrays.

{% highlight python lineanchors %}
# Example of Vectorized operation
>>> a = np.array([1,2,3,4])
>>> a * 2
array([2, 4, 6, 8])
{% endhighlight %}

Simply put, vectorization takes one elemental operation and applies to all the elements in that array for you. Underneath, the implementatiosn are in C, hence providing substantial speed gains. NumPy already has a large number of operations vectorized, for eg: all arithmetic operators, logical operators, etc.
Numpy also provides a way for you to vectorize your function. All you need to do is:

- Write a function to do the operation you want to do, taking the elements of the array as arguments.
- Vectorize the function.
- Provide the arrays as inputs to this vectorized function.
- Done

{% highlight python lineanchors %}
# Creating your own Vectorized function
# Note that the following function would already be vectorized since `+` and `**` operations are vectorized
>>> def add_square(a, b):
...     return (a + b)**2
...
>>> a_array = np.array([1,2,3])
>>> b_array = np.array([4,5,6])
>>> vadd_square = np.vectorize(add_square)
>>> vadd_square(a_array, b_array)
array([25, 49, 81])
{% endhighlight %}

This would help skip a lot of loops in your implementation. Towards the end of this post I'll provide an example which would help connect all the ideas listed here and enable you to perform really powerful operations rather trivially using NumPy. I've completely become a fan of NumPy!

### Broadcasting

If I ask you the answer to "banana * orange = ?", you'll most certainly look at me as if I'm crazy. But as it turns out, NumPy is also capable of handling operations between arrays of different sizes. The only criteria being that, NumPy should be able to extend all the arrays involved in an operation to a common shape. This is what we call Broadcasting. Let me give couple of examples to further elaborate on this idea.

{% highlight python lineanchors %}
# Broadcasting - Basic example
>>> a = np.array([0,1,2,3])
>>> b = np.array([0,1,2,3])
>>> a[:, np.newaxis] - b[np.newaxis, :]
array([[ 0, -1, -2, -3],
       [ 1, 0, -1, -2],
       [ 2, 1, 0, -1],
       [ 3, 2, 1, 0]])
{% endhighlight %}

Understanding `np.newaxis` would really be very helpful here. It basically just adds another dimension (axis). (duh!) But you can choose where you want to place the new axis as in x, y or z direction.

{% highlight python lineanchors %}
# Broadcasting using `np.newaxis`
# For illustration purpose, lets suppose that we want to compute the product of all the indices in a 3D grid
>>> a = np.array([0,1,2,3])
>>> b = np.array([0,1,2])
>>> c = np.array([0,1])
>>> a[:, np.newaxis, np.newaxis] * b[np.newaxis, :, np.newaxis] * c[np.newaxis, np.newaxis, :]
array([[[0, 0],
        [0, 0],
        [0, 0]],
       [[0, 0],
        [0, 1],
        [0, 2]],
       [[0, 0],
        [0, 2],
        [0, 4]],
       [[0, 0],
        [0, 3],
        [0, 6]]])
>>> a.strides
(8,)
>>> a[:, np.newaxis, np.newaxis].strides
(8, 0, 0)
{% endhighlight %}

**Comments**

Here, we are extending `a` and making it a 3D array using `np.newaxis` however, as you can see the strides, no additional memory is allocated. This makes things a lot easier, instead of creating 3, 3D arrays and then multiplying.
Practically, you can use this for indexing purposes.

{% highlight python lineanchors %}
# Using Broadcasting for indexing purpose
>>> m = np.arange(24).reshape(4,3,2)
>>> m
array([[[ 0,  1],
        [ 2,  3],
        [ 4,  5]],
       [[ 6,  7],
        [ 8,  9],
        [10, 11]],
       [[12, 13],
        [14, 15],
        [16, 17]],
       [[18, 19],
        [20, 21],
        [22, 23]]])
>>> m[a[:, np.newaxis, np.newaxis], b[np.newaxis, :, np.newaxis], c[np.newaxis, np.newaxis, 0]]
array([[[ 0],
        [ 2],
        [ 4]],
       [[ 6],
        [ 8],
        [10]],
       [[12],
        [14],
        [16]],
       [[18],
        [20],
        [22]]])
{% endhighlight %}


### Fancy That: `np.ogrid`

You can also this function to create a row array and a column array, and use broadcasting to generate a complete array as follows:

{% highlight python lineanchors %}
# Using `np.ogrid` - Basics
>>> row, col = np.ogrid[0:3, 0:3]  # note it's not a function
>>> row, col
array([[0],
       [1],
       [2]]), array([[0, 1, 2]])
>>> a = np.arange(16).reshape(4,4)
>>> a
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11],
       [12, 13, 14, 15]])
>>> a[0+row, 0+col]
array([[ 0,  1,  2],
       [ 4,  5,  6],
       [ 8,  9, 10]])
>>> a[1+row, 1+col]
array([[ 5,  6,  7],
       [ 9, 10, 11],
       [13, 14, 15]])
#  OR
>>> a[1:4, 1:4]
{% endhighlight %}

**Comments**

Here, we generate a row and column array using `np.ogrid` (note that its not a function). The option with slicing does look like a neater and cleaner approach, however if you want to do this in a loop with a lot of variables flying around, it may not be the best approach.

### Fancy That: `numpy.lib.stride_tricks.as_strided`

This is one of the wonders of NumPy which has the power to make loops outdated. Strides, is a method in which NumPy can keep a track or this is how it knows how to get to the nest element in the row or column. How many leaps in the memory to the next element ? This of course also depends on the data type you are using. For example:

{% highlight python lineanchors %}
# Concept of strides with data types
>>> a = np.array([[1,2,3], [4,5,6]], dtype=np.uint8)
>>> a.strides
(3, 1)
>>> a = np.array([[1,2,3], [4,5,6]], dtype=np.uint16)
>>> a.strides
(6, 2)
{% endhighlight %}

**Comments**

So, in the first case, take move 1 byte to get to the next column element and 3 bytes to get to the next row element. Similarly for the second case.

Now, let's see how `as_strided` works and how can we use this to perform many operations efficiently. Crudely, this function provides a way to access the same underlying array in different shapes. That being said there is also an option to define different strides for this particular view on the array. As we all know that by default, we access the elements in a row: C contiguous (or column: fortran contiguous), one after the other. But using this function it is possible to skip elements in the middle and point to say, all the diagonal elements only, hence enabling you to extract the diagonal entries of even a multi-dimensional tensor using just this function and not allocating any additional memory for the same. Some examples:

{% highlight python lineanchors %}
# Understanding `as_strided`
>>> a = np.arange(9, dtype=np.uint8).reshape(3,3)
>>> a
array([[0, 1, 2],
       [3, 4, 5],
       [6, 7, 8]], dtype=uint8)
>>> a.itemsize  # Number of bytes taken by each element
1
>>> a.strides
(3, 1)
>>> from numpy.lib.stride_tricks import as_strided
>>> a_view = as_strided(a, shape=(3, ), strides=((3+1)*a.itemsize, ))
>>> a_view
array([0, 4, 8], dtype=uint8)
>>> a_view.strides
(4,)
{% endhighlight %}

**Comments**

Any changes in this strided view will also get reflected in the original array.

{% highlight python lineanchors %}
# `as_strided` provides a view; not a copy of the array
>>> a_view[1] = 10
>>> a
array([[ 0,  1,  2],
       [ 3, 10,  5],
       [ 6,  7,  8]], dtype=uint8)
{% endhighlight %}

**Caution**

This function does not check whether you stay inside the memory block bounds. This could lead to some garbage values popping up.

{% highlight python lineanchors %}
# as_strided does not check for memory bounds - you have to!
>>> a_view = as_strided(a, shape=(4, ), strides=((3+1)*a.itemsize, ))
>>> a_view
array([ 0, 10,  8,  0], dtype=uint8)
{% endhighlight %}

*In fact, it is this command over memory layout which helps NumPy perform wonders like Broadcasting. This is how broadcasting is really implemented underneath, using 0 strides.*

{% highlight python lineanchors %}
# Broadcasting and as_strided - Same underneath
>>> a = np.arange(4, dtype=np.uint8)
>>> a
array([0, 1, 2, 3], dtype=uint8)
>>> a.strides
(1,)
>>> a_view = as_strided(a, shape=(4,4), strides=(0, 1))
>>> a_view
array([[0, 1, 2, 3],
       [0, 1, 2, 3],
       [0, 1, 2, 3],
       [0, 1, 2, 3]], dtype=uint8)
>>> b_view = as_strided(a, shape=(4,4), strides=(1, 0))
>>> b_view
array([[0, 0, 0, 0],
       [1, 1, 1, 1],
       [2, 2, 2, 2],
       [3, 3, 3, 3]], dtype=uint8)
>>> b_view[1,1] = 4
>>> b_view
array([[0, 0, 0, 0],
       [4, 4, 4, 4],
       [2, 2, 2, 2],
       [3, 3, 3, 3]], dtype=uint8)
{% endhighlight %}

**Comments**

As you can see, all the "1" were really just a single "1". Hence, when you change any of the element, the change is reflected everywhere since in reality they are all the same.

### Fancy That: `np.einsum`

This stands for Einstein's summation. Using this function you can implement a lot of in-built functions involving summation. The syntax is as follows:

{% highlight python lineanchors %}
# Basics of `np.einsum`
>>> a = np.arange(16).reshape(4,4)
>>> a
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11],
       [12, 13, 14, 15]])
>>> np.einsum('ii', a)
30
>>> np.einsum('ij', a)
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11],
       [12, 13, 14, 15]])
>>> np.einsum('ji', a)
array([[ 0,  4,  8, 12],
       [ 1,  5,  9, 13],
       [ 2,  6, 10, 14],
       [ 3,  7, 11, 15]])
{% endhighlight %}

The idea is to represent each dimension (axis) by a label, 'i' or 'j' here. It is similar to iterating over a loop. In the first case, it picks up all the elements where the indices in both the dimensions are equal and sums it over, summation of the diagonal elements, trace of the array. The order in which the label is alphabetical and important. In the second example, since 'j' appears after 'i', it first loops through elements along the column. The third example, hence produces the transpose. Now, let's see an example involving some what more complex applications of `np.einsum`.

{% highlight python lineanchors %}
# Advanced examples - check below for explanation
>>> a = np.arange(24).reshape(6,4)
>>> a
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11],
       [12, 13, 14, 15],
       [16, 17, 18, 19],
       [20, 21, 22, 23]])
>>> b = np.arange(6)
>>> b
array([0, 1, 2, 3, 4, 5])
>>> np.einsum('ij, i->j', a,b)  # Column sum
array([220, 235, 250, 265])
>>> np.einsum('ij, i->i', a,b)  # Row sum
array([  0,  22,  76, 162, 280, 430])
{% endhighlight %}

**Comments**

We use `->` to indicate the order of the output array. So think of `'ij, i->j'` as having left hand side (LHS) and right hand side (RHS). Any repetition of labels on the LHS computes the product element wise and then sums over. By changing the label on the RHS (output) side, we can define the axis in which we want to proceed with respect to the input array, i.e. summation along axis 0, 1 and so on. The above two examples can also be computed rather trivially, as follows:

{% highlight python lineanchors %}
# Alternate to `np.einsum`: Product and reduce
>>> a*b[:,np.newaxis]
array([[  0,   0,   0,   0],
       [  4,   5,   6,   7],
       [ 16,  18,  20,  22],
       [ 36,  39,  42,  45],
       [ 64,  68,  72,  76],
       [100, 105, 110, 115]])
>>> (a*b[:,np.newaxis]).sum(1)
array([  0,  22,  76, 162, 280, 430])
>>> (a*b[:,np.newaxis]).sum(0)
array([220, 235, 250, 265])
{% endhighlight %}


### Example

Finally, before concluding this post, I would like to give an example, real code which I am using in my project, to help connect all these ideas and put them in perspective.

Here is the task: **Template Matching**

- Extract a small windowXwindow template/patch about a pixel, i.e. this pixel should also be the centre of patch. (call it template patch)
- Loop: Compute a patch about all pixels in the image (call it, sample patch)
    - Find the Sum of Squared Differences (SSD) of the template and the sample patch
- Find the patch with the minimum SSD

The most obvious way to go about this would be using a nested loop. The code for the same is as follows:

{% highlight python lineanchors %}
# Template Matching using loops
total_weight = valid_mask.sum()
for i in xrange(h):
    for j in xrange(w):
        sample = image[i - (window / 2): i + (window / 2) + 1,
j - (window / 2): j + (window / 2) + 1]
dist = (template - sample) ** 2
ssd[i, j] = dist.sum() / total_weight  # `total_weight` is just for normalization
{% endhighlight %}

Now this piece of code, doesn't seem all that scary and probably you'd do this every now and then in C/C++. However, Python loops are pretty darn slow. My complete code with this ran in about 19.37 seconds. The majority of the time being spent here.

However, there is a better way to write this. For starters, lets get rid of the *huge slicing syntax* with... `np.ogrid`, of course.

{% highlight python lineanchors %}
# np.ogrid for better indexing
t_row, t_col = np.ogrid[-(window / 2):(window / 2) + 1, -(window / 2):(window / 2) + 1]
{% endhighlight %}

You would argue about how would using this be any different, well it's not! It's just cleaner and more convenient to write this once and henceforth simply use the following to extract a template of size `window X window` about a pixel.
{% highlight python lineanchors %}
#
input_image[i + t_row, j + t_col]
{% endhighlight %}

Much like shifting of origin in Geometry!
Let's rethink, why are we really using the nested loops here? To extract template of the given size pixel by pixel and perform the operations. Is it possible to create such templates beforehand and perform the arithmetic operations directly on all the templates, leveraging the vector property of operators? However, creating separate templates for every pixel would most certainly be expensive on the memory. How can we do this without any additional memory overhead? `as_strided`!

{% highlight python lineanchors %}
# as_strided to provide a `window` sized window to the `input_image`
y = as_strided(input_image,
               shape=(input_image.shape[0] - window_size[0] + 1,
                      input_image.shape[1] - window_size[1] + 1,) +
               window_size,
               strides=input_image.strides * 2)
{% endhighlight %}

Suppose here, that input_image has the dimensions `(M, N)`, the resulting strided view, y would be a 4D array with `(M, N, window, window)` dimensions, as specified by the shape key argument. Note that `input_image.strides * 2` represents a multiplication on a tuple which replicates and concatenates the tuple.

{% highlight python lineanchors %}
# as_strided work here
>>> input_image = np.arange(16).reshape(4,4)
>>> input_image
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11],
       [12, 13, 14, 15]])
>>> y = as_strided(input_image,
...                shape=(input_image.shape[0] - window_size[0] + 1,
...                       input_image.shape[1] - window_size[1] + 1,) +
...                window_size,
...                strides=input_image.strides * 2)
>>> y
array([[[[ 0,  1,  2],
         [ 4,  5,  6],
         [ 8,  9, 10]],
        [[ 1,  2,  3],
         [ 5,  6,  7],
         [ 9, 10, 11]]],
       [[[ 4,  5,  6],
         [ 8,  9, 10],
         [12, 13, 14]],
        [[ 5,  6,  7],
         [ 9, 10, 11],
         [13, 14, 15]]]])
>>> y[0,0]
array([[ 0,  1,  2],
       [ 4,  5,  6],
       [ 8,  9, 10]])
>>> y[1,1]
array([[ 5,  6,  7],
       [ 9, 10, 11],
       [13, 14, 15]])
>>> y.shape
(2, 2, 3, 3)
>>> y.strides
(32, 8, 32, 8)  # 8 is the min step since this data type is 'int64' which occupies 8 bytes
>>> y.itemsize
8
>>> y.dtype
dtype('int64')
{% endhighlight %}

So, as we can see here, if we iterate through the 1st 2 dimensions of `y` we will have a `3X3` array with elements surrounding the inner elements of input_image. I find this to be extremely fascinating and really just brilliant!

**Remember**: `as_strided` is just a view, so **Without** adding any additional memory overhead we are able to accomplish this.

**Warning**
In spite of being just a view, directly performing arithmetic operations on this would allocate additional memory, a 4D that too! This would neutralise the unique advantage this stands to offer. Here, is where things get even more exciting: `np.einsum`.

{% highlight python lineanchors %}
# Using `np.einsum` to calculate the SSD without additional mem allocation
ssd = np.einsum('ijkl, kl, kl->ij', y, template, valid_mask,
                dtype=np.float)
ssd *= - 2
ssd += np.einsum('ijkl, ijkl, kl->ij', y, y, valid_mask)
ssd += np.einsum('ij, ij, ij', template, template, valid_mask)
{% endhighlight %}

Instead of directly computing the square of the difference (proving to be rather costly), use the basic algebraic expansion: `(a - b)**2 = a**2 - 2*a*b + b**2`. Here, observe that, in the first computation, which calculates `sample * template` the output is being stored in a 2D array. Following which, the subsequent results are simply accumulated in this very array. So in the end, still the overall memory load is just of allocating this 2D array for storing the output, which is what we sought for.

The complete function which serves as a replacement for the nested loop is as follows:

{% highlight python lineanchors %}
# Complete Template Matching function - no loops
def _sum_sq_diff(input_image, template, valid_mask):
    """This function performs template matching. The metric used is Sum of
    Squared Difference (SSD). The input taken is the template who's match is
    to be found in image.
    Parameters
    ---------
    input_image : array, np.float
        Input image of shape (M, N)
    template : array, np.float
        (window, window) Template who's match is to be found in input_image.
    valid_mask : array, np.float
        (window, window), governs differences which are to be considered for
        SSD computation. Masks out the unknown or unfilled pixels and gives a
        higher weightage to the center pixel, decreasing as the distance from
        center pixel increases.
    Returns
    ------
    ssd : array, np.float
        (M - window +1, N - window + 1) The desired SSD values for all
        positions in the input_image
    """
    total_weight = valid_mask.sum()
    window_size = template.shape
    y = as_strided(input_image,
                   shape=(input_image.shape[0] - window_size[0] + 1,
                          input_image.shape[1] - window_size[1] + 1,) +
                    window_size,
                    strides=input_image.strides * 2)
    ssd = np.einsum('ijkl, kl, kl->ij', y, template, valid_mask,
                    dtype=np.float)
    # Refer to the comment below for the explanation
    ssd *= - 2
    ssd += np.einsum('ijkl, ijkl, kl->ij', y, y, valid_mask)
    ssd += np.einsum('ij, ij, ij', template, template, valid_mask)
    return ssd / total_weight
{% endhighlight %}

**Comments**

Above, since 'kl' are repeated on LHS, we first compute the product of elements in the 3rd and 4th dim of `y` with corresponding elements of `template` and `valid_mask` and sum. Since, the output labels are 'ij', we want the summation along the other axes. Hence, each element of `ssd` corresponds to element wise product of `template` and `valid_mask` with the `(3, 3)` "sample patch" stored in the 3rd and 4th dimensions of `y`.

**Results**

The fascinating part is that testing this implementation for the same test case as the one used for nested loops completes the computation in about 0.342 seconds! So that's 19.37 -> 0.34 seconds!

I am absolutely blown away by NumPy, and if not for anything else, I am definitely thankful to GSoC and scikit-image for introducing me to NumPy!

Next post will be on my second module for GSoC: Texture Synthesis, whose initial implementation is complete and now in the review phase.

Cheers!

**References**

- Much of the content of this post is inspired from the StackOverflow answer to my question by Jaime, of scikit-image community. Thanks a lot! [Answer](http://stackoverflow.com/questions/17881489/faster-way-to-calculate-sum-of-squared-difference-between-an-image-m-n-and-a/17885996?noredirect=1#17885996)
- [http://scipy-lectures.github.io/](http://scipy-lectures.github.io/)
- Van Der Walt, Stefan, S. Chris Colbert, and Gael Varoquaux. “The NumPy array: a structure for efficient numerical computation.” Computing in Science & Engineering 13.2 (2011): 22-30.

**Related articles**

- [NumPy: The tricks of the trade (Part I)](/2013/07/numpy-the-tricks-of-trade-part-i/)
- [Diving into NumPy Code, SciPy 2013 Tutorial](https://www.youtube.com/watch?v=G-Iep_MnSv8)
- [Using NumPy to Perform Mathematical Operations in Python](https://www.youtube.com/watch?v=vWkb7VahaXQ)

**Update:**

Added another example to the section on `np.einsum` and comment explaining the role of `np.einsum` in the `_sum_sq_diff` function. Thanks for pointing out, Juan!
