<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

const theme = ref('light')
const showZhuyin = ref(false)
const displayMode = ref('clock')
const currentTime = ref('00:00:00')
const ticker = ref(Date.now())
const countdownSeconds = ref(300)
const countdownMinutes = ref(5)
const countdownId = ref(null)
const clockId = ref(null)
const timerRunning = ref(false)

const examSubject = ref('')
const examDuration = ref('')
const examProctor = ref('')
const scheduleStart = ref({ month: new Date().getMonth() + 1, day: new Date().getDate(), hour: 9, minute: 0 })
const scheduleEnd = ref({ month: new Date().getMonth() + 1, day: new Date().getDate(), hour: 10, minute: 30 })

const months = Array.from({ length: 12 }, (_, i) => i + 1)
const days = Array.from({ length: 31 }, (_, i) => i + 1)
const hours = Array.from({ length: 24 }, (_, i) => i)
const minutes = Array.from({ length: 12 }, (_, i) => i * 5)

const labelMap = {
  title: {
    plain: '考試系統',
    zhuyin:
      '<ruby>考<rt>ㄎ⎯ㄠˇ</rt></ruby><ruby>試<rt>ㄕˋ</rt></ruby>系統',
  },
  leftButton: {
    plain: '淡江教科系',
    zhuyin:
      '<ruby>淡<rt>ㄉㄢˋ</rt></ruby><ruby>江<rt>ㄐㄧㄤ</rt></ruby><ruby>教<rt>ㄐㄧㄠˋ</rt></ruby><ruby>科<rt>ㄎㄜ</rt></ruby><ruby>系<rt>ㄒㄧˋ</rt></ruby>',
  },
  timeToggle: {
    plain: '倒數 / 現在時間',
    zhuyin:
      '<ruby>倒<rt>ㄉㄠˇ</rt></ruby><ruby>數<rt>ㄕㄨˋ</rt></ruby> / <ruby>現<rt>ㄒㄧㄢˋ</rt></ruby><ruby>在<rt>ㄗㄞˋ</rt></ruby>時間',
  },
  countdown: {
    plain: '倒數時間',
    zhuyin:
      '<ruby>倒<rt>ㄉㄠˇ</rt></ruby><ruby>數<rt>ㄕㄨˋ</rt></ruby><ruby>時<rt>ㄕˊ</rt></ruby><ruby>間<rt>ㄐㄧㄢ</rt></ruby>',
  },
  currentTime: {
    plain: '現在時間',
    zhuyin:
      '<ruby>現<rt>ㄒㄧㄢˋ</rt></ruby><ruby>在<rt>ㄗㄞˋ</rt></ruby><ruby>時<rt>ㄕˊ</rt></ruby><ruby>間<rt>ㄐㄧㄢ</rt></ruby>',
  },
  zhuyin: {
    plain: '注音',
    zhuyin:
      '<ruby>注<rt>ㄓㄨˋ</rt></ruby><ruby>音<rt>ㄧㄣ</rt></ruby>',
  },
  day: {
    plain: '白天',
    zhuyin:
      '<ruby>白<rt>ㄅㄞˊ</rt></ruby><ruby>天<rt>ㄊㄧㄢ</rt></ruby>',
  },
  night: {
    plain: '夜間',
    zhuyin:
      '<ruby>夜<rt>ㄧㄝˋ</rt></ruby><ruby>間<rt>ㄐㄧㄢ</rt></ruby>',
  },
  description: {
    plain:
      '這是一個為老師設計的考試小工具，包含時間顯示、倒數計時、注音與主題切換。',
    zhuyin:
      '<ruby>這<rt>ㄓㄜˋ</rt></ruby><ruby>是<rt>ㄕˋ</rt></ruby><ruby>一<rt>ㄧ</rt></ruby><ruby>個<rt>ㄍㄜˋ</rt></ruby><ruby>為<rt>ㄨㄟˋ</rt></ruby><ruby>老<rt>ㄌㄠˇ</rt></ruby><ruby>師<rt>ㄕ</rt></ruby><ruby>設<rt>ㄕㄜˋ</rt></ruby><ruby>計<rt>ㄐㄧˋ</rt></ruby><ruby>的<rt>˙ㄉㄜ</rt></ruby><ruby>考<rt>ㄎㄠˇ</rt></ruby><ruby>試<rt>ㄕˋ</rt></ruby><ruby>小<rt>ㄒㄧㄠˇ</rt></ruby><ruby>工<rt>ㄍㄨㄥ</rt></ruby><ruby>具<rt>ㄐㄩˋ</rt></ruby>，<ruby>包<rt>ㄅㄠ</rt></ruby><ruby>含<rt>ㄏㄢˊ</rt></ruby><ruby>時<rt>ㄕˊ</rt></ruby><ruby>間<rt>ㄐㄧㄢ</rt></ruby><ruby>顯<rt>ㄒㄧㄢˇ</rt></ruby><ruby>示<rt>ㄕˋ</rt></ruby>、<ruby>倒<rt>ㄉㄠˇ</rt></ruby><ruby>數<rt>ㄕㄨˋ</rt></ruby><ruby>計<rt>ㄐㄧˋ</rt></ruby><ruby>時<rt>ㄕˊ</rt></ruby>、<ruby>注<rt>ㄓㄨˋ</rt></ruby><ruby>音<rt>ㄧㄣ</rt></ruby><ruby>與<rt>ㄩˇ</rt></ruby><ruby>主<rt>ㄓㄨˇ</rt></ruby><ruby>題<rt>ㄊㄧˊ</rt></ruby><ruby>切<rt>ㄑㄧㄝ</rt></ruby><ruby>換<rt>ㄏㄨㄢˋ</rt></ruby>。',
  },
  examInfo: {
    plain: '考試資訊',
    zhuyin:
      '<ruby>考<rt>ㄎ⎯ㄠˇ</rt></ruby><ruby>試<rt>ㄕˋ</rt></ruby>資訊',
  },
  formTitle: {
    plain: '輸入考試資料',
    zhuyin:
      '<ruby>輸<rt>ㄕㄨ</rt></ruby><ruby>入<rt>ㄖㄨˋ</rt></ruby>考試資料',
  },
  subjectLabel: {
    plain: '科目',
    zhuyin:
      '<ruby>科<rt>ㄎㄜ</rt></ruby><ruby>目<rt>ㄇㄨˋ</rt></ruby>',
  },
  durationLabel: {
    plain: '考試時間',
    zhuyin:
      '<ruby>考<rt>ㄎ⎯ㄠˇ</rt></ruby><ruby>試<rt>ㄕˋ</rt></ruby>時間',
  },
  proctorLabel: {
    plain: '監考老師',
    zhuyin:
      '<ruby>監<rt>ㄐㄧㄢ</rt></ruby><ruby>考<rt>ㄎ⎯ㄠˇ</rt></ruby><ruby>老<rt>ㄌㄠˇ</rt></ruby><ruby>師<rt>ㄕ</rt></ruby>',
  },
  scheduleTitle: {
    plain: '考試時程',
    zhuyin:
      '<ruby>考<rt>ㄎ⎯ㄠˇ</rt></ruby><ruby>試<rt>ㄕˋ</rt></ruby>時程',
  },
  startLabel: {
    plain: '開始時間',
    zhuyin: '<ruby>開<rt>ㄎㄞ</rt></ruby><ruby>始<rt>ㄕˇ</rt></ruby>時間',
  },
  endLabel: {
    plain: '結束時間',
    zhuyin: '<ruby>結<rt>ㄐㄧㄝˊ</rt></ruby><ruby>束<rt>ㄕㄨˋ</rt></ruby>時間',
  },
  countdownMinLabel: {
    plain: '倒數分鐘',
    zhuyin: '<ruby>倒<rt>ㄉㄠˇ</rt></ruby><ruby>數<rt>ㄕㄨˋ</rt></ruby><ruby>分<rt>ㄈㄣ</rt></ruby><ruby>鐘<rt>ㄓㄨㄥ</rt></ruby>',
  },
  startButton: {
    plain: '開始',
    zhuyin: '<ruby>開<rt>ㄎㄞ</rt></ruby><ruby>始<rt>ㄕˇ</rt></ruby>',
  },
  resetButton: {
    plain: '重設',
    zhuyin: '<ruby>重<rt>ㄔㄨㄥˊ</rt></ruby><ruby>設<rt>ㄕㄜˋ</rt></ruby>',
  },
  formNote: {
    plain: '輸入資料後會立即顯示在下方考試資訊卡片。',
    zhuyin: '<ruby>輸<rt>ㄕㄨ</rt></ruby><ruby>入<rt>ㄖㄨˋ</rt></ruby><ruby>資<rt>ㄗ</rt></ruby><ruby>料<rt>ㄌㄧㄠˋ</rt></ruby><ruby>後<rt>ㄏㄡˋ</rt></ruby><ruby>會<rt>ㄏㄨㄟˋ</rt></ruby><ruby>立<rt>ㄌㄧˋ</rt></ruby><ruby>即<rt>ㄐㄧˊ</rt></ruby><ruby>顯<rt>ㄒㄧㄢˇ</rt></ruby><ruby>示<rt>ㄕˋ</rt></ruby><ruby>在<rt>ㄗㄞˋ</rt></ruby><ruby>下<rt>ㄒㄧㄚˋ</rt></ruby><ruby>方<rt>ㄈㄤ</rt></ruby><ruby>考<rt>ㄎㄠˇ</rt></ruby><ruby>試<rt>ㄕˋ</rt></ruby><ruby>資<rt>ㄗ</rt></ruby><ruby>訊<rt>ㄒㄩㄣˋ</rt></ruby><ruby>卡<rt>ㄎㄚˇ</rt></ruby><ruby>片<rt>ㄆㄧㄢˋ</rt></ruby>。',
  },
  minutesText: {
    plain: ' 分鐘',
    zhuyin: ' <ruby>分<rt>ㄈㄣ</rt></ruby><ruby>鐘<rt>ㄓㄨㄥ</rt></ruby>',
  },
  monthLabel: {
    plain: '月',
    zhuyin: '<ruby>月<rt>ㄩㄝˋ</rt></ruby>',
  },
  dayLabel: {
    plain: '日',
    zhuyin: '<ruby>日<rt>ㄖˋ</rt></ruby>',
  },
  hourLabel: {
    plain: '時',
    zhuyin: '<ruby>時<rt>ㄕˊ</rt></ruby>',
  },
  minuteLabel: {
    plain: '分',
    zhuyin: '<ruby>分<rt>ㄈㄣ</rt></ruby>',
  },
  remainingTime: {
    plain: '剩餘時間',
    zhuyin: '<ruby>剩<rt>ㄕㄥˋ</rt></ruby><ruby>餘<rt>ㄩˊ</rt></ruby>時間',
  },
  notFilled: {
    plain: '尚未輸入',
    zhuyin:
      '<ruby>尚<rt>ㄕㄤˋ</rt></ruby><ruby>未<rt>ㄨㄟˋ</rt></ruby>輸入',
  },
}

