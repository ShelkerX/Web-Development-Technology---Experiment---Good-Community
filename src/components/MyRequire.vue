<template>
  <el-card v-for="index in requireList" class="my-card" :key="index">
    <template #header>
      <div class="card-header">
        
        <span><el-tag style="margin-right: 0.5rem;">{{ state1str[index.req_type] }}</el-tag>&nbsp;&nbsp;{{ index.req_topic }}</span>
      </div>    
    </template>
    <div>{{ index.req_idct }}</div>
    <el-row justify="space-between" class="secondary">
      <span>状态：{{ state2str[index.req_status] }}</span>
      <span>发起时间：{{ index.req_time }}</span>
      <span>请求人数：{{ index.req_nop }}</span>
    </el-row>
    <el-row justify="end" align="bottom" style="margin-top:1.5rem;">
      <span v-if="index.req_status==1" >
        <el-button type="primary" round size="small" @click="loadResponse(index); this.lookResponseDialogVisible = true" plain>查看响应</el-button>
          <el-button type="primary" round size="small" @click="edit(index)" plain>编辑</el-button>
            <el-popconfirm title="确定删除该请求吗？" @confirm="del(index)" confirmButtonText="是" cancelButtonText="否">
              <template #reference>
                <el-button type="danger" round size="small" plain>删除</el-button>
              </template>
            </el-popconfirm>
      </span>
    </el-row>
  </el-card>

  <el-dialog v-model="editRequireDialogVisible" title="修改请求" width="30rem">
    <el-form label-width="5rem">
      <el-form-item label="请求类型">
        <el-select v-model="editRequireForm.req_type" style="width: 100%;">
          <el-option label="小时工" value=0></el-option>
          <el-option label="搬重物" value=1></el-option>
          <el-option label="上下班搭车" value=2></el-option>
          <el-option label="社区服务自愿者" value=3></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="主题">
        <el-input v-model="editRequireForm.req_topic"></el-input>
      </el-form-item>
      <el-form-item label="描述">
        <el-input v-model="editRequireForm.req_idct"></el-input>
      </el-form-item>
      <el-form-item label="请求人数">
        <el-input v-model="editRequireForm.req_nop" type="number"></el-input>
      </el-form-item>
      <el-form-item label="结束日期">
        <el-date-picker v-model="editRequireForm.end_time" type="date" value-format="YYYY-MM-DD" style="width: 100%;">
        </el-date-picker>
      </el-form-item>
      <!--<el-form-item label="请求介绍照片">
        <el-input v-model="editRequireForm.req_nop" type="number"></el-input>
      </el-form-item>-->
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="editRequireDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="submitEdit">提交修改</el-button>
      </span>
    </template>
  </el-dialog>

  <el-dialog v-model="lookResponseDialogVisible" title="全部响应" width="50rem">
    <el-table :data="responseList" style="width: 100%">
      <el-table-column prop="rsp_time" label="时间" />
      <el-table-column prop="rsp_uid" label="用户标识" />
      <el-table-column prop="rsp_idct" label="留言" />
      <el-table-column label="是否接受" align="center">
        <template #default="scope">
          <el-button style="text-align: center;" size="mini" type="success" @click="procResponse(scope.row, true)"><font-awesome-icon icon="check" /></el-button>
          <el-button style="text-align: center;" size="mini" type="danger" @click="procResponse(scope.row, false)"><font-awesome-icon icon="times" /></el-button>
        </template>
      </el-table-column>
    </el-table>
    <template #footer>
      <span class="dialog-footer">
        <el-button type="primary" @click="lookResponseDialogVisible = false">关闭</el-button>
      </span>
    </template>
  </el-dialog>
</template>

<script>
export default {
  name: 'MyRequire',
  props: {
    u_name: String,
    u_id: String
  },
  data() {
    return {
      editRequireDialogVisible: false,
      lookResponseDialogVisible: false,
      editRequireForm: {
        'req_id': '',
        'req_type': '',
        'req_topic': '',
        'req_idct': '',
        'req_nop': 0,
        'end_time': '',
        'req_photo': ''
      },
      state1str: {
        0: '小时工',
        1: '搬重物',
        2: '上下班搭车',
        3: '社区服务志愿者'
      },
      state2str: {
        0: '已完成',
        1: '待响应',
        2: '已取消',
        3: '到期未完成'
      },
      requireList: [],
      responseList: []
    }
  },
  methods: {
    checkTime(i){ 
      if(i<10){ i='0'+i } return i 
    },
    timeFormat(time){
      var date=new Date(time)
      var dateTime = date.getFullYear()+'-'+this.checkTime(date.getMonth()+1)+'-'+this.checkTime(date.getDate()); 
      console.log(dateTime)
      return dateTime
    },
    load() {
      // console.log(this.u_id+'--')
      this.$axios.get('/api/user/request/info', {
        params: {
          req_uid: this.u_id
        }
      })
        .then((res) => {
          if(res.data.result){
            this.requireList = res.data.info_arr
            console.log( this.requireList)
          } else {
            this.$notify({
              title: '用户未发布请求信息',
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
      this.editRequireForm.req_id = index.req_id
      this.editRequireForm.req_type = index.req_type
      this.editRequireForm.req_topic = index.req_topic
      this.editRequireForm.req_idct = index.req_idct
      this.editRequireForm.req_nop = index.req_nop
      this.editRequireForm.end_time = this.timeFormat(index.end_time)
      this.editRequireForm.req_photo = index.req_photo
      this.editRequireDialogVisible = true
    },

    submitEdit() {
      // console.log(this.editRequireForm.end_time)
      this.$axios.post('/api/user/request/modify', {
        req_id: this.editRequireForm.req_id,
        req_type: this.editRequireForm.req_type,
        req_topic: this.editRequireForm.req_topic,
        req_idct: this.editRequireForm.req_idct,
        req_nop: this.editRequireForm.req_nop,
        end_time: this.editRequireForm.end_time,
        req_photo: this.editRequireForm.req_photo
      })
        .then((res) => {
          if (res.data.result) {
            this.$notify({
              title: '编辑成功',
              type: 'success'
            })
            this.editRequireDialogVisible = false
            this.load()
          } else {
            this.$notify({
              title: '修改失败'+res.data.remark,
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
      this.$axios.post('/api/user/request/delete', {
        req_id: index.req_id,
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
    },

    loadResponse(index) {
      this.responseList = []
      this.$axios.get('/api/user/request/response_info', {
        params: {
          req_id: index.req_id
        }
      })
        .then((res) => {
          if(res.data.result){
            for(var i in res.data.info_arr)
              this.responseList.push(res.data.info_arr[i])
            // this.responseList = res.data.info_arr
            // console.log(this.responseList, typeof this.responseList)
            // console.log(res.data.info_arr, typeof res.data.info_arr)
          } else {
            this.$notify({
              title: '该请求无帮忙信息',
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

    procResponse(obj, option) {
      this.$axios.post('/api/user/request/opt_response', {
        req_id: obj.req_id,
        req_uid: this.u_id,
        rsp_id: obj.rsp_id,
        rsp_uid: obj.rsp_uid,
        option: option
      })
        .then((res) => {
          console.log('recv res')
          console.log(res.data)
          if (res.data.result) {
            this.$notify({
              title: '操作成功',
              type: 'success'
            })
          } else {
            // console.log('recv res result')
            // console.log(typeof res.data.remark)
            this.$notify({
              title: '重复请求或接受人数已达上限',
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
