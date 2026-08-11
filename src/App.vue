<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from "vue"

const MESSAGE =
  "Connection established. Мы наблюдаем. Если ты нашел это, значит умеешь замечать детали. Продолжай искать."

// Замени на ссылку своего Telegram-канала.
const TELEGRAM_URL = "https://t.me/utmnctf"

const displayedText = ref("")
const typingFinished = ref(false)
const buttonVisible = ref(false)
const glitching = ref(false)
const hardGlitch = ref(false)
const screenFlash = ref(false)

let typeTimer: number | undefined
let glitchTimer: number | undefined
let buttonTimer: number | undefined
let flashTimer: number | undefined

const glitchText = computed(() => displayedText.value)

function sleep(ms: number) {
  return new Promise<void>((resolve) => {
    window.setTimeout(resolve, ms)
  })
}

async function typeMessage() {
  // Небольшая задержка после загрузки страницы.
  await sleep(1200)

  for (let i = 0; i < MESSAGE.length; i++) {
    displayedText.value += MESSAGE[i]

    // Пауза после знаков препинания.
    let delay = 35 + Math.random() * 55

    if (".,!?".includes(MESSAGE[i])) {
      delay += 180 + Math.random() * 250
    }

    await sleep(delay)

    // Иногда во время набора случается короткий glitch.
    if (Math.random() < 0.025) {
      triggerSmallGlitch()
    }
  }

  typingFinished.value = true

  await sleep(900)

  // Сильный glitch перед появлением кнопки.
  triggerHardGlitch()

  await sleep(850)

  buttonVisible.value = true
}

function triggerSmallGlitch() {
  glitching.value = true

  window.setTimeout(() => {
    glitching.value = false
  }, 80 + Math.random() * 140)
}

function triggerHardGlitch() {
  hardGlitch.value = true
  screenFlash.value = true

  window.setTimeout(() => {
    screenFlash.value = false
  }, 90)

  window.setTimeout(() => {
    hardGlitch.value = false
  }, 600)
}

function scheduleRandomGlitch() {
  const delay = 2500 + Math.random() * 5500

  glitchTimer = window.setTimeout(() => {
    triggerSmallGlitch()

    if (Math.random() < 0.18) {
      window.setTimeout(() => {
        triggerSmallGlitch()
      }, 120)
    }

    scheduleRandomGlitch()
  }, delay)
}

function scheduleRandomFlash() {
  const delay = 8000 + Math.random() * 13000

  flashTimer = window.setTimeout(() => {
    if (Math.random() < 0.4) {
      screenFlash.value = true

      window.setTimeout(() => {
        screenFlash.value = false
      }, 40 + Math.random() * 50)
    }

    scheduleRandomFlash()
  }, delay)
}

function openTelegram() {
  window.open(TELEGRAM_URL, "_blank", "noopener,noreferrer")
}

onMounted(() => {
  typeMessage()
  scheduleRandomGlitch()
  scheduleRandomFlash()
})

onBeforeUnmount(() => {
  clearTimeout(typeTimer)
  clearTimeout(glitchTimer)
  clearTimeout(buttonTimer)
  clearTimeout(flashTimer)
})
</script>

<template>
  <main
    class="screen"
    :class="{
      'screen--glitch': glitching,
      'screen--hard-glitch': hardGlitch,
      'screen--flash': screenFlash,
    }"
  >
    <!-- Фоновый шум -->
    <div class="noise"></div>

    <!-- CRT-полосы -->
    <div class="scanlines"></div>

    <!-- Затемнение краёв -->
    <div class="vignette"></div>

    <section class="terminal">
      <div class="terminal-header">
        <span>SESSION://UNKNOWN</span>

        <span
          class="terminal-status"
          :class="{ connected: displayedText.length > 0 }"
        >
          {{ displayedText.length > 0 ? "CONNECTED" : "WAITING" }}
        </span>
      </div>

      <div class="terminal-body">
        <div class="terminal-line">
          <span class="prompt">&gt;</span>

          <div class="message-wrapper">
            <!-- Основной текст -->
            <span
              class="message"
              :data-text="glitchText"
            >
              {{ displayedText }}
            </span>

            <!-- Мигание курсора -->
            <span
              v-if="!typingFinished"
              class="cursor"
            ></span>
          </div>
        </div>

        <Transition name="button-appear">
          <div
            v-if="buttonVisible"
            class="access-area"
          >
            <div class="warning">
              [ UNAUTHORIZED SIGNAL DETECTED ]
            </div>

            <button
              class="tui-button"
              type="button"
              @click="openTelegram"
            >
              <span class="button-bracket">[</span>

              <span
                class="button-text"
                data-text="ENTER"
              >
                ENTER
              </span>

              <span class="button-bracket">]</span>
            </button>
          </div>
        </Transition>
      </div>
    </section>

    <div class="bottom-status">
      <span>NODE: ████████</span>
      <span>TRACE: ACTIVE</span>
    </div>
  </main>
