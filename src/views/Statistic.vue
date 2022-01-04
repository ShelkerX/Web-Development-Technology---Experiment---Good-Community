<template>
  <el-row justify="center" style="padding-top: 3rem;">
    <el-col :span="18">
      <el-tabs tab-position="top" style="height: 100%;">
        <el-tab-pane label="中介费查询">
          <el-row style="margin: 0.5rem;">
            <el-col :span="10">
              <el-date-picker
                v-model="filter.range"
                type="daterange"
                range-separator="至"
                start-placeholder="起始时间"
                end-placeholder="截止时间"
                style="width: 22.5rem; margin-left: 0.5rem;"
              >
              </el-date-picker>
              <el-button type="primary" style="margin-left: 0.5rem;" @click="queryFee">查询</el-button>
            </el-col>
            <el-col :span="4">
              <h3>累计中介费：{{ agency_fee }} 元</h3>
            </el-col>
          </el-row>

          <div id="main" ref="chart" style="width: 80rem; height:30rem"> <!--
            <chart autosize ></chart>-->
          </div>

        </el-tab-pane>

        <el-tab-pane label="用户列表">
          <el-table :data="userList" style="width: 100%">
            <el-table-column prop="u_name" label="用户名" />
            <el-table-column prop="r_name" label="姓名" />
            <el-table-column prop="c_type" label="证件类型" />
            <el-table-column prop="c_num" label="证件号" />
            <el-table-column prop="p_num" label="电话号码" />
            <el-table-column prop="u_idct" label="用户简介" />
            <el-table-column prop="r_city" label="城市" />
            <el-table-column prop="r_cmty" label="社区" />
            <el-table-column prop="r_time" label="注册时间" />
          </el-table>
        </el-tab-pane>
        
      </el-tabs>
    </el-col>
  </el-row>
</template>

<script>
import * as echarts from 'echarts'

export default {
  name: 'Statistic',
  props: {
    u_name: String,
    u_id: String,
    u_type: Number
  },
  data() {
    return {
      chart: null,
      filter: {
        type: '全部类型',
        city: '',
        community: '',
        range: null
      },
      userList: [],
      monthly_fee: [],
      agency_fee: 0
    }
  },
  methods: {
    formatDateTime(date) {
      return date.toISOString().replace(/T/,' ').replace(/.000Z/,'')
    },
    loadUserList() {
      this.$axios.get('/api/admin/all_users')
        .then((res) => {
          if(res.data.result){
            // console.log(res.data)
            for(var i in res.data.info_arr){
              this.userList.push(res.data.info_arr[i])
            }
          } else {
            this.$notify({
              title: '无普通用户信息',
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
    queryFee(){
      // console.log(this.filter.range)
      if(this.filter.range){
        this.queryTotalFee()
        this.queryMonthlyFee()
      } else {
        this.$notify({
          title: '请选择时间范围',
          type: 'error'
        })
      }
      
    },
    queryTotalFee() {
      this.$axios.get('/api/admin/fee', {
        params: {
          start_time: this.filter.range[0],
          end_time: this.filter.range[1]
        }
      })
        .then((res) => {
          // console.log(res.data)
          this.agency_fee = res.data.agency_fee
        })
        .catch(() => {
          this.$notify({
            title: '连接至服务器失败',
            type: 'error'
          })
        })
    },
    queryMonthlyFee() {
      this.chart.clear()
      let first_day = new Date(this.filter.range[1].getYear()+1900, this.filter.range[1].getMonth()+1, 1)
      let s_time = this.formatDateTime(first_day)
      let last_day = new Date(this.filter.range[1].getYear()+1900, this.filter.range[1].getMonth()+1, 0)
      let e_time = this.formatDateTime(last_day)
      console.log(s_time, e_time)
      this.$axios.get('/api/admin/statistics', {
        params: {
          start_time: s_time,
          end_time: e_time
        }
      })
        .then((res) => {
          if(res.data.result)
          {
            this.monthly_fee = res.data.info_arr
            let monthly_fee_arr = []
            for(let i in this.monthly_fee){
              monthly_fee_arr.push(this.monthly_fee[i])
            }
            let Option = {
              title: {
                show: true,
                text:"各月份中介费",
                x: 'center'
              },
              xAxis: {
              type: 'category',
              data: monthly_fee_arr.map(item => item.the_month)
              },
              yAxis: {
                type: 'value'
              },
              series: [
                {
                  type: 'bar',
                  barWidth: '7.5%',
                  data: monthly_fee_arr.map(item => item.agc_fee),
                  label: {
                    show: true,
                    position: 'top'
                  }
                }
              ]
            }
            this.chart.setOption(Option, true)

            this.$notify({
              title: '请求成功',
              type: 'success'
            })
          } else {
            this.$notify({
              title: '该时间范围无已完成请求信息',
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
  },
  mounted() {
    Object.defineProperty(document.getElementById('main'),'clientWidth',{get:function(){return Math.floor(window.innerWidth*0.7);}})
    Object.defineProperty(document.getElementById('main'),'clientHeight',{get:function(){return Math.floor(window.innerHeight*0.5);}})
    this.chart = echarts.init(document.getElementById('main'))
    this.loadUserList()
  }
}
</script>

<style>
</style>
