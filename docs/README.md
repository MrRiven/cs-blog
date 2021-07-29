# requestAnimationFrame

> 与 setTimeout 和 setInterval 不同，requestAnimationFrame 不需要设置时间间隔。计时器一直是 javascript 动画的核心技术。而编写动画循环的关键是要知道延迟时间多长合适。一方面，循环间隔必须足够短，这样才能让不同的动画效果显得平滑流畅；另一方面，循环间隔还要足够长，这样才能确保浏览器有能力渲染产生的变化

> 大多数电脑显示器的刷新频率是 60Hz，大概相当于每秒钟重绘 60 次。大多数浏览器都会对重绘操作加以限制，不超过显示器的重绘频率，因为即使超过那个频率用户体验也不会有提升。因此，最平滑动画的最佳循环间隔是 1000ms/60，约等于 16.6ms
>
> 而 setTimeout 和 setInterval 的问题是，它们都不精确。它们的内在运行机制决定了时间间隔参数实际上只是指定了把动画代码添加到浏览器 UI 线程队列中以等待执行的时间。如果队列前面已经加入了其他任务，那动画代码就要等前面的任务完成后再执行
> v
> requestAnimationFrame 采用系统时间间隔，保持最佳绘制效率，不会因为间隔时间过短，造成过度绘制，增加开销；也不会因为间隔时间太长，使用动画卡顿不流畅，让各种网页动画效果能够有一个统一的刷新机制，从而节省系统资源，提高系统性能，改善视觉效果。

---

## 特点

- 【1】requestAnimationFrame 会把每一帧中的所有 DOM 操作集中起来，在一次重绘或回流中就完成，并且重绘或回流的时间间隔紧紧跟随浏览器的刷新频率
- 【2】在隐藏或不可见的元素中，requestAnimationFrame 将不会进行重绘或回流，这当然就意味着更少的 CPU、GPU 和内存使用量
- 【3】requestAnimationFrame 是由浏览器专门为动画提供的 API，在运行时浏览器会自动优化方法的调用，并且如果页面不是激活状态下的话，动画会自动暂停，有效节省了 CPU 开销

---

## 使用

> requestAnimationFrame 的用法与 settimeout 很相似，只是不需要设置时间间隔而已。requestAnimationFrame 使用一个回调函数作为参数，这个回调函数会在浏览器重绘之前调用。它返回一个整数，表示定时器的编号，这个值可以传递给 cancelAnimationFrame 用于取消这个函数的执行

```
requestID = requestAnimationFrame(callback);
//控制台输出1和0 var timer = requestAnimationFrame(function(){
    console.log(0);
});
console.log(timer);//1
```

> cancelAnimationFrame 方法用于取消定时器

```
//控制台什么都不输出 var timer = requestAnimationFrame(function(){
    console.log(0);
});
cancelAnimationFrame(timer);
```

> 也可以直接使用返回值进行取消

```
var timer = requestAnimationFrame(function(){
    console.log(0);
});
cancelAnimationFrame(1);
```

---

## 兼容

> IE9-浏览器不支持该方法，可以使用 setTimeout 来兼容

- 简单兼容

```
if (!window.requestAnimationFrame) {
    requestAnimationFrame = function(fn) {
        setTimeout(fn, 17);
    };
}
```

```
window.requestAnimFrame = (function(){
  return  window.requestAnimationFrame       ||
          window.webkitRequestAnimationFrame ||
          window.mozRequestAnimationFrame    ||
          function( callback ){
            window.setTimeout(callback, 1000 / 60);
          };
})();
```

- 严格兼容

```
if(!window.requestAnimationFrame){
    var lastTime = 0;
    window.requestAnimationFrame = function(callback){
        var currTime = new Date().getTime();
        var timeToCall = Math.max(0,16.7-(currTime - lastTime));
        var id  = window.setTimeout(function(){
            callback(currTime + timeToCall);
        },timeToCall);
        lastTime = currTime + timeToCall;
        return id;
    }
}
```

```
if (!window.cancelAnimationFrame) {
    window.cancelAnimationFrame = function(id) {
        clearTimeout(id);
    };
}
```

