<template>
  <section class="product-card">
    <div v-if="product">
      <!-- Хлебные крошки -->
      <router-link to="/catalog">
        <nav class="nav">
          <span class="text">Главная // </span>
          <span class="text">Каталог </span>
        </nav>
      </router-link>

      <!-- Заголовок товара -->
      <div class="product-title">
        <span>{{ product.name }}</span>
      </div>

      <div class="product-wrapper">
        <!-- ===== Галерея ===== -->
        <div class="gallery">
          <!-- MOBILE: слайдер -->
          <div v-if="!isDesktop && activeImages.length" class="gallery-mobile">
            <div class="slides" ref="sliderRef" @scroll.passive="onSliderScroll">
              <div class="slide" v-for="(img, idx) in activeImages" :key="`m-${idx}`">
                <img :src="img" :alt="`${product.name} ${idx + 1}`" />
              </div>
            </div>
            <div v-if="activeImages.length > 1" class="dots">
              <button
                v-for="(img, idx) in activeImages"
                :key="`dot-${idx}`"
                :class="{ active: currentIndex === idx }"
                @click="goTo(idx)"
                type="button"
                :aria-label="`К слайду ${idx + 1}`"
              />
            </div>
          </div>

          <!-- DESKTOP: превью слева + большое фото -->
          <div v-else class="gallery-desktop">
            <div class="gallery-thumbnails" v-if="activeImages.length">
              <img
                v-for="(img, index) in activeImages"
                :key="`t-${index}`"
                :src="img"
                :alt="`${product.name} превью ${index + 1}`"
                :class="{ active: currentImage === img }"
                @click="changeImage(img)"
              />
            </div>
            <div class="product-image">
              <img :src="currentImage" :alt="product.name" />
            </div>
          </div>
        </div>

        <!-- ===== Правая колонка ===== -->
        <div class="product-info">
          <div class="info">
            <h1 class="product-name">{{ product.name }}</h1>
            <p class="product-code">Арт: {{ product.code }}</p>

            <!-- ✦ ПРАВКА: форматирование из computed -->
            <p class="product-price">{{ productPrice }} ₽</p>
            <!-- <p class="product-installment">4 платежа по {{ productInstallment }} ₽</p> -->
          </div>

          <!-- ✦ NEW: выбор цвета -->
          <div
            v-if="colorOptions.length"
            class="product-colors"
            aria-label="Выбор цвета"
          >
            <p class="label">Цвет</p>
            <div class="colors-list">
              <button
                v-for="c in colorOptions"
                :key="`color-${c.id}`"
                type="button"
                class="color-btn"
                :class="{ active: c.id === selectedColorId }"
                :title="c.name"
                @click="selectColor(c.id)"
              >
                <span class="color-btn__swatch" :style="{ backgroundColor: c.hex || '#ccc' }" />
                <span class="color-btn__name">{{ c.name }}</span>
              </button>
            </div>
          </div>

          <!-- Размеры (с учётом цвета) -->
          <div class="product-sizes" v-if="sizeOptions.length">
            <p class="label">Размер</p>
            <div class="sizes-list">
              <button
                v-for="opt in sizeOptions"
                :key="`size-${opt.size}`"
                :class="['size-btn', { active: selectedSize === opt.size, 'is-disabled': !opt.inStock }]"
                :disabled="!opt.inStock"
                @click="onSelectSize(opt)"
              >
                {{ opt.size }}
              </button>
            </div>
          </div>

          <!-- ✦ ПРАВКА: CTA-блоки под макет -->
          <div class="cta-row">
            <div class="product-qty-row">
              <QuantitySelector v-model="quantity" :min="1" />
            </div>

            <button
              class="add-to-cart"
              :disabled="!canAddToCart"
              @click="onAddToCart"
              type="button"
              title="Добавить в корзину"
            >
              В корзину
            </button>
          </div>

          <button
            class="buy-now"
            :disabled="!canAddToCart"
            @click="onBuyNow"
            type="button"
            title="Купить сейчас"
          >
            Быстрая покупка
          </button>

          <p>Таблица размеров</p>

          <accordion-product title="Состав" />
          <accordion-product title="Характеристики" />
          <accordion-product title="Уход за одеждой" />
        </div>
      </div>
    </div>

    <div v-else class="not-found">Товар не найден</div>

    <!-- Модальное подтверждение добавления -->
    <AddToCartModal
      v-if="showAddToCartModal"
      :product="product"
      @close="closeAddToCartModal"
      @confirm="() => { closeAddToCartModal(); router.push({ name: 'cart' }) }"
    />
  </section>
