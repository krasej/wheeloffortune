<script setup lang="ts">
import { ref, onMounted, computed, onUnmounted } from 'vue'
import ConfettiExplosion from "vue-confetti-explosion";

const props = defineProps({
    segments: {
        type: Array as () => string[],
        required: true
    }
})

const emit = defineEmits(['update:winner'])

const numSegments = computed(() => props.segments.length)
const anglePerSegment = computed(() => (2 * Math.PI) / numSegments.value)

const angle = ref(0)
const isSpinning = ref(false)
const winner = ref<string | undefined>('')

const wheelRadius = ref(190)
const imageSize = ref(400)
const imageCenter = ref(imageSize.value / 2)


function getPath(i: number) {
    const startAngle = i * anglePerSegment.value
    const endAngle = (i + 1) * anglePerSegment.value
    const x1 = imageCenter.value + wheelRadius.value * Math.cos(startAngle)
    const y1 = imageCenter.value + wheelRadius.value * Math.sin(startAngle)
    const x2 = imageCenter.value + wheelRadius.value * Math.cos(endAngle)
    const y2 = imageCenter.value + wheelRadius.value * Math.sin(endAngle)
    const largeArc = endAngle - startAngle > Math.PI ? 1 : 0
    return `M ${imageCenter.value} ${imageCenter.value} L ${x1} ${y1} A ${wheelRadius.value} ${wheelRadius.value} 0 ${largeArc} 1 ${x2} ${y2} Z`
}

function getTextX(i: number) {
    const textAngle = (i + 0.5) * anglePerSegment.value

    return imageCenter.value + Math.cos(textAngle) * (wheelRadius.value * 0.3)
}

function getTextY(i: number) {
    const textAngle = (i + 0.5) * anglePerSegment.value
    return imageCenter.value + Math.sin(textAngle) * (wheelRadius.value * 0.3)
}

function getRadialAngle(i: number) {
    return ((i + 0.5) * anglePerSegment.value * 180) / Math.PI
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
            winner.value = props.segments[randomSegment]
            emit('update:winner', winner.value)
            angle.value = ((angle.value % 360) + 360) % 360
        }
    }
    animate()
}

function resize() {
    if (window.innerWidth > 768) {
        wheelRadius.value = 300
        imageSize.value = 620
        imageCenter.value = imageSize.value / 2
    } else {

        imageSize.value = window.innerWidth - 20
        imageCenter.value = imageSize.value / 2
        wheelRadius.value = imageCenter.value - 10
    }
}

onMounted(() => {
    resize()
    window.addEventListener('resize', resize)
})


onUnmounted(() => {
    window.removeEventListener('resize', resize)
})


</script>

<template>
    <div class="spinner-module">
        <ConfettiExplosion v-if="winner" :active="!!winner" :duration="3000" :stageWidth="800" :stageHeight="imageSize"
            :colors="['var(--color-segment-first)', 'var(--color-segment-second)', 'var(--color-segment-third)', 'var(--color-segment-fourth)', 'var(--color-segment-third)', 'var(--color-segment-alt)']" />

        <svg :width="imageSize" :height="imageSize" :viewBox="`0 0 ${imageSize} ${imageSize}`" class="wheel-svg">
            <circle :cx="imageCenter" :cy="imageCenter" :r="wheelRadius" class="outer-circle" />
            <g v-for="(label, i) in segments" :key="i" class="segment"
                :style="{ transform: `rotate(${angle}deg)`, transformOrigin: `${imageCenter}px ${imageCenter}px` }">
                <path :d="getPath(i)" />
                <foreignObject :x="getTextX(i)" :y="getTextY(i) - 25" :width="wheelRadius * 0.55" height="50"
                    :transform="`rotate(${getRadialAngle(i)}, ${getTextX(i)}, ${getTextY(i)})`"
                    style="overflow: visible;">
                    <div xmlns="http://www.w3.org/1999/xhtml" class="segment-wrapper">
                        <span class="segment-text">{{ label }}</span>
                    </div>
                </foreignObject>
            </g>

            <circle :cx="imageCenter" :cy="imageCenter" r="20" class="center-circle" />
            <polygon :points="imageCenter + ',20 ' + (imageCenter - 15) + ',0 ' + (imageCenter + 15) + ',0'"
                class="pointer" />
        </svg>
        <button @click="spin" :disabled="isSpinning || numSegments === 0" class="spin-button">
            {{ isSpinning ? 'Spinning...' : 'Spin the Wheel' }}
        </button>
        <div v-if="winner" class="winner">
            Winner: {{ winner }}
        </div>
        <div v-else class="winner">
            &nbsp;
        </div>
    </div>
</template>

<style lang="css" scoped>
.spinner-module {
    display: flex;
    flex-direction: column;
    align-items: center;
    overflow-x: hidden;
}

.wheel-svg {
    max-width: 100%;
    height: auto;
}
</style>