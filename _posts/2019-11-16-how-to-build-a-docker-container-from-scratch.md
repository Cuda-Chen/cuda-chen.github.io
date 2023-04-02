---
layout: base
title: "How to Build a Docker Container from Scratch"
categories: [programming]
tags: [Mask R-CNN, machine learning, IT, Docker]
---

# How to Build a Docker Container from Scratch

> TL, DR: The docker of this article is hosted on [my GitHub](https://github.com/Cuda-Chen/mask-rcnn-docker) and [DockerHub](https://hub.docker.com/repository/docker/cudachen/mask-rcnn-docker), which works with no problem on [Matterports Mask R-CNN](https://github.com/matterport/Mask_RCNN) and [my Mask R-CNN](https://github.com/Cuda-Chen/Mask_RCNN).

I finally finished the very first workable version of Docker image used on my Mask R-CNN project after some stumbles, 
and I consider I should write something down to memorize this exciting moment. 
So in this article, I will show how to build a Docker image of your own machine learning project by scratch so that 
you won't have to worry about other guys won't know how to set up the environment and make your project deployment 
more easier.

So let's get started!

Before writing the container setting -- `Dockerfile` -- we take a look of the packages needed in this project:

```
numpy
scipy
Pillow
cython
matplotlib
scikit-image
tensorflow
keras>=2.0.8
opencv-python
h5py
imgaug
IPython[all]
```


We then create a `Dockerfile` with the following content:

```
FROM ubuntu:16.04
MAINTAINER Cuda Chen <clh960524@gmail.com>

ENV TEMP_MRCNN_DIR /tmp/mrcnn
ENV TEMP_COCO_DIR /tmp/coco
ENV MRCNN_DIR /mrcnn

# Supress warnings about missing front-end. As recommended at:
# http://stackoverflow.com/questions/22466255/is-it-possibe-to-answer-dialog-questions-when-installing-under-docker
ARG DEBIAN_FRONTEND=noninteractive

# Essentials: developer tools, build tools, OpenBLAS
RUN apt-get update && apt-get install -y --no-install-recommends \
    apt-utils git curl vim unzip openssh-client wget \
    build-essential cmake \
    libopenblas-dev

# Python 
#
# For convenience, alias (but don't sym-link) python & pip to python3 & pip3 as recommended in:
# http://askubuntu.com/questions/351318/changing-symlink-python-to-python3-causes-problems
RUN apt-get install -y --no-install-recommends python3.5 python3.5-dev python3-pip python3-tk && \
    pip3 install --no-cache-dir --upgrade pip setuptools && \
    echo "alias python='python3'" >> /root/.bash_aliases && \
    echo "alias pip='pip3'" >> /root/.bash_aliases

# Pillow and it's dependencies
RUN apt-get install -y --no-install-recommends libjpeg-dev zlib1g-dev && \
    pip3 --no-cache-dir install Pillow

# Science libraries and other common packages
RUN pip3 --no-cache-dir install \
    numpy scipy scikit-image matplotlib Cython imgaug

# Jupyter Notebook
#
# Allow access from outside the container, and skip trying to open a browser.
# NOTE: disable authentication token for convenience. DON'T DO THIS ON A PUBLIC SERVER.
RUN pip3 --no-cache-dir install jupyter && \
    mkdir /root/.jupyter && \
    echo "c.NotebookApp.ip = '*'" \
         "\nc.NotebookApp.open_browser = False" \
         "\nc.NotebookApp.token = ''" \
         > /root/.jupyter/jupyter_notebook_config.py
EXPOSE 8888

# TensorFlow 1.13.1 - CPU
RUN pip3 install --no-cache-dir --upgrade tensorflow==1.13.1

# Expose port for TensorBoard
EXPOSE 6006

#
# OpenCV 3.4.1
#
# Dependencies
RUN apt-get install -y --no-install-recommends \
    libjpeg8-dev libtiff5-dev libjasper-dev libpng12-dev \
    libavcodec-dev libavformat-dev libswscale-dev libv4l-dev libgtk2.0-dev \
    liblapacke-dev checkinstall
# Get source from github
RUN git clone -b 3.4.1 --depth 1 https://github.com/opencv/opencv.git /usr/local/src/opencv
# Compile
RUN cd /usr/local/src/opencv && mkdir build && cd build && \
    cmake -D CMAKE_INSTALL_PREFIX=/usr/local \
          -D BUILD_TESTS=OFF \
          -D BUILD_PERF_TESTS=OFF \
          -D PYTHON_DEFAULT_EXECUTABLE=$(which python3) \
          .. && \
    make -j"$(nproc)" && \
    make install

#
# Keras 2.2.4
#
RUN pip3 install --no-cache-dir --upgrade h5py pydot_ng keras==2.2.4

# PyCocoTools
#
# Using a fork of the original that has a fix for Python 3.
# I submitted a PR to the original repo (https://github.com/cocodataset/cocoapi/pull/50)
# but it doesn't seem to be active anymore.
RUN pip3 install --no-cache-dir git+https://github.com/waleedka/coco.git#subdirectory=PythonAPI

#ENV TEMP_MRCNN_DIR /tmp/mrcnn
#ENV TEMP_COCO_DIR /tmp/coco
#ENV MRCNN_DIR /mrcnn

# NOTE: cloning my Mask R-CNN master (might be unstable HEAD)
RUN git clone https://github.com/Cuda-Chen/Mask_RCNN.git $TEMP_MRCNN_DIR

RUN git clone https://github.com/waleedka/coco.git $TEMP_COCO_DIR

RUN cd $TEMP_MRCNN_DIR && \
    python3 setup.py install

RUN cd $TEMP_COCO_DIR/PythonAPI && \
    sed -i "s/\bpython\b/python3/g" Makefile && \
    make

RUN mkdir -p $MRCNN_DIR/coco

WORKDIR "/root"
CMD ["/bin/bash"]

```

As the basis of OS, I choose Ubuntu 16.04, and most of the packages used in my 
Mask R-CNN project are written pretty straightforward in `Dockerfile`.

One package I have to mentioned specifically is `opencv` because I would like 
to to the package from source.

After installing required packages, I clone the repo of my Mask R-CNN and 
build some binaries of it.

At the end of `Dockefile`, I set the working directory to `/root` and start `bash`.

At last, I push the Dockerfile repo to my GitHub and link the repo with Docker Hub 
to build the Docker image automatically 
(you can see how to link your GitHub repo with Docker Hub from [here](https://docs.docker.com/docker-hub/builds/link-source/)).

That's all, thanks for your reading.

If you feel this project helpful remember to star [my Docker image project](https://github.com/Cuda-Chen/mask-rcnn-docker).

If you still don't get how to build your own machine learn Docker image or 
find some mistake or some suggestions in this article, please E-mail me to let me know :)
