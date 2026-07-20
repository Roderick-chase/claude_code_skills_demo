<script setup>
import { onMounted, onBeforeUnmount, ref } from 'vue'

const props = defineProps({
  // 基础粒子数量(会按屏幕面积自动缩放)
  density: { type: Number, default: 110 },
  // 粒子之间连线的最大距离
  linkDistance: { type: Number, default: 130 },
  // 鼠标的作用半径
  mouseRadius: { type: Number, default: 180 },
})

const emit = defineEmits(['stats'])

const canvasRef = ref(null)

const PALETTE = [
  { r: 0, g: 229, b: 255 },   // 青
  { r: 64, g: 156, b: 255 },  // 蓝
  { r: 173, g: 109, b: 255 }, // 紫
  { r: 0, g: 255, b: 200 },   // 青绿
]

let ctx = null
let rafId = 0
let particles = []
let bursts = [] // 点击产生的短命爆发粒子
let width = 0
let height = 0
let dpr = 1
const mouse = { x: -9999, y: -9999, active: false }

// 帧率统计
let frameCount = 0
let lastStatTime = 0

function rand(min, max) {
  return min + Math.random() * (max - min)
}

function createParticle(x, y, isBurst = false) {
  const color = PALETTE[Math.floor(Math.random() * PALETTE.length)]
  const speed = isBurst ? rand(1.5, 4.5) : rand(0.1, 0.5)
  const angle = rand(0, Math.PI * 2)
  return {
    x: x ?? rand(0, width),
    y: y ?? rand(0, height),
    vx: Math.cos(angle) * speed,
    vy: Math.sin(angle) * speed,
    r: isBurst ? rand(1, 2.5) : rand(1.2, 3),
    color,
    // 呼吸闪烁的相位
    phase: rand(0, Math.PI * 2),
    life: isBurst ? 1 : Infinity,
  }
}

function targetCount() {
  // 按面积缩放粒子数,避免大屏过稀、小屏过挤
  const scale = (width * height) / (1280 * 720)
  return Math.round(Math.min(props.density * scale, 260))
}

function resize() {
  const canvas = canvasRef.value
  if (!canvas) return
  dpr = Math.min(window.devicePixelRatio || 1, 2)
  width = canvas.clientWidth
  height = canvas.clientHeight
  canvas.width = width * dpr
  canvas.height = height * dpr
  ctx.setTransform(dpr, 0, 0, dpr, 0, 0)

  const n = targetCount()
  if (particles.length < n) {
    while (particles.length < n) particles.push(createParticle())
  } else {
    particles.length = n
  }
}

