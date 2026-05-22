<script setup>
import { ref, reactive } from 'vue'
import HomePage from './components/HomePage.vue'
import DrawPage from './components/DrawPage.vue'

const appState = ref('HOME') // 'HOME' or 'DRAW'
const totalParticipants = ref(100)
const prizes = reactive([
  { id: 1, name: '一等奖', count: 1, perDraw: 1 },
  { id: 2, name: '二等奖', count: 3, perDraw: 1 },
  { id: 3, name: '三等奖', count: 5, perDraw: 1 }
])

const winners = reactive({})

const backgrounds = [
  './bg/BIN_7499.JPG',
  './bg/BIN_7619.JPG',
  './bg/BIN_8426.JPG'
]

const currentBgIndex = ref(Math.floor(Math.random() * backgrounds.length))

const startGame = () => {
  if (totalParticipants.value < 1) {
    alert('总人数必须大于 0')
    return
  }
  if (prizes.length === 0) {
    alert('请至少设置一个奖项')
    return
  }

  for (const key in winners) {
    delete winners[key]
  }
  prizes.forEach(p => {
    winners[p.id] = []
  })

  appState.value = 'DRAW'
}

const goHome = () => {
  appState.value = 'HOME'
}

const changeBg = () => {
  currentBgIndex.value = (currentBgIndex.value + 1) % backgrounds.length
}
</script>

<template>
  <img :src="backgrounds[currentBgIndex]" class="bg-layer" alt="background" />
  <div class="bg-overlay"></div>

  <div style="position: absolute; top: 1rem; right: 1rem; z-index: 100;">
    <button class="btn btn-success" @click="changeBg" style="padding: 0.5rem 1rem; font-size: 0.875rem;">
      切换背景
    </button>
  </div>

  <HomePage
    v-if="appState === 'HOME'"
    v-model:totalParticipants="totalParticipants"
    :prizes="prizes"
    @start="startGame"
  />

  <DrawPage
    v-else-if="appState === 'DRAW'"
    :totalParticipants="totalParticipants"
    :prizes="prizes"
    :winners="winners"
    @home="goHome"
  />
</template>
