---
layout: post
title: "How to Contribute to Linux Kernel"
category: [Programming]
tags: [LinuxKernel, Programming, OpenSourceContribution]
---

Years ago, I ways reading the assignment of Linux Kernel online
lecture lectured by Jserv [^1]. While I was reading the code of seqlock.h,
I found a grammar mistake (should be "neither nor" not "neither and"):

```c
static inline void do_write_seqcount_end(seqcount_t *s)
...
  * neither writes before and after the barrier are enclosed in a seq-writer
...

```

I therefore submitted a patch to Linux Kernel, and I thought
it was not accepted either the patch was nonsence or the maintainers
were too busy. However, I recently felt bored then click the
patch link, I suddenly found it was accepted and merged at least
for one year. Furthermore, I realized the merge to tip [^3]
was the acceptance of my patch.
For sharing my pleasure, I would like to write this post
down as a guide of subbmitting patched to Linux Kernel as
it uses a dedicated way for contribution.

## Tools You Will Need

- `git send-email`
- `mutt`

### `git send-email`

A utility built in Git for sending e-mails.

As Git utilizes SMTP you need to apply the following settings:

```
# in your Git config such as ~/.gitconfig
[sendemail]
        smtpencryption=tls
        smtpserver=smtp.gmail.com
        smtpuser=<your-email-address>
        smtpserverport=587
        smtpPass=<your email token, see Trivia section>
```

### `mutt`

A text-based mail client. Though
I don't actually use such tool as I don't receive any
discussions.

## Submit A Patch

First, download the Linux Kernel tarball from [^4] to save 
the Git cloning time.

After that, create your changes (of course, remember use
`git config` to set your username and e-mail):

```
$ git init
$ git add .
$ git commit -m "origin"
$ git checkout -b my-patch
```

For commit uses:

```
$ git commit -s
```

The `-s` is required by Linux Kernel to represent the changes
are done by yourself.

Then, create the patch:

```
$ git format-patch master
```

Remember to run scripts to satisfy the commit requirements:

```
$ ./scripts/checkpatch.pl <your patch>
```

Next, run the following command to get the list of maintainers
(taking changing docs as instance):

```
$ ./scripts/get_maintainer.pl <your patch>

Jonathan Corbet <corbet@lwn.net> (maintainer:DOCUMENTATION)
linux-doc@vger.kernel.org (open list:DOCUMENTATION)
linux-kernel@vger.kernel.org (open list)

```

Record the list so that all members will receive your patch.

Eventually, send your patch with either `git send-email`:

```
$ git send-email --to corbet@lwn.net \
--cc linux-doc@vger.kernel.org \
--cc linux-kernel@vger.kernel.org \
<your patch>
```

or `mutt`:

```
$ mutt -H ./patch_name.patch

```

## Trivia

For certain E-mail services such as GMail, to get the token
you are required to activate MFA if you want to send e-mails using `git send-email`.

Whats more, I found it is required to use legal name [^2] for
submitting patches. Remember to pay attention of this!

For the e-mail content, make sure it is pure text.
Other content such as HTML will be rejected as the Linux
Kernel mailing list (lkml) will consider it is a spam or a virus.

## References

[^1]: https://hackmd.io/@sysprog/linux2023-summer-quiz3#%E6%8C%91%E6%88%B0-3

[^2]: https://github.com/torvalds/linux/blob/master/Documentation/process/1.Intro.rst

[^3]: https://docs.kernel.org/process/maintainer-tip.html 

[^4]: https://github.com/torvalds/linux/tags
