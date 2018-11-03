---
title: Common Algorithm Problems - JS
comments: true
date: 2018-10-14 14:54:09
type: categories
categories: Algorithm
tags:
- sorting
- linked list
- DFS
- recursion
---

0. ## All credits to **<u>[ThatJSDude](https://www.thatjsdude.com)</u>**

1. ## [Verify a prime number?](https://www.thatjsdude.com/interview/js1.html#isPrime)

   1. version 1: **not too good**

      1. ```js
         function isPrime( num ){
             let i = 2
             while( i <= num/2 ){
                 if( num % 2 === 0 ) return false
                 i += 1
             }
             return true
         }
         
         // Test 
         console.log(isPrime(1))   // true
         console.log(isPrime(2))   // true
         console.log(isPrime(4))   // false
         console.log(isPrime(7))   // true
         console.log(isPrime(101)) // true
         ```

   2. Improved Version

      1. ```js
         function isPrime(n)
         {
           let divisor = 3, 
           let limit = Math.sqrt(n);
           
           //check simple cases
           if (n == 2 || n == 3) return true;
           if (n % 2 == 0) return false;
         
           while (divisor <= limit)
           {
             if (n % divisor == 0) return false;
             else  divisor += 2;
           }
           return true;
         }
         
         > isPrime(137);
           = true
         > isPrime(237);
           = false
         ```

2. ## [Find all prime factors of a number?](https://www.thatjsdude.com/interview/js1.html#primFactors)

   1. Find all prime factors

      1. ```js
         function findAllPrimeFactors( num ){
             let factors = []
             let i = 2
             while( num > 2 ){
                 if( num % i === 0 ){
                     factors.push(i)
                     num /= i
                 }else{
                     i += 1
                 }
             }
             return factors
         }
         
         // test
         console.log(findAllPrimeFactors(420)) // 2 2 3 5 7
         ```

   2. Find distinct prime factors

      1. ```js
         function findDistinctPrimeFactors( num ){
             let factors = []
             let i = 2
             let temp = num
             while( i <= num/2 ){
                 if( temp % i === 0 ) {
                     factors.push(i)
                     while(temp % i === 0) temp /= i
                 }
                 i += 1
             }
             return factors
         }
         
         // test
         console.log(findDistinctPrimeFactors(420)) // 2 3 5 7
         ```

3. ## [Get nth Fibonacci number?](https://www.thatjsdude.com/interview/js1.html#fibonacci)

   1. `f(n) = f(n-1) + f(n-2)`

   2. Give `n`, print fibonacci sequence from 0 ~ n

      1. ```js
         // 1. Iterative
         function f(n){
             let fb = []
             fb[0] = 1
             fb[1] = 1
             
             for( let i = 2; i <= n; i++ ){
                 fb[i] = fb[i-1] + fb[i-2]
             }
             return fb;
         }
         
         // 2. recursive
         function wrapperFb(n){
             let memo = new Array(n+1).fill(0)
             memo[0] = 1
             memo[1] = 1
             function f(n){
                 if( n===0 || n===1 ) return 1
                 if( memo[n] !== 0 ) 
                     return memo[n]
                 else 
                     memo[n] = f(n-1) + f(n-2)
                 return memo[n]
             }
             f(n)
             return memo
         }
         
         console.log(wrapperFb(5)) // 1 1 2 3 5 8
         
         ```

   3. Give `n`, print the `nth` fibonacci number

      1. ```js
         // 1. iterative
         function fth(n){
             let fb = []
             fb[0] = 1
             fb[1] = 1
             
             for( let i = 2; i <= n; i++ ){
                 fb[i] = fb[i-1] + fb[i-2]
             }
             return fb[n];
         }
         
         console.log(fth(5)) // 8
         
         
         // 2. recursive
         function wrapperFb(n){
             let memo = new Array(n+1).fill(0)
             memo[0] = 1
             memo[1] = 1
             function f(n){
                 if( n===0 || n===1 ) return 1
                 if( memo[n] !== 0 ) 
                     return memo[n]
                 else 
                     memo[n] = f(n-1) + f(n-2)
                 return memo[n]
             }
             return f(n)
         }
         
         console.log(wrapperFb(5)) //  8
         
         ```

4. ## [Find the greatest common divisor of two numbers?](https://www.thatjsdude.com/interview/js1.html#greatestCommonDivisor)

   1. ```js
      function greatCommonDevisor ( n1, n2 ){
          let res = 1
          let i = 2
          while( i <= n1 || i <= n2 ){
              while( n1%i === 0 && n2%i === 0 ){
                  res *= i
                  n1 /= i
                  n2 /= i
              }
              i += 1
          }
          return res
      }
      console.log(greatCommonDevisor(36,48)) // 12
      ```

5. ## [Remove duplicate members from an array?](https://www.thatjsdude.com/interview/js1.html#removeDuplicate)

   1. ```js
      function removeDup(arr){
          let set = new Set()
          return arr.filter( item =>{
              if( !set.has(item) ){
                  set.add(item)
                  return item
              }
          })
      }
      let arr = [3,2,3,3,1,2,2,4,4,3,5]
      console.log(removeDup(arr))  // [3,2,1,4,5]
      ```

