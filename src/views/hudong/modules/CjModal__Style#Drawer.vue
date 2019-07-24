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
          label="学科ID">
          <a-input placeholder="请输入学科ID" v-decorator="['cjId', validatorRules.cjId ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="孩子ID">
          <a-input placeholder="请输入孩子ID" v-decorator="['cjCdId', validatorRules.cjCdId ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="学科ID">
          <a-input placeholder="请输入学科ID" v-decorator="['cjXkId', validatorRules.cjXkId ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="学科成绩">
          <a-input placeholder="请输入学科成绩" v-decorator="['cjNumber', validatorRules.cjNumber ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="成绩时间">
          <a-input placeholder="请输入成绩时间" v-decorator="['cjTime', validatorRules.cjTime ]" />
        </a-form-item>
        <a-form-item
          :labelCol="labelCol"
          :wrapperCol="wrapperCol"
          label="孩子姓名">
          <a-input placeholder="请输入孩子姓名" v-decorator="['cjCdName', validatorRules.cjCdName ]" />
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
    name: "CjModal",
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
        cjId:{rules: [{ required: true, message: '请输入学科ID!' }]},
        cjCdId:{rules: [{ required: true, message: '请输入孩子ID!' }]},
        cjXkId:{rules: [{ required: true, message: '请输入学科ID!' }]},
        cjNumber:{rules: [{ required: true, message: '请输入学科成绩!' }]},
        cjTime:{rules: [{ required: true, message: '请输入成绩时间!' }]},
        cjCdName:{rules: [{ required: true, message: '请输入孩子姓名!' }]},
        },
        url: {
          add: "/cj/cj/add",
          edit: "/cj/cj/edit",
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
          this.form.setFieldsValue(pick(this.model,'cjId','cjCdId','cjXkId','cjNumber','cjTime','cjCdName'))
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