</template>

<style>
/* ==========================
   GLOBAL
   ========================== */

* {
  box-sizing: border-box;
}

html,
body,
#app {
  width: 100%;
  min-width: 100%;
  height: 100%;
  min-height: 100%;
  margin: 0;
}

body {
  overflow: hidden;
  background: #000;
}

/* ==========================
   SCREEN
   ========================== */

.screen {
  --terminal-color: #b7ffb7;
  --terminal-dim: #638c63;
  --terminal-glow: rgba(124, 255, 124, 0.35);

  position: relative;

  display: flex;
  align-items: center;
  justify-content: center;

  width: 100vw;
  height: 100vh;

  overflow: hidden;

  background:
    radial-gradient(
      circle at center,
      #071007 0%,
      #020502 45%,
      #000 100%
    );

  color: var(--terminal-color);

  font-family:
    "Courier New",
    "Lucida Console",
    monospace;

  transition:
    filter 60ms linear,
    transform 60ms linear;
}

/* ==========================
   TERMINAL
   ========================== */

.terminal {
  position: relative;
  z-index: 5;

  width: min(900px, 88vw);
  min-height: 300px;

  border: 1px solid rgba(130, 255, 130, 0.16);

  background: rgba(0, 5, 0, 0.58);

  box-shadow:
    0 0 40px rgba(0, 255, 50, 0.03),
    inset 0 0 80px rgba(0, 255, 50, 0.015);

  backdrop-filter: blur(2px);
}

.terminal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;

  height: 34px;

  padding: 0 12px;

  border-bottom: 1px solid rgba(130, 255, 130, 0.12);

  color: rgba(160, 255, 160, 0.35);

  font-size: 10px;
  letter-spacing: 0.18em;
}

.terminal-status {
  opacity: 0.55;
}

.terminal-status.connected {
  color: #7cff7c;

  text-shadow:
    0 0 8px rgba(80, 255, 80, 0.7);

  opacity: 1;
}

.terminal-body {
  display: flex;
  flex-direction: column;
  justify-content: center;

  min-height: 265px;

  padding: clamp(25px, 5vw, 60px);
}

.terminal-line {
  display: flex;
  align-items: flex-start;

  line-height: 1.75;
}

.prompt {
  margin-right: 15px;

  color: #4cff4c;

  text-shadow:
    0 0 8px rgba(60, 255, 60, 0.65);
}

.message-wrapper {
  position: relative;
}

.message {
  position: relative;

  color: var(--terminal-color);

  font-size: clamp(15px, 1.7vw, 20px);
  letter-spacing: 0.025em;

  text-shadow:
    0 0 4px rgba(180, 255, 180, 0.4),
    0 0 12px rgba(80, 255, 80, 0.12);
}

/* ==========================
   CURSOR
   ========================== */

.cursor {
  display: inline-block;

  width: 9px;
  height: 1.15em;

  margin-left: 5px;

  vertical-align: -0.15em;

  background: #8aff8a;

  box-shadow:
    0 0 7px rgba(100, 255, 100, 0.8);

  animation: cursor-blink 0.75s steps(1) infinite;
}

@keyframes cursor-blink {
  0%,
  45% {
    opacity: 1;
  }

  46%,
  100% {
    opacity: 0;
  }
}