const themeLabel = computed(() => activeLabel(theme.value === 'light' ? 'night' : 'day'))

function activeLabel(key) {
  return showZhuyin.value ? labelMap[key].zhuyin : labelMap[key].plain
}

function updateCurrentTime() {
  const now = new Date()
  ticker.value = now.getTime()
  currentTime.value = now
    .toLocaleTimeString('zh-TW', {
      hour12: false,
      hour: '2-digit',
      minute: '2-digit',
      second: '2-digit',
    })
}

function toggleTheme() {
  theme.value = theme.value === 'light' ? 'dark' : 'light'
}

function toggleZhuyin() {
  showZhuyin.value = !showZhuyin.value
}

function toggleMode() {
  const isClock = displayMode.value === 'clock'
  displayMode.value = isClock ? 'countdown' : 'clock'
  if (!isClock) {
    stopCountdown()
  }
}

function formatSeconds(value) {
  const minutes = Math.floor(Math.max(0, value) / 60)
    .toString()
    .padStart(2, '0')
  const seconds = (Math.max(0, value) % 60).toString().padStart(2, '0')
  return `${minutes}:${seconds}`
}

function startCountdown() {
  if (countdownId.value) {
    return
  }
  if (countdownSeconds.value === 0) {
    countdownSeconds.value = countdownMinutes.value * 60
  }
  timerRunning.value = true
  countdownId.value = window.setInterval(() => {
    if (countdownSeconds.value > 0) {
      countdownSeconds.value -= 1
    } else {
      stopCountdown()
    }
  }, 1000)
}

