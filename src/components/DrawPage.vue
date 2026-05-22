<script setup>
import { ref, computed, onUnmounted } from 'vue'

const props = defineProps({
  totalParticipants: Number,
  prizes: Array,
  winners: Object
})

const emit = defineEmits(['home'])

// 默认从最小奖项（列表最后一个）开始抽
const currentPrizeId = ref(props.prizes.length > 0 ? props.prizes[props.prizes.length - 1].id : null)
const isDrawing = ref(false)
const isRedrawing = ref(false)
const currentNumbers = ref([])
const redrawPreviousWinner = ref(null)
let drawInterval = null

const currentPrize = computed(() => props.prizes.find(p => p.id === currentPrizeId.value))

const drawnNumbersForCurrent = computed(() => {
  return currentPrizeId.value ? (props.winners[currentPrizeId.value] || []) : []
})

const isCurrentPrizeFinished = computed(() => {
  return currentPrize.value && drawnNumbersForCurrent.value.length >= currentPrize.value.count
})

const remainingCount = computed(() => {
  if (!currentPrize.value) return 0
  return currentPrize.value.count - drawnNumbersForCurrent.value.length
})

const getAllDrawnNumbers = () => {
  let all = []
  Object.values(props.winners).forEach(arr => {
    all = all.concat(arr)
  })
  return all
}

const startDraw = () => {
  if (isDrawing.value) return
  if (!currentPrizeId.value) return
  if (isCurrentPrizeFinished.value) {
    alert('该奖项已抽完')
    return
  }

  const allDrawn = getAllDrawnNumbers()
  if (allDrawn.length >= props.totalParticipants) {
    alert('所有参与者都已中奖')
    return
  }

  isDrawing.value = true
  const drawCount = Math.min(currentPrize.value.perDraw || 1, remainingCount.value)

  drawInterval = setInterval(() => {
    const temp = []
    for (let i = 0; i < drawCount; i++) {
      temp.push(Math.floor(Math.random() * props.totalParticipants) + 1)
    }
    currentNumbers.value = temp
  }, 50)
}

const stopDraw = () => {
  if (!isDrawing.value) return
  clearInterval(drawInterval)
  drawInterval = null

  if (isRedrawing.value) {
    const currentPrizeWinners = props.winners[currentPrizeId.value] || []
    const allDrawn = getAllDrawnNumbers()
    const candidates = []

    for (let i = 1; i <= props.totalParticipants; i++) {
      if (!allDrawn.includes(i) && i !== redrawPreviousWinner.value) candidates.push(i)
    }

    if (candidates.length === 0) {
      currentPrizeWinners.push(redrawPreviousWinner.value)
      currentNumbers.value = [redrawPreviousWinner.value]
      alert('没有可用的新号码可重抽')
    } else {
      const newWinner = candidates[Math.floor(Math.random() * candidates.length)]
      currentPrizeWinners.push(newWinner)
      currentNumbers.value = [newWinner]
    }

    isDrawing.value = false
    isRedrawing.value = false
    redrawPreviousWinner.value = null
    return
  }

  const drawCount = Math.min(currentPrize.value.perDraw || 1, remainingCount.value)
  const allDrawn = getAllDrawnNumbers()

  const available = []
  for (let i = 1; i <= props.totalParticipants; i++) {
    if (!allDrawn.includes(i)) available.push(i)
  }

  for (let i = available.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[available[i], available[j]] = [available[j], available[i]]
  }

  const results = available.slice(0, drawCount)
  currentNumbers.value = results
  isDrawing.value = false

  props.winners[currentPrizeId.value].push(...results)
}

const redraw = () => {
  if (isDrawing.value || !currentPrize.value || !currentPrizeId.value) return

  const perDraw = currentPrize.value.perDraw || 1
  if (perDraw !== 1) {
    alert('仅支持在“每次抽奖数量=1”时重抽')
    return
  }

  const currentPrizeWinners = props.winners[currentPrizeId.value] || []
  if (currentPrizeWinners.length === 0) {
    alert('当前奖项还没有可重抽的中奖号码')
    return
  }

  const previousWinner = currentPrizeWinners.pop()
  const allDrawn = getAllDrawnNumbers()
  const candidates = []
  for (let i = 1; i <= props.totalParticipants; i++) {
    if (!allDrawn.includes(i) && i !== previousWinner) candidates.push(i)
  }

  if (candidates.length === 0) {
    currentPrizeWinners.push(previousWinner)
    alert('没有可用的新号码可重抽')
    return
  }

  redrawPreviousWinner.value = previousWinner
  isRedrawing.value = true
  isDrawing.value = true

  drawInterval = setInterval(() => {
    currentNumbers.value = [candidates[Math.floor(Math.random() * candidates.length)]]
  }, 50)
}

onUnmounted(() => {
  if (drawInterval) clearInterval(drawInterval)
})
</script>

<template>
  <div class="glass-panel" style="max-width: 800px; text-align: center; position: relative;">
    <button class="btn btn-danger" style="position: absolute; top: 1rem; left: 1rem;" @click="emit('home')">
      返回设置
    </button>
    <button
      v-if="!isDrawing && currentPrize && (currentPrize.perDraw || 1) === 1 && drawnNumbersForCurrent.length > 0"
      class="btn btn-danger"
      style="position: absolute; top: 1rem; right: 1rem;"
      @click="redraw"
    >
      重抽
    </button>

    <h2>抽奖环节</h2>

    <div style="margin: 1.5rem 0;">
      <select v-model="currentPrizeId" style="font-size: 1.2rem; padding: 0.5rem 1rem; width: 60%; display: inline-block;">
        <option v-for="prize in prizes" :key="prize.id" :value="prize.id">
          {{ prize.name }} (共 {{ prize.count }} 名)
        </option>
      </select>
    </div>

    <div v-if="currentPrize" style="color: #a5b4fc; font-weight: bold;">
      当前抽取: {{ currentPrize.name }} | 剩余名额: {{ remainingCount }}
    </div>

    <div class="rolling-number-container" style="flex-wrap: wrap; gap: 2rem;">
      <div v-for="(num, idx) in currentNumbers" :key="idx" class="rolling-number" :class="{ active: isDrawing }">
        {{ num }}
      </div>
    </div>

    <div style="margin: 2rem 0;">
      <button v-if="!isDrawing" class="btn btn-primary" style="font-size: 1.5rem; padding: 1rem 3rem;" @click="startDraw" :disabled="isCurrentPrizeFinished">
        开始抽奖
      </button>
      <button v-else class="btn btn-danger" style="font-size: 1.5rem; padding: 1rem 3rem; background: var(--danger-hover);" @click="stopDraw">
        停止
      </button>
    </div>

    <div v-if="currentPrize && drawnNumbersForCurrent.length > 0">
      <h3 style="margin-bottom: 0.5rem;">中奖名单</h3>
      <div class="winner-list" style="justify-content: center;">
        <span v-for="num in drawnNumbersForCurrent" :key="num" class="winner-badge">
          {{ num }}
        </span>
      </div>
    </div>
  </div>
</template>
