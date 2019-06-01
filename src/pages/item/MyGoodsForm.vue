<template>
  <v-card>
    <v-stepper v-model="step">
      <v-stepper-header>
        <v-stepper-step :complete="step>1" step="1">基本信息</v-stepper-step>
        <v-divider/>
        <v-stepper-step :complete="step>2" step="2">商品描述</v-stepper-step>
        <v-divider/>
        <v-stepper-step :complete="step>3" step="3">基本信息</v-stepper-step>
        <v-divider/>
        <v-stepper-step step="4">SKU属性</v-stepper-step>
      </v-stepper-header>

      <v-stepper-items>

        <v-stepper-content step="1">
          <v-layout row wrap>
            <v-flex md5>
              <cascader
                url="/item/category/list"
                required
                showAllLevels
                v-model="goods.categories"
                label="请选择商品分类"/>
            </v-flex>
            <v-spacer></v-spacer>

            <v-flex md5>
              <v-select :items="brandOptions"
                        item-text="name"
                        item-value="id"
                        label="所属品牌"
                        v-model="goods.brandId"
                        required
                        autocomplete
                        clearable
              ></v-select>
            </v-flex>

          </v-layout>
          <v-layout row wrap>
            <v-text-field label="商品标题" v-model="goods.title" :cpunter="200" required></v-text-field>
          </v-layout>
          <v-layout row wrap>
            <v-text-field label="商品卖点" v-model="goods.subTitle" :cpunter="200"></v-text-field>
          </v-layout>
          <v-layout row wrap>
            <v-text-field label="包装清单" v-model="goods.spuDetail.packingList" :cpunter="1000" multi-line
                          :row="3"></v-text-field>
          </v-layout>

          <v-layout row wrap>
            <v-text-field label="售后服务" v-model="goods.spuDetail.afterService" :cpunter="1000" multi-line
                          :row="3"></v-text-field>
          </v-layout>
        </v-stepper-content>

        <v-stepper-content step="2">
          <!--upload-url：是图片上传的路径-->
          <!--v-model：双向绑定，将富文本编辑器的内容绑定到goods.spuDetail.description-->
          <!--<quill-editor v-model="goods.spuDetail.description" :options="editorOption"/>-->
          <!--属性传递如果数据存在与date中，需要用：表示v-bind注入参数，并不是表示加了：表示属性传递，
          只要给组件中的属性附了值，组件都可在props中进行接受-->
          <v-editor v-model="goods.spuDetail.description" upload-url="/upload/image"/>
        </v-stepper-content>

        <v-stepper-content step="3">
          <v-flex class="xs10 mx-auto px-3">
            <!--遍历每一个规格参数，获取每一组-->
            <v-card v-for="spec in specifications" :key="spec.group" class="my-2">
              <!--每组标题-->
              <v-card-title class="shubheading">{{spec.group}}</v-card-title>
              <!--遍历数组中的每一个属性，并判断是否全局属性，不是则不显示-->
              <v-card-text v-for="param in spec.params" :key="param.k" v-if="param.global" class="=px-5">

                <!--判断是否有可选项，没有显示文本框。判断是否是数字类型，是的话将unit显示到最后-->
                <v-text-field v-if="param.options.length<=0 " :label="param.k"
                              v-model="param.v" :suffix="param.unit || ''"/>
                <v-select v-else :label="param.k" v-model="param.v" :items="param.options"/>
              </v-card-text>
            </v-card>
          </v-flex>
        </v-stepper-content>

        <v-stepper-content step="4">
          <v-flex class="mx-auto">
            <!--特有参数的标题-->
            <v-card flat v-for="spec in specialSpecs" :key="spec.k">
              <!--特有参数标题-->
              <v-card-title class="subheading">{{spec.k}}</v-card-title>
              <!--特有参数的待选项，需要判断是否有options-->
              <v-card-text v-if="spec.options.length<=0" class="px-5">
                <div v-for="i in spec.selected.length+1" :key="i" class="layout row" >
                  <v-text-field :label="'输入新的'+spec.k" class="flex xs8" v-model="spec.selected[i-1]" :value="i" hide-details/>
                <v-btn small @click="spec.selected.splice(i-1,1)">删除</v-btn>
                </div>
              </v-card-text>
              <!--如果有options，需要展示成多个checkbox-->
              <v-card-text v-else class="container fluid grid-list-xs">
                <v-layout row wrap>
                  <v-flex xs3 v-for="o in spec.options" :key="o" >
                    <v-checkbox color="primary" :label="o" v-model="spec.selected" :value="o"/>
                  </v-flex>
                </v-layout>
              </v-card-text>
            </v-card>

            <v-card>
              <v-card-title class="subheading">SKU列表</v-card-title>
              <!--SKU表格，hide-actions因此分页等工具条-->
              <v-data-table :items="skus" :headers="headers" hide-actions
                            item-key="indexes">

                <template slot="items" slot-scope="props">
                    <tr @click="props.expanded = !props.expanded">
                      <!--价格和库存展示为文本框-->
                      <td v-for="(v,k) in props.item" :key="k" v-if="['price', 'stock'].includes(k)"
                          class="text-xs-center">
                        <v-text-field single-line v-model.number="props.item[k]"/>
                      </td>
                      <!--enable展示为checkbox-->
                      <td class="text-xs-center" v-else-if="k === 'enable'">
                        <v-checkbox v-model="props.item[k]"/>
                      </td>
                      <!--indexes和images不展示，其它展示为普通文本-->
                      <td class="text-xs-center" v-else-if="!['indexes','images'].includes(k)">{{v}}</td>
                    </tr>
                </template>
                <template slot="expand" slot-scope="props">
                  <v-card flat>
                    <v-card-text class="title">
                      <v-upload
                        v-model="props.item.images" url="/upload/image" :multiple="true" :pic-width="250" :pic-height="90"
                      />
                    </v-card-text>
                  </v-card>
                </template>
              </v-data-table>
            </v-card>
          </v-flex>
          <!--提交按钮-->
          <v-flex xs3 offset-xs9>
            <v-btn color="info" @click="submit">保存商品信息</v-btn>
          </v-flex>
        </v-stepper-content>

      </v-stepper-items>
    </v-stepper>
  </v-card>
