# tinymce 富文本编辑器

## 实现

> 需在 public 目录下引入 tinymce 字体和样式文件

> [见附件](https://github.com/MrRiven/cs-blog.git)

```
<template lang="pug">
div
  Editor(v-if='init', v-model='content', :init='init', :key='id', :id='tinyId')

  //- 图片
  el-dialog(
    :visible.sync='imgDialogVisible',
    title='添加图片',
    append-to-body,
    width='240px'
  )
    TheUpload(
      path='weapp',
      :fileArr='imgList',
      @changeFileList='handleSuccessUpload(arguments, 1)',
      @removeFileList='handleRemoveUpload(arguments, 1)'
    )
    el-row(slot='footer')
      el-button(type='primary', size='small', @click='closeImgDialog') 确定
      el-button(size='small', @click='imgDialogVisible = false') 取消

</template>

<script>
import tinymce from 'tinymce'
import 'tinymce/themes/silver'
import Editor from '@tinymce/tinymce-vue'
import 'tinymce/icons/default/icons'
import 'tinymce/plugins/autolink'
import 'tinymce/plugins/link'
import 'tinymce/plugins/lists'
import 'tinymce/plugins/image'
import 'tinymce/plugins/preview'
import 'tinymce/plugins/fullpage'
import 'tinymce/plugins/searchreplace'
import 'tinymce/plugins/directionality'
import 'tinymce/plugins/visualblocks'
import 'tinymce/plugins/visualchars'
import 'tinymce/plugins/media'
import 'tinymce/plugins/template'
import 'tinymce/plugins/codesample'
import 'tinymce/plugins/table'
import 'tinymce/plugins/charmap'
import 'tinymce/plugins/hr'
import 'tinymce/plugins/pagebreak'
import 'tinymce/plugins/nonbreaking'
import 'tinymce/plugins/noneditable'
import 'tinymce/plugins/anchor'
import 'tinymce/plugins/toc'
import 'tinymce/plugins/insertdatetime'
import 'tinymce/plugins/advlist'
import 'tinymce/plugins/wordcount'
import 'tinymce/plugins/imagetools'
import 'tinymce/plugins/contextmenu'
import 'tinymce/plugins/textpattern'
import 'tinymce/plugins/help'
import 'tinymce/plugins/paste'
import 'tinymce/plugins/textcolor'
import 'tinymce/plugins/colorpicker'
import 'tinymce/plugins/code'
import 'tinymce/plugins/autosave'
import 'tinymce/plugins/autoresize'
import '@npkg/tinymce-plugins/letterspacing'

import OSS from 'ali-oss'
import uuid from 'uuid'
import dayjs from 'dayjs'
import { getSTS, apiQueryProductBySearch } from '@/api/user.js'

export default {
  name: 'tinymce-vue',
  components: {
    Editor
  },
  props: {
    value: {
      type: String,
      default: ''
    },
    bucket: {
      type: String,
      default: 'kypublic'
    },
    path: {
      type: String,
      default: 'weapp-editor'
    }
  },
  data () {
    return {
      id: uuid(),
      tinyId: uuid(),
      editor: null,
      init: {
        selector: `#${this.id}`,
        language_url: '/tinymce/langs/zh_CN.js',
        language: 'zh_CN',
        skin_url: '/tinymce/skins/ui/oxide',
        content_css: '/tinymce/skins/content/default/content.css',
        height: 500,
        max_height: 500,
        plugins: 'preview searchreplace autolink directionality visualblocks visualchars  link media template  codesample table charmap hr pagebreak nonbreaking anchor insertdatetime advlist lists wordcount textpattern help autosave autoresize letterspacing code paste',
        toolbar: 'code | styleselect formatselect fontselect fontsizeselect | undo redo restoredraft | cut copy paste pastetext | forecolor backcolor bold italic underline strikethrough link anchor | alignleft aligncenter alignright alignjustify | outdent indent |  bullist numlist | blockquote subscript superscript | table  media charmap  hr pagebreak insertdatetime | preview | removeformat lineheight letterspacing | uploadImgBtn | productBtn | weappLinkBtn',
        font_formats: '微软雅黑=\'微软雅黑\';宋体=\'宋体\';黑体=\'黑体\';仿宋=\'仿宋\';楷体=\'楷体\';隶书=\'隶书\';幼圆=\'幼圆\';Andale Mono=andale mono,times;Arial=arial,helvetica,sans-serif;Arial Black=arial black,avant garde;Book Antiqua=book antiqua,palatino;Comic Sans MS=comic sans ms,sans-serif;Courier New=courier new,courier;Georgia=georgia,palatino;Helvetica=helvetica;Impact=impact,chicago;Symbol=symbol;Tahoma=tahoma,arial,helvetica,sans-serif;Terminal=terminal,monaco;Times New Roman=times new roman,times;Trebuchet MS=trebuchet ms,geneva;Verdana=verdana,geneva;Webdings=webdings;Wingdings=wingdings;DIN Alternate=DIN Alternate',
        fontsize_formats: '10px 11px 12px 13px 14px 15px 16px 17px 18px 20px 24px 26px 28px 30px 32px 36px 40px 44px 48px 50px 52px',
        lineheight_formats: '1 1.4 1.6 2 2.2 3',
        letterspacing: '0px 0.6px 1px',
        toolbar_mode: 'wrap',
        relative_urls: false,
        convert_urls: false,
        branding: false,
        menubar: true,
        zIndex: 1101,
        paste_data_images: true,
        images_upload_handler: (blobInfo, success, failure) => {
          this.handleUpload(blobInfo).then((path) => {
            success(path)
          }).catch((err) => {
            failure('上传失败: ' + err)
          })
        },
        setup: (editor) => {
          this.editor = editor
          editor.ui.registry.addButton('uploadImgBtn', {
            text: '上传图片',
            tooltip: '上传图片',
            onAction: (_) => {
              this.imgList = []
              this.imgDialogVisible = true
            }
          })
        }
      },
      content: this.value,

      //
      imgDialogVisible: false,
      imgList: [],
    }
  },
  mounted () {
    this.$nextTick(() => {
      tinymce.init({})
    })
  },
  methods: {
    async getClient () {
      const sts = await getSTS()
      this.client = new OSS({
        region: 'oss-cn-shanghai',
        accessKeyId: sts.AccessKeyId,
        accessKeySecret: sts.AccessKeySecret,
        stsToken: sts.SecurityToken,
        bucket: this.bucket
      })
    },
    closeImgDialog () {
      if (this.imgList.length === 0) {
        return this.$message.warning('请选择图片')
      }
      this.editor.insertContent(`<p><img src='${this.imgList[0].url}' /></p>`)
      this.imgDialogVisible = false
    },
    handleSuccessUpload ([list], index) {
      switch (index) {
        case 1:
          this.imgList = list
          break
      }
    },
    handleRemoveUpload ([file, list], index) {
      switch (index) {
        case 1:
          this.imgList = list
          break
      }
    },
    toBlob (urlData, fileType) {
      let bytes = window.atob(urlData)
      let n = bytes.length
      let u8arr = new Uint8Array(n)
      while (n--) {
        u8arr[n] = bytes.charCodeAt(n)
      }
      return new Blob([u8arr], { type: fileType })
    },
    async handleUpload (blobInfo) {
      return new Promise(async (resolve, reject) => {
        await this.getClient()
        const name = blobInfo.name()
        const fileNameArr = name.split('.')
        const ext = fileNameArr.length
          ? fileNameArr[fileNameArr.length - 1]
          : ''
        const path = `${this.path}${dayjs().format(
          'YYYY/MM/DD'
        )}/${uuid().replace(/-/g, '')}.${ext}`
        const blob = blobInfo.blob()
        const file = this.toBlob(blobInfo.base64(), blob.type)
        try {
          const { url } = await this.client.put(path, file)
          const newUrl = url.replace(
            'kypublic.oss-cn-shanghai.aliyuncs.com',
            'cdn-public.knowyourself.cc'
          )
          resolve(newUrl)
        } catch (err) {
          reject(err)
        }
      })
    }
  },
  watch: {
    content (newVal, oldVal) {
      this.$emit('input', this.content)
    },
    value (newVal, oldVal) {
      this.content = this.value
    }
  }
}
</script>

```

## 使用

```
<template>
...
  <Editor
    v-model='value'
  />
...
</template>

<script>
import Editor from './Editor'

export default {
  components: {
    Editor
  },
  data () {
    return {
      value: ''
    }
  }
}
</script>

```
