<template>
  <v-card>
    <v-flex xs12 sm10>
      <v-tree url="/item/category/list"
              :isEdit="isEdit"
              :treeData="treeData"
              @handleAdd="handleAdd"
              @handleEdit="handleEdit"
              @handleDelete="handleDelete"
              @handleClick="handleClick"
      />
    </v-flex>
  </v-card>
</template>
<!--:treeData="treeData" 当树组件访问url="/item/category/list"，如果tree有数据，将会启用延迟加载，优先读取treedata内容，
有数据则不发送请求，直接使用本地数据，因此只有删除之后才能正常向服务器发送请求-->
<script>
  import {treeData} from '../../mockDB'

  export default {
    name: "category",
    data() {
      return {
        treeData: null,
        isEdit: true
      }
    },
    methods: {
      handleAdd(node) {
        console.log("add .... ");
        console.log(node);
      },
      handleEdit(id, name) {
        console.log("edit... id: " + id + ", name: " + name)
      },
      handleDelete(id) {
        console.log("delete ... " + id)
      },
      handleClick(node) {
        this.$http.get("/item/category/list", {
          params: {
            pid: node.id
          }
        }).then((response) => {
          var data = response.data;
          //将json对象转为数组
          let keys = Object.keys(data);
          let dataArr = keys.map((i) => {
            return data[i]
          });
          console.log(dataArr);
          this.treeData = dataArr;
        }).catch((error) => {
          console.log("error---->" + error)
        })
      }
    }
  };
</script>
<style scoped>
</style>
