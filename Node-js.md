---
title: Node.js
comments: true
date: 2018-06-01 16:24:07
type: categories
categories: Full stack
tags: Node.js
---



## Event-driven & asynchrous IO & `Event Emitter`

1. **<u>NOTE :</u>**

   - Node.js is an application [<u>**a process**</u>]  in OS; but has <u>**multiple threads**</u>
   - **<u>Only one JS thread</u>**, all others are **<u>worker threads</u>** in thread pool used by OS
   - they share data via **<u>message queue</u>**

2. **Asynchrous I/O**

   - Each I/O operation is assigned a `worker thread` in Node.js <u>run time env</u>; 
   - But only one `master thread` (<u>js engine</u>) deal with computation

3. **Even Loop**

   - **Node.js** ---  **V8** --- **libuv** --- **OS**

   - **libuv [cross platform Asynchronous I/O]**: handles utilize other threads to complete asynchronous I/O

   - ```
        ┌───────────────────────────┐
     ┌─>│           timers          │
     │  └─────────────┬─────────────┘
     │  ┌─────────────┴─────────────┐
     │  │     pending callbacks     │
     │  └─────────────┬─────────────┘
     │  ┌─────────────┴─────────────┐
     │  │       idle, prepare       │
     │  └─────────────┬─────────────┘      ┌───────────────┐
     │  ┌─────────────┴─────────────┐      │   incoming:   │
     │  │           poll            │<─────┤  connections, │
     │  └─────────────┬─────────────┘      │   data, etc.  │
     │  ┌─────────────┴─────────────┐      └───────────────┘
     │  │           check           │
     │  └─────────────┬─────────────┘
     │  ┌─────────────┴─────────────┐
     └──┤      close callbacks      │
        └───────────────────────────┘
     ```


## Node.js use cases

1. **I/O-bound application**
   - Except the only one JS processing thread, all other threads deal with I/O operation, so that I/O processing can happend simultaneously => **save time**
   - JS processing thread has no CPU-bound computation, only stay in **<u>event loop</u>**, and keep staring at the top of **<u>message queue</u>**; When some event done, handle its callbacks



## `process.nextTick()` vs. `setImmediate()`

1. **<u>tick:</u>** each iteration of an Event Loop is called a *tick.*
2. `process.nextTick()` : will execute **<u>all callbacks</u>** set by it previousely in next tick
3. `setImmediate()`: will execute **<u>only one callback</u>** in next tick, even set multiple callbacks



## CommonJS: `require` & `exports`

1. **<u>Sad History</u>**: Before CommonJS, JavaScript doesn't have module importing feature, so it is limited to small script, cannot be used to develop large project

2. **<u>Current state :</u>** Many modules like `buffer`  `File`  `Http` `Socket` can be imported using ***CommonJS***

3. **Implementation:**

   - `module` is is <u>*glolal object*</u> in Node.js, have property `exports`

     ```js
     module = { exports: {} }
     ```

   - Before we step into the details, let's see before CommonJS, how to use import function. The public functions are exposed while the private properties and methods are encapsulated

     ```js
     var revealingModule = (function () {
         var privateVar = "Ben Thomas";
         function setNameFn( strName ) {
             privateVar = strName;
         }
     	return { setName: setNameFn, };
     })();
     revealingModule.setName( "Paul Adams" );
     ```

   - When a module is exported, inside Node.js it will be wrapped as `IIFE` (Immediately Invoked Function Expression)

     ```js
     // 1. original code
     var greet = function () { console.log('Hello World'); };
     module.exports = greet;
     
     // 2. Parsed code by Node.js
     (function (exports, require, module, __filename, __dirname) { //add by node.js
           var greet = function () { console.log('Hello World'); };
           module.exports = greet;
     }).apply();                                        //add by node.js
     
     return module.exports;     
     ```

4. **<u>How to use:</u>**

   - define module: `module.exports` or `exports` [`module.exports` === `exports`]

     ```js
     /* --- circle.js --- */
     
     // 1st way
     exports.area = (radius) => Math.pow(radius, 2) * 3.14;
     exports.circunference = (radius) => 2 * radius * 3.14;
     
     // 2nd way
     const area = (radius) => Math.pow(radius, 2) * 3.14;
     const circunference = (radius) => 2 * radius * 3.14;
     module.exports = {area: area, circumference: circunference};
     
     // 3rd way
     module.exports = {
         area: (radius) => Math.pow(radius, 2) * 3.14;
         circunference: (radius) => 2 * radius * 3.14;
     }
     ```

   - use module everywhere:

     ```js
     const circle = require('./circle');
     circle.area(3)
     circle.circunference(3)
     ```


