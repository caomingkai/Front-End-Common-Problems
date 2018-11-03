---
title: FS - RAM File System
comments: true
date: 2018-04-01 12:23:19
type: categories
categories: Courses
tags: OS
---

## 0. What?
A specific file system, using RAM
提供 file system "统一接口"的"具体实现"，即：Vnode ops函数指针的具体实现


## 1. filesystem structure
- 成员变量：相当于 superblock
    
    ```c
    struct ramfs {
        ramfs_inode_t *rfs_inodes[RAMFS_MAX_FILES];  /*        
        Array of all files */
    } ramfs_t;
    ```
- 成员函数
    
    ```
    static fs_ops_t ramfs_ops = {
        .read_vnode   = ramfs_read_vnode,
        .delete_vnode = ramfs_delete_vnode,
        .query_vnode  = ramfs_query_vnode,
        .umount       = ramfs_umount
    };
    
    ```
    
---

## 2. 'inode' structure
#### 成员变量：相当于 inode
    
```
typedef struct ramfs_inode {
    off_t     rf_size;       /* Total file size */
    ino_t     rf_ino;        /* Inode number */
    char     *rf_mem;        /* Memory for this file (1 page) */
    int       rf_mode;       /* Type of file */
    int       rf_linkcount;  /* Number of links to this file */
} ramfs_inode_t;
```
  
#### 成员函数:
    
```
static vnode_ops_t ramfs_dir_vops = {
        .read = NULL,
        .write = NULL,
        .mmap = NULL,
        .create = ramfs_create,
        .mknod = ramfs_mknod,
        .lookup = ramfs_lookup,
        .link = ramfs_link,
        .unlink = ramfs_unlink,
        .mkdir = ramfs_mkdir,
        .rmdir = ramfs_rmdir,
        .readdir = ramfs_readdir,
        .stat = ramfs_stat,
        .fillpage = NULL,
        .dirtypage = NULL,
        .cleanpage = NULL
};
    
static vnode_ops_t ramfs_file_vops = {
        .read = ramfs_read,
        .write = ramfs_write,
        .mmap = NULL,
        .create = NULL,
        .mknod = NULL,
        .lookup = NULL,
        .link = NULL,
        .unlink = NULL,
        .mkdir = NULL,
        .rmdir = NULL,
        .stat = ramfs_stat,
        .fillpage = NULL,
        .dirtypage = NULL,
        .cleanpage = NULL
};

```

---
          
## 3. Specific Filesystem operation
```
1. void ramfs_read_vnode(vnode_t *vn)

    - 这一步让 generic vnode 有了生命，具备file system specific operations
    - 根据传入的vnode中的vnum、fs参数;
    - 由该specific file system 对该vnode加载上其自身的 inode信息、vnode_operations等等；
    - 使该 vnode 成为完整的对象


2. void ramfs_delete_vnode(vnode_t *vn);
    - 从硬盘删除该inode

3. int ramfs_query_vnode(vnode_t *vn);
    - 查看该vnode是否在硬盘中有相应的inode


```

---

## 4. vnode operation

0. entry = VNODE_TO_DIRENT(vndoe dir);
   拿到该 vnode dir 所属的entry，即 [name:inode_num] pairs  
   需要使用循环，来遍历所有的entry；无法random access
   
1. ramfs_create(vnode\_t \*dir, const char \*name, size\_t name\_len, vnode\_t \**result)

    ```
    - 调用ramfs_lookup(),确定该dir中没有‘name’名字的 subdir；  
    - 在当前vnode dir中找第一个可用的subdir entry; 一个dir含有的entry不能超过 RAMFS_MAX_DIRENT
    - 调用 inode_num=ramfs_alloc_inode() 创建新的inode
    - 调用 vget(inode_num)
        - vget中会调用ramfs_read_vnode(vnode_t *vn)该vnode的加载完成
    - 设置subdir的名字，完成dir creation
    ```
    
2. ramfs\_mknod(struct vnode \*dir, const char \*name, size\_t name\_len, int mode, devid\_t devid)

    ```
    - 调用ramfs_lookup(),确定该dir中没有‘name’名字的 subdir；  
    - 在当前vnode dir中找可用的subdir entry; 一个dir含有的entry不能超过 RAMFS_MAX_DIRENT
    - 调用 inode_num=ramfs_alloc_inode() 创建新的inode
    - 不需要调用 vget(inode_num)，不需要加载vnode，因为其为特殊的special file【character、bulk】
    - 设置subdir的名字，完成dir mknod
    ```
    
3. ramfs\_lookup(vnode\_t \*dir, const char \*name, size\_t namelen, vnode\_t \**result)

    ```
    - 注意：这个函数很重要：是 name resolution 的重要一步；负责从 “name ==> vnode_num ==> vnode” 的解析。
    - 也是在这一步，这也是vget()的主要应用场景
    - vget()，又会调用 AFS的ramfs_read_vnode(),从而使返回vnode“活起来”，是vnode完整。
    - 至此，ramfs_lookup() 完成了 VFS-AFS-VFS 的旅程
    - 查看该‘dir'vnode下是否有名字为’name‘的vnode，结果放到result  
    - 调用VNODE_TO_RAMFSINODE，根据dir vnode或得 inode
    - 遍历该 inode 下属的entries，看是否有该“name”的 entry
    - if YES，调用vget(entry.ino_num)返回该vnode到result
    - vget中会调用ramfs_read_vnode(vnode_t *vn)该vnode的加载完成
    - vget( ) 内部创建一个“半成品”vnode，传入 ramfs_read_vnode( )
    - ramfs_read_vnode( ) 根据具体的file system将该vnode中的所有成员函数、成员变量补充完成
    - 至此，vget() 返回的vnode是“活的”、完整的
    ```
    
