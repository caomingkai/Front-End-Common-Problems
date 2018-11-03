---
title: Stack & Queue Implementation using JS
comments: true
date: 2018-10-08 22:08:58
type: categories
categories: Algorithm
tags:
- Stack
- Queue
- ES6 Class
- Data Structure
---



## Stack

1.  interfaces:

   - `stack.size`
   - `stack.peek()`
   - `stack.push`
   - `stack.pop()`
   - `stack.isEmpty()`
   - `stack.clear()`

2. implementation: 

   1. ```js
      /* 
       * Note: this version use `null` as Error symbol,
       *		 so you cannot push null into the stack
       */
      class Stack{
          constructor(){ this.data = [] }
          
          get size(){ return this.data.length }
          
          isEmpty(){  return this.data.length === 0 }
          
          peek(){ return this.data[this.data.length-1] }
          
          push(item){
              if( typeof item ==="undefined" || item === null ) return false
              this.data.push(item)
              return true
          }
          
          pop(item){
              if(this.data.length === 0) return null
              return this.data.pop();
          }
          
          clear(){ this.data = [] }
      }
      
      /* ---- Testing  ---- */
      let s = new Stack()
      console.log("size: ",s.size)  		// 0
      console.log(s.pop())  		// null
      console.log(s.push())  		// false
      console.log(s.push(null))  	// false
      console.log(s.push(1))  	// true
      console.log(s.push(1))  	// true
      console.log("size: ",s.size)  		// 2
      console.log(s.push(2))  	// true
      console.log("size: ",s.size)  		// 3
      console.log(s.pop())  		// 2
      console.log("size: ", s.size)  		// 2
      console.log(s.pop())  		// 1
      console.log(s.pop())  		// 1
      console.log("size: ", s.size)  		// 0
      console.log(s.pop())  		// null
      console.log("size: ", s.size)  		// 0
      console.log(s.isEmpty())   // true
      ```


## Queue

1.  interfaces:

   - `queue.size`
   - `queue.peek()`
   - `queue.push`
   - `queue.pop()`
   - `queue.isEmpty()`
   - `queue.clear()`

2. implementation:

   1. ```js
      /* 
       * Note: this version can add any object into Queue
       *		 But need to call `isEmpty()` when necessary
       */
      class Queue{
          constructor(){ this.items = []  }
          
          get size(){ return this.items.length; }
          
          isEmpty(){ return this.items.length === 0 }
          
          offer(item){ this.items.push(item) }
          
          poll(){ return this.items.shift() }
          
          peek(){ return this.items[0] }
          
          clear(){ this.items = [] }
      }
      
      /* ---- Testing  ---- */
      let q = new Queue()
      console.log("size: ", q.size)// 0
      console.log(q.offer(null))  
      console.log(q.offer(null))  
      console.log(q.items)	  			// [null, null]
      console.log(q.offer(1))  
      console.log(q.items)	  			// [null, null, 1]
      console.log(q.poll())  		// null
      console.log(q.offer(2))  	// [null, 1, 2]
      console.log("size: ",q.size)// 3
      console.log(q.poll())  	
      console.log(q.items)	  			// [1, 2]
      console.log(q.poll())  		// 1
      console.log(q.items)	  			// [2]
      console.log("size: ", q.size)// 1
      console.log(q.poll())  		 // 2
      console.log("size: ", q.size)// 0
      console.log(q.isEmpty())     // true
      ```
