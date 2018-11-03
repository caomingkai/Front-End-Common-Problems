---
title: Note of Eloquent JavaScript - [Pure JS]
comments: true
date: 2018-10-02 22:22:23
type: categories
categories: Full stack
tags: ES6
---



## I. Value, Type, Operators

1. Object, Number, String, Boolean, null, undefined, Symbol

   - Symbol: used for global unique value in Map: https://zhuanlan.zhihu.com/p/22652486

2. `typeof` will return **lowercase string type**

   - `typeof null` => `"object"`
   - `typeof undefined` => `"undefined"`
   - `typeof 5` => `"number"`
   - `typeof Number("4")` => `"number"`
   - `typeof NaN` => `"number"`
   - `typeof Infinity` => `"number"`
   - `typeof [1,2]` => `"object"`

3. comparsion

   There is only one value in JavaScript that is not equal to itself, and that is `NaN`(“not a number”).

   ```
   console.log(NaN == NaN)  // false !!!!
   ```

4. operator used on string / null / number

   ```
   console.log(8 * null)   // → 0
   console.log("5" - 1)    // → 4
   console.log("5" + 1)    // → "51"  !!!
   console.log("5" * 1)    // → 5
   console.log("five" * 2) // → NaN
   console.log(false == 0) // → true
   ```

5. when `null` or `undefined` occurs on either side of the operator, it produces true only if both sides are one of `null` or `undefined`.

   ```
   console.log(null == undefined);
   // → true
   console.log(null == 0);
   // → falseProgram Structure
   ```



## II. Program Structure

1. **<u>scope</u>**: Each variable has a <u>*scope*</u>, which is the part of the program in which the binding is visible.

   - all variables in JS have its own scope, we can use `this` to refer it
   - useful in **Closure**

2. **<u>variable</u>** :are reference, think it as **tentscles**, not **boxes**

3. **<u>function (Important List Inside)</u>**: 

   - **<u>optional arguments:</u>**

     - If you pass too many, the extra ones are <u>ignored</u>. 
     - If you pass too few, the missing parameters get assigned the value `undefined`.
       - use default value like this: `function power(base, exponent = 2) {...}`
     - How to call a pass a function with **<u>variant num of parameters</u>**?: `funName(...arg)`

   - **<u>function expression</u>**

     ```js
     // Define f to hold a function value
     const f = function(a) {
       console.log(a + 2);
     };
     ```

   - **<u>function declaration</u>**

     ```js
     // Declare g to be a function
     function g(a, b) {
       return a * b * 3.5;
     }
     ```

   - **<u>arrow function</u>**: automatically bind its `scope` to `this` inside the function (variables with implicit `this`)

     ```js
     // A less verbose function value
     let h = a => a % 3;
     ```

   - **<u>Closure</u>**

     - What happens to local bindings when the function call that created them is no longer active?
     - This feature—<u>*being able to reference a specific instance of a local variable in an **enclosing scope***</u>—is called ***[closure]***
     - *Local variables inside the outer function are created again for every call*
     - *Different calls can’t trample on one another’s local bindings.*

   - built-in functions

     - `prompt()` can get user's input using a popup box:  `let inputName = prompt("please input your name")`

     - `console.log()` can also print **<u>object</u>**, `console.log("this obj is", obj)`, instead of `console.log("this obj is" + obj)`


## III. Data Structures: Objects 
1. 

2. Property of Object** : 

   - Almost all JavaScript values have properties. The exceptions are `null` and `undefined`.
   - 2 ways to access properties:
     - obj.x
     - obj[a]    //  a = "x"

3. **Method of Object:**

   - 2 ways to call methods:
     - obj.x
     - **obj[a] (val)    //  a = "x"**

