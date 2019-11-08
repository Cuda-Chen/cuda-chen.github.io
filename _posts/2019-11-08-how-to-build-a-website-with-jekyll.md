---
layout: default
title: "How to build your Website with GitHub Pages and Jekyll"
---
 
After some struggles, I finally built my personal website with GitHub
Pages and Jekyll. <br>
I write this down because I want to leave as a record, and the OS I am using
is CentOS. Therefore, I will use CentOS as demonstration.

You can following the following steps to build your website:

1. Create a repo named ```<username>.github.io```. <br>
GitHub Pages only works when <username> is *exactly* matched to your username. [1]

2. Clone the repo create in step #1 to your computer.

3. Install ```rvm```. <br>
At first, I wanted to install Ruby via ```yum```, but I found the version
of Ruby installed by ```yum``` is toooooo old. After some searching, I will
recommend you to install ```rvm``` because it not only installs newer version of
Ruby but also provides as a great manager of your Ruby environments, especilly you 
are going to learn Ruby of Ruby on Rails later. Most of all, it gets rid of the problem
of needing ```sudo``` privilege. <br>
So here are the steps when I was installing ```rvm```[2]: <br>
    (1) install GPG keys: <br>
    ```$ gpg2 --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB``` <br>
    (2) install ```rvm```: <br>
    ```$ \curl -sSL https://get.rvm.io | bash -s stable```

4. Install ```jekyll``` and ```bundler```. <br>
```
$ gem install jekyll bundler
```

5. Change into your blog's directory, namely ```<username>.github.io```, and create your website. <br>
```
$ cd <username>.github.io
$ jekyll new .
```

6. Build the website and make it available on a local server. <br>
```
$ bundle exec jekyll serve
```

7. Browse to [http://localhost:4000](http://localhost:4000). <br>
It shows the following page like this if there is no any problem:
![image alt](/assets/images/2019/10/23/2019-10-23-00.png)

8. Push to remote repo mentioned in step #1. <br>

9. Browse your website via ```<username>.github.io```. <br>
Note that your website may not reply when you push your website repo because it may take some time 
for GitHub Pages to build your website in the very first time. <br>
Still, if your website does not appear over 30 minutes, you should try to search your problems because 
it means there is some problems in your website repo.

I hope this post helps you! If you have any question or suggestions to correct this post, please E-mail me 
to let me know :)

## Reference
[1] [https://pages.github.com/](https://pages.github.com/)

[2] [https://rvm.io/](https://rvm.io/)
