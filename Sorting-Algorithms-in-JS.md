---
title: Sorting Algorithms in JS
comments: true
date: 2018-11-01 15:25:34
type: categories
categories: Algorithm
tags:
- insertion sort
- selection sort
- bubble sort
- merge sort
- quick sort
---



## I. Selection Sort

- totally unsorted: O(n^2)

- partially sorted: O(n^2)

- ```js
  /*
    select min element from rest of unsorted elements, put into front
      - totally unsorted: O(n^2)
      - partially sorted: O(n^2)
  */
  
  function slectionSort( list ){
    if( list === null || list.length <= 1 ) return 
  
    for( let i = 0; i < list.length; i++ ){
      let curMinId = i
      for( let j = i+1; j < list.length; j++ ){
        list[curMinId] > list[j] && (curMinId = j)
      }
      [ list[i], list[curMinId] ] = [ list[curMinId], list[i] ]
    }
  }
  
  // Test
  let list = [ 2, 1, 3, 4, 5, 6, 3, 5, 1 ]
  console.log("Original list: " + list)
  slectionSort(list)
  console.log("Selection sort: " + list)
  ```


## II. Insertion Sort

- totally unsorted: O(n^2)

- partially sorted: O(n)

  ```js
  /*
    for each element, insert it into the sorted part exactly before its position
      - totally unsorted: O(n^2)
      - partially sorted: O(n)
  */
  
  function insertionSort(list){
    if( list === null || list.length <= 1 ) return 
    
    for( let i = 1; i < list.length; i++ ){
      let idToBe = i
      while( idToBe-1 >= 0 && list[idToBe] < list[idToBe-1] ){
        [ list[idToBe-1], list[idToBe] ] = [ list[idToBe], list[idToBe-1] ] 
        idToBe -= 1
      }
    }
  }
  
  // Test
  let list = [ 2, 1, 3, 4, 5, 6, 3, 5, 1 ]
  console.log("Original list: " + list)
  insertionSort(list)
  console.log("Insertion sort: " + list)
  ```




## III. Bubble Sort

- **BAD version:**

  - totally unsorted: O(n^2)

  - partially sorted: O(n^2)

  - ```js
    /*
      for each element, compare with its right element, swap if it's larger;
      keep doing util (n-1)th, (n-2)th,, (n-3)th ... 1st position
        - totally unsorted: O(n^2)
        - partially sorted: O(n^2)
    */
    
    function bubbleSort(list){
      if( list === null || list.length <= 1 ) return 
      for( let i = list.length-1; i > 0; i-- ){
        for( let k = 0; k < i; k++ ){
          if( list[k] > list[k+1] ){
             [ list[k], list[k+1] ] = [ list[k+1], list[k] ]
          }
        }
      }
    }
    
    
    // Test
    let list = [ 2, 1, 3, 4, 5, 6, 3, 5, 1 ]
    console.log("Original list: " + list)
    bubbleSort(list)
    console.log("Bubble sort: " + list)
    ```

- **Good Version**

  - totally unsorted: O(n^2)

  - partially sorted: O(n)  **<u>if no swap for a iteration, then every element is in its place, terminate</u>**



    ```js
    /*
      for each element, compare with its right element, swap if it's larger;
      keep doing util (n-1)th, (n-2)th,, (n-3)th ... 1st position
        - totally unsorted: O(n^2)
        - partially sorted: O(n)
        - if no swap for a iteration, then every element is in its place, terminate
    */
    
    function bubbleSort(list){
      if( list === null || list.length <= 1 ) return 
      let swapped = true
      for( let i = list.length-1; i > 0; i-- ){
      	if(swapped === false) return 
      	swapped = false
        for( let k = 0; k < i; k++ ){
          if( list[k] > list[k+1] ){
             [ list[k], list[k+1] ] = [ list[k+1], list[k] ]
             swapped = true
          }
        }
      }
    }
    
    // Test
    let list = [ 2, 1, 3, 4, 5, 6, 3, 5, 1 ]
    console.log("Original list: " + list)
    bubbleSort(list)
    console.log("Bubble sort: " + list)
    ```


## VI. Merge Sort

- Time Complexity

  - totally unsorted: O(n^2)
  - partially sorted: O(n)

- Space Complexity:  **<u>cannot sort in-place</u>**

  - O(n^2)

- ```js
  /*
    recursively divide list into two half, till each half have one or two elements
    recursively merge two half into one, till achieve original size
    
    Time Complexity
    	- totally unsorted: O(n^2)
      - partially sorted: O(n^2)
    Space Complexity:
    	- O(n^2)
  */
  
  function mergeSort(list){
    if( list === null || list.length <= 1 ) return 
    return divideAndMerge(list)
  }
  
  function divideAndMerge(list){
    let l1 = list.slice(0, list.length/2)  
    if( l1.length > 1 )  l1 = divideAndMerge(l1)
    
    let l2 = list.slice(list.length/2, list.length)
    if( l2.length > 1 )  l2 = divideAndMerge(l2)
    
    return merge(l1, l2)
  }
  
  function merge(l1, l2){
    let list = []
    let i = 0, j = 0, k = 0
    while(i < l1.length && j < l2.length){
      if( l1[i] > l2[j] ) 
        list[k++] = l2[j++]
      else
        list[k++] = l1[i++]
    }
    while( i < l1.length ) list[k++] = l1[i++]
    while( j < l2.length ) list[k++] = l2[j++]
    
    return list
  }
  
  // Test
  let list = [ 2, 1, 3, 4, 5, 6, 3, 5, 1 ]
  console.log("Original list: " + list)
  mergeSort(list)
  console.log("Merge sort: " + mergeSort(list))
  ```




## V. Quick Sort

- **<u>Time Complexity:</u>** 

  - totally unsorted: O(n^2)
  - partially sorted: O(n)

- **<u>Space Complexity:</u>** O(n) **<u>in-place</u>**

- **<u>Approach:</u>**

  - randomly choose a pivot, here we use the first element in the question list
  - loop from both ends, compare value with pivot, and swap their value
  - how to swap: **<u>front & back two pointers</u>**
    -  "LeetCode: sort color": https://leetcode.com/problems/sort-colors/description/

- ```js
  /*
    1. randomly choose a pivot, here we use the first element in the question list
    2. loop from both ends, compare value with pivot, and swap their value
    3. how to swap: "LeetCode: sort color" ==> "front & back two ponters"
       https://leetcode.com/problems/sort-colors/description/
       
       
         0......0   1......1  x1  x2 .... xm   2.....2
                    ^         ^            ^
                   zero       i           two
                smallerToBe			 largerToBe
                
          区间一：0  | 区间二：1 | 区间三：未知 |  区间四：2
  */
  
  function quickSort(list, left, right){
      if( list === null || left >= right ) return 
  
      let pivot = list[left],
      	smallerToBe = left, 
          largerToBe = right, 
          i = left
      	
      while( i <= largerToBe ){
          if( list[i] === pivot ){
              i++
          }else if( list[i] < pivot ){
              swap(list, i++, smallerToBe++)
          }else{
              swap(list, i, largerToBe--)
          }
      }
      quickSort(list, left, smallerToBe-1)  
      quickSort(list, largerToBe+1, right)
  }
  
  function swap(list, p1, p2){ [ list[p1], list[p2] ] = [ list[p2], list[p1] ] }
  
  // Test
  let list = [ 3, 1, 3, 4, 5, 1, 3, 2, 5, 1 ]
  console.log("Original list: " + list)
  quickSort(list, 0, list.length-1)
  console.log("Quick sort: " + list)
  ```
