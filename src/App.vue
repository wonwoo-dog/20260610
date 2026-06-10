<script setup>
import { ref, computed, onUnmounted, watch } from 'vue'

// ================== 課程 ==================
const courseTitle = ref('新課程單元')
const maxLimit = ref(46)

const segments = ref([
  { id: 1, label: '講解單元', time: 8, color: '#38bdf8', icon: 'auto_stories' },
  { id: 2, label: '小組討論', time: 18, color: '#34d399', icon: 'groups' },
  { id: 3, label: '課堂活動', time: 15, color: '#fbbf24', icon: 'sports_esports' },
  { id: 4, label: '隨堂測驗', time: 10, color: '#fb7185', icon: 'edit_note' }
])

const totalTime = computed(() =>
  segments.value.reduce((a, b) => a + Number(b.time || 0), 0)
)

const isOvertime = computed(() => totalTime.value > maxLimit.value)

// 🔴 改這裡：Modal控制
const showModal = ref(false)
const hasAlerted = ref(false)

watch(isOvertime, (val) => {
  if (val && !hasAlerted.value) {
    showModal.value = true
    hasAlerted.value = true
  }
  if (!val) hasAlerted.value = false
})

const addSegment = () => {
  segments.value.push({
    id: Date.now(),
    label: '新環節',
    time: 5,
    color: '#a78bfa',
    icon: 'add_circle'
  })
}

const deleteSegment = (id) => {
  segments.value = segments.value.filter(s => s.id !== id)
}

// ================== 計時器 ==================
const timerSeconds = ref(0)
const isRunning = ref(false)
let intervalId = null

const formattedTimer = computed(() => {
  const m = Math.floor(timerSeconds.value / 60).toString().padStart(2, '0')
  const s = (timerSeconds.value % 60).toString().padStart(2, '0')
  return `${m}:${s}`
})

const toggleTimer = () => {
  if (isRunning.value) clearInterval(intervalId)
  else intervalId = setInterval(() => timerSeconds.value++, 1000)
  isRunning.value = !isRunning.value
}

const resetTimer = () => {
  clearInterval(intervalId)
  isRunning.value = false
  timerSeconds.value = 0
}

onUnmounted(() => clearInterval(intervalId))

// ================== 抽籤 ==================
const studentRawList = ref('王大明 李小美 張一山 陳二魚 林志玲 周星星')
const pickedName = ref('')
const isSpinning = ref(false)

const drawStudent = () => {
  const list = studentRawList.value.trim().split(/\s+/).filter(Boolean)
  if (!list.length || isSpinning.value) return

  isSpinning.value = true
  let i = 0

  const t = setInterval(() => {
    pickedName.value = list[Math.floor(Math.random() * list.length)]
    i++
    if (i > 15) {
      clearInterval(t)
      isSpinning.value = false
    }
  }, 80)
}

// ================== 挑戰 ==================
const challenges = [
  '⚡ 閃電答題',
  '🔄 角色互換',
  '🤫 秘密互評',
  '⏳ 15字總結',
  '🔥 全班搶答',
  '🃏 猜拳決定'
]

const currentChallenge = ref('點擊產生挑戰')
const isRollingChallenge = ref(false)

const drawChallenge = () => {
  if (isRollingChallenge.value) return

  isRollingChallenge.value = true
  let i = 0

  const t = setInterval(() => {
    currentChallenge.value =
      challenges[Math.floor(Math.random() * challenges.length)]

    i++
    if (i > 10) {
      clearInterval(t)
      isRollingChallenge.value = false
    }
  }, 100)
}

// ================== 筆記 ==================
const notes = ref('')

const copyNotes = async () => {
  await navigator.clipboard.writeText(notes.value || '')
  alert('已複製教案')
}
</script>

