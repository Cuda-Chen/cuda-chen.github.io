---
layout: post
title: "Implement SRCNN with Pure C++"
category: [Machine Learning]
tags: [Machine Learning, Deep learning, Image Processing]
---

## Motivation
When I started to take my master degree, my professor told me to implement
with pure C++ and CUDA. After working for two months, I eventually finished
the first version but the result seems blurry.

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
![SRCNN structure](/assets/images/2020/05/15/SRCNN_structure.png)

## Implementation Details
### Convolution Layer

### Activation Layer
Compared to Convolution Layer, this layer is rather simple; you just pass the value
of each neuron to activation function and get the result.

I choose ReLU as this kind of activation function is used in SRCNN.

## Reference
[1] http://mmlab.ie.cuhk.edu.hk/projects/SRCNN.html
