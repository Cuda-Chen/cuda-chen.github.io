---
layout: post
title: "Add Google Analytics to Jekyll Minima Theme Simplified"
category: [blogging]
tags: [Jekyll, Minima(Jekyll)]
---

So you want to get some statistics about your blog, but you don't want to
install any plugins because you are too lazy and afraid of security issues.
What's more, you are using static site generator, which means many of items
are just files.

As a matter of fact, Google Analytics lets you get the statistics of your site by
just adding a code called Tracking ID.

So let's get started!

First [create an account of Google Analytics](https://marketingplatform.google.com/about/analytics/).

Then get your Tracking ID of Google Analytics. You can see this [post](https://support.google.com/analytics/thread/13109681?hl=en)
how to get your Tracking ID.

Next, add the following lines in your `_config.yml`:
```
# Google Analytics
google_analytics: UA-XXXXXXXXX-X
```
Remember to change the `UA-XXXXXXXXX-X` to your Tracking ID.

That's all!

## Reference
https://letianfeng.github.io/github/2018/05/27/github_pages_and_google_analytics.html
