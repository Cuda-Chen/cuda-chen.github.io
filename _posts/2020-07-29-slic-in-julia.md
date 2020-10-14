---
layout: post
title: A SLIC implementation in Pure Julia
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
and quick shift. Therefore, I want to implement SLIC in Julia
to let me familiar with SLIC algorithm but also Julia.

## Image segmentation
Image segmentation is a process to classify which cluster each 
pixel should belong to. The goal of this is to let image more representative
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
of SLIC?" Rather than search all the pixels like normal K-means, SLIC uses
a modified K-means which restricts the searching area into a certain space. 
To my opinion, SLIC adopts this way because each pixel surrounding to a 
center should have similar properties.

![slic_difference](/assets/images/2020/07/29/slic_difference.png)
<center>A comparison between normal K-means and SLIC modified K-means. Taken from [4]</center>

### Algorithm
SLIC just requires two input parameters: 
1. ***K***: number of superpixels.
2. ***m***: compactness factor to let superpixel be more compact or not.

And the simplified distance calculation is shown as follows:
![slic-distance_calculation](/assets/images/2020/07/29/slic_distance_calculation.png)

Where S = sqrt(N/K) 
(N represents the number of pixels.)

The following lines describe the algorithm of SLIC.
```
/∗ Initialization ∗/
Initialize cluster centers Ck = [lk , ak , bk , xk , yk ]T by sampling pixels at regular grid steps S.
Move cluster centers to the lowest gradient position in a 3 × 3 neighborhood.
Set label l(i) = −1 for each pixel i. Set distance d(i) = ∞ for each pixel i.
 
repeat
/∗ Assignment ∗/
for each cluster center Ck do
    for each pixel i in a 2S × 2S region around Ck do 
        Compute the distance D between Ck and i.
        if D < d(i) then
            set d(i) = D
            set l(i) = k 
        end if
    end for 
end for
 
/∗ Update ∗/
Compute new cluster centers. Compute residual error E.
until E ≤ threshold
```

## Why use Julia?
As a new-born language, Julia grasps my sight because it is fast, dynamic, and general.
Though SLIC has been implemented in various image processing libraries such as OpenCV and
scikit-image, there are no any implementations in pure Julia, which seems a pity that
we don't use the benefits of this programming language.

## Just Show Me The Code!
Because it is lengthy, you can view the code [here](https://github.com/Cuda-Chen/SLIC.jl/blob/master/src/SLIC.jl).
Feel free to ask any questions if you cannot figure out :)

## Recap
In this post, I introduce the concept of image segmentation as well as the history of
superpixel. Next, I make an introduction of SLIC superpixel and the algorithm. Last,
I express why I want to implement SLIC in pure Julia and leave the link of my implementation.

For the future aspect, I will work on enforce connectivity after we generate the superpixel.

> Oct. 4, 2020 added: Heads up! I have added enforce connectivity in my Julia SLIC implementation!
>

## References
[1] https://www2.eecs.berkeley.edu/Research/Projects/CS/vision/grouping/papers/ren_malik_iccv03.pdf

[2] https://www.pyimagesearch.com/2014/07/28/a-slic-superpixel-tutorial-using-python/

[3] https://www.iro.umontreal.ca/~mignotte/IFT6150/Articles/SLIC_Superpixels.pdf

[4] http://islab.ulsan.ac.kr/files/announcement/450/PAMI(2012)%20SLIC%20Superpixels%20Compared%20to%20State-of-the-Art%20Superpixel%20Methods.pdf

## Special Thanks
This post wouldn't appear if [laixintao](https://github.com/laixintao) had not create 
[this](https://www.kawabangga.com/posts/1923) awesome and clear post (written in Simplified Chinese).
I would like to give my gratitude to laixintao as a brick of this Julia implementation.
