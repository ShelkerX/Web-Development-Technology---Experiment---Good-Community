<template>
  <el-container style="height: 100%;">
    <el-header style="height:5rem; line-height: 5rem; box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);">
      <el-row justify="space-between" style="margin: 0 1rem;">
        <span style="font-size: 1.5rem;">
          <router-link to="/" style="color: unset; text-decoration: unset;">
            <font-awesome-icon icon="comments" style="margin-right: 0.5rem;" />
            好社区
          </router-link>
        </span>
        <span v-if="u_id">
          <span algin="middle" style="margin-right:2rem">{{ u_id }}</span>
          <font-awesome-icon icon="sign-out-alt" @click="logOut" style="cursor: pointer;" />
        </span>
      </el-row>
    </el-header>
    <el-container style="margin-top:1rem;">
      <el-aside v-if="u_id" style="box-shadow: 0 2px 4px rgba(0, 0, 0, 0.12), 0 0 6px rgba(0, 0, 0, 0.04);">
        <span id="nav">
          <el-row justify="center">
            <div :class="{ active: isActive('/explore') }" style="left: 14rem;">
              <router-link to="/explore">探索</router-link>
            </div>
          </el-row>
          <el-row justify="center">
          <div :class="{ active: isActive('/my') }" style="left: 20rem;">
            <router-link to="/my">我的</router-link>
          </div>
          </el-row>
          <el-row justify="center">
          <div :class="{ active: isActive('/account') }" style="left: 26rem;">
            <router-link to="/account">账户信息</router-link>
          </div>
          </el-row>
          <el-row justify="center">
          <div :class="{ active: isActive('/stat') }" style="left: 32rem;" v-if="!u_type">
            <router-link to="/stat">统计信息</router-link>
          </div>
          </el-row>
        </span>
      </el-aside>
      <el-main>
        <router-view v-model:u_id="u_id" v-model:u_type="u_type"></router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      u_id: null,
      u_type: null
    }
  },
  mounted() {
    this.u_id = sessionStorage.getItem('u_id')
    this.u_type = JSON.parse(sessionStorage.getItem('u_type'))
    
  },
  methods: {
    logOut() {
      sessionStorage.removeItem('u_id')
      sessionStorage.removeItem('u_type')
      this.$notify({
        title: '登出成功',
        type: 'success'
      })
      this.u_id = null
      this.u_type = null
      this.$router.replace('/login')
    },

    isActive(path) {
      return this.$route.path == path
    }
  }
}
</script>

<style>
html {
  height: 100%;
}

body {
  margin: 0;
  background-color: #e9eef3;
  height: 100%;
}

#app {
  height: 100%;
}

.el-header {
  background-color: rgba(116,52,129,1);
  color: #FFFFFF;
  /*height: 5rem;
  line-height: 5rem;*/
  position: fixed;
  top: 0;
  width: 100%;
}

.el-aside {
  background-color: rgba(116,52,129,1);
  width: 11rem;
  top: 3.5rem
}

.el-main {
  margin-top: 3.5rem;
  padding: 0;
}

.el-notification {
  margin-top: 3.5rem;
}

#nav {
  font-size: 1.3rem;
  margin-top: 4rem;
}

#nav div {
  /*display: inline-block;*/
  height: 2rem;
  width: 10rem;
  text-align: center;
  margin-top: 3rem;
  margin-left: 0.1rem;
  /*position: inherit;*/
  /*top: 0;*/
}

#nav a {
  color: rgba(255, 255, 255, 0.8);
  text-decoration: unset;
}

#nav .active {
  background-color: rgba(0, 0, 0, 0.2);
  border-right: solid #ffa726;
  box-sizing: border-box;
}

#nav .active a {
  color: rgba(255, 255, 255, 1);
}
</style>
