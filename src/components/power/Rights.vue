<template>
  <div class="">
    <!-- 头部面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 头部面包屑导航结束 -->

    <!-- 卡片视图区域 -->
    <el-card>
        <el-table :data="rightList" border stripe="">
            <el-table-column type="index"></el-table-column>
            <el-table-column label="权限名称" prop="authName"></el-table-column>
            <el-table-column label="路径" prop="path"></el-table-column>
            <el-table-column label="等级" prop="level">
                <template slot-scope="scope">
                    <el-tag v-if="scope.row.level === '1'">一级</el-tag>
                    <el-tag type="success" v-else-if="scope.row.level === '2'">二级</el-tag>
                    <el-tag type="warning" v-else>三级</el-tag>
                </template>
            </el-table-column>
        </el-table>
    </el-card>
    <!-- 卡片视图区域结束 -->
  </div>
</template>
 
<script>
export default {
  name: "vueName",
  data() {
    return {
      rightList: [],
    };
  },
  created() {
    this.getRightList();
  },
  methods: {
    async getRightList() {
      const { data: res } = await this.$http.get("rights/list");
      if (res.meta.status !== 200) {
        return this.$message.error("获取权限列表失败");
      }
      this.rightList = res.data;
      console.log(this.rightList)
    },
  },
};
</script>
 
<style scoped lang = "less">
</style>