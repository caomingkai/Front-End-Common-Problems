---
title: 'Coding Problem: Find 1st Bad Version'
comments: true
date: 2018-09-10 22:19:23
type: categories
categories: Algorithm
tags:
- Algorithm
- binary search

---





## I. problem

given 500 revisions of programs,  write a program that will find and return the FIRST bad revision given a `isBad(revision i)` function.



## II. Code

```js
function isBad( version ){ return version === 0 }

function firstBadVersion( versions ){
    if( typeof versions === 'undefined' || versions.length === 0 ) 
        return -1;
    
    let left = 0, right = versions.length-1;
    while( left < right ){
        let mid = (left + right) / 2
        if( isBad(mid) ){
            right = mid
        }else{
            left = mid+1
        }
        return versions[left]===0 ? left : -1
    }
 
}

// Testing		0 1 2 3 4 5 6  should return index: 4
let versions = [1,1,1,1,0,0,0]
console.log(firstBadVersion(versions))  // 4

versions = [1,1,1,1,1,0]
console.log(firstBadVersion(versions))  // 6

versions = [0, 0, 0]
console.log(firstBadVersion(versions))  // 0

versions = [0]
console.log(firstBadVersion(versions))  // 0

versions = [1]
console.log(firstBadVersion(versions))  // -1

versions = [1, 1, 1]
console.log(firstBadVersion(versions))  // -1

versions = []
console.log(firstBadVersion(versions))  // -1

console.log(firstBadVersion())  		// -1
```

