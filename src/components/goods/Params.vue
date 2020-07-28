<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片区域 -->
    <el-card>
      <el-alert title="注意：只允许为第三级分类设置相关参数" type="warning" show-icon :closable="false"></el-alert>
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 商品分类选择框 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="cateList"
            :props="cateProps"
            @change="handleChange"
            popper-class="custom_cascader"
          ></el-cascader>
        </el-col>
      </el-row>

      <!-- tab页签区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <!-- 动态参数表格 -->
        <el-tab-pane label="动态参数" name="many">
          <!-- 按钮 -->
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="addDialogVisible = true"
          >添加参数</el-button>
          <!-- 表格 -->
          <el-table :data="manyTableData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClose(scope.row, i)"
                >{{item}}</el-tag>
                <!-- 添加标签区域 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称"></el-table-column>
            <el-table-column label="操作" width="290px">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row)"
                >编辑</el-button>
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row)"
                >删除</el-button>
                <el-button
                  size="mini"
                  type="warning"
                  icon="el-icon-setting"
                  @click="showSetRightDialog(scope.row)"
                >分配权限</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 静态属性表格 -->
        <el-tab-pane label="静态属性" name="only">
          <!-- 按钮 -->
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="addDialogVisible = true"
          >添加属性</el-button>
          <!-- 表格 -->
          <el-table :data="onlyTableData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  v-for="(item, i) in scope.row.attr_vals"
                  :key="i"
                  closable
                  @close="handleClose(scope.row, i)"
                >{{item}}</el-tag>
                <!-- 添加标签区域 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                ></el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称"></el-table-column>
            <el-table-column label="操作" width="290px">
              <template slot-scope="scope">
                <el-button
                  size="mini"
                  type="primary"
                  icon="el-icon-edit"
                  @click="showEditDialog(scope.row)"
                >编辑</el-button>
                <el-button
                  size="mini"
                  type="danger"
                  icon="el-icon-delete"
                  @click="deleteParams(scope.row)"
                >删除</el-button>
                <el-button
                  size="mini"
                  type="warning"
                  icon="el-icon-setting"
                  @click="showSetRightDialog(scope.row)"
                >分配权限</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>

    <!-- addDialog -->
    <el-dialog
      :title="'添加'+titleText"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClose"
    >
      <!-- 表单区域 -->
      <el-form
        :model="addForm"
        :rules="addFormAndEditFormRules"
        ref="addFormRef"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>

    <!-- editDialog -->
    <el-dialog
      :title="'修改'+titleText"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClose"
    >
      <!-- 表单区域 -->
      <el-form
        :model="editForm"
        :rules="addFormAndEditFormRules"
        ref="editFormRef"
        label-width="100px"
      >
        <el-form-item :label="titleText" prop="attr_name">
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data: function () {
    return {
      // 选项列表数据
      cateList: [],
      // 选中的值
      selectedCateKeys: [],
      cateProps: {
        expandTrigger: 'hover',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      // 选中的标签页
      activeName: 'many',
      // 表单数据
      manyTableData: [],
      onlyTableData: [],
      // addDialog
      addDialogVisible: false,
      addForm: {
        attr_name: ''
      },
      addFormAndEditFormRules: {
        attr_name: [{ required: true, message: '不能为空', trigger: 'blur' }]
      },
      // editDialog
      editDialogVisible: false,
      editForm: {
        attr_name: '',
        attr_id: ''
      }

    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.cateList = res.data
    },

    // 选项发生变化时
    handleChange() {
      //  选中非三级分类不发起请求
      if (this.selectedCateKeys.length !== 3) {
        // 清空选中的值
        this.selectedCateKeys = []
        // 清空表格列表
        this.manyTableData = []
        this.onlyTableData = []
        return
      }
      // 选中的是三级分类
      this.getParamsData()
    },

    // tab 标签发生变化时触发
    handleTabClick(tab, event) {
      // console.log(this.activeName)
      if (this.cateId === null) {
        return
      }
      this.getParamsData()
    },

    // 发送获取参数请求
    async getParamsData() {
      // 获取参数列表
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
        params: { sel: this.activeName }
      })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 文本框输入的值
        item.inputValue = ''
        // 控制文本框的布尔值
        item.inputVisible = false
      })
      // 判断动态还是静态
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
      // console.log(this.manyTableData, '动态参数列表', this.onlyTableData, '静态属性列表')
    },

    // show input
    showInput(row) {
      row.inputVisible = true
      // 页面元素重新渲染只后才执行回调函数
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // newTag失去焦点后的操作
    async handleInputConfirm(row) {
      // console.log(row, 'newTag失去焦点后的操作')
      if (row.inputValue.trim().length === 0) {
        // 防止多空格&把空数据添加到列表
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      // push 到本地数据中
      row.attr_vals.push(row.inputValue.trim())
      // 清空标签中的缓存
      row.inputValue = ''
      // 添加标签请求
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: this.activeName,
        attr_vals: row.attr_vals.join(' ')
      })
      // console.log(row.attr_vals.join(' '))
      if (res.meta.status !== 200) {
        // 发生错误立刻清除本地数据刚添加的最后一项
        row.attr_vals.pop()
        return this.$message.error(res.meta.msg)
      }
      this.$message.success(res.meta.msg)
      row.inputVisible = false
    },

    // 重置add表单
    addDialogClose() {
      this.$refs.addFormRef.resetFields()
    },

    // 重置edit表单
    editDialogClose() {
      this.$refs.editFormRef.resetFields()
    },

    // 预验证&发起添加请求
    addParams() {
      this.$refs.addFormRef.validate(async v => {
        if (!v) {
          return
        }
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName
        })
        // console.log(res)
        if (res.meta.status !== 201) {
          return this.$message.error(res.meta.status + res.meta.msg)
        }
        this.getParamsData()
        this.$message.success('添加成功')
        // 等待添加成功后再关闭
        this.addDialogVisible = false
      })
    },

    // 预验证&发起修改请求
    editParams() {
      this.$refs.editFormRef.validate(async v => {
        if (!v) {
          return
        }
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
          attr_name: this.editForm.attr_name,
          attr_sel: this.activeName
        })
        if (res.meta.status !== 200) {
          return this.$message.error('编辑失败 - ' + res.meta.msg)
        }
        this.$message.success('编辑成功')
        this.getParamsData()
        // 等待编辑成功后再关闭
        this.editDialogVisible = false
      })
    },

    // 显示编辑项
    async showEditDialog(row) {
      // 获取当前行并赋值
      // 本地获取的方式不能整个对象赋值 会干扰
      this.editForm.attr_id = row.attr_id
      this.editForm.attr_name = row.attr_name
      // 方法二 调用API
      /*  const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${row.attr_id}`, {
          params: {
            attr_sel: this.activeName
          }
        }
      )
      this.editForm = res.data */
      this.editDialogVisible = true
    },

    //
    async removeParams(row) {
      // 提示是否删除
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(e => e)
      // 用户取消了删除
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      // 删除的逻辑
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${row.attr_id}`)
      // console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('删除失败, 后台返回 - ' + res.meta.msg)
      }
      this.$message.success(res.meta.msg)
      this.getParamsData()
    },

    //
    async handleClose(row, i) {
      // 浅拷贝（缓存删除操作前的数组）
      const cache = []
      row.attr_vals.forEach(item => {
        cache.push(item)
      })
      // 模仿 row.attr_vals.splice(i, 1)
      // 不会对本地数组造成影响 防止失败引起动画
      cache.splice(i, 1)
      // 移除（编辑）标签请求
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`, {
        attr_name: row.attr_name,
        attr_sel: this.activeName,
        attr_vals: cache.join(' ')
      })
      if (res.meta.status !== 200) {
        // 发生错误不会对本地数组造成影响
        return this.$message.error(res.meta.msg)
      }
      // 成功再修改本地数组
      row.attr_vals.splice(i, 1)
      this.$message.success(res.meta.msg)
    }

    // 添加参数的请求
    // async addParamsRequest() {}

  },
  computed: {
    // 计算是否禁用按钮
    isBtnDisabled() {
      if (this.selectedCateKeys.length !== 3) {
        return true
      }
      return false
    },
    // 分类ID（最后一个）
    cateId() {
      if (this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    },
    // 动态获取标题
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
.el-tag {
  margin-right: 15px;
}
.button-new-tag {
  margin-left: 10px;
  height: 32px;
  line-height: 30px;
  padding-top: 0;
  padding-bottom: 0;
}
.input-new-tag {
  width: auto;
  margin-left: 10px;
  vertical-align: bottom;
}
</style>
