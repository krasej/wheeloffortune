<script setup lang="ts">
import { ref, onMounted, computed, watch } from 'vue'
import ProModule from './components/ProModule.vue'

const angle = ref(0)
const isSpinning = ref(false)
const winner = ref<string | undefined>('')
const newPrize = ref('')

let title = ref('Who is the best director?')
let editTitle = ref(false)

const watchTitle = watch(title, (newValue) => {

  document.title = newValue
}
  , { immediate: true })

const segments = ref([
  'Quentin Tarantino',
  'Christopher Nolan',
])

const numSegments = computed(() => segments.value.length)
const anglePerSegment = computed(() => (2 * Math.PI) / numSegments.value)

function getPath(i: number) {
  const startAngle = i * anglePerSegment.value
  const endAngle = (i + 1) * anglePerSegment.value
  const x1 = 200 + 150 * Math.cos(startAngle)
  const y1 = 200 + 150 * Math.sin(startAngle)
  const x2 = 200 + 150 * Math.cos(endAngle)
  const y2 = 200 + 150 * Math.sin(endAngle)
  const largeArc = endAngle - startAngle > Math.PI ? 1 : 0
  return `M 200 200 L ${x1} ${y1} A 150 150 0 ${largeArc} 1 ${x2} ${y2} Z`
}

function getTextX(i: number) {
  const textAngle = (i + 0.5) * anglePerSegment.value
  return 200 + Math.cos(textAngle) * 70
}

function getTextY(i: number) {
  const textAngle = (i + 0.5) * anglePerSegment.value
  return 200 + Math.sin(textAngle) * 70
}

function getRadialAngle(i: number) {
  return ((i + 0.5) * anglePerSegment.value) * 180 / Math.PI
}

function spin() {
  if (isSpinning.value || numSegments.value === 0) return
  isSpinning.value = true
  winner.value = ''

  const randomSegment = Math.floor(Math.random() * numSegments.value)
  const centerAngleRad = (randomSegment + 0.5) * anglePerSegment.value
  const pointerAngleRad = -Math.PI / 2
  const targetAngleRad = pointerAngleRad - centerAngleRad
  const targetAngleDeg = targetAngleRad * 180 / Math.PI
  const spins = Math.floor(Math.random() * 20) + 5
  const targetAngle = targetAngleDeg + spins * 360

  const duration = 5000 // 5 seconds
  const startAngle = angle.value
  const startTime = Date.now()

  function animate() {
    const elapsed = Date.now() - startTime
    const progress = Math.min(elapsed / duration, 1)
    const easeProgress = 1 - Math.pow(1 - progress, 3) // ease out
    angle.value = startAngle + (targetAngle - startAngle) * easeProgress

    if (progress < 1) {
      requestAnimationFrame(animate)
    } else {
      isSpinning.value = false
      winner.value = segments.value[randomSegment]
      angle.value = ((angle.value % 360) + 360) % 360
    }
  }
  animate()
}

function addPrize() {
  if (!newPrize.value.trim()) return
  segments.value.push(newPrize.value.trim())
  newPrize.value = ''
}

function removePrize(index: number) {
  Array.prototype.splice.call(segments.value, index, 1)
}

function normalizeDegrees(deg: number) {
  return ((deg % 360) + 360) % 360
}

function getTopSegment(angleDeg: number) {
  if (numSegments.value === 0) return -1
  const segSize = 360 / numSegments.value
  const pointerAngle = normalizeDegrees(270 - angleDeg)
  return Math.floor(pointerAngle / segSize) % numSegments.value
}


onMounted(() => {
})
</script>

<template>
  <div class="wheel-container">
    <h1 class="title-heading" v-on:click="editTitle = !editTitle">{{ title }}</h1>

    <div v-if="editTitle" class="edit-title">
      <input class="edit-input" v-on:keydown.enter="editTitle = false" v-model="title" placeholder="Enter new title" />
    </div>

    <svg width="400" height="400" class="wheel-svg">
      <circle cx="200" cy="200" r="150" class="outer-circle" />
      <g v-for="(label, i) in segments" :key="i" class="segment"
        :style="{ transform: `rotate(${angle}deg)`, transformOrigin: '200px 200px' }">
        <path :d="getPath(i)" />
        <text :x="getTextX(i) + 10" :y="getTextY(i)"
          :transform="`rotate(${getRadialAngle(i)}, ${getTextX(i)}, ${getTextY(i)})`" class="segment-text">{{ label
          }}</text>
      </g>

      <circle cx="200" cy="200" r="20" class="center-circle" />
      <polygon points="200,60 185,35 215,35" class="pointer" />
    </svg>
    <button @click="spin" :disabled="isSpinning || numSegments === 0" class="spin-button">
      {{ isSpinning ? 'Spinning...' : 'Spin the Wheel' }}
    </button>
    <div v-if="winner" class="winner">
      Winner: {{ winner }}
    </div>
    <div v-else class="winner">
      <p>&nbsp;</p>
    </div>
  </div>

  <div class="manage-options">
    <h2>Manage Options</h2>
    <div class="add-prize">
      <input @keydown.enter="addPrize" v-model="newPrize" placeholder="Enter new option" class="prize-input" />
      <button @click="addPrize" class="add-button">Add Option</button>
    </div>
    <h3>Current Options:</h3>
    <ul v-if="segments.length > 0" class="prize-list">
      <li v-for="(prize, index) in segments" :key="index">
        {{ prize }}
        <button @click="removePrize(index)" class="remove-button">x</button>
      </li>
    </ul>

    <ProModule @options="(options) => { segments = options }" />

  </div>
</template>
