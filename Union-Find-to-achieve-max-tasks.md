---
title: Union Find to achieve max tasks
comments: true
date: 2018-04-08 12:47:03
type: categories
categories: Algorithm
tags: Union Find
---


### 1. 题意：
- 给n个任务 1～k；
- 以及 task dependency array A、B；其中 [Ai,Bi] - bi先完成，才能做ai；
- 同时规定：每个task顶多只能依赖一个task
- 问：最多可以完成的任务数

### 2. 分析
虽然任务的dependency可能成环(1-2-3-4-1),但有第三条可知：
每个任务最多出现在1个“环”里，表明该“环”上的任务有一个不可能完成，但可以完成(环任务数-1)的任务
对于没有出现在环中的任务，这些任务都能够执行完

因此，相当于根据 A、B，找 number of dependency cycle，
那么最多可以完成的任务数就是：`总任务数 - 环数`




### 3. Union Find Class

**Union Find Summary**: https://workflowy.com/s/HdnT.nwWuWS1F7W

**如何找有多少环？:`Union-Find.isConnected()`**

- 对每个 task dependency 进行 union
- union 之前调用 isConnected() 检查是否已连通(表明该[ai,bi]会导致“环”)
- 累加 每次isConnected() 结果为 True 的情况
- isConnected() 有几次为 True，就存在几个“环”


```java

// weighted quick UF

class UnionFind{
	int [] p;
	int [] w;
	int numOfCycles ;
	
	public UnionFind(int N){
		p = new int[N];
		w = new int[N];
		numOfCycles = 0;
		for( int i = 0; i < N; i++ )
			p[i] = i;
	}

	void union(int a, int b ){
		int pa = find(a);
		int pb = find(b);
		if( pa == pb ){
			numOfCycles++;
		}else{
			if( w[pa] < w[pb] ){
				p[pb] = pa;
				w[pa] += w[pb];
			}else{
				p[pa] = pb;
				w[pb] += w[pa];
			}
		}
	}


	int find(int a){
		while( p[a] != a ){
			p[a] = p[p[a]];
			a = p[a];
		}
		return a;
	}
}
```


### 4. Solution 

```java
class TaskMaster{
	public static int maxTasks(int n, int[] a, int[] b ){
		int ret = 0;
		int size = a.length;
		UnionFind UF = new UnionFind(n);
		
		for( int i = 0; i < size; i++ ){
			UF.union(a[i]-1,b[i]-1);
		}
		int numOfCycles = UF.numOfCycles;
		ret = n - numOfCycles;
		return ret;
	}

	public static void main( String[] args ){
		// int n = 8;
		// int[] a = {2,3,4,1,5,8,7};
		// int[] b = {1,2,3,4,6,7,8};

		// int n = 4;
		// int[] a = {2,3,4,1};
		// int[] b = {1,2,3,4};

		// int n = 2;
		// int[] a = {};
		// int[] b = {};

		int n = 2;
		int[] a = {2};
		int[] b = {1};

		System.out.println(maxTasks(n, a, b ));
	}
}
```



