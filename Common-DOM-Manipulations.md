---
title: Common DOM Manipulations
comments: true
date: 2018-10-15 17:40:29
type: categories
categories: Full stack
tags:
- DOM
- Vanilla JS
---

## `Window` VS `document`

1. `window`: a global object, and everything runs under it, including `document`
   - global variables, 
   - global functions, 
   - `location`
   - `history`
   - `setTimeout`
   - ajax call (XMLHttpRequest),
   - `console`
   - `localStorage`
2. `documnet`: represents the DOM and DOM is the object oriented representation of the html markup you have written; 
   - All the nodes are part of document. Hence you can use getElementById or addEventListener on document.

## `window.onload` vs `document.onload`

1. `window.onload`: When <u>**EVERYTHING**</u> is loaded. DOM is ready and all the contents including images, css, scripts, sub-frames, etc. finished loaded.
2. `document.onload`: once the **<u>DOM</u>** is loaded, regardless of the css, scripts,...

## `attribute` vs `property`

1. `attribute`: belongs to <u>HTML tag</u>, is **<u>static</u>** after being initialized

2. `property`: belongs to <u>DOM node</u>, is **<u>dynamic</u>** while interact with its element

3. ```html
   <input id="my-input" type="text" value="Name:">
   <script>
       var myInput = document.getElementById('my-input');
       myInput.getAttribute('value'); //"Name:"
       myInput.value; //'my dude'
   </script>
   ```



## Add `class` to element: `classList`

```js
function addClass(selector, className){
   var elm = document.querySelector(selector);
   if (elm){
      elm.classList.add(className);
   }
}
```

## Check Descendant

```js
function isDescendant(parent, child){ 
  while(child.parentNode ){ 
    if(child.parentNode == parent)
      return true;
    else
      child = child.parentNode;
  }
  return false;
}
```

## Create DOM element: `innerHTML` vs `appendChild`

1. `innerHTML`: browser removes all the current children of the elements. Parse the string and assign the parsed string to the element as children. So, **<u>Slow!</u>**

   1. ```js
      var ul = document.getElementById('myList');
      
      el.innerHTML = '<li>Only one item</li>';     
      ```

2. `appendChild`: you create a new Element. Since you are creating it, browser doesnt have to parse string and there is no invalid html.  so, **<u>Fast</u>**!

   1. ```js
      var li = document.createElement("li");
      var text = document.createTextNode('Only one Item');
      
      li.appendChild(text);
      ul.appendChild(li);
      ```


## `CrateDocumentFragment`: improve performance 

1. `documentFragment` a lightweight or minimal part of a DOM or a subtree of a DOM tree. 

2. It is very helpful when you are manipulating a part of DOM for multiple times.

3.  It becomes expensive to hit a certain portion of DOM for hundreds time. You might cause reflow for hundred times. Stay tuned for reflow.

   - BAD practice: 

     - ```js
       //bad practice. you are hitting the DOM every single time
       var list = ['foo', 'bar', 'baz', ... ],
           el, text;
       for (var i = 0; i < list.length; i++) {
           el = document.createElement('li');
           text = document.createTextNode(list[i]);
           el.appendChild(text);
           document.body.appendChild(el);
       }
       ```

   - GOOD practice:

     - ```js
       
       //good practice. you causing reflow one time
       var fragment = document.createDocumentFragment(),
           list = ['foo', 'bar', 'baz', ...],
           el, text;
       for (var i = 0; i < list.length; i++) {
           el = document.createElement('li');
           text = document.createTextNode(list[i]);
           el.appendChild(text);
           fragment.appendChild(el);
       }
       document.body.appendChild(fragment);
             
       ```




## reflow

1. **<u>[reflow] :</u>** flow of the elements in the page is changed due to **<u>change size or position</u>**
   - When you change size or position of an element in the page, all the elements after it has to change their position according to the changes you made.
   -  For example, if you change height on an element, all the elements under it has to move down in the page to accomodate a change in height.
2. Reasons: 
   - change layout (geometry of the page)
   - resize the window
   - change height/width of any element
   - <u>changing font, or font size</u>
   - move DOM element (<u>animation</u>)
   - adding or removing stylesheet
   - calculating offset height or offset width
   - `display: none;`

## repaint

1. It happens when you change the **<u>look of an element</u>** without changing the <u>size and shape</u>. This doesn't cause reflow as geometry of the element didn't changed.
2. Reasons:
   - change background color
   - change text color
   - visibility hidden

## destroy button

Create a button that is destroyed by clicking on it but two new buttons are created in it's place.

​	

```html

<div id="wrapper">
    <button id="doubleHolder" class="double">double</button>
</div>
<script>
    document.getElementById('doubleHolder').addEventListener('click', function (e) {   
        var btn = document.createElement('button');
        btn.setAttribute('class', 'double');
        btn.innerHTML = 'double1';

        var btn2 = document.createElement('button');
        btn2.setAttribute('class', 'double');
        btn2.innerHTML = 'double2';
        let wrapper = document.getElementById('wrapper')
        wrapper.appendChild(btn);
        wrapper.appendChild(btn2);
        wrapper.removeChild(e.target);   
    }); 
</script>

```



## Get text of a Node: `innerHTML` `textContent` `innerText`

1. **innerHTML** parses content as HTML and takes longer. Also vulnerable to [XSS attacks](https://developer.mozilla.org/en-US/docs/Glossary/Cross-site_scripting)

2. **textContent** uses straight text, does not parse HTML, and is faster.

   1. ```js
      // Given the following HTML fragment:
      //   <div id="divA">This is <span>some</span> text</div>
      
      // Get the text content:
      var text = document.getElementById("divA").textContent;
      // |text| is set to "This is some text".
      
      // Set the text content:
      document.getElementById("divA").textContent = "This is some text";
      // The HTML for divA is now:
      //   <div id="divA">This is some text</div>
      Polyfill
      ```

3. **innerText** Takes styles into consideration. It won't get hidden text for instance.: Bad, causing **<u>reflow</u>** !



## defer vs async

1. `defer` & `async` are only effective in `<script>` tag.  HTML parser will ignore `defer` and `async` keyword for inline script (script that does not have a src attribute).
2. **<u>normal:</u>** When you have a plain script tag (no defer or async keyword), parser will <u>*pause parsing*</u>, script would be <u>*downloaded and exectuted*</u>. After that parsing resume
3. <u>**async:**</u>  downloading script won't affect HTML paring, but executing will pause HTML parsing. Once downloaded, script begin to executed, will pause HTML parsing
4. **<u>defer:</u>** downloading script won't affect HTML paring, also executing won't affect HTML parsing. The script with `defer` will executed util HTML finish parsing