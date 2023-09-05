<template>
  <div>
    <el-tree
      :data="menus"
      v-loading="loading"
      :props="defaultProps"
      accordion
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId">
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button v-if="node.level <= 2" type="text" size="mini" @click="() => append(data)">
            Append
          </el-button>
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
    <el-dialog title="提示" :visible.sync="centerDialogVisible" width="30%" center>
      <span>需要注意的是内容是默认不居中的</span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="centerDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="centerDialogVisible = false">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>


<script>
  export default {
    components: {},
    props: {},
    data() {
      return {
        menus: [],
        loading: true,
        defaultProps: {
          children: 'children',
          label: 'name'
        }
      };
    },
    computed: {},
    watch: {},
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
        console.log(data);
      },
      remove(node, data) {
        console.log(node, data);
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
<style scoped>
</style>