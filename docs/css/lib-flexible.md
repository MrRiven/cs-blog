# lib-flexible

## 关于 lib-flexible

> lib-flexible 会自动在 html 的 head 中添加一个 meta name="viewport"的标签，同时会自动设置 html 的 font-size 为屏幕宽度除以 10，也就是 1rem 等于 html 根节点的 font-size。假如设计稿的宽度是 750px，此时 1rem 应该等于 75px。假如量的某个元素的宽度是 150px，那么在 css 里面定义这个元素的宽度就是 width: 2rem。但是当分辨率大于某个特定值时，它便不再生效。

## 移动端适配步骤

> 一般而言，lib-flexible 并不独立出现，而是搭配 px2rem-loader 一起做适配方案，目的是自动将 css 中的 px 转换成 rem。以下为它在 vue 中的使用。

## 安装 lib-flexible

```
npm install lib-flexible --save-dev
```

## 引入 lib-flexible

main.js

```
import 'lib-flexible' // px2rem 自适应
```

## 安装 px2rem-loader

```
npm install px2rem-loader --save-dev
```

## 配置 px2rem-loader

- vue-cli 2.x

  - 在 build/utils.js 中，找到 exports.cssLoaders，作出如下修改：

  ```
  const px2remLoader = {
    loader: 'px2rem-loader',
    options: {
        remUint: 75 // 以设计稿750为例， 750 / 10 = 75
    }
  }
  ```

  - 继续找到 generateLoaders 中的 loaders 配置，作出如下配置

  ```
  // 注释掉这一行
  // const loaders = options.usePostCSS ? [cssLoader, postcssLoader] : [cssLoader]
  // 修改为
  const loaders = [cssLoader, px2remLoader]
  if (options.usePostCSS) {
  loaders.push(postcssLoader)
  }
  ```

  - 重新 npm run dev，完成

- vue-cli 3.x
  - 根目录新建文件 vue.config.js，然后如下配置：
  ```
  module.exports = {
    css: {
        loaderOptions: {
            css: {},
            postcss: {
                plugins: [
                    require('postcss-px2rem')({
                        // 以设计稿750为例， 750 / 10 = 75
                        remUnit: 75
                    }),
                ]
            }
        }
    },
  };
  ```
  - 重新 npm run dev，完成

## 大屏不生效

> 打开./node_modules/lib-flexible/flexible.js，找到如下片段源码：

```
function refreshRem(){
    var width = docEl.getBoundingClientRect().width;
    if (width / dpr > 540) {
        width = 540 * dpr;
    }
    var rem = width / 10;
    docEl.style.fontSize = rem + 'px';
    flexible.rem = win.rem = rem;
}
```

从此段源码中我们不难看出，当屏幕宽度除以 dpr（dpr 是一个倍数，此处一般为 1）大于 540 这个特定值的时候，那么就不再进行适配了。那么我们如何解决这个问题呢？

## 修改源码

在上述源码中，进行修改。例如我要适配的大屏幕尺寸是基于 3840 的设计稿，而要求最小范围是 1980，最大为 5760，那么我们要修改的则变为：

```
function refreshRem(){
    var width = docEl.getBoundingClientRect().width;
    if (width / dpr < 1980) {
        width = 1980 * dpr;
    } else if (width / dpr > 5760) {
        width = 5760 * dpr;
    }
    var rem = width / 10;
    docEl.style.fontSize = rem + 'px';
    flexible.rem = win.rem = rem;
}
```

## 重启，完成
