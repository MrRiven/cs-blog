# px 转 rpx 自适应布局

## 安装

> npm

```bash
npm i postcss-pxtorpx-pro
```

## 使用

> postcss.config.js

```js
const path = require("path");
module.exports = {
  parser: require("postcss-comment"),
  plugins: [
    require("postcss-pxtorpx-pro")({
      // 转化的单位
      unit: "rpx",
      // 单位精度
      unitPrecision: 5,
      // 不需要处理的css选择器
      selectorBlackList: [],
      // 不需要转化的css属性
      propBlackList: [],
      // 直接修改px，还是新加一条css规则
      replace: true,
      // 是否匹配媒介查询的px
      mediaQuery: false,
      // 需要转化的最小的pixel值，低于该值的px单位不做转化
      minPixelValue: 0,
      // 不处理的文件
      exclude: /node_modules|components|wxcomponents/gi,
      // 转化函数
      // 默认设计稿按照750宽，2倍图的出
      transform: (x) => x,
    }),
    require("postcss-import")({
      resolve(id, basedir, importOptions) {
        if (id.startsWith("~@/")) {
          return path.resolve(process.env.UNI_INPUT_DIR, id.substr(3));
        } else if (id.startsWith("@/")) {
          return path.resolve(process.env.UNI_INPUT_DIR, id.substr(2));
        } else if (id.startsWith("/") && !id.startsWith("//")) {
          return path.resolve(process.env.UNI_INPUT_DIR, id.substr(1));
        }
        return id;
      },
    }),
    require("autoprefixer")({
      remove: process.env.UNI_PLATFORM !== "h5",
    }),
    require("@dcloudio/vue-cli-plugin-uni/packages/postcss"),
  ],
};
```
