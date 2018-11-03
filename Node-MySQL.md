---
title: Node+MySQL
comments: true
date: 2018-05-14 21:30:13
type: categories
categories: Full stack
tags: 
- Node.js
- MySQL
---

### Install
This is a Node.js module, install:
`npm install mysql --save`


### Connect
Before connecting to databases, make sure MySQL database is turned on.

【注意】：

- 先执行 `connect()`,让connection object与数据库建立连接。
- 然后，在对该connection执行`.query(MySQL_Command, callback_func)`

```javascript
var mysql      = require('mysql');
var connection = mysql.createConnection({
  host     : 'example.org',
  user     : 'bob',
  password : 'secret'
});

connection.query('SELECT 1', function (error, results, fields) {
  if (error) throw error;
  // connected!
});
```

### Disconnect

```javascript
connection.end(function(err) {
  // Callback function
});
```


### Escaping query values

Three ways to avoid **SQL injection**

- mysql.escape()
- connection.escape() 
- pool.escape()
- `?`placeholder


```javascript
// 1 -connection.escape()
var userId = 'some user provided value';
var sql    = 'SELECT * FROM users WHERE id = ' + connection.escape(userId);
connection.query(sql, function (error, results, fields) {
  if (error) throw error;
  // ...
});

// 2 - ? placeholder
var adr = 'Mountain 21';
var sql = 'SELECT * FROM customers WHERE address = ?';
con.query(sql, [adr], function (err, result) {
  if (err) throw err;
  console.log(result);
});
```

### Creating a Database

```javascript
connection.query("CREATE DATABASE mydb", function (err, result) {
    if (err) throw err;
    console.log("Database created");
  });
```
### Creating a Table
Two ways:

- create database and table simultenously
- first create database, and connect it; then create table with the connection

```javascript
// approach 1

connection.query("CREATE DATABASE db", function (error, rows, fields) {
  if (error) throw error;
});

connection.query("USE db", function (error, rows, fields) {
  if (error) throw error;
});

connection.query("CREATE TABLE tb(id INT, name VARCHAR(50))", function (error, rows, fields) {
  if (error) throw error;
});


//-----------------------
// approach 2

var con = mysql.createConnection({
  host: "localhost",
  user: "yourusername",
  password: "yourpassword",
  database: "mydb"
});

con.connect(function(err) {});

var sql = "CREATE TABLE customers (name VARCHAR(255), address VARCHAR(255))";

con.query(sql, function (err, result) {
if (err) throw err;
console.log("Table created");
});
```

### Selecting From a Table

To select data from a table in MySQL, use the `SELECT` statement.

```javascript
var sql = "SELECT * FROM customers WHERE address = 'Park Lane 38'";

con.query(sql, function (err, result) {
    if (err) throw err;
    console.log(result);
});
```

**Wildcard Characters**`%`: represent zero, one or multiple characters:

```javascript
var sql = "SELECT * FROM customers WHERE address LIKE 'S%'"
```


### Insert Into Table

To fill a table in MySQL, use the `INSERT INTO` statement.

```javascript
var sql = "INSERT INTO customers (name, address) VALUES ('Company Inc', 'Highway 37')";

con.query(sql, function (err, result) {
if (err) throw err;
console.log("1 record inserted");
});
```

### Update Table

You can update existing records in a table by using the `UPDATE` statement:

```javascript
var sql = "UPDATE customers SET address = 'Canyon 123' WHERE address = 'Valley 345'";

  con.query(sql, function (err, result) {
    if (err) throw err;
    console.log(result.affectedRows + " record(s) updated");
});
```

### Sort the Result
Use the `ORDER BY` statement to sort the result in ascending or descending order.

The `ORDER BY` keyword sorts the result ascending by default. To sort the result in descending order, use the `DESC` keyword.

```javascript
var sql = "SELECT * FROM customers ORDER BY name DESC";

con.query(sql, function (err, result) {
    if (err) throw err;
    console.log(result);
});
```

### Delete Record

You can delete records from an existing table by using the `DELETE FROM` statement:

```javascript
var sql = "DELETE FROM customers WHERE address = 'Mountain 21'";

con.query(sql, function (err, result) {
    if (err) throw err;
    console.log("Number of records deleted: " + result.affectedRows);
```


### Limit the Result

You can limit the number of records returned from the query, by using the `LIMIT` statement:

```javascript
var sql = "SELECT * FROM customers LIMIT 5";

con.query(sql, function (err, result) {
if (err) throw err;
console.log(result);
});
```

### Join Two or More Tables

Combine rows from two or more tables, based on a related column between them, by using a `JOIN` statement.

```
//users

[
  { id: 1, name: 'John', favorite_product: 154},
  { id: 2, name: 'Peter', favorite_product: 154},
  { id: 3, name: 'Amy', favorite_product: 155},
  { id: 4, name: 'Hannah', favorite_product:},
  { id: 5, name: 'Michael', favorite_product:}
]
```

```
// products
[
  { id: 154, name: 'Chocolate Heaven' },
  { id: 155, name: 'Tasty Lemons' },
  { id: 156, name: 'Vanilla Dreams' }
]
```

**1. Inner join**

These two tables can be combined by using users' favorite_product field and products' id field.
    
```javascript
var sql = "SELECT users.name AS user, products.name AS favorite FROM users JOIN products ON users.favorite_product = products.id";
    
con.query(sql, function (err, result) {
if (err) throw err;
console.log(result);
});
```

**2. Left Join**
If you want to return all users, no matter if they have a favorite product or not, use the LEFT JOIN statement:

```javascript
SELECT users.name AS user,
products.name AS favorite
FROM users
LEFT JOIN products ON users.favorite_product = products.id
```


**3. Right Join**
If you want to return **all products**, and the users who have them as their favorite, even if no user have them as their favorite, use the RIGHT JOIN statement:
```javascript
SELECT users.name AS user,
products.name AS favorite
FROM users
RIGHT JOIN products ON users.favorite_product = products.id
```

