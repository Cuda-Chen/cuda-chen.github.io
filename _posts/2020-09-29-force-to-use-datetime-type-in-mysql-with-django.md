---
layout: post
title: Force to Use DATETIME Type in MySQL with Django
category: [backend]
tags: [Django, backend, MySQL]
---

## TL;DR
- If you have to work on MySQL with version lower than 5.6, you
can change `DATETIME(6)`, which is default when using `DateTimeField()`
in Django ORM, into `DATETIME`, with the following two lines:
```python
from django.db.backends.mysql.base import DatabaseWrapper
DatabaseWrapper.data_types['DateTimeField'] = 'datetime
```
- Drawback: not able to record datetime to milliseconds.

## The Way I Solve the Problem
So here's the plot, I was dedicating myself on a Django project
recently. The database of this project is MySQL 5.1 due to
historical reason. Everytime when I tried to run migration, I
always receive the following error on Django built-in tables:
```python
pymysql.err.ProgrammingError: (1064, "You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '(6) NOT NULL) ' at line 1")
```

Therefore, I read the Django document about supporting database,
and I got a frustrating message [1]:
> Django supports MySQL 5.6 and higher.
>

However, when I started to work on the project today, I came across
an idea: "`(6) NOT NULL` seems like a type using in SQL. So what type
is it actually?" I then used `sqlmigrate` command to check what 
happened when running migrations of Django built-in tables under-the-hood.
Surprisingly, the `(6) NOT NULL` which caused error is actually
`DATETIME(6) NOT NULL`.

After searching on stackoverflow, I found a thread discussing about this [2],
and more interesting part is that you can patch `DATETIME(6)` to `DATETIME`
by this in your `settings.py`:
```python
from django.db.backends.mysql.base import DatabaseWrapper
DatabaseWrapper.data_types['DateTimeField'] = 'datetime
```
The main drawback is that you cannot record your datetime to milliseconds.
Yet, I am delighted because I don't need to upgrade the database.
Moreover, I don't need to record my datetime into such precision.

## Reference
[1] https://docs.djangoproject.com/en/3.1/ref/databases/#mysql-notes

[2] https://stackoverflow.com/questions/52730817/django-migrate-error-mysql-exceptions-programmingerror-1064-you-have-an-err
