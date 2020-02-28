<template>
  <el-container class="home-container">
    <!--    头部区-->
    <el-header>
      <div>
        <!--        <img src="../assets/heima.png">-->
        <span>电商后台管理系统</span>
      </div>
      <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!--    页面主体区-->
    <el-container>
      <!--      则边栏-->
      <el-aside :width="isCollapse?'64px':'200px'">
        <div class="toggle-button" @click="toggleCollapse">|||</div>

        <el-menu background-color="#333744" text-color="#fff" active-text-color="#409eff" :unique-opened="true"
                 :collapse="isCollapse" :collapse-transition="false" :router="true" :default-active="activePath">
          <!--          一级菜单-->
          <el-submenu :index="item.id+''" v-for="item in menuList" :key="item.id">
            <template slot="title">
              <i :class="iconObj[item.id]"></i>
              <span>{{item.authName}}</span>
            </template>

            <!--            二级菜单-->
            <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id"
                          @click="saveNavState('/'+subItem.path)">
              <template slot="title">
                <i class="el-icon-menu"></i>
                <span>{{subItem.authName}}</span>
              </template>
            </el-menu-item>
          </el-submenu>

        </el-menu>
      </el-aside>
      <!--      中央区域-->
      <el-main>
        <!--        路由占位符-->
        <router-view></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
  export default {
    name: 'Home',
    data () {
      return {
        menuList: [],
        iconObj: {
          125: 'el-icon-user',
          103: 'el-icon-camera-solid',
          102: 'el-icon-s-order',
          145: 'el-icon-s-platform',
          101: 'el-icon-s-release'
        },
        isCollapse: false,
        // 被激活的链接地址
        activePath: '',
      }
    },
    methods: {
      logout () {
        window.sessionStorage.clear()
        this.$router.push('/login')
        this.$message.success('退出成功')
      },
      async getMenuList () {
        // 获取所有的菜单
        const { data: res } = await this.$http.get('menus')

        if (res.meta.status !== 200) {
          return this.$message.error(res.meta.msg())
        }
        this.menuList = res.data

      },
      // 切换菜单的折叠
      toggleCollapse () {
        this.isCollapse = !this.isCollapse
      },
      saveNavState (activePath) {
        window.sessionStorage.setItem('activePath', activePath)
        this.activePath = activePath
      }
    },
    created () {
      this.getMenuList()
      this.activePath = window.sessionStorage.getItem('activePath')
    }
  }

</script>

<style lang="less" scoped>

  .el-header {
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    align-items: center;
    color: #ffffff;
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
      border-right: 0;
    }
  }

  .el-main {
    background-color: #eaedf1;
  }

  .home-container {
    height: 100%;
  }

  /*.iconfont{*/
  /*  margin-right: 20px;*/
  /*}*/

  .toggle-button {
    background-color: #4a5064;
    font-size: 10px;
    line-height: 24px;
    color: #ffffff;
    text-align: center;
    letter-spacing: 0.2em;
    cursor: pointer;
  }


</style>
