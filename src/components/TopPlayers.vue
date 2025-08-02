<template>
  <div class="p-4 bg-gray-900 text-white min-h-screen">
    <h1 class="text-2xl font-bold mb-4">🏆 Top Dota 2 Players by Rank & Server</h1>

    <div class="flex space-x-4 mb-6">
      <button
        v-for="srv in servers"
        :key="srv"
        :class="[
          'px-4 py-2 rounded-full transition',
          currentServer === srv
            ? 'bg-purple-600 text-white shadow-lg'
            : 'bg-gray-700 hover:bg-purple-500',
        ]"
        @click="currentServer = srv"
      >
        {{ srv }}
      </button>
    </div>

    <div class="flex space-x-4 mb-4">
      <button
        v-for="rank in ranks"
        :key="rank"
        :class="[
          'px-4 py-2 rounded-full transition',
          currentRank === rank
            ? 'bg-pink-500 text-white shadow'
            : 'bg-gray-600 hover:bg-pink-400',
        ]"
        @click="currentRank = rank"
      >
        {{ rank }}
      </button>
    </div>

    <div class="overflow-x-auto bg-gray-800 rounded-lg p-4">
      <table class="table-auto w-full text-left border-collapse">
        <thead>
          <tr class="bg-gray-700">
            <th class="px-4 py-2">Player Name</th>
            <th class="px-4 py-2">Last Match</th>
            <th class="px-4 py-2">Spam Hero</th>
            <th class="px-4 py-2">Win Streak</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="player in filteredPlayers"
            :key="player.account_id"
            class="hover:bg-gray-600 transition"
          >
            <td class="px-4 py-2">{{ player.name || 'Anonymous' }}</td>
            <td class="px-4 py-2">{{ player.last_match }}</td>
            <td class="px-4 py-2">{{ player.spam_hero }}</td>
            <td class="px-4 py-2 font-bold text-green-400">{{ player.win_streak }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import axios from 'axios'

// Sample values — you can replace this with real API when available
const servers = ['SEA', 'EU', 'NA']
const ranks = ['Herald', 'Guardian', 'Crusader', 'Archon', 'Legend', 'Ancient', 'Divine', 'Immortal']

const currentServer = ref(servers[0])
const currentRank = ref(ranks[0])
const players = ref([])

onMounted(async () => {
  // Use mock data for now
  players.value = await fetchMockPlayers()
})

const fetchMockPlayers = async () => {
  return Array.from({ length: 100 }).map((_, i) => ({
    account_id: i,
    name: `Player_${i + 1}`,
    last_match: `2025-08-${(i % 28) + 1}`,
    spam_hero: ['Invoker', 'Pudge', 'Sniper', 'Phantom Assassin'][i % 4],
    win_streak: Math.floor(Math.random() * 20),
    server: servers[i % servers.length],
    rank: ranks[i % ranks.length],
  }))
}

const filteredPlayers = computed(() =>
  players.value
    .filter((p) => p.server === currentServer.value && p.rank === currentRank.value)
    .sort((a, b) => b.win_streak - a.win_streak)
    .slice(0, 20)
)
</script>

<style scoped>
body {
  font-family: 'Segoe UI', sans-serif;
}
</style>
