<template>
  <div>
    <!-- 面包屑区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片区域 -->
    <el-card>
      <el-alert title="添加商品信息" type="info" center show-icon :closable="false"></el-alert>
      <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>

      <!-- 表单位置很重要表单为一个表单，el-tab-pane又只能在el-tabs下面 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-position="top">
        <!-- 标签页 -->
        <el-tabs
          :tab-position="'left'"
          v-model="activeIndex"
          :before-leave="tabBeforeLeave"
          @tab-click="tabClicked"
        >
          <el-tab-pane label="基本信息" name="0">
            <!-- 表单域 -->
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="addForm.goods_cat"
                :options="cateList"
                :props="cateProps"
                @change="handleChange"
                popper-class="custom_cascader"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <el-form-item v-for="item in manyTableData" :key="item.attr_id" :label="item.attr_name">
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox border v-for="(cbItem, i) in item.attr_vals" :key="i" :label="cbItem"></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品属性" name="2">
            <el-form-item v-for="item in onlyTableData" :key="item.attr_id" :label="item.attr_name">
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <el-upload
              :action="upAction"
              :headers="upHeaders"
              :on-success="upSuccess"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <quill-editor v-model="addForm.goods_introduce"></quill-editor>
            <el-button size="small" type="primary" @click="addGoods" class="btnAdd">添加商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>

    <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
      <img :src="imgSrc" width="100%">
    </el-dialog>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  data() {
    return {
      // 计步器&标签导航
      activeIndex: '0',
      //  表单数据对象
      addForm: {
        // 不定义会报错 因为下方有判断语句
        goods_cat: [],
        // photo
        pics: [],
        attrs: []
      },
      // 验证规则
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入基本信息', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入基本信息', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入基本信息', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入基本信息', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请选择分类', trigger: 'blur' }
        ]
      },
      // 商品列表（分类）
      cateList: [],
      // 级联选择器配置
      cateProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 返回的参数列表
      manyTableData: [],
      onlyTableData: [],
      // 上传图片地址
      upAction: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 上传文件请求头
      upHeaders: {
        Authorization: window.sessionStorage.getItem('token')
      },
      // 预览框
      previewVisible: false,
      imgSrc: ''
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取分类列表
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.cateList = res.data
      // console.log(this.cateList)
    },
    // 级联选择改变时
    handleChange() {
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
      // console.log(this.addForm)
    },
    // 能否选择下一页
    tabBeforeLeave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.info('请先选择分类')
        return false
      }
    },
    // 点击tab时
    async tabClicked() {
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'many' } })

        if (res.meta.status !== 200) {
          return this.$message.error('获取参数失败')
        }

        res.data.forEach(item => {
          // 如果没有元素就不分割转数组了
          // 因为 [].split(' ')=[''] !!! 会被渲染到页面
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
        // console.log(this.manyTableData)
      } else if (this.activeIndex === '2') {
        // 点击商品属性时
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: 'only' } })
        if (res.meta.status !== 200) {
          return this.$message.error('获取属性失败')
        }
        this.onlyTableData = res.data
      }
    },

    //  上传成功的钩子
    upSuccess(response, file, fileList) {
      // 上传成功后把返回的数据对应请求参数存到addform
      this.addForm.pics.push({ pic: response.data.tmp_path })
      this.fileList.push(file)
    },
    // 点击文件列表中已上传的文件时的钩子
    handlePreview(file) {
      this.previewVisible = true
      // 把图片地址赋值到绑定的值身上
      this.imgSrc = file.response.data.url
    },
    // 文件列表移除文件时的钩子
    handleRemove(file) {
      // 获取要删除的数组元素
      const tmpPath = file.response.data.tmp_path
      // 从addform中找到
      const i = this.addForm.pics.findIndex(item => item.pic === tmpPath)
      // 从addform中删除
      this.addForm.pics.splice(i, 1)
    },
    addGoods() {
      this.$refs.addFormRef.validate(async v => {
        // 预验证
        if (!v) {
          return this.$message.error('请完善表单')
        }

        // 处理参数值
        this.manyTableData.forEach(item => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals.join(' ') }
          this.addForm.attrs.push(newInfo)
        })
        this.onlyTableData.forEach(item => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals }
          this.addForm.attrs.push(newInfo)
        })

        // 深拷贝
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join()

        // 发起添加商品请求
        const { data: res } = await this.$http.post('goods', form)
        if (res.meta.status !== 201) {
          return this.$message.error('添加失败 请检查信息')
        }
        this.$message.success('添加成功')
        this.$router.push('/goods')
      })
    }

  },
  computed: {
    cateId() {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>

<style lang="less" scoped>
.el-steps {
  margin: 15px 0;
}
.el-checkbox {
  margin: 5px 10px 5px 0 !important;
}
.btnAdd {
  margin-top: 15px;
}
</style>
