<template>
  <el-row :span="24" justify="center" align="middle" style="height:100%">
    <el-col>
      <el-card>
        <el-row justify="center">
          <span id="title">登录</span>
        </el-row>
        <el-row :span="12"  justify="center">        
          <el-form style="margin-top: 2rem; padding: 0 0.5rem;">
            <el-form-item :error="errMsg.u_name">
              <el-input v-model="form.u_name" placeholder="用户名" @input="clearErr"></el-input>
            </el-form-item>
            <el-form-item :error="errMsg.u_pwd">
              <el-input v-model="form.u_pwd" placeholder="密码" show-password @input="clearErr"></el-input>
            </el-form-item>
            <el-row justify="end" style="font-size:100%">
              <router-link to="/register">-->去注册</router-link>
            </el-row>
            <el-form-item style="margin-bottom: 0;">
              <el-button type="primary" @click="onSubmit" style="width: 100%;background-color: rgba(116,52,129,1);">登录</el-button>
            </el-form-item>
          </el-form>
        </el-row>
      </el-card>
    </el-col>
  </el-row>
</template>

<script>
export default {
  name: 'Login',
  props: {
    u_name: String,
    u_id: String,
    u_type: Number
  },
  emits: ['update:u_name','update:u_id', 'update:u_type'],
  data() {
    return {
      form: {
        u_name: '',
        u_pwd: ''
      },
      errMsg: {
        u_name: '',
        u_pwd: ''
      }
    }
  },
  methods: {
    onSubmit() {
      if (!this.form.u_name) {
        this.errMsg.u_name = '请输入用户名'
      }
      
      if (!this.form.u_pwd) {
        this.errMsg.u_pwd = '请输入密码'
      }

      if (this.form.u_name && this.form.u_pwd) {
        this.$axios.post('/api/user/login', {
          u_name: this.form.u_name,
          u_pwd: this.form.u_pwd
        })
          .then((res) => {
            if (res.data.result) {
              let u_id = res.data.u_id
              let u_type = res.data.u_type
              sessionStorage.setItem('u_id', u_id)
              sessionStorage.setItem('u_type', u_type)
              this.$notify({
                title: '登录成功',
                type: 'success'
              })
              this.$emit('update:u_name',this.form.u_name)
              this.$emit('update:u_id', u_id)
              this.$emit('update:u_type', u_type)
              this.$router.replace('/')
            } else {
              this.errMsg.u_name = '用户名或密码错误'
              this.errMsg.u_pwd = '用户名或密码错误'
            }
          })
          .catch(() => {
            this.$notify({
              title: '连接至服务器失败',
              type: 'error'
            })
          })
      }
    },

    clearErr() {
      this.errMsg.u_name = ''
      this.errMsg.u_pwd = ''
    }
  }
}
</script>

<style>
.el-card {
  padding: 1rem;
  background-color: rgba(255, 255, 255, 0.8);
}

#title {
  font-size: 1.5rem;
  font-weight: bold;
  line-height: 2rem;
}

</style>
