<script setup>
import { ref } from 'vue'
import ParticleCanvas from './components/ParticleCanvas.vue'

const stats = ref({ count: 0, fps: 0 })

function onStats(s) {
  stats.value = s
}
</script>

<template>
  <div class="scene">
    <!-- 背景氛围光晕 -->
    <div class="glow glow-a"></div>
    <div class="glow glow-b"></div>

    <ParticleCanvas @stats="onStats" />

    <!-- 扫描线 -->
    <div class="scanline"></div>

    <!-- 前景内容(不拦截鼠标,保证粒子可交互) -->
    <div class="overlay">
      <header class="hud-top">
        <div class="brand">
          <span class="dot"></span>
          PARTICLE&nbsp;NEXUS
        </div>
        <div class="hud-stats">
          <span class="stat">NODES <b>{{ stats.count }}</b></span>
          <span class="stat">FPS <b>{{ stats.fps }}</b></span>
          <span class="stat online">● ONLINE</span>
        </div>
      </header>

      <main class="hero">
        <p class="tagline">/ / QUANTUM INTERFACE v2.6</p>
        <h1 class="title">粒子<span class="accent">星云</span>引擎</h1>
        <p class="subtitle">
          移动鼠标牵引粒子流 · 点击任意位置触发能量爆发
        </p>
        <div class="actions">
          <button class="btn btn-primary">进入系统</button>
          <button class="btn btn-ghost">了解更多</button>
        </div>
      </main>

      <footer class="hud-bottom">
        <span>SYS.CORE // STABLE</span>
        <span>LAT 31.23° · LNG 121.47°</span>
        <span>© 2026 NEXUS LAB</span>
      </footer>
    </div>

    <!-- 四角装饰边框 -->
    <div class="corner tl"></div>
    <div class="corner tr"></div>
    <div class="corner bl"></div>
    <div class="corner br"></div>
  </div>
</template>

<style scoped>
.scene {
  position: relative;
  width: 100%;
  height: 100%;
  background: radial-gradient(ellipse at 50% 40%, #0b1226 0%, #05070f 70%);
  overflow: hidden;
}

/* ---------- 氛围光晕 ---------- */
.glow {
  position: absolute;
  border-radius: 50%;
  filter: blur(120px);
  opacity: 0.35;
  pointer-events: none;
  animation: drift 14s ease-in-out infinite alternate;
}
.glow-a {
  width: 45vw;
  height: 45vw;
  left: -10vw;
  top: -15vw;
  background: #0e4b8a;
}
.glow-b {
  width: 40vw;
  height: 40vw;
  right: -12vw;
  bottom: -15vw;
  background: #4b1d8f;
  animation-delay: -7s;
}
@keyframes drift {
  from { transform: translate(0, 0) scale(1); }
  to { transform: translate(4vw, 3vw) scale(1.15); }
}

/* ---------- 扫描线 ---------- */
.scanline {
  position: absolute;
  left: 0;
  right: 0;
  height: 2px;
  background: linear-gradient(90deg, transparent, rgba(0, 229, 255, 0.5), transparent);
  animation: scan 7s linear infinite;
  pointer-events: none;
}
@keyframes scan {
  0% { top: -2px; opacity: 0; }
  8% { opacity: 1; }
  92% { opacity: 1; }
  100% { top: 100%; opacity: 0; }
}

/* ---------- 前景布局 ---------- */
.overlay {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 28px 40px;
  pointer-events: none; /* 让鼠标事件穿透到画布 */
}
.overlay .btn {
  pointer-events: auto;
}

.hud-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 13px;
  letter-spacing: 3px;
  color: #7ea6c9;
}
.brand {
  display: flex;
  align-items: center;
  gap: 10px;
  font-weight: 600;
  color: #cfe9ff;
}
.dot {
  width: 9px;
  height: 9px;
  border-radius: 50%;
  background: #00e5ff;
  box-shadow: 0 0 12px #00e5ff;
  animation: pulse 2s ease-in-out infinite;
}
@keyframes pulse {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.5; transform: scale(0.8); }
}
.hud-stats {
  display: flex;
  gap: 24px;
  font-variant-numeric: tabular-nums;
}
.stat b {
  color: #00e5ff;
  font-weight: 600;
  margin-left: 4px;
}
.stat.online {
  color: #38f8b8;
}

/* ---------- 主标题区 ---------- */
.hero {
  text-align: center;
  transform: translateY(-4%);
}
.tagline {
  font-size: 13px;
  letter-spacing: 6px;
  color: #58c8e8;
  margin-bottom: 18px;
  animation: fadeUp 0.9s ease both;
}
.title {
  font-size: clamp(44px, 7.5vw, 96px);
  font-weight: 800;
  letter-spacing: 8px;
  line-height: 1.1;
  background: linear-gradient(120deg, #e6f7ff 20%, #6cd4ff 50%, #b28dff 80%);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 0 0 60px rgba(0, 180, 255, 0.25);
  animation: fadeUp 0.9s 0.15s ease both;
}
.accent {
  background: linear-gradient(120deg, #00e5ff, #7c5cff);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
}
.subtitle {
  margin-top: 22px;
  font-size: 15px;
  letter-spacing: 2px;
  color: #8fb3d4;
  animation: fadeUp 0.9s 0.3s ease both;
}
@keyframes fadeUp {
  from { opacity: 0; transform: translateY(24px); }
  to { opacity: 1; transform: translateY(0); }
}

/* ---------- 按钮 ---------- */
.actions {
  margin-top: 40px;
  display: flex;
  justify-content: center;
  gap: 20px;
  animation: fadeUp 0.9s 0.45s ease both;
}
.btn {
  padding: 13px 38px;
  font-size: 14px;
  letter-spacing: 3px;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.25s ease;
  font-family: inherit;
}
.btn-primary {
  border: none;
  color: #041018;
  background: linear-gradient(120deg, #00e5ff, #4f8fff);
  box-shadow: 0 0 24px rgba(0, 229, 255, 0.45);
}
.btn-primary:hover {
  box-shadow: 0 0 42px rgba(0, 229, 255, 0.8);
  transform: translateY(-2px);
}
.btn-ghost {
  background: transparent;
  color: #9fd8ff;
  border: 1px solid rgba(0, 229, 255, 0.45);
}
.btn-ghost:hover {
  background: rgba(0, 229, 255, 0.1);
  border-color: #00e5ff;
  transform: translateY(-2px);
}

/* ---------- 底部信息 ---------- */
.hud-bottom {
  display: flex;
  justify-content: space-between;
  font-size: 11px;
  letter-spacing: 2px;
  color: #4e6f8f;
}

/* ---------- 四角装饰 ---------- */
.corner {
  position: absolute;
  width: 26px;
  height: 26px;
  border: 2px solid rgba(0, 229, 255, 0.55);
  pointer-events: none;
}
.corner.tl { top: 14px; left: 14px; border-right: none; border-bottom: none; }
.corner.tr { top: 14px; right: 14px; border-left: none; border-bottom: none; }
.corner.bl { bottom: 14px; left: 14px; border-right: none; border-top: none; }
.corner.br { bottom: 14px; right: 14px; border-left: none; border-top: none; }

@media (max-width: 640px) {
  .overlay { padding: 20px; }
  .hud-stats { display: none; }
  .hud-bottom span:nth-child(2) { display: none; }
}
</style>
