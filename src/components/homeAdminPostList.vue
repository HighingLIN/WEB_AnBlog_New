<template>
  <div>
    <div>
    <template>
    <div>
    <label style="margin:5px">标题:</label><Input v-model="title" placeholder="请输入标题部分关键字" style="width: auto" />
    <label style="margin:5px">分类:</label><Input v-model="category" placeholder="请输入帖子分类" style="width: auto" />
    <label style="margin:5px">摘要:</label><Input v-model="summary" placeholder="请输入摘要部分关键字" style="width: auto" />
    <Button type="info" shape="circle" icon="ios-search" style="margin-left:20px" @click="searchBlogList()">搜索</Button>
    <Button type="info" shape="circle" icon="ios-refresh" style="margin:5px" @click="reset()">重置搜索框</Button>
    <Button type="info" shape="circle" icon="ios-refresh" style="margin:5px" @click="getBlogList()">显示所有</Button>
    <Button type="info" shape="circle" icon="md-close-circle" style="margin:5px" @click="deleteMany()">批量删除</Button>
    </div>
     <Table border ref="selection":columns="tableHeader" :data="blogList">
        <template slot-scope="{ row }" slot="name">
          <strong>{{ row.name }}</strong>
        </template>
        <template slot-scope="{ row, index }" slot="操作">
            <Button @click="goBlogView(row.id)" type="success" size="small" style="margin-right: 5px;">预览</Button>
            <Button @click="goBlogEdit(row.id)" type="primary" size="small" style="margin-right: 5px;">编辑</Button>
            <Button @click=" deleteBlog(row.id)" type="error" size="small" style="margin-right: 5px;">删除</Button>
        </template>
      </Table>
    
    </template>
  </div>
  <div style="margin-top: 10px; display: flex; justify-content: flex-end; align-items: center;">
      <Page @on-change="pageChange" :total="blogTotal" :current="page" :page-size="size"/>
    </div>
  </div>
</template>
<script>
export default {
  data () {
    return {
      tableHeader: [
        {
        type: 'selection',
        width: 60,
        align: 'center'
         },
        {
          title: '帖子ID',
          key:"id",

        },
        {
          title: '作者ID',
          key: 'userId'
        },
        {
          title: '分类',
          key: 'categoryId'
        },
        {
          title: '状态',
          key: 'state'
        },
        {
          title: '标题',
          key: 'title',
        },
        {
          title: '摘要',
          key: 'summary',
        },
        {
        title: '操作',
        slot: '操作',
        width: 200,
        align: 'center'
        }
      ],
      model1:'',
      blogList: [],
      blogTotal: 0,
      page: 1,
      size: 10,
      title:null,
      category:null,
      summary:null,
      deleteList:[],
      deleteListId:[],
      categoryList:[],
      }
  },
  mounted () {
    this.page = 1
    this.size = 10
    this.getBlogList()
  },
  watch: {
    '$route' (to, from) {
      this.page = 1
      this.size = 10
      this.getBlogList()
    }
  },
  methods: {
    deleteMany(selection, index){
      this.deleteList=this.$refs.selection.getSelection()
      
      for(var i=0;i<this.deleteList.length;i++){
        this.deleteListId[i]=this.deleteList[i].id
      }
    this.$delete('/user/blogList/' + this.deleteListId)
        .then(data => {
          if (data.data.code === 0) { // 成功
            this.$Message.success(data.data.message) // 提示
            this.getBlogList() // 刷新页面
          } else { // 失败
            this.$Message.error(data.data.message) // 提示
          }
        })
      
    },
    pageChange (page) {
      this.page = page
      this.getBlogList()
    },
    goBlogView (id) {
      this.$router.push({path: '/indexBlog/' + id})
    },
    goBlogEdit (id) {
      this.$router.push({path: '/home/blog/edit/html/' + id})
    },
    deleteBlog (id) {
      this.$delete('/user/blog/' + id)
        .then(data => {
          if (data.data.code === 0) { // 成功
            this.$Message.success(data.data.message) // 提示
            this.getBlogList() // 刷新页面
          } else { // 失败
            this.$Message.error(data.data.message) // 提示
          }
        })
    },
    pageChange (page) {
      this.page = page
      this.getBlogList()
    },
    getBlogList () {
      this.$get('/admin/blogList/search' + '?page=' + this.page + '&size=' + this.size)
        .then(data => {
          this.blogList = data.data.data.blogList
          for(var i=0;i<this.blogList.length;i++){
          this.blogList[i].state="已发表"}
          this.blogTotal = data.data.data.blogTotal
        })
        
    },
    searchBlogList () {
      this.$get('/admin/blogList/search' + '?page=' + this.page + '&size=' + this.size +(this.title==null ? "" : "&title="+ this.title ) +(this.category==null ? "" : "&categoryId=" + this.category )+(this.summary==null ? "" : "&summary=" + this.summary ))
        .then(data => {
          this.blogList = data.data.data.blogList
          for(var i=0;i<this.blogList.length;i++){
          this.blogList[i].state="已发表"}
        this.blogTotal = data.data.data.blogTotal
          })
      }, 
    
    reset(){
      this.title=null,this.category=null,this.summary=null
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