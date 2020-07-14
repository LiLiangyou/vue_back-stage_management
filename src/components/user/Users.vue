<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <div>
        <el-row :gutter="20">
          <el-col :span="10">
            <el-input
              placeholder="请输入内容"
              clearable
              @clear="queryUser"
              @input="queryUser"
              v-model="queryInfo.query"
            >
              <el-button slot="append" icon="el-icon-search" @click="queryUser"></el-button>
            </el-input>
          </el-col>
          <el-col :span="14">
            <el-button type="primary" @click="dialogFormVisible = true">添加用户</el-button>
          </el-col>
        </el-row>
      </div>
      <!-- 列表 -->
      <el-table :data="userList" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="username" label="姓名"></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="mobile" label="电话"></el-table-column>
        <el-table-column prop="mg_state" label="状态">
          <!-- 状态 作用域插槽 -->
          <template slot-scope="scope">
            <!-- {{scope.row}} -->
            <el-switch v-model="scope.row.mg_state" @change="newStatusValue(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="215px">
          <template slot-scope="scope">
            <!-- 修改按钮 -->
            <el-button type="primary" icon="el-icon-edit" @click="openEditForm(scope.row.id)"></el-button>
            <!-- 删除按钮 -->
            <el-button type="danger" icon="el-icon-delete" @click="removeUserById(scope.row.id)"></el-button>
            <!-- 分配角色按钮 -->
            <el-tooltip class="item" :enterable="false" content="分配角色" placement="top">
              <el-button type="warning" icon="el-icon-setting"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>
    <!-- 添加用户对话框 -->
    <el-dialog title="添加用户" :visible.sync="dialogFormVisible" @close="closeAddForm">
      <el-form
        :model="addForm"
        :label-width="formLabelWidth"
        :rules="addFormRules"
        ref="addFormRef"
      >
        <!-- 内容主体 -->
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="电话" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 尾部 -->
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 修改用户对话框 -->
    <el-dialog title="修改用户" :visible.sync="editDialogForm" width="50%" @close="closeEditForm">
      <el-form ref="editFormRef" :model="editForm" label-width="80px" :rules="editFormRules">
        <el-form-item label="用户名">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email" @input="checkEditForm"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="editForm.mobile" @input="checkEditForm"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogForm = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo" :disabled="isD">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      // 查询参数
      queryInfo: {
        query: '',
        // 当前页数
        pagenum: 1,
        // 每页显示多少条
        pagesize: 2
      },
      // 用户看列表
      userList: [],
      // 总条数
      total: 0,
      // 控制添加用户是否显示
      dialogFormVisible: false,
      formLabelWidth: '70px',
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      editFormBack: {
      },
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
          { required: true, message: '请输入邮箱', trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入电话号码', trigger: 'blur' },
          { min: 8, max: 11, message: '长度在 8 到 11 个字符', trigger: 'blur' }
        ]
      },

      // 控制编辑用户是否显示
      editDialogForm: false,
      editForm: {
        username: 'ß',
        email: '',
        mobile: ''
      },
      editFormRules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入电话号码', trigger: 'blur' },
          { min: 8, max: 11, message: '长度在 8 到 11 个字符', trigger: 'blur' }
        ]
      },
      isD: true
    }
  },
  created () {
    // 执行获取用户列表
    this.getUserList()
  },
  methods: {
    // 对象浅拷贝
    copyObj (obj) {
      var newobj = {}
      for (var attr in obj) {
        newobj[attr] = obj[attr]
      }
      return newobj
    },
    // 定义获取用户列表
    async getUserList () {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      if (res.meta.status !== 200) return this.$message.error(res.meta.msg)
      // 赋值
      this.userList = res.data.users
      this.total = res.data.total
      // console.log(res)
    },
    //
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    handleCurrentChange (newCurrent) {
      this.queryInfo.pagenum = newCurrent
      this.getUserList()
    },
    // 修改状态
    async newStatusValue (newRow) {
      const { data: res } = await this.$http.put(`users/${newRow.id}/state/${newRow.mg_state}`)
      // console.log(res)
      if (res.meta.status !== 200) {
        newRow.mg_state = !newRow.mg_state
        return this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
    },
    // 查询用户
    queryUser () {
      // console.log(this.queryInfo.query)
      this.getUserList()
    },
    // 关闭对话框清空内容
    closeAddForm () {
      this.$refs.addFormRef.resetFields()
    },
    // 添加用户
    addUser () {
      this.$refs.addFormRef.validate(async v => {
        // console.log(v) // 1.true // 2.false
        // console.log(!v) // 1.false // 2.true
        if (!v) return // 1.x // 2.√
        const { data: res } = await this.$http.post('users', this.addForm) // 1.√ // 2.x
        if (res.meta.status !== 201) return this.$message.email(res.meta.msg)
        // 状态码 = 201
        this.$message.success(res.meta.msg)
        this.getUserList()
        this.dialogFormVisible = false
      })
    },

    // 关闭重置修改表单的信息
    closeEditForm () {
      this.$refs.editFormRef.resetFields()
    },
    // 查询并渲染到表单
    async openEditForm (id) {
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('获取用户信息失败')
      this.editForm = res.data
      this.editFormBack = this.copyObj(res.data)
      if (this.isD === false) this.isD = true
      this.editDialogForm = true
    },
    // 提交修改
    async editUserInfo () {
      const { data: res } = await this.$http.put('users/' + this.editForm.id, {
        email: this.editForm.email,
        mobile: this.editForm.mobile
      })
      if (res.meta.status !== 200) return this.$message.error('修改用户信息失败')
      this.$message.success('修改用户信息成功')
      this.getUserList()
      this.editDialogForm = false
      // console.log(this.editForm.id)
      // console.log(this.editForm)
    },
    // 输入框改变时
    checkEditForm () {
      const newForm = [this.editForm.email, this.editForm.mobile]
      const oldForm = [this.editFormBack.email, this.editFormBack.mobile]
      if (newForm[0] !== oldForm[0] || newForm[1] !== oldForm[1]) {
        this.isD = false
      } else this.isD = true
      // 预验证
      this.$refs.editFormRef.validate(v => {
        if (!v) {
          this.isD = true
        }
      })
    },
    // 删除用户
    async removeUserById (id) {
      // async-awiat
      const deleteUser = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(e => e)
      if (deleteUser !== 'confirm') return this.$message.info('已取消删除')
      // 发起删除请求
      const { data: res } = await this.$http.delete('users/' + id)
      if (res.meta.status !== 200) return this.$message.error('删除失败')
      this.$message.success('删除成功!')
      this.getUserList()
    }
    // 传统 .then 方式
    /* this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete('users/' + id)
        if (res.meta.status !== 200) return this.$message.error('删除失败')
        this.$message.success('删除成功!')
        this.getUserList()
      }).catch(() => {
        this.$message.info('已取消删除')
      }) */
  }
}
</script>
