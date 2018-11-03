---
title: '''this'' & arrow function'
comments: true
date: 2018-09-05 21:51:55
type: categories
categories: Full stack
tags: 
- ES6
- 'this'
- apply
- call
- bind
- arrow function
- scope
---



## I. `this` in function

1. **what is `this` ?**

   - all **<u>regular</u>** functions have `this` keyword
   - `this` determines how a function is called **<u>on whom</u>**!

2. **`this` in global context**

   - `this` refers  gloabl object, no matter in strict mode or not

     ```
     // In web browsers, the window object is also the global object:
     console.log(this === window); // true
     
     a = 37;
     console.log(window.a); // 37
     
     this.b = "MDN";
     console.log(window.b)  // "MDN"
     console.log(b)         // "MDN"
     ```

3. <u>**`this` in function context [important]**</u>

   There are 7 cases that different function context results in different `this` reference;

   > <u>The value of `this`</u> depend on how the function is called

   1. `this` in **<u>simple call</u>**

      - non-strict mode

        if `this` is not set explicitly, it default to global object

        ```js
        function f1() { return this; }
        
        // In a browser:
        f1() === window; // true 
        
        // In Node:
        f1() === global; // true
        ```

      - strict mode

        if `this` is not set explicitly, the value of `this` remains `undefined`

      - ```js
        function f2() {
          'use strict'; // see strict mode
          return this;
        }
        
        f2() === undefined; // true
        ```

      - change the value of `this`, using `apply()` or `call()`

        ```js
        // An object can be passed as the first argument to call or apply and this will be bound to it.
        var obj = {a: 'Custom'};
        
        // This property is set on the global object
        var a = 'Global';
        
        function whatsThis() {
          return this.a;  // The value of this is dependent on how the function is called
        }
        
        whatsThis();          // 'Global'
        whatsThis.call(obj);  // 'Custom'
        whatsThis.apply(obj); // 'Custom'
        ```

   2. `this` in **<u>`bind()` method</u>**

      - NOTE: `bind()` doesn't change the orignal function, instead it return a **<u>new function</u>** with updated value of `this`

        ```js
        function f() { return this.a; }
        
        var g = f.bind({a: 'azerty'});
        console.log(g()); // azerty
        
        var h = g.bind({a: 'yoo'}); // bind only works once!
        console.log(h()); // azerty
        
        var o = {a: 37, f: f, g: g, h: h};
        console.log(o.a, o.f(), o.g(), o.h()); // 37,37, azerty, azerty
        ```

   3. `this` in <u>**arrow function**</u>

      - `this` cannot be changed, it default to the value of the **<u>enclosing lexical context</u>**'s `this`

      - ```js
        var globalObject = this;
        var foo = (() => this);
        console.log(foo() === globalObject); // true
        ```

      - About `call()` & `apply()`

        - if `this` arg is passed to call, bind, or apply on invocation of an arrow function it will be **<u>ignored</u>**.

        - the `thisArg` still works

        - ```js
          // Call as a method of an object
          var obj = {func: foo};
          console.log(obj.func() === globalObject); // true
          
          // Attempt to set this using call
          console.log(foo.call(obj) === globalObject); // true
          
          // Attempt to set this using bind
          foo = foo.bind(obj);
          console.log(foo() === globalObject); // true
          ```

          **<u>another example:</u>**  `this` of arrow function, directly depends on its enclosing context 

          ```js
          // Create obj with a method bar that returns a function that
          // returns its this. The returned function is created as 
          // an arrow function, so its this is permanently bound to the
          // this of its enclosing function. The value of bar can be set
          // in the call, which in turn sets the value of the 
          // returned function.
          var obj = {bar: function() {
                              var x = (() => this);
                              return x;
                            }
                    };
          
          // Call bar as a method of obj, setting its this to obj
          // Assign a reference to the returned function to fn
          var fn = obj.bar();
          
          // Call fn without setting this, would normally default
          // to the global object or undefined in strict mode
          console.log(fn() === obj); // true
          
          // But caution if you reference the method of obj without calling it
          var fn2 = obj.bar;
          // Then calling the arrow function this is equals to window because it follows the this from bar.
          console.log(fn2()() == window); // true
          ```

   4. `this` in <u>**object method**</u>

      - When a function is called as a method of an object, its `this` is set to **<u>the object</u>** the method is called on.

      - ```js
        var o = {
          prop: 37,
          f: function() {
            return this.prop;
          }
        };
        
        console.log(o.f()); // 37
        ```

      - `this` on the object's prototype chain

        If the method is on an object's prototype chain, `this` refers to the object the method was called on, as if the method were on the object.

   5. `this` in <u>**constructor**</u>

      - `this` is bound to the **<u>new object being constructed</u>**

      - **#A#**  if `constructor` doesn't return: result of `new` will be the object bound to `this`

      - **#B#**  if `constructor` return an object: result of `new` will be the return value

      - ```js
        function C() { this.a = 37; }
        
        var o = new C();
        console.log(o.a); // 37
        
        function C2() {   
          this.a = 37;
          return {a: 38};
        }
        
        // if an object was returned during construction, then the new object that this was bound to simply gets discarded.
        
        o = new C2();
        console.log(o.a); // 38
        ```

   6. `this` in <u>**DOM event handler**</u>

      - `this` is set to the **<u>target element</u>** the event fired from

      - ```js
        // When called as a listener, turns the related element blue
        function bluify(e) {
          // Always true
          console.log(this === e.currentTarget); 
          // true when currentTarget and target are the same object
          console.log(this === e.target);
          this.style.backgroundColor = '#A5D9F3';
        }
        
        // Get a list of every element in the document
        var elements = document.getElementsByTagName('*');
        
        // Add bluify as a click listener so when the
        // element is clicked on, it turns blue
        for (var i = 0; i < elements.length; i++) {
          elements[i].addEventListener('click', bluify, false);
        }
        ```

   7. `this` in <u>**inline event handler**</u>

      - `this` is set to the DOM element **<u>where the listener is placed</u>**

      - however, only the outer code has its `this` set this way

      - ```html
        <!-- 1. this will refer to 'button' -->
        <button onclick="alert(this.tagName.toLowerCase());">
          Show this
        </button>
        
        <!-- 2. 'this' will refer to 'window' -->
        <button onclick="alert((function() { return this; })());">
          Show inner this
        </button>
        ```


## II. arrow function

1. Best suited for **<u>non-method</u>** functions, ie. not related to method in *object*

2. No separate `this` of its own, *using others'*

   - An arrow function does not have its own `this`

   - `this` value of the enclosing lexical context is used 

   - how to find `this` of enclosing lexical context:

     - check if `this`  is present in current scope;

     - check `this` is present in its enclosing scope;

     - ```js
       //---------- version1: works -----------------
       function Person(){
         this.age = 0;
       
         setInterval(() => {
           this.age++; // |this| properly refers to the Person object
         }, 1000);
       }
       
       var p = new Person();
       
       //---------- version2: doesn't work ----------
       function Person() {
         // The Person() constructor defines `this` as an instance of itself.
         this.age = 0;
       
         setInterval(function growUp() {
           // In non-strict mode, the growUp() function defines `this` 
           // as the global object (because it's where growUp() is executed.), 
           // which is different from the `this`
           // defined by the Person() constructor. 
           this.age++;
         }, 1000);
       }
       
       var p = new Person();
       ```


3. No binding of `arguments`

   - Arrow functions do not have their own [`arguments` object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments).

   - ```js
     var arguments = [1, 2, 3];
     var arr = () => arguments[0];
     
     arr(); // 1
     
     function foo(n) {
       var f = () => arguments[0] + n; // foo's implicit arguments binding. arguments[0] is n
       return f();
     }
     
     foo(3); // 6
     ```

   -  using [rest parameters](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/rest_parameters) is a good alternative to using an `arguments` object

   - ```js
     function foo(n) { 
       var f = (...args) => args[0] + n; 
       return f(10); 
     }
     
     foo(1); // 11
     ```


4. <u>**When not use arrow function?  ==>  when need `this` explicitly** </u>

   1. don't use as <u>methods of object</u>: 

      - because we need `this`, but arrow function doesn't have its own `this`

      - ```
        'use strict';
        
        var obj = {
          i: 10,
          b: () => console.log(this.i, this),
          c: function() {
            console.log(this.i, this);
          }
        }
        
        obj.b(); // prints undefined, Window {...} (or the global object)
        obj.c(); // prints 10, Object {...}
        ```

   2. don't use as <u>function constructor</u>:  because we will use `new` to construct , which need `this`

   3. don't add `prototype` to it:  because it doesn't have `this`

   4. don't use in `yeild` or `generators`: 
