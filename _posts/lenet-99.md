---
layout: post
title:
category:
tags:
---

## Introduction
Fine-tuning plays a great role in model training, and realizing
the meaning of each hypermeter lets you succeed.

In this post, I am going to show you how I achieve 99% top-1
accuracy on MNIST hand-written number recognition by just 
fine-tuning three hypermeters. I also try
to implemnet a classic CNN model, LeNet-5, from scratch for
making me familiarize the structure and the basic
components of a CNN mode.

## Base Model
> You can get the code from [my repo](https://github.com/Cuda-Chen/flux-lenet).
>

The base model is the well-known 5-layer LeNet [1], and the
implementation is adopted from Flux.jl model zoo [2]. As such, 
there are some differences between the original LeNet and 
the implementation in Flux.jl model zoo [3]:
1. The activation function of convolution layer in LeNet uses
**sigmoid**, whilst in Flux.jl model zoo uses **ReLU**.
2. The pooling layer in LeNet uses **average** pooling, whereas
in Flux.jl mode zoo uses **max** pooling.
3. The activation function of pooling layer in LeNet uses
**scaled hyperbolic tangent**, while the one in Flux.jl model zoo uses
**identity** (linear).
4. The multi-class classification used in original LeNet paper
uses **Euclidean radial basis (RBF) function**. [3] However, 
**softmax** is used in Flux.jl's implementation.

For your ease, I list the structure of my implementation:
![]() <--- pirture here

## Let's fine tuning!
As such, hypermeter tuning plays a crucial role in machine learning
model development. Though the LeNet implementation of Flux.jl can achieve
98% top-1 accuracy, I still want to try whether I can break the limits.
What's more, by experimenting fine tuning, I can attain the knowledge
which parameters plays the major role in certain task.

### batch size
Batch size means how many training samples are used in one iteration.
Furthermore, it represents you update, or formally, calculate the loss then
back-propagate, the parameters of the model after ingest certain number
of training samples. Therefore, assuming the following scenes:
1. If you update the parameters after ingest **the whole data**. You may
get a fast parameter updating time, but the model will perform poorly
on actual case because the model falls into the trap of local minima.
Besides, it needs a huge number of memory to load the data.
2. If you update the parameters after ingest **each number of data** (only
one data in each iteration). You may get a model with fantastic outcome, 
but it takes an extraordinary time to train as it updates the parameters 
in each iteration.

As such, choosing the right number of batch size can:
- reduce the training time and memory
- coverange in better performance

In this post, I tried different number of batch size, and the best batch
size of my training platform is **32**.  
 
### learning rate
### regularizer

## Conclusion

## Table to show the training environment 

## References
[1] http://yann.lecun.com/exdb/publis/pdf/lecun-98.pdf

[2] https://github.com/FluxML/model-zoo/blob/33f5c472c321a50fc2105358a00eb7b3ec0ffa5e/vision/conv_mnist/conv_mnist.jl#L21

[3] https://pabloinsente.github.io/the-convolutional-network
