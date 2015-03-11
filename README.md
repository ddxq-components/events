# events
==========

events 模块由 [arale-events](https://github.com/aralejs/events) 修改而来，提供基本的事件添加、移除和触发功能。

----------

## 使用说明

使用 `Events` 有两种方式，一种是直接实例化：

```
var Events = require('events');

var object = new Events();
object.on('expand', function() {
    alert('expanded');
});

object.trigger('expand');
```

另一种是将 `Events` 混入（mix-in）到其他类中：

```
var Events = require('events');

function Dog() {
}
Events.mixTo(Dog);

Dog.prototype.sleep = function() {
    this.trigger('sleep');
};

var dog = new Dog();
dog.on('sleep', function() {
    alert('狗狗睡得好香呀');
});

dog.sleep();
```

上面的例子已经展现了 `on`, `trigger` `mixTo` 等方法的基本用法，下面详细阐述所有 API 。


### on `object.on(event, callback, [context])`

给对象添加事件回调函数。

可以传入第三个参数 `context` 来改变回调函数调用时的 `this` 值：

```
post.on('saved', callback, that);
```

**注意**：`event` 参数有个特殊取值：`all` 对象上触发任何事件，都会触发 `all`
事件的回调函数，传给 `all` 事件回调函数的第一个参数是事件名。例如，下面的代码可以将一个对象上的所有事件代理到另一个对象上：

```
proxy.on('all', function(eventName) {
    object.trigger(eventName);
});
```

### off `object.off([event], [callback], [context])`

从对象上移除事件回调函数。三个参数都是可选的，当省略某个参数时，表示取该参数的所有值。例子：

```
// 移除 change 事件上名为 onChange 的回调函数
object.off('change', onChange);

// 移除 change 事件的所有回调函数
object.off('change');

// 移除所有事件上名为 onChange 的回调函数
object.off(null, onChange);

// 移除上下文为 context 的所有事件的所有回调函数
object.off(null, null, context);

// 移除 object 对象上所有事件的所有回调函数
object.off();
```


### trigger/emit `object.trigger(event, [*args])`

触发一个或多个事件（用空格分隔）。`*args` 参数会依次传给回调函数。

```
obj.on('event', function(arg1, arg2) {
  // your code
});

obj.trigger('event', arg1, arg2);
```

trigger 的返回值是一个布尔值，会根据所有 callback 的执行情况返回。只要有一个 callback 返回 false，trigger 就会返回 false。

```
obj.on('event', function() {
  // do sth.
});
obj.on('event', function() {
  // do sth.
  return false;
});
obj.on('event', function() {
  // do sth
});

obj.trigger('event'); // return false
```

**注意**：`on` 和 `off` 的 `event` 参数也可以表示多个事件（用空格分隔），比如：

```
var obj = new Events();

obj.on('x y', fn);

// 等价：
obj.on('x').on('y');
```


### mixTo `Events.mixTo(receiver)`

将 `Events` 的原型方法混入到指定对象或函数原型中。

## 问题讨论

- handler 的异常处理 https://github.com/aralejs/events/issues/1

## 性能对比

- <http://jsperf.com/events-perfs/6>

**注**：最开始，该模块的主要代码直接来自 Backbone.Events. 后来发现 Backbone
的代码实现有较大的改进空间，因此将内部的数据结构从链表改成了数组，重构后大幅度提升了性能。目前
Backbone 已合并 Arale 的代码：

- <https://github.com/documentcloud/backbone/pull/1284>



## Events 竞争对手分析

- order: 1

---

- 在 DOM 事件的封装上，YUI3 最强大，并带有很强的学术性。jQuery 则非常实用。还有其他一些类库，比如
KISSY、QWrap 等，都大同小异。

- 在 Arale 里，与 DOM 相关的事件，直接采用 jQuery 来实现。

- 该 Events 模块仅提供非常纯粹的事件驱动机制。

- 与该模块定位相同的同类产品有：Backbone.Events, NodeJS 的 EventEmitter 等。

- NodeJS 的 EventEmitter 功能点较多，主要考虑服务器端。

- Backbone 的 Events 定位与我们最像，同时也考虑了与 jQuery API 风格的一致性，非常不错。

- 该模块会重点借鉴 Backbone.Events. 在 API 上会有少量调整，以与类库的整体设计风格保持一致。
