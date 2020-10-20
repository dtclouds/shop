<template>
  <div class="roles">
    <!-- 头部面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 头部面包屑导航结束 -->

    <!-- 卡片式图区域 -->
    <el-card>
      <el-button type="primary" @click="showAddRoleFrom()">添加角色</el-button>
      <!-- 角色列表区域 -->
      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand">
            <template slot-scope="scope">
                <el-row :class="['bdbottom',i1=== 0?'bdtop':null,'vcenter']" v-for="(item1,i1) in scope.row.children" :key="item1.id">
                    <el-col :span="5">
                        <el-tag closable @close="deleteRight(scope.row,item1.id)">{{item1.authName}}</el-tag>
                        <i class="el-icon-caret-right"></i>
                    </el-col>
                    <el-col :span="19">
                        <el-row :class="[i2===0?null:'bdtop','vcenter']" v-for="(item2,i2) in item1.children" :key="item2.id">
                            <el-col :span="6">
                                <el-tag type="success" closable @close="deleteRight(scope.row,item2.id)">{{item2.authName}}</el-tag>
                                <i class="el-icon-caret-right"></i>
                            </el-col>
                            <el-col :span="18">
                                <el-tag type="warning" v-for="item3 in item2.children" :key="item3.id" closable @close="deleteRight(scope.row,item3.id)">{{item3.authName}}</el-tag>
                            </el-col>
                        </el-row>
                    </el-col>
                </el-row>
            </template>
        </el-table-column>
        <el-table-column type="index" label="序号"></el-table-column>
        <el-table-column prop="roleName" label="角色名称"></el-table-column>
        <el-table-column prop="roleDesc" label="角色描述"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditRoleForm(scope.row.id)"
              >编辑</el-button
            >
            <el-button type="danger" icon="el-icon-delete" size="mini" @click="deleteRole(scope.row.id)"
              >删除</el-button
            >
            <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
      <!-- 角色列表区域结束 -->

      <!-- 添加角色对话框 -->
      <el-dialog
        title="添加角色"
        :visible.sync="addRoleFormVisible"
        width="50%"
        @close="addRoleFormClosed()"
      >
        <el-form
          :model="addRoleForm"
          :rules="addRoleRules"
          ref="addRoleFormRef"
          label-width="80px"
        >
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="addRoleForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="addRoleForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addRoleFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="addRole()">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 添加角色对话框结束 -->

      <!-- 编辑角色对话框 -->
      <el-dialog
        title="编辑角色"
        :visible.sync="editRoleFormVisible"
        width="50%"
        @close="editRoleFormClosed()"
      >
        <el-form
          :model="editRoleForm"
          :rules="editRoleRules"
          ref="editRoleFormRef"
          label-width="80px"
        >
          <el-form-item label="角色名称" prop="roleName">
            <el-input v-model="editRoleForm.roleName"></el-input>
          </el-form-item>
          <el-form-item label="角色描述" prop="roleDesc">
            <el-input v-model="editRoleForm.roleDesc"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="editRoleFormVisible = false">取 消</el-button>
          <el-button type="primary" @click="editRole()">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 编辑角色对话框结束 -->

      <!-- 分配权限对话框 -->
      <el-dialog
        title="分配权限"
        :visible.sync="setRightDialogVisible"
        width="50%" @close="closeRightDialog()">
        <!-- 树形控件 -->
        <el-tree :data="rightsList" :props="treeProps" ref="treeRef" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys"></el-tree>
        <span slot="footer" class="dialog-footer">
          <el-button @click="setRightDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="allotRight()">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 分配权限对话框结束 -->
    </el-card>
    <!-- 卡片式图区域结束 -->
  </div>
</template>
 
