---
title: FS - VFS system call
comments: true
date: 2018-04-01 00:38:52
type: categories
categories: Courses
tags: OS
---
### VFS system call

1.  Interfaces for specific File Systems, like ramfs, s5fs. 
2. 在Linux中，文件名字没有缀；“file”与“directory”无区别
3. 所有涉及“绝对路径”最后一级拆分的，会先将trailing “/” 去掉
5. lookup()这个函数作用巨大:
    - **在dir_inode中，找名字是“name”的entry，并把该entry变为vnode放到result中**
    - **ramfs_lookup => entry => vget(entry) => ramfs_read_vnode(vnode) => 完整vnode**
    - **VFS-AFS-VFS: “虚、实连接的小能手”**
    - **递归进行name resolution**
        - 问 parent inode：你是否包含名字为“name”的entry？
        - if Yes，把该entry以 child inode的形式返回
        - 递归进行lookup，直至找到目标name的inode
6. 以vn_mode来区别inode的类型** 

    ```
    S_IFDIR  directory   
    S_IFREG  regular   
    S_IFLNK  symlink  
    S_IFCHR  character special  
    S_IFBLK  block special 
    ```
4. 为什么 dir_namev() 与 lookup() 一起使用？直接使用open_namev()可以么？？


### 1. open
```
int do_open(const char *filename, int oflags)

输入：
    user process 传入的 filename
    user process 传入的 oflags: READ | WRITE | APPEND
输出： fd（ 可以是 file／directory ）

0. 设置 f_mode 有点麻烦：
    - f_mode 由传入的 oflags 的位信息组合而成三种结果：
        + FMODE_READ    1
        + FMODE_WRITE   2
        + FMODE_APPEND  4
    - oflags 有以下必须有以下前三者中一个；
       额外有creat bit、以及status bit
      (只需考虑O_CREAT，O_TRUNC跟随O_CREAT)：
        + O_RDONLY   0
        + O_WRONLY   1
        + O_RDWR     2
        + O_CREAT    0x100 /* Create if non-exist */
        + O_TRUNC    0x200 /* Truncate to 0-length */
        + O_APPEND   0x400   /* Append to file. *／
    - 注意：O_CREAT 不必写到 f_mode 中去；这里的O_CREAT是
           告知 kernel：如没有此文件，帮我创建一个；而新建
           是由 open_namev() 根据 oflags=O_CREAT 执行的
    - 设置 f_mode 流程：
        + 首先，通过“与运算&”，取得oflags 低位的位信息；
        + 若不是 RD/WR/RDWR中的任一个，返回-EINVAL
        + 若合法，则使用 if-else 将后三位确定；
        + 然后，获得高位的位信息，取得creat/status bit
        + 若是 O_APPEND,当前f_mode“或运算|”上O_APPEND位
        + 不必考虑是否为 O_CREAT，open_namev()会检查
        + 如果不是其中任何一个，高位置 “0”
        + 使用 if-else 设置高位信息
1. 获得可用的 fd
2. 获得新的 open_file_conn(kernel全局唯一的open file table)
3. 更新 curproc->p_file[fd] = open_file_conn
4. 检查 file mode， based on oflags
5. 调用 open_namev(filename, oflags, &vno, NULL)
    - open_namev()调用 path_namev()
    - open_namev()调用 lookup() 通用接口
    - lookup() 通用接口调用 specific FS ramfs_lookup()
    - specific FS ramfs_lookup() 在当前 vnode 下，查找名字是“name”的 vnode，将结果存到result中
    - specific FS ramfs_lookup()找到了该“name”的entry，根据"name:vnode_num" pair，再次调用 vget(vnode_num)
    - vget中会调用ramfs_read_vnode(vnode_t *vn)该vnode的加载完成
    - vget( ) 内部创建一个“半成品”vnode，传入 ramfs_read_vnode( )
    - ramfs_read_vnode( ) 根据具体的file system将该vnode中的所有成员函数、成员变量补充完成
6. 至此，open_namev() 得到的vnode是“活的”、完整的
7. 最后，do_open中将open_conn的vnode指针指向刚刚创建的vnode
     f->f_vnode = vnode;
```

