<template>
  <div class="container">
    <label for="matchId">Enter Match ID for public matches only</label>
    <input
      v-model="matchId"
      type="text"
      id="matchId"
      class="match-input"
      placeholder="e.g. 1234567890"
    />
    <button @click="fetchMatch">Search Match</button>

    <div v-if="match">
      <h2>Match ID: {{ match.match_id }}</h2>
      <p>Radiant Score: {{ match.radiant_score }}</p>
      <p>Dire Score: {{ match.dire_score }}</p>
      <p>Winner: {{ match.radiant_win ? 'Radiant' : 'Dire' }}</p>
      <p>Game Mode: {{ match.game_mode }}</p>
      <p>Lobby Type: {{ match.lobby_type }}</p>
      <p>Duration: {{ Math.floor(match.duration / 60) }} minutes</p>
      <p>Start Time: {{ new Date(match.start_time * 1000).toLocaleString() }}</p>

      <h3>Players</h3>
      <ul>
        <li v-for="player in match.players" :key="player.account_id">
          Hero ID: {{ player.hero_id }} - K/D/A: {{ player.kills }}/{{ player.deaths }}/{{ player.assists }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      matchId: '',
      match: null,
    };
  },
  methods: {
    async fetchMatch() {
      if (!this.matchId || isNaN(this.matchId)) {
        alert("Please enter a valid Match ID.");
        return;
      }

      try {
        const response = await fetch(`https://api.opendota.com/api/matches/${this.matchId}`);
        if (!response.ok) {
          throw new Error('Match not found');
        }
        this.match = await response.json();
      } catch (error) {
        alert("Failed to fetch match data. Check Match ID or try again later.");
        console.error(error);
      }
    },
  },
};
</script>

<style>
.container {
  max-width: 600px;
  margin: 50px auto;
  padding: 20px;
  font-family: Arial, sans-serif;
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
</style>
