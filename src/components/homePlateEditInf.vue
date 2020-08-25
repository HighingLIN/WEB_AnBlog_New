<template>
  <div>
    <template>
        <Select v-model="categoryId" style="width: 100; margin-bottom: 20px; margin-right:6px;" placeholder="请选择板块" @on-change="SelectCategoryId()">
            <Option v-for="item in categoryList" :key="item.id" :value="item.id">{{item.name}}</Option>
        </Select>
    </template>
    <div>
      <Table :columns="tableHeader" :data="plateList">
        <template slot-scope="{ row }" slot="name">
          <strong>{{ row.name }}</strong>
        </template>
        <template slot-scope="{ row, index }" slot="edit">
          <i-button type="primary" shape="circle" @click="editCategory(row,index)">编辑</i-button>
        </template>
      </Table>
    </div>
    <div style="margin-top: 10px; display: flex; justify-content: flex-end; align-items: center;">
      <Page @on-change="pageChange" :total="plateTotal" :current="page" :page-size="size"/>
    </div>

    <template>
      <Modal v-model="modal1" title="编辑" @on-ok="changeCategoryInfo" @on-cancel="cancel">
        <Form :model="editPlate" label-position="left" :label-width="60">
          <!-- <FormItem label="版名">
            <Input
              type="text"
              maxlength="10"
              v-model="editPlate.name"
              placeholder="请输入"
              show-word-limit
            /> -->
          <!-- </FormItem> -->
          <FormItem label="描述">
            <Input
              type="text"
              maxlength="20"
              v-model="editPlate.describe"
              placeholder="请输入"
              show-word-limit
            />
          </FormItem>
          <FormItem label="公告">
            <Input
              type="textarea"
              :rows="4"
              maxlength="64"
              v-model="editPlate.notice"
              placeholder="请输入"
              show-word-limit
            />
          </FormItem>
        </Form>
      </Modal>
    </template>
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
          title: "描述",
          key: "describe",
        },
        {
          title: "公告",
          key: "notice",
        },
        {
          title: "操作",
          slot: "edit",
          align: "center",
        },
      ],
      editPlate: {
        // name: "",
        index:null,
        describe: "",
        notice: "",
      },
      plateList1:{},
      plateList: [
    //     {
    //         // 测试
        // // name: "s",
        // id:"1",
        // describe: "ss",
        // notice: "s",
    //   },
    //   板块公告描述传输格式
      ],
      categoryList:[
          {
            //   测试
            //   id:"1",
            //   name:"s",
          }
        //   板块名传输格式
      ],
      categoryId:null,
      page: 1,
      size: 10,
      plateTotal: 0,
      modal1:false,
    };
  },
  mounted() {
    this.page = 1;
    this.size = 10;
    this.getCategoryList();
  },
  watch: {
    '$route' (to, from) {
      this.page = 1;
      this.size = 10;
      this.getCategoryList();
    }
  },
  methods: {
    SelectCategoryId(){
        console.log(this.categoryId);
        this.getPlateInfo();
        // 根据板块id获取帖子信息
    },
    editCategory(data, idx) {
      this.editPlate = {
        // name: data.name,
        index:idx,
        describe: data.describe,
        notice: data.notice,
      };
      this.modal1 = true;
    },
    changeCategoryInfo() {
    //   put板块公告描述信息  
    this.$put('/user/userCategory', {   id:this.plateList[this.editPlate.index].id,
                                        describe:this.editPlate.describe,
                                        notice:this.editPlate.notice })
          .then(data => {
            if (data.data.code === 0) { // 成功
              this.plateList[this.editPlate.index].describe = this.editPlate.describe;
              this.plateList[this.editPlate.index].notice = this.editPlate.notice;
            } else { // 失败
              this.$Message.error(data.data.message) // 提示
            }
          })
      this.$Message.info("修改完成");
    },
    cancel() {
      this.$Message.info("放弃编辑");
    },
    getPlateInfo() {
        // 获取板块公告描述信息
        this.categoryList.forEach((item) =>{
                if(item.id==this.categoryId)
                    this.plateList1=item;
            });
        console.log(this.plateList1)
        this.plateList=[]
        this.plateList.push(this.plateList1)


    },
    getCategoryList () {
        // 获取板块信息
        this.$get('/user/categoryList')
        .then(data => {
          this.categoryList = data.data.data.categoryList;
        })
    },
    pageChange (page) {
      this.page = page
      this.getPlateInfo()
    }
  },
};
</script>

<style>
.flex-center {
  display: flex;
  justify-content: center;
  align-items: center;
}
.imageUpdate {
  box-shadow: 0px 2px 6px #cccccc;
}
.imageUpdate:hover {
  opacity: 0.6;
  box-shadow: 0px 2px 6px #222222;
  transition: 0.25s;
  cursor: pointer;
}
.profilePhotoImg {
  width: 60px;
  height: 60px;
  border-radius: 30px;
}
.appreciationCodeImg {
  width: 200px;
  height: 200px;
}
</style>