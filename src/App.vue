<script setup>
import { RouterLink, RouterView } from 'vue-router'
</script>

<template>
  <div>
    <RouterLink to="/signup">註冊</RouterLink> / <RouterLink to="/login">登入</RouterLink> /
    <RouterLink to="todo">Todo</RouterLink>
  </div>
  <RouterView />
</template>

<style scoped></style>
