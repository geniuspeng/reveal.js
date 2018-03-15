## OTHER

- fs支持使用whatwg-url的file协议，fs.SyncWriteStream废弃
- HTTP/HTTPS response对象添加getHeaderNames(), getHeaders(), hasHeader()方法
- ASSERT： 默认支持map，set
- Buffer： 支持Uint8Array类型输入
- 环境变量： 添加NODE_NO_WARNINGS，NODE_PRESERVE_SYMLINKS等
- DNS，Console，Domains，Errors，Child Process， TLS，Zlib ...


```sh
> const URL = require('url').URL;
undefined
> const myURL = new URL('file:///C:/path/to/file');
undefined
> fs.readFile(myURL, (err, data) => {});
TypeError: path must be a string or Buffer
// 7.6.0 ~
> fs.readFile(myURL, (err, data) => {});
undefined
```


```sh
const http = require('http');

http.createServer((req, res) => {
  res.setHeader('x-test-header', 'testing');
  res.setHeader('X-TEST-HEADER2', 'testing');
  console.log(res._headers); // { 'x-test-header': 'testing', 'x-test-header2': 'testing' }
  console.log(res.getHeaders()); // { 'x-test-header': 'testing', 'x-test-header2': 'testing' }
  console.log(res.getHeaderNames()); // [ 'x-test-header', 'x-test-header2' ]
  console.log(res.hasHeader('X-TEST-HEADER2')); // true
}).listen(3000, function() {
  http.get({ port: this.address().port }, (res) => {});
});
```


```sh
> assert.deepEqual(new Set([1, 2]), new Set([1]))
AssertionError: Set { 1, 2 } deepEqual Set { 1 }
    at repl:1:8
    at ContextifyScript.Script.runInThisContext (vm.js:44:33)
    at REPLServer.defaultEval (repl.js:239:29)
    at bound (domain.js:301:14)
    at REPLServer.runBound [as eval] (domain.js:314:12)
    at REPLServer.onLine (repl.js:433:10)
    at emitOne (events.js:120:20)
    at REPLServer.emit (events.js:210:7)
    at REPLServer.Interface._onLine (readline.js:262:10)
    at REPLServer.Interface._line (readline.js:611:8)
> assert.deepEqual(new Set([1, 2]), new Set([1, 2]))
undefined
```