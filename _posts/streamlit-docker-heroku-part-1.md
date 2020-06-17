---
layout: post
title: "Streamlit-docker-heroku-part-1"
category: [DevOps]
tags: [DevOps, Machine Learning, Streamlit]
---

> Part2 Part3

## Motivation
As time passes, you have done a lot of machine learning projects, but 
you need a visualization or other guys won't know the process and result
of your project. What's worse, some people think that you are not prepared
because you cannot use Docker and PaaS deployment (many of the machine
learning jobs in Taiwan need frontend or backend as well).

## Goal
This series will guide you to:
1. Use a framework to let your machine learning project run in web.
2. Use a Docker to ease the difficulty of environment setting and integrate
with DevOps workflow.
3. Use a PaaS deployment to show off you can deploy your project.

For this part, I will show how to run your streamlit app with OpenCV utility.

## Project Setup
In this series, I use my [fish-yolo-grabcut](https://github.com/Cuda-Chen/fish-yolo-grabcut) as a basis of streamlit app.

## Create a Streamlit App

