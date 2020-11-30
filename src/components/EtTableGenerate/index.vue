<!--/*
 * @Name: EtTableGenerate
 * @Module:
 * @Desc: 好家伙
 * @Author: fan
 * @Date: 2020-11-11 10:09:22
 * @Last Modified by: fan
 * @Last Modified time: 2020-11-11 10:09:22
 * @param {function} [apiMethod]          - table数据用到的api
 * @param {array} [labelList]             - 表头数组
 * @param {array} [propList]              - 对应表头变量名的数组(如：['teacher.name', 'code', 'project.file.name'])
 * @param {boolean} [showOperations]      - 是否展示操作列；默认是false
 * @param {string}} [operationsWidth]     - 自定义操作列的宽度
 * @slot name=search                      - 顶部搜索部分的插槽 参数：form (绑定到form上的对象，父组件使用时需要搭配<el-form-item>并绑定v-model)
    example: <template #search="scope">
               <el-form-item label="姓名">
                 <el-input v-model="scope.form.xm__contains" :placeholder="$t('table.input')" clearable />
              </el-form-item>
            </template>
 * @slot name=operation                   - 操作列表格的插槽 参数：row(用法同el-table-column的插槽)
 *
 * @example:
      <et-table-generate
        ref="table"
        :api-method="getFileCateInfoList"
        :label-list="labelList"
        :prop-list="propList"
        show-operations
        operations-width="300"
      ></et-table-generate>
*/-->

<template>
  <div>
    <el-form
      v-if="!disableSearch"
      :inline="true"
      label-suffix=":"
      :model="formInline"
      class="demo-form-inline"
    >
      <!-- 页眉的搜索部分 -->
      <slot
        name="search"
        :form="formInline"
      />
      <el-form-item v-if="!disableSearchBtn" class="searchBtn">
        <el-button
          class="Btn"
          type="primary"
          size="medium"
          @click="handleFilter"
        >{{ $t('button.search') }}</el-button>
      </el-form-item>
      <!-- <el-form-item class="searchBtn">
        <el-button
          style="margin-bottom: 20px"
          type="primary"
          @click="addBtnClick"
        >{{ $t('button.edit') }}</el-button>
      </el-form-item> -->
    </el-form>
    <!-- 表格 -->
    <el-table
      v-loading="listLoading"
      :data="list"
      border
      fit
      highlight-current-row
      stripe
    >
      <el-table-column
        v-for="(item, index) in propList"
        :key="index"
        :label="labelList[index]"
        :prop="typeof item === 'object' ? item.name : item"
        :formatter="typeof propList[index] === 'object' ? propList[index].format : null"
      />
      <slot name="content" />
      <el-table-column
        v-if="showOperations"
        :label="$t('common.operations')"
        :width="operationsWidth"
      >
        <template #default="scope">
          <slot
            name="operation"
            :row="scope.row"
          />
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <el-pagination
      background
      :page-sizes="listParameters.pageSizes"
      layout="total, sizes, prev, pager, next, jumper"
      :total="listParameters.total"
      class="pagination-container"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
  </div>
</template>

<script>
export default {
  props: {
    // 列表api方法
    apiMethod: {
      type: Function,
      default: null
    },
    // 每次调用api所需的额外参数
    apiObject: {
      type: Object,
      default() {
        return {}
      }
    },
    // 表头列表
    labelList: {
      type: Array,
      default() {
        return []
      }
    },
    // props列表
    propList: {
      type: Array,
      default() {
        return []
      }
    },
    // 展示操作列
    showOperations: {
      type: Boolean,
      default: false
    },
    // 操作列宽度
    operationsWidth: {
      type: String,
      default: ''
    },
    // 禁用搜索框
    disableSearch: {
      type: Boolean,
      default: false
    },
    // 禁用搜索按钮
    disableSearchBtn: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      // table
      apiQueryData: {},
      listLoading: false,
      formInline: {},
      list: [],
      listParameters: {
        total: 0, // 数据总数,默认为0
        pageSizes: [10, 20, 50, 100, 200], // 翻页组件中的每页显示的数量
        page: 1,
        limit: 10
      }
    }
  },
  computed: {
  },
  created() {
    this.init()
  },
  methods: {
    init() {
      this.getListData()
    },
    handleFilter() {
      this.apiQueryData = Object.assign({
        limit: this.listParameters.limit,
        offset: this.listParameters.limit * (this.listParameters.page - 1)
      }, this.formInline)
      this.getListData()
    },
    // 翻页事件
    handleSizeChange(val) {
      this.listParameters.limit = val
      this.apiQueryData.limit = this.listParameters.limit
      this.getListData()
    },
    handleCurrentChange(val) {
      this.listParameters.page = val
      this.apiQueryData.offset = this.listParameters.limit * (this.listParameters.page - 1)
      this.getListData()
    },
    getListData() {
      this.listLoading = true
      const submitData = Object.assign({}, this.apiQueryData, this.apiObject)
      // 调用接口：
      this.apiMethod(submitData).then(response => {
        const result = response.data
        this.list = result
        this.listParameters.total = response.count
        this.listLoading = false
        // 删除后，当前页面没有数据了，则返回上一页
        if (this.list.length <= 0 && this.listParameters.total > 0) {
          this.apiQueryData.offset = this.apiQueryData.offset - this.listParameters.limit
          this.getListData()
        }
      }).catch(() => {
        this.listLoading = false
      })
    }
  }
}
</script>

<style lang="scss" scoped>
</style>
