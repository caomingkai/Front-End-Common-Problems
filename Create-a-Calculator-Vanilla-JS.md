---
title: 'Create a Calculator - [Vanilla JS]'
comments: true
date: 2018-10-15 16:38:37
type: categories
categories: Full stack
tags:
- Vanilla JS
- event propagate
---



## Problem

Build a web calculator, that supports the basic operations of `add`, `remove`, `multiply` and `divide`.

In addition, support resetting the current state, as well as a decimal point button.



## Implementation

```html

<html>
  <head>
  	<style>
       .wrapper{
          width: 400px;
          height: 400px;
          margin: 15px auto;
          border: 1px solid black;
        }
        .display-panel{
          margin: 10px auto;
          width: 95%;
          height: 60px;
          line-height: 60px;
          font-size: 30px;
          border: 1px solid black;
          text-align: right;
          overflow-y: hidden;
          overflow-x: scroll;
          white-space: nowrap;
        }

        .ctl-panel{
          margin: 5px auto;
          width: 95%;
          height: 300px;
          border: 1px solid black;
          font-size: 30px;
          display: flex;
          flex-flow: row wrap;
          justify-content: space-around;
        }

        .left-part{
          border: 1px solid red;
          flex: 1 1 70%;
          display: grid;
          justify-content: space-around;
          align-content: space-around;
          grid-template-columns: auto auto auto;
          grid-template-rows: auto auto auto auto;
        }


        .right-part{
          flex: 1 1 20%;
          border: 1px solid red;
          display: grid;
          justify-content: space-around;
          align-content: space-around;
          grid-template-columns: auto;
          grid-template-rows: 30% 60%;
        }

        #equal{
          height: 100%;
        }

        button{
          width: 50px;
          height: 50px;
        }
    </style>
  </head>
  <body>
    <div class="wrapper"> 
      <div class="display-panel" id="display">
        display placeholdersdsfsdfsdfsdffsdfsdffsdf
      </div>
      <div class="ctl-panel" id="panel">
        <div class='left-part'>
            <div><button>+</button></div>
            <div><button>-</button></div>
            <div><button>&times;</button></div>
            <div><button>7</button></div>
            <div><button>8</button></div>
            <div><button>9</button></div>
            <div><button>4</button></div>
            <div><button>5</button></div>
            <div><button>6</button></div>
            <div><button>1</button></div>
            <div><button>2</button></div>
            <div><button>3</button></div>
            <div><button>0</button></div>
            <div><button>.</button></div>
            <div><button>C</button></div>
        </div>
        <div class='right-part'>
          <div><button>&divide;</button></div>
          <div><button id="equal">&equals;</button></div>
        </div>
      </div>
    </div>
      
    <script>
        function handleInput(e){
          let val = e.target.textContent
          let display = document.getElementById("display")
          if(isNum(val)){
             handleNum(val, display);
          }else if(isDot(val)){
             handleDot(val, display);      
          }else if(isOperator(val)){
             handleOperator(val, display)
          }else if(isEqual(val)){
             handleCalc(display)
          }else if(isClear(val)){
             handleClear(display)
          }else{
            return 
          }
        }

        function handleClear(display){ 
            display.textContent = "" 
        }
        
        function handleNum(v, display){
          let oldText = display.textContent
          display.textContent += v
        }

        function handleOperator(v, display){
          let oldText = display.textContent
          if( isOperator(oldText[oldText.length-1]) )
              oldText=oldText.substr(0,oldText.length-1)
          display.textContent = oldText+v
        }

        function handleDot(v, display){
          let oldText = display.textContent
          if( isDot(oldText[oldText.length-1]) ) return
          display.textContent += v
        }

        
        function handleCalc(display){
          let oldText = display.textContent
          if( !isNum(oldText[oldText.length-1]) ) {
            display.textContent = 0
            return
          }
          display.textContent = calc(oldText)
        }
		
        function calc( text ){
          let stack = []
          let numbers = text.split(/\+|\-|\×|\÷/)
          let operators =  text.replace(/[0-9]|\./g, "").split("")
          console.log(numbers)
          console.log(operators)

          stack.push(new Number(numbers[0]) )
          for( let i = 0; i < operators.length; i++ ){
            let num = new Number( numbers[i+1] )
            if(operators[i] === "+"){
              stack.push(num)
            }else if(operators[i] === "-"){
              stack.push(-num)
            }else if(operators[i] === "×"){
              let top = stack.pop()
              stack.push(top*num)
            }else if(operators[i] === "÷"){
              let top = stack.pop()
              stack.push(top/num)
            }
          }
          console.log(stack)
          return stack.reduce((item,accum)=>{
               return accum + item
          },0)
        }
        
        
        function isNum(v){  return v >= "0" && v <= "9" }
        function isClear(v){ return v === 'C' }
        function isEqual(v){ return v === '=' }
        function isDot(v){ return v === '.' }
		function isOperator(v){
          return v === '+' ||  v === '-' ||  v === '÷' ||  v === '×'
        }
        
        document.getElementById("panel").addEventListener( "click", handleInput )
    </script>
  </body>
</html>
```

