---
layout: post
title: "Limiting the Resources of A C Program"
category: [Programming]
tags: [Programming, C, cgroups]
---

Cgroups is a great tool for limiting the resources
that can be used in a cgroup member such as process
and thread. Though it usually used on containers,
we can use cgroups in any arbitrary programs to
restrict their resource usage.

## Prerequisites

Your host will run on cgoups v2, and you will
grant the main program with root or sufficient
privilege.

## Method 1: Fork A Process Then Apply

For the most obvious and robust one, you just fork
a child process then apply the cgroups. The benefit of this
method is that when the child process crashes, the
main parent process survives and can inpect the 
termination status of the child process, leaving no
harm to the main program.

Below shows the example of doing this:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/wait.h>
#include <sys/stat.h>

#define CGROUP_DIR "/sys/fs/cgroup/code_sandbox"

static int attach_pid_to_cgroup(pid_t pid) {
    char path[128];
    snprintf(path, sizeof(path), "%s/cgroup.procs", CGROUP_DIR);
    int fd = open(path, O_WRONLY);
    if (fd < 0) return -1;

    char buf[32];
    int len = snprintf(buf, sizeof(buf), "%d\n", pid);
    if (write(fd, buf, len) != len) {
        close(fd);
        return -1;
    }
    close(fd);
    return 0;
}

void resource_heavy_computation(void) {
    // Target code to limit (e.g., intensive compute, allocations)
    printf("[Worker %d] Executing limited code block...\n", getpid());
    sleep(2);
}

int run_with_limits(void (*workload)(void)) {
    // 1. Create synchronization pipe so child pauses until attached
    int sync_pipe[2];
    if (pipe(sync_pipe) != 0) return -1;

    pid_t pid = fork();
    if (pid < 0) {
        close(sync_pipe[0]);
        close(sync_pipe[1]);
        return -1;
    }

    if (pid == 0) {
        // Child: wait for parent to apply cgroup attachment
        close(sync_pipe[1]);
        char sig;
        read(sync_pipe[0], &sig, 1);
        close(sync_pipe[0]);

        workload();
        _exit(EXIT_SUCCESS);
    }

    // Parent: attach child PID before unblocking it
    close(sync_pipe[0]);
    if (attach_pid_to_cgroup(pid) != 0) {
        kill(pid, SIGKILL);
        close(sync_pipe[1]);
        waitpid(pid, NULL, 0);
        return -1;
    }

    // Release child to start execution
    write(sync_pipe[1], "GO", 1);
    close(sync_pipe[1]);

    // Wait for completion or inspect if killed by OOM (SIGKILL)
    int status;
    waitpid(pid, &status, 0);
    if (WIFSIGNALED(status)) {
        printf("Worker terminated by signal: %d (e.g. 9 = OOM/Killed)\n", WTERMSIG(status));
        return -1;
    }
    return WEXITSTATUS(status);
}
```

## Method 2: Create A Thread Then Apply

Alike Method 1, you create a thread then apply
the cgroups to the thread. The strength is that
you can share virtual address space and heap
allocations to the rest of your C program.

However, because cgroups cannot enforce per-thread
memory limits. Alternatively, you can use the parent
domain's `memory.max` to reach the memory limits goal (
you can use the shared `mm_struct` of the process).

I leave the demonstrative C program here:

```c
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <pthread.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <sys/syscall.h>
#include <time.h>
#include <errno.h>

#define CGROUP_ROOT        "/sys/fs/cgroup"
#define PARENT_CGROUP      CGROUP_ROOT "/threaded_demo"
#define THROTTLED_CHILD    PARENT_CGROUP "/throttled_worker"

static int g_throttled_threads_fd = -1;

/* Helper to write string to an arbitrary sysfs path */
static int write_cgroup_file(const char *path, const char *val) {
    int fd = open(path, O_WRONLY);
    if (fd < 0) {
        fprintf(stderr, "Failed to open '%s': %s\n", path, strerror(errno));
        return -1;
    }
    ssize_t len = strlen(val);
    if (write(fd, val, len) != len) {
        fprintf(stderr, "Failed writing '%s' to '%s': %s\n", val, path, strerror(errno));
        close(fd);
        return -1;
    }
    close(fd);
    return 0;
}

