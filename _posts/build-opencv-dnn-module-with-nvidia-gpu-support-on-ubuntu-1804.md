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
4. Your system account has `sudo` privilege.
5. OpenCV 4.2.0.

### Step #1: Install Nvidia CUDA driver, CUDA toolkit, and cuDNN
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
3. Install CUDA driver
Install CUDA drive by typing the following commands:
```
$ sudo add-apt-repository ppa:graphics-drivers/ppa
$ sudo apt-get update
$ sudo apt-get install nvidia-driver-418
```

After the installation, reboot the system:
```
$ sudo reboot now
```

Once your system brings up, type `nvidia-smi` and it should give a similar output:
```
$ nvidia-smi
Sat Feb 15 23:44:41 2020       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 440.33.01    Driver Version: 440.33.01    CUDA Version: 10.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  GeForce GTX 108...  On   | 00000000:01:00.0 Off |                  N/A |
|  0%   38C    P8    10W / 250W |    124MiB / 11176MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID   Type   Process name                             Usage      |
|=============================================================================|
|    0      1336      G   /usr/lib/xorg/Xorg                            49MiB |
|    0      1377      G   /usr/bin/gnome-shell                          71MiB |
+-----------------------------------------------------------------------------+
```

4. Install CUDA toolkit 
I will choose runfile packages to install CUDA drivers because you may upgrade the packages if you install
with distribution-specific packages methods, which will cause huge problem afterwards.

First, download the runfile suitable of your system from [here](https://developer.nvidia.com/cuda-downloads).

Then type the following command to run the runfile:
`sudo sh cuda_<version>_linux.run --override`

After finishing the installation, update your bash profile (`.bashrc`) by insert the following lines
at the bottom of the profile:
```
# NVIDIA CUDA Toolkit
export PATH=/usr/local/cuda-10.0/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-10.0/lib64
```

Then source the profile:
```
$ source ~/.bashrc
```

Type `nvcc -V`. It you get the similar output, you're done!
```
$ nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2018 NVIDIA Corporation
Built on Sat_Aug_25_21:08:01_CDT_2018
Cuda compilation tools, release 10.0, V10.0.130
```

5. Install cuDNN
Installing cuDNN is much less tedious than CUDA driver and CUDA toolkit.

In the beginning, download cuDNN v7.6.5 for CUDA 10.0 from [here](https://developer.nvidia.com/rdp/cudnn-archive)
(you may need to login or signup to download cuDNN).

Next, extract the zip file and put the headers and shared libraries:
```
$ tar -zvf cudnn-10.0-linux-x64-v7.6.5.32.tgz
$ cd cuda
$ sudo cp -P lib64/* /usr/local/cuda/lib64/
$ sudo cp -P include/* /usr/local/cuda/include/
$ cd ~
```

If you do not encounter any problem, Congrats! You have finish the hardest part!

### Step #2 Install OpenCV Dependencies
OpenCV has a huge of dependencies, and I write in here so that you just copy and past the following command:
```
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install build-essential cmake unzip pkg-config git \
libjpeg-dev libpng-dev libtiff-dev \
libavcodec-dev libavformat-dev libswscale-dev \
libv4l-dev libxvidcore-dev libx264-dev \
libgtk-3-dev \
libatlas-base-dev gfortran \
python3-dev
```

### Step #3 Download OpenCV Source Code
Currently there is no python wheels or dpkg packages that built with Nvidia GPU support. So we 
have to compile OpenCV from source.

I will use `git` to download the source code so that I can change the version I like.
```
$ cd ~
$ mkdir opencv
$ cd opencv
$ git clone https://github.com/opencv/opencv.git
$ git clone https://github.com/opencv/opencv_contrib.git
$ cd opencv
$ git checkout 4.2.0
$ cd ..
$ cd opencv_contrib
$ git checkout 4.2.0
$ cd ../opencv
```

## to be continued ...

{% if site.liker_id %}
<iframe
  style="width: 100%; max-width: 485px; height: 240px; margin: auto; overflow: hidden; display: block;"
  src="https://button.like.co/in/embed/{{site.liker_id}}/button?referrer={{ page.url | absolute_url | cgi_escape }}">
</iframe>
{% endif %}
