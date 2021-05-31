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

![title image](/assets/images/2021/05/31/django-mysql-github-actions-title.png)

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

As such, there are a lot of communities such as Julia Language [^6] switch their
CI/CD workflows onto GitHub Actions. What's more, these communities invent
plenty of custom workflows for the ease of themselves and the developers
using their project in the future.

## Apply GitHub Actions on Django with MySQL
In this paragraph, I list the procedures how to apply GitHub Actions to
your Django project.

> You can see full example project in [here](https://github.com/Cuda-Chen/django-mysql-github-actions-demo).

### 1. Create Django project
Frist, create your Django project, namely `example`, by typing:
```
$ django-admin startproject example
```

Then add MySQL settings into `DATABASES` variable in `example/settings.py`:
```python
DATABASES = {
    'default': {
        'ENGINE': os.environ.get('DBENGINE', ''),
        'NAME': os.environ.get('DBNAME', ''),
        'USER': os.environ.get('DBUSER', ''),
        'PASSWORD': os.environ.get('DBPASSWORD', ''),
        'HOST': os.environ.get('DBHOST', ''),
        'PORT': os.environ.get('DBPORT', ''),
    }
}
```

For demo purpose, I use environment variables to store the settings of databases.

### 2. Create Django App: `users`
Create a Django app called `users` by typing:
```
$ django-admin startapp users
```

### 3. Add Unit Tests in `users`
Because we want to run CI, we should add some unit test codes.

Substitude existing code with following codes in `users/tests.py`:
```python
from django.test import TestCase
from django.contrib.auth.models import User

# Create your tests here.

class UserTestCase(TestCase):
    def test_user(self):
        username = 'cudachen'
        password = 'carbotzergling'
        u = User(username=username)
        u.set_password(password)
        u.save()
        self.assertEqual(u.username, username)
        self.assertTrue(u.check_password(password))
``` 

### 4. Add GitHub Actions CI
And here comes the main dish! But before adding GitHub Actions' configuration, here are
some common mistake I encounter as kindly reminders for you:
- **branch** : always make sure which branches you want to trigger GitHub Actions, or
you will be confused why GitHub Actions doesn't working. 
In this case, I would like to trigger GitHub Actions when pull request or push on `main`
branch.
- **env**: as I use environment variables for database settings, make sure to set
environment variables in each step you are going to use database, e.g. database migration
and running unit tests.
- **DB port**: as indicated in [^7], GitHub Actions assign random port for each GitHub
Actions services defined by you. 
In order to access these services (e.g. database) port with no failure, you have to use 
`jobs.<job_id>.services.<service_id>.ports` syntax.

Then, here are the steps adding GitHub Actions configuration:
1. Create directory called `.github/workflows`.
2. In `.github/workflows` directory, add `django-ci.yml` (our GitHub Actions configuration)
as below:

```yaml
name: Django CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:

    runs-on: ubuntu-latest
    strategy:
      max-parallel: 4
      matrix:
        python-version: [3.7]

    services:
      mysql:
        image: mysql:5.7
        env:
          MYSQL_ROOT_PASSWORD: zergling
          MYSQL_DATABASE: mysql
        ports: ['3306:3306']

    steps:
    - uses: actions/checkout@v2
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v2
      with:
        python-version: ${{ matrix.python-version }}
    - name: Install Dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Run Migrations
      run: python manage.py migrate
      env: 
        DBENGINE: django.db.backends.mysql
        DBNAME: mysql
        DBUSER: root
        DBPASSWORD: zergling
        DBHOST: 127.0.0.1
        DBPORT: ${{ job.services.mysql.ports[3306] }}
    - name: Run Tests
      run: |
        python manage.py test
      env: 
        DBENGINE: django.db.backends.mysql
        DBNAME: mysql
        DBUSER: root
        DBPASSWORD: zergling
        DBHOST: 127.0.0.1
        DBPORT: ${{ job.services.mysql.ports[3306] }}
```

### 5. Push to GitHub and Enjoy!
After the above steps, push our project to GitHub, and GitHub Actions
will start to work!

### Trivia: Add GitHub Actions Badge for Showing CI Status
GitHub Actions provides status badge for showing CI status in ease. 
Usually, you can add the badge in `README.md` like this:
```markdown
[![<your-CI-name>](https://github.com/Cuda-Chen/<your-project-name>/actions/workflows/django-ci.yml/badge.svg)](https://github.com/Cuda-Chen/<your-project-name>/actions/workflows/django-ci.yml)
```

## Conclusion
In this post, I make an introduction how to apply CI/CD pipeline. I also introduce
what is GitHub Actions and its characteristics. I then make a Django demo project
using MySQL in order to show the processes running GitHub Actions, and leave
some marks for avoiding common gotchas. 

## References
[^1] https://www.jenkins.io/ 

[^2] https://travis-ci.org/

[^3] https://github.com/features/actions

[^4] https://github.com/marketplace/actions/github-pages-action

[^5] https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners

[^6] https://julialang.org/

[^7] https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions#example-using-localhost
