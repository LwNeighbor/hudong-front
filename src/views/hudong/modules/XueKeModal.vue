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
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="学科名称">
          <a-input placeholder="请输入学科名称" v-decorator="['xkName', validatorRules.xkName ]" />
        </a-form-item>
      </a-form>
    </a-spin>
  </a-modal>
</template>

<script>
import { httpAction } from '@/api/manage'
import pick from 'lodash.pick'
import moment from 'moment'

export default {
  name: 'XueKeModal',
  data() {
    return {
      title: '操作',
      visible: false,
      model: {},
      options:[],
      flId:'',
      flName:'',
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
        xkId: { rules: [{ required: true, message: '请输入学科ID!' }] },
        xkName: { rules: [{ required: true, message: '请输入学科名称!' }] }
      },
      url: {
        add: '/xk/xueKe/add',
        edit: '/xk/xueKe/edit',
        fllist: '/msfenlei/msFenLi/valueList'
      }
    }
  },
  created() {},
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
        this.form.setFieldsValue(pick(this.model, 'xkId', 'xkName'))
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
          this.model.flId = this.flId
          this.model.flName = this.flName
          let formData = Object.assign(this.model, values)
          //时间格式化

          console.log(formData)
          httpAction(httpurl, formData, method)
            .then(res => {
              if (res.success) {
                that.$message.success(res.message)
                that.$emit('ok')
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
    onChange(value,selectedOptions) {
      this.flId = selectedOptions[0].value
      this.flName = selectedOptions[0].label
    }
  }
}
</script>

<style scoped>
</style>