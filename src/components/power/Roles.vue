<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            <!-- 一级 -->
            <el-row
              :class="['rowBottom','vcenter', i1 === 0 ? 'rowTop' : '']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 第一列 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row, item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 第二列 -->
              <!-- 二级 & 三级 -->
              <el-col :span="19">
                <!-- 第二列中的行 -->
                <el-row
                  :class="['vcenter', i2 === 0 ? '' : 'rowTop']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <!-- 第二列行中的第一列 -->
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <!-- 第二列行中的第二列 -->
                  <!-- 三级 -->
                  <el-col :span="18">
                    <el-tag type="warning" closable v-for="(item3) in item2.children" :key="item3.id" @close="removeRightById(scope.row, item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <!-- <pre>
              {{scope.row}}
            </pre> -->
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作" width="290px">
          <template slot-scope>
            <el-button size="mini" type="primary" icon="el-icon-edit">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-search">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data () {
    return {
      rolesList: []
    }
  },
  created () {
    this.getRolesList()
  },
  methods: {
    async getRolesList () {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) return this.$message.error('获取角色列表失败 -- ' + res.meta.msg)
      this.rolesList = res.data
      console.log(this.rolesList)
    },
    async removeRightById (role, rightId) {
      /* this.$confirm('此操作将永久删除, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        console.log(res, role)
        if (res.meta.status !== 200) {
          return this.$message.error('删除失败 -- 错误码:' + res.meta.status + ' --' + res.meta.msg)
        }
        role.children = res.data
        this.$message.success('删除成功!')
      }).catch(() => {
        this.$message.info('已取消删除')
      }) */

      // 弹框确认
      const info = await this.$confirm('此操作将永久删除, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(e => e)
      // 判断用户操作
      if (info !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // 发起删除请求
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      // 判断是否删除成功
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败 -- 错误码:' + res.meta.status + ' --' + res.meta.msg)
      }
      // 防止关闭列表 增加用户体验
      role.children = res.data
      this.$message.success('删除成功!')
    }
  }
}
</script>

<style lang="less" scoped>
/* .el-row {
  padding: 7px;
} */
.el-tag {
  margin: 7px;
}
.rowTop {
  border-top: 1px solid #eee;
}
.rowBottom {
  border-bottom: 1px solid #eee;
}

.vcenter{
  display: flex;
  align-items: center;
}
</style>
