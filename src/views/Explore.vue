<template>
  <el-row justify="center" style="padding-top: 1rem;">
    <el-col>
      <el-row justify="start" style="margin: 1rem;" align="middle" :gutter="20">
        <el-col :span="6">
          选择请求类型:&nbsp;&nbsp;
          <el-select v-model="filter.type" style="width: 10rem;">
            <el-option label="全部类型" value="全部类型"></el-option>
            <el-option label="小时工" value="小时工"></el-option>
            <el-option label="搬重物" value="搬重物"></el-option>
            <el-option label="上下班搭车" value="上下班搭车"></el-option>
            <el-option label="社区服务自愿者" value="社区服务自愿者"></el-option>
          </el-select>
        </el-col>
      </el-row>
      
      <el-card v-for="item in filtered.slice((curPage-1)*pageSize,curPage*pageSize)" class="explore-card" :key="item.req_ID">
        <template #header>
          <div class="card-header">
            <span><el-tag>{{ parseType(item.req_type) }}</el-tag>&nbsp;&nbsp;{{ item.req_topic }}</span>
          </div>
        </template>
        <div>{{ item.req_idct }}</div>
        <el-row justify="space-between" class="secondary" align="middle">
          <span>请求用户：{{ item.req_uid }}</span>
          <span>发起时间：{{ item.req_time }}</span>
          <span>请求人数：{{ item.req_nop }}</span>
          <span>状态：{{ parseStatus(item.req_status)}}</span>
        </el-row>
        <el-row justify="end" align="middle" >
          <el-button v-if="item.req_status==1 && item.req_uid!=this.u_id" size="small"  @click="responseInfo.req_id = item.req_id; responseDialogVisible = true">我可以</el-button>
          <el-button v-if="item.req_status!=1 || item.req_uid==this.u_id" size="small"  disabled @click="responseInfo.req_id = item.req_id; responseDialogVisible = true">我可以</el-button>
        </el-row>
      </el-card>
    </el-col>
  </el-row>
  <el-row justify="center">
    <el-pagination 
      v-model:currentPage="curPage"
      layout="prev, pager, next"
      :page-size="pageSize"
      @current-change="curChange"
      :total="totalPage*pageSize"
    >
    </el-pagination>
  </el-row>

  <el-dialog v-model="postDialogVisible" title="发布请求" width="30rem">
    <el-form label-width="5rem">
      <el-form-item label="请求类型">
        <el-select v-model="form.req_type" style="width: 100%;">
          <el-option label="小时工" value=0></el-option>
          <el-option label="搬重物" value=1></el-option>
          <el-option label="上下班搭车" value=2></el-option>
          <el-option label="社区服务自愿者" value=3></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="主题">
        <el-input v-model="form.req_topic"></el-input>
      </el-form-item>
      <el-form-item label="描述">
        <el-input v-model="form.req_idct"></el-input>
      </el-form-item>
      <el-form-item label="请求人数">
        <el-input v-model="form.req_nop" type="number"></el-input>
      </el-form-item>
      <el-form-item label="结束时间">
        <el-date-picker v-model="form.end_time" type="date" value-format="YYYY-MM-DD" style="width: 100%;">
        </el-date-picker>
      </el-form-item>
    </el-form>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="postDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="post">发布</el-button>
      </span>
    </template>
  </el-dialog>

  <el-dialog v-model="responseDialogVisible" title="响应请求" width="30rem">
    <el-input v-model="responseInfo.response" placeholder="留言" :rows="2" type="textarea"></el-input>
    <template #footer>
      <span class="dialog-footer">
        <el-button @click="responseDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="response">确认</el-button>
      </span>
    </template>
  </el-dialog> 

  <div id="refresh" @click="load">
    <font-awesome-icon icon="sync-alt" />
  </div>
  <div id="post" @click="postDialogVisible = true">
    <font-awesome-icon icon="plus" />
  </div>
</template>

