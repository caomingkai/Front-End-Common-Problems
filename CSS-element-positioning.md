---
title: 'CSS: element positioning'
comments: true
date: 2018-09-20 11:46:43
type: categories
categories: Full stack
tags:
- CSS
- positioning
- scrollY
- scrollTop
- scrollLeft
- offsetTop
- offsetLeft
- getBoundingClientRect
---

```js
// 1. Wrong version: cannot do asynchronous function
function combineFetchers(arg){
    return function(){
        let params = [].slice.call(arguments, 0)
        arg.forEach(
            fn => fn(params[0], params[1])
        )
    }
}

let animalFetcher = function(msg, callback){
    callback( this)
    callback( arguments.callee.name + " " + msg)
}

let fruitFetcher = function(msg, callback){
    callback( arguments.callee.name + " " + msg)
}

let treeFetcher = function(msg, callback){
    callback( arguments.callee.name + " " + msg)
}

fetchAll = combineFetchers([animalFetcher, fruitFetcher, treeFetcher])
fetchAll("message", function(results) {console.log(results)} )

// 2.1 Correct version: callback
function combineFetchers(arr) {
  return function(input, callback) {
    const list = [];
    let count = 0;

    arr.forEach(fn => fn(input, res => {
      list.push(res);

      if (++count === arr.length) {
        callback(list);
      }
    }));
  };
}
 
function animalFetcher (userInput, callback) {
  setTimeout(() => callback(userInput + 1), 0);
}
 
function fruitFetcher (userInput, callback) {
  setTimeout(() => callback(userInput + 2), 0);
}
 
function mineralFetcher (userInput, callback) {
  setTimeout(() => callback(userInput + 3), 0);
}

const fetchAll = combineFetchers([animalFetcher, fruitFetcher, mineralFetcher]);
fetchAll(4, function (res) {console.log(res);});


// 2.2 Correct version: promise.all()
function combineFetchers(arr) {
  return function(input, callback) {
    Promise.all(arr.map(fn => new Promise(
      function(resolve){fn(input, resolve)})))
      .then(res => callback(res));
  };
}

function animalFetcher (userInput, callback) {
  setTimeout(() => callback(userInput + 1), 0);
}
 
function fruitFetcher (userInput, callback) {
  setTimeout(() => callback(userInput + 2), 0);
}
 
function mineralFetcher (userInput, callback) {
  setTimeout(() => callback(userInput + 3), 0);
}

const fetchAll = combineFetchers([animalFetcher, fruitFetcher, mineralFetcher]);
fetchAll(4, function (res) {console.log(res);});
```







retry n times

```js
function scb(v){ console.log("sucess: ", v) }
function ecb(v){ console.log("error : ", v) }

function callAPI(data, scb, ecb){
    setTimeout(()=>{ecb(data)}, 2000 )
}

//  1. elegant version
function retry(n, data, scb, ecb) {
  ecb1 =  n == 0 ? ecb  : function (){retry(n-1,data,scb,ecb) }
  
  callAPI(data,scb,ecb1)
}

// 2. verbose version
function retry(n, period, data, scb, ecb){
    let count = n
    let startTime = Date.now()
    let customErrorHanlder = function(){
        let timeElapsed = Date.now()-startTime
        if( n > 0 && timeElapsed < period ) {
            console.log(n + "times trial left! And " + (Date.now()-startTime) + "second elapsed." )
            callAPI(data, scb, customErrorHanlder)
            n -= 1
        }else{
            console.log(n + "times trial left! And " + (Date.now()-startTime) + "second elapsed." )
            callAPI(data, scb, ecb)
        }
    }
    callAPI(data, scb, customErrorHanlder)
}


retry(4, 5000, 3, scb, ecb)
```







考察JS事件循环，要求写一个mySetInterval(callback, interval)，要求callback能够每隔一段时间执行一次，或者如果单次执行事件太长的话则预约到下一次可执行的时间点执行。

```js
function mySetInterval(callback, interval){
    setInterval(function(){
        let startFlag = true
        setTimeout(function(){
            if(startFlag) 
        }, interval)
        callback()
        startFlag = false
        
    }, interval)
}
                
                
                
                
var counter = 0;

var interval = setInterval(function() { 
  setTimeout( function(){ console.log(counter)}, 2001 )
  counter++;
}, 2000);


```

​	



generator

```js
function *generator(){
    let a = 1
    yield a
   
    a += 1
    yield
    
    yield "the third output"
    return 4
}

let g = generator()
g.next()
g.next()
g.next()
g.next()
g.next()
```