/* Ensure directory exists */
static int safe_mkdir(const char *path) {
    if (mkdir(path, 0755) == -1 && errno != EEXIST) {
        fprintf(stderr, "mkdir '%s' failed: %s\n", path, strerror(errno));
        return -1;
    }
    return 0;
}

/* Cleanup open FDs and tear down the cgroup hierarchy */
void cleanup_threaded_cgroups(void) {
    if (g_throttled_threads_fd >= 0) {
        close(g_throttled_threads_fd);
        g_throttled_threads_fd = -1;
    }

    // Move main process back to root cgroup before deletion
    write_cgroup_file(CGROUP_ROOT "/cgroup.procs", "0\n");

    // Remove leaf first, then parent
    rmdir(THROTTLED_CHILD);
    rmdir(PARENT_CGROUP);
}

/* Programmatic setup for cgroup v2 threaded mode */
int setup_threaded_cgroup_hierarchy(void) {
    // 1. Create parent domain directory
    if (safe_mkdir(PARENT_CGROUP) != 0) return -1;

    // 2. Attach main process to parent before switching to threaded
    // (A cgroup can only transition to threaded if its processes match the subtree domain)
    if (write_cgroup_file(PARENT_CGROUP "/cgroup.procs", "0\n") != 0) {
        return -1;
    }

    // 3. Enable +cpu in subtree_control
    if (write_cgroup_file(PARENT_CGROUP "/cgroup.subtree_control", "+cpu") != 0) {
        fprintf(stderr, "Error: Unable to enable '+cpu'. Ensure root delegates cpu controller.\n");
        return -1;
    }

    // 4. Convert parent to threaded mode
    if (write_cgroup_file(PARENT_CGROUP "/cgroup.type", "threaded") != 0) {
        return -1;
    }

    // 5. Create child cgroup (automatically created as cgroup.type = threaded)
    if (safe_mkdir(THROTTLED_CHILD) != 0) return -1;

    // 6. Set CPU limit on child: 20ms per 100ms quota (20% of 1 core)
    if (write_cgroup_file(THROTTLED_CHILD "/cpu.max", "20000 100000") != 0) {
        return -1;
    }

    // 7. Keep open FD to throttled cgroup.threads for fast TID assignment
    g_throttled_threads_fd = open(THROTTLED_CHILD "/cgroup.threads", O_WRONLY);
    if (g_throttled_threads_fd < 0) {
        perror("open throttled/cgroup.threads");
        return -1;
    }

    return 0;
}

/* Self-migration primitive for calling thread */
static inline int attach_current_thread_to_throttled(void) {
    pid_t tid = (pid_t)syscall(SYS_gettid);
    char buf[32];
    int len = snprintf(buf, sizeof(buf), "%d\n", tid);
    return (pwrite(g_throttled_threads_fd, buf, len, 0) == len) ? 0 : -1;
}

/* CPU-bound workload helper */
static double burn_cpu(long iterations) {
    struct timespec start, end;
    clock_gettime(CLOCK_MONOTONIC, &start);

    volatile unsigned long acc = 0;
    for (long i = 0; i < iterations; i++) {
        acc += (i ^ (acc >> 1));
    }

    clock_gettime(CLOCK_MONOTONIC, &end);
    return (end.tv_sec - start.tv_sec) + (end.tv_nsec - start.tv_nsec) / 1e9;
}

/* Worker 1: Unthrottled thread (runs inside PARENT_CGROUP root domain) */
void *unthrottled_worker(void *arg) {
    (void)arg;
    pid_t tid = (pid_t)syscall(SYS_gettid);
    printf("[TID %d] Running unthrottled thread...\n", tid);
    double elapsed = burn_cpu(300000000);
    printf("[TID %d] Unthrottled thread finished in %.3f seconds.\n", tid, elapsed);
    return NULL;
}

/* Worker 2: Throttled thread (moves itself into THROTTLED_CHILD) */
void *throttled_worker(void *arg) {
    (void)arg;
    pid_t tid = (pid_t)syscall(SYS_gettid);

    if (attach_current_thread_to_throttled() != 0) {
        fprintf(stderr, "[TID %d] Failed to attach to throttled cgroup\n", tid);
        return NULL;
    }

    printf("[TID %d] Running throttled thread (capped at 20%% CPU)...\n", tid);
    double elapsed = burn_cpu(300000000);
    printf("[TID %d] Throttled thread finished in %.3f seconds.\n", tid, elapsed);
    return NULL;
}

