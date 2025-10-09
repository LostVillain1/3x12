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

            <!-- ✦ NEW: миниатюры для КОНКРЕТНОГО цвета под главным фото -->
            <div v-if="activeImages.length" class="gallery-mobile-thumbs" aria-label="Миниатюры цвета">
              <button
                v-for="(img, tIdx) in activeImages"
                :key="`mt-${tIdx}`"
                type="button"
                class="gallery-mobile-thumb"
                :class="{ active: currentIndex === tIdx }"
                @click="goTo(tIdx)"
              >
                <img :src="img" :alt="`Миниатюра ${tIdx + 1}`" />
              </button>
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
            <!-- <p class="product-code">Арт: {{ product.code }}</p> -->

            <!-- ✦ ПРАВКА: форматирование из computed -->
            <p class="product-price">{{ productPrice }} ₽</p>
            <!-- <p class="product-installment">4 платежа по {{ productInstallment }} ₽</p> -->
          </div>

          <!-- ✦ NEW: выбор цвета через мини-карточки с фото + названием -->
          <div v-if="colorOptions.length" class="product-colors" aria-label="Выбор цвета">
            <p class="label">Цвет:  </p>

            <!-- мини-карточки (мобайл-фёрст). старые .color-btn оставил в стилях -->
            <div class="color-cards">
              <button
                v-for="c in colorOptions"
                :key="`color-card-${c.id}`"
                type="button"
                class="color-card"
                :class="{ active: c.id === selectedColorId }"
                @click="selectColor(c.id)"
              >
                <div class="color-card__img-wrap">
                  <img :src="colorThumb(c)" :alt="c.name" />
                </div>
                <span class="color-card__name">{{ c.name }}</span>
              </button>
            </div>
          </div>

          <!-- Размеры (с учётом цвета) -->
          <div class="product-sizes" v-if="sizeOptions.length">
            <p class="label">Размер:</p>
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

          <!-- <p>Таблица размеров</p> -->

          <accordion-product title="Описание" />
          <accordion-product title="Доставка и возврат" />
          <!-- <accordion-product title="Уход за одеждой" /> -->
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
import { ref, computed, watch, nextTick } from 'vue'
import { useWindowSize } from '@vueuse/core'
import { useRouter } from 'vue-router'

// import { useCartStore } from '@/stores/useCartStore'

import AccordionProduct from './AccordionProduct.vue'
import AddToCartModal from '@/components/ProductCard/AddToCartModal.vue'
import QuantitySelector from '@/components/ProductCard/QuantitySelector.vue'

/* ✦ ТОЛЬКО пропс от ProductPage */
const props = defineProps({
  product: { type: Object, required: true }
})

const router = useRouter()
// const cartStore = useCartStore()

/* локальное состояние UI */
const currentImage = ref('')
const currentIndex = ref(0)
const sliderRef = ref(null)

const selectedColorId = ref(null)
const selectedSize = ref('')
const quantity = ref(1)

const showAddToCartModal = ref(false)

const { width } = useWindowSize()
const isDesktop = computed(() => width.value >= 1024)

/* алиас */
const product = computed(() => props.product || null)

/* цена */
function formatCurrency(n) { return new Intl.NumberFormat('ru-RU').format(Number(n ?? 0)) }
const productPrice = computed(() => formatCurrency(product.value?.price))
const productInstallment = computed(() => formatCurrency(Math.floor(Number(product.value?.price ?? 0) / 4)))

/* цвета / изображения / размеры */
const colorOptions = computed(() => Array.isArray(product.value?.colors) ? product.value.colors : [])

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

/* ✦ NEW: превьюшки цветов (первая картинка цвета) */
const colorThumb = (c) => {
  const imgs = Array.isArray(c?.images) ? c.images : []
  return imgs[0] || ''
}

/* init */
function initFromProduct(p) {
  if (!p) return
  selectedColorId.value = p.defaultColor || p.colors?.[0]?.id || null
  currentIndex.value = 0
  currentImage.value = activeImages.value?.[0] || ''
  const firstInStock = sizeOptions.value.find(s => s.inStock)
  selectedSize.value = firstInStock ? firstInStock.size : (sizeOptions.value[0]?.size || '')
  quantity.value = 1
  nextTick(() => { sliderRef.value?.scrollTo?.({ left: 0, behavior: 'auto' }) })
}
watch(() => props.product, (p) => { initFromProduct(p) }, { immediate: true })

/* смена цвета */
function selectColor(id) {
  if (selectedColorId.value === id) return
  selectedColorId.value = id
  currentIndex.value = 0
  currentImage.value = activeImages.value?.[0] || currentImage.value
  const firstInStock = sizeOptions.value.find(s => s.inStock)
  if (firstInStock) selectedSize.value = firstInStock.size
  nextTick(() => { sliderRef.value?.scrollTo?.({ left: 0, behavior: 'auto' }) })
}

/* галерея */
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

/* размеры */
function onSelectSize(opt) {
  if (!opt?.inStock) return
  selectedSize.value = opt.size
}

/* модалка */
function openAddToCartModal(){ showAddToCartModal.value = true }
function closeAddToCartModal(){ showAddToCartModal.value = false }

/* действия (оставил закомментированными как просил) */
// function onAddToCart() { ... }
// function onBuyNow() { ... }
</script>

