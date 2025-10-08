<template>
    <div class="header__wrapper">
        <div class="burger__menu" v-if="isMobile || isTablet">
            <img src="@/assets/BurgerMenu.svg " @click="toggleMenu" class="burger-icon" :class="{ 'open': isMenuOpen }"></img>
            <!-- <BurgerMenuIcon @click="toggleMenu" class="burger-icon" :class="{ 'open': isMenuOpen }"></BurgerMenuIcon> -->
            <transition name="menu">
                <div v-if="isMenuOpen" class="bg-menu_options">
                    <router-link to="/">Каталог</router-link>
                    <router-link to="/">О бренде</router-link>
                    <router-link to="/">Корзина</router-link>
                    <!-- <router-link to="/favourites">Избранное</router-link>                   -->
                </div>
            </transition>
        </div>
        <div class="header__options left" v-if="isDesktop">
            <router-link to="/">Каталог</router-link>
            <router-link to="/" class="brand">О бренде</router-link>
            <!-- <router-link to="/">Корзина</router-link> -->
            <!-- <div>Каталог</div>
            <div>О бренде</div>
            <div>Бутики</div> -->
        </div>
        <div class="brand__logo center">
            <router-link to="/">
                <img src="@/assets/Logo.svg "></img>
            </router-link>           
        </div>
        <div class="right-side_wrapper right">            
            <!-- <div class="favourites" v-if="!isMobile && !isTablet">
                <router-link to="/favourites">
                    <FavouritesHeader></FavouritesHeader>
                </router-link>                
            </div> -->
            <div class="cart">
                <img src="@/assets/Cart.svg" alt="">
                <!-- <router-link :to="{ name: 'cart' }">
                    <CartHeader></CartHeader>
                </router-link>                 -->
            </div>
        </div>        
    </div>
</template>

<script setup>
import { reactive, ref, computed, onMounted, onUnmounted } from 'vue';


const isMobile = ref(false)
const isTablet = ref(false)
const isDesktop = ref(false)

const checkScreenSize = () => {
    const width = window.innerWidth
    isMobile.value = width < 768
    isTablet.value = width >= 768 && width < 1024
    isDesktop.value = width >= 1024
}

onMounted(() => {
    checkScreenSize()
    window.addEventListener('resize', checkScreenSize)
})

onUnmounted(() => {
    window.removeEventListener('resize', checkScreenSize)
})


//Работа с отображением опций бургер меню
const isMenuOpen = ref(false);

const toggleMenu = () => {
  isMenuOpen.value = !isMenuOpen.value;
};

const closeMenu = () => {
  isMenuOpen.value = false; // Закрываем меню при клике на одну из опций
};


</script>

<style lang="scss" scoped>
@import "./AppHeader.scss";  
</style>