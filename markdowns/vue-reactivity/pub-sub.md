
## 观察者模式

```js
function Dep() {//主题对象
  this.subs = []; //订阅者列表
}
Dep.prototype.notify = function() { //主题对象通知订阅者
  this.subs.forEach(function(sub){ //遍历所有的订阅者，执行订阅者提供的更新方法
    sub.update();
  });
}
function Sub(x) { //订阅者
  this.x = x;
}、
Sub.prototype.update = function() { //订阅者更新
  this.x = this.x * this.x;
  console.log(this.x);
}
var pub = { //发布者
  publish: function() {
    dep.notify();
  }
};
var dep = new Dep(); //主题对象实例
Array.prototype.push.call(dep.subs, new Sub(1), new Sub(2), new Sub(3)); //新增 3 个订阅者
pub.publish(); //发布者发布更新
// 1
// 4
// 9
```
Note:首先有3个主体：发布者，主题对象，订阅者，其中发布者和主题对象是一对一的（某些其他场景可能一对多？），而主题对象和订阅者也是一对多的关系。发布者通过发布某个主题对象，主题对象更新并通知其所有的订阅者，然后每个订阅者执行update进行更新。
**对于Vue，我们这里可以认为Vue实例中的data中的每一项属性是一个Dep，而所有用到这个属性的地方都是这个Dep的一个订阅者（sub），当这个属性值变化时，观察者通过监听捕获变化，告诉这个dep去notify每一个订阅者。**