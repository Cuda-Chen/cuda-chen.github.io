---
layout: post
title: "Switch Your Jekyll Blog to Google Analytics 4 Simplified"
category: [blogging]
tags: [Jekyll, Minima(Jekyll)]
---

## Introduction
As a reminder from Google [^1]. Google Analytics will be replaced
by Google Analytics 4 after July 1st, 2023. As a user of Google
Analytics, I write down the switching procedures so that other
user can have a post to know how to switch your Jekyll blog
from Google Analytics to Google Analytics 4.

## Switching Procddures
1. Create a Google Analytics 4 Property.
For more information you can visit [the help center of Google](https://support.google.com/analytics/answer/9304153?hl=en).

2. Record your measurement ID.
This [answer](https://support.google.com/analytics/answer/9539598)
will let you find your measurement ID.

3. Put your measurement ID in `_config.yml`.
Usually you put your measurement ID like this:
```
google_analytics: <your-measurement-id>
```

4. (Minima and GitHub Pages only) use remote theme.
For [some reasons](https://github.com/jekyll/minima/issues/561),
you have to use remote theme if your Jekyll blog uses minima
theme and is hosted on GitHub Pages.
Usually you set your blog to use remote theme like this:
```
# _config.yml

- theme: minima
+ remote_theme: jekyll/minima

  plugins:
+ - jekyll-remote-theme
```

## References
[^1] https://support.google.com/analytics/answer/10089681?hl=en
