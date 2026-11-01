---
layout: post
title: "Demystifying Container Image Builds"
category: [DevOps]
tags: [Container, Docker, Podman, DevOps]
---

Container is used in a variety of domains nowadays:
from code development, product deployment, to
runtime environment of various applications.
You usually use the container as an ISO, but
under-the-hood, it acts differently yet you cannot
feel it because it is an illusion maintained by
the Linux Kernel.

In this post, I will talk about the container image
architecture with some pitfalls when your are going
to apply any applications in the image.

## OCI (Open Container Initiative)

Speaking of the image, you face a lot of image format
but the most common one is the OCI [^1]. Before the
proposal the OCI, each container engine held their
unique format, making the complexity when shifting
to other container engine. Thus, OCI was proposed
by defining a JSON manifest mapping a list of 
compressed tarballs (called layers).

## OverlayFS

For each layer, it will perform a certain action.
Obviously, it is a waste of space to record the whole filesystem.

As such, OverlayFS [^2] is proposed to record the difference
of each layer. Below list ome common terminologies:

- LowerDir: these are read-only, immutable image layers. For instance,
you will get five LowerDirs if you have five layers.
- UpperDir: it is created once the container is executed as
an ephemeral read-write layer.
- MergedDir: a unified view that any processes running in containers
will see.

Below shows the hierarchy of these common terminologies:

```
===========================================================================
                          MergedDir (Unified View)
                     What the container process actually sees
===========================================================================
  [ app.py (v2) ]        (hidden)           [ lib.so ]       [ logs.txt ]
        ^                                       ^                 ^
        |                                       |                 |
........|.......................................|.................|........
        |                                       |                 |
===========================================================================
                         UpperDir (Read-Write Layer)
                   The ephemeral layer created at runtime
===========================================================================
  [ app.py (v2) ]     [ .wh.secret.key ]   (no change)       [ logs.txt ]
        ^                     ^ 
 (CoW overrides)      (Whiteout hides) 
        |                     |
........|.....................|............................................
        |                     |
===========================================================================
                         LowerDir (Read-Only Layers)
                    The immutable container image tarballs
===========================================================================
  [ app.py (v1) ]       [ secret.key ]      [ lib.so ]       (not present)
===========================================================================
```

## Copy-on-Write (CoW) and Whiteouts

In the previous section, there are two terms: CoW and Whiteouts,
which are crucial in container. So, I will use a dedicated section
to explain them.

Whiteouts are used when deleting any files in a container.
As LowerDirs are immutable, Whiteouts are invented to mark the deleted
files so that you won't see such file in MergedDir.

CoW is a great mechanism dealing with large-size files:
it creates a writable layer so that the immutability of LowerDirs
won't be bothered, and you can let various containers share the
same layer of an image. Nevertheless, it is adviced to use Docker
volumns when you need to make massive I/O write to the container files,
boosting the write performance. 

## What You Should Beaware

Though containers provide a clean way to build and deployment
your environment, you should aware when you are dealing with
these scenarios:

- For container image space optimization, you should chain your commands,
especially setting up the dependencies (e.g., `apt-get update && apt-get install && apt-get clean`)
to prevent baking any unnecessary files being baked into a layer.
- For security, beaware the deleted files aren't gone because of Whiteouts.
Therefore, you should inspect each layer, not the MergedDir.

## Conclusion

In this post, I take you a tour of the container internals,
and let you know how does container handles any file modification
and delection. I also leave a note on container space optimization
and security so that you will put an eye when next time you
face the same scene.

Feel free to leave your comments for sharing your own tour
dealint with container internals!

## References

[^1]: https://opencontainers.org/

[^2]: https://docs.kernel.org/filesystems/overlayfs.html
