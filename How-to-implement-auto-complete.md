---
title: How to implement auto-complete?
comments: true
date: 2018-10-08 20:13:10
type: categories
categories: Full stack
tags:
- auto-complete
- event delegation
- debounce
- cache
---

## I. Logic

1. For each `input`, wait 500ms to make API call, to load 40 items
2. `scroll` down close to bottom ( less than 20px left ), make API call to load another 40 items
3. could add local `cache`, to speed up response

## II. code

1. ```html
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
   	
       let body = document.getElementsByTagName("BODY")[0]
       let data = Array.from({length: 40}, () => "recommdation"+Math.floor(Math.random() * 40));
       
   	const API = "https://www.demo.com/potentialList"
       let isFetching = false  // used for scroll
       let isEnd = false		// used for scroll
       
       
       // create new element below <input> inside div wrapper. For both 'input' & 'scroll'
       function appendList(elementCollection, wrapper, isScroll){
           if(isScroll){
           	let listWrapper = document.getElementsByClassName("listWrapper")[0]
               data.forEach( ele=>{
                   let itemNode = document.createElement("DIV")
                   let textNode = document.createTextNode(ele)
                   itemNode.appendChild(textNode)
                   listWrapper.appendChild(itemNode)
               })
               return
       	}
           
           let oldlistWrapper = document.getElementsByClassName("listWrapper")
           if(oldlistWrapper.length > 0){
             wrapper.removeChild(oldlistWrapper[0])
           }
       	let listWrapper = document.createElement("DIV")
       	listWrapper.setAttribute("class", "listWrapper")
       	let list = Array.prototype.slice.call(elementCollection)
           data.forEach( ele=>{
               let itemNode = document.createElement("DIV")
               let textNode = document.createTextNode(ele)
               itemNode.appendChild(textNode)
               listWrapper.appendChild(itemNode)
           })
           listWrapper.addEventListener("click", putToInput)
           listWrapper.addEventListener("scroll", scrollHandler)
           wrapper.appendChild(listWrapper)
           
       }
       
       
       // onclick callback for items provided
       function putToInput(e){
       	let input = document.getElementById("inputText")
       	let wrapper = document.getElementById("wrapper")
       	let listWrapper = document.getElementsByClassName("listWrapper")[0]
           input.value = e.target.innerHTML
           console.log(e.target)
           wrapper.removeChild(listWrapper)
       }
       
       
       // make API call
       function makeAPICall(API, val, isScroll){
         console.log("val: ", val)
         // 1. first check local cache
           
         // 2. hit fail, call API
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
             if(val.localeCompare("scroll made request")===0){
             	isFetching = false
             }
             appendList(data, wrapper, isScroll)
           }
         }
         ajax.send()
       }
       
       // scroll handler for listWrapper
       function scrollHandler(e){
       	let listWrapper = e.target
           let boxHeight = listWrapper.clientHeight
           let scrollTop = listWrapper.scrollTop
           let scrollHeight = listWrapper.scrollHeight
           
           let remainder = scrollHeight - scrollTop
           let diff = remainder - boxHeight
           console.log("box height: ", boxHeight)
           console.log("scrollTop: ", scrollTop)
           console.log("scrollHeight: ", scrollHeight)
           console.log("diff: ", diff)
           
           if( diff < 20 && !isFetching && !isEnd ){
           	isFetching = true
           	makeAPICall(API, "scroll made request", true)
           }
       }
       
   	// debounce 500ms to make API call
       let timeHandler = null
       function inputHandler(e){
       	clearTimeout(timeHandler)
           let val = e.target.value
       	timeHandler = setTimeout( function(){makeAPICall(API, "keypress made request", false)} ,500 )
       }
       
   	let input = document.getElementById("inputText")
       addEventListener("input", inputHandler)
   </script>
   
   </body>
   </html>
   
   ```
