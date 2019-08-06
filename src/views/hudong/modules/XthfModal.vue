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
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="年级与科目">
          <a-cascader :options="options" @change="onChange" placeholder="请选择年级与科目" />
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="注册后的天数">
          <a-input placeholder="注册后的天数" v-decorator="['psTime', {}]" />
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="消息内容">
          <a-select @change="handleChange" defaultValue="-1">
            <a-select-option value="-1">请选择消息内容</a-select-option>
            <a-select-option value="OK">OK</a-select-option>
            <a-select-option value="IMG">IMG</a-select-option>
            <a-select-option value="YY">YY</a-select-option>
          </a-select>
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="反馈内容">
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
import { constants } from 'fs'

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
  name: 'XthfModal',
  data() {
    return {
      title: '操作',
      visible: false,
      model: {},
      options: [],
      flId: '-1',
      xuekeId: '-1',
      type: '-1',
      content: '',
      disableSubmit: false,
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
        add: '/xthf/xthf/add',
        edit: '/xthf/xthf/edit',
        nflist: '/xthf/xthf/nflist'
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
    this.getOptions()
  },
  mounted() {
    tinymce.init({})
  },
  watch: {
    value(newValue) {
      this.content = newValue
    },
    content(newValue) {
      this.$emit('input', newValue)
    }
  },
  methods: {
    getOptions() {
      let httpUrl = this.url.nflist
      httpAction(httpUrl, '', 'get').then(res => {
        if (res.success) {
          this.options = res.result
        }
      })
    },
    add() {
      this.edit({})
    },
    edit(record) {
      this.type = record.introduce
      if (JSON.stringify(record) == '{}') {
        this.content = ''
      } else {
        this.content = record.content
      }
      this.type = record.introduce
      this.flId = '-1'
      this.xuekeId = '-1'
      this.form.resetFields()
      this.model = Object.assign({}, record)
      this.visible = true
      this.$nextTick(() => {
        this.form.setFieldsValue(pick(this.model, 'psTime', 'type', 'introduce', 'content', 'grade', 'kemu'))
        //时间格式化
      })
    },
    close() {
      this.$emit('close')
      this.visible = false
      this.disableSubmit = false
    },
    handleSubmit() {
      if (this.type == '-1' || this.type == undefined) {
        this.type = 'OK'
      }
      if (this.flId == '-1') {
        if (this.options.length > 0) {
          this.flId = this.options[0].value
          this.xuekeId = this.options[0].children[0].value
        }
      }
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

          this.model.introduce = this.type
          this.model.grade = this.flId
          this.model.kemu = this.xuekeId
          this.model.content = this.content

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
    },
    handleChange(value) {
      this.type = value
    },
    onClick(e) {
      this.$emit('onClick', e, tinymce)
    },
    onChange(value) {
      this.flId = value[0]
      this.xuekeId = value[1]
    }
  }
}
</script>

<style scoped>
.ant-table-tbody .ant-table-row td {
  padding-top: 10px;
  padding-bottom: 10px;
}

.drawer-bootom-button {
  position: absolute;
  bottom: -8px;
  width: 100%;
  border-top: 1px solid #e8e8e8;
  padding: 10px 16px;
  text-align: right;
  left: 0;
  background: #fff;
  border-radius: 0 0 2px 2px;
}
</style>