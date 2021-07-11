<template>
  <el-container class="home-container">
    <!-- 导航区域 -->
    <el-header>
      <div>
        <img src="" alt="">
        <span>电商管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>

    <el-container class="aside-container">
      <!-- 侧边栏区域 -->
      <el-aside :width="isCollapse ? '64px' : '200px'">
        <div class="toggleMenu" @click="toggleMenu">
          |||
        </div>
         <el-menu
          background-color="#333744"
          text-color="#fff"
          active-text-color="#ffd04b"
          unique-opened
          :collapse="isCollapse"
          :collapse-transition="false"
          :router="true"
          :default-active="currentActive">
          <!-- 一级菜单 -->
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
            <template slot="title">
              <i :class="iconsObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item
              :index="'/' + subItem.path" 
              v-for="subItem in item.children" 
              :key="subItem.id" 
              @click="changeActive('/' + subItem.path)">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 内容区域 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  data() {
    return {
      menuList: [],
      iconsObj: {
        '125':'iconfont icon-user',
        '103':'iconfont icon-tijikongjian',
        '101':'iconfont icon-shangpin',
        '102':'iconfont icon-danju',
        '145':'iconfont icon-baobiao'
      },
      isCollapse: false,
      currentActive: ''
    }
  },
  created () {
    this.getMenuList();
    this.currentActive = window.sessionStorage.getItem('activePath')
  },
  methods: {
    // 退出登录，跳转登录页，清空token
    logout() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 通过请求获取侧边栏列表数据
    async getMenuList() {
      const { data : res } = await this.$http.get('menus')
      if(res.meta.status !== 200) return this.$message.error(res.meta.status);
      this.menuList = res.data;
    },
    // 侧边栏收缩展示
    toggleMenu() {
      this.isCollapse = ! this.isCollapse;
    },
    changeActive(activePath) {
      window.sessionStorage.setItem('activePath', activePath);
      this.currentActive = activePath;
    }
  }
}
</script>

<style lang="less" scoped>

.home-container{
  height: 100%;
}
.el-header{
  background-color: #373d41;
  display: flex;
  justify-content: space-between;
  align-items: center;
  div{
    span{
      font-size: 24px;
      color: #fff;
    }
  }
}

// .aside-container, .el-header{
//   position: fixed;
// }

.el-aside{
  background-color: #333744;
  .el-menu{
    border: none;
  }
}

.el-main{
  background-color: #eaedf1;
}
.iconfont{
  margin-right: 10px;
}
.toggleMenu{
  background-color: #4a5064;
  font-size: 10px;
  line-height: 24px;
  color: #fff;
  text-align: center;
  letter-spacing: 0.2em;
  cursor: pointer;
}
</style>
