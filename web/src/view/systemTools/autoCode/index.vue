<template>
  <div>
    <!-- 初始版本自动化代码工具 -->
    <el-form ref="autoCodeForm" :rules="rules" :model="form" label-width="120px" :inline="true">
      <el-form-item label="Struct名称" prop="structName">
        <el-input v-model="form.structName" placeholder="首字母自动转换大写"></el-input>
      </el-form-item>
      <el-form-item label="Struct简称" prop="abbreviation">
        <el-input v-model="form.abbreviation" placeholder="简称会作为入参对象名和路由group"></el-input>
      </el-form-item>
      <el-form-item label="Struct中文名称" prop="description">
        <el-input v-model="form.description" placeholder="中文描述作为自动api描述"></el-input>
      </el-form-item>
      <el-form-item label="文件名称" prop="packageName">
        <el-input v-model="form.packageName"></el-input>
      </el-form-item>
      <el-form-item label="自动创建api">
        <el-checkbox v-model="form.autoCreateApiToSql"></el-checkbox>
      </el-form-item>
    </el-form>
    <!-- 组件列表 -->
    <div class="button-box clearflex">
      <el-button @click="editAndAddField()" type="primary">新增Field</el-button>
    </div>
    <el-table :data="form.fields" border stripe>
      <el-table-column type="index" label="序列" width="100">
      </el-table-column>
      <el-table-column prop="fieldName" label="Field名">
      </el-table-column>
      <el-table-column prop="fieldDesc" label="中文名">
      </el-table-column>
      <el-table-column prop="fieldJson" label="FieldJson">
      </el-table-column>
      <el-table-column prop="fieldType" label="Field数据类型" width="130">
      </el-table-column>
      <el-table-column prop="columnName" label="数据库字段" width="130">
      </el-table-column>
      <el-table-column prop="comment" label="数据库字段描述" width="130">
      </el-table-column>
      <el-table-column prop="fieldSearchType" label="搜索条件" width="130">
      </el-table-column>
      <el-table-column label="操作" width="300">
        <template slot-scope="scope">
          <el-button size="mini" type="primary" icon="el-icon-edit" @click="editAndAddField(scope.row)">编辑</el-button>
          <el-button size="mini" type="text" :disabled="scope.$index == 0" @click="moveUpField(scope.$index)">上移</el-button>
          <el-button size="mini" type="text" :disabled="(scope.$index + 1) == form.fields.length" @click="moveDownField(scope.$index)">下移</el-button>
          <el-popover placement="top" v-model="scope.row.visible">
            <p>确定删除吗？</p>
            <div style="text-align: right; margin: 0">
              <el-button size="mini" type="text" @click="scope.row.visible = false">取消</el-button>
              <el-button type="primary" size="mini" @click="deleteField(scope.$index)">确定</el-button>
            </div>
            <el-button size="mini" type="danger" icon="el-icon-delete" slot="reference">删除</el-button>
          </el-popover>
        </template>
      </el-table-column>
    </el-table>
    <!-- 组件列表 -->
    <div class="button-box clearflex">
      <el-button @click="enterForm" type="primary">生成代码包</el-button>
    </div>
    <!-- 组件弹窗 -->
    <el-dialog title="组件内容" :visible.sync="dialogFlag">
      <FieldDialog :dialogMiddle="dialogMiddle" ref="fieldDialog" />
      <div slot="footer" class="dialog-footer">
        <el-button @click="closeDialog">取 消</el-button>
        <el-button type="primary" @click="enterDialog">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
const fieldTemplate = {
  fieldName: '',
  fieldDesc: '',
  fieldType: '',
  fieldJson: '',
  columnName: '',
  comment:'',
  fieldSearchType:''
}

import FieldDialog from '@/view/systemTools/autoCode/component/fieldDialog.vue'
import { toUpperCase } from '@/utils/stringFun.js'
import { createTemp } from '@/api/autoCode.js'
export default {
  name: 'autoCode',
  data() {
    return {
      addFlag: '',
      form: {
        structName: '',
        packageName: '',
        abbreviation: '',
        description:'',
        autoCreateApiToSql: false,
        fields: [],
      },
      rules: {
        structName: [{ required: true, message: '请输入结构体名称', trigger: 'blur' }],
        abbreviation: [{ required: true, message: '请输入结构体简称', trigger: 'blur' }],
        description: [{ required: true, message: '请输入结构体描述', trigger: 'blur' }],
        packageName: [{ required: true, message: '请输入包名称', trigger: 'blur' }],
      },
      dialogMiddle: {},
      bk: {},
      dialogFlag: false,
    }
  },
  components: {
    FieldDialog,
  },
  methods: {
    editAndAddField(item) {
      this.dialogFlag = true
      if (item) {
        this.addFlag = 'edit'
        this.bk = JSON.parse(JSON.stringify(item))
        this.dialogMiddle = item
      } else {
        this.addFlag = 'add'
        this.dialogMiddle = JSON.parse(JSON.stringify(fieldTemplate))
      }
    },
    moveUpField(index) {
      if (index == 0) {
        return
      }
      const oldUpField = this.form.fields[index - 1]
      this.form.fields.splice(index - 1, 1)
      this.form.fields.splice(index, 0, oldUpField)
    },
    moveDownField(index) {
      const fCount = this.form.fields.length
      if (index == fCount - 1) {
        return
      }
      const oldDownField = this.form.fields[index + 1]
      this.form.fields.splice(index + 1, 1)
      this.form.fields.splice(index, 0, oldDownField)
    },
    enterDialog() {
      this.$refs.fieldDialog.$refs.fieldDialogFrom.validate((valid) => {
        if (valid) {
          this.dialogMiddle.fieldName = toUpperCase(this.dialogMiddle.fieldName)
          if (this.addFlag == 'add') {
            this.form.fields.push(this.dialogMiddle)
          }
          this.dialogFlag = false
        } else {
          return false
        }
      })
    },
    closeDialog() {
      if (this.addFlag == 'edit') {
        this.dialogMiddle = this.bk
      }
      this.dialogFlag = false
    },
    deleteField(index) {
      this.form.fields.splice(index, 1)
    },
    async enterForm() {
      if (this.form.fields.length <= 0) {
        this.$message({
          type: 'error',
          message: '请填写至少一个field',
        })
        return false
      }
      if(this.form.fields.some(item=>item.fieldName == this.form.structName)){
        this.$message({
          type: 'error',
          message: '存在与结构体同名的字段',
        })
        return false
      }
      this.$refs.autoCodeForm.validate(async (valid) => {
        if (valid) {
          this.form.structName = toUpperCase(this.form.structName)
          if (this.form.structName == this.form.abbreviation) {
            this.$message({
              type: 'error',
              message: 'structName和struct简称不能相同',
            })
            return false
          }
          const data = await createTemp(this.form)
          const blob = new Blob([data])
          const fileName = 'ginvueadmin.zip'
          if ('download' in document.createElement('a')) {
            // 不是IE浏览器
            let url = window.URL.createObjectURL(blob)
            let link = document.createElement('a')
            link.style.display = 'none'
            link.href = url
            link.setAttribute('download', fileName)
            document.body.appendChild(link)
            link.click()
            document.body.removeChild(link) // 下载完成移除元素
            window.URL.revokeObjectURL(url) // 释放掉blob对象
          } else {
            // IE 10+
            window.navigator.msSaveBlob(blob, fileName)
          }
        } else {
          return false
        }
      })
    },
  },
}
</script>
<style scope lang="scss">
.button-box {
  padding: 10px 20px;
  .el-button {
    float: right;
  }
}
</style>
