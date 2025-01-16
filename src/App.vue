<script setup lang="ts">
import infoView from './components/infoView.vue';
</script>

<template>
  <div id="app">
    <el-container style="min-height: 100vh">
      <el-container>
        <el-header style="background-color: #409EFF; text-align: center; font-size: 24px; height: 60px; padding-top: 20px;">
          信息管理系统
        </el-header>

        <el-main>
          <info-view/>
        </el-main>
      </el-container>
    </el-container>
  </div>
</template>

<style scoped>
#app{
  min-height: 100vh;
}
</style>
