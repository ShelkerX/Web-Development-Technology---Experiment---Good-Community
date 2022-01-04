<template>
  <el-row justify="center" style="padding-top: 3rem;">
    <el-col :span="14">
      <el-tabs tab-position="top"  v-model="tab" @tab-click="load">
        <el-tab-pane label="我的请求">
          <MyRequire ref="myRequire" :u_name="u_name" :u_id="u_id" />
        </el-tab-pane>
        <el-tab-pane label="我的响应">
          <MyResponse ref="myResponse" :u_name="u_name" :u_id="u_id" />
        </el-tab-pane>
      </el-tabs>
    </el-col>
  </el-row>

  <div id="refresh" @click="load">
    <font-awesome-icon icon="sync-alt" />
  </div>
</template>

<script>
import MyRequire from '../components/MyRequire.vue'
import MyResponse from '../components/MyResponse.vue'

export default {
  name: 'My',
  components: {
    MyRequire,
    MyResponse
  },
  props: {
    u_name: String,
    u_id: String,
    u_type: Number
  },
  data() {
    return {
      tab: '0'
    }
  },
  methods: {
    load() {
      // console.log(this.u_id + '**')
      if (this.tab == '0') {
        this.$refs.myRequire.load()
      } else {
        this.$refs.myResponse.load()
      }
    }
  },
  mounted() {
    this.load()
  }
}
</script>

<style>
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
</style>
