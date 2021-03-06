<!-- 角色列表 -->
<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">
        首页
      </el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区 -->
      <el-row>
        <el-button
          type="primary"
          @click="addDialogVisible = true"
        >
          添加角色
        </el-button>
      </el-row>
      <!-- 角色列表区 -->
      <el-table
        :data="rolelist"
        border
        stripe
      >
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '','vcenter']"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(scope.row, item1.id)"
                >
                  {{ item1.authName }}
                </el-tag>
                <i class="el-icon-caret-right" />
              </el-col>
              <!-- 渲染二级和三级权限 -->
              <el-col :span="19">
                <el-row
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                  :class="['bdtop',i2 === 0 ? '' : 'bdtop','vcenter']"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                    >
                      {{ item1.authName }}
                    </el-tag>
                    <i class="el-icon-caret-right" />
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      v-for="(item3) in item2.children"
                      :key="item3.id"
                      type="warning"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                    >
                      {{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index" />
        <el-table-column
          label="角色名称"
          prop="roleName"
        />
        <el-table-column
          label="角色描述"
          prop="roleDesc"
        />
        <el-table-column
          label="操作"
          width="300px"
        >
          <template slot-scope="scope">
            <el-button
              type="primary"
              size="mini"
              icon="el-icon-search"
              @click="showEditDialog(scope.row.id)"
            >
              编辑
            </el-button>
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="removeRolesById(scope.row.id)"
            >
              删除
            </el-button>
            <el-button
              type="warning"
              size="mini"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
            >
              分配权限
            </el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色的对话框 -->
    <el-dialog
      :visible.sync="addDialogVisible"
      title="添加角色"
      width="50%"
      @close="editDialogClose"
    >
      <!-- 内容主题区域 -->
      <el-form
        ref="addRolesRef"
        :model="addRoles"
        :rules="addRolesRules"
        label-width="100px"
      >
        <el-form-item
          label="角色名称"
          prop="roleName"
        >
          <el-input v-model="addRoles.roleName" />
        </el-form-item>
        <el-form-item
          label="角色描述"
          prop="roleDesc"
        >
          <el-input v-model="addRoles.roleDesc" />
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
          @click="addroleName"
        >确 定</el-button>
      </span>
      </el-form>
    </el-dialog>
    <!-- 修改角色的对话框 -->
    <el-dialog
      title="修改角色"
      width="50%"
      :visible.sync="editDialogVisible"
      @close="editDialogClose"
    >
      <el-form
        ref="editRolesRef"
        :model="editRoles"
        :rules="addRolesRules"
        label-width="120px"
      >
        <el-form-item
          label="角色 ID"
        >
          <el-input
            v-model="editRoles.roleId"
            disabled
          />
        </el-form-item>
        <el-form-item
          label="修改角色"
          prop="roleName"
        >
          <el-input
            v-model="editRoles.roleName"
          />
        </el-form-item>
        <el-form-item
          label="修改角色描述"
          prop="roleDesc"
        >
          <el-input
            v-model="editRoles.roleDesc"
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
          @click="editRolesInfo"
        >确 定</el-button>
      </span>
      </el-form>
    </el-dialog>
    <!-- 分配权限的对话框 -->
    <el-dialog
      :visible.sync="setRightDialogVisible"
      title="分配权限"
      width="50%"
    >
      <!-- 树形控件 -->
      <el-tree
        ref="treeRef"
        :data="rightsList"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defKeys"
      />

      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="allotRights"
        >确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    //   验证用户名是否为空
    return {
      // 所有角色列表数据
      rolelist: [],
      // 控制添加角色对话框的显示与隐藏
      addDialogVisible: false,
      //   添加角色的表单数据
      addRoles: {
        roleName: '',
        roleDesc: ''
      },
      editDialogVisible: false,
      editRoles: {},
      addRolesRules: {
        roleName: [
          { required: true, message: '请输入角色名称', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色描述', trigger: 'blur' }
        ]
      },
      //   控制分配权限对话框的
      setRightDialogVisible: false,
      //   所有权限的数据
      rightsList: [],
      //   树形控件的属性绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      //   默认选中ID值的数组
      defKeys: [],
      //   即将分配权限角色的ID
      roleId: []
    }
  },
  created() {
    this.getRolesList()
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败')
      this.rolelist = res.data
    },
    // 监听添加角色对话框的关闭事件
    addDialogClosed() {
      this.$refs.addRolesRef.resetFields()
    },
    // 点击按钮添加新角色
    addroleName() {
      this.$refs.addRolesRef.validate(async valid => {
        if (!valid) return
        // 可以发起添加角色的网络请求
        const { data: res } = await this.$http.post('roles', this.addRoles)
        if (res.meta.status !== 201) this.$message.error('添加角色失败')
        this.$message.success('添加角色成功！')
        // 隐藏添加觉得的对话框
        this.addDialogVisible = false
        // 重新获取角色列表
        this.getRolesList()
      })
    },
    // 展示编辑角色的对话框
    async showEditDialog(id) {
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('查询角色失败!')
      this.editRoles = res.data
      this.editDialogVisible = true
    },
    // 监听修改角色对话框的关闭事件
    editDialogClose() {
      this.$refs.editRolesRef.resetFields()
    },
    editRolesInfo() {
      this.$refs.editRolesInfo.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put('roles/' + this.editRoles.id, { roleName: this.editRoles.roleName, roleDesc: this.editRoles.roleDesc })
        if (res.meta.status !== 200) return this.$message.error('修改角色失败')
        // 关闭对话框
        this.editDialogVisible = false
        // 刷新数据列表
        this.getRolesList()
        this.$message.success('更新角色信息成功！')
      })
    },
    // 根据ID删除对应的用户信息
    async removeRolesById(id) {
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
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除用户失败')
      this.$message.success('删除用户成功')
      this.getRolesList()
    },
    // 根据ID删除对应的权限
    async removeRightById(role, rightId) {
      // 弹框提示用户是否要删除
      const confirmResult = await this.$confirm('此操作将永久删除该权限', '提示', { confirmButtonText: '确定', cancelButtonText: '取消', type: 'warning' }).catch(err => err)
      if (confirmResult !== 'confirm') return this.$message.info('取消了删除')
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) return this.$message.error('删除权限失败！')
      //   this.getRolesList()
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) return this.$message.error('获取权限失败')
      //   获取到的权限数据保存到data中
      this.rightsList = res.data
      //   递归获取三级节点的ID
      this.defKeys = []
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形式，获取角色下的所有三级权限的id，并保存到defKeys数组中
    getLeafKeys(node, arr) {
      // 如果当前节点不包含children属性，则是三级节点
      if (!node.children) return arr.push(node.id)
      node.children.forEach(item => this.getLeafKeys(item, arr))
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) return this.$message.error('更新权限失败！')
      this.$message.success('分配权限成功！')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}

</script>
<style lang="less" scoped>
.el-tag{
    margin: 7px;
}

.bdtop{
    border-top: 1px solid #eee;
}

.bdbottom{
    border-bottom: 1px solid #eee;
}

.vcenter{
    display:flex;
    align-items: center;
}
</style>
