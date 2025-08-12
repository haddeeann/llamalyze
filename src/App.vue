<script setup>
import { ref, computed, onMounted } from 'vue'

// use env when available, default to your Fly URL
const API_BASE = import.meta.env.VITE_API_BASE || 'https://llamalyze-api.fly.dev'

console.log(API_BASE)

const boards = ref([])
const loading = ref(true)
const error = ref(null)

async function loadBoards() {
  try {
    const res = await fetch(`${API_BASE}/api/boards`, {
      headers: { Accept: 'application/json' }
    })
    if (!res.ok) throw new Error(`HTTP ${res.status}`)
    boards.value = await res.json()
  } catch (e) {
    error.value = e.message || String(e)
  } finally {
    loading.value = false
  }
}

onMounted(loadBoards)

// group by category, keep a preferred display order
const preferredOrder = [
  'General Remote Job Boards',
  'Freelance & Gig-Based Platforms',
  'Tech & Startup Remote Jobs',
  'Writing & Design Remote Jobs',
  'Niche Remote Job Boards'
]

const grouped = computed(() => {
  const byCat = boards.value.reduce((acc, b) => {
    const cat = b.category || 'Other'
    ;(acc[cat] ||= []).push(b)
    return acc
  }, {})
  // sort names within each category
  for (const arr of Object.values(byCat)) {
    arr.sort((a, b) => a.name.localeCompare(b.name))
  }
  // build ordered list of [category, items]
  const ordered = []
  for (const cat of preferredOrder) if (byCat[cat]) ordered.push([cat, byCat[cat]])
  for (const cat of Object.keys(byCat)) if (!preferredOrder.includes(cat)) ordered.push([cat, byCat[cat]])
  return ordered
})
</script>

<template>
  <div class="job-boards">
    <p v-if="loading">Loading boards…</p>
    <p v-else-if="error">Error: {{ error }}</p>

    <section v-else v-for="[category, items] in grouped" :key="category">
      <h2>{{ category }}</h2>

      <div v-for="b in items" :key="b.id || b.name" class="job-board">
        <h3>
          <a :href="b.url" target="_blank" rel="noopener">{{ b.name }}</a>
        </h3>
        <p>{{ b.description || 'No description yet.' }}</p>
      </div>
    </section>
  </div>
</template>

<style>
.job-boards { max-width: 800px; margin: 2rem auto; padding: 0 1rem; }
.job-board { margin: 0 0 1rem; }
.job-board h3 { margin: 0 0 .25rem; }
</style>
