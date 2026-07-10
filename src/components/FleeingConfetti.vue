<script setup>
import { ref, reactive, computed, onMounted, onUnmounted } from 'vue'
import { useRouter } from 'vue-router'
import loveImg from '../love.png'
import passportImg from '../passport.png'
import salaryImg from '../salary.png'

const DODGE_RADIUS = 120
const IDLE_MS = 3000
const CARD_WIDTH = 150
const CARD_HEIGHT = 180

const PRESENTS = [
  {
    img: passportImg,
    title: 'Swiss Passport',
    desc: 'Perfect german is included.',
    outOfStock: true,
  },
  { img: salaryImg, title: 'Million dollars', desc: 'To buy some treats.', runaway: true },
  { img: loveImg, title: 'Endless Sis Love', desc: 'Unlimited hugs, zero refunds.' },
]

const winnableIndices = PRESENTS.reduce((acc, present, i) => {
  if (!present.runaway && !present.outOfStock) acc.push(i)
  return acc
}, [])
const winnerIndex = winnableIndices[Math.floor(Math.random() * winnableIndices.length)]

const cards = reactive(
  PRESENTS.map((present, i) => ({
    ...present,
    id: i,
    isWinner: i === winnerIndex,
    wrong: false,
  })),
)

const staticCards = computed(() => cards.filter((card) => !card.runaway))
const runawayCard = computed(() => cards.find((card) => card.runaway))

const wrongCount = ref(0)
const success = ref(false)
const showOutOfStock = ref(false)
const showAccepted = ref(false)
const isModalOpen = computed(() => showOutOfStock.value || showAccepted.value)

const PAGE_CONFETTI_COUNT = 80
const CONFETTI_COLORS = ['#ff5e78', '#ffd166', '#37c58f', '#5b8def', '#a18cd1', '#ff9a9e']
const pageConfettiPieces = ref([])

function launchPageConfetti() {
  pageConfettiPieces.value = Array.from({ length: PAGE_CONFETTI_COUNT }, (_, i) => ({
    id: i,
    style: {
      left: Math.random() * 100 + '%',
      background: CONFETTI_COLORS[Math.floor(Math.random() * CONFETTI_COLORS.length)],
      width: 6 + Math.random() * 6 + 'px',
      height: 4 + Math.random() * 8 + 'px',
      animationDuration: 2.5 + Math.random() * 2 + 's',
      animationDelay: Math.random() * 1.5 + 's',
    },
  }))
}

const router = useRouter()

function closeAccepted() {
  showAccepted.value = false
  pageConfettiPieces.value = []
}

function proceed() {
  closeAccepted()
  router.push('/thank-you')
}

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

function flee() {
  if (isModalOpen.value) return

  isEscaped.value = true
  const { x, y } = randomViewportPos(CARD_WIDTH, CARD_HEIGHT)
  runawayPos.x = x
  runawayPos.y = y
  resetIdleTimer()
}

function jumpAway() {
  if (isModalOpen.value) return

  runawayDodges.value += 1
  flee()
}

function handleGlobalMouseMove(event) {
  if (!runawayEl.value) return

  const rect = runawayEl.value.getBoundingClientRect()
  const centerX = rect.left + rect.width / 2
  const centerY = rect.top + rect.height / 2
  const distance = Math.hypot(event.clientX - centerX, event.clientY - centerY)

  if (distance < DODGE_RADIUS) {
    flee()
  }
}

function handleRunawayPointerDown(event) {
  event.preventDefault()
  jumpAway()
}

