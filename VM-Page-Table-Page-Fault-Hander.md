---
title: 'VM: Page Table & Page Fault Hander'
comments: true
date: 2018-04-18 15:22:02
type: categories
categories: Courses
tags: OS
---

## I. Page Table

#### 1. Page table的作用

**【简洁版】**

- 隔离user process，给每个process分配4GB内存【假象，至于到底怎么分配，交给kernel】
- 提供 CPU 可识别的 physical address

**【详细版】**

- 在没有 page table 之前，processes 如果想在memory中执行，必须做好process isolation。一个方案是为每个process 分配一段“大小合适的”内存，这样的缺点是：
    + 首先，分配的内存必须连续；否则还需要 page table
    + 其次，多大才算“大小合适”，因为process在加载之前无法预知它会使用多少内存
- 另一种方案是：将内存划分成很小的“页”，按需使用；这样解决了“大小合适”的问题，但仍然存在内存必须连续的问题（否则会引起内存侵扰）
- 最后的方案是：page table。建立虚拟内存，对于32bit机器而言：
    + 给每个process的内存都是4GB[0x00000000~0xffffffff]
    + 同时kernel在每个processz的PCB中都维护一个 page table  
      提供给 CPU 可操作的 physical address
    + 一般情况下不知直接采用一级 page table，即将所有 4GB 的vaddr映射到 4GB的physical addr，占空间太大了！！**【page table在memory中， kernel 1GB的address space中】**


#### 2. 代码分析 - Weenix
Weenix 中 page table 实际上由 pagedir struct (pagetable.c)负责维护，结构如下：

```c
struct pagedir {
        pde_t      pd_physical[PT_ENTRY_COUNT];
        uintptr_t *pd_virtual[PT_ENTRY_COUNT];
};


int pt_map(pagedir_t *pd, uintptr_t vaddr, uintptr_t   
           paddr, uint32_t pdflags, uint32_t ptflags)

将指定的 virtual addr 与 physical addr
```

1. 每一个 vaddr 都由唯一对应两个表的index： pagedir table index、page table index
    + pagedir index = (vaddr>>12) / PAGE\_ENTRY\_CNT
    + pagetable index = (vaddr>>12) % PAGE\_ENTRY\_CNT
    + 注意：pagedir table中存的是 [index1: pagetable physical addre]的entry
    + 注意：page table中存的才是[index2:physical addr]的entry
    + 这样就建立起 vaddr 的 Two-Level translation system；  
      先通过 pagedir table 找到 page table，在查到对应 physical addr
2. pagedir： 
    + 由 PCB 维护一个指针指向该 page directory table
    + 从pagedir开始，逐级查找到该 process的address space对应的physical address
3. pd_physical[i]： 
    + 由 (vaddr>>12) / PAGE\_ENTRY\_CNT 计算出vaddr所对应的 pagedir table中的entry
    + 该 entry 中维护一个指针指向下级 page table的 physical address；即vaddr所在的page table
4. *pd_virtual[i]：
    + 由(vaddr>>12) % PAGE\_ENTRY\_CNT计算出vaddr对应的 page table 中的entry
    + 该 entry 维护中vaddr 对应的 physical addr


<br>
## II. Page fault

#### 1. page fault handler 具体步骤

page fault 是由MMU检测到的硬件中断，由kernel处理；当一个进程发生缺页中断的时候，进程会**陷入(trap)内核态**，执行以下操作： 

1. 检查要访问的虚拟地址是否合法 
2. 查找/分配一个物理页 
3. 填充物理页内容（读取磁盘，或者直接置0，或者啥也不干） 
4. 建立映射关系（虚拟地址到物理地址） 
5. 重新执行发生缺页中断的那条指令 

#### 2. 代码分析 - page fault handler 

```c
void handle_pagefault(uintptr_t vaddr, uint32_t cause)

输入：vaddr【导致发生缺页中断的 vaddr】 
     cause【page fault的原因】
        + write on RDONLY
        + write on Reserved, etc.
输出：void
```

1. 检查该 vaddr 真的是cur\_proc 地址空间中的使用的 addr 么?
    + `vmarea = vmmap_lookup(curproc->p_vmmap, ADDR_TO_PN(vaddr))`
    + if == NULL，表明地址空间中就没有 任何一个 region用到该地址，返回Error
2. 至此，获得了该 vaddr 所属的 vmarea；
    + 获得vmarea的目的，是为了获得该vaddr实际对应的mmobj
    + 只有通过`vmmap_lookup()`**找出“幕后主使”**，才能够进行后续往物理内存中加载内容
    + 比如只有找出该vaddr对应的vmarea，继而找到vnode(即对应的file)，才能知道该往内存中读入什么！！
    + user process 程序操作的全是vaddr，不知道该vaddr实际对应的是什么；但是查询到该vaddr page 所属的 mmobj就知道该往物理内存中读入什么了
3. 注意特殊情况：forwrite，这会引起有意思的 Copy\_On\_Write
4. 重要的一步：往physical page加载mmobj，并更新page table with (vaddr：paddr)
    - 通过调用 `pframe_lookup(vmarea, pageNum, fw, *pframe)`,找该vmarea下的pframe
    - 实际上vmarea不直接管理pframe，而是vmarea对应的mmobj管理；继而内部调用 vmarea—>mmobj 的 `lookuppage()`
    - mmobj 调用`pframe_get()`寻找是否有相应的 pframe 对应于 mmonj&pageNum
    - 如果有：直接返回；
    - 如果没有：进入*“无中生有”*这一步：
        - 创建 pframe 结构体，获得physical address。 通过：`pframe_alloc(mmobj,pagenum)`  
            1. **这一步会调用 `page_alloc()` 申请到物理内存physical address**  
            2. 至此vaddr有了与之对应的物理内存，但还未载入vaddr对应的内容
        - 调用 `pframe_fill()`往刚申请到的physical addr中载入vaddr对应的内容  
            1. **这一步调用 `mmobj->ops->fillpage(obj,pf)`由vaddr对应的mmobj向物理内存载入内容**  
            2. 实际上如果是disk，会调用`read_block()`进入硬件过程；后续还会通过DMA(direct memory access)直接将disk中的内容读入到内存
        - 至此，mmobj的**幕后人物：vnode / file** 的内容成功读入到 page frame 中
5. 最后调用 `pt_map(curproc->pdir, vaddr, pt_virt_to_phys(pf_addr),...)`将 (vadd:paddr)写到 page table 中

```
【实质】

 virtualAddr <=> pframe <=> physicalAddr
      |            |             |
     user        kernel     physical mem
      |            |             |
      3G    1G=2^20*2^12=4K <==> 4G
       \_ _ _ _ _ _ _ _ _ _ _ _ _／
       
+ 任何一块物理内存都由一个对应的pframe指向,并管理；
+ pframe本身为virtual address；甚至其内部与phsical page
  直接关联的 pf_addr也是 virtual address。But why?**  
+ 实际上:
    - 1page = 4KB = 2^12 Byte
    - physical mem(32bit): 4GB=2^32 Byte
    - pframe #: 2^32 / 2^12 = 2^20 
    - kernel address space: 1GB = 2^20
    - 也就是说：使用pframe分配物理内存的话，
              kernel pframes正好与physical pages形成对应
    - 因此：在pframe创建之时(即对应物理内存征用之时)，
      只需要做一个简单的加减法就可以！
      因为 kernel 1G [c0000000~ffffffff],对应4GB；
      那么减去的 delta=c0000000（该值需要再调整）
    
```
