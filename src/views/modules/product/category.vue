<template>
  <div>
    <el-tree
      :data="menus"
      v-loading="loading"
      :props="defaultProps"
      accordion
      :expand-on-click-node="false"
      show-checkbox
      draggable
      :allow-drop="allowDrop"
      node-key="catId"
      :default-expanded-keys="expanded">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">
            Append
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">Edit</el-button>
          <el-button
            v-if="node.isLeaf == true"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog :title="dialogTitle" :visible.sync="dialogFormVisible" :close-on-click-modal="false">
      <el-form :model="category" :rules="rules" ref="categoryForm">
        <el-form-item label="分类名称" prop="name">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标" prop="icon">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位" prop="productUnit">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button
          v-if="this.dialogType == 'append'"
          type="primary"
          @click="addCategory('categoryForm')">
          确 定
        </el-button>
        <el-button
          v-else-if="this.dialogType == 'edit'"
          type="primary"
          @click="editCategory('categoryForm')">
          确 定
        </el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    components: {},
    props: {},
    data() {
      return {
        category: {
          name: '',
          parentCid: 0,
          catLevel: 0,
          showStatus: 1,
          sort: 0,
          icon: '',
          productUnit: '',
          catId: null
        },
        rules: { name: [{ required: true, message: '请输入分类名称', trigger: 'blur' }] },
        menus: [],
        expanded: [],
        dialogFormVisible: false,
        dialogType: '', //edit append
        dialogTitle: '',
        maxLevel: 0,
        loading: true,
        defaultProps: {
          children: 'children',
          label: 'name'
        }
      };
    },
    computed: {},
    watch: {
      dialogFormVisible(newVal) {
        if (newVal === false) {
          this.category.name = '';
        }
      }
    },
    methods: {
      getMenus() {
        this.$http({
          url: this.$http.adornUrl('/product/category/list/tree'),
          method: 'get'
        }).then(({ data }) => {
          console.log('获取菜单数据成功...', data.data);
          this.menus = data.data;
          this.loading = false;
        });
      },
      append(data) {
        this.dialogFormVisible = true;
        this.dialogType = 'append';
        this.dialogTitle = '添加分类';
        this.category.parentCid = data.catId;
        this.category.catLevel = data.catLevel * 1 + 1;
        this.category.catId = null;
        this.category.name = '';
        this.category.icon = '';
        this.category.productUnit = '';
      },
      addCategory(formName) {
        this.$refs[formName].validate((validate) => {
          if (validate) {
            this.$http({
              url: this.$http.adornUrl('/product/category/save'),
              method: 'post',
              data: this.$http.adornData(this.category, false)
            }).then(({ data }) => {
              this.dialogFormVisible = false;
              this.$message({
                type: 'success',
                message: '分类添加成功'
              });
              //刷新菜单
              this.getMenus();
              //设置默认展开菜单
              this.expanded = [this.category.parentCid];
              console.log('分类添加成功...');
            });
          } else {
            return false;
          }
        });
      },
      edit(data) {
        this.dialogFormVisible = true;
        this.dialogType = 'edit';
        this.dialogTitle = '修改分类';
        //发送请求从后端获取当前节点的最新数据
        this.$http({
          url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
          method: 'get'
        }).then(({ data }) => {
          this.category.catId = data.category.catId;
          this.category.name = data.category.name;
          this.category.icon = data.category.icon;
          this.category.productUnit = data.category.productUnit;
          this.category.parentCid = data.category.parentCid;
        });
      },
      editCategory(formName) {
        this.$refs[formName].validate((validate) => {
          if (validate) {
            let { catId, name, icon, productUnit } = this.category;
            this.$http({
              url: this.$http.adornUrl('/product/category/update'),
              method: 'post',
              data: this.$http.adornData({ catId, name, icon, productUnit }, false)
            }).then(({ data }) => {
              this.dialogFormVisible = false;
              this.$message({
                type: 'success',
                message: '分类修改成功'
              });
              //刷新菜单
              this.getMenus();
              //设置默认展开菜单
              this.expanded = [this.category.parentCid];
              console.log('分类修改成功...');
            });
          } else {
            return false;
          }
        });
      },
      remove(node, data) {
        this.$confirm(`此操作将删除 ${data.name} 分类, 是否继续?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
          .then(() => {
            var ids = [data.catId];
            this.$http({
              url: this.$http.adornUrl('/product/category/delete'),
              method: 'post',
              data: this.$http.adornData(ids, false)
            }).then(({ data }) => {
              this.$message({
                type: 'success',
                message: '分类删除成功'
              });
              //刷新菜单
              this.getMenus();
              //设置默认展开菜单
              this.expanded = [node.parent.data.catId];
              console.log('分类删除成功...');
            });
          })
          .catch(() => {
            this.$message({
              type: 'info',
              message: '已取消删除'
            });
          });
      },
      allowDrop(draggingNode, dropNode, type) {
        //1.判断被拖动的节点我以及所在的父节点总册数不能大于3
        //1)计算被拖动节点的总层数
        this.countNodeLevel(draggingNode.data);
        let deep = this.maxLevel - draggingNode.data.level + 1;
        console.log(this.maxLevel);
        return false;
      },
      countNodeLevel(node) {
        //DFS
        if (node.children != null && node.children.lenght > 0) {
          for (let i = 0; i < node.children.lenght; i++) {
            if (node.children[i].catLevel > maxLevel) {
              this.maxLevel = node.children[i].level;
            }
            this.countNodeLevel(children);
          }
        }
      }
    },
    created() {
      this.getMenus();
    },
    mounted() {},
    beforeCreate() {},
    beforeMount() {},
    beforeUpdate() {},
    updated() {},
    beforeDestroy() {},
    destroyed() {},
    activated() {}
  };
</script>
<style lang="scss" scoped></style>
