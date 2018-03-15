## V8引擎

- **v5.7更新：**
- promise 和 async 提速
- spread operator, destructuring 和 generators 提速
- 通过 TurboFan, RegExp 提速了 15%
- padStart 和 padEnd 被添加到了 es2017 (ECMA 262)
- 支持[Intl.DateTimeFormat.prototype.formatToParts](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DateTimeFormat)
- **WebAssembly 默认打开**
- **添加 PromiseHook**


- **v5.8更新：**
- 任意设置堆大小的限制值 (范围是带符号32位整数)
- 启动性能提升约5％
- 缩短 IC 系统的代码编译，分析，优化时间


<p><small>Async functions are now approximately as fast as the same code written with promises. The execution performance of async functions quadrupled according to our microbenchmarks. During the same period, overall promise performance also doubled.</small></p>
![v8async](/images/nodejs8/asyncimprovements.png)


![tfi](/images/nodejs8/tfi.png)

Note:

TurboFan & Ingnition
TURBOFAN 编译器

V8 优化了 JIT 编译器，它是用 Sea of Nodes 概念进行设计的。之前采用的编译技术是 Crankshaft，它支持优化更多的代码。但是 ES 的标准发展很快，后来发现 Crankshaft 已经很难去优化 ES2015 代码了，而通过 Ignition 和 TurboFan 可以做到。

IGNITION 解释器

已知的是到目前为止 V8 都没有自己的解析器，都是直接把 JS 编译成机器码。
作为JIT的JIT的问题，即使代码被执行一次，它也消耗大量的存储空间，所以需要尽量避免内存开销。
使用 Ignition，可以精简 25% - 50% 的机器码。
Ignition 是没有依赖 Turbofan 的底层架构，它采用宏汇编指令。为每个操作码生成一个字节码处理程序。
通过 Ignition 可以较少系统内存使用情况。

之前 v8 选择了直接将 JS 代码编译到机器代码执行，机器码的执行性能已经非常之高，而这次引入字节码则是选择编译 JS 代码到一个中间态的字节码，执行时是解释执行，性能是低于机器代码的。最终的性能测试势必会降低，而不是提高。那么 V8 为什么要做这样一个退步的选择呢？为 V8 引入字节码的动机又是什么呢？笔者总结下来有三条：

（主要动机）减轻机器码占用的内存空间，即牺牲时间换空间
提高代码的启动速度
对 v8 的代码进行重构，降低 v8 的代码复杂度

WebAssembly:WebAssembly 是除了 JavaScript 以外，另一种可以在浏览器中执行的编程语言。所以当人们说 WebAssembly 更快的时候，一般来讲是与 JavaScript 相比而言的。