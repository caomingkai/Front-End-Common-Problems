---
title: Facebook Interview Preparation
comments: true
date: 2018-10-06 19:07:47
type: categories
categories: Full stack
tags: 
- ReactJS
- DOM
- CSS
- HTML render
- AJAX
- Observable
- Accessibility
- Semantic tag
- Array-like Object
---



## HTML Accessibility Tag

1. To improve accessiblity, we are encouraged to use semantic tag of HTML5

   - **<u>MDS</u>**: https://developer.mozilla.org/en-US/docs/Learn/Tools_and_testing/Cross_browser_testing/Accessibility

2. Why to use it?

   - https://stackoverflow.com/questions/17272019/why-to-use-html5-semantic-tag-instead-of-div

3. Common semantic tags

   - **<u>Layout:</u>** `header` `footer`  `nav` `main` `section` `article` `aside`
     - `<header role="banner">`
     - `<section role="search">`
   - **<u>Form:</u>** `label` `input` `button`
     - `<label for="search">`
     - ` <input type="text" >` 
     - `<input type="submit" />`
   - **<u>Table:</u>** `<caption>` `<table summury="caption">` `th` 
   - **<u>Image:</u>** 
     - `<img alt="img.png" title="This is a beautiful girl" >`
     - `aria-labelledby`
     - empty `alt`: necessary if the image is only for decoration use
   - **<u>text:</u>**
     - `<em>` vs `<i>`
     - `<strong>` vs `<b>`
     - `<abbr title="Hypertext Markup Language"> HTML <abbr>`
   - **<u>position:</u>**
     - `absolution position`  is better than `display: none` or `visibility: hidden`: to overlap all div with different `z-index`
     - since `display: none` or `visibility: hidden` will hide content from screenreader

4. How to improve accessibility?

   - use `:focus` & `:hover` pseudo-class to highlight the chosen tag

     - ```css
       a:hover, input:hover, button:hover, select:hover,
       a:focus, input:focus, button:focus, select:focus {
         font-weight: bold;
       }
       ```

5. JavaScript in accessibility

   1. provide client-side form validating, rather than server-side
   2. `<video>` has control button for keyboard
   3. `mouseover` == `onfocus` &   `mouseout` == `onblur`

6. mobile accessibility

   - **<u>touch</u>**: `onmousedown` == `ontouchstart` && `onmouseup` == `ontouchstop`

   - **<u>side menu</u>**

   - `@media` 

     - <u>logical operator</u>: `and`, `not`, `only`, `,(like 'or')`

     - ```html
       <!-- 1. media in <link>  -->
       <link href="mobile.css" rel="stylesheet" media="screen and (max-width: 600px)">
       
       <!-- 2. media in <style>  -->
       <style media="all and (max-width: 500px)">
           p {
             color: blue;
             background-color: yellow;
           }
        </style> 
       
       <!-- 3. media in <script>  -->
       <script>
           if (window.matchMedia("(min-width: 400px)").matches) {
             // the viewport is at least 400 pixels wide
           } else {
             // the viewport is less than 400 pixels wide
           }
       </script>
       
       <!-- 4. media inside style  -->
       <style>
           @media screen and (min-width: 30em) and (max-width: 992px) {
             body { background-color: blue; }
           }
           
           @media (min-height: 680px), screen and (orientation: portrait){
               body { background-color: blue; }
           }
       </style>
       
       ```

   - `@import`

     - syntax: 

       ```css
       @import 'custom.css';
       @import url('landscape.css') screen and (orientation:landscape);
       ```

## Why use React?

Credit to: https://reactjs.org/blog/2013/06/05/why-react.html

1. React is not MVC, is only "V"
2. Support small, **reusable** component
3. efficient DOM update, using **diff algorithm**
   - `reconcilation process`: the way React used to update real DOM, by only updating changed components node and its children
   - Every time `props` or `state` change leading to `render()` execution, which will produce either a `String` or `virtual DOM node`; 
   - Either way, React will compare it with previous `String` or `virtual DOM node`, to determine which node in real DOM need to be updated
   - `diff/reconcilation` process only happens after `render()`, it will compare current tree with previous tree