```
(function() {
    var lastTime = 0;
    var vendors = ['webkit', 'moz'];
    for(var x = 0; x < vendors.length && !window.requestAnimationFrame; ++x) {
        window.requestAnimationFrame = window[vendors[x] + 'RequestAnimationFrame'];
        window.cancelAnimationFrame = window[vendors[x] + 'CancelAnimationFrame'] ||    // Webkit中此取消方法的名字变了
                                      window[vendors[x] + 'CancelRequestAnimationFrame'];
    }

    if (!window.requestAnimationFrame) {
        window.requestAnimationFrame = function(callback, element) {
            var currTime = new Date().getTime();
            var timeToCall = Math.max(0, 16.7 - (currTime - lastTime));
            var id = window.setTimeout(function() {
                callback(currTime + timeToCall);
            }, timeToCall);
            lastTime = currTime + timeToCall;
            return id;
        };
    }
    if (!window.cancelAnimationFrame) {
        window.cancelAnimationFrame = function(id) {
            clearTimeout(id);
        };
    }
}());
```

# new 关键字

## 创建一个实例

```
// ES5构造函数
let Parent = function (name, age) {
    this.name = name;
    this.age = age;
};
Parent.prototype.sayName = function () {
    console.log(this.name);
};
const child = new Parent('听风是风', 26);
child.sayName() //'听风是风'


//ES6 class类
class Parent {
    constructor(name, age) {
        this.name = name;
        this.age = age;
    }
    sayName() {
        console.log(this.name);
    }
};
const child = new Parent('echo', 26);
child.sayName() //echo
```

## new 的过程发生了哪些

- 创建一个空对象，将它的引用赋给 this，继承函数的原型。
- 通过 this 将属性和方法添加至这个对象
- 最后返回 this 指向的新对象，也就是实例（如果没有手动返回其他的对象）

```
// ES5构造函数
let Parent = function (name, age) {
    //1.创建一个新对象，赋予this，这一步是隐性的，
    // let this = {};
    //2.给this指向的对象赋予构造属性
    this.name = name;
    this.age = age;
    //3.如果没有手动返回对象，则默认返回this指向的这个对象，也是隐性的
    // return this;
};
const child = new Parent();
```

简单改写下 this , this 的创建与返回是隐性的

```
// ES5构造函数
let Parent = function (name, age) {
    let that = this;
    that.name = name;
    that.age = age;
    return that;
};
const child = new Parent('听风是风', '26');

```

new 过程中会新建对象，此对象会继承构造器的原型与原型上的属性，最后它会被作为实例返回这样一个过程

- 以构造器的 prototype 属性为原型，创建新对象；
- 将 this(也就是上一句中的新对象)和调用参数传给构造器，执行；
- 如果构造器没有手动返回对象，则返回第一步创建的新对象，如果有，则舍弃掉第一步创建的新对象，返回手动 return 的对象。

## 实现一个简单的 new 方法

```
// 构造器函数
let Parent = function (name, age) {
    this.name = name;
    this.age = age;
};
Parent.prototype.sayName = function () {
    console.log(this.name);
};
//自己定义的new方法
let newMethod = function (Parent, ...rest) {
    // 1.以构造器的prototype属性为原型，创建新对象；
    let child = Object.create(Parent.prototype);
    // 2.将this和调用参数传给构造器执行
    let result = Parent.apply(child, rest);
    // 3.如果构造器没有手动返回对象，则返回第一步的对象
    return typeof result  === 'object' ? result : child;
};
//创建实例，将构造函数Parent与形参作为参数传入
const child = newMethod(Parent, 'echo', 26);
child.sayName() //'echo';

//最后检验，与使用new的效果相同
child instanceof Parent//true
child.hasOwnProperty('name')//true
child.hasOwnProperty('age')//true
child.hasOwnProperty('sayName')//false
```

# 24 种设计模式

## 思维导图

