<template>
  <div>
    <template>
        <Select v-model="categoryId" style="width: 200px; margin-bottom: 20px; margin-right:6px;" placeholder="请选择板块" @on-change="SelectCategoryId()">
            <Option v-for="item in categoryList" :key="item.id" :value="item.id">{{item.name}}</Option>
        </Select>
    </template>
    <template>
        <i-input v-if="categoryId!=null" v-model="title" placeholder="请输入标题" style="width: 200px; margin-right:6px; margin-bottom: 20px;"></i-input>
        <Button v-if="categoryId!=null" type="primary" style="margin-bottom: 20px;" shape="circle" @click="getSearchData()">搜索</Button>
    </template>
    <div>
      <Table :columns="tableHeader" :data="postList">
        <template slot-scope="{ row }" slot="name">
          <strong>{{ row.name }}</strong>
        </template>
        <template slot-scope="{ row }" slot="edit">
          <Button @click="deletePost(row.id)" type="error" shape="circle">删除</Button>
        </template>
      </Table>
    </div>
    <div style="margin-top: 10px; display: flex; justify-content: flex-end; align-items: center;">
      <Page @on-change="pageChange" :total="postTotal" :current="page" :page-size="size" />
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      tableHeader: [
        {
          title: "id",
          key: "id",
        },
        // {
        //   title: "版名",
        //   key: "name",
        // },
        {
          title: "标题",
          key: "title",
        },
        {
          title: "编辑时间",
          key: "editDate",
        },
        {
          title: "操作",
          slot: "edit",
          align: "center",
        },
      ],
      postList: [
        {
        //   id: "1",
        // //   name: "s",
        //   title: "s",
        //   editDate: "s",
        },
        // 帖子传输格式
      ],
      categoryList:[
        //   {
        //       id:"1",
        //       name:"s",
        //   }
        //   板块名传输格式
      ],
      page: 1,
      size: 10,
      postTotal: 0,
      categoryId:null,
      title:"",
      searchTitle:"",
    };
  },
  mounted() {
    this.page = 1;
    this.size = 10;
    this.headers = { Authorization: localStorage.getItem("token") };
    this.getCategoryList();
  },
  watch: {
    $route(to, from) {
      this.page = 1;
      this.size = 10;
      this.getCategoryList();
    },
  },
  methods: {
    SelectCategoryId(){
        console.log(this.categoryId);
        this.title="";
        this.searchTitle="";
        this.getPostInfo();
        // 根据板块id获取帖子信息
    },
    deletePost(id) {
        // 根据id删除帖子
        this.$delete('/user/blog/' + id)
        .then(data => {
          if (data.data.code === 0) { // 成功
            this.$Message.success(data.data.message) // 提示
         // 刷新页面
            if(this.searchTitle)
                this.getSearchData();
             else
                this.getPostInfo();
          } else { // 失败
            this.$Message.error(data.data.message) // 提示
          }
        })
    },
    getPostInfo() {
        // 获取该板块下帖子信息
        this.$get('/user/cateBlogList/show' + '?categoryId=' + this.categoryId + '&page=' + this.page + '&size=' + this.size)
        .then(data => {
          this.postList = data.data.data.blogList;
          this.postTotal = data.data.data.blogTotal
        })
    },
    getCategoryList () {
        // 获取板块信息
        this.$get('/user/categoryList')
        .then(data => {
          this.categoryList = data.data.data.categoryList;
            // console.log(data)
        })
    },
    getSearchData(){
        // 根据title获取帖子信息
        this.searchTitle=this.title;
        this.$get('/user/cateBlogList/show' + '?categoryId=' + this.categoryId + '&title=' + this.searchTitle + '&page=' + this.page + '&size=' + this.size)
        .then(data => {
          this.postList = data.data.data.blogList;
          this.postTotal = data.data.data.blogTotal;
        })
    },
    pageChange(page) {
      this.page = page;
      if(this.searchTitle)
        this.getSearchData();
      else
        this.getPostInfo();
    },
  },
};
</script>