4. `key` in a list of nodes: make `diff` efficiently
   1. there are cases that a node is added in front to a list, without `key`, React just compare each pair of nodes from previous tree and current tree, and re-render all the nodes in real DOM in that list; Even thought, it only needs to insert a new node to real DOM tree
   2. Attaching a `key` to each node in a list, will help React know which node need to be re-render in real  DOM tree.
   3. Nodes with the same `key` inside a list, will be stable across different re-renders

 

## React Lifecyle

1. Mounting phase

   - `constructor()`
   - `getDerivedStateFromProps()`
   - `render()`
   - `componentDidMount()`

2. Updating phase

   - `getDerivedStateFromProps()`
   - `shouldComponentUpdate()`
   - `render()`: after either new `props` or new `state`
   - `componentDidUpdate()`

3. Unmounting phase

   - `componentWillUnmount()`

4. Other API: `setState()` & `forceUpdate()`

   1. `setState()`: can take both <u>object</u> and **<u>callback function</u>** as parameter

   2. `setState( (prevState, props) => updatedState )` 

      1. ```js
         // BAD: updated state will only take one increment
         // i.e. in the end, counter=1
         componentDidMount(){
             this.setState({counter: this.state.counter+1})
             this.setState({counter: this.state.counter+1})
             this.setState({counter: this.state.counter+1})
         })
            
         // GOOD: updated state will take all increment
         // i.e. in the end, counter=3
         componentDidMount(){
             this.setState( counter=>{counter: state.counter + 1} )
             this.setState( counter=>{counter: state.counter + 1} )
             this.setState( counter=>{counter: state.counter + 1} )
         })
         ```


## How to implement infinite scroll loading

1. Credit to: https://segmentfault.com/a/1190000004974824

2. Which event do we use?

   - We assume the user will scroll down the screen, in order to load more content
   - So we use the `scroll` event on `window` or `block element` with a height set already

3. Three conditions for AJAX cal

   1. `container bottom - window bottom < certain value` ,i.e. the container don't have too much content to show
   2. not in the middle of loading
   3. still have content to load, i.e. not the end of all the content

4. code 

5. ```js
   var isLoading = false;
   var isEnd = false;
   var triggerDistance = 200;
   
   function fetchData() {
   
     var distance = container.getBoundingClientRect().bottom - window.innerHeight;
     if ( !isLoading && !isEnd && distance < triggerDistance ) {
   
       isLoading = true;
   
       fetch(path).then(res => {
         isLoading = false;
         // if not end, will short-circut latter statement
         // if reach end, will still check the latter one, which will set isEnd to true
         res.data.length === 0 && isEnd = true; 
         doSomething(res.data);
       });
   
     }
   
   }
   window.addEventListener('scroll', fetchData);
   ```



## The rendering order: CSS, JS, HTML

1. credit to:
   - https://bitsofco.de/async-vs-defer/
   - https://www.cnblogs.com/yingsong/p/6170780.html
2. General order
   1. JS in `<head>` will block downloading of following resource
   2. Browser downloads and parses HTML to DOM
   3. when encounter `<script>`, 
      - if not external, **<u>render engine</u>** hand control power to **<u>js engine</u>**
      - if is external, render engine <u>keep rendering</u>, while downloads external



## CSS: profile & edit image link

1. Implement the requirement

2. consider accessibility


```html
// 1. Mouse is not over the picture
+-------------+
|             |
|             |
|   picture   |
|             |
|             |
+-------------+

// 2. Mouse is anywhere over picture
+--------+----+
|        |edit| <--- links to /edit.php
|        +----+      icon url: /edit_icon.png
|   picture   |
|             | <--- links to /profile.php
|             |      picture url: /profile_pic.png 
+-------------+

<html>
  <head>
    <style>
      .imgWrapper{
        width: 100px;
        height: 100px;
        border: 1px solid red;
        position: relative;
      }
      
      #image1{
        width: 100%;
        height: 100%;
      }
      
      #image2{
        position: absolute;
        top: 0;
        right:0;
        width: 50px;
        height: 50px;
        display: none;
      }
      
      .imgWrapper:hover #image2{
        display: block;
      }
      
    </style>
  </head>
  <body>

    
      <div class="imgWrapper">
        <a href="https://stackoverflow.com/questions/1827965/is-putting-a-div-inside-an-anchor-ever-correct" target="_blank">
          <img id="image1" src="https://www.w3schools.com/html/pic_trulli.jpg" />
        </a>
        
        <a href="https://developer.mozilla.org/en-US/docs/Learn/Accessibility" target="_blank">
          <img id="image2" src="https://upload.wikimedia.org/wikipedia/commons/thumb/4/40/UniversalEditButton3.png/220px-UniversalEditButton3.png" />
        </a>
      </div>
    <script>
    </script>
  </body>
</html>

```



