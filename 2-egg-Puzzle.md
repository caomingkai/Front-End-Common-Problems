---
title: The Joy of Egg-Dropping(2-egg Puzzle)
comments: true
date: 2018-05-17 22:03:40
type: categories
categories: Algorithm
tags: 
- binary search
- DP
---

### I. Problem statement

**1. Easy version**
> Given infinite number of same eggs, try with minimun trials to find out  the highest floor where tossing the egg won't result in egg crash. The building has 200 floors.

- 变量：N=200
- 目标：试验次数最小
- 约束：none

**2. Hard version**
> Only 2 eggs were given, try with minimun trials to find the highest floor where tossing the egg won't result in egg crash. The building has 200 floors. (The egg can be reused as long as it is intact)

- 变量：F=200, E=2
- 目标：试验次数最小
- 约束：



### II. Solution for easy version

Since the number of eggs we can use is unlimited, we can use the **fastest approach in the universe** to find the unknown target in a list: `binary search`

```
- left direction: break 
- right direction:intact

0          100        200
         /     \
        50      150
      /   \    /   \
     25   75  125  175
    / \  
  ...   

Toss an egg on floor 100:
1. if egg break, toss another on floor 50;
    1.1 if egg break, toss another on floor 25;
        1.1.1 ...
        1.1.2 ...
    1.2 if doesn't break, toss another on floor 75;
        1.2.1 ...
        1.2.2 ...
2. if doesn't break, toss another on floor 150;
    2.1 if egg break, toss another on floor 125;
        2.1.1 ...
        2.1.2 ...
    2.2 if doesn't break, toss another on floor 175;
        2.2.1 ...
        2.2.2 ...

```


### III. Solution for hard version

Since it is not so intuitive at first glance, we will take a few trials to get a deeper understanding of this problem.

General idea:   

>egg-1 used to narrow down possible range  
>egg-2 used to sequentially find the target floor

1. **[egg-1 with step=10]** egg-1 tossed on floor 20,40,60,80,100,120,... with a step=20
    - worst-case: 29 trials
        + egg-1 tossed on 200 and break: 10 trials
        + egg-2 tossed on 199 (from 181), break: 19 trials
    - best-case: 2 trials
        + egg-1 tossed on 20 and break: 1 trials
        + egg-2 tossed on 1: 1 trials
    - for this method, what is the min trials can we say?
        + 29 trials(the worst case), no worse than it
        + we cannot guarantee it is the best case

2. **[edge case: BS]** egg-1 tossed on floor 100, 200
    - worst-case: 201 trials
        + egg-1 tossed on 200 and break: 2 trials
        + egg-2 tossed on 199 (from 101), break: 199 trials
    - best-case: 2 trials
        + egg-1 tossed on 100 and break: 1 trials
        + egg-2 tossed on 1: 1 trials
    - for this method, what is the min trials can we say?
        + 201 trials(the worst case), no worse than it

3. **[Solution 1]** 
    - https://www.zhihu.com/question/19690210/answer/22937826
    - the 'step' affect min-trial
    - we want control 'step' to make the trials of worst case and best case as close as possible; This way it will divvy up the trial between worst and best cases
    - Goal: worst case trials = best case trials
    - egg-1 tossed on floor x, 2x-1, 3x-2, .. 1
    - 蛋一第一次扔在X层，第二次扔在比原来高 X-1层，... 高2， 高1，使得X+(X-1)+...+2+1=200
    - 最坏情况(=最好情况)总的次数就是：
        + 如果在第X层就碎，那么蛋二最坏情况要扔X-1，所以总数是 1+X-1=X
        + 如果在X+(X-1)碎，那么蛋二最坏情况要扔 X-2，所以2+X-2=X
    - x = 20
    
    ```
        |_|__
        |_|     x-2
        |_|__   
        |_|     x-1
        |_|___ 
        |_|     x
        |_|
        |_|___ 
    ```
    - 假设蛋一第一次扔在X层，第二次扔在比原来高 X-1层，... 高2， 高1，使得X+(X-1)+...+2+1=100，那么最坏情况总的次数就是：如果在第X层就碎，那么蛋二最坏情况要扔X-1，所以总数是 1+X-1=X如果在X+(X-1)碎，那么蛋二最坏情况要扔 X-2，所以2+X-2=X


4. **[Solution 2]** 
    - 链接：https://www.zhihu.com/question/19690210/answer/125713075

    - 子问题F[i, j]: H的可行范围为[0, i]，手上还有j个鸡蛋的时候，最坏情况下的最小试验次数是多少。我们的目标函数就是F[N, K]。
    - 这个能够设立是基于这样一种想法，如果H的可能范围目前是[L, R]，那么其状态与[0, R-L]是完全一致的，所以我们仅仅记录有可行范围的宽度就可以了。
    - `F[i, j]=min{max{F[k-1, j-1], F[i-k, j]}}+1，`
        + 其中1<=k<=i。k表示这一次实验是从k楼把鸡蛋扔下去，扔下去有两种结果。
        + 一种是鸡蛋碎了，那么剩余的鸡蛋个数是j-1，H的可行范围变为[0, k-1]，那么需要的最少实验次数为F[k-1, j-1]。
        + 另一种是鸡蛋没碎，那么剩余的鸡蛋个数是j，H的可行范围是[k, i]，那么需要的最少实验次数是F[k-i, j]。
    - 由于题目所求的是最坏情况下的最优策略，所以对每种k情况内取max，再对所有k取min。
    - 边界条件为`F[0, j]=0，F[i, 1]=i`。
    - 时间复杂度为O(NK^2)。
