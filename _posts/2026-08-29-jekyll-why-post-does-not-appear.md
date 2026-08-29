---
layout: post
title: "Why Your Jekyll Future Post Does Not Appear on GitHub Pages"
category: [Blog]
tags: [Jekyll]
---

Recently, I created a post with a future date (say I am in UTC+08
and the date is 2026/07/01). When I pushed my post to GitHub, the post
did not appear.

After searching the log of GitHub Pages, I found the line of `Skipping ...`
, and I realized the timezone of build server was different to my post.
Therefore, the post did not being generated.

So there are some solutions to let Jekyll build such post (Jekyll
names as 'future post'):

1. Force push in the moment that is larger or equal to the date of build server.
2. Set `--future` to generate future posts.

As I had no options to set the `--future`, I chose 1. and I finally
got the post appear.

Feel free to share your solution of meeting this scenario.
