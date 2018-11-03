---
title: 'FS - File System: Name Resolution'
comments: true
date: 2018-04-01 12:21:30
type: categories
categories: Courses
tags: OS
---
## lookup()

```
int lookup(vnode_t *dir,  char *name, 
           size_t   len,  vnode_t **result)

```

#### 1. 使 vnode “活”起来

lookup()、vget()、ramfs\_read\_vnode() 一起使 vnode “活起来，成为完整 vnode”
#### 2. lookup 流程：

在给定的 dir\_vnode 下，查找是否存在名字是"name"的 vnode

- 先调用 ***ramfs\_lookup( )*** 查找 dir_vnode 下是否有相应 entry
- 后根据 entry.ino，调用***vget(fs,ino)*** 查找该 inode_num对应的 inode
    1. if exist, 更新 refcount， 直接返回 vnode
    2. if none, 创建 new vnode，填充部分信息；传给ramfs\_read\_vnode()
- 调用 ***ramfs\_read\_vnode( )*** 获得完整vnode

#### 3. name resolution: 虚实连接、递归查找
- lookup() 会调用 ramfs\_lookup (a specific FS)，由AFS代为查找
- 递归进行name resolution：查找到相应的inode之后，继续以得到的inode查找下一个那么，



## dir_namev()

```
int dir_namev(const char *pathname, size_t *namelen, 
              const char **name, vnode_t *base, 
              vnode_t **res_vnode)
```

#### 1. man dirname
> it deletes the filename portion, beginning with the last slash '/' character to the end of string (after first stripping trailing slashes), and writes the result to the standard output.

- 提取 pathname 的 **parent directory inode**；  
- 同时保存最后一级 **directory/file 的 name**  

#### 2. 三个例子：

```
1. 正常情况
$ dirname /Users/kevin/Desktop/html-DOM.jpg 
/Users/kevin/Desktop

2. 即便文件名字后边有 “／”，会先将“／”去掉
$ dirname /Users/kevin/Desktop/html-DOM.jpg/
/Users/kevin/Desktop

3. 不但对文件(file)适用，对目录(directory)也适用
$ dirname /Users/kevin/Desktop/
/Users/kevin

```

#### 3. refcount


- 该函数如果成功返回的话，会使 res_vnode，也就是parent directory inode的 refcount++

- 因为函数内部会逐级调用 lookup() 进行 AFS call 以及  vput(base) 来抵消lookup(base)，给每级base带来的refcount++
- 但是lookup()找到了parent directory inode 的时候，刻意不调用vput(base)，是最后一级(即res_vnode)的refcount加“1”
- 这是有原因的：用户调用dir_namev()，就是要对这个dir就行操作的，需要停留在这里，因此需要对硬盘中相应的对象做相应的refcount更新

#### 4. dir_name() 流程

```
int dir_namev(const char *pathname, size_t *namelen, 
              const char **name, vnode_t *base, 
              vnode_t **res_vnode)
```

1. 根据 ***base*** 的值确定查找的起始节点
    - base == NULL, base = curproc -> p_cwd;
    - 如果 pathname 以 "/" 开始，忽略 base 的值；base = vfs_root_vn;
2. 更新 base 的refcount => vref(base);
3. 依次提取 pathname 中 name 部分，在当前 base inode 下查找是否存在对应 entry
    - 维护一个 start_ptr：指向"/"之后第一个位置
    - 维护一个 end_ptr： 指向下个"/"之前第一个位置
    - [start, end] 之间就是 name 部分
4. 调用 lookup 查找该 base vnode之下，是否有 name 的 vnode
    - 返回值 != 0, 失败，提前return
    - 返回值 == 0, 成功，递归查询下一级 name
    - 更新本次查询使用的 base 的refcount
5. 递归查询：将此次得到的 vnode 设置为下一次查询的 base
6. 直到start_ptr >= len，表明路径解析完成，跳出
    - 注意：在循环中的 lookup() 之前跳出
    - 因为这是查找上级 dir inode，并非查找最后一级inode
7. 检查最后得到的 inode 是否为 directory
8. 填充相应的返回值；返回 0(成功)


## open_namev()

```
int open_namev(const char *pathname, int flag, 
               vnode_t **res_vnode, vnode_t *base)

- 输入：
    + pathname：指定要打开的绝对路径
    + flag=O_CREAT && name 不存在，那么创建新的
    + base：指定从那个vnode 目录下开始解析路径
    
- 输出：将打开的 vnode 存在 res_vnode 中

```

#### 1. man open

> opens a file, a directory or even an URL, just as if you had double-clicked the file's icon

- 打开一个文件/目录/URL，相当于 Linux 的 open 指令
- open_namev = dir_namev + lookup + (create)

#### 2. open_namev 流程
1. 调用 dir_namev() 解析 pathname
    - retcode = dir_namev(pathname, &namelen, &name, base, &dir);
    - 输入参数的错误处理由 dir_namev() 解决了
    - 检查返回值，如果没错误，更新 refcount
    - 继续下一步
2. 根据刚得到的dir，调用 lookup() 打开dir下的该vnode
    - retVal = lookup(*dir,name,len,res_vnode);
    - 如果存在的话：将 vnode 返回到 res_vnode 中
    - 如果不存在的话，同时又满足(flag & O_CREAT) == O_CREAT，那就创建新的vnode
        - 调用(\*dir)->vn\_ops->create(*dir,name,len,res\_vnode);
    - return retval



