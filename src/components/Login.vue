<template>
  <div class="login_container">
    <div class="login_box">
      <!--        头像区-->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt="">
      </div>
      <!--        登陆表单区-->
      <el-form ref="loginFormRef" label-width="0px" :rules="loginFormRules" class="login_form" :model="loginForm">
        <!--        用户名-->
        <el-form-item prop="username">
          <el-input v-model="loginForm.username" prefix-icon="el-icon-user"></el-input>
        </el-form-item>
        <!--        密码-->
        <el-form-item prop="password">
          <el-input v-model="loginForm.password" type="password" prefix-icon="el-icon-lock"></el-input>
        </el-form-item>
        <!--        按钮-->
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登陆</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      loginForm: {
        username: 'admin',
        password: '123456'
      },
      loginFormRules: {
        username: [
          {
            required: true,
            message: '请输入用户名称',
            trigger: 'blur'
          },
          {
            min: 3,
            max: 15,
            message: '长度在 3 到 15 个字符',
            trigger: 'blur'
          }
        ],
        password: [
          {
            required: true,
            message: '请输入登陆密码',
            trigger: 'blur'
          },
          {
            min: 5,
            max: 15,
            message: '长度在 5 到 15 个字符',
            trigger: 'blur'
          }

        ]
      }
    }
  },
  methods: {
    resetLoginForm () {
      this.$refs.loginFormRef.resetFields()
    },
    login () {
      /* await只能放在async修饰的函数中，表示异步执行该函数 */
      this.$refs.loginFormRef.validate(async valid => {
        if (!valid) return
        const { data: result } = await this.$http.post('login', this.loginForm)
        if (result.meta.status !== 200) return this.$message.error('登陆失败！')
        window.sessionStorage.setItem('token', result.data.token)
        await this.$router.push('/home')
        this.$message.success('登陆成功！')
        /*
         if (this.loginForm.username === 'admin' && this.loginForm.password === 'admin') {
           window.sessionStorage.setItem('token', this.$token)
           this.$router.push('/home')
           return this.$message.success('登陆成功！')
         }
         return this.$message.error('用户名或密码错误，登陆失败！') */
        //  1.将登陆成功之后的token保存到客户端的sessionStorage中
        //    1.1 项目中除了登陆之外的其他API接口，必须在登陆之后才能访问
        //    1.2 token只应在当前网址打开期间生效，所以将token保存在sessionStorage中
        //  2.通过编程式导航跳转到后台主页，路由地址是/home
      })
    }
  }
}
</script>

<style lang="less" scoped>
.login_container {
  background-color: #2b4b6b;
  height: 100%;
}

.login_box {
  width: 450px;
  height: 300px;
  background-color: #fff;
  border-radius: 3px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);

  .avatar_box {
    height: 130px;
    width: 130px;
    border: 1px solid #eee;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0 0 10px #ddd;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #fff;

    img {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      background-color: #eee;
    }
  }
}

.btns {
  display: flex;
  justify-content: flex-end;
}

.login_form {
  position: absolute;
  bottom: 0;
  width: 100%;
  padding: 0 20px;
  box-sizing: border-box;
}
</style>
