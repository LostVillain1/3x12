<template>
  <AppHeader />
  <ProductCard v-if="product" :product="product" />
  <div v-else>Товар не найден</div>
  <AppFooter />
</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import { useRoute } from 'vue-router'
import { useProductStore } from '@/stores/useProductStore'
import ProductCard from '@/components/ProductCard/ProductCard.vue'
import AppHeader from '@/components/AppHeader/AppHeader.vue'
import AppFooter from '@/components/AppFooter/AppFooter.vue'

const product = ref(null)
const route = useRoute()
const store = useProductStore()

async function loadProduct () {
  if (!store.products.length) {
    await store.fetchProducts()
  }
  const productId = Number(route.params.id)
  product.value = Number.isFinite(productId)
    ? (store.products.find(p => p.id === productId) || null)
    : null
}

onMounted(loadProduct)
watch(() => route.params.id, loadProduct)
</script>
