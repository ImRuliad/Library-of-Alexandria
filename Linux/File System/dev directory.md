## ***Table of Contents***








## ***Introduction***
---
*Linux and Unix OS follow a Filesystem Hierarchy Standard (FHS). FHS defines the structure of the filesystem. One of the most important directories on the system is `/dev/`*

## ***What is the Function of the /dev Directory?***
---
- Consists of files that represent devices that are attached to the local system
- Not regular files users can read/write to and from
- These files are called **device files** or **special files**

**Device Files**
*Device files are abstractions of standard devices that applications interact with via I/O system calls*
- Device files that correspond to hardware devices fall into two categories
- Character special files
- block special files

```
[root@localhost dev] ls -l
total 0
crw-r--r--. 1 root root     10, 235 Sep 29 12:08 autofs
drwxr-xr-x. 3 root root          60 Sep 29 12:07 bus
lrwxrwxrwx. 1 root root           3 Sep 29 12:10 cdrom -> sr0
drwxr-xr-x. 2 root root        3100 Sep 29 12:10 char
crw-------. 1 root root      5,   1 Sep 29 12:08 console
lrwxrwxrwx. 1 root root          11 Sep 29 12:07 core -> /proc/kcore
drwxr-xr-x. 3 root root          60 Sep 29 12:07 cpu
crw-------. 1 root root     10,  62 Sep 29 12:08 cpu_dma_latency
drwxr-xr-x. 7 root root         140 Sep 29 12:08 disk
brw-rw----. 1 root disk    253,   0 Sep 29 12:08 dm-0
...
```


## ***Character Special Files & Block Special Files***
---

**Character Special Files**
- Simple interfaces to character devices
- Driver communicates by sending single characters as data such as bytes
- Don't require buffering when communicating with a driver
- denoted by the letter 'c' in the `ls -l` command

**Block Special Files**
- Simple interfaces to block devices
- Driver communicates by sending entire blocks of data
- Requires buffering when communicating with a driver
- Accesses data from block devices through a cache
- denoted by the letter 'b' in the `ls -l` command

```
[root@localhost dev] ls -l
total 0
crw-r--r--. 1 root root     10, 235 Sep 16 11:52 autofs
drwxr-xr-x. 2 root root         280 Sep 16 11:52 block
drwxr-xr-x. 2 root root          60 Sep 16 11:52 bsg
drwxr-xr-x. 3 root root          60 Sep 16 11:52 bus
lrwxrwxrwx. 1 root root           3 Sep 16 13:05 cdrom -> sr0
drwxr-xr-x. 2 root root        3100 Sep 16 13:05 char
crw-------. 1 root root      5,   1 Sep 16 11:52 console
lrwxrwxrwx. 1 root root          11 Sep 16 11:52 core -> /proc/kcore
drwxr-xr-x. 3 root root          60 Sep 16 11:52 cpu
crw-------. 1 root root     10,  62 Sep 16 11:52 cpu_dma_latency
drwxr-xr-x. 7 root root         140 Sep 16 11:52 disk
brw-rw----. 1 root disk    253,   0 Sep 16 11:52 dm-0
brw-rw----. 1 root disk    253,   1 Sep 16 11:52 dm-1
brw-rw----. 1 root disk    253,   2 Sep 16 11:52 dm-2
...
```

## ***What Are Pseudo-Devices***
---
*Pseudo-devices don't correspond to hardware devices.*

- Pseudo-devices are character devices
- Can tell the difference between pseudo-devices and other character devices using the major number of a device
- Device drivers associate with a device through a unique value called a major number
- Major number appears first
- Minor number appears second

We can see the Major and Minor numbers of a device by running `ls -l`
```
[root@localhost dev] ls -l |grep random
crw-rw-rw-. 1 root root      1,   8 Sep 29 12:08 random
```

We can also view the Major and Minor numbers by running `stat`
```
[root@localhost dev] stat /dev/random
  File: /dev/random
  Size: 0         	Blocks: 0          IO Block: 4096   character special file
Device: 6h/6d	Inode: 10216       Links: 1     Device type: 1,8
Access: (0666/crw-rw-rw-)  Uid: (    0/    root)   Gid: (    0/    root)
Context: system_u:object_r:random_device_t:s0
Access: 2021-09-16 11:52:35.644848848 -0400
Modify: 2021-09-16 11:52:35.644848848 -0400
Change: 2021-09-16 11:52:35.644848848 -0400
 Birth: -
```

## ***Symbolic Links***
---
*Some device files in /dev/ don't appear as block or character devices, they are symbolic links instead*
```
[root@localhost dev] ls -lhaF| grep ^l 
lrwxrwxrwx.  1 root root           3 Sep 17 09:07 cdrom -> sr0
lrwxrwxrwx.  1 root root          11 Sep 17 09:06 core -> /proc/kcore
lrwxrwxrwx.  1 root root          13 Sep 17 09:06 fd -> /proc/self/fd/
lrwxrwxrwx.  1 root root          12 Sep 17 09:06 initctl -> /run/initctl|
lrwxrwxrwx.  1 root root          28 Sep 17 09:06 log -> /run/systemd/journal/dev-log=
lrwxrwxrwx.  1 root root           4 Sep 17 09:06 rtc -> rtc0
lrwxrwxrwx.  1 root root          15 Sep 17 09:06 stderr -> /proc/self/fd/2
lrwxrwxrwx.  1 root root          15 Sep 17 09:06 stdin -> /proc/self/fd/0
lrwxrwxrwx.  1 root root          15 Sep 17 09:06 stdout -> /proc/self/fd/1|
```

- `/dev/stdin` points to `/proc/self/fd/0`
- `/proc/self` directory sits on the `/proc` filesystem
- When a process reads `/proc/self` it gets info about itself using its process ID.
- The `proc/self/` directory is a symbolic link as well

```
[root@localhost dev] ls -l /proc/self/fd/0
lrwx------. 1 root root 64 Sep 17 12:02 /proc/self/fd/0 -> /dev/pts/0
```
- The subdirectory `.../fd` is a file descriptor
- file descriptors are unique references to a file or an I/O resource
- Helps to identify the file or I/O resource that a process has to work on
- `dev/pts/0` directory exists in memory
- The contents under `dev/pts` are pseudo-terminals that allow applications to receive input and display output
```
[root@localhost pts] ls -l /dev/pts
total 0
crw--w----. 1 root tty  136, 0 Sep 17 12:33 0
c---------. 1 root root   5, 2 Sep 17 09:06 ptmx
```