int main(void) {
    printf("Configuring cgroup v2 threaded hierarchy programmatically...\n");
    if (setup_threaded_cgroup_hierarchy() != 0) {
        cleanup_threaded_cgroups();
        return EXIT_FAILURE;
    }
    printf("Threaded cgroups initialized successfully.\n\n");

    pthread_t t1, t2;

    // 1. Run unthrottled worker
    pthread_create(&t1, NULL, unthrottled_worker, NULL);
    pthread_join(t1, NULL);

    printf("\n");

    // 2. Run throttled worker
    pthread_create(&t2, NULL, throttled_worker, NULL);
    pthread_join(t2, NULL);

    printf("\nTearing down threaded cgroups...\n");
    cleanup_threaded_cgroups();
    printf("Done.\n");

    return EXIT_SUCCESS;
}
```

## Method 3: Dynamic Hopping

This is the interesting one: you can move the thread
itself into a cgroup when you start executing a certain scope,
then exit the cgroup when you leave the scope. It is
useful when you just need certain scopes that will
not use too many resources, interferring the whole system.

The cgroups hierarchy is shows as follows:

```
               /sys/fs/cgroup/
                     │
         [mkdir] dyn_hop_demo/
                     │
     [write "+cpu" to cgroup.subtree_control]
           ┌─────────┴─────────┐
       [mkdir]             [mkdir]
    unrestricted/        restricted/
                           │
             [write "20000 100000" to cpu.max]
```

For the demonstration, this is the reference C program:

```c
#define _GNU_SOURCE
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <time.h>
#include <errno.h>

#define CGROUP_ROOT         "/sys/fs/cgroup"
#define DEMO_PARENT_CGROUP  CGROUP_ROOT "/dyn_hop_demo"
#define RESTRICTED_CGROUP   DEMO_PARENT_CGROUP "/restricted"
#define UNRESTRICTED_CGROUP DEMO_PARENT_CGROUP "/unrestricted"

static int g_fd_restricted   = -1;
static int g_fd_unrestricted = -1;

/* Helper to write string to an arbitrary sysfs path */
static int write_cgroup_file(const char *path, const char *val) {
    int fd = open(path, O_WRONLY);
    if (fd < 0) {
        fprintf(stderr, "Failed to open '%s': %s\n", path, strerror(errno));
        return -1;
    }
    ssize_t len = strlen(val);
    if (write(fd, val, len) != len) {
        fprintf(stderr, "Failed writing '%s' to '%s': %s\n", val, path, strerror(errno));
        close(fd);
        return -1;
    }
    close(fd);
    return 0;
}

/* Ensure directory exists; if not, create it */
static int safe_mkdir(const char *path) {
    if (mkdir(path, 0755) == -1 && errno != EEXIST) {
        fprintf(stderr, "mkdir '%s' failed: %s\n", path, strerror(errno));
        return -1;
    }
    return 0;
}

/* Clean up open FDs and rmdir groups in bottom-up order */
void cleanup_cgroups(void) {
    if (g_fd_restricted >= 0) {
        close(g_fd_restricted);
        g_fd_restricted = -1;
    }
    if (g_fd_unrestricted >= 0) {
        close(g_fd_unrestricted);
        g_fd_unrestricted = -1;
    }

    // Move calling process back to root cgroup before deleting leaf groups
    write_cgroup_file(CGROUP_ROOT "/cgroup.procs", "0\n");

    // Remove leaf nodes first, then parent
    rmdir(RESTRICTED_CGROUP);
    rmdir(UNRESTRICTED_CGROUP);
    rmdir(DEMO_PARENT_CGROUP);
}

