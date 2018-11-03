---
title: 'VM - Virtual Memory Allocation: brk() and mmap() '
comments: true
date: 2018-04-14 11:33:45
type: categories
categories: Courses
tags: OS
---

## Useful Link
Credit goes to:   
https://blog.csdn.net/gfgdsg/article/details/42709943

## Notice: page-granularity
All memory in virtual memory address space are **page-based**. It is the smallest unit of granularity, including the Heap segment in the following.

## Virtual Memory Allocation (2 ways):

1. `brk()` on Heap segment
2. `mmap()` mapped_file segment, between Heap and Stack

Usually user process calls `malloc()` to invoke kernel
system calls `brk()` or `mmap()` to get a virtual memory. Which one will be executed depends on **requesting memory size(128KB is the line)**

- *size<128KB*, kernel calls `brk()` to assign virtual memory on Heap to user process
    + **BAD:** only be sequentially assign and free, like Stack
    + **GOOD:** can be used as cache ()
- *size>128KB*, kerel calls `mmap()` to get a piece of memory between Heap and Stack to user process
    + **GOOD:** can independently assign and free (flexible)
    + **BAD:** repeately causing page_fault to involve physical memory (CPU costly)


## System API：do_brk( )

```c
int do_brk(void *addr, void **ret)

设置 process 地址空间中Heap Segment的break，即Heap的“堆顶”。  
- malloc()、free()均会调用该API，增加、减小heap break
- malloc()时向高地址增加; free()时向低地址减小

输入： addr 要更新的break地址； ret 更新后的break地址
输出：0-成功
```

1. 如果 addr == NULL，返回当前 brk
2. 判断 addr 的合法性：与 proc\_start\_brk比较，不能比他还小
3. 处理 addr 与 proc\_brk 在 page\_not\_aligned 的情况；并获得old proc\_brk所在 virtual area 的最后一页 end\_page
    - addr : newvfn (new virtual frame number)
    - curproc->p_brk: oldvma->vma_end
    - if(newvfn > oldvma->vma_end): 对应于`malloc()`
    - if(newvfn < oldvma->vma_end): 对应于`free()`
4. 最后更新两个量：
    - oldvma->vma_end = newvfn;
    - curproc->p_brk = addr;

