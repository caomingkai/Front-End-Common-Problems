---
title: 'Note of Eloquent JavaScript - [Browser]'
comments: true
date: 2018-10-03 15:53:04
type: categories
categories: Full stack
tags: 
- ES6
- debounce
- throttle
- DOM
- Event Handler
---

## I. DOM

1. **what:** document object model

2. **who&why**: 

   - <u>the browser</u> build the DOM
   - used to <u>draw the page</u> on screen
   - provide <u>interface to JS</u> to dynamically manipulate elements on the page 

3. **How to access it:** **`document` object**

   - `document.documentElement` => '<html> ...</html>'

4. **moving throught the DOM tree**

   - **traverse only element nodes**
     - `children`
     - 
   - **traverse all node types**: element node, text node, comment node, ...
     - `childNodes`
     - `parentNode`
     - `firstChild`
     - `lastChild`
     - `previousSibling`
     - `nextSibling`

5. **traverse tree e.g.:**

   - <u>recursive functions</u> are often useful. The following function scans a document for text nodes containing a given string and returns `true` when it has found one:

     - ```js
       function talksAbout(node, string) {
         if (node.nodeType == Node.ELEMENT_NODE) {
           for (let i = 0; i < node.childNodes.length; i++) {
             if (talksAbout(node.childNodes[i], string)) {
               return true;
             }
           }
           return false;
         } else if (node.nodeType == Node.TEXT_NODE) {
           return node.nodeValue.indexOf(string) > -1;
         }
       }
       
       console.log(talksAbout(document.body, "book"));
       // → true
       ```