function stopCountdown() {
  if (countdownId.value) {
    window.clearInterval(countdownId.value)
    countdownId.value = null
  }
  timerRunning.value = false
}

function resetCountdown() {
  stopCountdown()
  countdownSeconds.value = countdownMinutes.value * 60
}

const countdownDisplay = computed(() => formatSeconds(countdownSeconds.value))

const currentYear = new Date().getFullYear()

function makeScheduleDate({ month, day, hour, minute }) {
  const date = new Date(currentYear, month - 1, day, hour, minute, 0)
  if (date.getMonth() !== month - 1 || date.getDate() !== day) {
    return null
  }
  return date
}

const scheduleStartDate = computed(() => makeScheduleDate(scheduleStart.value))
const scheduleEndDate = computed(() => makeScheduleDate(scheduleEnd.value))

const scheduleRange = computed(() => {
  if (!scheduleStartDate.value || !scheduleEndDate.value) {
    return '尚未設定'
  }
  if (scheduleEndDate.value <= scheduleStartDate.value) {
    return '結束時間必須晚於開始時間'
  }
  const start = scheduleStartDate.value
  const end = scheduleEndDate.value
  return `${start.getMonth() + 1}/${start.getDate()} ${start
    .getHours()
    .toString()
    .padStart(2, '0')}:${start
    .getMinutes()
    .toString()
    .padStart(2, '0')} → ${end.getMonth() + 1}/${end.getDate()} ${end
    .getHours()
    .toString()
    .padStart(2, '0')}:${end
    .getMinutes()
    .toString()
    .padStart(2, '0')}`
})

