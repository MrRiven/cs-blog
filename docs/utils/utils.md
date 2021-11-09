## 时间格式化

```js
/**
 * Parse the time to string
 * @param {(Object|string|number)} time
 * @param {string} cFormat
 * @returns {string | null}
 */
export function parseTime(time, cFormat) {
  if (arguments.length === 0) {
    return null
  }
  const format = cFormat || '{y}-{m}-{d} {h}:{i}:{s}'
  let date
  if (typeof time === 'object') {
    date = time
  } else {
    if (typeof time === 'string' && /^[0-9]+$/.test(time)) {
      time = parseInt(time)
    }
    if (typeof time === 'number' && time.toString().length === 10) {
      time = time * 1000
    }
    date = new Date(time)
  }
  const formatObj = {
    y: date.getFullYear(),
    m: date.getMonth() + 1,
    d: date.getDate(),
    h: date.getHours(),
    i: date.getMinutes(),
    s: date.getSeconds(),
    a: date.getDay(),
  }
  const timeStr = format.replace(/{([ymdhisa])+}/g, (result, key) => {
    const value = formatObj[key]
    // Note: getDay() returns 0 on Sunday
    if (key === 'a') {
      return ['日', '一', '二', '三', '四', '五', '六'][value]
    }
    return value.toString().padStart(2, '0')
  })
  return timeStr
}
```

## 返回多久前

```js
/**
 * @param {number} time
 * @param {string} option
 * @returns {string}
 */
export function formatTime(time, option) {
  if (('' + time).length === 10) {
    time = parseInt(time) * 1000
  } else {
    time = +time
  }
  const d = new Date(time)
  const now = Date.now()

  const diff = (now - d) / 1000

  if (diff < 30) {
    return '刚刚'
  } else if (diff < 3600) {
    // less 1 hour
    return Math.ceil(diff / 60) + '分钟前'
  } else if (diff < 3600 * 24) {
    return Math.ceil(diff / 3600) + '小时前'
  } else if (diff < 3600 * 24 * 2) {
    return '1天前'
  }
  if (option) {
    return parseTime(time, option)
  } else {
    return (
      d.getMonth() +
      1 +
      '月' +
      d.getDate() +
      '日' +
      d.getHours() +
      '时' +
      d.getMinutes() +
      '分'
    )
  }
}
```

## 判断设备信息

```js
/**
 * @description 判断设备信息
 * @returns {String}
 */
export function judgeDevice(key = 'app') {
  const ua = window.navigator.userAgent

  if (ua.toLowerCase().includes(key)) {
    // app 端
    if (ua.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/)) {
      // ios端
      return 'ios'
    } else if (ua.indexOf('Android') > -1 || ua.indexOf('Adr') > -1) {
      // android端
      return 'android'
    }
  } else {
    if (ua.toLocaleLowerCase().includes('micromessenger')) {
      // 微信浏览器
      return 'wx'
    } else {
      // 其他浏览器
      return 'other-browser'
    }
  }
}
```

## 隐藏手机号中间几位

```js
/**
 * @description 隐藏手机号中间几位
 * @param {String} phone 手机号
 *   @returns {String}
 */
export function noPassByMobile(phone) {
  if (phone) {
    const reg = /(\d{3})\d*(\d{4})/
    return phone.replace(reg, '$1****$2')
  } else {
    return ''
  }
}
```

## 替换字符串

```js
/**
 * 替换字符串
 * @param {*} str 被替换内容
 * @param {*} start 开始位置从1
 * @param {*} end
 * @param {*} changeStr 替换内容
 * @returns
 */
export function replaceStr(str, start, end, changeStr) {
  if (!str) {
    return ''
  }
  return (
    str.substring(0, start - 1) + changeStr + str.substring(end, str.length)
  )
}
```

## 判断端终访问

```js
// 判断端终访问
export const browser = {
  versions: (function () {
    const u = navigator.userAgent
    return {
      trident: u.indexOf('Trident') > -1, // IE内核
      presto: u.indexOf('Presto') > -1, // opera内核
      webKit: u.indexOf('AppleWebKit') > -1, // 苹果、谷歌内核
      gecko: u.indexOf('Gecko') > -1 && u.indexOf('KHTML') === -1, // 火狐内核
      mobile: !!u.match(/AppleWebKit.*Mobile.*/), // 是否为移动终端
      ios: !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/), // ios终端
      android: u.indexOf('Android') > -1 || u.indexOf('Adr') > -1, // android终端
      iPhone: u.indexOf('iPhone') > -1, // 是否为iPhone或者QQHD浏览器
      iPad: u.indexOf('iPad') > -1, // 是否iPad
      webApp: u.indexOf('Safari') === -1, // 是否web应该程序，没有头部与底部
      weixin: u.indexOf('MicroMessenger') > -1, // 是否微信 （2015-01-22新增）
      qq: u.match(/\sQQ/i) === ' qq', // 是否QQ
    }
  })(),
  language: (navigator.browserLanguage || navigator.language).toLowerCase(),
}
```

