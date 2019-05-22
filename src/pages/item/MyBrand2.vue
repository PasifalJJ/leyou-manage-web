<template>
  <div>
    <v-layout row wrap>
      <v-flex xs2>
        <v-btn color="info" @click="addBrand">新增品牌</v-btn>
      </v-flex>
      <v-spacer></v-spacer>
      <v-flex xs4 pb-2>
        <v-text-field
          append-icon="search"
          label="搜索"
          single-line
          hide-details
          v-model="search"
        />
      </v-flex>
    </v-layout>

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
        <td class="text-xs-center"><img :src="props.item.image" alt=""></td>
        <td class="text-xs-center">{{ props.item.letter }}</td>
      </template>
    </v-data-table>

    <v-dialog v-model="show"  max-width="500" persistent>
      <v-card>
        <v-toolbar dense dark color="primary">
          <v-toolbar-title>新增品牌</v-toolbar-title>
          <v-spacer/>
          <v-btn icon @click="closeWindow">
            <v-icon>close</v-icon>
          </v-btn>
        </v-toolbar>
        <v-card-text class="px-5">
          <my-brand2-form></my-brand2-form>
        </v-card-text>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
import MyBrand2Form from './MyBrand2Form.vue'
  export default {
    name: "MyBrand2",
    data() {
      return {
        headers: [{text: '商品id', align: 'center', value: 'id', sortable: true,},
          {text: '商品名称', align: 'center', value: 'name'},
          {text: '商品logo', align: 'center', value: 'image'},
          {text: '商品首字母', align: 'center', value: 'letter'}],
        totalBrands: 5,
        brands: [],
        pagination: {},
        loading: false,
        search: '',
        show: false,
      }
    },
    methods: {
      addBrand() {
        this.show=true
      },
      closeWindow(){
        this.show=false
      }
      ,
      loadPage() {
        if (this.pagination.page != null && this.pagination.rowsPerPage != null) {
          this.$http.get("/item/brand/list", {
            params: {
              page: this.pagination.page,
              rowsPerpage: this.pagination.rowsPerPage,
              descending: this.pagination.descending,
              sortBy: this.pagination.sortBy,
            }
          }).then( (response)=> {
            console.log(response.data.brands)
            this.brands = response.data.brands;
            this.totalBrands=response.data.totalBrands
          }).catch( (error)=> {
            console.log(error);
          })
        }
      }
    },
    components:{
      MyBrand2Form
    }
    ,
    //创建组件的时候查询数据库，查询数据
    watch: {
      pagination: {
        deep: true,
        handler() {
          this.loadPage()
        }
      },
      search: {
        handler() {
          this.loadPage()
        }
      }
    }
  }
</script>

<style scoped>

</style>
