<template>
  <div>
    <Button type="primary" style="width: 100%; margin-bottom: 20px;" @click="addCategoryModal()">添加</Button>
    <div>
      <Table :columns="tableHeader" :data="categoryList">
        <template slot-scope="{ row }" slot="name">
          <strong>{{ row.name }}</strong>
        </template>
        <template slot-scope="{ row, index }" slot="action">
          <Button type="success" size="small" style="margin-right: 5px" @click="updateCategoryModal(row)">编辑</Button>
          <Button type="error" size="small" @click="deleteCategory(row)">删除</Button>
        </template>
      </Table>
    </div>
    <!-- <div style="margin-top: 10px; display: flex; justify-content: flex-end; align-items: center;">
      <Page @on-change="pageChange" :total="categoryTotal" :current="page" :page-size="size"/>
    </div> -->
    <Modal v-model="modal" :title="modalTitle" @on-ok="modalOk" @on-cancel="modalCancel">
      <!-- <Form ref="categoryForm" :model="categoryFormData" :rules="categoryFormRules" label-position="left" :label-width="80"> -->
      <Form ref="categoryForm" :model="categoryFormData" label-position="left" :label-width="80">
        <FormItem label="名称" prop="passwordOriginal">
          <Input type="text" maxlength="32" v-model="categoryFormData.name" placeholder="请输入"/>
        </FormItem>
        <FormItem label="描述" prop="passwordChanged">
          <Input type="text" maxlength="32" v-model="categoryFormData.describe" placeholder="请输入"/>
        </FormItem>
        <FormItem v-if="modalTitle=='编辑板块'" label="公告" prop="passwordChangedCheck">
          <Input type="text" maxlength="256" v-model="categoryFormData.notice" placeholder="请输入"/>
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
          title: '名称',
          key: 'name'
        },
        {
          title: '描述',
          key: 'describe'
        },
        {
          title: '公告',
          key: 'notice'
        },
        {
          title: '操作',
          slot: 'action',
          align: 'center'
        }
      ],
      categoryList: [],
      categoryTotal: 0,
      page: 1,
      size: 10,
      modal: false,
      modalTitle: '',
      categoryFormData: {
        id: null,
        name: null,
        describe: null,
        notice: null
      }
    }
  },
  mounted () {
    this.page = 1
    this.size = 10
    this.getCategoryList()
  },
  watch: {
    '$route' (to, from) {
      this.page = 1
      this.size = 10
      this.getCategoryList()
    }
  },
  methods: {
    getCategoryList () {
      this.$get('/tourist/categoryList' + '?page=' + this.page + '&size=' + this.size)
        .then(data => {
          this.categoryList = data.data.data.categoryList
          this.categoryTotal = data.data.data.categoryTotal
        })
    },
    pageChange (page) {
      this.page = page
      this.getCategoryList()
    },
    addCategoryModal () {
      this.modalTitle = '添加板块'
      this.modal = true
      this.categoryFormData = {
        id: null,
        name: null,
        describe: null,
        notice: null
      }
    },
    updateCategoryModal (row) {
      this.modalTitle = '编辑板块'
      this.modal = true
      this.categoryFormData = {
        id: row.id,
        name: row.name,
        describe: row.describe,
        notice: row.notice
      }
    },
    deleteCategory (row) {
      this.$delete('/admin/category/' + row.id)
        .then(data => {
          if (data.data.code === 0) { // 成功
            this.$Message.success(data.data.message) // 提示
            this.getCategoryList()
          } else { // 失败
            this.$Message.error(data.data.message) // 提示
          }
        })
    },
    modalOk () {
      if (this.categoryFormData.id == null) { // 添加
        this.$post('/admin/category', this.categoryFormData)
          .then(data => {
            if (data.data.code === 0) { // 成功
              this.$Message.success(data.data.message) // 提示
              this.getCategoryList()
            } else { // 失败
              this.$Message.error(data.data.message) // 提示
            }
          })
      } else { // 修改
        this.$put('/admin/category', this.categoryFormData)
          .then(data => {
            if (data.data.code === 0) { // 成功
              this.$Message.success(data.data.message) // 提示
              this.getCategoryList()
            } else { // 失败
              this.$Message.error(data.data.message) // 提示
            }
          })
      }
    },
    modalCancel () {
      this.categoryFormData = {
        id: null,
        name: null,
        describe: null,
        notice: null
      }
    }
  }
}
</script>
<style>
</style>