<script>
export default {
  name: 'Explore',
  props: {
    u_id: String,
    u_type: Number
  },
  data() {
    return {
      postDialogVisible: false,
      responseDialogVisible: false,
      responseInfo: {
        req_id: '',
        rsp_idct: ''
      },
      filter: {
        type: "全部类型",
        keyword: ''
      },
      form: {
        req_cmty: '',
        req_uid: '',
        req_type: '',
        req_topic: '',
        req_idct: '',
        req_nop: 1,
        end_time: null,
        req_photo:null
      },
      info_arr: {},

      totalPage: 1,
      curPage: 1,
      pageSize: 6
    }
  },
  computed: {
    data() {
      let ans = []
      for (var i = 0; i < this.getJsonLength(this.info_arr); i++) {
        ans.push({
          'req_id': this.info_arr[i].req_id,
          'req_uid': this.info_arr[i].req_uid,
          'req_type': this.info_arr[i].req_type,
          'req_topic': this.info_arr[i].req_topic,
          'req_idct': this.info_arr[i].req_idct,
          'req_nop': this.info_arr[i].req_nop,
          'req_time': this.info_arr[i].req_time,
          'req_status': this.info_arr[i].req_status
        })
      }
      return ans
    },
    filtered() {
      // console.log(this.data)
      return this.data.filter(item => ( this.filter.type == "全部类型" || (item.req_type == 0 && this.filter.type=="小时工")
      ||(item.req_type == 1 && this.filter.type=="搬重物")||(item.req_type == 2 && this.filter.type=="上下班搭车")||(item.req_type == 3 && this.filter.type=="社区服务自愿者")))
    }
  },
  methods: {
    parseStatus(status)
    {
      if(status=='0'){
        return "已完成"
      }
      if(status=='1'){
        return "待响应"
      }
      if(status=='2'){
        return "已取消"
      }
      if(status=='3'){
        return "到期未达成"
      }
    },
    parseType(status)
    {
      if(status=='0'){
        return "小时工"
      }
      if(status=='1'){
        return "搬重物"
      }
      if(status=='2'){
        return "上下班搭车"
      }
      if(status=='3'){
        return "社区服务自愿者"
      }
    },
    getJsonLength(jsonData){
      var jsonLength = 0;
      for(var item in jsonData){
        jsonLength++;
        item
      }
      return jsonLength;
    },
    load() {
      this.$axios.get('/api/user/info', {
        params: {
          u_id: this.u_id
        }
      })
        .then((res) => {
          let community = res.data.info_obj.r_cmty
          this.form.req_cmty=community
          this.$axios.get('/api/user/response/request_info', {
            params: {
              community: community
              }
          })
            .then((res) => {
              let len = this.getJsonLength(res.data.info_arr)
              for (var i = 0; i < len; i++){
                this.info_arr[i] = {
                  'req_id': res.data.info_arr[i].req_id,
                  'req_uid': res.data.info_arr[i].req_uid,
                  'req_type': res.data.info_arr[i].req_type,
                  'req_topic': res.data.info_arr[i].req_topic,
                  'req_idct': res.data.info_arr[i].req_idct,
                  'req_nop': res.data.info_arr[i].req_nop,
                  'end_time':res.data.info_arr[i].end_time,
                  'req_time': res.data.info_arr[i].req_time,
                  'req_status': res.data.info_arr[i].req_status
                }
              }
              this.totalPage = Math.ceil(len/this.pageSize)
              // console.log(this.totalPage)
            })
            .catch(() => {
              this.$notify({
                title: '连接至服务器失败',
                type: 'error'
              })
            })
        })
        .catch(() => {
          this.$notify({
            title: '连接至服务器失败',
            type: 'error'
          })
        })
    },
    post() {
      this.$axios.post('/api/user/request/release', {
        req_cmty: this.form.req_cmty,
        req_uid: this.u_id,
        req_type: parseInt(this.form.req_type),
        req_topic: this.form.req_topic,
        req_idct: this.form.req_idct,
        req_nop: parseInt(this.form.req_nop),
        end_time: this.form.end_time,
        req_photo:null
      })
        .then((res) => {
          if (res.data.result) {
            this.$notify({
              title: '发布成功'+res.data.req_id,
              type: 'success'
            })
            this.postDialogVisible = false
            this.load()
          } else {
            this.$notify({
              title: '发布失败'+res.data.remark,
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
    response() {
      this.$axios.post('/api/user/response/release', {
        req_id: this.responseInfo.req_id,
        rsp_uid : this.u_id,
        rsp_idct: this.responseInfo.rsp_idct,
      })
        .then((res) => {
          if (res.data.result) {
            this.responseDialogVisible = false
            this.load()
            this.$notify({
              title: '响应成功'+res.data.rsp_id,
              type: 'success'
            })
          } else {
            this.$notify({
              title: '响应失败'+res.data.remark,
              type: 'error'
            })
          }
          this.responseInfo.rsp_idct=''
        })
        .catch(() => {
          this.$notify({
            title: '连接至服务器失败',
            type: 'error'
          })
          this.responseInfo.rsp_idct=''
        })
    },
    curChange:function(curPage) {
      this.curPage = curPage
    }
  },
  mounted() {
    this.load()
  }
}
</script>

<style>
.explore-card {
  display: inline-block;
  width: 30rem;
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

#refresh {
  position: fixed;
  bottom: 3rem;
  right: 3rem;
  height: 3.5rem;
  width: 3.5rem;
  background-color: #E65100;
  border-radius: 3rem;
  text-align: center;
  line-height: 3.5rem;
  cursor: pointer;
  color: #fff;
  font-size: 1.2rem;
}

#refresh:hover {
  background-color: #F57C00;
}

#post {
  position: fixed;
  bottom: 7.2rem;
  right: 3rem;
  height: 3.5rem;
  width: 3.5rem;
  background-color: #1B5E20;
  border-radius: 3rem;
  text-align: center;
  line-height: 3.5rem;
  cursor: pointer;
  color: #fff;
  font-size: 1.2rem;
}

#post:hover {
  background-color: #388E3C;
}

.secondary {
  color: #757575;
  margin-top: 1rem;
  font-size: 0.9rem;
}
</style>
