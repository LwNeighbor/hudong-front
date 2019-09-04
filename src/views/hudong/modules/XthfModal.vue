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
        <!-- <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="反馈内容">
          <editor v-model="content" :init="init" :disabled="disabled" @onClick="onClick"></editor>
        </a-form-item>-->
        <a-form-item label="内容类型" :labelCol="labelCol" :wrapperCol="wrapperCol">
          <a-radio-group @change="onChangeMenuType" :defaultValue="0" v-model="type">
            <a-radio :value="0">文字</a-radio>
            <a-radio :value="1">图片</a-radio>
          </a-radio-group>
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="消息内容"
          v-show="localMenuType == 0"
        >
          <a-textarea placeholder="请输入消息内容" :rows="4" v-decorator="['content', {}]" />
        </a-form-item>
        <a-form-item
          label="图片"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          v-show="localMenuType == 1"
        >
          <a-upload
            listType="picture-card"
            class="avatar-uploader"
            :showUploadList="false"
            :action="uploadAction"
            :data="{'isup':1}"
            :headers="headers"
            :beforeUpload="beforeUpload"
            @change="handleChange1"
          >
            <img
              v-if="content"
              :src="getAvatarView()"
              alt="头像"
              style="height:104px;max-width:300px"
            />
            <div v-else>
              <a-icon :type="uploadLoading ? 'loading' : 'plus'" />
              <div class="ant-upload-text">上传</div>
            </div>
          </a-upload>
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
      default: 'lists image textcolor wordcount contextmenu'
    },
    toolbar: {
      type: [String, Array],
      default:
        ' bold italic | alignleft aligncenter alignright alignjustify | bullist numlist outdent indent | link image'
    }
  },
  name: 'XthfModal',
  data() {
    return {
      title: '操作',
      uploadLoading: false,
      visible: false,
      model: {},
      options: [],
      flId: '-1',
      xuekeId: '-1',
      zType: '-1',
      content: '',
      localMenuType: '0',
      type: 0,
      headers: {},
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
        nflist: '/xthf/xthf/nflist',
        imgerver: window._CONFIG['domianURL'] + '/sys/common/view',
        fileUpload: window._CONFIG['domianURL'] + '/sys/common/uploadAll'
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
        images_upload_handler: function(blobInfo, success, failure) {
          let httpurl = window._CONFIG['domianURL'] + '/front/parent/vipCenter/uploadTinyImg'
          let formData = new FormData()
          formData.append('file', blobInfo.blob())
          httpAction(httpurl, formData, 'post').then(res => {
            if (res.success) {
              success(window._CONFIG['domianURL'] + '/' + res.result.url)
            } else {
              this.$message.warning(res.message)
            }
          })
        }
      },
    }
  },
  created() {
    this.getOptions()
    const token = Vue.ls.get(ACCESS_TOKEN)
    this.headers = { 'X-Access-Token': token }
  },
  mounted() {
    tinymce.init({})
  },
  computed: {
    uploadAction: function() {
      return this.url.fileUpload
    }
  },
  methods: {
    onChangeMenuType(e) {
      this.localMenuType = e.target.value
    },
    beforeUpload: function(file) {
      var fileType = file.type
      if (fileType.indexOf('image') < 0) {
        this.$message.warning('请上传图片')
        return false
      }
      //TODO 验证文件大小
    },
    getAvatarView() {
      return window._CONFIG['domianURL'] + "/" +this.content
    },
    handleChange1(info) {
      if (info.file.status === 'uploading') {
        this.uploadLoading = true
        return
      }
      if (info.file.status === 'done') {
        var response = info.file.response
        this.uploadLoading = false
        if (response.success) {
          this.content = response.message
        } else {
          this.$message.warning(response.message)
        }
      }
    },
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
    if (JSON.stringify(record) == '{}') {
        this.type = 0
        this.localMenuType = 0
      } else {
        this.type = parseInt(record.type)
        this.localMenuType = parseInt(record.type)
        this.zType = record.introduce
      }
      this.content = record.content
      
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

          this.model.introduce = this.zType
          this.model.grade = this.flId
          this.model.kemu = this.xuekeId

          let formData = Object.assign(this.model, values)
          if (this.localMenuType == '1') {
            formData.content = this.content
          }
          formData.type = this.localMenuType.toString()
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
              this.content = ''
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
      this.zType = value
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