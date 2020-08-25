<template>
  <div>
    <div>
    <template>
    <div>
    <label style="margin:5px">用户名::</label><Input v-model="username" prefix="ios-contact" placeholder="请输入用户账户名" style="width: auto" />
    <label style="margin:5px">昵称:</label><Input v-model="nickname" placeholder="请输入用户昵称" style="width: auto" />
    <label style="margin:5px">邮箱:</label><Input v-model="email" placeholder="请输入用户邮箱" style="width: auto" />
    <Button type="info" shape="circle" icon="ios-search" style="margin-left:20px" @click="userSearch()">搜索</Button>
    <Button type="info" shape="circle" icon="ios-refresh" style="margin:5px" @click="reset()">重置搜索框</Button>
    <Button type="info" shape="circle" icon="ios-refresh" style="margin:5px" @click="getUserList()">显示全部</Button>
    </div>
    <div style="margin:10px;display: flex; justify-content: flex-start; align-items: center;">
       <Button type="primary" icon="ios-add-circle-outline" size="large"  @click="Add = true">新增</Button>
    <Modal
        v-model="Add"
        title="添加新用户"
        @on-ok="userAdd()"
        @on-cancel="cancel"
        class-name="vertical-center-modal"
        >
        <div style="margin:10px;display: flex;align-items: center;justify-content: center;">
        <label >账号：</label><Input v-model="usernameAdd" placeholder="请输入用户账号" style="width: auto" />
        </div>
        <div style="margin:10px;display: flex;align-items: center;justify-content: center;">
        <label >昵称：</label><Input v-model="nicknameAdd" placeholder="请输入用户昵称" style="width: auto" />
        </div>
        <div style="margin:10px;display: flex;align-items: center;justify-content: center;">
        <label >邮箱：</label><Input v-model="emailAdd" placeholder="请输入用户邮箱" style="width: auto" />
        </div>
    </Modal>
     
    </div>
    </template>
      <Table :columns="tableHeader" :data="userList">
        <template slot-scope="{ row }" slot="name">
          <strong>{{ row.name }}</strong>
        </template>
        <template slot-scope="{ row, index }" slot="roleList">
          <Tag type="border" v-for="item in row.roleList" :key="item.id">{{item.name}}</Tag>
          <Poptip transfer>
            <Button icon="ios-settings" type="dashed" size="small">管理</Button>
            <div slot="content">
              <CheckboxGroup v-model="roleIdListCheckbox">
                <Checkbox :label="role.id" v-for="role in roleList" :key="role.id">{{role.name}}</Checkbox>
              </CheckboxGroup>
              <Button type="primary" size="small" @click="changeRoleList(index)">确定</Button>
              <Modal
                  v-model="Edit"
                  title="修改用户账号"
                  @on-ok="confirmEdit()"
                  @on-cancel="cancel"
                  class-name="vertical-center-modal"
                  >
                  <div style="margin:10px;display: flex;align-items: center;justify-content: center;">
                  <label >昵称：</label><Input v-model="nicknameEdit" placeholder="修改用户昵称" style="width: auto" />
                  </div>
                  <div style="margin:10px;display: flex;align-items: center;justify-content: center;">
                  <label >邮箱：</label><Input v-model="emailEdit" placeholder="修改用户邮箱" style="width: auto" />
                  </div>
               </Modal>
            </div>
          </Poptip>
        </template>
        <template slot-scope="{ row, index }" slot="action">
          <i-switch :value="row.enabled" :true-value="1" :false-value="0" @on-change="changeEnabled(index)"/>
          <Button  type="primary" size="small" style="margin-right: 5px;" @click="userEdit(index)">编辑</Button>
        </template>
      </Table>
    </div>
    <div style="margin-top: 10px; display: flex; justify-content: flex-end; align-items: center;">
      <Page @on-change="pageChange" :total="userTotal" :current="page" :page-size="size"/>
    </div>
  </div>
