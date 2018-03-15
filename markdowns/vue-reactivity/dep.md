## Dep对象

Dep类定义在[src/core/observer/dep.js](https://github.com/vuejs/vue/blob/v2.5.13/src/core/observer/dep.js)中。
- addSub: 接收的参数为Watcher实例，并把Watcher实例存入记录依赖的数组中
- removeSub: 与addSub对应，作用是将Watcher实例从记录依赖的数组中移除
- depend: Dep.target上存放这当前需要操作的Watcher实例，调用depend会调用该Watcher实例的addDep方法
- notify: 通知依赖数组中所有的watcher进行更新操作