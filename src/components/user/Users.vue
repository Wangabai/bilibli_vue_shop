<!-- User用户列表组件 -->
<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">
        首页
      </el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            v-model="queryInfo.query"
            placeholder="请输入内容"
            clearable
            @clear="getUserList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUserList"
            />
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button
            type="primary"
            @click="addDialogVisible = true"
          >
            添加用户
          </el-button>
        </el-col>
      </el-row>

      <!-- 用户列表区域 -->
      <el-table
        :data="userlist"
        border
        stripe
      >
        <el-table-column
          type="index"
          label="#"
        />
        <el-table-column
          label="姓名"
          prop="username"
        />
        <el-table-column
          label="邮箱"
          prop="email"
        />
        <el-table-column
          label="电话"
          prop="mobile"
        />
        <el-table-column
          label="角色"
          prop="role_name"
        />
        <el-table-column
          label="状态"
        >
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              @change="userStateChanged(scope.row)"
            />
          </template>
        </el-table-column>
        <el-table-column
          label="操作"
          width="180px"
        >
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(scope.row.id)"
            />
            <!-- 删除按钮 -->
            <el-button
              type="danger"
              size="mini"
              icon="el-icon-delete"
              @click="removeUserById(scope.row.id)"
            />
            <!-- 分配角色按钮 -->
            <el-tooltip
              effect="dark"
              content="分配角色"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="warning"
                size="mini"
                icon="el-icon-setting"
                @click="setRole(scope.row)"
              />
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
      />
    </el-card>

    <!-- 添加用户的对话框 -->
    <el-dialog
      :visible.sync="addDialogVisible"
      title="添加用户"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 内容主题区域 -->
      <el-form
        ref="addFormRef"
        :model="addForm"
        :rules="addFormRules"
        label-width="70px"
      >
        <el-form-item
          label="用户名"
          prop="username"
        >
          <el-input v-model="addForm.username" />
        </el-form-item>
        <el-form-item
          label="密码"
          prop="password"
        >
          <el-input v-model="addForm.password" />
        </el-form-item>
        <el-form-item
          label="邮箱"
          prop="email"
        >
          <el-input v-model="addForm.email" />
        </el-form-item>
        <el-form-item
          label="手机"
          prop="mobile"
        >
          <el-input v-model="addForm.mobile" />
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
          @click="addUser"
        >确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户的对话框 -->
    <el-dialog
      title="修改用户"
      width="50%"
      :visible.sync="editDialogVisible"
      @close="editDialogClose"
    >
      <el-form
        ref="editFormRef"
        :model="editForm"
        :rules="editFormRules"
        label-width="70px"
      >
        <el-form-item
          label="用户名"
        >
          <el-input
            v-model="editForm.username"
            disabled
          />
        </el-form-item>
        <el-form-item
          label="邮箱"
          prop="email"
        >
          <el-input
            v-model="editForm.email"
          />
        </el-form-item>
        <el-form-item
          label="手机"
          prop="mobile"
        >
          <el-input
            v-model="editForm.mobile"
          />
        </el-form-item>
      </el-form>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="editUserInfo"
        >确 定</el-button>
      </span>
      </el-form>
    </el-dialog>

    <!-- 分配角色的对话框 -->
    <el-dialog
      :visible.sync="setRoleDialogVisible"
      title="分配角色"
      width="50%"
    >
      <div>
        <p>当前的用户：{{ userInfo.username }}</p>
        <p>当前的角色：{{ userInfo.role_name }}</p>
        <p>
          分配新角色:
          <el-select
            v-model="selectedRoleId"
            placeholder="请选择"
          >
            <el-option
              v-for="item in rolesList"
              :key="item.id"
              :label="item.roleName"
              :value="item.id"
            />
          </el-select>
        </p>
      </div>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="setRoleDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="saveRoleInfo"
        >确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    // 验证邮箱的规则
    // eslint-disable-next-line no-unused-vars
    var checkEmail = (rule, value, cb) => {
      // 验证邮箱的正则表达式
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) return cb()
      cb(new Error('请输入合法的邮箱'))
    }

    // 验证手机的规则
    // eslint-disable-next-line no-unused-vars
    var checkMobile = (rule, value, cb) => {
      // 验证手机号的正则表达式
      const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (regMobile.test(value)) return cb()
      cb(new Error('请输入正确的手机号'))
    }

    return {
      // 获取用户列表的参数对象
      queryInfo: {
        query: '',
        // 当前页数
        pagenum: 1,
        // 当前每页显示多少条数据
        pagesize: 2
      },
      userlist: [],
      total: 0,
      // 控制添加用户对话框的显示与隐藏
      addDialogVisible: false,
      // 添加用户的表单数据
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 添加表单的验证规则对象
      addFormRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      editDialogVisible: false,
      // 查询到的用户信息对象
      editForm: {},
      // 修改表单的验证规则对象
      editFormRules: {
        email: [
          { required: true, message: '请输入用户邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 控制分配角色对话框的显示与隐藏
      setRoleDialogVisible: false,
      // 需要被分配角色的用户信息
      userInfo: {},
      // 所有角色的数据列表
      rolesList: [],
      // 已选中的角色ID值
      selectedRoleId: []
    }
  },
  created() {
    this.getUserList()
  },
  methods: {
    async getUserList() {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error('获取用户列表失败！')
      this.userlist = res.data.users
      this.total = res.data.total
      console.log(res)
    },
    // 监听pagesize 改变的事件
    handleSizeChange(newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听页码值改变的事件
    handleCurrentChange(newPage) {
      // console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 监听switch开关状态的改变
    async userStateChanged(userinfo) {
      // console.log(userinfo)
      const { data: res } = await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`)
      if (res.meta.status !== 200) {
        userinfo.meta.status = !userinfo.meta.status
        return this.$message.error('更新用户状态失败!')
      }
      this.$message.success('更新用户状态成功！')
    },
    // 监听添加用户对话框的关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 点击按钮添加新用户
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        // 可以发起添加用户的网络请求
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) this.$message.error('添加用户失败！')
        this.$message.success('添加用户成功！')
        // 隐藏添加用户的对话框
        this.addDialogVisible = false
        // 重新获取用户列表
        this.getUserList()
      })
    },
    // 展示编辑用户的对话框
    async showEditDialog(id) {
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('查询用户信息失败！')
      this.editForm = res.data
      this.editDialogVisible = true
    },
    // 监听修改用户对话框的关闭事件
    editDialogClose() {
      this.$refs.editFormRef.resetFields()
    },
    // 编辑用户添加
    editUserInfo() {
      this.$refs.editFormRef.validate(async valid => {
        // eslint-disable-next-line no-useless-return
        if (!valid) return
        // 发起修改用户信息的数据请求
        const { data: res } = await this.$http.put('users/' + this.editForm.id, { email: this.editForm.email, mobile: this.editForm.mobile })
        if (res.meta.status !== 200) return this.$message.error('更新用户数据失败！')
        // 关闭对话框
        this.editDialogVisible = false
        // 刷新数据列表
        this.getUserList()
        // 提示修改成功
        this.$message.success('更新用户信息成功！')
      })
    },
    // 根据ID删除对应的用户信息
    async removeUserById(id) {
      // 询问用户是否删除数据
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      // 如果用户确认删除，则返回值为字符串 confirm
      // 如果用户取消删除，则返回值为字符串 cancel
      // console.log(confirmResult)
      if (confirmResult !== 'confirm') return this.$message.info('已取消删除')
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除用户失败')
      this.$message.success('删除用户成功')
      this.getUserList()
    },
    // 展示分配角色的对话框
    async setRole(userInfo) {
      this.userInfo = userInfo
      // 在展示对话框之前，获取所有角色列表
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败！')
      this.rolesList = res.data
      this.setRoleDialogVisible = true
    },
    // 点击按钮分配角色
    async saveRoleInfo() {
      if (!this.selectedRoleId) return this.$message.error('请选择要分配的角色！')
      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, { rid: this.selectedRoleId })
      if (res.meta.status !== 200) return this.$message.error('更新角色失败！')
      this.$message.success('更新角色成功')
      this.getUserList()
      this.selectedRoleId = ''
      this.userInfo =
      this.setRoleDialogVisible = false
    }
  }
}

</script>
<style lang="less" scoped>
</style>
