# vue 表格拖拽排序

> 基于 Elemet-Ui 的 table

## 安装

```bash
npm i vuedraggable -S
```

## 使用

```vue
<template>
  ...
  <el-table :data="tableData" row-key="id">
    <el-table-column label="ID" prop="id" />
    <el-table-column label="姓名" prop="name" />
    <el-table-column label="年龄" prop="age" />
  </el-table>
  ...
</template>

<script>
import Sortable from 'sortablejs'

export default {
  data() {
    return {
      tableData: [
        { id: 1, name: '张三', age: 18 },
        { id: 2, name: '李四', age: 28 },
        { id: 3, name: '王五', age: 38 },
      ],
    }
  },
  mounted() {
    this.rowDrop()
  },
  methods: {
    // 行拖拽
    rowDrop() {
      const tbody = document.querySelector('.el-table__body-wrapper tbody')
      const _this = this
      Sortable.create(tbody, {
        onEnd({ newIndex, oldIndex }) {
          const currRow = _this.tableData.splice(oldIndex, 1)[0]
          _this.tableData.splice(newIndex, 0, currRow)
          console.log(_this.tableData)
        },
      })
    },
  },
}
</script>
```
