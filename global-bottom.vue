<script setup lang="ts">
import { useNav } from '@slidev/client'
import seedrandom from 'seedrandom'
import { computed, ref, onMounted, onUnmounted, watch } from 'vue'

const { currentSlideRoute } = useNav()

export type Distribution
  = | 'full'
    | 'top'
    | 'bottom'
    | 'left'
    | 'right'
    | 'top-left'
    | 'top-right'
    | 'bottom-left'
    | 'bottom-right'
    | 'center'

export type MeshType = 'square' | 'hexagon'

// Retrieve configuration from Slidev's frontmatter
const formatter = computed(() => (currentSlideRoute.value.meta?.slide as any)?.frontmatter || {})
const distribution = computed(() => (formatter.value.meshGlow || 'center') as Distribution)
const meshSize = computed(() => +(formatter.value.meshSize || 1))
const meshSpacing = computed(() => +(formatter.value.meshSpacing || 4))
const holeSize = computed(() => +(formatter.value.meshHoleSize || 0.7))
const lightIntensity = computed(() => +(formatter.value.meshLightIntensity || 0.4))
const lightRadius = computed(() => +(formatter.value.meshLightRadius || 400))
const lightHue = computed(() => +(formatter.value.meshLightHue || 0))
const meshType = computed(() => (formatter.value.meshType || 'square') as MeshType)
const seed = computed(() => (formatter.value.meshSeed === 'false' || formatter.value.meshSeed === false)
  ? Date.now().toString()
  : formatter.value.meshSeed || 'default',
)
const animationSpeed = computed(() => +(formatter.value.meshAnimationSpeed || 2.5))
const lightMovement = computed(() => +(formatter.value.meshLightMovement || 0.1))
const lightMovementType = computed(() => formatter.value.meshLightMovementType || 'smooth')

// Canvas related refs
const container = ref<HTMLDivElement>()
const canvas = ref<HTMLCanvasElement>()
const width = ref(1920)
const height = ref(1080)
const animationId = ref<number>()
const time = ref(0)

// Light source position
const currentLightX = ref(0)
const currentLightY = ref(0)
const targetLightX = ref(0)
const targetLightY = ref(0)
const baseLightX = ref(0)
const baseLightY = ref(0)

// Movement pattern for each slide
const slideMovementPattern = ref<{
  offsetX: number
  offsetY: number
  speedX: number
  speedY: number
  phaseX: number
  phaseY: number
}>({
  offsetX: 0,
  offsetY: 0,
  speedX: 1,
  speedY: 1,
  phaseX: 0,
  phaseY: 0
})

// Determine light source position range based on distribution
function distributionToPosition(distribution: Distribution): { x: [number, number], y: [number, number] } {
  const min = 0
  const max = 1
  let x: [number, number] = [min, max]
  let y: [number, number] = [min, max]

  switch (distribution) {
    case 'top':
      y = [min, 0.3]
      break
    case 'bottom':
      y = [0.7, max]
      break
    case 'left':
      x = [min, 0.3]
      break
    case 'right':
      x = [0.7, max]
      break
    case 'top-left':
      x = [min, 0.3]
      y = [min, 0.3]
      break
    case 'top-right':
      x = [0.7, max]
      y = [min, 0.3]
      break
    case 'bottom-left':
      x = [min, 0.3]
      y = [0.7, max]
      break
    case 'bottom-right':
      x = [0.7, max]
      y = [0.7, max]
      break
    case 'center':
      x = [0.3, 0.7]
      y = [0.3, 0.7]
      break
    case 'full':
      x = [min, max]
      y = [min, max]
      break
  }

  return { x, y }
}

// HSL to RGB conversion
function hslToRgb(h: number, s: number, l: number) {
  h = h / 360
  const a = s * Math.min(l, 1 - l)
  const f = (n: number, k = (n + h * 12) % 12) => 
    l - a * Math.max(Math.min(k - 3, 9 - k, 1), -1)
  
  return {
    r: Math.round(f(0) * 255),
    g: Math.round(f(8) * 255),
    b: Math.round(f(4) * 255)
  }
}

