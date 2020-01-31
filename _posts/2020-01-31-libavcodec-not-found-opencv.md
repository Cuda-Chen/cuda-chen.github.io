---
layout: post
title: "Solution of libavcodec problem in OpenCV"
category: [programming, image processing]
tags: [programming, opencv, image processing, troubleshooting]
---

I had manually compiled OpenCV version 3.4.2 on my CentOS system, and it worked well until
today. This morning, I received a issue of my [SRCNN-cpp](https://github.com/Cuda-Chen/SRCNN-cpp) project.
After I tried to execute the programming I got the following error message:
`libavcodec.so.56: cannot open shared object file: No such file or directory`. In the beginning,
I thought maybe something wrong of the executable, so I tried to recompile the program.
Surprisingly, I got the compile errors related to `FFmpeg`. I realized I upgraded `FFmpeg` a 
month ago which breaks OpenCV (my `libavcodec.so` is symlinked to `libavcodec.so.57`).

As mentioned in this [thread](https://stackoverflow.com/questions/58247177/updating-ffmpeg-breaks-opencv-library),
I can solve this problem by these two approaches:
1. Recompile OpenCV to link against the latest version of `libavcodec`.
2. Revert the version of `libavcodec` to 56.

As a hardcore programmer, I selected the solution of recompiling. What's more, I wanted to upgrade
my OpenCV to newer version. For compiling OpenCV from source on CentOS, you can see this [article](https://linuxize.com/post/how-to-install-opencv-on-centos-7/).
