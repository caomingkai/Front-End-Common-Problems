---
title: setTimeout vs setInterval
comments: true
date: 2018-10-31 20:23:26
type: categories
categories: Full stack
tags:
- Vanilla JS
- setTimeout
- setInterval
- Event Loop
---

## 0. Credits to: 

http://www.laruence.com/2009/09/23/1089.html

https://www.jeffjade.com/2015/08/03/2015-08-03-javascript-this/

https://www.youtube.com/watch?v=8aGhZQkoFbQ

https://www.zhihu.com/question/55050034

https://codepen.io/discountry/pen/XRXdVX?editors=1011

## I. `setTimeout` 

1. After `delay` period, put `cb` to <u>**taskQueue**</u> of main thread, rather than executing `cb` right away!
   - only guarantees after exact`delay`, `cb` is scheduled,
   - cannot guarantee `cb` being executed after exact `delay`, because maybe at that time, the main thread is still dealing a time-consuming task, which will postpone the execution of `cb`
2. Good use case: <u>**recursively schedule delayed task**</u>, like a `node` data structure
   - deal with CPU intensive task, set aside period for UI update to be executed to avoid freezing
   - eg.  break down large task into smaller ones, schedule smaller task execution every delay period 
     - the bigger `delay` period is, the longer time left for UI
     - the time left for UI is exactly `delay` period
     - NOTE: don't use `setInterval`, since the execution time for `cb`  can be longer than`delay` , resulting in there's no time left for UI update. When `cb` finishes execution,  the UI update and next `cb` will **<u>race</u>** to be executed in next **event loop**

3. **`setTimeout(cb, 0):` ** 

   - 1. **等到Call Stack为空，即下一轮 Event Loop的时候 执行 `cb`**
   - 2. <u>**Render Queue**</u> 比 <u>**Callback Queue**</u> 在browser中的**优先级高**，他们<u>**共用一个JS V8 Engine**</u>。因此，callback中的`cb`会等到DOM被render之后在调用，这就是为什么使用 **`setTimeout(cb, 0)` ** 
   - 3. 这样，即便有很多`setTimeout(cb, 0)`**短时间大量**被调用，也会因为**<u>render queue的高优先级</u>**，使得它能够在两个`cb` 调用之间，插入到 JS V8 thread中update DOM，从而使**<u>页面不会freeze</u>**

   - <u>**Waiting for elements mounted to DOM by GUI thread, then switch to JS thread, do the manipulation by `cb`**</u>

   - Every function put inside `setTimeout` with `0` delay, will **<u>respect</u> all the DOM task**, and will **scheduled after all DOM tasck**

   - will execute right after all **<u>synchronous task</u>** is finished on stack by main thread

   - i.e. has to wait for next **<u>event loop</u>** come

   - **Great Use Case:**

     - set the order of event handler: 

       - ```js
         // 1. parent event_handler will be executed first, 
         // 2. then the target element 
         let input = document.getElementById('myButton');
         
         input.onclick = function A() {
           setTimeout(function B() {
             input.value +=' input';
           }, 0)
         };
         
         document.body.onclick = function C() {
           input.value += ' body'
         };
         ```

     - tackle the **<u>race problem of keypress</u>**

       - ```js
         /*
         	keypress has two results:
         	1. give input the value of pressed key [asynchronous - hardware]
         	2. call the keypress event handler [asynchronous - event]
         	
         	BAD version: 
         		- cannot get the current key value when callback is called
         		- can only get the value of previous pressed key value
         	GOOD version:
         		- use setTimeout(fn, 0)
         		- callback will be executed after the key value is updated in DOM
         */
         
         <input type="text" id="input-box">
         
         // 1. BAD Version
         document.getElementById('input-box').onkeypress = function (event) {
           	this.value = this.value.toUpperCase();
         }
         
         // 2. GOOD Version
         document.getElementById('input-box').onkeypress = function() {
           let self = this;
           setTimeout(function() {
             self.value = self.value.toUpperCase();
           }, 0);
         }
         ```


## II. `setInterval`

1.  Every `delay` period, put `cb` to <u>**taskQueue**</u> of main thread if there is no `cb` waiting to be executed on the taskQueue; Otherwise, won't schedule `cb` even `delay` period is achieved
2. **<u>Note:</u>** 
   1. when delay is up, even there exists an CPU-intensive task , `setInterval` won't wait for the finish of the current task, instead put the `cb` directly to **<u>taskQueue at the exact time spot</u>**; 
   2. But if the main thread is still executing task, the `cb` has to waits to be executed 
   3. There is NO accumulated `cb` scheduled on event queue,  even if there is a CPU intensive task blocking the periodic `cb` scheduling.
   4. `setInterval` timer has a check mechnism to ensure, there will be only one `cb` on event queue

3. Common use case: **CSS animation**

   - Note1: 
     - it only focus scheduling `cb` after `delay`, don't care how long vacant time left
     - it's possible that execution of `cb` is longer than `delay`, then next `cb` will be executed right after first `cb` finishes

   - Note2: 

     - mustn't use **time** variable as terminating condition to clear interval, 

     - because some time it cost longer to execute `cb`, making some scheduled task won't be executed

     - use other variable as terminating condition to clear interval

     - ```js
       let div = document.getElementById('someDiv');
       let opacity = 1;
       let fader = setInterval(function() {
       	opacity -= 0.1;
       		if (opacity >= 0) {
       			div.style.opacity = opacity;
       		} else {
       			clearInterval(fader);   // use `opacity` as terminating condition
       		}
       }, 100);
       ```


