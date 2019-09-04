<template>
  <a-row :gutter="10">
    <a-col :md="4" :sm="24">
      <a-card :bordered="false">
        <a-tree :treeData="treeData" defaultExpandAll @select="onSelect">
          <template slot="custom"></template>
        </a-tree>
      </a-card>
    </a-col>
    <a-modal title="添加详情" v-model="msvisible" @ok="handleOk(1)">
      <a-form>
        <a-form-item label="注册天数" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-input placeholder="注册天数" v-model="registerDay" />
        </a-form-item>
        <a-form-item label="提前分钟数" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-input placeholder="提前分钟数" v-model="mqTime" />
        </a-form-item>
        <a-form-item label="科目" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-cascader :options="options" @change="onChange" />
        </a-form-item>
        <a-form-item label="提醒内容" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-textarea placeholder="请输入提醒内容" :rows="4" v-model="mqContent" />
        </a-form-item>
        <a-form-item label="提醒类型" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-select placeholder="请选择提醒类型" @change="handleChange">
            <a-select-option :value="0">静音</a-select-option>
            <a-select-option :value="1">响铃</a-select-option>
            <a-select-option :value="2">震动</a-select-option>
            <a-select-option :value="3">响铃与震动</a-select-option>
          </a-select>
        </a-form-item>
      </a-form>
    </a-modal>
    <a-modal title="修改详情" v-model="msvisible1" @ok="handleOk(2)">
      <a-form>
        <a-form-item label="注册天数" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-input placeholder="注册天数" v-model="registerDay" />
        </a-form-item>
        <a-form-item label="提前分钟数" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-input placeholder="提前分钟数" v-model="mqTime" />
        </a-form-item>
        <a-form-item label="科目" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-cascader :options="options" @change="onChange" />
        </a-form-item>
        <a-form-item label="提醒内容" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-textarea placeholder="请输入提醒内容" :rows="4" v-model="mqContent" />
        </a-form-item>
        <a-form-item label="提醒类型" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-select placeholder="请选择提醒类型" @change="handleChange" :defaultValue="txTypeText">
            <a-select-option :value="0">静音</a-select-option>
            <a-select-option :value="1">响铃</a-select-option>
            <a-select-option :value="2">震动</a-select-option>
            <a-select-option :value="3">响铃与震动</a-select-option>
          </a-select>
        </a-form-item>
      </a-form>
    </a-modal>
    <!--  -->
    <a-col :md="20" :sm="24">
      <a-card :bordered="false">
        <!-- 操作按钮区域 -->
        <div class="table-operator">
          <a-button @click="msadd()" type="primary" icon="plus">新增</a-button>
          <!--  <a-button type="primary" icon="download" @click="handleExportXls">导出模版</a-button> -->
          <a-upload
            name="file"
            :showUploadList="false"
            :multiple="false"
            :action="importExcelUrl"
            @change="handleImportExcel"
            :headers="headers"
            :beforeUpload="beforeUpload"
          >
            <a-button type="primary" icon="import">导入</a-button>
          </a-upload>
          <a-button
            @click="batchDel"
            style="margin-left:8px"
            v-if="selectedRowKeys.length > 0"
            ghost
            type="primary"
            icon="delete"
          >批量删除</a-button>
        </div>

        <!-- table区域-begin -->
        <div>
          <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
            <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择
            <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
            <a style="margin-left: 24px" @click="onClearSelected">清空</a>
          </div>

          <a-table
            ref="table"
            size="middle"
            bordered
            rowKey="id"
            :columns="columns"
            :dataSource="dataSource"
            :pagination="ipagination"
            :loading="loading"
            :rowSelection="{onChange: onSelectChange}"
            @change="handleTableChange"
          >
            <span slot="action" slot-scope="text, record">
              <a @click="msedit(record)">编辑</a>
              <a-divider type="vertical" />
              <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                <a>删除</a>
              </a-popconfirm>
            </span>
          </a-table>
        </div>
        <!-- table区域-end -->

        <!-- 表单区域 -->
        <mqXQing-modal ref="modalForm" @ok="modalFormOk"></mqXQing-modal>
      </a-card>
    </a-col>
  </a-row>
