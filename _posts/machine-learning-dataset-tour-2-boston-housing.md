---
layout: post
title: "Machine Learning Dataset Tour (2): Boston Housing"
category: [machine learning]
tags: [machine learning, boston housing, XGBoost, machine learning dataset]
---

In this post, I will make a brief introduction of boston housing dataset, and
I will share my solution with some explanations.

## What's Boston Housing?
The dataset consists of information collected by U.S. Census Service concerning
housing in the area of Boston Mass.

The following show the meaning of each variable (column) in the dataset:
> Variable  Description
> CRIM      per capita crime rate by town
> ZN        proportion of residential land zoned for lots over 25,000 sq.ft.
> INDUS     proportion of non-retail business acres per town.
> CHAS      Charles River dummy variable (1 if tract bounds river; 0 otherwise)
> NOX       nitric oxides concentration (parts per 10 million)
> RM        average number of rooms per dwelling
> AGE       proportion of owner-occupied units built prior to 1940
> DIS       weighted distances to five Boston employment centres
> RAD       index of accessibility to radial highways
> TAX       full-value property-tax rate per $10,000
> PTRATIO   pupil-teacher ratio by town
> B         1000(Bk - 0.63)^2 where Bk is the proportion of blacks by town
> LSTAT     % lower status of the population
> MEDV      Median value of owner-occupied homes in $1000's

For more details, you can visit this [website](https://www.cs.toronto.edu/~delve/data/boston/bostonDetail.html).

## Goal and Evaluation Metric
### Goal
- predict the value of `MEDV`
### Evaluation Metric
- RMSE, i.e.
<img src="https://render.githubusercontent.com/render/math?math=RMSE(y)%20=%20\sqrt{\displaystyle\sum_{i=1}^{n}%20(y_i%20-%20\hat{y_i})^2}">

## My Solution
