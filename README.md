# CRUD with Node Express Js

Step 1. Create the express project with the name yourname:
```
express --view=ejs yourname
```

After successfully creating yourname folder in your system, install nodejs in your project:
```
cd yourname
npm install
```
<br>

Step 2. Install flash,validator,session ,override MySQL Libraries into your node js express crud + MySQL application.
You will install:
express-flash
Is an extension of connect-flash with the ability to define a flash message and render it without redirecting the request.
In this node js mysql crud tutorial express flash is used to display a warning, error and information message
```
npm install express-flash --save
```

express-session
Used to made a session as like in PHP. In this node js mysql crud tutorial, session is needed as the express requirement of express-flash.
```
npm install express-session --save
```

express-validator
Used to validate form data it is easy to use. express-validator highly effective and efficient way to accelerate the creation of applications.
```
npm install express-validator --save
```

method-override
NPM is used to run a DELETE and PUT method from an HTML form. In several web browsers only support GET and POST methods.
```
npm install method-override --save
```

MySQL
Driver to connect node.js with MySQL
```
npm install mysql --save
```
<br>

Step 3. Connect to Node js Express CRUD App
To do so, create one folder name lib and create a new file name db.js inside this folder. We will connect node js to mysql using this file

<strong>lib/db.js</strong>
```
1. var mysql=require('mysql');
2.  var connection=mysql.createConnection({
3.    host:'localhost',
4.    user:'your username',
5.    password:'your password',
6.    database:'your database name'
7.  });
8. connection.connect(function(error){
9.    if(!!error){
10.      console.log(error);
11.    }else{
12.      console.log('Connected!:)');
13.    }
14.  });  
15. module.exports = connection;
```
<br>

Step 4. Create Server.js File
In your app root directory, create a new file name server.js and add:
```
1. var createError = require('http-errors');
2.  var express = require('express');
3.  var path = require('path');
4.  var cookieParser = require('cookie-parser');
5.  var logger = require('morgan');
6.  var expressValidator = require('express-validator');
7.  var flash = require('express-flash');
8.  var session = require('express-session');
9.  var bodyParser = require('body-parser');
10.  
11.  var mysql = require('mysql');
12.  var connection  = require('./lib/db');
13.  
14.  var indexRouter = require('./routes/index');
15.  var usersRouter = require('./routes/users');
16.  var customersRouter = require('./routes/customers');
17.  
18.  var app = express();
19.  
20. // view engine setup
21.  app.set('views', path.join(__dirname, 'views'));
22.  app.set('view engine', 'ejs');
23.  
24.  app.use(logger('dev'));
25.  app.use(bodyParser.json());
26.  app.use(bodyParser.urlencoded({ extended: true }));
27.  app.use(cookieParser());
28.  app.use(express.static(path.join(__dirname, 'public')));
29.  
30.  app.use(session({ 
31.      secret: '123456cat',
32.      resave: false,
33.      saveUninitialized: true,
34.      cookie: { maxAge: 60000 }
35.  }))
36.  
37.  app.use(flash());
38.  app.use(expressValidator());
39.  
40.  app.use('/', indexRouter);
41.  app.use('/users', usersRouter);
42.  app.use('/customers', customersRouter);
43.  
44.  // catch 404 and forward to error handler
45.  app.use(function(req, res, next) {
46.    next(createError(404));
47.  });
48.  
49.  // error handler
50.  app.use(function(err, req, res, next) {
51.    // set locals, only providing error in development
52.    res.locals.message = err.message;
53.    res.locals.error = req.app.get('env') === 'development' ? err : {};
54.  // render the error page
55.    res.status(err.status || 500);
56.    res.render('error');
57.  });
58. // port must be set to 3000 because incoming http requests are routed from port 80 to port 8080
59. app.listen(3000, function () {
60.     console.log('Node app is running on port 3000');
61. });
62.  module.exports = app;
```
<br>

