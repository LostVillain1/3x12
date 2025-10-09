<template>
  <div class="acc">
    <button
      class="acc__header"
      @click="toggle"
      :aria-expanded="isOpen.toString()"
      type="button"
    >
      <span class="acc__title">{{ title }}</span>

      <!-- Иконка-стрелка: цвет берётся из currentColor -->
      <span class="acc__chevron" :class="{ 'is-open': isOpen }" aria-hidden="true">
        <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
          <path
            d="M17.0837 6.65836C16.972 6.54933 16.8222 6.48828 16.6662 6.48828C16.5101 6.48828 16.3603 6.54933 16.2487 6.65836L9.99991 12.7496L3.75241 6.65836C3.64079 6.54933 3.49094 6.48828 3.33491 6.48828C3.17887 6.48828 3.02902 6.54933 2.91741 6.65836C2.86298 6.71136 2.81971 6.77472 2.79018 6.84471C2.76064 6.9147 2.74542 6.9899 2.74542 7.06586C2.74542 7.14183 2.76064 7.21703 2.79018 7.28702C2.81971 7.35701 2.86298 7.42037 2.91741 7.47336L9.56366 13.9509C9.68037 14.0646 9.83692 14.1283 9.99991 14.1283C10.1629 14.1283 10.3194 14.0646 10.4362 13.9509L17.0824 7.47336C17.1368 7.42037 17.1801 7.35701 17.2096 7.28702C17.2392 7.21703 17.2544 7.14183 17.2544 7.06586C17.2544 6.9899 17.2392 6.9147 17.2096 6.84471C17.1801 6.77472 17.1368 6.71136 17.0824 6.65836H17.0837Z"
            fill="currentColor"
          />
        </svg>
      </span>

      <!-- альтернативный вариант, оставляем как было -->
      <!-- <span :class="{ rotated: isOpen }">⌄</span> -->
    </button>

    <div v-if="isOpen" class="acc__content">
      <slot>
        <p>Тут может быть любой контент.</p>
      </slot>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

defineProps({ title: String })

const isOpen = ref(false)
const toggle = () => { isOpen.value = !isOpen.value }
</script>

<style scoped>
/* Карточка-аккордеон */
.acc {
  background: rgba(255,255,255,0.18);
  border: 1px solid rgba(30, 48, 53, 0.55); /* #1E3035 с прозрачностью под макет */
  border-radius: 12px;
  padding: 12px 14px;
  color: #1E3035;
  margin: 12px 0;             /* расстояние между блоками */
}

/* Кнопка-шапка */
.acc__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  width: 100%;
  background: transparent;
  border: 0;
  padding: 4px 2px;
  cursor: pointer;
  color: inherit;
  font-family: 'PT Serif', serif;
  font-size: 20px;            /* мобильный размер как в макете */
  line-height: 1.2;
}

/* Фокус по доступности */
.acc__header:focus-visible {
  outline: 2px solid rgba(0, 255, 255, 0.45);
  outline-offset: 2px;
  border-radius: 8px;
}

/* Стрелка */
.acc__chevron {
  display: inline-flex;
  transition: transform .25s ease, opacity .2s ease;
  opacity: .9;
}
.acc__chevron.is-open { transform: rotate(180deg); }

/* Контент */
.acc__content {
  margin-top: 10px;
  padding-top: 6px;
  border-top: 1px solid rgba(30, 48, 53, 0.35);
  font-size: 16px;
  line-height: 1.45;
  color: #1E3035;
}

/* Планшет/десктоп: немного крупнее */
@media (min-width: 768px) {
  .acc { padding: 14px 16px; border-radius: 14px; }
  .acc__header { font-size: 24px; }
  .acc__content { font-size: 20px; }
}
</style>
