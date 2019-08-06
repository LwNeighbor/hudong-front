<template>
  <a-modal
    :title="title"
    :width="800"
    :visible="visible"
    :confirmLoading="confirmLoading"
    @ok="handleOk"
    @cancel="handleCancel"
    cancelText="关闭"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item label="年级" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-cascader :options="options" @change="onChange" />
        </a-form-item>

        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="模版路径">
          <a-upload
            name="file"
            :multiple="true"
            :action="uploadAction"
            :headers="headers"
            @change="handleChange"
          >
            <a-button>
              <a-icon type="upload" />Click to Upload
            </a-button>
          </a-upload>
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
import { httpAction } from '@/api/manage'
import pick from 'lodash.pick'
import moment from 'moment'
import Vue from 'vue'
import { ACCESS_TOKEN } from '@/store/mutation-types'

export default {
  name: 'HdExcelModal',
  data() {
    return {
      title: '操作',
      visible: false,
      model: {},
      headers: {},
      options: [],
      path: '',
      flId: '',
      flName: '',
      labelCol: {
        xs: { span: 24 },
        sm: { span: 5 }
      },
      wrapperCol: {
        xs: { span: 24 },
        sm: { span: 16 }
      },

      confirmLoading: false,
      form: this.$form.createForm(this),
      validatorRules: {
        excelName: { rules: [{ required: true, message: '请输入模版名称!' }] },
        excelAddress: { rules: [{ required: true, message: '请输入模版路径!' }] }
      },
      url: {
        fllist: '/msfenlei/msFenLi/valueList',
        add: '/hdExcel/hdExcel/add',
        edit: '/hdExcel/hdExcel/edit',
        fileUpload: window._CONFIG['domianURL'] + '/hdExcel/hdExcel/uploadExcel'
      }
    }
  },
  created() {
    const token = Vue.ls.get(ACCESS_TOKEN)
    this.headers = { 'X-Access-Token': token }
  },
  computed: {
    uploadAction: function() {
      return this.url.fileUpload
    }
  },
  mounted() {
    let httpUrl = this.url.fllist
    httpAction(httpUrl, '', 'get').then(res => {
      if (res.success) {
        var list = res.result
        const data = []
        list.forEach(r => {
          data.push({
            value: r.value,
            label: r.text
          })
        })
        this.options = data
      }
    })
  },
  methods: {
    add() {
      this.edit({})
    },
    edit(record) {
      this.form.resetFields()
      this.model = Object.assign({}, record)
      this.visible = true
      this.$nextTick(() => {
        this.form.setFieldsValue(pick(this.model, 'excelName'))
        //时间格式化
      })
    },
    close() {
      this.$emit('close')
      this.visible = false
    },
    handleOk() {
      const that = this
      // 触发表单验证
      this.form.validateFields((err, values) => {
        if (!err) {
          that.confirmLoading = true
          let httpurl = ''
          let method = ''
          if (!this.model.id) {
            httpurl += this.url.add
            method = 'post'
          } else {
            httpurl += this.url.edit
            method = 'put'
          }

          this.model.excelAddress = this.path
          this.model.flId = this.flId
          this.model.flName = this.flName

          let formData = Object.assign(this.model, values)
          //时间格式化
          httpAction(httpurl, formData, method)
            .then(res => {
              if (res.success) {
                that.$message.success(res.message)
                that.$emit('ok')
                this.loadData();
              } else {
                that.$message.warning(res.message)
              }
            })
            .finally(() => {
              that.confirmLoading = false
              that.close()
            })
        }
      })
    },
    handleCancel() {
      this.close()
    },
    handleChange(info) {
      if (info.file.status !== 'uploading') {
      }
      if (info.file.status === 'done') {
        this.path = info.file.response.result.path
        this.$message.success(`${info.file.name} file uploaded successfully`)
      } else if (info.file.status === 'error') {
        this.$message.error(`${info.file.name} file upload failed.`)
      }
    },
    onChange(value,selectedOptions) {
      this.flId = selectedOptions[0].value
      this.flName = selectedOptions[0].label
    }
  }
}
</script>

<style scoped>
</style>