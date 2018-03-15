## Object.defineProperty
-----
关于[Object.defineProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty):
```js
Object.defineProperty(obj, prop, descriptor)
```
- obj 要在其上定义属性的对象。
- prop 要定义或修改的属性的名称。
- descriptor 将被定义或修改的属性描述符。
Note:Object.defineProperty() 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并返回这个对象。


描述符可同时具有的键值

||configurable|enumerable|
|-|-|-|
|数据描述符|Yes|Yes|
|存取描述符|Yes|Yes|

----
||value|writable|get|set|
|-|-|-|-|-|
|数据描述符|Yes|Yes|No|No|
|存取描述符|No|No|Yes|Yes|


示例：创建对象
```js
var o = {}; // 创建一个新对象

// 在对象中添加一个属性与数据描述符的示例
Object.defineProperty(o, "a", {
  value : 37,
  writable : true,
  enumerable : true,
  configurable : true
});

// 对象o拥有了属性a，值为37

// 在对象中添加一个属性与存取描述符的示例
var bValue;
Object.defineProperty(o, "b", {
  get : function(){
    return bValue;
  },
  set : function(newValue){
    bValue = newValue;
  },
  enumerable : true,
  configurable : true
});

o.b = 38;
// 对象o拥有了属性b，值为38

// o.b的值现在总是与bValue相同，除非重新定义o.b

// 数据描述符和存取描述符不能混合使用
Object.defineProperty(o, "conflict", {
  value: 0x9f91102, 
  get: function() { 
    return 0xdeadbeef; 
  } 
});
// throws a TypeError: value appears only in data descriptors, get appears only in accessor descriptors
```


示例Setters 和 Getters
```js
function Archiver() {
  var temperature = null;
  var archive = [];

  Object.defineProperty(this, 'temperature', {
    get: function() {
      console.log('get!');
      return temperature;
    },
    set: function(value) {
      temperature = value;
      archive.push({ val: temperature });
    }
  });

  this.getArchive = function() { return archive; };
}

var arc = new Archiver();
arc.temperature; // 'get!'
arc.temperature = 11;
arc.temperature = 13;
arc.getArchive(); // [{ val: 11 }, { val: 13 }]
```
Note:例子展示了如何实现一个自存档对象。 当设置temperature 属性时，archive 数组会获取日志条目。


极简的双向绑定

<iframe height='300' scrolling='no' title='极简的双向绑定' src='//codepen.io/geniuspeng/embed/dJVeGj/?height=300&theme-id=15655&default-tab=js,result&embed-version=2' frameborder='no' allowtransparency='true' allowfullscreen='true' style='width: 100%;'>See the Pen <a href='https://codepen.io/geniuspeng/pen/dJVeGj/'>极简的双向绑定</a> by Yunpeng Bai (<a href='https://codepen.io/geniuspeng'>@geniuspeng</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>