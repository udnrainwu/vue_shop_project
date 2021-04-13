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
          <el-button 
            type="primary" 
            size="default"
            @click="showAddCateDialog"
          >新增分類</el-button>
        </el-col>
      </el-row>
      <tree-table
        class="treeTable"
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
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page.sync="queryInfo.pagenum"
        :page-sizes="[5, 10, 20, 40]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total" 
      />      
    </el-card>
    <el-dialog
      title="新增分類"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogColosed"
    >
      <el-form :model="addCateForm" ref="addCateFormRef" :rules="addCateFormRules" label-width="80px">
        <el-form-item label="分類名稱">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父級分類">
          <el-cascader 
            v-model="selectedKeys" 
            :options="parentCateList" 
            :props="cascaderProps" 
            :show-all-levels="false"
            clearable   
            filterable 
            @change="parentCateChanged"
          >
          </el-cascader>
          
        </el-form-item>
      </el-form>
      
      <span slot="footer">
        <el-button @click=" addCateDialogVisible = false">Cancel</el-button>
        <el-button type="primary" @click="addCate()">OK</el-button>
      </span>
    </el-dialog>
    
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 指定級聯選擇器的 object
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
      },
      parentCateList: [],
      addCateForm: {
        cat_name: '',
        cat_pid: 0,
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: [
          {required: true, message: '請輸入名稱', trigger:'blur'}
        ]
      },
      addCateDialogVisible: false,
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
      ],
      selectedKeys: []
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async addCate() {
      this.$refs.addCateFormRef.validate( async v => {
        if(!v) return 
        const {data:res} = await this.$http.post('categories', this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('新增失敗')
        }
        this.$message.success('新增分類成功')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    addCateDialogColosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_name = ''
    },
    async getParentCateList() {
      const {data:res} = await this.$http.get('categories', {params: {
        type: 2
      }})
      if (res.meta.status !== 200) return this.$message.error('獲取父類分級失敗')
      this.parentCateList = res.data

    },
    showAddCateDialog() {
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    async getCateList() {
      const {data: res} = await this.$http.get('categories', {params:this.queryInfo})
      if (res.meta.status !== 200) return this.$message.error('獲取資料失敗')
      this.catelist = res.data.result
      this.total = res.data.total
    }, parentCateChanged() {
      if (this.selectedKeys.length > 0) {
        this.addCateForm.cat_pid =  this.selectedKeys[this.selectedKeys.length-1]
        this.addCateForm.cat_level = this.selectedKeys.length
        return
      } else {
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }


    }
  },
}
</script>

<style lang="less" socpe>
.treeTable {
  margin-top: 15px;
  margin-bottom: 15px;
}
</style>