## 移除 html 字符串上所有的样式

```js
/* 移除html字符串上所有的样式 */
export function removeHtmlAllStyle(htmlStr) {
  return htmlStr
    ? htmlStr
        .replace(/[ \t]*style[ \t]*=[ \t]*("[^"]+")|('[^']+')/gi, '')
        .replace(
          /[ \t]*width[ \t]*=[ \t]*("[^"]+")|('[^']+')/gi,
          `width="100%"`
        )
        .replace(
          /[ \t]*height[ \t]*=[ \t]*("[^"]+")|('[^']+')/gi,
          `height="auto"`
        )
    : ''
}
```

## 判断网络

```js
export function isWifi() {
  try {
    let wifi = true
    const ua = window.navigator.userAgent
    const con = window.navigator.connection
    // 如果是微信
    if (/MicroMessenger/.test(ua)) {
      if (ua.indexOf('WIFI') >= 0) {
        return true
      } else {
        wifi = false
      }
      // 如果支持navigator.connection
    } else if (con) {
      const network = con.type
      if (network !== 'wifi' && network !== '2' && network !== 'unknown') {
        wifi = false
      }
    }
    return wifi
  } catch (e) {
    return false
  }
}
```

## 判断是元素是否在可视区内

```js
/**
 *判断是元素是否在可视区内
 * @export
 * @param {*} el
 * @returns
 */
export function isInViewPortOfTwo(el) {
  const viewPortHeight =
    window.innerHeight ||
    document.documentElement.clientHeight ||
    document.body.clientHeight
  const top = el.getBoundingClientRect() && el.getBoundingClientRect().top
  console.log('top', top)
  return top <= viewPortHeight
}
```

## 函数节流

```js
/**
 * 函数节流
 * @export
 * @param {Function} callback 需要接口的方法
 * @param {Int} delay 延迟执行时间
 * @returns
 */
export function throttle(delay, callback) {
  let timeoutID
  let lastExec = 0

  function wrapper() {
    const self = this
    const elapsed = Number(new Date()) - lastExec
    const args = arguments

    function exec() {
      lastExec = Number(new Date())
      callback.apply(self, args)
    }

    clearTimeout(timeoutID)

    if (elapsed > delay) {
      exec()
    } else {
      timeoutID = setTimeout(exec, delay - elapsed)
    }
  }

  return wrapper
}
```

## 函数防抖

```js
/**
 * 函数防抖
 * @export
 * @param {Function} callback 需要接口的方法
 * @param {Int} delay 延迟执行时间
 * @returns
 */
export function debounce(callback, delay) {
  let timeout
  return function () {
    let context = this
    let args = arguments

    if (timeout) clearTimeout(timeout)

    timeout = setTimeout(() => {
      callback.apply(context, args)
    }, delay)
  }
}
```

## 获取百分比，主要用于图片

```js
/**
 * @description 获取百分比，主要用于图片
 * @export
 * @param {*} w 宽
 * @param {*} h 高
 * @returns 百分比
 */
export function getPercent(w, h) {
  let ratio = Math.floor((h / w) * 100)
  return ratio
}
```

## 多维数组去重 根据某一唯一 Key

```js
export function unique(arr, key) {
  const res = new Map()
  return arr.filter((arr) => !res.has(arr[key]) && res.set(arr[key], 1))
}
```

## 首字母大小

```js
export function titleCase(str) {
  return str.replace(/( |^)[a-z]/g, (L) => L.toUpperCase())
}
```

## 下划转驼峰

```js
export function camelCase(str) {
  return str.replace(/-[a-z]/g, (str1) => str1.substr(-1).toUpperCase())
}
```

## 下载 excel URL

```js
/**
 * 下载excel
 * @param {*} url
 */
export function downLoadUrl(url) {
  let iframe = document.createElement('iframe')
  iframe.style.display = 'none'
  iframe.src = url
  iframe.onload = function () {
    document.body.removeChild(iframe)
  }
  document.body.appendChild(iframe)
}
```

## 下载 excel Post 下载数据流

```js
/**
 * 下载excel
 * @param {content} 文本
 * @param {fileName} 文件名
 */
export function downLoadInPost(
  content,
  fileName,
  downType = 'application/vnd.ms-excel'
) {
  const blob = new Blob([content])
  if ('download' in document.createElement('a')) {
    // 非IE下载
    const elink = document.createElement('a')
    elink.download = fileName
    elink.style.display = 'none'
    elink.href = URL.createObjectURL(blob)
    document.body.appendChild(elink)
    elink.click()
    URL.revokeObjectURL(elink.href) // 释放URL 对象
    document.body.removeChild(elink)
  } else {
    // IE10+下载
    navigator.msSaveBlob(blob, fileName)
  }
}
```

