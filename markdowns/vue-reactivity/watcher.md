## Watcher对象

Watcher类定义在[src/core/observer/watcher.js](https://github.com/vuejs/vue/blob/v2.5.13/src/core/observer/watcher.js)。
- get: 将Dep.target设置为当前watcher实例，在内部调用this.getter，如果此时某个被Observer观察的数据对象被取值了，那么当前watcher实例将会自动订阅数据对象的Dep实例
- addDep: 接收参数dep(Dep实例)，让当前watcher订阅dep
- cleanupDeps: 清除newDepIds和newDep上记录的对dep的订阅信息
- update: 立刻运行watcher或者将watcher加入队列中等待统一flush


- run: 运行watcher，调用this.get()求值，然后触发回调
- evaluate: 调用this.get()求值
- depend: 遍历this.deps，让当前watcher实例订阅所有dep
- teardown: 去除当前watcher实例所有的订阅