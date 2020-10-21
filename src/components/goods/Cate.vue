<template>
  <div class="categories">
    <!-- 头部面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 头部面包屑导航结束 -->

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 添加分类按钮 -->
      <el-button type="primary" @click="showAddCateForm()">添加分类</el-button>
      <!-- 添加分类按钮结束 -->

      <!-- 表格 -->
      <tree-table
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        show-index
        :expand-type="false"
        border
        :show-row-hover="false"
      >
        <template slot="isok" slot-scope="scope">
          <i v-if="scope.row.cat_deleted === false" class="el-icon-success"></i>
          <i v-else class="el-icon-error"></i>
        </template>
        <template slot="order" slot-scope="scope">
          <el-tag v-if="scope.row.cat_level === 0" type="primary" size="mini"
            >一级</el-tag
          >
          <el-tag
            v-else-if="scope.row.cat_level === 1"
            type="success"
            size="mini"
            >二级</el-tag
          >
          <el-tag v-else type="warning" size="mini">三级</el-tag>
        </template>
        <template slot="opt" slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" size="mini"
            >编辑</el-button
          >
          <el-button type="danger" icon="el-icon-delete" size="mini"
            >删除</el-button
          >
        </template>
      </tree-table>
      <!-- 表格结束 -->

      <!-- 分页 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagesnum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
      <!-- 分页结束 -->
    </el-card>
    <!-- 卡片视图区域结束 -->

    <!-- 添加分类表单 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateFormVisible"
      width="50%"
      @close="addCateFormClosed()"
    >
      <el-form
        :model="addCateForm"
        :rules="addCateRules"
        ref="addCateFormRef"
        label-width="80px"
      >
        <el-form-item label="分类名称" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类">
          <el-cascader 
            :options="parentCateList"
            :props="cascaderProps"
            v-model="selectKeys"
            @change="parentCateChange()"
            clearable
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate()">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
 
<script>
export default {
  name: "vueName",
  data() {
    return {
      // 获取商品分类列表参数
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5,
      },
      // 商品分类列表数据
      cateList: [],
      // 父级分类列表数据
      parentCateList: [],
      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: "分类名称",
          prop: "cat_name",
        },
        {
          label: "是否有效",
          type: "template",
          template: "isok",
        },
        {
          label: "排序",
          type: "template",
          template: "order",
        },
        {
          label: "操作",
          type: "template",
          template: "opt",
        },
      ],
      // 控制添加分类表单的显示与隐藏
      addCateFormVisible: false,
      // 添加分类表单数据对象
      addCateForm: {
        // 分类名称
        cat_name: "",
        // 父级分类id
        cat_pid: 0,
        // 分类等级
        cat_level: 0,
      },
      // 添加分类表单规则
      addCateRules: {
        cat_name: [
          { required: true, message: "请输入分类名称", trigger: "blur" },
        ],
      },
      // 制定级联选择器的配置对象
      cascaderProps: {
        value: "cat_id",
        label: "cat_name",
        children: "children",
        expandTrigger: "hover",
        checkStrictly: true
      },
      // 选中父级分类的id数组
      selectKeys: []
    };
  },
  created() {
    this.getCateList();
  },
  methods: {
    // 获取商品列表数据
    async getCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品列表失败！");
      }
      this.cateList = res.data.result;
      this.total = res.data.total;
    },
    // 分页每页条数改变时触发
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize;
      this.getCateList();
    },
    // 当前页数（pagenum）改变时触发
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getCateList();
    },
    // 显示添加分类表单
    showAddCateForm() {
      this.getParentCateList();
      this.addCateFormVisible = true;
    },
    // 关闭添加分类表单
    addCateFormClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectKeys = []
      this.addCateForm.cat_pid = 0
      this.addCateForm.cat_level = 0
    },
    // 添加分类表单确定
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if(!valid) return
        const { data : res } = await this.$http.post('categories',this.addCateForm)
        console.log(res)
        if(res.meta.status !== 201){
          return this.$message.error("添加分类失败！")
        }
        this.$message.success("添加分类成功！")
        this.getCateList()
        this.addCateFormVisible = false
      })
    },
    // 获取父级分类数据
    async getParentCateList() {
      const { data : res } = await this.$http.get("categories", {
        params: { type: 2 },
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取父级分类数据失败！");
      }
      this.parentCateList = res.data;
    },
    //选择项变化触发函数
    parentCateChange(){
      if(this.selectKeys.length > 0){
        this.addCateForm.cat_pid = this.selectKeys[this.selectKeys.length-1]
        this.addCateForm.cat_level = this.selectKeys.length 
        return
      }else{
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    }
  },
};
</script>
 
<style scoped lang = "less">
.zk-table {
  margin-top: 15px;
}
.el-icon-success {
  color: lightgreen;
}
.el-icon-error {
  color: red;
}
.el-cascader{
  width: 100%;
}
</style>