---
title: Nodejs API 笔记
---

### Local System

###### [File System](https://nodejs.org/dist/latest-v5.x/docs/api/fs.html)
File I/O is provided by simple wrappers around standard POSIX functions. To use this module do `require('fs')`. All the methods have asynchronous and synchronous forms.

```JavaScript
const fs = require('fs');

// Asynchronous
fs.unlink('/tmp/hello', (err) => {
  if (err) throw err;
  console.log('successfully deleted /tmp/hello');
});

// Synchronous
fs.unlinkSync('/tmp/hello');
console.log('successfully deleted /tmp/hello');
```

###### [Path](https://nodejs.org/dist/latest-v5.x/docs/api/path.html)
This module contains utilities for handling and transforming file paths. Use require('path') to use this module.

```JavaScript
var path = require('path');

// returns 'quux.html'
path.basename('/foo/bar/baz/asdf/quux.html');

// The platform-specific path delimiter, ; or ':'
console.log(path.delimiter);

// returns '/foo/bar/baz/asdf'
path.dirname('/foo/bar/baz/asdf/quux')

// returns '.html'
path.extname('index.html')
// returns '.md'
path.extname('index.coffee.md')
// returns '.'
path.extname('index.')
// returns ''
path.extname('index')
 // returns ''
path.extname('.index')

// returns '/home/user/dir/file.txt'
path.format({
    root : "/",
    dir : "/home/user/dir",
    base : "file.txt",
    ext : ".txt",
    name : "file"
});

// returns '/foo/bar/baz/asdf'
path.join('/foo', 'bar', 'baz/asdf', 'quux', '..')

// returns '/foo/bar/baz/asdf'
path.normalize('/foo/bar//baz/asdf/quux/..')

// returns
// {
//    root : "/",
//    dir : "/home/user/dir",
//    base : "file.txt",
//    ext : ".txt",
//    name : "file"
// }
path.parse('/home/user/dir/file.txt')
```

###### [Child Process](https://nodejs.org/dist/latest-v5.x/docs/api/child_process.html)
The child_process module provides the ability to spawn child processes in a manner that is similar, but not identical, to popen(3).

```JavaScript
const spawn = require('child_process').spawn;
const ls = spawn('ls', ['-lh', '/usr']);

ls.stdout.on('data', (data) => {
  console.log(`stdout: ${data}`);
});

ls.stderr.on('data', (data) => {
  console.log(`stderr: ${data}`);
});

ls.on('close', (code) => {
  console.log(`child process exited with code ${code}`);
});
```

###### [OS](https://nodejs.org/dist/latest-v5.x/docs/api/os.html)
Provides a few basic operating-system related utility functions.

### Network

###### [HTTP](https://nodejs.org/dist/latest-v5.x/docs/api/http.html)
The HTTP interfaces in Node.js are designed to support many features of the protocol which have been traditionally difficult to use. To use the HTTP server and client one must `require('http')`.

###### [URL](https://nodejs.org/dist/latest-v5.x/docs/api/url.html)
This module has utilities for URL resolution and parsing. Call require('url') to use it.

Example: 'http://user:pass@host.com:8080/p/a/t/h?query=string#hash'

* `href`: 'http://user:pass@host.com:8080/p/a/t/h?query=string#hash'
* `protocol`: 'http:'
* `slashes`: true or false (The protocol requires slashes after the colon)
* `host`: 'host.com:8080'
* `auth`: 'user:pass'
* `hostname`: 'host.com'
* `port`: '8080'
* `pathname`: '/p/a/t/h'
* `search`: '?query=string'
* `path`: '/p/a/t/h?query=string'
* `query`: 'query=string' or {'query':'string'}
* `hash`: '#hash'

```JavaScript
url.resolve('/one/two/three', 'four')         // '/one/two/four'
url.resolve('http://example.com/', '/one')    // 'http://example.com/one'
url.resolve('http://example.com/one', '/two') // 'http://example.com/two'
```

### [Global Objects](https://nodejs.org/dist/latest-v5.x/docs/api/globals.html)
These objects are available in all modules.

###### [Console](https://nodejs.org/dist/latest-v5.x/docs/api/console.html)
The console module provides a simple debugging console that is similar to the JavaScript console mechanism provided by web browsers.

```JavaScript
const output = fs.createWriteStream('./stdout.log');
const errorOutput = fs.createWriteStream('./stderr.log');
// custom simple logger
const logger = new Console(output, errorOutput);
// use it like console
var count = 5;
logger.log('count: %d', count);
// in stdout.log: count 5
```

###### [Exports & Modules](https://nodejs.org/dist/latest-v5.x/docs/api/modules.html)
TODO: ...

###### [Process](https://nodejs.org/dist/latest-v5.x/docs/api/process.html)
The `process` object is a global object and can be accessed from anywhere. It is an instance of [EventEmitter](https://nodejs.org/dist/latest-v5.x/docs/api/events.html#events_class_events_eventemitter).

###### [Timers](https://nodejs.org/dist/latest-v5.x/docs/api/timers.html)
TODO: ...

### Misc.

###### [ArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays)
The ArrayBuffer is a data type that is used to represent a generic, fixed-length binary data buffer.

```JavaScript
var buffer = new ArrayBuffer(16);
var int32View = new Int32Array(buffer);
for (var i = 0; i < int32View.length; i++) {
  int32View[i] = i * 2;
}
```

###### [Events](https://nodejs.org/dist/latest-v5.x/docs/api/events.html)
Much of the Node.js core API is built around an idiomatic asynchronous event-driven architecture in which certain kinds of objects (called "emitters") periodically emit named events that cause Function objects ("listeners") to be called.

```JavaScript
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();
myEmitter.on('event', () => {
  console.log('an event occurred!');
});
myEmitter.emit('event');
```
