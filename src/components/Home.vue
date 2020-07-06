<template>
  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <img src="../assets/shop-logo.png" alt />
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 页面主题 -->
    <el-container>
      <!--  侧边栏 -->
      <el-aside :width="isCollapse ? '64px':'200px'">
         <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu :router="true" background-color="#304156" text-color="#bfcbd9"  active-text-color="#409EFF"
        unique-opened :collapse="isCollapse" :collapse-transition="false" :default-active="activePath">
          <!-- 一级菜单 -->
          <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
            <!-- 一级菜单模板区 -->
            <template slot="title">
              <i :class="iconList[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id" @click="saveNavState('/'+subItem.path)">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 右侧内容主题 -->
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
      iconList : {
          "125":'iconfont icon-user',
          "103":'iconfont icon-tijikongjian',
          "101":'iconfont icon-shangpin',
          "102":'iconfont icon-danju',
          "145":'iconfont icon-baobiao'
      },
      isCollapse:false,
      activePath : ''
    };
  },
  methods: {
    logout() {
      window.sessionStorage.clear();
      this.$router.push("/");
    },
    async getMenuList() {
      const { data: res } = await this.$http.get("menus");
      if (res.meta.status != 200) return this.$message.error(res.meta.msg)
      this.menuList = res.data
    },
    //菜单的折叠与展开
    toggleCollapse() {
        this.isCollapse = !this.isCollapse;
    },
    saveNavState(activePath) {
        window.sessionStorage.setItem("activePath",activePath)
        this.activePath = activePath
    }
    
  },
  created() {
    this.getMenuList();
    this.activePath = window.sessionStorage.getItem("activePath")
  }
};
</script >

<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  background-color: rgb(30, 45, 60);
  display: flex;
  justify-content: space-between;
  padding-left: 0;
  align-items: center;
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
  background-color: #304156;
  .el-menu {
      border-right: none;
  }
}

.el-main {
  background-color: #eaedf1;
}
img {
  width: 70px;
  height: 70px;
}
.iconfont {
    margin-right: 10px;
}
.toggle-button {
    background:#304156;
    font-size: 10px;
    line-height: 24px;
    color: #fff;
    text-align:center;
    letter-spacing: 0.2em;
    cursor: pointer;
}
</style>