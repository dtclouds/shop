<template>
  <div class="params">
    <!-- 头部面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 头部面包屑导航结束 -->

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert
        show-icon
        title="注意：只允许为第三级分类设置相关参数！"
        type="warning"
        :closable="false"
      ></el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col
          >选择商品分类：
          <!-- 商品分类级联样式表 -->
          <el-cascader
            :options="cateList"
            :props="cateProps"
            v-model="selectKeys"
            @change="cateChange()"
          ></el-cascader>
        </el-col>
      </el-row>
      <!-- tab 页签区域 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="showAddForm()"
            >添加参数</el-button
          >
          <el-table :data="manyTableData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  class="tag"
                  v-for="(item,i) in scope.row.attr_vals"
                  :key="item.attr_id"
                  closable
                  @close="handleClose(i,scope.row)"
                  >{{ item }}</el-tag
                >
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index" label="序号"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-deit"
                  size="mini"
                  @click="showEditDialog(scope.row.attr_id)"
                  >编辑</el-button
                >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="removeParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <el-button
            type="primary"
            size="mini"
            :disabled="isBtnDisabled"
            @click="showAddForm()"
            >添加属性</el-button
          >
          <el-table :data="onlyTableData" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag
                  class="tag"
                  v-for="(item,i) in scope.row.attr_vals"
                  :key="item.attr_id"
                  closable
                  @close="handleClose(i,scope.row)"
                  >{{ item }}</el-tag
                >
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                  >+ New Tag</el-button
                >
              </template>
            </el-table-column>
            <el-table-column type="index" label="序号"></el-table-column>
            <el-table-column prop="attr_name" label="属性名称">
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  icon="el-icon-deit"
                  size="mini"
                  @click="showEditDialog(scope.row.attr_id)"
                  >编辑</el-button
                >
                <el-button
                  type="danger"
                  icon="el-icon-delete"
                  size="mini"
                  @click="removeParams(scope.row.attr_id)"
                  >删除</el-button
                >
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>

      <!-- 添加参数/属性对话框 -->
      <el-dialog
        :title="'添加' + addFormTitle"
        :visible.sync="addFormVisible"
        width="50%"
        @close="addFormClosed()"
      >
        <el-form
          :model="addFormInfo"
          :rules="addFormRules"
          ref="addFormRef"
          label-width="100px"
        >
          <el-form-item :label="addFormTitle" prop="attr_name">
            <el-input v-model="addFormInfo.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="addForm()">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 编辑参数/属性对话框 -->
      <el-dialog
        :title="'修改' + addFormTitle"
        :visible.sync="editFormVisible"
        width="50%"
        @close="editFormClosed()"
      >
        <el-form
          :model="editFormInfo"
          :rules="editFormRules"
          ref="editFormRef"
          label-width="100px"
        >
          <el-form-item :label="addFormTitle" prop="attr_name">
            <el-input v-model="editFormInfo.attr_name"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="editForm()">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
    <!-- 卡片视图区域结束 -->
  </div>
</template>
 
