## An introduction to (Linux) containers

---

## What is/does a container?

### Some kind of 

 - "lightweight" VM?
 - separate runtime environment?
 - package format?
 - a better chroot?

+++

### Actually can feel like an own VM

 - can run its own processes
 - you can SSH into it (if sshd is running)
 - processes in a container usually cannot look outside

+++

### But it does

 - not have an own OS kernel
 - not need an own init system
 - not use hardware isolation

+++

Instead, there are other mechanisms at play.

---




### There are no containers!

@[1](At least not in the Linux kernel.)

---

### Instead, in Linux, there are the concepts of 


+++
# cgroups (control groups)

+++
### Subsystem for resource control

 - CPU, memory, block I/O, ...
 - each resource type has its own hierarchy
 - each node is a group of processes
 - per default, all processes start at the root node

As the root node is always present, you are always in a container, even if you don't use containers!

+++

```
cpu                     memory
|--- background         |--- 27
|    |--- cron          |--- 785
|    |    |-- 27        |--- 56
|    |--- monitoring    |--- 57
|         |-- 785       |--- 58
|--- foreground         |--- databases
     |--- nginx              |-- 854
     |    |-- 56             |-- 1563
     |    |-- 57
     |    |-- 58
     |--- postgres
          |-- 854
     |--- redis
          |-- 1563
  

```
+++
### cgroup controls how many resources can be consumed by each node

e.g.

 - memory cgroup `database` is allowed to take 512MB of memory
 - CPU cgroup `foreground` is allowed to take 1.5 CPU shares

+++

There are other cgroups, like block I/O, devices, freezer, but the most "visible" thing when getting into container-land are not cgroups, but

+++
# Namespaces

 - used to separate resources from each other
 - each resource only sees other resources if they are in the same namespace
 - everything outside its namespace doesn't exist for a process (if you disregard Meltdown and Spectre ;-))

+++

## Some Types of Namespaces

+++

### PID namespaces

 - process ids
 - each process only sees other process in the same pid namespace
 - the first process in a namespace has pid 1 (similar to init)
 - however that same process has a different pid in the root namespace

+++

### Network namespaces

 - separate network resources from each other
 - have their own virtual eth0 interfaces (vethXXXX on the host)
 - own routing tables, own iptables rule
 - localhost is within the namespace, no longer on the complete host

+++

### Mount namespaces

 - chroot (but better)
 - cannot see the filesystem outside the namespace
 - processes have their own private rootfs & other mounts

---

Quick Namespace Demo

Note:

unshare --pid --fork bash

ps -A   # what??? still all the PIDs?
# try killing a process (and show it doesn't work)

mount -t proc proc /proc


---
# Container runtimes

+++

## LXC

 - set of userland tools
 - create a root filesystem with a whole OS inside (sans kernel)
 - "system" containers - intended to behave like a VM
 - advanced features nowadays, like live migration between hosts
 - Google Cloud actually uses LXC containers for its IaaS Linux instances

+++

## Docker Engine

 - the most obvious one
 - "application" containers - usually one application per container, not a whole OS
 - provides advanced features like image & build management

+++

## rkt

 - slim engine
 - focus on just executing containers
 - no image management, etc...

---

# Container Storage

+++

### Containers use CoW (Copy-on-Write) Storage

 - base image from which container gets started
 - all changes that happen in the container are done in a delta FS
   - AUFS
   - device-mapper
   - OverlayFS
   - btrfs

+++

### Container images contain everything that an application needs

 - OS dependencies (basic libraries)
 - application frameworks
 - application binaries themselves

You could see a Docker image essentially as a packaging format with the big plus of portability!

+++

### But not

 - configuration files
 - secrets

You wouldn't want your portable app image to leak your config & secrets between environments, much less outside the company!

+++

When a container is removed, all its changes are gone.

So, how do we persist storage and get configuration inside, then?

+++

External volumes & environment variables
 
```
docker run -d --name http -e TEST=new -v /tmp/nginx:/usr/share/nginx/html:ro -p 8080:80 nginx
```

---

# Bonus

+++

### "Old-school" enterprise applications and containers

Does that work?

+++

# Oh yeah!



---

# Thank you!
