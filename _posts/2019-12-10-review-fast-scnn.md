---
layout: post
title: "Review: Fast-SCNN"
categories: [paper review]
tags: [semantic segmentation, machine learning, image processing, image segmentation]
---

# Note
- still working...
- propose a fast segmentation network for above real-time scene understanding.
- Sharing the computational cost of the multi-branch network yields run-time efficiency.
- our skip connection is shown beneficial for recovering the spatial results in experiment.
- If train for long enough, large-scale pre-training of the model on an additional auxiliary
task is not necessary for the low capacity network.

In this post, Fast-SCNN (fast segmentation convolutional neural network) is briefly reviewed.
This architecture aims on real-time semantic segmentation tasks, and it can reach 123.5 frames
per second on Cityscapes dataset with a high resolution of input image (1024 x 2048 px).

# Outline
- Fast-SCNN architecture overview
- Experiment Results
