<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="孩子姓名">
              <a-input placeholder="请输入孩子姓名" v-model="queryParam.name"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <a-form-item label="孩子手机号">
              <a-input placeholder="请输入孩子手机号" v-model="queryParam.phone"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <a-form-item label="星期">
              <a-input placeholder="请输入星期数" v-model="queryParam.weekday"></a-input>
            </a-form-item>
          </a-col>

          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button
                type="primary"
                @click="searchReset"
                icon="reload"
                style="margin-left: 8px"
              >重置</a-button>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <!-- <a-upload
        name="file"
        :showUploadList="false"
        :multiple="false"
        :action="importExcelUrl"
        @change="handleImportExcel"
      >
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload> -->
      <!-- <a-button type="primary" icon="download" @click="handleExportXls">下载课程模版</a-button> -->
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete" />删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px">
          批量操作
          <a-icon type="down" />
        </a-button>
      </a-dropdown>
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
        :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
        @change="handleTableChange"
      >
        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">
              更多
              <a-icon type="down" />
            </a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <kc-modal ref="modalForm" @ok="modalFormOk"></kc-modal>
  </a-card>
</template>

<script>
import KcModal from './modules/KcModal'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'

export default {
  name: 'KcList',
  mixins: [JeecgListMixin],
  components: {
    KcModal
  },
  data() {
    return {
      description: '课程表管理管理页面',
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
          title: '孩子姓名',
          align: 'center',
          dataIndex: 'name'
        },
        {
          title: '孩子手机号',
          align: 'center',
          dataIndex: 'phone'
        },
        {
          title: '科目名称',
          align: 'center',
          dataIndex: 'kmName'
        },
        {
          title: '年级',
          align: 'center',
          dataIndex: 'flName'
        },
        {
          title: '上课时间',
          align: 'center',
          dataIndex: 'startTime'
        },
        {
          title: '下课时间',
          align: 'center',
          dataIndex: 'endTime'
        },
        {
          title: '星期',
          align: 'center',
          dataIndex: 'weekday',
          sortOrder: 'ascend'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: '/kc/kc/list',
        delete: '/kc/kc/delete',
        deleteBatch: '/kc/kc/deleteBatch',
        exportXlsUrl: '/kcExcel.xls',
        importExcelUrl: '/front/parent/vipCenter/exportKc'
      }
    }
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    },
   /*  handleExportXls: function() {
      return `${window._CONFIG['domianURL']}/${this.url.exportXlsUrl}`
    } */
  },
  methods: {}
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