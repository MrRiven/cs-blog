# reset css

## 使用

> app.vue

```vue
<style lang="scss">
/*每个页面公共css */
*,
*::before,
*::after,
view,
text,
rich-text,
button,
image,
scroll-view {
  box-sizing: border-box;
}

html {
  -webkit-text-size-adjust: 100%;
  -webkit-tap-highlight-color: transparent;
}

body {
  margin: 0;
  -webkit-overflow-scrolling: touch;
  background-color: #f7f7f7;
}

h1,
h2,
h3,
h4,
h5,
h6,
p,
ul,
ol,
dl,
input,
button,
select,
optgroup,
textarea {
  margin: 0;
  padding: 0;
}

input,
textarea,
select,
button {
  font: 28rpx 'PingFangSC-Regular', 'monospace', 'sans', 'serif', 'Microsoft YaHei';
}
i {
  font-style: normal;
}
ol,
ul,
dl {
  list-style: none;
}

a {
  text-decoration: none;
}

input {
  background: none;
  outline: none;
  border: 0px;
}

img,
image {
  vertical-align: middle;
  border-style: none;
}

a,
area,
button,
label,
select,
textarea {
  -ms-touch-action: manipulation;
  touch-action: manipulation;
}

table {
  border-collapse: collapse;
}

label {
  display: inline-block;
}

button,
html [type='button'],
[type='reset'],
[type='submit'] {
  -webkit-appearance: button;
}

.text-ellipsis-1 {
  overflow: hidden;
  word-break: break-all;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 1;
  -webkit-box-orient: vertical;
}

.text-ellipsis-2 {
  overflow: hidden;
  word-break: break-all;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}

.text-ellipsis-3 {
  overflow: hidden;
  word-break: break-all;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
}
</style>
```