</template>

<script setup>
/**
 * ProductCard.vue
 * ✦ Mobile-first.
 * ✦ Работает ТОЛЬКО от пропса :product (источник — ProductPage).
 * ✦ Добавлен выбор цвета; галерея и размеры зависят от выбранного цвета.
 * ✦ Количество — кастомный QuantitySelector.
 * ✦ CTA для десктопа: количество + "Добавить" в ряд, "Быстрая покупка" ниже.
 * ✦ Исправлена ошибка _ctx.formatPrice через computed.
 */
import { ref, computed, watch, nextTick } from 'vue'
import { useWindowSize } from '@vueuse/core'
import { useRouter } from 'vue-router'

// import { useCartStore } from '@/stores/useCartStore'

import AccordionProduct from './AccordionProduct.vue'
import AddToCartModal from '@/components/ProductCard/AddToCartModal.vue'
import QuantitySelector from '@/components/ProductCard/QuantitySelector.vue'

/* ✦ ВАЖНО: получаем товар ТОЛЬКО из пропса */
const props = defineProps({
  product: { type: Object, required: true }
})

const router = useRouter()
// const cartStore = useCartStore()

/* локальное состояние UI */
const currentImage = ref('')
const currentIndex = ref(0)
const sliderRef = ref(null)

const selectedColorId = ref(null) // 'bordo' и т.п.
const selectedSize = ref('')
const quantity = ref(1)

const showAddToCartModal = ref(false)

const { width } = useWindowSize()
const isDesktop = computed(() => width.value >= 1024)

/* удобный алиас на пропс (в шаблоне — product, как и раньше) */
const product = computed(() => props.product || null)

/* форматирование цены (как было) */
function formatCurrency(n) {
  return new Intl.NumberFormat('ru-RU').format(Number(n ?? 0))
}
const productPrice = computed(() => formatCurrency(product.value?.price))
const productInstallment = computed(() =>
  formatCurrency(Math.floor(Number(product.value?.price ?? 0) / 4))
)

/* === ДАННЫЕ ПО ЦВЕТУ / ГАЛЕРЕЕ / РАЗМЕРАМ === */
const colorOptions = computed(() =>
  Array.isArray(product.value?.colors) ? product.value.colors : []
)

const activeColor = computed(() => {
  if (!colorOptions.value.length || !selectedColorId.value) return null
  return colorOptions.value.find(c => c.id === selectedColorId.value) || null
})

const activeImages = computed(() => {
  const imgs = activeColor.value?.images
  if (Array.isArray(imgs) && imgs.length) return imgs
  const base = product.value?.images
  return Array.isArray(base) && base.length ? base : []
})

const sizeOptions = computed(() => {
  if (Array.isArray(activeColor.value?.sizes) && activeColor.value.sizes.length) {
    return activeColor.value.sizes.map(s => ({ size: s.size, inStock: !!s.inStock }))
  }
  if (Array.isArray(product.value?.sizes) && product.value.sizes.length) {
    return product.value.sizes.map(s => ({ size: s, inStock: true }))
  }
  return []
})

const canAddToCart = computed(() => {
  const opts = sizeOptions.value
  if (!opts.length) return true
  if (!selectedSize.value) return false
  const opt = opts.find(o => o.size === selectedSize.value)
  return !!(opt && opt.inStock)
})

