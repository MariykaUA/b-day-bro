<script setup>
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue'
import HonestConfetti from './HonestConfetti.vue'
import loveImg from '../love.png'
import passportImg from '../passport.png'
import salaryImg from '../salary.png'

const DODGE_RADIUS = 120
const IDLE_MS = 5000
const CARD_WIDTH = 150
const CARD_HEIGHT = 180

const PRIZES = [
  {
    img: passportImg,
    title: 'Swiss Passport',
    desc: 'Perfect german is included.',
    outOfStock: true,
  },
  { img: salaryImg, title: 'Million dollars', desc: 'To buy some treats.', runaway: true },
  { img: loveImg, title: 'Endless Sis Love', desc: 'Unlimited hugs, zero refunds.' },
]

const winnableIndices = PRIZES.reduce((acc, prize, i) => {
  if (!prize.runaway && !prize.outOfStock) acc.push(i)
  return acc
}, [])
const winnerIndex = winnableIndices[Math.floor(Math.random() * winnableIndices.length)]

const cards = reactive(
  PRIZES.map((prize, i) => ({
    ...prize,
    id: i,
    isWinner: i === winnerIndex,
    wrong: false,
  })),
)

const staticCards = computed(() => cards.filter((card) => !card.runaway))
const runawayCard = computed(() => cards.find((card) => card.runaway))

const wrongCount = ref(0)
const success = ref(false)
const confettiRef = ref(null)
const showOutOfStock = ref(false)

const runawayEl = ref(null)
const runawayPos = reactive({ x: 0, y: 0 })
const runawayDodges = ref(0)
const isEscaped = ref(false)
let idleTimer = null

function randomViewportPos(width, height) {
  const maxX = Math.max(0, window.innerWidth - width)
  const maxY = Math.max(0, window.innerHeight - height)
  return {
    x: Math.random() * maxX,
    y: Math.random() * maxY,
  }
}

function resetIdleTimer() {
  if (idleTimer) window.clearTimeout(idleTimer)
  idleTimer = window.setTimeout(() => {
    isEscaped.value = false
  }, IDLE_MS)
}

function jumpAway() {
  isEscaped.value = true
  const { x, y } = randomViewportPos(CARD_WIDTH, CARD_HEIGHT)
  runawayPos.x = x
  runawayPos.y = y
  runawayDodges.value += 1
  resetIdleTimer()
}

function handleGlobalMouseMove(event) {
  if (!runawayEl.value) return

  const rect = runawayEl.value.getBoundingClientRect()
  const centerX = rect.left + rect.width / 2
  const centerY = rect.top + rect.height / 2
  const distance = Math.hypot(event.clientX - centerX, event.clientY - centerY)

  if (distance < DODGE_RADIUS) {
    jumpAway()
  }
}

function handleRunawayPointerDown(event) {
  event.preventDefault()
  jumpAway()
}

function pick(card) {
  if (success.value) return

  if (card.outOfStock) {
    showOutOfStock.value = true
    return
  }

  if (card.isWinner) {
    success.value = true
    confettiRef.value?.launch()
    return
  }

  wrongCount.value += 1
  card.wrong = true
  window.setTimeout(() => {
    card.wrong = false
  }, 200)
}

onMounted(() => {
  window.addEventListener('mousemove', handleGlobalMouseMove)
})

onUnmounted(() => {
  window.removeEventListener('mousemove', handleGlobalMouseMove)
  if (idleTimer) window.clearTimeout(idleTimer)
})
</script>

