---
layout: post
title: "CMake Superbuild Deminisfied"
category: programming
tags: [programming, C++, CMake, build system]
---

## Introduction

When compiling external projects as dependencies, it is a nuance to
compile them if your project uses CMake.
To resolve this problem, CMake provides plenty of options 
for you to build your dependencies, and superbuild is one of the options.

## What's Superbuild?

Long story shorts: it is a design pattern to build your dependencies
and your main project. You can specify how to build the dependencies
by plenty of means such as shell script and CMake [^1].

For shell script, it is the most intuitive way, but usually it is
not a cross-platform sotluion (you have to write each shell script
with the corresponding platform).

For the CMake method it provides `ExternalProject_Add()` so that
CMake will fetch then build your dependencies with a cross-platform
ability (you write the pure CMake grammer in your build script).

## Difference between `FetchContent()`

There is another CMake function called `FetchContent()`, and it has
the ability to fetch your dependencies. After that, you can build
the dependencies just like `ExternalProject_Add()`.

Though they have the similiar effect, you can choose one of them
by the following situations:

- choose `FetchContent()` if:
    - The dependency is a CMake project.
    - You need a transitive dependencies resolved: if two or more
projects require a project, `FetchContent()` can prevent downloading
or compiling the needed project twice.
    - You need standard compilation of the needed project.
- choose `ExternalProject_Add()` if:
    - The dependency uses other build system.
    - You need isolation of needed project.
    - Target nameing clashes.

## Conclusion

There are many ways to compile your dependencies such as shell
script and CMake functions such as `FetchContent()` and `ExternalProject_Add()`. 
As an ultimate guide, choose the suitable method according to
the characteristics of your dependencies so that you can have
the least effort to maintain your project.

## References

[^1]: https://www.acarg.ch/posts/cmake-superbuild/