const scheduleRemaining = computed(() => {
  if (!scheduleEndDate.value) {
    return '尚未設定結束時間'
  }
  const diff = scheduleEndDate.value.getTime() - ticker.value
  if (diff <= 0) {
    return '已結束'
  }
  const totalSeconds = Math.floor(diff / 1000)
  const days = Math.floor(totalSeconds / 86400)
  const hoursLeft = Math.floor((totalSeconds % 86400) / 3600)
  const mins = Math.floor((totalSeconds % 3600) / 60)
  const secs = totalSeconds % 60
  return `${days}天 ${hoursLeft}小時 ${mins}分 ${secs}秒`
})

onMounted(() => {
  updateCurrentTime()
  clockId.value = window.setInterval(updateCurrentTime, 1000)
})

onBeforeUnmount(() => {
  stopCountdown()
  if (clockId.value) {
    window.clearInterval(clockId.value)
    clockId.value = null
  }
})
</script>

<template>
  <div :class="['app-shell', theme === 'dark' ? 'theme-night' : 'theme-day']">
    <div class="container">
      <header class="header">
        <a
          class="link-button"
          href="https://www.tku.edu.tw/"
          target="_blank"
          rel="noreferrer"
          v-html="activeLabel('leftButton')"
        ></a>
        <div class="title-wrap">
          <h1 v-html="activeLabel('title')" class="page-title"></h1>
          <p class="subtext" v-html="activeLabel('description')"></p>
        </div>
      </header>

      <div class="toolbar">
        <button class="button primary" type="button" @click="toggleMode" v-html="activeLabel('timeToggle')"></button>
        <button class="button" type="button" @click="toggleZhuyin" v-html="activeLabel('zhuyin')"></button>
        <button class="button" type="button" @click="toggleTheme" v-html="themeLabel"></button>
      </div>

      <section class="display-card">
        <div class="display-title" v-html="displayMode === 'clock' ? activeLabel('currentTime') : activeLabel('countdown')"></div>
        <div class="display-value">{{ displayMode === 'clock' ? currentTime : countdownDisplay }}</div>
        <div class="display-note" v-if="displayMode === 'countdown'">
          <label>
            <span v-html="activeLabel('countdownMinLabel')"></span>
            <input
              type="number"
              min="1"
              v-model.number="countdownMinutes"
              @change="resetCountdown"
            />
          </label>
          <div class="countdown-actions">
            <button class="button small" type="button" @click="startCountdown" :disabled="timerRunning" v-html="activeLabel('startButton')"></button>
            <button class="button small" type="button" @click="resetCountdown" v-html="activeLabel('resetButton')"></button>
          </div>
        </div>
      </section>

      <div class="form-card">
        <h2 v-html="activeLabel('formTitle')"></h2>
        <div class="form-row">
          <label for="subject-input" v-html="activeLabel('subjectLabel')"></label>
          <input id="subject-input" v-model="examSubject" placeholder="例：國文、數學" />
        </div>
        <div class="form-row">
          <label for="duration-input" v-html="activeLabel('durationLabel')"></label>
          <input
            id="duration-input"
            v-model.number="examDuration"
            type="number"
            min="1"
            placeholder="輸入考試時間（分鐘）"
          />
        </div>
        <div class="form-row">
          <label for="proctor-input" v-html="activeLabel('proctorLabel')"></label>
          <input id="proctor-input" v-model="examProctor" placeholder="監考老師姓名" />
        </div>

        <div class="schedule-block">
          <h3 v-html="activeLabel('scheduleTitle')"></h3>
          <div class="schedule-row">
            <div>
              <label v-html="activeLabel('startLabel')"></label>
              <div class="schedule-selects">
                <select v-model.number="scheduleStart.month">
                  <option v-for="month in months" :key="month" :value="month">{{ month }}月</option>
                </select>
                <select v-model.number="scheduleStart.day">
                  <option v-for="day in days" :key="day" :value="day">{{ day }}日</option>
                </select>
                <select v-model.number="scheduleStart.hour">
                  <option v-for="hour in hours" :key="hour" :value="hour">{{ hour }}時</option>
                </select>
                <select v-model.number="scheduleStart.minute">
                  <option v-for="minute in minutes" :key="minute" :value="minute">{{ minute }}分</option>
                </select>
              </div>
            </div>
            <div>
              <label v-html="activeLabel('endLabel')"></label>
              <div class="schedule-selects">
                <select v-model.number="scheduleEnd.month">
                  <option v-for="month in months" :key="month" :value="month">{{ month }}月</option>
                </select>
                <select v-model.number="scheduleEnd.day">
                  <option v-for="day in days" :key="day" :value="day">{{ day }}日</option>
                </select>
                <select v-model.number="scheduleEnd.hour">
                  <option v-for="hour in hours" :key="hour" :value="hour">{{ hour }}時</option>
                </select>
                <select v-model.number="scheduleEnd.minute">
                  <option v-for="minute in minutes" :key="minute" :value="minute">{{ minute }}分</option>
                </select>
              </div>
            </div>
          </div>
        </div>

        <p class="form-note" v-html="activeLabel('formNote')"></p>
      </div>

      <section class="panel">
        <div class="panel-item exam-summary-card">
          <h2 v-html="activeLabel('examInfo')"></h2>
          <p><strong v-html="activeLabel('subjectLabel')"></strong>：<span v-html="examSubject || activeLabel('notFilled')"></span></p>
          <p><strong v-html="activeLabel('durationLabel')"></strong>：{{ examDuration ? examDuration : '' }}<span v-if="examDuration" v-html="activeLabel('minutesText')"></span><span v-else v-html="activeLabel('notFilled')"></span></p>
          <p><strong v-html="activeLabel('proctorLabel')"></strong>：<span v-html="examProctor || activeLabel('notFilled')"></span></p>
          <p><strong v-html="activeLabel('startLabel')"></strong>：<span>{{ scheduleRange }}</span></p>
          <p><strong v-html="activeLabel('remainingTime')"></strong>：{{ scheduleRemaining }}</p>
        </div>
      </section>
    </div>
  </div>
