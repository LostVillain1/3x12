<template>
  <footer class="footer">
    <div class="footer__container">
      <!-- Подписка -->
      <div class="footer__subscribe">
        <h3 class="footer__heading">Узнавайте первыми о новинках</h3>
        <p class="footer__subheading">Получи скидку на первую покупку</p>

        <form class="footer__form" @submit.prevent>
          <label class="visually-hidden" for="footer-email">Email</label>
          <input
            id="footer-email"
            type="email"
            required
            placeholder="email"
            class="footer__input"
          />
          <button class="footer__button" type="submit">Подписаться</button>
        </form>
      </div>

      <!-- Соцсети -->
      <ul class="footer__social-list" aria-label="Социальные сети">
        <li><img class="footer__social-icon footer__social-icon--inst" alt="Instagram" /></li>
        <li><img class="footer__social-icon footer__social-icon--vk"   alt="VK" /></li>
        <li><img class="footer__social-icon footer__social-icon--tt"   alt="TikTok" /></li>
      </ul>

      <!-- Моб/планшет: аккордеоны -->
      <FooterAccordion class="footer__accordion" :sections="accordionSections" />

      <!-- Десктоп: колонки -->
      <nav class="footer__columns" aria-label="Навигация по разделам">
        <ul
          v-for="column in footerColumns"
          :key="`column-${column.id}`"
          class="footer__column"
        >
          <li
            v-for="link in column.links"
            :key="`link-${column.id}-${link.id}`"
            class="footer__item"
          >
            <a :href="link.href" class="footer__link">{{ link.text }}</a>
          </li>
        </ul>
      </nav>
    </div>

    <!-- Декоративное изображение в правом нижнем углу -->
    <div class="footer__decor" aria-hidden="true"></div>
  </footer>
</template>

<script setup>
import FooterAccordion from './FooterAccordion.vue'

const accordionSections = [
  {
    id: 'customers',
    title: 'Покупателям',
    items: [
      { id: 'account',     text: 'Личный кабинет', href: '#' },
      { id: 'collections', text: 'Каталог',        href: '#' },
      { id: 'about',       text: 'О бренде',       href: '#' },
    ],
  },
  {
    id: 'information',
    title: 'Информация',
    items: [
      { id: 'delivery', text: 'Доставка', href: '#' },
      { id: 'returns',  text: 'Возврат',  href: '#' },
      { id: 'payment',  text: 'Оплата',   href: '#' },
    ],
  },
  {
    id: 'documents',
    title: 'Документы',
    items: [
      { id: 'privacy', text: 'Политика конфиденциальности', href: '#' },
      { id: 'offer',   text: 'Оферта',                      href: '#' },
    ],
  },
]

const footerColumns = [
  {
    id: 'customers',
    links: [
      { id: 'account',     text: 'Личный кабинет', href: '#' },
      { id: 'collections', text: 'Каталог',        href: '#' },
      { id: 'about',       text: 'О бренде',       href: '#' },
    ],
  },
  {
    id: 'information',
    links: [
      { id: 'delivery', text: 'Доставка', href: '#' },
      { id: 'returns',  text: 'Возврат',  href: '#' },
      { id: 'payment',  text: 'Оплата',   href: '#' },
    ],
  },
  {
    id: 'documents',
    links: [
      { id: 'privacy', text: 'Политика конфиденциальности', href: '#' },
      { id: 'offer',   text: 'Оферта',                      href: '#' },
    ],
  },
]
</script>

<style lang="scss" scoped>
@import './AppFooter.scss';
</style> 
 
 