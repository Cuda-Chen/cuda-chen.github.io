---
layout: post
title: "Brief Introduction of miniSEED and libmseed"
category: [seismology, seismology data format]
tags: [miniSEED, libmseed]
---

## THIS WORK IS IMCOMPLETE

These days, I was working on a project about seismic wave analysis.
For reading a data containing seismic waves, it comes up to several
format, and the one I get in touch frequently is miniSEED. 

miniSEED is famous of its simplexity compared to its superset SEED,
which stands for Standard for the Exchange of Earthquake Data. The
biggest difference between them is that miniSEED only contains a
header -- to carry necessary information about the data -- and 
the values representing the amplitude of the seismic wave. [^1]
As usual, numeruous libraries support to read and write miniSEED
data in ease. For instance, `libmseed` is provided by IRIS team
to let people to maniplulate miniSEED data and create their own
binding on other programming languages.

## Why I Write This Post
1. Seismology is a fresh new area to me, and I think getting
familiar with the data maniplulating tool is a good start for
realizing what seismologists are doing.
2. Compared to the version before 3, `libmseed` made a great
number of changes in order to support the newer version standard
of miniSEED (or miniSEED 3.x). [^2] And writing this post makes 
not only some records what I have done for porting this library but also 
a brief introduction for others to use this work.

## Installation
Installation of `libmseed` will be quite easy, and it has almost
no dependencies (strictly, `glibc`).
For Unix-like system, type these commands will be sufficient:
```
$ make
$ sudo make install # to install libmseed so that you can link against to other programs
```
For Windows system type the comand instead:
```
$ nmake -f Makefile.win
```
> Edit: I have no idea how to install libmseed on Windows to
> link against to other programs. Maybe someone can leave some commands
> to make supplement for Windows user.
>

## Linking against to Other Programs
Assume you put `libmseed.h` in `/usr/local/include` and the shared libraries
in `/usr/loca/lib`, you can link `libmseed` in the following way for instance:
```
$ gcc -I/usr/local/include -o main main.c -L/usr/local/lib -lmseed
```

(Hmm... may need to write LD_LIBRARY_PATH?)

## Demo
In this part, I am going to demostrate how to read the infomation of header
and the values of each trace. The demo procedures are shown as follows:
1. Clone the [repo](https://github.com/Cuda-Chen/libmseed-hello-world) written by me.
2. Compile it.
3. Run with the following command:
```
$ 
```

# References
[^1] http://ds.iris.edu/ds/nodes/dmc/data/formats/miniseed/
[^2] https://github.com/iris-edu/libmseed/blob/master/ChangeLog#L62