4. **Mutability:**

   - **Tip1:** Regardless of whether you use regular or strict equality, object comparisons only evaluate to `true` **if you compare the same exact object**.

   - Why we need **Mutability**? - since we want want one obj affect others when we change one of them

     ```js
     let object1 = {value: 10};
     let object2 = object1;
     
     object1.value = 15; 		 // we update the value of object1
     console.log(object2.value);  // the value of object2 also changed, which is not what we want!
     ```

   - How to keep **Mutability**?

     ```js
     let object1 = {value: 10};
     let object2 = {...object1};
     
     object1.value = 15; 		 // we update the value of object1
     console.log(object2.value);  // the value of object2 also changed, which is not what we want!
     ```

## IV. Data Structures: Array
1. **`for...of` vs `for...in` vs `forEach()`** : 

    - `for..in` operates on **any object**; it serves as a way to inspect <u>enumerable properties</u> on this object

    - `for...of` : mainly used to iterate **only iterable object**, it serves as a way to inspect all <u>iterable entries</u> of this object; It cannot used on Object, can be only used on Map, Set, Array, String

      ```
      //  The e.g.
      
      Object.prototype.objCustom = function() {}; 
      Array.prototype.arrCustom = function() {};
      
      let iterable = [3, 5, 7];
      iterable.foo = 'hello';
      
      for (let i in iterable) {
        console.log(i); // logs 0, 1, 2, "foo", "arrCustom", "objCustom"
      }
      
      for (let i in iterable) {
        if (iterable.hasOwnProperty(i)) {
          console.log(i); // logs 0, 1, 2, "foo"
        }
      }
      
      for (let i of iterable) {
        console.log(i); // logs 3, 5, 7
      }
      ```

    - `forEach()`: is a **method** only belonging to the royal family of Arrays, including Map, Set (since Map, Set are implemented using Array). It iterate <u>iterable elements</u> in an array. It mainly depends on its callback function

      - `map.forEach( callback(val, key, map) )`
      - `array.forEach( callback(val, index, array) )`
      - `set.forEach( callback(val, key, set) )`

2. **common methods**

    - `includes()`: determines whether an array includes a certain element, returning `true` or `false` as appropriate.

        - ```js
            var array1 = [1, 2, 3];
            
            console.log(array1.includes(2));
            // expected output: true
            
            var pets = ['cat', 'dog', 'bat'];
            
            console.log(pets.includes('cat'));
            // expected output: true
            
            console.log(pets.includes('at'));
            // expected output: false
            ```

    - `forEach()`

    - `map()`

    - `reduce()`

    - `filter()`

    - `push()`: push on end

    - `pop()`: pop from end

    - `shift()`: remove from front

    - `unshift()`: add to front

    - `sort()`

    - `reverse()`

    - `slice()`: return a new array (sub-array)

    - `splice()`: modify current array (current array)

    - `join()`: returns a new string by concatenating all of the elements in an array

        - ```js
            var elements = ['Fire', 'Wind', 'Rain'];
            
            console.log(elements.join());
            // expected output: Fire,Wind,Rain
            
            console.log(elements.join(''));
            // expected output: FireWindRain
            
            console.log(elements.join('-'));
            // expected output: Fire-Wind-Rain
            ```

    - `concat()`

    - `indexOf()`

    - `lastIndexOf()`

    - `find()`:  returns the **value** of the **first element** in the array that satisfies the provided testing function. Otherwise [`undefined`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/undefined) is returned.

        - ```js
            var array1 = [5, 12, 8, 130, 44];
            
            var found = array1.find(function(element) {
              return element > 10;
            });
            
            console.log(found);
            // expected output: 12
            
            ```

    - `findIndex()`: returns the **index** of the first element in the array **that satisfies the provided testing function**. Otherwise, it returns -1

        - ```js
            var array1 = [5, 12, 8, 130, 44];
            
            function findFirstLargeNumber(element) {
              return element > 13;
            }
            
            console.log(array1.findIndex(findFirstLargeNumber));
            // expected output: 3
            
            ```

    - `values()`:  returns a new **Array Iterator** object that contains the values for each index in the array

        - ```js
            const array1 = ['a', 'b', 'c'];
            const iterator = array1.values();
            
            for (const value of iterator) {
              console.log(value); // expected output: "a" "b" "c"
            }
            
            ```

    - `fill()`: fills all the elements of an array from a start index to an end index with a static value. The end index is not included.

        - ```js
            var array1 = [1, 2, 3, 4];
            
            // fill with 0 from position 2 until position 4
            console.log(array1.fill(0, 2, 4));
            // expected output: [1, 2, 0, 0]
            
            // fill with 5 from position 1
            console.log(array1.fill(5, 1));
            // expected output: [1, 5, 5, 5]
            
            console.log(array1.fill(6));
            // expected output: [6, 6, 6, 6]
            ```

    - `flat()`:  creates a new array with all sub-array elements concatenated into it recursively up to the specified depth.

        - ```js
            var arr1 = [1, 2, [3, 4]];
            arr1.flat(); 
            // [1, 2, 3, 4]
            
            var arr2 = [1, 2, [3, 4, [5, 6]]];
            arr2.flat();
            // [1, 2, 3, 4, [5, 6]]
            
            var arr3 = [1, 2, [3, 4, [5, 6]]];
            arr3.flat(2);
            // [1, 2, 3, 4, 5, 6]
            ```


