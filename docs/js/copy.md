# js 复制

```
import { Message } from 'element-ui'

window.Clipboard = (function (window, document, navigator) {
  var textArea, copy

  // 判断是不是ios端
  function isOS() {
    return navigator.userAgent.match(/ipad|iphone/i)
  }
  //创建文本元素
  function createTextArea(text) {
    textArea = document.createElement('textArea')
    textArea.value = text
    document.body.appendChild(textArea)
  }
  //选择内容
  function selectText() {
    var range, selection

    if (isOS()) {
      range = document.createRange()
      range.selectNodeContents(textArea)
      selection = window.getSelection()
      selection.removeAllRanges()
      selection.addRange(range)
      textArea.setSelectionRange(0, 999999)
    } else {
      textArea.select()
    }
  }

  //复制到剪贴板
  function copyToClipboard(msg) {
    try {
      if (document.execCommand('Copy')) {
        Message({
          message: msg || '复制成功',
          type: 'success'
        })
      } else {
        Message({
          message: msg || '复制错误！请手动复制！',
          type: 'error'
        })
      }
    } catch (err) {
      Message({
        message: msg || '复制错误！请手动复制！',
        type: 'error'
      })
    }
    document.body.removeChild(textArea)
  }

  copy = function (text, msg = '') {
    createTextArea(text)
    selectText()
    copyToClipboard(msg)
  }

  return {
    copy: copy
  }
})(window, document, navigator)
```
