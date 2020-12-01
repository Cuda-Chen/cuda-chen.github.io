---
layout: post
title: Django Async Worker
category: [backend]
tags: [Django, backend, Gunicorn]
---

## Scenario
Recently, I have been working on a project using Django as an API server,
and there is one special API which **generate a file with arbitraty
generating time and size**. When calling this API, I always got
a 502 error. In the beginning, I thought it might be the problem of
timeout setting, so I changed the timeout settings of both Django
and Apache (using as reverse proxy) to 300 seconds. However, I
still received 502 error.

Then I started to investgate the casue and realized the meaning of
worker setting of Gunicorn. After I altered the worker type from
`sync`, which is the default type, to asynchronous worker, this
particular API call acts normally not only its responses but also
its execution speed.

In order to let both myself and others to realize the meaning of
Gunicorn workers, I wrote this post as a reminder.
