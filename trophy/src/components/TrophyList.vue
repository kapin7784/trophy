<template>
  <div class="container">
    <h1 class="title">🏆 人生トロフィー 🏆</h1>

    <div class="trophy-section" v-for="(trophies, rank) in groupedTrophies" :key="rank">
      <h2 class="rank-title">{{ rankLabels[rank] }}</h2>
      <ul class="trophy-list">
        <li v-for="trophy in trophies" :key="trophy.id" class="trophy-item">
          <div class="trophy-content">
            <input
              type="checkbox"
              :checked="isChecked(trophy.id)"
              :disabled="trophy.id === 'platinum'"
              @change="toggleTrophy(trophy.id)"
              @click.stop
              class="checkbox"
            />
            <span @click="editTrophyName(trophy)" class="trophy-name">{{ trophy.label }}</span>
          </div>

          <div v-if="editingTrophy && editingTrophy.id === trophy.id" class="edit-section">
            <input
              v-model="editingTrophy.newName"
              class="edit-input"
              placeholder="新しい名前を入力"
            />
            <div class="edit-buttons">
              <button class="btn btn-save" @click="saveTrophyName(trophy)">保存</button>
              <button class="btn btn-cancel" @click="cancelEdit">キャンセル</button>
            </div>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'

const TROPHIES = [
  { id: 'platinum', label: '他の全トロフィーを達成', rank: 'platinum' },
  ...Array.from({ length: 3 }, (_, i) => ({ id: `gold-${i + 1}`, label: `ゴールド ${i + 1}`, rank: 'gold' })),
  ...Array.from({ length: 5 }, (_, i) => ({ id: `silver-${i + 1}`, label: `シルバー ${i + 1}`, rank: 'silver' })),
  ...Array.from({ length: 10 }, (_, i) => ({ id: `bronze-${i + 1}`, label: `ブロンズ ${i + 1}`, rank: 'bronze' })),
]

const rankLabels = {
  platinum: '🌈 プラチナ',
  gold: '🥇 ゴールド',
  silver: '🥈 シルバー',
  bronze: '🥉 ブロンズ'
}

const checkedTrophies = ref(new Set())
const editingTrophy = ref(null)

onMounted(() => {
  const saved = localStorage.getItem('trophies')
  if (saved) checkedTrophies.value = new Set(JSON.parse(saved))
  updatePlatinum()

  const savedNames = localStorage.getItem('trophyNames')
  if (savedNames) {
    const trophyNames = JSON.parse(savedNames)
    TROPHIES.forEach(t => {
      if (trophyNames[t.id]) t.label = trophyNames[t.id]
    })
  }
})

function toggleTrophy(id) {
  checkedTrophies.value.has(id) ? checkedTrophies.value.delete(id) : checkedTrophies.value.add(id)
  updatePlatinum()
}

function updatePlatinum() {
  const ids = TROPHIES.filter(t => t.id !== 'platinum').map(t => t.id)
  const allChecked = ids.every(id => checkedTrophies.value.has(id))
  if (allChecked) checkedTrophies.value.add('platinum')
  else checkedTrophies.value.delete('platinum')
}

function isChecked(id) {
  return checkedTrophies.value.has(id)
}

function editTrophyName(trophy) {
  editingTrophy.value = { id: trophy.id, newName: trophy.label }
}

function saveTrophyName(trophy) {
  if (editingTrophy.value?.newName.trim()) {
    trophy.label = editingTrophy.value.newName.trim()
    localStorage.setItem('trophyNames', JSON.stringify({
      ...getCurrentTrophyNames(),
      [trophy.id]: trophy.label
    }))
  }
  editingTrophy.value = null
}

function getCurrentTrophyNames() {
  const map = {}
  TROPHIES.forEach(t => map[t.id] = t.label)
  return map
}

watch(checkedTrophies, () => {
  localStorage.setItem('trophies', JSON.stringify([...checkedTrophies.value]))
}, { deep: true })

const groupedTrophies = computed(() => {
  return TROPHIES.reduce((acc, trophy) => {
    if (!acc[trophy.rank]) acc[trophy.rank] = []
    acc[trophy.rank].push(trophy)
    return acc
  }, {})
})

function cancelEdit() {
  editingTrophy.value = null
}
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 2rem auto;
  padding: 2rem;
  font-family: 'Segoe UI', sans-serif;
  background: linear-gradient(135deg, #fdfbfb, #ebedee);
  border-radius: 12px;
  box-shadow: 0 12px 24px rgba(0, 0, 0, 0.1);
}

.title {
  text-align: center;
  font-size: 2.5rem;
  color: #2c3e50;
  margin-bottom: 2rem;
  font-weight: 700;
  letter-spacing: 1px;
}

.rank-title {
  font-size: 1.75rem;
  color: #555;
  margin: 2.5rem 0 1rem;
  padding-left: 0.5rem;
}

.trophy-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.trophy-item {
  background: #fff;
  margin-bottom: 1rem;
  padding: 1rem;
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.05);
  transition: transform 0.2s ease;
}

.trophy-item:hover {
  transform: scale(1.01);
}

.trophy-content {
  display: flex;
  align-items: center;
  justify-content: flex-start;
}

.checkbox {
  margin-right: 1rem;
  width: 20px;
  height: 20px;
  accent-color: #42b983;
  cursor: pointer;
}

.trophy-name {
  font-size: 1.2rem;
  font-weight: 600;
  color: #333;
  cursor: pointer;
  flex-grow: 1;
  transition: color 0.3s ease;
}

.trophy-name:hover {
  color: #42b983;
}

.edit-section {
  margin-top: 0.5rem;
  width: 100%;
}

.edit-input {
  width: 100%;
  font-size: 1rem;
  padding: 0.6rem;
  border-radius: 6px;
  border: 1px solid #ccc;
  outline: none;
  transition: border-color 0.3s ease;
}

.edit-input:focus {
  border-color: #42b983;
}

.edit-buttons {
  margin-top: 0.5rem;
  display: flex;
  gap: 0.5rem;
}

.btn {
  padding: 0.4rem 0.8rem;
  font-size: 0.9rem;
  border-radius: 5px;
  border: none;
  cursor: pointer;
  transition: background 0.2s ease;
}

.btn-save {
  background-color: #42b983;
  color: white;
}

.btn-save:hover {
  background-color: #369d6b;
}

.btn-cancel {
  background-color: #ccc;
  color: #333;
}

.btn-cancel:hover {
  background-color: #bbb;
}

</style>
