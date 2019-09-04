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
              <a-col :md="6" :sm="8">
                <a-form-item label="孩子姓名">
                  <a-input placeholder="请输入孩子姓名" v-model="queryParam.cdName"></a-input>
                </a-form-item>
              </a-col>
              <a-col :md="6" :sm="8">
                <a-form-item label="孩子手机号">
                  <a-input placeholder="请输入孩子手机号" v-model="queryParam.cdPhone"></a-input>
                </a-form-item>
              </a-col>
              <a-col :md="6" :sm="8">
                <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
                  <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
                </span>
                <a-button
                  type="primary"
                  @click="searchReset"
                  icon="reload"
                  style="margin-left: 8px"
                >重置</a-button>
              </a-col>
            </a-row>
          </a-form>
        </div>

        <!-- 操作按钮区域 -->
        <!-- <div class="table-operator">
          <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
        </div>-->

        <!-- table区域-begin -->
        <div>
          <!-- <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择
        <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
          </div>-->

          <a-modal title="关联模版" v-model="msvisible" @ok="handleOk1">
            <a-form>
              <a-form-item label="分类模版" :label-col="{ span: 5 }" :wrapper-col="{ span: 12 }">
                <a-cascader :options="options" @change="onChange" />
              </a-form-item>
            </a-form>
          </a-modal>

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
              <!-- <a @click="relateMs(record)" v-if="record.msId == null">关联模版</a>
              <a-tooltip placement="topLeft" v-if="record.msId != null">
                <template slot="title">
                  <span>{{record.msName}}</span>
                </template>
                <a @click="relateMs(record)">编辑模版</a>
              </a-tooltip>-->
              <a-upload
                name="file"
                :showUploadList="false"
                :multiple="false"
                :action="importExcelUrl"
                @change="handleImportExcel"
                :data="{childid: record.id}"
              >
                <a type="primary" icon="import">导入课表</a>
              </a-upload>
              <a-divider type="vertical" />
              <!--  <a @click="handleEdit(record)">编辑</a> -->

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
        <child-modal ref="modalForm" @ok="modalFormOk"></child-modal>
      </a-card>
    </a-col>
  </a-row>
</template>

<script>
import ChildModal from './modules/ChildModal'
import { JeecgListMixin } from '@/mixins/JeecgListMixin'
import { httpAction } from '@/api/manage'

const treeData = [
  {
    title: '家长列表',
    key: '0',
    selectable: false,
    children: []
  }
]

export default {
  name: 'ChildList',
  mixins: [JeecgListMixin],
  components: {
    ChildModal
  },
  data() {
    return {
      description: '儿童管理管理页面',
      visible: false,
      cid: '',
      pthing: '', //父母的姓名或手机号
      msvisible: false,
      msid: '',
      options: [],
      pid: '',
      treeData,
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
          dataIndex: 'cdName'
        },
        {
          title: '孩子手机号',
          align: 'center',
          dataIndex: 'cdPhone'
        },
        {
          title: '年级',
          align: 'center',
          dataIndex: 'flName'
        },
        {
          title: '注册时间',
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
        list: '/child/child/list',
        delete: '/child/child/delete',
        deleteBatch: '/child/child/deleteBatch',
        exportXlsUrl: 'child/child/exportXls',
        msListUrl: '/msfenlei/msFenLi/msListUrl', //下拉选择模版
        updateChild: '/child/child/saveMsChild',
        parentList: '/parent/parent/valueList',
        importExcelUrl: '/front/parent/vipCenter/exportKc' //导入孩子的课程表
      }
    }
  },
  mounted() {
    this.handSearch()
  },
  computed: {
    importExcelUrl: function() {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`
    }
  },
  methods: {
    handSearch() {
      this.$http.get(this.url.parentList).then(res => {
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
          this.queryParam.ptId = data[0].key
          this.searchQuery()
        } else {
          this.$message.warning(res.message)
        }
      })
    },
    onSelect(selectedKeys) {
      if (this.selectedKeys[0] !== selectedKeys[0]) {
        this.selectedKeys = selectedKeys[0]
        this.queryParam.ptId = selectedKeys[0]
        this.pid = selectedKeys[0]
        this.searchQuery()
      }
    },
    handleAdd: function() {
      let parentId = this.pid
      if (parentId == '') {
        this.$message.warning('请先选择父母')
      } else {
        this.$refs.modalForm.add(parentId)
        this.$refs.modalForm.title = '新增'
      }
    },
    add() {
      this.edit({})
    },
    edit(record) {
      this.form.resetFields()
      this.model = Object.assign({}, record)
      this.visible = true
      this.$nextTick(() => {
        this.form.setFieldsValue(
          pick(this.model, 'cdId', 'ptId', 'cdName', 'cdBirthday', 'cdSex', 'cdPhone', 'cdPassword')
        )
        //时间格式化
      })
      this.loadData()
    },
    relateMs(record) {
      this.options = []
      let httpUrl = this.url.msListUrl
      httpAction(httpUrl, '', 'get').then(res => {
        if (res.success) {
          this.options = res.result.options
          this.msvisible = true
          this.cid = record.id
        }
      })
    },
    handleOk1: function() {
      let msids = this.msid //模式传过来的是带分类的,所以把它去掉
      console.log(msids)

      //使用child的修改接口
      let httpUrl = this.url.updateChild
      let data = {
        id: this.cid,
        flId: msids[0]
      }
      let formData = Object.assign(data)
      let that = this
      httpAction(httpUrl, formData, 'post').then(res => {
        if (res.success) {
          this.msvisible = false
          that.$message.success(res.message)
          that.loadData()
        } else {
          that.$message.warning(res.message)
        }
      })
    },
    onChange(value) {
      this.msid = value
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