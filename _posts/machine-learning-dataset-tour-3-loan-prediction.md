---
layout: post
title: "Machine Learning Dataset Tour (3): Loan Prediction"
category: [machine learning]
tags: [machine learning dataset, random forest, loan prediction]
---

In this post, I am going to make a brief introduction of loan prediction
dataset, and I will share my solution with some explaination.

## Brief Introduction of Loan Prediction Dataset
Provided by Analytics Vidhya, the loan prediction task is to dicide
whether we should approve the loan request according to their status. Each
record contains the following variables with description:
> Variable              Description
> Loan_ID               Unique Loan ID
> Gender                Male/Female
> Married               Applicant married (Y/N)
> Dependents            Number of dependents
> Education             Applicant Education (Graduate/ Under Graduate)
> Self_Employed         Self employed (Y/N)
> ApplicantIncome       Applicant income
> CoapplicantIncome     Coapplicant income
> LoanAmount            Loan amount in thousands
> Loan_Amount_Term      Term of loan in months
> Credit_History        credit history meets guidelines
> Property_Area         Urban/ Semi Urban/ Rural
> Loan_Status           Loan approved (Y/N)

For more details, you can visit the official [post](https://datahack.analyticsvidhya.com/contest/practice-problem-loan-prediction-iii/).

## Goal and Evaluation Metric
### Goal
- The goal is to approve/decline a person's loan by given condition.

### Evaluation Metric
- accuracy
    - percentage of loan approval you correctly predict.

## My Solution

