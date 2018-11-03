---
title: 239.Sliding Window Maximum
comments: true
date: 2018-04-07 16:30:20
type: categories
categories: LeetCode
tags: sliding window
---

## 方法一：Priority Queue

0. O(nlogn)
1. 前后两个窗口差别只有两个数，尽量用到上一个窗口的信息
2. 上一个窗口的头部元素从窗口元素集剔出，加入下一个窗口的尾部元素；形成新的窗口元素集
3. 想办法从“窗口元素集”中找最大值==> priority queue 最合适


```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        
        if( nums == null || nums.length == 0 )
            return new int[0];
        
        int len = nums.length;
        int[] ret = new int[len-k+1];
        PriorityQueue<Integer> pq = new PriorityQueue<>(k, Collections.reverseOrder());
        
        for( int i = 0; i < len; i++ ){
            if( i >= k ){
                ret[i-k] = pq.peek();
                pq.remove(nums[i-k]);
            }
            pq.add(nums[i]);
        }
        ret[len-k] = pq.peek();
        
        return ret;
    }
}
```



## 方法二：单调队列 / Monotonic Queue / Deque

0. O(n)  - 每个元素顶多 add to／remove from queue，共2次
1. 上一个方法在从窗口元素集找最大 O(logn),继续优化只能为 O(1)
2. 想办法维护一个与窗口区对应的“有序数据结构”
3. 进入下一个窗口前，上一个窗口对应的 potential\_max queue 是有序的
4. 进入下一个窗口后，
    - 如果第一个 potential\_max 仍然在新窗口范围内； 
      该peotential\_max不需要从 queue中删除，仍然有可能是后边窗口的max
    - 如果第一个 potential\_max 不在新窗口范围内； 
      该value就不是potential max了，而是 definitely\_not\_possible\_max for later windows
5. 窗口末尾新加入的元素：
    - 如果值比 potential\_max\_queue 中的最后一个 potential\_max\_value 小，
      那么该值与之前的值都有可能是后边窗口的max
    - 如果值比后边的“某一段”potential\_max values 都大，
      表明“那一段”potential values 因为更大新元素的加入，由 potential 变为 never\_possible\_max
    - 直接删除“那一段” potential values，并将该值插入到 queue之后
      
6. 【小结】：potential\_max\_queue 的维护有4点：
    - 维护这么一个“单调队列”目的：某个window的max，就是其对应 petential\_max\_queue 中的第一个元素
    - 头部 potential\_max\_value 在当前window下是否valid，是否需要删除；
    - 头部 potential\_max\_value 是否valid，实际是根据其 index来判断的，因此在queue中维护的实际是 index
    - 新加入的元素是否应该加入potential max queue，加入到哪里


```java
class Solution {
    public int[] maxSlidingWindow(int[] nums, int k) {
        
        if( nums == null || nums.length == 0 )
            return new int[0];
        
        int len = nums.length;
        int[] ret = new int[len-k+1];
        Deque<Integer> dq = new LinkedList<>();
        
        for( int i = 0; i < len; i++ ){
            // check head items of the potential_max_queue
            if( !dq.isEmpty() && dq.peekFirst() < i-k+1 )
                dq.removeFirst();
            
            // handle new item
            while( !dq.isEmpty() && nums[dq.peekLast()] < nums[i] ){
                dq.removeLast();
            }
            dq.addLast(i);
            
            // update ret values
            if(i-k+1 >= 0)
                ret[i-k+1] = nums[dq.peekFirst()];
        }
        return ret;
    }
}
```
