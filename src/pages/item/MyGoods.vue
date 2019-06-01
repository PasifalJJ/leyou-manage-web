<template>
  <v-card>
    <v-card-title>
      <v-btn color="primary" @click="addGoods">新增商品</v-btn>
      <!--搜索框，与search属性关联-->
      <v-spacer/>
      <v-flex xs3>
        <v-btn-toggle v-model="filter.saleable">
          <v-btn flat>
            全部
          </v-btn>
          <v-btn flat :value="true">
            上架
          </v-btn>
          <v-btn flat :value="false">
            下架
          </v-btn>
        </v-btn-toggle>
      </v-flex>
      <v-spacer/>
      <v-text-field lang="输入搜索关键字" v-model.lazy="filter.search" append-icon="search" hide-details/>
    </v-card-title>
    <v-divider/>
    <!--表格-->
    <v-data-table
      :headers="headers"
      :items="goodsList"
      :search="filter.search"
      :pagination.sync="pagination"
      :total-items="totalGoods"
      :loading="loading"
      class="elevation-1"
    >
      <!--表格数据渲染-->
      <template slot="items" slot-scope="props">
        <td>{{ props.item.id }}</td>
        <td class="text-xs-center">{{ props.item.title }}</td>
        <td class="text-xs-center">{{props.item.cname}}</td>
        <td class="text-xs-center">{{ props.item.bname }}</td>
        <td class="justify-center layout px-0">
          <v-btn icon small @click="editItem(props.item)">
            <i class="el-icon-edit"/>
          </v-btn>
          <v-btn icon small @click="deleteItem(props.item.id)">
            <i class="el-icon-delete"/>
          </v-btn>
          <v-btn icon small v-if="props.item.saleable">下架</v-btn>
          <v-btn icon v-else>上架</v-btn>
        </td>
      </template>
      <template slot="no-data">
        <v-alert :value="true" color="error" icon="warning">
          未查询到任何数据
        </v-alert>
      </template>
      <template slot="pageText" slot-scope="props">
        共{{props.itemsLength}}条,当前:{{ props.pageStart }}
      </template>
    </v-data-table>

    <!--弹出对话框-->
    <v-dialog max-width="800" v-model="show" persistent scrollable>
      <v-card>
        <!--对话框标题-->
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>{{isEdit ? '修改':'新增'}}商品</v-toolbar-title>
          <v-spacer/>
          <!--关闭窗口按钮-->
          <v-btn icon @click="closeWindow">
            <v-icon>close</v-icon>
          </v-btn>
        </v-toolbar>
        <!--对话框的内容，表单-->
        <v-card-text class="px-5" style="height: 600px">
          <my-goods-form :oldGoods="oldGoods" :step="step"/>
        </v-card-text>
        <!--底部按钮，用来操作步骤线，dialog设置scrollable时，弹框的大小就可以确定-->
        <v-card-actions class="elevation-10">
          <v-flex class="xs3 mx-auto">
            <v-btn @click="previous" color="primary" :disbale="step===1">上一步</v-btn>
            <v-btn @click="next" color="primary" :disbale="step===4">下一步</v-btn>
          </v-flex>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-card>
</template>

<script>
  import MyGoodsForm from "./MyGoodsForm";

  export default {
    name: "MyGoods",
    data() {
      return {
        totalGoods: 0,  //总条数
        goodsList: [],  //当前页品牌数据
        loading: true,  //是否在加载中
        pagination: {},  //分页信息
        headers: [
          {text: 'id', align: 'center', value: 'id'},
          {text: '标题', align: 'center', value: 'title', sortable: false},
          {text: '商品分类', align: 'center', value: 'cname', sortable: false},
          {text: '品牌', align: 'center', value: 'bname', sortable: false},
          {text: '操作', align: 'center', sortable: false},
        ],
        show: false, //控制对话框的显示
        oldGoods: {}, //即将编辑的商品信息，用做商品的回显
        isEdit: false,  //是否是编辑
        filter: {
          saleable: null, //商家还是下架
          search: ''  //搜索过滤字段
        },
        step: 1
      }
    },
    mounted() {
      this.getDataFromServer();
    },
    watch: {
      pagination: { //监视pagination属性的变化
        deep: true, //deep为true，会监视pagination的属性及属性中的对象属性变化
        handler() {
          this.getDataFromServer();
        }
      },
      filter: {
        deep: true,
        handler() {
          this.getDataFromServer();
        }
      }
    },
    methods: {
      getDataFromServer() {  //从服务器加载数据
        this.loading = true;
        //发起请求
        this.$http.get("/item/spu/page", {
          params: {
            key: this.filter.search, // 搜索条件
            page: this.pagination.page,// 当前页
            rows: this.pagination.rowsPerPage,// 每页大小
            sortBy: this.pagination.sortBy,// 排序字段
            desc: this.pagination.descending,// 是否降序
            saleable: this.filter.saleable === 0 ? null : this.filter.saleable //是否上架
          }
        }).then(resp => { //必须使用箭头函数，不然this.goodsList收取不到数据
          this.goodsList = resp.data.items;
          this.totalGoods = resp.data.total;
          // 完成赋值以后，把加载状态赋值为false
          this.loading = false;
        }).catch(() => {
          this.goodsList = [];
          this.totalGoods = 0;
          this.loading = false;
        })
      },
      addGoods() {
        //修改标记
        this.isEdit = false;
        //弹窗可见
        this.show = true;
        //将oldGoods变为null
        this.oldGoods = null;
      },
      closeWindow() {
        //重新加载分页数据
        this.getDataFromServer();
        //关闭窗口
        this.show = false;
      },
      previous() {
        if (this.step > 1) {
          this.step--;
        } else {
          this.step = 1;
        }
      },
      next() {
        if (this.step < 4) {
          this.step++;
        } else {
          this.step = 4;
        }
      }
    },
    components: {MyGoodsForm}
  }
</script>
<style scoped>
</style>
