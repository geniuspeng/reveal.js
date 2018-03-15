## INSPECTOR

命令行在 --inspector基础上，新增--inspector-brk，同时可以添加=[port]指定调试端口，默认9229，node --debug废弃

--debug-brk 通过这个参数，在开始调试的时候能够定位到代码的第一行。


新增inspector内置模块，支持程序化的debug。

```js
const inspector = require('inspector');
```
- inspector.open([port[, host[, wait]]])
- inspector.close()
- inspector.url()
- Class: inspector.Session

Note: 如果wait为true，将阻塞，直到客户端连接到检查端口。
If wait is true, will block until a client has connected to the inspect port and flow control has been passed to the debugger client.


A new experimental JavaScript API for the Inspector protocol has been introduced enabling developers new ways of leveraging the debug protocol to inspect running Node.js processes.

```js
const inspector = require('inspector');

const session = new inspector.Session();
session.connect();

// Listen for inspector events
session.on('inspectorNotification', (message) => { /** ... **/ });

// Send messages to the inspector
session.post(message);

session.disconnect();
```
Note:Note that the inspector API is experimental and may change significantly.
Session这个class支持监听inspector的一些相关事件，可以向v8的inspector后台发送或接收message