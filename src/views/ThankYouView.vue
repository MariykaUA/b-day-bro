<script setup>
import { ref } from 'vue'
import voucherImg from '../voucher.png'

const isZoomed = ref(false)
</script>

<template>
  <main class="thanks">
    <h1 class="thanks__title">Thank you for choosing the best present ever!</h1>
    <p class="thanks__subtitle">With big love comes small addition.</p>

    <img
      class="voucher"
      :class="{ 'voucher--zoomed': isZoomed }"
      :src="voucherImg"
      alt="Gift voucher"
      @click="isZoomed = !isZoomed"
    />

    <p class="thanks__redeem">
      *If you want to redeem it, please take your sister to the store of your choice.
    </p>

    <p class="thanks__again">
      Not satisfied?
      <RouterLink to="/" class="thanks__again-link">Choose another present</RouterLink>
    </p>
  </main>
</template>

<style lang="scss" scoped>
.thanks {
  margin: 0 auto;
  padding: 3rem 1.25rem 4rem;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  gap: 0.75rem;

  &__title {
    font-size: clamp(1.75rem, 5vw, 2.5rem);
    font-weight: 800;
    color: #301e72;
    text-shadow: 0 4px 16px rgba(47, 47, 53, 0.15);
  }

  &__subtitle {
    font-size: 1.1rem;
    color: #091186;
  }

  &__redeem {
    color: #091186;
    font-size: 0.95rem;
  }

  &__again {
    margin-top: 1.5rem;
    color: #091186;
    font-size: 1rem;

    &-link {
      color: #091186;
      font-weight: 700;
      text-decoration: underline;
      transition: color 0.15s ease;

      &:hover {
        color: #4c70b8;
      }
    }
  }
}

.voucher {
  width: min(420px, 100%);
  margin-top: 1.5rem;
  margin-bottom: 0.5rem;
  border-radius: 16px;
  box-shadow: 0 14px 26px rgba(45, 36, 56, 0.25);
  cursor: zoom-in;
  transition: transform 0.3s ease;

  &--zoomed {
    position: relative;
    z-index: 10;
    transform: scale(2);
    cursor: zoom-out;
  }
}
</style>
