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
      <button @click="goBack" class="back-button">← Back</button>

      <div class="match-container">
        <div class="team radiant-team">
          <h2 :class="{ winner: match.radiant_win, loser: !match.radiant_win }">
            Radiant <span v-if="match.radiant_win"> Victory</span>
          </h2>

          <ul>
            <li v-for="player in radiantPlayers" :key="player.account_id">
              <strong>Name:</strong> {{ player.personaname || 'Anonymous' }}<br />
              <strong>Hero:</strong>
              <img
                v-if="heroMap[player.hero_id]"
                :src="`https://cdn.cloudflare.steamstatic.com/apps/dota2/images/dota_react/heroes/${heroMap[player.hero_id]}_full.png`"
                :alt="heroMap[player.hero_id]"
                width="80"
                style="vertical-align: middle;"
              /><br />
              <strong>K/D/A:</strong> {{ player.kills }}/{{ player.deaths }}/{{ player.assists }}<br />
              <strong>GPM:</strong> {{ player.gold_per_min }}<br />
              <strong>XPM:</strong> {{ player.xp_per_min }}<br />
              <strong>L/H:</strong> {{ player.last_hits }}/{{ player.denies }}<br />
              <strong>Hero Damage:</strong> {{ player.hero_damage }}<br />
              <strong>Tower Damage:</strong> {{ player.tower_damage }}<br />
              <strong>Hero Healing:</strong> {{ player.hero_healing }}
            </li>
          </ul>
        </div>

        <div class="vs-center">
          <h1 class="vs-text">VS</h1>
        </div>

        <div class="team dire-team">
          <h2 :class="{ winner: !match.radiant_win, loser: match.radiant_win }">
            Dire <span v-if="!match.radiant_win"> Victory</span>
          </h2>

          <ul>
            <li v-for="player in direPlayers" :key="player.account_id">
              <strong>Name:</strong> {{ player.personaname || 'Anonymous' }}<br />
              <strong>Hero:</strong>
              <img
                v-if="heroMap[player.hero_id]"
                :src="`https://cdn.cloudflare.steamstatic.com/apps/dota2/images/dota_react/heroes/${heroMap[player.hero_id]}_full.png`"
                :alt="heroMap[player.hero_id]"
                width="80"
                style="vertical-align: middle;"
              /><br />
              <strong>K/D/A:</strong> {{ player.kills }}/{{ player.deaths }}/{{ player.assists }}<br />
              <strong>GPM:</strong> {{ player.gold_per_min }}<br />
              <strong>XPM:</strong> {{ player.xp_per_min }}<br />
              <strong>L/H:</strong> {{ player.last_hits }}/{{ player.denies }}<br />
              <strong>Hero Damage:</strong> {{ player.hero_damage }}<br />
              <strong>Tower Damage:</strong> {{ player.tower_damage }}<br />
              <strong>Hero Healing:</strong> {{ player.hero_healing }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      matchId: '',
      match: null,
      radiantPlayers: [],
      direPlayers: [],
      heroMap: {},
    };
  },
  methods: {
    async fetchMatch() {
      if (!this.matchId || isNaN(this.matchId)) {
        alert('Please enter a valid Match ID.');
        return;
      }

      try {
        // Fetch match data
        const response = await fetch(`https://api.opendota.com/api/matches/${this.matchId}`);
        if (!response.ok) throw new Error('Match not found');
        const matchData = await response.json();
        this.match = matchData;

        this.radiantPlayers = matchData.players.filter(p => p.player_slot < 128);
        this.direPlayers = matchData.players.filter(p => p.player_slot >= 128);

        // Fetch hero list
        const heroesResponse = await fetch('https://api.opendota.com/api/heroes');
        const heroes = await heroesResponse.json();
        this.heroMap = heroes.reduce((map, hero) => {
          map[hero.id] = hero.name.replace('npc_dota_hero_', '');
          return map;
        }, {});

        // Fetch player names
        for (let player of [...this.radiantPlayers, ...this.direPlayers]) {
          if (player.account_id) {
            try {
              const playerResponse = await fetch(`https://api.opendota.com/api/players/${player.account_id}`);
              const playerData = await playerResponse.json();
              player.personaname = playerData.profile?.personaname || 'Anonymous';
            } catch {
              player.personaname = 'Anonymous';
            }
          } else {
            player.personaname = 'Anonymous';
          }
        }
      } catch (error) {
        alert('Failed to fetch match data. Check Match ID or try again later.');
        console.error(error);
      }
    },
    goBack() {
      this.match = null;
      this.radiantPlayers = [];
      this.direPlayers = [];
    },
  },
};
</script>

<style>
.container {
  max-width: 800px;
  margin: 50px auto;
  padding: 20px;
  font-family: Arial, sans-serif;
  text-align: center;
}

label {
  display: block;
  font-size: 1.2em;
  margin-bottom: 8px;
}

.match-input {
  width: 100%;
  padding: 18px;
  font-size: 1.4em;
  border: 2px solid #888;
  border-radius: 6px;
  margin-bottom: 16px;
  box-sizing: border-box;
}

button {
  padding: 14px 28px;
  font-size: 1.2em;
  background-color: #3366cc;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

button:hover {
  background-color: #254c99;
}

.back-button {
  background-color: #888;
  margin-bottom: 20px;
  font-size: 1em;
}

.versus {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 3em;
  margin: 20px 0;
}

.vs-text {
  margin: 0 8px;
  font-weight: bold;
}

.match-container {
  display: flex;
  justify-content: space-between;
  gap: 24px;
  height: 100%;
}

.team {
  flex: 1;
  padding: 20px;
}

.team ul {
  list-style-type: none;
  padding: 0;
}

.team li {
  margin-bottom: 20px;
  border-bottom: 1px solid #ccc;
  padding-bottom: 10px;
}

.winner {
  color: #2bec2b;
  font-weight: bold;
  font-size: xxx-large;
   white-space: nowrap;
}

.loser {
  color: #d62424;
  font-weight: bold;
  font-size: xxx-large;
   white-space: nowrap;
}
</style>
