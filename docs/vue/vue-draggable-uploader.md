# vue-draggable-uploader

> 基于 Element-ui 做了封装, 选取后自动上传到 OSS 返回一个 URL

## 实现

```
<template>
  <div>
    <div>
      <draggable
        v-if="type === 'img'"
        v-model="fileList"
        class="list-group"
        tag="ul"
        v-bind="dragOptions"
        @start="dragStart"
        @end="dragEnd"
      >
        <div
          v-for="(item, index) in fileList"
          :key="index"
          class="list-item"
          @mouseenter="showDelBtn(index)"
          @mouseleave="hiddenDelBtn"
        >
          <img
            v-if="item"
            :src="item.url"
            width="148"
            height="148"
            class="imgSty"
          />
          <span>
            <!-- 预览 -->
            <!-- <i v-show="index === currentDelBtn"
               class="el-icon-zoom-in"
               @click="handPrevie(item, index)" /> -->
            <i
              v-show="index === currentDelBtn"
              class="el-icon-delete delIcon"
              @click="deleImg(item, index)"
            />
          </span>
        </div>
      </draggable>
      <!-- 图片上传 -->
      <el-upload
        v-show="imgIsShow"
        v-loading="loading"
        ref="upload"
        slot="reference"
        class="upload-demo"
        :list-type="type === 'img' ? 'picture-card' : ''"
        :auto-upload="false"
        :file-list="fileList"
        :action="actionUrl"
        :on-change="handChange"
        :on-success="handSuccess"
        :on-error="handError"
        :on-preview="handPrevie"
        :on-remove="handRemove"
        :before-upload="handBefore"
        :accept="accept"
        :headers="uploadHeaders"
        :show-file-list="type === 'img' ? false : true"
      >
        <el-button v-if="type !== 'img'">{{ btnTxt }}</el-button>
        <i v-else class="el-icon-plus" />
        <div
          v-if="type !== 'img'"
          slot="tip"
          class="el-upload__tip"
          v-html="tips"
        />
      </el-upload>
    </div>
    <!-- 上传提示 -->
    <div v-if="type === 'img' && tips !== ''" class="tips-img" v-html="tips" />
  </div>
</template>

<script>
import draggable from 'vuedraggable'
import OSS from 'ali-oss'
import uuid from 'uuid'
import dayjs from 'dayjs'

export default {
  components: {
    draggable
  },
  props: {
    bucket: {
      type: String,
      default: 'public'
    },
    path: {
      // 上传文件地址
      type: String
    },
    btnTxt: {
      type: String,
      default: '选择文件'
    },
    type: {
      type: String,
      default: 'img'
    },
    actionUrl: {
      type: String,
      default: '#'
    },
    fileArr: {
      type: Array,
      default: () => {
        return []
      }
    },
    tips: {
      type: String,
      default: ''
    },
    accept: {
      type: String,
      default: '.jpg, .png, .gif, .jpeg'
    },
    limit: {
      type: Number,
      default: 1
    }
  },
  data () {
    return {
      // upload
      uploadHeaders: {},
      fileList: this.fileArr,
      // 拖拽
      currentDelBtn: -1,
      drag: false,
      //
      loading: false
    }
  },
  computed: {
    // 这一部分直接拿过来用的
    dragOptions () {
      return {
        animation: 200,
        group: 'description',
        disabled: false,
        ghostClass: 'ghost'
      }
    },
    imgIsShow () {
      let flag = false
      if (this.type === 'img' && this.fileList.length < this.limit) {
        flag = true
      } else if (this.type !== 'img') {
        flag = true
      }
      return flag
    }
  },
  watch: {
    fileArr (val) {
      this.fileList = val
    }
  },
  methods: {
    handSuccess (response, file, fileList) {
      this.clearFile()
      this.onSuccess(response)
    },
    handCheckAccept (fileList) {
      const flag = fileList.every(
        item => this.accept.indexOf(item.name.substr(item.name.lastIndexOf('.')).toLocaleLowerCase()) !== -1
      )
      if (!flag) {
        this.$message.warning('文件格式不正确,请重新选择')
        this.fileList = []
        this.$emit('changeFileList', this.fileList)
      }
      return flag
    },
    handChange (file, fileList) {
      if (!this.handCheckAccept(fileList)) return
      if (fileList.length > this.limit) {
        this.$message.warning(
          `当前限制选择${this.limit}个文件，共选择了 ${fileList.length} 个文件,默认从第一个文件替换`
        )
      }
      if (!this.path) return this.$message.warning('上传路径未传')

      const result = fileList.slice(-this.limit)
      let proArr = []
      let arr = result.filter(it => it.status === 'ready')
      this.fileList = result.filter(it => it.status !== 'ready')

      arr.forEach((item, index) => {
        proArr[index] = new Promise(async (resolve) => {
          let url = await this.handleUpload(item.raw)
          if (url) {
            this.fileList.push({
              name: item.name,
              url: url,
              status: 'success'
            })
            resolve()
          }
        })
      })
      Promise.all(proArr).then(() => {
        this.$emit('changeFileList', this.fileList)
      }).catch(() => { })
    },
    handRemove (file, fileList) {
      this.fileList = fileList
      this.$emit('removeFileList', file, this.fileList)
    },
    handError (res, file, fileList) {
      this.$refs.upload.clearFiles()
      this.$alert(res.errorMessage || '上传失败', '提示', {
        confirmButtonText: '确定',
        dangerouslyUseHTMLString: true
      })
    },
    handPrevie (file) { },
    handBefore (file) {
      return new Promise((resolve, reject) => {
        this.$nextTick(() => {
          resolve()
        })
      })
    },
    handleExceed (files, fileList) {
      this.fileList = [files[0]]
      this.$emit('changeFileList', this.fileList)
      this.$message.warning(
        `当前限制选择1个文件，共选择了 ${files.length + fileList.length} 个文件,默认替换第一个文件`
      )
    },
    clearFiles () {
      this.fileList = []
      this.$refs.upload.clearFiles()
    },
    onSuccess (res) {
    },
    // drag
    dragStart () {
      this.drag = true
    },
    dragEnd () {
      this.drag = false
      this.$emit('changeFileList', this.fileList)
    },
    // 显示删除图片的图标
    showDelBtn (index) {
      this.currentDelBtn = index
    },
    // 隐藏删除图片的图标
    hiddenDelBtn () {
      this.currentDelBtn = -1
    },
    // 删除图片
    deleImg (data, index) {
      this.fileList.splice(index, 1)
      this.$emit('removeFileList', data, this.fileList)
    },
    async handleUpload (file) {
      try {
        this.loading = true
        const sts = await this.$api.getSTS()
        const client = new OSS({
          region: 'oss-cn-shanghai',
          accessKeyId: sts.AccessKeyId,
          accessKeySecret: sts.AccessKeySecret,
          stsToken: sts.SecurityToken,
          bucket: this.bucket
        })
        const { name } = file
        const fileNameArr = name.split('.')
        const ext = fileNameArr.length ? fileNameArr[fileNameArr.length - 1] : ''
        const path = `${this.path}/${dayjs().format('YYYY/MM/DD')}/${uuid().replace(/-/g, '')}.${ext}`
        let { url } = await client.put(path, file)
        url = url.replace('kypublic.oss-cn-shanghai.aliyuncs.com', 'cdn-public.knowyourself.cc')
        this.loading = false
        return url
      } catch (err) {
        this.$message.warning('上传失败请重试')
        this.loading = false
      }
    }

  }
}
</script>

<style lang="scss" scoped>
.tips-img {
  line-height: 20px;
  display: inline-block;
  color: #999;
  font-weight: 400;
  font-size: 12px;
  margin: 12px 0;
}

.upload-demo {
  display: inline;
  vertical-align: top;
}

.list-group {
  display: inline;
  vertical-align: top;
  margin-block-start: 0px;
  margin-block-end: 0px;
  margin-inline-start: 0px;
  margin-inline-end: 0px;
  padding-inline-start: 0px;

  .list-item {
    display: inline-block;
    margin: 0 8px 8px 0;
    position: relative;
    border-radius: 8px;
    width: 148px;
    height: 148px;

    img {
      width: 148px;
      height: 148px;
      border-radius: 8px;
    }

    span {
      position: absolute;
      width: 100%;
      height: 100%;
      left: 0;
      top: 0;
      cursor: default;
      text-align: center;
      color: #fff;
      opacity: 0;
      font-size: 20px;
      background-color: rgba(0, 0, 0, 0.5);
      transition: opacity 0.3s;
      border-radius: 8px;

      i {
        cursor: pointer;
        position: relative;
        top: 35%;
        margin: 0 10px;
        vertical-align: middle;
      }
    }
  }

  .list-item:hover {
    span:hover {
      opacity: 1;
    }
  }
}
</style>

<style lang="scss">
.el-upload__tip {
  line-height: 1;
  display: inline-block;
  padding-left: 30px;
  color: #999;
  font-weight: 400;
}

.el-upload-list {
  max-width: 360px;
}

.el-upload-list--picture .el-upload-list__item-thumbnail {
  width: 160px;
  height: 90px;
}

.el-upload-list--picture .el-upload-list__item {
  min-height: 112px;
}

.el-upload-list__item-name {
  margin-right: 30px;
}
.imgSty {
  width: 148px;
  height: auto;
}
</style>
```

## 使用

```
<template>
...
  <DragUpload
    path='kyapp_meditation'
    :fileArr='list'
    @changeFileList='handleSuccessUpload(arguments)'
    @removeFileList='handleRemoveUpload(arguments)'
  />
...
</template>

<script>
import DragUpload from './DragUpload'

export default {
  data() {
    list: []
  },
  components: {
    DragUpload
  },
  methods: {
    handleSuccessUpload ([list]) {
      this.list = list
    },
    handleRemoveUpload ([file, list]) {
      this.list = list
    },
  }
}
</script>
```
