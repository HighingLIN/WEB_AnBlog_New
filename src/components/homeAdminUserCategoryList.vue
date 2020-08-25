<template>
  <div>
    <Select v-model="categoryId" style="width: 100%; margin-bottom: 20px;" placeholder="请选择板块" @on-change="SelectCategoryId()">
      <Option v-for="item in categoryList" :key="item.id" :value="item.id">{{item.name}}</Option>
    </Select>
    <Button v-if="categoryId!=null" type="primary" style="width: 100%; margin-bottom: 20px;" @click="addCategoryModal()">添加版主</Button>
    <div>
      <Table :columns="tableHeader" :data="userCategoryList">
        <template slot-scope="{ row }" slot="name">
          <strong>{{ row.name }}</strong>
        </template>
        <template slot-scope="{ row, index }" slot="action">
          <Button type="error" size="small" @click="deleteUserCategory(row)">删除</Button>
        </template>
      </Table>
    </div>
    <!-- <div style="margin-top: 10px; display: flex; justify-content: flex-end; align-items: center;">
      <Page @on-change="pageChange" :total="categoryTotal" :current="page" :page-size="size"/>
    </div> -->
    <Modal v-model="modal" :title="modalTitle" @on-ok="modalOk" @on-cancel="modalCancel">
      <!-- <Form ref="categoryForm" :model="categoryFormData" :rules="categoryFormRules" label-position="left" :label-width="80"> -->
      <Form ref="categoryForm" :model="userCategoryFormData" label-position="left" :label-width="80">
        <FormItem label="用户ID" prop="passwordOriginal">
          <Input type="text" maxlength="11" v-model="userCategoryFormData.userId" placeholder="请输入"/>
        </FormItem>
      </Form>
    </Modal>
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
          title: '板块名称',
          key: 'category.name',
          render: (h, params) => {
            return h('div', params.row.category.name)
          }
        },
        {
          title: '用户昵称',
          key: 'user.nickname',
          render: (h, params) => {
            return h('div', params.row.user.nickname)
          }
        },
        {
          title: '操作',
          slot: 'action',
          align: 'center'
        }
      ],
      categoryList: [],
      categoryId: null,
      userCategoryList: [],
      userCategoryTotal: 0,
      page: 1,
      size: 10,
      modal: false,
      modalTitle: '',
      userCategoryFormData: {
        id: null,
        userId: null,
        categoryId: null
      }
    }
  },
  mounted () {
    this.page = 1
    this.size = 10
    this.getCategoryList()
    // this.getUserCategoryList()
  },
  watch: {
    '$route' (to, from) {
      this.page = 1
      this.size = 10
      this.getCategoryList()
      // this.getUserCategoryList()
    }
  },
  methods: {
    getCategoryList () {
      this.$get('/tourist/categoryList')
        .then(data => {
          this.categoryList = data.data.data.categoryList
        })
    },
    getUserCategoryList () {
      if (this.categoryId != null) {
        this.$get('/tourist/userCategoryList/' + this.categoryId + '?page=' + this.page + '&size=' + this.size)
          .then(data => {
            this.userCategoryList = data.data.data.userCategoryList
            this.userCategoryTotal = data.data.data.userCategoryTotal
          })
      }
    },
    SelectCategoryId () {
      this.getUserCategoryList()
    },
    pageChange (page) {
      this.page = page
      this.getCategoryList()
    },
    addCategoryModal () {
      this.modalTitle = '添加板主'
      this.modal = true
      this.userCategoryFormData = {
        id: null,
        userId: null,
        categoryId: this.categoryId
      }
    },
    deleteUserCategory (row) {
      this.$delete('/admin/userCategory/' + row.id)
        .then(data => {
          if (data.data.code === 0) { // 成功
            this.$Message.success(data.data.message) // 提示
            this.getUserCategoryList()
          } else { // 失败
            this.$Message.error(data.data.message) // 提示
          }
        })
    },
    modalOk () {
      this.$post('/admin/userCategory', this.userCategoryFormData)
        .then(data => {
          if (data.data.code === 0) { // 成功
            this.$Message.success(data.data.message) // 提示
            this.getUserCategoryList()
          } else { // 失败
            this.$Message.error(data.data.message) // 提示
          }
        })
    },
    modalCancel () {
      this.categoryFormData = {
        id: null,
        userId: null,
        categoryId: null
      }
    }
  }
}
</script>
<style>
</style>
