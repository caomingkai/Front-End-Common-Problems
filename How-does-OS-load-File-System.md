---
title: FS - How does OS load File System
comments: true
date: 2018-04-01 16:04:24
type: categories
categories: Courses
tags: OS
---

## BIOS(code stored on ROM)

At BIOS session, several tasks are executed: 

1. Power-on self test (POST), making suring: **memory, disk,screen,ect.** are in good condition    - so it knows where to load the boot program( on **DISK**)
    - also prepare **memory(run process)** and  **disk(boot program)**2. Load and transfer control to boot program
3. Provide drivers for all devices,

## Boot(code stored on DISK)

1. Set up stack, clear BSS, uncompress kernel, then transfer control to Kernel

2. **Idle process(pid=0)** is created; set up initial page tables, turn on address translation

3. Initialize rest of kernel
    - create vfs\_root\_vnode
    - set the "cwd" of idle/init process
    - make the null, zero, and tty devices using "mknod"
        + "/dev" 
        + "/dev/null"   "/dev/zero", 
        + "/dev/tty0"   "/dev/tty1"
    - create the **"init" process(pid=1)**,   
      which is the ancestor of all other user processes 
    - invoke the scheduler

## Kernel(code stored on disk)

1. Now the File System is ready to use
2. Also need to execute other configuration

