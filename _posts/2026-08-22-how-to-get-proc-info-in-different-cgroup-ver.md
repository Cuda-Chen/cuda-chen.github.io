---
layout: post
title: "How to Retrieve Docker Container Process Information under Different Cgroup Version"
category: [Container]
tags: [Programming, C, Container, Docker]
---

## Introduction

Recently, I was working on retrieving Docker container process information
for monitoring such as detect whether the container is running
any malicious processes. As I am going to support all types of Cgroup
version, things become complex. To
let myself familiar with Cgroup and its architecture in different
version, I make this post for my reading record.

## Cgroup Introduction

Before We talk about the Cgroup version layouts, I think I would
like to answer the question: what is Cgroup? Cgroup is a Linux
Kernel mechanism in order to create a set of restrictions to
regulate the process, in a certain group, to system resource usage.
For process virtualization, Cgroup is an ideal mechanism to achieve
such goal, thus it is widely used on container implementation.

As there are certain versions of Cgroup, let's talk about its major
versions: v1 and v2. Also, for the ease of migrating from v1 to v2, 
there exists a special "version" called Hybrid.

### Controller Architecture and Hierarchy

The most vital difference is that how resources (e.g., CPU, memory,
I/O) and mounted and managed.

- cgroup v1
  - Use a **multiple hierarchy** mode.
  - Each resource controller (cpu, memory, blkio) is mounted as its 
own separate tree in `/sys/fs/cgroup/`.
  - If a container needs limits on CPU and memory, its process must 
be added to two entirely separate directories 
(e.g., `/sys/fs/cgroup/cpu/...` and `/sys/fs/cgroup/memory/...`).
- cgroup v2
  - Use a **single, unified hierarchy.**
  - Controllers are enabled top-down via configuration files 
(`cgroup.subtree_control`), meaning a process belongs to exactly one 
cgroup path which governs all its resources.
- Hybrid
  - A transitional mode designed for legacy compatibility.
  - It mounts cgroup v1 controllers for resource management (CPU, memory), 
but simultaneously mounts an empty cgroup v2 hierarchy 
(often at `/sys/fs/cgroup/unified`).
  - Systemd uses this v2 hierarchy solely for process tracking,
while resource limits remain managed by v1.

### Where Container Processes Place

- cgroup v1: PIDs can be placed in any node or sub-node within
the hierarchy tree.
- cgroup v2: imposes a strict **"no internal process (or top-down) constraint".**
Processes can only be placed in leaf nodes (i.e., the node has no any
child nodes) if resource controllers are enabled in that branch.

### File System Representation for Processes

When retrieving container process information via `/proc/[PID]/cgroup`,
the output format dictates how you parse the container ID:

- cgroup v1: the file contains multiple lines, and each line maps
a subsystem to its specific cgroup path.
- cgroup v2: the file contains exactly one line representing the unified
path start with `0::`.
- Hybrid: the file contains multiple v1 lines, plus a single
`1:name=systemd:/...` or `0::/...` line showing the process's placement 
in the v2 unified tracker.

### Architecture Overview of Different Versions

For your takeways, I provide the architecture overview:

```
====================================================================
                       cgroup v1 (Multiple)
====================================================================
 /sys/fs/cgroup/
  ├── cpu/
  │    └── docker/
  │         └── <container_id>/      <-- PID added here for CPU
  ├── memory/
  │    └── docker/
  │         └── <container_id>/      <-- PID added here for Memory
  └── pids/
       └── docker/
            └── <container_id>/      <-- PID added here for PIDs

====================================================================
                       cgroup v2 (Unified)
====================================================================
 /sys/fs/cgroup/
  └── system.slice/
       └── docker-<container_id>.scope/
            ├── cgroup.controllers   <-- enabled: "cpu memory pids"
            ├── cgroup.procs         <-- PID added here ONLY ONCE
            ├── cpu.max
            └── memory.max

====================================================================
                       Hybrid Mode
====================================================================
 /sys/fs/cgroup/
  ├── cpu/         (v1 hierarchy)    <-- Resource limiting
  ├── memory/      (v1 hierarchy)    <-- Resource limiting
  └── unified/     (v2 hierarchy)    <-- Systemd process tracking
```

## Retrieve Container Process Information

To retrieve counter processes, we can locate the `/proc/[PID]/cgroup` file.

Below lists the example format of the output so that you can
parse the process information accordingly.

### cgroup v1

```
12:cpuset:/docker/a1b2c3d4e5f6g7h8i9j0...
11:cpu,cpuacct:/docker/a1b2c3d4e5f6g7h8i9j0...
1:name=systemd:/docker/a1b2c3d4e5f6g7h8i9j0...

# for Kubernetes, the path will look like this
/kubepods/burstable/pod<UUID>/a1b2c3d4...
```

### cgroup v2

There is only one cgroup path per process under cgroup v2 environment.
Container runtimes heavily utilize systemd transient scopes (`.scope`) 
to manage these containers within slices.

```
0::/system.slice/docker-a1b2c3d4e5f6g7h8i9j0.scope
```

(Note: Podman uses `libpod-<ID>.scope`, and containerd/CRI-O often 
use `crio-<ID>.scope` or similar under `kubepods.slice`)

### Hybrid

`/proc/[PID]/cgroup` contains a mix of v1 resource controllers and a systemd tracking line.

```
12:cpuset:/docker/a1b2c3d4e5f6g7h8i9j0
11:cpu,cpuacct:/docker/a1b2c3d4e5f6g7h8i9j0
0::/system.slice/docker-a1b2c3d4e5f6g7h8i9j0.scope
```

### Process Parsing Flow Graph

```
[ Read /proc/<PID>/cgroup ]
                               |
                               v
            +-------------------------------------+
            |        Does line start with         |
            |                "0::" ?              |
            +-------------------------------------+
                   /                       \
             [ YES ]                       [ NO ]
           (v2/Hybrid)                  (v1/Hybrid)
               |                             |
               v                             v
+-----------------------------+ +-----------------------------+
| Check for systemd boundary  | | Check for runtime boundary  |
| pattern (Regex match)       | | (String split by '/')       |
|                             | |                             |
| MATCH:                      | | MATCH:                      |
| *-<CONTAINER_ID>.scope      | | .../<RUNTIME>/<CONTAINER_ID>|
|                             | |                             |
| (e.g., docker-xyz123.scope) | | (e.g., /docker/xyz123)      |
+-----------------------------+ +-----------------------------+
               \                             /
                \                           /
                 v                         v
            +-------------------------------------+
            |         Return <CONTAINER_ID>       |
            +-------------------------------------+
```

## Conclusion

In this post, I first make a brief introduction of Cgroup. Then,
I make a comparison with the architecture figures to illustrate
the difference between Cgroup versions. Last, I make a minimal
example to retrieve process information under differenr Cgroup
version running environment.

As there are many types of container implementation beside Docker, I believe
there are something I haven't covered up. Feel free to make your
comments for discussion!