<script>
export default {
  name: "vueName",
  data() {
    return {
      rolesList: [],
      // 所有权限数据
      rightsList: [],
      addRoleForm: {
        roleName: "",
        roleDesc: "",
      },
      editRoleForm: {},
      addRoleFormVisible: false,
      editRoleFormVisible: false,
      // 控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      addRoleRules: {
        roleName: [
          { required: true, message: "角色名称不能为空", trigger: "blur" },
        ],
        roleDesc: [
          { required: true, message: "角色描述不能为空", trigger: "blur" },
        ],
      },
      editRoleRules: {
        roleName: [
          { required: true, message: "角色名称不能为空", trigger: "blur" },
        ],
        roleDesc: [
          { required: true, message: "角色描述不能为空", trigger: "blur" },
        ],
      },
      // 树形控件的属性绑定对象
      treeProps: {
          label: 'authName',
          children: 'children'
      },
      // 默认选中的节点id值数组
      defKeys: [],
      // 展开分配角色权限面板时获取对应角色id
      roleId: [],
    };
  },
  created() {
    this.getRolesList();
  },
  methods: {
    async getRolesList() {
      const { data: res } = await this.$http.get("roles");
      if (res.meta.status !== 200) {
        return this.$message.error("获取角色列表失败！");
      }
      this.rolesList = res.data;
    },
    // 显示添加角色表单事件
    showAddRoleFrom() {
      this.addRoleFormVisible = true;
    },
    // 添加角色
    addRole() {
      this.$refs.addRoleFormRef.validate(async (val) => {
        if (!val) return;
        const { data: res } = await this.$http.post("roles", this.addRoleForm);
        if (res.meta.status !== 201) {
          return this.$message.error("添加角色失败！");
        }
        this.$message.success("添加角色成功！");
        this.addRoleFormVisible = false;
        this.getRolesList();
      });
    },
    // 关闭添加角色表单事件
    addRoleFormClosed() {
      this.addRoleFormVisible = false;
    },
    // 显示编辑角色表单事件
    async showEditRoleForm(id) {
      const { data: res } = await this.$http.get("roles/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("修改角色信息失败！");
      }
      this.editRoleForm = res.data;
      this.editRoleFormVisible = true;
    },
    // 编辑角色
    editRole() {
        this.$refs.editRoleFormRef.validate(async val => {
            if(!val) return
            const { data: res } = await this.$http.put("roles/" + this.editRoleForm.roleId, {
                roleName : this.editRoleForm.roleName,
                roleDesc : this.editRoleForm.roleDesc,
            });
            if (res.meta.status !== 200) {
                return this.$message.error("修改角色信息失败！");
            }
            this.$message.success("修改角色信息成功！")
            this.editRoleFormVisible = false;
            this.getRolesList();
        })
    },
    // 关闭编辑角色表单事件
    editRoleFormClosed() {
        this.editRoleFormVisible = false;
    },
    // 删除角色
    async deleteRole(id){
        const res = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
        }).catch( err => err)
        if(res !== "confirm"){
            return this.$message.info("已取消删除！")
        }
        const { data : delRes } = await this.$http.delete("roles/" + id)
        if(delRes.meta.status !== 200){
            return this.$message.error("删除角色失败！")
        }
        this.$message.success("删除角色成功！")
        this.getRolesList();
    },
    // 根据id删除对应权限
    async deleteRight(role,rightId){
        const res = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
        }).catch( err => err)
        if(res !== "confirm"){
            return this.$message.info("已取消删除！")
        }
        const { data: delRes } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if(delRes.meta.status !== 200){
            return this.$message.error("删除权限失败！")
        }
        this.$message.success("删除权限成功！")
        role.children = delRes.data
    },
    // 展示分配权限对话框
    async showSetRightDialog(role){
        this.roleId = role.id
        // 获取所有权限数据
        const { data : res } = await this.$http.get("rights/tree")
        if(res.meta.status !== 200){
            return this.$message.error("获取权限数据失败！")
        }
        this.rightsList = res.data
        this.getLeafKeys(role,this.defKeys)
        this.setRightDialogVisible = true;
    },
    // 通过递归方式获取角色下所有三级权限的id，并保存到数组defKeys中
    getLeafKeys(node,arr){
        if(!node.children){
            return arr.push(node.id)
        }
        node.children.forEach(item => {
            this.getLeafKeys(item,arr)
        })
    },
    // 关闭分配权限对话框
    closeRightDialog(){
        this.defKeys = []
    },
    // 点击给角色分配权限
    async allotRight(){
      const arr = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      const idStr = arr.join(",")
      const { data : res } = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
      if(res.meta.status !== 200){
        return this.$message.error("分配权限失败！")
      }
      this.$message.success("分配权限成功！")
      this.setRightDialogVisible = false
      this.getRolesList()
    }
 },
};
</script>
 
<style scoped lang = "less">
.el-tag{
    margin: 7px;
}
.bdtop{
    border-top: 1px solid #eee;
}
.bdbottom{
    border-bottom: 1px solid #eee;
}
.vcenter{
    display: flex;
    align-items: center;
}
</style>