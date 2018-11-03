---
title: Write A Promise
comments: true
date: 2018-11-02 13:25:27
type: categories
categories: Full stack
tags:
- Promise
- vanilla JS
- ES6
- callback hell
- callback
---

## I. Why do we need `Promise`  - "callback hell"

1. **<u>Asynchronous call</u>** is the core of JavaScript across browser and Node.js. 

2. <u>JavsScript runtime thread</u> use `callback` to execute result of asynchronous task handled by <u>other threads</u> (**Ajax thread**, **Timer thread**, **Rendering Thread**)

3. Example below are callback hell , which is a headache for programmer. For 

   ```js
   // when the nesting level exceeds 3, people will get crazy!!
   getData(function(x){				// getData of x
       getMoreData(x, function(y){   	// getMoreData of y, using x
           getMoreData(y, function(z){ // getMoreData of x, using y
               ...
           });
       });
   });
   ```

4. Is there a way to **<u>flatten</u>** the nesting callbacks, in a chainable way?  => `Node`
   - Consider the `Node` data structure, it contains child node in itself, similar to our requirement!!
   - `Promise` is such a **<u>stateful utility object</u>**, which can be chained to implement nested callbacks

## II. Promise Usage

1. **executer**: 

   - execute asynchronous action, generating result

   - when create new Promise, need to pass in pre-defined executer

     ```js
     // executer: a function with parameters: `resolve` and `reject`
     let p = new Promise( function(resolve, reject){
         ajaxGet( url, function(res){
             if(res.status == 200)  
                 resolve(res.data)
             else     			   
                 reject(res.error)
         })
     })
     ```

2. **callbacks** in `then`:

   -  `onFulfilled` & `onRejected` specify how to handle the result of asynchronous action

   - **use case 1:** <u>multiple callbacks on a promise</u>

     ```js
     // Assume executor will: resolve(1)
     let callback1 = function(res){ console.log("This is from callback1: " + res*1) }
     let callback2 = function(res){ console.log("This is from callback2: " + res*2)}
     p.then(callback1)  // This is from callback1: 1
     p.then(callback2)  // This is from callback1: 2
     ```

   - **use case 2:** <u>chaining callbacks</u>

     ```js
     // Assume executor will: resolve(1)
     let callback1 = function(res){ return res+1 }
     let callback2 = function(res){ return res+1 }
     let callback3 = function(res){ console.log("The final res will be 3: " + res) }
     p.then(callback1).then(callback1).then(callback3)  // The final res will be 3: 3
     ```


## III. Promise: stateful utility object

1. **"Stateful"**

   - Promise is an object, its properties are **<u>stateful</u>**, ie. property value can <u>change dynamially</u> 

   - state can only change in 2 ways: `pending => fulfilled`,  `pending => rejected`

   - the cause of stateful is: <u>asynchronous actions</u> inside the **<u>executor</u>**. Once instantiated to promise object, the executor will be executed inside promise object, and will update `state`,  `value`, `consumers`

     ```js
     function MyPromise( executor ){
         this.state = "pending"  // keep record of the state update
     	this.value = undefined  // store the result of asynchronous action
     	this.consumers = []     // place to store callbacks (actually child promises)
         
         executor(this.resolve.bind(this), this.reject.bind(this)) // run executor 
     }
     ```

2. `resolve` & `reject`: hooks to receive asynchronous result. 

   - The asynchronous function `executor` , will pass its result to Promise using its hooks: `resolve` and `reject`

3. `then`: hooks to receive passed in callbacks  `onFulfilled` & `onRejected`

   - the callbacks are passed to Promise using `then` hook
   - Promise will take care of executing callback, when asynchronous result is ready

4. How to it chainable

   - In fact, `then()` method can **<u>not only receive callbacks</u>**, 
   - **<u>but also can return a new Promise</u>**, which can be chained by using another `then`, ...., and so on
   - `then()` create a new Promise, then put the callbacks inside the newly created Promise, then put into the `consumers` array to consume the asynchronous result when its ready 



## VI. Write One!

1. **<u>Promise state:</u>**

   There are only 2 state transitions allowed: from pending to fulfilled, and from pending to rejected. No other transition should be possible, and once a transition has been performed, the promise value (or rejection reason) should not change.

   ```js
   function MyPromise(executor) {
       this.state = 'pending';
       this.value = undefined;
       executor(this.resolve.bind(this), this.reject.bind(this));
   }
   
   // provide only two ways to transition
   MyPromise.prototype.resolve = function (value) {
       if (this.state !== 'pending') return; // cannot transition anymore
       this.state = 'fulfilled'; 		      // can transition
       this.value = value; 				  // must have a value
   }
   
   MyPromise.prototype.reject = function (reason) {
       if (this.state !== 'pending') return; // cannot transition anymore
       this.state = 'rejected'; 			  // can transition
       this.value = reason; 				  // must have a reason
   }
   ```


