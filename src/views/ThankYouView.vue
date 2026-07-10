<script setup>
import { ref } from 'vue'
import voucherImg from '../voucher.png'

const isZoomed = ref(false)

const PARTICLE_COUNT = 18
const FIREWORK_COLORS = ['#ff5e78', '#ffd166', '#37c58f', '#5b8def', '#a18cd1', '#ff9a9e', '#ffffff']

function makeParticles() {
  return Array.from({ length: PARTICLE_COUNT }, (_, i) => ({
    angle: (360 / PARTICLE_COUNT) * i + (Math.random() * 10 - 5) + 'deg',
    distance: 70 + Math.random() * 50 + 'px',
  }))
}

const fireworks = Array.from({ length: 16 }, (_, i) => ({
  id: i,
  left: 15 + Math.random() * 70 + '%',
  top: 10 + Math.random() * 45 + '%',
  color: FIREWORK_COLORS[Math.floor(Math.random() * FIREWORK_COLORS.length)],
  delay: Math.random() * 4 + 's',
  particles: makeParticles(),
}))
</script>

<template>
  <div class="fireworks">
    <div
      v-for="fw in fireworks"
      :key="fw.id"
      class="firework"
      :style="{ left: fw.left, top: fw.top, animationDelay: fw.delay }"
    >
      <span class="firework__trail" :style="{ background: fw.color, animationDelay: fw.delay }" />
      <span class="firework__flash" :style="{ background: fw.color, animationDelay: fw.delay }" />
      <span
        v-for="(particle, i) in fw.particles"
        :key="i"
        class="firework__particle"
        :style="{
          '--angle': particle.angle,
          '--distance': particle.distance,
          background: fw.color,
          boxShadow: `0 0 6px 1px ${fw.color}`,
          animationDelay: fw.delay,
        }"
      />
    </div>
  </div>

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
.fireworks {
  position: fixed;
  inset: 0;
  z-index: -1;
  overflow: hidden;
  pointer-events: none;
}

.firework {
  position: absolute;
  width: 4px;
  height: 4px;
}

.firework__trail {
  position: absolute;
  left: -1px;
  width: 2px;
  height: 16px;
  border-radius: 2px;
  opacity: 0;
  animation: firework-trail 3.5s ease-out infinite;
}

.firework__flash {
  position: absolute;
  left: -8px;
  top: -8px;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  opacity: 0;
  filter: blur(1px);
  animation: firework-flash 3.5s ease-out infinite;
}

.firework__particle {
  position: absolute;
  width: 5px;
  height: 5px;
  border-radius: 50%;
  transform: rotate(var(--angle)) translateX(0);
  opacity: 0;
  animation: firework-burst 3.5s ease-out infinite;
}

@keyframes firework-trail {
  0% {
    transform: translateY(120vh);
    opacity: 1;
  }
  20% {
    transform: translateY(0);
    opacity: 1;
  }
  22% {
    opacity: 0;
  }
  100% {
    opacity: 0;
  }
}

@keyframes firework-flash {
  0%,
  19% {
    transform: scale(0);
    opacity: 0;
  }
  20% {
    transform: scale(0.5);
    opacity: 1;
  }
  32% {
    transform: scale(2.2);
    opacity: 0;
  }
  100% {
    opacity: 0;
  }
}

@keyframes firework-burst {
  0%,
  19% {
    transform: rotate(var(--angle)) translateX(0) scale(1);
    opacity: 0;
  }
  20% {
    transform: rotate(var(--angle)) translateX(4px) scale(1);
    opacity: 1;
  }
  75% {
    opacity: 0.9;
  }
  100% {
    transform: rotate(var(--angle)) translateX(var(--distance)) scale(0.3);
    opacity: 0;
  }
}

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
