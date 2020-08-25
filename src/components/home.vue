<template>
  <div class="layout">
    <Layout style="height: 100%;">
      <Sider ref="side1" hide-trigger collapsible :collapsed-width="0" v-model="isCollapsed">
        <Menu @on-select="menuItemOnSelect" :class="menuitemClasses" :active-name="$route.name" theme="dark" width="auto" accordion>
          <MenuItem name="index">
            <Icon type="ios-home" />
            <span>首页</span>
          </MenuItem>
          <Submenu name="user">
            <template slot="title">
              <Icon type="ios-contact" />
              <span>账号</span>
            </template>
            <MenuItem name="homeUserInfoEdit"><span>资料</span></MenuItem>
            <MenuItem name="homeUserSafeEdit"><span>安全</span></MenuItem>
          </Submenu>
          <Submenu name="blog">
            <template slot="title">
              <Icon type="ios-paper" />
              <span>博客</span>
            </template>
            <MenuItem name="homeBlogEditHtmlInsert"><span>新建</span></MenuItem>
            <MenuItem name="homeBlogListState0"><span>草稿箱</span></MenuItem>
            <MenuItem name="homeBlogListState1"><span>已发表</span></MenuItem>
            <MenuItem name="homeBlogListState2"><span>回收站</span></MenuItem>
          </Submenu>
          <Submenu name="adminCategory" v-if="existRoleName('admin')">
            <template slot="title">
              <Icon type="ios-people" />
              <span>板块管理</span>
            </template>
            <MenuItem name="homeAdminCategoryList"><span>板块</span></MenuItem>
            <MenuItem name="homeAdminUserCategoryList"><span>板主</span></MenuItem>
          </Submenu>
          <Submenu name="plate">
            <template slot="title">
              <Icon type="ios-paper" />
              <span>板块</span>
            </template>
            <MenuItem name="homePlateEditInf"><span>板块信息</span></MenuItem>
            <MenuItem name="homePlateEditPost"><span>板块帖子</span></MenuItem>
          </Submenu>
          <Submenu name="adminUser" v-if="existRoleName('admin')">
            <template slot="title">
              <Icon type="ios-people" />
              <span>用户管理</span>
            </template>
            <MenuItem name="homeAdminUserList"><span>用户</span></MenuItem>
          </Submenu>

        </Menu>
      </Sider>
      <Layout style="height: 100%; background: #f2f2f2;">
        <Header class="layout-header-bar">
          <Icon @click.native="collapsedSider" :class="rotateIcon" style="margin: 0px 20px;" type="md-menu" size="24"></Icon>
          <Button @click="logout">注销{{user==null?'':user.nickname}}</Button>
          {{$route.name}}
        </Header>
        <Content style="padding: 20px;">
          <template>
            <keep-alive>
              <router-view v-if="$route.meta.keepAlive"></router-view>
            </keep-alive>
            <router-view v-if="!$route.meta.keepAlive"></router-view>
          </template>
        </Content>
      </Layout>
    </Layout>
  </div>
</template>
<script>
export default {
  data () {
    return {
      isCollapsed: false,
      user: null,
      roleList: []
    }
  },
  mounted () {
    this.$get('/user/info')
      .then(data => {
        if (data.data.code === 0) { // 成功
          this.user = data.data.data.user
        } else { // 失败
          this.$Message.error(data.data.message) // 提示
        }
      })
    this.$get('/user/roleList')
      .then(data => {
        if (data.data.code === 0) { // 成功
          this.roleList = data.data.data.roleList
        } else { // 失败
          this.$Message.error(data.data.message) // 提示
        }
      })
  },
  computed: {
    rotateIcon () {
      return [
        'menu-icon',
        this.isCollapsed ? 'rotate-icon' : ''
      ]
    },
    menuitemClasses () {
      return [
        'menu-item',
        this.isCollapsed ? 'collapsed-menu' : ''
      ]
    }
  },
  methods: {
    collapsedSider () {
      this.$refs.side1.toggleCollapse()
    },
    logout () {
      localStorage.removeItem('token') // 注销并删除token
      this.$router.push('/login')
    },
    menuItemOnSelect (MenuItemName) {
      switch (MenuItemName) {
        case 'index': {
          this.$router.push({path: '/'})
          break
        }
        case 'homeUserInfoEdit': {
          this.$router.push({path: '/home/user/info/edit'})
          break
        }
        case 'homeUserSafeEdit': {
          this.$router.push({path: '/home/user/safe/edit'})
          break
        }
        case 'homeBlogEditHtmlInsert': {
          this.$router.push({path: '/home/blog/edit/html'})
          break
        }
        case 'homeBlogListState0': {
          this.$router.push({path: '/home/blogList/state/0'})
          break
        }
        case 'homeBlogListState1': {
          this.$router.push({path: '/home/blogList/state/1'})
          break
        }
        case 'homeBlogListState2': {
          this.$router.push({path: '/home/blogList/state/2'})
          break
        }
        case 'homePlateEditInf': {
          this.$router.push({path: '/home/plate/inf'})
          break
        }
        case 'homePlateEditPost': {
          this.$router.push({path: '/home/plate/post'})
          break
        }
        case 'homeAdminUserList': {
          this.$router.push({path: '/home/admin/userList'})
          break
        }
        case 'homeAdminCategoryList': {
          this.$router.push({path: '/home/admin/categoryList'})
          break
        }
        case 'homeAdminUserCategoryList': {
          this.$router.push({path: '/home/admin/userCategoryList'})
          break
        }
      }
    },
    existRoleName (roleName) {
      for (let index = 0; index < this.roleList.length; index++) {
        if (this.roleList[index].name === roleName) return true
      }
      return false
    }
  }
}
</script>
<style>
.layout{
  height: 100vh;
}
.layout-header-bar{
  padding: 0px !important;
  background: #fff !important;
  box-shadow: 0px 2px 10px #cccccc !important;
  overflow: hidden !important;
  display: flex !important;
  justify-content: flex-start !important;
  align-items: center !important;
}
.menu-icon{
  transition: all .3s;
}
.rotate-icon{
  transform: rotate(-90deg);
}
.menu-item span{
  display: inline-block;
  overflow: hidden;
  width: 69px;
  text-overflow: ellipsis;
  white-space: nowrap;
  vertical-align: bottom;
  transition: width .2s ease .2s;
}
.menu-item i{
  transform: translateX(0px);
  transition: font-size .2s ease, transform .2s ease;
  vertical-align: middle;
  font-size: 16px;
}
.collapsed-menu span{
  width: 0px;
  transition: width .2s ease;
}
.collapsed-menu i{
  transform: translateX(5px);
  transition: font-size .2s ease .2s, transform .2s ease .2s;
  vertical-align: middle;
  font-size: 22px;
}
</style>