## IV. Data Structures: Map

1.  **internal structure**
   - `new Map([iterable])`
   - iterable:  <u>key-value pairs</u> (**arrays  with two elements**, e.g. `[[ 1, 'one' ],[ 2, 'two' ]]`)
   - `new Map( [ ["a", 1], ["b", 2], ["c", 3], ["d", 4] ] )`
2. **Map vs Object**
   - The keys of an `Object` are [`Strings`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String) and [`Symbols`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol), whereas they can be any value for a `Map`, including functions, objects, and any primitive.
   - The keys in Map are ordered while keys added to object are not. Thus, when iterating over it, a Map object returns keys in order of insertion.
   - You can get the size of a `Map` easily with the `size` property, while the number of properties in an `Object` must be determined manually.
   - A `Map` is an [iterable](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/iterable) and can thus be directly iterated, whereas iterating over an `Object` requires obtaining its keys in some fashion and iterating over them.
   - An `Object` has a prototype, so there are default keys in the map that could collide with your keys if you're not careful. As of ES5 this can be bypassed by using `map = Object.create(null)`, but this is seldom done.
   - A `Map` may perform better in scenarios involving frequent addition and removal of key pairs.

## V. Higher-Order Functions

1.  **what?**

   - Functions that **<u>operate on other functions</u>**, either by taking them <u>as arguments</u> or <u>as return value</u>, are called *higher-order functions*

2. **why?   => handle actions, not just values**

   - **eg1 : use function as arguments**

   - ```js
     function add(obj){ obj += 1 };
     function substract(obj){ obj -= 1};
     
     function repeatAction(times, action) {
       for (let i = 0; i < times; i++) {
         action(i);
       }
     }
     
     repeatAction(5, add );
     repeatAction(5, substract );
     ```

   - **eg2 : use function as return value**

     ```js
     // utility function: returns another function,
     // can check if a number > certain number
     
     function greaterThan(n) {
       return m => m > n;
     }
     
     let greaterThan10 = greaterThan(10);
     console.log(greaterThan10(11));
     // → true
     ```

   - **eg3:** 

   - ```js
     // ----------- 1. filter() -----------
     function filter(array, test) {
       let passed = [];
       for (let element of array) {
         if (test(element)) {
           passed.push(element);
         }
       }
       return passed;
     }
     
     console.log(filter(SCRIPTS, script => script.living));
     // → [{name: "Adlam", …}, …]
     
     
     // ----------- 2. map() -----------
     function map(array, transform) {
       let mapped = [];
       for (let element of array) {
         mapped.push(transform(element));
       }
       return mapped;
     }
     
     let rtlScripts = SCRIPTS.filter(s => s.direction == "rtl");
     console.log(map(rtlScripts, s => s.name));
     // → ["Adlam", "Arabic", "Imperial Aramaic", …]
     
     
     // ----------- 3. reduce() -----------
     function reduce(array, combine, start) {
       let current = start;
       for (let element of array) {
         current = combine(current, element);
       }
       return current;
     }
     
     console.log(reduce([1, 2, 3, 4], (a, b) => a + b, 0));
     // → 10
     ```

