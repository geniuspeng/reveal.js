## N-API

目前 Node 的实现中，V8 的 API 是直接暴露出来的。由于 V8 经常变更 API，那就存在下面的这些问题：

- Native 模块在每个版本间需要重新编译
- Native 模块需要变更代码
- Native 模块无法工作在其他JS engine 上 (比如: ChakraCore)
这些问题之前的 NAN(Native Abstractions for Node.js)(nodejs/nan) 搞不定。


经历过 Node.js 大版本升级的同学肯定会发现，每次升级后我们都得重新编译像 node-sass 这种用 C++ 写的扩展模块

```sh
Error: The module 'node-sass'
was compiled against a different Node.js version using
NODE_MODULE_VERSION 51. This version of Node.js requires
NODE_MODULE_VERSION 55. Please try re-compiling or re-installing
the module (for instance, using `npm rebuild` or `npm install`).
```

Note:
NODE_MODULE_VERSION 是每一个 Node.js 版本内人为设定的数值，意思为 ABI (Application Binary Interface) 的版本号。一旦这个号码与已经编译好的二进制模块的号码不符，便判断为 ABI 不兼容，需要用户重新编译。

这其实是一个工程难题，亦即 Node.js 上游的代码变化如何最小地降低对 C++ 模块的影响，从而维持一个良好的向下兼容的模块生态系统。最坏的情况下，每次发布 Node.js 新版本，因为 API 的变化，C++ 模块的作者都要修改它们的源代码，而那些不再有人维护或作者失联的老模块就会无法继续使用，在作者修改代码之前社区就失去了这些模块的可用性。其次坏的情况是，每次发布 Node.js 新版本，虽然 API 保持兼容使得 C++ 模块的作者不需要修改他们的代码，但 ABI 的变化导致必须这些模块必须重新编译。而最好的情况就是，Node.js 新版本发布后，所有已编译的 C++ 模块可以继续正常工作，完全不需要任何人工干预。


Node.js 8 的 Node.js API (N-API) 就是为了解决这个问题，做到上述最好的情况，为 Node.js 模块生态系统的长期发展铺平道路。N-API 追求以下目标：
1. 有稳定的 ABI
2. 抽象消除 Node.js 版本之间的接口差异
3. 抽象消除 V8 版本之间的接口差异
4. 抽象消除 V8 与其他 JS 引擎（如 ChakraCore）之间的接口差异


### 已经进行N-API适配的模块
!['napi'](/images/nodejs8/napi.png)

