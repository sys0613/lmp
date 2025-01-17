---
layout: post
title: mountsnoop
date: 2022-10-10 16:18
category: bpftools
author: yunwei37
tags: [bpftools, syscall, perf-event, tracepoint]
summary: mountsnoop traces the mount() and umount syscalls system-wide.
---


## origin

origin from:

https://github.com/iovisor/bcc/blob/master/libbpf-tools/mountsnoop.bpf.c

## Compile and Run

 
Compile:

```shell
docker run -it -v `pwd`/:/src/ yunwei37/ebpm:latest
```

Run:

```shell
sudo ./ecli run /package.json
```

TODO: support enum types in C

# details in bcc

Demonstrations of mountsnoop.

mountsnoop traces the mount() and umount syscalls system-wide. For example,
running the following series of commands produces this output:
```console
# mount --bind /mnt /mnt
# umount /mnt
# unshare -m
# mount --bind /mnt /mnt
# umount /mnt

# ./mountsnoop.py
COMM             PID     TID     MNT_NS      CALL
mount            710     710     4026531840  mount("/mnt", "/mnt", "", MS_MGC_VAL|MS_BIND, "") = 0
umount           714     714     4026531840  umount("/mnt", 0x0) = 0
unshare          717     717     4026532160  mount("none", "/", "", MS_REC|MS_PRIVATE, "") = 0
mount            725     725     4026532160  mount("/mnt", "/mnt", "", MS_MGC_VAL|MS_BIND, "") = 0
umount           728     728     4026532160  umount("/mnt", 0x0) = 0

# ./mountsnoop.py -P
COMM             PID     TID     PCOMM            PPID    MNT_NS      CALL
mount            51526   51526   bash             49313   3222937920  mount("/mnt", "/mnt", "", MS_MGC_VAL|MS_BIND, "", "") = 0
umount           51613   51613   bash             49313   3222937920  umount("/mnt", 0x0) = 0
```
The output shows the calling command, its process ID and thread ID, the mount
namespace the call was made in, and the call itself.

The mount namespace number is an inode number that uniquely identifies the
namespace in the running system. This can also be obtained from readlink
/proc/$PID/ns/mnt.

Note that because of restrictions in BPF, the string arguments to either
syscall may be truncated.
