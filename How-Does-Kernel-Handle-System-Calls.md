---
title: How Does Kernel Handle System Calls
comments: true
date: 2018-04-13 11:52:25
type: categories
categories: Courses
tags: OS
---

## I. An Example: read( )

1. **User space:** User process makes `read()` system call
2. **Interrupt:** This will **trap** into kernel by **software interrupt**
3. **System API:** Kernel executes its `interrupt handler`, in which `sys_read() API` will be called
    - `copy_from_user()` the read\_args(fd/buff/...)
    - `page_alloc()` a temporary buffer
    - call `do_read()`, and `copy_to_user()` the read bytes
    - `page_free()` that buffer
    - return the number of bytes actually read, or if anything goes wrong
4. **File system API**: Specific file system function `do_read()` will be executed.
    - 注意：`do_read()`是 file system 的 system call

## II. System API for file operation
   
#### 1. System Read API

```c 
int sys_read(read_args_t *arg)

kernel “读”操作的API，自此由kernel执行一系列“读”相关的操作

输入：arg，包含了user process read()的所有参数
输出：实际读出的字节数
```

1. 作为 interrupt routine 被调用；发生于 user process `read()` leading to *trap as software interrupt* into kernel 的时候
2. `copy_from_user()` 读取 user process `read()`的传入参数
3. 在 kernel address space 分配一块buffer，作为临时接收数据的区域
4. 调用 file system 的 `do_read()`，从 disk 中 读取数据到 buffer
5. `copy_to_user()`,将buffer 中读取到的数据 copy 到 user address space（低地址空间 0xc0000000 - 0xc0000000）
6. `page_free()`,将 buffer 释放
7. 根据 file system 读取的结果，返回ERROR 或者 读取的字节数


#### 2. System Write API

```c
int sys_write(write_args_t *arg)

kernel “写”操作的API，自此由kernel执行一系列“写”相关的操作

输入：arg，包含了user process write()的所有参数
输出：实际写入的字节数
```

1. 在kernel address space 开辟临时buffer，读取放置user 要写的数据
2. 从 user address space 拷贝 `read()` 输入参数，包括要写的数据
3. 调用 file system 的`do_write()`操作， write to disk
4. 释放 kernel address space 的临时 buffer
5. 返回ERROR 或者 写入的字符数

#### 3. System Get Directory Entry

```c
int do_getdent(int fd, struct dirent *dirp)

对目录文件的操作，比如：
- 获得位于指定字节数位置的子目录entry
- 显示某个文件夹下所有子目录名称、大小等信息

输入：fd(要读取的文件指针)、dirp(目标dir entry的容器)
输出：count_bytes

```

1. `copy_from_user()`从 user address space 读取参数到 kernel address space（从而知道用户都想干些啥？） 
2. 循环调用 `do_getdent()` 以及 `copy_to_user()`，直到达到user process 指定的 字节数目
    - `do_getdent()` 调用 file system API 读取目录，并将结果放在 dir 中
    - `copy_to_user()`将 dir 中的信息 copy 到 user address space 中去；这样就会在用户的 fd 下逐渐显示出读入的 sub dir entry 的名字、大小等信息
    - 读入的 count_bytes >= user_args.count 时，跳出循环
3. 返回实际读取到的目录所占字节数 count_bytes



#### 4. Other System APIs related to File System
```c
int sys_close(int fd)
int sys_dup(int fd)
int sys_dup2(const dup2_args_t *arg)
int sys_mkdir(mkdir_args_t *arg)
int sys_rmdir(argstr_t *arg)
int sys_unlink(argstr_t *arg)
int sys_link(link_args_t *arg)
int sys_rename(rename_args_t *arg)
int sys_chdir(argstr_t *arg)
int sys_lseek(lseek_args_t *args)
int sys_open(open_args_t *arg)

```

以上的 system API 都与 file system 有关。都经过以下过程：

1. User process makes system call: `read()`
2. Traps into kernel as software interrupt: `interrupt handler`
3. Kernel execute corresponding API: `sys_read()`
4. Copy from user address space parameters into kernel address space
4. Ask File System to complete relavent operation: `do_read()`
5. Copy to user address space with the result at kernel address space



## III. Other System API related to Virtual Memory System

```c
int sys_munmap(munmap_args_t *args)
void *sys_mmap(mmap_args_t *arg)
pid_t sys_waitpid(waitpid_args_t *args)
void *sys_brk(void *addr)
void sys_sync(void)
int sys_stat(stat_args_t *arg)
int sys_fork(regs_t *regs)
int sys_execve(execve_args_t *args, regs_t *regs)
int sys_kshell(int ttyid)
```

这些函数与“用户态” Process/ Thread/ Virtual Memory 的操作有关：

1. 由 user process 发起的 system call： `wait()`
2. 经过software interrupt trap into kernel
3. kernel 在调用相应的 system API，代替 user process 执行任务: `sys_waitpid()`
4. 交给“进程系统” Process& Memory System，来具体处理任务：`do_waitpid()`

## VI. Interrupt handler for syscalls 

```c
void syscall_handler(regs_t *regs)

很重要的一个函数：
1. 使“用户态”程序在 make syscall 的时候，进入“内核态”
2. 根据 syscall_number，调用相应的 system API 
```
