---
layout: post
title: A Brief Introduction for sudo alternatives -- doas and polkit
category: Security
tags: [Security, sudo, doas, polkit, Linux]
---

## Introduction
`sudo` is a commonly used linux command which grants a command with root
permission temporarily. However, as its code is bloated and complicated
settings, it usually happens of misconfiguration, not mention to commonly
reports about vulnerabilities including the latest one [1].

## Alternatives of `sudo`
As a consequence, there are many alternatives of `sudo` not only to avoid
security vulnerabilities but also configuration in simplicity.

### `doas`
On famous substitution is `doas`. Originated from freeBSD, `doas` aims to
provide a lightweighted package and less complex setting for grant root
permission [2].

If you want to try `doas`, you can install it either with your distro's
package manager or [here](https://github.com/slicer69/doas).

### `polkit`
Seen commonly in RHEL distro, `polkit` is yet-another substitution of
`sudo`. Unlike `sudo`, it does not grant root permission to a whole
process, but allows a finer control of control of centralized 
system policy [3].

What's more, `polkit` restricts actions -- such as running `dd` -- and
users by group or by name. [3]

## Conclusion
`sudo` is used for granting temporary root permission for a process.
However, due to its complex configuration and code size, we often get
the news about its vulnerabilities. As a result, there are a lot of
`sudo` substitution aims either simple configuration or less code size,
or even both. In this article, I introduce two of them: `doas` and `polkit`.

Nevertheless, many of `sudo` substitution, including `doas` and `polkit`,
are not fully tested in harsh condition. So if you want to use these
alternatives in production, measure your risk!

## Reference
[1] http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-3156

[2] https://wiki.gentoo.org/wiki/Doas

[3] https://wiki.archlinux.org/index.php/Polkit