</template>

<script>
import MqXQingModal from './modules/MqXQingModal'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction } from '@/api/manage'
import { httpAction } from '@/api/manage'
import { deleteAction } from '@/api/manage'
import moment from 'moment'

const treeData = [
  {
    title: '年级',
    key: '0',
    selectable: false,
    disabled: true
  }
]
export default {
  name: 'MqXQingList',
  mixins: [JeecgListMixin],
  components: {
    MqXQingModal
  },
  data() {
    return {
      description: '模式详情管理页面',
      // 表头
      columns: [
        {
          title: '#',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: 'center',
          customRender: function(t, r, index) {
            return parseInt(index) + 1
          }
        },
        {
          title: '注册天数',
          align: 'center',
          dataIndex: 'registerDay'
        },
        {
          title: '提前分钟数',
          align: 'center',
          dataIndex: 'mqTime'
        },
        {
          title: '科目',
          align: 'center',
          dataIndex: 'mqKemu'
        },
        {
          title: '详情内容',
          align: 'center',
          dataIndex: 'mqContent'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          scopedSlots: { customRender: 'action' }
        }
      ],
      id: ' ',
      mqTime: '',
      mqKemu: '',
      mqContent: '',
      flId: '',
      headers:{},
      txType: '',
      registerDay: '',
      txTypeText: '',
      options: [],
      treeData,
      selectedKeys: '',
      msvisible: false,
      msvisible1: false,
      url: {
        list: '/mqx/mqXQing/list',
        delete: '/mqx/mqXQing/delete',
        deleteBatch: '/mqx/mqXQing/deleteBatch',
        exportXlsUrl: 'mqx/mqXQing/exportXls',
        importExcelUrl: '/front/parent/vipCenter/importExcel',
        addMq: '/mqx/mqXQing/add',
        editMq: '/mqx/mqXQing/edit',
        deleteMq: '/mqx/mqXQing/delete',
        fenliList: '/msfenlei/msFenLi/valueList',
        xuekeList: '/xk/xueKe/valueList'
      }
    }
  },
  mounted() {
    this.handSearch()
  },
  computed: {
    currentId() {
      return this.id
    }
  },
  methods: {
    loadData(arg) {
      if (!this.url.list) {
        this.$message.error('请设置url.list属性!')
        return
      }
      //加载数据 若传入参数1则加载第一页的内容
      if (arg === 1) {
        this.ipagination.current = 1
      }
      var params = this.getQueryParams() //查询条件
      getAction(this.url.list, params).then(res => {
        if (res.success) {
          this.dataSource = res.result.records
          this.ipagination.total = res.result.total
        }
      })
    },
    callback(key) {
      var params = this.getQueryParams() //查询条件
      getAction(this.url.list, params).then(res => {
        if (res.success) {
          this.dataSource = res.result.records
          this.ipagination.total = res.result.total
        }
      })
    },
    selectKemu() {
      let httpUrl = this.url.xuekeList
      httpAction(httpUrl, { flId: this.flId }, 'post').then(res => {
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
    handSearch() {
      this.$http.get(this.url.fenliList).then(res => {
        if (res.success) {
          const result = res.result
          const data = []
          result.forEach(r => {
            data.push({
              key: r.value,
              title: r.text
            })
          })
          this.treeData[0].children = data
          /*  this.selectedKeys = data[0].key */
          this.queryParam.flId = data[0].key
          this.searchQuery()
        } else {
          this.$message.warning(res.message)
        }
      })
    },
    onSelect(selectedKeys) {
      if (selectedKeys.length > 0) {
        if (this.selectedKeys[0] !== selectedKeys[0]) {
          this.selectedKeys = selectedKeys[0]
          this.queryParam.flId = selectedKeys[0]
          this.flId = selectedKeys[0]
          this.searchQuery()
        }
      }
    },
    msadd() {
      this.selectKemu()
      let key = this.selectedKeys
      if (key.length > 0) {
        //编辑字典数据
        this.msvisible = true
      } else {
        this.$message.info('请先选择年级')
      }
    },
    msedit(record) {
      this.selectedKeys = record.flId
      this.flId = record.flId
      this.selectKemu()
      this.transTxType(record.txType)
      this.id = record.id
      let key = this.selectedKeys
      this.txType = record.txType
      this.registerDay = record.registerDay
      //编辑字典数据
      this.msvisible1 = true
      this.mqTime = record.mqTime
      this.mqKemu = record.mqKemu
      this.mqContent = record.mqContent
    },
    handleOk(num) {
      let mqTime = this.mqTime
      let mqKemu = this.mqKemu
      let mqContent = this.mqContent
      let id = this.id
      let txType = this.txType
      let registerDay = this.registerDay
      const that = this

      that.confirmLoading = true
      let httpurl
      let method
      let data
      if (num == '1') {
        httpurl = this.url.addMq //添加详情
        method = 'post'
        data = {
          flId: this.selectedKeys,
          mqTime: mqTime,
          mqKemu: mqKemu,
          mqContent: mqContent,
          registerDay: registerDay,
          txType: this.txType
        }
      } else if (num == '2') {
        httpurl = this.url.editMq //修改详情
        method = 'put'
        data = {
          flId: this.selectedKeys,
          mqTime: mqTime,
          mqKemu: mqKemu,
          mqContent: mqContent,
          registerDay: registerDay,
          id: id,
          txType: this.txType
        }
      }

      let formData = Object.assign(data)
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
          this.mqTime = ''
          this.mqKemu = ''
          this.mqContent = ''
          this.registerDay = ''
          that.confirmLoading = false
          this.msvisible = false
          this.msvisible1 = false
          this.queryParam.flId = this.selectedKeys
          this.searchQuery()
        })
    },
    handleChange(value) {
      this.txType = value
    },
    transTxType(value) {
      if (value == '0') {
        this.txTypeText = '默认'
      } else if (value == '1') {
        this.txTypeText = '响铃'
      } else if (value == '2') {
        this.txTypeText = '震动'
      } else if (value == '3') {
        this.txTypeText = '响铃并震动'
      }
    },
    onChange(value) {
      this.mqKemu = value[0]
    },
    /* 导入 */
    handleImportExcel(info) {
      if (info.file.status !== 'uploading') {
        console.log(info.file, info.fileList)
      }
      if (info.file.status === 'done') {
        this.$message.success(`${info.file.name} 文件上传成功`)
        this.loadData()
      } else if (info.file.status === 'error') {
        this.$message.error(`${info.file.name} 文件上传失败.`)
      }
    },
    beforeUpload(file) {
      let key = this.selectedKeys
      this.headers = {
        fiId: this.selectedKeys
      }
      if (key.length > 0) {
        //编辑字典数据
        //this.msvisible = true
        return true
      } else {
        this.$message.info('请先选择年级')
        return false
      }
    },
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  }
}
</script>
<style lang="less" scoped>
/** Button按钮间距 */
.ant-btn {
  margin-left: 3px;
}
.ant-card-body .table-operator {
  margin-bottom: 18px;
}
.ant-table-tbody .ant-table-row td {
  padding-top: 15px;
  padding-bottom: 15px;
}
.anty-row-operator button {
  margin: 0 5px;
}
.ant-btn-danger {
  background-color: #ffffff;
}

.ant-modal-cust-warp {
  height: 100%;
}
.ant-modal-cust-warp .ant-modal-body {
  height: calc(100% - 110px) !important;
  overflow-y: auto;
}
.ant-modal-cust-warp .ant-modal-content {
  height: 90% !important;
  overflow-y: hidden;
}
</style>