3. **Composability** [<u>**pipeline**</u>]

   - <u>what</u>:  we want to *compose* operations, to make **meaningful & readable** code

   - <u>How</u>:  create functions, to make a series action into a ***<u>pipeline</u>***

   - e.g.:

     1. we start with all scripts, 
     2. filter out the living (or dead) ones, 
     3. take the years from those, 
     4. average them, 
     5. and round the result.

   - ```js
     function average(array) {
       return array.reduce((a, b) => a + b) / array.length;
     }
     
     console.log(Math.round(average(
       SCRIPTS.filter(s => s.living).map(s => s.year))));
     ```


## VI. The life of Object

1. **Encapsulation / 封装**

   1. **What?**: 

      - <u>Internal details</u> of one object cannot be access from outside
      - objects communicate via their <u>interface</u>

   2. **How?**

      - <u>**private**</u>: put an **underscore** (`_`) character at the start of property names to indicate that those properties are private.

   3. **property**

      - belongs to that object itself
      - or can be traced back up to its prototype

   4. **Method**

      - each function has its own `this` property
      - **<u>arrow function</u>**: they do not bind their own `this` but can see the `this` binding of the scope around them

      - `this`: every method have this implicit parameter passed into the function. The following are the same

        ```js
        function speak(line) {
          console.log(`The ${this.type} rabbit says '${line}'`);
        }
        let whiteRabbit = {type: "white", speak};
        
        whiteRabbit.speak("Burp");		  // same effect
        speak.call(whiteRabbit, "Burp!"); // same effect
        ```

2. **Polymorphism**

   1. **overriding derived <u>property**</u>

      1. ```js
         Rabbit.prototype.teeth = "small";
         console.log(killerRabbit.teeth);
         // → small
         killerRabbit.teeth = "long, sharp, and bloody";
         console.log(killerRabbit.teeth);
         // → long, sharp, and bloody
         console.log(blackRabbit.teeth);
         // → small
         console.log(Rabbit.prototype.teeth);
         // → small
         ```

   2. **override derived <u>method</u>**

      1. ```js
         Rabbit.prototype.toString = function() {
           return `a ${this.type} rabbit`;
         };
         
         console.log(String(blackRabbit));
         // → a black rabbit
         ```

   3. **Why use Map instead of pure Object?**

      - We certainly didn’t list anybody named toString in our map. 

      - Yet, because plain objects derive from `Object.prototype`, it looks like the property is there.

      - ```
        let ages = {
          Boris: 39,
          Liang: 22,
          Júlia: 62
        };
        
        console.log("Is Jack's age known?", "Jack" in ages);
        // → Is Jack's age known? true
        console.log("Is Jack's age known?", "Jack" in ages);
        // → Is Jack's age known? false
        console.log("Is toString's age known?", "toString" in ages);
        // → Is toString's age known? true 
        // ====>【what we don't want！！！】
        ```

   4. getters, setters, and static

      1. `get`: The **get** syntax binds an object property to a function that will be called when that property is looked up.

         1. ```
            var obj = {
              log: ['a', 'b', 'c'],
              get latest() {
                if (this.log.length == 0) {
                  return undefined;
                }
                return this.log[this.log.length - 1];
              }
            }
            
            console.log(obj.latest);
            ```

      2. `set`: The **set** syntax binds an object property to a function to be called when there is an attempt to set that property.

         1. ```js
            var language = {
              set current(name) {
                this.log.push(name);
              },
              log: []
            }
            
            language.current = 'EN';
            language.current = 'FA';
            
            console.log(language.log);
            // expected output: Array ["EN", "FA"]
            ```

      3. `static`: methods that have `static` written before their name are stored on the constructor.

         1. ```js
            class Temperature {
              constructor(celsius) {
                this.celsius = celsius;
              }
              get fahrenheit() {
                return this.celsius * 1.8 + 32;
              }
              set fahrenheit(value) {
                this.celsius = (value - 32) / 1.8;
              }
            
              static fromFahrenheit(value) {
                return new Temperature((value - 32) / 1.8);
              }
            }
            
            let temp = new Temperature(22);
            console.log(temp.fahrenheit);
            // → 71.6
            temp.fahrenheit = 86;
            console.log(temp.celsius);
            // → 30
            
            Temperature.fromFahrenheit(100)
            // -> Temperature {celsius: 37.77777777777778}
            ```

