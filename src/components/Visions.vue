<!-- src/components/VisionOfTomorrow.vue -->
<template>
  <section class="vision" ref="rootRef">
    <div class="vision__content">
      <div class="vision__text anim anim--1">
        <h2 class="vision__title">Vision of <br />Tomorrow</h2>
        <p class="vision__subtitle">
          Создавай настоящее <br />Вдохновляйся будущим
        </p>
      </div>

      <!-- соцсети (иконки подменяются через CSS) -->
      <ul class="vision__social anim anim--2" aria-label="Социальные сети">
        <li><img class="icon icon--inst" alt="Instagram" /></li>
        <li><img class="icon icon--vk"   alt="VK" /></li>
        <li><img class="icon icon--tt"   alt="TikTok" /></li>
      </ul>

      <div class="vision__photo anim anim--3">
        <img src="@/assets/model.png" alt="Фотография модели на корте" />
      </div>

      <div class="vision__logoImg anim anim--4" aria-hidden="true">
        <img src="@/assets/blur_logo.svg" alt="" />
      </div>
    </div>

    <button class="vision__btn">Смотреть</button>
  </section>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const rootRef = ref(null)
let observer = null

onMounted(() => {
  if (!rootRef.value) return
  const reduce = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  if (reduce) {
    rootRef.value.classList.add('is-visible')
    return
  }
  observer = new IntersectionObserver((entries) => {
    if (entries[0]?.isIntersecting) {
      rootRef.value.classList.add('is-visible')
      observer && observer.disconnect()
    }
  }, { threshold: 0.4 })
  observer.observe(rootRef.value)
})
onBeforeUnmount(() => observer?.disconnect())
</script>

<style scoped>
/* ===== MOBILE FIRST ===== */
.vision{
  background: #001417 url("@/assets/mob_bg.png") center/cover no-repeat;
  color:#D5D9D9;
  position: relative;
  overflow: hidden;
  min-height: 100vh; min-height: 100svh; min-height: 100dvh;

  --pad-x: 20px;
  --pad-y: 24px;

  --photo-size: 260px;
  --photo-top: 210px;
  --icons-shift: -8px;

  padding: var(--pad-y) var(--pad-x);
  display:flex; justify-content:center;
}

.vision__content{ width:100%; position:relative; display:flex; flex-direction:column; align-items:flex-start; }

/* текст */
.vision__title{ font-size:34px; font-weight:600; line-height:1.12; margin:0 0 14px; }
.vision__subtitle{ font-size:18px; line-height:1.38; margin:0; }

/* соцсети (моб) */
.vision__social{
  list-style:none; padding:0; margin:0;
  position:absolute; left:6px;
  top: calc(var(--pad-y) + var(--photo-top) + var(--photo-size) + var(--icons-shift));
  display:flex; flex-direction:column; gap:14px;
}
.icon{ display:block; width:22px; height:auto; opacity:.9; transition:opacity .25s ease; }
.icon:hover{ opacity:1; }
.icon--inst{ content: url("@/assets/inst.svg"); }
.icon--vk  { content: url("@/assets/vk.svg"); }
.icon--tt  { content: url("@/assets/tt.svg"); }

/* фото */
.vision__photo{ position:absolute; left:50%; top: calc(var(--pad-y) + var(--photo-top)); transform:translateX(-50%); z-index:2; }
.vision__photo img{ width: var(--photo-size); border-radius:6px; transform: rotate(-7deg); box-shadow:0 8px 28px rgba(0,0,0,.45); }
.vision__photo::before{ content:""; position:absolute; inset:-16px -18px -20px -16px; background: rgba(0,0,0,.25); transform: rotate(-3deg); border-radius:8px; filter: blur(1px); z-index:-1; }

/* 3X12 */
.vision__logoImg{ position:absolute; left:50%; transform:translateX(-50%); bottom: 132px; z-index:1; pointer-events:none; }
.vision__logoImg img{ width:320px; opacity:.7; filter: blur(2px); }