Step 5. Create CRUD Routes
To create crud route file name customers.js, visit inside routes folder and create this file and add:
```
var express = require('express');
var router = express.Router();
var connection  = require('../lib/db');
/* GET home page. */
router.get('/', function(req, res, next) {
connection.query('SELECT * FROM customers ORDER BY id desc',function(err,rows)     {
if(err){
req.flash('error', err); 
res.render('customers',{page_title:"Customers - Node.js",data:''});   
}else{
res.render('customers',{page_title:"Customers - Node.js",data:rows});
}
});
});
// SHOW ADD USER FORM
router.get('/add', function(req, res, next){    
// render to views/user/add.ejs
res.render('customers/add', {
title: 'Add New Customers',
name: '',
email: ''       
})
})
// ADD NEW USER POST ACTION
router.post('/add', function(req, res, next){    
req.assert('name', 'Name is required').notEmpty()           //Validate name
req.assert('email', 'A valid email is required').isEmail()  //Validate email
var errors = req.validationErrors()
if( !errors ) {   //No errors were found.  Passed Validation!
var user = {
name: req.sanitize('name').escape().trim(),
email: req.sanitize('email').escape().trim()
}
connection.query('INSERT INTO customers SET ?', user, function(err, result) {
//if(err) throw err
if (err) {
req.flash('error', err)
// render to views/user/add.ejs
res.render('customers/add', {
title: 'Add New Customer',
name: user.name,
email: user.email                    
})
} else {                
req.flash('success', 'Data added successfully!');
res.redirect('/customers');
}
})
}
else {   //Display errors to user
var error_msg = ''
errors.forEach(function(error) {
error_msg += error.msg + '<br>'
})                
req.flash('error', error_msg)        
/**
* Using req.body.name 
* because req.param('name') is deprecated
*/
res.render('customers/add', { 
title: 'Add New Customer',
name: req.body.name,
email: req.body.email
})
}
})
// SHOW EDIT USER FORM
router.get('/edit/(:id)', function(req, res, next){
connection.query('SELECT * FROM customers WHERE id = ' + req.params.id, function(err, rows, fields) {
if(err) throw err
// if user not found
if (rows.length <= 0) {
req.flash('error', 'Customers not found with id = ' + req.params.id)
res.redirect('/customers')
}
else { // if user found
// render to views/user/edit.ejs template file
res.render('customers/edit', {
title: 'Edit Customer', 
//data: rows[0],
id: rows[0].id,
name: rows[0].name,
email: rows[0].email                    
})
}            
})
})
// EDIT USER POST ACTION
router.post('/update/:id', function(req, res, next) {
req.assert('name', 'Name is required').notEmpty()           //Validate nam           //Validate age
req.assert('email', 'A valid email is required').isEmail()  //Validate email
var errors = req.validationErrors()
if( !errors ) {   
var user = {
name: req.sanitize('name').escape().trim(),
email: req.sanitize('email').escape().trim()
}
connection.query('UPDATE customers SET ? WHERE id = ' + req.params.id, user, function(err, result) {
//if(err) throw err
if (err) {
req.flash('error', err)
// render to views/user/add.ejs
res.render('customers/edit', {
title: 'Edit Customer',
id: req.params.id,
name: req.body.name,
email: req.body.email
})
} else {
req.flash('success', 'Data updated successfully!');
res.redirect('/customers');
}
})
}
else {   //Display errors to user
var error_msg = ''
errors.forEach(function(error) {
error_msg += error.msg + '<br>'
})
req.flash('error', error_msg)
/**
* Using req.body.name 
* because req.param('name') is deprecated
*/
res.render('customers/edit', { 
title: 'Edit Customer',            
id: req.params.id, 
name: req.body.name,
email: req.body.email
})
}
})
// DELETE USER
router.get('/delete/(:id)', function(req, res, next) {
var user = { id: req.params.id }
connection.query('DELETE FROM customers WHERE id = ' + req.params.id, user, function(err, result) {
//if(err) throw err
if (err) {
req.flash('error', err)
// redirect to users list page
res.redirect('/customers')
} else {
req.flash('success', 'Customer deleted successfully! id = ' + req.params.id)
// redirect to users list page
res.redirect('/customers')
}
})
})
module.exports = router;
```
<br>

