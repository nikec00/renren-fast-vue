<!--三级Tree-->
<template>
  <el-tree
    :data="menus"
    :props="defaultProps"
    node-key="catId"
    ref="menuTree"
    @node-click="nodeClick"
  >
  </el-tree>
</template>

<script>
export default {
  data() {
    return {
      expandedKey: [],
      menus: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  //生命周期 - 创建完成（访问当前this实例）
  created() {
    this.getMenus();
  },
  //生命周期 - 挂载完成（访问DOM元素）
  mounted() {},
  methods: {
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.menus = data.data;
        console.log("成功获取数据：", data.data);
      });
    },
    nodeClick(data, node, component) {
      console.log("子组件category的节点被点击：", data, node, component);
      // 向父组件发送事件
      this.$emit("tree-node-click", data, node, component);
    },
  },
};
</script>
<style scoped>
/* @import url(); 引入css类 */
</style>