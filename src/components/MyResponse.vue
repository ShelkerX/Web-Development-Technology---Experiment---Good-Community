<template>
  <el-card v-for="index in responseList" class="my-card" :key="index">
    <template #header>
      <div class="card-header">
        <span><el-tag style="margin-right: 0.5rem;">{{ state1str[index.req_type] }}</el-tag>{{ index.req_topic }}</span>
        <span v-if="index.rsp_status == 0">
          <el-button type="warning" round size="small" plain @click="edit(index)">编辑</el-button>
          <el-popconfirm title="确定删除该响应吗？" @confirm="del(index)" confirmButtonText="是" cancelButtonText="否">
            <template #reference>
              <el-button type="danger" round size="small" plain>删除</el-button>
            </template>
          </el-popconfirm>
        </span>
      </div>
    </template>
    <div>{{  index.req_idct}}</div>
    <el-divider border-style="dashed" content-position="center">我的留言</el-divider>
    <div>{{ index.rsp_idct }}</div>
    <el-row justify="space-between" class="secondary">
      <span>状态：{{ state2str[index.rsp_status] }}</span>
      <span>响应时间：{{ index.rsp_time }}</span>
    </el-row>
  </el-card>


  <el-dialog v-model="editResponseDialogVisible" title="编辑留言" width="30rem">
    <el-input v-model="editResponseForm.rsp_idct" placeholder="留言" :rows="2" type="textarea"></el-input>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="editResponseDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="submitEdit" plain>确认</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script>
export default {
  name: 'MyResponse',
  props: {
    u_name: String,
    u_id: String,
  },
  data() {
    return {
      editResponseDialogVisible: false,
      editResponseForm: {
        'rsp_id': '',
        'rsp_idct': ''
      },
      state1str: {
        0: '小时工',
        1: '搬重物',
        2: '上下班搭车',
        3: '社区服务志愿者'
      },
      state2str: {
        0: '待接受',
        1: '接受',
        2: '拒绝',
        3: '取消'
      },
      responseList: [],
      requireDetail: {}
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
    load() {
      this.status = false
      this.responseDetail = {}
      this.$axios.get('/api/user/response/response_info', {
        params: {
          rsp_uid: this.u_id
        }
      })
        .then((res) => {
          if(res.data.result){
            this.responseList=res.data.info_arr
            console.log(this.responseList)
          } else {
            this.$notify({
            title: '用户未发布帮忙信息',
            type: 'warning'
          })
          }
        })
        .catch(() => {
          this.$notify({
            title: '连接至服务器失败',
            type: 'error'
          })
        })
    },

    edit(index) {
      this.editResponseForm.rsp_id = index.rsp_id
      this.editResponseForm.rsp_idct = index.rsp_idct
      this.editResponseDialogVisible = true
    },
    submitEdit() {
      console.log(this.editResponseForm)
      this.$axios.post('/api/user/response/modify', {
        rsp_id: this.editResponseForm.rsp_id,
        rsp_idct: this.editResponseForm.rsp_idct,
      })
        .then((res) => {
          if (res.data.result) {
            this.$notify({
              title: '编辑成功',
              type: 'success'
            })
            this.editResponseDialogVisible = false
            this.load()
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
    },
    del(index) {
      this.$axios.post('/api/user/response/delete', {
        rsp_id: index.rsp_id,
      })
        .then((res) => {
          if (res.data.result) {
            this.$notify({
              title: '删除成功',
              type: 'success'
            })
            this.load()
          } else {
            this.$notify({
              title: '删除失败',
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
  }
}
</script>

<style>
.my-card {
  display: block;
  margin: 0.5rem;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.5rem;
}

.el-card__header {
  padding: 0 1rem;
}

.el-card__body {
  padding-bottom: 0.6rem;
}

.secondary {
  color: #757575;
  margin-top: 1rem;
  font-size: 0.9rem;
}
</style>
