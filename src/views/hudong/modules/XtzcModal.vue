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
          <a-input placeholder="请输入注册后的天数" v-decorator="['psTime', {}]" />
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="发送时间">
          <a-time-picker
            @change="onChange"
            :v-model="sendTime"
            :defaultOpenValue="moment('00:00:00', 'HH:mm:ss')"
          />
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="消息标题">
          <a-input placeholder="请输入消息标题" v-decorator="['title', {}]" />
        </a-form-item>
        <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="消息简介">
          <a-input placeholder="请输入消息简介" v-decorator="['introduce', {}]" />
        </a-form-item>
        <!--  <a-form-item :labelCol="labelCol" :wrapperCol="wrapperCol" label="消息内容">
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
          v-show="localMenuType ==0"
        >
          <a-textarea placeholder="请输入消息内容" :rows="4" v-decorator="['content', {}]" />
        </a-form-item>
        <a-form-item
          label="图片"
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          v-show="localMenuType ==1"
        >
          <a-upload
            listType="picture-card"
            class="avatar-uploader"
            :showUploadList="false"
            :action="uploadAction"
            :data="{'isup':1}"
            :headers="headers"
            :beforeUpload="beforeUpload"
            @change="handleChange"
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
      content: '',
      localMenuType: '0',
      type: 0,
      headers: {},
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
      }
    }
  },
  created() {
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
    moment,
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
      return window._CONFIG['domianURL']  + this.content
    },
    handleChange(info) {
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
    onChange(time, timeString) {
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
        this.type = 0
        this.localMenuType = 0
      } else {
        this.type = parseInt(record.type)
        this.localMenuType = parseInt(record.type)
      }
      this.content = record.content
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

          this.model.sendTime = this.sendTime

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