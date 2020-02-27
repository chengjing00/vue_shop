<template>
  <div>
    Home 组件
    <el-button type="info" @click="logout">exit</el-button>
  </div>
</template>

<script>
  export default {
    name: 'Home',
    methods: {
      logout () {
        window.sessionStorage.clear()
        this.$router.push('/login')
        this.$message.success('退出成功')
      }
    }
  }

</script>

<style lang="less" scoped>

</style>
