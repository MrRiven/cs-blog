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
