<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import notFoundGif from './404.gif'

const MIN_WIDTH = 1200
const windowWidth = ref(window.innerWidth)
const isTooSmall = computed(() => windowWidth.value < MIN_WIDTH)

function handleResize() {
  windowWidth.value = window.innerWidth
}

onMounted(() => {
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})
</script>

<template>
  <div v-if="isTooSmall" class="too-small">
    <img class="too-small__gif" :src="notFoundGif" alt="404 error" />
    <h1 class="too-small__title">This page is only visible on a computer or laptop.</h1>
    <p class="too-small__subtitle">
      Grab a bigger screen - the birthday surprises need room to breathe.
    </p>
  </div>

  <template v-else>
    <router-view />

    <footer class="footer">
      <p>Made with ❤️ and questionable UX decisions.</p>
    </footer>
  </template>
</template>

<style lang="scss" scoped>
.footer {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  text-align: center;
  color: #5E503F;
  font-size: 0.9rem;
  padding: 0.75rem 1rem;
}

.too-small {
  width: 100%;
  min-height: 100vh;
  background: #EFE0FC;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  gap: 0.75rem;
  padding: 2rem 1.5rem;

  &__gif {
    width: min(280px, 80%);
    border-radius: 16px;
    margin-bottom: 1rem;
  }

  &__title {
    font-size: 1.5rem;
    font-weight: 800;
    color: #301e72;
    text-shadow: 0 4px 16px rgba(47, 47, 53, 0.15);
  }

  &__subtitle {
    font-size: 1rem;
    color: #091186;
    max-width: 520px;
  }
}
</style>