### 2. read
```
int do_read(int fd, void *buf, size_t nbytes)

输入：
    user process 传入的 fd
    user process 传入的 buf，用以接受从 file 读取的内容
    user process 传入的 nbytes，指明需要读的字节数
输出： 实际读入字节数、or 错误信息

0. 检查传入参数是否正确
1. 调用 fget(fd) 获得由OS维护的全局唯一的open_conn_table中对应的 file_conn
    - fget()会自动对该file_conn执行refcount++
2. 检查获得的file_conn，是否valid
    - 检查该file是否“可读”：return -EBADF;
    - 检查该file是否是“dir”，而非“data”：return -EISDIR;
3. 通过 file_conn->f_vnode，获得 vnode_table中的 vnode entry,
4. 调用获得的vnode的 vn_ops(函数指针) 中的对应的 read()
    - 虽然这里的 read() 为抽象的接口，
    - 但是在do_open()的时候specific FS 的read()已实现了该接口
5. 读操作完成后，需要移动 open_conn中的f_pos指针
6. 最后，调用fput()，更新 f_refcount；返回读操作的字节数
```

### 3. write
```
int do_write(int fd, const void *buf, size_t nbytes)

输入： 
    user process 传入的 fd
    user process 传入的 buf，用从buf向 file 写数据
    user process 传入的 nbytes，指明需要写的字节数
输出： 实际写字节数、or 错误信息

0. 检查传入参数是否正确
1. 调用 fget(fd) 获得由OS维护的全局唯一的open_conn_table中对应的 file_conn
    - fget()会自动对该file_conn执行refcount++
2. 检查获得的file_conn，是否valid
    - 检查该file是否“可写”：return -EBADF;
    - 检查该file是否是“dir”，而非“data”：return -EISDIR;
    - 同时应该检查是否为APPEND模式，需要调用do_lseek()重置cursor
3. 通过 file_conn->f_vnode，获得 vnode_table中的 vnode entry
4. 调用获得的vnode的 vn_ops(函数指针) 中的对应的 write()
    - 虽然这里的 read() 为抽象的接口，
    - 但是在do_open()的时候specific FS 的read()已实现了该接口
5. 写操作完成后，需要移动 open_conn中的f_pos指针
6. 最后，调用fput()，更新 f_refcount； 返回写操作的字节数



```

### 4. close
```

输入：user process 传入的 fd
输出： 0-成功；(-EBADF)-失败

1. 检查传入参数正确性
2. 更新 fd 对应的 open_conn 的信息
    - fput( curproc->p_files[fd] )
        + f->f_refcount--;
        + if(f->f_refcount==0) vput(curproc->p_files[fd]->f_vnode)
3. 将该process的fd table中该fd 置为 NULL
4. 返回 0 ：成功

```

### 5. duplicate a file_descriptor
```
int do_dup(int fd)

输入：user process 传入的 fd
输出： fd   OR  (-EBADF)-失败 

1. 检查传入参数正确性
2. 获得该 fd 对应的 open_conn
    - file_t *file = fget(fd);
3. 获得下一个可用的 fd
    - availFd = get_empty_fd(curproc);
4. 更新该 process 的 fd table 中 fd 对应的 open_conn
    - curproc->p_files[availFd] = file;
5. 返回 availFd

```

### 6. duplicate with a designated file_descriptor
```
int do_dup2(int ofd, int nfd)

输入：user process 传入的 fd
输出： nfd   OR  (-EBADF)-失败 

1. 检查传入参数正确性
2. 获得该 fd 对应的 open_conn
    - file_t *file = fget(fd);
3. 检查 nfd
    - 如果 ofd==nfd， 需要fput(file)，因为之前fget()了，  
      导致open_conn的refcount增加了；这里是同一个fd，要再变回去
    - 如果 nfd 在使用中：curproc->p_files[nfd] != NULL
      那么需要先将nfd原来对应的 open_conn 关了
        + do_close(nfd);
4. curproc->p_files[nfd] = file;
5. 返回 nfd

```

