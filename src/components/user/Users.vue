<template>
  <div class="users">
    <!-- 头部面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 头部面包屑导航结束 -->

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索与添加区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getUserList()"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUserList()"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="dialogVisible = true"
            >添加用户</el-button
          >
        </el-col>
      </el-row>
      <!-- 搜索与添加区域结束 -->

      <!-- 用户与列表区域 -->
      <el-table :data="userList" border :stripe="true">
        <el-table-column type="index" label="序号"></el-table-column>
        <el-table-column prop="username" label="姓名"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="mobile" label="电话"></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column prop="mg_state" label="状态">
          <template slot-scope="scope">
            <el-switch v-model="scope.row.mg_state" @change="userStateChanged(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button @click="showEditForm(scope.row.id)" type="primary" icon="el-icon-edit" size="mini"></el-button>
            <el-button @click="deleteUser(scope.row.id)" type="danger" icon="el-icon-delete" size="mini"></el-button>
            <el-tooltip effect="dark" content="分配角色" placement="top" :enterable="false">
              <el-button @click="showSetRoleForm(scope.row)" type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 用户与列表区域结束 -->

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

      <!-- 添加用户对话框 -->
      <el-dialog title="添加用户" :visible.sync="dialogVisible" width="50%" @close="userFormClosed()">
        <el-form :model="addUserForm" :rules="addUserRules" ref="addRuleFormRef" label-width="70px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addUserForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addUserForm.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addUserForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机" prop="mobile">
            <el-input v-model="addUserForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser()">确 定</el-button
          >
        </span>
      </el-dialog>
      <!-- 添加用户对话框结束 -->

      <!-- 修改用户信息对话框 -->
      <el-dialog title="修改用户信息" :visible.sync="editFormVisible" width="50%">
        <el-form :model="editUserForm" :rules="editUserRules" ref="editRuleFormRef" label-width="70px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="editUserForm.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editUserForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机" prop="mobile">
            <el-input v-model="editUserForm.mobile"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUserInfo()">确 定</el-button
          >
        </span>
      </el-dialog>
      <!-- 修改用户信息对话框结束 -->

       <!-- 分配角色对话框 -->
      <el-dialog title="分配角色" :visible.sync="setRoleFormVisible" width="50%" @close="setRoleFormClosed()">
        <div>
          <p>当前的用户：{{userInfo.username}}</p>
          <p>当前的角色：{{userInfo.role_name}}</p>
          <p>分配新角色：
            <el-select v-model="selectRoleId" placeholder="请选择">
              <el-option v-for="item in roleList" :key="item.id" :label="item.roleName"
                :value="item.id">
              </el-option>
            </el-select>
          </p>
        </div>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRoleFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="setRole()">确 定</el-button
          >
        </span>
      </el-dialog>
      <!-- 分配角色对话框结束 -->
    </el-card>
    <!-- 卡片视图区域结束 -->
  </div>
</template>

