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