## AMD: (Asynchronous Module Definition)

1. Why need AMD?**

   - Since Node.js are always running locally inside server, the `require` are instantly completed, i.e. **synchronously**
   - Browser need to download js files from server **asynchorously**. While one file want `require` another, the other one might still not downloaded => <u>ERROR</u>!
   - Thus, we need another mechnism able to **<u>import modules asynchronously :</u>** AMD comes in!

2. **How to use it in browser?**

   - **<u>[Good Article]</u>**: https://www.sitepoint.com/understanding-requirejs-for-effective-javascript-module-loading/

   - use `define()` to declare the module dependency for each file

   - download `require.js` & `main.js` in same project folder as other js files

   - only add one `<script>` tag to HTML is OK, as follows

     - ```html
       <script data-main="scripts/main" src="scripts/require.js"></script>
       ```



## Web

1. **Request Method**

   - `request.method`: GET, POST, PUT, DELETE

     - ```JS
       function (req, res) { 
           switch (req.method) 
               case 'POST':
          			update(req, res);
           		break;
         		case 'DELETE':
           		remove(req, res);
           		break;
         		case 'PUT':
           		create(req, res);
          		 	break;
         		case 'GET':
         		default:
       			get(req, res); 
       	}
       }
       ```

2. **Request URL**

   - complete URL like the following: (<u>Hash part will be discarded</u>)

     ```html
     http://user:pass@host.com:8080/p/a/t/h?query=string#hash
     ```

   - `pathname` =  `url.parse(req.url).pathname`

   - **<u>routers in Node.js:</u>**

     - url: `/[controller]/[action]/a/b/c`

     - handlers for different <u>controllers</u> & <u>actions</u>

     - ```js
       // routes different urls to their related handlers
       var url = require('url')
       function (req, res) {
       	var pathname = url.parse(req.url).pathname;
       	var paths = pathname.split('/');
       	var controller = paths[1] || 'index';
       	var action = paths[2] || 'index';
       	var args = paths.slice(3);
       	if (handles[controller] && handles[controller][action]) {
       		handles[controller][action].apply(null, [req, res].concat(args)); 
           } else {
       		res.writeHead(500);
       		res.end('找不到响应控制器'); }
       	}
       
       // handlers object
       handles.index = {};
       handles.index.index = function (req, res, foo, bar) {
       	res.writeHead(200);
       	res.end(foo); 
       };
       ```

3. **Request Query String**

   - `query` =  `url.parse(req.url).query`

   - url: `/domainname/path?foo=bar&baz=val`

     - ```js
       //eg1.  '/domainname/path?foo=bar&baz=val' 
       {
       	foo: 'bar',
       	baz: 'val' 
       }
       
       //eg2.  '/domainname/path?foo=bar&foo=val' 
       {
       	foo: ['bar', 'val'],
       }
       ```

4. **Cookie**

   - workflow: [ key point: browser sends cookie for each request]

     - server sends cookie to browser
     - browser stores it in localstorage
     - browser sends cookie <u>every time making request</u>

   - **[For browser:]**  string: `Cookie: foo=bar; baz=val`  => `req.header.cookie` 

     - ```js
       // browser parse cookie
       var parseCookie = function (cookie) { 
           var cookies = {};
       	if (!cookie) { return cookies; }
       	var list = cookie.split(';');
       	for (var i = 0; i < list.length; i++) {
       		var pair = list[i].split('=');
       		cookies[pair[0].trim()] = pair[1]; 
           }
       	return cookies; 
       };
       
       // browser adds cookie to `req` object
       function (req, res) {
       	req.cookies = parseCookie(req.headers.cookie); 
           hande(req, res);
       }
       ```

   - **[For server:]**  

     - Format:

       - ```
         Set-Cookie: name=value; Path=/ ; Expires = Sun, 01/21/1987 09:01:01 GMT; Domain = .domain.com;
         ```

     - arguments: 

       -  `name&value`: the useful stuff
       -  `domain`: cookie only sent to the domain
       -  `path`: cookie only sent when this domain/path
       -  `expire` / `maxAge`: when or how long expires
       - `HttpOnly`: will make this cookie cannot be access by `document.cookie`
       - `secure`: only sent when HTTPS; not sent when HTTP

     - **Serialize Cookie [Server] : **convert cookie to string

       - ```js
         var serialize = function (name, val, opt) { 
             var pairs = [name + '=' + encode(val)]; opt = opt || {};
             if (opt.maxAge) pairs.push('Max-Age=' + opt.maxAge);
             if (opt.domain) pairs.push('Domain=' + opt.domain);
         	if (opt.path) pairs.push('Path=' + opt.path);
             if (opt.expires) pairs.push('Expires=' + opt.expires.toUTCString()); 
             if (opt.httpOnly) pairs.push('HttpOnly');
             if (opt.secure) pairs.push('Secure');
             return pairs.join('; '); 
         };
         ```