### 7. mknode
```
int do_mknod(const char *path, int mode, unsigned devid)

输入：path(node的绝对路径) | mode(S_IFCHR／S_IFBLK) |  devid(device identifier 对应该 node)
输出： 0:成功  负数：失败

1. 检查输入参数的正确性
    - 检查mode，必须只能在 S_IFCHR／S_IFBLK 两者之中取值
    - 检查path，不能超过 path_name 的最大长度
2. 获得该path对应的vnode，通过调用 dir_namev()
    - retval = dir_namev(path,&nameLen,&name,NULL,&dir);
    - 该dir路径不是我们想要驻足操作的地方，只是“瞟一眼”
        + vput(dir)：因为dir_namev()会将dir的refcount++
    - 检查该 path 是否合法；不合法的话提前return
3. 检查最后一级目录(dir)下是否已经有名字为name的目录
    - value_lookup = lookup(dir,name,nameLen,&result);
4. 如果有的话，value_lookup==0，表明最后一级路径不合法
    - vput(result)：因为lookup()会讲result的refcount++
5. 如果没有的话：value_lookup==-ENOENT，这是我们期望的情况：
    - 在当前result directory下调用 specific FS 的 mknod()创建相应node；
    - 返回
6. 【注意】熊的版本
    - vput() 不必写那么麻烦，写在最外层即可



```

### 8. make directory

```
int do_mkdir(const char *path)

输入：user process 传入的path(创建的node放在哪)
输出： specific FS的返回值

1. Use dir_namev() to find the vnode of the dir we want to make the new directory in.  

2. Then use lookup() to make sure it doesn't already exist.

3. Finally call the dir's mkdir vn_ops. Return what it returns.

```

### 9. remove directory
```
int do_rmdir(const char *path)

输入：user process 传入的path(最后一级是要将被删除的)
输出：specific FS的返回值

0. 删除 directory ( 下一条就是 删除 file )
    - 删除dir的操作是： target dir 的 parent dir 根据 target dir's name 执行删除操作
    - 不需考虑 target dir是否存在，parent dir的rmdir会handle（如下边的do_unlink()不一样）
1. 调用 dir_namev() 获得path最后一级vnode的parent directory
    - Use dir_namev() to find the vnode of the directory containing the dir to be removed.
2. 检查 dir_namev()的返回值，根据情况处理Error
3. 根据返回的vnode，调用specific FS 的rmdir()
    - call the containing dir's rmdir() v_op
    - 注意 name 不能是 "." or ".."
    - 调用 value_remove = dir->vn_ops->rmdir(dir, name, namelen);
4. 最后 处理 dir的 refcount
    - vput(dir);

```

### 10. unlink
```
int do_unlink(const char *path)

输入：user process 传入的path(最后的file是要将被删除的)
输出：specific FS的返回值

0. 删除 file ( 上一条就是 删除 directory )
    - 删除file的操作是： target dir 的 parent dir 根据 target file's name 执行删除操作
    - 需要考虑 target file是否存在，在调用parent dir的unlink()（如下边的do_rmdir()不一样）
1. 调用 dir = dir_namev() 获得目标 file 的dir；
    - 如果有错误，提前返回
    - 没有错误，vput(dir);
2. 调用 lookup(dir,name,&result)检查 目标file是否存在于该dir下
    - 不存在的话，提前返回
    - 如果该file是一个directory的话，vput(file)，提前返回
3. 根据返回的dir vnode，调用specific FS 的unlink()
    - call the containing dir's unlink() v_op
4. 最后 处理 dir的 refcount
    - vput(result);

```

### 11. link
```
int do_link(const char *from, const char *to)

【注意1】这里的link是hard link，就是源文件如果删除了，  
那么被hard link的file不会一同消失，复制的文件靠hard link
(即vnode中也存了一个指向硬盘中对应data block的inode的指针)
与硬盘中的inode建立了联系
【注意2】hard link的源文件只能是 file，不能是dir
输入：源文件绝对路径from、目标路径to
输出：0:成功； 负数：失败

1. 调用open_namev(from)，获得源文件file的 vnode
    - 检查源文件的 vnode，确保是file，不是dir
2. 调用dir_namev(to)，获得目标路径dir，以及银的文件名name
    - value_tolink = dir_namev(to, &namelen, &name, NULL, &to_dir)

3. 调用 lookup()，检查目标dir下，是否已经存在 名字是name的inode
    - 如果已经存在，提前返回
    - 如果不存在，执行下一步
4. 调用 specific FS 的 link() 操作
    - to_dir -> vn_ops -> link(from_vnode, to_dir, name, namelen)
5. 注意 from_vnode、to_dir 的refcount值

```

