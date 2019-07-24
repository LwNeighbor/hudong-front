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
        <a-form-item label="提醒时间" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <!-- <a-input format="HH:mm" v-model="mqTime" /> -->
          <a-input style=" width: 100px; text-align: center" placeholder="小时(两位数)" v-model="start" />
          <a-input
            style=" width: 30px; border-left: 0; pointer-events: none; backgroundColor: #fff"
            placeholder=":"
            disabled
          />
          <a-input
            style="width: 100px; text-align: center; border-left: 0"
            placeholder="分钟(两位数)"
            v-model="end"
          />
        </a-form-item>
        <a-form-item label="项目" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-input placeholder="项目" v-model="mqKemu" />
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
        <a-form-item label="提醒时间" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <!-- <a-input format="HH:mm" v-model="mqTime" /> -->
          <a-input style=" width: 100px; text-align: center" placeholder="小时(两位数)" v-model="start" />
          <a-input
            style=" width: 30px; border-left: 0; pointer-events: none; backgroundColor: #fff"
            placeholder=":"
            disabled
          />
          <a-input
            style="width: 100px; text-align: center; border-left: 0"
            placeholder="分钟(两位数)"
            v-model="end"
          />
        </a-form-item>
        <a-form-item label="项目" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-input placeholder="项目" v-model="mqKemu" />
        </a-form-item>
        <a-form-item label="提醒内容" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-textarea placeholder="请输入提醒内容" :rows="4" v-model="mqContent" />
        </a-form-item>
        <a-form-item label="提醒类型" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
          <a-select placeholder="请选择提醒类型" @change="handleChange"  :defaultValue="txTypeText">
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
        <a-tabs defaultActiveKey="2" @change="callback">
          <a-tab-pane tab="周一" key="2" forceRender>
            <!-- 操作按钮区域 -->
            <div class="table-operator">
              <a-button @click="msadd()" type="primary" icon="plus">新增</a-button>
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
          </a-tab-pane>
          <a-tab-pane tab="周二" key="3" forceRender>
            <!-- 操作按钮区域 -->
            <div class="table-operator">
              <a-button @click="msadd()" type="primary" icon="plus">新增</a-button>
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
          </a-tab-pane>
          <a-tab-pane tab="周三" key="4" forceRender>
            <!-- 操作按钮区域 -->
            <div class="table-operator">
              <a-button @click="msadd()" type="primary" icon="plus">新增</a-button>
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
          </a-tab-pane>
          <a-tab-pane tab="周四" key="5" forceRender>
            <!-- 操作按钮区域 -->
            <div class="table-operator">
              <a-button @click="msadd()" type="primary" icon="plus">新增</a-button>
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
          </a-tab-pane>
          <a-tab-pane tab="周五" key="6" forceRender>
            <!-- 操作按钮区域 -->
            <div class="table-operator">
              <a-button @click="msadd()" type="primary" icon="plus">新增</a-button>
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
          </a-tab-pane>
          <a-tab-pane tab="周六" key="7" forceRender>
            <!-- 操作按钮区域 -->
            <div class="table-operator">
              <a-button @click="msadd()" type="primary" icon="plus">新增</a-button>
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
          </a-tab-pane>
          <a-tab-pane tab="周日" key="1" forceRender>
            <!-- 操作按钮区域 -->
            <div class="table-operator">
              <a-button @click="msadd()" type="primary" icon="plus">新增</a-button>
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
          </a-tab-pane>
        </a-tabs>
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
    title: '模式',
    key: '0',
    selectable: false,
    disabled: true,
    children: []
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
          title: '详情时间',
          align: 'center',
          dataIndex: 'mqTime'
        },
        {
          title: '项目',
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
      week: '2', //默认是周一
      start: '',
      end: '',
      id: ' ',
      mqTime: '',
      mqKemu: '',
      mqContent: '',
      msValue: '',
      msId: '',
      txType: '',
      txTypeText: '',
      treeData,
      selectedKeys: '',
      msvisible: false,
      msvisible1: false,
      url: {
        list: '/mqx/mqXQing/list',
        delete: '/mqx/mqXQing/delete',
        deleteBatch: '/mqx/mqXQing/deleteBatch',
        exportXlsUrl: 'mqx/mqXQing/exportXls',
        importExcelUrl: 'mqx/mqXQing/importExcel',
        addMq: '/mqx/mqXQing/add',
        editMq: '/mqx/mqXQing/edit',
        deleteMq: '/mqx/mqXQing/delete',
        msList: '/msm/msm/valueList'
      }
    }
  },
  mounted() {
    this.handSearch()
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    },
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
      params.week = this.week
      getAction(this.url.list, params).then(res => {
        if (res.success) {
          this.dataSource = res.result.records
          this.ipagination.total = res.result.total
        }
      })
    },
    callback(key) {
      this.week = key
      var params = this.getQueryParams() //查询条件
      params.week = key
      getAction(this.url.list, params).then(res => {
        if (res.success) {
          this.dataSource = res.result.records
          this.ipagination.total = res.result.total
        }
      })
    },
    handSearch() {
      this.$http.get(this.url.msList).then(res => {
        if (res.success) {
          const result = res.result
          const data = []

          result.forEach(r => {
            const data1 = []
            var innerData = r.children
            innerData.forEach(i => {
              data1.push({
                key: i.value,
                title: i.text
              })
            })
            data.push({
              key: r.value,
              title: r.text,
              disabled: true,
              children: data1
            })
          })
          this.treeData[0].children = data
          //this.selectedKeys = data[0].key
          this.queryParam.msId = data[0].key
          this.queryParam.week = this.week
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
          this.queryParam.msId = selectedKeys[0]
          this.queryParam.week = this.week
          this.searchQuery()
        }
      }
    },
    msadd() {
      let key = this.selectedKeys
      if (key.length > 0) {
        //编辑字典数据
        this.msvisible = true
      } else {
        this.$message.info('请先选择模式')
      }
    },
    msedit(record) {
      this.transTxType(record.txType)
      this.id = record.id
      let key = this.selectedKeys
      this.txType = record.txType
      //编辑字典数据
      this.msvisible1 = true
      this.mqTime = record.mqTime
      this.start = record.mqTime.split(':')[0]
      this.end = record.mqTime.split(':')[1]
      this.mqKemu = record.mqKemu
      this.mqContent = record.mqContent
    },
    handleOk(num) {
      let start = this.start
      let end = this.end
      let mqTime = start + ':' + end
      let mqKemu = this.mqKemu
      let mqContent = this.mqContent
      let id = this.id
      let txType = this.txType
      const that = this

      that.confirmLoading = true
      let httpurl
      let method
      let data
      if (num == '1') {
        httpurl = this.url.addMq //添加详情
        method = 'post'
        data = {
          msId: this.selectedKeys,
          mqTime: mqTime,
          mqKemu: mqKemu,
          mqContent: mqContent,
          week: this.week,
          txType: this.txType
        }
      } else if (num == '2') {
        httpurl = this.url.editMq //修改详情
        method = 'put'
        data = {
          msId: this.selectedKeys,
          mqTime: mqTime,
          mqKemu: mqKemu,
          mqContent: mqContent,
          id: id,
          week: this.week,
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
          this.start = ''
          this.end = ''
          this.mqTime = ''
          this.mqKemu = ''
          this.mqContent = ''
          that.confirmLoading = false
          this.msvisible = false
          this.msvisible1 = false
          this.queryParam.msId = this.selectedKeys
          this.queryParam.week = this.week
          this.searchQuery()
        })
    },
    handleChange(value) {
      this.txType = value
    },
    transTxType(value){
      if(value == '0'){
        this.txTypeText = "默认"
      }else if(value == '1'){
        this.txTypeText = '响铃'
      }else if(value == '2'){
        this.txTypeText = '震动'
      }else if(value == '3'){
        this.txTypeText = '响铃并震动'
      }
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