---
title: VM - Page Frame
comments: true
date: 2018-04-12 17:04:02
type: categories
categories: Courses
tags: OS
---

## Why do we need page frame?

实际 page frame 只是 kernel 的一个 struct，用于：

1. 记录：由其包含的 physical page 地址（其实还是virtual addr）
2. 记录：该page 所属的 memeory object(即 vm_area_struct/bss... 所指向的)；以及该page 的 page\_num
3. 第一条“由其包含”表明了，OS需要一个结构体来管理所“征用”的physical page，这个结构体就是 page frame
4. 注意：page frame 中的page 不一定都在 physical memory 中，也有可能被 page out 到了 disk。前者称为 resident page。



#### 1. pframe_init()

> 使用物理内存之前的准备工作，开机时？

初始化与 pframe 有关的量：  

- alloc\_list  
- pinned\_list  
- pframe_allocator  
- pageout deamon

#### 2. pframe_shutdown（）

> 停止使用物理内存之后的清理工作，关机时？

- 停止 pageout deamon
- 清理及 free 所有 page

#### 3. pframe_get_resident（）

> 获得“驻足内存的pframe”，based on mmobj & pagenum; 如果pframe 不再内存中，返回NULL


#### 4. pframe_alloc（）
> 为 mmobj、pagenum 指定的 page 分配**物理内存**


#### 5. pframe_lookup（）
> 实际上就是执行 pframe_get()，

#### 6. pframe_get（）
**#参见`page\_fault()`:**  
https://caomingkai.github.io/2018/04/18/VM-Page-Table-Page-Fault-Hander/
> 根据 ‘mmobj & pagenum’ 得到 pframe  
> - 如果物理内存中已有该page，直接返回  
> - 如果物理内存中没有该page，allocate 新的，并（有可能从disk）fill 进内容

1. 先判断由 ‘mmobj & pagenum’ 指明的pframe 是否存在于内存中，by calling pframe\_get\_resident() 
    - 如果存在，继续判断 pframe 是否 busy（因为有可能被 pageout deamon 操作）
        + if busy 则 sleep；醒来时再回到第1步，判断pframe是否存在于物理内存中
        + if not busy，直接返回 pframe
    - 如果不存在，则创建新的pframe
        + 先判断是否需要启动 pageout deamon 释放部分物理内存，需要的话就唤醒 deamon
        + 然后分配pframe空间， pframe_alloc(o,pagenum)；注意返回值ERROR
        + 再向刚获得的pframe中添加内容， pframe_fill()；注意返回值ERROR

#### 7. pframe_migrate（）
> 将pframe 移到父节点；缩短 shadow object tree 的高度（在某个mmobj被清除的时候）

#### 8. pframe_fill（）
> 调用 specific mmobj 的 fillpage() 真正实现**将外部数据读入物理内存**
 
 
#### 9. pframe_pin（）
> 对该 pframe 锁定 pin down，防止被 pageout

1. 判断该 pframe 的 pf_pincount 是否为“0”
    - 如果==0，表明该 pframe page 在 allo\_chain上
        + 从原来 chain 卸下来
        + nallocated--
        + 加入 pinned_chain 中
        + npinned++
        + pf->pf_pincount++
    - 如果 != 0，表明该 pframe page 已经在 pinned_chain上了
        + 只需要 pf->pf_pincount++ 即可


#### 10. pframe_unpin（）
> 将该 pframe 解锁，向 pageout deamon 开放

1. 首先更新 pf_pincount： pf_pincount--;
2. 判断该 pframe 的 pf_pincount 目前是否为“0”
    - 如果==0，那么
        + 从 pinned_chain 中卸下来
        + npinned--
        + 加入 alloc_chain 中
        + nallocated++
    - 如果 !=0, 那么啥也不做

#### 11. pframe_dirty（）
> 对 pframe（仅指file_obj，shadow、anony 无法pageout）进行更改之前，将该page设置为 DIRTY，待 pageout deamon 回收的时候检查到DIRTY时，会将page的内容更新到 disk 上

