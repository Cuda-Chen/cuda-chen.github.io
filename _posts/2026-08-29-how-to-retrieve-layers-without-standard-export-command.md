---
layout: post
title: "How to Retrieve Container Layers without Standard Export Command"
category: [Programming]
tags: [Container, Docker, Podman]
---

## Introduction

For container security, we need to extract the content of an image
to check whether there are any malicious hide in the image.

For an intuitive way, using a standard export command (`docker save`),
can retrieve the layers composing an image. However, the export
command is not feasible, and it becomes an attack vector.

So let's take another angle: as the export command just gives
you the layers, the layers must store somewhere else. Voilà!
You find the way to extract the layers just like some open-source
projects do like skopeo [^1].

## Retrieving the Layers of A Docker Image

### Docker Image

Usually stores in `/var/lib/docker/overlay2/`. Each layer directory
contains a `diff/` folder, which holds the actual filesystem manipulation
(files, directories, and whiteouts) for that specific layer.

You can use these commands to get the layers:

```
# Copy the layer contents to a local directory
sudo cp -a /var/lib/docker/overlay2/<hash>/diff /tmp/my-extracted-layer

# Or package the layer directly into a tarball
sudo tar -czvf layer.tar.gz -C /var/lib/docker/overlay2/<hash>/diff .
```

### Podman Image 

Usually stores in `/var/lib/containers/storage/overlay/<id>/merged`.
If run in rootless mode, it will be a path in your home directory.

You can use this command to get the layers:

```
cp -r /path/outputted/by/podman/mount /tmp/extracted-container-fs
```

## Conclusion

In this post, I express the reason you are willing to retrieve
the layers of an image. I then make examples to retrieve the layers
for commonly used containers such as Docker and Podman.

As there are many types of the image format, you may think this post
does give you little help. Feel free to make comments so that other
folks understand to retrieve their layers.

## References

[^1]: https://github.com/podman-container-tools/skopeo 
