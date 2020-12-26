---
layout: post
title: Create Your GitHub Personal Access Token
category: Version Control System
tags: [Git, GitHub]
---

## Introduction
On Dec. 15, 2020, GitHub released a post talking about they won't accept account
passwords for authenticating Git oprations on GitHub after Aug. 13, 2021. [1] 
As a consequences, GitHub provides the following two methods for authentication:
1. A personal access token over HTTPS.
2. SSH key over SSH.

As a person perfer using HTTPS (I switch working computer frequently, for instance,
I use one computer at my workspace, and use another computer at home). What's more
GitHub recommends HTTPS method because of these reasons:
1. **SSH key, when being stolen, will cause a huge problem.** Not only replacing
it is cumbersome, but also may let you cannot use the service for a while or permenantly
for the reason of all authentication rely on SSH key. It your SSH key is stolen and
you don't have other authentication methods, then you have to contact the 
adminstrator or even you can't login the service forever (such as AWS EC2 machine).
2. **Some firewalls in your organization will block SSH connections.** For security
reasons, some companies block SSH connections, but they usually don't block HTTPS
connections as you have to use HTTPS to browse the website.
3. **You can revoke the Person Access Token (PAT) at anytime with almost no cost.**

## GitHub Person Access Token Creation Procedures
Long story short, here are the procedures to create a GitHub Person Access Token [2]:
1. In the upper-right corner of any page, click your profile photo, then click **Settings**.
2. In the left sidebar, click **Developer settings**.
3. In the left sidebar, click **Personal access tokens**.
4. Click **Generate new token**.
5. Give your token a descriptive name.
6. Select the scopes, or permissions, you'd like to grant this token. 
We want to use token to access repositories, select **repo**.
7. Click **Generate token**.
8. Copy the token to your clipboard. 
As a reminder,if you navigate off the page (such as closing page), you **will not** be able to see the token again.

## How to Use Personal Access Token in Command Line
You use PAT in command line as follows:
```
$ git clone https://github.com/username/repo.git
Username: your_username
Password: *your_token*
```

You can copy-and-paste the PAT in Password field instead.

## How to Save Personal Access Token
Entering the PAT is tedious, so here are two typical approaches for saving PAT:
1. **use `gitcredentials`** [3] 
As this method varies on OS, visit [4] to get more information.
2. **use `curl`**
As I don't fully get idea about this, visit [5] to get further information.

Nevertheless, I come up with the following method:
1. Use `gitcredentials` to save the PAT.
2. Open `~/.gitcredentials` (usually `gitcredentials` saves credentials in this file)
and you will seethis line:
```
https://<your_username>:<your_token>@github.com/<your_username>
```
You can copy-and-paste `<your_token>` to password field when GitHub prompts you to enter password.

## Recap
In this post, I teach you how to create your GitHub Personal Access Token. Moreover,
I show a trick how to save your token and copy-and-paste the token as password.

## References
[1] https://github.blog/2020-12-15-token-authentication-requirements-for-git-operations/

[2] https://docs.github.com/en/free-pro-team@latest/github/authenticating-to-github/creating-a-personal-access-token

[3] https://git-scm.com/docs/gitcredentials

[4] https://docs.github.com/en/free-pro-team@latest/github/using-git/caching-your-github-credentials-in-git

[5] https://stackoverflow.com/questions/18935539/authenticate-with-github-using-a-token

[6] https://stackoverflow.com/questions/46645843/where-to-store-the-personal-access-token-from-github
