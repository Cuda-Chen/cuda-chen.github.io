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
we can take the models trained by other deep learning frameworks and perform inference using OpenCV
with incredible speed on CPU.

However, the DNN module does not support Nvidia GPU until now. In Google Summer of Code (GSoC) 2019,
OpenCV DNN module finally [supports Nvidia GPU](https://summerofcode.withgoogle.com/archive/2019/projects/6169918255398912/)
 for inference with the work of [Davis King](https://github.com/davisking) -- the creator of [dlib](https://github.com/davisking/dlib) --
and [Yashas Samaga](https://github.com/YashasSamaga), and this work was made in public in [version 4.2.0](https://github.com/opencv/opencv/wiki/ChangeLog#version420).
As a programmer expertised with OpenCV,
I think I should share this exciting news in the first moment. Besides, I will teach you how to compile and install 
OpenCV with the utility of Nvidia GPU for deep neural network inference, and I will provide the minimal workable
example code in both C++ and Python.

## Menu with Compiling OpenCV with CUDA and cuDNN-enabled DNN Module
### Assumptions
I assume you are using the following settings:
1. A Nvidia GPU (of course!).
2. Ubuntu 18.04 (or other equivalent Debian-based distro).
3. CUDA 10.0 and corresponding version of cuDNN (in here I use v7.6.5).
4. Your system account has `sudo` privilege.
5. OpenCV version 4.2.0.

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
Install CUDA driver by typing the following commands:
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

After finishing the installation, update your bash profile (`~/.bashrc`) by insert the following lines
at the bottom of the profile:
```
# NVIDIA CUDA Toolkit
export PATH=/usr/local/cuda-10.0/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-10.0/lib64:$LD_LIBRARY_PATH
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

If you are here and do not encounter any problem, Congrats! You have finish the hardest part!

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

In this tutorial, I put both `opencv` and `opencv_contrib` repository in `~/opencv` directory.
I will use `git` to download the source code so that I can change the version I like:
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

### Step #4 Configure Python Virtual Environment 

For developping with Python, it is a good practice to use virtual environment due to using
different versions of Python libraries in isolation and causing less problem in production
environment.

In this part, I will use `virtualenv` and `virtualenvwrapper` as the virtual environment.

1. Install `virtualenv` and `virtualenvwrapper` using `pip`.
```
$ sudo pip install virtualenv virtualenvwrapper
```

After installing these two packages, you need to add these lines in `~/.bashrc` in order to
let bash load virtualenv and virtualenvwrapper each time when terminal is up:
```
# virtualenv and virtualenvwrapper
export WORKON_HOME=$HOME/.virtualenvs
export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
source /usr/local/bin/virtualenvwrapper.sh
```

Then reload you `~/.bashrc` to let the settings activate immediately:
```
$ source ~/.bashrc
```

2. Create a virtual environment.
The first step is to create the virtual environment:
```
$ mkvirtualenv opencv_cuda -p python3
```
This command will create a virtual environment called `opencv_cuda` with Python 3. After the
creation, your current working virtual environment should be `opencv_cuda`.

**NOTE**: If you ever close your terminal or deactivate the virtual environment, you can re-activate
it by typing:
```
$ workon opencv_cuda
```

Because OpenCV Python will use `numpy`, we then install `numpy`:
```
$ pip install numpy
```

### Step #5 Determine Your CUDA Architecture Version
As a experienced CUDA programmer, determining the CUDA architecture version is a required practice
because it lets the compiler generate more efficient code on your GPU. Furthermore, setting architecture 
params which does not include the architecture of your Nvidia GPU will let your program not working
while executing.

We can use `nvidia-smi` to figure out what model of your Nvidia GPU is:
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

You can see that I am using an **Nvidia Geforce GTX 1080 GPU** written in the Name section. 
Please make sure you have *verfied your GPU model* by running `nvidia-smi` before you continue the next part.

After you get the model of Nvidia GPU, you can find your CUDA Architecture using this page:
> https://developer.nvidia.com/cuda-gpus
>

Scroll down to the Paragraph of "Your GPU Compute Capability". As I'm using GTX 1080, I will click on
the "CUDA-Enabled GeForce and TITAN Products" sections.

After examining it, I realize my Nvidia GPU architecture version is `6.1`. As a reminder, your GPU
architecture version may vary.

Once you got the GPU architecture version, *leave a note of it* because we will use it on the next
step.

### Step #6 Configure OpenCV with Nvidia GPU Support
OpenCV uses CMake to configure and generate the build. First of all activate the `opencv_cuda`
virtual environment:
```
$ workon opencv_cuda
```

Then, change directory to the location you cloned the OpenCV source code (e.g., `~/opencv`, and then 
create a build directory (we use out-of-source building):
```
$ cd ~/opencv
$ cd opencv
$ mkdir build && cd build
```

Next, run the following `cmake` command, and **change the `CUDA_ARCH_BIN` variable you wrote down 
in step #5**:
```
$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D INSTALL_PYTHON_EXAMPLES=ON \
-D INSTALL_C_EXAMPLES=OFF \
-D OPENCV_ENABLE_NONFREE=ON \
-D WITH_CUDA=ON \
-D WITH_CUDNN=ON \
-D OPENCV_DNN_CUDA=ON \
-D ENABLE_FAST_MATH=1 \
-D CUDA_FAST_MATH=1 \
-D CUDA_ARCH_BIN=6.1 \
-D WITH_CUBLAS=1 \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv/opencv_contrib-4.2.0/modules/ \
-D HAVE_opencv_python3=ON \
-D PYTHON_EXECUTABLE=~/.virtualenvs/opencv_cuda/bin/python \
-D BUILD_EXAMPLES=ON
```

For one more point, check the `install path` in `Python 3` section of CMake output. We will use
`install path` in step #8. **So please leave a note of `install path`**.
```
--   Python 3:
--     Interpreter:                 /home/cudachen/.virtualenvs/opencv_cuda/bin/python3 (ver 3.6.9)
--     Libraries:                   /usr/lib/x86_64-linux-gnu/libpython3.6m.so (ver 3.6.9)
--     numpy:                       /home/cudachen/.virtualenvs/opencv_cuda/lib/python3.6/site-packages/numpy/core/include (ver 1.18.1)
--     install path:                lib/python3.6/site-packages/cv2/python-3.6
```

### Step #7 Compile OpenCV
If `cmake` exited with no errors, you then compile OpenCV with the following command:
```
$ make -j$(nproc)
```

### Step #8 Install OpenCV with CUDA DNN Module
If `make` completed in success, you the type the following commands to install OpenCV:
```
$ sudo make install
$ sudo ldconfig
```

Then, we are going to create a sym-link the OpenCV Python bindings into your Python virtual environment. 
Mentioned in Step #6, we know that the `install path` is `/usr/local/lib/python3.6/site-packages/cv2/python-3.6`.

To confirm, you can use the `ls` command:
```
$ ls -l /usr/local/lib/python3.6/site-packages/cv2/python-3.6
...
```

You can see the name of my OpenCV Python bindings is `cv2.cpython-36m-x86_64-linux-gnu.so`
(you may have the similar name of your own built bindings).

Next, create a sym-link to your virtual environment:
```
$ ln -s /usr/local/lib/python3.6/site-packages/cv2/python-3.6 ~/.virtualenvs/opencv_cuda/lib/python3.6/site-packages/cv2.so
```

Remember to take a second to check your file paths because `ln` will *slient fail* if the path
of OpenCV bindings are not correct.

## Verify the Installation
### C++
We can verify the installation is successful with two means:
1. The program compiles with OpenCV in no problem. 
2. The program executes with no errors.

#### Steps to Verify, C++ Part
1. Download the [repo](https://github.com/Cuda-Chen/opencv-dnn-cuda-test) and the weights mentioned in README.
2. Go to `cpp_code` directory and type this command: 
`g++ -o opencv_dnn_cuda_test_cpp main.cpp -I/usr/local/include/opencv4 -lopencv_core -lopencv_dnn -lopencv_highgui -lopencv_imgcodecs`
3. Run the executable by using this command: `opencv_dnn_cuda_test_cpp`
4. If terminal outputs similar message, you're done!
```
$ ./opencv_dnn_cuda_test_cpp 
Time per inference: 14 ms
FPS: 71.4286
```


### Python
We can verify the installation is successful with two means:
1. We can import OpenCV in Python script.
2. We are able to use Nvidia GPU via the DNN module.

#### Steps to Verify, Python Part
1. Download the [repo](https://github.com/Cuda-Chen/opencv-dnn-cuda-test) and the weights mentioned in README.
2. Activate the virtual environment (that is, `opencv_cuda`).
3. Go to `python_code` directory and type the command: `python main.py`
4. If terminal outputs similar message, you're done!
```
$ python main.py 
Time per inference: 14.480803 ms
FPS:  69.05694381124881
```

## Recap
In this post, I teach you how to install OpenCV with CUDA-enabled DNN modules from scratch on Ubuntu 18.04. Also, I provided
a minimal sample code both in C++ and Python so that you can adapt them in the later projects for your convenience.

To use CUDA as the backend of OpenCV DNN module, you can simply add these two lines after you load the pre-trained model:
```
// C++
net.setPreferableBackend(cv::dnn::DNN_BACKEND_CUDA);
net.setPreferableTarget(cv::dnn::DNN_TARGET_CUDA);

...

# Python
net.setPreferableBackend(cv.dnn.DNN_BACKEND_CUDA)
net.setPreferableTarget(cv.dnn.DNN_TARGET_CUDA)
```

## Special Thanks
I would like to give a big gratitude to [this post](https://www.pyimagesearch.com/2020/02/03/how-to-use-opencvs-dnn-module-with-nvidia-gpus-cuda-and-cudnn/) on pyimagesearch.com. Without this post, I would not complete this post in simplicity.

I would also like to give specia thanks to [YashasSamaga](https://github.com/YashasSamaga), the main contributor of OpenCV DNN module with CUDA.
He also teaches a lot in the issues on OpenCV GitHub repo which helps plenty of people to solve the problems of compilation with OpenCV
CUDA-enabled DNN modules.
