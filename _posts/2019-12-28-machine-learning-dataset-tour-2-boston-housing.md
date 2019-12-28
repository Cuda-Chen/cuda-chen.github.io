---
layout: post
title: "Machine Learning Dataset Tour (2): Boston Housing"
category: [machine learning]
tags: [machine learning, boston housing, XGBoost, machine learning dataset]
---

> TL;DR: You can view my work on my [GitHub](https://github.com/Cuda-Chen/machine-learning-note/blob/master/boston-housing/boston_xgboost.ipynb).

In this post, I will make a brief introduction of boston housing dataset, and
I will share my solution with some explanations.

## What's Boston Housing?
The dataset consists of information collected by U.S. Census Service concerning
housing in the area of Boston Mass.

The following show the meaning of each variable (column) in the dataset:

| Variable      | Description                                                           |
|------------   |---------------------------------------------------------------------  |
| CRIM          | per capita crime rate by town                                         |
| ZN            | proportion of residential land zoned for lots over 25,000 sq.ft.      |
| INDUS         | proportion of non-retail business acres per town.                     |
| CHAS          | Charles River dummy variable (1 if tract bounds river; 0 otherwise)   |
| NOX           | nitric oxides concentration (parts per 10 million)                    |
| RM            | average number of rooms per dwelling                                  |
| AGE           | proportion of owner-occupied units built prior to 1940                |
| DIS           | weighted distances to five Boston employment centres                  |
| RAD           | index of accessibility to radial highways                             |
| TAX           | full-value property-tax rate per $10,000                              |
| PTRATIO       | pupil-teacher ratio by town                                           |
| B             | 1000(Bk - 0.63)^2 where Bk is the proportion of blacks by town        |
| LSTAT         | % lower status of the population                                      |
| MEDV          | Median value of owner-occupied homes in $1000's                       |

For more details, you can visit this [website](https://www.cs.toronto.edu/~delve/data/boston/bostonDetail.html).

## Goal and Evaluation Metric
### Goal
- predict the value of `MEDV`
### Evaluation Metric
- RMSE, i.e.
<img src="https://render.githubusercontent.com/render/math?math=RMSE(y)%20=%20\sqrt{\displaystyle\sum_{i=1}^{n}%20(y_i%20-%20\hat{y_i})^2}">

## My Solution
- Though decision tree based model performs awesome on tabular data, I will choose linear regression model for
practice.
    - Specifically, I choose Lasso regression according to sklearn's [cheat sheet](https://scikit-learn.org/stable/tutorial/machine_learning_map/index.html).
    - I will try linear regression in later days, stay tuned!
    - I will also demonstrate using less features to produce akin result in later days.
- Because there is no any null value, we need not to fill or drop the missing value.
- Since the testing data contains no ground truth, we will train the model with cross validation.

## Step-by-stop Explaination
### 1. Prepare Data and Make Some Explainations
Using pandas' `info()` method, we find all features are numbers, so there is no need for one-hot encoding.

Using pandas' `isnull()` method, we realize there is no any empty value, so no need to fill the empty value.

### 2. Visualization
I would like to find the relationship between each feature and prices, so I draw some plots to visualize the relationship.

As there are a lot of plots, you can see [here](https://github.com/Cuda-Chen/machine-learning-note/blob/master/boston-housing/boston_xgboost.ipynb)
to check out the result for clarity.

### 3. Check Feature Importance
Maybe we don't need to put every feature into the model, so I use XGBoost to find feature importance. <br>
Many of decision tree based models have the ability to find feature importance as they splitting the data by calculating
information gain or Gini coefficient.<br>
After running XGBoost Regressor, we get the following results:
```python
xgbr.get_booster().get_score(importance_type='gain')

# Out:
{'LSTAT': 1170.5912806284505,
 'RM': 407.86173977156585,
 'NOX': 114.57274366695417,
 'CRIM': 48.06237422548619,
 'DIS': 65.78717631524752,
 'PTRATIO': 109.93901440305558,
 'AGE': 26.895764984946428,
 'TAX': 56.70414882315789,
 'B': 30.702484215217385,
 'INDUS': 15.392935187827584,
 'CHAS': 45.74311416166666,
 'RAD': 18.774944386363632,
 'ZN': 9.174343283333334}
```
We can realize the most three important features: `LSTAT`, `RM`, and `PTRATIO`.

### 4. Build Model
Let's build our model with Lasso Regression!<br>
I create the model object with params `alpha=0.1`:
```python
lasso = Lasso(alpha=0.1)
```
I also do a train-test split to train the model, and use splitted test data to produce the predictions.<br>
We then gain `4.48` of `RMSE` value. Let's try whether cross validation can boost up the performance or not.

Scikit-learn provides Lasso Regression with cross validation called `LassoCV()`. After training, we get `4.37`
of `RMSE` value. Hmm, the value lowers.

### 5. Generate the Result of Competition
Let's update the result to Kaggle and we gain score of `5.16976` and rank #4 in leaderboard. You can see the 
leaderboard [here](https://www.kaggle.com/c/boston-housing-dataset/leaderboard).

## Conclusion
In this post, I make a brief introduction about famous Boston Housing Dataset, and I demonstrate step-by-step
procedures of my solution. Furthermore, I show we can use Lasso Regression with cross validation, but the 
performance does not boost up.<br>
See you in the next tour!

> You can view my work on my [GitHub](https://github.com/Cuda-Chen/machine-learning-note/blob/master/boston-housing/boston_xgboost.ipynb).
