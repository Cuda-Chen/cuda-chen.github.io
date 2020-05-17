---
layout: post
title: "Create a FFTW 3 Docker from Scratch"
category: [DevOps]
tags: [DevOps, Docker, FFTW]
---

> TL;DR: You can pull the image from [here](https://hub.docker.com/r/cudachen/fftw-docker).
>

## Motivation
The speed of FFTW is pretty amazing, but the installation is a little
bit complex. What's more, I want to try to build FFTW source code
from its GitHub repo rather than zip file, and I want a well-configured
environment on-the-go. Therefore, I create this project and learn
some basic about Docker.

## Contents of Dockerfile
Long story short, you can the whole Dockerfile [here](https://github.com/Cuda-Chen/fftw-docker/blob/master/Dockerfile).

I will outline some important parts of this Dockerfile.

### Base Image
In the first line, you can see I choose Ubuntu base image.
Though some people argue using such image results in huge image size, I think
I should use this kind of image because it contains standard `glibc`, which
is crucial for FFTW.

### Why FFTW requires OCaml to build from scratch?
In line 15~18, you can see I download some packages including OCaml. The reason 
is that the author uses OCaml to generate "codelets", which are hard-coded code
in order to reach the best performance under certain condition. [1] 

If you have ever taken a course of compiler, you will realize that using C to
implement a small compiler is not only tedious but also requires knowledges about
maintaining AST (Abstract Snytax Tree) [2] even you do the homework with `lex` and
`yacc`. So some cool guys in class will use other languages to create AST such as Haskell and
OCaml with no pain and thus finish the homework perfectly. And guess what? The author
choose latter to create those "codelets" to let FFTW generate optimized C code.

So why I download other four packages? Because while I was trying to build FFTW
in Ubuntu Docker, the container always complained that it can't compile without
those package. At that moment, I realized that a base image usually drops some
packages in order to shrink the image size.

So how to solve the question? Pretty easy. You just read the error, make a note, 
add the missing package before compiling, and loop until you build with no error
(seriously).

### At last, why we need ocaml Num library?
Quite simple. Mentioned in README on FFTW GitHub repo, OCaml removed ocaml Num library
in OCaml 4.06.0 (3 Nov 2017). [3]

The interesting part is that Ubuntu 18.04 does not have such package on APT. As a result,
I manually download it and compile from source.

## Thought of This Project
After finishing the project, I not only realize how to compile FFTW from scratch. What's
more, I make a great practice to create a Docker image meeting my need, and I have a 
chance to dig deeper of this masterpiece of using other lanauges to solve the pitfall
of C.

In the end, hope you gusy enjoying this article!

## Reference
[1] http://www.fftw.org/fftw3_doc/Generating-your-own-code.html

[2] https://en.wikipedia.org/wiki/Abstract_syntax_tree

[3] https://github.com/FFTW/fftw3/blob/master/README
