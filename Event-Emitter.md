---
title: Event Emitter
comments: true
date: 2018-10-01 20:21:44
type: categories
categories: Full stack
tags: ES6
---




#### I. What is an Event Emitter?

An `event emitter` is a **utility object**, which can be used to emit event, which can be subscribed by as many functions as we can. 

1. It is an event manager, manages differents event and its subscribers.

2. Also, we can see it as the `Redux`, which also dispatches(emits) actions, and handled by its reducers.
3. It widely used at Node.js ( asynchronous event-driven architecture)
   - certain kind of objects call emitters periodically to emit events that cause listener objects to be called.
   - when a request comes requesting some resource, Node.js will register the subscriber: `responseToThisRequest()` on event: `thisTypeDataDone`. When data is obtained, will call `emitter.emit( thisTypeDataDone )`, to call the `responseToThisRequest()` corresponding all the requests.

#### II. Why and when do we need it?

> Whenever it makes sense for code to SUBSCRIBE to something rather than get a callback from something
>
> https://stackoverflow.com/questions/38881170/when-should-i-use-eventemitter

If when a certain event happens, we have to deal many type of manipulation for that event's result, then it is better to use the event emitter pattern. 

Instead put all functions inside an event callback, we subscribe all the function to that event, when the event happens, we emit the event!

#### III. How to use it?

**[Good Link]:** https://netbasal.com/javascript-the-magic-behind-event-emitter-cce3abcbcef9

**[Note]:  when pass in the callback function, we use arrow function, which already bind 'this' to the function, so that when the function be called, it knows what value it will update**

```js

let input = document.querySelector('input[type="text"]');
let button = document.querySelector('button');
let h1 = document.querySelector('h1');

button.addEventListener('click', () => {
  emitter.emit('event:name-changed', {name: input.value});
});

let emitter = new EventEmitter();
emitter.subscribe('event:name-changed', data => {
  h1.innerHTML = `Your name is: ${data.name}`;
});
```

1.  create an emitter util object

2. subscribe the desired callback function, using `emitter.subscribe(event, function.bind(this))`

3. call `emitter.emit()` when the event happen


#### IV. Write one!

1. **<u>object:</u>** we need an object (utility object)
2. <u>**data structure:**</u> inside it, we need a data structure to store the `event` and its `subscribers`(callbacks)
3. **<u>methods:</u>**  all the methods
   - `subscribe(event, callback)`: we need the return value to be an `unsubscribe()`, so we won't result in the memory leak!
   - `emit()`

```js
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, listener) {
      if (typeof this.events[event] !== 'object') {
          this.events[event] = [];
      }
      this.events[event].push(listener);
      return () => this.removeListener(event, listener);
  }
    
  removeListener(event, listener) {
    if (typeof this.events[event] === 'object') {
        const idx = this.events[event].indexOf(listener);
        if (idx > -1) {
          this.events[event].splice(idx, 1);
        }
    }
  }
  
  emit(event, ...args) {
    if (typeof this.events[event] === 'object') {
      this.events[event].forEach(
        listener => {if(typeof listener === 'function') listener.apply(null, args)}
      );
    }
  }
  
  // register a wrapper function containing the listener on this event,
  // when get executed, 
  //   - first remove it on this event;
  //   - then execute real listener
  once(event, listener) {
    const remove = this.on(event, (...args) => {
        remove();
        listener.apply(null, args);
    });
  }
};


let emitter = new EventEmitter()
// Test 1: invalid callback function
console.log("Test 1: Nothing should be print out")
emitter.on("a", 1);
emitter.emit("a");
console.log()

// Test 2: add mutilple callback function to an event
console.log("Test 2: callback 1, 2, 3")
emitter.on("b", ()=>{console.log("event b callback 1")});
emitter.on("b", ()=>{console.log("event b callback 2")});
let off1 = emitter.on("b", ()=>{console.log("event b callback 3")});
emitter.emit("b");
console.log()


// Test 3: unsubscribe one callback from an event queue
console.log("Test 3: callback 1, 2")
off1();
emitter.emit("b"); 
console.log()


// Test 4: subscribe other event & callbacks at the same time
let data = {digit: 1}
console.log("Test 4:  digit should be 11 & 14 ")
emitter.on("c", function(data){ data.digit+=1 });
emitter.on("c", function(data){ data.digit+=2 });
emitter.on("d", function(data){ data.digit+=3 });;
let off2 = emitter.on("d", function(data){ data.digit+=4 });

emitter.emit("c", data); 
emitter.emit("d", data);  
console.log(data.digit)
      
off2()
emitter.emit("d", data);  
console.log(data.digit)
console.log()
      


// Test 5: subscribe other event & callbacks
let data2 = {digit: 5}
console.log("Test 5: digit should be 11")
emitter.emit("d", data2 ); 
emitter.emit("c", data2 ); 
console.log(data2.digit)   
console.log()

// Test 6: test once
console.log("Test 6: should only print once")
emitter.once("test 6", ()=>{console.log("should only print once")} ); 
emitter.emit("test 6" ); 
emitter.emit("test 6" ); 
emitter.emit("test 6" ); 
console.log()
```

***FaceBook interview version***
https://gist.github.com/kutyel/7d8f204a347840a6ee7220743957e504

```js
/*
 * Create an event emitter that goes like this
 * emitter = new Emitter();
 *
 * Allows you to subscribe to some event
 * sub1 = emitter.subscribe('function_name', callback1);
 * (you can have multiple callbacks to the same event)
 * sub2 = emitter.subscribe('function_name', callback2);
 *
 * You can emit the event you want with this api
 * (you can receive 'n' number of arguments)
 * sub1.emit('function_name', foo, bar);
 *
 * And allows you to release the subscription like this
 * (but you should be able to still emit from sub2)
 * sub1.release();
 */
 
// facebook.js

class Emitter {
  constructor(events = {}) {
    this.events = events
  }

  subscribe(name, cb) {
    (this.events[name] || (this.events[name] = [])).push(cb)

    return {
      release: () => this.events[name] &&
        this.events[name].splice(this.events[name].indexOf(cb) >>> 0, 1)
    }
  }

  emit(name, ...args) {
    return (this.events[name] || []).map(fn => fn(...args))
  }
}

export default Emitter





// test.js

const expect = require('expect')

const Emitter = require('./facebook')

// Step 1

let test = 0

const emitter = new Emitter()

emitter.emit('sum', 1)

expect(test).toBe(0) // should trigger nothing

const sub1 = emitter.subscribe('sum', num => (test = test + num))

emitter.emit('sum', 1)

expect(test).toBe(1) // should trigger 1 callback

const sub2 = emitter.subscribe('sum', num => (test = test * num))

emitter.emit('sum', 2)

expect(test).toBe(6) // should trigger 2 callbacks

// Step 2

sub1.release()

emitter.emit('sum', 3)

expect(test).toBe(18) // should trigger 1 callback

// Step 3

const myEvent1 = emitter.subscribe('myEvent', () => 1)
const myEvent2 = emitter.subscribe('myEvent', () => 2)
const myEvent3 = emitter.subscribe('myEvent', () => true)
const result = emitter.emit('myEvent')

expect(result).toEqual([1, 2, true])

console.info('You passed the test!!! 👏🏼 👏🏼 👏🏼')

```
