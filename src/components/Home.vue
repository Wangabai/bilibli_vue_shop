<!-- Home组件 -->
<template>
  <div>
    <el-button
      type="info"
      @click="logout"
    >
      退出
    </el-button>
  </div>
</template>

<script>
export default {
  name: 'Home',
  methods: {
    //   退出按钮
    logout () {
      // 清空token
      window.sessionStorage.clear()
      //   跳转到登录也
      this.$router.push('./login')
    }
  }
}
</script>
<style lang="less" scoped></style>
