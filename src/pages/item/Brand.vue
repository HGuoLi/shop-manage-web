<template>
  <v-card>
    <v-card-title class="layout row">
      <v-btn @click="show = true" color="primary">新增品牌</v-btn>
      <v-spacer/>
      <v-text-field
        append-icon="search"
        label="输入关键字搜索"
        single-line
        hide-details
        v-model="search"
        class="flex sm3"
      />
    </v-card-title>
    <v-divider/>
    <v-data-table
      :headers="headers"
      :items="brands"
      :pagination.sync="pagination"
      :total-items="totalBrands"
      :loading="loading"
      class="elevation-1"
    >
      <template slot="items" slot-scope="props">
        <td class="text-xs-center">{{ props.item.id }}</td>
        <td class="text-xs-center">{{ props.item.name }}</td>
        <td class="text-xs-center">
          <img v-if="props.item.image" :src="props.item.image" width="130" height="40">
          <span v-else>敬请期待...</span>
        </td>
        <td class="text-xs-center">{{ props.item.letter }}</td>
        <td class="justify-center layout px-0">
          <v-btn icon class="mx-0" @click="editItem(props.item)">
            <v-icon color="info">edit</v-icon>
          </v-btn>
          <v-btn icon class="mx-0" @click="editItem(props.item)">
            <v-icon color="error">delete</v-icon>
          </v-btn>
        </td>
      </template>
    </v-data-table>
    <!--弹出的对话框-->
    <v-dialog max-width="500" v-model="show" persistent scrollable>
      <v-card>
        <!--对话框的标题-->
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>新增品牌</v-toolbar-title>
          <v-spacer/>
          <v-btn icon @click="show = false">
            <v-icon>close</v-icon>
          </v-btn>
        </v-toolbar>
        <!--对话框的内容，表单-->
        <v-card-text style="height: 500px">
          <brand-form @close="closeWindow"/>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script>
  import BrandForm from './BrandForm'
  export default {
    name: "brand",
    data() {
      return {
        totalBrands: 0, // 总条数
        brands: [], // 当前页品牌数据
        loading: false, // 是否在加载中
        pagination: {}, // 分页信息
        search:'',
        headers: [
          {text: 'id', align: 'center', sortable: true,value: 'id'},
          {text: '名称', align: 'center', sortable: false, value: 'name'},
          {text: 'LOGO', align: 'center', sortable: false, value: 'image'},
          {text: '首字母', align: 'center',  sortable: false,value: 'letter'},
          {text: '操作', align: 'center',  sortable: false}
        ],
        show:false
      }
    },
    watch:{
      pagination:{
        deep:true,
        handler(){
          this.getDataFromServer();
        }
      },
      search(){
        this.getDataFromServer();
      }
    },
    created(){ // 渲染后执行
      // 查询数据
      this.getDataFromServer();
    },
    methods:{
      getDataFromServer(){ // 从服务的加载数的方法。
        //开启进度条
        this.loading = true;
        //发起ajax请求，page,rows,key,sortBy,desc
        this.$http.get("/item/brand/page", {
          params: {
            page: this.pagination.page,// 当前页
            rows: this.pagination.rowsPerPage,// 每页大小
            sortBy: this.pagination.sortBy,// 排序字段
            desc: this.pagination.descending,// 是否降序
            key: this.search, // 搜索条件
          }
        }).then(resp => { // 这里使用箭头函数
          this.brands = resp.data.items;
          this.totalBrands = resp.data.total;
          // 完成赋值后，把加载状态赋值为false
          this.loading = false;
        }).catch(resp => { // 这里使用箭头函数
          this.brands = [];
          this.totalBrands = 0;
          this.loading = false;
        })
      },
      closeWindow(){
        // 关闭窗口
        this.show = false;
        // 重新加载数据
        this.getDataFromServer();
      }
    },
    components:{
      BrandForm
    }
  }
</script>

<style scoped>

</style>
