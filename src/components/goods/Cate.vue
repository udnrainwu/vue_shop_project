<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home}' }">首頁</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分類</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" size="default">新增分類</el-button>
        </el-col>
      </el-row>
      <tree-table
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        :show-row-hover="false"
        show-index
        index-text="#"
        border
      >
        <template slot="isok" slot-scope="scope">
          <i v-if="scope.row.cat_deleted" class="el-icon-success" style="color:green;"/>
          <i v-else class="el-icon-error" style="color:red;"/>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0" size="mini">一級</el-tag>
          <el-tag v-else-if="scope.row.cat_level === 1" type="success" size="mini">二級</el-tag>
          <el-tag v-else type="warning" size="mini">三級</el-tag>
        </template>
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" size="mini" icon="el-icon-edit" @click="">編輯</el-button>
          <el-button type="danger" size="mini" icon="el-icon-delete" @click="">刪除</el-button>
          
        </template>
      </tree-table>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      catelist:[],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      columns: [
        {label:'分類名稱', prop:'cat_name'},
        {label:'是否有效', type: 'template',template:'isok'},
        {label:'排序', type: 'template',template:'order'},
        {label:'操作', type: 'template',template:'opt'},
      ]
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const {data: res} = await this.$http.get('categories', {params:this.queryInfo})
      if (res.meta.status !== 200) return this.$message.error('獲取資料失敗')
      this.catelist = res.data.result
    }
  },
}
</script>

<style>
</style>