6. ## [Merge two sorted array?](https://www.thatjsdude.com/interview/js1.html#mergeSotedArray)

   1. ```js
      function merge( arr1, arr2 ){
          let arr = []
          let i = 0, j = 0
          while( i < arr1.length && j < arr2.length ){
              if( arr1[i] < arr2[j] ){
                  arr.push(arr1[i])
                  i += 1
              }else{
                  arr.push(arr2[j])
                  j += 1
              }
          }
          if( i < arr1.length ) arr.push(...arr1.slice(i))
          if( j < arr2.length ) arr.push(...arr2.slice(j))
          return arr
      }
      
      let arr1 = [ 1, 3, 5, 5, 6, 8 ]
      let arr2 = [ 1, 2, 2, 4, 7, 8, 9 ]
      console.log(merge(arr1, arr2)) // [1,1,2,2,3,4,5,5,6,7,8,8,9]
      ```

7. ## [Swap two numbers without using a temp variable?](https://www.thatjsdude.com/interview/js1.html#swapTemp)

   1. using `diff`

      1. ```js
         function swap(a, b){
             a = a - b
             b = a + b
             a = b - a
         }
         ```

   2. using `XOR`

      1. ```js
         function swap(a, b){
             a = a ^ b
             b = a ^ b
             a = a ^ b
         }
         ```

8. ## [Reverse a string in JavaScript?](https://www.thatjsdude.com/interview/js1.html#string_reverse)

   1. ```js
      // 1. recursive version
      function reverseStr( str ){
          if( str === "" ) return ""
          return reverseStr(str.substring(1)) + str.charAt(0)
      }
      console.log(reverseStr("abcdef")) // "fedcba"
      
      // 2. built-in method
      function reverse(str){
        if(!str || str.length <2) return str;
        return str.split('').reverse().join('');
      }
      console.log(reverseStr("abcdef")) // "fedcba"
      
      // 3. string util method
      String.prototype.reverse = function(){
          if(!this || this.length <2) return this;
        	return this.split('').reverse().join('');
      }
      let str = "abcdef"
      console.log(str.reverse()) // "fedcba"
      ```

9. ## [Find the first non repeating char in a string?](https://www.thatjsdude.com/interview/js1.html#firstNonRepeating)

   1. Using `Map` instead of `object` to collect the frequency for each char

   2. Since we want to iterate the map in-order, while `object` cannot guanrentee this

      1. ```js
         // In the end, we need to iterate the key in-order, so we cannot count on object, but instead of Map, which give in-order traverse
         
         function firstNonRepeat(str){
             let map = new Map()
             for( let i = 0; i < str.length; i++ ){
                 let char = str[i]
                 if( map.has(char) ){
                     map.set( char, map.get(char)+1 )
                 }else{
                     map.set( char, 1 )
                 }
             }
             let arr = Array.from(map)
             for( let i = 0; i < arr.length; i++ ){
                 if( arr[i][1] === 1 ) return arr[i][0]
             }
             return -1;
         }
         
         let str = "the quick brown fox jumps then quickly blow air"
         console.log(firstNonRepeat(str))
         
         ```

10. ## [Match substring of a sting?](https://www.thatjsdude.com/interview/js1.html#subString)

   1. ```js
      // aaabc
      // aab
      function subStringFinder(str, subStr){
          let index = 0
          while( index <= str.length-subStr.length ){
              let sub = str.substring(index, index+subStr.length)
              if( sub === subStr ) 
                  return index
              else 
                  index += 1
          }
          return -1
      }
      
      // test
      console.log(subStringFinder("aaabc", "aab")) // 1
      console.log(subStringFinder('abbcdabbbbbck', 'bbbck')  ) // 8
      ```

11. ## [Create all permutation of a string?](https://www.thatjsdude.com/interview/js1.html#permutation)

    1. ```js
       /*  
               0         1    2     3
           1   2   3
          2 3  
         3   2
       
       */
       
       // [version 1.1 recursive]: String with no-dup character
       function permutations(str){
           let perm = []
           let prefix = ""
           function dfs(str){
               if( prefix.length === str.length ){
                   perm.push(new String(prefix))
                   return
               }
       
               for( let i = 0; i < str.length; i++ ){
                   if( prefix.indexOf(str[i]) === -1 ){
                       prefix += str[i]
                       dfs(str)
                       prefix = prefix.substring(0, prefix.length-1)
                   }
               }
           }
           dfs(str)
           return perm
       }
       console.log(permutations("abc"))
       
       // [version 1.2 iterative]: String with no-dup character
       function permutations(str){
           let queue = []
           queue.push("")
           let level = 0
           while( level < str.length ){
               let size = queue.length
               for( let k = 0; k < size; k++ ){
                   let prefix = queue.shift()
                   for( let i = 0; i < str.length; i++ ){
                       if( prefix.indexOf(str[i]) === -1 ){
                           queue.push(prefix+str[i])
                       }
               	}
               }
               level += 1
           }
           return queue
       }
       console.log(permutations("abc"))
       
       
       
       // [version 2.1-recursive]: string with duplicate character
       function permutations(str){
           let map = new Map()
           let perm = []
           let prefix = ""
           
           for( let i = 0; i < str.length; i++ ){
               if( map.has(str[i]) ){
                   map.set( str[i], map.get(str[i])+1 )
               }else{
                   map.set( str[i], 1 )
               }
           }
           
           function dfs(str){
               if( prefix.length == str.length ){
                   perm.push(prefix)
                   return
               }
               let freqArr = Array.from(map)
               console.log(freqArr)
               for( let i = 0; i < freqArr.length; i++ ){
                   let char = freqArr[i][0]
                   let freq = freqArr[i][1]
                   if( map.get(char) > 0){
                       prefix += char
                       map.set(char, freq-1 )
                       dfs(str)
                       map.set(char, freq )
                       prefix = prefix.substring(0, prefix.length-1)
                   }
               }
           }
           dfs(str)
           return perm
       }
       
       console.log(permutations("abc"))
       console.log(permutations("aac"))
       
       ```
