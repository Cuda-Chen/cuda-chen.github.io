---
layout: post
title: Django+MySQL CI/CD with GitHub Actions
category: [DevOps]
tags: [DevOps,
       Django,
       MySQL,
       CI,
       CD,
       GitHub Actions]
---

## Introduction
In nowdays, when we develop a web application, we usually apply CI (Continuous
Integration) and CD (Continous Deployment) to automate the processes of testing
and deployment.

To run such automations, we use Jenkins [^1] and Travis [^2] in the old days.
However, in this decade, there are lots of tool popping up for our needs, and
one of them is GitHub Actions. As a web developer and GitHub lovers, I would
like to use GitHub Actions for not only can be easily integrated with GitHub
projects but also show off my abaility to customize GitHub Actions [^3]
for my needs. Therefore, I would like to share my note apply GitHub Actions (especially CI)
on an example Django project using MySQL as its database.

## Introduction of GitHub Actions
Just like other CI/CD platform, GitHub Actions is a yet-another platform for
CI/CD. However, it has the following features:
1. **Highly integrated with GitHub Services.**
GitHub Actions can be invoked with arbitrary GitHub events. Therefore,
if you host your repos on GitHub, you can consider to use GitHub Actions
for better development experiences.
2. **Community-powered workflows.**
GitHub Actions lets you to create your own worflow and publish to its marketplace
to share your work with others. For example, once I switch a Crystal project
from Travis to GitHub Actions. The project needs CD of built website with
custom domain and different repo name settings. And the work of 
GitHub Pages action [^4] helped me a lot with painless worflow switch.
3. **Self-hosted machines are permitted.**
You can not only run GitHub Actions on GitHub, but also your self-hosted
machines with much more flexiable configuations and better bargain! And
the official document [^5] has great explainations to teach you how
to achieve this.

As such, there are a lot of communities such as Julia Language switch their
CI/CD workflows onto GitHub Actions. What's more these communities invent
plenty of custom workflows for the ease of themselves and the developers
using their project in the future.

## Apply GitHub Actions on Django with MySQL

## Conclusion

## References
[^1] https://www.jenkins.io/ 

[^2] https://travis-ci.org/

[^3] https://github.com/features/actions

[^4] https://github.com/marketplace/actions/github-pages-action

[^5] https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners

https://github.com/adamchainz/django-mysql
https://blog.healthchecks.io/2020/11/using-github-actions-to-run-django-tests/
https://medium.com/intelligentmachines/github-actions-end-to-end-ci-cd-pipeline-for-django-5d48d6f00abf
