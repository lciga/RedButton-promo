<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from "vue"
import fsocietyArt from "../fsociety.txt?raw"

const MESSAGE = "Connection established. Мы наблюдаем. Если ты нашёл это, значит умеешь замечать детали. Продолжай искать."
const CHOICE_MESSAGE = "Выбираешь синюю таблетку — и сказке конец. Выбираешь красную — остаёшься в Стране чудес."
const TELEGRAM_URL = import.meta.env.VITE_TELEGRAM_URL || "https://t.me/"
const VK_URL = import.meta.env.VITE_VK_URL || "https://vk.com/"
type Scene = "message" | "glitch" | "choice"

const scene = ref<Scene>("message")
const displayedText = ref("")
const choiceText = ref("")
const choicesVisible = ref(false)
const glitching = ref(false)
const alive = ref(true)
const timers = new Set<number>()
const typingFinished = computed(() => displayedText.value.length === MESSAGE.length)
const choiceTypingFinished = computed(() => choiceText.value.length === CHOICE_MESSAGE.length)

function wait(ms: number) {
  return new Promise<void>((resolve) => {
    const timer = window.setTimeout(() => { timers.delete(timer); resolve() }, ms)
    timers.add(timer)
  })
}

async function pulseGlitch() {
  glitching.value = true
  await wait(110)
  glitching.value = false
}

async function typeText(text: string, target: { value: string }) {
  for (const character of text) {
    if (!alive.value) return
    target.value += character

    let delay = 28 + Math.random() * 38
    if (".,!?—".includes(character)) delay += 140 + Math.random() * 180

    await wait(delay)
    if (Math.random() < 0.04) void pulseGlitch()
  }
}

async function runSequence() {
  await wait(900)
  await typeText(MESSAGE, displayedText)
  await wait(850)
  scene.value = "glitch"
  await wait(2400)
  if (!alive.value) return

  scene.value = "choice"
  await wait(450)
  await typeText(CHOICE_MESSAGE, choiceText)
  await wait(650)
  if (alive.value) choicesVisible.value = true
}

onMounted(() => {
  if (window.matchMedia("(prefers-reduced-motion: reduce)").matches) {
    displayedText.value = MESSAGE
    choiceText.value = CHOICE_MESSAGE
    choicesVisible.value = true
    scene.value = "choice"
  } else void runSequence()
})

onBeforeUnmount(() => {
  alive.value = false
  timers.forEach((timer) => window.clearTimeout(timer))
  timers.clear()
})
</script>

<template>
  <main class="screen" :class="[`screen--${scene}`, { 'screen--pulse': glitching }]">
    <div class="noise" aria-hidden="true"></div>
    <div class="scanlines" aria-hidden="true"></div>
    <div class="vignette" aria-hidden="true"></div>

    <div v-if="scene === 'glitch'" class="rupture" aria-hidden="true">
      <pre class="ascii ascii--ghost ascii--cyan">{{ fsocietyArt }}</pre>
      <pre class="ascii ascii--ghost ascii--white">{{ fsocietyArt }}</pre>
      <pre class="ascii ascii--main">{{ fsocietyArt }}</pre>
    </div>

    <section class="terminal" aria-live="polite">
      <header class="terminal__header">
        <span>SESSION://UNKNOWN</span>
        <span :class="{ connected: displayedText.length }">{{ scene === "glitch" ? "SIGNAL LOST" : displayedText.length ? "CONNECTED" : "WAITING" }}</span>
      </header>

      <div class="terminal__body">
        <div v-if="scene === 'message'" class="terminal-line">
          <span class="prompt">&gt;</span>
          <p class="message" :data-text="displayedText">{{ displayedText }}<span v-if="!typingFinished" class="cursor" aria-hidden="true"></span></p>
        </div>

        <Transition name="reveal">
          <article v-if="scene === 'choice'" class="choice">
            <div class="terminal-line">
              <span class="prompt">&gt;</span>
              <p class="message choice__message" :data-text="choiceText">{{ choiceText }}<span v-if="!choiceTypingFinished" class="cursor" aria-hidden="true"></span></p>
            </div>
            <Transition name="buttons">
              <nav v-if="choicesVisible" class="pills" aria-label="Выбор социальной сети">
              <a class="pill pill--red" :href="TELEGRAM_URL" target="_blank" rel="noopener noreferrer"><span class="pill__bracket">[</span><span>Красная</span><span class="pill__bracket">]</span></a>
              <a class="pill pill--blue" :href="VK_URL" target="_blank" rel="noopener noreferrer"><span class="pill__bracket">[</span><span>Синяя</span><span class="pill__bracket">]</span></a>
              </nav>
            </Transition>
          </article>
        </Transition>
      </div>
    </section>

    <footer class="bottom-status"><span>NODE: ████████</span><span>TRACE: {{ scene === "choice" ? "SUSPENDED" : "ACTIVE" }}</span></footer>
  </main>