3. **Inheritance**

   1. `extends` operator

      1. ```js
         class SymmetricMatrix extends Matrix {
           constructor(size, element = (x, y) => undefined) { 
             super(size, size, (x, y) => { // 'super' call its parents constructor
               if (x < y) return element(y, x);
               else return element(x, y);
             });
           }
         
           set(x, y, value) {
             super.set(x, y, value);       // 'super' call its parents methods
             if (x != y) {
               super.set(y, x, value);     // 'super' call its parents methods
             }
           }
         }
         
         let matrix = new SymmetricMatrix(5, (x, y) => `${x},${y}`);
         console.log(matrix.get(2, 3));
         // → 3,2
         ```

   2. `instanceof` operator

      1. ```js
         console.log(
           new SymmetricMatrix(2) instanceof SymmetricMatrix);
         // → true
         console.log(new SymmetricMatrix(2) instanceof Matrix);
         // → true
         console.log(new Matrix(2, 2) instanceof SymmetricMatrix);
         // → false
         console.log([1] instanceof Array);
         // → true
         ```

   3. **prototype: 也是一个object, 用来表明该object中的哪些method用来 inherite**

      - `prototype`  vs `__proto__`

        - `prototype`: an shared object contains all methods to be inherited by child objects
        - `__proto__`: indicating the object it's inheriting, ie, `obj.prototype`

      - **what**:  can be treated as <u>parent class</u> in java. A prototype is another object that is used as a fallback source of properties.

      - `object.prototype === null`

      - `Object.create()`:  create an object using anther object as the prototype. like `constructor()`

      - ```js
        let protoRabbit = {
          speak(line) {
            console.log(`The ${this.type} rabbit says '${line}'`);
          }
        };
        let killerRabbit = Object.create(protoRabbit);
        killerRabbit.type = "killer";
        killerRabbit.speak("SKREEEE!");
        // → The killer rabbit says 'SKREEEE!'
        ```

   4. **class**: clearer way to create prototype

      - **What & Details**

        - The one named `constructor` is treated specially. It provides the **<u>actual constructor function</u>**, which will be bound to the name `Rabbit`. 
        - The others are packaged into that **<u>constructor’s prototype</u>**. Thus, the earlier class declaration is equivalent to the constructor definition from the previous section. It just looks nicer.

      - ```js
        class Rabbit {
          constructor(type) {
            this.type = type;
          }
          speak(line) {
            console.log(`The ${this.type} rabbit says '${line}'`);
          }
        }
        
        let killerRabbit = new Rabbit("killer");
        let blackRabbit = new Rabbit("black");
        ```


      - NOTE: the <u>constructor</u>  are **capitalized**
    
      - ```js
        function Rabbit(type) {   // note: name are capitalized
          this.type = type;
        }
        Rabbit.prototype.speak = function(line) {
          console.log(`The ${this.type} rabbit says '${line}'`);
        };
        
        let weirdRabbit = new Rabbit("weird");
        ```


## VII. Bugs

