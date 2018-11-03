---
title: Design Pattern - JS
comments: true
date: 2018-10-07 14:53:37
type: categories
categories: Full stack
tags: 
- Design Pattern
- Factory Pattern
- Constructor Pattern
- Command Pattern
- Prototype Pattern
- Mixin Pattern
- Singleton Pattern
- Observer Pattern
- Obervable
---



**<u>All Credit to:</u>** [Learning JavaScript Design Patterns](https://addyosmani.com/resources/essentialjsdesignpatterns/book/)

## Constructor Pattern

1. <u>**What**</u>: a constructor is a special method used to initialize a newly created object once memory has been allocated for it

2. **<u>Why need it</u>**

   - e.g.: create different student with: `name`, `age`, `eat` method

   - <u>without constructor :</u> each time we create a new student again and again

     - ```js
       let alice ={name: "alice", age: 12, eat: function(){ ... } }
       let bob  = {name: "bob",   age: 15, eat: function(){ ... } }
       let carol ={name: "carol", age: 11, eat: function(){ ... } }
       ```

   - <u>with constructor **[BAD]** :</u> each time we create a new student trivially 

     - ```js
       /*
        BAD: each object will take a copy of `eat()` from memory, 
             which is in fact not necessary
       */
       function Student( name, age ){
           let student = {}
           student.name = name
           student.age = age
           student.eat = function(){ console.log(name, " is eating")}
           return student
       }
       let alice = new Student("alice", 12)
       let bob   = new Student("bob",   15)
       let carol = new Student("carol", 11)
       alice.eat() 
       ```

   - <u>with constructor **[GOOD]** :</u> see next section **<u>Prototype pattern</u>**

     - ```js
       
       ```


## Prototype Pattern

 1. **<u>what:</u> ** view prototype pattern as being based on prototypal inheritance where we create objects which act as prototypes for other objects. 

 2. improvement of `Student` class, using `prototype.eat()`

     1. ```js
        /*
         GOOD: each object will share `eat()` of Student prototype
        */
        function Student( name, age ){
            this.name = name
            this.age = age
        }
        Student.prototype.eat = function(){console.log(this.name, " is eating")}
        
        let alice = new Student("alice", 12)
        let bob   = new Student("bob",   15)
        let carol = new Student("carol", 11)
        alice.eat()
        ```


## Factory Pattern

1. **<u>what :</u>** combine similar classes together into a factory, to make code more compact

2. **<u>Implement one!</u>**

   1. ```js
      // Types.js - Constructors used behind the scenes
       
      // A constructor for defining new cars
      function Car( options ) {
       
        // some defaults
        this.doors = options.doors || 4;
        this.state = options.state || "brand new";
        this.color = options.color || "silver";
       
      }
       
      // A constructor for defining new trucks
      function Truck( options){
       
        this.state = options.state || "used";
        this.wheelSize = options.wheelSize || "large";
        this.color = options.color || "blue";
      }
       
       
      // FactoryExample.js
       
      // Define a skeleton vehicle factory
      function VehicleFactory() {}
       
      // Define the prototypes and utilities for this factory
       
      // Our default vehicleClass is Car
      VehicleFactory.prototype.vehicleClass = Car;
       
      // Our Factory method for creating new Vehicle instances
      VehicleFactory.prototype.createVehicle = function ( options ) {
       
        switch(options.vehicleType){
          case "car":
            this.vehicleClass = Car;
            break;
          case "truck":
            this.vehicleClass = Truck;
            break;
          //defaults to VehicleFactory.prototype.vehicleClass (Car)
        }
       
        return new this.vehicleClass( options );
       
      };
       
      // Create an instance of our factory that makes cars
      var carFactory = new VehicleFactory();
      var car = carFactory.createVehicle( {
                  vehicleType: "car",
                  color: "yellow",
                  doors: 6 } );
       
      // Test to confirm our car was created using the vehicleClass/prototype Car
       
      // Outputs: true
      console.log( car instanceof Car );
       
      // Outputs: Car object of color "yellow", doors: 6 in a "brand new" state
      console.log( car );
      ```

## Singleton Pattern

1. **<u>What :</u>** It restricts instantiation of a class to a single object. Across the whole app, there is only one object of it exists.

2. **<u>Why :</u>**  

   1. can be used for <u>**data source of truth**</u>
   2. anyone can call its methods to update the data

3. **<u>Trick :</u>** we can store private variable as `closure`, as singleton used here

   1. the singleton is `instance`, its private property & methods are: `privateMethod`, `privateVariable`, `privateRandomNumber`

   2. but return `privateRandomNumber` as public varible can keep it alive

   3. ```js
      function init() {
          // private methods and fileds
          let privateVariable = "I am a private var";
          let privateRandomNumber = Math.random();
          function privateMethod(){console.log("I am private method" )}
       
          return {
            // Public methods and fields
            publicMethod: privateMethod,
            getPrivateVar: function(){ return privateVariable },
            getRandomNumber: function(){ return privateRandomNumber }
          };
      }
      ```

4. **<u>Implementation</u>**

   1. ```js
      var mySingleton = (function () {
        var instance; // Instance stores a reference to the Singleton
        function init() {
          // Singleton
          var privateRandomNumber = Math.random();
          return {
            // Public methods and variables
            getRandomNumber: function() {
              return privateRandomNumber;
            }
          };
        };
       
        return {
          // Get the Singleton instance if one exists
          // or create one if it doesn't
          getInstance: function () {
            if ( !instance ) { instance = init(); }
            return instance;
          }
        };
      })();
       
      
      
      var myBadSingleton = (function () {
        // Instance stores a reference to the Singleton
        var instance;
          function init() {
          // Singleton
          var privateRandomNumber = Math.random();
          return {
            getRandomNumber: function() {
              return privateRandomNumber;
            }
          };
        };
       
        return {
          // Always create a new Singleton instance
          getInstance: function () {
            instance = init();
            return instance;
          }
        };
       
      })();
       
       
      // Usage:
      var singleA = mySingleton.getInstance();
      var singleB = mySingleton.getInstance();
      console.log( singleA.getRandomNumber() === singleB.getRandomNumber() ); // true
       
      var badSingleA = myBadSingleton.getInstance();
      var badSingleB = myBadSingleton.getInstance();
      console.log( badSingleA.getRandomNumber() !== badSingleB.getRandomNumber() ); // true
       
      // Note: as we are working with random numbers, there is a
      // mathematical possibility both numbers will be the same,
      // however unlikely. The above example should otherwise still
      // be valid.
      ```


## Observer Pattern

1. **<u>Subject :</u>** an object maintains a list of objects depending on it (observers), automatically notifying them of any changes to state.

2. **<u>Observers :</u>** objects who are interesting in the messages from the subject

3. Compared to **<u>Observable</u>**

   1. <u>**Observer Pattern**</u>: <u>*actively*</u> notify all observers when events happen
   2. **<u>observable Pattern</u>**: <u>*passively / lazily*</u>  start the events watching process

4. <u>advantage</u>: focus on observers, not their actions/methods

5. <u>disadvantage</u>: cannot flexible on the observers' methods, must be the same

6. Implementation

   1. ```js
      // Util class: ObserverList, used by `Subject`
      function ObserverList(){
        this.observerList = [];
      }
       
      ObserverList.prototype.add = function( obj ){
        return this.observerList.push( obj );
      };
       
      ObserverList.prototype.count = function(){
        return this.observerList.length;
      };
       
      ObserverList.prototype.get = function( index ){
        if( index > -1 && index < this.observerList.length ){
          return this.observerList[ index ];
        }
      };
       
      ObserverList.prototype.indexOf = function( obj, startIndex ){
        var i = startIndex;
       
        while( i < this.observerList.length ){
          if( this.observerList[i] === obj ){
            return i;
          }
          i++;
        }
       
        return -1;
      };
       
      ObserverList.prototype.removeAt = function( index ){
        this.observerList.splice( index, 1 );
      };
      
      
      
      // -----1----- Subject
      function Subject(){
        this.observers = new ObserverList();
      }
       
      Subject.prototype.addObserver = function( observer ){
        this.observers.add( observer );
      };
       
      Subject.prototype.removeObserver = function( observer ){
        this.observers.removeAt( this.observers.indexOf( observer, 0 ) );
      };
       
      Subject.prototype.notify = function( context ){
        var observerCount = this.observers.count();
        for(var i=0; i < observerCount; i++){
          this.observers.get(i).update( context );
        }
      };
      
      // -----2----- Observer
      function Observer(){
        this.update = function(){
          // ...
        };
      }
      ```



## RxJS - Observable

1. **Observer pattern**: subject (publisher) & observer (subscriber)

   - use case: <u>add event to DOM</u> 

   - implementation

     - ```js
       // 1. Subject
       class Subject {
           
           constructor() {
               this.observerCollection = [];
           }
           
           registerObserver(observer) {
               this.observerCollection.push(observer);
           }
           
           unregisterObserver(observer) {
               let index = this.observerCollection.indexOf(observer);
               if(index >= 0) this.observerCollection.splice(index, 1);
           }
           
           notifyObservers() {
               this.observerCollection.forEach((observer)=>observer.notify());
           }
       }
       
       // 2. Observer
       class Observer {
           
           constructor(name) {
               this.name = name;
           }
           
           notify() {
               console.log(`${this.name} has been notified.`);
           }
       }
       
       // use case
       let subject = new Subject(); // 创建主题对象
       
       let observer1 = new Observer('semlinker'); // 创建观察者A - 'semlinker'
       let observer2 = new Observer('lolo'); // 创建观察者B - 'lolo'
       
       subject.registerObserver(observer1); // 注册观察者A
       subject.registerObserver(observer2); // 注册观察者B
        
       subject.notifyObservers(); // 通知观察者
       
       subject.unregisterObserver(observer1); // 移除观察者A
       
       subject.notifyObservers(); // 验证是否成功移除
       ```

2. **Iterator pattern**

   - <u>Why use it</u>:  offer a way to visit interal elements <u>without exposing internal implementing details 

   - drawback: one-way & irreversible

   - data structure of the return value: `next()` 

     - => `{ done: false, value: elementValue }`
     - => `{ done: true, value: undefined }`

   - Implementation of Array Iterator

     - ```js
       // Array Iterator using closure
       function makeIterator(array){
           var nextIndex = 0;
           
           return {
              next: function(){
                  return nextIndex < array.length ?
                      {value: array[nextIndex++], done: false} :
                      {done: true};
              }
           }
       }
       
       // use case
       var it = makeIterator(['yo', 'ya']);
       console.log(it.next().value); // 'yo'
       console.log(it.next().value); // 'ya'
       console.log(it.next().done);  // true
       
       ```

   - **ES6 Iterator: ** `Symbol.iterator` can create iterator for an iterable object

     - How to use?

     - ```js
       let arr = ['a', 'b', 'c'];
       let iter = arr[Symbol.iterator]();
       
       > iter.next()
       { value: 'a', done: false }
       > iter.next()
       { value: 'b', done: false }
       > iter.next()
       { value: 'c', done: false }
       > iter.next()
       { value: undefined, done: true }
       ```

     - iterable object: **Arrays, Strings, Maps, Sets, DOM**

3. **Observable** in RxJS

   1. ```js
      var observable = Rx.Observable.create(function (observer) {
        observer.next(1);
        observer.next(2);
        observer.next(3);
        setTimeout(() => {
          observer.next(4);
          observer.complete();
        }, 1000);
      });
      
      console.log('just before subscribe');
      observable.subscribe({
        next: x => console.log('got value ' + x),
        error: err => console.error('something wrong occurred: ' + err),
        complete: () => console.log('done'),
      });
      console.log('just after subscribe');
      ```

1. **Characters**

   - observable will not automatically run, only triggered by `subscribe()`; which is opposed to promise.

   - <u>observable is a function</u>, which can be observed by observer

   - <u>observer is a object,</u> which has three callback: next, error, complete

   - how to subscribe the observable? observable.subsribe(observer)

   - how to manage the relationships between observable and observer? -- subscription var subscription1 = observable.subscribe(observer1);

   - how to unsubscribe? --- subscription1.unsubscribe();

   - Subscribing to an Observable is analogous to calling a Function.

   - <u>two Observable subscribes trigger two separate side effects. As opposed to EventEmitters which share the side effects and have eager execution regardless of the existence of subscribers, Observables have no shared execution and are lazy.</u>


## Command Pattern

1. **<u>What :</u>** commonly exists in many tool `config.json` file: `run("fun", "arg1", arg2)`

2. **<u>How to Implement :</u>**

   1. ```js
      
      var carManager = {
          // request information
          requestInfo: function( model, id ){
              return "The information for " + model + " with ID " + id + " is foobar";
          },
      
          // purchase the car
          buyVehicle: function( model, id ){
              return "You have successfully purchased Item " + id + ", a " + model;
          },
      
          // arrange a viewing
          arrangeViewing: function( model, id ){
              return "You have successfully booked a viewing of " + model + " ( " + id + " ) ";
          }
      };
      
      carManager.execute = function ( name ) {
          return carManager[name] && carManager[name].apply( carManager, [].slice.call(arguments, 1) );
      };
      
      
      // Use case
      carManager.execute( "arrangeViewing", "Ferrari", "14523" );
      carManager.execute( "requestInfo", "Ford Mondeo", "54323" );
      carManager.execute( "requestInfo", "Ford Escort", "34232" );
      carManager.execute( "buyVehicle", "Ford Escort", "34232" );
      ```




## Mixin Pattern: `_.extend()`

1. **<u>what:</u>** Mixins allow objects to borrow (or inherit) functionality from them with a minimal amount of complexity

2. **<u>How to use:</u>**  using `_.extend()` from `Underscore.js`

   1. ```js
      var myMixins = {
        moveUp: function(){ console.log( "move up" ); },
        moveDown: function(){ console.log( "move down" ); },
        stop: function(){ console.log( "stop! in the name of love!" );}
      };
      
      // A skeleton carAnimator constructor
      function CarAnimator(){
        this.moveLeft = function(){
          console.log( "move left" );
        };
      }
       
      // A skeleton personAnimator constructor
      function PersonAnimator(){
        this.moveRandomly = function(){ /*..*/ };
      }
       
      // Extend both constructors with our Mixin
      _.extend( CarAnimator.prototype, myMixins );
      _.extend( PersonAnimator.prototype, myMixins );
       
      // Create a new instance of carAnimator
      var myAnimator = new CarAnimator();
      myAnimator.moveLeft();
      myAnimator.moveDown();
      myAnimator.stop();
       
      // Outputs:
      // move left
      // move down
      // stop! in the name of love!
      ```

3. Write ourselves:  `_.extend()` <=> `augument()`

   1. ```js
      // Define a simple Car constructor
      var Car = function ( settings ) {
       
          this.model = settings.model || "no model provided";
          this.color = settings.color || "no colour provided";
       
      };
       
      // Mixin
      var Mixin = function () {};
       
      Mixin.prototype = {
       
          driveForward: function () {
              console.log( "drive forward" );
          },
       
          driveBackward: function () {
              console.log( "drive backward" );
          },
       
          driveSideways: function () {
              console.log( "drive sideways" );
          }
       
      };
       
       
      // Extend an existing object with a method from another
      function augment( receivingClass, givingClass ) {
       
          // only provide certain methods
          if ( arguments[2] ) {
              for ( var i = 2, len = arguments.length; i < len; i++ ) {
                  receivingClass.prototype[arguments[i]] = givingClass.prototype[arguments[i]];
              }
          }
          // provide all methods
          else {
              for ( var methodName in givingClass.prototype ) {
       
                  // check to make sure the receiving class doesn't
                  // have a method of the same name as the one currently
                  // being processed
                  if ( !Object.hasOwnProperty.call(receivingClass.prototype, methodName) ) {
                      receivingClass.prototype[methodName] = givingClass.prototype[methodName];
                  }
              }
          }
      }
       
       
      // Augment the Car constructor to include "driveForward" and "driveBackward"
      augment( Car, Mixin, "driveForward", "driveBackward" );
       
      // Create a new Car
      var myCar = new Car({
          model: "Ford Escort",
          color: "blue"
      });
       
      // Test to make sure we now have access to the methods
      myCar.driveForward();
      myCar.driveBackward();
       
      // Outputs:
      // drive forward
      // drive backward
       
      // We can also augment Car to include all functions from our mixin
      // by not explicitly listing a selection of them
      augment( Car, Mixin );
       
      var mySportsCar = new Car({
          model: "Porsche",
          color: "red"
      });
       
      mySportsCar.driveSideways();
       
      // Outputs:
      // drive sideways
      ```


