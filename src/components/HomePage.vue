<script setup>
const props = defineProps({
  totalParticipants: Number,
  prizes: Array
})

const emit = defineEmits(['update:totalParticipants', 'start'])

const onTotalChange = (e) => {
  emit('update:totalParticipants', parseInt(e.target.value) || 0)
}

const addPrize = () => {
  const newId = props.prizes.length > 0 ? Math.max(...props.prizes.map(p => p.id)) + 1 : 1
  props.prizes.push({ id: newId, name: '新奖项', count: 1, perDraw: 1 })
}

const removePrize = (index) => {
  props.prizes.splice(index, 1)
}

const confirmAndStart = () => {
  emit('start')
}
</script>

<template>
  <div class="glass-panel">
    <h1>婚礼抽奖系统</h1>

    <div class="form-group">
      <label>参与总人数</label>
      <input
        type="number"
        :value="totalParticipants"
        @input="onTotalChange"
        min="1"
      />
    </div>

    <div class="form-group">
      <div style="display: flex; justify-content: space-between; align-items: center; margin-bottom: 0.5rem;">
        <label style="margin-bottom: 0;">奖项设置</label>
        <button class="btn btn-success" style="padding: 0.25rem 0.75rem; font-size: 0.875rem;" @click="addPrize">
          + 添加奖项
        </button>
      </div>

      <div v-for="(prize, index) in prizes" :key="prize.id" class="prize-item">
        <div class="prize-inputs" style="flex-wrap: wrap; gap: 0.5rem; align-items: center;">
          <input type="text" v-model="prize.name" placeholder="奖项名称（如：一等奖）" style="flex: 2; min-width: 120px;" />
          <div style="display: flex; gap: 0.5rem; flex: 2; align-items: center; background: rgba(0,0,0,0.2); padding: 0.5rem; border-radius: 8px;">
            <span style="font-size: 0.85rem; color: #ccc; white-space: nowrap;">总名额</span>
            <input type="number" v-model="prize.count" min="1" style="width: 60px; padding: 0.5rem; background: rgba(0,0,0,0.3);" />
            <span style="font-size: 0.85rem; color: #ccc; white-space: nowrap; margin-left: 0.5rem;">每次抽</span>
            <input type="number" v-model="prize.perDraw" min="1" style="width: 60px; padding: 0.5rem; background: rgba(0,0,0,0.3);" />
          </div>
        </div>
        <button class="btn btn-danger" @click="removePrize(index)">删除</button>
      </div>
    </div>

    <button class="btn btn-primary" style="margin-top: 1rem; padding: 1rem; font-size: 1.2rem;" @click="confirmAndStart">
      进入抽奖
    </button>
  </div>
</template>
