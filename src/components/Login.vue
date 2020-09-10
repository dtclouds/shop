<template>
  <div class="login-container">
    <div class="login-box">
      <div class="avator-box">
        <img src="../assets/images/avator.jpg" alt />
      </div>
      <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules">
        <el-form-item prop="username">
          <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user"></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input v-model="loginForm.password" prefix-icon="iconfont icon-lock" type="password"></el-input>
        </el-form-item>
        <el-row class="btn-box">
          <el-button type="primary" @click="login()">登录</el-button>
          <el-button type="info" @click="resetLoginForm()">重置</el-button>
        </el-row>
      </el-form>
    </div>
  </div>
</template>
 
<script>
export default{
  data() {
    return {
      loginForm:{
        username: 'admin',
        password: '123456'
      },
      loginFormRules: {
        username:[
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password:[
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ],
      }
    }
  },
  methods:{
    resetLoginForm(){
      this.$refs.loginFormRef.resetFields();
    },
    login(){
      this.$refs.loginFormRef.validate(async valid => {
        if(!valid) return
        var {data:res} = await this.$http.post('login',this.loginForm);
        if(res.meta.status !== 200) return this.$message.error("登陆失败");
        this.$message.success("登陆成功");
        window.sessionStorage.setItem('token',res.data.token);
        this.$router.push('/home')
      })
    }
  }
}
</script>
 
<style scoped lang = "less">
.login-container {
  display: flex;
  width: 100%;
  height: 100%;
  justify-content: center;
  align-items: center;
  background: #2b4b6b;
  .login-box {
    width: 450px;
    height: 304px;
    margin: 0 auto;
    border-radius: 4px;
    background: #fff;
    .avator-box {
      display: block;
      width: 130px;
      height: 130px;
      margin: 0 auto;
      padding: 8px;
      border: 1px solid #eee;
      border-radius: 50%;
      box-shadow: 0 0 10px #eee;
      background: #fff;
      transform: translateY(-50%);
      img {
        width: 100%;
        height: 100%;
        border-radius: 50%;
      }
    }
    .el-form{
      width: 100%;
      margin: -60px 0 0;
      padding: 20px;
      box-sizing: border-box;
    }
  }
}
.btn-box{
  display: flex;
  justify-content: flex-end;
}
</style>