/* Fully programmatic creation and configuration */
int setup_dynamic_cgroup_hierarchy(void) {
    // 1. Create top-level container
    if (safe_mkdir(DEMO_PARENT_CGROUP) != 0) return -1;

    // 2. Enable +cpu in subtree_control so child cgroups can enforce cpu.max
    // In cgroup v2, this activates CPU controller distribution down the tree
    if (write_cgroup_file(DEMO_PARENT_CGROUP "/cgroup.subtree_control", "+cpu") != 0) {
        fprintf(stderr, "Error: Unable to enable '+cpu'. Make sure root cgroup enables cpu controller.\n");
        return -1;
    }

    // 3. Create the two leaf sub-cgroups
    if (safe_mkdir(UNRESTRICTED_CGROUP) != 0) return -1;
    if (safe_mkdir(RESTRICTED_CGROUP) != 0) return -1;

    // 4. Set CPU quota on restricted: 20,000us per 100,000us window (20% of 1 core)
    if (write_cgroup_file(RESTRICTED_CGROUP "/cpu.max", "20000 100000") != 0) {
        return -1;
    }

    // 5. Pre-open cgroup.procs file descriptors for zero-open hopping
    g_fd_restricted = open(RESTRICTED_CGROUP "/cgroup.procs", O_WRONLY);
    if (g_fd_restricted < 0) {
        perror("open restricted/cgroup.procs");
        return -1;
    }

    g_fd_unrestricted = open(UNRESTRICTED_CGROUP "/cgroup.procs", O_WRONLY);
    if (g_fd_unrestricted < 0) {
        perror("open unrestricted/cgroup.procs");
        return -1;
    }

    // 6. Adopt current process into the unrestricted leaf group
    if (pwrite(g_fd_unrestricted, "0\n", 2, 0) != 2) {
        perror("Initial migration to unrestricted failed");
        return -1;
    }

    return 0;
}

/* Fast hop primitives */
static inline int enter_restricted_scope(void) {
    return (pwrite(g_fd_restricted, "0\n", 2, 0) == 2) ? 0 : -1;
}

static inline int leave_restricted_scope(void) {
    return (pwrite(g_fd_unrestricted, "0\n", 2, 0) == 2) ? 0 : -1;
}

/* Workload timer helper */
static double burn_cpu(long iterations) {
    struct timespec start, end;
    clock_gettime(CLOCK_MONOTONIC, &start);

    volatile unsigned long acc = 0;
    for (long i = 0; i < iterations; i++) {
        acc += (i ^ (acc >> 1));
    }

    clock_gettime(CLOCK_MONOTONIC, &end);
    return (end.tv_sec - start.tv_sec) + (end.tv_nsec - start.tv_nsec) / 1e9;
}

/* The restricted target function */
void limited_computation_function(void) {
    if (enter_restricted_scope() != 0) {
        fprintf(stderr, "Failed to hop into restricted cgroup\n");
        return;
    }

    printf("  [Inside function] CPU throttled to 20%%...\n");
    double elapsed = burn_cpu(300000000);
    printf("  [Inside function] Finished in %.3f seconds.\n", elapsed);

    if (leave_restricted_scope() != 0) {
        fprintf(stderr, "Failed to hop out of restricted cgroup\n");
        return;
    }
}

int main(void) {
    printf("Setting up cgroup hierarchy programmatically...\n");
    if (setup_dynamic_cgroup_hierarchy() != 0) {
        cleanup_cgroups();
        return EXIT_FAILURE;
    }
    printf("Cgroups ready.\n\n");

    printf("1. Running outside function (unthrottled)...\n");
    double t_outside1 = burn_cpu(300000000);
    printf("   Unrestricted time: %.3f seconds.\n\n", t_outside1);

    printf("2. Calling limited_computation_function()...\n");
    limited_computation_function();
    printf("   Returned from function.\n\n");

    printf("3. Running outside function again (restored)...\n");
    double t_outside2 = burn_cpu(300000000);
    printf("   Unrestricted time: %.3f seconds.\n\n", t_outside2);

    printf("Cleaning up cgroups from /sys/fs/cgroup...\n");
    cleanup_cgroups();
    printf("Done.\n");

    return EXIT_SUCCESS;
}
```

## Recap

In this post, I demonstrate how to restrict
the resource usage of a C program. I then list
three methods to reach the goal with an explaination
of the scenario of each method.

As there are plenty of ways to do so, feel free
to leave your comments to share your approach
to restrict the resource access of your program!