<style>
@import './ProductCard.scss';

/* ===== ДОБАВЛЕНО под макет, не ломает существующее ===== */

/* 1) миниатюры конкретного цвета (под главным фото, мобайл) */
.gallery-mobile-thumbs{
  display:flex;
  gap:8px;
  margin-top:8px;
  overflow-x:auto;
  padding-bottom:4px;
}
.gallery-mobile-thumb{
  flex:0 0 auto;
  width:64px;
  height:64px;
  border:1px solid #cfc7b8;
  border-radius:6px;
  background:#fff;
  padding:2px;
  cursor:pointer;
  opacity:.9;
}
.gallery-mobile-thumb.active{ border-color:#1E3035; opacity:1; }
.gallery-mobile-thumb img{
  width:100%;
  height:100%;
  object-fit:cover;
  border-radius:4px;
  display:block;
}

/* 2) выбор цвета — мини-карточки с фото и подписью */
.product-colors {
  margin-bottom: 16px;
}

.product-colors .color-cards{
  display:flex;
  gap:10px;
  overflow-x:auto;
  padding:4px 2px 8px;
}
.color-card{
  flex:0 0 auto;
  width:84px;
  border:1px solid transparent;
  border-radius:8px;
  background:#D5D9D9;
  padding:6px 6px 8px;
  text-align:center;
  cursor:pointer;
  color:#1E3035;
}
.color-card.active{ border-color:#1E3035; }
.color-card__img-wrap{
  width:100%;
  height:70px;
  border-radius:6px;
  overflow:hidden;
  border:1px solid #e1dacf;
  margin-bottom:6px;
}
.color-card__img-wrap img{
  width:100%;
  height:100%;
  object-fit:cover;
  display:block;
}
.color-card__name{
  display:block;
  font-size:14px;
  line-height:1.2;
  white-space:nowrap;
  overflow:hidden;
  text-overflow:ellipsis;
}

/* старые «кнопки-палитры» оставлены на будущее */
.product-colors p {
  font-size: 24px;
  margin-bottom: 16px;
}

.product-colors .colors-list { display:flex; flex-wrap:wrap; gap:12px; margin-bottom:10px; }
.color-btn { display:inline-flex; align-items:center; gap:8px; background:none; border:1px solid transparent; padding:6px 10px; border-radius:6px; cursor:pointer; }
.color-btn.active { border-color:#1E3035; }
.color-btn__swatch { width:18px; height:18px; border-radius:50%; border:1px solid rgba(0,0,0,.2); display:inline-block; }
.size-btn.is-disabled { opacity:.45; cursor:not-allowed; }

/* ============================
   TABLET OVERRIDES (768–1023)
   Макет: мобильная композиция,
   увеличенные элементы
   ============================ */
@media (min-width:768px){

  /* контейнер карточки */
  .product-card{ padding: 24px; }

  /* всё остаётся в одну колонку */
  .product-wrapper{
    flex-direction: column;          /* ← критично */
    align-items: stretch;
    gap: 24px;
    max-width: 720px;                /* центрируем контент по макету */
    margin: 0 auto;
  }

  /* хлебные крошки / заголовок */
  .nav{ font-size: 18px; letter-spacing: .10em; margin-bottom: 16px; }
  .product-title{ font-size: 36px; letter-spacing: .12em; margin-bottom: 18px; }

  /* галерея — используем мобильную версию, но крупнее */
  .gallery-mobile .slide{ padding-right: 12px; }
  .gallery-mobile img{ border-radius: 12px; }
  .gallery-mobile .dots{ gap: 10px; }

  /* миниатюры активного цвета под главным фото — крупнее */
  .gallery-mobile-thumbs{ gap: 10px; margin-top: 10px; }
  .gallery-mobile-thumb{ width: 180px; height: 180px; border-radius: 8px; padding: 4px}

  /* правая колонка (текст/кнопки) — ширина на всю строку */
  .product-info{
    max-width: 100%;                 /* ← убираем «узкую» колонку */
    margin-right: 0;
  }

  .product-name{ font-size: 24px; }
  .product-price{ font-size: 24px; margin: 22px 0; }

  /* выбор цвета — мини-карточки */
  .product-colors{ margin-bottom: 18px; }
  .product-colors p.label{ font-size: 20px; margin-bottom: 12px; }
  .product-colors .color-cards{ gap: 12px; }
  .color-card{ width: 116px; padding: 8px 8px 10px; border-radius: 10px; }
  .color-card__img-wrap{ height: 92px; border-radius: 8px; }
  .color-card__name{ font-size: 16px; }

  /* размеры — «пилюли» чуть крупнее и плотнее */
  .product-sizes{ margin-bottom: 26px; }
  .product-sizes .label{ font-size: 20px; margin-bottom: 10px; }
  .sizes-list{ gap: 20px; }
  .size-btn{
    height: 40px;
    min-width: 56px;
    padding: 0 16px;
    font-size: 18px;
    border-radius: 12px;
  }

  /* CTA */
  .product-qty-row{ margin-bottom: 20px; }
  .add-to-cart{
    font-size: 20px;
    min-height: 54px;
    border-radius: 10px;
  }
  .buy-now{
    font-size: 18px;
    min-height: 50px;
    border-radius: 10px;
  }

  .accordion-item{ border-radius: 10px; }
}

</style>
