---
title: Negative Margin in Layout
comments: true
date: 2018-10-30 17:41:46
type: categories
categories: Full stack
tags:
- margin
- layout
- CSS
---



## Credits to:

1. http://icewind-blog.com/2014/05/27/css-negative-margin/
2. https://blog.csdn.net/zhoulei1995/article/details/80161240
3. https://github.com/rccoder/blog/issues/6
4. https://www.jianshu.com/p/487d131537b4
5. https://alistapart.com/article/holygrail

## Document flow

All block elements like boxes (block/inline) flowing on water; The water push them to left, and to top

1. inline elements are only push to left; When there is no space in a row, it will automatically switch to next line **<u>without new line generated</u>**
2. `float` will make element escape from normal document flow
3. <u>inline element</u> `float` will make it to be <u>block element</u>
4. `position: absolute` & `position: fixed` will also make escape

## Margin Used in Domcument FLow

- **`margin`** determine the <u>hidden outer border</u> (not `border`) for all elements. 

- All elements have a hidden outer border used by browser to arrange elements.

## Positive Margin

1. block element:  will make margins on 4 directions have a gap between its <u>parents</u> or its <u>siblings</u>
2. inline element:  will only respect `left-margin` and `right-margin`

## Negative Margin

1. <u>normal document flow</u>

   - **positive**: margin will separate boxes by a gap
   - **negative**:
     - `margin-left` 和 `margin-top`：affect itself, move to its left/top by value
     - `margin-right` 和 `margin-bottom`：affect adjacent elements, pull them to its right/bottom by value

2. <u>escape document flow</u>:  **absolute position**

   - `margin-left` 和 `margin-top`：can affect itself, move to its left/top by value
   - `margin-right` 和 `margin-bottom`：won't affect adjacent, since it out of normal document flow

3. <u>escape document flow</u>:  **float**

   determined by `float` direction (like normal doc flow)

   - when `float: left`: 

     - `margin-left: -5px` will move itself to left by 5px
     - `margin-right: -5px` will move adjacent element to its right by 5px
     -  `margin-top`：can affect itself, move to its top by value
     - `margin-bottom`：will move adjacent element to its bottom by value

   - when `float: right`: 

     - `margin-right: -5px` will move adjacent element to its right by 5px
     - `margin-left: -5px` will move itself to left by 5px
     -  `margin-top`：can affect itself, move to its top by value
     - `margin-bottom`：will move adjacent element to its bottom by value

   - **<u>[NOTE]：</u>** `margin-left:-100%` in `float` document flow

     - since `float` flow is like inline elements, no new line between lines, even though they are in different lines

     - For example below: even though at first `left` is on bottom of `main`, does not mean it break off from `main`, it still adjacent to `main`

     - `margin-left: -100%; ` moves itself to its left by 100% width of its parent

     -  Example

       - ```html
         <div class="main" >  main </div>
         <div class="left" >  left </div>
         <div class="right"> right </div>
         
         <style>
             .main{
                 float: left;
                 width: 100%;   // occupy whole width
             }
             .left{
                 float: left;
                 width: 50px;   // its own width
                 margin-left: -100%; 
                 // even though at first it is on bottom of
                 // main, does not mean break off from main
                 // when '-100%' negative margin,
                 // it move itself to its left by 
                 // 100% width of parent
             }
             .right{
                 float: left;
                 width: 50px;
                 margin-left: -50px;
             }
         </style>
         ```


## Use case of negative margin

1. **Layout Difference: <u>Holy Grail</u> vs <u>Double wings**</u>

   - <u>Holy Grail</u>: background won't affect two sides, even it's higher than two sides
   - <u>Double wings</u>: background affects parts below two sides, when it's higher than two sides

2. **Layout of Holy Grail (圣杯布局) ** 

   - https://blog.csdn.net/zhoulei1995/article/details/80161240

   - container set aside `left/right padding` padding for left/right section

   - move left/right section onto main using `negative margin`

   - move left/right section to padding place using `position:relative`

     - ```html
       <html>
           <head>
               <style>
               	.container {
                       padding: 0 300px 0 200px;
                   }
                   .content {
                       float: left;
                       width: 100%;
                       background: #f00;
                   }
                   .left {
                       position: relative;
                       left: -200px;
                       float: left;
                       width: 200px;
                       margin-left: -100%;
                       background: #0f0;
                   }
                   .right {
                       position: relative;
                       right: -300px;
                       float: left;
                       width: 300px;
                       margin-left: -300px;
                       background: #00f;
                   }
               </style>
           </head>
           
           <body>
               <div class="container">
                   <div class="content">
                       中间内容区域 中间内容区域 中间内容区域 
                       中间内容区域 中间内容区域 中间内容区域 
                       中间内容区域 中间内容区域 中间内容区域 
                   </div>
                   <div class="left">左侧边栏区域</div>
                   <div class="right">右侧边栏区域</div>
               </div>
           </body>
       </html>
       ```

3. **Double Wings layout (双飞翼布局)**

   - https://blog.csdn.net/zhoulei1995/article/details/80161240

   - move left/right section onto wrapper using `negative margin`

   - set `margin left/right` to main, to make it won't hidden by two sides

     - ```html
       <html>
           <head>
               <style>
               	.wrapper {
                       float: left;
                       width: 100%;
                       background: #f00;
                   }
                   .content {
                       margin-left: 200px;
                       margin-right: 300px;
                   }
                   .left {
                       float: left;
                       width: 200px;
                       margin-left: -100%;
                       background: #0f0;
                   }
                   .right {
                       float: left;
                       width: 300px;
                       margin-left: -300px;
                       background: #00f;
                   }
               </style>
           </head>
           
           <body>
               <div class="wrapper">
                   <div class="content">
                       中间内容区域 中间内容区域 中间内容区域 
                       中间内容区域 中间内容区域 中间内容区域 
                       中间内容区域 中间内容区域 中间内容区域			</div>
               </div>
               <div class="left">左侧边栏区域</div>
               <div class="right">右侧边栏区域</div>
           </body>
       </html>
       ```

4. Horizontal & Vertical Center

   - For `position: absolute`, even thought it out of the normal document flow, the negative left/top still affect itself, moving to its left/top by value

     - ```html
       <!DOCTYPE HTML>
       <html>
       <head>
           <meta charset="UTF-8">
           <title> Center </title>
           <style>
            body{ margin:0;padding:0;}
            #test{
               width:200px;
               height:200px;
               background:#F60;
               position:absolute;
               left:50%;
               top:50%;
               margin-left: -100px;
               margin-top: -100px;
           }
           </style>
       </head>
       <body>
        <div id="test"></div>
       </body>
       </html>
       ```