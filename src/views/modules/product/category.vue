<template>
  <div>
    <el-tree
      :data="menus"
      v-loading="loading"
      :props="defaultProps"
      accordion
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expanded"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">
            Append
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
          </el-button>
          <el-button
            v-if="node.isLeaf == true"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog title="添加分类" :visible.sync="appendFormVisible">
      <el-form :model="categroy" :rules="rules" ref="appendForm">
        <el-form-item label="分类名称" prop="name">
          <el-input v-model="categroy.name" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="appendFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCategroy('appendForm')">确 定</el-button>
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
      categroy: { name: '', parentCid: 0, catLevel: 0, showStatus: 1, sort: 0, catId: null },
      rules: { name: [{ required: true, message: '请输入分类名称', trigger: 'blur' }] },
      menus: [],
      expanded: [],
      appendFormVisible: false,
      loading: true,
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    };
  },
  computed: {},
  watch: {
    appendFormVisible(newVal) {
      if (newVal === false) {
        this.categroy.name = '';
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
      this.appendFormVisible = true;
      this.categroy.parentCid = data.catId;
      this.categroy.catLevel = data.catLevel * 1 + 1;
    },
    addCategroy(formName) {
      this.$refs[formName].validate((validate) => {
        if (validate) {
          this.$http({
            url: this.$http.adornUrl('/product/category/save'),
            method: 'post',
            data: this.$http.adornData(this.categroy, false)
          }).then(({ data }) => {
            this.appendFormVisible = false;
            this.$message({
              type: 'success',
              message: '分类添加成功'
            });
            //刷新菜单
            this.getMenus();
            //设置默认展开菜单
            this.expanded = [this.categroy.parentCid];
            console.log('分类添加成功...');
          });
        } else {
          return false;
        }
      });
    },
    edit(data) {},
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