/* ==========================
   TUI BUTTON
   ========================== */

.access-area {
  margin-top: 45px;
  margin-left: 31px;
}

.warning {
  margin-bottom: 15px;

  color: #ff6b6b;

  font-size: 10px;
  letter-spacing: 0.2em;

  text-shadow:
    0 0 6px rgba(255, 60, 60, 0.7),
    0 0 18px rgba(255, 0, 0, 0.35);

  animation: warning-flicker 3s infinite;
}

@keyframes warning-flicker {
  0%,
  94%,
  100% {
    opacity: 0.45;
  }

  95% {
    opacity: 0.9;
  }

  96% {
    opacity: 0.1;
  }

  97% {
    opacity: 0.75;
  }
}

.tui-button {
  position: relative;

  padding: 8px 13px;

  border: 0;

  background: transparent;

  color: #ff3434;

  font: inherit;
  font-size: 16px;
  font-weight: 700;
  letter-spacing: 0.2em;

  cursor: pointer;

  text-shadow:
    0 0 5px rgba(255, 0, 0, 0.8),
    0 0 14px rgba(255, 0, 0, 0.35);

  transition:
    background 100ms linear,
    color 100ms linear,
    text-shadow 100ms linear;
}

.tui-button:hover,
.tui-button:focus-visible {
  outline: none;

  background: #b60000;

  color: #000;

  text-shadow: none;

  box-shadow:
    0 0 8px rgba(255, 0, 0, 0.85),
    0 0 30px rgba(255, 0, 0, 0.35);
}

.button-bracket {
  opacity: 0.7;
}

.button-text {
  position: relative;

  margin: 0 8px;
}

.tui-button:hover .button-text {
  animation: button-glitch 150ms infinite;
}

@keyframes button-glitch {
  0% {
    transform: translate(0);
  }

  25% {
    transform: translate(-2px, 1px);
  }

  50% {
    transform: translate(2px, -1px);
  }

  75% {
    transform: translate(-1px, -1px);
  }

  100% {
    transform: translate(0);
  }
}

/* ==========================
   BUTTON TRANSITION
   ========================== */

.button-appear-enter-active {
  animation: button-materialize 650ms steps(2, end);
}

@keyframes button-materialize {
  0% {
    opacity: 0;
    transform: translateX(-15px);
    filter: blur(6px);
  }

  20% {
    opacity: 1;
    transform: translateX(12px);
    filter: blur(0);
  }

  35% {
    opacity: 0.15;
    transform: translateX(-8px);
  }

  50% {
    opacity: 1;
    transform: translateX(4px);
  }

  65% {
    opacity: 0.25;
  }

  100% {
    opacity: 1;
    transform: translateX(0);
  }
}

/* ==========================
   GLITCH
   ========================== */

.screen--glitch .terminal {
  animation: terminal-glitch 110ms steps(2, end);
}

.screen--glitch .message::before,
.screen--glitch .message::after {
  content: attr(data-text);

  position: absolute;
  inset: 0;

  pointer-events: none;
}

.screen--glitch .message::before {
  color: #ff193f;

  transform: translate(-2px, 0);

  clip-path: inset(20% 0 55% 0);

  opacity: 0.6;
}

.screen--glitch .message::after {
  color: #00ffff;

  transform: translate(2px, 0);

  clip-path: inset(60% 0 10% 0);

  opacity: 0.45;
}

@keyframes terminal-glitch {
  0% {
    transform: translate(0);
  }

  25% {
    transform: translate(-3px, 1px);
  }

  50% {
    transform: translate(4px, -1px);
  }

  75% {
    transform: translate(-1px, 0);
  }

  100% {
    transform: translate(0);
  }
}

/* Сильный glitch перед появлением ENTER */

.screen--hard-glitch .terminal {
  animation: hard-glitch 600ms steps(1, end);
}

