<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline">
        <a-row :gutter="24">
          <a-col :md="6" :sm="8">
            <a-form-item label="开始时间">
              <a-date-picker v-model="queryParam.beginTime" />
            </a-form-item>
          </a-col>
          <a-col :md="6" :sm="8">
            <a-form-item label="结束时间">
              <a-date-picker v-model="queryParam.endTime" />
            </a-form-item>
          </a-col>
          <template v-if="toggleSearchStatus">
            <a-col :md="6" :sm="8">
              <a-form-item label="请输入家长手机号">
                <a-input placeholder="请输入家长手机号" v-model="queryParam.ptPhone"></a-input>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="请输入家长姓名">
                <a-input placeholder="请输入家长姓名" v-model="queryParam.ptName"></a-input>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="请输入孩子手机号">
                <a-input placeholder="请输入孩子手机号" v-model="queryParam.chPhone"></a-input>
              </a-form-item>
            </a-col>
            <a-col :md="6" :sm="8">
              <a-form-item label="请输入孩子姓名">
                <a-input placeholder="请输入孩子姓名" v-model="queryParam.chName"></a-input>
              </a-form-item>
            </a-col>
          </template>
          <a-col :md="6" :sm="8">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button
                type="primary"
                @click="searchReset"
                icon="reload"
                style="margin-left: 8px"
              >重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'" />
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <!-- <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
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
      </a-dropdown>-->
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
          <!-- <a @click="handleEdit(record)">编辑</a> -->
          <a @click="transacText(record)" v-if="record.ltYtype == 'YY'">转文字</a>

          <!-- <a-divider type="vertical" />
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
          </a-dropdown>-->
        </span>
      </a-table>
    </div>
    <!-- table区域-end -->

    <!-- 表单区域 -->
    <liaoTian-modal ref="modalForm" @ok="modalFormOk"></liaoTian-modal>
  </a-card>
</template>

<script>
import LiaoTianModal from './modules/LiaoTianModal'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { httpAction } from '@/api/manage'
import { getAction } from '@/api/manage'

export default {
  name: 'LiaoTianList',
  mixins: [JeecgListMixin],
  components: {
    LiaoTianModal
  },
  data() {
    return {
      description: '聊天管理页面',
      beginTime: '',
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
          title: '消息所属类型',
          align: 'center',
          dataIndex: 'ltVtype_dictText'
        },
        {
          title: '消息类型',
          align: 'center',
          dataIndex: 'ltYtype_dictText'
        },
        {
          title: '家长',
          align: 'center',
          dataIndex: 'ptName'
        },
        {
          title: '孩子',
          align: 'center',
          dataIndex: 'chName'
        },
        {
          title: '语音内容',
          align: 'center',
          dataIndex: 'yytext'
        },
        {
          title: '消息内容',
          align: 'center',
          dataIndex: 'ltContent',
          customRender: function(text, record, index) {
            let imageUrl = window._CONFIG['img'] + record.ltContent
            if (record.ltYtype == 'WZ') {
              return record.ltContent
            }
            if (record.ltYtype == 'IMG') {
              return <img src={imageUrl} style="width: 100px;height: 50px" />
            }

            if (record.ltYtype == 'YY') {
              return <audio src={imageUrl} controls="controls" style="width: 200px;height: 50px"></audio>
            }
          }
        },
        {
          title: '消息发送时间',
          align: 'center',
          dataIndex: 'createTime'
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: 'center',
          scopedSlots: { customRender: 'action' }
        }
      ],
      url: {
        list: '/liaotian/liaoTian/list',
        delete: '/liaotian/liaoTian/delete',
        deleteBatch: '/liaotian/liaoTian/deleteBatch',
        exportXlsUrl: 'liaotian/liaoTian/exportXls',
        importExcelUrl: 'liaotian/liaoTian/importExcel',
        transacText: 'liaotian/liaoTian/transacText'
      }
    }
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    transacText(record) {
      let httpUrl = this.url.transacText
      let data = {
        id: record.id
      }
      let formData = Object.assign(data)
      let that = this
      this.loading = true
      httpAction(httpUrl, formData, 'post').then(res => {
        if (res.success) {
          this.loading = false
          that.$message.success(res.message)
          that.loadData()
        } else {
          this.loading = false
          that.$message.warning(res.message)
        }
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