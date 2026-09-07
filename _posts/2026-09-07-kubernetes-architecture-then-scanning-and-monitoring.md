---
layout: post
title: "How to Retrieve the Container and Images in A Kubernetes Cluster"
category: [Kubernetes]
tags: [k8s, Kubernetes, Container]
---

## Introduction

Kubernetes [^1] is widely used nowadays for container orchestration
with its ability for automating deployment, scaling, and management.
From a cybersecurity aspect, containerization creates a lot of
vulnerabilities, and there is no exception on Kubernetes.

For my point of view, the attackers can compromise a Kubernetes
cluster by either loading an image with payloads and create
a malicious program which runs in containers. As a result,
I am going to retrieve the files in an image and monitoring the processes
running in a container.

## Kubernetes Architecture

Before we get our needed information, we should take a look of
the architecture of Kubernetes:

```
+---------------------------------------------------------------+
|                       EXTERNAL CLIENTS                        |
|           (Users, kubectl, Jenkins, Management UI)            |
|                               |                               |
+-------------------------------|-------------------------------+
                                V (via REST/HTTPS)
+---------------------------------------------------------------+
|                         CONTROL PLANE                         |
|                                                               |
|     +-----------------------+       +-----------------------+ |
|     |                       |       |                       | |
|     |    kube-apiserver     |------>|         etcd          | |
|     | (Hub/Auth/Validation) |<------| (State/Configuration) | |
|     |                       |       |                       | |
|     +-----------------------+       +-----------------------+ |
|          |             |                                      |
|          |             |                                      |
|     +----|----+   +----|----+                                 |
|     | kube-   |   | kube-   |                                 |
|     | sched-  |   | cont-   |                                 |
|     | uler    |   | roller- |                                 |
|     |         |   | manager |                                 |
|     +---------+   +---------+                                 |
|                                                               |
+----------|------------------|------------------|--------------+
           | (Control Data)   | (Control Data)   | (Status Data)
           V                  V                  V
+----------|------------------|------------------|--------------+
|          |             WORKER NODE             |              |
|          |                                     |              |
|    +-----|-----+                         +-----|-----+        |
|    |           |                         |           |        |
|    |  kubelet  |<----------------------->|   kube-   |        |
|    |  (Agent)  |   (Local Node State)    |   proxy   |        |
|    |           |                         |   (Net)   |        |
|    +-----+-----+                         +-----+-----+        |
|          |                                     |              |
|          | (CRI/Socket)                        | (Net Rules)  |
|          V                                     |              |
|    +-----|-----------+                         V              |
|    | Container       |                   [Node Network]       |
|    | Runtime (CR)    |                                        |
|    | (containerd/    |                                        |
|    |  CRI-O)         |                                        |
|    +-----|-----------+                                        |
|          | (Runs Container Process)                           |
|          V                                     ^              |
|    +-----|------------------------------+      |              |
|    |                POD                 |      |              |
|    |                                    |      |              |
|    |  +------------------------------+  |      |              |
|    |  |          CONTAINER           |  |      |              |
|    |  | [Namespaces (Isolation)]     |  |------+              |
|    |  | [Cgroups (Resource Limit)]   |  | (Process/Net Event) |
|    |  |                              |  |                     |
|    |  | (Process/Memory Allocation)  |  |                     |
|    |  +------------------------------+  |                     |
|    |                                    |                     |
|    +------------------------------------+                     |
|                                                               |
+---------------------------------------------------------------+
```

A Kubernetes can be seperated into two main domains: the Control Plan
and the Worker Nodes.

### Control Plan

It is responsible to make global decisions, and it is composed with
these parts:

1. **kube-apiserver**: The central hub. All communication (from users, nodes, and your custom tools) goes through this REST API.
2. **etcd**: A highly available key-value store containing the cluster's entire state.
3. **kube-scheduler**: Watches for newly created Pods that have no assigned node and selects a node for them to run on.
4. **kube-controller-manager**: Runs control loops that watch the state of the cluster and make changes to drive the current state toward the desired state.

### Worker Nodes

It is responsible for running the actual workloads with the composition
of:

1. **kubelet**: The primary node agent. It takes instructions from the API server and ensures the containers described in those instructions are running and healthy.
2. **Container Runtime (e.g., containerd, CRI-O)**: The software responsible for pulling images and actually spinning up the containers via the Container Runtime Interface (CRI). Under the hood, it utilizes Linux namespaces (for isolation) and cgroups (for resource limitation).
3. **kube-proxy**: Maintains network rules on the nodes, allowing network communication to your Pods from inside or outside the cluster.
4. **Pods**: The smallest and simplest unit representing a single instance of
an application. Each pod is made up of a container or a series
of tightly compled containers, along with options that govern
how the containers are run. Also, you can connect persistent storage
to pods to run stateful applications.

## Scanning Images for Hunting Malicious Files

We can utilize the Control Plane with Validating Admission Webhook [^2].
The pipeline is described as follows:

1. **Interception**: You register a webhook with the `kube-apiserver`. When a user attempts to create a Pod, the API server pauses the request and sends the Pod specification (containing the image name) to your scanning program.
2. **Image Retrieval**: Your program acts as an OCI (Open Container Initiative) client. It connects to the container registry where the image is hosted, authenticates, and downloads the image manifest and the compressed image layers (tarballs).
3. **Static Analysis**: Your program unpacks the layers in memory or a temporary directory. It then scans the filesystem against malware signatures (e.g., using Yara rules), inspects the OS packages, and analyzes application binaries for known vulnerabilities.

You then use webhook to reply either `allowed: true` or `allowed: false`
to accept/reject user requests.

### Monitoring Processes and Its Memory

You must bridge the gap between kernel-level events and Kubernetes-level metadata.
The standard pattern is to adopt DaemonSet [^3] that it ensures one instance
runs on every worker node.

The following list depicts the pipeline:

1. **Kernel Interception (eBPF)**: Instead of polling `procfs`, your agent loads eBPF programs into the kernel. You can attach to tracepoints like `sys_enter_execve` to catch every new process execution in real-time, and trace memory allocation events or page faults to track active memory behavior. 
2. **Resource Polling (cgroups)**: For overall memory telemetry, your agent reads the Linux cgroup filesystem (e.g., `/sys/fs/cgroup/memory/...`). Because each container gets its own cgroup hierarchy, you can read the exact memory footprint directly from the kernel interface.
3. **Context Enrichment**: eBPF and cgroups only understand low-level constructs like Process IDs (PIDs) and cgroup paths. Your agent must query the local container runtime (via the CRI socket) or the local Kubelet to map that PID to a Kubernetes Pod Name, Namespace, and Container ID.
4. **Export**: The agent formats these enriched events (e.g., "Process 'curl' executed in Pod 'web-frontend', currently using 45MB of RAM") and streams them to a centralized backend or dashboard.

## Conclusion

In this post, I first illustrate my goals of scanning images and monitoring containers.
I then make a brief introduction of Kubernetes. Next, I make a possible way
to achieve my goals.

As this post serves for the demonstration purpose, feel free to write
your techniques and the scenarios of monitoring containers and scanning images.

## References

[^1]: https://kubernetes.io/

[^2]: https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/

[^3]: https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/