## Flex Layout

1. flex container:

   1. **<u>NOTE：</u>** `float` `clear` `vertical-align` for the items inside flex boxwill become *invalid*

   2. `display: inline-flex` vs `display: flex`:

      - only apply to flex container, to make it display as `inline` or `block`
      - won't affect flex items inside

   3. `flex-direction`

   4. `flex-wrap`: break new line or not when exceeding it container

   5. `flex-flow` is combinatin of first 2 items: 

      1. ```css
         flex-direction: row;
         flex-wrap: wrap;
         <==>
         flex-flow: row wrap;
         ```

   6. `align-items: center;`:  where flex items sit on the **cross axis**

   7. `justify-content: space-around;`: where the flex items sit on the **main axis**

2. flex items:

   1. `flex: 1 1 20px`:  three attributes: `flex-grow`   `flex-shrink`    `flex-basis`
   2. `order: 3`: like the smaller will be put in front
   3. `align-self`: override its container's `align-items` layout, update position of itself

## Grid Layout

1. Grid Container

   - **<u>display</u>**: `display: grid` vs `display: inline-grid`: only apply on grid container
   - **<u>gap</u>**: `grid-row-gap` `grid-column-gap` `grid-gap`: set the gap between items
   - **<u>col num/width</u>**: `grid-template-columns`: define num of columns and their width
   - **<u>row num/width:</u>**`grid-template-rows`: define num of rows and their width
   - **<u>horizontally align:</u>** `justify-content`: center, start, end, space-evenly, space-around, space-between
   - **<u>vertically align:</u>** `align-content`: center, start, end, space-evenly, space-around, space-between
   - 

2. Grid Items 

   1. <u>**line**</u>: `grid-row-start` `grid-column-end`: specify the start & end of span it covers 

   2. **<u>line shorthand 1</u>** : `grid-row ` `grid-column`

      -  `grid-row:1 / 3`:  `range: [line1  line3]`
      -  `grid-column: 1/span 3`:  from line1, take 3 units,  `range: [line1  line4]`

   3. **<u>line shorthand 2</u>** : `grid-area` = `grid-row-start`+ `grid-column-start` + `grid-row-end` + `grid-column-end` 

      - `grid-area: 1 / 2 / 5 / 6`:  start on row-line 1 and column-line 2, and end on row-line 5 and column line 6:

      - `grid-area: 2 / 1 / span 2 / span 3`: 

      - **<u>NOTE:</u>** **can assign names to grid items**, cooperate with `grid-template-areas`. Used to assign space to the named item

        ```css
        https://www.w3schools.com/css/tryit.asp?filename=trycss_grid_grid-area_named1
        
        .item1 { grid-area: header; }
        .item2 { grid-area: menu; }
        .item3 { grid-area: main; }
        .item4 { grid-area: right; }
        .item5 { grid-area: footer; }
        
        .grid-container {
          grid-template-areas:
            'header header header header header header'
            'menu main main main right right'
            'menu footer footer footer footer footer';
        } 
        ```

      - order: specified by `starting line#` and `ending line#`

      - ```css
        .item1 { grid-area: 1 / 3 / 2 / 4; }
        .item2 { grid-area: 2 / 3 / 3 / 4; }
        .item3 { grid-area: 1 / 1 / 2 / 2; }
        .item4 { grid-area: 1 / 2 / 2 / 3; }
        .item5 { grid-area: 2 / 1 / 3 / 2; }
        .item6 { grid-area: 2 / 2 / 3 / 3; }
        ```


## CSS:  

1. `float` &  `position` & `box model`
    - `float`
      - when container box has no height, and  need to cover all it floating children: `clear: both` & `overflow: hidden`

