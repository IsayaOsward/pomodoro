<template>
  <div class="relative flex flex-col items-center justify-center h-screen bg-white text-black px-4">
    <!-- Settings still top-right -->
    <div class="absolute top-4 right-4">
      <!-- … same gear icon and settings panel … -->
    </div>

    <!-- Timer ring -->
    <div class="relative w-64 h-64 mb-8">
      <svg class="transform -rotate-90 w-full h-full">
        <circle class="text-gray-300" stroke-width="12" stroke="currentColor"
                fill="transparent" :r="radius" cx="128" cy="128" />
        <circle class="text-gray-500 transition-all duration-1000 ease-linear"
                :stroke-dasharray="circumference"
                :stroke-dashoffset="strokeDashoffset"
                stroke-width="12" stroke="currentColor"
                fill="transparent" :r="radius" cx="128" cy="128" />
      </svg>
      <div :class="['absolute inset-0 flex items-center justify-center font-bold',
                   isRunning ? 'text-gray-700' : 'text-black']"
           class="text‑4xl">
        {{ formattedTime }}
      </div>
    </div>

    <!-- Controls -->
    <div class="flex gap-4 mb-8">
      <button class="bg-green-600 px-4 py-2 rounded hover:bg-green-700 transition"
              @click="toggleTimer">
        {{ isRunning ? 'Pause' : 'Start' }}
      </button>
      <button class="bg-red-600 px-4 py-2 rounded hover:bg-red-700 transition"
              @click="resetTimer">
        Reset
      </button>
    </div>

    <!-- History Log -->
    <div class="w-full max-w-md text-left">
      <h2 class="text-lg font-semibold mb-2">Session History</h2>
      <ul>
        <li v-for="(h,i) in history" :key="i" class="mb-1 p-2 bg-gray‑100 rounded">
          <div>Session: {{ h.start | time }} → {{ h.end | time }} 
            <span v-if="h.reason" class="text-red-500">({{ h.reason }})</span>
          </div>
          <div>Break: {{ h.breakStart | time }} → {{ h.breakEnd | time }}</div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onUnmounted } from 'vue'

// Constants
const radius = 90
const circumference = 2 * Math.PI * radius

// State
const isRunning = ref(false)
const isOnBreak = ref(false)
const timeLeft = ref(0)
const sessionDuration = ref(5 * 60)
const breakDuration = ref(1 * 60)
const history = ref([])

const sessionInput = ref(sessionDuration.value / 60)
const breakInput = ref(breakDuration.value / 60)

let interval = null
let sessionStart = null

const formattedTime = computed(() => {
  const m = Math.floor(timeLeft.value / 60)
  const s = timeLeft.value % 60
  return `${String(m).padStart(2,'0')}:${String(s).padStart(2,'0')}`
})

const strokeDashoffset = computed(() => {
  const total = isOnBreak.value ? breakDuration.value : sessionDuration.value
  return circumference - (timeLeft.value / total) * circumference
})

function startTimer() {
  sessionStart = Date.now()
  interval = setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value--
    } else {
      finishPeriod()
    }
  }, 1000)
}

function finishPeriod() {
  stopTimer()
  const now = Date.now()
  const rec = {
    start: sessionStart,
    end: now,
    breakStart: now,
    breakEnd: now + breakDuration.value * 1000,
    reason: ''
  }
  // check if session ran long
  if (!isOnBreak.value) {
    const expected = sessionStart + sessionDuration.value * 1000
    if (rec.end > expected) rec.reason = 'You had a pause'
  }
  history.value.unshift(rec)
  // start break or new session
  isOnBreak.value = !isOnBreak.value
  timeLeft.value = isOnBreak.value ? breakDuration.value : sessionDuration.value
  isRunning.value = true
  sessionStart = Date.now()
  startTimer()
}

function stopTimer() {
  clearInterval(interval); interval = null
}

function toggleTimer() {
  isRunning.value = !isRunning.value
  if (isRunning.value) {
    if (!interval) sessionStart = Date.now()
    timeLeft.value = timeLeft.value || (isOnBreak.value ? breakDuration.value : sessionDuration.value)
    startTimer()
  } else {
    stopTimer()
  }
}

function resetTimer() {
  stopTimer()
  isRunning.value = false
  isOnBreak.value = false
  timeLeft.value = sessionDuration.value
}

// Apply settings like before...
function applySettings() {
  sessionDuration.value = Math.max(1, sessionInput.value) * 60
  breakDuration.value = Math.max(1, breakInput.value) * 60
  resetTimer()
  showSettings.value = false
}

onUnmounted(stopTimer)

// Time filter if using options API or register manually
</script>