4. ramfs\_link(vnode\_t \*oldvnode, vnode\_t \*dir, const char *name, size_t name_len)

    ```
    - sets up a hard link. it links oldvnode into dir with the specified name.  
    - 调用ramfs_lookup()保证“name”不在“dir”中
    - 遍历当前vnode dir，找第一个可用的subdir entry; 一个dir含有的entry不能超过 RAMFS_MAX_DIRENT
    - 将新拿到的entry的inode_num，设置为oldvnode的vnode_num
        entry->rd_ino = oldvnode->vn_vno;
    - 设置名字
        strncpy(entry->rd_name, name, MIN(name_len, NAME_LEN - 1));
        entry->rd_name[MIN(name_len, NAME_LEN - 1)] = '\0';
    - 增加 linkcount
        VNODE_TO_RAMFSINODE(oldvnode)->rf_linkcount++;
    ```

5. ramfs\_unlink(vnode\_t \*dir, const char \*name, size\_t namelen)

    ```
    - removes the link to the vnode in dir specified by name  
    - 调用ramfs_lookup()保证“name”在“dir”中
    - 遍历当前vnode dir，找到该entry，并将其名字设置为空
        entry->rd_name[0] = '\0';
    - 减少 linkcount
        VNODE_TO_RAMFSINODE(oldvnode)->rf_linkcount++;

    - 调用vput(vn);处理相应的vnode
        - 减少 vn_refcount
        - 当 vn_refcount==0 时候，调用 ramfs_delete_vnode()
    
    ```
    
6. ramfs\_mkdir(vnode\_t \*dir, const char \*name, size\_t name\_len)

    ```
    - mkdir creates a directory called name in dir
    - 调用ramfs_lookup(),确定该dir中没有‘name’名字的 subdir；  
    - 在当前vnode dir中找第一个可用的subdir entry; 一个dir含有的entry不能超过 RAMFS_MAX_DIRENT
    - 调用 inode_num=ramfs_alloc_inode() 创建新的inode
    - Set entry in its parent: dir
        - 关联inode，entry->rd_ino = inode_num;
        - 设置名字，strncpy(entry->rd_name, name, MIN(name_len, NAME_LEN - 1));
            entry->rd_name[MIN(name_len, NAME_LEN - 1)] = '\0';
    - Set up '.' and '..' in the directory
    - 设置subdir的名字，完成dir mknod
    ```

7. ramfs\_rmdir(vnode\_t \*dir, const char \*name, size\_t name\_len)

    ```
    - removes the directory called name from dir. the directory to be removed must be empty (except for . and .. of course).
    - 调用ramfs_lookup(),确定该dir中没有‘name’名字的 entry，并获得 inode；
    - 调用S_ISDIR(),确保该name是一个directory 
    - 遍历该inode下的entry，make sure that this directory is empty
    - 便利parent dir，找到对应的name，将其删除
        - 删除名字
        - vput(vn)
    ```

8. ramfs\_read(vnode\_t \*file, off\_t offset, void \*buf, size\_t count)

    ```
    - read transfers at most count bytes from file into buf. It begins reading from the file at offset bytes into the file.
    - 调用VNODE_TO_RAMFSINODE(),获得vnode对应的inode
    - 确保inode不是directory
    - 将inode中的内容inode->rf_mem复制到buf中
         memcpy(buf, inode->rf_mem + offset, ret);
    ```

9. ramfs\_write(vnode\_t \*file, off\_t offset, const void \*buf, size\_t count)

    ```
    - transfers count bytes from buf into file.
    - 调用VNODE_TO_RAMFSINODE(),获得vnode对应的inode
    - 确保inode不是directory
    - 将buf中的内容复制到inode中的inode->rf_mem
         memcpy(inode->rf_mem + offset, buf, ret);
    ```

10. ramfs\_readdir(vnode\_t \*dir, off\_t offset, struct dirent \*d)

    ```
    - 在当前dir中，从offset处，“挨个”读取dir中的entry到数据结构d中；成功读取某个entry后，返回值是下一次调用该函数时应该起始的offset（相当于加了刚刚读取entry/inode的file_size）
    - reads one directory entry from the dir into the struct dirent. On success, it returns the amount that offset should be increased by to obtain the next directory entry with a subsequent call to readdir. 
    - 确保inode是directory
    - 将buf中的内容复制到inode中的inode->rf_mem
         memcpy(inode->rf_mem + offset, buf, ret);
    ```

11. ramfs\_stat(vnode\_t \*file, struct stat \*buf)

    ```
    - sets the fields in the given buf, filling it with information about file
    - 在 stat 中填入各种统计信息
    ```