</template>

<style scoped>
.app-shell {
  min-height: 100vh;
  padding: 24px;
  box-sizing: border-box;
  transition: background 0.3s ease, color 0.3s ease;
  display: flex;
  justify-content: center;
  align-items: center;
}

.theme-day {
  background: #f6f8ff;
  color: #111827; /* 深色文字 */
}

.theme-night {
  background: #0f172a;
  color: #f8fafc; /* 淺色文字 */
}

.app-shell * {
  color: inherit; /* 確保所有子元素繼承主題顏色 */
}

.container {
  width: min(100%, 980px);
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 22px;
}

.header {
  display: flex;
  flex-wrap: wrap;
  align-items: flex-start;
  justify-content: space-between;
  gap: 16px;
}

.title-wrap {
  flex: 1 1 320px;
  min-width: 240px;
}

.page-title {
  margin: 0;
  font-size: clamp(2.4rem, 5vw, 4rem);
}

.subtext {
  margin: 12px 0 0;
  font-size: 1rem;
  max-width: 720px;
  line-height: 1.7;
  color: inherit;
  opacity: 0.82;
}

.link-button {
  appearance: none;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 160px;
  padding: 14px 18px;
  border-radius: 14px;
  border: 1px solid rgba(255, 255, 255, 0.18);
  background: rgba(59, 130, 246, 0.95);
  color: #fff;
  font-weight: 600;
  text-decoration: none;
  text-align: center;
  transition: transform 0.2s ease, background 0.2s ease;
}

