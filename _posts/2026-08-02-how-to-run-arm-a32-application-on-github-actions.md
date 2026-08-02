---
layout: post
title: "How to Run Arm A32 Application on GitHub Actions"
category: [DevOps]
tags: [DevOps, testing, benchmarking]
---

Recently, I have some needs to run Arm A32 applications
for benchmarking. As I don't have any physical Arm devices
supporting A32 instructions, I request a help to community.
To my surprise, the maintainer of sse2neon replied that I can still
run A32 application on GitHub Actions native Arm runner
as the CPU supports both A64 and A32. [^1] As such, I
write this down for a record of setting all stuffs up.

For the image, it will be `ubuntu-<version>-arm`.
Furthermore, I am benchmarking a C/C++ program.

## So How Does This "Magic" happens?

GitHub Actions native Arm runner is based on Microsoft
Azure Cobalt 100 [^2], supporting A64. What's more, it
supports A32 instructions, which in turn delivers a better
emulated benchmark results compared to running benchmark on QEMU.

## Setting Up

### Verify Kernel Support

You need to make sure the `CONFIG_COMPAT` is set to `=y`:

```
zcat /proc/config.gz | grep CONFIG_COMPAT
```

### Enable Multi-arch Support

Remember to add the `armhf` architecture when running the pipeline:

```
sudo dpkg --add-architecture armhf
```

### Install 32-bit Libraries

Add the following libraries for linking:

```
sudo apt install libc6:armhf libstdc++6:armhf
```

### (Optional) Symlink to 32-bit Linker

If your binary throws a "No such file or directory" error even though the file exists, create a symbolic link to the expected 32-bit interpreter location:

```
sudo ln -s /lib/arm-linux-gnueabihf/ld-linux-armhf.so.3 /lib/ld-linux.so.3
```

## Alternative Way: Docker Contaienr

If you think it is troublesome to set up and you do not need
for almost bare-metal benchmarking, you can use Docker container
to create such environment:

```
docker run --rm -v \$(pwd):/app -w /app arm32v7/ubuntu ./your_executable_name
```


## Recap

In this post, I summarize the characteristic of GitHub Actions
native Arm runner, which supports A32 instructions.
I then make steps to set up the pipeline to run
A32 applications on GitHub Actions utilizing its native Arm runner.
Last, I provide an alternavie method if you don't need almost
bare-metal running experience with the ease for setting the
A32 environment up.

Feel free to comment to share your thoughts of usage! 

## References

[^1]: https://github.com/DLTcollab/sse2neon/pull/769#issuecomment-5151223800 

[^2]: https://github.com/orgs/community/discussions/148648
