# wangeditor 富文本编辑器

> vue 中使用 wangeditor

## 安装

```bash
npm i wangeditor -S
```

## 实现

```vue
<template lang="pug">
div
  div(:id='editorId', :key='editorId')
</template>

<script>
import E from 'wangeditor'
import OSS from 'ali-oss'
import uuid from 'uuid'
import dayjs from 'dayjs'
import { getSTS, apiQueryProductBySearch } from '@/api/user.js'

export default {
  name: 'Editor',
  props: {
    value: {
      type: String,
      default: '',
    },
    bucket: {
      type: String,
      default: 'public',
    },
    path: {
      // 上传文件地址  exam-admin
      type: String,
      default: 'weapp-editor',
    },
  },
  data() {
    return {
      editorId: null,
      editor: null,
      editorData: this.value,
      client: null,
    }
  },
  created() {
    this.editorId = `${new Date().getTime()}${Math.round(
      Math.random() * 10000
    )}`
  },
  methods: {
    async getClient() {
      const sts = await getSTS()
      this.client = new OSS({
        region: 'oss-cn-shanghai',
        accessKeyId: sts.AccessKeyId,
        accessKeySecret: sts.AccessKeySecret,
        stsToken: sts.SecurityToken,
        bucket: this.bucket,
      })
    },
  },
  mounted() {
    this.$nextTick(() => {
      const _this = this
      const editor = new E(document.getElementById(this.editorId))

      // 配置 onchange 回调函数，将数据同步到 vue 中
      editor.config.onchange = (newHtml) => {
        this.editorData = newHtml
      }
      // 配置颜色（文字颜色、背景色）
      editor.config.colors = [
        '#9D9D9D',
        '#1A1A1A',
        '#3B3B3B',
        '#FFC483',
        '#98B6E4',
      ]
      // 配置字体
      editor.config.fontNames = [
        '黑体',
        '仿宋',
        '楷体',
        '标楷体',
        '华文仿宋',
        '华文楷体',
        '宋体',
        '微软雅黑',
        'Arial',
        'Tahoma',
        'Verdana',
        'Times New Roman',
        'Courier New',
        'DIN Alternate',
      ]
      editor.config.fontSizes = {
        'x-small': { name: '13px', value: '1' },
        small: { name: '15px', value: '2' },
        normal: { name: '17px', value: '3' },
        large: { name: '20px', value: '4' },
        'x-large': { name: '28px', value: '5' },
        'xx-large': { name: '32px', value: '6' },
      }

      // 配置粘贴文本的内容处理
      editor.config.pasteTextHandle = function (pasteStr) {
        var newStr = `<p>
          ${pasteStr.replace(new RegExp('<br/>', 'g'), '</p><p>')}
        </p>
        `
        return newStr
      }
      // message
      editor.config.customAlert = function (s, t) {
        switch (t) {
          case 'success':
            _this.$message.success(s)
            break
          case 'info':
            _this.$message.info(s)
            break
          case 'warning':
            _this.$message.warning(s)
            break
          case 'error':
            _this.$message.error(s)
            break
          default:
            _this.$message.info(s)
            break
        }
      }
      // 修改上传图片
      editor.config.customUploadImg = async function (
        resultFiles,
        insertImgFn
      ) {
        try {
          await _this.getClient()
          const path = `${_this.path}/${dayjs().format(
            'YYYY/MM/DD'
          )}/${uuid().replace(/-/g, '')}.${resultFiles[0].name.split('.')[1]}`
          let { url } = await _this.client.put(path, resultFiles[0])
          url = url.replace(
            'kypublic.oss-cn-shanghai.aliyuncs.com',
            'cdn-public.knowyourself.cc'
          )
          insertImgFn(url)
        } catch (error) {
          _this.$message.error('上传图片异常')
        }
      }
      // 自定义按钮
      const { $, BtnMenu } = E
      // 测试
      class TestMenu extends BtnMenu {
        constructor(editor) {
          const $elem = E.$(
            `<div class="w-e-menu" data-title="Test">
            <i class='el-icon-menu'></i>
          </div>`
          )
          super($elem, editor)
        }
        // 菜单点击事件
        clickHandler() {}
        tryChangeActive() {}
      }
      E.registerMenu('TestMenu', TestMenu)

      // 创建编辑器
      editor.create()
      // 同步
      editor.txt.html(`${this.editorData}`)
      this.editor = editor
    })
  },
  watch: {
    value(val) {
      if (val !== this.editor.txt.html()) {
        this.editor.txt.html(val)
      }
    },
    editorData(val) {
      this.$emit('input', this.editorData)
    },
  },
  beforeDestroy() {
    // 调用销毁 API 对当前编辑器实例进行销毁
    this.editor.destroy()
    this.editor = null
  },
}
</script>
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
