<template>
  <div class="execution">
    <basic-container>
      <avue-crud
        ref="crud"
        :page="page"
        :data="tableData"
        :table-loading="tableLoading"
        :option="tableOption"
        @on-load="getList"
        @search-change="searchChange"
        @refresh-change="refreshChange"
        @row-del="rowDel"
      >
        <template slot-scope="scope" slot="menuBtn">
          <el-dropdown-item
            v-if="permissions.act_process_manage"
            divided
            @click.native="handlePic(scope.row,scope.index)"
          >流程图
          </el-dropdown-item>
          <el-dropdown-item
            v-if="permissions.act_process_manage && scope.row.suspend"
            divided
            @click.native="handleStatus(scope.row,'active')"
          >激活
          </el-dropdown-item>

          <el-dropdown-item
            v-if="permissions.act_process_manage && !scope.row.suspend"
            divided
            @click.native="handleStatus(scope.row,'suspend')"
          >失效
          </el-dropdown-item>

          <el-dropdown-item
            v-if="permissions.act_process_manage"
            divided
            @click.native="handleDel(scope.row,'suspend')"
          >删除
          </el-dropdown-item>
        </template>
      </avue-crud>
    </basic-container>
    <el-dialog :visible.sync="showPicDialog" title="流程图">
      <img :src="actPicUrl" width="100%">
    </el-dialog>
  </div>
</template>

<script>
import { delObj, fetchList, status } from '@/api/activiti/process'
import { tableOption } from '@/const/crud/activiti/process'
import { mapGetters } from 'vuex'

export default {
  name: 'Process',
  data() {
    return {
      searchForm: {},
      showPicDialog: false,
      actPicUrl: '',
      tableData: [],
      page: {
        total: 0, // 总页数
        currentPage: 1, // 当前页数
        pageSize: 20 // 每页显示多少条
      },
      tableLoading: false,
      tableOption: tableOption
    }
  },
  created() {
  },
  mounted: function() {
  },
  computed: {
    ...mapGetters(['permissions'])
  },
  methods: {
    getList(page, params) {
      this.tableLoading = true
      fetchList(
        Object.assign(
          {
            descs: 'create_time',
            current: page.currentPage,
            size: page.pageSize
          },
          params,
          this.searchForm
        )
      ).then(response => {
        this.tableData = response.data.data.records
        this.page.total = response.data.data.total
        this.tableLoading = false
      })
    },
    handlePic(row) {
      this.actPicUrl =
          `/act/process/resource/` +
          row.deploymentId +
          '/' +
          row.processonDefinitionId +
          '/image'
      this.showPicDialog = true
    },
    handleStatus(row, type) {
      var _this = this
      this.$confirm(
        '是否确认操作ID为"' + row.processonDefinitionId + '"的流程?',
        '警告',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(function() {
        return status(row.processonDefinitionId, type)
      }).then(() => {
        this.getList(this.page)
        _this.$message({
          showClose: true,
          message: '操作成功',
          type: 'success'
        })
        this.getList(this.page)
      })
    },
    handleDel(row, index) {
      this.$refs.crud.rowDel(row, index)
    },
    rowDel: function(row) {
      var _this = this
      this.$confirm(
        '是否确认删除ID为"' + row.deploymentId + '"的模型?',
        '警告',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).then(function() {
        return delObj(row.deploymentId)
      }).then(() => {
        this.getList(this.page)
        _this.$message({
          showClose: true,
          message: '删除成功',
          type: 'success'
        })
        this.getList(this.page)
      }).catch(function() {
      })
    },
    /**
       * 搜索回调
       */
    searchChange(form) {
      this.searchForm = form
      this.getList(this.page, form)
    },
    /**
       * 刷新回调
       */
    refreshChange() {
      this.getList(this.page)
    }
  }
}
</script>

<style lang="scss" scoped>
</style>

