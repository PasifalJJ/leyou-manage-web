<template>
  <!--iview的Cascader插件，级联选择框，:data="options"表示渲染的数据
  :load-data="loadOption" 加载数据的调用的方法，
  @on-change="handleChange当数据发生改变是调用handleChange函数-->
  <cas :data="options" :load-data="loadOption" @on-change="handleChange"></cas>

  <!--  &lt;!&ndash;vuetify提供的组件，下拉选择框label标签，选中的选项会议chips的形式包裹
    clearable 添加选择框清楚功能，结尾有一个叉  rulues校验写入的数据的函数&ndash;&gt;


    &lt;!&ndash;插槽最主要的作用是传递标签中的text和其他标签内容提。传统的prop只能传递属性值，不能传递标签中展示的内容&ndash;&gt;
    &lt;!&ndash;vuetify中的v-select组件会在这里渲染，v-select中默认有自己的插槽，容许标签的内容体插入&ndash;&gt;
    <v-select :label="label" chips clearable v-model="select"
              tags :required="required" :rules="defaultRules">
      &lt;!&ndash;slot="selection" 是一个标记符，他会将当前template标签的内容渲染到
      v-select组件中name为selection的slot插槽中，slot-scope表示父组件里面的数据，取名为data，
      可以使用父组件中定义的属性，
      数据能够调用的属性在vuetify中可以查询 parent: VueComponent", item: object", index: number", selected: boolean", disabled: boolean" &ndash;&gt;
      <template slot="selection" slot-scope="data">
        &lt;!&ndash;@click.stop=""组织事件冒泡  @click.prevent阻止事件的默认行为
         v-if与v-else表示判断，只会显示其中的一个标签， close表示添加删除按钮
         @input="remove(data.item) 被绑定模型的更新，删除data.item属性
         outline 删除背景并应用边框和文本颜色
         green 将指定的色彩应用与控件
         &ndash;&gt;
        <v-chip @click.stop="" v-if="multiple" close @input="remove(data.item)"
                small outline color="green">
          {{data.item.label}}&nbsp
        </v-chip>
        <v-chip @click.stop="" v-else small="" color="green">{{data.item}}</v-chip>
      </template>
    </v-select>-->

</template>

<!--suppress ES6CheckImport -->
<script>
  import {Cascader} from 'iview'
  import 'iview/dist/styles/iview.css'

  export default {
    name: "MyCascader",
    components: {    //注册组件
      cas: Cascader
    },
    props: {         //接受父组件通过属性传过来的参数
      value: {},     //
      label: {       //select框的标签
        type: String
      },
      url: {
        type: String //请求数据的访问路径
      },
      itemText: {
        type: String,
        default: 'name'
      },
      itemValue: {
        type: String,
        default: 'id'
      },
      children: {  //  子数据。cas级联下拉框需要使用的数据
        type: String,
        default: 'id'
      },
      multiple: {  //是否显示多个，chips多选时可以使用
        type: Boolean,
        default: false
      },
      showAllLevels: {   //是否显示所有级别
        type: Boolean,
        default: false
      },
      required: {
        type: Boolean,
        default: false
      },
      rules: {
        type: Array,
      }
    },
    data() {
      return {
        options: [{    //下拉框参数条件，如果没有children: []属性，不会出现级联效果
          value: '1',
          label: '北京',
          children: []
        }],
        selected: [],
        defaultRules: []
      }
    },
    methods: {
      //选择完成后的回调，返回值 value 即已选值 value，selectedData 为已选项的具体数据
      handleChange(value, selectedDate) {
        //每次级联数据改变，获取当前数据的最后一级
        const option = selectedDate[selectedDate.length - 1];
        //如果是多选，默认只保存最后一级选项
        if (this.multiple) {
          //将最后一级保存到selecte中，判断当前select中的value是否与option中的value相同,
          //找到相同返回索引，没有相同返回-1，如果找不到，则再赋值一次
          if (this.selected.findIndex((o) => {
            o.value === option.value
          }) < 0) {
            //在select对象末尾添加option，可以统计三个所选
            this.selected.push(option);
          }
          //执行子组件或父组件中的 @input方法，并传递参数 this.selected。???
          this.$emit("input", this.transfer(this.selected))
        } else {
          //只显示最后一级
          this.selected = [option.label];
          //返回
          this.$emit("input", this.transfer([option][0]))
        }
      },
      //cas插件中加载数据，延迟加载次级选项，点击时会加载下一级数据
      loadOption(item, callback) {
        //加载数据时，必须将loading设置为true
        item.loading = this;
        this.loadDate(item.value).then()

      },
      //从指定的url地址加载数据，并格式化
      loadData(pid) {
        return new Promise(resolve => {
          this.$http.get(this.url, {
            params: {
              pid: pid
            }
          }).then(resp => {
            const data = [];
            for (let d of resp.data) {
              const node = {
                value: d[this.itemValue],
                label: d[this.itemText],
              }
              if (d.parent) {
                node['children'] = [];
                node['loading'] = false;
              }
              data.push(node);
            }
            resolve(data);
          }).catch()
        })
      },
      remove(item) {
        this.selected = this.selected.filter(o => o.vakue !== item.value);
        this.$emit("input", this.transfer(this.selected));
      },
      transfer(arr) {
        return arr.map(({label, value}) => {
          const obj = [];
          obj[this.itemText] = label;
          obj[this.itemValue] = value;
          return obj;
        })
      },
      validate() {
        if (this.required) {
          this.$refs.form.validate();
        }
      },
      created() {
        this.loadData(0).then(data => {
          this.options = data;
        })
        if (this.required) {
          this.defaultRules.push(v => v.length > 0 || this.label + "不能为空");
        }
        if (this.rules) {
          this.rules.forEach(r => this.defaultRules.push(r))
        }
      },
      watch: {
        value: {
          deep: true,
          handler(val) {
            if (val && this.showAllLevels && !this.multiple) {
              this.selected = [val.map(o => o[this.itemText].join("/"))]
            } else if (this.multiple && val) {
              this.selected = val.map(o => {
                return {
                  label: 0[this.itemText],
                  value: o[this.itemValue]
                }
              })
            } else {
              this.selected = [val[this.itemText]]
            }
          }
        }
      }
    }
  }
</script>

<style scoped>
  .ivu-cascader-menu-item{
    font-size:14px;
  }
</style>
