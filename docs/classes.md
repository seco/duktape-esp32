#Modules
Built into the solution are a variety of Modules.

* [Globals](#globals)
* [console](#console)
* [DUKF](#dukf)
* [ESP32](#esp32)
* [FS](#fs)
* [HTTP](#http)
* [HTTPClientResponse](#httpclientresponse)
* [HTTPServerRequest](#httpserverrequest)
* [HTTPServerResponse](#httpserverresponse)
* [HTTPParser](#httpparser)
* [net](#net)
* [NVS](#nvs)
* [OS](#os)
* [PARTITIONS](#partitions)
* [RMT](#rmt)
* [Stream](#stream)
* [StreamReader](#streamreader)
* [StreamWriter](#streamwriter)
* [URL](#url)
* [WS](#ws)
* [WebSocketConnection](#websocketconnection)
* [WebSocketServer](#websocketserver)
* [WiFi](#wifi)


##Globals
These are functions that are in the global environment.

###cancelInterval
Cancel an interval timer created by a call to `setInterval()`.
Syntax:
`cancelInterval(id)`

The `id` is a value returned from a previous call to `setInterval()`.  The function associated with the timer will
no longer be fired.  It may not even fire once if the initial interval since it was started is not reached.

###cancelTimeout
Cancel a timer created by a call to `setTimeout()`.
Syntax:
`cancelTimeout(id)`

The `id` is a value returned from a previous call to `setTimeout()`.  If the timer has not expired, the function
associated with the timer will not be called.
 
###setInterval
Set a repeating timer to call a function periodically.

Syntax:
`setTimer(function, interval)`

The `interval` is the period in milliseconds after which the function will be called.  When the function
completes, the timer will be restarted using the same interval.
The `function` is a function which will be invoked when the timer fires.

`setTimer` returns an id which can be used through a call to `cancelTimer()` to cancel any subsequent fires of the timer.

For example:
```
var id = setInterval(function() {
   log("tick!");
}, 1000);
```

###setTimeout
Fire a single shot timer.

Syntax:
`setTimeout(function, delay)`

The `delay` is a time in milliseconds after which the function will be called.  This is a single shot timer and
once the function has executed, it will not be scheduled for re-execution.
The `function` is a function which will be invoked after the delay interval.

`setTimeout` returns an id which can used through a call to `cancelTimeout()` to cancel a timer before
it has fired.

##console
###handler
This property (if set) should be a function with the signature:
```
function(message)
```

It is called to write a console message in place of the default function.

###log
Log a text string to the console.

Syntax:
`console.log(string)`

##DUKF
A general handler for  DUKF related functions.

###debug
Attach the debugger.

###FILE_SYSTEM_ROOT
This is a string property that is the `local` file system root.

###gc
Perform a garbage collection.

Syntax:
`gc()`


###gc
Retrieve the global object

Syntax:
`global()`


###loadFile
Load a text file from the local file system.  If the file can not be found or read
then the result is `null`.

Syntax:
`loadFile(path)`

On the ESP32, this uses ESPFS to load the file so no storage need be released.

For example:
```
var text = DUKF.loadFile("/index.html");
log("About to send: " + text);
```

###OS
A string property that defines the platform we are running upon.  Values are:

* `ESP32`
* `Linux`


###runFile
Run a script loaded from the named file.

Syntax:
`runFile(path)`

##ESP32


###dumpESPFS
Dump the files contained in the ESPFS to the log.

Syntax:
`dumpESPFS()`




###getNativeFunction
Retrieve a built-in native function by name.

Syntax:

`getNativeFunction(name)`

Returns a function reference or null if not found.

###getState
Retrieve the state of the ESP32 device.

Syntax:

`getState()`

This returns an object which contains:
```
{
   heapSize - The size of the free heap.
   sdkVersion - The version of the ESP32 SDK we are running upon.
}
```

For example:
```
setInterval(function() {
   log("Current heap is + " ESP32.getState().heapSize);
}, 2000);
```


###loadFileESPFS
Load a text file from the ESPFS file system.  If the file can not be found or read
then the result is `null`.

Syntax:

`loadFileESPFS(path)`

For example:
```
var text = ESP32.loadFileESPFS("index.html");
log("About to send: " + text);
```

###reset
Reset the state of the JavaScript environment.

Syntax:

`reset()`

###setLogLevel
Set the log level for messages in the JavaScript environment.

Syntax:
`setLogLevel(tag, level)`

The `tag` is the tag on which to set the level.  The special value of `*` means all tags.  The `level`
can be one of the levels to set for that log output.  Choices are:

* `none`
* `error`
* `warn`
* `info`
* `debug`
* `verbose`


For example:
```
ESP32.setLogLevel("*", "verbose");
```

##FS

Example:
```
var fd = FS.openSync("/spiffs/a");
console.log("fd = " + fd);
var buf = new Buffer(500);
var i = FS.readSync(fd, buf, 0, buf.length, 0);
console.log("We read " + i + " bytes: \"" + buf.toString("ascii", 0, i) + "\"");
FS.closeSync(fd);
```

###closeSync
Closes a previously opened file.

Syntax:
`closeSync(fileDescriptor)`

###dump
Dumps a listing of the file system to debug.

Syntax:
`dump()`

###fstatSync
Retrieve details about the file.

`fstatSync(fd)`

The return is an object that contains details of the file:

```
{
   size: <The size of the file in bytes>
}
```


###openSync
Open a file for access.  The return is an integer file descriptor.

Syntax:
`openSync(path, flags)`

* `path` - The path to the file to open.
* `flags` - The flags used to describe how the file should be opened.
  * `r` - Reading. File must exist.
  * `r+` - Reading and writing. File must exist.
  * `w` - Writing. File is created if it does not exist.
  * `w+`- Reading and writing.  File is created if it does not exist.
  * `a` - Appending.  File is created if it does not exist.
  * `a+` - Reading and writing.  File is created if it does not exist.
  
The return is a file descriptor.

For example:

```
var fd = openSync("/spiffs/index.html", "r");
// Do something else ...
closeSync(fd);
```

###readSync
Read data from a file.

Syntax:
`readSync(fd, buffer, offset, length, position)`

 * fd <Integer> - file descriptor
 * buffer <Buffer> - The buffer into which to read data
 * writeOffset <Integer> - The offset into the buffer to start writing
 * maxToRead <Integer> - The maximum number of bytes to read
 * position <Integer> - The position within the file to read from.  If position is null then we read from the current file position.
 
 The return is the number of bytes actually read.
 
 For example:
 
 ```
var fd = FS.openSync("/spiffs/web/ide.html", "r");
var buffer = new Buffer(128);
// Do something else ...
while(1) {
	var sizeRead = FS.readSync(fd, buffer, 0, buffer.length, null);
	log("Size read: " + sizeRead);
	if (sizeRead <= 0) {
		break;
	}
}
FS.closeSync(fd);
 ```
 
###spiffsDir
Returns an array of SPIFFS file system files.

Syntax:
`spiffsDir()`

The result is an array where each element of the array contains:
```
{
   name: <The file name>
   size: <The file size in bytes>
}
```

###statSync
Retrieve details about the file.

Syntax:
`statSync(path)`

The return is an object that contains details of the file:

```
{
   size: <The size of the file in bytes>
}
```

###unlink
Unlink (remove) a file.

Syntax:
`unlink(path)`

The `path` is the file path to the file to be removed.


###writeSync
Write data into a file.

Syntax:
`writeSync(fd, buffer, [offset [, length]])`

Write the buffer into the file specified by the file descriptor.  If an `offset` is supplied, then
write starting at the offset within the buffer.  If `length` is specified, then write the specified number
of bytes into the file.


##GPIO
Own the interface to GPIO.  We construct a new instance of a GPIO to represent the operations
and state of a GPIO pin.  For example:

```
var GPIO = require("gpio");
var pin12 = new GPIO(12);
pin12.setDirection(pin12.OUTPUT);
pin12.setLevel(pin12.HIGH);
```

###getLevel
Read the signal level present as input on the pin. 

###HIGH
A constant property that defines the signal level of `high`.

###INPUT
A constant property that defines the direction as INPUT of a GPIO as used by the
`setDirection()` function.  

###LOW
A constant property that defines the signal level of `low`.

###OUTPUT
A constant property that defines the direction as OUTPUT of a GPIO as used by the
`setDirection()` function. 

###setDirection
Set the direction of the pin.  Either DIRECTION_INPUT or DIRECTION_OUTPUT.

Syntax:
`setDirection(direction)`

###setLevel
Set the signal level present as output on the pin.  The choices should be either
`HIGH` or `LOW`.

Syntax:
`setLevel(level)` 


##HTTP
This class provides the HTTP module.

###request
Make an HTTP request.

Syntax:

`request(options, [callback])`

The options is an object controlling the request being made.  It contains:
* `host` - The address to call (DNS or dotted decimal).
* `method` - The method to pass in the request.  This is optional.  If not supplied then
`GET` is assumed.  Valid values are:
  * GET
  * POST
* `path` - The URL path.  This is optional.  If not supplied then `/` is assumed.
* `port` - The port number.  This is optional.  If not supplied then `80` is assumed.
* `headers` - An object that contains name/value properties.  The name of a property
will be used as an HTTP header name in the outgoing request while its value will
be used as the value of the corresponding header.
 

Example:
```
var http = require("http");

function http_test1() {
	var address = "54.175.219.8"; // httpbin.org
	function logHTTP(obj) {
		log("HTTP Status Code: " + obj.httpStatus);
		log("Headers: " + JSON.stringify(obj.headers));
	}

	http.request({
		address : address,
		port : 80,
		path : "/get"
	}, function(response) {
		log("HTTP Response ready ...");
		response.on("data", function(data) {
			log("**************")
			log("Response data:");
			logHTTP(response);
			log(data);
			log("**************")

		});
		response.on("end", function() {
			log("**********************")
			log("*** Response complete");
			logHTTP(response);
			log("**********************")
		})
	});
}

http_test1();

```

###createServer
Create an HTTP server.
Syntax:
`createServer(requestHandler)`

The `requestHandler` is a callback function that will be invoked when ever there is an incoming
HTTP request.  The signature of the `requestHandler` is:

`function(request, response)`

For example:

```
var HTTP = require("http");
function requestHandler(request, response) {
   log("We have received a new HTTP client request!");
   request.on("data", function(data) {
      log("HTTP Request handler: " + data);
   });
   request.on("end", function() {
      log("HTTP request received:");
      log(" - path: " + request.path);
      log(" - method: " + request.method);
      log(" - headers: " + JSON.stringify(request.headers));
   });
}
var server = HTTP.createServer(requestHandler);
server.listen(80);
```

##HTTPClientResponse
The HTTPClientResponse object is not created directly.  Instead, it is returned by a call to
`HTTP.request()` and represents access to the response sent by that request.

On the event callbacks registered with the `on()` method, properties will be presents on the
`HTTPClientResponse` object that are populated if present.  These include:

* `httpStatus` - The HTTP status code returned from the HTTP request.
* `headers` - An object with name/value properties corresponding to the returned header items.

###on
Register callback handlers for events.  The event types that can be registered include:

* `data` - Any data that may have been sent by the responder.
* `end` - The response has been fully retrieved.


##HTTPServerRequest
The HTTPServerRequest object is not created directly.  Instead it is passed as the first parameter
to an HTTP request handler.  Its purpose is to provide the data supplied by the HTTP requester.

###getHeader
Return the value of a given named header if it is present.

Syntax:
`getHeader(name)`

Returns the String value of the named header or `null` if it is not present.

###getSocket
Return the raw, underlying socket object that represents the incoming connection from the partner.
This is not expected to be commonly used.  In fact, the only known consumer of this is HTTP WebSockets
where an HTTP request is "converted" to a socket connection.  Use with great care and only if you know
what you are doing.

Syntax:
`getSocket()`

###headers
A property object that contains the headers supplied by the requester.  One would normally use `getHeader()` to obtain
the current value of a header property.

###method
A property describing the method supplied by the requester.  Common examples are:
* `GET`
* `POST`
* `PUT`
* `DELETE`

###on
A callback event registration handler.
Syntax:
`on(eventType, callback)`

The eventType can be one of:
* `data` - An indication that new data is available.
* `end` - An indication that the request has arrived in total.


###path
The Path part of the incoming request.

##HTTPServerResponse
The HTTPResponse object is not created directly.  Instead it is passed as the second parameter
to an HTTP request handler.  Its purpose in life is to send responses back to a connected HTTP partner.

Example:
```
response.writeHead(200);
response.write("Hello world!");
response.end();
```

###end
Indicate that the response data is finalized.

Syntax:
`end()`

###write
Write data to the partner.  We can supply either a String or a buffer.

Syntax:
`write(data)`

###writeHead
Write the HTTP headers sent in the response.

Syntax:
`writeHead(statusCode [,statusMessage][,headers])`

The status code is the HTTP code returned.
The `statusMessage` is an optional string that specifies the message sent back with the status code.
The `headers` is an optional object where the name/value pairs will be used as the response headers.

##HTTPParser
Primarily designed as an internal module, this module parses HTTP protocol requests and responses.  When we are
acting as a Web Server, we expected to receive incoming HTTP requests that are asking to retrieve or put data.  When
we are acting as an HTTP client, we are expecting to receive response messages.  Both are HTTP protocol
transmissions and are expected to come over sockets.  This class parses that data.

We create a new instance of an HTTPParser passing in the type of parsing requested and a handler to be
invoked as the parsing progresses.  The instance object returned from creating a new instance an HTTPParser is a stream writer and we
write in HTTP protocol data arriving over the network as it arrives.  When we create a parser instance we supply
the type of the parser can be one of:

* `HTTPParser.REQUEST` - We are parsing an HTTP request.
* `HTTPParser.RESPONSE` - We are parsing an HTTP response.

The handler is passed a stream reader from which the parsed HTTP protocol can be obtained.  Any data passed will
to the reader will be just the content
of any payload for a POST or PUT request.  Status and headers will not be presented in the data stream.
  
The stream object has properties added to it:

For a request:
* `method` - The HTTP method in the request (eg. `GET`, `POST`, etc).
* `path` - The local URL path (eg. "/" or "/dir/myfile.html").
* `headers` - An object with name/value pairs for each of the headers received.

For a response:
* `status` - The HTTP status code (eg. 200).
* `headers`  - An object with name/value pairs for each of the headers received.

To be clear, when we create an instance of an HTTPParser, we must pass in a callback function
with the signature:

`function(parserStreamReader)`

where `parserStreamReader` is a reader stream and hence has events on it such as `data` and `end`.

For example:

```
var HTTPParser = require("httpparser");
var parserStreamWriter = new HTTPParser(HTTPParser.REQUEST, function(parserStreamReader) {
   // We now have a parser stream reader which includes on("data") and on("end").
});
```

## net
The net module owns the lowest level networking components.

###Socket
Create a new socket.
Syntax:
`new net.Socket([options])`

The optional `options` object can contain:
```
{
   sockfd: <A socket numeric descriptor of an existing socket>
}
```

The creation of a new object is an object with methods:

* on - Register an event handler
 * `close`
 * `data`
 * `end`
* `write` - Write data to a target.
* `end([data])` - End the connection optionally sending some final data.

###createServer
Returns a `SocketServer` object instance.

Syntax:
`createServer(connectionListener)`

The `connectionListener` is a callback function that will be invoked when a new client connects
to our server socket.  The callback function will be passed the new Socket instance that relates to the partner.

For example:
```
var net = require("net");
function connectionListener(sock) {
   log("connectionListener: We have received a new connection!");
   sock.on("data", function(data) {
      log("connectionListener: Data received was: " + data);
   });
   sock.on("end", function() {
      log("connectionListener: connection ended!");
   });
}
var server = net.createServer(connectionListener);
server.listen(8888);
```

###connect
Connect to a partner socket server.

Syntax:
`connect(options, connectionListener)`

The `options` is an object that must contain:
```
{
   address: <IP address of target>
   port: <Port number of target>
}
```

The `connectionListener` is a callback function that will be invoked when a a connections has been formed to
our partner server socket.  The callback function will be passed the socket.

For example:

```
var net = require("net");
var socket = new net.Socket();
function connectionListener(sock) {
   log("connectionListener: The connection has been formed!");
   sock.on("data", function(data) {
      log("connectionListener: Data received was: " + data);
   });
   sock.on("end", function() {
      log("connectionListener: connection ended!");
   });   
}
socket.connect({address: "127.0.0.1", port: 99}, connectionListener);
```

###getByName
Retrieve the IP address for the supplied DNS address.

Syntax:
`getByName(name)`

The name may also be supplied as a dotted decimal value.  The value `null` is returned on error.

###SocketServer
This class is returned from createServer.  It is responsible for owning a server socket that is listening for
new incoming connection requests.  It has the following methods:

* `listen(port)` - Start listening on the specified port.



##NVS

For example:

```
var NVS = require("nvs");
var nvsNamespace = NVS.open("neil", "readwrite");
try {
    var val = nvsNamespace.get("i1", "int");
    log("Value of i1 is " + val);
} catch(e) {
    log(e);
}


nvsNamespace.set("i1", 9876, "int");
try {
    var val = nvsNamespace.get("i2", "int");
    log("Value of i2 is " + val);
}
catch(e) {
    log(e);
}
nvsNamespace.close();
```


###open
Open the given namespace.

Syntax:
`var nvsNamespace = open(namespace, mode)`

The mode can be either:

* `readonly`
* `readwrite`

##nvsNamespace

###close
Close a previously opened namespace.

Syntax:
`close()`

###commit
Commit changes made to the store.

Syntax:
`commit()`

###erase
Erase a named key.

Syntax:
`erase(key)`

###easeAll
Erase all keys.

Syntax:
`eraseAll()`

###get
Get the value of a given key.

Syntax:
`get(key, type)`

The type must be one of:
* `int`
* `blob`
* `string`

For example:
```
var value = namespace.get("name", "string");
```

###set
Set the value of a given key to a value.

Syntax:
`set(key, value, type)`

The type must be one of:
* `int`
* `blob`
* `string`

For example:
```
var bootWiFi_ns = NVS.open("bootwifi", "readwrite");
bootWiFi_ns.set("ssid", formObj.ssid, "string");
bootWiFi_ns.close();
```

##OS

###accept
Accept a connection from an incoming client request.

Syntax:
`accept(options)`

The `options` is an object that contains:

```
{
   sockfd: <the file descriptor of the existing socket>
}

```

The return is a an object that contains:
```
{
   sockfd: <the file descriptor of the partner socket>
}

```

###bind
Bind a socket to a local address.  The return is the return code of the underlying bind.  This will
be zero on success.

Syntax:
`bind(options)`

The `options` is an object that contains:

```
{
   sockfd: <the file descriptor of the existing socket>
   port: <The local port number to bind against>
}

```

###close
Close a socket.

Syntax:
`close(options)`

The `options` is an object that contains:

```
{
   sockfd: <the file descriptor of the existing socket>
}

```

There is no return code from this function.


###connect
Connect to a remote network partner.

Syntax:
`connect(options)`

The `options` is an object that contains:

```
{
   sockfd: <the file descriptor of the existing socket>
   address: <The IP address as a dotted decimal string to which to connect>
   port: <The port number to which to connect>
}

```

###gethostbyname
###gpioGetLevel
###gpioInit
###gpioSetDirection
###gpioSetLevel


###listen
Listen on a server socket for incoming client requests.

Syntax:
`listen(options)`

The `options` is an object that contains:

```
{
   sockfd: <the file descriptor of the existing socket>
}

```

###recv
Receive data from a socket.

Syntax:
`recv(options)`

The `options` is an object that contains:

```
{
   sockfd: <the file descriptor of the existing socket>
   data: <The data buffer into which the received data will be stored>
}
```

The return is the length of the data actually received.


###select
Select readiness of an array of sockets.

Syntax:
`select(options)`

The `options` is an object that contains:

```
{
   readfds: <An array of file descriptors to examine for their ability to read>
   writefds: <An array of file descriptors to examine for their ability to write>
   exceptfds: <An array of file descriptors to examine for their notification of an exception>
}
``` 

The return is an object that contains:
```
{
   readfds: <An array of file descriptors that are ready to read>
   writefds: <An array of file descriptors that are ready to write>
   exceptfds: <An array of file descriptors that have exceptions>
}
``` 

###send
Send data down a socket.

Syntax:
`send(options)`

The `options` is an object that contains:

```
{
   sockfd: <the file descriptor of the existing socket>
   data: <The data to send.  Either a buffer or a string>.
}
```

The return is the response code from `send()` at the OS level.


###socket
Create a new socket.

Syntax:
`socket()`

Create a new socket.  The return is an object which contains:
```
{
   sockfd: <the file descriptor for the new socket>
}
```

##PARTITIONS
The `PARTITIONS` object provides access to the ESP32 partition table.

###list
Retrieve the details of a partition table type.

Syntax:
`list(type)`

The `type` is the type of partition table set to return.  The choices are "app" and "data".  The return value
is an array of object where each object contains:

```
{
   "type":    <String> - Type of partition.
   "subtype": <String> - Subtype of partition.
   "label":   <String> - Label of partition.
   "address": <number> - Address in flash of partition start.
   "size":    <number> - Length of partition (bytes).
}
``` 


##RMT
The RMT module is a built-in and is available through "RMT".

The ESP32 has hardware support for a "Remote Control Peripheral".  At first this may seem like an odd choice.
The purpose of this is to control an infrared transmitter or receiver and modulate the transmission of 
the LED being on or off for given pulse durations.  This coding is what is then understood by the partner
and decoded as an instruction.  For example, when pick up your TV remote and click to power on your TV,
a series of infrared LED pulses are transmitted which are decoded by your TV to mean that it should turn on.
Breaking this down, we then understand this to be a series of signal highs and lows for very specific durations
and THAT is the core of RMT.  We can supply a sequence train of pulse durations that correspond to 1s and 0s
that then manifest themselves as GPIO outputs.  Since this is all done at the hardware level as opposed to
the software level, it can be achieved at rates far faster than software control.

###getState
Retrieve the state of a give channel.

Syntax:

`getState(channel)`

An object is returned which contains the state of the channel as understood by the ESP32:
```
{
   channel: <number> - The channel id.
   loopEnabled: <boolean> - Whether or not the TX loop is enabled.
   clockDiv: <number> - The clock divisor.
   memBlocks: <number> - The memory blocks being used.
   idleLevel: <number> - The idle level.
   memoryOwner: <String> - The memory owner "TX" or "RX"
}
```

###txConfig
Configure a RMT channel for transmission.

Syntax:

`txConfig(channel, options)`

The options is an object with the following properties:

```
{
   gpio: <number> - GPIO pin to use.
   memBlocks: <number> - Memory blocks to use. [Default=1]
   idleLevel: <boolean> - Idle value to use. [Default=false]
   clockDiv: <number> - Clock divider. [Default=1]
 }
 ```
 
###write
Write items into the output stream.  We wait until the output has completed.

Syntax:
 
`write(channel, arrayOfItems)`
 
The arrayOfItems is an array of item objects where each item object contains
```
{
   level: <boolean> - The output signal level.
   duration: <number> - The duration of this signal.
}
```


##Stream
A stream is a pair of object where one acts as a stream writer and the other as a stream reader.  When
data is writer into the stream writer it is available at the stream reader.  The stream can also
buffer data such that if the writer wants to write data but the reader isn't ready, the data can
be buffered in the steam until the reader is ready.  The model also allows for loose coupling between
components as if one component has a reader and reads data from it, it doesn't usually matter who
the writer of the data may actually be.

A new instance of this class returns an object that contains a stream connected pair.  The object
looks like:

```
{
   reader: <A reader object>
   writer: <A writer object>
}
```

##StreamReader
This object can not be manually created but is instead created as a result of creating a new
Stream instance.  The object provides a source of input data.

###on
Register an event handler.  The events that can be registered are:
* data - Called when new data is available.  The signature of the callback function is `function(data)` where `data` is a `Buffer`.
* end - Called when the stream has ended and no further data will be received.

Syntax:
`on(eventType, callback)`

###read
Read data if data is available otherwise just return.

Syntax:
`read()`
  
##StreamWriter
This object can not be manually created but is instead created as a result of creating a new
Stream instance.  The object provides a target of output data.

###end
Marks the stream as complete.

Syntax:
`end([data]])`

The `data` is optional and is any final data that should be written before closing the stream.

###write
Write data down the stream.

Syntax:
`write(data)`

The `data` is the data to be written down the stream.  It may be a `Buffer` or a `String`. 

##URL
This is a module which provides URL processing.  It must be required,

```
var URL = require("url");
```

###parse
Parse a URL string into its constituent pieces.

Syntax:
`parse(urlString)`

The return is an object:

```
{
   host:     <The host part of the URL>
   hostname: <The hostname part of the URL; no port number>
   href:     <The original URL>
   pathname: <The pathname of the URL>
   port:     <The port number>
   protocol: <The protocol>
   query:    <An object with the query parts broken out>
   search:   <The search part of the URL including the ?>
}
```	 

###queryParse
Parse a URL query part of the form:
`name=value&name=value&...&name=value`

and return an object that contains the name/value pairs as properties.

Syntax:
`queryParse(queryString)`


##WS
This module provides WebSocket support.

For example:
```
var PORT = 8002;
var ws = require("ws.js");
var webSocketServer = ws.Server();

webSocketServer.on("connection", function(wsConnection) {
	log("We have received a new WebSocket connection");
	wsConnection.on("message", function(data) {
		log("We have received an incoming message: " + data);
		wsConnection.send("Hello back");
		wsConnection.close();
	});
});
webSocketServer.listen(PORT);
log("Being a WebSocket server on port " + PORT);
```

###Server
Create a web socket server instance.

Syntax:
`Server()`

The return from the function is a WebSocket server instance.

##WebSocketConnection
This object is not created manually but is instead passed to a WebSocketServer
`on("connection")` handler.

###close
Close the connection to the partner.
Syntax:
`close()`

###on
Register an event handler.  The event handlers are:
* `close` - Called when the connection is closed.
* `message` - A frame received from a WebSocket partner.

For example:
```
connection.on("close", function() {
   // do something when the connection closes.
});
```

###path
The path of the websocket connection.

###send
Send a message to the partner.

Syntax:
`send(data)`

##WebSocketServer
This object is not created manually but is instead created by calling
`WS.Server()`.

###listen
Start listening on a given port number.

Syntax:
`listen(port)`

###on
Register an event handler.  The event handlers are:

* `connection` - Called when there is a new connection from a 
WebSocket client.  A WebSocketConnection object is passed.

##WIFI
Thus module provides WiFi processing.  It is built in and should not be explicitly required.

* connect - Connect to an access point.
* disconnect - Disconnect from an access point.
* getDNS - Get the DNS settings.
* getState - Get the current WiFi state.
* listen - Start being an access point.
* scan - Scan for access points.

###connect
Connect to an access point.

Syntax:
`connect(options, callback)`

The options is an object that declares how we should connect to the access point.  It contains:
```
{
   ssid: <SSID of access point to connect to>
   password: <Password to connect to access point> [optional]
   network: { <Specific network parameters for connection> [optional]
      ip: <IP Address to assign to the ESP32>
      gw: <Gateway to use to route IP traffic>
      netmask: <Netmask for network>
   }
}
```

The `password` is optional assuming that the access point doesn't require one.

For example:

```
WIFI.connect({
	ssid: "RASPI3",
	password: "password",
	network: {
		ip: "192.168.1.211",
		gw: "192.168.1.1",
		netmask: "255.255.255.0"
	}
}, function() {
	log("### We are now connected!");
});
```


###disconnect
Disconnect from the currently connected access point when we are connected as a station.

Syntax:

`disconnect()`


###getDNS
Retrieve an array of the IP addresses (as strings) that represent the DNS servers that we may know about.

Syntax:

`getDNS()`


###getState
Retrieve the current state of the WiFi environment.

Syntax:
`getState()`

This returns an object which contains:
```
{
   isStation - True if we are being a station and false otherwise.
   isAcessPoint - True if we are being an access point and false otherwise.
   apMac - The MAC address of our access point interface.
   staMac - The MAC address our station interface.
   country - The country code in effect.
   apIP - The IP Address of the access point interface.
   staIP - The IP Address of the station point interface.   
}
```


###listen
Start being an access point.

Syntax:
`listen(options, callback)`

The `options` is an object that declares how we are going to be an access point.  It contains
```
{
   ssid:     <The ssid of the access point we will advertize>
   auth:     <The authentication mechanism used to connect by clients>
   password: <The password to connect to the access point used by clients> [optional if auth is "open"]
}
```

The `auth` must be one of:

* open
* wep
* wpa
* wpa2
* wap_wpa2


###scan
Perform an access point scan and invoke a callback when complete.

Syntax:

`scan(callback)`

The `callback` is an array of objects where each object contains:

```
{
   ssid - The SSID of the network.
   mac  - The MAC address of the access point.  Format is xx:xx:xx:xx:xx:xx.
   rssi - Signal strength.
   auth - One of "open", "wep", "wpa", "wpa2", "wpa_wpa2"
}
```

Example:

```
var wifi = require("WiFi");
wifi.scan(function(list) {
   log(JSON.stringify(list));
});
```

###start
Perform a start of the WiFi subsystem.

Syntax:
`start()`

##stop
Perform a stop of the WiFi subsystem.

Syntax:
`stop()`