![Image text](https://raw.githubusercontent.com/MrRiven/note/main/static/设计模式思维导图.png)

## 七大原则

- 开放-封闭原则 通俗：对扩展开发，对修改关闭
- 单一职责原则 通俗：一个类只做一件事
- 依赖倒转原则 通俗：类似 IOC，采用接口编程
- 迪米特法则（也称为最小知识原则） 通俗：高内聚，低耦合
- 接口隔离原则 通俗：细节接口
- 合成/聚合复用原则 通俗：避免使用继承
- 里氏代换原则 通俗：子类不能去修改父类的功能

## 结构性模式：

1. 适配器模式
2. 桥接模式
3. 组合模式
4. 装饰者模式
5. 门面模式
6. 享元模式
7. 代理模式

## 创建模式

1. 抽象工厂模式
2. 建造者模式
3. 工厂方法
4. 原型模式
5. 单例模式

## 行为模式

1. 责任链
2. 命令模式
3. 解释器模式
4. 迭代器模式
5. 中介者模式
6. 备忘录模式
7. 空对象模式
8. 观察者模式
9. 状态模式
10. 策略模式
11. 模板方法模式
12. 访问者模式

# html2canvas 使用

```
import html2canvas from "html2canvas";
.
.
.
// 回到顶部, 防止部分截图出现空白不全
window.pageYOffset = 0
document.documentElement.scrollTop = 0
document.body.scrollTop = 0
// 部分安卓上还是无法回滚到顶部 app是要截图的容器最顶部
document.querySelector('.app').scrollIntoView()

let dom = this.$refs.imageTofile
const width = dom.offsetWidth
const height = dom.offsetHeight //包裹容器的高度
html2canvas(dom, {
  scrollY: 0,
  background: 'transparent', //  透明
  width: width, //画布的宽度，即生成图片的宽度
  height: height, //画布的宽度，同上
  scale: 3, // 缩放
  useCORS: true, // 是否使用图片跨域，如何使本地图片不用配置
  allowTaint: true, //允许污染
}).then((canvas) => {
  let url = canvas.toDataURL("image/png");
  //TODO : url do something
});

```

# ES5 实现 Map 数据结构

```
(function (global) {
    'use strict';

    var hasWeakMap = 'WeakMap' in global;
    var hasMap = 'Map' in global;
    var hasForEach = true;

    if (hasMap) {
        hasForEach = 'forEach' in new global.Map();
    }

    if (hasWeakMap && hasMap && hasForEach) { return; }

    var hasObject_create = 'create' in Object;
    var createNPO = function () {
        return hasObject_create ? Object.create(null) : {};
    };

    var namespaces = createNPO();
    var count = 0;
    var reDefineValueOf = function (obj) {
        var privates = createNPO();
        var baseValueOf = obj.valueOf;
        var valueOf = function (namespace, n) {
            if ((this === obj) &&
                (n in namespaces) &&
                (namespaces[n] === namespace)) {
                if (!(n in privates)) { privates[n] = createNPO(); }
                return privates[n];
            }
            else {
                return baseValueOf.apply(this, arguments);
            }
        };
        if (hasObject_create && 'defineProperty' in Object) {
            Object.defineProperty(obj, 'valueOf', {
                value: valueOf,
                writable: true,
                configurable: true,
                enumerable: false
            });
        }
        else {
            obj.valueOf = valueOf;
        }
    };

    if (!hasWeakMap) {
        global.WeakMap = function WeakMap() {
            var namespace = createNPO();
            var n = count++;
            namespaces[n] = namespace;
            var map = function (key) {
                if (key !== Object(key)) {
                    throw new Error('value is not a non-null object');
                }
                var privates = key.valueOf(namespace, n);
                if (privates !== key.valueOf()) {
                    return privates;
                }
                reDefineValueOf(key);
                return key.valueOf(namespace, n);
            };
            var m = this;
            if (hasObject_create) {
                m = Object.create(WeakMap.prototype, {
                    get: {
                        value: function (key) { return map(key).value; },
                        writable: false,
                        configurable: false,
                        enumerable: false
                    },
                    set: {
                        value: function (key, value) { map(key).value = value; },
                        writable: false,
                        configurable: false,
                        enumerable: false
                    },
                    has: {
                        value: function (key) { return 'value' in map(key); },
                        writable: false,
                        configurable: false,
                        enumerable: false
                    },
                    'delete': {
                        value: function (key) { return delete map(key).value; },
                        writable: false,
                        configurable: false,
                        enumerable: false
                    },
                    clear: {
                        value: function () {
                            delete namespaces[n];
                            n = count++;
                            namespaces[n] = namespace;
                        },
                        writable: false,
                        configurable: false,
                        enumerable: false
                    }
                });
            }
            else {
                m.get = function (key) { return map(key).value; };
                m.set = function (key, value) { map(key).value = value; };
                m.has = function (key) { return 'value' in map(key); };
                m['delete'] = function (key) { return delete map(key).value; };
                m.clear = function () {
                    delete namespaces[n];
                    n = count++;
                    namespaces[n] = namespace;
                };
            }

            if (arguments.length > 0 && Array.isArray(arguments[0])) {
                var iterable = arguments[0];
                for (var i = 0, len = iterable.length; i < len; i++) {
                    m.set(iterable[i][0], iterable[i][1]);
                }
            }
            return m;
        };
    }

    if (!hasMap) {
        var objectMap = function () {
            var namespace = createNPO();
            var n = count++;
            var nullMap = createNPO();
            namespaces[n] = namespace;
            var map = function (key) {
                if (key === null) { return nullMap; }
                var privates = key.valueOf(namespace, n);
                if (privates !== key.valueOf()) { return privates; }
                reDefineValueOf(key);
                return key.valueOf(namespace, n);
            };
            return {
                get: function (key) { return map(key).value; },
                set: function (key, value) { map(key).value = value; },
                has: function (key) { return 'value' in map(key); },
                'delete': function (key) { return delete map(key).value; },
                clear: function () {
                    delete namespaces[n];
                    n = count++;
                    namespaces[n] = namespace;
                }
            };
        };
        var noKeyMap = function () {
            var map = createNPO();
            return {
                get: function () { return map.value; },
                set: function (_, value) { map.value = value; },
                has: function () { return 'value' in map; },
                'delete': function () { return delete map.value; },
                clear: function () { map = createNPO(); }
            };
        };
        var scalarMap = function () {
            var map = createNPO();
            return {
                get: function (key) { return map[key]; },
                set: function (key, value) { map[key] = value; },
                has: function (key) { return key in map; },
                'delete': function (key) { return delete map[key]; },
                clear: function () { map = createNPO(); }
            };
        };
        if (!hasObject_create) {
            var stringMap = function () {
                var map = {};
                return {
                    get: function (key) { return map['!' + key]; },
                    set: function (key, value) { map['!' + key] = value; },
                    has: function (key) { return ('!' + key) in map; },
                    'delete': function (key) { return delete map['!' + key]; },
                    clear: function () { map = {}; }
                };
            };
        }
        global.Map = function Map() {
            var map = {
                'number': scalarMap(),
                'string': hasObject_create ? scalarMap() : stringMap(),
                'boolean': scalarMap(),
                'object': objectMap(),
                'function': objectMap(),
                'unknown': objectMap(),
                'undefined': noKeyMap(),
                'null': noKeyMap()
            };
            var size = 0;
            var keys = [];
            var m = this;
            if (hasObject_create) {
                m = Object.create(Map.prototype, {
                    size: {
                        get : function () { return size; },
                        configurable: false,
                        enumerable: false
                    },
                    get: {
                        value: function (key) {
                            return map[typeof(key)].get(key);
                        },
                        writable: false,
                        configurable: false,
                        enumerable: false
                    },
                    set: {
                        value: function (key, value) {
                            if (!this.has(key)) {
                                keys.push(key);
                                size++;
                            }
                            map[typeof(key)].set(key, value);
                        },
                        writable: false,
                        configurable: false,
                        enumerable: false
                    },
                    has: {
                        value: function (key) {
                            return map[typeof(key)].has(key);
                        },
                        writable: false,
                        configurable: false,
                        enumerable: false
                    },
                    'delete': {
                        value: function (key) {
                            if (this.has(key)) {
                                size--;
                                keys.splice(keys.indexOf(key), 1);
                                return map[typeof(key)]['delete'](key);
                            }
                            return false;
                        },
                        writable: false,
                        configurable: false,
                        enumerable: false
                    },
                    clear: {
                        value: function () {
                            keys.length = 0;
                            for (var key in map) { map[key].clear(); }
                            size = 0;
                        },
                        writable: false,
                        configurable: false,
                        enumerable: false
                    },
                    forEach: {
                        value: function (callback, thisArg) {
                            for (var i = 0, n = keys.length; i < n; i++) {
                                callback.call(thisArg, this.get(keys[i]), keys[i], this);
                            }
                        },
                        writable: false,
                        configurable: false,
                        enumerable: false
                    }
                });
            }
            else {
                m.size = size;
                m.get = function (key) {
                    return map[typeof(key)].get(key);
                };
                m.set = function (key, value) {
                    if (!this.has(key)) {
                        keys.push(key);
                        this.size = ++size;
                    }
                    map[typeof(key)].set(key, value);
                };
                m.has = function (key) {
                    return map[typeof(key)].has(key);
                };
                m['delete'] = function (key) {
                    if (this.has(key)) {
                        this.size = --size;
                        keys.splice(keys.indexOf(key), 1);
                        return map[typeof(key)]['delete'](key);
                    }
                    return false;
                };
                m.clear = function () {
                    keys.length = 0;
                    for (var key in map) { map[key].clear(); }
                    this.size = size = 0;
                };
                m.forEach = function (callback, thisArg) {
                    for (var i = 0, n = keys.length; i < n; i++) {
                        callback.call(thisArg, this.get(keys[i]), keys[i], this);
                    }
                };
            }
            if (arguments.length > 0 && Array.isArray(arguments[0])) {
                var iterable = arguments[0];
                for (var i = 0, len = iterable.length; i < len; i++) {
                    m.set(iterable[i][0], iterable[i][1]);
                }
            }
            return m;
        };
    }

    if (!hasForEach) {
        var OldMap = global.Map;
        global.Map = function Map() {
            var map = new OldMap();
            var size = 0;
            var keys = [];
            var m = Object.create(Map.prototype, {
                size: {
                    get : function () { return size; },
                    configurable: false,
                    enumerable: false
                },
                get: {
                    value: function (key) {
                        return map.get(key);
                    },
                    writable: false,
                    configurable: false,
                    enumerable: false
                },
                set: {
                    value: function (key, value) {
                        if (!map.has(key)) {
                            keys.push(key);
                            size++;
                        }
                        map.set(key, value);
                    },
                    writable: false,
                    configurable: false,
                    enumerable: false
                },
                has: {
                    value: function (key) {
                        return map.has(key);
                    },
                    writable: false,
                    configurable: false,
                    enumerable: false
                },
                'delete': {
                    value: function (key) {
                        if (map.has(key)) {
                            size--;
                            keys.splice(keys.indexOf(key), 1);
                            return map['delete'](key);
                        }
                        return false;
                    },
                    writable: false,
                    configurable: false,
                    enumerable: false
                },
                clear: {
                    value: function () {
                        if ('clear' in map) {
                            map.clear();
                        }
                        else {
                            for (var i = 0, n = keys.length; i < n; i++) {
                                map['delete'](keys[i]);
                            }
                        }
                        keys.length = 0;
                        size = 0;
                    },
                    writable: false,
                    configurable: false,
                    enumerable: false
                },
                forEach: {
                    value: function (callback, thisArg) {
                        for (var i = 0, n = keys.length; i < n; i++) {
                            callback.call(thisArg, this.get(keys[i]), keys[i], this);
                        }
                    },
                    writable: false,
                    configurable: false,
                    enumerable: false
                }
            });
            if (arguments.length > 0 && Array.isArray(arguments[0])) {
                var iterable = arguments[0];
                for (var i = 0, len = iterable.length; i < len; i++) {
                    m.set(iterable[i][0], iterable[i][1]);
                }
            }
            return m;
        };
    }
})(cs.global);

```

# V8 介绍

哪些程序用到 V8

- Chrome 浏览器的 JS 引擎是 V8
- Nodejs 的运行时环境是 V8
- electron 的底层引擎是 V8 跨平台桌面应用开发工具

blink 是渲染引擎，V8 是 JS 引擎

访问 Dom 的接口是由 Blink 提供的

## 功能

接收 JavaScript 代码，编译代码后执行 C++程序，编译后的代码可以在多种操作系统多种处理器上运行。

1. 编译和执行 JS 代码
2. 处理调用栈
3. 内存分配
4. 垃圾回收

## V8 的 js 编译和执行

- 解析器 parser js --> 解析成功抽象语法树 AST
- 解释器 interpreter AST --> 字节码 bytecode，也有直接执行字节码的能力
- 编译器 compiler bytecode --> 更高效的机器码

V8 版本 5.9 之前没有解释器，但是有两个编译器

## 5.9 版本的 V8

1. parser 解释器生成抽象语法树 AST
2. compiler 编译器 Full-codegen 基准编译器 直接生成机器码
3. 运行一段时间后，由分析器线程优化 js 代码
4. compiler 编译器 CrankShaft 优化编译器 重新生成 AST 提升运行效率

这样设计的缺点

1. 机器码会占用大量的内存
2. 缺少中间层机器码，无法实现一些优化策略
3. 无法很好的支持和优化 JS 的新语特性，无法拥抱未来

## 新版本的 V8

1. parser 解析器 生成 AST 抽象语法树
2. interpreter 解释器 Ignition 生成 byteCode 字节码 并直接执行
3. 清除 AST 释放内存空间
4. 得到 25% - 50%的等效机器代码大小
5. compiler 运行过程中，解释器收集优化信息发送给编译器 TurboFan
6. 重新生成机器码
7. 有些热点函数变更会由优化后的机器码还原成字节码 也就是 deoptimization 回退字节码操作执行

优化点：

1. 值声明未调用，不会被解析生成 AST
2. 函数只被调用一次，bytcode 直接被解释执行，不会进入到编译优化阶段
3. 函数被调用多次，Igniton 会收集函数类型信息，可能会被标记为热点函数，可能被编译成优化后的机器代码

好处：

1. 由于一开始不需要直接编译成机器码，生成了中间层的字节码，从而节约了时间
2. 优化编译阶段，不需要从源码重新解析,直接通过字节码进行优化，也可以 deoptimization 回退操作

```javascript
function sum(x, y) {
  return x + y
}
sum(1, 2)
sum(3, 4)
sum(5, 6)
sum('7', '8') //会回退字节码操作执行
```

# Utils

> An Utils.
