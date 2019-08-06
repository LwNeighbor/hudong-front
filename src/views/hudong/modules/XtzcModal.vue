<template>
  <a-drawer
    :title="title"
    :width="800"
    placement="right"
    :closable="false"
    @close="close"
    :visible="visible"
  >
    <a-spin :spinning="confirmLoading">
      <a-form :form="form">
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="注册后的天数">
           <a-input placeholder="请输入注册后的天数" v-decorator="['psTime', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="发送时间">
           <a-time-picker @change="onChange" :v-model="sendTime" :defaultOpenValue="moment('00:00:00', 'HH:mm:ss')" />
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="消息标题">
          <a-input placeholder="请输入消息标题" v-decorator="['title', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="消息简介">
          <a-input placeholder="请输入消息简介" v-decorator="['introduce', {}]"/>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="消息内容">
          <editor v-model="content" :init="init" :disabled="disabled" @onClick="onClick"></editor>
        </a-form-item>
      </a-form>
    </a-spin>
    <div class="drawer-bootom-button" v-show="!disableSubmit">
      <a-popconfirm title="确定放弃编辑？" @confirm="handleCancel" okText="确定" cancelText="取消">
        <a-button style="margin-right: .8rem">取消</a-button>
      </a-popconfirm>
      <a-button @click="handleSubmit" type="primary" :loading="confirmLoading">提交</a-button>
    </div>
    <!-- <a-button type="primary" @click="handleOk">确定</a-button>
    <a-button type="primary" @click="handleCancel">取消</a-button>-->
  </a-drawer>
</template>

<script>
import { httpAction } from '@/api/manage'
import pick from 'lodash.pick'
import moment from 'moment'
import Vue from 'vue'
import { ACCESS_TOKEN } from '@/store/mutation-types'
import tinymce from 'tinymce/tinymce'
import Editor from '@tinymce/tinymce-vue'
import 'tinymce/themes/silver/theme'
import 'tinymce/plugins/image'
import 'tinymce/plugins/media'
import 'tinymce/plugins/table'
import 'tinymce/plugins/lists'
import 'tinymce/plugins/contextmenu'
import 'tinymce/plugins/wordcount'
import 'tinymce/plugins/colorpicker'
import 'tinymce/plugins/textcolor'
import { constants } from 'fs';

export default {
  components: {
    Editor
  },
  props: {
    value: {
      type: String,
      required: false
    },
    disabled: {
      type: Boolean,
      default: false
    },
    plugins: {
      type: [String, Array],
      default: 'lists image media table textcolor wordcount contextmenu'
    },
    toolbar: {
      type: [String, Array],
      default:
        'undo redo |  formatselect | bold italic | alignleft aligncenter alignright alignjustify | bullist numlist outdent indent | lists image media table | removeformat'
    }
  },
  name: 'XtzcModal',
  data() {
    return {
      title: '操作',
      visible: false,
      drawerWidth: 1200,
      uploadLoading: false,
      disableSubmit: false,
      sendTime: '',
      model: {},
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
      validatorRules: {},
      url: {
        add: '/xtzc/xtzc/add',
        edit: '/xtzc/xtzc/edit',
        imgerver: window._CONFIG['domianURL'] + '/sys/common/view',
        fileUpload: window._CONFIG['domianURL'] + '/sys/common/upload'
      },
      //初始化配置
      init: {
        language_url: '/tinymce/langs/zh_CN.js',
        language: 'zh_CN',
        skin_url: '/tinymce/skins/lightgray',
        height: 800,
        plugins: this.plugins,
        toolbar: this.toolbar,
        branding: false,
        menubar: false,
        images_upload_handler: (blobInfo, success) => {
          const img = 'data:image/jpeg;base64,' + blobInfo.base64()
          success(img)
        }
      },
      content: this.value
    }
  },
  created() {
    const token = Vue.ls.get(ACCESS_TOKEN)
    this.headers = { 'X-Access-Token': token }
  },
  watch: {
    value(newValue) {
      this.content = newValue
    },
    content(newValue) {
      this.$emit('input', newValue)
    }
  },
  mounted() {
    tinymce.init({})
  },
  methods: {
    moment,
    onChange(time, timeString){
      this.sendTime = timeString
    },
    onClick(e) {
      this.$emit('onClick', e, tinymce) 
    },
    add() {
      this.edit({})
    },
    edit(record) {
      if (JSON.stringify(record) == '{}') {
        this.content = ''
      } else {
        this.content = record.content
      }
      this.sendTime = record.sendTime
      this.form.resetFields()
      this.model = Object.assign({}, record)
      this.visible = true
      this.$nextTick(() => {
        this.form.setFieldsValue(pick(this.model, 'psTime', 'title', 'introduce', 'content'))
        //时间格式化
      })
    },
    close() {
      this.$emit('close')
      this.visible = false
      this.disableSubmit = false
    },
    handleSubmit() {
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
          this.model.content = this.content
          this.model.sendTime = this.sendTime
          let formData = Object.assign(this.model, values)
          //时间格式化

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

<style lang="less" scoped>
/** Button按钮间距 */
.ant-btn {
  margin-left: 30px;
  margin-bottom: 30px;
  float: right;
}
</style>