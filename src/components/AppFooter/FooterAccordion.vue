<template>
  <div class="footer-accordion">
    <details
      v-for="(section, index) in sections"
      :key="`accordion-section-${section.id ?? index}`"
      class="accordion"
    >
      <summary class="accordion__summary">
        <span class="accordion__title">{{ section.title }}</span>
        <svg class="accordion__chevron" viewBox="0 0 20 20" aria-hidden="true">
          <path
            d="M5 7l5 6 5-6"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
            stroke-linecap="round"
          />
        </svg>
      </summary>

      <ul v-if="section.items?.length" class="accordion__list">
        <li
          v-for="(item, itemIndex) in section.items"
          :key="`accordion-link-${section.id ?? index}-${item.id ?? itemIndex}`"
          class="accordion__item"
        >
          <a :href="item.href || '#'" class="accordion__link">{{ item.text }}</a>
        </li>
      </ul>

      <p v-else-if="section.text" class="accordion__text">
        {{ section.text }}
      </p>
    </details>
  </div>
</template>

<script setup>
defineProps({
  /**
   * sections: [{ id?: string, title: string, items?: [{ id?: string, text: string, href?: string }] }]
   */
  sections: { type: Array, default: () => [] },
})
</script>

<style scoped>
.footer-accordion {
  width: 100%;
  display: grid;
  gap: 12px;
}

/* item */
.accordion {
  border: 1px solid rgba(0, 255, 255, 0.35);
  border-radius: 10px;
  overflow: hidden;
  background: rgba(255, 255, 255, 0.02);
}

.accordion[open] .accordion__chevron {
  transform: rotate(180deg);
}

/* header */
.accordion__summary {
  list-style: none;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 16px;
  cursor: pointer;
  font-size: 16px;
}
.accordion__summary::-webkit-details-marker { display: none; }

.accordion__title { font: inherit; }

.accordion__chevron {
  width: 18px;
  height: 18px;
  transition: transform .25s ease;
  opacity: .8;
}

/* content */
.accordion__list {
  padding: 6px 16px 14px;
  margin: 0;
  display: grid;
  gap: 8px;
}
.accordion__item { list-style: none; }

.accordion__link {
  color: #d7dede;
  text-decoration: none;
  opacity: .9;
}
.accordion__link:hover { opacity: 1; }

.accordion__text {
  padding: 0 16px 14px;
  margin: 0;
  color: #cfd6d6;
  opacity: .9;
}
</style>
