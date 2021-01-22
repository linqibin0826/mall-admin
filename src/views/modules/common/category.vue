<template>
  <div>
    <el-input placeholder="输入关键字进行过滤" v-model="filterText"></el-input>
    <el-tree
      :data="menus"
      :props="defaultProps"
      node-key="catId"
      ref="menuTree"
      @node-click="nodeClick"
      :filter-node-method="filterNode"
      :highlight-current="true"
    ></el-tree>
  </div>
</template>

<script>
/*
* 父子组件传递数据
* 1).子组件给父组件传递数据, 事件机制
*   this.$emit("事件名称", 想要携带的数据...)
* 2).父组件中使用这个组件,然后自定义一个事件来接收这些数据.
*   <category @tree-node-click="treenodeclick"></category>
* */
export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  name: "category",
  props: {},
  data () {
    //这里存放数据
    return {
      filterText: '',
      menus: [],
      expandedKey: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  //计算属性类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {
    filterText (val) {
      this.$refs.menuTree.filter(val)
    }
  },
  //方法集合
  methods: {
    //树节点过滤
    filterNode (value, data) {
      if (!value) return true
      return data.name.indexOf(value) !== -1
    },
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({data}) => {
        this.menus = data.data
      })
    },
    nodeClick (data, node, component) {
      //向父组件发送事件；
      this.$emit('tree-node-click', data, node, component)
    }
  },
  //生命周期创建完成（可以访问当前this实例）
  created () {
    this.getMenus()
  }
}
</script>
<style scoped>

</style>
