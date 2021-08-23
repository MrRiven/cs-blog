# vue 中集成 UEditor 富文本编辑器 包含 秀米

> 需在 public 目录下引入 UEditor 字体和样式文件

> [见附件](https://github.com/MrRiven/cs-blog/tree/main/public/UEditor)

## 安装

```bash
npm i vue-ueditor-wrap -S
```

## 实现

```vue
<template>
  <div>
    <vue-ueditor-wrap
      ref="editor"
      v-model="content"
      :config="myConfig"
      @beforeInit="addCustomButtom"
    />
  </div>
</template>

<script>
import VueUeditorWrap from 'vue-ueditor-wrap'
import '../../../public/UEditor/ueditor.config.js'
import '../../../public/UEditor/ueditor.all.js'
import '../../../public/UEditor/xiumi-ue-dialog-v5.js'
import '../../../public/UEditor/xiumi-ue-v5.css'
import request from '@/utils/request'

export default {
  components: {
    VueUeditorWrap,
  },
  props: {
    /* 编辑器的内容 */
    value: {
      type: String,
      default: '',
    },
  },
  data() {
    return {
      content: this.value,
      myConfig: {
        autoHeightEnabled: false,
        autoHeight: false,
        // 初始容器宽度
        initialFrameWidth: '100%',
        initialFrameHeight: 300,
        // 上传文件接口（这个地址是我为了方便各位体验文件上传功能搭建的临时接口，请勿在生产环境使用！！！）
        // serverUrl: 'http://35.201.165.105:8000/controller.php'
      },
    }
  },
  watch: {
    value() {
      this.content = this.value
    },
    content() {
      this.$emit('input', this.content)
    },
  },
  created() {},
  methods: {
    addCustomButtom(editorId) {
      const _this = this
      window.UE.registerUI(
        'test-button',
        function (editor, uiName) {
          // 注册按钮执行时的 command 命令，使用命令默认就会带有回退操作
          editor.registerCommand(uiName, {
            execCommand: function () {
              editor.execCommand('inserthtml', ``)
            },
          })

          // 创建一个 button
          var btn = new window.UE.ui.Button({
            // 按钮的名字
            name: uiName,
            // 提示
            title: '自定义上传',
            // 需要添加的额外样式，可指定 icon 图标，图标路径参考常见问题 2
            cssRules:
              // eslint-disable-next-line quotes
              "background-image: url('/UEditor/test-img.png') !important;background-size: cover;",
            // 点击时执行的命令
            onclick: function () {
              // 这里可以不用执行命令，做你自己的操作也可
              editor.execCommand(uiName)
              var input = document.createElement('input')
              input.type = 'file'
              input.style.display = 'none'
              document.body.appendChild(input)
              input.click()
              input.addEventListener('change', (e) => {
                // 利用 AJAX 上传，上传成功之后销毁 DOM
                const fd = new FormData()

                fd.append('file', e.target.files[0])
                _this.$loading({
                  fullscreen: true,
                  lock: true,
                  text: '上传中...',
                })
                request({
                  url: '/hy-goods/bms/common/uploadNew',
                  method: 'post',
                  data: fd,
                  config: {
                    headers: {
                      'Content-Type': 'multipart/form-data',
                    },
                  },
                })
                  .then((res) => {
                    _this.$loading().close()
                    let insertHtml = ''
                    if (e.target.files[0].type.indexOf('image/') !== -1) {
                      insertHtml = `<img src='${res.url}' />`
                    } else if (
                      e.target.files[0].type.indexOf('video/') !== -1
                    ) {
                      insertHtml = `<p><video src='${res.url}' style='width:385px;'controls >你的版本不知道Video视频播放,请升级游览器版本</video></p>`
                    }
                    editor.execCommand('inserthtml', insertHtml)
                    document.body.removeChild(input)
                  })
                  .catch((err) => {
                    _this.$loading().close()
                    console.error(`上传图片失败---${err}`)
                    document.body.removeChild(input)
                  })
              })
            },
          })

          // 当点到编辑内容上时，按钮要做的状态反射
          editor.addListener('selectionchange', function () {
            var state = editor.queryCommandState(uiName)
            if (state === -1) {
              btn.setDisabled(true)
              btn.setChecked(false)
            } else {
              btn.setDisabled(false)
              btn.setChecked(state)
            }
          })

          // 因为你是添加 button，所以需要返回这个 button
          return btn
        },
        0 /* 指定添加到工具栏上的哪个位置，默认时追加到最后 */,
        editorId /* 指定这个 UI 是哪个编辑器实例上的，默认是页面上所有的编辑器都会添加这个按钮 */
      )
    },
  },
}
</script>

<style lang="scss" scoped></style>
```

## 使用

```vue
<template>
  ...
  <Editor v-model="value" />
  ...
</template>

<script>
import Editor from './Editor'

export default {
  components: {
    Editor,
  },
  data() {
    return {
      value: '',
    }
  },
}
</script>
```
