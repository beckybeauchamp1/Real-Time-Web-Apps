
**Transporting Data Over the Web**

I am currently a student taking General Assembly’s Programming Bootcamp in DC and recently created an application for one of our projects using a lot Ajax calls to our backend and a third Party API in order to create a real-time, single page application. I had very little exposure to Ajax or Single Page Applications prior to starting this project and wanted to give a broad overview for any other fellow beginners starting out as well!

In this blog post I will be discussing a few different ways of pushing data over the web between the client and the server, and then implement a simple messaging chat application using Nodejs, Express, and Socket.io.

Let's start by disscuing the foundation for all data communications over the web, the HTTP protocol.

**Summary of HTTP**

Throughout my time during GA's web immersive, we have especially focused on creating RESTful applications, focused on HTTP requests. 

To further explain, HTTP is a protocol for transmitting data over the web. For example, when a user wants to make a purchase off Amazon and types in and submits the URL http://www.amazon.com/, an HTTP request is sent to a web server. 

HTTP behaves as a request-response protocol in the client-server model. The client(the browser in the example above) submits an HTTP request message to the server. The server, may provide HTML files, content, or even perform additional functions on behalf of the client, then returns a response message back. The response will contain the requested information if available in it's message body along with a completion status information. 

However, HTTP is considered to be a "stateless protocol", but what does that mean exactly?

A web (HTTP) server does not remember previous requests from a user, and treats each request independently. There is no recorded continuity, and the server does not retain session information. Because of this, applications must implement another layer on top of HTTP to establish state, which is typically accomplished using session cookies. Websites maintain record of users by cookies, which are passed back and fourth for every request. 

Moreover, in order to send data asynchrously through an HTTP request, developers created the XMLHttpRequest object or jquery's version called Ajax.

**What is Ajax?**

Ajax, stands for Asynchronous JavaScript and XML, and consists of small exchanges of data from the client to the the server, so that the entire web page does not have to be reloaded each time the user makes a change. As developers we want a way to make requests to the server from the client side without a full page refresh, utlimately to make our applications faster and improve user experience. 

But what if the core goal of our application is to provide continous real time updates? What if we are creating an application that displays real time sports and game updates?

We could use Ajax and “polling” to accomplish this by sending periodic requests for updates to the server, and dynamically manipulating the DOM to reflect those changes. This would essentially create a connection and keep it open for a period of time in which the client can receive data from server. 

We have many options as developers with strong Front-End Frameworks such as Angular which help making Ajax requests a lot easier or additional APIs to assist in these requests. 

However, this seems like an efficient way to grab changes from our server. One answer to this problem is are WebSockets, made available through HTML5, which allow an open connection to persist between the client and server in which both can send data at any time. 

**What are Web Sockets?**

WebSockets are a TCP based Protocol application that establishes a single, bi-directional connection between the client and server, allowing full duplex, persistent messages to be instantously distributed. Once a connection is established, it stays open as long as needed. Ajax is a one-way request that's always intiated by the client, and when the server has sent the response, the connection will close. However with WebSockets both the client and server can send requests and the connection is kept open. Because WebSockets are built on the TCP protocol, which is a stateful protocol, we can accomplish this connection. 

As you can imagine, this is pretty powerful when we need to create applications that require real-time data. 

Below, we are going to be using one API available, Socket.io which allows us to easily establish a WebSocket in our application. We are going to be using Nodejs and Expressjs along side Socket.io to build a simple chat messaging application. 

First please make sure you have Node installed on your computer. If you do not, you will need to install it first before continuing https://nodejs.org/en/. 

**Chat Messaging App Example**

Let's create a new directory on our local computers and initialize npm. If your new to Node or npm, feel free to just hit enter for all of the metadata questions, as we can edit it later on in the package.json file. 

```
$ npm init
```
After we have initlized npm, let's make sure we install and save Express as a dependency. 

```
$ npm install express --save
```
Now that we have express installed, saved, and added to our package.json file, we can begin by creating the files an folders we will need for this application. We will create need to create a main JavaScript file, which we will call index.js, which will define and establish all of our dependencies for our express application.

```
$ touch index.js
```
In our index.js we need to include the following:

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
To run express from your command line, just type node index.js or use nodemon and you should see listing on port 4000 display on your terminal. If you do not have nodemon install you can easily install it gobally on your computer. This will automatically restart the server each time you make changes:

```
$ npm install -g nodemon
```

Now, go to your browser and enter in localhost:4000 and you should see hello world displaying as well. 

Great! Now that the basics are configured, let’s instead create an index.html file for our html content instead of passing in the string for the route. We can do this through the following commands below. 

For the purpose of this application, we are going to create a public directory which will contain all of our static files including our index.html, script.js, and stylesheet. 

```
$ mkdir public
$ cd public
$ touch index.html
$ mkdir js
$ touch js/script.js
$ mkdir css
$ touch css/style.css
```
In order to render the index.html we need to call Express Middleware to look into the public directory and change our route:

```js
app.use(express.static(__dirname + '/public'));

app.get('/', function(req, res){
  res.render("index.html");
});
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
  font: 16px Helvetica, Arial;
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
Now that we set up our index.html file and stylesheet, it's time to config socket.io. First, we need to install and save socket.io to our package.json file. 
```
$ npm install socket.io --save 
```

In our main application index.js, we need to require socket.io. In this example we are attaching socket.io to an HTTP server listening on port 4000. 

```js
var io = require("socket.io")(http);

io.on("connection", function(socket){
  console.log("Socket.io has been connected!");
});

```

And in our index.html file we need to add both jQuery and socket.io script tags:

```html
<script src="/socket.io/socket.io.js"></script>
<script src="http://code.jquery.com/jquery-1.11.1.js"></script>
<script src="js/script.js"></script>
```
Finally, in our script.js, we need to declare a socket variable to call socket.io:

```js

$(document).ready(function(){
  var socket = io(); 
});
```
Restart your server, and you should see that socket.io has been connected. Now let's actually grab the content of the message input and have append it to the DOM when a user click's send. Add this to our script.js file after we have declared socket as a variable.

```js

$("button").on("click", function(){
    socket.emit("message", $("#msg").val());
    $("#msg").val("");
    return false;
  });

  socket.on("message", function(msg){
    $(".messages").append($("<li></li>").text(msg));
  });
```

Now we have all the code written on the client side, let's update our back end to recieve the message. 

```js

io.on("connection", function(socket){
  socket.on('message', function(msg){
    io.emit("message", msg)
    console.log('message:' + msg);
  });
});
```

With socket.io, .emit method is available which emits an event to the socket identified by the string name to all connected clients. Both of the following synatax would work correctly in our example: 

```js
io.sockets.emit();
io.emit();
```

Also, we are sending "message" consistently on the client and server, but please note you could name this anything including "pizza" and as long as it's consistent, it will work the same. Socket.io allows you to do this and create custom events. 

Finally, at localhost://4000, you sould be able to send messages that appear on your application's webpage. Socket.io's documentation examples additional methods available to continue building your chat messaging application. https://github.com/socketio/socket.io

In Summary, we have a lot of resources available as developers to send data over the web, asynchronously. Front-End Frameworks like Angular help us to more easily make Ajax requests and communicate between client and server. However, as we continue to build more applications that rely on real-time data, it appears WebSockets could be a useful resource in helping us accomplish our domain. 