// Draw square mesh pattern
function drawSquareMesh(ctx: CanvasRenderingContext2D) {
  const baseColor = { r: 18, g: 18, b: 18 }
  const lightColor = hslToRgb(180 + lightHue.value, 0.3, 0.3)
  const holeRadius = (meshSpacing.value * holeSize.value) / 2
  
  // Fill background with black
  ctx.fillStyle = '#000000'
  ctx.fillRect(0, 0, width.value, height.value)
  
  // Draw mesh
  ctx.save()
  
  // Create clipping mask (excluding holes)
  ctx.beginPath()
  ctx.rect(0, 0, width.value, height.value)
  
  // Add holes (reverse path)
  for (let x = meshSpacing.value / 2; x < width.value + meshSpacing.value; x += meshSpacing.value) {
    for (let y = meshSpacing.value / 2; y < height.value + meshSpacing.value; y += meshSpacing.value) {
      ctx.moveTo(x + holeRadius, y)
      ctx.arc(x, y, holeRadius, 0, Math.PI * 2, true)
    }
  }
  ctx.clip('evenodd')
  
  // Draw the entire mesh with light effect
  for (let x = 0; x < width.value + meshSpacing.value; x += meshSpacing.value) {
    for (let y = 0; y < height.value + meshSpacing.value; y += meshSpacing.value) {
      const distance = Math.sqrt(
        Math.pow(x - currentLightX.value, 2) + 
        Math.pow(y - currentLightY.value, 2)
      )
      
      const lightEffect = Math.max(0, 1 - (distance / lightRadius.value))
      const intensity = lightEffect * lightIntensity.value
      
      const r = Math.round(baseColor.r + (lightColor.r - baseColor.r) * intensity)
      const g = Math.round(baseColor.g + (lightColor.g - baseColor.g) * intensity)
      const b = Math.round(baseColor.b + (lightColor.b - baseColor.b) * intensity)
      
      ctx.fillStyle = `rgb(${r}, ${g}, ${b})`
      ctx.fillRect(x - meshSpacing.value / 2, y - meshSpacing.value / 2, meshSpacing.value, meshSpacing.value)
    }
  }
  
  ctx.restore()
}

// Draw hexagon mesh pattern
function drawHexagonMesh(ctx: CanvasRenderingContext2D) {
  const baseColor = { r: 18, g: 18, b: 18 }
  const lightColor = hslToRgb(180 + lightHue.value, 0.3, 0.3)
  const hexRadius = meshSpacing.value / 2
  const holeRadius = hexRadius * holeSize.value
  const hexHeight = hexRadius * Math.sqrt(3)
  
  ctx.fillStyle = '#000000'
  ctx.fillRect(0, 0, width.value, height.value)
  
  ctx.save()
  
  // Draw hexagonal mesh
  for (let row = 0; row < height.value / hexHeight + 2; row++) {
    for (let col = 0; col < width.value / (hexRadius * 1.5) + 2; col++) {
      const x = col * hexRadius * 1.5
      const y = row * hexHeight + (col % 2) * hexHeight / 2
      
      const distance = Math.sqrt(
        Math.pow(x - currentLightX.value, 2) + 
        Math.pow(y - currentLightY.value, 2)
      )
      
      const lightEffect = Math.max(0, 1 - (distance / lightRadius.value))
      const intensity = lightEffect * lightIntensity.value
      
      const r = Math.round(baseColor.r + (lightColor.r - baseColor.r) * intensity)
      const g = Math.round(baseColor.g + (lightColor.g - baseColor.g) * intensity)
      const b = Math.round(baseColor.b + (lightColor.b - baseColor.b) * intensity)
      
      // Create path for hexagon minus the hole
      ctx.beginPath()
      for (let i = 0; i < 6; i++) {
        const angle = (Math.PI / 3) * i
        const px = x + hexRadius * Math.cos(angle)
        const py = y + hexRadius * Math.sin(angle)
        if (i === 0) ctx.moveTo(px, py)
        else ctx.lineTo(px, py)
      }
      ctx.closePath()
      
      // Inner hole
      ctx.moveTo(x + holeRadius, y)
      for (let i = 0; i < 6; i++) {
        const angle = (Math.PI / 3) * i
        const px = x + holeRadius * Math.cos(angle)
        const py = y + holeRadius * Math.sin(angle)
        ctx.lineTo(px, py)
      }
      ctx.closePath()
      
      ctx.fillStyle = `rgb(${r}, ${g}, ${b})`
      ctx.fill('evenodd')
    }
  }
  
  ctx.restore()
}

// Draw the mesh pattern based on meshType
function drawPattern() {
  if (!canvas.value) return
  
  const ctx = canvas.value.getContext('2d', { alpha: false })!
  
  if (meshType.value === 'hexagon') {
    drawHexagonMesh(ctx)
  } else {
    drawSquareMesh(ctx)
  }
  
  // Gradient overlay
  const gradient = ctx.createRadialGradient(
    currentLightX.value, currentLightY.value, 0,
    currentLightX.value, currentLightY.value, lightRadius.value
  )
  gradient.addColorStop(0, 'rgba(255, 255, 255, 0.015)')
  gradient.addColorStop(0.5, 'rgba(255, 255, 255, 0.008)')
  gradient.addColorStop(1, 'rgba(0, 0, 0, 0)')
  ctx.fillStyle = gradient
  ctx.fillRect(0, 0, width.value, height.value)
}