function step(now) {
  ctx.clearRect(0, 0, width, height)

  const linkDist = props.linkDistance
  const mouseR = props.mouseRadius

  // 更新普通粒子
  for (const p of particles) {
    p.x += p.vx
    p.y += p.vy

    // 鼠标引力:轻微吸向光标,靠太近则推开,形成环绕感
    if (mouse.active) {
      const dx = mouse.x - p.x
      const dy = mouse.y - p.y
      const dist = Math.hypot(dx, dy)
      if (dist < mouseR && dist > 0.01) {
        const force = (1 - dist / mouseR) * 0.06
        const dir = dist < 40 ? -1.6 : 1
        p.vx += (dx / dist) * force * dir
        p.vy += (dy / dist) * force * dir
      }
    }

    // 阻尼,防止速度失控
    p.vx *= 0.99
    p.vy *= 0.99

    // 边界环绕
    if (p.x < -10) p.x = width + 10
    else if (p.x > width + 10) p.x = -10
    if (p.y < -10) p.y = height + 10
    else if (p.y > height + 10) p.y = -10
  }

  // 更新爆发粒子(有寿命,逐渐消散)
  for (const b of bursts) {
    b.x += b.vx
    b.y += b.vy
    b.vx *= 0.96
    b.vy *= 0.96
    b.life -= 0.016
  }
  bursts = bursts.filter((b) => b.life > 0)

  // 粒子间连线
  ctx.lineWidth = 1
  for (let i = 0; i < particles.length; i++) {
    const a = particles[i]
    for (let j = i + 1; j < particles.length; j++) {
      const b = particles[j]
      const dx = a.x - b.x
      const dy = a.y - b.y
      if (Math.abs(dx) > linkDist || Math.abs(dy) > linkDist) continue
      const dist = Math.hypot(dx, dy)
      if (dist < linkDist) {
        const alpha = (1 - dist / linkDist) * 0.35
        ctx.strokeStyle = `rgba(${a.color.r}, ${a.color.g}, ${a.color.b}, ${alpha})`
        ctx.beginPath()
        ctx.moveTo(a.x, a.y)
        ctx.lineTo(b.x, b.y)
        ctx.stroke()
      }
    }
  }

  // 鼠标与附近粒子连线,亮度更高
  if (mouse.active) {
    for (const p of particles) {
      const dist = Math.hypot(p.x - mouse.x, p.y - mouse.y)
      if (dist < mouseR) {
        const alpha = (1 - dist / mouseR) * 0.6
        ctx.strokeStyle = `rgba(0, 229, 255, ${alpha})`
        ctx.beginPath()
        ctx.moveTo(p.x, p.y)
        ctx.lineTo(mouse.x, mouse.y)
        ctx.stroke()
      }
    }
  }

  // 绘制粒子(带呼吸发光)
  const t = now * 0.002
  for (const p of particles) {
    const twinkle = 0.7 + 0.3 * Math.sin(t + p.phase)
    const { r: cr, g: cg, b: cb } = p.color
    ctx.shadowBlur = 8
    ctx.shadowColor = `rgba(${cr}, ${cg}, ${cb}, 0.9)`
    ctx.fillStyle = `rgba(${cr}, ${cg}, ${cb}, ${twinkle})`
    ctx.beginPath()
    ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2)
    ctx.fill()
  }

  for (const b of bursts) {
    const { r: cr, g: cg, b: cb } = b.color
    ctx.shadowBlur = 12
    ctx.shadowColor = `rgba(${cr}, ${cg}, ${cb}, ${b.life})`
    ctx.fillStyle = `rgba(${cr}, ${cg}, ${cb}, ${b.life})`
    ctx.beginPath()
    ctx.arc(b.x, b.y, b.r, 0, Math.PI * 2)
    ctx.fill()
  }
  ctx.shadowBlur = 0

  // 每 500ms 上报一次统计(粒子数 / FPS),供 HUD 显示
  frameCount++
  if (now - lastStatTime >= 500) {
    const fps = Math.round((frameCount * 1000) / (now - lastStatTime))
    emit('stats', { count: particles.length + bursts.length, fps })
    frameCount = 0
    lastStatTime = now
  }

  rafId = requestAnimationFrame(step)
}

function onPointerMove(e) {
  const rect = canvasRef.value.getBoundingClientRect()
  mouse.x = e.clientX - rect.left
  mouse.y = e.clientY - rect.top
  mouse.active = true
}

function onPointerLeave() {
  mouse.active = false
  mouse.x = -9999
  mouse.y = -9999
}

function onClick(e) {
  const rect = canvasRef.value.getBoundingClientRect()
  const x = e.clientX - rect.left
  const y = e.clientY - rect.top
  for (let i = 0; i < 24; i++) {
    bursts.push(createParticle(x, y, true))
  }
}

onMounted(() => {
  const canvas = canvasRef.value
  ctx = canvas.getContext('2d')
  resize()
  window.addEventListener('resize', resize)
  canvas.addEventListener('pointermove', onPointerMove)
  canvas.addEventListener('pointerleave', onPointerLeave)
  canvas.addEventListener('click', onClick)
  rafId = requestAnimationFrame((t) => {
    lastStatTime = t
    step(t)
  })
})

onBeforeUnmount(() => {
  cancelAnimationFrame(rafId)
  window.removeEventListener('resize', resize)
})
</script>

<template>
  <canvas ref="canvasRef" class="particle-canvas"></canvas>
</template>

<style scoped>
.particle-canvas {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  display: block;
}
</style>