<script>
export default {
  name: "vueName",
  data() {
    // 验证手机号规则
    var checkPhone = (rule, value, callback) => {
      const regMobile = /^(0|86|17951)?(13[0-9]|15[0123456789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if(regMobile.test(value)){
        return callback()
      }
      callback("请输入正确的手机号码")
    };
    return {
      // 获取用户列表参数对象
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 2,
      },
      userList: [],
      total: 0,
      // 点击分配角色按钮获取对应用户信息
      userInfo: {},
      // 所有角色的数据列表
      roleList: [],
      // 已选中的角色id值
      selectRoleId: "",
      // 控制添加用户对话框的显示与隐藏
      dialogVisible: false,
      // 控制修改用户信息表单的显示与隐藏
      editFormVisible: false,
      // 控制分配角色对话框的显示与隐藏
      setRoleFormVisible: false,
      // 添加用户表单数据
      addUserForm: {
        username: "",
        password: "",
        email: "",
        mobile: "",
      },
      // 查询到的用户表单信息
      editUserForm: {},
      // 添加用户表单验证规则对象
      addUserRules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { type: 'email', message: '请输入正确的邮箱地址', trigger: ['blur', 'change'] }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkPhone, trigger: 'blur' }
        ],
      },
      // 修改用户信息表单验证规则
      editUserRules: {
        email: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { type: 'email', message: '请输入正确的邮箱地址', trigger: ['blur', 'change'] }
        ],
        mobile: [
          { required: true, message: '请输入手机号码', trigger: 'blur' },
          { validator: checkPhone, trigger: 'blur' }
        ],
      },
    };
  },
  created() {
    this.getUserList();
  },
  methods: {
    async getUserList() {
      var { data: res } = await this.$http.get("users", {
        params: this.queryInfo,
      });
      if (res.meta.status !== 200) {
        return this.$message.error("获取用户信息失败!");
      }
      this.userList = res.data.users;
      this.total = res.data.total;
    },
    async userStateChanged(userinfo) {
      const { data: res } = await this.$http.put(
        `users/${userinfo.id}/state/${userinfo.mg_state}`
      );
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state;
        return this.$message.error("更新状态失败!");
      }
      this.$message.success("更新状态成功!");
    },
    handleSizeChange(newsize) {
      this.queryInfo.pagesize = newsize;
      this.getUserList();
    },

    handleCurrentChange(newpage) {
      this.queryInfo.pagenum = newpage;
      this.getUserList();
    },
    // 用户对话框关闭事件
    userFormClosed(){
      this.$refs.addRuleFormRef.resetFields();
    },
    // 添加用户
    addUser(){
      this.$refs.addRuleFormRef.validate(async val => {
        if(!val) return
        const {data:res} = await this.$http.post("users",this.addUserForm)
        if(res.meta.status !== 201){
          return this.$message.error("添加用户失败!")
        }
        this.$message.success("添加用户成功!")
        this.dialogVisible = false;
        this.getUserList();
      })
    },
    // 修改用户信息
    async showEditForm(id){
      const { data : res } = await this.$http.get("users/" + id)
      if(res.meta.status !== 200){
        return this.$message.error("修改用户信息失败!")
      }
      this.editUserForm = res.data
      this.editFormVisible = true
    },
    // 修改用户信息表单确认
    editUserInfo(){
      this.$refs.editRuleFormRef.validate(async val => {
        if(!val) return
        const { data : res} = await this.$http.put("users/" + this.editUserForm.id, {
          email: this.editUserForm.email,
          mobile: this.editUserForm.mobile
        })
        if(res.meta.status !== 200){
          return this.$message.error("修改用户信息失败!")
        }
        this.editFormVisible = false;
        this.getUserList();
        this.$message.success("修改用户信息成功!");
      })
    },
    // 删除用户
    async deleteUser(id){
      const res = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if(res !== "confirm"){
        return this.$message.info("已取消删除")
      }
      const { data : deleteRes } = await this.$http.delete("users/" + id)
      if(deleteRes.meta.status !== 200){
        return this.$message.error("删除用户失败！")
      }
      this.$message.success("删除用户成功！");
      this.getUserList();
    },
    // 点击展开分配角色对话框
    async showSetRoleForm(role){
      this.userInfo = role
      // 获取所有角色列表
      const { data : res } = await this.$http.get("roles");
      if(res.meta.status !== 200){
        return this.$message.error("获取角色列表失败！")
      }
      this.roleList = res.data
      this.setRoleFormVisible = true
    },
    // 点击分配角色确定按钮事件
    async setRole(){
      if(!this.selectRoleId){
        return this.$message.error("请选择要分配的角色！")
      }
      const { data : res } = await this.$http.put(`users/${this.userInfo.id}/role`,{rid: this.selectRoleId})
      if(res.meta.status !== 200){
        return this.$message.error("更新角色失败！")
      }
      this.$message.success("更新角色成功！")
      this.getUserList()
      this.setRoleFormVisible = false;
    },
    // 分配角色对话框关闭事件
    setRoleFormClosed(){
      this.selectRoleId = ''
      this.userInfo = {}
    },

  },
};
</script>

<style scoped lang="less"></style>
