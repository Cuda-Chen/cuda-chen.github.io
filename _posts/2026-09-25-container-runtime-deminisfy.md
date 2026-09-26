---
layout: post
title: "Deminisfy Container Runtime: from Kubernetes to Process"
category: [Container]
tags: [Container Runtime, Container, k8s, Kubernetes]
---

## Introduction

When running any containerized apps, I always confused by the nouns
of "Docker", "Podman", and "runc", and Kubernetes does deepen
my confusion. Consequently, I write this post for not only my record
of deminisfying container runtime but also a guide for others not
to confused by these terms.

## What is A Container Runtime?

Briefly, container runtime is the actual component that manages and
spawns the containers. By its functionality, it can be parted as
high-level and low-level runtimes.

### High-Level Runtimes

High-level runtimes handle the following:

- Pull OCI images from registries.
- Extract layers.
- Manage network inferfaces.
- Provide an API for end-user interaction.

For examples, CRI-O (the standard of Kubernetes), Docker, Podman,
containerd, etc., are all high-level runtimes.

### Low-Level Runtimes

Once the high-level runtime has the OverlayFS root filesystem ready,
it hands execution off to a low-level runtime. This is the component
that actually executes the process, sets up Linux namespaces, configures
`cgroups`, and applies security mechanism such as seccomp or AppArmor.

There are plenty of low-level runtimes, and I list the comparison in
this table:

```
+-------------------+--------------------------------+-------------------------------------------------------------+
| Low-Level Runtime | Architecture & Implementation  | Security & Isolation Strategy                               |
+-------------------+--------------------------------+-------------------------------------------------------------+
| runc              | The reference standard,        | Standard Namespaces/cgroups. Relies entirely on the host    |
|                   | written in Go.                 | Linux kernel's built-in isolation. Vulnerable to kernel-    |
|                   |                                | level privilege escalation.                                 |
+-------------------+--------------------------------+-------------------------------------------------------------+
| crun              | Written in C. Highly optimized | Standard Namespaces/cgroups. Faster startup than runc and   |
|                   | and lightweight.               | provides excellent native support for cgroups v2            |
|                   |                                | hierarchies and rootless containers.                        |
+-------------------+--------------------------------+-------------------------------------------------------------+
| gVisor (runsc)    | Written in Go, developed by    | User-space Kernel Sandboxing. Intercepts application        |
|                   | Google.                        | syscalls (via ptrace or KVM) and handles them in a secure,  |
|                   |                                | isolated user-space kernel (the Sentry) before talking to   |
|                   |                                | the host kernel. Prevents direct kernel exploits.           |
+-------------------+--------------------------------+-------------------------------------------------------------+
| Kata Containers   | Written in Go/Rust.            | Hardware Virtualization. Wraps each container in an         |
|                   |                                | extremely lightweight virtual machine (using QEMU or        |
|                   |                                | Firecracker). Provides hardware-level isolation while       |
|                   |                                | maintaining a container-like user experience.               |
+-------------------+--------------------------------+-------------------------------------------------------------+
```

## Container Creation Flow

After we understand the difference of high-level and low-leve
runtimes, let us have a flow graph of how does Kubernetes/Docker
creates a container:

```
+---------+       +------------+       +-------------+       +--------+       +--------+
| Kubelet |       | containerd |       | Snapshotter |       |  runc  |       | Kernel |
| / CLI   |       | (High-Level|       | (OverlayFS) |       | (Low-  |       |        |
|         |       |  Runtime)  |       |             |       | Level) |       |        |
+---------+       +------------+       +-------------+       +--------+       +--------+
     |                  |                     |                   |                |
     | 1. Create Cont.  |                     |                   |                |
     |----------------->|                     |                   |                |
     |                  |                     |                   |                |
     |                  | 2. Pull OCI Image   |                   |                |
     |                  |---------------------|                   |                |
     |                  |                     |                   |                |
     |                  | 3. Unpack & Mount   |                   |                |
     |                  |-------------------->|                   |                |
     |                  |                     |                   |                |
     |                  | 4. Create & Start (rootfs, config.json) |                |
     |                  |---------------------------------------->|                |
     |                  |                     |                   |                |
     |                  |                     |                   | 5. Config NS,  |
     |                  |                     |                   |    cgroups,    |
     |                  |                     |                   |    seccomp     |
     |                  |                     |                   |--------------->|
     |                  |                     |                   |                |
     |                  |                     |                   | 6. Exec Cont.  |
     |                  |                     |                   |    Process     |
     |                  |                     |                   |--------------->|
     |                  |                     |                   |                |
     |                  |                     |                   | [Process Runs] |
     |                  |                     |                   |                |
     |                  |                     |                   | 7. runc exits  |
     |                  |                     |                   |<---------------|
     |                  |                     |                   |                |
```

## Conclusion

In this post, I list the difference of high-level and low-level runtimes
for clarifying the usage of each runtime. I also list the flow when
creating a container from Kubernetes to actual runtime.

As the depth of container runtime, I do not listed such details.
You may leave your comments for the supplement information.
