---
layout: post
title: "How to Switch to cgroup v1 in Ubuntu 24.04"
category: [Control group]
tags: [Ubuntu, Container, cgroup, Docker]
---

## Introduction

Recently, I need to test the programs that will utilize containers
based on control group v1 (cgroup v1). Nevertheless, Ubuntu adapts
cgroup v2 years ago, and there is no exception to Ubuntu 24.04.

After some chats with AI agent, it told me it is still possible
as cgroup v1 is deprecated and gave me a set of instructions
of switching to cgroup v1.

I want to make a record so that I can reference my work in the future.

## Steps

### 0. Check current cgroup version

```bash
docker info | grep -i cgroup
stat -fc %T /sys/fs/cgroup/
```

If it prints `cgroup2fs`, you can start to execute the following steps.
Otherwise, there is no need to switch.

### 1. Set the kernel to boot with cgroup v1

Ubuntu 24.04 uses systemd to decide which hierarchy to mount based on a kernel command-line flag.

First, edit GRUB:

```bash
sudo nano /etc/default/grub
```

Second, Modify the GRUB_CMDLINE_LINUX_DEFAULT line to include:

```
GRUB_CMDLINE_LINUX_DEFAULT="systemd.unified_cgroup_hierarchy=0"
```

If you need the pure legacy hierarchy (no v2 controllers mounted at all), include this instead:

```
GRUB_CMDLINE_LINUX_DEFAULT="systemd.unified_cgroup_hierarchy=0 systemd.legacy_systemd_cgroup_controller=1"
```

Last, update GRUB and reboot:

```bash
sudo update-grub
sudo reboot
```

### 2. Verify after reboot

```bash
stat -fc %T /sys/fs/cgroup/
mount | grep cgroup
```

You should get `tmpfs` instead of `cgroup2fs`

### 3. Point Docker's cgroup driver at cgroupfs

Docker on cgroup v2 typically uses the systemd cgroup driver. On v1, cgroupfs is the traditional/safer choice. 

First, edit `/etc/docker/daemon.json`:

```json
{
"exec-opts": ["native.cgroupdriver=cgroupfs"]
}
```

Next, restart Docker:

```bash
sudo systemctl restart docker

```

### 4. Confirm Docker sees v1

Use this command to verify:

```bash
docker info | grep -i "cgroup"
```

Expected output includes something like:

```
Cgroup Driver: cgroupfs
Cgroup Version: 1
```

## Conclusion

In this post, I make a set of procedures so that you can
switch to cgroup v1 by your needs. Though cgroup v1 is deprecated,
you sometimes need to use cgroup v1 for application compatibility.

Feel free to leave your comments of your thoughts after reading!
