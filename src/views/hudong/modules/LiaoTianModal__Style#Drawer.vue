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
      
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="聊天记录ID">
          <a-input placeholder="请输入聊天记录ID" v-decorator="['ltId', validatorRules.ltId ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="消息所属用户ID">
          <a-input placeholder="请输入消息所属用户ID" v-decorator="['ltVipId', validatorRules.ltVipId ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="消息所属类型 XT/系统,HZ/孩子,JZ/家长">
          <a-input placeholder="请输入消息所属类型 XT/系统,HZ/孩子,JZ/家长" v-decorator="['ltVtype', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="消息类型 WZ/文字,YY/语音,IMG/图片">
          <a-input placeholder="请输入消息类型 WZ/文字,YY/语音,IMG/图片" v-decorator="['ltYtype', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="消息内容">
          <a-input placeholder="请输入消息内容" v-decorator="['ltContent', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="消息标记(用于标记这些记录同属于一天之内的指定用户的对话)">
          <a-input placeholder="请输入消息标记(用于标记这些记录同属于一天之内的指定用户的对话)" v-decorator="['ltMark', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="消息时间">
          <a-input placeholder="请输入消息时间" v-decorator="['ltTime', {}]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="消息读取状态 Y/已读.N/未读,默认为N">
          <a-input placeholder="请输入消息读取状态 Y/已读.N/未读,默认为N" v-decorator="['ltRead', {}]" />
        </a-form-item>
		
      </a-form>
    </a-spin>
    <a-button type="primary" @click="handleOk">确定</a-button>
    <a-button type="primary" @click="handleCancel">取消</a-button>
  </a-drawer>
</template>

<script>
  import { httpAction } from '@/api/manage'
  import pick from 'lodash.pick'
  import moment from "moment"

  export default {
    name: "LiaoTianModal",
    data () {
      return {
        title:"操作",
        visible: false,
        model: {},
        labelCol: {
          xs: { span: 24 },
          sm: { span: 5 },
        },
        wrapperCol: {
          xs: { span: 24 },
          sm: { span: 16 },
        },

        confirmLoading: false,
        form: this.$form.createForm(this),
        validatorRules:{
        ltId:{rules: [{ required: true, message: '请输入聊天记录ID!' }]},
        ltVipId:{rules: [{ required: true, message: '请输入消息所属用户ID!' }]},
        },
        url: {
          add: "/liaotian/liaoTian/add",
          edit: "/liaotian/liaoTian/edit",
        },
      }
    },
    created () {
    },
    methods: {
      add () {
        this.edit({});
      },
      edit (record) {
        this.form.resetFields();
        this.model = Object.assign({}, record);
        this.visible = true;
        this.$nextTick(() => {
          this.form.setFieldsValue(pick(this.model,'ltId','ltVipId','ltVtype','ltYtype','ltContent','ltMark','ltTime','ltRead'))
		  //时间格式化
        });

      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleOk () {
        const that = this;
        // 触发表单验证
        this.form.validateFields((err, values) => {
          if (!err) {
            that.confirmLoading = true;
            let httpurl = '';
            let method = '';
            if(!this.model.id){
              httpurl+=this.url.add;
              method = 'post';
            }else{
              httpurl+=this.url.edit;
               method = 'put';
            }
            let formData = Object.assign(this.model, values);
            //时间格式化
            
            console.log(formData)
            httpAction(httpurl,formData,method).then((res)=>{
              if(res.success){
                that.$message.success(res.message);
                that.$emit('ok');
              }else{
                that.$message.warning(res.message);
              }
            }).finally(() => {
              that.confirmLoading = false;
              that.close();
            })



          }
        })
      },
      handleCancel () {
        this.close()
      },


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