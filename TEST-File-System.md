---
title: 'FS - TEST: File System'
comments: true
date: 2018-04-02 22:10:48
type: categories
categories: Courses
tags: OK
---

## Preface

1. dir_namev() 没有处理如下情况：
    - extra slashes： paths_equal("1/2/3", "1///2///3///");

  
## vfstest\_main

1. int makedirs(const char \*dir)
    - 对传入的 绝对路径：dir，逐级建立相应的directory
    
2. int getdent(const char \*dir, dirent\_t \*dirent)
    - what： 读取 dir 下的第一个非 ./.. 的entry
        + 有的话，存到 dirent，返回 1
        + 没有的话， 返回 0
    - 先调用open(dir) 得到 fd
    - 然后调用getdents(fd, dirent)依次读取该dir 下的 entries
    - 将该dir下：除了"."以及".."的第一个entry读出来，读到dirent
        + 如果有其它 entry，那么关闭fd，返回 1
        + 如果只有 "." ".."， 那么关闭fd，返回 0

3. int removeall(const char \*dir)
    - what: 
        + 将 dir 下的所有 files 以及 sub-files 全部unlink
        + 删除 本 dir
        + 将cur_proc 置为 ".."

4. void vfstest\_start(void)
    - what
        + 随机生成 root_dir 的名字，并mkdir 该dir
        + 注意这只是 vfs_test 的root dir
    
5. void vfstest\_term(void)
    - what：Terminates the testing environment
    - 调用 removeall (root\_dir) 将vfstest 过程中所有文件／目录（包括 root\_dir 自己)删除

6. 宏函数：test\_assert(expr, fmt, args...)
    - what: 
        + expr应该是 true；
        + otherwise，打印format信息，以及后续的参数
    
7. 宏函数：syscall\_success(expr)
    - what：
        + expr所对应的测试的代码 should fail；

8. 宏函数：syscall\_fail(expr, err)
    - what：
        + expr所对应的测试的代码 should fail；即 -1 == (expr)
        + 并且，返回的错误代码应该跟此处传入的err一致

9. 宏函数：create\_file(file)
    - what:
        + 用于测试“成功打开open”、“成功关闭close” file;
        + 【注意】创建 file，不是 dir
        + 调用 open(file, O_RDONLY|O_CREAT)
        + 生成相应 fd

10. 宏函数：read\_fd(fd, size, goal)  
    - what： 
        + 用于测试 read() 是否能够从文件fd：
        + 1-读取指定 size 的内容到buf
        + 2-读取的内容无误： buf与goal 一致
        
11. 宏函数：test\_fpos(fd, exp)    
    - what：
        + 用于测试 fd 对应 f\_pos 目前应该出在 exp 指示的位置

12. vfstest\_stat()
    - what : 测试 do_stat()
    - 涉及函数：
        + mkdir
        + chdir
        + open
        + write
        + close

13. vfstest\_mkdir()
    - what : 测试 mkdir()
    - 涉及函数：
        + mkdir
        + chdir
        + rmdir
        + unlink

14. vfstest\_chdir()
    - what : 测试 mkdir()
    - 涉及函数：
        + mkdir
        + chdir
        + stat

15. vfstest\_paths()
    - what : 
        + 测试 directory path 的各种 “edge cases”
        + 主要函数paths\_equal(p1,p2) 

    - paths_equal(p1,p2) 
        + 传入的 char* str，满足 p1 == p2
        + 首先调用 makedirs(p1)，逐级创建 二者对应的 dir
        + 调用 stat(p1) 、stat(p2) 比对二者的信息
    - 涉及函数：
        + mkdir
        + chdir
        + stat
        + open

 
16. vfstest\_fd()
    - what : 测试 fd：file descriptor
    - how:
        +  read/write/close/getdents/dup nonexistent file descriptors
        +  dup2 has some extra cases since it takes a second fd
        +  if the fds are equal, but the first is invalid or out of the  allowed range
        +  dup works properly in normal usage
        +  dup2 works properly in normal usage
        +  dup2-ing a file to itself works
        +  dup2 closes previous file
        +  如果dup2传入的fd是已在使用中的,
           会首先关闭该fd对应的open_conn，
           然后将该fd赋给新的 open_conn
    - 涉及函数：
        + mkdir
        + open
        + read
        + write
        + lseek
        + getdents
        + dup
        + dup2
        + close
        + chdir

 
