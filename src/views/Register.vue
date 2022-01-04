<template>
  <el-row :span="24" justify="center" align="middle" style="height: 100%;">
    <el-col>
      <el-card>
        <el-row justify="center">
          <span id="title">注册</span>
        </el-row>
        <el-row :span="8"  justify="center">
          <el-form style="margin-top: 2rem; padding: 0 0.5rem;">
            <el-form-item :error="errMsg.u_name">
              <el-input v-model="form.u_name" placeholder="用户名" @input="clearErr"></el-input>
            </el-form-item>
            <el-form-item :error="errMsg.r_name">
              <el-input v-model="form.r_name" placeholder="姓名" @input="clearErr"></el-input>
            </el-form-item>
            <el-row justify="space-between">
              <el-form-item :error="errMsg.u_pwd" style="width: 49%;">
                <el-input v-model="form.u_pwd" placeholder="密码" show-password @input="clearErr"></el-input>
              </el-form-item>
              <el-form-item :error="errMsg.repeat_pwd" style="width: 49%;">
                <el-input v-model="form.repeat_pwd" placeholder="重复密码" show-password @input="clearErr"></el-input>
              </el-form-item>
            </el-row>
            <el-form-item :error="errMsg.p_num">
              <el-input v-model="form.p_num" placeholder="电话" type="number" @input="clearErr"></el-input>
            </el-form-item>
            <el-row justify="space-between">
              <el-form-item :error="errMsg.c_type" style="width: 13.5rem;">
                <el-select v-model="form.c_type" placeholder="证件类型" style="width: 100%;" @change="form.r_city = ''; errMsg.c_type = ''">
                  <el-option label="中华人民共和国居民身份证" value=0></el-option>
                  <el-option label="台湾居民往来大陆通行证" value=1></el-option>
                  <el-option label="港澳居民来往内地通行证" value=2></el-option>
                  <el-option label="军人证件" value=3></el-option>
                  <el-option label="护照" value=4></el-option>
                  <el-option label="香港身份证" value=5></el-option>
                  <el-option label="澳门身份证" value=6></el-option>
                </el-select>
              </el-form-item>
              <el-form-item :error="errMsg.c_num" style="width: calc(98% - 13.5rem);">
                <el-input v-model="form.c_num" placeholder="证件号" @input="clearErr" @change="inputID"></el-input>
              </el-form-item>
            </el-row>
            <el-row justify="space-between">
              <el-form-item :error="errMsg.r_city" style="width: 49%;">
                <el-input v-model="form.r_city" placeholder="城市" @input="clearErr" :disabled="form.c_type == 0"></el-input>
              </el-form-item>
              <el-form-item :error="errMsg.r_cmty" style="width: 49%;">
                <el-input v-model="form.r_cmty" placeholder="社区" @input="clearErr"></el-input>
              </el-form-item>
            </el-row>
            <el-form-item :error="errMsg.u_idct	">
              <el-input v-model="form.u_idct	" placeholder="个人简介" :rows="2" type="textarea" @input="clearErr"></el-input>
            </el-form-item>
            <el-row justify="end" style="font-size:100%">
              <router-link to="/login">-->去登录</router-link>
            </el-row>
            <el-form-item style="margin-bottom: 0;">
              <el-button type="primary" @click="onSubmit"  style="width: 100%; background-color: rgba(116,52,129,1)">注册</el-button>
            </el-form-item>
          </el-form>
        </el-row>
      </el-card>
    </el-col>
  </el-row>
</template>

<script>
import { parseID } from '../parseID'

export default {
  name: 'Register',
  data() {
    return {
      form: {
        u_name: '',
        r_name: '',
        u_type: 1,
        u_level: 1,
        u_pwd: '',
        repeat_pwd: '',
        p_num: '',
        c_type: null,
        c_num: '',
        r_city: '',
        r_cmty: '',
        u_idct: '',
      },
      errMsg: {
        u_name: '',
        r_name: '',
        u_pwd: '',
        repeat_pwd: '',
        p_num: '',
        c_type: '',
        c_num: '',
        r_city: '',
        r_cmty: '',
        u_idct: ''
      }
    }
  },
  methods: {
    onSubmit() {
      if (!this.form.u_name) {
        this.errMsg.u_name = '用户名不能为空'
      } 
      if (!this.form.r_name) {
        this.errMsg.r_name = '姓名不能为空'
      } 
      if (!this.form.u_pwd) {
        this.errMsg.u_pwd = '密码不能为空'
      } 
      if (this.form.u_pwd != this.form.repeat_pwd) {
        this.errMsg.repeat_pwd = '两次输入的密码不一致'
      } 
      if (!this.form.p_num) {
        this.errMsg.p_num = '电话不能为空'
      } 
      if (this.form.p_num.length != 11) {
        this.errMsg.p_num = '请输入11位电话号码'
      } 
      if (!this.form.c_type) {
        this.errMsg.c_type = '请选择证件类型'
      } 
      if (!this.form.c_num) {
        this.errMsg.c_num = '证件号不能为空'
      } 
      if (!this.form.r_city) {
        this.errMsg.r_city = '城市不能为空'
      } 
      if (!this.form.r_cmty) {
        this.errMsg.r_cmty = '社区不能为空'
      } 
      if((this.form.u_name!=null) && (this.form.r_name!=null) && (this.form.u_pwd!=null) 
          && (this.form.u_pwd == this.form.repeat_pwd) && (this.form.p_num!=null) && (this.form.p_num.length == 11) 
          && (this.form.c_type!=null) && (this.form.c_num!=null) && (this.form.r_city!=null) && (this.form.r_cmty!=null)) {
        this.$axios.post('/api/user/register', {
          u_name: this.form.u_name,
          u_pwd: this.form.u_pwd,
          u_type: this.form.u_type,
          r_name: this.form.r_name,
          c_type: parseInt(this.form.c_type),
          c_num: this.form.c_num,
          p_num: this.form.p_num,
          u_level: this.form.u_level,
          u_idct: this.form.u_idct,
          r_city: this.form.r_city,
          r_cmty: this.form.r_cmty
        })
          .then((res) => {
            if (res.data.result) {
              this.$notify({
                title: '注册成功',
                type: 'success'
              })
              this.$router.replace('/login')
            } else {
              this.errMsg.u_name = res.data.remark
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
      this.errMsg.r_name = ''
      this.errMsg.u_pwd = '',
      this.errMsg.repeat_pwd = ''
      this.errMsg.p_num = ''
      this.errMsg.c_type = ''
      this.errMsg.c_num = ''
      this.errMsg.r_city = ''
      this.errMsg.r_cmty = ''
      this.errMsg.u_idct = ''
    },

    inputID() {
      if (this.form.c_type == 0) {
        this.form.r_city = parseID(this.form.c_num)
      }
    }
  }
}
</script>

<style>
.el-card {
  padding: 1rem;
}

#title {
  font-size: 2rem;
  font-weight: bold;
  line-height: 2rem;
}
</style>
