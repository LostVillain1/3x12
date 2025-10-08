<!-- src/views/MainPage.vue -->
<template>
  <div class="mp__wrapper">
    <AppHeader />
    <Visions></Visions>
  </div>
  <button @click="$router.push({ name: 'catalog' })">Go Catalog</button>
  <AppFooter />
</template>

<script setup>
import { ref, onMounted } from 'vue'
import AppHeader from '@/components/AppHeader/AppHeader.vue'
import Visions from '@/components/Visions.vue';
import AppFooter from '@/components/AppFooter/AppFooter.vue'


</script>

<style lang="scss" scoped>
@import '@/views/MainPage.scss';
</style>
