<template>
  <el-container>
    <el-header>
      <div>
        <img src="../assets/52logo.png" alt />
        <span>后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <el-container>
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <!-- 侧边菜单-->
        <el-menu :default-active="active" class="el-menu-vertical-demo" unique-opened :collapse="isCollapse" :collapse-transition="false" router>
          <div>
            <h5 @click="toggleCollapse">| | |</h5>
          </div>
          <!-- 一级菜单 -->
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
            <template slot="title">
              <!-- 图标 -->
              <i :class="item.icon"></i>
              <!-- 菜单标题 -->
              <span>{{item.authName}}</span>
            </template>

            <!-- 二级菜单 -->
            <el-menu-item
              :index="'/' + childrenItem.path"
              v-for="childrenItem in item.children"
              :key="childrenItem.id" @click="saveActive('/' + childrenItem.path)"
            >
              <i class="el-icon-menu"></i>
              <span>{{childrenItem.authName}}</span>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  // 钩子函数
  created () {
    // 创建实例时 调用函数？
    this.gitMenuList()
    this.active = window.sessionStorage.getItem('active')
  },
  data () {
    return {
      menuList: [],
      // 存储图标信息
      iconsArr: [
        'iconfont icon-users',
        'iconfont icon-3702mima',
        'iconfont icon-shangpin',
        'iconfont icon-danju',
        'iconfont icon-tijikongjian'
      ],
      /* 方法二
      iconsObj: {
      } */
      isCollapse: false,
      active: ''
    }
  },
  methods: {
    // 退出登录
    logout () {
      window.sessionStorage.clear()
      this.$router.push('./login')
    },
    // 读取左侧列表
    async gitMenuList () {
      // 结构赋值 + 异步
      const { data } = await this.$http.get('menus')
      // 判断是否获取成功 给出响应提示
      if (data.meta.status !== 200) return this.$message.error('读取列表失败')
      // console.log(data)
      this.menuList = data.data
      // 为menuList添加icon
      for (let index = 0; index < this.menuList.length; index++) {
        this.menuList[index].icon = this.iconsArr[index]
      }
      /* 方法二 整合到新的对象
      for (let index = 0; index < this.menuList.length; index++) {
        const id = this.menuList[index].id
        this.iconsObj[id] = this.iconsArr[index]
      } console.log(this.iconsObj) */
      // console.log(this.menuList)
    },
    // 是否隐藏菜单
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    },
    // 激活列表
    saveActive (isActive) {
      // this.active = isActive
      // 储存path
      window.sessionStorage.setItem('active', isActive)
    }
  }
}
</script>

<style lang="less" scoped>
.el-container {
  height: 100%;
}
.el-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  box-shadow: 0 1px 8px #999;
  z-index: 999;
  div {
    display: flex;
    align-items: center;
    height: 80%;
    img {
      height: 100%;
    }
    span {
      padding-left: 18px;
    }
  }
}
.el-aside {
  border-right: solid 1px #eee;
}
.el-menu {
  border-right: none;
  div {
    border-bottom: solid 1px #eee;
    h5 {
      color: #999;
      text-align: center;
      cursor: pointer;
    }
  }
}
.iconfont {
  margin-right: 10px;
}
</style>
