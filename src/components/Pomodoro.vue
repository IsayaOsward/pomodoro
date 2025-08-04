<template>
  <div class="min-h-screen bg-gray-50 py-8 px-4">
    <div class="max-w-4xl mx-auto">
      <div class="text-center mb-8">
        <h1 class="text-4xl font-bold text-gray-900 mb-2">Pomodoro Timer</h1>
        <p class="text-gray-600">Stay focused and productive</p>
      </div>

      <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
        <div class="bg-white rounded-2xl shadow-lg p-8">
          <div class="text-center mb-8">
            <div class="relative inline-block">
              <svg class="w-64 h-64 transform -rotate-90" viewBox="0 0 100 100">
                <circle
                  cx="50"
                  cy="50"
                  r="45"
                  stroke="#e5e7eb"
                  stroke-width="8"
                  fill="none"
                />
                
                <circle
                  cx="50"
                  cy="50"
                  r="45"
                  :stroke="isBreak ? '#6b7280' : '#111827'"
                  stroke-width="8"
                  fill="none"
                  stroke-linecap="round"
                  :stroke-dasharray="circumference"
                  :stroke-dashoffset="strokeDashoffset"
                  class="transition-all duration-1000 ease-linear"
                />
              </svg>
              
              <!-- Timer Text Overlay -->
              <div class="absolute inset-0 flex flex-col items-center justify-center">
                <div class="text-4xl font-mono font-bold text-gray-900 mb-2">
                  {{ formatTime(timeLeft) }}
                </div>
                <div class="text-lg font-medium" :class="isBreak ? 'text-gray-600' : 'text-gray-900'">
                  {{ isBreak ? 'Break Time' : 'Focus Time' }}
                </div>
                <div class="text-sm text-gray-500 mt-1">
                  Session {{ currentSession }}
                </div>
                <div v-if="sessionStartTime" class="text-xs text-gray-500 mt-2 space-y-1">
                  <div class="flex items-center justify-center space-x-2">
                    <span>Started: {{ formatTime12Hour(sessionStartTime) }}</span>
                    <div v-if="isOvertime" class="w-2 h-2 bg-red-500 rounded-full animate-pulse"></div>
                  </div>
                  <div v-if="expectedEndTime">
                    Expected end: {{ formatTime12Hour(expectedEndTime) }}
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Control Buttons -->
          <div class="flex justify-center space-x-4 mb-6">
            <button
              @click="toggleTimer"
              class="bg-gray-900 hover:bg-gray-800 text-white px-8 py-3 rounded-lg font-medium transition-colors duration-200 flex items-center space-x-2"
            >
              <Play v-if="!isRunning" class="w-5 h-5" />
              <Pause v-else class="w-5 h-5" />
              <span>{{ isRunning ? 'Pause' : 'Start' }}</span>
            </button>
            
            <button
              @click="resetTimer"
              class="bg-gray-200 hover:bg-gray-300 text-gray-900 px-8 py-3 rounded-lg font-medium transition-colors duration-200 flex items-center space-x-2"
            >
              <RotateCcw class="w-5 h-5" />
              <span>Reset</span>
            </button>
          </div>

          <!-- Settings -->
          <div class="border-t pt-6">
            <h3 class="text-lg font-semibold text-gray-900 mb-4">Timer Settings</h3>
            <div class="grid grid-cols-2 gap-4">
              <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">
                  Focus Time (minutes)
                </label>
                <input
                  v-model.number="settings.focusTime"
                  type="number"
                  min="1"
                  max="60"
                  class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-gray-500 focus:border-transparent"
                  :disabled="isRunning"
                />
              </div>
              <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">
                  Break Time (minutes)
                </label>
                <input
                  v-model.number="settings.breakTime"
                  type="number"
                  min="1"
                  max="30"
                  class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-gray-500 focus:border-transparent"
                  :disabled="isRunning"
                />
              </div>
              <div class="col-span-2 mt-4 p-4 bg-gray-50 rounded-lg border">
                <label class="flex items-center space-x-3 cursor-pointer">
                  <div class="relative">
                    <input
                      v-model="settings.autoStart"
                      type="checkbox"
                      class="sr-only"
                      :disabled="isRunning"
                    />
                    <div 
                      class="w-11 h-6 bg-gray-200 rounded-full shadow-inner transition-colors duration-200"
                      :class="settings.autoStart ? 'bg-gray-900' : 'bg-gray-200'"
                    ></div>
                    <div 
                      class="absolute w-4 h-4 bg-white rounded-full shadow transition-transform duration-200 top-1"
                      :class="settings.autoStart ? 'translate-x-6' : 'translate-x-1'"
                    ></div>
                  </div>
                  <div>
                    <span class="text-sm font-medium text-gray-900">
                      Auto-start breaks and sessions
                    </span>
                    <p class="text-xs text-gray-500">
                      Automatically start the next session or break without manual intervention
                    </p>
                  </div>
                </label>
              </div>
            </div>
          </div>

          <!-- Stats -->
          <div class="border-t pt-6 mt-6">
            <div class="grid grid-cols-2 gap-4 text-center">
              <div>
                <div class="text-2xl font-bold text-gray-900">{{ completedSessions }}</div>
                <div class="text-sm text-gray-600">Completed Sessions</div>
              </div>
              <div>
                <div class="text-2xl font-bold text-gray-900">{{ formatTotalTime(totalTimeSpent) }}</div>
                <div class="text-sm text-gray-600">Total Time</div>
              </div>
            </div>
          </div>
        </div>

        <!-- History Section -->
        <div class="bg-white rounded-2xl shadow-lg p-8">
          <h3 class="text-lg font-semibold text-gray-900 mb-6 flex items-center">
            <History class="w-5 h-5 mr-2" />
            Session History
          </h3>
          
          <div class="space-y-3 max-h-96 overflow-y-auto">
            <div
              v-for="session in sessionHistory"
              :key="session.id"
              class="flex items-center justify-between p-4 bg-gray-50 rounded-lg"
            >
              <div class="flex items-center space-x-3">
                <div class="flex items-center space-x-1">
                  <div class="w-3 h-3 rounded-full" :class="session.type === 'focus' ? 'bg-gray-900' : 'bg-gray-500'"></div>
                  <div v-if="session.wasOvertime" class="w-2 h-2 bg-red-500 rounded-full" title="Session went overtime"></div>
                </div>
                <div>
                  <div class="font-medium text-gray-900">
                    {{ session.type === 'focus' ? 'Focus Session' : 'Break Session' }}
                  </div>
                  <div class="text-sm text-gray-600">
                    {{ formatTime(session.duration) }} • {{ session.status }}
                  </div>
                  <div v-if="session.startTime && session.endTime" class="text-xs text-gray-500">
                    {{ formatTime12Hour(session.startTime) }} - {{ formatTime12Hour(session.endTime) }}
                  </div>
                </div>
              </div>
              <div class="text-sm text-gray-500">
                {{ formatDate(session.timestamp) }}
              </div>
            </div>
            
            <div v-if="sessionHistory.length === 0" class="text-center py-8 text-gray-500">
              <Clock class="w-12 h-12 mx-auto mb-3 opacity-50" />
              <p>No sessions yet. Start your first Pomodoro!</p>
            </div>
          </div>
          
          <button
            v-if="sessionHistory.length > 0"
            @click="clearHistory"
            class="w-full mt-4 bg-gray-100 hover:bg-gray-200 text-gray-700 py-2 px-4 rounded-lg font-medium transition-colors duration-200"
          >
            Clear History
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'
import { Play, Pause, RotateCcw, History, Clock } from 'lucide-vue-next'

