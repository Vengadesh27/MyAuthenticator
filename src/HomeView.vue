<template>
  <div class="app-container">
    <div class="otp-card">
      <div class="header">Your One-Time Password</div>

      <div class="otp-wrapper" @click="copyOTP">
        <div class="otp-display" :class="{ 'pulse': isActive }">
          {{ otpValue }}
        </div>
        <div class="copy-hint" :class="{ 'copied': justCopied }">
          {{ copyText }}
        </div>
      </div>

      <div class="info">
        <span>Refreshes every 30 seconds</span>
        <div class="timer" v-if="secondsLeft > 0">
          Next refresh in <strong>{{ secondsLeft }}s</strong>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, onUnmounted } from 'vue'
import { TOTP } from "totp-generator"

const otpValue = ref('Loading...')
const secondsLeft = ref(30)
const isActive = ref(true)
const justCopied = ref(false)
const copyText = ref('Click to copy')

let intervalId = null
let timerId = null

async function generateOTP() {
  try {
    const { otp } = await TOTP.generate("CMYTNC3VNDJM2NDW")
    otpValue.value = otp
    secondsLeft.value = 30
    isActive.value = true

    setTimeout(() => {
      isActive.value = false
    }, 1200)

  } catch (error) {
    console.error("OTP generation failed:", error)
    otpValue.value = "Error"
  }
}

async function copyOTP() {
  if (!otpValue.value || otpValue.value === 'Loading...' || otpValue.value === 'Error') return

  try {
    await navigator.clipboard.writeText(otpValue.value)
    justCopied.value = true
    copyText.value = 'Copied! ✓'

    setTimeout(() => {
      justCopied.value = false
      copyText.value = 'Click to copy'
    }, 1800)
  } catch (err) {
    console.error('Failed to copy:', err)
    copyText.value = 'Copy failed'
    setTimeout(() => {
      copyText.value = 'Click to copy'
    }, 2000)
  }
}

function startTimers() {
  generateOTP()

  intervalId = setInterval(() => {
    generateOTP()
  }, 30000)

  timerId = setInterval(() => {
    secondsLeft.value = Math.max(0, secondsLeft.value - 1)
  }, 1000)
}

onMounted(() => {
  startTimers()
})

onUnmounted(() => {
  if (intervalId) clearInterval(intervalId)
  if (timerId) clearInterval(timerId)
})
</script>

<style scoped>
.app-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #0f0f1a 0%, #131324 100%);
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Segoe UI', system-ui, sans-serif;
  color: #e0e0ff;
}

.otp-card {
  background: rgba(25, 25, 40, 0.75);
  backdrop-filter: blur(10px);
  border-radius: 20px;
  padding: 3rem 2.5rem;
  width: 380px;
  max-width: 90%;
  box-shadow:
    0 20px 40px rgba(0, 0, 0, 0.6),
    0 0 0 1px rgba(100, 100, 255, 0.12);
  border: 1px solid rgba(100, 100, 255, 0.15);
  text-align: center;
}

.header {
  font-size: 1.1rem;
  color: #a0a0ff;
  letter-spacing: 1.5px;
  text-transform: uppercase;
  margin-bottom: 1.8rem;
  font-weight: 500;
}

.otp-wrapper {
  position: relative;
  cursor: pointer;
  user-select: none;
  transition: transform 0.15s ease;
}

.otp-wrapper:hover {
  transform: translateY(-2px);
}

.otp-display {
  font-size: 3.6rem;
  font-weight: 700;
  letter-spacing: 12px;
  font-family: 'Courier New', Courier, monospace;
  color: #00d4ff;
  text-shadow: 0 0 20px rgba(0, 212, 255, 0.45);
  background: rgba(0, 40, 60, 0.35);
  padding: 1.2rem 1.8rem;
  border-radius: 16px;
  border: 1px solid rgba(0, 212, 255, 0.25);
  transition: all 0.4s ease;
}

.otp-display.pulse {
  animation: pulse-glow 1.8s ease-in-out;
}

@keyframes pulse-glow {
  0%, 100% { box-shadow: 0 0 20px rgba(0, 212, 255, 0.3); }
  50% { box-shadow: 0 0 40px rgba(0, 212, 255, 0.7), 0 0 60px rgba(0, 170, 255, 0.4); }
}

.copy-hint {
  position: absolute;
  bottom: -2.2rem;
  left: 50%;
  transform: translateX(-50%);
  font-size: 0.9rem;
  color: #9090cc;
  opacity: 0.7;
  transition: all 0.3s ease;
  pointer-events: none;
}

.copy-hint.copied {
  color: #00ff9d;
  opacity: 1;
  font-weight: 500;
  transform: translateX(-50%) scale(1.1);
}

.info {
  color: #9090cc;
  font-size: 0.95rem;
  margin-top: 3.2rem;
}

.timer {
  margin-top: 0.6rem;
  color: #60a0ff;
  font-weight: 500;
}

.timer strong {
  color: #00d4ff;
}
</style>