.link-button:hover {
  transform: translateY(-1px);
  background: rgba(37, 99, 235, 0.95);
}

.toolbar {
  display: flex;
  flex-wrap: wrap;
  gap: 14px;
  justify-content: center;
}

.button {
  border: 1px solid transparent;
  border-radius: 14px;
  padding: 14px 18px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.2s ease, background 0.2s ease, color 0.2s ease;
  background: rgba(255, 255, 255, 0.72);
  color: inherit;
}

.button:hover {
  transform: translateY(-1px);
}

.button.primary {
  background: #4338ca;
  color: #fff;
}

.button.small {
  padding: 10px 14px;
  font-size: 0.95rem;
}

.button:disabled {
  opacity: 0.55;
  cursor: not-allowed;
}

.panel {
  display: grid;
  grid-template-columns: 1fr;
  gap: 18px;
}

.panel-item {
  border-radius: 24px;
  padding: 24px;
  background: rgba(255, 255, 255, 0.7);
  box-shadow: 0 20px 45px rgba(15, 23, 42, 0.08);
  color: inherit;
}

.theme-night .panel-item {
  background: rgba(15, 23, 42, 0.75);
}

.panel-item h2 {
  margin: 0 0 14px;
  font-size: 1.3rem;
}

.panel-item ul {
  margin: 0;
  padding-left: 18px;
  line-height: 1.8;
}

.countdown-card {
  display: grid;
  gap: 18px;
  border-radius: 24px;
  padding: 24px;
  background: rgba(59, 130, 246, 0.12);
  border: 1px solid rgba(59, 130, 246, 0.18);
}

.theme-night .countdown-card {
  background: rgba(30, 58, 138, 0.26);
  border-color: rgba(148, 163, 184, 0.22);
}

.countdown-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  font-weight: 700;
}

.badge {
  border-radius: 999px;
  padding: 6px 12px;
  background: rgba(16, 185, 129, 0.14);
  color: #065f46;
  font-size: 0.95rem;
}

