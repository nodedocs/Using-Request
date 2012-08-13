# Using mikeal/request

Node has an easy way of performing HTTP requests, but if you need to encode / decode data to / from the request or response bodies or follow redirects, the Node Core API won't help you. For this and other types of common tasks, mikeal/request has become the defacto standard in Node for making HTTP requests.

You can install mikeal/request locally by doing:

    $ npm install request

Then you can import the module:

```javascript
var request = require('request');
````

And use it to make requests. But first let's code this test HTTP server:

```javascript
require('http').createServer(function(req, res) {

function printBack() {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end(JSON.stringify({
      url: req.url,
      method: req.method,
      headers: req.headers
    }));
}

switch (req.url) {

    case '/redirect':
      res.writeHead(301, {"Location": '/'});
      res.end();
      break;
    
    case '/print/body':
      req.setEncoding('utf8');
      var body = '';
      req.on('data', function(d) {
        body += d;
      });
      req.on('end', function() {
        res.end(JSON.stringify(body));
      });
      break;
    
    default:
      printBack();
      break;
    
    }
}).listen(4001);
```

If we save this snnippet to file `server.js` and run it:

    $ node server.js

We can then make a simple call to this server:

```javascript
var request = require('request');
var inspect = require('util').inspect;

request(
  'http://localhost:4001/abc/def',
  function(err, res, body) {
    if (err) { throw err; }
    console.log(inspect({
      err: err,
      res: {
        statusCode: res.statusCode
      },
      body: body
    }))
  });
```

**simple.js**

We can also observe that request follows redirects:

```javascript
var request = require('request');
var inspect = require('util').inspect;

request(
  'http://localhost:4001/redirect',
  function(err, res, body) {
    if (err) { throw err; }
    console.log(inspect({
      err: err,
      res: {
        statusCode: res.statusCode
      },
      body: body
    }))
  });
```

You can perform HTTP methods other than GET. You can use the following shortcuts:

* request.get
* request.put
* request.post
* request.del

Here is an example using a POST request:

```javascript
var request = require('request');
var inspect = require('util').inspect;

request.post(
  'http://localhost:4001/abc/def',
  function(err, res, body) {
    if (err) { throw err; }
    console.log(inspect({
      err: err,
      res: {
        statusCode: res.statusCode
      },
      body: body
    }))
  });
```

Or, more generically, request accepts an options object instead of a string URL:

```javascript
var request = require('request');
var inspect = require('util').inspect;

var options = {
  url: 'http://localhost:4001/abc/def',
  method: 'PUT'
};

request(options, function(err, res, body) {
  if (err) { throw err; }
  console.log(inspect({
    err: err,
    res: {
      statusCode: res.statusCode
    },
    body: body
  }))
});
```

You can also send some custom headers:

```javascript
var request = require('request');
var inspect = require('util').inspect;

var options = {
  url: 'http://localhost:4001/abc/def',
  method: 'PUT',
  headers: {
    'X-My-Header': 'value'
  }
};

request(options, function(err, res, body) {
  if (err) { throw err; }
  console.log(inspect({
    err: err,
    res: {
      statusCode: res.statusCode,
      headers: res.headers
    },
    body: JSON.parse(body)
  }))
});
```

You can also encode the body. You can use form-encoding:

```javascript
var request = require('request');
var inspect = require('util').inspect;

body = {
  a: 1,
  b: 2
};

var options = {
  url: 'http://localhost:4001/print/body',
  form: body
};

request(options, function(err, res, body) {
  if (err) { throw err; }
  console.log(inspect({
    err: err,
    res: {
      statusCode: res.statusCode,
      headers: res.headers
    },
    body: JSON.parse(body)
  }))
});
```

Or JSON encoding:

```javascript
var request = require('request');
var inspect = require('util').inspect;

body = {
  a: 1,
  b: 2
};

var options = {
  url: 'http://localhost:4001/print/body',
  json: body
};

request(options, function(err, res, body) {
  if (err) { throw err; }
  console.log(inspect({
    err: err,
    res: {
      statusCode: res.statusCode,
      headers: res.headers
    },
    body: JSON.parse(body)
  }))
});
```

You can also send a query string in the URL:

```javascript
var request = require('request');
var inspect = require('util').inspect;