<template>
  <div class="app">

    <div class="bg"></div>

    <!-- ✅ 改這裡：標題升級 -->
    <header class="header">
      <div class="title-group">
        <div class="main-title">課程節奏計時器</div>
        <div class="sub-title">📘 {{ courseTitle }}</div>
      </div>

      <div class="inputs">
        <input v-model="courseTitle" />
        <input v-model.number="maxLimit" type="number" />
      </div>
    </header>

    <div class="layout">

      <div class="card timer">
        <div class="time">{{ formattedTimer }}</div>

        <div class="btns">
          <button @click="toggleTimer">{{ isRunning ? '暫停' : '開始' }}</button>
          <button @click="resetTimer">重置</button>
        </div>
      </div>

      <div class="main">

        <div class="card">
          <div class="progress-head">
            <span>{{ totalTime }} / {{ maxLimit }} min</span>
            <span v-if="isOvertime" class="warn">超時</span>
          </div>

          <div class="bar">
            <div
              v-for="s in segments"
              :key="s.id"
              class="bar-seg"
              :style="{ width: (s.time / (totalTime || 1)) * 100 + '%', background: s.color }"
            />
          </div>
        </div>

        <div class="card">
          <button class="add" @click="addSegment">＋ 新增環節</button>

          <div v-for="s in segments" :key="s.id" class="row">

            <span class="material-icons" :style="{ color: s.color }">
              {{ s.icon }}
            </span>

            <input v-model="s.label" class="label" />

            <input
              type="range"
              min="0"
              max="60"
              v-model.number="s.time"
              class="slider"
              :style="{ '--c': s.color }"
            />

            <span class="num">{{ s.time }}</span>

            <button class="del" @click="deleteSegment(s.id)">×</button>
          </div>
        </div>

        <div class="grid">

          <div class="card">
            <div class="card-title">🎲 抽籤</div>
            <div class="big">{{ pickedName }}</div>
            <button @click="drawStudent">抽學生</button>
            <input v-model="studentRawList" />
          </div>

          <div class="card">
            <div class="card-title">⚡ 挑戰</div>
            <div class="big small">{{ currentChallenge }}</div>
            <button @click="drawChallenge">產生</button>
          </div>

          <div class="card">
            <div class="card-title">📝 筆記</div>
            <textarea v-model="notes"></textarea>
            <button @click="copyNotes">複製</button>
          </div>

        </div>

      </div>
    </div>

    <!-- 🔴 新增：彈出視窗 -->
    <div v-if="showModal" class="overlay">
      <div class="modal">
        <h2>⚠️ 已超過預定時間</h2>
        <p>目前 {{ totalTime }} 分鐘（超過 {{ maxLimit }}）</p>
        <button @click="showModal = false">關閉</button>
      </div>
    </div>

  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/icon?family=Material+Icons');

/* 新增標題樣式 */
.title-group {
  display: flex;
  flex-direction: column;
}
.main-title {
  font-size: 20px;
  font-weight: bold;
}
.sub-title {
  font-size: 12px;
  opacity: 0.7;
}

/* Modal */
.overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.7);
  display:flex;
  justify-content:center;
  align-items:center;
  z-index: 999;
}
.modal {
  background: white;
  color: black;
  padding: 20px;
  border-radius: 12px;
  text-align: center;
}

/* 下面全部保留你原本 */
.app { min-height: 100vh; color: white; font-family: sans-serif; position: relative; overflow: hidden; }
.bg { position: absolute; inset: 0; background: radial-gradient(circle at 20% 20%, #1d4ed8, transparent),
              radial-gradient(circle at 80% 30%, #9333ea, transparent),
              radial-gradient(circle at 50% 80%, #0ea5e9, transparent);
  filter: blur(80px); opacity: 0.4; }

.header { position: relative; display: flex; justify-content: space-between; padding: 20px; backdrop-filter: blur(10px); }

.inputs input { margin-left: 8px; background: rgba(0,0,0,0.4); border: 1px solid rgba(255,255,255,0.1); color: white; padding: 4px 8px; }

.layout { display: grid; grid-template-columns: 300px 1fr; gap: 16px; padding: 20px; position: relative; }

.card { background: rgba(0,0,0,0.4); border: 1px solid rgba(255,255,255,0.08); border-radius: 16px; padding: 16px; backdrop-filter: blur(12px); }

.timer .time { font-size: 48px; text-align: center; color: #4ade80; }

.btns { display: flex; gap: 8px; margin-top: 10px; }

.bar { display: flex; height: 10px; border-radius: 999px; overflow: hidden; margin-top: 10px; background: rgba(255,255,255,0.1); }

.row { display: flex; align-items: center; gap: 8px; margin-top: 8px; }

.slider { flex: 1; }

.slider::-webkit-slider-thumb { background: var(--c); }

.grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; margin-top: 12px; }

.big { font-size: 20px; text-align: center; margin: 10px 0; }

button { background: #3b82f6; border: none; color: white; padding: 6px 10px; border-radius: 8px; cursor: pointer; }

.warn { color: #f87171; }
</style>