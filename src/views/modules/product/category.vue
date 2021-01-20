<!--  -->
<template>
  <div>
    <el-tree
      :data="categories"
      :props="defaultProps"
      :show-checkbox="true"
      :expand-on-click-node="false"
      :default-expanded-keys="expandedKeys"
      node-key="catId"
      :draggable="true"
      :allow-drop="allowDrop"
      ref="tree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
          <el-button type="text" size="mini" @click="() => update(data)">
            Update
          </el-button>
          <el-button
            v-if="data.children.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>

    <!-- 添加与修改分类对话框 开始 -->
    <el-dialog
      :title="dialogTitle"
      :visible.sync="dialogFormVisible"
      :close-on-click-modal="false"
    >
      <el-form :model="newCategory">
        <el-form-item label="分类名称" :label-width="formLabelWidth">
          <el-input v-model="newCategory.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="分类图标" :label-width="formLabelWidth">
          <el-input v-model="newCategory.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" :label-width="formLabelWidth">
          <el-input
            v-model="newCategory.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addOrUpdate">确 定</el-button>
      </div>
    </el-dialog>
    <!-- 添加与修改分类对话框 结束 -->
  </div>
</template>

<script>
// 这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
// 例如：import 《组件名称》 from '《组件路径》';

export default {
  // import引入的组件需要注入到对象中才能使用
  components: {},
  data () {
    return {
      categories: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      },
      expandedKeys: [],
      dialogFormVisible: false,
      newCategory: {
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: '',
        productUnit: ''
      },
      formLabelWidth: '120px',
      dialogTitle: '',
      maxLevel: 0
    }
  },
  methods: {
    // 获取分类信息
    getCategories () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(({ data }) => {
        this.categories = data.data
      })
    },

    // 修改按钮点击事件
    update (data) {
      this.dialogFormVisible = true
      this.dialogTitle = '修改分类'
      // 一定条件下 防止数据脏读, 重新发送get请求
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({ data }) => {
        this.newCategory.name = data.data.name
        this.newCategory.catId = data.data.catId
        this.newCategory.icon = data.data.icon
        this.newCategory.productUnit = data.data.productUnit
        this.newCategory.parentCid = data.data.parentCid
      })
    },

    // 添加按钮点击事件
    append (data) {
      this.newCategory = {
        name: '',
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: '',
        productUnit: ''
      }
      this.dialogFormVisible = true
      this.dialogTitle = '添加分类'
      this.newCategory.parentCid = data.catId
      this.newCategory.catLevel = data.catLevel * 1 + 1 // 需确保转换成数值类型
    },

    // 对话框提交事件, 判断是添加分类还是修改已有分类
    addOrUpdate () {
      if (this.dialogTitle === '添加分类') {
        this.addCategory()
      }
      if (this.dialogTitle === '修改分类') {
        this.updateCategory()
      }
    },

    // 添加分类
    addCategory () {
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.newCategory, false)
      }).then(({ data }) => {
        this.$message({
          message: '已成功添加分类!',
          type: 'success'
        })
        // 关闭对话框
        this.dialogFormVisible = false
        // 重新请求数据
        this.getCategories()
        // 设置默认展开的分类
        this.expandedKeys = [this.newCategory.parentCid]
      })
    },

    // 修改分类s
    updateCategory () {
      // 只解构需要修改的数据
      let { catId, name, icon, productUnit } = this.newCategory
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({ catId, name, icon, productUnit }, false)
      }).then(({ data }) => {
        this.$message({
          message: '已成功修改分类!',
          type: 'success'
        })
        // 关闭对话框
        this.dialogFormVisible = false
        // 重新请求数据
        this.getCategories()
        // 设置默认展开的分类
        this.expandedKeys = [this.newCategory.parentCid]
      })
    },

    // 删除分类按钮点击事件
    remove (node, data) {
      let ids = [data.catId]
      this.$confirm(`是否确定删除【${data.name}】分类`, '提示', {
        confirmButtonText: '删除',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl('/product/category/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({ data }) => {
            this.$message({
              message: '已成功删除该分类',
              type: 'success'
            })
            // 1.刷新页面
            this.getCategories()
            // 2.打开删除前打开的分类位置
            this.expandedKeys = [node.parent.data.catId]
          })
        })
        .catch(() => {
          this.$message({
            message: '删除操作已取消',
            type: 'information'
          })
        })
    },

    allowDrop (draggingNode, dropNode, type) {
      // 1.被拖动的当前节点以及目标节点相加的总层数不能大于3

      // 1).被拖动的当前节点的总层数
      this.countNodeLevel(draggingNode.data)
      //  当前正在拖动的节点+父节点所在的深度不大于3即可
      let deep = this.maxLevel - draggingNode.data.catLevel + 1

      if (type == 'inner') {
        return deep + dropNode.level <= 3
      } else {
        return deep + dropNode.parent.level <= 3
      }
    },

    countNodeLevel (node) {
      // 找到所有子节点, 求出最大深度
      if (node.children != null && node.children.length > 0) {
        for (let i = 0; i < node.children.length; i++) {
          if (node.children[i].catLevel > this.maxLevel) {
            this.maxLevel = node.children[i].catLevel
          }
          // 递归调用  继续查找
          this.countNodeLevel(node.children[i])
        }
      }
    }
  },
  // 监听属性 类似于data概念
  computed: {},
  // 监控data中的数据变化
  watch: {},
  // 生命周期 - 创建完成（可以访问当前this实例）
  created () {
    this.getCategories()
  },
  // 生命周期 - 挂载完成（可以访问DOM元素）
  mounted () {},
  beforeCreate () {}, // 生命周期 - 创建之前
  beforeMount () {}, // 生命周期 - 挂载之前
  beforeUpdate () {}, // 生命周期 - 更新之前
  updated () {}, // 生命周期 - 更新之后
  beforeDestroy () {}, // 生命周期 - 销毁之前
  destroyed () {}, // 生命周期 - 销毁完成
  activated () {} // 如果页面有keep-alive缓存功能，这个函数会触发
}
</script>
<style scoped>
/* @import url(); 引入公共css类 */
</style>
