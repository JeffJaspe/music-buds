<template>
  <div>
    <div class="flex flex-wrap justify-center gap-2 p-4">
      <button
        v-for="div in divisions"
        :key="div"
        @click="selected = div"
        :class="['text-xl px-6 py-3 rounded-2xl shadow-lg', selected === div ? 'bg-blue-600 text-white' : 'bg-slate-800 text-white']"
      >
        {{ div.toUpperCase() }}
      </button>
    </div>

    <div v-for="div in divisions" v-if="selected === div" :key="div" class="p-4">
      <div v-for="(players, rank) in groupByRank(data[div] || [])" :key="rank" class="mb-8">
        <h2 :class="['text-2xl font-bold mb-2', rankColors[rank]]">{{ rank }}</h2>
        <table class="min-w-full bg-white rounded shadow-md">
          <thead class="bg-slate-200">
            <tr>
              <th class="px-4 py-2 text-left">Rank</th>
              <th class="px-4 py-2 text-left">Name</th>
              <th class="px-4 py-2 text-left">MMR</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(p, idx) in players"
              :key="p.account_id"
              class="border-b hover:bg-slate-100"
            >
              <td class="px-4 py-2">#{{ idx + 1 }}</td>
              <td class="px-4 py-2">{{ p.name || "Anonymous" }}</td>
              <td class="px-4 py-2">{{ p.mmr }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue"
import axios from "axios"

const divisions = ["americas", "europe", "se_asia", "china"]
const selected = ref("americas")
const data = ref({})

const rankMap = {
  10: "Herald",
  20: "Guardian",
  30: "Crusader",
  40: "Archon",
  50: "Legend",
  60: "Ancient",
  70: "Divine",
  80: "Immortal",
}

const rankColors = {
  Herald: "bg-gray-500",
  Guardian: "bg-green-500",
  Crusader: "bg-blue-500",
  Archon: "bg-indigo-500",
  Legend: "bg-yellow-500",
  Ancient: "bg-orange-500",
  Divine: "bg-red-500",
  Immortal: "bg-purple-700",
}

function groupByRank(players) {
  const grouped = {}
  players.forEach((player) => {
    const tier = Math.floor((player.rank_tier || 0) / 10) * 10
    const label = rankMap[tier] || "Unranked"
    if (!grouped[label]) grouped[label] = []
    grouped[label].push(player)
  })
  return grouped
}

onMounted(() => {
  divisions.forEach(async (division) => {
    try {
      const res = await axios.get(
        `https://api.opendota.com/api/leaderboards?division=${division}`
      )
      data.value[division] = res.data.leaderboard || []
    } catch (err) {
      console.error(`Error fetching ${division}`, err)
    }
  })
})
</script>
