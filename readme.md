
**Transporting Data Over the Web**

I am currently a student taking General Assembly’s Programming Bootcamp in DC and recently created an application for one of our projects using a lot Ajax calls to our backend and a third Party API in order to create a real-time, single page application. I had very little exposure to Ajax or Single Page Applications prior to starting this project and wanted to give a broad overview for any other fellow beginners starting out as well!

In this blog post I will be discussing a few different ways of pushing data over the web between the client and the server, and then implement a simple messaging chat application using Nodejs, Express, and Socket.io.

**Summary of HTTP**

Throughout my time during GA's web immersive, we have especially focused on creating RESTful applications, focused on HTTP requests. 

To further explain, HTTP is a transfer protocol for transmitting data over the web. For example, when a user wants to make a purchase off Amazon and types in and submits the URL http://www.amazon.com/, an HTTP request is sent to a web server.  HTTP is considered to be a "stateless protocol", but what does that mean exactly?

A Web (HTTP) server does not remember previous requests from a user, and treats each request independently. There is no recorded continuity, and the server does not retain session information. 

Because of this, applications must implement another layer on top of HTTP to establish state, which is typically accomplished using session cookies. Websites maintain record of users by cookies, which are passed back and fourth for every request. Furthermore, all communication is initiated one-way from the client. 

**What is Ajax?**

Ajax, stands for Asynchronous JavaScript and XML, and consists of small exchanges of data from the client to the the server asynchronously, so that the entire web page does not have to be reloaded each time the user makes a change. As developers we want a way to make requests to the server from the client side without a full page refresh, utlimately to make our applications faster and improve user experience. 

But what if the core goal of our application is to provide continous real time updates? What if we are creating an application that displays real time sports and game updates?

We could use Ajax and “polling” to accomplish this by sending periodic requests for updates to the server, and dynamically manipulating the DOM to reflect those changes. This would essentially create a similar connection to server like Ajax does, but keeps it open for a period of time in which the client can receive data from server. 

Luckliy we have many options as developers like strong Front-End Frameworks such as Angular which help making Ajax requests alot easier. However, ultimately this seems like an efficient way to grab changes from our server. One answer to this problem is are WebSockets, made available through HTML5, which allow an open connection to persist between the client and server in which both can send data at any time. 

**What are Web Sockets?**

WebSockets are a TCP based Protocol application that establishes a bi-directional connection between the client and server, creating full duplex, persistent messages to be instantously distributed. Once a connection is established, it stays open as long as needed. Ajax requests are one way, always initiated by the client. When the server has sent the response, the connection will close. However with WebSockets this not the case and it does not even require the user to make a request in order for the client to recieve updates from the server. 

In this tutorial, we are going to be using one API available, Socket.IO which allows us to easily establish a WebSocket in our application. We are going to be using Nodejs and Expressjs along side Socket.Io to create a simple chat messaging application. 

First please make sure you have Node installed on your computer. If you do not, you will need to install it first before continuing https://nodejs.org/en/. 

Let's create a new directory on our local computers and initialize npm. If your new to Node or npm, feel free to just hit enter for all of the metadata information, as we can edit it in the package.json file after. 

```
$ npm init
```
After we have initlized npm, let's make sure we install and save Express as a depedency. 
```
$ npm install express --save
```
Now that we have express installed, saved and added to our package.json file, we can begin by creating the files an folders we will need for this application. We will create need to create a main js file, which we will call index.js, which will define and establish all of our dependencies for our express application.
```
$ touch index.js
```
Inside index.js we need to include the following:


```js
var express = require("express");
var app = express();
var http = require('http').Server(app);

app.get('/', function(req, res){
  res.send('<h1>Hello world</h1>');
});

http.listen(4000, function(){
  console.log('listening on port 4000');
});

```

To run express from your command line, just type node index.js or use nodemon and you should see listing on port 4000 display on your terminal. Go to your browser and enter in localhost:4000 and you should see hello world displaying as well. 

Great! Now that everything is configured, let’s instead create a static index.html file that will serve up a html document instead of passing in the string for the route. We can do this through the following commands below. 

For the purpose of this application, we are going to put all of our static file into a public directory which will contain all of our static files including our script.js file and stylesheet. 
```
$ mkdir public
$ cd public
$ touch index.html
$ mkdir js
$ touch js/script.js
$ mkdir css
$ touch css/style.css
```
In order to render the index.html we need to call Express Middleware to look into the public directory:

```js
app.use(express.static(__dirname + '/public'));

```


In our main index.html file we need to add the following and make sure to link our stylesheet and script files:

```html

<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Chat Message App</title>
    <script src="js/script.js"></script>
    <link rel="stylesheet" href="css/style.css" type="text/css">
  </head>
  <body>
    <ul class="messages"></ul>
    <form action="">
      <input id="msg" autocomplete="off" /><button>Send</button>
    </form>
  </body>
</html>
```
And in our style.css file:

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  font: 13px Helvetica, Arial;
}
form {
  background: #000;
  padding: 3px;
  position: fixed;
  bottom: 0;
  width: 100%;
}
form input {
  border: 0;
  padding: 10px;
  width: 90%;
  margin-right: .5%;
}
form button {
  width: 9%;
  background: rgb(130, 224, 255);
  border: none;
  padding: 10px;
}
.messages {
  list-style-type: none;
  margin: 0;
  padding: 0;
}
.messages li {
  padding: 5px 10px;
}
.messages li:nth-child(odd) {
  background: #eee;
}
```

Now that we have the basic font end files set up, it's time to config Socket.io. First, we need to install and save Socket.io to our package.json file. 
```
$ npm install socket.io --save 
```

Socket.io is composed of two parts on both the server and client side. In our main application index.js, we need to require it:

```js
var io = require("socket.io")(http);

io.on("connection", function(socket){
  console.log("Socket.io has been connected!");
});

```

And in our index.html file we need to add both jQuery and Socket.io script tags:

```html
<script src="/socket.io/socket.io.js"></script>
<script src="http://code.jquery.com/jquery-1.11.1.js"></script>
<script src="js/script.js"></script>
```
Finally, in our script.js, we need to declare a socket variable:
```js

$(document).ready(function(){
  var socket = io(); 
});
```