## III. Common Problems

1. **`this` problem => because of `window.setTimeout` or `window.setInterval`**

   - `setTimeout( function(){ this.value=50 }, 1000 )`

   - <u>Problem:</u> 

     - example above, exist `this` in the callback function, if not `bind` actual object, `this` will refer `window`
     - `setTimeout` `setInterval` will change the reference of `this` to `window`

   - <u>Why?</u>

     -  `this` always refers to the object who calls the function where `this` resides
     - when `cb` executed, it is executed like `cb()`, which omits the prefix object: `window`

   - <u>How to solve?</u>

     - use `bind()` to produce a new `cb` function carrying this reference to right object

     - use `apply` or `call`

     - ```js
       a = "string belongs to window"    // note: not " let a = ... ";
       obj = {
           a: "string belongs to obj",
           printA: function(){ console.log(this.a) }
       }
       
       // 1. "string belongs to window", after 1s
       setTimeout(obj.printA, 1000)  
       
       // 2. "string belongs to window", after 1s
       let temp = obj.printA
       setTimeout(temp, 1000)  	  
       
       // 3. "string belongs to obj"
       setTimeout(function(){ obj.printA.apply(obj) }, 2000) 
       
       // 4. "string belongs to obj" 
       setTimeout(function(){ obj.printA() }, 3000)
       ```

2. Implement a `setInterval` using `setTimeout`

   1. ```js
      function customSetInterval( cb, delay ){
          let time_handle = null
          function wrapper(){
              cb.apply(null)
              time_handle= setTimeout(wrapper, delay)
          }
          time_handle= setTimeout(wrapper, delay)
          return time_handle
      }
      
      
      // Test
      let i = 0
      function cb(){ i+=1; console.log(i) }
      
      let interval_handle = customSetInterval(cb, 1000)  // print  1 , 2 , 3 , 4 , ...
      
      // clearTimeout(interval_handle)   // use to clear interval
      ```

3. <u>how to handle encoding task for a file (CPU intensive)?</u>

   - **Problem**: it might freeze the UI when encode the whole file

   - **Solution**: 

     - Divide the file into several part, using a symbol, or using size

     - for each small part, just take over CPU, since it's small task

     - between each encoding, leave time for UI interaction

     - Note: use `setTimeout` instead of `setInterval`

     - Since the execution time for `cb`  can be longer than`delay` , resulting in there's no time left for UI update. When `cb` finishes execution,  the UI update and next `cb` will **<u>race</u>** to be executed in next **event loop**

     - ```js
       /*
       	input: the file to be encoded
       	output: the encoded goal file 
       	totalSize: the size of the input file
       	encode: the function used to encode
       */
       
       function encodeWrapper( input, output, totalSize, encode ){
           let output = ""
           let accumSize = 0
           let i = 0
           let timeoutHandle = null
           /* delayed recursion
              i: ith step    
              stepSize: how big to encod for a step
           */
           function encodeHelper(i, stepSize ){
               output += encode( input.substring(i*stepSize,stepSize) )
               if( i*stepSize > totalSize ) {
                   clearInterval(timeoutHandle)
                   return output
               }
               timeoutHandle = setTimeout(encodeHelper(i+1, stepSize), 100)
           }
           return encodeHelper(0, 1024) 
       }
       ```

4. <u>How to write a **delayed recursion** ?</u>

   - Problem: Calculate the sum from 0 to n

   - Solution:  f(n) = f(n-1) + n;    f(0) = 0


   - ```js
     /*
     	f(n) = f(n-1) + n
     	f(0) = 0
     */
     
     function wrapper(n){
         let timeHandle = 0
         let accum = 0
     
         // recursive call itself every 1000ms
         function sum(n){
             timeHandle=setTimeout(function(){
                 if (n === 0) {
                     clearTimeout(timeHandle)
                     alert(accum) // output the result
                     return 
                 }
                 accum += n
                 sum(n-1)
             },1000)
         }
         sum(n)
     }
     ```


4. **Debouncify a function**

   - **<u>Problem</u>**: Some Ajax functions can be frequently called using events, which will cause severe performance problem on backend

   - **<u>Solution</u>**:  

     - we want to call Ajax function only after a period <u>**delay**</u> time (so that we can make sure the user is really waiting to for to make Ajax call, rather than in the middle of typing)
     - when there is another intention to call the Ajax function in the middle of last delay, we want to reschedule the ajax call after another delay

   - ```js
     function callAjaxRightAway(){ 
         console.log("Ajax call starting, with arguments: " + [...arguments] ) 
     }
     
     
     function Debouncify( fn, delay ){
         let time_handle = null
         return function(){
             clearTimeout(time_handle)
             let args = arguments
             time_handle = setTimeout( function(){ fn.apply(null,args) }, delay)
         }
     }
     
     let delayedFun = Debouncify(callAjaxRightAway, 2000) 
     
     delayedFun({arr:[1, 2, 3], heigh: 10 }) // will be ignored
     delayedFun(1,2)							// will be ignored
     delayedFun("a", "b", "c")    // "Ajax call starting, with arguments: a,b,c"
     
     ```