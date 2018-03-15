## Observer对象

Observer对象在[src/core/observer/index.js](https://github.com/vuejs/vue/blob/v2.5.13/src/core/observer/index.js)中。
- observeArray: 遍历数组，对数组的每个元素调用observe
- observe: 检查对象上是否有__ob__属性，如果存在，则表明该对象已经处于Observer的观察中，如果不存在，则new Observer来观察对象
- walk: 遍历对象的每个key，对对象上每个key的数据调用defineReactive
- defineReactive: 通过Object.defineProperty设置对象的key属性，使得能够捕获到该属性值的set/get动作。
Note:一般是由Watcher的实例对象进行get操作，此时Watcher的实例对象将被自动添加到Dep实例的依赖数组中，在外部操作触发了set时，将通过Dep实例的notify来通知所有依赖的watcher进行更新。（后面详细介绍）