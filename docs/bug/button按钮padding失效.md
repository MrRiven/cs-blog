# 小程序中 button 按钮 padding 失效

## -webkit-appearance 重置样式导致

```css
button,
html [type="button"],
[type="reset"],
[type="submit"] {
  -webkit-appearance: button;
  // 真机已失效
  // ios 14.x height高度也会撑不起盒子
}
```
