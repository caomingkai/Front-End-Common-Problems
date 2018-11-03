---
title: 'VM: How does fork() work?'
comments: true
date: 2018-04-18 15:12:11
type: categories
categories: Courses
tags: OS
---

## *fork*( ) in Kernel

跟`read()`一样: 都是user process调用kernel syscall，trap into kernel，在interrupt handler中指明interrupt routine，最终由kernel代为执行任务；

具体在kernle中要做的工作有以下几步：

1. 创建新的 PCB,`newProc = proc_create()`
2. 拷贝address space,`vmmap_copy(newProc)`
    - 拷贝 vmmap(address space)的vmarea(只初步copy memory area):
      `newProc->p_vmmap = vmmap_clone(curproc->p_vmmap);`
    - 拷贝每个vmarea对应的mmobj, from curproc
        + 两个process的vmarea是一一对应的，遍历copy
        + 首先获得即将要拷贝的curproc的第一个vmarea
        + 从newProc的第一个vmarea开始，执行copy： 
           `curproc->vmarea->vma_obj`
        + 更新vma_obj的refcount
        + 判断要该vmarea是否为Private,执行**shadow copy**
            1. if private，执行shadow copy
            2. 并且要新vmarea插入到bottom\_mmobj->mmo_vmas之后
3. 清除 curproc 的pageTable 以及TLB缓存(必须在thread copy之前)
    - 因为curproc的pageTable也会被copy(在thread copy的时候)  
      若不重新映射pageTable的话，两个process的vaddr会映射同一paddr
    - 既然pageTable需要重新映射，那么TLB也需要重新缓存新映射
4. 创建新thread，拷贝cur thread内容，包括 register、stack、pageTable
    - 拷贝parent的TCB到新的TCB：kth\_new,`kth_new = kthread_clone(curthr)`
    - 设置所属process，`kth_new->kt_proc = newProc`
    - 将新thread插入到process的“线程链”之后
    - 更新register信息（包括page table信息）
        + **其实pageTable是给CPU使用的(以下可看出)**
        + `kt_ctx.c_pdptr = newProc->p_pagedir`
        + `kt_ctx.c_eip = (uint32_t)userland_entry`
        + `kt_ctx.c_esp = fork_setup_stack(regs,kth_new->kt_kstack)`
        + `kt_ctx.c_kstack = (uintptr_t) kth_new->kt_kstack`
        + `kt_ctx.c_kstacksz = DEFAULT_STACK_SIZE`
5. 拷贝file system的内容
    - 拷贝curProc的file-descriptor-table
    - 注意这里是拷贝的 fd table的内容，并非 open-connection-table
    - 因为fd-table很大：NFILES，遍历拷贝
        + 如果 p->p_files[i] != NULL，表明对应实际open-file
        + 还需要更新该open-file的refcount
6. 拷贝p\_cwd，并更新p\_cwd的refcount:`p->p_cwd = curproc->p_cwd;`
    - 注意：这里的`p_cwd`在`proc_creat()`的时候复制了一遍了
    - 同时也要考虑`p_cwd=NULL`的情况如何处理
7. 设置 heap segment 的break，与 curproc 的一致
8. 将新thread加入到 run queue 中
9. 设置 parent proc的fork()返回值：newProc->p\_pid
