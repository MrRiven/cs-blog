# 适配 iPhoneX

> iPhoneX 的出现将手机的颜值带上了一个新的高度，它取消了物理按键，改成了底部的小黑条，但是这样的改动给开发者适配移动端又增加了难度。

## viewport-fit

> viewport-fit 是专门为了适配 iPhoneX 而诞生的一个属性，它用于限制网页如何在安全区域内进行展示。

- contain: 可视窗口完全包含网页内容
- cover：网页内容完全覆盖可视窗口
- 默认情况下或者设置为 auto 和 contain 效果相同。

## env、constant

> 我们需要将顶部和底部合理的摆放在安全区域内，iOS11 新增了两个 CSS 函数 env、constant，用于设定安全区域与边界的距离。

函数内部可以是四个常量：

- safe-area-inset-left：安全区域距离左边边界距离
- safe-area-inset-right：安全区域距离右边边界距离
- safe-area-inset-top：安全区域距离顶部边界距离
- safe-area-inset-bottom：安全区域距离底部边界距离

注意：我们必须指定 viweport-fit 后才能使用这两个函数：

```
<meta name="viewport" content="viewport-fit=cover">
```

constant 在 iOS < 11.2 的版本中生效，env 在 iOS >= 11.2 的版本中生效，这意味着我们往往要同时设置他们，将页面限制在安全区域内：

```
body {
  padding-bottom: constant(safe-area-inset-bottom);
  padding-bottom: env(safe-area-inset-bottom);
}
```

当使用底部固定导航栏时，我们要为他们设置 padding 值：

```
{
  padding-bottom: constant(safe-area-inset-bottom);
  padding-bottom: env(safe-area-inset-bottom);
}
```