6. **find element**

   1. `document.querySelector("idName");`

      -  first [`Element`](https://developer.mozilla.org/en-US/docs/Web/API/Element) within the document that matches the specified selector, or group of selectors. If no matches are found, `null` is returned.
      - `var el = document.querySelector(".myclass");`
      - `var el = document.querySelector("div.user-panel.main input[name='login']");`
        - Here, the first **&lt;input>** element with the name "login" (`<input name="login"/>`) located inside a **&lt;div>** whose class is "user-panel main" (`<div class="user-panel main">`) in the document is returned.

   2. `document.getElementById("idName");`

   3. `document.getElementsByTagName("idName");`

   4. `document.getElementsByClassName("idName");`

   5. `document.getElementsByName()`  when inside html we have `<input name="down">`

7. **change document**

   1. ```html
      <script>
        let paragraphs = document.body.getElementsByTagName("p");
        document.body.insertBefore(paragraphs[2], paragraphs[0]);
      </script>
      ```

8. **create nodes**

   1. `document.createTextNode(image.alt);`

   2. `image.parentNode.replaceChild(text, image);`

   3. ```html
      <p>The <img src="img/cat.png" alt="Cat"> in the
        <img src="img/hat.png" alt="Hat">.</p>
      
      <p><button onclick="replaceImages()">Replace</button></p>
      
      <script>
        function replaceImages() {
          let images = document.body.getElementsByTagName("img");
          for (let i = images.length - 1; i >= 0; i--) {
            let image = images[i];
            if (image.alt) {
              let text = document.createTextNode(image.alt);
              image.parentNode.replaceChild(text, image);
            }
          }
        }
      </script>
      ```

9. **attributes**

   - HTML allows setting any attribute you want on nodes. `<p data="52">`
   - `setAttribute` & `getAttribute`

   - original attributes: `href`, `class`, `style`
   - custom attributes: `data`, `myown`

## II. CSS

1. **CSS specificity**

   - inline_style=1000

   - id=100

   - class=10

   - element=1

   - e.g. 

     - ```css
       A: h1
       B: #content h1
       C: <div id="content"><h1 style="color: #ffffff">Heading</h1></div>
       
       The specificity of A is 1 (one element)
       The specificity of B is 101 (one ID reference and one element)
       The specificity of C is 1000 (inline styling)
       
       Since 1 < 101 < 1000, the third rule (C) has a greater level of specificity, and therefore will be applied.
       
       ```

2. **multiple class name for element**: `<p class="leftAlign title"> content </p>`

3. **e.g.:**

   1. ```css
      /* p elements with id 'main' and with classes a and b */
      p#main.a.b {
        margin-bottom: 20px;
      }
      ```

   2.  `p > a {…}` applies the given styles to all `<a>` tags that are **direct** children of `<p>` tags. 

   3.  `p a {…}` applies to all `<a>` tags inside `<p>`tags, whether they are **direct or indirect** children.

4. **position & animation**

   1. <u>**position:**</u> `transform`: used for <u>rotation</u>, <u>move</u>, <u>skew</u>

      - `translate(x, y)`:  horizontally and vertically move
      - `translateX(x)`: horizontally move
      - `translateY(y)`: vertically move
      - `scale(x,y)`: zoom in & out according to x-axis/ y-axis
      - `rotate(90deg)`: **num&deg: **the angle is specified in the parameter 
      - `skew(x-angle,y-angle)`: **num&deg: **the angle is specified in the parameter 

   2. <u>**animation:**</u> 

      - `animation-name: mymove;`, `mymove` is created using `@keyframes` rule specifies the animation code

      - `animation-duration`:

      - `animation-delay`:

      - `animation-iteration-count: 2`:

      - e.g. 

        ```css
        div {
            width: 100px;
            height: 100px;
            background: red;
            position: relative;
            -webkit-animation-name: mymove; /* Safari 4.0 - 8.0 */
            -webkit-animation-duration: 5s; /* Safari 4.0 - 8.0 */
            animation-name: mymove;
            animation-duration: 5s;
        }
        
        /* Safari 4.0 - 8.0 */
        @-webkit-keyframes mymove {
            from {left: 0px;}
            to {left: 200px;}
        }
        
        @keyframes mymove {
            from {left: 0px;}
            to {left: 200px;}
        }
        ```


## III. event handler

1. ***event object***

   - event handler functions are passed an argument: ***event object***
   - `event.target.innerHTML` or `event.target.value`

2. **two ways to handle event**

   - `onclick` attribute: But can <u>*only add one handler*</u>, since a node can have only one `onclick` attr

   - `addEventListener(event, callback)`: can add <u>*many handlers*</u>

   - `removeEventListener(event, callback)`: will remove handler on this event

     - ```js
       <button>Act-once button</button>
       <script>
         let button = document.querySelector("button");
         function once() {
           console.log("Done.");
           button.removeEventListener("click", once);
         }
         button.addEventListener("click", once);
       </script>
       ```

3. **event loop**

   - Never put too much <u>synchronous</u> work inside <u>asynchronous</u> callback! It will make page freezing!
   - If have to, use the <u>**web worker**</u>:  is a JS *<u>process</u>* that runs alongside the main script, on its own timeline

4. **propagation** 

   - the event will like ' **<u>bubble</u>**', propagation upward to elements who also registered for this event

     - ```html
       // when inner div clicked, both "fun_2", "fun_1" will be called!!
       <div id='outer' onclick="fun_1()"> 
       	<div id='inner' onclick="fun_2()" >
       		
       	<div>
       </div>
       ```

   - `stopPropagation` can <u>prevent bubbling</u>, prevent handlers further up from receiving the event

     - ```JS
       <p>A paragraph with a <button>button</button>.</p>
       <script>
         let para = document.querySelector("p");
         let button = document.querySelector("button");
         para.addEventListener("mousedown", () => {
           console.log("Handler for paragraph.");
         });
         button.addEventListener("mousedown", event => {
           console.log("Handler for button.");
           event.stopPropagation();
         });
       </script>
       ```

5. **Default action**

   - **[Important]: **JavaScript event handlers are called ***before*** the default behavior takes place. 

   - If you click a link, you will be taken to the link’s target. 

   - If you press the down arrow, the browser will scroll the page down. 

   - If you right-click, you’ll get a context menu. 

   - ```js
     <a href="https://developer.mozilla.org/">MDN</a>
     <script>
       let link = document.querySelector("a");
       link.addEventListener("click", event => {
         console.log("Nope.");
         event.preventDefault();
       });
     </script>
     ```


5. event type:

   1. key

   2. mouse

   3. touchscreen

   4. **scroll**

      - Whenever an element is scrolled, a `"scroll"` event is fired on it

      - **e.g.:** draws a progress bar above the document and updates it to fill up as you scroll down

        - ```js
          <style>
            #progress {
              border-bottom: 2px solid blue;
              width: 0;
              position: fixed;
              top: 0; left: 0;
            }
          </style>
          <div id="progress"></div>
          <script>
            // Create some content
            document.body.appendChild(document.createTextNode(
              "supercalifragilisticexpialidocious ".repeat(1000)));
          
            let bar = document.querySelector("#progress");
            window.addEventListener("scroll", () => {
              let max = document.body.scrollHeight - innerHeight;
              bar.style.width = `${(pageYOffset / max) * 100}%`;
            });
          </script>
          ```

      - **scrollHeight**: `ENTIRE  content & padding (visible or not)`

      - **offsetHeight**: `VISIBLE content & padding` `+ border + scrollbar`

      - **clientHeight**: `VISIBLE content & padding`

      - https://stackoverflow.com/questions/22675126/what-is-offsetheight-clientheight-scrollheight

      - **outerHeight:** height in pixels of the whole browser window, browser's out-most height

      - **innerHeight:** Height in pixels of the browser window viewport including the horizontal scrollbar

      - https://developer.mozilla.org/en-US/docs/Web/API/Window/innerHeight

   5. **focus event**

      1. `focus` & `blur`

      2. e.g.: **prompt** 

         1. ```js
            <p>Name: <input type="text" data-help="Your full name"></p>
            <p>Age: <input type="text" data-help="Your age in years"></p>
            <p id="help"></p>
            
            <script>
              let help = document.querySelector("#help");
              let fields = document.querySelectorAll("input");
              for (let field of Array.from(fields)) {
                field.addEventListener("focus", event => {
                  let text = event.target.getAttribute("data-help");
                  help.textContent = text;
                });
                field.addEventListener("blur", event => {
                  help.textContent = "";
                });
              }
            </script>
            ```


## IV. Timers

1. `handler = setTimeout(callback, time)`
2. `clearTimeout(handler)`
3. `handler = setInterval(callback, time)`
4. `clearInterval(handler)`

5. e.g.: 

   1. <u>make a bomb</u>: cancel a function you have scheduled

      1. ```js
         // 1. make it explods after 500ms
         let bombTimer = setTimeout(() => {
           console.log("BOOM!");
         }, 500);
         
         // 2. since synchronous, can clear timer before it explods
         if (Math.random() < 0.5) {
           console.log("Defused.");
           clearTimeout(bombTimer);
         }
         ```

   2. make a ticker: 

      1. ```js
         let ticks = 0;
         let clock = setInterval(() => {
           console.log("tick", ticks++);
           if (ticks == 10) {
             clearInterval(clock);     // can access outter var: clock
             console.log("stop.");
           }
         }, 200);
         ```


## V. Debounce / Throttle

1. **why <u>Debounce & Throttling</u>?** 

   - **<u>Debounce</u>**: For events like <u>keydown, scroll</u>, we don't want to trigger event in the middle of it, but only want to trigger after user pause! *(ie. we only care the **final result**)*
   - <u>**Throttling**</u>: For events like <u>mouseover</u>, we don't want to trigger event every for every single move, but only want to trigger it every `200ms` if such event happens (*ie. we only care **sample results***)

2. **Debounce e.g. : **debouce keydown  [ `setTimeout()` ]

   1. ```js
      // 1. clear last timer for each keydown event, 
      // 2. execute callback function after setting time
      
      // This way, if two keydown events happen less than 500ms, 
      // the first one will be cleard, won't execute the callback function
      <textarea>Type something here...</textarea>
      <script>
        let textarea = document.querySelector("textarea");
        let timeout;
        textarea.addEventListener("input", () => {
          clearTimeout(timeout);  
          timeout = setTimeout(() => {console.log("Typed!"); }, 500);
        });
      </script>
      ```

3. **Throttling e.g.: **throttle mouseover[ `setTimeout()` ]

   1. ```js
      // 1. scheduled first sampling
      // 2. only when scheduled, we execute function;
      //    - set scheduled to be false
      //    - after 1000ms, set scheduled to be true
      <script>
        let scheduled = true;
        window.addEventListener("mousemove", event => {
          if (scheduled) {
            document.body.textContent = `Mouse at ${event.pageX}, ${event.pageY}`;
            scheduled = false;
            setTimeout(() => { scheduled = true; }, 1000);
          }
        });
      </script>
      ```

4. **<u>Debouncify a function</u>**

   - Given a function and a delay period, we want return a ***debouncified function***:

     - whenever called, will only be executed  after the specified delay. 
     - Ignore calls in the middle of the delay, and reschedule the starting time of delay
     - must receive variant number of arguments

   - code implementation

     -  '**<u>closure</u>**'  is used here: the returned function have access to its outer function var: `timeout_handler`, and modify it by the returned functions

     - ```js
       /* Each time the returned function is called,
        * First clear the previous scheduled 'func' call
        * Then, reschedule a new one
        * <NOTE> we used 'closure' here: the returned function have
        * access to its outer function var: timeout_handler, and modify  
        * it
        */ 
       function debouncify( func, delay ){
           let timeout_handler   
           return function(){       // Closure is used here
               clearTimeout(timeout_handler)
               timeout_handler = setTimeout(function(arguments){
                   func(arguments)
               }, delay)
           }
       }
       
       // test
       function printTime(){
           console.log(new Date().toLocaleTimeString());
       }
       
       let printTimeAfterFiveSecs = debouncify(printTime, 5000)
       // need to call 
       console.log(new Date().toLocaleTimeString()) // get the start time
       printTimeAfterFiveSecs()
       printTimeAfterFiveSecs()
       printTimeAfterFiveSecs()
       printTimeAfterFiveSecs()
       printTimeAfterFiveSecs()
       printTimeAfterFiveSecs()
       printTimeAfterFiveSecs()   // should be 5sec after the start time
       
       ```
