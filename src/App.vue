<script setup></script>

<template>
  <router-view />

  <footer class="footer">
    <p>Made with ❤️ and questionable UX decisions.</p>
  </footer>
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
</style>