.countdown-value {
  font-size: clamp(2.15rem, 5vw, 3rem);
  font-weight: 700;
  letter-spacing: 1px;
}

.countdown-actions {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
}

.status-grid {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 18px;
}

.status-card,
.form-card {
  border-radius: 24px;
  padding: 22px;
  background: rgba(255, 255, 255, 0.78);
  box-shadow: 0 20px 45px rgba(15, 23, 42, 0.08);
}

.theme-night .status-card,
.theme-night .form-card {
  background: rgba(15, 23, 42, 0.82);
}

.status-title {
  display: block;
  margin-bottom: 10px;
  font-weight: 700;
}

.status-value {
  font-size: clamp(1.8rem, 4vw, 2.6rem);
  font-weight: 700;
}

.exam-summary p {
  margin: 0 0 10px;
  line-height: 1.7;
}

.form-row {
  margin-bottom: 16px;
}

.form-row label {
  display: block;
  margin-bottom: 8px;
  font-weight: 600;
}

.form-row input,
.form-row select {
  width: 100%;
  border-radius: 14px;
  border: 1px solid rgba(15, 23, 42, 0.12);
  padding: 12px 14px;
  font-size: 1rem;
  background: rgba(255, 255, 255, 0.95);
  color: inherit;
  outline: none;
}

.theme-night .form-row input,
.theme-night .form-row select,
.theme-night .schedule-selects select {
  background: rgba(15, 23, 42, 0.92);
  border-color: rgba(148, 163, 184, 0.25);
}

.schedule-selects select {
  width: 100%;
  border-radius: 14px;
  border: 1px solid rgba(15, 23, 42, 0.12);
  padding: 12px 14px;
  font-size: 1rem;
  background: rgba(255, 255, 255, 0.95);
  color: inherit;
  outline: none;
}

.form-note {
  margin: 0;
  color: inherit;
  opacity: 0.8;
  font-size: 0.95rem;
}

.display-card {
  border-radius: 24px;
  padding: 24px;
  background: rgba(255, 255, 255, 0.9);
  box-shadow: 0 20px 45px rgba(15, 23, 42, 0.08);
}

.theme-night .display-card {
  background: rgba(15, 23, 42, 0.82);
}

.display-title {
  margin-bottom: 16px;
  font-size: 1.25rem;
  font-weight: 700;
}

.display-value {
  font-size: clamp(2rem, 6vw, 3.5rem);
  font-weight: 800;
  letter-spacing: 0.04em;
}

.display-note {
  margin-top: 18px;
  display: grid;
  gap: 16px;
}

.display-note label {
  display: grid;
  gap: 10px;
  font-weight: 600;
}

.display-note input {
  width: 100%;
  border-radius: 14px;
  border: 1px solid rgba(15, 23, 42, 0.15);
  padding: 12px 14px;
  font-size: 1rem;
  background: rgba(255, 255, 255, 0.96);
  color: inherit;
}

.theme-night .display-note input {
  background: rgba(15, 23, 42, 0.9);
  border-color: rgba(148, 163, 184, 0.25);
}

.schedule-block {
  margin-top: 24px;
}

.schedule-row {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 18px;
}

.schedule-selects {
  display: grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap: 12px;
  margin-top: 12px;
}

.exam-summary-card {
  background: rgba(255, 255, 255, 0.82);
}

.theme-night .exam-summary-card {
  background: rgba(15, 23, 42, 0.82);
}

ruby {
  display: inline-flex;
  flex-direction: row;
  align-items: center;
  vertical-align: middle;
  ruby-position: inter-character;
}

rt {
  display: block;
  font-size: 0.45em;
  line-height: 1;
  margin-left: 0.2em;
  margin-right: 0.2em;
  white-space: nowrap;
}

@media (max-width: 680px) {
  .header {
    flex-direction: column;
    align-items: stretch;
  }

  .link-button {
    width: 100%;
  }

  .toolbar {
    justify-content: stretch;
  }

  .button {
    width: 100%;
  }

  .status-grid {
    grid-template-columns: 1fr;
  }
}
</style>