## 下载 excel Link

```js
/**
 * 下载excel
 * @param {content} link
 * @param {fileName} 文件名
 */
export function downLoadInLink(url, fileName) {
  // , {type: "application/vnd.ms-excel"}
  if ('download' in document.createElement('a')) {
    // 非IE下载
    const elink = document.createElement('a')
    elink.download = fileName
    elink.style.display = 'none'
    elink.href = url
    elink.target = '_blank'
    document.body.appendChild(elink)
    elink.click()
    document.body.removeChild(elink)
  } else {
    // IE10+下载
    navigator.msSaveBlob(url, fileName)
  }
}
```

## 浮点数加法运算

```js
export function FloatAdd(a, b) {
  var c, d, e
  try {
    c = a.toString().split('.')[1] ? a.toString().split('.')[1].length : 0
  } catch (f) {
    c = 0
  }
  try {
    d = b.toString().split('.')[1] ? b.toString().split('.')[1].length : 0
  } catch (f) {
    d = 0
  }
  return (
    (e = Math.pow(10, Math.max(c, d))), (FloatMul(a, e) + FloatMul(b, e)) / e
  ) // eslint-disable-line
}
```

## 浮点数减法运算

```js
export function FloatSub(a, b) {
  var c, d, e
  try {
    c = a.toString().split('.')[1].length
  } catch (f) {
    c = 0
  }
  try {
    d = b.toString().split('.')[1].length
  } catch (f) {
    d = 0
  }
  return (
    (e = Math.pow(10, Math.max(c, d))), (FloatMul(a, e) - FloatMul(b, e)) / e
  ) // eslint-disable-line
}
```

## 浮点数乘法运算

```js
export function FloatMul(a, b) {
  var c = 0
  var d = a.toString()
  var e = b.toString()
  try {
    c += d.split('.')[1] ? d.split('.')[1].length : 0
  } catch (f) {
    console.log(f)
  }
  try {
    c += e.split('.')[1] ? e.split('.')[1].length : 0
  } catch (f) {
    console.log(f)
  }
  return (
    (Number(d.replace('.', '')) * Number(e.replace('.', ''))) / Math.pow(10, c)
  )
}
```

## 获取两个数值的小数点后面数字的长度

```js
/**
 * 获取两个数值的小数点后面数字的长度
 *
 * @inner
 * @param {number} num1 第一个数值
 * @param {number} num2 第二个数值
 * @return {Array} 每个数值小数点后面数字的个数
 */
function decimalLength(num1, num2) {
  var length1
  var length2
  try {
    length1 = num1.toString().split('.')[1].length
  } catch (e) {
    length1 = 0
  }
  try {
    length2 = num2.toString().split('.')[1].length
  } catch (e) {
    length2 = 0
  }
  return [length1, length2]
}
```

## 浮点数除法运算

```js
/**
 * 两个浮点数相除
 *
 * @public
 * @param {number} num1 第一个数值
 * @param {number} num2 第二个数值
 * @return {number} 相除的结果
 */
export function FloatDivide(num1, num2) {
  var result = decimalLength(num1, num2)
  var length1 = result[0]
  var length2 = result[1]
  // var maxLength = Math.max(length1, length2)
  var integer1 = +num1.toString().replace('.', '')
  var integer2 = +num2.toString().replace('.', '')
  // 默认保留小数点最长的个数
  return (
    Math.ceil((integer1 / integer2) * Math.pow(10, length2 - length1) * 100) /
    100
  )
}
```

## 保留小数

```js
/**
 * 保留小数
 * @param {*} x
 */
export function toDecimal(x, num = 100) {
  var f = parseFloat(x)
  if (isNaN(f)) {
    return
  }
  f = Math.ceil(x * num) / num
  return f
}
```

## 秒转成对象

```js
/**
 * 秒转成对象
 * @return {*}
 */
export function formatSeconds(value) {
  var ss = parseInt(value / 1000) // 秒
  var mm = 0 // 分
  var hh = 0 // 小时
  var dd = 0
  if (ss > 60) {
    mm = parseInt(ss / 60)
    ss = parseInt(ss % 60)
    if (mm > 60) {
      hh = parseInt(mm / 60)
      mm = parseInt(mm % 60)
      if (hh > 24) {
        dd = parseInt(hh / 24)
        hh = parseInt(hh % 24)
      }
    }
  }
  return {
    dd,
    hh,
    mm,
    ss,
  }
}
```
