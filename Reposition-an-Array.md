---
title: Reposition an Array
comments: true
date: 2018-10-14 19:59:57
type: categories
categories: Algorithm
tags:
- Algorithm
- in-place
---



## Problem 

Given an original array, and it new position to be. Reposition it according the given indices array. For example:

```js
let arr  =    ["a", "b", "c", "d", "e", "f"]
let indices = [ 1,   2,   3,   5,   0,   4 ]

// the new array should be ["e", "a", "b", "c", "f", "d"]
```

## Follow Up

Try to do it **<u>in-place</u>**, i.e. : Space Complexity `O(1)`



## Solution 1: Space complexity `O(n)`

```js
function rePosition(arr, indices){
    if( arr.length !== indices.length ) 
        throw new Error("they are not the same length")
    
    let newArr = indices.map((item, index)=>{
        return arr[indices.indexOf(index)]
    })
    return newArr
}

// Tests
let newArr = []
let arr  =     ["a", "b", "c", "d", "e", "f"]

// test 1
let indices1 = [ 1,   2,   3,   5,   0,   4 ]
newArr = rePosition(arr, indices1)
console.log(newArr)  // ["e", "a", "b", "c", "f", "d"]

// test 2
let indices2 = [ 0,   1,   2,   3,   4,   5 ]
newArr = rePosition(arr, indices2)
console.log(newArr) //  ["a", "b", "c", "d", "e", "f"]

// test 3
let indices3 = [ 2,   1,   0,   4,   3,   5 ]
newArr = rePosition(arr, indices3)
console.log(newArr) //  ["c", "b", "a", "e", "d", "f"]
```



## Solution 2: Space complexity `O(1)`

We need two variables: `kickedItem` and `kickedIndex`, to denote the kicked item in `arr` and kicked index in `indices`

1. Every time we update an item in its new position,  we put item in its new position, and swap the kicked item and index to their counterpart in `arr` & `indices`

2. Since the kicked item & its new position are just swapped to previous position, we can keep staring the same position to deal with the chaining update, until there is an end to the chain

3. NOTE: there are also cases (*<u>test case 3</u>*) that the new indices are not  depended upon one another, but instead they are partially dependent. That's why we need to maintain two variable for both `arr` & `indices`, rather than just for `arr`


```js

function rePosition(arr, indices){
	if( arr.length !== indices.length ) 
        throw new Error("they are not the same length")
    
    for( let i = 0; i < arr.length; i++ ){
        while( i !== indices[i] ){
            let kickedItem = arr[indices[i]]
            let kickedIndex = indices[indices[i]]
            arr[indices[i]] = arr[i]
            indices[indices[i]] = indices[i]
            arr[i] = kickedItem
            indices[i] = kickedIndex
        }
    }
}

// Tests

// test 1
let arr  =     ["a", "b", "c", "d", "e", "f"]
let indices1 = [ 1,   2,   3,   5,   0,   4 ]
rePosition(arr, indices1)
console.log(arr)  // ["e", "a", "b", "c", "f", "d"]

// test 2
arr  =     ["a", "b", "c", "d", "e", "f"]
let indices2 = [ 0,   1,   2,   3,   4,   5 ]
rePosition(arr, indices2)
console.log(arr) //  ["a", "b", "c", "d", "e", "f"]

// test 3
arr  =     ["a", "b", "c", "d", "e", "f"]
let indices3 = [ 2,   1,   0,   4,   3,   5 ]
rePosition(arr, indices3)
console.log(arr) //  ["c", "b", "a", "e", "d", "f"]
```

