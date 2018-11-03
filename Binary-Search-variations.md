---
title: Binary Search variations
date: 2018-03-27 09:12:32
type: "categories"
comments: true
categories: Algorithm
tags: binary search
---


***Credit to : http://izualzhy.cn/algorithm/2014/04/18/binary-search-analysis***

## I. 二分法及变种的注意点：

#### 1. 如何得到循环不变式

【定义】就是在每次循环里，我们都可以保证要找的index在我们新构造 的区间里。  
【例子】如正常二分法，循环不变式表述如下：  

		
    if array[mid] > key:
	    right = mid - 1
	 else if array[mid] < key:
		 left = mid + 1
	 else
		 return mid
	  
	 
考虑中间值与key的关系： 

- 如果中间值比key大，那么[mid, right]的值我们都可以忽略掉了，这些值都比key要大。只要在[left, mid-1]里查找就是了。
- 如果中间值比key小，那么[left, mid]的值可以 忽略掉，这些值都比key要小，只要在[mid+1, right]里查找就可以了。
- 如果相等，表示找到了，可以直接返回。如果最后这个区间没有，那么就确实是没有 。所以说循环不变式，就是在每次循环里，我们都可以保证要找的index在我们新构造的区间里


#### 2. 数组是非递增还是非递减  


#### 3. 结束条件，即while (condition) 应当是<还是<=  
    
    
    “< ” 表明可选区间长度 2～n ；  OR    
    “<= ” 表明可选区间长度 1～n 
    


#### 4. 求mid应当是向上取整还是向下取整

    
     mid = (left + right) >>1       OR
     mid = (left + right + 1) >>1
    
#### 5.  while 结束后是否需要判断一次条件

    
    对应可选区间为2～n情况，将所有元素check完毕；  
    同时，调整mid取整方式，使该判断优雅
    

---

## II. 常见问题：

#### 1. 查找值key的下标，如果不存在返回-1.

```java
int BS(const Vec Int& vec, int key)
{
    int left = 0, right = vec.size() - 1;
    while (left <= right)
    {
        int mid = (left + right) >> 1;
        if (vec[mid] > key)
            right = mid - 1;
        else if (vec[mid] < key)
            left = mid + 1;
        else
            return mid;
    }

    return -1;
}
```

- 如果中间值比key大，那么[mid, right]的值我们都可以忽略掉了，这些值都比key要大。只要在[left, mid-1]里查找就是了。
- 如果中间值比key小，那么[left, mid]的值可以 忽略掉，这些值都比key要小，只要在[mid+1, right]里查找就可以了。
- 如果相等，表示找到了， 可以直接返回。

因此，循环不变式就是在每次循环里，我们都可以保证要找的index在我们新构造 的区间里。如果最后这个区间没有，那么就确实是没有。
**注意mid的求法**，可能会int越界，但我们先不用考虑这个问题，要记住的是这点：<u>mid是偏向left的，即如果left=1,right=2,则mid=1。</u>

#### 2. 查找值key第一次出现的下标x，如果不存在返回-1.

```java
int BS_First(const VecInt& vec, int key)
{
    int left = 0, right = vec.size() - 1;
    while (left < right)                 //问题1，left < right时继续，相等就break.
    {
        int mid = (left + right) >> 1;
        if (vec[mid] < key)
            left = mid + 1;
        else
            right = mid;
    }

    if (vec[left] == key)               //问题2,再判断一次。
        return left;

    return -1;
}
```

我们仍然考虑中间值与key的关系：

- 如果array[mid]<key，那么x一定在[mid+1, right]区间里。
- 如果array[mid]>key，那么x一定在[left, mid-1]区间里。
- 如果array[mid]≤key，那么不能推断任何关系。比如对key=1,数组{0,1,1,2,3},{0,0,0,1,2},array[mid] = array[2] ≤ 1，但一个在左半区间，一个在右半区间。
- 如果array[mid]≥key，那么x一定在[left, mid]区间里。
- 综合上面的结果，我们可以采用1,4即<和≥的组合判断来实现我们的循环不变式，即循环过程中一直满足key在我们的区间里。

**注意问题：**

- 循环能否退出,我们注意到4的区间改变里是令right = mid,如果left=right=mid时，循环是无法退出的。换句话说，第一个问题我们始终在减小着区间，而在这个问题里，某种情况下区间是不会减小的！ ----- 见代码
- 循环退出后的判断问题，再看下我们的条件1,4组合，只是使得我们最后的区间满足了≥key，是否=key，还需要再判断一次。 ----- 见代码

#### 3. 查找值key最后一次出现的下标x，如果不存在返回-1.

```java
int BS_Last(const VecInt& vec, int key)
{
    int left = 0, right = vec.size() - 1;
    while (left < right)
    {
        int mid = (left + right + 1) >> 1;
        if (vec[mid] > key)
            right = mid - 1;
        else
            left = mid;
    }
    
    if (vec[left] == key)
        return left;

    return -1;
}
```

循环不变式（省去分析的过程，同上）：

- 如果array[mid]>key，那么x一定在[left, mid-1]区间里。
- 如果array[mid]≤key, 那么x一定在[mid, right]区间里。
	
**注意问题：**
	
- 在条件2里，实际上我们是令left=mid，但是如前面提到的，如果left=1,right=2,那么mid=left=1，同时又进入到条件2,left=mid=1，即使我们在while设定了left < right仍然无法退出循环，
- 解决的办法很简单：mid = (left + right + 1) >> 1， 向上取整就可以了。

#### 4. 查找刚好小于key的元素下标x，如果不存在返回-1.

```java
int BS_Last_Less(const VecInt& vec, int key)
{
    int left = 0, right = vec.size() - 1;
    while (left < right)
    {
        int mid = (left + right + 1) >> 1;
        if (vec[mid] < key)
            left = mid;
        else
            right = mid - 1;
    }

    if (vec[left] < key)
        return left;

    return -1;
}
```

循环不变式（省去分析的过程，同上）：

- 如果array[mid]<key，那么x在区间[mid, right]
- 如果array[mid]≥key，那么x在区间[left, mid-1]

#### 5. 查找刚好大于key的元素下标x，如果不存在返回-1,等价于std::upper_bound.

```java
int BS_First_Greater(const VecInt& vec, int key)
{
    int left = 0, right = vec.size() - 1;
    while (left < right)
    {
        int mid = (left + right) >> 1;
        if (vec[mid] > key)
            right = mid;
        else
            left = mid + 1;
    }

    if (vec[left] > key)
        return left;

    return -1;
}
```

循环不变式（省去分析的过程，同上）：

- 如果array[mid]>key，那么x在区间[left, mid]
- 如果array[mid]≤key，那么x在区间[mid + 1, right]

#### 6. 查找第一个>=key的下标，如果不存在返回-1,等价于std::lower_bound.

```java

int BS_First_Greater_Or_Equal(const VecInt& vec, int key)
{
    int left = 0, right = vec.size() - 1;
    while (left < right)
    {
        int mid = (left + right) >> 1;
        if (vec[mid] < key)
            left = mid +1;
        else
            right = mid;
    }

    if (vec[left] >= key)
        return left;

    return -1;
}
```

循环不变式（省去分析的过程，同上）：

- 如果array[mid]<key，那么x在区间[mid + 1, right]
- 如果array[mid]≥key，那么x在区间[left, mid]

---
