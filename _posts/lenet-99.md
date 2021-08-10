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

## Let's fine tuning!
> You can get the code from [my repo](https://github.com/Cuda-Chen/flux-lenet).
### batch size
### learning rate
### regularizer

## Conclusion

## References
[1] http://yann.lecun.com/exdb/publis/pdf/lecun-98.pdf

[2] https://github.com/FluxML/model-zoo/blob/33f5c472c321a50fc2105358a00eb7b3ec0ffa5e/vision/conv_mnist/conv_mnist.jl#L21

[3] https://pabloinsente.github.io/the-convolutional-network