@keyframes hard-glitch {
  0% {
    transform: translate(0);
    filter: none;
  }

  8% {
    transform: translate(-10px, 1px);
    filter: contrast(1.8);
  }

  16% {
    transform: translate(8px, -3px);
  }

  20% {
    transform: translate(0);
  }

  31% {
    transform: skewX(2deg);
  }

  33% {
    transform: skewX(-4deg) translateX(8px);
  }

  37% {
    transform: none;
  }

  65% {
    filter: brightness(1.5);
  }

  67% {
    filter: brightness(0.4);
  }

  70% {
    filter: none;
  }

  100% {
    transform: none;
  }
}

/* ==========================
   SCANLINES
   ========================== */

.scanlines {
  position: absolute;
  z-index: 20;
  inset: 0;

  pointer-events: none;

  background:
    repeating-linear-gradient(
      to bottom,
      rgba(255, 255, 255, 0) 0,
      rgba(255, 255, 255, 0) 2px,
      rgba(0, 0, 0, 0.13) 3px,
      rgba(0, 0, 0, 0.13) 4px
    );

  opacity: 0.55;
}

/* Бегущая светлая CRT-линия */

.scanlines::after {
  content: "";

  position: absolute;

  width: 100%;
  height: 80px;

  top: -100px;
  left: 0;

  background: linear-gradient(
    to bottom,
    transparent,
    rgba(120, 255, 120, 0.02),
    transparent
  );

  animation: scan 8s linear infinite;
}

@keyframes scan {
  from {
    transform: translateY(-100px);
  }

  to {
    transform: translateY(calc(100vh + 200px));
  }
}

/* ==========================
   NOISE
   ========================== */

.noise {
  position: absolute;
  z-index: 15;
  inset: -100px;

  pointer-events: none;

  opacity: 0.025;

  background-image: url(
    "data:image/svg+xml,%3Csvg viewBox='0 0 180 180' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noiseFilter'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noiseFilter)'/%3E%3C/svg%3E"
  );

  animation: noise 0.18s steps(2) infinite;
}

@keyframes noise {
  0% {
    transform: translate(0, 0);
  }

  25% {
    transform: translate(4%, -3%);
  }

  50% {
    transform: translate(-3%, 4%);
  }

  75% {
    transform: translate(2%, 5%);
  }

  100% {
    transform: translate(-4%, -2%);
  }
}

/* ==========================
   VIGNETTE
   ========================== */

.vignette {
  position: absolute;
  z-index: 25;
  inset: 0;

  pointer-events: none;

  background:
    radial-gradient(
      ellipse at center,
      transparent 45%,
      rgba(0, 0, 0, 0.45) 75%,
      rgba(0, 0, 0, 0.85) 100%
    );
}

/* ==========================
   SCREEN FLASH
   ========================== */

.screen::after {
  content: "";

  position: absolute;
  z-index: 50;
  inset: 0;

  pointer-events: none;

  background: rgba(210, 255, 210, 0);

  mix-blend-mode: screen;
}

.screen--flash::after {
  background: rgba(210, 255, 210, 0.08);
}

/* ==========================
   BOTTOM STATUS
   ========================== */

.bottom-status {
  position: absolute;
  z-index: 6;

  display: flex;
  justify-content: space-between;

  right: 20px;
  bottom: 15px;
  left: 20px;

  color: rgba(120, 255, 120, 0.14);

  font-size: 8px;
  letter-spacing: 0.18em;

  user-select: none;
}

/* ==========================
   SLIGHT CRT FLICKER
   ========================== */

.screen {
  animation: crt-flicker 7s infinite;
}

@keyframes crt-flicker {
  0%,
  91%,
  93%,
  97%,
  100% {
    opacity: 1;
  }

  92% {
    opacity: 0.985;
  }

  96% {
    opacity: 0.995;
  }
}

/* ==========================
   MOBILE
   ========================== */

@media (max-width: 600px) {
  .terminal {
    width: 92vw;
  }

  .terminal-body {
    padding: 25px 20px;
  }

  .message {
    font-size: 14px;
  }

  .access-area {
    margin-left: 29px;
  }

  .bottom-status {
    display: none;
  }
}

/*
  Если пользователь попросил ОС уменьшить анимации,
  не устраиваем ему эпилептический пиздец.
*/
@media (prefers-reduced-motion: reduce) {
  .noise,
  .scanlines::after,
  .screen {
    animation: none;
  }
}
</style>