1. strict mode

   1. why?   => <u>we don't want to write bugs!</u>

   2. how?

      - `use strict` <u>at  top of a file</u>

      - `use strict` in body of function

      - ```js
        function canYouSpotTheProblem() {
          "use strict";
          for (counter = 0; counter < 10; counter++) {
            console.log("Happy happy");
          }
        }
        
        canYouSpotTheProblem();
        // → ReferenceError: counter is not defined
        ```

2. **regular express**

   1. A regular expression is **a type of object**

      - with the `RegExp` constructor
      - as a literal value by enclosing a pattern in forward slash (`/`) characters

   2. ```
      let re1 = new RegExp("abc");
      let re2 = /abc/;
      ```

## VIII. Modules

1. **what & why**

   - <u>Modules</u> are code snippet can be reused repeatedly
   - We want do **DRY**

2. **Inner implementation: [ important ]**

   - we need JS has the ability to <u>**execute string**</u> of code, because what imported in are **strings**!

   - 2 ways to execute strings as code in JS

     - `eval()`: drawback => **<u>change scope!</u>**

       - ```js
         const x = 1;
         function evalAndReturnX(code) {
           eval(code);
           return x;
         }
         
         console.log(evalAndReturnX("var x = 2"));
         // → 2
         console.log(x);
         // → 1
         ```

     - `Function ( params_string, body_string )`:  **<u>self-contained module scope</u>**

       - ```js
         let plusOne = Function("n", "return n + 1;");
         console.log(plusOne(4));
         // → 5
         ```


3. **CommonJS:  `require` &  `exports`**

   - How to use

     - ```js
       const ordinal = require("ordinal");
       const {days, months} = require("date-names");
       
       exports.formatDate = function(date, format) {
         return format.replace(/YYYY|M(MMM)?|Do?|dddd/g, tag => {
           if (tag == "YYYY") return date.getFullYear();
           if (tag == "M") return date.getMonth();
           if (tag == "MMMM") return months[date.getMonth()];
           if (tag == "D") return date.getDate();
           if (tag == "Do") return ordinal(date.getDate());
           if (tag == "dddd") return days[date.getDay()];
         });
       };
       ```

   - `require`: a function mainly used by CommonJS

     - calling a function to access a dependency
     - When you call this with the module name of a dependency, it makes sure the module is loaded and returns its interface.
     - The inner implementation of `require()`

     ```js
     require.cache = Object.create(null);    // a running-time  global obj
     
     function require(name) {
       if (!(name in require.cache)) {       // check if alreay has the module in cache
         let code = readFile(name);          // read [code STRING] from file
         let module = {exports: {}};         // an callback container to fetch real one
         require.cache[name] = module;       // bind this container to cache
         let wrapper = Function("require, exports, module", code);// 'eval' code String in <local function scope>
         wrapper(require, module.exports, module);   // put all the code to module, 
       }										// while tackle module of the required one
       return require.cache[name].exports;   // output the 
     }
     ```


4. **ES6 Module: `export` & `import`**

   1. what & why [*imported module are reference, not value*]

      - <u>Same</u>: try to tackle dependency, but
      - <u>Difference:</u> 
        - imported module using ES6 are reference, not value; So local scope can access its value when there is update inside the imported module
        - exporting module may change the value of the binding at any time, and the modules that import it will see its new value.

   2. How to use

      1. ```js
         import ordinal from "ordinal";
         import {days, months} from "date-names";
         
         export function formatDate(date, format) { /* ... */ }
         ```


## IX. Asynchronous Programming

1. callback function

   - Asynchronous behavior happens on its <u>own empty function call stack</u>

   - callbacks are called by others, not directly called by the code that scheduled them. 
   - If I call `setTimeout` from within a function, that function will have returned by the time the callback function is called. 
   - And when the callback returns, control does not go back to the function that scheduled it.

2. Promise

3. `async` & `await`

4. generator

   - What:  functions can be paused and then resumed again
   - how: `function* funName(){ ... }`

5. event loop

   - all asynchronous events are put onto a `Queue` 
   - all synchronous function are executed in `Stack`