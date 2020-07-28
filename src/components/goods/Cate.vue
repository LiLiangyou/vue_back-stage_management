<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加角色</el-button>
        </el-col>
      </el-row>

      <!-- 表格 -->
      <tree-table
        :data="cateList"
        :columns="columns"
        border
        :show-row-hover="false"
        :expand-type="false"
        :selection-type="false"
        show-index
        index-text="#"
        class="treetable"
      >
        <!-- 是否有效 -->
        <template slot="isOk" slot-scope="scope">
          <i
            class="el-icon-success"
            style="color: lightgreen"
            v-if="scope.row.cat_deleted === false"
          ></i>
          <i class="el-icon-error" style="color: red" v-else></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt">
          <el-button type="primary" icon="el-icon-edit" size="mini">搜索</el-button>
          <el-button type="danger" icon="el-icon-delete" size="mini">搜索</el-button>
        </template>
      </tree-table>

      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      ></el-pagination>
    </el-card>

    <!-- 添加角色dialog -->
    <el-dialog
      title="添加角色"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClose"
    >
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <!-- Cascader 级联选择器 -->
        <el-form-item label="父级分类">
          <el-cascader
            v-model="selectedKeys"
            :options="options"
            :props="cascaderProps"
            @change="handleChange"
            clearable
            popper-class="custom_cascader"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 查询条件
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 商品分类列表
      cateList: [],
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        }, {
          label: '是否有效',
          // 表示将当前列定义为模板列
          type: 'template',
          // 当前模板名称
          template: 'isOk'
        }, {
          label: '排序',
          // 表示将当前列定义为模板列
          type: 'template',
          // 当前模板名称
          template: 'order'
        }, {
          label: '操作',
          align: 'center',
          // 表示将当前列定义为模板列
          type: 'template',
          // 当前模板名称
          template: 'opt'
        }
      ],
      addCateDialogVisible: false,
      // 添加分类表单数据对象(请求参数)
      addCateForm: {
        cat_name: '',
        // 不能为空，如果要添加1级分类，则父分类Id应该设置为  `0`
        cat_pid: 0,
        // 不能为空，`0`表示一级分类；`1`表示二级分类；`2`表示三级分类
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [{ required: true, message: '请输入分类名称', trigger: 'blur' }]
      },
      // 选项列表
      options: [],
      // 选中的值
      selectedKeys: [],
      cascaderProps: {
        expandTrigger: 'hover',
        checkStrictly: 'true',
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取分类列表
    async getCateList() {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      // this.$message.success(res.meta.msg)
      this.cateList = res.data.result
      this.total = res.data.total
      // console.log(res)
    },
    // 监听pagesize改变
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pagenum改变
    handleCurrentChange(newNum) {
      this.queryInfo.pagenum = newNum
      this.getCateList()
    },
    // 展示添加角色对话框
    showAddCateDialog() {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // dialog 获取父级分类的数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类列表失败')
      }
      this.options = res.data
      // console.log(res.data, 2)
    },
    // 选项改变时触发(配置请求参数)
    handleChange() {
      // console.log(this.selectedKeys)
      // 发送请求前判断
      if (this.selectedKeys.length > 0) {
        // 如果有选父级分类
        // 父级id 等于最后一项
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
      } else {
        // 没选父级分类 自己是一级分类
        // 父级id 等于 0
        this.addCateForm.cat_pid = 0
      }
      // 自己所在的级别
      this.addCateForm.cat_level = this.selectedKeys.length
    },
    // 关闭对话框时的重置操作
    addCateDialogClose() {
      // 重置表单
      this.$refs.addCateFormRef.resetFields()
      // 重置级联显示的名称
      this.selectedKeys = []
      // 如果在点击按钮时再配置请求参数 则下两项不用重置
      // 因为点击按钮时配置参数时下两项在发送请求前会被刷新
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    // 点击确定按钮时添加新类
    addCate() {
      // 预验证
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        // 发送添加新类请求
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加角色失败')
        }
        this.$message.success('添加角色成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    }
  }
}
</script>

<style lang="less" scoped>
.treetable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>>