</template>

<script>
  import Cascader from "../../components/cascader/Cascader";
  import 'quill/dist/quill.core.css'
  import 'quill/dist/quill.snow.css'
  import 'quill/dist/quill.bubble.css'
  import {quillEditor} from 'vue-quill-editor'

  export default {
    name: "MyGoodsForm",
    props: {    //  父组件传递过来的参数
      oldGoods: {
        type: Object
      },
      isEdit: {
        type: Boolean,
        default: false
      },
      step: {
        type: Number,
        default: 1
      }
    },
    components: {
      cascader: Cascader,
      quillEditor: quillEditor
    },
    data() {
      return {
        categories: [],
        isShow: false,
        goods: {
          categories: [],  //商品三级分类数组信息
          brandId: 0,  //品牌id
          title: '', //标题
          subTitle: '', //子标题
          spuDetail: {
            packingList: '',  // 包装列表
            afterService: '',  // 售后服务
            description:''
          },
        },
        brandOptions: [],  //品牌列表
        specifications: [],  //商品规格的参数模板，用来显示参数使用，每次类别变化，进行搜索
        specialSpecs: [],  //特有规格参数的模板
      }
    },
    methods: {
      showDialog() {
        this.isShow = true
      },
      submit(){
        //表单校验
        //处理goods用结构表达式接受，除了categories外，都接收到goodsparams中
        const {categories: [{id:cid1},{id:cid2},{id:cid3}], ...goodsParams} = this.goods;
        // 处理规格参数
        const specs = this.specifications.map(({group,params}) => {
          const newParams = params.map(({options,...rest}) => {
            return rest;
          })
          return {group,params:newParams};
        });
        // 处理特有规格参数模板
        const specTemplate = {};
        this.specialSpecs.forEach(({k, selected}) => {
          specTemplate[k] = selected;
        });
        // 处理sku
        const skus = this.skus.filter(s => s.enable).map(({price,stock,enable,images,indexes, ...rest}) => {
          // 标题，在spu的title基础上，拼接特有规格属性值
          const title = goodsParams.title + " " + Object.values(rest).join(" ");
          return {
            price: this.$format(price+""),stock,enable,indexes,title,// 基本属性
            images: !images ? '' : images.join(","), // 图片
            ownSpec: JSON.stringify(rest), // 特有规格参数
          }
        });
        Object.assign(goodsParams, {
          cid1,cid2,cid3, // 商品分类
          skus, // sku列表
        })
        goodsParams.spuDetail.specifications= JSON.stringify(specs);
        goodsParams.spuDetail.specTemplate = JSON.stringify(specTemplate);

        console.log(goodsParams)
      }
    },
    watch: {
      "goods.categories": {
        deep: true,
        //判断商品是否存在，存在才查询
        handler(val) {
          if (val.length === 3) {
            //根据商品分类查询品牌
            this.$http.get("/item/brand/cid/" + this.goods.categories[2].id).then(({data}) => {
              this.brandOptions = data;
            })
            this.$http.get("/item/spec/" + this.goods.categories[2].id).then(({data}) => {
              //保存全部规格
              this.specifications = data;
              //对特有规格进行筛选
              const temp = [];
              data.forEach(({params}) => {
                params.forEach(({k, options, global}) => {
                  if (!global) {
                    temp.push({
                      k, options, selected: []
                    })
                  }
                })
              });
              this.specialSpecs = temp;
            })
          }
        }
      }
    },
    computed:{
     skus(){
       //过滤掉没有填写的参数
       const arr=this.specialSpecs.filter(s=>s.selected.length>0);
       //通过reduce进行笛卡尔积累加
       return arr.reduce((last,spec,index)=>{
         const result=[];
         last.forEach(o=>{
           for (let i = 0; i < spec.selected.length; i++) {
             const option=spec.selected[i];
             const obj={};
             Object.assign(obj,o);
             obj[spec.k]=option;
             //拼接当前这个特有属性的索引
             obj.indexes=(o.indexes || '')+'_'+i

               //如果发现是最后一组，则添加价格，库存等字段
               Object.assign(obj,{price:0, stock:0,enable:false, images:[]})
             obj.indexes = obj.indexes.substring(1);
             result.push(obj);
           }
         })
         return result;
       },[{}])
     },
      headers(){
        if (this.skus.length<=0){
          return[];
        }
        const headers=[];
        //获取sku中任何一个，然后遍历其属性
        Object.keys(this.skus[0]).forEach(k=>{
          let value=k;
          if(k === 'price'){
            // enable，表头要翻译成“价格”
            k = '价格'
          }else if(k === 'stock'){
            // enable，表头要翻译成“库存”
            k = '库存';
          }else if(k === 'enable'){
            // enable，表头要翻译成“是否启用”
            k = '是否启用'
          }else if(k === 'indexes' || k === 'images'){
            // 图片和索引不在表格中展示
            return;
          }
          headers.push({
            text:k,
            align:'center',
            sortable:false,
            value
          })
        })
        return headers;
      }
    },
  }
</script>
<style scoped>

</style>
