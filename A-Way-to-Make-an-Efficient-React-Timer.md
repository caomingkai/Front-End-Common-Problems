---
title: A Way to Make an Efficient React Timer
comments: true
date: 2018-10-05 15:38:28
type: categories
categories: Full stack
tags: 
- ReactJS
- efficient state management
---



## I. The Problem

It is very common we need to implement a **<u>Timer</u>** in our daily projects. Of course, there are several ways to make one. 

Most online resource only introduces how to make a Timer as the only one component itself, while in reality we want our Timer to be an sub-component of some parent component

We don't want to put the `time_elapsed` state in the `state` of its parent component. That way, every one seconds, the parent component will rerender over and over, even though all other state field stay the same.



## II. We Want Efficiency

Here, the basic idea is: 

1. we put the `time_elapsed`  state in a single component with no other state field in it.
   - start `hanler = setInterval()` to update the `time_elapsed` , in `componentDidMount()` stage
   - pass out the `handler` to its parent component
2. we also want the parent component can `startTimer` and `stopTimer`
   - we need to get the handler of the `setInterval()`, in order to stop it in outside
   - BUT, how to get the handler???  ==> we pass in an object as `props` to timer component

## III. Implementation

1. ```js
   <html>
     <head>
       <meta charset="UTF-8" />
       <title>Hello World</title>
       <script src="https://cdnjs.cloudflare.com/ajax/libs/react/16.2.0/umd/react.production.min.js"></script>
       <script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/16.2.0/umd/react-dom.production.min.js"></script>
       <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/6.26.0/babel.min.js"></script>
     </head>
     <body>
       <div id="root"></div>
       <script type="text/jsx">
   
         class Timer extends React.Component{
           constructor(props){
             super(props)
             this.state={ currTime: 0 }
           }
           
           componentDidMount (){
             this.props.timerObj.handler = setInterval(()=>{
               this.setState({ currTime: this.state.currTime+1000 })
             }, 1000)
           }
           
           render(){
             console.log("Inside Timer Component: ", this.state)
             const timePassed = this.state.currTime / 1000
             return <span>{timePassed}</span>
           }
         }
                       
   
         class App extends React.Component{
           constructor(props){
             super(props)
             this.state = { showTimer: false }
             this.timerObj = { handler: null }
           }
           
           startTimer = ()=>{
             this.setState({ showTimer: true })
           }
           
           stopTimer = ()=>{
             clearInterval(this.timerObj.handler)
             this.setState({showTimer: false})
           }
           
   
           render(){
             console.log("App Component: ", this.state)
             const TimerPlaceholder = this.state.showTimer ? <Timer timerObj={this.timerObj} /> : null
             return (
               <div>
                 <h1> React Inner Timer Component</h1>
                 <button onClick={this.startTimer}> startTimer </button>
                 <br /><br />
                 
                 Time Passed: {TimerPlaceholder }
                 
                 <br /><br />
                 <button onClick={this.stopTimer}> stopTimer </button>
               </div>
             )
           }
         }
   
               
         ReactDOM.render(
           <App />,
           document.getElementById('root')
         );
       </script>
     </body>
   </html>
   ```
