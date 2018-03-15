## WHATWG URL

nodejs现在完美支持标准URL对象：

```js
const url = require('url');
const URL = url.URL;
// or const { URL } = require('url');
const myURL =
  new URL('https://user:pass@sub.host.com:8080/p/a/t/h?query=string#hash');
myURL;
```


![url](/images/nodejs8/url.png)

Note:url 模块提供了一些实用函数，用于 URL 处理与解析。 可以通过以下方式使用：const url = require('url');
一个 URL 字符串是一个结构化的字符串，它包含多个有意义的组成部分。 当被解析时，会返回一个 URL 对象，它包含每个组成部分作为属性。


Class: URLSearchParams

```js
const { URLSearchParams } = require('url');
```
- urlSearchParams.get(name)
- urlSearchParams.getAll(name)
- urlSearchParams.set(name, value)
- urlSearchParams.append(name, value)
- urlSearchParams.delete(name)
- urlSearchParams.entries()
- urlSearchParams.forEach(fn[, thisArg])
- urlSearchParams.has(name)
- urlSearchParams.sort()
Note: 还有urlSearchParams.values()，urlSearchParams.toString()