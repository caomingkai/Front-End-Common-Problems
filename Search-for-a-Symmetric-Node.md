---
title: Search for a Symmetric Node
comments: true
date: 2018-10-07 20:34:30
type: categories
categories: Full stack
tags: 
- DOM
- parentNode
- Algorithm
---



## Problem

Given two DOM trees with the same structure, and one node in tree1, try to find its symmetric node in tree2



## Solution 1

If we are not so familiar with DOM API, we might forget there is an attribute for each DOM element: `parentNode`. Then we might just traverse both trees at the same time, when we encounter the node in tree1, the node in tree2 is what we want.

```js
function getSymmetricNode(target, root1, root2){
    if( target === root1 ) return root2
    
    for( let i = 0; i < root1.childNodes.length; i++ ){
        let node1 = root1.childNodes[i]     
        let node2 = root2.childNodes[i]
        if( node1 === target ) return node2
		getSymmetricNode(target, node1, node2 )
    }
    return null
}
```

1. Time Complexity: **O(b^d)  [d: depth, b: branches]**, which is BAD!

## Solution 2 [Improved ]

Actually there is an convenient attribute we could utilize, that is `parentNode`

1. First, we could go up to find that <u>**path** from root1 to target</u>
   - The path is consist of indices of node on that path
2. Using that path of indices, we could go downwards from root2 to find the desired node

```js
let getSymmetricNode = (target, root1, root2) => {
    let path = getIndexPath( target, root1 )
    return getNode( path, root2 )
}

let getIndexPath = (target, root1)=>{
    let path = []
    let curNode = target
    while( curNode && curNode.parentNode ){
        let index = curNode.parentNode.indexOf(curNode)
        path.push(index)
        curNode = curNode.parentNode
    }
    return path
}

let getNode =(path, root)=>{
    let node
    while( path.length > 0 ){
        node = root.childNodes[path.pop()]
    }
    return node
}

```



## Testing

```html
// Testing
<!doctype html>

<html>
  <head>
    <meta charset="UTF-8" />
    <title>Hello World</title>
  </head>
  <body>
    <div id="root1">
      <div>
        <div></div>
      </div>
      <div>
        <div id="node1"></div>
        <div></div>
      </div>
    </div>

    <div id="root2">
      <div>
        <div></div>
      </div>
      <div>
        <div id="node2"></div>
        <div></div>
      </div>
    </div>
    <script>
      
      
      // --------------- 1: O(b^d) --------------
      function getSymmetricNode1(target, root1, root2){
          if( target === root1 ) return root2
          for( let i = 0; i < root1.childNodes.length; i++ ){
              let node1 = root1.childNodes[i]     
              let node2 = root2.childNodes[i]
              let res = getSymmetricNode1(target, node1, node2)
              if( null !== res ) return res
                
          }
          return null
      }
      
      
      
      
      // -------------- 2: O(b*d) ---------------
      function getSymmetricNode(target, root1, root2){
          let path = getIndexPath( target, root1 )
          return getNode( path, root2 )
      }

      function getChildNodesArray(node){
        return Array.prototype.slice.call(node.childNodes)
      }
      
      
      function getIndexPath(target, root1){
          let path = []
          let curNode = target
          while( curNode !==root1 && curNode && curNode.parentNode ){
              let index = getChildNodesArray(curNode.parentNode).indexOf(curNode)
              path.push(index)
              curNode = curNode.parentNode
          }
          return path
      }

      
      function getNode(path, root){
          let node = root
          while( path.length > 0 ){
              node = getChildNodesArray(node)[path.pop()]
          }
          return node
      }

      const root1 = document.getElementById('root1');
      const root2 = document.getElementById('root2');
      const node1 = document.getElementById('node1');
      const node2 = document.getElementById('node2');

      const nodeX = getSymmetricNode(node1, root1, root2 );
      console.log(nodeX)
      console.log(nodeX === node2); // true
    </script>
  </body>
</html>

```

