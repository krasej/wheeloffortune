<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'

const angle = ref(0)
const isSpinning = ref(false)
const winner = ref<string | undefined>('')
const newPrize = ref('')

let title = ref('Who is the best director?')
let editTitle = ref(false)


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
      <g v-for="(label, i) in segments" :key="i" class="segment"
        :style="{ transform: `rotate(${angle}deg)`, transformOrigin: '200px 200px' }">
        <path :d="getPath(i)" />
        <text :x="getTextX(i)+10" :y="getTextY(i)"
          :transform="`rotate(${getRadialAngle(i)}, ${getTextX(i)}, ${getTextY(i)})`" class="segment-text">{{ label
          }}</text>
      </g>
      <circle cx="200" cy="200" r="20" class="center-circle" />
      <polygon points="200,50 185,30 215,30" class="pointer" />
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
    <ul v-if="segments.length > 0" class="prize-list" >
      <li v-for="(prize, index) in segments" :key="index">
        {{ prize }}
        <button @click="removePrize(index)" class="remove-button">x</button>
      </li>
    </ul>
  </div>
</template>

<style>

:root {
  --color-background: #222;
  --color-text: #ddd;
  --color-card-border: #333;
  --color-card-shadow: rgba(0, 0, 0, 0.2);
  --color-segment-alt: #976bff;
  --color-segment-first: #F7DC6F;
  --color-segment-second: #FF6B6B;
  --color-segment-third: #6ba6ff;
  --color-segment-fourth: #4CAF50;
  --color-text-dark: #333;
  --color-text-muted: #555;
  --color-button-primary: #4CAF50;
  --color-button-primary-hover: #45a049;
  --color-button-disabled: #cccccc;
  --color-button-accent: #2196F3;
  --color-button-accent-hover: #1976D2;
  --color-border-muted: #ccc;
  --color-surface: #111;
  --color-white: #ffffff;
}

html, body {
  margin: 0;
  padding: 0;
  background-color: var(--color-background);
  color: var(--color-text);
  font-family: 'Arial', sans-serif;
}
.wheel-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
}

.wheel-svg {
  border: 2px solid var(--color-card-border);
  border-radius: 50%;
  box-shadow: 0 4px 8px var(--color-card-shadow);
  margin-bottom: 20px;
}

g.segment:nth-of-type(1n) {
  fill: var(--color-segment-first);
}

g.segment:last-of-type {
  fill: var(--color-segment-alt);
}

g.segment:nth-of-type(2n) {
  fill: var(--color-segment-second);
}
g.segment:nth-of-type(3n) {
  fill: var(--color-segment-third);
}

g.segment:nth-of-type(4n) {
  fill: var(--color-segment-fourth);
}

.segment-text {
  fill: var(--color-text-dark);
  font-size: 14px;
  text-anchor: middle;
  dominant-baseline: middle;
}

.center-circle {
  fill: var(--color-card-border);
  stroke: var(--color-white);
  stroke-width: 2;
}

.pointer {
  fill: var(--color-segment-secondary);
  stroke: var(--color-card-border);
  stroke-width: 1;
}

.title-heading {
  position: relative;
  display: inline-block;
}

.title-heading::after {
  content: ' _';
  animation: blink-cursor 1.5s steps(1, end) infinite;
}

@keyframes blink-cursor {
  0%, 50% {
    opacity: 1;
  }
  50.01%, 100% {
    opacity: 0;
  }
}

.spin-button {
  padding: 10px 20px;
  font-size: 16px;
  background-color: var(--color-button-primary);
  color: var(--color-white);
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
  margin-bottom: 20px;
}

.spin-button:hover:not(:disabled) {
  background-color: var(--color-button-primary-hover);
}

.spin-button:disabled {
  background-color: var(--color-button-disabled);
  cursor: not-allowed;
}

.remove-button {
  margin-left: 10px;
  padding: 2px 6px;
  font-size: 14px;
  font-weight: bold;
  background-color: var(--color-card-border);
  color: var(--color-text);
  border: none;
  border-radius: 30px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.remove-button:hover {
  background-color: var(--color-text-muted);
}

.prize-list{
  padding-left: 0;
}

.winner {
  font-size: 24px;
  font-weight: bold;
  color: var(--color-text);
  margin-top: 10px;
  margin-bottom: 10px;
}

.add-prize {
  display: flex;
  gap: 10px;
  align-items: center;
  justify-items: stretch;
  flex-wrap: wrap;
}

.prize-input, .edit-input {
  padding: 8px;
  font-size: 16px;
  border: 1px solid var(--color-border-muted);
  border-radius: 4px;
  width: 250px;
}

.edit-input{
  margin-top: 10px;
  margin-bottom: 20px;
}

.add-button {
  padding: 8px 16px;
  font-size: 16px;
  background-color: var(--color-button-accent);
  border: 1px solid var(--color-button-accent);
  color: var(--color-white);
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
  width: 100%;
  flex: 1;
}

.add-button:hover {
  background-color: var(--color-button-accent-hover);
  border-color: var(--color-button-accent-hover);
}

.manage-options {
  padding-top: 20px;
  padding-bottom: 20px;
}

@media (min-width : 1024px) {
  .manage-options {
      padding: 20px;
  }

  .add-button {
    flex: 1;
  }
}

.prize-list ul {
  list-style: none;
  padding: 0;
}

.prize-list li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 12px;
}

.prize-list li:nth-child(odd) {
  background-color: var(--color-surface);
}

h2{
  margin-bottom: 30px;
  margin-top: 40px;
}

h3{
  margin-top: 30px;
  margin-bottom: 20px;
}
</style>