### 12. rename
```
int do_rename(const char *oldname, const char *newname)

【注意】这里不是改名字！！而是修改“硬盘文件”在file system中的绝对路径

输入：旧文件绝对路径oldname、旧文件绝对路径newname
输出：unlink()的返回值

1. 调用 do_link()，对旧文件建立一个hardlink到新文件
    - 检查hardlink是否创建成功
2. 调用 do_unlink()，将旧文件删除
3. 返回 do_unlink() 的返回值
 


```

### 13. change dir
```
int do_chdir(const char *path)

输入：当前process的新cwd的绝对路径
输出： 0:成功  负值：失败

【注意】改变current working directory，
       实际上是改变 process中vnode的指针

1. 调用open_namev()获得新cwd的vnode
    - 检查返回值是否valid
    - 检查该vnode，确保其实dir，非file
2. 更新之前的cwd的refcount
    - vput(curproc->p_cwd);
3. 更新process 的cwd
    - curproc->p_cwd = vno;


```

### 14. get dir entry
```
int do_getdent(int fd, struct dirent *dirp)

- 操作目录：给定文件，依次读取目录到 dirp 中
- 与之前的 read、write直接对文件操作不同，这是对“directory entry”的操作
- 使用场景：搜索第8个目标子目录，对该目录依次调用8次该函数

输入：fd、dirp（下面的三个信息作为成员变量存在dirp中）
输出：returns the amount that offset should be increased by to obtain the next directory entry with a subsequent call to readdir
【注意0】每个FS的 dir_inode 实际上维护着一个entry array，
        但是总数有上限：RAMFS_MAX_DIRENT；
        因此只要一直调用do_getdent()，有可能读到 NULL值
        即：“没有entry了怎么办？管他呢！一直读。。。”
        [====Oh my god！不是很确定。。。]
        
【注意1】从fd所代表的directory依次读取它里边的entry；
        开始时从头读取entry，信息放到dirp结构体中；
        之后entry的读取起始位置offset，
        会由上一次该函数的调用自动更新f_pos += ent_size;，
        从而实现顺序读取entry的目的
【注意2】这个函数比较叼～
       - 读取到fd对应的inode下的entry的 name（ 不论 file／dir ）
       - 读取到了这个file在硬盘中对应的 entry inode number
       - 读取到该目录下一个 entry的起始offset

1. 调用fget()获得 open_conn
2. 由open_conn进而获得 vnode
3. 由vnode进而调用 specific FB的 readdir()，获得第一个 entry，存到dirp

```

### 15. lseek
```
int do_lseek(int fd, int offset, int whence)

输入：fd、偏移量offset、从哪里计算偏移whence
输出： 0:成功

1. 调用 fget() 获得 open_conn
2. 直接对相应的 open_conn进行操作，不需要往下深入到vnode
3. 根据 whence 情况，设定 new_cursor
    - whence==SEEK_SET， new_pos = offset;
    - whence==SEEK_CUR，new_pos = file->f_pos + offset; 
    - whence==SEEK_END，new_pos = file->f_vnode->vn_len + offset;
    - else，出错了。fput()
4. 更新open_conn的f_pos、以及refcount
    - file->f_pos = new_pos;
    - fput(file);

```

### 16. statistic
```
int do_stat(const char *path, struct stat *buf)

输入：需要统计的绝对路径path、用于存储统计信息的stat
输出：specific FS 的stat()返回值

1. 调用open_namev()，获得该path的vnode
    - open_namev(path, 0,&res_vnode,NULL);
    - 检查返回值
2. 由拿到的 vnode，调用specific FS的stat()
    -  res_vnode->vn_ops->stat(res_vnode, buf);
3. 更新res_vnode的refcount
    -  vput(res_vnode);
```