// Update light source position based on movement pattern
function updateLightPosition() {
  switch (lightMovementType.value) {
    case 'circular':
      targetLightX.value = baseLightX.value + 
        Math.cos(time.value * slideMovementPattern.value.speedX + slideMovementPattern.value.phaseX) * slideMovementPattern.value.offsetX * width.value * lightMovement.value
      targetLightY.value = baseLightY.value + 
        Math.sin(time.value * slideMovementPattern.value.speedY + slideMovementPattern.value.phaseY) * slideMovementPattern.value.offsetY * height.value * lightMovement.value
      break
      
    case 'lissajous':
      targetLightX.value = baseLightX.value + 
        Math.sin(time.value * slideMovementPattern.value.speedX + slideMovementPattern.value.phaseX) * slideMovementPattern.value.offsetX * width.value * lightMovement.value
      targetLightY.value = baseLightY.value + 
        Math.sin(time.value * slideMovementPattern.value.speedY + slideMovementPattern.value.phaseY) * slideMovementPattern.value.offsetY * height.value * lightMovement.value
      break
      
    case 'random':
      if (Math.random() < 0.02) {
        const rng = seedrandom(`${seed.value}-${time.value}`)
        targetLightX.value = baseLightX.value + (rng() - 0.5) * width.value * lightMovement.value
        targetLightY.value = baseLightY.value + (rng() - 0.5) * height.value * lightMovement.value
      }
      break
      
    case 'smooth':
    default:
      targetLightX.value = baseLightX.value + 
        Math.sin(time.value * slideMovementPattern.value.speedX + slideMovementPattern.value.phaseX) * slideMovementPattern.value.offsetX * width.value * lightMovement.value * 0.5
      targetLightY.value = baseLightY.value + 
        Math.cos(time.value * slideMovementPattern.value.speedY + slideMovementPattern.value.phaseY) * slideMovementPattern.value.offsetY * height.value * lightMovement.value * 0.5
      break
  }
}

// Animation loop
function animate() {
  time.value += 0.01
  updateLightPosition()
  
  const ease = 0.08
  currentLightX.value += (targetLightX.value - currentLightX.value) * ease
  currentLightY.value += (targetLightY.value - currentLightY.value) * ease
  
  drawPattern()
  animationId.value = requestAnimationFrame(animate)
}

// Set new light position and pattern for the slide
function setNewLightPosition() {
  const rng = seedrandom(`${seed.value}-${currentSlideRoute.value.no}`)
  const pos = distributionToPosition(distribution.value)
  
  baseLightX.value = (pos.x[0] + rng() * (pos.x[1] - pos.x[0])) * width.value
  baseLightY.value = (pos.y[0] + rng() * (pos.y[1] - pos.y[0])) * height.value
  
  slideMovementPattern.value = {
    offsetX: rng() * 0.4 - 0.2,
    offsetY: rng() * 0.4 - 0.2,
    speedX: 0.5 + rng() * 2,
    speedY: 0.5 + rng() * 2,
    phaseX: rng() * Math.PI * 2,
    phaseY: rng() * Math.PI * 2
  }
  
  if (currentLightX.value === 0 && currentLightY.value === 0) {
    currentLightX.value = baseLightX.value
    currentLightY.value = baseLightY.value
    targetLightX.value = baseLightX.value
    targetLightY.value = baseLightY.value
  }
}

// Update canvas size
function updateCanvasSize() {
  if (!container.value) return
  
  width.value = container.value.clientWidth || 1920
  height.value = container.value.clientHeight || 1080
  
  setNewLightPosition()
}

const handleResize = () => {
  updateCanvasSize()
}

// Reset time and set new light position when slide changes
watch(currentSlideRoute, () => {
  time.value = 0
  setNewLightPosition()
})

onMounted(() => {
  updateCanvasSize()
  animate()
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  if (animationId.value) {
    cancelAnimationFrame(animationId.value)
  }
})
</script>

<template>
  <div 
    ref="container"
    class="mesh-pattern-background"
    aria-hidden="true"
  >
    <canvas 
      ref="canvas" 
      :width="width" 
      :height="height"
      :style="{ 
        transition: `all ${animationSpeed}s cubic-bezier(0.4, 0, 0.2, 1)`
      }"
    />
  </div>
</template>

<style scoped>
.mesh-pattern-background {
  position: absolute;
  inset: 0;
  z-index: -10;
  overflow: hidden;
  background-color: #000;
  pointer-events: none;
}

canvas {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
