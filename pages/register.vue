<template>
  <div class="page-register">
    <article class="header">
      <header>
        <a href="#" class="site-logo"/>
        <span class="login">
          <em class="bold">已有美团账号？</em>
          <a href="/login">
            <el-button type="primary" size="small">登录</el-button>
          </a>
        </span>
      </header>
    </article>
    <section>
      <el-form
        :model="ruleForm"
        :rules="rules"
        ref="ruleForm"
        label-width="100px"
        class="demo-ruleForm"
      >
        <el-form-item label="昵称" prop="name">
          <el-input v-model="ruleForm.name"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input type="email" v-model="ruleForm.email"></el-input>
          <el-button type="mini" round @click="sendMsg">免费获取验证码</el-button>
          <span class="status">{{statusMsg}}</span>
        </el-form-item>
        <el-form-item label="验证码" prop="code">
          <el-input v-model="ruleForm.code" maxlength="4"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="pwd">
          <el-input type="password" v-model="ruleForm.pwd"></el-input>
        </el-form-item>
        <el-form-item label="确认密码" prop="cpwd">
          <el-input type="password" v-model="ruleForm.cpwd"></el-input>
        </el-form-item>
        <el-form-item label="确认协议">
          <el-button type="primary" @click="register">同意以下协议并注册</el-button>
          <div class="error">{{error}}</div>
        </el-form-item>
        <el-form-item>
          <a class="f1" href="http://www.meituan.com/about/terms" target="_blank">《美团网用户协议》</a>
        </el-form-item>
      </el-form>
    </section>
  </div>
</template>

<script>
import cryptoJs from 'crypto-js'
export default {
  data() {
    return {
      statusMsg: "",
      error: '',
      ruleForm: {
        name: '',
        email: '',
        code: '',
        pwd: '',
        cpwd: ''
      },
      rules: {
        name: [{
          required: true, type: 'string', message: '请输入昵称', trigger: 'blur'
        }],
        email: [{
          required: true, type: 'email', message: '请输入邮箱', trigger: 'blur'
        }],
        // code: [{
        //   required: true, type: 'number', message: '请输入验证码', trigger: 'blur'
        // }],
        pwd: [{
          required: true, type: 'string', message: '请输入密码', trigger: 'blur'
        }],
        cpwd: [{
          required: true, type: 'string', message: '请在输一次密码', trigger: 'blur'
        }, {
          validator: (rule, value, callback) => { // 自定义校验规则
            if (value === '') {
              return callback(new Error('请再次输入密码'))
            } else if (value !== this.ruleForm.pwd) {
              return callback(new Error('两次输入的密码不一致'))
            } else {
              callback()
            }
          }
        }]
      }
    };
  },
  layout: "blank",
  methods: {
    sendMsg: function() {
      // 发送邮箱验证码
      // 校验规则 调用接口 发送验证码
      const self = this;
      let namePass
      let emailPass
      if (self.timered) {
        return false;
      }
      this.$refs['ruleForm'].validateField('name', (valid) => {
         namePass = valid
      })
      self.statusMsg = ''
      if (namePass){
        return false;
      } 
      this.$refs['ruleForm'].validateField('email', (valid) => {
        emailPass = valid
      })
      if (!namePass && !emailPass) {
        self.$axios.post('/users/verify', {
          username: encodeURIComponent(self.ruleForm.name),
          email: self.ruleForm.email
        }).then(({status, data}) => {
          if (status === 200 && data && data.code === 0) {
            let count = 60;
            self.statusMsg = `验证码已发送${count--}`
            self.timered = setInterval(() => {
              self.statusMsg = `验证码已发送${count--}`
              if (count===0) {
                clearInterval(self.timered)
              }
            }, 1000)
          } else {
            self.statusMsg = data.msg
          }
        })
      } 
    },
    register: function() {
      const self = this
      this.$refs['ruleForm'].validate((valid) => {
        console.log('valid'+ valid)
        if (valid) {
          self.$axios.post('/users/signup', {
            username: window.encodeURIComponent(self.ruleForm.name),
            password: cryptoJs.MD5(self.ruleForm.pwd).toString(),
            email: self.ruleForm.email,
            code: self.ruleForm.code
          }).then(({status, data}) => {
            if (status === 200) {
              if (data && data.code === 0) {
                location.href = '/login'
              } else {
                self.error = data.msg
              }
            } else {
              self.error = `服务端出错,状态码是： ${status}`
            }
          })
          setTimeout(function() {
            self.error = ''
          }, 1500)
        }
      })

    }
  }
};
</script>


<style lang="scss">
@import "@/assets/css/register/index.scss";
</style>