2. <u>**`then` method: the core of chaining**</u>

   - **[Note 1]:** each `then()` will return a new Promise!

   - The `then` method is the core to Promise, which can be chained using `then`. It has two tasks:
     - receive callbacks, and store them for later use when asynchorous result is ready
     - create new promise, to make promise chainable

   - In fact, we put callbacks of the current promise, into the newly created child promises. Each callback will correspond a new Promise creation

   - This way, we can also easily consume the `outcome` of the `callback(result)`, to `resolve(outcome) `for the child promises

   - **[Note 2]:** 

     - child promise execute its parent's callback ; 
     - and resolve outcome of that callback, pass the outcome to promise created by next `then`

     ```js
     // For each then(), create a new Promise to embody this callback pair
     MyPromise.prototype.then = function(onFulfilled, onRejected) {
         var consumer = new MyPromise(function () {});
         
         // ignore onFulfilled/onRejected if not a function
         consumer.onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : null;
         consumer.onRejected  = typeof onRejected  === 'function' ? onRejected  : null;
         
         // put the consumer with parent's callbacks onto `this.consumers`
         this.consumers.push(consumer);
         
         // It's possible that the promise was already resolved, 
         // so, try to consume the resolved result 
         this.broadcast();
         
         // must return a promise to make it chainable
         // used by next potential `then()` in the chain
         return consumer;
     };
     
     ```

   - `broadcase()`: Consume the asynchorous result

     After the asynchronous action return a result, we need to

     - first, update the state to `fulfilled` or `rejected`

     - execute the each callback of  `onFulfilled`, `onRejected` (in child promise) collected by `then`

     - `onFulfilled`, `onRejected` must be called asynchronously

       for example, `new Promise(executor).then(callback)`, callback execution must happen after executor. Thus, we can correctly get result even if executor is synchorous

     ```js
     MyPromise.prototype.broadcast = function() {
         var promise = this;
         // only get called after promise is resolved
         if (this.state === 'pending') return;
         
         // callback must be executed by its corresponding promise
         var callbackName = this.state == 'fulfilled' ? 'onFulfilled' : 'onRejected';
         
         // when no callback specified in then(), still need to resolve/reject it
         var resolver = this.state == 'fulfilled' ? 'resolve' : 'reject';
         
         // onFulfilled/onRejected must be called asynchronously
         setTimeout(function() {
             // traverse in order & call only once, using `splice(0)` for deletion
             promise.consumers.splice(0).forEach(function(consumer) {
                 try {
                     var callback = consumer[callbackName];
                     // if a functon, call callback as plain function
                     // else, ignore callback
                     if (callback) {
                         // child promise resolve outcome of callback(value)
                         consumer.resolve(callback(promise.value)); 
                     } else {
                         // child promise resolve value without callback
                         consumer[resolver](promise.value);
                     }
                 } catch (e) {
                     consumer.reject(e);
                 };
             })
         }, 0 );
     }
     ```

   - **[Note 2]:** the current `then` can only handle chainable promise when callback is pure function; Cannot handle when callback return a new Promise.

3. <u>**[TODO]: make `then` can response another Promise**</u>



## V. Test Our Own Promise

```js
function MyPromise(executor) {
    this.state = 'pending';
    this.value = undefined;
    this.consumers = [];
    executor(this.resolve.bind(this), this.reject.bind(this));
}

// provide only two ways to transition
MyPromise.prototype.resolve = function (value) {
    if (this.state !== 'pending') return; 
    this.state = 'fulfilled';
    this.value = value;
    this.broadcast();
}    

MyPromise.prototype.reject = function (reason) {
    if (this.state !== 'pending') return;
    this.state = 'rejected'; 
    this.value = reason; 
    this.broadcast();
}    

// A promise’s then method accepts two arguments:
MyPromise.prototype.then = function(onFulfilled, onRejected) {
    var consumer = new MyPromise(function () {});
    consumer.onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : null;
    consumer.onRejected = typeof onRejected === 'function' ? onRejected : null;
    this.consumers.push(consumer);
    this.broadcast();
    return consumer;
};

MyPromise.prototype.broadcast = function() {
    var promise = this;
    if (this.state === 'pending') return;
    var callbackName = this.state == 'fulfilled' ? 'onFulfilled' : 'onRejected';
    var resolver = this.state == 'fulfilled' ? 'resolve' : 'reject';
    setTimeout(function() {
        promise.consumers.splice(0).forEach(function(consumer) {
            try {
                var callback = consumer[callbackName];
                if (callback) {
                    consumer.resolve(callback(promise.value)); 
                } else {
                    consumer[resolver](promise.value);
                }
            } catch (e) {
                consumer.reject(e);
            };
        })
    });
};


// ------------------ TEST ------------------ 
let p0 = new MyPromise((resolve, reject)=>{
    setTimeout(()=>{ resolve(0) },1000)
})

let p1 = p0.then((res)=>{ 
    console.log("p1 get result from p0, value should be 0: " + res)
    return res+1
})

let p2 = p0.then((res)=>{ 
    console.log("p2 get result from p0, value should be 0: " + res)
    return res+2
})

let p3 = p0.then((res)=>{ 
    console.log("p3 get result from p0, value should be 0: " + res)
    return res+3
})

console.log("This should be an array of 3 Promises: " + p0.consumers)

let p4 = p1.then(res=>{ console.log("p4 get result from p1, value should be 1: " + res)})
let p5 = p2.then(res=>{ console.log("p5 get result from p2, value should be 2: " + res)})
let p6 = p3.then(res=>{ console.log("p6 get result from p3, value should be 3: " + res)})
```



##  Credits to: 

1. https://stackoverflow.com/questions/23772801/basic-javascript-promise-implementation-attempt/42057900#42057900
2. https://www.promisejs.org/implementing/
3. https://javascript.info/promise-chaining
4. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise
5. http://thecodebarbarian.com/write-your-own-node-js-promise-library-from-scratch.html