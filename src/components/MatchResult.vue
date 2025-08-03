<template>
  <div class="viewer">
    <div v-if="!match">
      <label for="matchId">Enter Match ID for public matches only</label>
      <input
        id="matchId"
        v-model="matchId"
        placeholder="Match ID"
        class="match-input"
      />
      <button @click="fetchMatch">Search</button>
      <p v-if="error" class="error">{{ error }}</p>
    </div>

    <div v-else>
      <div class="versus">
        <div :class="match.radiant_win ? 'winner' : 'loser'">
          Radiant {{ match.radiant_win ? 'Victory' : '' }}
        </div>
        <div>vs</div>
        <div :class="!match.radiant_win ? 'winner' : 'loser'">
          Dire {{ !match.radiant_win ? 'Victory' : '' }}
        </div>
      </div>

      <div class="match-info">
        <p><strong>Match ID:</strong> {{ match.match_id }}</p>
        <p><strong>Duration:</strong> {{ formatDuration(match.duration) }}</p>
        <p><strong>Game Mode:</strong> {{ match.game_mode }}</p>
        <p><strong>Start Time:</strong> {{ formatDate(match.start_time) }}</p>
      </div>

      <div class="teams">
        <div class="team">
          <h3>Radiant</h3>
          <ul>
            <li v-for="player in radiantPlayers" :key="player.account_id">
              Hero ID: {{ player.hero_id }} |
              Player: {{ player.personaname || 'Anonymous' }} |
              Score: {{ getHeroScore(player.account_id, player.hero_id) }}
            </li>
          </ul>
        </div>
        <div class="team">
          <h3>Dire</h3>
          <ul>
            <li v-for="player in direPlayers" :key="player.account_id">
              Hero ID: {{ player.hero_id }} |
              Player: {{ player.personaname || 'Anonymous' }} |
              Score: {{ getHeroScore(player.account_id, player.hero_id) }}
            </li>
          </ul>
        </div>
      </div>

      <div class="bans">
        <h3>Picks & Bans</h3>
        <ul>
          <li v-for="ban in match.picks_bans" :key="ban.order">
            {{ ban.is_pick ? 'Pick' : 'Ban' }} - Hero ID: {{ ban.hero_id }} ({{ ban.team === 0 ? 'Radiant' : 'Dire' }})
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted, computed  } from 'vue'
import axios from 'axios'



const matchId = ref('')
const match = ref(null)
const error = ref('')
const rankingsCache = {}

const fetchMatch = async () => {
  if (!matchId.value || isNaN(matchId.value)) {
    error.value = 'Please enter a valid Match ID'
    return
  }
  error.value = ''
  match.value = null

  try {
    const res = await axios.get(`https://api.opendota.com/api/matches/${matchId.value}`)
    match.value = res.data
    await fetchAllRankings()
  } catch (err) {
    error.value = 'Match not found or not public'
  }
}

const fetchAllRankings = async () => {
  const allPlayers = match.value.players
  for (const player of allPlayers) {
    if (!player.account_id || player.account_id === 4294967295) continue
    if (!rankingsCache[player.account_id]) {
      try {
        const { data } = await axios.get(`https://api.opendota.com/api/players/${player.account_id}/rankings`)
        rankingsCache[player.account_id] = data
      } catch {
        rankingsCache[player.account_id] = []
      }
    }
  }
}

const getHeroScore = (accountId, heroId) => {
  const playerRankings = rankingsCache[accountId]
  if (!playerRankings) return 'N/A'
  const heroRank = playerRankings.find(r => r.hero_id === heroId)
  return heroRank ? heroRank.score : 'N/A'
}

const radiantPlayers = computed(() => match.value?.players?.filter(p => p.isRadiant))
const direPlayers = computed(() => match.value?.players?.filter(p => !p.isRadiant))

const formatDuration = seconds => {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${mins}m ${secs}s`
}

const formatDate = timestamp => {
  const date = new Date(timestamp * 1000)
  return date.toLocaleString()
}
</script>

<style>
.viewer {
  font-family: Arial, sans-serif;
  padding: 1rem;
  max-width: 900px;
  margin: auto;
}
.match-input {
  font-size: 1.5rem;
  padding: 0.5rem;
  width: 100%;
  margin: 1rem 0;
}
button {
  font-size: 1.2rem;
  padding: 0.5rem 1rem;
}
.error {
  color: red;
}
.versus {
  display: flex;
  justify-content: space-around;
  align-items: center;
  font-size: 3rem;
  margin: 2rem 0;
}
.winner {
  color: #1ccc33; /* Apple Green */
  font-weight: bold;
}
.loser {
  color: #d63c3c; /* Maroon Red */
  opacity: 0.7;
}
.match-info,
.teams,
.bans {
  margin: 1rem 0;
}
.team {
  margin-bottom: 1rem;
}
</style>
