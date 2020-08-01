<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <!-- 搜索框 -->
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodsList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodsList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addGoods">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- 表格 -->
      <el-table :data="goodsListData.goods" style="width: 100%" stripe border>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="goods_name" label="商品名称"></el-table-column>
        <el-table-column prop="goods_price" label="商品价格(元)" width="100px"></el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="80px"></el-table-column>
        <el-table-column label="创建时间" width="150px">
          <template slot-scope="scope">{{scope.row.add_time | dataFormat}}</template>
        </el-table-column>
        <el-table-column label="操作" width="120px">
          <template slot-scope="scope">
            <el-button type="primary" size="mini" icon="el-icon-edit"></el-button>
            <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeGoodsById(scope.row.goods_id)"></el-button>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页 -->
      <el-pagination
        @size-change="goodsListSizeChange"
        @current-change="goodsListCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[10, 20, 30, 40]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="goodsListData.total"
      ></el-pagination>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      queryInfo: {
        // 查询参数
        query: '',
        // 当前页码
        pagenum: 1,
        // 每页显示条数
        pagesize: 10
      },
      // 服务器返回的列表数据
      goodsListData: {
        goods: [],
        pagenum: 1,
        total: 0
      },
      searchValue: ''
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    // 获取商品列表
    async getGoodsList() {
      const { data: res } = await this.$http.get('goods', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.console.error(res.meta.msg)
      }
      this.goodsListData = res.data
    },
    // 页显示条数改变时
    goodsListSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    // 页码改变时
    goodsListCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getGoodsList()
    },
    // 查询商品
    queryGoods() {
      // 调用服务器接口方式查询
      this.getGoodsList()
      /* 本地查询思路0
      数组过滤器，要用back的数据
      const newGoods = this.oldGoodsBack.filter(item =>
        字符串匹配
        item.goods_name.includes(this.queryInfo.query.trim()))
      console.log(newGoods)
      this.goodsListData.goods = newGoods
      */
    },
    // 根据id删除商品
    async removeGoodsById(id) {
      const confirmInfo = await this.$confirm(
        '此操作将永久删除该商品, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(e => e)
      if (confirmInfo !== 'confirm') {
        return this.$message({
          type: 'info',
          message: '已取消删除'
        })
      }
      const { data: res } = await this.$http.delete(`goods/${id}`)
      if (res.meta.status !== 200) {
        return this.$message({
          type: 'error',
          message: res.meta.msg
        })
      }
      this.$message({
        type: 'success',
        message: '删除成功!'
      })
      this.getGoodsList()
    },
    addGoods() {
      this.$router.push('/goods/add')
    }
  }
}
</script>
<style>
</style>
