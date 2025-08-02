<template>
  <div v-if="loading">Loading...</div>
  <div v-else>
    <div v-for="(player, index) in players" :key="index" class="mb-4 p-4 bg-gray-800 rounded-lg">
      <p><strong>#{{ index + 1 }} {{ player.name || 'Anonymous' }}</strong></p>
      <p>Win Streak: {{ player.win_streak }}</p>
      <p>Last Match: {{ new Date(player.last_match_time * 1000).toLocaleString() }}</p>
      <p>Most Played Hero: {{ heroNames[player.spam_hero] || 'Unknown Hero' }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const players = ref([])
const heroNames = ref({})
const loading = ref(true)

async function fetchTopPlayers() {
  loading.value = true

  // 1. Get hero names
  const heroRes = await axios.get('https://api.opendota.com/api/heroes')
  heroRes.data.forEach(hero => {
    heroNames.value[hero.id] = hero.localized_name
  })

  // 2. Get top 100 players by win streak
  const res = await axios.get('https://api.opendota.com/api/playersByRank')
  const topPlayers = res.data
    .filter(p => p.rank_tier !== null)
    .slice(0, 100)

  // 3. Fetch player details to find win streak, last match, and most spammed hero
  const detailedPlayers = await Promise.all(topPlayers.slice(0, 50).map(async (p) => {
    try {
      const [wlRes, matchesRes, heroesRes, profileRes] = await Promise.all([
        axios.get(`https://api.opendota.com/api/players/${p.account_id}/wl`),
        axios.get(`https://api.opendota.com/api/players/${p.account_id}/matches?limit=1`),
        axios.get(`https://api.opendota.com/api/players/${p.account_id}/heroes`),
        axios.get(`https://api.opendota.com/api/players/${p.account_id}`),
      ])

      const lastMatch = matchesRes.data[0]
      const spamHero = heroesRes.data.sort((a, b) => b.games - a.games)[0]

      return {
        name: profileRes.data.profile?.personaname,
        last_match_time: lastMatch?.start_time,
        win_streak: p.win_streak || 0,
        spam_hero: spamHero?.hero_id,
      }
    } catch (err) {
      return null
    }
  }))

  players.value = detailedPlayers.filter(Boolean).sort((a, b) => b.win_streak - a.win_streak).slice(0, 20)
  loading.value = false
}

onMounted(fetchTopPlayers)
</script>
