---
layout: post
title: "Implement SRCNN with Pure C++"
category: [Machine Learning]
tags: [Machine Learning, Deep learning, Image Processing]
---

## Motivation
When I started to take my master degree, my professor told me to implement
with pure C++ and CUDA. After working for two months, I eventually finished
the first version but the result seems not only blurry but also wrong size.

However, I learned how to use pointer and basic CUDA concepts in this project.
Also, I got some concepts of how a CNN work and its architecture. As a revenge, 
I want to implement SRCNN from scratch, and this post is talking
about the note when working on this project.

## What is Super Resolution?
Super resolution is an image processing technique that to construct an image with
a higher resolution. So far, we use a lot of features to represent the properties
of image for such application, and there is no exception of neural network. With
the help of neural network, we create a tempting result ever.

## So What is SRCNN? 
Convolutional Neural Network (CNN) is widely adopted on visual mission. As for 
super resolution, SRCNN is the first one using CNN to complete such task. Proposed
by Chao Dong *et al* in 2014 [1], SRCNN outperforms plenty of methods including
sparsed-coding.

What's more, the composition of SRCNN is quitely easy: 3 convolution and 2 activation
layer. I was pretty amazed that such structure can beat traditional methods, though
come up with a price that you have to resize the image to output size before pass
the image to CNN.

## Overall Structure
The following image shows the structure of SRCNN and is taken from [1].
![SRCNN structure](/assets/images/2020/06/15/SRCNN_structure.png)

Suggested by the picture, what we have to do is to implement convolution and activation
layer.

## Implementation Details
### Convolution Layer
#### Naive Method
It is simple to implement naive convolution: the value of each output neuron is the
summation of input neuron times kernel parameters.
The following pseudo-code helps you to realize this calculation:
```
for(k = 0; k < # of output channels; k += stride)
    for(n = 0; n < # of input channels; n += stride)
        for(i = 0; i < # of input height; i += stride)
            for(j = 0; j < # of input width; j += stride)
                sum = 0;
                for(l = 0; l < # of kernel height; l++)
                    for(m = 0; m < # of kernel width; m++)
                        sum += input[n][i][j] * kernel[k][n][l][m];
                output[k][i][j] += sum;
```

#### `im2col` Method
We see that we need 6 for-loop to do naive convolution in the previous section.
What's more, if the stride is sufficient large, we have to collect the values that
are far together, hence lost the locality usage of cache.

Suggested by [the author of Caffe](https://github.com/Yangqing/caffe/wiki/Convolution-in-Caffe:-a-memo), 
we can use `im2col` techinique to transform convolution calculation
into matrix multiplication problem. In this case, we utilize the locality
of computer architecture when performing multiplication. Besides, we
can further use a highly-optimized BLAS (Basic Linear Algebra Subprograms) library 
to boost our convolution calculation.

The following image show the concept of `im2col` and is taken from [2]:
![im2col concept](/assets/images/2020/06/15/Convolution_With_Im2col.png)

### Activation Layer
Compared to Convolution Layer, this layer is rather simple; you just pass the value
of each neuron to activation function and get the result.

I choose ReLU as this kind of activation function is used in SRCNN.

### Reading Weights
Though Caffe uses NCHW format, the author of SRCNN uses CHWN format to store their network weights:
```matlab=26
for i = 1 : channel
        for j = 1 : fnum
             temp = conv_filters(:,:,i,j);
             if channel == 1
                weights(:,j) = temp(:);
             else
                weights(i,:,j) = temp(:);
             end 
        end 
    end
```
For more information about weight format layout, you can visit [this post](https://oneapi-src.github.io/oneDNN/understanding_memory_formats.html) 
on Intel-MKLDNN.

In my implementation, I apply NCHW format.

## Result
> Just the old saying goes, show me the result!

The results of my SRCNN implementation are shown in this table:
| naive | im2col |
|-------|--------|
![](/assets/images/2020/06/15/srcnn_result_naive.bmp) | ![](/assets/images/2020/06/15/srcnn_result_im2col.bmp)

## Benckmark
Though `im2col` method can gain a boost with sacrificing the memory, it might be slower because we
have to do `im2col` which takes a lot of time.

The benchmark parameter is shown below:
- OS: macOS 10.13.4
- CPU: Intel Core-i7 7700 with one core
- Memory: 32 GB
- Compiler: Apple LLVM version 10.0.0
- Desired output size: 512 x 512 pixel
- Testing method: run program with one factor three time then get the average
- Time counting method: use `std::chrono`

And here are the benchmarks of each SRCNN implementation method comparison:
naive:
125.414
122.221
124.625

im2col:
323.543
339.007
316.313

### Making Faster
Convolution is a kind of embrassing parallel problem; that is, it applies an operation on a lot of data.
So we can use some parallel computing techniques to speed up the execution.

On CPU parallel scenario, we can choose OpenMP as a parallel computing target for its simplecity.

Here are the benchmarks after applying OpenMP of each SRCNN implementation comparison:

naive:
37.6111
39.9825
38.3595

im2col:
97.9533
103.525
94.5232

## Conclusion

## Special Thanks

## Reference
[1] http://mmlab.ie.cuhk.edu.hk/projects/SRCNN.html
[2] https://leonardoaraujosantos.gitbooks.io/artificial-inteligence/content/making_faster.html