<template>
  <div class="prize-row">
    <button
      v-for="card in staticCards"
      :key="card.id"
      class="prize-card"
      :class="{ 'prize-card--wrong': card.wrong }"
      @click="pick(card)"
    >
      <img class="prize-card__img" :src="card.img" :alt="card.title" />
      <h3 class="prize-card__title">{{ card.title }}</h3>
      <p class="prize-card__desc">{{ card.wrong ? "Nope, it's empty." : card.desc }}</p>
    </button>

    <button
      v-if="!isEscaped"
      ref="runawayEl"
      class="prize-card"
      :class="{ 'prize-card--wrong': runawayCard.wrong }"
      @click="pick(runawayCard)"
      @mouseenter="jumpAway"
      @pointerdown="handleRunawayPointerDown"
    >
      <img class="prize-card__img" :src="runawayCard.img" :alt="runawayCard.title" />
      <h3 class="prize-card__title">{{ runawayCard.title }}</h3>
      <p class="prize-card__desc">
        {{ runawayCard.wrong ? "Nope, it's empty." : runawayCard.desc }}
      </p>
    </button>
  </div>

  <p v-if="runawayDodges >= 3 && !success" class="prize-hint">
    Million dollars clicked {{ runawayDodges }} times. It was never real anyway.
  </p>
  <p v-else-if="!success && wrongCount > 0" class="prize-hint">
    Tried {{ wrongCount }} time{{ wrongCount === 1 ? '' : 's' }}. Keep going.
  </p>

  <Teleport to="body">
    <button
      v-if="isEscaped"
      ref="runawayEl"
      class="prize-card prize-card--runaway"
      :style="{ left: runawayPos.x + 'px', top: runawayPos.y + 'px' }"
      @click="pick(runawayCard)"
      @mouseenter="jumpAway"
      @pointerdown="handleRunawayPointerDown"
    >
      <img class="prize-card__img" :src="runawayCard.img" :alt="runawayCard.title" />
      <h3 class="prize-card__title">{{ runawayCard.title }}</h3>
      <p class="prize-card__desc">
        {{ runawayCard.wrong ? "Nope, it's empty." : runawayCard.desc }}
      </p>
    </button>
  </Teleport>

  <Teleport to="body">
    <div v-if="showOutOfStock" class="modal-backdrop" @click.self="showOutOfStock = false">
      <div class="modal">
        <h3 class="modal__title">Out of Stock</h3>
        <p class="modal__text">Someone already grabbed the last Swiss passport.</p>
        <button class="modal__close" @click="showOutOfStock = false">OK</button>
      </div>
    </div>
  </Teleport>

  <HonestConfetti ref="confettiRef" />
</template>

<style lang="scss" scoped>
.prize-row {
  display: flex;
  flex-wrap: nowrap;
  justify-content: center;
  gap: 1rem;
  padding: 0.5rem 0;
}

.prize-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  flex-shrink: 0;
  width: 210px;
  min-height: 180px;
  padding: 1.25rem 1rem;
  background: #fcf2eb;
  border-radius: 18px;
  box-shadow: 0 10px 18px rgba(45, 36, 56, 0.18);
  transition: transform 0.15s ease, box-shadow 0.15s ease;

  &:hover {
    transform: translateY(-4px);
    box-shadow: 0 14px 22px rgba(45, 36, 56, 0.22);
  }

  &:active {
    transform: translateY(0) scale(0.98);
  }

  &__img {
    width: 64px;
    height: 64px;
    object-fit: contain;
  }

  &__title {
    font-size: 1rem;
    font-weight: 700;
    color: #2d2438;
  }

  &__desc {
    font-size: 0.85rem;
    color: #6b6178;
    text-align: center;
  }

  &--wrong {
    animation: shake 0.4s ease;

    .prize-card__desc {
      color: #e8425c;
    }
  }

  &--runaway {
    position: fixed;
    width: 200px;
    z-index: 999;
    cursor: not-allowed;
    box-shadow: 0 14px 26px rgba(45, 36, 56, 0.3);
    transition: left 0.18s ease-out, top 0.18s ease-out, transform 0.15s ease;
  }
}

.prize-hint {
  text-align: center;
  font-size: 0.9rem;
  color: #5e503f;
}

.prize-success {
  text-align: center;
  font-size: 1.1rem;
  font-weight: 700;
  color: #091186;
  padding: 0.5rem 0;
}

.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(45, 36, 56, 0.45);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 1rem;
}

.modal {
  background: #fff;
  border-radius: 18px;
  padding: 2rem 1.75rem;
  max-width: 300px;
  width: 100%;
  text-align: center;
  box-shadow: 0 20px 40px rgba(45, 36, 56, 0.3);

  &__emoji {
    font-size: 2.5rem;
  }

  &__title {
    font-size: 1.2rem;
    font-weight: 800;
    color: #2d2438;
    margin: 0.5rem 0 0.25rem;
  }

  &__text {
    font-size: 0.9rem;
    color: #6b6178;
    margin-bottom: 1.25rem;
  }

  &__close {
    padding: 0.65rem 1.75rem;
    border-radius: 999px;
    background: linear-gradient(135deg, #ff8a9c, #ff5e78 55%, #e8425c);
    color: #fff;
    font-weight: 700;
    box-shadow: 0 8px 18px rgba(255, 94, 120, 0.35);
  }
}

@keyframes shake {
  0%,
  100% {
    transform: translateX(0);
  }
  25% {
    transform: translateX(-6px);
  }
  75% {
    transform: translateX(6px);
  }
}
</style>