</template>

<style scoped>
.screen { --green:#a8ffaf; position:relative; display:grid; min-height:100svh; place-items:center; overflow:hidden; padding:24px; color:var(--green); background:radial-gradient(circle at 50% 45%,#071408 0,#010402 48%,#000 100%); font-family:"Courier New",ui-monospace,monospace }
.terminal { position:relative; z-index:4; width:min(900px,92vw); min-height:390px; border:1px solid rgb(132 255 144/18%); background:rgb(0 7 1/66%); box-shadow:0 0 80px rgb(0 255 54/5%),inset 0 0 80px rgb(0 255 54/2%); backdrop-filter:blur(2px) }
.terminal__header { display:flex; justify-content:space-between; padding:11px 14px; border-bottom:1px solid rgb(132 255 144/14%); color:rgb(168 255 175/35%); font-size:10px; letter-spacing:.18em }
.terminal__header .connected { color:#7cff86; opacity:1; text-shadow:0 0 8px #42ff55 }.screen--glitch .terminal__header span:last-child{color:#ff3030}
.terminal__body { display:grid; min-height:350px; padding:clamp(28px,6vw,64px); place-items:center stretch }.terminal-line{display:flex;align-items:flex-start;line-height:1.75}.prompt{margin-right:15px;color:#4cff5a;text-shadow:0 0 8px #3cff4e}
.message{position:relative;margin:0;font-size:clamp(15px,1.7vw,20px);letter-spacing:.025em;text-shadow:0 0 5px rgb(180 255 180/40%)}.cursor{display:inline-block;width:9px;height:1.15em;margin-left:5px;vertical-align:-.15em;background:#8aff93;box-shadow:0 0 7px #64ff70;animation:blink .75s steps(1) infinite}
.choice{width:100%}.choice__message{max-width:720px}.pills{display:grid;grid-template-columns:repeat(2,180px);justify-content:center;gap:clamp(34px,7vw,80px);margin:44px 0 0}.pill{position:relative;justify-self:center;padding:8px 13px;border:0;background:transparent;font:inherit;font-size:16px;font-weight:700;letter-spacing:.2em;text-decoration:none;text-transform:uppercase;transition:background .1s,color .1s,text-shadow .1s,box-shadow .1s}.pill span:not(.pill__bracket){display:inline-block;margin:0 8px}.pill__bracket{opacity:.7}.pill--red{color:#ff3434;text-shadow:0 0 5px rgb(255 0 0/80%),0 0 14px rgb(255 0 0/35%)}.pill--blue{color:#2995ff;text-shadow:0 0 5px rgb(0 119 255/85%),0 0 14px rgb(0 119 255/40%)}.pill--red:hover,.pill--red:focus-visible{background:#b60000;box-shadow:0 0 8px rgb(255 0 0/85%),0 0 30px rgb(255 0 0/35%)}.pill--blue:hover,.pill--blue:focus-visible{background:#0871c9;box-shadow:0 0 8px rgb(0 140 255/85%),0 0 30px rgb(0 119 255/35%)}.pill:hover,.pill:focus-visible{color:#000;text-shadow:none;outline:none}.pill:hover span:not(.pill__bracket),.pill:focus-visible span:not(.pill__bracket){animation:button-glitch .15s infinite}
.noise,.scanlines,.vignette,.rupture{position:absolute;inset:0;pointer-events:none}.noise{z-index:2;inset:-50%;opacity:.035;background-image:url("data:image/svg+xml,%3Csvg viewBox='0 0 180 180' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='.9' numOctaves='4'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E");animation:noise .16s steps(2) infinite}.scanlines{z-index:8;background:repeating-linear-gradient(to bottom,transparent 0 2px,rgb(0 0 0/18%) 3px 4px)}.scanlines::after{content:"";position:absolute;inset:-120px 0 auto;height:100px;background:linear-gradient(transparent,rgb(120 255 130/4%),transparent);animation:scan 7s linear infinite}.vignette{z-index:9;background:radial-gradient(ellipse,transparent 42%,rgb(0 0 0/50%) 78%,#000 110%)}.bottom-status{position:absolute;z-index:5;right:20px;bottom:15px;left:20px;display:flex;justify-content:space-between;color:rgb(120 255 130/16%);font-size:8px;letter-spacing:.18em}
.screen--pulse .terminal{animation:pulse .12s steps(2)}.screen--pulse .message::before,.screen--pulse .message::after{content:attr(data-text);position:absolute;inset:0}.screen--pulse .message::before{color:#ff1945;clip-path:inset(15% 0 55%);transform:translateX(-4px)}.screen--pulse .message::after{color:#00f7ff;clip-path:inset(60% 0 8%);transform:translateX(4px)}
.screen--glitch{animation:blackout 2.4s steps(1)}.screen--glitch .terminal{animation:terminal-rupture 2.4s steps(1)}.rupture{z-index:7;display:grid;place-items:center;overflow:hidden;background:#000}.ascii{grid-area:1/1;margin:0;color:#f2182f;font:700 min(1.55vw,1.55vh)/.96 "Courier New",ui-monospace,monospace;white-space:pre;text-shadow:0 0 5px rgb(255 0 30/85%),0 0 18px rgb(255 0 30/45%);transform-origin:center;animation:ascii-glitch .18s steps(2) infinite}.ascii--ghost{opacity:.5;mix-blend-mode:screen}.ascii--cyan{color:#00eaff;transform:translateX(5px);animation-delay:-.07s}.ascii--white{color:#fff;transform:translateX(-4px);animation-delay:-.12s}.ascii--main{position:relative;z-index:2}
.reveal-enter-active{animation:reveal 1s steps(5)}
.buttons-enter-active{animation:button-materialize .65s steps(2,end)}
@keyframes blink{0%,45%{opacity:1}46%,100%{opacity:0}}@keyframes noise{0%{transform:translate(0)}25%{transform:translate(4%,-3%)}50%{transform:translate(-3%,4%)}75%{transform:translate(2%,5%)}100%{transform:translate(-4%,-2%)}}@keyframes scan{to{transform:translateY(calc(100vh + 240px))}}@keyframes pulse{25%{transform:translate(-5px,2px)}60%{transform:translate(6px,-1px);filter:contrast(1.8)}}
@keyframes ascii-glitch{0%,100%{clip-path:inset(0);translate:0}20%{clip-path:inset(8% 0 70%);translate:-7px 0}40%{clip-path:inset(48% 0 32%);translate:9px 0}60%{clip-path:inset(76% 0 5%);translate:-4px 0}80%{clip-path:inset(24% 0 51%);translate:6px 0}}@keyframes terminal-rupture{0%{transform:translate(0)}4%{transform:translate(-8vw,2px) scaleX(1.2);filter:invert(1)}8%{transform:translate(7vw,-4px)}12%{transform:scaleY(.04)}16%{transform:scaleY(1.1) skewX(12deg);filter:hue-rotate(140deg)}20%{opacity:.12;transform:translateX(-15vw)}24%{opacity:1;transform:translateX(12vw) scaleX(.7)}28%{transform:scale(1.5);filter:invert(1) contrast(3)}32%,100%{opacity:0;transform:scaleY(0)}}@keyframes blackout{0%,12%,26%,88%{background-color:#000}5%,18%{background-color:#effff0}30%{filter:hue-rotate(160deg) contrast(2)}34%,100%{background-color:#000}}@keyframes reveal{0%{opacity:0;filter:blur(12px);transform:scaleY(.05)}35%{opacity:1;filter:blur(0);transform:scaleY(1.05) translateX(8px)}55%{opacity:.2;transform:translateX(-5px)}100%{opacity:1;transform:none}}
@keyframes button-materialize{0%{opacity:0;transform:translateX(-15px);filter:blur(6px)}20%{opacity:1;transform:translateX(12px);filter:blur(0)}35%{opacity:.15;transform:translateX(-8px)}50%{opacity:1;transform:translateX(4px)}65%{opacity:.25}100%{opacity:1;transform:none}}
@keyframes button-glitch{0%,100%{transform:translate(0)}25%{transform:translate(-2px,1px)}50%{transform:translate(2px,-1px)}75%{transform:translate(-1px,-1px)}}
@media(max-width:600px){.screen{padding:12px}.terminal{width:96vw;min-height:430px}.terminal__body{min-height:390px;padding:28px 20px}.pills{grid-template-columns:repeat(2,minmax(0,1fr));gap:12px;margin-top:34px}.pill{padding-inline:5px;font-size:11px}.bottom-status{display:none}}
@media(prefers-reduced-motion:reduce){*,*::before,*::after{animation-duration:.01ms!important;animation-iteration-count:1!important;transition-duration:.01ms!important}}
</style>
