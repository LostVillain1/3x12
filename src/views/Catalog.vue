<template>
  <AppHeader />

  <div class="catalog">
    <!-- Заголовок -->
    <h1 class="catalog__title">Каталог</h1>

    <!-- Сетка товаров (без фильтрации по категориям) -->
    <div class="catalog__grid">
      <ProductItem
        v-for="product in allProducts"
        :key="product.id"
        :product="product"
      />
    </div>
  </div>

  <AppFooter />
</template>

<script setup>
import { computed, onMounted } from 'vue'
import { useProductStore } from '../stores/useProductStore'
import ProductItem from '../components/ProductItem.vue'
import AppHeader from '../components/AppHeader/AppHeader.vue'
import AppFooter from '../components/AppFooter/AppFooter.vue'

const store = useProductStore()

onMounted(() => {
  // грузим товары (все — «для женщин» по твоему требованию)
  store.fetchProducts()
})

// без категорий: просто отдаём всё, что есть
const allProducts = computed(() => store.products)
</script>

<style lang="scss" scoped>
@import '@/views/Catalog.scss'
</style>