<template>
    <div>
      <v-form v-model="valid" ref="myBrandForm">
        <v-text-field v-model="brand.name" label="请输入品牌名称" required :rules="nameRules"></v-text-field>
        <v-text-field v-model="brand.letter" label="请输入品牌首字母" required :rules="letterRules" ></v-text-field>
        <v-cascader
          url="/item/category/list"
          multiple
          required
          v-model="brand.categories"
          label="请选择商品分类"/>
        <v-layout row>
          <v-flex xs3>
            <span style="font-size: 16px; color: #444">品牌LOGO：</span>
          </v-flex>
          <v-flex>
            <v-upload
              v-model="brand.image"
              url="/upload"
              :multiple="false"
              :pic-width="250"
              :pic-height="90"
            />
          </v-flex>
        </v-layout>
        <v-layout class="my-4" row>
          <v-spacer/>
          <v-btn @click="submit" color="primary">提交a</v-btn>
          <v-btn @click="clear" >重置</v-btn>
        </v-layout>
      </v-form>
    </div>
</template>

<script>
    export default {
        name: "MyBrand2Form",
      data(){
          return{
            valid:false,
            brand:{
              name:'', // 品牌名称
              letter:'', // 品牌首字母
              image:'',// 品牌logo
              categories:[], // 品牌所属的商品分类数组
            },
            nameRules:[
              v => !!v||"品牌不能为空",
              v => v.length>1 || "品牌名称至少为2位"
            ],
            letterRules:[
              v => !!v || "首字母不能为空",
              v =>/^[A-Z]{1}$/.test(v) || "品牌只能是A~Z的大写字母"
            ]
          }
      },
      methods:{
          submit(){
              if (this.$refs.myBrandForm.validate()) {
                const {categories,letter,...params}=this.brand;
                params.cids=categories.map(c=>c.id).join(",");
                params.letter=letter.toUpperCase();
                this.$http.post('item/brand',params).then(()=>{
                  this.$message.success("保存成功");
                }).catch(()=>{
                  this.$message.success("保存成功");
                });
              }
          },
          clear(){
            this.$refs.myBrandForm.reset();
            this.categories=[];
          }
      }
    }
</script>

<style scoped>

</style>
