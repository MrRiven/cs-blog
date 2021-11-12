# 获取系统状态栏高度和导航栏高度

```js
uni.getSystemInfo({
  success: (e) => {
    let statusBar = 0
    let customBar = 0

    // #ifdef MP
    statusBar = e.statusBarHeight
    customBar = e.statusBarHeight + 45
    if (e.platform === 'android') {
      customBar = e.statusBarHeight + 50
    }
    // #endif

    // #ifdef MP-WEIXIN
    statusBar = e.statusBarHeight
    const custom = wx.getMenuButtonBoundingClientRect()
    customBar = custom.bottom + custom.top - e.statusBarHeight
    // #endif

    // #ifdef MP-ALIPAY
    statusBar = e.statusBarHeight
    customBar = e.statusBarHeight + e.titleBarHeight
    // #endif

    // #ifdef APP-PLUS
    statusBar = e.statusBarHeight
    customBar = e.statusBarHeight + 45
    // #endif

    // #ifdef H5
    statusBar = 0
    customBar = e.statusBarHeight + 45
    // #endif

    console.log(`statusBar=${statusBar}`)
    console.log(`customBar=${customBar}`)
  },
})
```
