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

## SLIC
### Algorithm

## Why use Julia?

## Just Show Me The Code!

## Recap

## References
