<template>
  <div id="app">
    <el-container class="home-container">
      <!-- 头部区域 -->
      <el-header>
        <div>
          <img src="../assets/heima.png" alt />
          <span>电商后台管理系统</span>
        </div>
        <el-button type="info" @click="logout">退出登录</el-button>
      </el-header>
      <el-container>
        <!-- 左侧区域 -->
        <el-aside :width="isCollapse ? '64px' : '200px' ">
          <!-- toggle 切换键  collapse 倒塌 折叠 -->
          <div class="toggle-button" @click="toggleCollapse">|||</div>
          <el-menu
            :default-active="activePath"
            class="el-menu-vertical-demo"
            background-color="#545c64"
            text-color="#fff"
            active-text-color="#409EFF"
            unique-opened
            :collapse="isCollapse"
            :collapse-transition="false"
            router
          >
            <!-- 一级菜单 -->
            <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
              <!-- 一级菜单模板区域 -->
              <template slot="title">
                <!-- 图标 -->
                <i :class="iconsObj[item.id]"></i>
                <!-- 文本 -->
                <span>{{item.authName}}</span>
              </template>

              <!-- 二级菜单区域 -->
              <el-menu-item
                :index="'/' + subitem.path + ''"
                v-for="subitem in item.children"
                :key="subitem.id"
                @click="saveNavState('/' + subitem.path + '')"
              >
                <template slot="title">
                  <!-- 图标 -->
                  <i class="el-icon-menu"></i>
                  <!-- 文本 -->
                  <span>{{subitem.authName}}</span>
                </template>
              </el-menu-item>
            </el-submenu>
          </el-menu>
        </el-aside>
        <!-- 右侧区域 -->
        <el-main>
          <!-- 路由占位符 -->
          <router-view></router-view>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 左侧菜单数据
      menulist: [],
      iconsObj: {
        '125': 'iconfont icon-yonhu',
        '103': 'iconfont icon-quanxianguanli',
        '101': 'iconfont icon-shangpin',
        '102': 'iconfont icon-dingdan',
        '145': 'iconfont icon-shuju'
      },
      isCollapse: false,
      activePath: ''
    }
  },
  created() {
    this.getMenuList()
    this.activePath = window.sessionStorage.getItem('activePath')
  },
  methods: {
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 获取所有菜单数据
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      this.menulist = res.data
    },
    // 切换菜单折叠
    toggleCollapse() {
      // console.log(111)
      this.isCollapse = !this.isCollapse
    },
    // 保存链接的激活状态
    saveNavState(activePath) {
      window.sessionStorage.setItem('activePath', activePath)
      this.activePath = activePath
    }
  }
}
</script>

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-left: 5px;
  color: #fff;
  font-size: 20px;
  > div {
    display: flex;
    align-items: center;
    span {
      margin-left: 15px;
    }
  }
}
.el-aside {
  background-color: #333744;
  .el-menu {
    border-right: none;
  }
}
.el-main {
  background-color: #eaedf1;
}
.iconfont {
  margin-right: 10px;
}
.toggle-button {
  background-color: #4a5064;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
