<template>
  <el-container id="home">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <img src="../assets/logo.png" alt="" />
        <span>后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 页面主体 -->
    <el-container>
      <!-- 侧边栏 -->
      <el-aside :width="isShow ? '64px' : '200px'">
        <div class="toggle-button" @click="toggleClick">|||</div>
        <el-menu
          unique-opened
          class="el-menu-vertical-demo"
          background-color="#333744"
          text-color="#fff"
          active-text-color="#409bff"
          :collapse="isShow"
          :collapse-transition="false"
          router
          :default-active="$route.path"
        >
          <!-- 一级菜单 -->
          <el-submenu
            :index="item.id + ''"
            v-for="item in menulist"
            :key="item.id"
          >
            <!-- 一级菜单模板 -->
            <template slot="title">
              <i :class="iconList[item.id]" class="icon-font"></i>
              <span>{{ item.authName }}</span>
            </template>
            <!-- 二级菜单 -->
            <el-menu-item
              :index="'/' + submenu.path"
              v-for="submenu in item.children"
              :key="submenu.id"
            >
              <!-- 二级菜单模板 -->
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{ submenu.authName }}</span>
              </template>
            </el-menu-item>
          </el-submenu>
        </el-menu>
      </el-aside>
      <!-- 内容主体 -->
      <el-main>
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  name: "Home",
  created() {
    this.getMenuList();
  },
  data() {
    return {
      menulist: [],
      iconList: {
        125: "iconfont icon-users",
        103: "iconfont icon-tijikongjian",
        101: "iconfont icon-shangpin",
        102: "iconfont icon-danju",
        145: "iconfont icon-baobiao",
      },
      isShow: false,
    };
  },
  methods: {
    logout() {
      sessionStorage.removeItem("token");
      this.$router.push("/login");
    },
    async getMenuList() {
      const { data: res } = await this.$http.get("menus");
      if (res.meta.status != 200) return this.$message.error(res.meta.msg);
      this.menulist = res.data;
    },
    toggleClick() {
      this.isShow = !this.isShow;
    },
  },
};
</script>

<style scoped lang="less">
#home {
  height: 100%;
}
.el-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-left: 0;
  background-color: #373d41;
  color: #fff;
  font-size: 20px;
  div {
    display: flex;
    align-items: center;
    span {
      margin-left: 15px;
    }
    img {
      width: 61px;
      height: 56px;
    }
  }
}
.el-aside {
  background-color: #333744;
  .toggle-button {
    background-color: #4a5065;
    color: #fff;
    text-align: center;
    line-height: 24px;
    font-size: 10px;
    letter-spacing: 0.2em;
    cursor: pointer;
  }
  .el-menu {
    border-right: 0;
  }
}
.el-main {
  background-color: #eaedf1;
}
.icon-font {
  margin-right: 10px;
}
</style>
