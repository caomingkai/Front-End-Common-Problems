---
title: Express Framework
comments: true
date: 2018-06-01 14:44:43
type: categories
categories: Full stack
tags: 
- Express
- Node.js
---



## `express()`

1. `let express = require('express')`: Creates an Express application

2. `express.static(root, [options])`: serves <u>**static files**</u> and is based on [serve-static](https://expressjs.com/en/resources/middleware/serve-static.html).

   - here we can set `etag` `lastModified` `maxAge` etc.

   - ```js
     var options = {
       dotfiles: 'ignore',
       etag: false,
       extensions: ['htm', 'html'],
       index: false,
       maxAge: '1d',
       redirect: false,
       setHeaders: function (res, path, stat) {
         res.set('x-timestamp', Date.now())
       }
     }
     
     app.use(express.static('public', options))
     ```

3. `express.Router([options])`: Creates a new [router](https://expressjs.com/en/4x/api.html#router) object. Refer to **Routing** section below


## App

1. The `app` object denotes the Express application

   1. ```js
      var express = require('express');
      var app = express();
      
      app.get('/', function(req, res){
        res.send('hello world');
      });
      
      app.listen(3000);
      ```

2. **<u>sub-app :</u>**

   1. `app.mountpath`

      1. ```js
         var express = require('express');
         
         var app = express(); // the main app
         var admin = express(); // the sub app
         
         admin.get('/', function (req, res) {
           console.log(admin.mountpath); // /admin
           res.send('Admin Homepage');
         });
         
         app.use('/admin', admin); // mount the sub app
         ```

   2. `app.on('mount', callback(parent))`

      1. ```JS
         var admin = express();
         
         admin.on('mount', function (parent) {
           console.log('Admin Mounted');
           console.log(parent); // refers to the parent app
         });
         
         admin.get('/', function (req, res) {
           res.send('Admin Homepage');
         });
         
         app.use('/admin', admin);
         ```

3. `app.METHOD(path, callback [, callback ...])`

   -  METHOD is the HTTP method of the request, such as GET, PUT, POST, and so on, **<u>in lowercase</u>**. 
   - `callback` can be: 
     - A middleware function.
     - A series of middleware functions (separated by commas).
     - An array of middleware functions.
     - A combination of all of the above.

4. `app.listen([port[, host[, backlog]]][, callback])` : Starts a UNIX **<u>socket</u>** , binds and listens for connections on the specified host and port.

5. `app.param([name], callback)`: Add callback triggers to [route parameters](https://expressjs.com/en/guide/routing.html#route-parameters), where `name` is the name of the parameter or an array of them, and `callback` is the callback function. 

6. `app.path()`: returns the canonical path of the app, a string.

   1. ```js
      var app = express()
        , blog = express()
        , blogAdmin = express();
      
      app.use('/blog', blog);
      blog.use('/admin', blogAdmin);
      
      console.log(app.path()); // ''
      console.log(blog.path()); // '/blog'
      console.log(blogAdmin.path()); // '/blog/admin'
      ```

7. `app.route(path)`: **<u>Returns an instance of a single route</u>**

   1. Use `app.route()` to avoid duplicate route names

   2. e.g. 

   3. ```js
      var app = express();
      
      app.route('/events')
      .all(function(req, res, next) {
        // runs for all HTTP verbs first
        // think of it as route specific middleware!
      })
      .get(function(req, res, next) {
        res.json(...);
      })
      .post(function(req, res, next) {
        // maybe add a new event...
      });
      ```

8. `app.use([path,] callback [, callback...])`: mount specified middleware at the specified path

## Routing

1. `app.get( url, handler )`	

2. `app.post( url, handler )`

3. `app.put( url, handler )`

4. `app.delete( url, handler )`

   - ```js
     let express = require('express')
     let app = express()
     
     app.get('/', function(req, res){
         res.send("Hello world")
     })
     ```

5. `app.all( url, handler )`: special route method to load all headlers for GET, POST, PUT, DELETE, etc,

6. **route parameters**

   1. ```js
      
      app.get('/users/:userId/books/:bookId', function (req, res) {
        res.send(req.params)
      })
      // If Request URL: http://localhost:3000/users/34/books/8989
      // Then req.params: { "userId": "34", "bookId": "8989" }
      ```

7. **route handlers**: 

   - <u>Multiple route handlers</u> can be put inside a route as *function array*

   - use `next()` to invoke the next handler to handle route

   - ```js
     app.get('/example/d', [cb0, cb1], function (req, res, next) {
       console.log('the response will be sent by the next function ...')
       next()
     }, function (req, res) {
       res.send('Hello from D!')
     })
     ```

8. `app.route()`

   - create chainable route handlers for a route path by using `app.route()`

   - ```js
     app.route('/book')
       .get(function (req, res) {
         res.send('Get a random book')
       })
       .post(function (req, res) {
         res.send('Add a book')
       })
       .put(function (req, res) {
         res.send('Update the book')
       })
     ```

9. `express.Router()`: achieve <u>*relative*</u> & <u>*modular*</u> route handling

   - using `express.Router()` to create a modular route, which can handle relative route as a single module/ file

   - then pass it to the <u>*abosolute route handling module*</u>

   - **<u>[e.g. ]:</u>** The app will now be able to handle requests to `/birds` and `/birds/about`, as well as call the `timeLog` middleware function that is specific to the route.

   - ```js
     /*--- bird.js : modular route handler ---*/
     
     var express = require('express')
     var router = express.Router()
     
     // middleware that is specific to this router
     router.use(function timeLog (req, res, next) {
       console.log('Time: ', Date.now())
       next()
     })
     // define the home page route
     router.get('/', function (req, res) {
       res.send('Birds home page')
     })
     // define the about route
     router.get('/about', function (req, res) {
       res.send('About birds')
     })
     
     module.exports = router
     
     
     /*--- app.js: include bird route handler ---*/
     var birds = require('./birds')
     // ...
     app.use('/birds', birds)
     ```


## Middleware

1. Credit to: https://segmentfault.com/a/1190000010877994

2. different levels of middleware

   - [Application-level middleware](https://expressjs.com/en/guide/using-middleware.html#middleware.application): `app.use()`

   - [Router-level middleware](https://expressjs.com/en/guide/using-middleware.html#middleware.router) `router.use()`

   - [Built-in middleware](https://expressjs.com/en/guide/using-middleware.html#middleware.built-in)  `express.static()`

   - [Third-party middleware](https://expressjs.com/en/guide/using-middleware.html#middleware.third-party) `app.use(cookieParser())`

     - ```js
       var express = require('express')
       var app = express()
       var cookieParser = require('cookie-parser')
       
       // load the cookie-parsing middleware
       app.use(cookieParser())
       ```

3. write our own middleware!

   ```js
   // 1. create our own middleware
   function App() {
       if (!(this instanceof App))
           return new App();
       this.init();
   }
   App.prototype = {
       constructor: App,
       init: function() {
           this.request = { //模拟的request
               requestLine: 'POST /iven_ HTTP/1.1',
               headers: 'Host:www.baidu.com\r\nCookie:BAIDUID=E063E9B2690116090FE24E01ACDDF4AD:FG=1;BD_HOME=0',
               requestBody: 'key1=value1&key2=value2&key3=value3',
           };
           this.response = {}; //模拟的response
           this.chain = []; //存放中间件的一个数组
           this.index = 0; //当前执行的中间件在chain中的位置
       },
       use: function(handle) { //这里默认 handle 是函数，并且这里不做判断
           this.chain.push(handle);
       },
       next: function() { //当调用next时执行index所指向的中间件
           if (this.index >= this.chain.length)
               return;
           let middleware = this.chain[this.index];
           this.index++;
           middleware(this.request, this.response, this.next.bind(this));
       },
   }
   
   // 2.  use it
    function lineParser(req, res, next) {
           let items = req.requestLine.split(' ');
           req.methond = items[0];
           req.url = items[1];
           req.version = items[2];
           next(); //执行下一个中间件
       }
   
   function headersParser(req, res, next) {
       let items = req.headers.split('\r\n');
       let header = {}
       for(let i in items) {
           let item = items[i].split(':');
           let key = item[0];
           let value = item[1];
           header[key] = value;
       }
       req.header = header;
       next(); //执行下一个中间件
   }
   
   
   let app = App();
   app.use(lineParser);
   app.use(headersParser);
   app.next();
   ```



## request

The `req` object represents the HTTP request and has properties for the *request query string*, *parameters*, *body*, *HTTP headers*, and so on

1. `req.baseUrl`: 

   - The URL path on which a router instance was mounted; 

   - similar to the [mountpath](https://expressjs.com/en/4x/api.html#app.mountpath) property of the `app` object, except `app.mountpath` returns the matched path pattern(s).

   - ```js
     var greet = express.Router();
     
     greet.get('/jp', function (req, res) {
       console.log(req.baseUrl); // /greet
       res.send('Konichiwa!');
     });
     
     app.use('/greet', greet); // load the router on '/greet'
     ```

2. `req.body`:  key-value pairs of data submitted in the request body; Need `body-parser()`

   1. ```js
      var app = require('express')();
      var bodyParser = require('body-parser');
      
      app.use(bodyParser.json()); // for parsing application/json
      app.use(bodyParser.urlencoded({ extended: true })); // for parsing application/x-www-form-urlencoded
      
      // Now all JSON & Raw text are parsed, and ready to use as `req.body`
      app.post('/profile', function (req, res, next) {
        console.log(req.body);
        res.json(req.body);
      });
      ```

3. `req.cookies`:   using `cookie-parse` to set cookies onto request

4. `req.hostname` : hostname of the `Host` HTTP header.

5. `req.ip`:  remote IP address of the request.

6. `req.method`: method of request: `GET`, `POST`, `PUT`, and so on.

7. `req.originalUrl` vs `req.baseUrl` vs `req.path`: 

   1. ```js
      app.use('/admin', function(req, res, next) {  // GET 'http://www.example.com/admin/new'
        console.log(req.originalUrl); // '/admin/new'
        console.log(req.baseUrl); // '/admin'
        console.log(req.path); // '/new'
        next();
      });
      ```

8. `req.query`: object containing a property for each query string parameter in the route

   1. ```js
      // GET /search?q=tobi+ferret
      req.query.q
      // => "tobi ferret"
      
      // GET /shoes?order=desc&shoe[color]=blue&shoe[type]=converse
      req.query.order
      // => "desc"
      
      req.query.shoe.color
      // => "blue"
      
      req.query.shoe.type
      // => "converse"
      ```

9. `req.route`: currently-matched route, a string. 

   1. ```js
      app.get('/user/:id?', function userIdHanler(req, res) {
        console.log(req.route);
        res.send('GET');
      });
      
      ==>
      { path: '/user/:id?',
        stack:
         [ { handle: [Function: userIdHandler],
             name: 'userIdHandler',
             params: undefined,
             path: undefined,
             keys: [],
             regexp: /^\/?$/i,
             method: 'get' } ],
        methods: { get: true } }
      ```


## response

represents the HTTP response that an Express app sends when it gets an HTTP request.

1. `res.app` === `req.app`: a reference to the instance of the Express application that is using the middleware.

2. `res.headersSent`:  indicates if the app sent HTTP headers for the response.

3. `res.append(field [, value])`: Appends the specified `value` to the HTTP response header `field`

4. `res.cookie(name, value [, options])`: Sets cookie `name` to `value`

   1. ```js
      res.cookie('name', 'tobi', { domain: '.example.com', path: '/admin', secure: true });
      res.cookie('rememberme', '1', { expires: new Date(Date.now() + 900000), httpOnly: true });
      //Default encoding
      res.cookie('some_cross_domain_cookie', 'http://mysubdomain.example.com',{domain:'example.com'});
      // Result: 'some_cross_domain_cookie=http%3A%2F%2Fmysubdomain.example.com; Domain=example.com; Path=/'
      
      ```

5. `res.clearCookie(name [, options])`: Clears the cookie specified by `name`.

   1. ```js
      res.cookie('name', 'tobi', { path: '/admin' });
      res.clearCookie('name', { path: '/admin' });
      ```


6. `res.download(path [, filename] [, options] [, fn])`: Transfers the file at `path` as an “*attachment*”. Typically, browsers will prompt the user for download. 

   1. ```js
      res.download('/report-12345.pdf', 'report.pdf', function(err){
        if (err) {
          // Handle error, but keep in mind the response may be partially-sent
          // so check res.headersSent
        } else {
          // decrement a download credit, etc.
        }
      });
      ```

7. `res.end([data] [, encoding])`: Ends the response process

   1. If you need to respond with data, instead use methods such as [res.send()](https://expressjs.com/en/4x/api.html#res.send)and [res.json()](https://expressjs.com/en/4x/api.html#res.json).

8. `res.format(object)`: when present. It uses [req.accepts()](https://expressjs.com/en/4x/api.html#req.accepts) to select a handler for the request

   1. ```
      ``res.format({
        'text/plain': function(){
          res.send('hey');
        },
      
        'text/html': function(){
          res.send('<p>hey</p>');
        },
      
        'application/json': function(){
          res.send({ message: 'hey' });
        },
      
        'default': function() {
          // log the request and respond with 406
          res.status(406).send('Not Acceptable');
        }
      });
      ```

9. `res.set(field, vale)`: set response header

10. `res.get(field)`: get HTTP response header specified by `field`

11. `res.json([body])`: Sends a JSON response.

    1. ```js
       res.json({ user: 'tobi' });
       res.status(500).json({ error: 'message' });
       ```

12. `res.redirect([status,] path)`: Redirects to the URL derived from the specified `path`, with specified `status`

    1. ```js
       res.redirect(301, 'http://example.com');
       res.redirect('../login');
       ```

13. `res.send([body])`: Sends the HTTP response.

    1. ```js
       res.send(new Buffer('whoop'));
       res.send({ some: 'json' });
       res.send('<p>some html</p>');
       res.status(404).send('Sorry, we cannot find that!');
       res.status(500).send({ error: 'something blew up' });
       ```

14. `res.sendFile(path [, options] [, fn])`: Transfers the file at the given `path`.

    1. ```js
       app.get('/file/:name', function (req, res, next) {
       
         var options = {
           root: __dirname + '/public/',
           dotfiles: 'deny',
           headers: {
               'x-timestamp': Date.now(),
               'x-sent': true
           }
         };
       
         var fileName = req.params.name;
         res.sendFile(fileName, options, function (err) {
           if (err) {
             next(err);
           } else {
             console.log('Sent:', fileName);
           }
         });
       
       });
       ```

15. `res.sendStatus(statusCode)`:  Sets the response HTTP status code to `statusCode` and send its string representation as the response body.

    1. ```js
       res.sendStatus(200); // equivalent to res.status(200).send('OK')
       res.sendStatus(403); // equivalent to res.status(403).send('Forbidden')
       res.sendStatus(404); // equivalent to res.status(404).send('Not Found')
       res.sendStatus(500); // equivalent to res.status(500).send('Internal Server Error')
       ```



##  `bodyParser`

1. `urlencoded` format data submitted from `<form>`，can be parsed into`res.body`

    ```javascript
    app.use(bodyParser.urlencoded({ extended: false }));
    ```

2. `json` format data transferred using API, can be parsed into `res.body`

    ```javascript
    app.use(bodyParser.json());
    ```



## `set()`: set globle variable in Express

> You can use the express instance to store and retrieve variables. In this case, you can set the title to be "My Site" and retrieve it later with something like `var title = app.get('title');`**without the need to declare and keep a global variable messing around**

```
app.set('title','My Website');  // set()
var title = app.get('title');   // get()

<=>

title = 'My Website'
```

