<template>
  <div class="p-6 space-y-6 bg-gray-100 min-h-screen">
    <div class="flex gap-4 items-center">
      <input
        v-model="matchId"
        type="text"
        placeholder="Enter Match ID"
        class="px-4 py-2 border rounded w-64"
      />
      <button @click="fetchMatch" class="bg-blue-600 text-white px-4 py-2 rounded">
        Fetch Match
      </button>
    </div>

    <div v-if="match" class="mt-6">
      <div class="flex justify-between">
        <!-- Radiant Team -->
        <div class="w-1/2 p-4 text-center" :class="match.radiant_win ? 'bg-green-200' : 'bg-red-200'">
          <h2 class="text-3xl font-bold">Radiant</h2>
          <p class="text-xl font-semibold mt-2" :class="match.radiant_win ? 'text-green-800' : 'text-red-800'">
            {{ match.radiant_win ? 'WINNER' : 'LOSER' }}
          </p>
          <ul class="mt-4 space-y-2">
            <li v-for="player in radiantPlayers" :key="player.account_id">
              {{ player.personaname || 'Anonymous' }} ({{ player.kills }}/{{ player.deaths }}/{{ player.assists }})
            </li>
          </ul>
        </div>

        <!-- Dire Team -->
        <div class="w-1/2 p-4 text-center" :class="!match.radiant_win ? 'bg-green-200' : 'bg-red-200'">
          <h2 class="text-3xl font-bold">Dire</h2>
          <p class="text-xl font-semibold mt-2" :class="!match.radiant_win ? 'text-green-800' : 'text-red-800'">
            {{ !match.radiant_win ? 'WINNER' : 'LOSER' }}
          </p>
          <ul class="mt-4 space-y-2">
            <li v-for="player in direPlayers" :key="player.account_id">
              {{ player.personaname || 'Anonymous' }} ({{ player.kills }}/{{ player.deaths }}/{{ player.assists }})
            </li>
          </ul>
        </div>
      </div>

      <!-- Match Details -->
      <div class="mt-6 bg-white p-4 rounded shadow">
        <h3 class="text-xl font-semibold mb-2">Match Details</h3>
        <p><strong>Match ID:</strong> {{ match.match_id }}</p>
        <p><strong>Duration:</strong> {{ Math.floor(match.duration / 60) }} min</p>
        <p><strong>Radiant Score:</strong> {{ match.radiant_score }}</p>
        <p><strong>Dire Score:</strong> {{ match.dire_score }}</p>
        <p><strong>Game Mode:</strong> {{ match.game_mode }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const matchId = ref('');
const match = ref(null);
const radiantPlayers = ref([]);
const direPlayers = ref([]);

const fetchMatch = async () => {
  if (!matchId.value) return;
  try {
    const res = await axios.get(`https://api.opendota.com/api/matches/${matchId.value}`);
    match.value = res.data;
    radiantPlayers.value = res.data.players.filter(p => p.isRadiant);
    direPlayers.value = res.data.players.filter(p => !p.isRadiant);
  } catch (error) {
    console.error('Failed to fetch match:', error);
    match.value = null;
    radiantPlayers.value = [];
    direPlayers.value = [];
  }
};
</script>

<style>
body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}
</style>
