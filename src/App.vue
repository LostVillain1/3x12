<script setup>
import { RouterLink, RouterView } from 'vue-router'
import MainPage from '@/views/MainPage.vue';

</script>


<template>
  <RouterView  :key="$route.fullPath"></RouterView>
</template>

<style scoped>

</style>
