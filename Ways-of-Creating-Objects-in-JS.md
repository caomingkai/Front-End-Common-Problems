---
title: 'Create new Objects with Inheritance'
comments: true
date: 2018-10-04 11:54:04
type: categories
categories: Full stack
tags: JavaScript Core
---

## I. **`{ }`: Literal Constructor**

1. ```js
   var person = {
       name = "Anand",
       getName = function(){
         return this.name 
       }
   }
   ```

## II. **`Object.create()`: can be used on inheritance** 

- This can create a new object from another object as it prototype

- More efficient to use a prototype if you are going to have multiple objects sharing properties or methods.

- ```js
  let proto = {name: "default", foo: function() {}}
  let obj1 = Object.create(proto); obj1.name = "a"
  let obj2 = Object.create(proto); obj1.name = "b"
  let obj3 = Object.create(proto); obj1.name = "c"
  let obj4 = Object.create(proto); obj1.name = "d"
  ```

## III. **Function Constructor with `this` & `new`**

- **Bad thing:** every time, when we create an new object, it will occupy space for all attributes of `Person`
- **Using `Prototype`**

- ```js
  function Person(name){
    this.name = name
    this.getName = function(){
      return this.name
    } 
  } 
  
  let mycar = new Person('Eagle');
  ```

## IV. **Function Constructor & Prototype**

- `prototype` is a **<u>shared object</u>**, where all its inheriting objects will share this same space, won't create extra space for attributes inherited from `prototype`

- ```js
  function Graph() {
    this.vertices = [];
    this.edges = [];
  }
  
  Graph.prototype = {
    addVertex: function(v) {
      this.vertices.push(v);
    }
  };
  
  var g = new Graph();
  // g is an object with own properties 'vertices' and 'edges'.
  // g.[[Prototype]] is the value of Graph.prototype when new Graph() is executed.
  ```

## V. **ES6 `class`**

1. introduced a new set of keywords implementing [classes](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes). The new keywords include [`class`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/class), [`constructor`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/constructor), [`static`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/static), [`extends`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes/extends), and [`super`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/super).

2. all functions except `constructor` are put into the prototype of that object (`Square`)

3. ```js
   class Polygon {
     constructor(height, width) {
       this.height = height;
       this.width = width;
     }
   }
   
   // inherit from another class
   class Square extends Polygon {
     constructor(sideLength) {
       super(sideLength, sideLength);
     }
     get area() {
       return this.height * this.width;
     }
     set sideLength(newLength) {
       this.height = newLength;
       this.width = newLength;
     }
   }
   
   // Use new to create object from class
   var square = new Square(2);
   ```

## VI. **Singleton: *<u>one object</u>* of a kind** 

-  `Singleton`  is used on class-based language

- In jS, All objects created using literal constructor are singletons

  - ```js
    var Robot = {
      metal: "Titanium",
      killAllHumans: function(){
        alert("Exterminate!");
      }
    };
    
    Robot.killAllHumans();
    ```
