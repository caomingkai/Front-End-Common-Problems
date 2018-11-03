---
title: 'Tasks assignment: Kernel Virtual File System'
comments: true
date: 2018-03-30 01:16:38
type: categories
categories: Courses
tags: OS
---
    
## Knowledge frame
[Useful Link 1 on ***Workflowy***](https://workflowy.com/s/HdnT.WkLWxpmsKu) 

[Useful Link 2 on ***USNA***](https://www.usna.edu/Users/cs/wcbrown/courses/IC221/classes/L09/Class.html)   


{% asset_img file_system_big_pic.png  %}

{% asset_img 3_tables.jpg  %}

{% asset_img OFT.jpg  %}



## Tasks allocation
    
#### 1. vnode.c
    A: special_file_read()
    A: special_file_write()
    
#### 2. name.c
    A: lookup()
    A: dir_namev()
    A: open_namev()
    
#### 3. open.c
    A: do_open()

    
#### 4. vfs_syscall.c
    A: do_close()
    
    B: do_read()
    B: do_write()
    B: do_dup()
    B: do_dup2()
    
    C: do_mknod()
    C: do_mkdir()
    C: do_rmdir()
    C: do_unlink()
    C: do_link()
    
    D: do_rename()
    D: do_chdir()
    D: do_getdent()
    D: do_lseek()
    D: do_stat()


#### 5. main.c
    idleproc_run() ：2 

#### 6. proc.c
    proc_create()
