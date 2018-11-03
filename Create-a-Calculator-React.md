---
title: 'Create a Calculator - [React]'
comments: true
date: 2018-10-16 09:51:15
type: categories
categories: Full stack
tags:
- React
---



## Problem

Build a web calculator, that supports the basic operations of `add`, `remove`, `multiply` and `divide`.

In addition, support resetting the current state, as well as a decimal point button.

## Implementation

1. HTML

   1. ```html
      <div class="center-child">
        <div id="app" ></div>
       </div>
      ```

2. CSS

   1. ```css
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
      ```

3. JS

   1. ```js
      
      
      class Calc extends React.Component{
        constructor(){
          super()
          this.state = {
            display: ""
          }
        }
        
        handleClick(e){
          let val = e.target.textContent
          let display = this.state.display
          if(this.isNum(val)){
            this.handleNum(val, display);
          }else if(this.isDot(val)){
            this.handleDot(val, display);      
          }else if(this.isOperator(val)){
            this.handleOperator(val, display)
          }else if(this.isEqual(val)){
            this.handleCalc(display)
          }else if(this.isClear(val)){
            this.handleClear(display)
          }else{
            return 
          }
        }
        
        
              handleClear(display){ 
                  this.setState({display: ""})
              }
              
              handleNum(v, display){
                 this.setState({display: this.state.display+v})
              }
      
              handleOperator(v, display){
                let oldText = display
                if( oldText === "" ) return 
                if( this.isOperator(oldText[oldText.length-1]) )
                    oldText=oldText.substr(0,oldText.length-1)
                this.setState({display: oldText+v})
              }
      
              handleDot(v, display){
                let oldText = display
                if( this.isDot(oldText[oldText.length-1]) ) return
                display.textContent += v
                this.setState({display: this.state.display+v})
              }
      
              
              handleCalc(display){
                let oldText = display
                if( !this.isNum(oldText[oldText.length-1]) ) {
                  this.setState({display:0})
                  return
                }
                this.setState({display: this.calc(oldText)})
              }
      		
            calc( text ){
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
              
          isNum(v){  return v >= "0" && v <= "9" }
          isClear(v){ return v === 'C' }
          isEqual(v){ return v === '=' }
          isDot(v){ return v === '.' }
      	isOperator(v){ return v === '+' ||  v === '-' ||  v === '÷' ||  v === '×' }
        
        
        render(){
          return (
            <div className="wrapper"> 
            <div className="display-panel" id="display">{this.state.display}
            </div>
            <div className="ctl-panel" id="panel" onClick={(e)=>{this.handleClick(e)}}>
              <div className='left-part'>
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
              <div className='right-part'>
                <div><button>&divide;</button></div>
                <div><button id="equal">=</button></div>
              </div>
            </div>
          </div>
          )
        }
        
      }
      
      
      ReactDOM.render(<Calc />, document.getElementById('app'));
      ```


