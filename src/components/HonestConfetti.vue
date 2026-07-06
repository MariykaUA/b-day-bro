<script setup>
import { ref } from 'vue'

// Confetti that does NOT work properly on purpose:
// - "up" mode: it falls upward, defying gravity
// - "pile" mode: it just dumps in a sad heap in the corner
const PIECE_COUNT = 28
const COLORS = ['#ff5e78', '#ffd166', '#37c58f', '#5b8def', '#a18cd1', '#ff9a9e']

const runId = ref(0)
const mode = ref('up')
const active = ref(false)
const caption = ref('')

const upCaptions = [
  "Confetti physics: denied.",
  "Gravity called in sick today.",
  "This confetti has never seen a floor.",
]
const pileCaptions = [
  "The confetti union went on strike. This is all they could afford.",
  "Some assembly required. None occurred.",
  "It's the thought that counts.",
]

function pick(list) {
  return list[Math.floor(Math.random() * list.length)]
}

function launch() {
  mode.value = Math.random() < 0.5 ? 'up' : 'pile'
  caption.value = mode.value === 'up' ? pick(upCaptions) : pick(pileCaptions)
  runId.value += 1
  active.value = true

  window.clearTimeout(launch._t)
  launch._t = window.setTimeout(() => {
    active.value = false
  }, 2200)
}

defineExpose({ launch })

function pieceStyle(i) {
  const left = mode.value === 'up' ? Math.random() * 100 : 78 + Math.random() * 18
  const color = pick(COLORS)
  const size = 6 + Math.random() * 6

  if (mode.value === 'up') {
    return {
      left: left + '%',
      bottom: '-5%',
      width: size + 'px',
      height: size * 0.4 + 'px',
      background: color,
      animationDuration: 1 + Math.random() * 1.2 + 's',
      animationDelay: Math.random() * 0.5 + 's',
    }
  }

  // "pile" mode: pieces stack low in the corner, barely moving
  return {
    left: left + '%',
    bottom: Math.random() * 10 + '%',
    width: size + 'px',
    height: size * 0.4 + 'px',
    background: color,
    animationDuration: '0.4s',
    animationDelay: Math.random() * 0.3 + 's',
    transform: `rotate(${Math.random() * 360}deg)`,
  }
}
</script>

<template>
  <div class="confetti-zone">
    <div v-if="active" :key="runId" class="confetti-zone__pieces" :class="mode">
      <span
        v-for="i in PIECE_COUNT"
        :key="i"
        class="confetti-piece"
        :style="pieceStyle(i)"
      />
    </div>
    <p v-if="active" class="confetti-zone__caption">{{ caption }}</p>
  </div>
</template>

<style lang="scss" scoped>
.confetti-zone {
  position: relative;
  min-height: 2rem;

  &__pieces {
    position: absolute;
    inset: 0;
    pointer-events: none;
  }

  &__caption {
    position: relative;
    text-align: center;
    font-size: 0.85rem;
    color: #6b6178;
    font-style: italic;
    padding-top: 0.5rem;
  }
}

.confetti-piece {
  position: absolute;
  border-radius: 2px;
  opacity: 0.95;
}

.up .confetti-piece {
  animation-name: float-up;
  animation-timing-function: ease-in;
  animation-fill-mode: forwards;
}

.pile .confetti-piece {
  animation-name: plop;
  animation-timing-function: ease-out;
  animation-fill-mode: forwards;
}

@keyframes float-up {
  0% {
    transform: translateY(0) rotate(0deg);
    opacity: 1;
  }
  100% {
    transform: translateY(-160px) rotate(200deg);
    opacity: 0;
  }
}

@keyframes plop {
  0% {
    transform: translateY(-14px) scale(0.6);
    opacity: 0;
  }
  100% {
    transform: translateY(0) scale(1);
    opacity: 1;
  }
}
</style>
