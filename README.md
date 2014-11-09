[![build status](https://secure.travis-ci.org/mjackson/mach.png)](http://travis-ci.org/mjackson/mach)

[Mach](https://github.com/mjackson/mach) is an HTTP server and client library that runs in both node.js and the browser. It has the following goals:

  * Simplicity: straightforward mapping of HTTP requests to JavaScript function calls
  * Asynchronous: responses can be deferred using Promises/A+ promises
  * Streaming: request and response bodies can be streamed
  * Composability: middleware composes easily using promises
  * Robustness: promises propagate errors up the call stack, simplifying error handling

### Servers

Writing a "Hello world" HTTP server in Mach is simple.

```js
var mach = require('mach');

mach.serve(function (conn) {
  return "Hello world!";
});
```

Responses can be deferred using JavaScript promises. Simply return a promise from your app that resolves when the response is ready.

```js
var app = mach.stack();

app.use(mach.logger);

app.get('/users/:id', function (conn) {
  var id = conn.params.id;

  return getUser(userID).then(function (user) {
    conn.json(200, user);
  });
});
```

The call to `app.use` above illustrates how middleware is used to compose applications. Mach ships with the following middleware:

- `mach.basicAuth`: Provides authentication using [HTTP Basic auth](http://en.wikipedia.org/wiki/Basic_access_authentication)
- `mach.catch`: Error handling at any position in the stack
- `mach.contentType`: Provides a default [`Content-Type`](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.17)
- `mach.favicon`: Handles requests for `/favicon.ico`
- `mach.file`: Efficiently serves static files
- `mach.gzip`: [Gzip](http://en.wikipedia.org/wiki/Gzip)-encodes response content for clients that `Accept: gzip`
- `mach.logger`: Logs HTTP requests to the console
- `mach.mapper`: Provides virtual host mapping, similar to [Apache's Virtual Hosts](http://httpd.apache.org/docs/2.2/vhosts/) or [nginx server blocks](http://nginx.org/en/docs/http/ngx_http_core_module.html#server)
- `mach.methodOverride`: Overrides the HTTP method used in the request, for clients (like HTML forms) that don't support methods other than `GET` and `POST`
- `mach.modified`: HTTP caching using [`Last-Modified`](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.29) and [`ETag`](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.19)
- `mach.params`: Multipart request parsing and handling
- `mach.proxy`: Proxy request through to an alternate location
- `mach.rewrite`: Rewrites request URLs on the fly, similar to [Apache's mod_rewrite](http://httpd.apache.org/docs/current/mod/mod_rewrite.html)
- `mach.router`: Request routing (ala [Sinatra](http://www.sinatrarb.com/)) based on the URL pathname
- `mach.session`: HTTP sessions with pluggable storage including [memory](#) (for development and testing), [cookies](#), and [Redis](#)
- `mach.stack`: Provides a `use` mechanism for composing applications fronted by middleware
- `mach.token`: Cross-site request forgery protection

Please check out the source of a middleware file for detailed documentation on how to use it.

### Clients

Writing an HTTP client is similarly straightforward.

```js
var mach = require('mach');

mach.get('http://twitter.com').then(function (conn) {
  console.log(conn.status, conn.responseText);
});
```

By default client responses are buffered and stored in the `responseText` connection variable for convenience. However, if you'd like to access the raw stream of binary data in the response, you can use the `binary` flag.

```js
var fs = require('fs');

mach.get({
  url: 'http://twitter.com',
  binary: true
}).then(function (conn) {
  conn.responseText; // undefined
  conn.response.content.pipe(fs.createWriteStream('twitter.html'));
});
```

### Installation

Using [npm](https://www.npmjs.org/):

    $ npm install mach

### Issues

Please file issues on the [issue tracker on GitHub](https://github.com/machjs/mach/issues).

### Tests

To run the tests in node:

    $ npm install
    $ npm test

The Redis session store tests rely on Redis to run successfully. By default they are skipped, but if you want to run them fire up a Redis server on the default host and port and set the `$WITH_REDIS` environment variable.

    $ WITH_REDIS=1 npm test

To run the tests in a browser, first run:

    $ npm install
    $ npm run bundle-tests
    $ npm run serve-tests

Then open [http://localhost:8080/](http://localhost:8080/) in a browser.

### Influences

  * [Strata](http://stratajs.org/)
  * [Q-HTTP](https://github.com/kriskowal/q-http)
  * [JSGI & Jack](http://jackjs.org/)
  * [node.js](http://nodejs.org/)

### License

[MIT](http://opensource.org/licenses/MIT)