</template>
<script>
export default {
  data () {
    return {
      tableHeader: [
        {
          title: 'id',
          key: 'id'
        },
        {
          title: '账号',
          key: 'username'
        },
        {
          title: '邮箱',
          key: 'email'
        },
        {
          title: '昵称',
          key: 'nickname'
        },
        {
          title: '权限',
          slot: 'roleList',
          align: 'center'
        },
        {
          title: '账号封禁/编辑',
          slot: 'action',
          align: 'center'
        }
      ],
      userList: [],
      userTotal: 0,
      roleList: [],
      roleIdListCheckbox: [],
      page: 1,
      size: 10,
      Add: false,
      Edit:false,
      username:null,
      nickname:null,
      email:null,
      nicknameEdit:null,
      emailEdit:null,
      usernameAdd:null,
      nicknameAdd:null,
      emailAdd:null,
      userEditConfirm: [{"id":''},{"nickname":''},{"email":''}],
      userEditId:'',
      userAddConfirm:[{"id":""},{"nickname":""},{"email":""}]
    }
  },
  mounted () {
    this.page = 1
    this.size = 10
    this.getRoleList()
    this.getUserList()
  },
  watch: {
    '$route' (to, from) {
      this.page = 1
      this.size = 10
      this.getRoleList()
      this.getUserList()
    }
  },
  methods: {
    getRoleList () {
      this.$get('/user/roleList')
        .then(data => {
          this.roleList = data.data.data.roleList
        })
    },
    userSearch(){
       this.$get('/admin/userList' + '?page=' + this.page + '&size=' + this.size +(this.username==null ? "" : "&username="+ this.username ) +(this.nickname==null ? "" : "&nickname=" + this.nickname )+(this.email==null ? "" : "&email=" + this.email ))
        .then(data => {
          this.userList = data.data.data.userList
          this.userTotal = data.data.data.userTotal
        })
    },
    userAdd(){
      
      this.userAddConfirm[0].username=this.usernameAdd
      this.userAddConfirm[0].nickname=this.nicknameAdd
      this.userAddConfirm[0].email=this.emailAdd
      this.$post('/admin/user/addUserData', this.userAddConfirm[0])
        
    },
    userEdit(index){
      this.Edit=true
      this.nicknameEdit=this.userList[index].nickname
      this.emailEdit=this.userList[index].email
      this.userEditId=this.userList[index].id
      console.log(this.userEditId)
    },
    confirmEdit(){

      this.userEditConfirm[0].id=this.userEditId
      this.userEditConfirm[0].nickname=this.nicknameEdit
      this.userEditConfirm[0].email=this.emailEdit
      this.$post('/admin/user/userData',this.userEditConfirm[0]).then(data => {
          if (data.data.code === 0) { // 成功
            this.userList[index].enabled = enabled
          } else { // 失败
            this.$Message.error(data.data.message) // 提示
          }
        })
        this.getUserList ()
    },
    getUserList () {
      this.$get('/admin/userList' + '?page=' + this.page + '&size=' + this.size)
        .then(data => {
          this.userList = data.data.data.userList
          this.userTotal = data.data.data.userTotal
        })
    },
    changeEnabled (index) {
      var enabled = this.userList[index].enabled
      if (enabled === 1) enabled = 0
      else enabled = 1
      this.$put('/admin/user/' + this.userList[index].id + '/enabled', { enabled: enabled })
        .then(data => {
          if (data.data.code === 0) { // 成功
            this.userList[index].enabled = enabled
          } else { // 失败
            this.$Message.error(data.data.message) // 提示
          }
        })
    },
    changeRoleList (index) {
      var roleList = []
      for (let i = 0; i < this.roleIdListCheckbox.length; i++) {
        roleList.push({
          id: this.roleIdListCheckbox[i],
          name: this.getRoleName(this.roleIdListCheckbox[i])
        })
      }
      if (!(roleList.length > 0)) {
        this.$Message.error('请选择至少一个') // 提示
      } else {
        this.$put('/admin/user/' + this.userList[index].id + '/roleList', { roleList: roleList })
          .then(data => {
            if (data.data.code === 0) { // 成功
              this.userList[index].roleList = roleList
            } else { // 失败
              this.$Message.error(data.data.message) // 提示
            }
          })
      }
    },
    getRoleName (id) {
      for (let i = 0; i < this.roleList.length; i++) {
        if (this.roleList[i].id === id) return this.roleList[i].name
      }
    },
    pageChange (page) {
      this.page = page
      this.getUserList()
    },
     cancel () {
      this.$Message.info('取消操作');
    },
    reset(){
      this.username=null,this.nickname=null,this.email=null
    }
  }
}
</script>
<style lang="less">
    .vertical-center-modal{
        display: flex;
        align-items: center;
        justify-content: center;

        .ivu-modal{
            top: 0;
        }
    }
</style>