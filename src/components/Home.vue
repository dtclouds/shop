<template>
  <div class="home-container">
    <el-container>
      <el-header class="header">
        <div class="left">
          <img src="../assets/images/avator.jpg" alt />
          <span>后台管理系统</span>
        </div>
        <el-button type="info" @click="exit()">退出</el-button>
      </el-header>
      <el-container class="container">
        <el-aside :width="isCollapse?'64px':'200px'">
          <div class="toggle-button" @click="toggleCollapse">|||</div>
          <el-menu class="menu-left" background-color="#333744" text-color="#fff" active-text-color="#ffd04b" unique-opened :collapse='isCollapse' :collapse-transition='false' router :default-active="this.$route.path">
            <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
              <template slot="title">
                <i :class="iconsObj[item.id]"></i>
                <span>{{item.authName}}</span>
              </template>
              <el-menu-item :index="'/' + subitem.path" v-for="subitem in item.children" :key="subitem.id">
                <i class="el-icon-menu"></i>
                <span>{{ subitem.authName }}</span>
              </el-menu-item>
            </el-submenu>
          </el-menu>
        </el-aside>
        <el-main>
          <router-view></router-view>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>
 
<script>
export default {
  data(){
    return{
      menulist: [],
      iconsObj: {
        "125": "iconfont icon-user",
        "103": "iconfont icon-tijikongjian",
        "101": "iconfont icon-shangpin",
        "102": "iconfont icon-danju",
        "145": "iconfont icon-baobiao",
      },
      isCollapse: false,
    }
  },
  created(){
    this.getMenuList()
    console.log(this.$route.path)
  },
  methods: {
    exit() {
      window.sessionStorage.clear();
      this.$router.push("/login");
    },
    // 获取菜单数据
    async getMenuList(){
      const { data : res } = await this.$http.get('menus')
      if(res.meta.status !== 200) return this.$message.error(res.meat.message)
      this.menulist = res.data
    },
    // 侧边导航收起、展开
    toggleCollapse(){
      this.isCollapse = !this.isCollapse
      console.log(this.isCollapse)
    }
  },
};
</script>
 
<style scoped lang = "less">
.home-container {
  height: 100%;
  .el-container {
    height: 100%;
  }
}
.header {
  display: flex;
  padding: 10px;
  justify-content: space-between;
  align-items: center;
  background: #373d41;
  .left {
    display: flex;
    align-items: center;
    img {
      width: 40px;
      height: 40px;
      margin-right: 15px;
      border-radius: 50%;
    }
    span {
      color: #fff;
      font-size: 22px;
    }
  }
}
.container {
  height: 100%;
}
.el-aside {
  background: #333744;
  .el-menu{
    border-right: 0;
  }
  .toggle-button{
    padding: 10px 0 0;
    color: #fff;
    font-size: 12px;
    text-align: center;
    letter-spacing: 2px;
    cursor: default;
    user-select: none;
  }
  .el-menu--collapse{
    /* width: 64px; */
  }
}
.el-main {
  background: #eaedf1;
}
.menu-left{
  background: #333744;
}
.iconfont{
  margin-right: 10px;
}
</style>