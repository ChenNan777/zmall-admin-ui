<template>
  <div>
    <el-tree
      :data="menus"
      v-loading="loading"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      draggable
      @node-drop="handleDrop"
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
        categoryDragChanges: [],
        dialogFormVisible: false,
        dialogType: '', //edit append
        dialogTitle: '',
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
        //判断被拖动的节点我以及所在的父节点总层数不能大于3
        let depth = this.countNodeDepth(draggingNode.data);
        if (type == 'inner') {
          return depth + dropNode.data.catLevel <= 3;
        } else {
          if (dropNode.data.catLevel == 1) {
            return depth + 0 <= 3;
          } else {
            return depth + dropNode.parent.data.catLevel <= 3;
          }
        }
      },
      countNodeDepth(node) {
        // 如果节点为 null，返回深度 0
        if (node === null) {
          return 0;
        }
        // 初始化深度为 1，因为当前节点存在
        let depth = 1;
        // 如果有子节点，则递归计算每个子节点的深度
        if (node.children && node.children.length > 0) {
          let maxChildDepth = 0;
          for (let child of node.children) {
            let childDepth = this.countNodeDepth(child);
            if (childDepth > maxChildDepth) {
              maxChildDepth = childDepth;
            }
          }
          // 最大子节点深度加上当前节点深度即为整棵树的深度
          depth += maxChildDepth;
        }
        return depth;
      },
      handleDrop(draggingNode, dropNode, dropType, ev) {
        //拖拽完成后向数据库更新的三个值 parentId catLevel sort
        let parentCid;
        let catLevel;
        //1、parentCId
        //如果dropType为inner，父节点就是dropNode。
        //如果dropType为before或after，父节点就是dropNode的父节点
        //2、catLevel
        //如果dropType为inner，level就是dropNode的level+1
        //如果dropType为before或after，level就是dropNode的level
        //还需要修改子节点的level
        //3、sort
        if (dropType == 'inner') {
          parentCid = dropNode.data.catId;
          catLevel = dropNode.data.catLevel + 1;
          this.categoryDragChanges = dropNode.data.children;
        } else {
          parentCid = dropNode.parent.data.catId == undefined ? 0 : dropNode.parent.data.catId;
          catLevel = dropNode.data.catLevel;
          if (dropNode.data.catLevel == 1) {
            this.categoryDragChanges = this.menus.filter(
              (node) => node.catLevel == 1 || node.catId == draggingNode.data.catId
            );
          } else {
            this.categoryDragChanges = dropNode.parent.data.children;
          }
        }

        //将要修改的分类加入数组
        for (let i = 0; i < this.categoryDragChanges.length; i++) {
          this.categoryDragChanges[i].sort = i;
          if (this.categoryDragChanges[i].catId == draggingNode.data.catId) {
            this.categoryDragChanges[i].parentCid = parentCid;
            if (this.categoryDragChanges[i].catLevel != catLevel) {
              this.updateNodeLevel(this.categoryDragChanges[i], catLevel);
            }
            this.categoryDragChanges[i].catLevel = catLevel;
          }
        }
        console.log(this.categoryDragChanges);
      },
      updateNodeLevel(node, rootLevel) {
        if (!node) {
          return;
        }
        // 设置当前节点的 level
        node.level = rootLevel;
        // 递归处理子节点
        if (node.children.length > 0) {
          for (let child of node.children) {
            this.updateNodeLevel(child, rootLevel + 1);
          }
        }
      }
    },
    created() {},
    mounted() {
      this.getMenus();
    },
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
