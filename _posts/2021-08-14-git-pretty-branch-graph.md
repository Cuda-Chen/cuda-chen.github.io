---
layout: post
title: How to Show Git Branch Graph in Terminal
category: Git
tags: [Git]
---

Sometimes I would like to watch the history of my Git commits
as well as the branches. What's more, I would like to see
these changes in terminal so that I don't need to install
then execute other programs.

As a Git user, I can watch the Git commits by typing
`git log`. However, sometimes I want to watch the branch
graph so that I can know which branch merges to another
branch.

As such, you can type this command:
```
$ git log --all --decorate --oneline --graph
```

And this stackoverflow answer [^1] provides an interesting
rhythm -- A DOG -- and a meme to memorize it:

![git a dog meme](/assets/images/2021/08/14/adog_meme.jpeg)
<center>This delighted dog with rainbow background will help you to memorize A DOG command :)</center><br/>

Thanks you stackoverflow, you save me the day!

## Reference
[^1] https://stackoverflow.com/a/35075021