5. **Session**

   - <u>**why need it**</u>: *cookie with too much data* will make delay in network transmission 

   - **<u>Solution:</u>** 

     - store data on server side, give a key (sessionID) to cookie to refer its data
     - set `expire` to its corresponding  cookie

6. **Cache** ***<u>response file</u>*** [ <u>has nothing to do with cookie</u> ]

   - Client: `If-Modified-Since`,  Server:  `Last-Modified`
   - Client: `If-None-Match`,  Server:  `ETag`
     - solve problem: even if file modified, but content didn't modified
     - compare the digest of target file
   - Server:  `Expires` / `Cache-Control`
     - the first 2 ways still need to send request, and wait for response => waste time!
     - This approach doesn't need to send request if doesn't `expires`
     - But there is a problem: the time are not the same on two sides, `max-age` of `cache-control`comes in
   - how to update cache when server file updated within expired period?
     - **<u>Problem</u>**: browser will use stale file, since it not expire
     - We also update the URL, since *Cache pairs with URL*

7. **Authentication / Authorization** 

   - <u>basic authentication</u>
     - request with : `useranme` & `password`
   - <u>OAuth2.0</u>
     - request with `access_token`
   - <u>Bearer token: JWT</u>
     - request with `Json Web Token`

8. **Form Data**

   - content-type: `application/x-www-form-urlencoded` ("foo=bar&baz=val")
   - content-type: `application/json;`
   - content-type: `application/xml;`

9. **Simulate a Client(browser) & a Server**

   1. server implementation [send response as a server]

      1. ```js
         let http = require('http')
         let fs = require('fs')
         let url = require('url')
         
         http.creatServer( function(req, res){
             let pathname = url.parse(req.url).pathname
             console.log("request for " + pathname + "received.")
             
             fs.readFile(pathname.substr(1), function(err, data){
                 if(err){
                     console.log(err)
                     res.writeHead(404, {'Content-Type': 'text/html'})
                 }else{
                     res.writeHead(200, {'Content-Type': 'text/html'})
                     res.write(data.toString())
                 }
                 res.end()
             })
         }).listen(8080)
         
         console.log("Server running at http://127.0.0.1:8080/")
         ```

   2. client implementation [send request as a browser]

      1. ```js
         let http = require('http')
         
         let options ={
             host: 'localhost',
             port: '8080',
             path: '/index.html'
         }
         
         let callback = function(res){
             let body = ''
             res.on( 'data', function(data){
                 body += data
             })
             
             res.on( 'end', function(){
                 console.log(body)
             })
         }
         
         let req = http.request(options, callback)
         req.end()
         ```


## XSS / CSRF

1.  XSS: cross site script
   - inject script from unsafe input, and runs it to get cookie, ie sessionID
   - **<u>[Solution]</u>**: convert to <u>HTML entities</u>: `&` will be `&amp;`
2. CSRF: 
   - don't bother to get sessionID
   - lure customer to another website, while he is in middle of session of some weak-protection Bank website
   - the other website will submit a form to Bank website indicating transfer money
   - Bank website server cannot tell from whom that request is. It still believe that is from its customer since he is in the middle of the session
   - **<u>[Solution]</u>**: 
     - the Bank form add another random hidden input field
     - each time receive request, check if the random value equal to sent one
     - This way the form cannot be duplicate

## Global variables
1. `console` 
   - `console.log('11')`
2. `process` 
   - `process.exit(0)`
   - `process.nextTick(callback)`
3. `__filename`: **<u>absolute path + name</u>** for the current script being executed
4. `__dirname`: **<u>absolute path</u>** for the current script being executed



## Util Objects

1. `util`: make one object inherit from the other one
   1. `util.inherits(constructor, superConstructor)`

## Versions

1. **semantic versioning** 
    - consists of three numbers, separated by periods, such as 2.3.0
    - the middle number has to be incremented, when new functionality is added
    - the first number has to be incremented, when compatibility is broken, so that existing code that uses the package might not work with the new version,
2. **caret character (^)**
    - any version compatible with the given number may be installed
    - `^2.3.0` would mean that any version greater than or equal to 2.3.0 and less than 3.0.0 is allowed



