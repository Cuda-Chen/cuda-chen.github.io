---
layout: page
title: "A Detailed Guide of YOLO on OpenCV Python"
categories: [programming]
tags: [OpenCV, machine learning, YOLO, Python]
---

# Introduction
When I was undergoing internship in Weeview, it was the first I
heard OpenCV. With the help of OpenCV, I wrote the code of barrel 
distortion, camera calibration, and video pip program. (you can 
see [here](https://github.com/Cuda-Chen/lens-distortion/tree/master/LD), [here](https://github.com/Cuda-Chen/camera-Calibration), and [here](https://github.com/Cuda-Chen/lens-distortion/tree/master/0percent) on my GitHub)
Though I spent some time on how to mastering it and even being "notice"
that I was totally not on schedule by my menter (such embarrassing ...),
I eventually learned how to use OpenCV and felt how powerful it is.
You can integrate OpenCV to your existing C++ project with its
noticeable function with fast processing speed. What's more,
by glancing its source code, you can realize both theories of
each image processing method and the implementation.

As time passes, OpenCV comes with plenty of features, and there
is no exception with deep learning. Begin from version 3.4, OpenCV
gradually adds the features of deep learning inference. However, it
still doesn't provide training a deep learning model via its API called `DNN`.
So usually creating a deep learning model with OpenCV will exists 
the following workflow:
<center>train your model with other framework -> load trained model with OpenCV `readNet`-like function -> make prediction</center>

YOLO is a well-known object detection neural network architecture,
and I am quite impressed pjreddie can code such network with C and
his will. Nevertheless, we convert the weight file commonly in
practice because our boss says we have to use certain programming
language. In addition, I have heard that when inferencing YOLO
using OpenCV `DNN` module is much faster than using pjreddie or AlexeyAB's verions (see [here](https://www.learnopencv.com/deep-learning-based-object-detection-using-yolov3-with-opencv-python-c/)),
especially on CPU. Summing up, I write this article as a record
of inferencing YOLO using OpenCV `DNN` model and make an experiment
of inferencing time comparison between pjreddic, AlexeyAB, and OpenCV.

# Goal of This Article
In this article, I am going to:
1. Make an example of fish YOLO object detection on OpenCV (you can 
copy and paste my code at will on your custom object detection work).
2. Make an execution time experiment between pjreddid, AlexeyAB, and OpenCV YOLO inference.

# Prepare
1. Clone my repo from [here](https://github.com/Cuda-Chen/fish-opencv-yolo-python).
2. Download the pretrained weights from [my Google Drive](https://drive.google.com/file/d/1L6JgzbFhC7Bb_5w_V-stAkPSgMplvsmq/view?usp=sharing) and put it to `yolo-fish`
directory.
3. Create `conda` virtual environment and install the dependencies:
```
$ conda create -n fish-opencv-yolo-python python=3.6 pip 
$ conda activate fish-opencv-yolo-python
$ pip install -r requirements.txt
```
4. Activate the virtual environment.
5. Run prediction of `七星斑.jpg` with this command: `$ python yolo.py --image ./images/七星斑.jpg --yolo yolo-fish`

You should get a pop-up window if there is no any problem:
![image alt](/assets/images/2019/11/29/2019-10-29-00.jpg)

# Explanations
Of course, you would like to know how to use OpenCV for YOLO object detection.
In this section, I will write the keypoints one-by-one.

## read weights

## create blog (tensor)

## non-maxima supression

# Inferencing time Comparison
In introduction section I have mentioned YOLO object detection runs much faster than darknet written by
pjreddie and AlexeyAB. Therefore, I would like to make an experiment to show you this fact is true.

## Specification
The hardware specification is listed below:
- CPU: Core i5-3230M
- RAM: 16GB

And I list the software specification here:
- OS: CentOS 7.6
- OpenCV
    - version: 4.1.0
    - donloaded from `pip`
- pjreddie's darknet
    - compiled without OpenMP
    - compiled *with* OpenMP
- AlexeyAB's darknet
    - compiled without OpenMP and AVX
    - compiled *with* OpenMP
    - compiled *with* AVX 
        - I don't test this because the compiled program gives me a seg fault.

> You can find pjreddie and AlexeyAB's work in the following link:
> pjreddie: https://github.com/pjreddie/darknet
> AlexeyAB: https://github.com/AlexeyAB/darknet

# Conclusion
In this article, I demonstrate how to do custom data YOLO object detection with OpenCV via
an example of fish object image. I also show some outlines when using YOLO object detection
with OpenCV. Finally, I made a inferencing time comparison and show the OpenCV version runs
the fastest.

# Trivia

# Special Thanks
This article would not appear if [pyimagesearch](https://www.pyimagesearch.com/) did not write an awesome
article about teaching you how to use OpenCV for YOLO object detection. You can find his work on this
[article](https://www.pyimagesearch.com/2018/11/12/yolo-object-detection-with-opencv/).
