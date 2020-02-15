---
layout: post
title: "Build OpenCV DNN Module with Nvidia GPU Support on Ubuntu 18.04"
category: [image processing, programming]
tags: [CUDA, cuDNN, OpenCV]
---

In 2017, OpenCV 3.3 brought a revolutionary DNN module. As time passes, it currently
supports plenty of deep learning framework such as TensorFlow, Caffe, and Darknet, etc.
With the help of this module, we can use OpenCV to:
1. Load a pre-trained model from disk.
2. Making a preprocessing to an input image.
3. Pass the image through the network and obtain the output results.
4. Least dependency (only OpenCV !).

Although we *cannot* train a deep learning models with OpenCV (this does not make sense, though),
we can take the models trained by other deep learning tools and perform inference using OpenCV
with incredible speed on CPU.

However, the DNN module does not support Nvidia GPU until now. In Google Summer of Code (GSoC) 2019,
OpenCV DNN module finally [supports Nvidia GPU](https://summerofcode.withgoogle.com/archive/2019/projects/6169918255398912/)
 for inference with the work of [Davis King](https://github.com/davisking) -- the creator of [dlib](https://github.com/davisking/dlib) --
and [Yashas Samaga](https://github.com/YashasSamaga), and this work was made in public in [version 4.2.0](https://github.com/opencv/opencv/wiki/ChangeLog#version420).
As a programmer expertised with OpenCV,
I think I should share this exciting news in the first moment. Besides, I will teach you how to compile and install 
OpenCV with the utility of Nvidia GPU for deep neural network inference, and I will provide the workable
example code both C++ and Python.

## Compile OpenCV with CUDA and cuDNN-enabled DNN Module
### Assumptions
1. A Nvidia GPU (of course!).
2. Ubuntu 18.04 (or other equivalent Debian-based distro).
3. CUDA 10.0 and corresponding version of cuDNN.

### Step #1: Install Nvidia CUDA drivers, CUDA toolkit, and cuDNN
![image alt](/assets/images/2020/02/15/opencv_dnn_gpu_cuda_drivers.png)
1. ***Backup your system, period.***
To my experience, you may accidently break the system when installing
any kind of driver. Although you can recover the system doing the 
installation backwards, sometimes you just can't recover successfully
because you forget the steps or even the driver does some changes and
you don't notice at all. As a person who broke the system twice (both
with install CUDA drivers), I ***strongly suggest*** you backup your system in any condition.
2. Pre-installation Actions
I make some outlines here. For detailed information you can visit 
[here](https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html#pre-installation-actions). 
    - `lspci | grep -i nvidia`: verify you have a CUDA-capable GPU.
    - `uname -m && cat /etc/*release`: verify you have a supported version of Linux.
    - `gcc --version`: verify the system has GCC installed.
        - If not, install GCC by `sudo apt-get update && sudo apt-get upgrade && sudo apt-get install build-essential`.
    - `uname -r`: check the system has the correct kernel headers and development packages installed.
    - `sudo apt-get install linux-headers-$(uname -r)`: install the kernel headers and development packages of
current running kernel.
3. Install CUDA drivers
I will choose runfile packages to install CUDA drivers because you may upgrade the packages if you install
with distribution-specific packages methods, which will cause huge problem afterwards.

## to be continued ...

{% if site.liker_id %}
<iframe
  style="width: 100%; max-width: 485px; height: 240px; margin: auto; overflow: hidden; display: block;"
  src="https://button.like.co/in/embed/{{site.liker_id}}/button?referrer={{ page.url | absolute_url | cgi_escape }}">
</iframe>
{% endif %}
