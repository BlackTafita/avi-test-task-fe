<script setup lang="ts">
import { onMounted } from "vue";
import { RouterLink, RouterView } from 'vue-router'

onMounted(() => {

})
</script>

<template>
  <div class="common-layout">
    <el-container>
      <el-header>
        <nav>
          <RouterLink to="/">Home</RouterLink>
          <RouterLink to="/about">About</RouterLink>
          <RouterLink to="/products">Products</RouterLink>
        </nav>
      </el-header>
      <el-main>
        <div class="router-view">
          <RouterView />
        </div>
      </el-main>
      <el-footer>Footer</el-footer>
    </el-container>
  </div>
</template>

<style scoped>
  .el-header {
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .router-view {
    min-height: 100vh;
    display: flex;
    flex-direction: column;
    align-items: center;
  }
</style>