/* === ИНИЦИАЛИЗАЦИЯ И ПЕРЕИНИЦИАЛИЗАЦИЯ === */
function initFromProduct(p) {
  if (!p) return
  // Выставляем цвет
  selectedColorId.value = p.defaultColor || p.colors?.[0]?.id || null

  // Картинки
  currentIndex.value = 0
  currentImage.value = activeImages.value?.[0] || ''

  // Размер — первый доступный
  const firstInStock = sizeOptions.value.find(s => s.inStock)
  selectedSize.value = firstInStock ? firstInStock.size : (sizeOptions.value[0]?.size || '')

  // Количество
  quantity.value = 1

  // Сброс скролла мобильного слайдера
  nextTick(() => {
    sliderRef.value?.scrollTo?.({ left: 0, behavior: 'auto' })
  })
}

/* реагируем на смену товара (навигация по ссылкам) */
watch(() => props.product, (p) => {
  initFromProduct(p)
}, { immediate: true })

/* при смене цвета переинициализируем галерею и размер */
function selectColor(id) {
  if (selectedColorId.value === id) return
  selectedColorId.value = id
  currentIndex.value = 0
  currentImage.value = activeImages.value?.[0] || currentImage.value
  const firstInStock = sizeOptions.value.find(s => s.inStock)
  if (firstInStock) selectedSize.value = firstInStock.size
  nextTick(() => {
    sliderRef.value?.scrollTo?.({ left: 0, behavior: 'auto' })
  })
}

/* === ГАЛЕРЕЯ === */
function changeImage(img) { currentImage.value = img }
function goTo(idx) {
  if (!sliderRef.value) return
  const w = sliderRef.value.clientWidth || 0
  currentIndex.value = Math.max(0, Math.min(idx, activeImages.value.length - 1))
  sliderRef.value.scrollTo({ left: w * currentIndex.value, behavior: 'smooth' })
  currentImage.value = activeImages.value?.[currentIndex.value] || currentImage.value
}
function onSliderScroll() {
  if (!sliderRef.value) return
  const w = sliderRef.value.clientWidth || 1
  const idx = Math.round(sliderRef.value.scrollLeft / w)
  if (idx !== currentIndex.value) {
    currentIndex.value = idx
    currentImage.value = activeImages.value?.[idx] || currentImage.value
  }
}

/* === РАЗМЕР === */
function onSelectSize(opt) {
  if (!opt?.inStock) return
  selectedSize.value = opt.size
}

/* === МОДАЛКА === */
function openAddToCartModal(){ showAddToCartModal.value = true }
function closeAddToCartModal(){ showAddToCartModal.value = false }

/* === ДЕЙСТВИЯ === */
// function onAddToCart() {
//   if (!product.value) return
//   if (!canAddToCart.value) { alert('Выберите размер'); return }

//   cartStore.addToCart(product.value, {
//     selectedColor: selectedColorId.value, // ✦ NEW
//     selectedSize: selectedSize.value,
//     quantity: quantity.value
//   })
//   openAddToCartModal()
// }
// function onBuyNow() {
//   if (!product.value) return
//   if (!canAddToCart.value) { alert('Выберите размер'); return }

//   cartStore.addToCart(product.value, {
//     selectedColor: selectedColorId.value, // ✦ NEW
//     selectedSize: selectedSize.value,
//     quantity: quantity.value
//   })
//   router.push({ name: 'cart' })
// }
</script>

<style>
@import './ProductCard.scss';

/* ✦ НОВОЕ (аккуратно, не ломает твои стили) */
.product-colors .colors-list { display:flex; flex-wrap:wrap; gap:12px; margin-bottom:10px; }
.color-btn { display:inline-flex; align-items:center; gap:8px; background:none; border:1px solid transparent; padding:6px 10px; border-radius:6px; cursor:pointer; }
.color-btn.active { border-color:#39213D; }
.color-btn__swatch { width:18px; height:18px; border-radius:50%; border:1px solid rgba(0,0,0,.2); display:inline-block; }
.size-btn.is-disabled { opacity:.45; cursor:not-allowed; }
</style>
