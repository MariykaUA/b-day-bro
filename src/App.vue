<script setup>
import FleeingConfetti from './components/FleeingConfetti.vue'
</script>

<template>
  <main class="page">
    <header class="hero">
      <h1 class="hero__title">Happy Birthday, Bro!</h1>
      <p class="hero__subtitle">
        This page does not want to make things easy for you. Good luck.
      </p>
    </header>

    <section class="card">
      <h2 class="card__title">Choose your prize</h2>
      <p class="card__hint">Only one of these boxes actually works.</p>
      <FleeingConfetti />
    </section>
  </main>

  <footer class="footer">
    <p>Made with ❤️ and questionable UX decisions.</p>
  </footer>
</template>

<style lang="scss" scoped>
.page {
  max-width: 720px;
  margin: 0 auto;
  padding: 2.5rem 1.25rem 4rem;
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

.hero {
  text-align: center;
  color: #301e72;
  display: flex;
  flex-direction: column;
  gap: 0.5rem;

  &__title {
    font-size: clamp(2.25rem, 6vw, 3.25rem);
    font-weight: 800;
    text-shadow: 0 4px 16px rgba(47, 47, 53, 0.15);
  }

  &__subtitle {
    font-size: 1.05rem;
    opacity: 0.95;
  }
}

.card {
  display: flex;
  flex-direction: column;
  text-align: center;
  gap: 0.5rem;
  margin-top: 1rem;
  color: #091186;

  &__title {
    font-size: 1.4rem;
    font-weight: 700;
    text-shadow: 0 2px 10px rgba(0, 0, 0, 0.15);
  }

  &__hint {
    color: #5E503F;
    margin-bottom: 0.5rem;
  }
}

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
</style>
