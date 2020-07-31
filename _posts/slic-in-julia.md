---
layout: post
title: slic-in-julia
category: [image processing]
tags: [image processing, SLIC, image segmentation, Julia]
---

## Motivation
Image segmentation plays a great role in order to realize which
objects construct an image. In order to reach this goal, people
have invented a lot of strategies -- one of which is superpixel.

SLIC is a well-known algorithm which runs in linear time complexity
, and has been implemented in various image processing libraries
including OpenCV and scikit-image. However, [JulaImages](
https://juliaimages.org/latest/) has not implemented SLIC yet,
though this Julia image processing package contains Felzenswalb
and quick shift. Therefore, I want to implmenet SLIC in Julia
to let me famlilar with SLIC algorithm but also Julia.

## Image segmentation
Image segmentation is a process to classify which cluster each 
pixel should belong to. The goal of this is to let image more representive
and easier for us to analyze. So far, people have invented a lot of
techniques according to each characteristic of each object such
as color, texture, and intensity, etc.

![image alt](/assets/images/2020/07/29/image_segmentation_example.png)
<center>A clear representation of image segmentation. Image source: jeremyjordan.me</center>

## What's Superpixel?
We use the properties of a pixel to classify which object it should belongs
to. However, let me give you a question: is it proper to determine a
range of an object in such small scale? Seems not. In 2003, Xiaofeng Ren and Jitendra Malik
discussed about this problem and proposed superpixel. [1] The intuition
of superpixel is pretty simple: rather than determine each pixel, we can 
group pixels with akin properties into a larger one -- called superpixel -- 
for further analyze. In this way, we not only get more meaningful regions
but also improve computational efficiency. [2]

## SLIC (Simple Linear Iterative Clustering)
We can use some strategies to cluster pixels with similar properties into
a superpixel such as graph theorem, gradient decent, or even machine learning
methods. As the most well-known one of using machine learning method, SLIC
comes to the place. The intuition of SLIC is pretty naive: we can set some
seeds to cluster the surrounding pixels to form superpixels.

You will ask a problem: "It seems pretty straightforward. So what's special
of SLIC?" ~

### Algorithm

## Why use Julia?

## Just Show Me The Code!

## Recap

## References
[1] https://www2.eecs.berkeley.edu/Research/Projects/CS/vision/grouping/papers/ren_malik_iccv03.pdf
[2] https://www.pyimagesearch.com/2014/07/28/a-slic-superpixel-tutorial-using-python/
[3] https://www.iro.umontreal.ca/~mignotte/IFT6150/Articles/SLIC_Superpixels.pdf

## Special Thanks