17. vfstest\_memdev()
    - what : 
        + 测试 /dev/null 以及 /dev/zero 的读写
    - /dev/null，或称空设备，被称为比特桶或者黑洞。
      是一个特殊的设备文件，它丢弃一切写入其中的数据
      但报告写入操作成功；读取它则会立即得到一个EOF
    - /dev/zero，当你读它的时候，它会提供无限的空字符(NULL)
      可以将数据写入 /dev/zero 文件，但实际上不会有任何影响；   
      不过一般我们还是使用 /dev/null 来做这件事
    - 涉及函数：
        + mkdir
        + chdir
        + rmdir
        + unlink

 
18. vfstest\_write()
    - what : 
        + 测试 在不同位置 f_pos 的 write() 操作
        + 测试 lseek()
    - test_assert(lseek(fd, 0, SEEK_END) == (NUM_CHUNKS-1) * CHUNK_SIZE + (int)strlen(str)
        + 返回的 f_pos在文件尾，但并不是 (NUM_CHUNKS) * CHUNK_SIZE
        + 因为每个chunk中，内容只占strlen(str)部分，应该为：
        + (NUM_CHUNKS-1) * CHUNK_SIZE + (int)strlen(str)
    - 涉及函数：
        + mkdir
        + chdir
        + lseek
        + write
        + close


19. vfstest\_infinite()
    - what : 
        + 申请一个 很大的 buf空间
        + 持续向 dev/null 中写，一直写到爆；
          即写入字节数的返回值<= 0 
        + 持续从 dev/zeor 中读NULL，一直读到爆；
          即读入字节数的返回值<= 0 
    - 涉及函数：
        + mkdir
        + open
        + read
        + write
        + close


20. vfstest\_open()
    - what : 
        + 测试 open()
        + 测试 close()
        + 测试 unlink()
    - how:
         - Accepts only valid combinations of flags
         - Cannot open nonexistent file without O_CREAT
         - Cannot write to readonly file
         - Cannot read from writeonly file
         - Cannot close non-existent file descriptor
         - Lowest file descriptor is always selected
         - Cannot unlink a directory
         - Cannot unlink a non-existent file
         - Cannot open a directory for writing
         - File descriptors are correctly released when a proc exits
    - 涉及函数：
        + mkdir
        + chdir
        + open
        + write
        + unlink
        + stat
        + close


21. vfstest\_read()
    - what : 测试 do\_read()
    - how:
        + Can read and write to a file
        + cannot read from a directory
        + Can seek to beginning, middle, and end of file
        + O_APPEND works properly
    - 涉及函数：
        + mkdir
        + chdir
        + open
        + read
        + write
        + lseek
        + close


22. vfstest_getdents()
    - what : 
        + 测试 do_getdent() 读取dir目录的entries的情况
    - 涉及函数：
        + mkdir
        + chdir
        + open
        + close
        + do_getdent

    - 问题：vfstest\_getdents()这个函数：
      在dir01中建立了一个目录、一个文件，
      为啥getdents(fd, dirents, 4 * sizeof(dirent\_t))
      返回值是4 * sizeof(dirent\_t)呀？
    - 答案：
        + 每个目录下还有 '.' 以及 '..' ，所以是4个！
    - 会调用 user process 的 getdents()
    - 继而调用 ksys\_getdents(fd, *dirp, count)：  
      + 将fd代表的directory目录下所有的entries读取出来，  
      + 结果按顺序放到dir中，直到读入的信息达到指定的count
      + 返回实际读到的字节数(就是所有entries构成的dir)
   

## faber\_fs\_thread\_test

1. 检查 argc， 要求在执行faber\_fs\_thread\_test()的时候，必须带参数
    - if ( argc == 1 ) { //  打印错误 }
    
2. 如果满足，即 argc>1；再检查输入的参数:n
    - if sscanf() != 1, 表明argv中的参数没有输入正确，
        + lim = 1
    - if sscanf() == 1，表明argv中的参数正确输入了
        + 即告知该程序：我想要开辟几个进程／线程
        + 然后只需要检测一下，lim不超过上下限即可
    - sscanf()
        + dtm ="Saturday March 25 1989" );
        + int n = sscanf( dtm, "%s %s %d  %d", weekday, month, &day, &year )
        + 将dtm的data，按照format的形式，赋值给后边的变量
        + 返回值：一共赋值了几个变量。此处 n = 4
   - snprintf()
        + BUFLEN = 256
        + char tname[BUFLEN]; 定义了一个字符数组，最大程度256
        + snprintf(tname, BUFLEN, "thread%03d", i);
        + 会将"thread%03d"与i 构成的字符串写道 tname 中
        + snprintf 中 BUFLEN 指定了tname接受的最大字符数

3.  开辟 lim 个进程／线程 make\_dir
    - name：" thread k " 
    - 每个进程／线程 执行：make\_dir\_thread()
4. do\_waitpid()等待所有的线程结束
    - if rv<0 : 打印 pid + rv + err\_info
    - else    : 打印 pid + rv
    - 至此所有的 make\_dir\_thread 进程／线程 终结了
5. 再开辟 lim 个进程／线程 clean\_dir
    - name: " clean\_dir\_thread k "
    - 每个进程／线程 执行 clean\_dir\_thread()
        + 如果上次 all passed，kthread\_create最后参数为：NULL
        + 如果上次有未  passed，kthread\_create最后参数为：1
    - do\_waitpid()等待当前线程结束，打印信息
6. 最后删除所有的 lim 个dir
    - 根据前几次的 pass 值
    - 如果全部删除成功， 打印成功
    - 如果有未删除的，打印失败


#### make\_dir\_thread

1. 该线程创建一个目录： /dir00k
2. 然后在每个目录中创建 20 个file： 
    - /dir00n/test000 ～ /dir00n/test019
    - 调用 do\_open(O\_CREAT)的方式创建新文件
        + 如果有错误，直接返回 err\_val
    - 调用 do\_write()向每个文件中写 “look！”
    - 调用 do\_close()关闭该文件


#### clean\_dir\_thread

1. 该线程会删除 arg1 指定的目录(/dir00k)下的所有files
2. arg2的值由 make\_dir\_thread的执行结果确定
    - Arg2 == 0 means directory should exist.
    - Arg2 == 1 means if directory does not exist,
      don't treat it as error. 
        + 尝试性的删除 make\_dir\_thread 创建的 file
3. 调用 f=do\_open()打开 该 dir
    - 如果该dir未打开，并且 Arg2 == 1，该线程退出 with rv=0
    - 如果 Arg2 == 0，该线程退出 with rv=f
4. 调用 do\_getdent() 挨个handle 每个 entry
    - 跳过 "." , ".." entry
    - 拼接其要删除的file 的绝对路径
    - 调用 do\_unlink(file\_path) 删除该file 的连接
        + 如果删除成功， 置got\_one = 1; 跳出
        + 关闭该 fd 的open\_conn



## faber\_directory\_test

0. 与faber\_fs\_thread_test区别在于：
    - 第一个
      所有mk\_dir\_threads成功之后，回收threads；
      再创建rm\_dir\_threads，成功后，回收threads
    - 第二个
      mk\_dir\_thread 与 rm\_dir\_thread 混合执行
      然后同时等待这两类thread的终结
1. 该函数创造“成对的进程／线程”
    - proc_1 name: " thread k "
    - proc_2 name: " rmthread k "
2. 每次 iteration
    - 创建一个 thread， 并运行之
    - 创建一个 rmthread，并运行之
3. 调用 do\_waitpid() 等待所有子进程结束
    - 如果某个进程返回rv < 0 , 打印：pid  + rv + err\_info
    - 如果某个进程返回rv ==0 , 打印：pid  + rv 

4. 循环删除所有 directory 
    - do\_rmdir(tname)


#### rm\_dir\_thread

1. 该线程创建一个目录： /dir00k
2. 然后删除目录中名字为：“/dir00n/test000” 的file
    - /dir00n/test000 ～ /dir00n/test019
    - 调用 do\_open(O\_CREAT)的方式创建新文件
        + 如果有错误，直接返回 err\_val
    - 调用 do\_write()向每个文件中写 “look！”
    - 调用 do\_close()关闭该文件








