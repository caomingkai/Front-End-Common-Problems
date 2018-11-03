---
title: 'VM: Virtual memory + File system = Memory object(in VM)'
comments: true
date: 2018-04-18 15:07:05
type: categories
categories: Courses
tags: OS
---

## I. Virtual memory & File system

#### 1. file可以直接映射进memory，更快，但无cache。  
#### 2. 从disk读取文件有两种方法：`read()` and `mmap()`

- read(): 
    + user process： 系统调用`read()`，trap into kernel
    + kernel： 执行interrupt handler，调用do_read()；继而调用 file system的 read() 指令
    + kernel space：读入的内容放到 kernel address space
    + user space： kernel 将内容copy到 user address space
- mmap(): 
    + user process：系统调用`mmap()`，trap into kernel
    + kernel ：执行interrupt handler，调用 do_mmap()；继而调用 file system 的 mmap() 指令
    + user space： kernel 直接将 file 映射到 user address space中

#### 3. file system 中的 vnode 结构体：
    - 对应一个电脑文件系统中的一个 file
    - 其中还有一个 mmobj vn_mmobj 结构体： 是该file到address space 中的映射。 mmobj 包含：
        + mmo_operations
            - lookuppage：根据[mmobj:page#], 找对应的 pframe
            - fillpage：将 disk file 内容加载到 pframe
            - dirtypage：将该 mmobj 指定的 pframe 设置为 dirty
            - cleanpage：将该 mmobj指定的 pframe 清楚
            - ref
            - put
        + mmo_refcount
        + mmo_resident_pages
    - vnode 为抽象的 inode，来实现 polymophism。也有与file 相关的 operations：
        + read
        + write
        + mmap(): 注意这个mmap 是vnode层面的function；**实际上返回的是 vnode->vn\_mmobj; 而这个vn\_mmobj则是通过上述的fillpage()实际将 disk 中的数据加载进 pframe**



## II. vmmap.c


#### 1. vmmap_lookup

```c
vmarea_t * vmmap_lookup(vmmap_t *map, uint32_t vfn)

Find the vm_area that vfn(virtual frame number) lies in
```




#### 2. vmmap_map

```c
vmmap_map(vmmap_t *map, vnode_t *file, 
          uint32_t lopage, uint32_t npages, 
          int prot, int flags, off_t off, 
          int dir, vmarea_t **new)
- map 为cur process 的地址空间
- file 为被映射到虚拟内存的 vnode/shadow/anonymous obj
- lopage、npages 指明从地址空间的什么具体位置开始映射、映射长度
- prot、flag、off 指明 vmarea 的 read/write、public/private、 vmarea的起始地址
- new 为接收 映射file到虚拟内存的 mapped\_file\_obj 的指针

【注意】该函数被 system API:do_mmap()调用；完成
```
 

将指定的 file(anonymous／shadow) 映射map到该进程的“地址空间”address space中；经历一下步骤：

1. 现在cur\_proc的地址空间中开辟一块 vmarea 
    - if lopage=0: 开辟一个新的 vmarea
    - if lopage!=0: 如果该位置已经被别的vmarea占用，那么删除就的vmarea

2. 对刚获得的 vmarea 赋值：
    - newvma->vma_start = lopage;
    - newvma->vma_end = lopage + npages;
    - newvma->vma_off = ADDR_TO_PN(off);
    - newvma->vma_prot = prot;
    - newvma->vma_flags = flags;
    - list_link_init(&newvma->vma_plink);
    - list_link_init(&newvma->vma_olink);

3. 创建 mmobj（**这是 vmarea 存在的意义**）
    - if file==NULL: mmobj为新创建的 anonymous obj;  
    `mmobj = anon_create();`
    - if file!=NULL: mmobj为从vnode读入的 mmobj;  
    `file->vn_ops->mmap(file, newvma, &mmobj);`
    - 判断是否需要创建 shadow obj  
    ` if MAP_PRIVATE & flags != 0` 需要在 mmobj内部创建 shadow obj

4. 将mmobj 链接到 vmarea上；`newvma->vma_obj = mmobj;`

5. 将vmarea 插入到 cur\_proc 的 地址空间 map 中


#### 3. vmmap_remove()

```c
int vmmap_remove(vmmap_t *map, uint32_t lopage, uint32_t npages)

```

1. 删除虚拟内存/virtual address 中的对应的 mmobj
2. 并删除相应 page table entry，相当于释放物理内存

#### 4. vmmap_is_range_empty

```c
int vmmap_is_range_empty(vmmap_t *map, uint32_t startvfn, uint32_t npages)


```

判断指定的一段pages 在否在虚拟内存中未被占用


#### 5. vmmap_read

```c
int vmmap_read(vmmap_t *map, const void *vaddr, void *buf, size_t count)

```

从 cur\_proc 当前地址空间map中指定的vaddr开始，读取“内存”中大小为count的内容到指定的 “内存”buf 中（**因为这里的“内存”是虚拟内存，必须要借助pframe_lookup()读取physical memory的值**）


1. 循环执行以下步骤，直到读取字节数 达到count 数量；
    - 注意：因为count 是字节数，而 address space 中以 page 为单位；并且page之间并非一直连续，是由 vmarea 链接而成；因此需要遍历 地址空间逐个vmarea、逐个page 读取
2. 在 address space 中查找 start vaddr所属的 vmarea
3. 获得该 vmarea 中所包含的 pages
4. 循环读取所有 pages，memcpy到指定buf
    - 调用`pframe_lookup()` 找到 virtual page 的 physical page 
    - 注意1：如果之前并没有 mapped physical address；那么会调用`pframe_get()` 即刻map一个physical address
    - 注意2: 这里的 physical address 并非真正的，而依然是存在kernel address space 的 virtual address
    - 调用 `memcpy()`将对应的pframe中的内容copy到指定的buf中去
    - 注意3:因为`memcpy()`是C语言库函数，这里实际上还是“virtual address”之间的内容拷贝。很有可能在`memcpy()`实现了“virtual address to physical address”的转换
5. 直到该vmarea所有pages都copy完
    - if count 未满足，则跳到下一个vmarea继续拷贝
    - if count 满足，则退出
    
