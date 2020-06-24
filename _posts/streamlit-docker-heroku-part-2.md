---
layout: post
title: "Show Off Your Machine Learning Project on Web Part 2: Dockerize You Streamlit App"
category: [DevOps]
tags: [DevOps, Machine Learning, Streamlit, Docker]
---

> Part1 Part3

## Motivation
So you finally finish your machine learning project with streamlit for 
presentation, but you fell tedious that you have to create a virtual
environment every time on a new computer. What's worse, sometimes your
virtual environment breaks because you are using different OS.

Hopefully, we can use Docker to ease the problems of setting up and deploying
envirouments, and you create a microservice which is suitable for scaling.

## Goal of This part
For this part, I will show how to create a workable streamlit + OpenCV Docker
Image, and show some simple usage about Docker.

## Dive in with An Example
Long story short, let me show the Dockerfile of my streamlit + OpenCV machine
learning project:
```
FROM ubuntu:18.04
MAINTAINER Cuda Chen <clh960524@gmail.com>

# streamlit-specific commands for config
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8
RUN mkdir -p /root/.streamlit
RUN bash -c 'echo -e "\
[general]\n\
email = \"\"\n\
" > /root/.streamlit/credentials.toml'

RUN bash -c 'echo -e "\
[server]\n\
enableCORS = false\n\
" > /root/.streamlit/config.toml'
# install Python and Pip
#
# NOTE: libSM.so.6 is required for OpenCV Docker
# or you will get seg fault when import OpenCV
RUN apt-get update && \
    apt-get install -y \
    python3.7 python3-pip \
    libsm6 libxext6 libxrender-dev

# expose port 8501 for streamlit
EXPOSE 8501

# make app directiry
WORKDIR /streamlit-docker

# copy requirements.txt
COPY requirements.txt ./requirements.txt

# install dependencies
RUN pip3 install -r requirements.txt

# copy all files over
COPY . .

# download YOLO weights
RUN gdown --output ./yolo-fish/fish.weights --id 1L6JgzbFhC7Bb_5w_V-stAkPSgMplvsmq 

# launch streamlit app
CMD streamlit run app.py 
```

### Some Dockerfile Commands Usage Simplified
1. `FROM`: Get a base image. Just like you need an OS as a basis of your application.
2. `MAINTAINTER`: Show message about the author of this image.
3. `ENV`: set an environment variable to a certain value.
4. `RUN`: execute the commands.
5. `EXPOSE`: to let your container listens on the specific port while running.
6. `WORKDIR`: set the working directory.
7. `COPY`: copy new files/directories.
8. `CMD`: to provide default actions when executing container.

For more information, visit [official Dockerfile reference](https://docs.docker.com/engine/reference/builder/).

### Some reminders about This Dockerfile
1. The purpose of line 13~16 is to let streamlit runs normally in Docker. Otherwise,
streamlit will fail launching.
2. The purpose of line 24 is to let OpenCV runs normally in Docker. I encountered an
issue that if you do not install `libSM.so.6` then import OpenCV when buiding Docker Image, your 
container will sliently crash with this message: `segmentation faule (core dumped)` when
executing. You can see this kind of issue [here](https://github.com/NVIDIA/nvidia-docker/issues/864).

## Build and Execute This Container
Use the following commands to build and execute this container:
```
$ docker image build -t fish-yolo-grabcut:app .
$ docker container run -p 8501:8501 --rm -d grabcut:app
```

After that, type `localhost:8501` in browser's URL, and the content of app will show up.

For shutdowning the container type:
```
$ docker kill <weird id of fish-yolo-grabcut.app>
```

## Warping Up
In this part, I show the Docker configuration of this project, and tell you the problems
when I was building and executing this container.
