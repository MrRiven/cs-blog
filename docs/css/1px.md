# 1px 问题

## 为什么会有 1px 问题

- 要处理这个问题，必须先补充一个知识点，就是设备的 物理像素[设备像素] & 逻辑像素[CSS 像素]；
  - 物理像素： 移动设备出厂时，不同设备自带的不同像素，也称硬件像素；
  - 逻辑像素：即 css 中记录的像素。

## 媒体查询利用设备像素比缩放，设置小数像素；

```
.border { border: 1px solid #999 }
@media screen and (-webkit-min-device-pixel-ratio: 2) {
    .border { border: 0.5px solid #999 }
}
@media screen and (-webkit-min-device-pixel-ratio: 3) {
    .border { border: 0.333333px solid #999 }
}
```

```
<body><div id="main" style="border: 1px solid #000000;"></div></body>
<script type="text/javascript">
    if (window.devicePixelRatio && devicePixelRatio >= 2) {
        var main = document.getElementById('main');
        main.style.border = '.5px solid #000000';
    }
</script>
```

## 设置 border-image 方案

```
.border-image-1px {
    border-width: 1px 0px;
    -webkit-border-image: url("border.png") 2 0 stretch;
    border-image: url("border.png") 2 0 stretch;
}
```

## background-image 渐变实现

```
.border {
      background-image:linear-gradient(180deg, red, red 50%, transparent 50%),
      linear-gradient(270deg, red, red 50%, transparent 50%),
      linear-gradient(0deg, red, red 50%, transparent 50%),
      linear-gradient(90deg, red, red 50%, transparent 50%);
      background-size: 100% 1px,1px 100% ,100% 1px, 1px 100%;
      background-repeat: no-repeat;
      background-position: top, right top,  bottom, left top;
      padding: 10px;
}
```

## box-shadow

```
div {
    -webkit-box-shadow: 0 1px 1px -1px rgba(0, 0, 0, 0.5);
}
```

## viewport + rem

```
<meta name="viewport" id="WebViewport" content="initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no">
.
.
.
var viewport = document.querySelector("meta[name=viewport]")
if (window.devicePixelRatio == 1) {
    viewport.setAttribute('content', 'width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no')
}
if (window.devicePixelRatio == 2) {
    viewport.setAttribute('content', 'width=device-width, initial-scale=0.5, maximum-scale=0.5, minimum-scale=0.5, user-scalable=no')
}
if (window.devicePixelRatio == 3) {
    viewport.setAttribute('content', 'width=device-width, initial-scale=0.333333333, maximum-scale=0.333333333, minimum-scale=0.333333333, user-scalable=no')
}
var docEl = document.documentElement;
var fontsize = 10 * (docEl.clientWidth / 320) + 'px';
docEl.style.fontSize = fontsize;
```

## transform: scale(0.5) 方案 - 推荐: 很灵活

- 1.) 设置 height: 1px，根据媒体查询结合 transform 缩放为相应尺寸。

```
div {
    height:1px;
    background:#000;
    -webkit-transform: scaleY(0.5);
    -webkit-transform-origin:0 0;
    overflow: hidden;
}
```

- 2.) 用::after 和::befor,设置 border-bottom：1px solid #000,然后在缩放-webkit-transform: scaleY(0.5);可以实现两根边线的需求

```
div::after{
    content:'';width:100%;
    border-bottom:1px solid #000;
    transform: scaleY(0.5);
}
```

- 3.) 用::after 设置 border：1px solid #000; width:200%; height:200%,然后再缩放 scaleY(0.5); 优点可以实现圆角，京东就是这么实现的，缺点是按钮添加 active 比较麻烦。

```
.div::after {
    content: '';
    width: 200%;
    height: 200%;
    position: absolute;
    top: 0;
    left: 0;
    border: 1px solid #bfbfbf;
    border-radius: 4px;
    -webkit-transform: scale(0.5,0.5);
    transform: scale(0.5,0.5);
    -webkit-transform-origin: top left;
}
```

## 媒体查询 + transfrom 对方案 1 的优化

```
/* 2倍屏 */
@media only screen and (-webkit-min-device-pixel-ratio: 2.0) {
    .border-bottom::after {
        -webkit-transform: scaleY(0.5);
        transform: scaleY(0.5);
    }
}
/* 3倍屏 */
@media only screen and (-webkit-min-device-pixel-ratio: 3.0) {
    .border-bottom::after {
        -webkit-transform: scaleY(0.33);
        transform: scaleY(0.33);
    }
}
```

## svg

- 借助 PostCSS 的 postcss-write-svg 我们能直接使用 border-image 和 background-image 创建 svg 的 1px 边框：

```
@svg border_1px {
  height: 2px;
  @rect {
    fill: var(--color, black);
    width: 100%;
    height: 50%;
    }
  }
.example { border: 1px solid transparent; border-image: svg(border_1px param(--color #00b1ff)) 2 2 stretch; }
```

编译成

```
.example { border: 1px solid transparent; border-image: url("data:image/svg+xml;charset=utf-8,%3Csvg xmlns='http://www.w3.org/2000/svg' height='2px'%3E%3Crect fill='%2300b1ff' width='100%25' height='50%25'/%3E%3C/svg%3E") 2 2 stretch; }

```
