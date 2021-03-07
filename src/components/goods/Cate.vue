<!-- 商品分类组件 -->
<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">
        首页
      </el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 按钮 -->
      <el-row>
        <el-col>
          <el-button
            type="primary"
            @click="showAddCateDialog"
          >
            添加分类
          </el-button>
        </el-col>
      </el-row>

      <!-- 表格 -->
      <tree-table
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
        class="tree-table"
      >
        <!-- 是否有效 -->
        <template
          slot="isok"
          slot-scope="scope"
        >
          <i
            v-if="scope.row.cat_deleted === false"
            class="el-icon-success"
            style="color: lightgreen;"
          />
          <i
            v-if="scope.row.cat_deleted === true"
            style="color: lightgreen;"
            class="el-icon-error"
          />
        </template>

        <!-- 排序 -->
        <template
          slot="order"
          slot-scope="scope"
        >
          <el-tag
            v-if="scope.row.cat_level === 0"
            size="mini"
          >
            一级
          </el-tag>
          <el-tag
            v-if="scope.row.cat_level === 1"
            size="mini"
            type="success"
          >
            二级
          </el-tag>
          <el-tag
            v-if="scope.row.cat_level === 2"
            size="mini"
            type="warning"
          >
            三级
          </el-tag>
        </template>

        <!-- 操作 -->
        <template
          slot="opt"
          slot-scope=""
        >
          <el-button
            type="primary"
            icon="el-icon-edit"
            size="mini"
          >
            编辑
          </el-button>
          <el-button
            size="mini"
            type="danger"
            icon="el-icon-delete"
          >
            删除
          </el-button>
        </template>
      </tree-table>

      <!-- 分页区域 -->
      <el-pagination
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </el-card>

    <!-- 添加分类的对话框 -->
    <el-dialog
      :visible.sync="addDialogVisible"
      title="添加分类"
      width="50%"
      @close="addCateDialogClosed"
    >
      <!-- 添加分类的表单-->
      <el-form
        ref="addCateFormeRef"
        :model="addCateForm"
        :rules="addCateFormRules"
        label-width="100px"
      >
        <el-form-item
          label="分类名称:"
          prop="cat_name"
        >
          <el-input v-model="addCateForm.cat_name" />
        </el-form-item>
        <el-form-item
          label="父级分类"
        >
          <!-- option指定数据源 -->
          <!-- props用来指定配置对象 -->
          <el-cascader
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            clearable
            @change="parentCateChange"
          />
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="addCate"
        >确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类的数据列表，默认为空
      catelist: [],
      //   总数据条数
      total: 0,
      //   为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        }, {
          label: '是否有效',
          // 表示，当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'isok'
        }, {
          label: '排序',
          // 表示，当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'order'
        }, {
          label: '操作',
          // 表示，当前列定义为模板列
          type: 'template',
          // 表示当前这一列使用模板名称
          template: 'opt'
        }
      ],
      // 控制添加分类对话框的显示与隐藏
      addDialogVisible: false,
      // 添加分类的表单数据对象
      addCateForm: {
        // 将要添加的分类的名称
        cat_name: '',
        // 父级分类的ID
        cat_pid: 0,
        // 分类的等级默认添加1级分类
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          { required: true, message: '请输入分类名称', trigger: 'blur' }
        ]
      },
      // 父级分类的列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        checkStrictly: true
      },
      // 选中的父级分类的id数组
      selectedKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    //   获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取商品分类数据失败！')
      //   把数据列表赋值给catelist
      this.catelist = res.data.result
      //   为总数据条数赋值
      this.total = res.data.total
    },
    // 监听pagesize改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum改变
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    addDialogClosed() {
      this.addDialogVisible = false
    },
    // 点击按钮展示添加分类的对话框
    showAddCateDialog() {
      // 先获取父级分类的数据列表
      this.getParentCateList()
      // 在展示出对话框
      this.addDialogVisible = true
    },
    // 获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) return this.$message.error('获取父级分类数据列表失败')
      this.parentCateList = res.data
    },
    // 选择项发生变化触发这个函数
    parentCateChange() {
      // 反之就说明没有选中任何父级分类
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
        this.addCateForm.cat_level = this.selectedKeys.length
      } else {
        // 父级分类的Id
        this.addCateForm.cat_pid = 0
        // 为当前分类的等级赋值
        this.addCateForm.cat_level = 0
      }
    },
    // 监听对话框的关闭事件，重置表单数据
    addCateDialogClosed() {
      this.$refs.addCateFormeRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    // 点击按钮添加新的分类
    addCate() {
      // console.log(this.addCateForm)
      this.$refs.addCateFormeRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) return this.$message.error('添加分类失败！')
        this.$message.success('添加分类成功！')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    }
  }
}

</script>
<style lang="less" scoped>
.tree-table{
  margin-top: 15px;
}
.el-cascader{
  width: 100%;
}

</style>
