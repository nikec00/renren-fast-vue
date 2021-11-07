<!-- category -->
<template>
  <div class="category">
    <el-button type="danger" size="small" @click="batchDel">批量删除</el-button>
    <el-tree
      :data="menus"
      show-checkbox
      :default-expanded-keys="expandedKey"
      node-key="catId"
      :props="defaultProps"
      :expand-on-click-node="false"
      ref="menuTree"
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
          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
          </el-button>
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :before-close="handleClose"
    >
      <el-form :model="category">
        <el-form-item label="分类名称" :label-width="formLabelWidth">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标" :label-width="formLabelWidth">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" :label-width="formLabelWidth">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      expandedKey: [],
      menus: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
      dialogVisible: false,
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        catId: null,
        icon: "",
        productUnit: "",
      },
      formLabelWidth: "100px",
      title: "",
      dialogType: "",
      isClose: false,
    };
  },
  methods: {
    handleNodeClick(data) {
      console.log(data);
    },
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        this.menus = data.data;
        console.log("成功获取数据：", data.data);
      });
    },
    edit(data) {
      this.dialogType = "edit";
      this.dialogVisible = true;
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
      });
      this.title = "修改分类";
    },
    append(data) {
      this.category.name = null;
      this.dialogType = "add";
      this.dialogVisible = true;
      this.category.catId = null;
      this.category.icon = null;
      this.category.productUnit = null;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.title = "添加分类";
      console.log("添加数据：", data);
    },
    batchDel() {
      let checkedNodes = this.$refs.menuTree.getCheckedNodes().map((ele) => ele.catId);
      console.log(checkedNodes);
      this.$confirm(`是否批量删除菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(checkedNodes, false),
          }).then(({ data }) => {
            console.log("删除成功：", checkedNodes);
            this.$message.success("菜单删除成功");
            // 刷新
            this.getMenus();
            // 设置需要默认展开的节点
            this.expandedKey = [node.parent.data.catId];
          });
          console.log("remove", node, data);
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    submitData() {
      if (this.dialogType === "add") {
        this.addCategory();
      }
      if (this.dialogType === "edit") {
        this.editCategory();
      }
    },
    editCategory() {
      var { catId, name, icon, productUnit } = this.category;
      var data = {
        catId: catId,
        name: name,
        icon: icon,
        productUnit: productUnit,
      };
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData(data, false),
      }).then(({ data }) => {
        this.$message.success({
          message: "菜单保存成功",
          type: "success",
        });
        // 刷新
        this.getMenus();
        // 设置需要默认展开的节点
        this.expandedKey = [this.category.parentCid];
        this.dialogVisible = false;
      });
    },
    addCategory() {
      console.log("提交的三级分类数据", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message.success({
          message: "菜单保存成功",
          type: "success",
        });
        // 刷新
        this.getMenus();
        // 设置需要默认展开的节点
        this.expandedKey = [this.category.parentCid];
        this.dialogVisible = false;
      });
    },
    remove(node, data) {
      var ids = [data.catId];
      this.$confirm(`是否删除【${data.name}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            console.log("删除成功：", ids);
            this.$message.success("菜单删除成功");
            // 刷新
            this.getMenus();
            // 设置需要默认展开的节点
            this.expandedKey = [node.parent.data.catId];
          });
          console.log("remove", node, data);
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    handleClose(done) {
      this.$confirm("确认关闭？")
        .then((_) => {
          done();
        })
        .catch((_) => {});
    },
    allowDrop(draggingNode, dropNode, type) {
      // 被拖动的当前节点以及所在的总层数不能打于3
      console.log("拖动：", draggingNode, dropNode, type);
    },
  },
  created() {
    this.getMenus();
  },
  computed: {},
  watch: {},
  mounted() {},
};
</script>

<style lang="scss" scoped></style>