const isRunning = ref(false)
const isBreak = ref(false)
const timeLeft = ref(25 * 60)
const currentSession = ref(1)
const completedSessions = ref(0)
const totalTimeSpent = ref(0)
const sessionHistory = ref([])

const sessionStartTime = ref(null)
const sessionEndTime = ref(null)
const expectedEndTime = ref(null)
const isOvertime = ref(false)

// default initial data kwaajili ya  focus time and break time
const settings = ref({
  focusTime: 25,
  breakTime: 5,
  autoStart: false
})

let timerInterval = null
let sessionStartTimeLeft = null

const circumference = computed(() => 2 * Math.PI * 45)

const strokeDashoffset = computed(() => {
  const totalTime = isBreak.value ? settings.value.breakTime * 60 : settings.value.focusTime * 60
  const progress = (totalTime - timeLeft.value) / totalTime
  return circumference.value * (1 - progress)
})

// Method ya kuweka muda into human form
const formatTime = (seconds) => {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`
}

const formatTotalTime = (seconds) => {
  const hours = Math.floor(seconds / 3600)
  const mins = Math.floor((seconds % 3600) / 60)
  if (hours > 0) {
    return `${hours}h ${mins}m`
  }
  return `${mins}m`
}

const formatDate = (timestamp) => {
  return new Date(timestamp).toLocaleTimeString([], { 
    hour: '2-digit', 
    minute: '2-digit' 
  })
}

const formatTime12Hour = (timestamp) => {
  if (!timestamp) return ''
  return new Date(timestamp).toLocaleTimeString([], { 
    hour: '2-digit', 
    minute: '2-digit',
    hour12: true
  })
}

const toggleTimer = () => {
  if (isRunning.value) {
    pauseTimer()
  } else {
    startTimer()
  }
}

const startTimer = () => { 
  isRunning.value = true
  const now = Date.now()
  
  if (!sessionStartTime.value) {
    sessionStartTime.value = now
    const sessionDuration = isBreak.value ? settings.value.breakTime * 60 * 1000 : settings.value.focusTime * 60 * 1000
    expectedEndTime.value = now + sessionDuration
  }
  
  sessionStartTimeLeft = timeLeft.value
  
  timerInterval = setInterval(() => {
    if (timeLeft.value > 0) {
      timeLeft.value--
      isOvertime.value = Date.now() > expectedEndTime.value
    } else {
      completeSession()
    }
  }, 1000)
}

const pauseTimer = () => {
  isRunning.value = false
  if (timerInterval) {
    clearInterval(timerInterval)
    timerInterval = null
  }
}

const resetTimer = () => {
  pauseTimer()
  
  if (sessionStartTime.value && sessionStartTimeLeft !== timeLeft.value) {
    const sessionDuration = sessionStartTimeLeft - timeLeft.value
    logSession(isBreak.value ? 'break' : 'focus', sessionDuration, 'reset', sessionStartTime.value, Date.now())
  }
  
  // Reset timer and timing data
  timeLeft.value = isBreak.value ? settings.value.breakTime * 60 : settings.value.focusTime * 60
  sessionStartTime.value = null
  sessionEndTime.value = null
  expectedEndTime.value = null
  isOvertime.value = false
  sessionStartTimeLeft = null
}

const completeSession = () => {
  pauseTimer()
  
  sessionEndTime.value = Date.now()
  
  const sessionDuration = isBreak.value ? settings.value.breakTime * 60 : settings.value.focusTime * 60
  logSession(isBreak.value ? 'break' : 'focus', sessionDuration, 'completed', sessionStartTime.value, sessionEndTime.value)
  
  if (!isBreak.value) {
    completedSessions.value++
  }
  totalTimeSpent.value += sessionDuration
  
  isBreak.value = !isBreak.value
  
  if (!isBreak.value) {
    currentSession.value++
  }
  
  sessionStartTime.value = null
  sessionEndTime.value = null
  expectedEndTime.value = null
  isOvertime.value = false
  
  // Set new timer duration
  timeLeft.value = isBreak.value ? settings.value.breakTime * 60 : settings.value.focusTime * 60
  
  // Auto-start next session if enabled
  if (settings.value.autoStart) {
    setTimeout(() => {
      startTimer()
    }, 1000)
  }
}

const logSession = (type, duration, status, startTime = null, endTime = null) => {
  sessionHistory.value.unshift({
    id: Date.now(),
    type,
    duration,
    status,
    timestamp: Date.now(),
    startTime,
    endTime,
    wasOvertime: endTime && startTime && (endTime - startTime) > duration * 1000
  })
  
  // itatunza only data 50
  if (sessionHistory.value.length > 50) {
    sessionHistory.value = sessionHistory.value.slice(0, 50)
  }
}

const clearHistory = () => {
  sessionHistory.value = []
  localStorage.clear()
}


watch(() => settings.value.focusTime, (newVal) => {
  if (!isRunning.value && !isBreak.value) {
    timeLeft.value = newVal * 60
  }
})

watch(() => settings.value.breakTime, (newVal) => {
  if (!isRunning.value && isBreak.value) {
    timeLeft.value = newVal * 60
  }
})

onMounted(() => {
  
  // Load saved data from localStorage
  const savedData = localStorage.getItem('pomodoro-data')
  if (savedData) {
    const data = JSON.parse(savedData)
    completedSessions.value = data.completedSessions || 0
    totalTimeSpent.value = data.totalTimeSpent || 0
    sessionHistory.value = data.sessionHistory || []
    settings.value = { ...settings.value, ...data.settings }
  }
  
  // Set initial timer
  timeLeft.value = settings.value.focusTime * 60
})

onUnmounted(() => {
  if (timerInterval) {
    clearInterval(timerInterval)
  }
})

// Save data to localStorage when state changes
watch([completedSessions, totalTimeSpent, sessionHistory, settings], () => {
  const dataToSave = {
    completedSessions: completedSessions.value,
    totalTimeSpent: totalTimeSpent.value,
    sessionHistory: sessionHistory.value,
    settings: settings.value
  }
  localStorage.setItem('pomodoro-data', JSON.stringify(dataToSave))
}, { deep: true })
</script>

<style scoped>

/*  STYLE KWAAJILI YA SCROLL INDICATOR */
.overflow-y-auto::-webkit-scrollbar {
  width: 6px;
}

.overflow-y-auto::-webkit-scrollbar-track {
  background: #f1f5f9;
  border-radius: 3px;
}

.overflow-y-auto::-webkit-scrollbar-thumb {
  background: #cbd5e1;
  border-radius: 3px;
}

.overflow-y-auto::-webkit-scrollbar-thumb:hover {
  background: #94a3b8;
}
</style>
