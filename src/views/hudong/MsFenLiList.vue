<template>
  <a-card :bordered="false">
    <!-- 左侧面板 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="12">
          <a-col :md="7" :sm="8">
            <a-form-item label="分类">
              <a-input placeholder="请输入分类" v-model="queryParam.flName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
            </span>
          </a-col>
          <a-col :md="24" :sm="24">
            <a-button style="margin-bottom: 20px" @click="handleAdd" type="primary" icon="plus">添加</a-button>
          </a-col>
        </a-row>
      </a-form>

      <a-table
        ref="table"
        rowKey="id"
        size="middle"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        @change="handleTableChange"
      >
        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">
            <a-icon type="edit"/>编辑
          </a>
          <a-divider type="vertical"/>
          <a @click="handleDelete(record.id)">删除</a>
        </span>
      </a-table>
    </div>
    <msFenLi-modal ref="modalForm" @ok="modalFormOk"></msFenLi-modal>
  </a-card>
</template>

<script>
import { filterObj } from '@/utils/util'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import MsFenLiModal from './modules/MsFenLiModal'
import MsmList from './MsmList'
import { delDict } from '@/api/api'

export default {
  name: 'MsFenLiList',
  mixins: [JeecgListMixin],
  components: { MsFenLiModal, MsmList },
  data() {
    return {
      description: '模式分类管理页面',
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
          title: '分类',
          align: 'center',
          dataIndex: 'flName'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: '/msfenlei/msFenLi/list',
        delete: '/msfenlei/msFenLi/delete',
        deleteBatch: '/msfenlei/msFenLi/deleteBatch',
        exportXlsUrl: 'msfenlei/msFenLi/exportXls',
        importExcelUrl: 'msfenlei/msFenLi/importExcel'
      }
    }
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
   
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