2. `box-sizing`=>`content-box` / `border-box`
    - `box-sizing: content-box;` actual width= `width` + `padding` + `border`
       The [`width`](https://developer.mozilla.org/en-US/docs/Web/CSS/width)and [`height`](https://developer.mozilla.org/en-US/docs/Web/CSS/height) properties include the content, but does not include the padding, border, or margin.
    - `box-sizing: border-box;`  actual width = `width` = `content` + `padding` + `border`
      The [`width`](https://developer.mozilla.org/en-US/docs/Web/CSS/width) and [`height`](https://developer.mozilla.org/en-US/docs/Web/CSS/height) properties include the content, padding, and border, but do not include the margin

3. `cursor: pointer` cursor acts like `<a>` when hover

4. `px` vs `pt` vs `rem` vs `em`
   https://kyleschaeffer.com/development/css-font-size-em-vs-px-vs-pt-vs/

   - `pt`: physical measurement, fixed when media once produced
   - `px`: calculated relative to `pt`; can be adjusted
   - `rem`: related to root font-size (default: 16px)
   - `em`: related to its parents font-size (default: 16px)

5. `resize`: indicate how this block element can be resized

   - <u>horizontal</u>: can only resized horizontally
   - <u>vertical</u>: can only resized vertically
   - <u>both</u>: can be resized on both directions
   - <u>none</u>: none resizable
   - <u>initial</u>:  == none resizable

6. `width` & `height`

   - make  the aspect-ratio stay the same: **<u>adjust one, set other one `auto`</u>**

     - tweak image while keep aspect-ratio the same

     - some time will overflow: can add `overflow: hidden` to its parent

       - ```css
         div{
             width: 100px;
             height: 100px;
         }
         .img{
             width: 100%;
             height: auto;
         }
         ```

7. `white-space:` ` nowrap`, `pre`, `normal`

   - how to handle **<u>text's space</u>** inside a block to be displayed
   - ` nowrap`: ignore all white-space in one line, like in `<span>`
   - `normal`: make text content displayed against the block, change line based on space
   - `pre`: take in all kind of space (include new line) displayed as they specified in html tag

8. `overflow`: deal with inner box exceeding outer box: `hidden` or `scroll`

9. `text-overflow: `  deal with too many words inside box, **<u>clip</u>** or **<u>ellipsis</u>**

   - `clip(default)`
   - `ellipsis(省略号)`

10. `word-break: ` deal with long word, which may exceeds width of its container

   1. `normal`
   2. `break-word;`: the best clipping way to show the word

11. `max-width` vs `width`

    - `width`: set the tentative value`

    - `max-width` set the rule, to restrict the value of `width`

    - some time can change the aspect-ratio

      - ```
        .image{
        	width: 100%;
            height: 100%;
         }
        ```



## CSS: `block` vs `inline` vs `inline-block`

credit to: http://dustwell.com/div-span-inline-block.html

1. `<div>`
   - A "block-level element"
   - can contain <u>all</u> other elements!
   - can only be inside other block-level elements
   - defines a rectangular region on the page
   - **<u>tries to be as wide as possible</u>**
   - begins on a "new line", and has an "carriage return" at the end, like a `<p>`
2. `<span>`
   - An "inline element"
   - cannot contain block-level elements!!
   - can be inside <u>any</u> other element
   - <u>**tries to be as narrow as possible**</u>
   - defines a "snake" on the page
   - doesn't create any new lines.
3. `block` vs `inline` vs `inline-block`
   1. `block`: 
      - has width, height, margin, padding
      - force a new line **after the block**
   2. `inline`: 
      - no width, no height; only respect left/right margin, left/right padding 
      - allow other elements to **sit on the same line**
   3. `inline-block`: 
      - has width, height, margin, padding
      - allow other elements to **sit in a line**, that's why called `inline`

## CSS: how to center an element?

1. https://css-tricks.com/centering-css-complete-guide/
2. Horizontally
   - center **inline element** inside **block element**:
     - `text-align: center`
   - center **block element** inside another **block element**:
     - <u>outer block with specified width</u>: `margin: 0 auto`
     - <u>outer block with no width</u>: inner width will spread its width to outer, so no need for `margin: 0 auto`
   - center **multiple block element in a row** inside another **block element**:
     - `display: inline-block`
     - `display: flex`
   - center **multiple block element in different rows** inside another **block element**:
     - `margin: 0 auto`
3. Vertically for <u>inline elements</u>
   - center **<u>single inline element</u>** inside **block element**:
     - `height`==`line-height`  
     - `padding-top == padding-bottom`
   - center **<u>multiple inline elements</u>** inside **block element**:
     - `display: table`& `display: table-cell`+ `vertial-align: middle` : 
     - `display: flex` & `align-items: center`
     - ghost element: 
       - full-height pseudo element is placed inside the container and the text is vertically aligned with that (both are `inline-block`)
       - **<u>trick</u>**: contain multiple lines into a `inline-block` to be a whole inline element
       - adjacent with another `inline-block` with `height=100%`
       - `inline-block` elements will align on <u>baseline</u>, if `vertical-align` doesn't specify any other value
       - if `vertical-align` specify `middle`: browser will try to align all inline boxes in middle, rather than baseline
       - <u>***NOTE:***</u> `vertical-align`  **specify how will itself to be aligned vertically**
4. Vertically for <u>block elements</u>
   - if `height` of inner block is known: use `position: abosulute` & `top` & `margin:0.5*height`
   - if `height` of inner block is not known: use `position: abosulute ` `top` & `transform: translateY(-50%)` 
   - Again: `display: flex`
5. Both Horizontally & Vertically
   - if `height` of inner block is known: use `position: abosulute` & `top` & `margin:0.5*height`
   - if `height` of inner block is not known: use `position: abosulute ` `top` & `transform: translateY(-50%)` 
   - Again: `display: flex`
   - `display: grid`



## DOM structure

1. difference between `HTMLCollection` & `NodeList`

   - `HTMLCollection`: a collection of **element nodes**

   - `NodeList`: a collection of **all types of nodes: element node/ text node/ comment node**
   - convert them to array, to extract info: `Array.prototype.slice.call(NodeList)`

2. everything in an HTML document is a <u>node</u>:

   - The entire document is a ***document* node**
   - Every HTML element is an ***element* node**
   - The text inside HTML elements are ***text* nodes**
   - All comments are ***comment* nodes**

3. navigate across the DOM tree using attibutes: [<u>*element node, text node, comment node*</u>]

   1. parentNode
   2. childNodes [*nodenumber*]
   3. firstChild
   4. lastChild
   5. nextSibling
   6. previousSibling

4. <u>**A common error in DOM processing is to expect an element node to contain text.**</u>

   1. `<title id="demo">DOM Tutorial</title>`

   2. The element node `<title>` (in the example above) does **not** contain text.

   3. It contains a **text node** with the value "DOM Tutorial".

   4. The value of the text node can be accessed by the node's **innerHTML** property:

   5. ```js
      var myTitle = document.getElementById("demo").innerHTML;
      
      // or
      var myTitle = document.getElementById("demo").firstChild.nodeValue;
      
      // or
      var myTitle = document.getElementById("demo").childNodes[0].nodeValue;
      ```


## DOM API: 

1. **get style & set style of an element**
   - set style

     - directly use it as attribute `elements.style.fontSize = '10px'`
     - NOTE: the style name must be **camelCase**

   - get style: 

     - if style is set using `style` or `js`, directly get the style

     - if style is set using `css` without `element.style` keyword, we will use `getComputedStyle()`

     - `document.defaultView.getComputedStyle(ele,null).getPropertyValue("font-size")`

     - ```html
       <!DOCTYPE html>
       <html lang="en">
       <head>
       
       <script>
       function cStyles() {
         var RefDiv = document.getElementById("d1");
         var txtHeight = document.getElementById("t1");
         var h_style = document.defaultView.getComputedStyle(RefDiv, null).getPropertyValue("height");
       
         txtHeight.value = h_style;
       }
       </script>
       <style>
       #d1 {
         margin-left: 10px;
         background-color: rgb(173, 216, 230); 
         height: 20px;
         max-width: 20px;
       }
       </style>
       </head>
       
       <body>
       
       <div id="d1">&nbsp;</div>
       <form action="">
         <p>
           <button type="button" onclick="cStyles();">getComputedStyle</button>
           height<input id="t1" type="text" value="1" />
         </p>
       </form>
       
       </body>
       </html>
       ```

2. **`console.trace()` can be used to trace the call stack of a certain function**

3. `addEventListener` & `removeEventListener`

   - ```js
     // Attach an event handler to the document
     document.addEventListener("mousemove", myFunction);
     
     // Remove the event handler from the document
     document.removeEventListener("mousemove", myFunction);
     ```

4. `createAttribute()` & `setAttributeNode()`

   - ```js
     function myFunction() {
         var h1 = document.getElementsByTagName("H1")[0];
         var att = document.createAttribute("class");
         att.value = "democlass";
         h1.setAttributeNode(att);
     }
     ```

5. `createElement()`  & `intersetBefore()` &  ` appendChild()`

   1. ```js
      var btn = document.createElement("BUTTON");        // Create a <button> element
      var t = document.createTextNode("CLICK ME");       // Create a text node
      btn.appendChild(t);                                // Append the text to <button>
      document.body.appendChild(btn);                    // Append <button> to <body>
      ```

6. `querySelector`  & `querySelectorAll()`

   1. https://www.w3schools.com/cssref/css_selectors.asp

   2.  `querySelector()` returns **the first element** that matches a specified *CSS selector(s)* 

      1. ```js
         // 1. select first p element whose parent is a div
         function myFunction() {
             var x = document.querySelector("div > p");
             x.style.backgroundColor = "red";
         }
         
         // 2. select all a element has target attribute
         function myFunction() {
             document.querySelector("a[target]").style.border = "10px solid red";
         }
         ```

   3. `querySelectorAll()` returns **all elements** in the document that matches a specified CSS selector(s), as a static **<u>NodeList object</u>**.

      1. ```js
         function myFunction() {
             var x = document.querySelectorAll("a[target]");
             var i;
             for (i = 0; i < x.length; i++) {
                 x[i].style.border = "10px solid red";
             }
         }
         ```

7. **`element.childNodes` vs `element.children`**

   - `element.childNodes` :  returns a `NodeList object`,  including <u>element nodes</u>, <u>text nodes,</u> and <u>comment nodes</u>
   - `element.children`: return a `HTMLCollection object`
   - difference: 
     - **childNodes**: contain all nodes, including <u>text nodes</u> and <u>comment nodes</u> 
     - **children**: only contain <u>element nodes.</u>

8. **`element.firstChild` vs `element.firstElementChild`**

   - difference: 
     - **firstChild** returns the first child node as an <u>element node</u>, a <u>text node</u> or a <u>comment node</u> (depending on which one's first), 
     - **firstElementChild** returns the first child node as an <u>element node</u> (ignores text and comment nodes).

9. **`element.parentNode` vs `element.parentElement`**

   - **parentElement: ** returns *null* if the parent node is not an element node

   - **parentNode: ** is probably the most popular

   - ```js
     document.body.parentNode; // Returns the <html> element
     document.body.parentElement; // Returns the <html> element
     
     document.documentElement.parentNode; // Returns the Document node
     document.documentElement.parentElement; // Returns null (<html> does not have a parent ELEMENT node)
     ```

10. **`element.previousSibling` vs `element.previousElementSibling`**

    - **previousSibling: ** returns the previous sibling node as an element node, a text node or a comment node
    - **previousElementSibling: ** returns the previous sibling node as an element node (ignores text and comment nodes).


## DOM events

1. **<u>[Trick]</u>: ** bind data on element as `attribute`, then use `e.target.getAttribute("data-info")`

2. common types: 

   - `onload`

   - `oninput`

   - `onchange`
   - `onfocus`
   - `onmouseover` & `onmouseout`
   - **`onclick` & `onmousedown` & `onmouseup`**
   - **`onkeypress` & `onkeydown` & `onkeyup`**
   - `onresize`
   - `onscroll`

3. event propagation: *bubbling* & *capturing*

   - **Event propagation** is a way of defining the element <u>order</u> when an event occurs. 
   -  <u>***bubbling***</u>:  the inner most element's event is handled first and then the outer: the `<p>` element's click event is handled first, then the `<div>` element's click event.
   - ***<u>capturing</u>***:  the outer most element's event is handled first and then the inner: the `<div>` element's click event will be handled first, then the`<p>` element's click event.

## Array-like object under the hood

1. **All credit to**: http://2ality.com/2013/05/quirk-array-like-objects.html
2. An array-like object
   - only has: 
     - `index` to access to elements: `a[0]`
     - the property `length` that tells us how many elements the object has: `a.length`
   - does not have: array methods such as `push`, `forEach` and `indexOf`; **have to <u>borrow</u> these methods**
   - **<u>NOTE:</u>**
     -  `NodeList` has the `forEach` attribute
     - `HTMLCollection` doesn't have `forEach` attribute
   - <u>**"borrow" : **</u> that why we use:
     -  `[].slice(array-like)` 
     - `[].indexOf(array-like)`
     - `[].forEach(array-like)`
3. Two examples of array-like objects
   - result of the DOM method `document.getElementsByClassName()` (many DOM methods return array-like objects)
   - the special variable `arguments` . 
     - You can determine the number of arguments via `arguments.length`
     - you can access a single argument, e.g. read the first argument: `arguments[0]`
4. Array methods, however, have to be borrowed. You can do that, because most of those methods are generic

5.  <u>**Three ways**</u> to convert Array-like objects to Array.

   1. Array.prototype.slice.call(arguments)`

   2. `Array.from(arguments)`

   3. use `bind` to create a **new function** `slice()` function on array-like object, so we **<u>can reuse it</u>**

      ```js
      let = getIndexOfA = function(){ 
        let myIndexOf = [].indexOf.bind(arguments); 
        return myIndexOf("a") // NOTE: mustn't use arguments.myIndexOf()
      }
      
      console.log(  getIndexOfA("d", "d", "a","d")  ) // 2
      ```


## What can be put inside `<head>`?

https://github.com/xiaoyu2er/HEAD



## JS coding: mutate array according new indices

```js
Given an input array and another array that describes a new index for each element, mutate the input array so that each element ends up in their new index. 

// ---------------- in-place mutate array ------------
var arr =    ["a","b","c","d","e","f"];
var indices = [2,  3,  4,  0,  5, 1];


// Solution1: real in-place solution!
function inPlaceChange( arr, indices){
    if( arr.length <= 1 ) return arr

    let i = 0;
    let toPut = arr[0];
    let index = indices[0];
    while( i < arr.length ){
        let swap = arr[index]
        arr[index] = toPut
        toPut = swap
        index = indices[index]
        i++
    }
}
inPlaceChange(arr, indices)
console.log(arr)

//---------------------------------------------------

var arr =    ["a","b","c","d","e","f"];
var indices = [2,  3,  4,  0,  5, 1];
// Solution 2: not real in-place, cause map() use extra space
let arr = indices.map( (id, index)=>arr[indices.indexOf(index)] )
console.log("updated arr: ", arr)


```



## `<script async>` & `<script defer>`

1. normal `<script src="script.js" >`
   - The HTML parsing is paused for the script to be both **fetched** and **executed**
2. `<script async>`
   - The HTML parsing is <u>not</u> paused for the script to be **fetched**,
   - but still be paused for js to be **executed**

3. `<script defer>`
   - The HTML parsing is <u>not</u> paused for the script to be **fetched**,
   - oppositely, the **execution of js is paused** util the HTML finish parsing
   - same as `<script>` right before the `</body>` tag

## How to check data types in JS

https://webbjocke.com/javascript-check-data-types/



## write a promise

 Promise

```
<!DOCTYPE html>
<html>
<head>
<style>
#wrapper {
    margin-top: 10px;
    overflow: auto;
    width: 250px;
    display: inline-block;
    border: 1px solid red;
    overflow: hidden;
}


.listWrapper{
	border: 1px solid blue;
    height: 200px;
    width: 140px;
    overflow: auto;
    scroll: auto;
}

</style>
</head>
<body>


<div id="wrapper">
  <div>
      <input id="inputText" /> 
      <button>submit</button>
  </div>

</div>

<script>
	
    let data = new Array(45)
    data.fill("recommendation word" )
    
    
	const API = "https://www.demo.com/potentialList"
    
    // create new element below <input>, inside div wrapper
   
   
    function appendList(elementCollection, wrapper){
    	let oldlistWrapper = document.getElementsByClassName("listWrapper")[0];
    	wrapper.removeChild(oldlistWrapper)
        
    	let listWrapper = document.createElement("DIV")
    	listWrapper.setAttribute("class", "listWrapper")
        wrapper.appendChild(listWrapper)
    	let list = Array.prototype.slice.call(elementCollection)
        data.forEach( ele=>{
        	console.log("wrapper: ", wrapper)
            let itemNode = document.createElement("DIV")
            let textNode = document.createTextNode(ele)
            itemNode.appendChild(textNode)
            listWrapper.appendChild(itemNode)
        })
        
    }
    // make API call
    function makeAPICall(API, val){
    	let ajax = new XMLHttpRequest()
        ajax.open("GET", "https://www.w3schools.com/js/ajax_info.txt", true)
        ajax.onreadystatechange = function(){
        	if(this.readyState === 4 && this.status === 200 ){
            	console.log(this.responseText)
                let parser = new DOMParser()
                let a = parser.parseFromString(this.responseText, "text/html")
                console.log(a)
                let body = a.getElementsByTagName("BODY")[0]
                let wrapper = document.getElementById("wrapper")
                console.log(body.children)
                // append return list to <input>
                appendList(body.children, wrapper)
            }
        }
        ajax.send()
    	console.log(val)
    }
    
    
	// debounce 500ms to make API call
    let timeHandler = null
    function inputHandler(e){
    	clearTimeout(timeHandler)
        let val = e.target.value
    	timeHandler = setTimeout( function(){makeAPICall(API, val)} ,500 )
    }
    
	let input = document.getElementById("inputText")
    addEventListener("input", inputHandler)
</script>

</body>
</html>

```



## AJAX

1.  `XMLHttpRequest` is browser built-in object, used to pull data in background

2. **Methods**:

   - `open(method, url, async, user, psw)`: **specify** the request
     - <u>*method*</u>: the request type GET or POST
     - <u>*url*</u>: the file location
     - <u>*async*</u>: true (asynchronous) or false (synchronous)
     - <u>*user*</u>: optional user name
     - <u>*psw*</u>: optional password
   - `setRequestHeader()`: Adds a label/value pair to the **header** to be sent
   - `send(str)`: Sends the request to the server (`GET` or `POST`)
     - if no `str` provided, send as `GET`
     - if `str` is provided, send as the parameters for `POST`

3. **properties:**

   - `onreadystatechange`: **function** used to be called when state change
   - `readyState`: Holds the status of the XMLHttpRequest.
     - 0: request not initialized 
     - 1: server connection established
     - 2: request received 
     - 3: processing request 
     - 4: request finished and response is ready
   - `status`: Returns the status-number of a request
     - 200: "OK"
     - 403: "Forbidden"
     - 404: "Not Found"
   - `responseText`: Returns the response data as a string
   - `responseXML`: Returns the response data as XML data

4. **steps**

   1. An event occurs in a web page (the page is loaded, a button is clicked)
   2. An XMLHttpRequest object is created by JavaScript
   3. The XMLHttpRequest object sends a request to a web server
   4. The server processes the request
   5. The server sends a response back to the web page
   6. The response is read by JavaScript
   7. Proper action (like page update) is performed by JavaScript

5. **Code**

   1. ```js
      /* --------------- 1. GET  --------------- */
      function loadDoc() {
        var xhttp = new XMLHttpRequest();
        xhttp.onreadystatechange = function() {
          if (this.readyState == 4 && this.status == 200) {
            document.getElementById("demo").innerHTML = this.responseText;
          }
        };
        xhttp.open("GET", "demo_get.asp", true);
        xhttp.send();
      }
      
      /* --------------- 2. POST ---------------  */
      function loadDoc() {
        var xhttp = new XMLHttpRequest();
        xhttp.onreadystatechange = function() {
          if (this.readyState === 4 && this.status === 200) {
            document.getElementById("demo").innerHTML = this.responseText;
          }
        };
        xhttp.open("POST", "demo_post2.asp", true);
        xhttp.setRequestHeader("Content-type", "application/x-www-form-urlencoded");
        xhttp.send("fname=Henry&lname=Ford");
      }
      ```



## Check if `undeclared`, `undefined` or `null`

```js
// 1. `undeclared` check
try{
  undeclaredVar
}
catch(e) {
  if(e.name === 'ReferenceError') {
    console.log('var is undeclared')
  }
}

// 2. `undefined` check
var undefinedVar

if (typeof undefinedVar === 'undefined') {
   console.log('var is undefined')
}

// 3. `null` check
var nullVar = null

if (nullVar === null) {
   console.log('var is null')
}

```