<script>
export default {
  name: "vueName",
  data() {
    return {
      // 商品分类数据列表
      cateList: [],
      // 指定级联选择器的匹配对象
      cateProps: {
        value: "cat_id",
        label: "cat_name",
        children: "children",
        expandTrigger: "hover",
      },
      // 级联选择框匹配双向绑定数组
      selectKeys: [],
      // 被激活的页签名称
      activeName: "many",
      // 动态参数数据
      manyTableData: [],
      // 静态属性数据
      onlyTableData: [],
      // 控制添加参数/属性对话框的显示与隐藏
      addFormVisible: false,
      // 控制编辑参数/属性对话框的显示与隐藏
      editFormVisible: false,
      // 添加参数/属性表单数据
      addFormInfo: {
        attr_name: "",
      },
      // 编辑参数/属性表单数据
      editFormInfo: {
        attr_name: "",
      },
      // 添加参数/属性表单数据验证
      addFormRules: {
        attr_name: [
          { required: true, message: "添加信息不能为空!", trigger: "blur" },
        ],
      },
      // 编辑参数/属性表单数据验证
      editFormRules: {
        attr_name: [
          { required: true, message: "添加信息不能为空!", trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.getCateList();
  },
  methods: {
    // 获取商品分类数据
    async getCateList() {
      const { data: res } = await this.$http.get("categories");
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品分类失败！");
      }
      this.cateList = res.data;
    },
    // 级联选择框选项变化时触发函数
    cateChange() {
      this.getParamsCate();
    },
    // tab页签点击
    handleTabClick() {
      this.getParamsCate();
    },
    // 获取商品参数列表
    async getParamsCate() {
      // 保证选择的id等级为三级
      if (this.selectKeys.length !== 3) {
        this.selectKeys = [];
        this.manyTableData = [];
        this.onlyTableData = [];
        return
      }
      // 根据所选id和面板参数渲染列表
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes`,
        {
          params: { sel: this.activeName },
        }
      );
      if (res.meta.status !== 200) {
        return this.$$message.error("获取参数列表失败！");
      }
      res.data.forEach((item) => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(" ") : [];
        item.inputVisible = false
        item.inputValue = ''
      });
      if (this.activeName === "many") {
        return (this.manyTableData = res.data);
      } else {
        this.onlyTableData = res.data;
      }
    },
    // 关闭添加参数/属性对话框
    addFormClosed() {
      this.$refs.addFormRef.resetFields();
    },
    // 显示添加参数/属性对话框
    showAddForm() {
      this.addFormVisible = true;
    },
    // 添加参数/属性确定按钮
    addForm() {
      this.$refs.addFormRef.validate(async (val) => {
        if (!val) return;
        const { data: res } = await this.$http.post(
          `categories/${this.cateId}/attributes`,
          {
            attr_name: this.addFormInfo.attr_name,
            attr_sel: this.activeName,
          }
        );
        if (res.meta.status !== 201) {
          return this.$message.error("添加数据失败！");
        }
        this.$message.success("添加数据成功！");
        this.addFormVisible = false;
        this.getParamsCate();
      });
    },
    // 显示编辑表单的会话框
    async showEditDialog(attr_id) {
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${attr_id}`,
        {
          params: { attr_sel: this.activeName },
        }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("获取参数信息失败！");
      }
      this.editFormInfo = res.data;
      this.editFormVisible = true;
    },
    // 关闭添加参数/属性对话框
    editFormClosed() {
      this.$refs.editFormRef.resetFields();
    },
    // 提交编辑表单列表
    editForm() {
      this.$refs.editFormRef.validate(async (val) => {
        if (!val) return;
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editFormInfo.attr_id}`,
          {
            attr_name: this.editFormInfo.attr_name,
            attr_sel: this.activeName,
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("修改参数失败！");
        }
        this.$message.success("修改参数成功！");
        this.getParamsCate();
        this.editFormVisible = false;
      });
    },
    // 删除属性
    async removeParams(attr_id) {
      const flag = await this.$confirm(
        "此操作将永久删除该属性, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).catch((err) => err);
      if (flag !== "confirm") {
        return this.$message.info("已取消成功！");
      }
      const { data: res } = await this.$http.delete(
        `categories/${this.cateId}/attributes/${attr_id}`
      );
      if (res.meta.status !== 200) {
        return this.$message.error("删除参数失败！");
      }

      this.$message.success("删除参数成功！");
      this.getParamsCate();
    },
    // 文本框失去焦点或者按下enter时触发
    handleInputConfirm(row){
      if(row.inputValue.trim().length === 0){
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      this.saveAttrVals(row)
    },
    // 点击按钮展示文本框
    showInput(row){
      row.inputVisible = true
      // $nextTick作用：当页面元素被重新渲染之后，才会指定回调函数中的代码
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    // 将对attr_vals的操作保存到数据库
    async saveAttrVals(row){
      const { data : res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,{
        attr_name: row.attr_name,
        attr_sel: row.attr_sel,
        attr_vals: row.attr_vals.join(" ")
      })
      if(res.meta.status !== 200){
        return this.$message.error("修改参数项内容失败！")
      } 
      this.$message.success("修改参数项内容成功！")
    },
    // 删除tag标签
    handleClose(i,row){
      row.attr_vals.splice(i,1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    // 是否禁用添加参数/属性的按钮
    isBtnDisabled() {
      if (this.selectKeys.length > 0) {
        return false;
      }
      return true;
    },
    // 当前选中三级分类id
    cateId() {
      if (this.selectKeys.length === 3) {
        return this.selectKeys[2];
      }
      return null;
    },
    // 添加参数/属性对话框的title
    addFormTitle() {
      if (this.activeName === "many") {
        return "动态参数";
      }
      return "静态属性";
    },
  },
};
</script>
 
<style scoped lang = "less">
.cat_opt {
  margin: 15px 0;
}
.input-new-tag{
  width: 120px;
}
.tag{
  margin: 0 15px 0 0;
}
</style>