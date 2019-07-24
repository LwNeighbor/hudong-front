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
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="孩子姓名">
          <a-input placeholder="请输入孩子姓名" v-decorator="['cdName', validatorRules.cdName ]" />
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="孩子手机号">
          <a-input placeholder="请输入孩子手机号" v-decorator="['cdPhone', validatorRules.cdPhone ]" />
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="登录密码">
          <a-input placeholder="请输入登录密码" v-decorator="['cdPassword', validatorRules.cdPassword ]" />
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
  name: 'ChildModal',
  data() {
    return {
      title: '操作',
      visible: false,
      model: {},
      pid: '',
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
        cdId: { rules: [{ required: true, message: '请输入学生ID!' }] },
        ptId: { rules: [{ required: true, message: '请输入家长ID!' }] },
        cdName: { rules: [{ required: true, message: '请输入孩子姓名!' }] },
        cdBirthday: { rules: [{ required: true, message: '请输入孩子生日!' }] },
        cdSex: { rules: [{ required: true, message: '请输入孩子性别 0/男,1/女!' }] },
        cdPhone: { rules: [{ required: true, message: '请输入孩子手机号!' }] },
        cdPassword: { rules: [{ required: true, message: '请输入登录密码!' }] }
      },
      url: {
        add: '/child/child/add',
        edit: '/child/child/edit'
      }
    }
  },
  created() {},
  methods: {
    add(parentId) {
      this.pid = parentId
      this.edit({})
    },
    edit(record) {
      this.form.resetFields()
      if (this.pid != '') {
        record.ptId = this.pid
      }
      this.model = Object.assign({}, record)
      this.visible = true
      this.$nextTick(() => {
        this.form.setFieldsValue(
          pick(this.model, 'cdId', 'ptId', 'cdName', 'cdBirthday', 'cdSex', 'cdPhone')
        )
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
    }
  }
}
</script>

<style scoped>
</style>