---
title: How to Make Animation using JS
comments: true
date: 2018-08-11 22:56:29
type: categories
categories: Full stack
tags:
- animation
- vanilla JS

---



## I. problem

We want a function to move an `element` to its right by `distance` in specified `time`

- API: `function moveToRight(ele, dis, time) {}`

## II. Implementation

1. must not be `position: static`: using `ele.style.left` + `ele.offsetLeft`
2. no restrict on `position`: using `

```html
<!DOCTYPE html>
<html>
<head>
<style>
	#container{
    	width: 100px;
        height: 100px;
    	border: 1px solid red;
        position: relative;
    }
</style>
</head>
<body>
	<div id="container">
</div>

<script>


// Version 1
function moveToRight_2(ele, dis, time) {
  var interval = 10
  var oneStep = interval * dis / time
  var totalSteps = dis / oneStep
  var step = 0
  let curLeft = ele.offsetLeft
 
  var interval_handler= setInterval(function(){
    curLeft += oneStep
    ele.style.left = curLeft + 'px'
    step += 1
    if( step === totalSteps ) clearInterval(interval_handler)
  }, interval)
}



// Version 2   
function moveToRight(ele, dis, time) {
  let d = dis/time * 10;
  let cur = 0;
  let myInterval;
  
  myInterval = setInterval(function(){ 
      cur += d;
      if (cur <= dis) {
        ele.style.transform = "translate(" + cur +"px, 0)";
      } else {
         clearInterval(myInterval);
         console.log("stop");
      }
  }, 1);
}
 
    
// Test  
var ele = document.getElementById("container");
moveToRight_2(ele, 100, 500);

</script>

</body>
</html>

```