function pick(card) {
  if (isModalOpen.value) return

  if (success.value) {
    if (card.isWinner) {
      showAccepted.value = true
      launchPageConfetti()
    }
    return
  }

  if (card.outOfStock) {
    showOutOfStock.value = true
    return
  }

  if (card.isWinner) {
    success.value = true
    showAccepted.value = true
    launchPageConfetti()
    return
  }

  wrongCount.value += 1
  card.wrong = true
  window.setTimeout(() => {
    card.wrong = false
  }, 5000)
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
  <div class="present-row" :class="{ 'present-row--locked': isModalOpen }">
    <button
      v-for="card in staticCards"
      :key="card.id"
      class="present-card"
      :class="{ 'present-card--wrong': card.wrong }"
      @click="pick(card)"
    >
      <img class="present-card__img" :src="card.img" :alt="card.title" />
      <h3 class="present-card__title">{{ card.title }}</h3>
      <p class="present-card__desc">{{ card.wrong ? "Nope, it's empty." : card.desc }}</p>
    </button>

    <button
      v-if="!isEscaped"
      ref="runawayEl"
      class="present-card"
      :class="{ 'present-card--wrong': runawayCard.wrong }"
      @click="pick(runawayCard)"
      @mouseenter="jumpAway"
      @pointerdown="handleRunawayPointerDown"
    >
      <img class="present-card__img" :src="runawayCard.img" :alt="runawayCard.title" />
      <h3 class="present-card__title">{{ runawayCard.title }}</h3>
      <p class="present-card__desc">
        {{ runawayCard.wrong ? "Nope, it's empty." : runawayCard.desc }}
      </p>
    </button>
  </div>

  <p v-if="isEscaped && runawayDodges > 0 && !success" class="present-hint">
    Million dollars tried {{ runawayDodges }} time{{ runawayDodges === 1 ? '' : 's' }}. It was never real anyway.
  </p>
  <p v-else-if="!success && wrongCount > 0" class="present-hint">
    Tried {{ wrongCount }} time{{ wrongCount === 1 ? '' : 's' }}. Keep going.
  </p>

  <Teleport to="body">
    <button
      v-if="isEscaped"
      ref="runawayEl"
      class="present-card present-card--runaway"
      :class="{ 'present-card--locked': isModalOpen }"
      :style="{ left: runawayPos.x + 'px', top: runawayPos.y + 'px' }"
      @click="pick(runawayCard)"
      @mouseenter="jumpAway"
      @pointerdown="handleRunawayPointerDown"
    >
      <img class="present-card__img" :src="runawayCard.img" :alt="runawayCard.title" />
      <h3 class="present-card__title">{{ runawayCard.title }}</h3>
      <p class="present-card__desc">
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

  <Teleport to="body">
    <div v-if="pageConfettiPieces.length" class="page-confetti">
      <span
        v-for="piece in pageConfettiPieces"
        :key="piece.id"
        class="page-confetti__piece"
        :style="piece.style"
      />
    </div>
  </Teleport>

  <Teleport to="body">
    <div v-if="showAccepted" class="modal-backdrop" @click.self="closeAccepted">
      <div class="modal">
        <h3 class="modal__title">Accepted! <br> You have just signed up for the endless Sis love!</h3>
        <button class="modal__proceed" @click="proceed">
          Proceed
          <span class="modal__proceed-arrow">&rarr;</span>
        </button>
        <p class="modal__text"><em>*No returns, no exchanges possible</em></p>
      </div>
    </div>
  </Teleport>

</template>

<style lang="scss" scoped>
.present-row {
  display: flex;
  flex-wrap: nowrap;
  justify-content: center;
  gap: 1rem;
  padding: 0.5rem 0;

  &--locked {
    pointer-events: none;
    opacity: 0.6;
  }
}

.present-card {
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

    .present-card__desc {
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

  &--locked {
    pointer-events: none;
    opacity: 0.6;
  }
}

.present-hint {
  text-align: center;
  font-size: 0.9rem;
  color: #5e503f;
}

.present-success {
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
  }

  &__text {
    font-size: 0.9rem;
    color: #6b6178;
    margin: 1.25rem 0;
  }

  &__close {
    padding: 0.65rem 1.75rem;
    border-radius: 999px;
    background: linear-gradient(135deg, #4bbed4, #4c70b8 55%, #091186);
    color: #fff;
    font-weight: 700;
    box-shadow: 0 8px 18px rgba(9, 17, 134, 0.35);
  }

  &__proceed {
    display: inline-flex;
    align-items: center;
    gap: 0.4rem;
    padding: 0.65rem 1.75rem;
    margin-top: 1rem;
    border-radius: 999px;
    background: linear-gradient(135deg, #6fe6a9, #37c58f 55%, #1f9d6b);
    color: #fff;
    font-weight: 700;
    box-shadow: 0 8px 18px rgba(55, 197, 143, 0.35);
  }

  &__proceed-arrow {
    font-weight: 900;
    transition: transform 0.15s ease;
  }

  &__proceed:hover &__proceed-arrow {
    transform: translateX(3px);
  }
}

.page-confetti {
  position: fixed;
  inset: 0;
  z-index: 2000;
  overflow: hidden;
  pointer-events: none;

  &__piece {
    position: absolute;
    top: -10%;
    border-radius: 2px;
    opacity: 0.95;
    animation-name: confetti-fall;
    animation-timing-function: linear;
    animation-iteration-count: infinite;
  }
}

@keyframes confetti-fall {
  0% {
    transform: translateY(0) rotate(0deg);
    opacity: 1;
  }
  100% {
    transform: translateY(115vh) rotate(360deg);
    opacity: 0.85;
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