Step 6. Create Views
Create a folder named customers inside the views folder
Create three views files named add.ejs, edit.ejs and index.ejs in the views/customers folder.
Create first file index.ejs which will display the list of customers:
```
<!DOCTYPE html>
<html>
<head>
<title>Customers</title>
<link rel='stylesheet' href='/stylesheets/style.css' />
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js" integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy" crossorigin="anonymous"></script>
<link href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
</head>
<body>
<div>
<a href="/" class="btn btn-primary ml-3">Home</a>  
<a href="/customers/add" class="btn btn-secondary ml-3">New Customer</a> 
<a href="/customers" class="btn btn-info ml-3">Customer List</a>
</div>    
<!--   <% if (messages.error) { %>
<p style="color:red"><%- messages.error %></p>
<% } %> -->
<% if (messages.success) { %>
<p class="alert alert-success mt-4"><%- messages.success %></p>
<% } %>  
<br>
<table class="table">
<thead>
<tr>
<th scope="col">#</th>
<th scope="col">Name</th>
<th scope="col">Email</th>
<th width="200px">Action</th>
</tr>
</thead>
<tbody>
<% if(data.length){
for(var i = 0; i< data.length; i++) {%>  
<tr>
<th scope="row"><%= (i+1) %></th>
<td><%= data[i].name%></td>
<td><%= data[i].email%></td>
<td>
<a class="btn btn-success edit" href="../customers/edit/<%=data[i].id%>">Edit</a>                       
<a class="btn btn-danger delete" onclick="return alert('Are You sure?')" href="../customers/delete/<%=data[i].id%>">Delete</a>                       
</td>
</tr>
<% }
}else{ %>
<tr>
<td colspan="3">No user</td>
</tr>
<% } %>    
</tbody>
</table>
</body>
</html>
```

Create add.ejs which will create form for sending data to database:
```
<!DOCTYPE html>
<html>
<head>
<title>Customers</title>
<link rel='stylesheet' href='/stylesheets/style.css' />
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js" integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy" crossorigin="anonymous"></script>
<link href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
</head>
<body>
<% if (messages.error) { %>
<p style="color:red"><%- messages.error %></p>
<% } %>
<% if (messages.success) { %>
<p style="color:green"><%- messages.success %></p>
<% } %>
<form action="/customers/add" method="post" name="form1">
<div class="form-group">
<label for="exampleInputPassword1">Name</label>
<input type="text" class="form-control" name="name" id="name" value="" placeholder="Name">
</div>
<div class="form-group">
<label for="exampleInputEmail1">Email address</label>
<input type="email" name="email" class="form-control" id="email" aria-describedby="emailHelp" placeholder="Enter email" value="">
</div>
<input type="submit" class="btn btn-primary" value="Add">
</form>
</body>
</html>
```

Create edit.ejs which will edit data in the form:
```
<!DOCTYPE html>
<html>
<head>
<title>Customers</title>
<link rel='stylesheet' href='/stylesheets/style.css' />
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
<script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js" integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy" crossorigin="anonymous"></script>
<link href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO" crossorigin="anonymous">
</head>
<body>
<form action="/customers/update/<%= id %>" method="post" name="form1">
<div class="form-group">
<label for="exampleInputPassword1">Name</label>
<input type="text" class="form-control" name="name" id="name" value="<%= name %>" placeholder="Name">
</div>
<div class="form-group">
<label for="exampleInputEmail1">Email address</label>
<input type="email" class="form-control" name="email" id="email" aria-describedby="emailHelp" placeholder="Enter email" value="<%= email %>">
</div>
<button type="submit" class="btn btn-info">Update</button>
</form>
</body>
</html>
```
<br>

Step 7. Start Node Express js Crud + MySQL app
To do so, run the command:
```
npm start
```
