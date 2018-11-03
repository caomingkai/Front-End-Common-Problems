---
title: How to Flatten an Array
comments: true
date: 2018-10-14 19:54:39
type: categories
categories: Algorithm
tags:
- Algorithm
- Stack
- recursion
---

## Problem

Given an nested array, flatten it to  1-dimension array.

For example: 

```
[1, [2, [ [3, 4], 5], 6], [7, 8], 9]
=> 
[1, 2, 3, 4, 5, 6, 7, 8,  9]
```



## Solution 1:  iterative version 

```js
// 1. iterative [stack]
function flatten1( arr ){
    let res = []
    let stack = []
    stack.push(arr)
    while( stack.length > 0 ){
        let item = stack.pop()
        if( !Array.isArray(item) ){
            res.push(item)
        }else{
            for( let i = item.length-1; i >=0; i-- ){
                stack.push(item[i])
            } 
        }
    }
    return res
}


```



## Solution 2:  recursive version

```js
function flatten2(arr){
    if( !Array.isArray(arr) ){
        return [arr]
    }
    return [].concat(...arr.map(flatten2))
}
```



## Solution 3:  recursive version [shorter]

```js
function flatten(a) {
	return Array.isArray(a) ? [].concat(...a.map(flatten)) : a;
}


```



## Testing

```js
let test1 = [1, [2, 3, [4, 5]], 6, [7, [8, 9], 10], 11]
let test2 = []
let test3 = [1]
let test4 = 1
let test5 = [[[1,2,3]]]
let test6 = [[[]],[],[],[],[[[]]]]


// let arrayFlattened1 = flatten1(test1).toString()
// let arrayFlattened2 = flatten1(test2).toString()
// let arrayFlattened3 = flatten1(test3).toString()
// let arrayFlattened4 = flatten1(test4).toString()
// let arrayFlattened5 = flatten1(test5).toString()
// let arrayFlattened6 = flatten1(test6).toString()
let arrayFlattened1 = flatten2(test1).toString()
let arrayFlattened2 = flatten2(test2).toString()
let arrayFlattened3 = flatten2(test3).toString()
let arrayFlattened4 = flatten2(test4).toString()
let arrayFlattened5 = flatten2(test5).toString()
let arrayFlattened6 = flatten2(test6).toString()
console.log(arrayFlattened1)
console.log(arrayFlattened2)
console.log(arrayFlattened3)
console.log(arrayFlattened4)
console.log(arrayFlattened5)
console.log(arrayFlattened6)


```

