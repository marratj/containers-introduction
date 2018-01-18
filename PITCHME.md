## An introduction to (Linux) containers

---

### What is/does a container?

Some kind of 

 - "lightweight" VM?
 - separate runtime environment?
 - package format?
 - a better chroot?

+++

Actually can feel like an own VM

 - can run its own processes
 - you can SSH into it (if sshd is running)
 - processes in a container usually cannot look outside

+++

But it does

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

The last point also means, you are always in a container, even if you don't use it!

+++
# Namespaces

 - Used to separate resources from each other

 - Each resource only sees other resources if they are in the same namespace

+++

## Types of Namespaces

+++

### PID namespaces

 - process ids
 - each process only sees other process in the same pid namespace
 - the first process in a namespace has pid 1 (similar to init)
 - however that same process has a different pid in the root namespace

+++

### Network namespaces

 - separate network resources from each other
 - have their own virtual ethernet interfaces


---

# Thank you!
