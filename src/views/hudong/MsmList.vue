<template>
  <a-row :gutter="10">
    <a-col :md="8" :sm="24">
      <a-card :bordered="false">
        <a-tree :treeData="treeData" defaultExpandAll @select="onSelect">
          <template slot="custom"></template>
        </a-tree>
      </a-card>
    </a-col>
    <a-col :md="16" :sm="24">
      <a-card :bordered="false">
        <!-- 查询区域 -->
        <div class="table-page-search-wrapper">
          <a-form layout="inline">
            <a-row :gutter="24">
              <!-- <a-col :md="16" :sm="24">
                <a-form-item label="描述归属编号">
                  <a-input placeholder="请输入描述归属编号" v-model="queryParam.msBNo"></a-input>
                </a-form-item>
              </a-col>-->

              <a-col :md="16" :sm="24">
                <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                  <!--  <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                  <a-button
                    type="primary"
                    @click="searchReset"
                    icon="reload"
                    style="margin-left: 8px"
                  >重置</a-button>
                  <a @click="handleToggleSearch" style="margin-left: 8px">
                    {{ toggleSearchStatus ? '收起' : '展开' }}
                    <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
                  </a>-->
                </span>
              </a-col>
            </a-row>
          </a-form>
        </div>
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

        <a-modal title="添加详情" v-model="msvisible" @ok="handleOk1()">
          <a-form>
            <a-form-item label="提醒时间" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
              <!-- <a-input format="HH:mm" v-model="mqTime" /> -->
              <a-input
                style=" width: 100px; text-align: center"
                placeholder="小时(两位数)"
                v-model="start"
              />
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
        <a-modal title="修改详情" v-model="msvisible1" @ok="handleOk2()">
          <a-form>
            <a-form-item label="提醒时间" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
              <!-- <a-input format="HH:mm" v-model="mqTime" /> -->
              <a-input
                style=" width: 100px; text-align: center"
                placeholder="小时(两位数)"
                v-model="start"
              />
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
              <a-select placeholder="请选择提醒类型" @change="handleChange" :defaultValue="txTypeText">
                <a-select-option :value="0">静音</a-select-option>
                <a-select-option :value="1">响铃</a-select-option>
                <a-select-option :value="2">震动</a-select-option>
                <a-select-option :value="3">响铃与震动</a-select-option>
              </a-select>
            </a-form-item>
          </a-form>
        </a-modal>

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
        <msm-modal ref="modalForm" @ok="modalFormOk"></msm-modal>
      </a-card>
    </a-col>
  </a-row>
</template>

<script>
import MsmModal from './modules/MsmModal'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { getAction } from '@/api/manage'
import { httpAction } from '@/api/manage'
import { deleteAction } from '@/api/manage'
import moment from 'moment'

const treeData = [
  {
    title: '模式分类',
    key: '0',
    selectable: false,
    children: []
  }
]

export default {
  name: 'MsmList',
  mixins: [JeecgListMixin],
  components: {
    MsmModal
  },
  data() {
    return {
      description: '模版描述',
      dictOptions: [
        {
          text: 'sdasd',
          value: '1'
        }
      ],
      id: ' ',
      msName: '',
      flvalue: '',
      flid: '',
      treeData,
      fltext: '',
      selectedKeys: '',
      msvisible: false,
      msvisible1: false,
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
          title: '描述名称',
          align: 'center',
          dataIndex: 'msName'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: '/msm/msm/list',
        delete: '/msm/msm/delete',
        deleteBatch: '/msm/msm/deleteBatch',
        customerListByMainId: '/mqx/mqXQing/inlineList',
        addMsxq: '/mqx/mqXQing/add',
        editMsxq: '/mqx/mqXQing/edit',
        deleteMsxq: '/mqx/mqXQing/delete',
        addMs: '/msm/msm/add',
        editMs: '/msm/msm/edit',
        fenliList: '/msfenlei/msFenLi/valueList'
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
    moment,
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
          this.selectedKeys = data[0].key
          this.queryParam.flId = data[0].key
          this.searchQuery()
        } else {
          this.$message.warning(res.message)
        }
      })
    },
    onSelect(selectedKeys) {
      if (this.selectedKeys[0] !== selectedKeys[0]) {
        this.selectedKeys = selectedKeys[0]
        this.queryParam.flId = selectedKeys[0]
        this.searchQuery()
      }
    },
    msadd() {
      let key = this.selectedKeys
      if (key.length > 0) {
        //编辑字典数据
        this.msvisible = true
      } else {
        this.$message.info('请先选择模式分类')
      }
    },
    msedit(record) {
      this.id = record.id
      let key = this.selectedKeys
      //编辑字典数据
      this.msvisible1 = true
      this.msName = record.msName
    },
    handleOk1(num) {
      let msName = this.msName
      let id = this.id
      const that = this
      that.confirmLoading = true
      let httpurl
      let method
      let data
      if (num == '1') {
        httpurl = this.url.addMs //添加详情
        method = 'post'
        data = {
          flId: this.selectedKeys,
          msName: msName
        }
      } else if (num == '2') {
        httpurl = this.url.editMs //添加详情
        method = 'put'
        data = {
          flId: this.selectedKeys,
          msName: msName,
          id: id
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
          this.msName = ''
          that.confirmLoading = false
          this.msvisible = false
          this.msvisible1 = false
          this.queryParam.flId = this.selectedKeys
          this.searchQuery()
        })
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