body = {
  a: 1,
  b: 2
};

var options = {
  url: 'http://localhost:4001/print/body',
  qs: body
};

request(options, function(err, res, body) {
  if (err) { throw err; }
  console.log(inspect({
    err: err,
    res: {
      statusCode: res.statusCode,
      headers: res.headers
    },
    body: JSON.parse(body)
  }))
});
```

## Buffering and Events

If you provide a callback, request will buffer the response body. If you don't want that to happen, you can attach to the returned event emitter like this:

```javascript
var request = require('request');

var req = request('http://localhost:4001/');

req.on('response', function(res) {
  console.log('we have a response', {res: {
    statusCode: res.statusCode,
    headers: res.headers
  }});

  res.setEncoding('utf8');

  res.on('data', function(data) {
    console.log('response data:', data);
  });

  res.on('end', function() {
    console.log('response ended');
  });

});
```

## Piping

A request object is a duplex stream, which means you can pipe in data or pipe it out.

For instance, you can proxy a request to an HTTP Serer response like this:

```javascript
var request = require('request');

require('http').createServer(function(req, res) {
  request('https://www.google.pt/images/srpr/logo3w.png').pipe(res);
}).listen(4001);
```

You can save this into a file named "reverse_proxy.js" and then point your browser to http://localhost:4001/.

You can also inspect the headers of the response by using curl:

    $ curl -I http://localhost:4001/

You should see something like this:

    HTTP/1.1 200 OK
    content-type: image/png
    last-modified: Mon, 02 Apr 2012 02:13:37 GMT
    date: Thu, 03 May 2012 14:46:03 GMT
    expires: Thu, 03 May 2012 14:46:03 GMT
    cache-control: private, max-age=31536000
    x-content-type-options: nosniff
    server: sffe
    content-length: 7007
    x-xss-protection: 1; mode=block
    Connection: keep-alive

All of these headers are injected by request based on the response from the Google server. Request is pipe-aware and transfers the headers from one response to another.

You can also pipe into a request:

```javascript
var request = require('request');

var target = request.put('http://localhost:4001/');

request('https://www.google.pt/images/srpr/logo3w.png')
  .pipe(target)
  .pipe(process.stdout);
```

This pipes the google doodle into our server and pipes the response into out process standard output.

## Agent

Node reutilizes the TCP connections to the same server by keeping a connection pool. By default this connection pool has a default maximum of 5 open connections for any given (hostname, port) pair. After that the requests get queued up waiting for an available socket.

Let's test this effect by having a server that takes one second to respond:

```javascript
var delay = process.argv[2] && parseInt(process.argv[2], 10) || 1000;

var pendingRequests = 0;

require('http').createServer(function(req, res) {
  pendingRequests ++;
  console.log('have %d pending requests', pendingRequests);
  setTimeout(function() {
    res.end();
    pendingRequests --;
  }, delay);
}).listen(4001);
```

Save the server code into a file named "delayed_server.js" and start it up:

    $ node delayed_server.js

Then we are ready to call our delayed server:

```javascript
var request = require('request');

var maxRequests = 100;

for( var i = 0; i < maxRequests; i ++) {
  request('http://localhost:4001/', function(err) {
    if (err) { throw err; }
  });
}
```

This script launches 100 parallel requests into our server. If you fire this up you will notice that our server only has at most 5 pending requests at a time.

You can change this by changing the request.pool.maxSockets variable. Let's change it to 10:

```javascript
var request = require('request');

var maxRequests = 100;

for( var i = 0; i < maxRequests; i ++) {
  var req = request('http://localhost:4001/', function(err) {
    if (err) { throw err; }
  });
  req.pool.maxSockets = 10;
}
```

## Cookies

Request also keeps the cookies in between requests in a global cookie jar. If you want to you can provide a specific cookie jar to be used on a request by passing in a `.jar` property like this:

```javascript
var request = require('request');

var cookieJar = request.jar();

var options = {
  url: 'http://google.com',
  jar: cookieJar
};

request(options, callback);
```

If you want ot turn off the cookie memory you can pass `jar: false` as an option.

## Summary

Mikeal/request is a really powerful HTTP client that lets you easily do most of the things you may want to do with a typical HTTP request.
