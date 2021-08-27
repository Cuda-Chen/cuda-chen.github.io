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

The base model is the well-known 5-layer LeNet [^1], and the
implementation is adopted from Flux.jl model zoo [^2]. As such, 
there are some differences between the original LeNet and 
the implementation in Flux.jl model zoo [^3]:
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
![model structure]()

## Let's fine tuning!
As such, hypermeter tuning plays a crucial role in machine learning
model development. Though the LeNet implementation of Flux.jl can achieve
98% top-1 accuracy, I still want to try whether I can break the limits.
What's more, by experimenting fine tuning, I can attain the knowledge
which parameters plays the major role in certain task.

### baseline
> Baseline model can be found here:
> https://github.com/FluxML/model-zoo/blob/master/vision/conv_mnist/conv_mnist.jl

```
Epoch: 0   Train: (loss = 2.3162f0, acc = 12.1333)   Test: (loss = 2.316f0, acc = 12.11)
Progress: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████| Time: 0:00:41
Epoch: 1   Train: (loss = 0.1586f0, acc = 95.3417)   Test: (loss = 0.145f0, acc = 95.53)
Progress: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████| Time: 0:00:12
Epoch: 2   Train: (loss = 0.1079f0, acc = 96.7733)   Test: (loss = 0.0958f0, acc = 97.03)
Progress: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████| Time: 0:00:12
Epoch: 3   Train: (loss = 0.0829f0, acc = 97.515)   Test: (loss = 0.0717f0, acc = 97.75)
Progress: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████| Time: 0:00:13
Epoch: 4   Train: (loss = 0.0639f0, acc = 98.0883)   Test: (loss = 0.0573f0, acc = 98.21)
Progress: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████| Time: 0:00:12
Epoch: 5   Train: (loss = 0.0614f0, acc = 98.12)   Test: (loss = 0.0539f0, acc = 98.25)
[ Info: Model saved in "runs/model.bson"
Progress: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████| Time: 0:00:12
Epoch: 6   Train: (loss = 0.0593f0, acc = 98.2017)   Test: (loss = 0.058f0, acc = 98.13)
Progress: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████| Time: 0:00:12
Epoch: 7   Train: (loss = 0.0464f0, acc = 98.6083)   Test: (loss = 0.0464f0, acc = 98.52)
Progress: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████| Time: 0:00:11
Epoch: 8   Train: (loss = 0.04f0, acc = 98.7867)   Test: (loss = 0.039f0, acc = 98.77)
Progress: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████| Time: 0:00:12
Epoch: 9   Train: (loss = 0.0393f0, acc = 98.7833)   Test: (loss = 0.0416f0, acc = 98.63)
Progress: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████| Time: 0:00:11
Epoch: 10   Train: (loss = 0.0348f0, acc = 98.9667)   Test: (loss = 0.0388f0, acc = 98.67)

```

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
![model performance comparison of different batch size]()
![model training time comparison of differences batch size]()

32

64

256

512
 
### regularizer parameter

In this post, the best L2 regularizer parameter is **1e-6**.
![model performance comparison of regularizer]()
![model training time comparison of regularizer]()

### optimizer

In this post, the best optimizer is **AdaBelief**.
![model performance comparison of optimizer]()
![model training time comparison of optimizer]()

## Conclusion

## Table to show the training environment 
- CPU: Intel(R) Core(TM) i7-9700 CPU @ 3.00GHz
- RAM: 16 GiB
- OS: Fedora 33 (Linux Kernel 5.13.12)
- Julia version: 1.6.2
- Flux.jl version: v0.12.4

## References
[^1] http://yann.lecun.com/exdb/publis/pdf/lecun-98.pdf

[^2] https://github.com/FluxML/model-zoo/blob/33f5c472c321a50fc2105358a00eb7b3ec0ffa5e/vision/conv_mnist/conv_mnist.jl#L21

[^3] https://pabloinsente.github.io/the-convolutional-network
