<template>
  <div class="container">
    <div v-if="!match">
      <label for="matchId">Enter Match ID for public matches only</label>
      <input
        v-model="matchId"
        type="text"
        id="matchId"
        class="match-input"
        placeholder="e.g. 1234567890"
      />
      <button @click="fetchMatch">Search Match</button>
    </div>

    <div v-else>
      <h2>Match ID: {{ match.match_id }}</h2>
      <p>
        Duration: {{ (match.duration / 60).toFixed(2) }} minutes<br />
        Winner: <strong>{{ match.radiant_win ? 'Radiant Victory' : 'Dire Victory' }}</strong>
      </p>

      <div class="team">
        <h3>Radiant - Total Kills: {{ radiantKills }}</h3>
        <ul>
          <li v-for="player in radiantPlayers" :key="player.account_id">
            <img
              :src="getHeroIcon(player.hero_id)"
              :alt="heroMap[player.hero_id] || 'Unknown Hero'"
              width="80"
              @error="onImageError"
            />
            <span>{{ player.personaname || 'Anonymous' }}</span>
          </li>
        </ul>
      </div>

      <div class="team">
        <h3>Dire - Total Kills: {{ direKills }}</h3>
        <ul>
          <li v-for="player in direPlayers" :key="player.account_id">
            <img
              :src="getHeroIcon(player.hero_id)"
              :alt="heroMap[player.hero_id] || 'Unknown Hero'"
              width="80"
              @error="onImageError"
            />
            <span>{{ player.personaname || 'Anonymous' }}</span>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue'

export default {
  setup() {
    const matchId = ref('')
    const match = ref(null)
    const radiantPlayers = ref([])
    const direPlayers = ref([])
    const heroMap = ref({})
    const dummyImage = 'https://via.placeholder.com/80x45?text=No+Image'

    const onImageError = (e) => {
      e.target.src = dummyImage
    }

    const getHeroIcon = (heroId) => {
      const heroName = heroMap.value[heroId]
      return heroName
        ? `https://cdn.cloudflare.steamstatic.com/apps/dota2/images/dota_react/heroes/${heroName}_full.png`
        : dummyImage
    }

    const fetchMatch = async () => {
      if (!matchId.value) return
      try {
        const res = await fetch(`https://api.opendota.com/api/matches/${matchId.value}`)
        match.value = await res.json()
        radiantPlayers.value = match.value.players.slice(0, 5)
        direPlayers.value = match.value.players.slice(5, 10)
      } catch (err) {
        console.error('Failed to fetch match data:', err)
      }
    }

    const fetchHeroes = async () => {
      try {
        const res = await fetch('https://api.opendota.com/api/heroStats')
        const heroes = await res.json()
        heroes.forEach(hero => {
          heroMap.value[hero.id] = hero.name.replace('npc_dota_hero_', '')
        })
      } catch (err) {
        console.error('Failed to fetch hero stats:', err)
      }
    }

    const radiantKills = computed(() =>
      radiantPlayers.value.reduce((sum, p) => sum + (p.kills || 0), 0)
    )
    const direKills = computed(() =>
      direPlayers.value.reduce((sum, p) => sum + (p.kills || 0), 0)
    )

    onMounted(() => {
      fetchHeroes()
    })

    return {
      matchId,
      match,
      radiantPlayers,
      direPlayers,
      heroMap,
      fetchMatch,
      getHeroIcon,
      onImageError,
      radiantKills,
      direKills,
    }
  },
}
</script>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}

label {
  display: inline-block;
  white-space: nowrap;
  margin-bottom: 0.5rem;
}

.match-input {
  margin: 0.5em 0 1em 0;
  padding: 0.5em;
  width: 100%;
  max-width: 300px;
}

button {
  padding: 0.5em 1em;
  cursor: pointer;
}

.team {
  margin-top: 2em;
}

.team h3 {
  font-size: 1.25rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
}

.team ul {
  list-style: none;
  padding: 0;
}

.team li {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  margin-bottom: 0.75rem;
}

.team img {
  border-radius: 6px;
  background: #eee;
}
</style>
