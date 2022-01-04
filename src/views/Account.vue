<template>
  <el-row justify="center" style="padding-top: 3rem;">
    <el-col :span="14">
          <el-form label-width="5rem" style="width: 80%; text-align: center;">
            <el-form-item label="用户名">
              {{form.u_name}}
            </el-form-item>
            <el-form-item label="姓名">
              {{form.r_name}}
            </el-form-item>
            <el-form-item label="证件号">
              {{form.c_num}}
            </el-form-item>
            <el-form-item label="电话号码">
              <el-input v-model="form.p_num"></el-input>
            </el-form-item>
            <el-form-item label="个人简介">
              <el-input v-model="form.u_idct" :rows="2" type="textarea"></el-input>
            </el-form-item>
            <el-form-item label="城市">
              {{form.r_city}}
            </el-form-item>
            <el-form-item label="社区">
              {{form.r_cmty}}
            </el-form-item>
            <el-form-item label="原密码" :error="errMsg.o_pwd">
              <el-input v-model="form.o_pwd" show-password></el-input>
            </el-form-item>
            <el-form-item label="新密码">
              <el-input v-model="form.n_pwd" show-password></el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="onSubmit">确认修改</el-button>
            </el-form-item>
          </el-form>
    </el-col>
  </el-row>
</template>

<script>
export default {
  name: 'Account',
  props: {
    u_id: String,
    u_type: Number,
  },
  data() {
    return {
      form: {
        u_name: '',
        r_name: '',
        c_num: '',
        p_num: '',
        u_idct: '',
        r_city: '',
        r_cmty: '',
        o_pwd: '',
        n_pwd:''
      },
      errMsg: {
        o_pwd: ''
      }
    }
  },
  methods: {
    formatDate() {
      let date = new Date()
      let yyyy = date.getFullYear().toString()
      let MM = (date.getMonth() + 1).toString()
      let dd = date.getDate().toString()
      return yyyy + '-' + MM + '-' + dd
    },
    onSubmit() {
      if(!this.form.o_pwd){
        this.errMsg.o_pwd = '请输入密码'
      } else if(!this.form.n_pwd){
        this.form.n_pwd = this.form.o_pwd
      }

      if(this.form.o_pwd && this.form.n_pwd){
        this.$axios.post('/api/user/login', {
          u_name: this.form.u_name,
          u_pwd: this.form.o_pwd
        })
          .then((res) => {
            if (res.data.result) {
              this.$axios.post('/api/user/modify', {
                u_id: this.u_id,
                u_pwd: this.form.n_pwd,
                p_num: this.form.p_num,
                u_idct: this.form.u_idct,
              })
                .then((res) => {
                  if (res.data.result) {
                    this.$notify({
                      title: '修改成功',
                      type: 'success'
                    })
                  } else {
                    this.$notify({
                      title: '修改失败',
                      type: 'error'
                    })
                  }
                })
                .catch(() => {
                  this.$notify({
                    title: '连接至服务器失败',
                    type: 'error'
                  })
                })
            } else {
              this.$notify({
                title: '原密码错误',
                type: 'error'
              })
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
    getInfo() {
      this.$axios.get('/api/user/info', {
        params: {
          u_id: this.u_id
        }
      })
        .then((res) => {
          this.form.u_name = res.data.info_obj.u_name
          this.form.r_name = res.data.info_obj.r_name
          this.form.c_num = res.data.info_obj.c_num
          this.form.p_num = res.data.info_obj.p_num
          this.form.u_idct = res.data.info_obj.u_idct
          this.form.r_city = res.data.info_obj.r_city
          this.form.r_cmty = res.data.info_obj.r_cmty
        })
        .catch(() => {
          this.$notify({
            title: '连接至服务器失败',
            type: 'error'
          })
        })
    }
  },
  mounted() {
    this.getInfo()
  }
}
</script>