/* кнопка (моб — fixed) */
.vision__btn{
  border:1px solid #00ffff; background:transparent; color:#D5D9D9;
  border-radius:999px; font-size:16px; padding:14px 0; line-height:1;
  cursor:pointer; text-align:center; transition: background .25s ease, transform .15s ease;
  z-index: 5; width:min(360px, calc(100% - 40px));
}
@media (max-width: 767.98px){
  .vision__btn{ position: absolute; left: 50%; transform: translateX(-50%); bottom: 10%; }
}

/* ===== TABLET ===== */
@media (min-width: 768px) and (max-width: 1199.98px){
  .vision{
    background-image: url("@/assets/tb_bg.png");
    --pad-x: 40px; --pad-y: 80px;
    --photo-size: 330px; --photo-top: 140px; --icons-shift: 0px;
  }
  .vision__title{ font-size:44px; }
  .vision__title br{ display:none; }
  .vision__subtitle{ font-size:20px; margin-top:6px; }
  .vision__photo{ top: calc(var(--pad-y) + var(--photo-top)); }
  .vision__photo img{ width: var(--photo-size); transform: rotate(-8deg); }

  .vision__social{ left: calc(var(--pad-x) - 6px); top: calc(var(--pad-y) + 120px); gap: 18px; }
  .icon{ width:24px; }
  .icon--inst{ content: url("@/assets/inst_tb.svg"); }
  .icon--vk  { content: url("@/assets/vk_tb.svg"); }
  .icon--tt  { content: url("@/assets/tt_tb.svg"); }

  .vision__btn{ position: absolute; bottom: 10%; width:372px; padding:14px 36px; font-size:16px; }
  .vision__logoImg{ bottom:140px; left: auto}
  .vision__logoImg img{ width:420px; }
}

/* ===== DESKTOP (правки) ===== */
@media (min-width: 1200px){
  .vision{
    background-image: url("@/assets/desc_bg.png");
    --pad-x: 120px; --pad-y: 100px;
    --photo-size: 420px;
    display:flex; align-items:center;
  }

  .vision__content{ width:100%; height: calc(100dvh - var(--pad-y)*2); }

  /* 1) убираем перенос и смещаем блок текста левее */
  .vision__title{ font-size:56px; line-height:1.1; }
  .vision__title br{ display:none; }
  .vision__subtitle{ font-size:22px; margin-top:8px; }

  .vision__text{
    position:absolute;
    top: calc(var(--pad-y) - 10px);
    max-width: 560px;
  }

  /* 2) фото выше */
  .vision__photo{
    position:absolute;
    right: var(--pad-x);
    top: 0%; 
    left: auto;                  /* было 50% */
    transform: translateY(-50%);
  }
  .vision__photo img{ width: var(--photo-size); transform: rotate(-11deg); }
  .vision__photo::before{ inset:-20px -26px -26px -20px; transform: rotate(-4deg); }

  /* 3) кнопка — fixed у низа экрана слева */
  .vision__btn{
    position: absolute;
    width: 370px;
    left: 120px;                 /* одинаково с текстом */
    bottom: 15%;
    transform: none;
    font-size:18px;
    padding:14px 44px; 
  }

  /* 4) соцсети — в правом нижнем углу */
  .vision__social{
    position: fixed;
    right: 64px;
    bottom: 36px;
    left: auto; top: auto;
    display:flex; flex-direction:column; align-items:center; gap:18px;
  }
  .vision__social .icon{ width:28px; }

  /* 3X12 оставил как было — при необходимости сместите */
  .vision__logoImg{ position:absolute; left: calc(var(--pad-x)); bottom: 140px; z-index:1; }
  .vision__logoImg img{ width: clamp(520px, 38vw, 640px); filter: blur(3px); opacity:.75; }
}

/* ===== анимация ===== */
.anim{ opacity:0; transform: translateY(20px); transition: opacity 700ms ease, transform 700ms ease; will-change: opacity, transform; }
.is-visible .anim{ opacity:1; transform:none; }
.anim--1{ transition-delay:0ms; } .anim--2{ transition-delay:140ms; }
.anim--3{ transition-delay:280ms; } .anim--4{ transition-delay:420ms; }
@media (prefers-reduced-motion: reduce){
  .anim, .is-visible .anim{ transition:none!important; opacity:1!important; transform:none!important; }
}
</style>
