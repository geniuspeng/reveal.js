## ASYNC_HOOKS

<p><small>async_hooks模块提供了一个API来回溯跟踪Node.js应用创建的异步资源的生命周期。API预览如下：</small></p>
 
```javascript
const async_hooks = require('async_hooks');

// 当前异步资源唯一id
const cid = async_hooks.currentId();

// 触发当前异步行为的异步资源id
const tid = async_hooks.triggerId();

//创建异步勾子监听
const asyncHook = async_hooks.createHook({ init, before, after, destroy });

//开始监听
asyncHook.enable();

//结束监听
asyncHook.disable();

// 资源对象初始化
function init(asyncId, type, triggerId, resource) { }

// 异步回调开始之前
function before(asyncId) { }

// 异步回调完成之后
function after(asyncId) { }

// AsyncWrap实例销毁.
function destroy(asyncId) { }
```
Note: 
init()异步资源生成时执行，用于初始化，这个函数执行的时候异步资源可能还没有完全生成，但是一定会生成一个唯一的asyncID，before()异步回调开始之前执行，它可能被执行0-N次(e.g. TCPWrap)，取决于这个异步资源执行了多少次回调，after()在异步回调finish的时候执行，destroy is called when an AsyncWrap instance is destroyed.


init(asyncId, type, triggerId, resource)

- asyncId &lt;number&gt; 当前异步资源的唯一标识
- type &lt;string&gt; 异步资源的类型
- triggerId &lt;number&gt; 触发当前异步行为的异步资源的asyncId
- resource &lt;Object&gt; 异步资源对象，将在destory时释放，但是并不意味着每个实例都要在destroy之前执行before和after
Note: 
asyncId当前异步资源的唯一标识，type异步资源的类型，triggerId触发当前异步行为的异步资源的asyncId，resource异步资源对象，将在destory时释放，但是并不意味着每个实例都要在destroy之前执行before和after。


type:

FSEVENTWRAP, FSREQWRAP, GETADDRINFOREQWRAP, GETNAMEINFOREQWRAP, HTTPPARSER,
JSSTREAM, PIPECONNECTWRAP, PIPEWRAP, PROCESSWRAP, QUERYWRAP, SHUTDOWNWRAP,
SIGNALWRAP, STATWATCHER, TCPCONNECTWRAP, TCPWRAP, TIMERWRAP, TTYWRAP,
UDPSENDWRAP, UDPWRAP, WRITEWRAP, ZLIB, SSLCONNECTION, PBKDF2REQUEST,
RANDOMBYTESREQUEST, TLSWRAP


trigger_id:

```javascript
async_hooks.createHook({
  init(asyncId, type, triggerId) {
    const cId = async_hooks.currentId();
    fs.writeSync(
      1, `${type}(${asyncId}): trigger: ${triggerId} scope: ${cId}\n`);
  }
}).enable();

require('net').createServer((conn) => {}).listen(8080);
```
Note: 演示，console


一个功能：追踪完整的Error Stack<!-- .element: class="fragment" data-fragment-index="1" -->

这只是 Node.js 8 的 Async Hooks 的用途之一，有了这个功能，我们甚至可以来测量一些事件各个阶段所花费的时间。只要我们有异步资源 ID 这枚钥匙，配合回调函数，就可以在事件循环的多个周期那看似毫无头绪的执行过程中，筛选出有用的信息。<!-- .element: class="fragment" data-fragment-index="2" -->