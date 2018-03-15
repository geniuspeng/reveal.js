## UTIL

<div style="text-align: left;">util.inspect添加[Array]符号, 同时可以展示symbol key</div>

```sh
> const obj = util.inspect({'a': {'b': ['c']}}, false, 1)
> obj
'{ a: { b: [Object] } }'
> assert.strictEqual(obj, '{ a: { b: [Array] } }')
AssertionError: '{ a: { b: [Object] } }' === '{ a: { b: [Array] } }'

// 8.0.0
> obj
'{ a: { b: [Array] } }'
> assert.strictEqual(obj, '{ a: { b: [Array] } }')
undefined
```


<div style="text-align: left;">util.format新增整形——%i，浮点型——%f</div>

```sh
> util.format('%d', 42.2)
'42.2'
> util.format('%i', 42.2)
'%i 42.2'
> util.format('%f', 42.2)
'%f 42.2'

// 8.0.0
> util.format('%d', 42.2)
'42.2'
> util.format('%i', 42.2)
'42'
> util.format('%f', 42.2)
'42.2'
```


- 添加 promisify()

<img src="/images/nodejs8/promisify.png" alt="promisify" style="width:400px;height:500px"></img>

Note: Node.js 一直以来的关键设计就是把用户关在一个“异步编程的监狱”里，以换取非阻塞 I/O 的高性能，让用户轻易开发出高度可扩展的网络服务器。这从 Node.js 的 API 设计上就可见一斑，很多API——如 fs.open(path, flags[, mode], callback)——要求用户必须把该操作执行成功后的逻辑放在最后参数里，作为函数传递进去；而 fs.open 本身是立即返回的，用户不能把依赖于 fs.open 结果的逻辑与 fs.open 本身线性地串联起来。为了减轻异步编程的痛苦，几年间我们见证了数个解决方案的出现：从深度嵌套的回调金字塔，到带有长长的 then() 链条的 Promise 设计模式，再到 Generator 函数，到如今 Node.js 8 的 async/await 操作符。笔者认为，所有这些解决方案中，async/await 操作符是最接近命令式编程风格的，使用起来最为自然的。


```js
const writeFile = require('fs').writeFile;
const stat = require('fs').stat;

await writeFile('a_new_file.txt', 'hello');
let result = await stat('a_new_file.txt');
console.log(result.size);
```
Note:例如我们想先创建一个文件，再读取、输出它的大小，只需三行代码,这简直是最简单的异步编程了！我们用自然、流畅的代码表达了线性业务逻辑，同时还得到了 Node.js 非阻塞 I/O 带来的高性能，简直是兼得了鱼和熊掌。
但别着急，这段代码不是立即就可以执行的，细心的读者肯定会问：例子中的 writeFile 和 stat 分别是什么？其实它们就是标准库的 fs.writeFile 和 fs.stat，但又不完全是。这是因为 async 和 await 本质上是对 Promise 设计模式的封装，一般情况下 await 的参数应是一个返回 Promise 对象的函数。而 fs.writeFile 和 fs.stat 这些标准库 API 没有返回值（返回 undefined），需要一个方法把他们包装成返回 Promise 对象的函数。


```js
const util = require('util');
const fs = require('fs');
const writeFile = util.promisify(fs.writeFile);
const stat = util.promisify(fs.stat);

(async function () {
  await writeFile('a_new_file.txt', 'hello');
  let result = await stat('a_new_file.txt');
  console.log(result.size);
})();
```
